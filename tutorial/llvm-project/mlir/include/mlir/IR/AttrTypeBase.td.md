# AttrTypeBase.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `mlir/include/mlir/IR/AttrTypeBase.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: Defines TableGen records for the MLIR AttrTypeBase component, including operation metadata, traits, constraints, and textual assembly rules. The leading comments describe it as: This file contains the base set of constructs for defining Attribute and.
- **用途（CN）**: 为 MLIR 的 AttrTypeBase 组件定义 TableGen 记录，包括操作元数据、trait、约束以及文本汇编格式规则。 文件开头的注释还对该职责进行了补充说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-30
````tablegen
//===-- AttrTypeBase.td - Base Attr/Type definition file ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the base set of constructs for defining Attribute and
// Type classes.
//
//===----------------------------------------------------------------------===//

#ifndef ATTRTYPEBASE_TD
#define ATTRTYPEBASE_TD

include "mlir/IR/CommonAttrConstraints.td"
include "mlir/IR/CommonTypeConstraints.td"
include "mlir/IR/Constraints.td"
include "mlir/IR/DialectBase.td"
include "mlir/IR/Traits.td"

//-------------------------------------------------------------------------===//
// AttrTrait definitions
//===----------------------------------------------------------------------===//

// These classes are used to define attribute specific traits.

// Specify attribute specific declarations and definitions in `extraAttrDeclaration`
// and `extraAttrDefinition` template arguments.
````
- **EN**: This block establishes the file guard and imports the MLIR/LLVM/TableGen dependencies that the rest of the file builds on.
- **CN**: 该代码块同时建立文件保护宏，并引入后续实现所依赖的 MLIR/LLVM/TableGen 头文件或记录文件。

### Lines 31-32
````tablegen
class NativeAttrTrait<string name,
                      code extraAttrDeclaration = [{}],
````
- **EN**: This TableGen block defines `NativeAttrTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `NativeAttrTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 33-34
````tablegen
                      code extraAttrDefinition = [{}]>
    : NativeTrait<name, "Attribute", extraAttrDeclaration, extraAttrDefinition>;
````
- **EN**: This section focuses on code extraattrdefinition = [{}]>, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“code extraAttrDefinition = [{}]>”这一主题，把相关声明与辅助接口组织在一起。

### Lines 36-37
````tablegen
class ParamNativeAttrTrait<string prop, string params>
    : ParamNativeTrait<prop, params, "Attribute">;
````
- **EN**: This TableGen block defines `ParamNativeAttrTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ParamNativeAttrTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 38-38
````tablegen
class GenInternalAttrTrait<string prop> : GenInternalTrait<prop, "Attribute">;
````
- **EN**: This TableGen block defines `GenInternalAttrTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `GenInternalAttrTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 39-48
````tablegen
class PredAttrTrait<string descr, Pred pred> : PredTrait<descr, pred>;

//===----------------------------------------------------------------------===//
// TypeTrait definitions
//===----------------------------------------------------------------------===//

// These classes are used to define type specific traits.

// Specify type specific declarations and definitions in `extraTypeDeclaration`
// and `extraTypeDefinition` template arguments.
````
- **EN**: This TableGen block defines `PredAttrTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `PredAttrTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 49-50
````tablegen
class NativeTypeTrait<string name,
                      code extraTypeDeclaration = [{}],
````
- **EN**: This TableGen block defines `NativeTypeTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `NativeTypeTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 51-52
````tablegen
                      code extraTypeDefinition = [{}]>
    : NativeTrait<name, "Type", extraTypeDeclaration, extraTypeDefinition>;
````
- **EN**: This section focuses on code extratypedefinition = [{}]>, grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“code extraTypeDefinition = [{}]>”这一主题，把相关声明与辅助接口组织在一起。

### Lines 54-55
````tablegen
class ParamNativeTypeTrait<string prop, string params>
    : ParamNativeTrait<prop, params, "Type">;
````
- **EN**: This TableGen block defines `ParamNativeTypeTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ParamNativeTypeTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 56-56
````tablegen
class GenInternalTypeTrait<string prop> : GenInternalTrait<prop, "Type">;
````
- **EN**: This TableGen block defines `GenInternalTypeTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `GenInternalTypeTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 57-59
````tablegen
class PredTypeTrait<string descr, Pred pred> : PredTrait<descr, pred>;

// Trait required to be added to any type which is mutable.
````
- **EN**: This TableGen block defines `PredTypeTrait` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `PredTypeTrait` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 60-78
````tablegen
def MutableType : NativeTypeTrait<"IsMutable">;

//===----------------------------------------------------------------------===//
// Builders
//===----------------------------------------------------------------------===//

// Class for defining a custom getter.
//
// TableGen generates several generic getter methods for each attribute and type
// by default, corresponding to the specified dag parameters. If the default
// generated ones cannot cover some use case, custom getters can be defined
// using instances of this class.
//
// The signature of the `get` is always either:
//
// ```c++
// static <ClassName> get(MLIRContext *context, <other-parameters>...) {
//   <body>...
// }
````
- **EN**: This TableGen block defines `MutableType` as a `def` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `MutableType` 定义为 `def` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 79-119
````tablegen
// ```
//
// or:
//
// ```c++
// static <ClassName> get(MLIRContext *context, <parameters>...);
// ```
//
// To define a custom getter, the parameter list and body should be passed
// in as separate template arguments to this class. The parameter list is a
// TableGen DAG with `ins` operation with named arguments, which has either:
//   - string initializers ("Type":$name) to represent a typed parameter, or
//   - CArg-typed initializers (CArg<"Type", "default">:$name) to represent a
//     typed parameter that may have a default value.
// The type string is used verbatim to produce code and, therefore, must be a
// valid C++ type. It is used inside the C++ namespace of the parent Type's
// dialect; explicit namespace qualification like `::mlir` may be necessary if
// Types are not placed inside the `mlir` namespace. The default value string is
// used verbatim to produce code and must be a valid C++ initializer the given
// type. For example, the following signature specification
//
// ```
// AttrOrTypeBuilder<(ins "int":$integerArg, CArg<"float", "3.0f">:$floatArg)>
// ```
//
// has an integer parameter and a float parameter with a default value.
//
// If an empty string is passed in for `body`, then *only* the builder
// declaration will be generated; this provides a way to define complicated
// builders entirely in C++. If a `body` string is provided, the `Base::get`
// method should be invoked using `$_get`, e.g.:
//
// ```
// AttrOrTypeBuilder<(ins "int":$integerArg, CArg<"float", "3.0f">:$floatArg), [{
//   return $_get($_ctxt, integerArg, floatArg);
// }]>
// ```
//
// This is necessary because the `body` is also used to generate `getChecked`
// methods, which have a different underlying `Base::get*` call.
//
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 120-132
````tablegen
class AttrOrTypeBuilder<dag parameters, code bodyCode = "",
                        string returnTypeStr = ""> {
  dag dagParams = parameters;
  code body = bodyCode;

  // Change the return type of the builder. By default, it is the type of the
  // attribute or type.
  string returnType = returnTypeStr;

  // The context parameter can be inferred from one of the other parameters and
  // is not implicitly added to the parameter list.
  bit hasInferredContextParam = 0;
}
````
- **EN**: This TableGen block defines `AttrOrTypeBuilder` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `AttrOrTypeBuilder` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 133-134
````tablegen
class AttrBuilder<dag parameters, code bodyCode = "", string returnType = "">
  : AttrOrTypeBuilder<parameters, bodyCode, returnType>;
````
- **EN**: This TableGen block defines `AttrBuilder` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `AttrBuilder` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 135-140
````tablegen
class TypeBuilder<dag parameters, code bodyCode = "", string returnType = "">
  : AttrOrTypeBuilder<parameters, bodyCode, returnType>;

// A class of AttrOrTypeBuilder that is able to infer the MLIRContext parameter
// from one of the other builder parameters. Instances of this builder do not
// have `MLIRContext *` implicitly added to the parameter list.
````
- **EN**: This TableGen block defines `TypeBuilder` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `TypeBuilder` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 141-145
````tablegen
class AttrOrTypeBuilderWithInferredContext<dag parameters, code bodyCode = "",
                                           string returnType = "">
  : TypeBuilder<parameters, bodyCode, returnType> {
  let hasInferredContextParam = 1;
}
````
- **EN**: This TableGen block defines `AttrOrTypeBuilderWithInferredContext` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `AttrOrTypeBuilderWithInferredContext` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 146-148
````tablegen
class AttrBuilderWithInferredContext<dag parameters, code bodyCode = "",
                                     string returnType = "">
  : AttrOrTypeBuilderWithInferredContext<parameters, bodyCode, returnType>;
````
- **EN**: This TableGen block defines `AttrBuilderWithInferredContext` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `AttrBuilderWithInferredContext` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 149-158
````tablegen
class TypeBuilderWithInferredContext<dag parameters, code bodyCode = "",
                                     string returnType = "">
  : AttrOrTypeBuilderWithInferredContext<parameters, bodyCode, returnType>;

//===----------------------------------------------------------------------===//
// Definitions
//===----------------------------------------------------------------------===//

// Define a new attribute or type, named `name`, that inherits from the given
// C++ base class.
````
- **EN**: This TableGen block defines `TypeBuilderWithInferredContext` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `TypeBuilderWithInferredContext` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 159-233
````tablegen
class AttrOrTypeDef<string valueType, string name, list<Trait> defTraits,
                    string baseCppClass> {
  // The name of the C++ base class to use for this def.
  string cppBaseClassName = baseCppClass;

  // Additional, longer human-readable description of what the def does.
  string description = "";

  // Name of storage class to generate or use.
  string storageClass = name # valueType # "Storage";

  // Namespace (withing dialect c++ namespace) in which the storage class
  // resides.
  string storageNamespace = "detail";

  // Specify if the storage class is to be generated.
  bit genStorageClass = 1;

  // Specify that the generated storage class has a constructor which is written
  // in C++.
  bit hasStorageCustomConstructor = 0;

  // The list of parameters for this type. Parameters will become both
  // parameters to the get() method and storage class member variables.
  //
  // The format of this dag is:
  //    (ins
  //        "<c++ type>":$param1Name,
  //        "<c++ type>":$param2Name,
  //        AttrOrTypeParameter<"c++ type", "param description">:$param3Name)
  // AttrOrTypeParameters (or more likely one of their subclasses) are required
  // to add more information about the parameter, specifically:
  //  - Documentation
  //  - Code to allocate the parameter (if allocation is needed in the storage
  //    class constructor)
  //
  // For example:
  //    (ins "int":$width,
  //         ArrayRefParameter<"bool", "list of bools">:$yesNoArray)
  //
  // (ArrayRefParameter is a subclass of AttrOrTypeParameter which has
  // allocation code for re-allocating ArrayRefs. It is defined below.)
  dag parameters = (ins);

  // Custom builder methods.
  // In addition to the custom builders provided here, and unless
  // skipDefaultBuilders is set, a default builder is generated with the
  // following signature:
  //
  // ```c++
  // static <ClassName> get(MLIRContext *, <parameters>);
  // ```
  //
  // Note that builders should only be provided when a def has parameters.
  list<AttrOrTypeBuilder> builders = ?;

  // The list of traits attached to this def.
  list<Trait> traits = defTraits;

  // Use the lowercased name as the keyword for parsing/printing. Specify only
  // if you want tblgen to generate declarations and/or definitions of
  // the printer/parser. If specified and the Attribute or Type contains
  // parameters, `assemblyFormat` or `hasCustomAssemblyFormat` must also be
  // specified.
  string mnemonic = ?;

  // Custom assembly format. Requires 'mnemonic' to be specified. Cannot be
  // specified at the same time as 'hasCustomAssemblyFormat'. The generated
  // printer requires 'genAccessors' to be true.
  string assemblyFormat = ?;
  /// This field indicates that the attribute or type has a custom assembly format
  /// implemented in C++. When set to `1` a `parse` and `print` method are generated
  /// on the generated class. The attribute or type should implement these methods to
  /// support the custom format.
  bit hasCustomAssemblyFormat = 0;
````
- **EN**: This TableGen block defines `AttrOrTypeDef` as a `class` record for `AttrTypeBase`. It covers assembly syntax, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `AttrOrTypeDef` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 汇编语法, trait/接口组合。

### Lines 234-254
````tablegen
  // If set, generate accessors for each parameter.
  bit genAccessors = 1;

  // Avoid generating default get/getChecked functions. Custom get methods must
  // be provided.
  bit skipDefaultBuilders = 0;

  // Generate the verify and getChecked methods.
  bit genVerifyDecl = 0;

  // Extra code to include in the class declaration.
  code extraClassDeclaration = [{}];

  // Additional code that will be added to the generated source file. The
  // generated code is placed inside the class's C++ namespace. `$cppClass` is
  // replaced by the class name.
  code extraClassDefinition = [{}];

  // Generate a default 'getAlias' method for OpAsm{Type,Attr}Interface.
  bit genMnemonicAlias = 0;
}
````
- **EN**: This section focuses on if set, generate accessors for each parameter., grouping the declarations and helpers needed for that concern.
- **CN**: 本节聚焦于“If set, generate accessors for each parameter.”这一主题，把相关声明与辅助接口组织在一起。

### Lines 256-257
````tablegen
// Define a new attribute, named `name`, belonging to `dialect` that inherits
// from the given C++ base class.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 259-288
````tablegen
class AttrDef<Dialect dialect, string name, list<Trait> traits = [],
              string baseCppClass = "::mlir::Attribute">
    : DialectAttr<dialect, CPred<"">, /*descr*/"">,
      AttrOrTypeDef<"Attr", name, traits, baseCppClass> {
  // The name of the C++ Attribute class.
  string cppClassName = name # "Attr";
  let storageType = dialect.cppNamespace # "::" # cppClassName;

  // The underlying C++ value type
  let returnType = dialect.cppNamespace # "::" # cppClassName;

  // Make it possible to use such attributes as parameters for other attributes.
  string cppType = dialect.cppNamespace # "::" # cppClassName;

  // The unique attribute name.
  string attrName = dialect.name # "." # mnemonic;

  // The call expression to convert from the storage type to the return
  // type. For example, an enum can be stored as an int but returned as an
  // enum class.
  //
  // Format: $_self will be expanded to the attribute.
  //
  // For example, `$_self.getValue().getSExtValue()` for `IntegerAttr val` will
  // expand to `getAttrOfType<IntegerAttr>("val").getValue().getSExtValue()`.
  let convertFromStorage = "::llvm::cast<" # cppType # ">($_self)";

  // The predicate for when this def is used as a constraint.
  let predicate = CPred<"::llvm::isa<" # cppType # ">($_self)">;
}
````
- **EN**: This TableGen block defines `AttrDef` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `AttrDef` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 289-290
````tablegen
// Provide a LocationAttrDef for dialects to provide their own locations
// that subclass LocationAttr.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 292-297
````tablegen
class LocationAttrDef<Dialect dialect, string name, list<Trait> traits = []>
    : AttrDef<dialect, name, traits # [NativeAttrTrait<"IsLocation">],
              "::mlir::LocationAttr">;

// Define a new type, named `name`, belonging to `dialect` that inherits from
// the given C++ base class.
````
- **EN**: This TableGen block defines `LocationAttrDef` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `LocationAttrDef` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 298-318
````tablegen
class TypeDef<Dialect dialect, string name, list<Trait> traits = [],
              string baseCppClass = "::mlir::Type">
    : DialectType<dialect, CPred<"">, /*descr*/"", name # "Type">,
      AttrOrTypeDef<"Type", name, traits, baseCppClass> {
  // The name of the C++ Type class.
  string cppClassName = name # "Type";

  // Make it possible to use such type as parameters for other types.
  string cppType = dialect.cppNamespace # "::" # cppClassName;

  // The unique type name.
  string typeName = dialect.name # "." # mnemonic;

  // A constant builder provided when the type has no parameters.
  let builderCall = !if(!empty(parameters),
                           "$_builder.getType<" # cppType # ">()",
                           "");

  // The predicate for when this def is used as a constraint.
  let predicate = CPred<"::llvm::isa<" # cppType # ">($_self)">;
}
````
- **EN**: This TableGen block defines `TypeDef` as a `class` record for `AttrTypeBase`. It covers trait/interface composition.
- **CN**: 该 TableGen 代码块将 `TypeDef` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 trait/接口组合。

### Lines 319-324
````tablegen
//===----------------------------------------------------------------------===//
// Parameters
//===----------------------------------------------------------------------===//

// 'Parameters' should be subclasses of this or simple strings (which is a
// shorthand for AttrOrTypeParameter<"C++Type">).
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 326-362
````tablegen
class AttrOrTypeParameter<string type, string desc, string accessorType = ""> {
  // Custom memory allocation code for storage constructor.
  code allocator = ?;
  // Comparator used to compare two instances for equality. By default, it uses
  // the C++ equality operator.
  code comparator = ?;
  // The C++ type of this parameter.
  string cppType = type;
  // The C++ type of the accessor for this parameter.
  string cppAccessorType = !if(!empty(accessorType), type, accessorType);
  // The C++ storage type of this parameter if it is a reference, e.g.
  // `std::string` for `StringRef` or `SmallVector` for `ArrayRef`.
  string cppStorageType = cppType;
  // The C++ code to convert from the storage type to the parameter type.
  string convertFromStorage = "$_self";
  // One-line human-readable description of the argument.
  string summary = desc;
  // The format string for the asm syntax (documentation only).
  string syntax = ?;
  // The default parameter parser is `::mlir::FieldParser<T>::parse($_parser)`,
  // which returns `FailureOr<T>`. Specialize `FieldParser` to support parsing
  // for your type. Or you can provide a customer printer. For attributes,
  // "$_type" will be replaced with the required attribute type.
  string parser = ?;
  // The default parameter printer is `$_printer << $_self`. Overload the stream
  // operator of `AsmPrinter` as necessary to print your type. Or you can
  // provide a custom printer.
  string printer = ?;
  // Provide a default value for the parameter. Parameters with default values
  // are considered optional. If a value was not parsed for the parameter, it
  // will be set to the default value. Parameters equal to their default values
  // are elided when printing. Equality is checked using the `comparator` field,
  // which by default is the C++ equality operator. The current MLIR context is
  // made available through `$_ctxt`, e.g., for constructing default values for
  // attributes and types.
  string defaultValue = "";
}
````
- **EN**: This TableGen block defines `AttrOrTypeParameter` as a `class` record for `AttrTypeBase`. It covers semantic documentation.
- **CN**: 该 TableGen 代码块将 `AttrOrTypeParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 语义文档。

### Lines 363-364
````tablegen
class AttrParameter<string type, string desc, string accessorType = "">
 : AttrOrTypeParameter<type, desc, accessorType>;
````
- **EN**: This TableGen block defines `AttrParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `AttrParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 365-368
````tablegen
class TypeParameter<string type, string desc, string accessorType = "">
 : AttrOrTypeParameter<type, desc, accessorType>;

// An optional parameter.
````
- **EN**: This TableGen block defines `TypeParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `TypeParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 369-372
````tablegen
class OptionalParameter<string type, string desc = ""> :
    AttrOrTypeParameter<type, desc> {
  let defaultValue = cppStorageType # "()";
}
````
- **EN**: This TableGen block defines `OptionalParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `OptionalParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 373-373
````tablegen
// A parameter with a default value.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 375-378
````tablegen
class DefaultValuedParameter<string type, string value, string desc = ""> :
    AttrOrTypeParameter<type, desc> {
  let defaultValue = value;
}
````
- **EN**: This TableGen block defines `DefaultValuedParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `DefaultValuedParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 379-379
````tablegen
// For StringRefs, which require allocation.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 381-387
````tablegen
class StringRefParameter<string desc = "", string value = ""> :
    AttrOrTypeParameter<"::llvm::StringRef", desc> {
  let allocator = [{$_dst = $_allocator.copyInto($_self);}];
  let printer = [{$_printer.printString($_self);}];
  let cppStorageType = "std::string";
  let defaultValue = value;
}
````
- **EN**: This TableGen block defines `StringRefParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `StringRefParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 388-390
````tablegen
// For APInts, which require comparison supporting different bitwidths. The
// default APInt comparison operator asserts when the bitwidths differ, so
// a custom implementation is necessary.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 392-395
````tablegen
class APIntParameter<string desc> :
    AttrOrTypeParameter<"::llvm::APInt", desc, "const ::llvm::APInt &"> {
  let comparator = "$_lhs.getBitWidth() == $_rhs.getBitWidth() && $_lhs == $_rhs";
}
````
- **EN**: This TableGen block defines `APIntParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `APIntParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 396-396
````tablegen
// For APFloats, which require comparison.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 398-401
````tablegen
class APFloatParameter<string desc> :
    AttrOrTypeParameter<"::llvm::APFloat", desc> {
  let comparator = "$_lhs.bitwiseIsEqual($_rhs)";
}
````
- **EN**: This TableGen block defines `APFloatParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `APFloatParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 402-402
````tablegen
// For standard ArrayRefs, which require allocation.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 404-408
````tablegen
class ArrayRefParameter<string arrayOf, string desc = ""> :
    AttrOrTypeParameter<"::llvm::ArrayRef<" # arrayOf # ">", desc> {
  let allocator = [{$_dst = $_allocator.copyInto($_self);}];
  let cppStorageType = "::llvm::SmallVector<" # arrayOf # ">";
}
````
- **EN**: This TableGen block defines `ArrayRefParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `ArrayRefParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 409-410
````tablegen
// Regular array parameters cannot be parsed when empty. This optional array
// parameter can be used with optional groups to be parsed when empty.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 412-417
````tablegen
class OptionalArrayRefParameter<string arrayOf, string desc = ""> :
    OptionalParameter<"::llvm::ArrayRef<" # arrayOf # ">", desc> {
  let allocator = [{$_dst = $_allocator.copyInto($_self);}];
  let cppStorageType = "::llvm::SmallVector<" # arrayOf # ">";
  let comparator = cppType # "($_lhs) == " # cppType # "($_rhs)";
}
````
- **EN**: This TableGen block defines `OptionalArrayRefParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `OptionalArrayRefParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 418-418
````tablegen
// For classes which require allocation and have their own allocateInto method.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 420-423
````tablegen
class SelfAllocationParameter<string type, string desc> :
    AttrOrTypeParameter<type, desc> {
  let allocator = [{$_dst = $_self.allocateInto($_allocator);}];
}
````
- **EN**: This TableGen block defines `SelfAllocationParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `SelfAllocationParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 424-424
````tablegen
// For ArrayRefs which contain things which allocate themselves.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 426-434
````tablegen
class ArrayRefOfSelfAllocationParameter<string arrayOf, string desc> :
    AttrOrTypeParameter<"::llvm::ArrayRef<" # arrayOf # ">", desc> {
  let allocator = [{
    llvm::SmallVector<}] # arrayOf # [{, 4> tmpFields;
    for (size_t i = 0, e = $_self.size(); i < e; ++i)
      tmpFields.push_back($_self[i].allocateInto($_allocator));
    $_dst = $_allocator.copyInto(ArrayRef<}] # arrayOf # [{>(tmpFields));
  }];
}
````
- **EN**: This TableGen block defines `ArrayRefOfSelfAllocationParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `ArrayRefOfSelfAllocationParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 435-442
````tablegen
// This is a special attribute parameter that represents the "self" type of the
// attribute. It is specially handled by the assembly format generator to derive
// its value from the optional trailing type after each attribute.
//
// By default, the self type parameter is optional and has a default value of
// `none`. If a derived type other than `::mlir::Type` is specified, the
// parameter loses its default value unless another one is specified by
// `typeBuilder`.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 444-451
````tablegen
class AttributeSelfTypeParameter<string desc,
                                 string derivedType = "::mlir::Type",
                                 string typeBuilder = ""> :
    AttrOrTypeParameter<derivedType, desc> {
  let defaultValue = !if(!and(!empty(typeBuilder),
                              !eq(derivedType, "::mlir::Type")),
                         "::mlir::NoneType::get($_ctxt)", typeBuilder);
}
````
- **EN**: This TableGen block defines `AttributeSelfTypeParameter` as a `class` record for `AttrTypeBase`.
- **CN**: 该 TableGen 代码块将 `AttributeSelfTypeParameter` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。

### Lines 452-458
````tablegen
//===----------------------------------------------------------------------===//
// ArrayOfAttr
//===----------------------------------------------------------------------===//

/// This class defines an attribute that contains an array of elements. The
/// elements can be any type, but if they are attributes, the nested elements
/// are parsed and printed using the custom attribute syntax.
````
- **EN**: This comment block frames the surrounding section, capturing intent, specification notes, or usage guidance before the real declarations begin.
- **CN**: 该注释块用于为后续代码提供上下文，说明设计意图、规范约束或使用方式。

### Lines 460-481
````tablegen
class ArrayOfAttr<Dialect dialect, string name, string attrMnemonic,
                  string eltName, list<Trait> traits = []>
    : AttrDef<dialect, name, traits> {
  let parameters = (ins OptionalArrayRefParameter<eltName>:$value);
  let mnemonic = attrMnemonic;
  let assemblyFormat = "`[` (`]`) : ($value^ `]`)?";

  let returnType = "::llvm::ArrayRef<" # eltName # ">";
  let constBuilderCall = "$_builder.getAttr<" # name # "Attr>($0)";
  let convertFromStorage = "$_self.getValue()";

  let extraClassDeclaration = [{
    auto begin() const { return getValue().begin(); }
    auto end() const { return getValue().end(); }
    bool empty() const { return getValue().empty(); }
    size_t size() const { return getValue().size(); }
    auto &front() const { return getValue().front(); }
    auto &back() const { return getValue().back(); }
    auto &operator[](size_t index) { return getValue()[index]; }
    operator }] # returnType # [{() const { return getValue(); }
  }];
}
````
- **EN**: This TableGen block defines `ArrayOfAttr` as a `class` record for `AttrTypeBase`. It covers assembly syntax, trait/interface composition.
- **CN**: 该 TableGen 代码块将 `ArrayOfAttr` 定义为 `class` 记录，用于描述 `AttrTypeBase` 相关的声明式信息。 其中涉及 汇编语法, trait/接口组合。

### Lines 482-482
````tablegen
#endif // ATTRTYPEBASE_TD
````
- **EN**: This block manages the file guard so the header or TableGen fragment is only processed once per translation or inclusion path.
- **CN**: 该代码块管理文件保护宏，确保头文件或 TableGen 片段在一次编译/包含路径中只被处理一次。

## Key Concepts / 关键概念

- **EN**: Declarative TableGen modeling of MLIR entities
  **CN**: 以声明式 TableGen 方式建模 MLIR 实体
- **EN**: Custom assembly syntax specification
  **CN**: 自定义汇编语法规格
- **EN**: Trait and interface composition
  **CN**: trait 与接口组合
- **EN**: Embedded semantic documentation
  **CN**: 内嵌语义文档

## Dependencies / 依赖关系

- mlir/IR/CommonAttrConstraints.td
- mlir/IR/CommonTypeConstraints.td
- mlir/IR/Constraints.td
- mlir/IR/DialectBase.td
- mlir/IR/Traits.td
- MutableType builds on NativeTypeTrait<"IsMutable">;
