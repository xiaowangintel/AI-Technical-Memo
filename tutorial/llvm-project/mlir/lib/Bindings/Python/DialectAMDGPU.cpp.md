# DialectAMDGPU.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `mlir/lib/Bindings/Python/DialectAMDGPU.cpp`
- **Repository / 仓库**: `llvm-project` (`/root/xw/llvm-project`)
- **Purpose / 目的**:
  - **EN**: Implements the MLIR library support logic associated with `DialectAMDGPU`.
  - **CN**: 实现与 `DialectAMDGPU` 相关的 MLIR 库支持逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7 / 第 1-7 行

```cpp
1 | //===--- DialectAMDGPU.cpp - Pybind module for AMDGPU dialect API support -===//
2 | //
3 | // Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
4 | // See https://llvm.org/LICENSE.txt for license information.
5 | // SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
6 | //
7 | //===----------------------------------------------------------------------===//
```

- **L1**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。
- **L2**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L3**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`. / 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used for visual grouping. / 用于视觉分组的分隔注释。
- **L7**: Banner comment marking a file or section boundary. / 横幅注释，用于标记文件或章节边界。

### Lines 8-15 / 第 8-15 行

```cpp
 8 | 
 9 | #include "mlir-c/Dialect/AMDGPU.h"
10 | #include "mlir-c/IR.h"
11 | #include "mlir/Bindings/Python/IRCore.h"
12 | #include "mlir/Bindings/Python/Nanobind.h"
13 | #include "mlir/Bindings/Python/NanobindAdaptors.h"
14 | #include "nanobind/nanobind.h"
15 | 
```

- **L8**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Includes "mlir-c/Dialect/AMDGPU.h" to access local declarations used by this file. / 引入 "mlir-c/Dialect/AMDGPU.h" 以使用本文件使用的本地声明。
- **L10**: Includes "mlir-c/IR.h" to access local declarations used by this file. / 引入 "mlir-c/IR.h" 以使用本文件使用的本地声明。
- **L11**: Includes "mlir/Bindings/Python/IRCore.h" to access local declarations used by this file. / 引入 "mlir/Bindings/Python/IRCore.h" 以使用本文件使用的本地声明。
- **L12**: Includes "mlir/Bindings/Python/Nanobind.h" to access local declarations used by this file. / 引入 "mlir/Bindings/Python/Nanobind.h" 以使用本文件使用的本地声明。
- **L13**: Includes "mlir/Bindings/Python/NanobindAdaptors.h" to access local declarations used by this file. / 引入 "mlir/Bindings/Python/NanobindAdaptors.h" 以使用本文件使用的本地声明。
- **L14**: Includes "nanobind/nanobind.h" to access local declarations used by this file. / 引入 "nanobind/nanobind.h" 以使用本文件使用的本地声明。
- **L15**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 16-20 / 第 16-20 行

```cpp
16 | namespace nb = nanobind;
17 | using namespace mlir::python::nanobind_adaptors;
18 | 
19 | namespace mlir {
20 | namespace python {
```

- **L16**: Initializes variable `nb` from the right-hand expression. / 使用右侧表达式初始化变量 `nb`。
- **L17**: Brings namespace `mlir::python::nanobind_adaptors` into the local scope. / 将命名空间 `mlir::python::nanobind_adaptors` 引入当前作用域。
- **L18**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L19**: Opens namespace scope `mlir`. / 打开命名空间作用域 `mlir`。
- **L20**: Opens namespace scope `python`. / 打开命名空间作用域 `python`。

### Lines 21-30 / 第 21-30 行

```cpp
21 | namespace MLIR_BINDINGS_PYTHON_DOMAIN {
22 | namespace amdgpu {
23 | struct TDMBaseType : PyConcreteType<TDMBaseType> {
24 |   static constexpr IsAFunctionTy isaFunction = mlirTypeIsAAMDGPUTDMBaseType;
25 |   static constexpr GetTypeIDFunctionTy getTypeIdFunction =
26 |       mlirAMDGPUTDMBaseTypeGetTypeID;
27 |   static constexpr const char *pyClassName = "TDMBaseType";
28 |   static inline const MlirStringRef name = mlirAMDGPUTDMBaseTypeGetName();
29 |   using Base::Base;
30 | 
```

- **L21**: Opens namespace scope `MLIR_BINDINGS_PYTHON_DOMAIN`. / 打开命名空间作用域 `MLIR_BINDINGS_PYTHON_DOMAIN`。
- **L22**: Opens namespace scope `amdgpu`. / 打开命名空间作用域 `amdgpu`。
- **L23**: Declares struct `TDMBaseType`. / 声明 struct `TDMBaseType`。
- **L24**: Initializes variable `isaFunction` from the right-hand expression. / 使用右侧表达式初始化变量 `isaFunction`。
- **L25**: Continues the surrounding expression or declaration: `static constexpr GetTypeIDFunctionTy getTypeIdFunction =`. / 继续构造周围的表达式或声明：`static constexpr GetTypeIDFunctionTy getTypeIdFunction =`。
- **L26**: Executes a standalone statement or declaration: `mlirAMDGPUTDMBaseTypeGetTypeID;`. / 执行一条独立语句或声明：`mlirAMDGPUTDMBaseTypeGetTypeID;`。
- **L27**: Executes a standalone statement or declaration: `static constexpr const char *pyClassName = "TDMBaseType";`. / 执行一条独立语句或声明：`static constexpr const char *pyClassName = "TDMBaseType";`。
- **L28**: Initializes variable `name` from the right-hand expression. / 使用右侧表达式初始化变量 `name`。
- **L29**: Executes a standalone statement or declaration: `using Base::Base;`. / 执行一条独立语句或声明：`using Base::Base;`。
- **L30**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 31-40 / 第 31-40 行

```cpp
31 |   static void bindDerived(ClassTy &c) {
32 |     c.def_static(
33 |         "get",
34 |         [](const PyType &elementType, DefaultingPyMlirContext context) {
35 |           return TDMBaseType(
36 |               context->getRef(),
37 |               mlirAMDGPUTDMBaseTypeGet(context.get()->get(), elementType));
38 |         },
39 |         "Gets an instance of TDMBaseType in the same context",
40 |         nb::arg("element_type"), nb::arg("context").none() = nb::none());
```

- **L31**: Starts a function, method, lambda, or structured scope: `static void bindDerived(ClassTy &c) {`. / 开始一个函数、方法、lambda 或结构化作用域：`static void bindDerived(ClassTy &c) {`。
- **L32**: Continues logic associated with callable symbol `def_static`. / 继续与可调用符号 `def_static` 相关的逻辑。
- **L33**: Continues a multi-line argument list, initializer, or aggregate entry: `"get",`. / 继续一个多行参数列表、初始化器或聚合项：`"get",`。
- **L34**: Starts a function, method, lambda, or structured scope: `[](const PyType &elementType, DefaultingPyMlirContext context) {`. / 开始一个函数、方法、lambda 或结构化作用域：`[](const PyType &elementType, DefaultingPyMlirContext context) {`。
- **L35**: Returns from the current function with `TDMBaseType(`. / 以 `TDMBaseType(` 从当前函数返回。
- **L36**: Continues a multi-line argument list, initializer, or aggregate entry: `context->getRef(),`. / 继续一个多行参数列表、初始化器或聚合项：`context->getRef(),`。
- **L37**: Executes a call or declaration centered on `mlirAMDGPUTDMBaseTypeGet`. / 执行以 `mlirAMDGPUTDMBaseTypeGet` 为核心的调用或声明。
- **L38**: Continues a multi-line argument list, initializer, or aggregate entry: `},`. / 继续一个多行参数列表、初始化器或聚合项：`},`。
- **L39**: Continues a multi-line argument list, initializer, or aggregate entry: `"Gets an instance of TDMBaseType in the same context",`. / 继续一个多行参数列表、初始化器或聚合项：`"Gets an instance of TDMBaseType in the same context",`。
- **L40**: Executes a call or declaration centered on `nb::arg`. / 执行以 `nb::arg` 为核心的调用或声明。

### Lines 41-50 / 第 41-50 行

```cpp
41 |   }
42 | };
43 | 
44 | struct TDMDescriptorType : PyConcreteType<TDMDescriptorType> {
45 |   static constexpr IsAFunctionTy isaFunction =
46 |       mlirTypeIsAAMDGPUTDMDescriptorType;
47 |   static constexpr GetTypeIDFunctionTy getTypeIdFunction =
48 |       mlirAMDGPUTDMDescriptorTypeGetTypeID;
49 |   static constexpr const char *pyClassName = "TDMDescriptorType";
50 |   static inline const MlirStringRef name = mlirAMDGPUTDMDescriptorTypeGetName();
```

- **L41**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L42**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L43**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L44**: Declares struct `TDMDescriptorType`. / 声明 struct `TDMDescriptorType`。
- **L45**: Continues the surrounding expression or declaration: `static constexpr IsAFunctionTy isaFunction =`. / 继续构造周围的表达式或声明：`static constexpr IsAFunctionTy isaFunction =`。
- **L46**: Executes a standalone statement or declaration: `mlirTypeIsAAMDGPUTDMDescriptorType;`. / 执行一条独立语句或声明：`mlirTypeIsAAMDGPUTDMDescriptorType;`。
- **L47**: Continues the surrounding expression or declaration: `static constexpr GetTypeIDFunctionTy getTypeIdFunction =`. / 继续构造周围的表达式或声明：`static constexpr GetTypeIDFunctionTy getTypeIdFunction =`。
- **L48**: Executes a standalone statement or declaration: `mlirAMDGPUTDMDescriptorTypeGetTypeID;`. / 执行一条独立语句或声明：`mlirAMDGPUTDMDescriptorTypeGetTypeID;`。
- **L49**: Executes a standalone statement or declaration: `static constexpr const char *pyClassName = "TDMDescriptorType";`. / 执行一条独立语句或声明：`static constexpr const char *pyClassName = "TDMDescriptorType";`。
- **L50**: Initializes variable `name` from the right-hand expression. / 使用右侧表达式初始化变量 `name`。

### Lines 51-60 / 第 51-60 行

```cpp
51 |   using Base::Base;
52 | 
53 |   static void bindDerived(ClassTy &c) {
54 |     c.def_static(
55 |         "get",
56 |         [](DefaultingPyMlirContext context) {
57 |           return TDMDescriptorType(
58 |               context->getRef(),
59 |               mlirAMDGPUTDMDescriptorTypeGet(context.get()->get()));
60 |         },
```

- **L51**: Executes a standalone statement or declaration: `using Base::Base;`. / 执行一条独立语句或声明：`using Base::Base;`。
- **L52**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L53**: Starts a function, method, lambda, or structured scope: `static void bindDerived(ClassTy &c) {`. / 开始一个函数、方法、lambda 或结构化作用域：`static void bindDerived(ClassTy &c) {`。
- **L54**: Continues logic associated with callable symbol `def_static`. / 继续与可调用符号 `def_static` 相关的逻辑。
- **L55**: Continues a multi-line argument list, initializer, or aggregate entry: `"get",`. / 继续一个多行参数列表、初始化器或聚合项：`"get",`。
- **L56**: Starts a function, method, lambda, or structured scope: `[](DefaultingPyMlirContext context) {`. / 开始一个函数、方法、lambda 或结构化作用域：`[](DefaultingPyMlirContext context) {`。
- **L57**: Returns from the current function with `TDMDescriptorType(`. / 以 `TDMDescriptorType(` 从当前函数返回。
- **L58**: Continues a multi-line argument list, initializer, or aggregate entry: `context->getRef(),`. / 继续一个多行参数列表、初始化器或聚合项：`context->getRef(),`。
- **L59**: Executes a call or declaration centered on `mlirAMDGPUTDMDescriptorTypeGet`. / 执行以 `mlirAMDGPUTDMDescriptorTypeGet` 为核心的调用或声明。
- **L60**: Continues a multi-line argument list, initializer, or aggregate entry: `},`. / 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 61-65 / 第 61-65 行

```cpp
61 |         "Gets an instance of TDMDescriptorType in the same context",
62 |         nb::arg("context").none() = nb::none());
63 |   }
64 | };
65 | 
```

- **L61**: Continues a multi-line argument list, initializer, or aggregate entry: `"Gets an instance of TDMDescriptorType in the same context",`. / 继续一个多行参数列表、初始化器或聚合项：`"Gets an instance of TDMDescriptorType in the same context",`。
- **L62**: Executes a call or declaration centered on `nb::arg`. / 执行以 `nb::arg` 为核心的调用或声明。
- **L63**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L64**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L65**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 66-74 / 第 66-74 行

```cpp
66 | struct TDMGatherBaseType : PyConcreteType<TDMGatherBaseType> {
67 |   static constexpr IsAFunctionTy isaFunction =
68 |       mlirTypeIsAAMDGPUTDMGatherBaseType;
69 |   static constexpr GetTypeIDFunctionTy getTypeIdFunction =
70 |       mlirAMDGPUTDMGatherBaseTypeGetTypeID;
71 |   static constexpr const char *pyClassName = "TDMGatherBaseType";
72 |   static inline const MlirStringRef name = mlirAMDGPUTDMGatherBaseTypeGetName();
73 |   using Base::Base;
74 | 
```

- **L66**: Declares struct `TDMGatherBaseType`. / 声明 struct `TDMGatherBaseType`。
- **L67**: Continues the surrounding expression or declaration: `static constexpr IsAFunctionTy isaFunction =`. / 继续构造周围的表达式或声明：`static constexpr IsAFunctionTy isaFunction =`。
- **L68**: Executes a standalone statement or declaration: `mlirTypeIsAAMDGPUTDMGatherBaseType;`. / 执行一条独立语句或声明：`mlirTypeIsAAMDGPUTDMGatherBaseType;`。
- **L69**: Continues the surrounding expression or declaration: `static constexpr GetTypeIDFunctionTy getTypeIdFunction =`. / 继续构造周围的表达式或声明：`static constexpr GetTypeIDFunctionTy getTypeIdFunction =`。
- **L70**: Executes a standalone statement or declaration: `mlirAMDGPUTDMGatherBaseTypeGetTypeID;`. / 执行一条独立语句或声明：`mlirAMDGPUTDMGatherBaseTypeGetTypeID;`。
- **L71**: Executes a standalone statement or declaration: `static constexpr const char *pyClassName = "TDMGatherBaseType";`. / 执行一条独立语句或声明：`static constexpr const char *pyClassName = "TDMGatherBaseType";`。
- **L72**: Initializes variable `name` from the right-hand expression. / 使用右侧表达式初始化变量 `name`。
- **L73**: Executes a standalone statement or declaration: `using Base::Base;`. / 执行一条独立语句或声明：`using Base::Base;`。
- **L74**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 75-84 / 第 75-84 行

```cpp
75 |   static void bindDerived(ClassTy &c) {
76 |     c.def_static(
77 |         "get",
78 |         [](const PyType &elementType, const PyType &indexType,
79 |            DefaultingPyMlirContext context) {
80 |           return TDMGatherBaseType(
81 |               context->getRef(),
82 |               mlirAMDGPUTDMGatherBaseTypeGet(context.get()->get(), elementType,
83 |                                              indexType));
84 |         },
```

- **L75**: Starts a function, method, lambda, or structured scope: `static void bindDerived(ClassTy &c) {`. / 开始一个函数、方法、lambda 或结构化作用域：`static void bindDerived(ClassTy &c) {`。
- **L76**: Continues logic associated with callable symbol `def_static`. / 继续与可调用符号 `def_static` 相关的逻辑。
- **L77**: Continues a multi-line argument list, initializer, or aggregate entry: `"get",`. / 继续一个多行参数列表、初始化器或聚合项：`"get",`。
- **L78**: Continues a multi-line argument list, initializer, or aggregate entry: `[](const PyType &elementType, const PyType &indexType,`. / 继续一个多行参数列表、初始化器或聚合项：`[](const PyType &elementType, const PyType &indexType,`。
- **L79**: Continues the surrounding expression or declaration: `DefaultingPyMlirContext context) {`. / 继续构造周围的表达式或声明：`DefaultingPyMlirContext context) {`。
- **L80**: Returns from the current function with `TDMGatherBaseType(`. / 以 `TDMGatherBaseType(` 从当前函数返回。
- **L81**: Continues a multi-line argument list, initializer, or aggregate entry: `context->getRef(),`. / 继续一个多行参数列表、初始化器或聚合项：`context->getRef(),`。
- **L82**: Continues a multi-line argument list, initializer, or aggregate entry: `mlirAMDGPUTDMGatherBaseTypeGet(context.get()->get(), elementType,`. / 继续一个多行参数列表、初始化器或聚合项：`mlirAMDGPUTDMGatherBaseTypeGet(context.get()->get(), elementType,`。
- **L83**: Executes a standalone statement or declaration: `indexType));`. / 执行一条独立语句或声明：`indexType));`。
- **L84**: Continues a multi-line argument list, initializer, or aggregate entry: `},`. / 继续一个多行参数列表、初始化器或聚合项：`},`。

### Lines 85-90 / 第 85-90 行

```cpp
85 |         "Gets an instance of TDMGatherBaseType in the same context",
86 |         nb::arg("element_type"), nb::arg("index_type"),
87 |         nb::arg("context").none() = nb::none());
88 |   }
89 | };
90 | 
```

- **L85**: Continues a multi-line argument list, initializer, or aggregate entry: `"Gets an instance of TDMGatherBaseType in the same context",`. / 继续一个多行参数列表、初始化器或聚合项：`"Gets an instance of TDMGatherBaseType in the same context",`。
- **L86**: Continues a multi-line argument list, initializer, or aggregate entry: `nb::arg("element_type"), nb::arg("index_type"),`. / 继续一个多行参数列表、初始化器或聚合项：`nb::arg("element_type"), nb::arg("index_type"),`。
- **L87**: Executes a call or declaration centered on `nb::arg`. / 执行以 `nb::arg` 为核心的调用或声明。
- **L88**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L89**: Closes the current declaration scope such as a class, struct, or enum. / 结束当前声明作用域，例如类、结构体或枚举。
- **L90**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 91-100 / 第 91-100 行

```cpp
 91 | static void populateDialectAMDGPUSubmodule(nb::module_ &m) {
 92 |   TDMBaseType::bind(m);
 93 |   TDMDescriptorType::bind(m);
 94 |   TDMGatherBaseType::bind(m);
 95 | }
 96 | } // namespace amdgpu
 97 | } // namespace MLIR_BINDINGS_PYTHON_DOMAIN
 98 | } // namespace python
 99 | } // namespace mlir
100 | 
```

- **L91**: Starts a function, method, lambda, or structured scope: `static void populateDialectAMDGPUSubmodule(nb::module_ &m) {`. / 开始一个函数、方法、lambda 或结构化作用域：`static void populateDialectAMDGPUSubmodule(nb::module_ &m) {`。
- **L92**: Executes a call or declaration centered on `TDMBaseType::bind`. / 执行以 `TDMBaseType::bind` 为核心的调用或声明。
- **L93**: Executes a call or declaration centered on `TDMDescriptorType::bind`. / 执行以 `TDMDescriptorType::bind` 为核心的调用或声明。
- **L94**: Executes a call or declaration centered on `TDMGatherBaseType::bind`. / 执行以 `TDMGatherBaseType::bind` 为核心的调用或声明。
- **L95**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。
- **L96**: Closes a namespace scope while preserving the trailing comment: `} // namespace amdgpu`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace amdgpu`。
- **L97**: Closes a namespace scope while preserving the trailing comment: `} // namespace MLIR_BINDINGS_PYTHON_DOMAIN`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace MLIR_BINDINGS_PYTHON_DOMAIN`。
- **L98**: Closes a namespace scope while preserving the trailing comment: `} // namespace python`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace python`。
- **L99**: Closes a namespace scope while preserving the trailing comment: `} // namespace mlir`. / 结束一个命名空间作用域，并保留尾部注释：`} // namespace mlir`。
- **L100**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 101-106 / 第 101-106 行

```cpp
101 | NB_MODULE(_mlirDialectsAMDGPU, m) {
102 |   m.doc() = "MLIR AMDGPU dialect.";
103 | 
104 |   mlir::python::MLIR_BINDINGS_PYTHON_DOMAIN::amdgpu::
105 |       populateDialectAMDGPUSubmodule(m);
106 | }
```

- **L101**: Starts a function, method, lambda, or structured scope: `NB_MODULE(_mlirDialectsAMDGPU, m) {`. / 开始一个函数、方法、lambda 或结构化作用域：`NB_MODULE(_mlirDialectsAMDGPU, m) {`。
- **L102**: Executes a call or declaration centered on `m.doc`. / 执行以 `m.doc` 为核心的调用或声明。
- **L103**: Blank line separating nearby declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L104**: Continues the surrounding expression or declaration: `mlir::python::MLIR_BINDINGS_PYTHON_DOMAIN::amdgpu::`. / 继续构造周围的表达式或声明：`mlir::python::MLIR_BINDINGS_PYTHON_DOMAIN::amdgpu::`。
- **L105**: Executes a call or declaration centered on `populateDialectAMDGPUSubmodule`. / 执行以 `populateDialectAMDGPUSubmodule` 为核心的调用或声明。
- **L106**: Closes the current lexical scope or compound statement. / 结束当前词法作用域或复合语句块。

## Key Concepts / 关键概念

- **Type system / 类型系统**:
  - **EN**: Queries or constructs MLIR types and type relationships.
  - **CN**: 查询或构造 MLIR 类型及其关系。

## Dependencies / 依赖关系

- **Direct MLIR/LLVM includes / 直接的 MLIR/LLVM 包含**: `mlir-c/Dialect/AMDGPU.h`, `mlir-c/IR.h`, `mlir/Bindings/Python/IRCore.h`, `mlir/Bindings/Python/Nanobind.h`, `mlir/Bindings/Python/NanobindAdaptors.h`, `nanobind/nanobind.h`
