# BuiltinsAMDGPU.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsAMDGPU.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: AMDGPU Builtin function defs *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsAMDGPU` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1329

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//===--- BuiltinsAMDGPU.td - AMDGPU Builtin function defs -------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the AMDGPU-specific builtin function database.
//
//===----------------------------------------------------------------------===//

include "clang/Basic/BuiltinsBase.td"
include "clang/Basic/BuiltinsAMDGPUDocs.td"

//===----------------------------------------------------------------------===//
// AMDGPU builtin base classes
//===----------------------------------------------------------------------===//

class AMDGPUBuiltin<string prototype, list<Attribute> Attr = [], string Feat = ""> : TargetBuiltin {
  let Spellings = [NAME];
  let Prototype = prototype;
  let Attributes = !listconcat([NoThrow], Attr);
  let Features = Feat;
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, constraints, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, constraints, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释解释附近代码的逻辑、约束或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, constraints, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the AMDGPU-specific builtin function database.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the AMDGPU-specific builtin function database.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Includes TableGen file `"clang/Basic/BuiltinsBase.td"` so later records can reuse shared definitions.
  **L13 CN**: 引入 TableGen 文件 `"clang/Basic/BuiltinsBase.td"`，以便后续记录复用共享定义。
- **L14 EN**: Includes TableGen file `"clang/Basic/BuiltinsAMDGPUDocs.td"` so later records can reuse shared definitions.
  **L14 CN**: 引入 TableGen 文件 `"clang/Basic/BuiltinsAMDGPUDocs.td"`，以便后续记录复用共享定义。
- **L15 EN**: Blank line separating nearby declarations or logic blocks.
  **L15 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L16 EN**: Banner comment marking a file or section boundary.
  **L16 CN**: 横幅注释，用于标记文件或章节边界。
- **L17 EN**: Comment explains nearby logic, constraints, or intent: `AMDGPU builtin base classes`.
  **L17 CN**: 注释解释附近代码的逻辑、约束或设计意图：`AMDGPU builtin base classes`。
- **L18 EN**: Banner comment marking a file or section boundary.
  **L18 CN**: 横幅注释，用于标记文件或章节边界。
- **L19 EN**: Blank line separating nearby declarations or logic blocks.
  **L19 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L20 EN**: Declares TableGen class record `AMDGPUBuiltin`.
  **L20 CN**: 声明 TableGen class 记录 `AMDGPUBuiltin`。
- **L21 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Spellings = [NAME];`.
  **L21 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Spellings = [NAME];`。
- **L22 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Prototype = prototype;`.
  **L22 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Prototype = prototype;`。
- **L23 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = !listconcat([NoThrow], Attr);`.
  **L23 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = !listconcat([NoThrow], Attr);`。
- **L24 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = Feat;`.
  **L24 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = Feat;`。

### Lines 25-48

````tablegen
}

//===----------------------------------------------------------------------===//
// SI+ only builtins.
//===----------------------------------------------------------------------===//

def __builtin_amdgcn_dispatch_ptr
    : AMDGPUBuiltin<"void address_space<4> *()", [Const]> {
  let Documentation = [DocABIDispatchPtr];
}
def __builtin_amdgcn_kernarg_segment_ptr
    : AMDGPUBuiltin<"void address_space<4> *()", [Const]> {
  let Documentation = [DocABIKernargSegmentPtr];
}
def __builtin_amdgcn_implicitarg_ptr
    : AMDGPUBuiltin<"void address_space<4> *()", [Const]> {
  let Documentation = [DocABIImplicitargPtr];
}
def __builtin_amdgcn_queue_ptr
    : AMDGPUBuiltin<"void address_space<4> *()", [Const]> {
  let Documentation = [DocABIQueuePtr];
}

def __builtin_amdgcn_workgroup_id_x : AMDGPUBuiltin<"unsigned int()", [Const]> {
````
- **L25 EN**: Closes the current lexical scope or compound statement.
  **L25 CN**: 结束当前词法作用域或复合语句块。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Banner comment marking a file or section boundary.
  **L27 CN**: 横幅注释，用于标记文件或章节边界。
- **L28 EN**: Comment explains nearby logic, constraints, or intent: `SI+ only builtins.`.
  **L28 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SI+ only builtins.`。
- **L29 EN**: Banner comment marking a file or section boundary.
  **L29 CN**: 横幅注释，用于标记文件或章节边界。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Declares TableGen def record `__builtin_amdgcn_dispatch_ptr`.
  **L31 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_dispatch_ptr`。
- **L32 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`.
  **L32 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`。
- **L33 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIDispatchPtr];`.
  **L33 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIDispatchPtr];`。
- **L34 EN**: Closes the current lexical scope or compound statement.
  **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Declares TableGen def record `__builtin_amdgcn_kernarg_segment_ptr`.
  **L35 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_kernarg_segment_ptr`。
- **L36 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`.
  **L36 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`。
- **L37 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIKernargSegmentPtr];`.
  **L37 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIKernargSegmentPtr];`。
- **L38 EN**: Closes the current lexical scope or compound statement.
  **L38 CN**: 结束当前词法作用域或复合语句块。
- **L39 EN**: Declares TableGen def record `__builtin_amdgcn_implicitarg_ptr`.
  **L39 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_implicitarg_ptr`。
- **L40 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`.
  **L40 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`。
- **L41 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIImplicitargPtr];`.
  **L41 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIImplicitargPtr];`。
- **L42 EN**: Closes the current lexical scope or compound statement.
  **L42 CN**: 结束当前词法作用域或复合语句块。
- **L43 EN**: Declares TableGen def record `__builtin_amdgcn_queue_ptr`.
  **L43 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_queue_ptr`。
- **L44 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`.
  **L44 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"void address_space<4> *()", [Const]> {`。
- **L45 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIQueuePtr];`.
  **L45 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIQueuePtr];`。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Blank line separating nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L48 EN**: Declares TableGen def record `__builtin_amdgcn_workgroup_id_x`.
  **L48 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workgroup_id_x`。

### Lines 49-72

````tablegen
  let Documentation = [DocABIWorkgroupIdX];
}
def __builtin_amdgcn_workgroup_id_y : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIWorkgroupIdY];
}
def __builtin_amdgcn_workgroup_id_z : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIWorkgroupIdZ];
}

def __builtin_amdgcn_cluster_id_x
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterIdX];
}
def __builtin_amdgcn_cluster_id_y
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterIdY];
}
def __builtin_amdgcn_cluster_id_z
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterIdZ];
}

def __builtin_amdgcn_cluster_workgroup_id_x
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
````
- **L49 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkgroupIdX];`.
  **L49 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkgroupIdX];`。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Declares TableGen def record `__builtin_amdgcn_workgroup_id_y`.
  **L51 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workgroup_id_y`。
- **L52 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkgroupIdY];`.
  **L52 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkgroupIdY];`。
- **L53 EN**: Closes the current lexical scope or compound statement.
  **L53 CN**: 结束当前词法作用域或复合语句块。
- **L54 EN**: Declares TableGen def record `__builtin_amdgcn_workgroup_id_z`.
  **L54 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workgroup_id_z`。
- **L55 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkgroupIdZ];`.
  **L55 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkgroupIdZ];`。
- **L56 EN**: Closes the current lexical scope or compound statement.
  **L56 CN**: 结束当前词法作用域或复合语句块。
- **L57 EN**: Blank line separating nearby declarations or logic blocks.
  **L57 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L58 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_id_x`.
  **L58 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_id_x`。
- **L59 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L59 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L60 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterIdX];`.
  **L60 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterIdX];`。
- **L61 EN**: Closes the current lexical scope or compound statement.
  **L61 CN**: 结束当前词法作用域或复合语句块。
- **L62 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_id_y`.
  **L62 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_id_y`。
- **L63 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L63 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L64 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterIdY];`.
  **L64 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterIdY];`。
- **L65 EN**: Closes the current lexical scope or compound statement.
  **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_id_z`.
  **L66 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_id_z`。
- **L67 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L67 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L68 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterIdZ];`.
  **L68 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterIdZ];`。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Blank line separating nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L71 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_id_x`.
  **L71 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_id_x`。
- **L72 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L72 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。

### Lines 73-96

````tablegen
  let Documentation = [DocABIClusterWorkgroupIdX];
}
def __builtin_amdgcn_cluster_workgroup_id_y
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterWorkgroupIdY];
}
def __builtin_amdgcn_cluster_workgroup_id_z
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterWorkgroupIdZ];
}
def __builtin_amdgcn_cluster_workgroup_flat_id
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterWorkgroupFlatId];
}

def __builtin_amdgcn_cluster_workgroup_max_id_x
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterWorkgroupMaxIdX];
}
def __builtin_amdgcn_cluster_workgroup_max_id_y
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterWorkgroupMaxIdY];
}
def __builtin_amdgcn_cluster_workgroup_max_id_z
````
- **L73 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupIdX];`.
  **L73 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupIdX];`。
- **L74 EN**: Closes the current lexical scope or compound statement.
  **L74 CN**: 结束当前词法作用域或复合语句块。
- **L75 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_id_y`.
  **L75 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_id_y`。
- **L76 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L76 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L77 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupIdY];`.
  **L77 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupIdY];`。
- **L78 EN**: Closes the current lexical scope or compound statement.
  **L78 CN**: 结束当前词法作用域或复合语句块。
- **L79 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_id_z`.
  **L79 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_id_z`。
- **L80 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L80 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L81 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupIdZ];`.
  **L81 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupIdZ];`。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_flat_id`.
  **L83 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_flat_id`。
- **L84 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L84 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L85 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupFlatId];`.
  **L85 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupFlatId];`。
- **L86 EN**: Closes the current lexical scope or compound statement.
  **L86 CN**: 结束当前词法作用域或复合语句块。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L88 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_max_id_x`.
  **L88 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_max_id_x`。
- **L89 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L89 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L90 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupMaxIdX];`.
  **L90 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupMaxIdX];`。
- **L91 EN**: Closes the current lexical scope or compound statement.
  **L91 CN**: 结束当前词法作用域或复合语句块。
- **L92 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_max_id_y`.
  **L92 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_max_id_y`。
- **L93 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L93 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L94 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupMaxIdY];`.
  **L94 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupMaxIdY];`。
- **L95 EN**: Closes the current lexical scope or compound statement.
  **L95 CN**: 结束当前词法作用域或复合语句块。
- **L96 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_max_id_z`.
  **L96 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_max_id_z`。

### Lines 97-120

````tablegen
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterWorkgroupMaxIdZ];
}
def __builtin_amdgcn_cluster_workgroup_max_flat_id
    : AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {
  let Documentation = [DocABIClusterWorkgroupMaxFlatId];
}

def __builtin_amdgcn_workitem_id_x : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIWorkitemIdX];
}
def __builtin_amdgcn_workitem_id_y : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIWorkitemIdY];
}
def __builtin_amdgcn_workitem_id_z : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIWorkitemIdZ];
}

def __builtin_amdgcn_workgroup_size_x
    : AMDGPUBuiltin<"unsigned short()", [Const]> {
  let Documentation = [DocABIWorkgroupSizeX];
}
def __builtin_amdgcn_workgroup_size_y
    : AMDGPUBuiltin<"unsigned short()", [Const]> {
````
- **L97 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L97 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L98 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupMaxIdZ];`.
  **L98 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupMaxIdZ];`。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_workgroup_max_flat_id`.
  **L100 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_workgroup_max_flat_id`。
- **L101 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`.
  **L101 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned int()", [Const], "gfx1250-insts"> {`。
- **L102 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIClusterWorkgroupMaxFlatId];`.
  **L102 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIClusterWorkgroupMaxFlatId];`。
- **L103 EN**: Closes the current lexical scope or compound statement.
  **L103 CN**: 结束当前词法作用域或复合语句块。
- **L104 EN**: Blank line separating nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L105 EN**: Declares TableGen def record `__builtin_amdgcn_workitem_id_x`.
  **L105 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workitem_id_x`。
- **L106 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkitemIdX];`.
  **L106 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkitemIdX];`。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Declares TableGen def record `__builtin_amdgcn_workitem_id_y`.
  **L108 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workitem_id_y`。
- **L109 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkitemIdY];`.
  **L109 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkitemIdY];`。
- **L110 EN**: Closes the current lexical scope or compound statement.
  **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Declares TableGen def record `__builtin_amdgcn_workitem_id_z`.
  **L111 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workitem_id_z`。
- **L112 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkitemIdZ];`.
  **L112 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkitemIdZ];`。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Declares TableGen def record `__builtin_amdgcn_workgroup_size_x`.
  **L115 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workgroup_size_x`。
- **L116 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned short()", [Const]> {`.
  **L116 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned short()", [Const]> {`。
- **L117 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkgroupSizeX];`.
  **L117 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkgroupSizeX];`。
- **L118 EN**: Closes the current lexical scope or compound statement.
  **L118 CN**: 结束当前词法作用域或复合语句块。
- **L119 EN**: Declares TableGen def record `__builtin_amdgcn_workgroup_size_y`.
  **L119 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workgroup_size_y`。
- **L120 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned short()", [Const]> {`.
  **L120 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned short()", [Const]> {`。

### Lines 121-144

````tablegen
  let Documentation = [DocABIWorkgroupSizeY];
}
def __builtin_amdgcn_workgroup_size_z
    : AMDGPUBuiltin<"unsigned short()", [Const]> {
  let Documentation = [DocABIWorkgroupSizeZ];
}

def __builtin_amdgcn_grid_size_x : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIGridSizeX];
}
def __builtin_amdgcn_grid_size_y : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIGridSizeY];
}
def __builtin_amdgcn_grid_size_z : AMDGPUBuiltin<"unsigned int()", [Const]> {
  let Documentation = [DocABIGridSizeZ];
}

def __builtin_amdgcn_mbcnt_hi : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_mbcnt_lo : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int)", [Const]>;

def __builtin_amdgcn_s_memtime : AMDGPUBuiltin<"uint64_t()", [], "s-memtime-inst">;

//===----------------------------------------------------------------------===//
// Instruction builtins.
````
- **L121 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkgroupSizeY];`.
  **L121 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkgroupSizeY];`。
- **L122 EN**: Closes the current lexical scope or compound statement.
  **L122 CN**: 结束当前词法作用域或复合语句块。
- **L123 EN**: Declares TableGen def record `__builtin_amdgcn_workgroup_size_z`.
  **L123 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_workgroup_size_z`。
- **L124 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `: AMDGPUBuiltin<"unsigned short()", [Const]> {`.
  **L124 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`: AMDGPUBuiltin<"unsigned short()", [Const]> {`。
- **L125 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIWorkgroupSizeZ];`.
  **L125 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIWorkgroupSizeZ];`。
- **L126 EN**: Closes the current lexical scope or compound statement.
  **L126 CN**: 结束当前词法作用域或复合语句块。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L128 EN**: Declares TableGen def record `__builtin_amdgcn_grid_size_x`.
  **L128 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_grid_size_x`。
- **L129 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIGridSizeX];`.
  **L129 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIGridSizeX];`。
- **L130 EN**: Closes the current lexical scope or compound statement.
  **L130 CN**: 结束当前词法作用域或复合语句块。
- **L131 EN**: Declares TableGen def record `__builtin_amdgcn_grid_size_y`.
  **L131 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_grid_size_y`。
- **L132 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIGridSizeY];`.
  **L132 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIGridSizeY];`。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Declares TableGen def record `__builtin_amdgcn_grid_size_z`.
  **L134 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_grid_size_z`。
- **L135 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocABIGridSizeZ];`.
  **L135 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocABIGridSizeZ];`。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Declares TableGen def record `__builtin_amdgcn_mbcnt_hi`.
  **L138 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mbcnt_hi`。
- **L139 EN**: Declares TableGen def record `__builtin_amdgcn_mbcnt_lo`.
  **L139 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mbcnt_lo`。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L141 EN**: Declares TableGen def record `__builtin_amdgcn_s_memtime`.
  **L141 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_memtime`。
- **L142 EN**: Blank line separating nearby declarations or logic blocks.
  **L142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L143 EN**: Banner comment marking a file or section boundary.
  **L143 CN**: 横幅注释，用于标记文件或章节边界。
- **L144 EN**: Comment explains nearby logic, constraints, or intent: `Instruction builtins.`.
  **L144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Instruction builtins.`。

### Lines 145-168

````tablegen
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_s_getreg : AMDGPUBuiltin<"unsigned int(_Constant int)">;
def __builtin_amdgcn_s_setreg : AMDGPUBuiltin<"void(_Constant int, unsigned int)">;
def __builtin_amdgcn_s_getpc : AMDGPUBuiltin<"uint64_t()">;
def __builtin_amdgcn_s_waitcnt : AMDGPUBuiltin<"void(_Constant int)">;
def __builtin_amdgcn_s_sendmsg : AMDGPUBuiltin<"void(_Constant int, unsigned int)">;
def __builtin_amdgcn_s_sendmsghalt : AMDGPUBuiltin<"void(_Constant int, unsigned int)">;
def __builtin_amdgcn_s_barrier : AMDGPUBuiltin<"void()">;
def __builtin_amdgcn_s_ttracedata : AMDGPUBuiltin<"void(int)">;
def __builtin_amdgcn_wave_barrier : AMDGPUBuiltin<"void()">;
def __builtin_amdgcn_sched_barrier : AMDGPUBuiltin<"void(_Constant int)">;
def __builtin_amdgcn_sched_group_barrier : AMDGPUBuiltin<"void(_Constant int, _Constant int, _Constant int)">;
def __builtin_amdgcn_iglp_opt : AMDGPUBuiltin<"void(_Constant int)">;
def __builtin_amdgcn_s_dcache_inv : AMDGPUBuiltin<"void()">;
def __builtin_amdgcn_buffer_wbinvl1 : AMDGPUBuiltin<"void()">;
def __builtin_amdgcn_fence : AMDGPUBuiltin<"void(unsigned int, char const *, ...)">;
def __builtin_amdgcn_groupstaticsize : AMDGPUBuiltin<"unsigned int()">;
def __builtin_amdgcn_wavefrontsize : AMDGPUBuiltin<"unsigned int()", [Const]>;

def __builtin_amdgcn_atomic_inc32 : AMDGPUBuiltin<"uint32_t(uint32_t volatile *, uint32_t, unsigned int, char const *)">;
def __builtin_amdgcn_atomic_inc64 : AMDGPUBuiltin<"uint64_t(uint64_t volatile *, uint64_t, unsigned int, char const *)">;

def __builtin_amdgcn_atomic_dec32 : AMDGPUBuiltin<"uint32_t(uint32_t volatile *, uint32_t, unsigned int, char const *)">;
def __builtin_amdgcn_atomic_dec64 : AMDGPUBuiltin<"uint64_t(uint64_t volatile *, uint64_t, unsigned int, char const *)">;
````
- **L145 EN**: Banner comment marking a file or section boundary.
  **L145 CN**: 横幅注释，用于标记文件或章节边界。
- **L146 EN**: Declares TableGen def record `__builtin_amdgcn_s_getreg`.
  **L146 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_getreg`。
- **L147 EN**: Declares TableGen def record `__builtin_amdgcn_s_setreg`.
  **L147 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_setreg`。
- **L148 EN**: Declares TableGen def record `__builtin_amdgcn_s_getpc`.
  **L148 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_getpc`。
- **L149 EN**: Declares TableGen def record `__builtin_amdgcn_s_waitcnt`.
  **L149 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_waitcnt`。
- **L150 EN**: Declares TableGen def record `__builtin_amdgcn_s_sendmsg`.
  **L150 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_sendmsg`。
- **L151 EN**: Declares TableGen def record `__builtin_amdgcn_s_sendmsghalt`.
  **L151 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_sendmsghalt`。
- **L152 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier`.
  **L152 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier`。
- **L153 EN**: Declares TableGen def record `__builtin_amdgcn_s_ttracedata`.
  **L153 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_ttracedata`。
- **L154 EN**: Declares TableGen def record `__builtin_amdgcn_wave_barrier`.
  **L154 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_barrier`。
- **L155 EN**: Declares TableGen def record `__builtin_amdgcn_sched_barrier`.
  **L155 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sched_barrier`。
- **L156 EN**: Declares TableGen def record `__builtin_amdgcn_sched_group_barrier`.
  **L156 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sched_group_barrier`。
- **L157 EN**: Declares TableGen def record `__builtin_amdgcn_iglp_opt`.
  **L157 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_iglp_opt`。
- **L158 EN**: Declares TableGen def record `__builtin_amdgcn_s_dcache_inv`.
  **L158 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_dcache_inv`。
- **L159 EN**: Declares TableGen def record `__builtin_amdgcn_buffer_wbinvl1`.
  **L159 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_buffer_wbinvl1`。
- **L160 EN**: Declares TableGen def record `__builtin_amdgcn_fence`.
  **L160 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fence`。
- **L161 EN**: Declares TableGen def record `__builtin_amdgcn_groupstaticsize`.
  **L161 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_groupstaticsize`。
- **L162 EN**: Declares TableGen def record `__builtin_amdgcn_wavefrontsize`.
  **L162 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wavefrontsize`。
- **L163 EN**: Blank line separating nearby declarations or logic blocks.
  **L163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L164 EN**: Declares TableGen def record `__builtin_amdgcn_atomic_inc32`.
  **L164 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_atomic_inc32`。
- **L165 EN**: Declares TableGen def record `__builtin_amdgcn_atomic_inc64`.
  **L165 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_atomic_inc64`。
- **L166 EN**: Blank line separating nearby declarations or logic blocks.
  **L166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L167 EN**: Declares TableGen def record `__builtin_amdgcn_atomic_dec32`.
  **L167 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_atomic_dec32`。
- **L168 EN**: Declares TableGen def record `__builtin_amdgcn_atomic_dec64`.
  **L168 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_atomic_dec64`。

### Lines 169-192

````tablegen

// FIXME: Need to disallow constant address space.
def __builtin_amdgcn_div_scale : AMDGPUBuiltin<"double(double, double, bool, bool *)">;
def __builtin_amdgcn_div_scalef : AMDGPUBuiltin<"float(float, float, bool, bool *)">;
def __builtin_amdgcn_div_fmas : AMDGPUBuiltin<"double(double, double, double, bool)", [Const]>;
def __builtin_amdgcn_div_fmasf : AMDGPUBuiltin<"float(float, float, float, bool)", [Const]>;
def __builtin_amdgcn_div_fixup : AMDGPUBuiltin<"double(double, double, double)", [Const]>;
def __builtin_amdgcn_div_fixupf : AMDGPUBuiltin<"float(float, float, float)", [Const]>;
def __builtin_amdgcn_trig_preop : AMDGPUBuiltin<"double(double, int)", [Const]>;
def __builtin_amdgcn_trig_preopf : AMDGPUBuiltin<"float(float, int)", [Const]>;
def __builtin_amdgcn_rcp : AMDGPUBuiltin<"double(double)", [Const]>;
def __builtin_amdgcn_rcpf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_sqrt : AMDGPUBuiltin<"double(double)", [Const]>;
def __builtin_amdgcn_sqrtf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_rsq : AMDGPUBuiltin<"double(double)", [Const]>;
def __builtin_amdgcn_rsqf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_rsq_clamp : AMDGPUBuiltin<"double(double)", [Const]>;
def __builtin_amdgcn_rsq_clampf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_sinf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_cosf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_logf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_exp2f : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_log_clampf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_ldexp : AMDGPUBuiltin<"double(double, int)", [Const]>;
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L170 EN**: Comment records a pending task or caution: `FIXME: Need to disallow constant address space.`.
  **L170 CN**: 注释记录待办事项或注意点：`FIXME: Need to disallow constant address space.`。
- **L171 EN**: Declares TableGen def record `__builtin_amdgcn_div_scale`.
  **L171 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_div_scale`。
- **L172 EN**: Declares TableGen def record `__builtin_amdgcn_div_scalef`.
  **L172 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_div_scalef`。
- **L173 EN**: Declares TableGen def record `__builtin_amdgcn_div_fmas`.
  **L173 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_div_fmas`。
- **L174 EN**: Declares TableGen def record `__builtin_amdgcn_div_fmasf`.
  **L174 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_div_fmasf`。
- **L175 EN**: Declares TableGen def record `__builtin_amdgcn_div_fixup`.
  **L175 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_div_fixup`。
- **L176 EN**: Declares TableGen def record `__builtin_amdgcn_div_fixupf`.
  **L176 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_div_fixupf`。
- **L177 EN**: Declares TableGen def record `__builtin_amdgcn_trig_preop`.
  **L177 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_trig_preop`。
- **L178 EN**: Declares TableGen def record `__builtin_amdgcn_trig_preopf`.
  **L178 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_trig_preopf`。
- **L179 EN**: Declares TableGen def record `__builtin_amdgcn_rcp`.
  **L179 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rcp`。
- **L180 EN**: Declares TableGen def record `__builtin_amdgcn_rcpf`.
  **L180 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rcpf`。
- **L181 EN**: Declares TableGen def record `__builtin_amdgcn_sqrt`.
  **L181 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sqrt`。
- **L182 EN**: Declares TableGen def record `__builtin_amdgcn_sqrtf`.
  **L182 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sqrtf`。
- **L183 EN**: Declares TableGen def record `__builtin_amdgcn_rsq`.
  **L183 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rsq`。
- **L184 EN**: Declares TableGen def record `__builtin_amdgcn_rsqf`.
  **L184 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rsqf`。
- **L185 EN**: Declares TableGen def record `__builtin_amdgcn_rsq_clamp`.
  **L185 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rsq_clamp`。
- **L186 EN**: Declares TableGen def record `__builtin_amdgcn_rsq_clampf`.
  **L186 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rsq_clampf`。
- **L187 EN**: Declares TableGen def record `__builtin_amdgcn_sinf`.
  **L187 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sinf`。
- **L188 EN**: Declares TableGen def record `__builtin_amdgcn_cosf`.
  **L188 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cosf`。
- **L189 EN**: Declares TableGen def record `__builtin_amdgcn_logf`.
  **L189 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_logf`。
- **L190 EN**: Declares TableGen def record `__builtin_amdgcn_exp2f`.
  **L190 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_exp2f`。
- **L191 EN**: Declares TableGen def record `__builtin_amdgcn_log_clampf`.
  **L191 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_log_clampf`。
- **L192 EN**: Declares TableGen def record `__builtin_amdgcn_ldexp`.
  **L192 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ldexp`。

### Lines 193-216

````tablegen
def __builtin_amdgcn_ldexpf : AMDGPUBuiltin<"float(float, int)", [Const]>;
def __builtin_amdgcn_frexp_mant : AMDGPUBuiltin<"double(double)", [Const]>;
def __builtin_amdgcn_frexp_mantf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_frexp_exp : AMDGPUBuiltin<"int(double)", [Const]>;
def __builtin_amdgcn_frexp_expf : AMDGPUBuiltin<"int(float)", [Const]>;
def __builtin_amdgcn_fract : AMDGPUBuiltin<"double(double)", [Const]>;
def __builtin_amdgcn_fractf : AMDGPUBuiltin<"float(float)", [Const]>;
def __builtin_amdgcn_lerp : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const], "lerp-inst">;
def __builtin_amdgcn_class : AMDGPUBuiltin<"bool(double, int)", [Const]>;
def __builtin_amdgcn_classf : AMDGPUBuiltin<"bool(float, int)", [Const]>;
def __builtin_amdgcn_cubeid : AMDGPUBuiltin<"float(float, float, float)", [Const], "cube-insts">;
def __builtin_amdgcn_cubesc : AMDGPUBuiltin<"float(float, float, float)", [Const], "cube-insts">;
def __builtin_amdgcn_cubetc : AMDGPUBuiltin<"float(float, float, float)", [Const], "cube-insts">;
def __builtin_amdgcn_cubema : AMDGPUBuiltin<"float(float, float, float)", [Const], "cube-insts">;
def __builtin_amdgcn_s_sleep : AMDGPUBuiltin<"void(_Constant int)">;
def __builtin_amdgcn_s_incperflevel : AMDGPUBuiltin<"void(_Constant int)">;
def __builtin_amdgcn_s_decperflevel : AMDGPUBuiltin<"void(_Constant int)">;
def __builtin_amdgcn_s_setprio : AMDGPUBuiltin<"void(_Constant short)">;
def __builtin_amdgcn_ds_swizzle : AMDGPUBuiltin<"int(int, _Constant int)", [Const]>;
def __builtin_amdgcn_ds_permute : AMDGPUBuiltin<"int(int, int)", [Const]>;
def __builtin_amdgcn_ds_bpermute : AMDGPUBuiltin<"int(int, int)", [Const]>;
def __builtin_amdgcn_readfirstlane : AMDGPUBuiltin<"int(int)", [Const]>;
def __builtin_amdgcn_readlane : AMDGPUBuiltin<"int(int, int)", [Const]>;
def __builtin_amdgcn_wave_shuffle : AMDGPUBuiltin<"int(int, int)", [Const]> {
````
- **L193 EN**: Declares TableGen def record `__builtin_amdgcn_ldexpf`.
  **L193 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ldexpf`。
- **L194 EN**: Declares TableGen def record `__builtin_amdgcn_frexp_mant`.
  **L194 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_frexp_mant`。
- **L195 EN**: Declares TableGen def record `__builtin_amdgcn_frexp_mantf`.
  **L195 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_frexp_mantf`。
- **L196 EN**: Declares TableGen def record `__builtin_amdgcn_frexp_exp`.
  **L196 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_frexp_exp`。
- **L197 EN**: Declares TableGen def record `__builtin_amdgcn_frexp_expf`.
  **L197 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_frexp_expf`。
- **L198 EN**: Declares TableGen def record `__builtin_amdgcn_fract`.
  **L198 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fract`。
- **L199 EN**: Declares TableGen def record `__builtin_amdgcn_fractf`.
  **L199 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fractf`。
- **L200 EN**: Declares TableGen def record `__builtin_amdgcn_lerp`.
  **L200 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_lerp`。
- **L201 EN**: Declares TableGen def record `__builtin_amdgcn_class`.
  **L201 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_class`。
- **L202 EN**: Declares TableGen def record `__builtin_amdgcn_classf`.
  **L202 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_classf`。
- **L203 EN**: Declares TableGen def record `__builtin_amdgcn_cubeid`.
  **L203 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cubeid`。
- **L204 EN**: Declares TableGen def record `__builtin_amdgcn_cubesc`.
  **L204 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cubesc`。
- **L205 EN**: Declares TableGen def record `__builtin_amdgcn_cubetc`.
  **L205 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cubetc`。
- **L206 EN**: Declares TableGen def record `__builtin_amdgcn_cubema`.
  **L206 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cubema`。
- **L207 EN**: Declares TableGen def record `__builtin_amdgcn_s_sleep`.
  **L207 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_sleep`。
- **L208 EN**: Declares TableGen def record `__builtin_amdgcn_s_incperflevel`.
  **L208 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_incperflevel`。
- **L209 EN**: Declares TableGen def record `__builtin_amdgcn_s_decperflevel`.
  **L209 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_decperflevel`。
- **L210 EN**: Declares TableGen def record `__builtin_amdgcn_s_setprio`.
  **L210 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_setprio`。
- **L211 EN**: Declares TableGen def record `__builtin_amdgcn_ds_swizzle`.
  **L211 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_swizzle`。
- **L212 EN**: Declares TableGen def record `__builtin_amdgcn_ds_permute`.
  **L212 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_permute`。
- **L213 EN**: Declares TableGen def record `__builtin_amdgcn_ds_bpermute`.
  **L213 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_bpermute`。
- **L214 EN**: Declares TableGen def record `__builtin_amdgcn_readfirstlane`.
  **L214 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_readfirstlane`。
- **L215 EN**: Declares TableGen def record `__builtin_amdgcn_readlane`.
  **L215 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_readlane`。
- **L216 EN**: Declares TableGen def record `__builtin_amdgcn_wave_shuffle`.
  **L216 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_shuffle`。

### Lines 217-240

````tablegen
  let Documentation = [DocWaveShuffle];
  let ArgNames = ["src", "idx"];
}
def __builtin_amdgcn_fmed3f : AMDGPUBuiltin<"float(float, float, float)", [Const]>;
def __builtin_amdgcn_ds_faddf : AMDGPUBuiltin<"float(float address_space<3> *, float, _Constant int, _Constant int, _Constant bool)">;
def __builtin_amdgcn_ds_fminf : AMDGPUBuiltin<"float(float address_space<3> *, float, _Constant int, _Constant int, _Constant bool)">;
def __builtin_amdgcn_ds_fmaxf : AMDGPUBuiltin<"float(float address_space<3> *, float, _Constant int, _Constant int, _Constant bool)">;
def __builtin_amdgcn_ds_append : AMDGPUBuiltin<"int(int address_space<3> *)">;
def __builtin_amdgcn_ds_consume : AMDGPUBuiltin<"int(int address_space<3> *)">;
def __builtin_amdgcn_alignbit : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_alignbyte : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_ubfe : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_sbfe : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_cvt_pkrtz : AMDGPUBuiltin<"_ExtVector<2, __fp16>(float, float)", [Const]>;
def __builtin_amdgcn_cvt_pknorm_i16 : AMDGPUBuiltin<"_ExtVector<2, short>(float, float)", [Const], "cvt-pknorm-vop2-insts">;
def __builtin_amdgcn_cvt_pknorm_u16 : AMDGPUBuiltin<"_ExtVector<2, unsigned short>(float, float)", [Const], "cvt-pknorm-vop2-insts">;
def __builtin_amdgcn_cvt_pk_i16 : AMDGPUBuiltin<"_ExtVector<2, short>(int, int)", [Const]>;
def __builtin_amdgcn_cvt_pk_u16 : AMDGPUBuiltin<"_ExtVector<2, unsigned short>(unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_cvt_pk_u8_f32 : AMDGPUBuiltin<"unsigned int(float, unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_cvt_off_f32_i4 : AMDGPUBuiltin<"float(int)", [Const]>;
def __builtin_amdgcn_msad_u8 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_sad_u8 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const], "sad-insts">;
def __builtin_amdgcn_sad_hi_u8 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const], "sad-insts">;
def __builtin_amdgcn_sad_u16 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const], "sad-insts">;
````
- **L217 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWaveShuffle];`.
  **L217 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWaveShuffle];`。
- **L218 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["src", "idx"];`.
  **L218 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["src", "idx"];`。
- **L219 EN**: Closes the current lexical scope or compound statement.
  **L219 CN**: 结束当前词法作用域或复合语句块。
- **L220 EN**: Declares TableGen def record `__builtin_amdgcn_fmed3f`.
  **L220 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fmed3f`。
- **L221 EN**: Declares TableGen def record `__builtin_amdgcn_ds_faddf`.
  **L221 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_faddf`。
- **L222 EN**: Declares TableGen def record `__builtin_amdgcn_ds_fminf`.
  **L222 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_fminf`。
- **L223 EN**: Declares TableGen def record `__builtin_amdgcn_ds_fmaxf`.
  **L223 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_fmaxf`。
- **L224 EN**: Declares TableGen def record `__builtin_amdgcn_ds_append`.
  **L224 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_append`。
- **L225 EN**: Declares TableGen def record `__builtin_amdgcn_ds_consume`.
  **L225 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_consume`。
- **L226 EN**: Declares TableGen def record `__builtin_amdgcn_alignbit`.
  **L226 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_alignbit`。
- **L227 EN**: Declares TableGen def record `__builtin_amdgcn_alignbyte`.
  **L227 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_alignbyte`。
- **L228 EN**: Declares TableGen def record `__builtin_amdgcn_ubfe`.
  **L228 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ubfe`。
- **L229 EN**: Declares TableGen def record `__builtin_amdgcn_sbfe`.
  **L229 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sbfe`。
- **L230 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pkrtz`.
  **L230 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pkrtz`。
- **L231 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pknorm_i16`.
  **L231 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pknorm_i16`。
- **L232 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pknorm_u16`.
  **L232 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pknorm_u16`。
- **L233 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_i16`.
  **L233 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_i16`。
- **L234 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_u16`.
  **L234 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_u16`。
- **L235 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_u8_f32`.
  **L235 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_u8_f32`。
- **L236 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_off_f32_i4`.
  **L236 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_off_f32_i4`。
- **L237 EN**: Declares TableGen def record `__builtin_amdgcn_msad_u8`.
  **L237 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_msad_u8`。
- **L238 EN**: Declares TableGen def record `__builtin_amdgcn_sad_u8`.
  **L238 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sad_u8`。
- **L239 EN**: Declares TableGen def record `__builtin_amdgcn_sad_hi_u8`.
  **L239 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sad_hi_u8`。
- **L240 EN**: Declares TableGen def record `__builtin_amdgcn_sad_u16`.
  **L240 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sad_u16`。

### Lines 241-264

````tablegen
def __builtin_amdgcn_qsad_pk_u16_u8 : AMDGPUBuiltin<"uint64_t(uint64_t, unsigned int, uint64_t)", [Const], "qsad-insts">;
def __builtin_amdgcn_mqsad_pk_u16_u8 : AMDGPUBuiltin<"uint64_t(uint64_t, unsigned int, uint64_t)", [Const]>;
def __builtin_amdgcn_mqsad_u32_u8 : AMDGPUBuiltin<"_ExtVector<4, unsigned int>(uint64_t, unsigned int, _ExtVector<4, unsigned int>)", [Const]>;

def __builtin_amdgcn_make_buffer_rsrc : AMDGPUBuiltin<"__amdgpu_buffer_rsrc_t(void *, short, int64_t, int)", [Const]>;
def __builtin_amdgcn_raw_buffer_store_b8 : AMDGPUBuiltin<"void(unsigned char, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_store_b16 : AMDGPUBuiltin<"void(unsigned short, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_store_b32 : AMDGPUBuiltin<"void(unsigned int, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_store_b64 : AMDGPUBuiltin<"void(_ExtVector<2, unsigned int>, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_store_b96 : AMDGPUBuiltin<"void(_ExtVector<3, unsigned int>, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_store_b128 : AMDGPUBuiltin<"void(_ExtVector<4, unsigned int>, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_load_b8 : AMDGPUBuiltin<"unsigned char(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_load_b16 : AMDGPUBuiltin<"unsigned short(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_load_b32 : AMDGPUBuiltin<"unsigned int(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_load_b64 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_load_b96 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_load_b128 : AMDGPUBuiltin<"_ExtVector<4, unsigned int>(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;

def __builtin_amdgcn_raw_buffer_load_format_v4f32 : AMDGPUBuiltin<"_ExtVector<4, float>(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_load_format_v4f16 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(__amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_store_format_v4f32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_raw_buffer_store_format_v4f16 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;
def __builtin_amdgcn_struct_buffer_load_format_v4f32 : AMDGPUBuiltin<"_ExtVector<4, float>(__amdgpu_buffer_rsrc_t, int, int, int, _Constant int)">;
def __builtin_amdgcn_struct_buffer_load_format_v4f16 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(__amdgpu_buffer_rsrc_t, int, int, int, _Constant int)">;
````
- **L241 EN**: Declares TableGen def record `__builtin_amdgcn_qsad_pk_u16_u8`.
  **L241 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_qsad_pk_u16_u8`。
- **L242 EN**: Declares TableGen def record `__builtin_amdgcn_mqsad_pk_u16_u8`.
  **L242 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mqsad_pk_u16_u8`。
- **L243 EN**: Declares TableGen def record `__builtin_amdgcn_mqsad_u32_u8`.
  **L243 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mqsad_u32_u8`。
- **L244 EN**: Blank line separating nearby declarations or logic blocks.
  **L244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L245 EN**: Declares TableGen def record `__builtin_amdgcn_make_buffer_rsrc`.
  **L245 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_make_buffer_rsrc`。
- **L246 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_b8`.
  **L246 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_b8`。
- **L247 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_b16`.
  **L247 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_b16`。
- **L248 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_b32`.
  **L248 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_b32`。
- **L249 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_b64`.
  **L249 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_b64`。
- **L250 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_b96`.
  **L250 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_b96`。
- **L251 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_b128`.
  **L251 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_b128`。
- **L252 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_b8`.
  **L252 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_b8`。
- **L253 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_b16`.
  **L253 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_b16`。
- **L254 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_b32`.
  **L254 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_b32`。
- **L255 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_b64`.
  **L255 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_b64`。
- **L256 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_b96`.
  **L256 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_b96`。
- **L257 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_b128`.
  **L257 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_b128`。
- **L258 EN**: Blank line separating nearby declarations or logic blocks.
  **L258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L259 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_format_v4f32`.
  **L259 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_format_v4f32`。
- **L260 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_load_format_v4f16`.
  **L260 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_load_format_v4f16`。
- **L261 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_format_v4f32`.
  **L261 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_format_v4f32`。
- **L262 EN**: Declares TableGen def record `__builtin_amdgcn_raw_buffer_store_format_v4f16`.
  **L262 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_buffer_store_format_v4f16`。
- **L263 EN**: Declares TableGen def record `__builtin_amdgcn_struct_buffer_load_format_v4f32`.
  **L263 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_struct_buffer_load_format_v4f32`。
- **L264 EN**: Declares TableGen def record `__builtin_amdgcn_struct_buffer_load_format_v4f16`.
  **L264 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_struct_buffer_load_format_v4f16`。

### Lines 265-288

````tablegen
def __builtin_amdgcn_struct_buffer_store_format_v4f32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, __amdgpu_buffer_rsrc_t, int, int, int, _Constant int)">;
def __builtin_amdgcn_struct_buffer_store_format_v4f16 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, __amdgpu_buffer_rsrc_t, int, int, int, _Constant int)">;

def __builtin_amdgcn_raw_ptr_buffer_atomic_add_i32 : AMDGPUBuiltin<"int(int, __amdgpu_buffer_rsrc_t, int, int, _Constant int)">;

def __builtin_amdgcn_raw_ptr_buffer_atomic_fadd_f32 : AMDGPUBuiltin<"float(float, __amdgpu_buffer_rsrc_t, int, int, _Constant int)", [], "atomic-fadd-rtn-insts">;
def __builtin_amdgcn_raw_ptr_buffer_atomic_fadd_v2f16 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(_ExtVector<2, _Float16>, __amdgpu_buffer_rsrc_t, int, int, _Constant int)", [], "atomic-buffer-global-pk-add-f16-insts">;

def __builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f32 : AMDGPUBuiltin<"float(float, __amdgpu_buffer_rsrc_t, int, int, _Constant int)", [], "atomic-fmin-fmax-global-f32">;
def __builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f32 : AMDGPUBuiltin<"float(float, __amdgpu_buffer_rsrc_t, int, int, _Constant int)", [], "atomic-fmin-fmax-global-f32">;
def __builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f64 : AMDGPUBuiltin<"double(double, __amdgpu_buffer_rsrc_t, int, int, _Constant int)", [], "atomic-fmin-fmax-global-f64">;
def __builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f64 : AMDGPUBuiltin<"double(double, __amdgpu_buffer_rsrc_t, int, int, _Constant int)", [], "atomic-fmin-fmax-global-f64">;

def __builtin_amdgcn_raw_ptr_buffer_load_lds : AMDGPUBuiltin<"void(__amdgpu_buffer_rsrc_t, void address_space<3> *, _Constant unsigned int, int, int, _Constant int, _Constant int)", [], "vmem-to-lds-load-insts">;
def __builtin_amdgcn_raw_ptr_buffer_load_async_lds : AMDGPUBuiltin<"void(__amdgpu_buffer_rsrc_t, void address_space<3> *, _Constant unsigned int, int, int, _Constant int, _Constant int)", [], "vmem-to-lds-load-insts">;
def __builtin_amdgcn_struct_ptr_buffer_load_lds : AMDGPUBuiltin<"void(__amdgpu_buffer_rsrc_t, void address_space<3> *, _Constant unsigned int, int, int, int, _Constant int, _Constant int)", [], "vmem-to-lds-load-insts">;
def __builtin_amdgcn_struct_ptr_buffer_load_async_lds : AMDGPUBuiltin<"void(__amdgpu_buffer_rsrc_t, void address_space<3> *, _Constant unsigned int, int, int, int, _Constant int, _Constant int)", [], "vmem-to-lds-load-insts">;

//===----------------------------------------------------------------------===//
// Async mark builtins.
//===----------------------------------------------------------------------===//

// FIXME: Not supported on GFX12 yet. Will need a new feature when we do.
def __builtin_amdgcn_asyncmark : AMDGPUBuiltin<"void()", [], "vmem-to-lds-load-insts">;
````
- **L265 EN**: Declares TableGen def record `__builtin_amdgcn_struct_buffer_store_format_v4f32`.
  **L265 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_struct_buffer_store_format_v4f32`。
- **L266 EN**: Declares TableGen def record `__builtin_amdgcn_struct_buffer_store_format_v4f16`.
  **L266 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_struct_buffer_store_format_v4f16`。
- **L267 EN**: Blank line separating nearby declarations or logic blocks.
  **L267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L268 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_atomic_add_i32`.
  **L268 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_atomic_add_i32`。
- **L269 EN**: Blank line separating nearby declarations or logic blocks.
  **L269 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L270 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_f32`.
  **L270 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_f32`。
- **L271 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_v2f16`.
  **L271 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_atomic_fadd_v2f16`。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L273 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f32`.
  **L273 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f32`。
- **L274 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f32`.
  **L274 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f32`。
- **L275 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f64`.
  **L275 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_atomic_fmin_f64`。
- **L276 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f64`.
  **L276 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_atomic_fmax_f64`。
- **L277 EN**: Blank line separating nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L278 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_load_lds`.
  **L278 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_load_lds`。
- **L279 EN**: Declares TableGen def record `__builtin_amdgcn_raw_ptr_buffer_load_async_lds`.
  **L279 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_raw_ptr_buffer_load_async_lds`。
- **L280 EN**: Declares TableGen def record `__builtin_amdgcn_struct_ptr_buffer_load_lds`.
  **L280 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_struct_ptr_buffer_load_lds`。
- **L281 EN**: Declares TableGen def record `__builtin_amdgcn_struct_ptr_buffer_load_async_lds`.
  **L281 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_struct_ptr_buffer_load_async_lds`。
- **L282 EN**: Blank line separating nearby declarations or logic blocks.
  **L282 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L283 EN**: Banner comment marking a file or section boundary.
  **L283 CN**: 横幅注释，用于标记文件或章节边界。
- **L284 EN**: Comment explains nearby logic, constraints, or intent: `Async mark builtins.`.
  **L284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Async mark builtins.`。
- **L285 EN**: Banner comment marking a file or section boundary.
  **L285 CN**: 横幅注释，用于标记文件或章节边界。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L287 EN**: Comment records a pending task or caution: `FIXME: Not supported on GFX12 yet. Will need a new feature when we do.`.
  **L287 CN**: 注释记录待办事项或注意点：`FIXME: Not supported on GFX12 yet. Will need a new feature when we do.`。
- **L288 EN**: Declares TableGen def record `__builtin_amdgcn_asyncmark`.
  **L288 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_asyncmark`。

### Lines 289-312

````tablegen
def __builtin_amdgcn_wait_asyncmark : AMDGPUBuiltin<"void(_Constant unsigned short)", [], "vmem-to-lds-load-insts">;

//===----------------------------------------------------------------------===//
// Ballot builtins.
//===----------------------------------------------------------------------===//

def __builtin_amdgcn_ballot_w32 : AMDGPUBuiltin<"uint32_t(bool)", [Const], "wavefrontsize32">;
def __builtin_amdgcn_ballot_w64 : AMDGPUBuiltin<"uint64_t(bool)", [Const]>;

def __builtin_amdgcn_inverse_ballot_w32 : AMDGPUBuiltin<"bool(uint32_t)", [Const], "wavefrontsize32">;
def __builtin_amdgcn_inverse_ballot_w64 : AMDGPUBuiltin<"bool(uint64_t)", [Const], "wavefrontsize64">;

// Deprecated intrinsics in favor of __builtin_amdgn_ballot_{w32|w64}
def __builtin_amdgcn_uicmp : AMDGPUBuiltin<"uint64_t(unsigned int, unsigned int, _Constant int)", [Const]>;
def __builtin_amdgcn_uicmpl : AMDGPUBuiltin<"uint64_t(uint64_t, uint64_t, _Constant int)", [Const]>;
def __builtin_amdgcn_sicmp : AMDGPUBuiltin<"uint64_t(int, int, _Constant int)", [Const]>;
def __builtin_amdgcn_sicmpl : AMDGPUBuiltin<"uint64_t(int64_t, int64_t, _Constant int)", [Const]>;
def __builtin_amdgcn_fcmp : AMDGPUBuiltin<"uint64_t(double, double, _Constant int)", [Const]>;
def __builtin_amdgcn_fcmpf : AMDGPUBuiltin<"uint64_t(float, float, _Constant int)", [Const]>;

//===----------------------------------------------------------------------===//
// Flat addressing builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_is_shared : AMDGPUBuiltin<"bool(void const address_space<0> *)", [Const]>;
````
- **L289 EN**: Declares TableGen def record `__builtin_amdgcn_wait_asyncmark`.
  **L289 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wait_asyncmark`。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Banner comment marking a file or section boundary.
  **L291 CN**: 横幅注释，用于标记文件或章节边界。
- **L292 EN**: Comment explains nearby logic, constraints, or intent: `Ballot builtins.`.
  **L292 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Ballot builtins.`。
- **L293 EN**: Banner comment marking a file or section boundary.
  **L293 CN**: 横幅注释，用于标记文件或章节边界。
- **L294 EN**: Blank line separating nearby declarations or logic blocks.
  **L294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L295 EN**: Declares TableGen def record `__builtin_amdgcn_ballot_w32`.
  **L295 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ballot_w32`。
- **L296 EN**: Declares TableGen def record `__builtin_amdgcn_ballot_w64`.
  **L296 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ballot_w64`。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L298 EN**: Declares TableGen def record `__builtin_amdgcn_inverse_ballot_w32`.
  **L298 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_inverse_ballot_w32`。
- **L299 EN**: Declares TableGen def record `__builtin_amdgcn_inverse_ballot_w64`.
  **L299 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_inverse_ballot_w64`。
- **L300 EN**: Blank line separating nearby declarations or logic blocks.
  **L300 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L301 EN**: Comment explains nearby logic, constraints, or intent: `Deprecated intrinsics in favor of __builtin_amdgn_ballot_{w32|w64}`.
  **L301 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Deprecated intrinsics in favor of __builtin_amdgn_ballot_{w32|w64}`。
- **L302 EN**: Declares TableGen def record `__builtin_amdgcn_uicmp`.
  **L302 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_uicmp`。
- **L303 EN**: Declares TableGen def record `__builtin_amdgcn_uicmpl`.
  **L303 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_uicmpl`。
- **L304 EN**: Declares TableGen def record `__builtin_amdgcn_sicmp`.
  **L304 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sicmp`。
- **L305 EN**: Declares TableGen def record `__builtin_amdgcn_sicmpl`.
  **L305 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sicmpl`。
- **L306 EN**: Declares TableGen def record `__builtin_amdgcn_fcmp`.
  **L306 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fcmp`。
- **L307 EN**: Declares TableGen def record `__builtin_amdgcn_fcmpf`.
  **L307 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fcmpf`。
- **L308 EN**: Blank line separating nearby declarations or logic blocks.
  **L308 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L309 EN**: Banner comment marking a file or section boundary.
  **L309 CN**: 横幅注释，用于标记文件或章节边界。
- **L310 EN**: Comment explains nearby logic, constraints, or intent: `Flat addressing builtins.`.
  **L310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Flat addressing builtins.`。
- **L311 EN**: Banner comment marking a file or section boundary.
  **L311 CN**: 横幅注释，用于标记文件或章节边界。
- **L312 EN**: Declares TableGen def record `__builtin_amdgcn_is_shared`.
  **L312 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_is_shared`。

### Lines 313-336

````tablegen
def __builtin_amdgcn_is_private : AMDGPUBuiltin<"bool(void const address_space<0> *)", [Const]>;

//===----------------------------------------------------------------------===//
// GWS builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_ds_gws_init : AMDGPUBuiltin<"void(unsigned int, unsigned int)", [], "gws">;
def __builtin_amdgcn_ds_gws_barrier : AMDGPUBuiltin<"void(unsigned int, unsigned int)", [], "gws">;
def __builtin_amdgcn_ds_gws_sema_v : AMDGPUBuiltin<"void(unsigned int)", [], "gws">;
def __builtin_amdgcn_ds_gws_sema_br : AMDGPUBuiltin<"void(unsigned int, unsigned int)", [], "gws">;
def __builtin_amdgcn_ds_gws_sema_p : AMDGPUBuiltin<"void(unsigned int)", [], "gws">;

//===----------------------------------------------------------------------===//
// CI+ only builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_s_dcache_inv_vol : AMDGPUBuiltin<"void()", [], "ci-insts">;
def __builtin_amdgcn_buffer_wbinvl1_vol : AMDGPUBuiltin<"void()", [], "ci-insts">;
def __builtin_amdgcn_ds_gws_sema_release_all : AMDGPUBuiltin<"void(unsigned int)", [], "ci-insts">;

//===----------------------------------------------------------------------===//
// Interpolation builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_interp_p1_f16 : AMDGPUBuiltin<"float(float, unsigned int, unsigned int, bool, unsigned int)", [Const]>;
def __builtin_amdgcn_interp_p2_f16 : AMDGPUBuiltin<"__fp16(float, float, unsigned int, unsigned int, bool, unsigned int)", [Const]>;
def __builtin_amdgcn_interp_p1 : AMDGPUBuiltin<"float(float, unsigned int, unsigned int, unsigned int)", [Const]>;
````
- **L313 EN**: Declares TableGen def record `__builtin_amdgcn_is_private`.
  **L313 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_is_private`。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Banner comment marking a file or section boundary.
  **L315 CN**: 横幅注释，用于标记文件或章节边界。
- **L316 EN**: Comment explains nearby logic, constraints, or intent: `GWS builtins.`.
  **L316 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GWS builtins.`。
- **L317 EN**: Banner comment marking a file or section boundary.
  **L317 CN**: 横幅注释，用于标记文件或章节边界。
- **L318 EN**: Declares TableGen def record `__builtin_amdgcn_ds_gws_init`.
  **L318 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_gws_init`。
- **L319 EN**: Declares TableGen def record `__builtin_amdgcn_ds_gws_barrier`.
  **L319 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_gws_barrier`。
- **L320 EN**: Declares TableGen def record `__builtin_amdgcn_ds_gws_sema_v`.
  **L320 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_gws_sema_v`。
- **L321 EN**: Declares TableGen def record `__builtin_amdgcn_ds_gws_sema_br`.
  **L321 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_gws_sema_br`。
- **L322 EN**: Declares TableGen def record `__builtin_amdgcn_ds_gws_sema_p`.
  **L322 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_gws_sema_p`。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Banner comment marking a file or section boundary.
  **L324 CN**: 横幅注释，用于标记文件或章节边界。
- **L325 EN**: Comment explains nearby logic, constraints, or intent: `CI+ only builtins.`.
  **L325 CN**: 注释解释附近代码的逻辑、约束或设计意图：`CI+ only builtins.`。
- **L326 EN**: Banner comment marking a file or section boundary.
  **L326 CN**: 横幅注释，用于标记文件或章节边界。
- **L327 EN**: Declares TableGen def record `__builtin_amdgcn_s_dcache_inv_vol`.
  **L327 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_dcache_inv_vol`。
- **L328 EN**: Declares TableGen def record `__builtin_amdgcn_buffer_wbinvl1_vol`.
  **L328 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_buffer_wbinvl1_vol`。
- **L329 EN**: Declares TableGen def record `__builtin_amdgcn_ds_gws_sema_release_all`.
  **L329 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_gws_sema_release_all`。
- **L330 EN**: Blank line separating nearby declarations or logic blocks.
  **L330 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L331 EN**: Banner comment marking a file or section boundary.
  **L331 CN**: 横幅注释，用于标记文件或章节边界。
- **L332 EN**: Comment explains nearby logic, constraints, or intent: `Interpolation builtins.`.
  **L332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Interpolation builtins.`。
- **L333 EN**: Banner comment marking a file or section boundary.
  **L333 CN**: 横幅注释，用于标记文件或章节边界。
- **L334 EN**: Declares TableGen def record `__builtin_amdgcn_interp_p1_f16`.
  **L334 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_interp_p1_f16`。
- **L335 EN**: Declares TableGen def record `__builtin_amdgcn_interp_p2_f16`.
  **L335 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_interp_p2_f16`。
- **L336 EN**: Declares TableGen def record `__builtin_amdgcn_interp_p1`.
  **L336 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_interp_p1`。

### Lines 337-360

````tablegen
def __builtin_amdgcn_interp_p2 : AMDGPUBuiltin<"float(float, float, unsigned int, unsigned int, unsigned int)", [Const]>;
def __builtin_amdgcn_interp_mov : AMDGPUBuiltin<"float(unsigned int, unsigned int, unsigned int, unsigned int)", [Const]>;

//===----------------------------------------------------------------------===//
// VI+ only builtins.
//===----------------------------------------------------------------------===//

def __builtin_amdgcn_div_fixuph : AMDGPUBuiltin<"_Float16(_Float16, _Float16, _Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_rcph : AMDGPUBuiltin<"_Float16(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_sqrth : AMDGPUBuiltin<"_Float16(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_rsqh : AMDGPUBuiltin<"_Float16(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_sinh : AMDGPUBuiltin<"_Float16(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_cosh : AMDGPUBuiltin<"_Float16(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_ldexph : AMDGPUBuiltin<"_Float16(_Float16, int)", [Const], "16-bit-insts">;
def __builtin_amdgcn_frexp_manth : AMDGPUBuiltin<"_Float16(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_frexp_exph : AMDGPUBuiltin<"short(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_fracth : AMDGPUBuiltin<"_Float16(_Float16)", [Const], "16-bit-insts">;
def __builtin_amdgcn_classh : AMDGPUBuiltin<"bool(_Float16, int)", [Const], "16-bit-insts">;
def __builtin_amdgcn_s_memrealtime : AMDGPUBuiltin<"uint64_t()", [], "s-memrealtime">;
def __builtin_amdgcn_mov_dpp : AMDGPUBuiltin<"int(int, _Constant int, _Constant int, _Constant int, _Constant bool)", [Const, CustomTypeChecking], "dpp">;
def __builtin_amdgcn_update_dpp : AMDGPUBuiltin<"int(int, int, _Constant int, _Constant int, _Constant int, _Constant bool)", [Const, CustomTypeChecking], "dpp">;
def __builtin_amdgcn_s_dcache_wb : AMDGPUBuiltin<"void()", [], "gfx8-insts">;
def __builtin_amdgcn_perm : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)", [Const], "gfx8-insts">;

````
- **L337 EN**: Declares TableGen def record `__builtin_amdgcn_interp_p2`.
  **L337 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_interp_p2`。
- **L338 EN**: Declares TableGen def record `__builtin_amdgcn_interp_mov`.
  **L338 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_interp_mov`。
- **L339 EN**: Blank line separating nearby declarations or logic blocks.
  **L339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L340 EN**: Banner comment marking a file or section boundary.
  **L340 CN**: 横幅注释，用于标记文件或章节边界。
- **L341 EN**: Comment explains nearby logic, constraints, or intent: `VI+ only builtins.`.
  **L341 CN**: 注释解释附近代码的逻辑、约束或设计意图：`VI+ only builtins.`。
- **L342 EN**: Banner comment marking a file or section boundary.
  **L342 CN**: 横幅注释，用于标记文件或章节边界。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L344 EN**: Declares TableGen def record `__builtin_amdgcn_div_fixuph`.
  **L344 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_div_fixuph`。
- **L345 EN**: Declares TableGen def record `__builtin_amdgcn_rcph`.
  **L345 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rcph`。
- **L346 EN**: Declares TableGen def record `__builtin_amdgcn_sqrth`.
  **L346 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sqrth`。
- **L347 EN**: Declares TableGen def record `__builtin_amdgcn_rsqh`.
  **L347 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rsqh`。
- **L348 EN**: Declares TableGen def record `__builtin_amdgcn_sinh`.
  **L348 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sinh`。
- **L349 EN**: Declares TableGen def record `__builtin_amdgcn_cosh`.
  **L349 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cosh`。
- **L350 EN**: Declares TableGen def record `__builtin_amdgcn_ldexph`.
  **L350 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ldexph`。
- **L351 EN**: Declares TableGen def record `__builtin_amdgcn_frexp_manth`.
  **L351 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_frexp_manth`。
- **L352 EN**: Declares TableGen def record `__builtin_amdgcn_frexp_exph`.
  **L352 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_frexp_exph`。
- **L353 EN**: Declares TableGen def record `__builtin_amdgcn_fracth`.
  **L353 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fracth`。
- **L354 EN**: Declares TableGen def record `__builtin_amdgcn_classh`.
  **L354 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_classh`。
- **L355 EN**: Declares TableGen def record `__builtin_amdgcn_s_memrealtime`.
  **L355 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_memrealtime`。
- **L356 EN**: Declares TableGen def record `__builtin_amdgcn_mov_dpp`.
  **L356 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mov_dpp`。
- **L357 EN**: Declares TableGen def record `__builtin_amdgcn_update_dpp`.
  **L357 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_update_dpp`。
- **L358 EN**: Declares TableGen def record `__builtin_amdgcn_s_dcache_wb`.
  **L358 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_dcache_wb`。
- **L359 EN**: Declares TableGen def record `__builtin_amdgcn_perm`.
  **L359 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_perm`。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 361-384

````tablegen
//===----------------------------------------------------------------------===//
// GFX9+ only builtins.
//===----------------------------------------------------------------------===//

def __builtin_amdgcn_fmed3h : AMDGPUBuiltin<"__fp16(__fp16, __fp16, __fp16)", [Const], "gfx9-insts">;

def __builtin_amdgcn_global_atomic_fadd_f64 : AMDGPUBuiltin<"double(double address_space<1> *, double)", [], "gfx90a-insts">;
def __builtin_amdgcn_global_atomic_fadd_f32 : AMDGPUBuiltin<"float(float address_space<1> *, float)", [], "atomic-fadd-rtn-insts">;
def __builtin_amdgcn_global_atomic_fadd_v2f16 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(_ExtVector<2, _Float16> address_space<1> *, _ExtVector<2, _Float16>)", [CustomTypeChecking], "atomic-buffer-global-pk-add-f16-insts">;
def __builtin_amdgcn_global_atomic_fmin_f64 : AMDGPUBuiltin<"double(double address_space<1> *, double)", [], "gfx90a-insts">;
def __builtin_amdgcn_global_atomic_fmax_f64 : AMDGPUBuiltin<"double(double address_space<1> *, double)", [], "gfx90a-insts">;

def __builtin_amdgcn_flat_atomic_fadd_f64 : AMDGPUBuiltin<"double(double address_space<0> *, double)", [], "gfx90a-insts">;
def __builtin_amdgcn_flat_atomic_fmin_f64 : AMDGPUBuiltin<"double(double address_space<0> *, double)", [], "gfx90a-insts">;
def __builtin_amdgcn_flat_atomic_fmax_f64 : AMDGPUBuiltin<"double(double address_space<0> *, double)", [], "gfx90a-insts">;

def __builtin_amdgcn_ds_atomic_fadd_f64 : AMDGPUBuiltin<"double(double address_space<3> *, double)", [], "gfx90a-insts">;
def __builtin_amdgcn_ds_atomic_fadd_f32 : AMDGPUBuiltin<"float(float address_space<3> *, float)", [], "gfx8-insts">;

def __builtin_amdgcn_flat_atomic_fadd_f32 : AMDGPUBuiltin<"float(float address_space<0> *, float)", [], "gfx940-insts">;
def __builtin_amdgcn_flat_atomic_fadd_v2f16 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(_ExtVector<2, _Float16> address_space<0> *, _ExtVector<2, _Float16>)", [CustomTypeChecking], "atomic-flat-pk-add-16-insts">;
def __builtin_amdgcn_flat_atomic_fadd_v2bf16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short> address_space<0> *, _ExtVector<2, short>)", [CustomTypeChecking], "atomic-flat-pk-add-16-insts">;
def __builtin_amdgcn_global_atomic_fadd_v2bf16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short> address_space<1> *, _ExtVector<2, short>)", [CustomTypeChecking], "atomic-global-pk-add-bf16-inst">;
def __builtin_amdgcn_ds_atomic_fadd_v2bf16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short> address_space<3> *, _ExtVector<2, short>)", [CustomTypeChecking], "atomic-ds-pk-add-16-insts">;
````
- **L361 EN**: Banner comment marking a file or section boundary.
  **L361 CN**: 横幅注释，用于标记文件或章节边界。
- **L362 EN**: Comment explains nearby logic, constraints, or intent: `GFX9+ only builtins.`.
  **L362 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX9+ only builtins.`。
- **L363 EN**: Banner comment marking a file or section boundary.
  **L363 CN**: 横幅注释，用于标记文件或章节边界。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L365 EN**: Declares TableGen def record `__builtin_amdgcn_fmed3h`.
  **L365 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fmed3h`。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L367 EN**: Declares TableGen def record `__builtin_amdgcn_global_atomic_fadd_f64`.
  **L367 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_atomic_fadd_f64`。
- **L368 EN**: Declares TableGen def record `__builtin_amdgcn_global_atomic_fadd_f32`.
  **L368 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_atomic_fadd_f32`。
- **L369 EN**: Declares TableGen def record `__builtin_amdgcn_global_atomic_fadd_v2f16`.
  **L369 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_atomic_fadd_v2f16`。
- **L370 EN**: Declares TableGen def record `__builtin_amdgcn_global_atomic_fmin_f64`.
  **L370 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_atomic_fmin_f64`。
- **L371 EN**: Declares TableGen def record `__builtin_amdgcn_global_atomic_fmax_f64`.
  **L371 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_atomic_fmax_f64`。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L373 EN**: Declares TableGen def record `__builtin_amdgcn_flat_atomic_fadd_f64`.
  **L373 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_atomic_fadd_f64`。
- **L374 EN**: Declares TableGen def record `__builtin_amdgcn_flat_atomic_fmin_f64`.
  **L374 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_atomic_fmin_f64`。
- **L375 EN**: Declares TableGen def record `__builtin_amdgcn_flat_atomic_fmax_f64`.
  **L375 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_atomic_fmax_f64`。
- **L376 EN**: Blank line separating nearby declarations or logic blocks.
  **L376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L377 EN**: Declares TableGen def record `__builtin_amdgcn_ds_atomic_fadd_f64`.
  **L377 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_atomic_fadd_f64`。
- **L378 EN**: Declares TableGen def record `__builtin_amdgcn_ds_atomic_fadd_f32`.
  **L378 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_atomic_fadd_f32`。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L380 EN**: Declares TableGen def record `__builtin_amdgcn_flat_atomic_fadd_f32`.
  **L380 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_atomic_fadd_f32`。
- **L381 EN**: Declares TableGen def record `__builtin_amdgcn_flat_atomic_fadd_v2f16`.
  **L381 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_atomic_fadd_v2f16`。
- **L382 EN**: Declares TableGen def record `__builtin_amdgcn_flat_atomic_fadd_v2bf16`.
  **L382 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_atomic_fadd_v2bf16`。
- **L383 EN**: Declares TableGen def record `__builtin_amdgcn_global_atomic_fadd_v2bf16`.
  **L383 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_atomic_fadd_v2bf16`。
- **L384 EN**: Declares TableGen def record `__builtin_amdgcn_ds_atomic_fadd_v2bf16`.
  **L384 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_atomic_fadd_v2bf16`。

### Lines 385-408

````tablegen
def __builtin_amdgcn_ds_atomic_fadd_v2f16 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(_ExtVector<2, _Float16> address_space<3> *, _ExtVector<2, _Float16>)", [CustomTypeChecking], "atomic-ds-pk-add-16-insts">;
def __builtin_amdgcn_load_to_lds : AMDGPUBuiltin<"void(void *, void address_space<3> *, _Constant unsigned int, _Constant int, _Constant unsigned int)", [], "vmem-to-lds-load-insts">;
def __builtin_amdgcn_load_async_to_lds : AMDGPUBuiltin<"void(void *, void address_space<3> *, _Constant unsigned int, _Constant int, _Constant unsigned int)", [], "vmem-to-lds-load-insts">;
def __builtin_amdgcn_global_load_lds : AMDGPUBuiltin<"void(void address_space<1> *, void address_space<3> *, _Constant unsigned int, _Constant int, _Constant unsigned int)", [], "vmem-to-lds-load-insts">;
def __builtin_amdgcn_global_load_async_lds : AMDGPUBuiltin<"void(void address_space<1> *, void address_space<3> *, _Constant unsigned int, _Constant int, _Constant unsigned int)", [], "vmem-to-lds-load-insts">;

//===----------------------------------------------------------------------===//
// Deep learning builtins.
//===----------------------------------------------------------------------===//

def __builtin_amdgcn_fdot2 : AMDGPUBuiltin<"float(_ExtVector<2, _Float16>, _ExtVector<2, _Float16>, float, _Constant bool)", [Const], "dot10-insts">;
def __builtin_amdgcn_fdot2_f16_f16 : AMDGPUBuiltin<"_Float16(_ExtVector<2, _Float16>, _ExtVector<2, _Float16>, _Float16)", [Const], "dot9-insts">;
def __builtin_amdgcn_fdot2_bf16_bf16 : AMDGPUBuiltin<"short(_ExtVector<2, short>, _ExtVector<2, short>, short)", [Const], "dot9-insts">;
def __builtin_amdgcn_fdot2_f32_bf16 : AMDGPUBuiltin<"float(_ExtVector<2, short>, _ExtVector<2, short>, float, _Constant bool)", [Const], "dot12-insts">;
def __builtin_amdgcn_sdot2 : AMDGPUBuiltin<"int(_ExtVector<2, short>, _ExtVector<2, short>, int, _Constant bool)", [Const], "dot2-insts">;
def __builtin_amdgcn_udot2 : AMDGPUBuiltin<"unsigned int(_ExtVector<2, unsigned short>, _ExtVector<2, unsigned short>, unsigned int, _Constant bool)", [Const], "dot2-insts">;
def __builtin_amdgcn_sdot4 : AMDGPUBuiltin<"int(int, int, int, _Constant bool)", [Const], "dot1-insts">;
def __builtin_amdgcn_udot4 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, _Constant bool)", [Const], "dot7-insts">;
def __builtin_amdgcn_sudot4 : AMDGPUBuiltin<"int(_Constant bool, int, _Constant bool, int, int, _Constant bool)", [Const], "dot8-insts">;
def __builtin_amdgcn_sdot8 : AMDGPUBuiltin<"int(int, int, int, _Constant bool)", [Const], "dot1-insts">;
def __builtin_amdgcn_udot8 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, _Constant bool)", [Const], "dot7-insts">;
def __builtin_amdgcn_sudot8 : AMDGPUBuiltin<"int(_Constant bool, int, _Constant bool, int, int, _Constant bool)", [Const], "dot8-insts">;
def __builtin_amdgcn_dot4_f32_fp8_bf8 : AMDGPUBuiltin<"float(unsigned int, unsigned int, float)", [Const], "dot11-insts">;
def __builtin_amdgcn_dot4_f32_bf8_fp8 : AMDGPUBuiltin<"float(unsigned int, unsigned int, float)", [Const], "dot11-insts">;
````
- **L385 EN**: Declares TableGen def record `__builtin_amdgcn_ds_atomic_fadd_v2f16`.
  **L385 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_atomic_fadd_v2f16`。
- **L386 EN**: Declares TableGen def record `__builtin_amdgcn_load_to_lds`.
  **L386 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_load_to_lds`。
- **L387 EN**: Declares TableGen def record `__builtin_amdgcn_load_async_to_lds`.
  **L387 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_load_async_to_lds`。
- **L388 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_lds`.
  **L388 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_lds`。
- **L389 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_async_lds`.
  **L389 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_async_lds`。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Banner comment marking a file or section boundary.
  **L391 CN**: 横幅注释，用于标记文件或章节边界。
- **L392 EN**: Comment explains nearby logic, constraints, or intent: `Deep learning builtins.`.
  **L392 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Deep learning builtins.`。
- **L393 EN**: Banner comment marking a file or section boundary.
  **L393 CN**: 横幅注释，用于标记文件或章节边界。
- **L394 EN**: Blank line separating nearby declarations or logic blocks.
  **L394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L395 EN**: Declares TableGen def record `__builtin_amdgcn_fdot2`.
  **L395 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fdot2`。
- **L396 EN**: Declares TableGen def record `__builtin_amdgcn_fdot2_f16_f16`.
  **L396 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fdot2_f16_f16`。
- **L397 EN**: Declares TableGen def record `__builtin_amdgcn_fdot2_bf16_bf16`.
  **L397 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fdot2_bf16_bf16`。
- **L398 EN**: Declares TableGen def record `__builtin_amdgcn_fdot2_f32_bf16`.
  **L398 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fdot2_f32_bf16`。
- **L399 EN**: Declares TableGen def record `__builtin_amdgcn_sdot2`.
  **L399 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sdot2`。
- **L400 EN**: Declares TableGen def record `__builtin_amdgcn_udot2`.
  **L400 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_udot2`。
- **L401 EN**: Declares TableGen def record `__builtin_amdgcn_sdot4`.
  **L401 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sdot4`。
- **L402 EN**: Declares TableGen def record `__builtin_amdgcn_udot4`.
  **L402 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_udot4`。
- **L403 EN**: Declares TableGen def record `__builtin_amdgcn_sudot4`.
  **L403 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sudot4`。
- **L404 EN**: Declares TableGen def record `__builtin_amdgcn_sdot8`.
  **L404 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sdot8`。
- **L405 EN**: Declares TableGen def record `__builtin_amdgcn_udot8`.
  **L405 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_udot8`。
- **L406 EN**: Declares TableGen def record `__builtin_amdgcn_sudot8`.
  **L406 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sudot8`。
- **L407 EN**: Declares TableGen def record `__builtin_amdgcn_dot4_f32_fp8_bf8`.
  **L407 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_dot4_f32_fp8_bf8`。
- **L408 EN**: Declares TableGen def record `__builtin_amdgcn_dot4_f32_bf8_fp8`.
  **L408 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_dot4_f32_bf8_fp8`。

### Lines 409-432

````tablegen
def __builtin_amdgcn_dot4_f32_fp8_fp8 : AMDGPUBuiltin<"float(unsigned int, unsigned int, float)", [Const], "dot11-insts">;
def __builtin_amdgcn_dot4_f32_bf8_bf8 : AMDGPUBuiltin<"float(unsigned int, unsigned int, float)", [Const], "dot11-insts">;
def __builtin_amdgcn_fdot2c_f32_bf16 : AMDGPUBuiltin<"float(_ExtVector<2, __bf16>, _ExtVector<2, __bf16>, float, _Constant bool)", [Const], "dot13-insts">;

//===----------------------------------------------------------------------===//
// GFX10+ only builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_permlane16 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, unsigned int, _Constant bool, _Constant bool)", [Const], "gfx10-insts">;
def __builtin_amdgcn_permlanex16 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, unsigned int, _Constant bool, _Constant bool)", [Const], "gfx10-insts">;
def __builtin_amdgcn_mov_dpp8 : AMDGPUBuiltin<"unsigned int(unsigned int, _Constant unsigned int)", [Const, CustomTypeChecking], "gfx10-insts">;
def __builtin_amdgcn_s_ttracedata_imm : AMDGPUBuiltin<"void(_Constant short)", [], "gfx10-insts">;

//===----------------------------------------------------------------------===//
// Raytracing builtins.
// By default the 1st argument is i32 and the 4/5-th arguments are float4.
// Postfix l indicates the 1st argument is i64.
// Postfix h indicates the 4/5-th arguments are half4.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_image_bvh_intersect_ray : AMDGPUBuiltin<"_ExtVector<4, unsigned int>(unsigned int, float, _ExtVector<4, float>, _ExtVector<4, float>, _ExtVector<4, float>, _ExtVector<4, unsigned int>)", [Const], "gfx10-insts">;
def __builtin_amdgcn_image_bvh_intersect_ray_h : AMDGPUBuiltin<"_ExtVector<4, unsigned int>(unsigned int, float, _ExtVector<4, float>, _ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<4, unsigned int>)", [Const], "gfx10-insts">;
def __builtin_amdgcn_image_bvh_intersect_ray_l : AMDGPUBuiltin<"_ExtVector<4, unsigned int>(uint64_t, float, _ExtVector<4, float>, _ExtVector<4, float>, _ExtVector<4, float>, _ExtVector<4, unsigned int>)", [Const], "gfx10-insts">;
def __builtin_amdgcn_image_bvh_intersect_ray_lh : AMDGPUBuiltin<"_ExtVector<4, unsigned int>(uint64_t, float, _ExtVector<4, float>, _ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<4, unsigned int>)", [Const], "gfx10-insts">;


````
- **L409 EN**: Declares TableGen def record `__builtin_amdgcn_dot4_f32_fp8_fp8`.
  **L409 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_dot4_f32_fp8_fp8`。
- **L410 EN**: Declares TableGen def record `__builtin_amdgcn_dot4_f32_bf8_bf8`.
  **L410 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_dot4_f32_bf8_bf8`。
- **L411 EN**: Declares TableGen def record `__builtin_amdgcn_fdot2c_f32_bf16`.
  **L411 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_fdot2c_f32_bf16`。
- **L412 EN**: Blank line separating nearby declarations or logic blocks.
  **L412 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L413 EN**: Banner comment marking a file or section boundary.
  **L413 CN**: 横幅注释，用于标记文件或章节边界。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `GFX10+ only builtins.`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX10+ only builtins.`。
- **L415 EN**: Banner comment marking a file or section boundary.
  **L415 CN**: 横幅注释，用于标记文件或章节边界。
- **L416 EN**: Declares TableGen def record `__builtin_amdgcn_permlane16`.
  **L416 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane16`。
- **L417 EN**: Declares TableGen def record `__builtin_amdgcn_permlanex16`.
  **L417 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlanex16`。
- **L418 EN**: Declares TableGen def record `__builtin_amdgcn_mov_dpp8`.
  **L418 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mov_dpp8`。
- **L419 EN**: Declares TableGen def record `__builtin_amdgcn_s_ttracedata_imm`.
  **L419 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_ttracedata_imm`。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L421 EN**: Banner comment marking a file or section boundary.
  **L421 CN**: 横幅注释，用于标记文件或章节边界。
- **L422 EN**: Comment explains nearby logic, constraints, or intent: `Raytracing builtins.`.
  **L422 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Raytracing builtins.`。
- **L423 EN**: Comment explains nearby logic, constraints, or intent: `By default the 1st argument is i32 and the 4/5-th arguments are float4.`.
  **L423 CN**: 注释解释附近代码的逻辑、约束或设计意图：`By default the 1st argument is i32 and the 4/5-th arguments are float4.`。
- **L424 EN**: Comment explains nearby logic, constraints, or intent: `Postfix l indicates the 1st argument is i64.`.
  **L424 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Postfix l indicates the 1st argument is i64.`。
- **L425 EN**: Comment explains nearby logic, constraints, or intent: `Postfix h indicates the 4/5-th arguments are half4.`.
  **L425 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Postfix h indicates the 4/5-th arguments are half4.`。
- **L426 EN**: Banner comment marking a file or section boundary.
  **L426 CN**: 横幅注释，用于标记文件或章节边界。
- **L427 EN**: Declares TableGen def record `__builtin_amdgcn_image_bvh_intersect_ray`.
  **L427 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_bvh_intersect_ray`。
- **L428 EN**: Declares TableGen def record `__builtin_amdgcn_image_bvh_intersect_ray_h`.
  **L428 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_bvh_intersect_ray_h`。
- **L429 EN**: Declares TableGen def record `__builtin_amdgcn_image_bvh_intersect_ray_l`.
  **L429 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_bvh_intersect_ray_l`。
- **L430 EN**: Declares TableGen def record `__builtin_amdgcn_image_bvh_intersect_ray_lh`.
  **L430 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_bvh_intersect_ray_lh`。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````tablegen
//===----------------------------------------------------------------------===//
// GFX11+ only builtins.
//===----------------------------------------------------------------------===//

// TODO: This is a no-op in wave32. Should the builtin require wavefrontsize64?
def __builtin_amdgcn_permlane64 : AMDGPUBuiltin<"unsigned int(unsigned int)", [Const], "gfx11-insts">;
def __builtin_amdgcn_s_wait_event_export_ready : AMDGPUBuiltin<"void()", [], "gfx11-insts">;
def __builtin_amdgcn_s_wait_event : AMDGPUBuiltin<"void(_Constant short)", [], "gfx11-insts">;

//===----------------------------------------------------------------------===//
// WMMA builtins.
// Postfix w32 indicates the builtin requires wavefront size of 32.
// Postfix w64 indicates the builtin requires wavefront size of 64.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_wmma_f32_16x16x16_f16_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _ExtVector<8, float>)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAF32_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_bf16_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<16, short>, _ExtVector<16, short>, _ExtVector<8, float>)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAF32_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f16_16x16x16_f16_w32 : AMDGPUBuiltin<"_ExtVector<16, _Float16>(_ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAOpsel_16x16x16_GFX11];
````
- **L433 EN**: Banner comment marking a file or section boundary.
  **L433 CN**: 横幅注释，用于标记文件或章节边界。
- **L434 EN**: Comment explains nearby logic, constraints, or intent: `GFX11+ only builtins.`.
  **L434 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX11+ only builtins.`。
- **L435 EN**: Banner comment marking a file or section boundary.
  **L435 CN**: 横幅注释，用于标记文件或章节边界。
- **L436 EN**: Blank line separating nearby declarations or logic blocks.
  **L436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L437 EN**: Comment records a pending task or caution: `TODO: This is a no-op in wave32. Should the builtin require wavefrontsize64?`.
  **L437 CN**: 注释记录待办事项或注意点：`TODO: This is a no-op in wave32. Should the builtin require wavefrontsize64?`。
- **L438 EN**: Declares TableGen def record `__builtin_amdgcn_permlane64`.
  **L438 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane64`。
- **L439 EN**: Declares TableGen def record `__builtin_amdgcn_s_wait_event_export_ready`.
  **L439 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_wait_event_export_ready`。
- **L440 EN**: Declares TableGen def record `__builtin_amdgcn_s_wait_event`.
  **L440 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_wait_event`。
- **L441 EN**: Blank line separating nearby declarations or logic blocks.
  **L441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L442 EN**: Banner comment marking a file or section boundary.
  **L442 CN**: 横幅注释，用于标记文件或章节边界。
- **L443 EN**: Comment explains nearby logic, constraints, or intent: `WMMA builtins.`.
  **L443 CN**: 注释解释附近代码的逻辑、约束或设计意图：`WMMA builtins.`。
- **L444 EN**: Comment explains nearby logic, constraints, or intent: `Postfix w32 indicates the builtin requires wavefront size of 32.`.
  **L444 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Postfix w32 indicates the builtin requires wavefront size of 32.`。
- **L445 EN**: Comment explains nearby logic, constraints, or intent: `Postfix w64 indicates the builtin requires wavefront size of 64.`.
  **L445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Postfix w64 indicates the builtin requires wavefront size of 64.`。
- **L446 EN**: Banner comment marking a file or section boundary.
  **L446 CN**: 横幅注释，用于标记文件或章节边界。
- **L447 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_f16_w32`.
  **L447 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_f16_w32`。
- **L448 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX11];`.
  **L448 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX11];`。
- **L449 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L449 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L450 EN**: Closes the current lexical scope or compound statement.
  **L450 CN**: 结束当前词法作用域或复合语句块。
- **L451 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32`.
  **L451 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32`。
- **L452 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX11];`.
  **L452 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX11];`。
- **L453 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L453 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L454 EN**: Closes the current lexical scope or compound statement.
  **L454 CN**: 结束当前词法作用域或复合语句块。
- **L455 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x16_f16_w32`.
  **L455 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x16_f16_w32`。
- **L456 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`.
  **L456 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`。

### Lines 457-480

````tablegen
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32 : AMDGPUBuiltin<"_ExtVector<16, short>(_ExtVector<16, short>, _ExtVector<16, short>, _ExtVector<16, short>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAOpsel_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w32 : AMDGPUBuiltin<"_ExtVector<16, _Float16>(_ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w32 : AMDGPUBuiltin<"_ExtVector<16, short>(_ExtVector<16, short>, _ExtVector<16, short>, _ExtVector<16, short>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu8_w32 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<4, int>, _Constant bool, _ExtVector<4, int>, _ExtVector<8, int>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAIU_16x16x16_GFX11];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu4_w32 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<2, int>, _Constant bool, _ExtVector<2, int>, _ExtVector<8, int>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAIU_16x16x16_GFX11];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}

def __builtin_amdgcn_wmma_f32_16x16x16_f16_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _ExtVector<4, float>)", [Const], "wmma-256b-insts,wavefrontsize64"> {
````
- **L457 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L457 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L458 EN**: Closes the current lexical scope or compound statement.
  **L458 CN**: 结束当前词法作用域或复合语句块。
- **L459 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32`.
  **L459 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32`。
- **L460 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`.
  **L460 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`。
- **L461 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L461 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L462 EN**: Closes the current lexical scope or compound statement.
  **L462 CN**: 结束当前词法作用域或复合语句块。
- **L463 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w32`.
  **L463 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w32`。
- **L464 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`.
  **L464 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`。
- **L465 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L465 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L466 EN**: Closes the current lexical scope or compound statement.
  **L466 CN**: 结束当前词法作用域或复合语句块。
- **L467 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w32`.
  **L467 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w32`。
- **L468 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`.
  **L468 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`。
- **L469 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L469 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L470 EN**: Closes the current lexical scope or compound statement.
  **L470 CN**: 结束当前词法作用域或复合语句块。
- **L471 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32`.
  **L471 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32`。
- **L472 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX11];`.
  **L472 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX11];`。
- **L473 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L473 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L474 EN**: Closes the current lexical scope or compound statement.
  **L474 CN**: 结束当前词法作用域或复合语句块。
- **L475 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32`.
  **L475 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32`。
- **L476 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX11];`.
  **L476 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX11];`。
- **L477 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L477 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L478 EN**: Closes the current lexical scope or compound statement.
  **L478 CN**: 结束当前词法作用域或复合语句块。
- **L479 EN**: Blank line separating nearby declarations or logic blocks.
  **L479 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L480 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_f16_w64`.
  **L480 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_f16_w64`。

### Lines 481-504

````tablegen
  let Documentation = [DocWMMAF32_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_bf16_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<16, short>, _ExtVector<16, short>, _ExtVector<4, float>)", [Const], "wmma-256b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAF32_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f16_16x16x16_f16_w64 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _ExtVector<8, _Float16>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAOpsel_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64 : AMDGPUBuiltin<"_ExtVector<8, short>(_ExtVector<16, short>, _ExtVector<16, short>, _ExtVector<8, short>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAOpsel_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w64 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<16, _Float16>, _ExtVector<16, _Float16>, _ExtVector<8, _Float16>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w64 : AMDGPUBuiltin<"_ExtVector<8, short>(_ExtVector<16, short>, _ExtVector<16, short>, _ExtVector<8, short>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];
  let ArgNames = ["a", "b", "c", "opsel"];
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu8_w64 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, _ExtVector<4, int>, _Constant bool, _ExtVector<4, int>, _ExtVector<4, int>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize64"> {
````
- **L481 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX11];`.
  **L481 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX11];`。
- **L482 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L482 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L483 EN**: Closes the current lexical scope or compound statement.
  **L483 CN**: 结束当前词法作用域或复合语句块。
- **L484 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64`.
  **L484 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64`。
- **L485 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX11];`.
  **L485 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX11];`。
- **L486 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L486 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L487 EN**: Closes the current lexical scope or compound statement.
  **L487 CN**: 结束当前词法作用域或复合语句块。
- **L488 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x16_f16_w64`.
  **L488 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x16_f16_w64`。
- **L489 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`.
  **L489 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`。
- **L490 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L490 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L491 EN**: Closes the current lexical scope or compound statement.
  **L491 CN**: 结束当前词法作用域或复合语句块。
- **L492 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64`.
  **L492 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64`。
- **L493 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`.
  **L493 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpsel_16x16x16_GFX11];`。
- **L494 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L494 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L495 EN**: Closes the current lexical scope or compound statement.
  **L495 CN**: 结束当前词法作用域或复合语句块。
- **L496 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w64`.
  **L496 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x16_f16_tied_w64`。
- **L497 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`.
  **L497 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`。
- **L498 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L498 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L499 EN**: Closes the current lexical scope or compound statement.
  **L499 CN**: 结束当前词法作用域或复合语句块。
- **L500 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w64`.
  **L500 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_tied_w64`。
- **L501 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`.
  **L501 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAOpselTied_16x16x16_GFX11];`。
- **L502 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c", "opsel"];`.
  **L502 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c", "opsel"];`。
- **L503 EN**: Closes the current lexical scope or compound statement.
  **L503 CN**: 结束当前词法作用域或复合语句块。
- **L504 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64`.
  **L504 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64`。

### Lines 505-528

````tablegen
  let Documentation = [DocWMMAIU_16x16x16_GFX11];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu4_w64 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, _ExtVector<2, int>, _Constant bool, _ExtVector<2, int>, _ExtVector<4, int>, _Constant bool)", [Const], "wmma-256b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAIU_16x16x16_GFX11];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}

def __builtin_amdgcn_s_sendmsg_rtn : AMDGPUBuiltin<"unsigned int(_Constant unsigned int)", [], "gfx11-insts">;
def __builtin_amdgcn_s_sendmsg_rtnl : AMDGPUBuiltin<"uint64_t(_Constant unsigned int)", [], "gfx11-insts">;

def __builtin_amdgcn_ds_bvh_stack_rtn : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(unsigned int, unsigned int, _ExtVector<4, unsigned int>, _Constant int)", [], "gfx11-insts">;

//===----------------------------------------------------------------------===//
// Special builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_read_exec : AMDGPUBuiltin<"uint64_t()", [Const]>;
def __builtin_amdgcn_read_exec_lo : AMDGPUBuiltin<"unsigned int()", [Const]>;
def __builtin_amdgcn_read_exec_hi : AMDGPUBuiltin<"unsigned int()", [Const]>;

def __builtin_amdgcn_endpgm : AMDGPUBuiltin<"void()", [NoReturn]>;

def __builtin_amdgcn_get_fpenv : AMDGPUBuiltin<"uint64_t()">;
def __builtin_amdgcn_set_fpenv : AMDGPUBuiltin<"void(uint64_t)">;
````
- **L505 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX11];`.
  **L505 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX11];`。
- **L506 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L506 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L507 EN**: Closes the current lexical scope or compound statement.
  **L507 CN**: 结束当前词法作用域或复合语句块。
- **L508 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64`.
  **L508 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64`。
- **L509 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX11];`.
  **L509 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX11];`。
- **L510 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L510 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L511 EN**: Closes the current lexical scope or compound statement.
  **L511 CN**: 结束当前词法作用域或复合语句块。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L513 EN**: Declares TableGen def record `__builtin_amdgcn_s_sendmsg_rtn`.
  **L513 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_sendmsg_rtn`。
- **L514 EN**: Declares TableGen def record `__builtin_amdgcn_s_sendmsg_rtnl`.
  **L514 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_sendmsg_rtnl`。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L516 EN**: Declares TableGen def record `__builtin_amdgcn_ds_bvh_stack_rtn`.
  **L516 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_bvh_stack_rtn`。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L518 EN**: Banner comment marking a file or section boundary.
  **L518 CN**: 横幅注释，用于标记文件或章节边界。
- **L519 EN**: Comment explains nearby logic, constraints, or intent: `Special builtins.`.
  **L519 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Special builtins.`。
- **L520 EN**: Banner comment marking a file or section boundary.
  **L520 CN**: 横幅注释，用于标记文件或章节边界。
- **L521 EN**: Declares TableGen def record `__builtin_amdgcn_read_exec`.
  **L521 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_read_exec`。
- **L522 EN**: Declares TableGen def record `__builtin_amdgcn_read_exec_lo`.
  **L522 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_read_exec_lo`。
- **L523 EN**: Declares TableGen def record `__builtin_amdgcn_read_exec_hi`.
  **L523 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_read_exec_hi`。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Declares TableGen def record `__builtin_amdgcn_endpgm`.
  **L525 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_endpgm`。
- **L526 EN**: Blank line separating nearby declarations or logic blocks.
  **L526 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L527 EN**: Declares TableGen def record `__builtin_amdgcn_get_fpenv`.
  **L527 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_get_fpenv`。
- **L528 EN**: Declares TableGen def record `__builtin_amdgcn_set_fpenv`.
  **L528 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_set_fpenv`。

### Lines 529-552

````tablegen

// These are special FE only builtins intended for forwarding the requirements
// to the ME.
def __builtin_amdgcn_processor_is : AMDGPUBuiltin<"__amdgpu_feature_predicate_t(char const *)", [NoThrow, Const, CustomTypeChecking, UnevaluatedArguments]>;
def __builtin_amdgcn_is_invocable : AMDGPUBuiltin<"__amdgpu_feature_predicate_t()", [NoThrow, Const, CustomTypeChecking, UnevaluatedArguments]>;

//===----------------------------------------------------------------------===//

// Wave Reduction builtins.

//===----------------------------------------------------------------------===//

def __builtin_amdgcn_wave_reduce_add_u32 : AMDGPUBuiltin<"uint32_t(uint32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_sub_u32 : AMDGPUBuiltin<"uint32_t(uint32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_min_i32 : AMDGPUBuiltin<"int32_t(int32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_min_u32 : AMDGPUBuiltin<"uint32_t(uint32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_max_i32 : AMDGPUBuiltin<"int32_t(int32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_max_u32 : AMDGPUBuiltin<"uint32_t(uint32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_and_b32 : AMDGPUBuiltin<"int32_t(int32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_or_b32 : AMDGPUBuiltin<"int32_t(int32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_xor_b32 : AMDGPUBuiltin<"int32_t(int32_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_add_u64 : AMDGPUBuiltin<"uint64_t(uint64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_sub_u64 : AMDGPUBuiltin<"uint64_t(uint64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_min_i64 : AMDGPUBuiltin<"int64_t(int64_t, _Constant int32_t)", [Const]>;
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Comment explains nearby logic, constraints, or intent: `These are special FE only builtins intended for forwarding the requirements`.
  **L530 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These are special FE only builtins intended for forwarding the requirements`。
- **L531 EN**: Comment explains nearby logic, constraints, or intent: `to the ME.`.
  **L531 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to the ME.`。
- **L532 EN**: Declares TableGen def record `__builtin_amdgcn_processor_is`.
  **L532 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_processor_is`。
- **L533 EN**: Declares TableGen def record `__builtin_amdgcn_is_invocable`.
  **L533 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_is_invocable`。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L535 EN**: Banner comment marking a file or section boundary.
  **L535 CN**: 横幅注释，用于标记文件或章节边界。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Comment explains nearby logic, constraints, or intent: `Wave Reduction builtins.`.
  **L537 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Wave Reduction builtins.`。
- **L538 EN**: Blank line separating nearby declarations or logic blocks.
  **L538 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L539 EN**: Banner comment marking a file or section boundary.
  **L539 CN**: 横幅注释，用于标记文件或章节边界。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L541 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_add_u32`.
  **L541 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_add_u32`。
- **L542 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_sub_u32`.
  **L542 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_sub_u32`。
- **L543 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_min_i32`.
  **L543 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_min_i32`。
- **L544 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_min_u32`.
  **L544 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_min_u32`。
- **L545 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_max_i32`.
  **L545 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_max_i32`。
- **L546 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_max_u32`.
  **L546 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_max_u32`。
- **L547 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_and_b32`.
  **L547 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_and_b32`。
- **L548 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_or_b32`.
  **L548 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_or_b32`。
- **L549 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_xor_b32`.
  **L549 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_xor_b32`。
- **L550 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_add_u64`.
  **L550 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_add_u64`。
- **L551 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_sub_u64`.
  **L551 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_sub_u64`。
- **L552 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_min_i64`.
  **L552 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_min_i64`。

### Lines 553-576

````tablegen
def __builtin_amdgcn_wave_reduce_min_u64 : AMDGPUBuiltin<"uint64_t(uint64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_max_i64 : AMDGPUBuiltin<"int64_t(int64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_max_u64 : AMDGPUBuiltin<"uint64_t(uint64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_and_b64 : AMDGPUBuiltin<"int64_t(int64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_or_b64 : AMDGPUBuiltin<"int64_t(int64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_xor_b64 : AMDGPUBuiltin<"int64_t(int64_t, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fadd_f32 : AMDGPUBuiltin<"float(float, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fsub_f32 : AMDGPUBuiltin<"float(float, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fmin_f32 : AMDGPUBuiltin<"float(float, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fmax_f32 : AMDGPUBuiltin<"float(float, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fadd_f64 : AMDGPUBuiltin<"double(double, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fsub_f64 : AMDGPUBuiltin<"double(double, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fmin_f64 : AMDGPUBuiltin<"double(double, _Constant int32_t)", [Const]>;
def __builtin_amdgcn_wave_reduce_fmax_f64 : AMDGPUBuiltin<"double(double, _Constant int32_t)", [Const]>;

//===----------------------------------------------------------------------===//
// R600-NI only builtins.
//===----------------------------------------------------------------------===//

def __builtin_r600_implicitarg_ptr : AMDGPUBuiltin<"unsigned char address_space<7> *()", [Const]>;

def __builtin_r600_read_tgid_x : AMDGPUBuiltin<"unsigned int()", [Const]>;
def __builtin_r600_read_tgid_y : AMDGPUBuiltin<"unsigned int()", [Const]>;
def __builtin_r600_read_tgid_z : AMDGPUBuiltin<"unsigned int()", [Const]>;
````
- **L553 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_min_u64`.
  **L553 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_min_u64`。
- **L554 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_max_i64`.
  **L554 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_max_i64`。
- **L555 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_max_u64`.
  **L555 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_max_u64`。
- **L556 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_and_b64`.
  **L556 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_and_b64`。
- **L557 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_or_b64`.
  **L557 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_or_b64`。
- **L558 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_xor_b64`.
  **L558 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_xor_b64`。
- **L559 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fadd_f32`.
  **L559 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fadd_f32`。
- **L560 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fsub_f32`.
  **L560 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fsub_f32`。
- **L561 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fmin_f32`.
  **L561 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fmin_f32`。
- **L562 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fmax_f32`.
  **L562 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fmax_f32`。
- **L563 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fadd_f64`.
  **L563 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fadd_f64`。
- **L564 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fsub_f64`.
  **L564 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fsub_f64`。
- **L565 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fmin_f64`.
  **L565 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fmin_f64`。
- **L566 EN**: Declares TableGen def record `__builtin_amdgcn_wave_reduce_fmax_f64`.
  **L566 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wave_reduce_fmax_f64`。
- **L567 EN**: Blank line separating nearby declarations or logic blocks.
  **L567 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L568 EN**: Banner comment marking a file or section boundary.
  **L568 CN**: 横幅注释，用于标记文件或章节边界。
- **L569 EN**: Comment explains nearby logic, constraints, or intent: `R600-NI only builtins.`.
  **L569 CN**: 注释解释附近代码的逻辑、约束或设计意图：`R600-NI only builtins.`。
- **L570 EN**: Banner comment marking a file or section boundary.
  **L570 CN**: 横幅注释，用于标记文件或章节边界。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Declares TableGen def record `__builtin_r600_implicitarg_ptr`.
  **L572 CN**: 声明 TableGen def 记录 `__builtin_r600_implicitarg_ptr`。
- **L573 EN**: Blank line separating nearby declarations or logic blocks.
  **L573 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L574 EN**: Declares TableGen def record `__builtin_r600_read_tgid_x`.
  **L574 CN**: 声明 TableGen def 记录 `__builtin_r600_read_tgid_x`。
- **L575 EN**: Declares TableGen def record `__builtin_r600_read_tgid_y`.
  **L575 CN**: 声明 TableGen def 记录 `__builtin_r600_read_tgid_y`。
- **L576 EN**: Declares TableGen def record `__builtin_r600_read_tgid_z`.
  **L576 CN**: 声明 TableGen def 记录 `__builtin_r600_read_tgid_z`。

### Lines 577-600

````tablegen

def __builtin_r600_read_tidig_x : AMDGPUBuiltin<"unsigned int()", [Const]>;
def __builtin_r600_read_tidig_y : AMDGPUBuiltin<"unsigned int()", [Const]>;
def __builtin_r600_read_tidig_z : AMDGPUBuiltin<"unsigned int()", [Const]>;

def __builtin_r600_recipsqrt_ieee : AMDGPUBuiltin<"double(double)", [Const]>;
def __builtin_r600_recipsqrt_ieeef : AMDGPUBuiltin<"float(float)", [Const]>;

//===----------------------------------------------------------------------===//
// MFMA builtins.
//===----------------------------------------------------------------------===//

def __builtin_amdgcn_mfma_f32_32x32x1f32 : AMDGPUBuiltin<"_ExtVector<32, float>(float, float, _ExtVector<32, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x1f32 : AMDGPUBuiltin<"_ExtVector<16, float>(float, float, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_4x4x1f32 : AMDGPUBuiltin<"_ExtVector<4, float>(float, float, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_32x32x2f32 : AMDGPUBuiltin<"_ExtVector<16, float>(float, float, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x4f32 : AMDGPUBuiltin<"_ExtVector<4, float>(float, float, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_32x32x4f16 : AMDGPUBuiltin<"_ExtVector<32, float>(_ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<32, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x4f16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_4x4x4f16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_32x32x8f16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x16f16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_i32_32x32x4i8 : AMDGPUBuiltin<"_ExtVector<32, int>(int, int, _ExtVector<32, int>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_i32_16x16x4i8 : AMDGPUBuiltin<"_ExtVector<16, int>(int, int, _ExtVector<16, int>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
````
- **L577 EN**: Blank line separating nearby declarations or logic blocks.
  **L577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L578 EN**: Declares TableGen def record `__builtin_r600_read_tidig_x`.
  **L578 CN**: 声明 TableGen def 记录 `__builtin_r600_read_tidig_x`。
- **L579 EN**: Declares TableGen def record `__builtin_r600_read_tidig_y`.
  **L579 CN**: 声明 TableGen def 记录 `__builtin_r600_read_tidig_y`。
- **L580 EN**: Declares TableGen def record `__builtin_r600_read_tidig_z`.
  **L580 CN**: 声明 TableGen def 记录 `__builtin_r600_read_tidig_z`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Declares TableGen def record `__builtin_r600_recipsqrt_ieee`.
  **L582 CN**: 声明 TableGen def 记录 `__builtin_r600_recipsqrt_ieee`。
- **L583 EN**: Declares TableGen def record `__builtin_r600_recipsqrt_ieeef`.
  **L583 CN**: 声明 TableGen def 记录 `__builtin_r600_recipsqrt_ieeef`。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Banner comment marking a file or section boundary.
  **L585 CN**: 横幅注释，用于标记文件或章节边界。
- **L586 EN**: Comment explains nearby logic, constraints, or intent: `MFMA builtins.`.
  **L586 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MFMA builtins.`。
- **L587 EN**: Banner comment marking a file or section boundary.
  **L587 CN**: 横幅注释，用于标记文件或章节边界。
- **L588 EN**: Blank line separating nearby declarations or logic blocks.
  **L588 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L589 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x1f32`.
  **L589 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x1f32`。
- **L590 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x1f32`.
  **L590 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x1f32`。
- **L591 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_4x4x1f32`.
  **L591 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_4x4x1f32`。
- **L592 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x2f32`.
  **L592 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x2f32`。
- **L593 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x4f32`.
  **L593 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x4f32`。
- **L594 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x4f16`.
  **L594 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x4f16`。
- **L595 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x4f16`.
  **L595 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x4f16`。
- **L596 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_4x4x4f16`.
  **L596 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_4x4x4f16`。
- **L597 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x8f16`.
  **L597 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x8f16`。
- **L598 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x16f16`.
  **L598 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x16f16`。
- **L599 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_32x32x4i8`.
  **L599 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_32x32x4i8`。
- **L600 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_16x16x4i8`.
  **L600 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_16x16x4i8`。

### Lines 601-624

````tablegen
def __builtin_amdgcn_mfma_i32_4x4x4i8 : AMDGPUBuiltin<"_ExtVector<4, int>(int, int, _ExtVector<4, int>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_i32_32x32x8i8 : AMDGPUBuiltin<"_ExtVector<16, int>(int, int, _ExtVector<16, int>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_i32_16x16x16i8 : AMDGPUBuiltin<"_ExtVector<4, int>(int, int, _ExtVector<4, int>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_32x32x2bf16 : AMDGPUBuiltin<"_ExtVector<32, float>(_ExtVector<2, short>, _ExtVector<2, short>, _ExtVector<32, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x2bf16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<2, short>, _ExtVector<2, short>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_4x4x2bf16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<2, short>, _ExtVector<2, short>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_32x32x4bf16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<2, short>, _ExtVector<2, short>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x8bf16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<2, short>, _ExtVector<2, short>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;

def __builtin_amdgcn_mfma_f32_32x32x4bf16_1k : AMDGPUBuiltin<"_ExtVector<32, float>(_ExtVector<4, short>, _ExtVector<4, short>, _ExtVector<32, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx90a-insts">;
def __builtin_amdgcn_mfma_f32_16x16x4bf16_1k : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, short>, _ExtVector<4, short>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx90a-insts">;
def __builtin_amdgcn_mfma_f32_4x4x4bf16_1k : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, short>, _ExtVector<4, short>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx90a-insts">;
def __builtin_amdgcn_mfma_f32_32x32x8bf16_1k : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, short>, _ExtVector<4, short>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx90a-insts">;
def __builtin_amdgcn_mfma_f32_16x16x16bf16_1k : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, short>, _ExtVector<4, short>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx90a-insts">;
def __builtin_amdgcn_mfma_f64_16x16x4f64 : AMDGPUBuiltin<"_ExtVector<4, double>(double, double, _ExtVector<4, double>, _Constant int, _Constant int, _Constant int)", [Const], "gfx90a-insts">;
def __builtin_amdgcn_mfma_f64_4x4x4f64 : AMDGPUBuiltin<"double(double, double, double, _Constant int, _Constant int, _Constant int)", [Const], "gfx90a-insts">;

def __builtin_amdgcn_mfma_i32_16x16x32_i8 : AMDGPUBuiltin<"_ExtVector<4, int>(int64_t, int64_t, _ExtVector<4, int>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_i32_32x32x16_i8 : AMDGPUBuiltin<"_ExtVector<16, int>(int64_t, int64_t, _ExtVector<16, int>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x8_xf32 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<2, float>, _ExtVector<2, float>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_32x32x4_xf32 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<2, float>, _ExtVector<2, float>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "mai-insts">;
def __builtin_amdgcn_mfma_f32_16x16x32_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<4, float>(int64_t, int64_t, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_mfma_f32_16x16x32_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<4, float>(int64_t, int64_t, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_mfma_f32_16x16x32_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<4, float>(int64_t, int64_t, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
````
- **L601 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_4x4x4i8`.
  **L601 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_4x4x4i8`。
- **L602 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_32x32x8i8`.
  **L602 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_32x32x8i8`。
- **L603 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_16x16x16i8`.
  **L603 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_16x16x16i8`。
- **L604 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x2bf16`.
  **L604 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x2bf16`。
- **L605 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x2bf16`.
  **L605 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x2bf16`。
- **L606 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_4x4x2bf16`.
  **L606 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_4x4x2bf16`。
- **L607 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x4bf16`.
  **L607 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x4bf16`。
- **L608 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x8bf16`.
  **L608 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x8bf16`。
- **L609 EN**: Blank line separating nearby declarations or logic blocks.
  **L609 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L610 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x4bf16_1k`.
  **L610 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x4bf16_1k`。
- **L611 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x4bf16_1k`.
  **L611 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x4bf16_1k`。
- **L612 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_4x4x4bf16_1k`.
  **L612 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_4x4x4bf16_1k`。
- **L613 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x8bf16_1k`.
  **L613 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x8bf16_1k`。
- **L614 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x16bf16_1k`.
  **L614 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x16bf16_1k`。
- **L615 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f64_16x16x4f64`.
  **L615 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f64_16x16x4f64`。
- **L616 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f64_4x4x4f64`.
  **L616 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f64_4x4x4f64`。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_16x16x32_i8`.
  **L618 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_16x16x32_i8`。
- **L619 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_32x32x16_i8`.
  **L619 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_32x32x16_i8`。
- **L620 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x8_xf32`.
  **L620 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x8_xf32`。
- **L621 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x4_xf32`.
  **L621 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x4_xf32`。
- **L622 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x32_bf8_bf8`.
  **L622 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x32_bf8_bf8`。
- **L623 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x32_bf8_fp8`.
  **L623 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x32_bf8_fp8`。
- **L624 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x32_fp8_bf8`.
  **L624 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x32_fp8_bf8`。

### Lines 625-648

````tablegen
def __builtin_amdgcn_mfma_f32_16x16x32_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<4, float>(int64_t, int64_t, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_mfma_f32_32x32x16_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<16, float>(int64_t, int64_t, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_mfma_f32_32x32x16_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<16, float>(int64_t, int64_t, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_mfma_f32_32x32x16_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<16, float>(int64_t, int64_t, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_mfma_f32_32x32x16_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<16, float>(int64_t, int64_t, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x32_f16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, _Float16>, _ExtVector<8, _Float16>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx940-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x16_f16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, _Float16>, _ExtVector<8, _Float16>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx940-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x32_bf16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, short>, _ExtVector<8, short>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx940-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x16_bf16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, short>, _ExtVector<8, short>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx940-insts">;
def __builtin_amdgcn_smfmac_i32_16x16x64_i8 : AMDGPUBuiltin<"_ExtVector<4, int>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<4, int>, int, _Constant int, _Constant int)", [Const], "gfx940-insts">;
def __builtin_amdgcn_smfmac_i32_32x32x32_i8 : AMDGPUBuiltin<"_ExtVector<16, int>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<16, int>, int, _Constant int, _Constant int)", [Const], "gfx940-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x64_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x64_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x64_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x64_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x32_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x32_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x32_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x32_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "fp8-insts">;

def __builtin_amdgcn_cvt_f32_bf8 : AMDGPUBuiltin<"float(int, _Constant int)", [Const], "fp8-conversion-insts">;
def __builtin_amdgcn_cvt_f32_fp8 : AMDGPUBuiltin<"float(int, _Constant int)", [Const], "fp8-conversion-insts">;
def __builtin_amdgcn_cvt_f32_fp8_e5m3 : AMDGPUBuiltin<"float(int, _Constant int)", [Const], "fp8e5m3-insts">;
def __builtin_amdgcn_cvt_pk_f32_bf8 : AMDGPUBuiltin<"_ExtVector<2, float>(int, _Constant bool)", [Const], "fp8-conversion-insts">;
````
- **L625 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x32_fp8_fp8`.
  **L625 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x32_fp8_fp8`。
- **L626 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x16_bf8_bf8`.
  **L626 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x16_bf8_bf8`。
- **L627 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x16_bf8_fp8`.
  **L627 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x16_bf8_fp8`。
- **L628 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x16_fp8_bf8`.
  **L628 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x16_fp8_bf8`。
- **L629 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x16_fp8_fp8`.
  **L629 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x16_fp8_fp8`。
- **L630 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x32_f16`.
  **L630 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x32_f16`。
- **L631 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x16_f16`.
  **L631 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x16_f16`。
- **L632 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x32_bf16`.
  **L632 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x32_bf16`。
- **L633 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x16_bf16`.
  **L633 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x16_bf16`。
- **L634 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_i32_16x16x64_i8`.
  **L634 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_i32_16x16x64_i8`。
- **L635 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_i32_32x32x32_i8`.
  **L635 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_i32_32x32x32_i8`。
- **L636 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x64_bf8_bf8`.
  **L636 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x64_bf8_bf8`。
- **L637 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x64_bf8_fp8`.
  **L637 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x64_bf8_fp8`。
- **L638 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x64_fp8_bf8`.
  **L638 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x64_fp8_bf8`。
- **L639 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x64_fp8_fp8`.
  **L639 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x64_fp8_fp8`。
- **L640 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x32_bf8_bf8`.
  **L640 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x32_bf8_bf8`。
- **L641 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x32_bf8_fp8`.
  **L641 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x32_bf8_fp8`。
- **L642 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x32_fp8_bf8`.
  **L642 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x32_fp8_bf8`。
- **L643 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x32_fp8_fp8`.
  **L643 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x32_fp8_fp8`。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L645 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_f32_bf8`.
  **L645 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_f32_bf8`。
- **L646 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_f32_fp8`.
  **L646 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_f32_fp8`。
- **L647 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_f32_fp8_e5m3`.
  **L647 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_f32_fp8_e5m3`。
- **L648 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_f32_bf8`.
  **L648 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_f32_bf8`。

### Lines 649-672

````tablegen
def __builtin_amdgcn_cvt_pk_f32_fp8 : AMDGPUBuiltin<"_ExtVector<2, float>(int, _Constant bool)", [Const], "fp8-conversion-insts">;
def __builtin_amdgcn_cvt_pk_bf8_f32 : AMDGPUBuiltin<"int(float, float, int, _Constant bool)", [Const], "fp8-conversion-insts">;
def __builtin_amdgcn_cvt_pk_fp8_f32 : AMDGPUBuiltin<"int(float, float, int, _Constant bool)", [Const], "fp8-conversion-insts">;
def __builtin_amdgcn_cvt_sr_bf8_f32 : AMDGPUBuiltin<"int(float, int, int, _Constant int)", [Const], "fp8-conversion-insts">;
def __builtin_amdgcn_cvt_sr_fp8_f32 : AMDGPUBuiltin<"int(float, int, int, _Constant int)", [Const], "fp8-conversion-insts">;

//===----------------------------------------------------------------------===//
// GFX950 only builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_mfma_scale_f32_16x16x128_f8f6f4 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<8, int32_t>, _ExtVector<8, int32_t>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int, int, _Constant int, int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_mfma_scale_f32_32x32x64_f8f6f4 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<8, int32_t>, _ExtVector<8, int32_t>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int, int, _Constant int, int)", [Const], "gfx950-insts">;

def __builtin_amdgcn_mfma_f32_16x16x32_f16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<8, _Float16>, _ExtVector<8, _Float16>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_mfma_f32_16x16x32_bf16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<8, __bf16>, _ExtVector<8, __bf16>, _ExtVector<4, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_mfma_f32_32x32x16_f16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<8, _Float16>, _ExtVector<8, _Float16>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_mfma_f32_32x32x16_bf16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<8, __bf16>, _ExtVector<8, __bf16>, _ExtVector<16, float>, _Constant int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_mfma_i32_16x16x64_i8 : AMDGPUBuiltin<"_ExtVector<4, int>(_ExtVector<4, int>, _ExtVector<4, int>, _ExtVector<4, int>, _Constant int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_mfma_i32_32x32x32_i8 : AMDGPUBuiltin<"_ExtVector<16, int>(_ExtVector<4, int>, _ExtVector<4, int>, _ExtVector<16, int>, _Constant int, _Constant int, _Constant int)", [Const], "gfx950-insts">;

def __builtin_amdgcn_smfmac_f32_16x16x64_f16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<8, _Float16>, _ExtVector<16, _Float16>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x32_f16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<8, _Float16>, _ExtVector<16, _Float16>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x64_bf16 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<8, __bf16>, _ExtVector<16, __bf16>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x32_bf16 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<8, __bf16>, _ExtVector<16, __bf16>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_i32_16x16x128_i8 : AMDGPUBuiltin<"_ExtVector<4, int>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<4, int>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
````
- **L649 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_f32_fp8`.
  **L649 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_f32_fp8`。
- **L650 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_bf8_f32`.
  **L650 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_bf8_f32`。
- **L651 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_fp8_f32`.
  **L651 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_fp8_f32`。
- **L652 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_bf8_f32`.
  **L652 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_bf8_f32`。
- **L653 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_fp8_f32`.
  **L653 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_fp8_f32`。
- **L654 EN**: Blank line separating nearby declarations or logic blocks.
  **L654 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L655 EN**: Banner comment marking a file or section boundary.
  **L655 CN**: 横幅注释，用于标记文件或章节边界。
- **L656 EN**: Comment explains nearby logic, constraints, or intent: `GFX950 only builtins.`.
  **L656 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX950 only builtins.`。
- **L657 EN**: Banner comment marking a file or section boundary.
  **L657 CN**: 横幅注释，用于标记文件或章节边界。
- **L658 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_scale_f32_16x16x128_f8f6f4`.
  **L658 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_scale_f32_16x16x128_f8f6f4`。
- **L659 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_scale_f32_32x32x64_f8f6f4`.
  **L659 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_scale_f32_32x32x64_f8f6f4`。
- **L660 EN**: Blank line separating nearby declarations or logic blocks.
  **L660 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L661 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x32_f16`.
  **L661 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x32_f16`。
- **L662 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_16x16x32_bf16`.
  **L662 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_16x16x32_bf16`。
- **L663 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x16_f16`.
  **L663 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x16_f16`。
- **L664 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_f32_32x32x16_bf16`.
  **L664 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_f32_32x32x16_bf16`。
- **L665 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_16x16x64_i8`.
  **L665 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_16x16x64_i8`。
- **L666 EN**: Declares TableGen def record `__builtin_amdgcn_mfma_i32_32x32x32_i8`.
  **L666 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_mfma_i32_32x32x32_i8`。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L668 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x64_f16`.
  **L668 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x64_f16`。
- **L669 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x32_f16`.
  **L669 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x32_f16`。
- **L670 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x64_bf16`.
  **L670 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x64_bf16`。
- **L671 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x32_bf16`.
  **L671 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x32_bf16`。
- **L672 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_i32_16x16x128_i8`.
  **L672 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_i32_16x16x128_i8`。

### Lines 673-696

````tablegen
def __builtin_amdgcn_smfmac_i32_32x32x64_i8 : AMDGPUBuiltin<"_ExtVector<16, int>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<16, int>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x128_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x128_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x128_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_16x16x128_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<4, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x64_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x64_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x64_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;
def __builtin_amdgcn_smfmac_f32_32x32x64_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<4, int>, _ExtVector<8, int>, _ExtVector<16, float>, int, _Constant int, _Constant int)", [Const], "gfx950-insts">;

def __builtin_amdgcn_permlane16_swap : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(unsigned int, unsigned int, _Constant bool, _Constant bool)", [Const], "permlane16-swap">;
def __builtin_amdgcn_permlane32_swap : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(unsigned int, unsigned int, _Constant bool, _Constant bool)", [Const], "permlane32-swap">;

def __builtin_amdgcn_ds_read_tr4_b64_v2i32 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<3> *)", [Const], "gfx950-insts">;
def __builtin_amdgcn_ds_read_tr6_b96_v3i32 : AMDGPUBuiltin<"_ExtVector<3, int>(_ExtVector<3, int> address_space<3> *)", [Const], "gfx950-insts">;
def __builtin_amdgcn_ds_read_tr8_b64_v2i32 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<3> *)", [Const], "gfx950-insts">;
def __builtin_amdgcn_ds_read_tr16_b64_v4i16 : AMDGPUBuiltin<"_ExtVector<4, short>(_ExtVector<4, short> address_space<3> *)", [Const], "gfx950-insts">;
def __builtin_amdgcn_ds_read_tr16_b64_v4f16 : AMDGPUBuiltin<"_ExtVector<4, __fp16>(_ExtVector<4, __fp16> address_space<3> *)", [Const], "gfx950-insts">;
def __builtin_amdgcn_ds_read_tr16_b64_v4bf16 : AMDGPUBuiltin<"_ExtVector<4, __bf16>(_ExtVector<4, __bf16> address_space<3> *)", [Const], "gfx950-insts">;

def __builtin_amdgcn_ashr_pk_i8_i32 : AMDGPUBuiltin<"unsigned short(unsigned int, unsigned int, unsigned int)", [Const], "ashr-pk-insts">;
def __builtin_amdgcn_ashr_pk_u8_i32 : AMDGPUBuiltin<"unsigned short(unsigned int, unsigned int, unsigned int)", [Const], "ashr-pk-insts">;

def __builtin_amdgcn_cvt_scalef32_2xpk16_fp6_f32 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<16, float>, _ExtVector<16, float>, float)", [Const], "gfx950-insts">;
````
- **L673 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_i32_32x32x64_i8`.
  **L673 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_i32_32x32x64_i8`。
- **L674 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x128_bf8_bf8`.
  **L674 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x128_bf8_bf8`。
- **L675 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x128_bf8_fp8`.
  **L675 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x128_bf8_fp8`。
- **L676 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x128_fp8_bf8`.
  **L676 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x128_fp8_bf8`。
- **L677 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_16x16x128_fp8_fp8`.
  **L677 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_16x16x128_fp8_fp8`。
- **L678 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x64_bf8_bf8`.
  **L678 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x64_bf8_bf8`。
- **L679 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x64_bf8_fp8`.
  **L679 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x64_bf8_fp8`。
- **L680 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x64_fp8_bf8`.
  **L680 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x64_fp8_bf8`。
- **L681 EN**: Declares TableGen def record `__builtin_amdgcn_smfmac_f32_32x32x64_fp8_fp8`.
  **L681 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_smfmac_f32_32x32x64_fp8_fp8`。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L683 EN**: Declares TableGen def record `__builtin_amdgcn_permlane16_swap`.
  **L683 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane16_swap`。
- **L684 EN**: Declares TableGen def record `__builtin_amdgcn_permlane32_swap`.
  **L684 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane32_swap`。
- **L685 EN**: Blank line separating nearby declarations or logic blocks.
  **L685 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L686 EN**: Declares TableGen def record `__builtin_amdgcn_ds_read_tr4_b64_v2i32`.
  **L686 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_read_tr4_b64_v2i32`。
- **L687 EN**: Declares TableGen def record `__builtin_amdgcn_ds_read_tr6_b96_v3i32`.
  **L687 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_read_tr6_b96_v3i32`。
- **L688 EN**: Declares TableGen def record `__builtin_amdgcn_ds_read_tr8_b64_v2i32`.
  **L688 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_read_tr8_b64_v2i32`。
- **L689 EN**: Declares TableGen def record `__builtin_amdgcn_ds_read_tr16_b64_v4i16`.
  **L689 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_read_tr16_b64_v4i16`。
- **L690 EN**: Declares TableGen def record `__builtin_amdgcn_ds_read_tr16_b64_v4f16`.
  **L690 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_read_tr16_b64_v4f16`。
- **L691 EN**: Declares TableGen def record `__builtin_amdgcn_ds_read_tr16_b64_v4bf16`.
  **L691 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_read_tr16_b64_v4bf16`。
- **L692 EN**: Blank line separating nearby declarations or logic blocks.
  **L692 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L693 EN**: Declares TableGen def record `__builtin_amdgcn_ashr_pk_i8_i32`.
  **L693 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ashr_pk_i8_i32`。
- **L694 EN**: Declares TableGen def record `__builtin_amdgcn_ashr_pk_u8_i32`.
  **L694 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ashr_pk_u8_i32`。
- **L695 EN**: Blank line separating nearby declarations or logic blocks.
  **L695 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L696 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_2xpk16_fp6_f32`.
  **L696 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_2xpk16_fp6_f32`。

### Lines 697-720

````tablegen
def __builtin_amdgcn_cvt_scalef32_2xpk16_bf6_f32 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<16, float>, _ExtVector<16, float>, float)", [Const], "gfx950-insts">;

//===----------------------------------------------------------------------===//
// GFX12+ only builtins.
//===----------------------------------------------------------------------===//

def __builtin_amdgcn_s_sleep_var : AMDGPUBuiltin<"void(unsigned int)", [], "gfx12-insts">;
def __builtin_amdgcn_permlane16_var : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, _Constant bool, _Constant bool)", [Const], "gfx12-insts">;
def __builtin_amdgcn_permlanex16_var : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, _Constant bool, _Constant bool)", [Const], "gfx12-insts">;
def __builtin_amdgcn_s_barrier_signal : AMDGPUBuiltin<"void(_Constant int)", [], "gfx12-insts">;
def __builtin_amdgcn_s_barrier_signal_var : AMDGPUBuiltin<"void(void *, int)", [], "gfx12-insts">;
def __builtin_amdgcn_s_barrier_wait : AMDGPUBuiltin<"void(_Constant short)", [], "gfx12-insts">;
def __builtin_amdgcn_s_barrier_signal_isfirst : AMDGPUBuiltin<"bool(_Constant int)", [], "gfx12-insts">;
def __builtin_amdgcn_s_barrier_init : AMDGPUBuiltin<"void(void *, int)", [], "gfx12-insts">;
def __builtin_amdgcn_s_barrier_join : AMDGPUBuiltin<"void(void *)", [], "gfx12-insts">;
def __builtin_amdgcn_s_barrier_leave : AMDGPUBuiltin<"void(_Constant short)", [], "gfx12-insts">;
def __builtin_amdgcn_s_get_barrier_state : AMDGPUBuiltin<"unsigned int(int)", [], "gfx12-insts">;
def __builtin_amdgcn_s_get_named_barrier_state : AMDGPUBuiltin<"unsigned int(void *)", [], "gfx12-insts">;
def __builtin_amdgcn_s_prefetch_data : AMDGPUBuiltin<"void(void const *, unsigned int)", [Const], "gfx12-insts">;
def __builtin_amdgcn_s_buffer_prefetch_data : AMDGPUBuiltin<"void(__amdgpu_buffer_rsrc_t, _Constant int, unsigned int)", [Const], "gfx12-insts">;

def __builtin_amdgcn_global_load_tr_b64_v2i32 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<1> *)", [Const], "gfx12-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr_b128_v8i16 : AMDGPUBuiltin<"_ExtVector<8, short>(_ExtVector<8, short> address_space<1> *)", [Const], "gfx12-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr_b128_v8f16 : AMDGPUBuiltin<"_ExtVector<8, __fp16>(_ExtVector<8, __fp16> address_space<1> *)", [Const], "gfx12-insts,wavefrontsize32">;
````
- **L697 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_2xpk16_bf6_f32`.
  **L697 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_2xpk16_bf6_f32`。
- **L698 EN**: Blank line separating nearby declarations or logic blocks.
  **L698 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L699 EN**: Banner comment marking a file or section boundary.
  **L699 CN**: 横幅注释，用于标记文件或章节边界。
- **L700 EN**: Comment explains nearby logic, constraints, or intent: `GFX12+ only builtins.`.
  **L700 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX12+ only builtins.`。
- **L701 EN**: Banner comment marking a file or section boundary.
  **L701 CN**: 横幅注释，用于标记文件或章节边界。
- **L702 EN**: Blank line separating nearby declarations or logic blocks.
  **L702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L703 EN**: Declares TableGen def record `__builtin_amdgcn_s_sleep_var`.
  **L703 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_sleep_var`。
- **L704 EN**: Declares TableGen def record `__builtin_amdgcn_permlane16_var`.
  **L704 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane16_var`。
- **L705 EN**: Declares TableGen def record `__builtin_amdgcn_permlanex16_var`.
  **L705 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlanex16_var`。
- **L706 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier_signal`.
  **L706 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier_signal`。
- **L707 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier_signal_var`.
  **L707 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier_signal_var`。
- **L708 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier_wait`.
  **L708 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier_wait`。
- **L709 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier_signal_isfirst`.
  **L709 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier_signal_isfirst`。
- **L710 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier_init`.
  **L710 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier_init`。
- **L711 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier_join`.
  **L711 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier_join`。
- **L712 EN**: Declares TableGen def record `__builtin_amdgcn_s_barrier_leave`.
  **L712 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_barrier_leave`。
- **L713 EN**: Declares TableGen def record `__builtin_amdgcn_s_get_barrier_state`.
  **L713 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_get_barrier_state`。
- **L714 EN**: Declares TableGen def record `__builtin_amdgcn_s_get_named_barrier_state`.
  **L714 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_get_named_barrier_state`。
- **L715 EN**: Declares TableGen def record `__builtin_amdgcn_s_prefetch_data`.
  **L715 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_prefetch_data`。
- **L716 EN**: Declares TableGen def record `__builtin_amdgcn_s_buffer_prefetch_data`.
  **L716 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_buffer_prefetch_data`。
- **L717 EN**: Blank line separating nearby declarations or logic blocks.
  **L717 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L718 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b64_v2i32`.
  **L718 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b64_v2i32`。
- **L719 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b128_v8i16`.
  **L719 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b128_v8i16`。
- **L720 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b128_v8f16`.
  **L720 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b128_v8f16`。

### Lines 721-744

````tablegen
def __builtin_amdgcn_global_load_tr_b128_v8bf16 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_ExtVector<8, __bf16> address_space<1> *)", [Const], "gfx12-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr_b64_i32 : AMDGPUBuiltin<"int(int address_space<1> *)", [Const], "gfx12-insts,wavefrontsize64">;
def __builtin_amdgcn_global_load_tr_b128_v4i16 : AMDGPUBuiltin<"_ExtVector<4, short>(_ExtVector<4, short> address_space<1> *)", [Const], "gfx12-insts,wavefrontsize64">;
def __builtin_amdgcn_global_load_tr_b128_v4f16 : AMDGPUBuiltin<"_ExtVector<4, __fp16>(_ExtVector<4, __fp16> address_space<1> *)", [Const], "gfx12-insts,wavefrontsize64">;
def __builtin_amdgcn_global_load_tr_b128_v4bf16 : AMDGPUBuiltin<"_ExtVector<4, __bf16>(_ExtVector<4, __bf16> address_space<1> *)", [Const], "gfx12-insts,wavefrontsize64">;

def __builtin_amdgcn_ds_bpermute_fi_b32 : AMDGPUBuiltin<"int(int, int)", [Const], "gfx12-insts">;

// For the following two builtins, the second and third return values of the
// intrinsics are returned through the last two pointer-type function arguments.
def __builtin_amdgcn_image_bvh8_intersect_ray : AMDGPUBuiltin<"_ExtVector<10, unsigned int>(uint64_t, float, unsigned char, _ExtVector<3, float>, _ExtVector<3, float>, unsigned int, _ExtVector<4, unsigned int>, _ExtVector<3, float> *, _ExtVector<3, float> *)", [Const], "gfx12-insts">;
def __builtin_amdgcn_image_bvh_dual_intersect_ray : AMDGPUBuiltin<"_ExtVector<10, unsigned int>(uint64_t, float, unsigned char, _ExtVector<3, float>, _ExtVector<3, float>, _ExtVector<2, unsigned int>, _ExtVector<4, unsigned int>, _ExtVector<3, float> *, _ExtVector<3, float> *)", [Const], "gfx12-insts">;

def __builtin_amdgcn_ds_bvh_stack_push4_pop1_rtn : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(unsigned int, unsigned int, _ExtVector<4, unsigned int>, _Constant int)", [], "gfx11-insts">;
def __builtin_amdgcn_ds_bvh_stack_push8_pop1_rtn : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(unsigned int, unsigned int, _ExtVector<8, unsigned int>, _Constant int)", [], "gfx12-insts">;

// The intrinsic returns {i64, i32}, the builtin returns <2 x i64>.
// The second return value of the intrinsic is zext'ed.
def __builtin_amdgcn_ds_bvh_stack_push8_pop2_rtn : AMDGPUBuiltin<"_ExtVector<2, uint64_t>(unsigned int, unsigned int, _ExtVector<8, unsigned int>, _Constant int)", [], "gfx12-insts">;

//===----------------------------------------------------------------------===//
// GFX1170, GFX12+ only builtins.
//===----------------------------------------------------------------------===//

````
- **L721 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b128_v8bf16`.
  **L721 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b128_v8bf16`。
- **L722 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b64_i32`.
  **L722 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b64_i32`。
- **L723 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b128_v4i16`.
  **L723 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b128_v4i16`。
- **L724 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b128_v4f16`.
  **L724 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b128_v4f16`。
- **L725 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr_b128_v4bf16`.
  **L725 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr_b128_v4bf16`。
- **L726 EN**: Blank line separating nearby declarations or logic blocks.
  **L726 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L727 EN**: Declares TableGen def record `__builtin_amdgcn_ds_bpermute_fi_b32`.
  **L727 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_bpermute_fi_b32`。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L729 EN**: Comment explains nearby logic, constraints, or intent: `For the following two builtins, the second and third return values of the`.
  **L729 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For the following two builtins, the second and third return values of the`。
- **L730 EN**: Comment explains nearby logic, constraints, or intent: `intrinsics are returned through the last two pointer-type function arguments.`.
  **L730 CN**: 注释解释附近代码的逻辑、约束或设计意图：`intrinsics are returned through the last two pointer-type function arguments.`。
- **L731 EN**: Declares TableGen def record `__builtin_amdgcn_image_bvh8_intersect_ray`.
  **L731 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_bvh8_intersect_ray`。
- **L732 EN**: Declares TableGen def record `__builtin_amdgcn_image_bvh_dual_intersect_ray`.
  **L732 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_bvh_dual_intersect_ray`。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L734 EN**: Declares TableGen def record `__builtin_amdgcn_ds_bvh_stack_push4_pop1_rtn`.
  **L734 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_bvh_stack_push4_pop1_rtn`。
- **L735 EN**: Declares TableGen def record `__builtin_amdgcn_ds_bvh_stack_push8_pop1_rtn`.
  **L735 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_bvh_stack_push8_pop1_rtn`。
- **L736 EN**: Blank line separating nearby declarations or logic blocks.
  **L736 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L737 EN**: Comment explains nearby logic, constraints, or intent: `The intrinsic returns {i64, i32}, the builtin returns <2 x i64>.`.
  **L737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The intrinsic returns {i64, i32}, the builtin returns <2 x i64>.`。
- **L738 EN**: Comment explains nearby logic, constraints, or intent: `The second return value of the intrinsic is zext'ed.`.
  **L738 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The second return value of the intrinsic is zext'ed.`。
- **L739 EN**: Declares TableGen def record `__builtin_amdgcn_ds_bvh_stack_push8_pop2_rtn`.
  **L739 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_bvh_stack_push8_pop2_rtn`。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Banner comment marking a file or section boundary.
  **L741 CN**: 横幅注释，用于标记文件或章节边界。
- **L742 EN**: Comment explains nearby logic, constraints, or intent: `GFX1170, GFX12+ only builtins.`.
  **L742 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX1170, GFX12+ only builtins.`。
- **L743 EN**: Banner comment marking a file or section boundary.
  **L743 CN**: 横幅注释，用于标记文件或章节边界。
- **L744 EN**: Blank line separating nearby declarations or logic blocks.
  **L744 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 745-768

````tablegen
//===----------------------------------------------------------------------===//
// WMMA builtins.
// Postfix w32 indicates the builtin requires wavefront size of 32.
// Postfix w64 indicates the builtin requires wavefront size of 64.
//
// Some of these are very similar to their base GFX11 counterparts, but they
// don't require replication of the A,B matrices, so they use fewer vector
// elements. Therefore, we add an "_gfx12" suffix to distinguish them from the
// existing builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_wmma_f32_16x16x16_f16_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, _Float16>, _ExtVector<8, _Float16>, _ExtVector<8, float>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAF32_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_bf16_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, short>, _ExtVector<8, short>, _ExtVector<8, float>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAF32_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f16_16x16x16_f16_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, _Float16>, _ExtVector<8, _Float16>, _ExtVector<8, _Float16>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAHalf_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, short>(_ExtVector<8, short>, _ExtVector<8, short>, _ExtVector<8, short>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAHalf_16x16x16_GFX12];
````
- **L745 EN**: Banner comment marking a file or section boundary.
  **L745 CN**: 横幅注释，用于标记文件或章节边界。
- **L746 EN**: Comment explains nearby logic, constraints, or intent: `WMMA builtins.`.
  **L746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`WMMA builtins.`。
- **L747 EN**: Comment explains nearby logic, constraints, or intent: `Postfix w32 indicates the builtin requires wavefront size of 32.`.
  **L747 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Postfix w32 indicates the builtin requires wavefront size of 32.`。
- **L748 EN**: Comment explains nearby logic, constraints, or intent: `Postfix w64 indicates the builtin requires wavefront size of 64.`.
  **L748 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Postfix w64 indicates the builtin requires wavefront size of 64.`。
- **L749 EN**: Separator comment used for visual grouping.
  **L749 CN**: 用于视觉分组的分隔注释。
- **L750 EN**: Comment explains nearby logic, constraints, or intent: `Some of these are very similar to their base GFX11 counterparts, but they`.
  **L750 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Some of these are very similar to their base GFX11 counterparts, but they`。
- **L751 EN**: Comment explains nearby logic, constraints, or intent: `don't require replication of the A,B matrices, so they use fewer vector`.
  **L751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`don't require replication of the A,B matrices, so they use fewer vector`。
- **L752 EN**: Comment explains nearby logic, constraints, or intent: `elements. Therefore, we add an "_gfx12" suffix to distinguish them from the`.
  **L752 CN**: 注释解释附近代码的逻辑、约束或设计意图：`elements. Therefore, we add an "_gfx12" suffix to distinguish them from the`。
- **L753 EN**: Comment explains nearby logic, constraints, or intent: `existing builtins.`.
  **L753 CN**: 注释解释附近代码的逻辑、约束或设计意图：`existing builtins.`。
- **L754 EN**: Banner comment marking a file or section boundary.
  **L754 CN**: 横幅注释，用于标记文件或章节边界。
- **L755 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_f16_w32_gfx12`.
  **L755 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_f16_w32_gfx12`。
- **L756 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX12];`.
  **L756 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX12];`。
- **L757 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L757 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L758 EN**: Closes the current lexical scope or compound statement.
  **L758 CN**: 结束当前词法作用域或复合语句块。
- **L759 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32_gfx12`.
  **L759 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32_gfx12`。
- **L760 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX12];`.
  **L760 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX12];`。
- **L761 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L761 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L762 EN**: Closes the current lexical scope or compound statement.
  **L762 CN**: 结束当前词法作用域或复合语句块。
- **L763 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x16_f16_w32_gfx12`.
  **L763 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x16_f16_w32_gfx12`。
- **L764 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAHalf_16x16x16_GFX12];`.
  **L764 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAHalf_16x16x16_GFX12];`。
- **L765 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L765 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L766 EN**: Closes the current lexical scope or compound statement.
  **L766 CN**: 结束当前词法作用域或复合语句块。
- **L767 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32_gfx12`.
  **L767 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32_gfx12`。
- **L768 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAHalf_16x16x16_GFX12];`.
  **L768 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAHalf_16x16x16_GFX12];`。

### Lines 769-792

````tablegen
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu8_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<2, int>, _Constant bool, _ExtVector<2, int>, _ExtVector<8, int>, _Constant bool)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAIU_16x16x16_GFX12];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu4_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, int, _Constant bool, int, _ExtVector<8, int>, _Constant bool)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAIU_16x16x16_GFX12];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}
// These are gfx1170 and gfx12 only, but for consistency with the other WMMA
// variants we're keeping the "_gfx12" suffix.
def __builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<2, int>, _ExtVector<8, float>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<2, int>, _ExtVector<8, float>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<2, int>, _ExtVector<8, float>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
````
- **L769 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L769 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L770 EN**: Closes the current lexical scope or compound statement.
  **L770 CN**: 结束当前词法作用域或复合语句块。
- **L771 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32_gfx12`.
  **L771 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32_gfx12`。
- **L772 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX12];`.
  **L772 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX12];`。
- **L773 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L773 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L774 EN**: Closes the current lexical scope or compound statement.
  **L774 CN**: 结束当前词法作用域或复合语句块。
- **L775 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32_gfx12`.
  **L775 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32_gfx12`。
- **L776 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX12];`.
  **L776 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX12];`。
- **L777 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L777 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L778 EN**: Closes the current lexical scope or compound statement.
  **L778 CN**: 结束当前词法作用域或复合语句块。
- **L779 EN**: Comment explains nearby logic, constraints, or intent: `These are gfx1170 and gfx12 only, but for consistency with the other WMMA`.
  **L779 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These are gfx1170 and gfx12 only, but for consistency with the other WMMA`。
- **L780 EN**: Comment explains nearby logic, constraints, or intent: `variants we're keeping the "_gfx12" suffix.`.
  **L780 CN**: 注释解释附近代码的逻辑、约束或设计意图：`variants we're keeping the "_gfx12" suffix.`。
- **L781 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w32_gfx12`.
  **L781 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w32_gfx12`。
- **L782 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L782 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L783 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L783 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L784 EN**: Closes the current lexical scope or compound statement.
  **L784 CN**: 结束当前词法作用域或复合语句块。
- **L785 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w32_gfx12`.
  **L785 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w32_gfx12`。
- **L786 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L786 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L787 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L787 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L788 EN**: Closes the current lexical scope or compound statement.
  **L788 CN**: 结束当前词法作用域或复合语句块。
- **L789 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w32_gfx12`.
  **L789 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w32_gfx12`。
- **L790 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L790 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L791 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L791 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L792 EN**: Closes the current lexical scope or compound statement.
  **L792 CN**: 结束当前词法作用域或复合语句块。

### Lines 793-816

````tablegen
def __builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<2, int>, _ExtVector<8, float>)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_i32_16x16x32_iu4_w32_gfx12 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<2, int>, _Constant bool, _ExtVector<2, int>, _ExtVector<8, int>, _Constant bool)", [Const], "wmma-128b-insts,wavefrontsize32"> {
  let Documentation = [DocWMMAIU4_16x16x32_GFX12];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}

def __builtin_amdgcn_wmma_f32_16x16x16_f16_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<4, float>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAF32_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_bf16_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, short>, _ExtVector<4, short>, _ExtVector<4, float>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAF32_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f16_16x16x16_f16_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(_ExtVector<4, _Float16>, _ExtVector<4, _Float16>, _ExtVector<4, _Float16>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAHalf_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, short>(_ExtVector<4, short>, _ExtVector<4, short>, _ExtVector<4, short>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAHalf_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
````
- **L793 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w32_gfx12`.
  **L793 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w32_gfx12`。
- **L794 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L794 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L795 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L795 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L796 EN**: Closes the current lexical scope or compound statement.
  **L796 CN**: 结束当前词法作用域或复合语句块。
- **L797 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x32_iu4_w32_gfx12`.
  **L797 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x32_iu4_w32_gfx12`。
- **L798 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU4_16x16x32_GFX12];`.
  **L798 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU4_16x16x32_GFX12];`。
- **L799 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L799 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L800 EN**: Closes the current lexical scope or compound statement.
  **L800 CN**: 结束当前词法作用域或复合语句块。
- **L801 EN**: Blank line separating nearby declarations or logic blocks.
  **L801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L802 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_f16_w64_gfx12`.
  **L802 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_f16_w64_gfx12`。
- **L803 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX12];`.
  **L803 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX12];`。
- **L804 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L804 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L805 EN**: Closes the current lexical scope or compound statement.
  **L805 CN**: 结束当前词法作用域或复合语句块。
- **L806 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64_gfx12`.
  **L806 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w64_gfx12`。
- **L807 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAF32_16x16x16_GFX12];`.
  **L807 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAF32_16x16x16_GFX12];`。
- **L808 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L808 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L809 EN**: Closes the current lexical scope or compound statement.
  **L809 CN**: 结束当前词法作用域或复合语句块。
- **L810 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x16_f16_w64_gfx12`.
  **L810 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x16_f16_w64_gfx12`。
- **L811 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAHalf_16x16x16_GFX12];`.
  **L811 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAHalf_16x16x16_GFX12];`。
- **L812 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L812 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L813 EN**: Closes the current lexical scope or compound statement.
  **L813 CN**: 结束当前词法作用域或复合语句块。
- **L814 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64_gfx12`.
  **L814 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w64_gfx12`。
- **L815 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAHalf_16x16x16_GFX12];`.
  **L815 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAHalf_16x16x16_GFX12];`。
- **L816 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L816 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。

### Lines 817-840

````tablegen
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu8_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, int, _Constant bool, int, _ExtVector<4, int>, _Constant bool)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAIU_16x16x16_GFX12];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}
def __builtin_amdgcn_wmma_i32_16x16x16_iu4_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, int, _Constant bool, int, _ExtVector<4, int>, _Constant bool)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAIU_16x16x16_GFX12];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}
// These are gfx1170 and gfx12 only, but for consistency with the other WMMA
// variants we're keeping the "_gfx12" suffix.
def __builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, _ExtVector<4, float>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, _ExtVector<4, float>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, _ExtVector<4, float>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, _ExtVector<4, float>)", [Const], "wmma-128b-insts,wavefrontsize64"> {
````
- **L817 EN**: Closes the current lexical scope or compound statement.
  **L817 CN**: 结束当前词法作用域或复合语句块。
- **L818 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64_gfx12`.
  **L818 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w64_gfx12`。
- **L819 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX12];`.
  **L819 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX12];`。
- **L820 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L820 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L821 EN**: Closes the current lexical scope or compound statement.
  **L821 CN**: 结束当前词法作用域或复合语句块。
- **L822 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64_gfx12`.
  **L822 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w64_gfx12`。
- **L823 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU_16x16x16_GFX12];`.
  **L823 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU_16x16x16_GFX12];`。
- **L824 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L824 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L825 EN**: Closes the current lexical scope or compound statement.
  **L825 CN**: 结束当前词法作用域或复合语句块。
- **L826 EN**: Comment explains nearby logic, constraints, or intent: `These are gfx1170 and gfx12 only, but for consistency with the other WMMA`.
  **L826 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These are gfx1170 and gfx12 only, but for consistency with the other WMMA`。
- **L827 EN**: Comment explains nearby logic, constraints, or intent: `variants we're keeping the "_gfx12" suffix.`.
  **L827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`variants we're keeping the "_gfx12" suffix.`。
- **L828 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w64_gfx12`.
  **L828 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_fp8_fp8_w64_gfx12`。
- **L829 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L829 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L830 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L830 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L831 EN**: Closes the current lexical scope or compound statement.
  **L831 CN**: 结束当前词法作用域或复合语句块。
- **L832 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w64_gfx12`.
  **L832 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_fp8_bf8_w64_gfx12`。
- **L833 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L833 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L834 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L834 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L835 EN**: Closes the current lexical scope or compound statement.
  **L835 CN**: 结束当前词法作用域或复合语句块。
- **L836 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w64_gfx12`.
  **L836 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf8_fp8_w64_gfx12`。
- **L837 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L837 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L838 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L838 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L839 EN**: Closes the current lexical scope or compound statement.
  **L839 CN**: 结束当前词法作用域或复合语句块。
- **L840 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w64_gfx12`.
  **L840 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x16_bf8_bf8_w64_gfx12`。

### Lines 841-864

````tablegen
  let Documentation = [DocWMMAFP8_16x16x16_GFX12];
  let ArgNames = ["a", "b", "c"];
}
def __builtin_amdgcn_wmma_i32_16x16x32_iu4_w64_gfx12 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, int, _Constant bool, int, _ExtVector<4, int>, _Constant bool)", [Const], "wmma-128b-insts,wavefrontsize64"> {
  let Documentation = [DocWMMAIU4_16x16x32_GFX12];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];
}

def __builtin_amdgcn_swmmac_f32_16x16x32_f16_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, __fp16>, _ExtVector<16, __fp16>, _ExtVector<8, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x32_bf16_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, short>, _ExtVector<16, short>, _ExtVector<8, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f16_16x16x32_f16_w32 : AMDGPUBuiltin<"_ExtVector<8, __fp16>(_ExtVector<8, __fp16>, _ExtVector<16, __fp16>, _ExtVector<8, __fp16>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w32 : AMDGPUBuiltin<"_ExtVector<8, short>(_ExtVector<8, short>, _ExtVector<16, short>, _ExtVector<8, short>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_i32_16x16x32_iu8_w32 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<2, int>, _Constant bool, _ExtVector<4, int>, _ExtVector<8, int>, int, _Constant bool)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_i32_16x16x32_iu4_w32 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, int, _Constant bool, _ExtVector<2, int>, _ExtVector<8, int>, int, _Constant bool)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_i32_16x16x64_iu4_w32 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<2, int>, _Constant bool, _ExtVector<4, int>, _ExtVector<8, int>, int, _Constant bool)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<8, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<8, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<8, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w32 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, int>, _ExtVector<4, int>, _ExtVector<8, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize32">;

def __builtin_amdgcn_swmmac_f32_16x16x32_f16_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, __fp16>, _ExtVector<8, __fp16>, _ExtVector<4, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_f32_16x16x32_bf16_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(_ExtVector<4, short>, _ExtVector<8, short>, _ExtVector<4, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_f16_16x16x32_f16_w64 : AMDGPUBuiltin<"_ExtVector<4, __fp16>(_ExtVector<4, __fp16>, _ExtVector<8, __fp16>, _ExtVector<4, __fp16>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w64 : AMDGPUBuiltin<"_ExtVector<4, short>(_ExtVector<4, short>, _ExtVector<8, short>, _ExtVector<4, short>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
````
- **L841 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAFP8_16x16x16_GFX12];`.
  **L841 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAFP8_16x16x16_GFX12];`。
- **L842 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c"];`.
  **L842 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c"];`。
- **L843 EN**: Closes the current lexical scope or compound statement.
  **L843 CN**: 结束当前词法作用域或复合语句块。
- **L844 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x32_iu4_w64_gfx12`.
  **L844 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x32_iu4_w64_gfx12`。
- **L845 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMAIU4_16x16x32_GFX12];`.
  **L845 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMAIU4_16x16x32_GFX12];`。
- **L846 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`.
  **L846 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "clamp"];`。
- **L847 EN**: Closes the current lexical scope or compound statement.
  **L847 CN**: 结束当前词法作用域或复合语句块。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_f16_w32`.
  **L849 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_f16_w32`。
- **L850 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w32`.
  **L850 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w32`。
- **L851 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f16_16x16x32_f16_w32`.
  **L851 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f16_16x16x32_f16_w32`。
- **L852 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w32`.
  **L852 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w32`。
- **L853 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w32`.
  **L853 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w32`。
- **L854 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w32`.
  **L854 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w32`。
- **L855 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w32`.
  **L855 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w32`。
- **L856 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w32`.
  **L856 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w32`。
- **L857 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w32`.
  **L857 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w32`。
- **L858 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w32`.
  **L858 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w32`。
- **L859 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w32`.
  **L859 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w32`。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_f16_w64`.
  **L861 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_f16_w64`。
- **L862 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w64`.
  **L862 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_bf16_w64`。
- **L863 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f16_16x16x32_f16_w64`.
  **L863 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f16_16x16x32_f16_w64`。
- **L864 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w64`.
  **L864 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_bf16_16x16x32_bf16_w64`。

### Lines 865-888

````tablegen
def __builtin_amdgcn_swmmac_i32_16x16x32_iu8_w64 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, int, _Constant bool, _ExtVector<2, int>, _ExtVector<4, int>, int, _Constant bool)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_i32_16x16x32_iu4_w64 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, int, _Constant bool, int, _ExtVector<4, int>, int, _Constant bool)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_i32_16x16x64_iu4_w64 : AMDGPUBuiltin<"_ExtVector<4, int>(_Constant bool, int, _Constant bool, _ExtVector<2, int>, _ExtVector<4, int>, int, _Constant bool)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(int, _ExtVector<2, int>, _ExtVector<4, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(int, _ExtVector<2, int>, _ExtVector<4, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(int, _ExtVector<2, int>, _ExtVector<4, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;
def __builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w64 : AMDGPUBuiltin<"_ExtVector<4, float>(int, _ExtVector<2, int>, _ExtVector<4, float>, int)", [Const], "swmmac-gfx1200-insts,wavefrontsize64">;

def __builtin_amdgcn_prng_b32 : AMDGPUBuiltin<"unsigned int(unsigned int)", [Const], "prng-inst">;
def __builtin_amdgcn_cvt_scalef32_pk32_fp6_f16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, _Float16>, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_bf6_f16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, _Float16>, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_fp6_bf16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, __bf16>, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_bf6_bf16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, __bf16>, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_f16_fp8 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(_ExtVector<2, _Float16>, int, float, _Constant int, _Constant bool)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_f16_bf8 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(_ExtVector<2, _Float16>, int, float, _Constant int, _Constant bool)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_f32_fp8 : AMDGPUBuiltin<"float(int, float, _Constant int)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_f32_bf8 : AMDGPUBuiltin<"float(int, float, _Constant int)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_fp8_f32 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, float, float, float, _Constant bool)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_bf8_f32 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, float, float, float, _Constant bool)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_f32_fp8 : AMDGPUBuiltin<"_ExtVector<2, float>(unsigned int, float, _Constant bool)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_f32_bf8 : AMDGPUBuiltin<"_ExtVector<2, float>(unsigned int, float, _Constant bool)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_fp8_f16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, _ExtVector<2, _Float16>, float, _Constant bool)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_fp8_bf16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, _ExtVector<2, __bf16>, float, _Constant bool)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_bf8_f16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, _ExtVector<2, _Float16>, float, _Constant bool)", [Const], "bf8-cvt-scale-insts">;
````
- **L865 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w64`.
  **L865 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_i32_16x16x32_iu8_w64`。
- **L866 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w64`.
  **L866 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_i32_16x16x32_iu4_w64`。
- **L867 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w64`.
  **L867 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_i32_16x16x64_iu4_w64`。
- **L868 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w64`.
  **L868 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_fp8_w64`。
- **L869 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w64`.
  **L869 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_fp8_bf8_w64`。
- **L870 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w64`.
  **L870 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_fp8_w64`。
- **L871 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w64`.
  **L871 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x32_bf8_bf8_w64`。
- **L872 EN**: Blank line separating nearby declarations or logic blocks.
  **L872 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L873 EN**: Declares TableGen def record `__builtin_amdgcn_prng_b32`.
  **L873 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_prng_b32`。
- **L874 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_fp6_f16`.
  **L874 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_fp6_f16`。
- **L875 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_bf6_f16`.
  **L875 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_bf6_f16`。
- **L876 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_fp6_bf16`.
  **L876 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_fp6_bf16`。
- **L877 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_bf6_bf16`.
  **L877 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_bf6_bf16`。
- **L878 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_f16_fp8`.
  **L878 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_f16_fp8`。
- **L879 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_f16_bf8`.
  **L879 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_f16_bf8`。
- **L880 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_f32_fp8`.
  **L880 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_f32_fp8`。
- **L881 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_f32_bf8`.
  **L881 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_f32_bf8`。
- **L882 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_fp8_f32`.
  **L882 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_fp8_f32`。
- **L883 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_bf8_f32`.
  **L883 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_bf8_f32`。
- **L884 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_f32_fp8`.
  **L884 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_f32_fp8`。
- **L885 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_f32_bf8`.
  **L885 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_f32_bf8`。
- **L886 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_fp8_f16`.
  **L886 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_fp8_f16`。
- **L887 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_fp8_bf16`.
  **L887 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_fp8_bf16`。
- **L888 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_bf8_f16`.
  **L888 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_bf8_f16`。

### Lines 889-912

````tablegen
def __builtin_amdgcn_cvt_scalef32_pk_bf8_bf16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, _ExtVector<2, __bf16>, float, _Constant bool)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_f32_fp4 : AMDGPUBuiltin<"_ExtVector<2, float>(unsigned int, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_fp4_f32 : AMDGPUBuiltin<"unsigned int(unsigned int, float, float, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_f16_fp4 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(unsigned int, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_bf16_fp4 : AMDGPUBuiltin<"_ExtVector<2, __bf16>(unsigned int, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_f32_fp6 : AMDGPUBuiltin<"_ExtVector<32, float>(_ExtVector<6, unsigned int>, float)", [Const], "fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_f32_bf6 : AMDGPUBuiltin<"_ExtVector<32, float>(_ExtVector<6, unsigned int>, float)", [Const], "fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_f16_fp6 : AMDGPUBuiltin<"_ExtVector<32, _Float16>(_ExtVector<6, unsigned int>, float)", [Const], "fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_bf16_fp6 : AMDGPUBuiltin<"_ExtVector<32, __bf16>(_ExtVector<6, unsigned int>, float)", [Const], "fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_f16_bf6 : AMDGPUBuiltin<"_ExtVector<32, _Float16>(_ExtVector<6, unsigned int>, float)", [Const], "fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk32_bf16_bf6 : AMDGPUBuiltin<"_ExtVector<32, __bf16>(_ExtVector<6, unsigned int>, float)", [Const], "fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_f16_fp8 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(unsigned int, float, _Constant bool)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_bf16_fp8 : AMDGPUBuiltin<"_ExtVector<2, __bf16>(unsigned int, float, _Constant bool)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_f16_bf8 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(unsigned int, float, _Constant bool)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_bf16_bf8 : AMDGPUBuiltin<"_ExtVector<2, __bf16>(unsigned int, float, _Constant bool)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_fp4_f16 : AMDGPUBuiltin<"unsigned int(unsigned int, _ExtVector<2, _Float16>, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_pk_fp4_bf16 : AMDGPUBuiltin<"unsigned int(unsigned int, _ExtVector<2, __bf16>, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk_fp4_f16 : AMDGPUBuiltin<"unsigned int(unsigned int, _ExtVector<2, _Float16>, unsigned int, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk_fp4_bf16 : AMDGPUBuiltin<"unsigned int(unsigned int, _ExtVector<2, __bf16>, unsigned int, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk_fp4_f32 : AMDGPUBuiltin<"unsigned int(unsigned int, _ExtVector<2, float>, unsigned int, float, _Constant int)", [Const], "fp4-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_bf8_bf16 : AMDGPUBuiltin<"int(int, __bf16, unsigned int, float, _Constant int)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_bf8_f16 : AMDGPUBuiltin<"int(int, _Float16, unsigned int, float, _Constant int)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_bf8_f32 : AMDGPUBuiltin<"int(int, float, unsigned int, float, _Constant int)", [Const], "bf8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_fp8_bf16 : AMDGPUBuiltin<"int(int, __bf16, unsigned int, float, _Constant int)", [Const], "fp8-cvt-scale-insts">;
````
- **L889 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_bf8_bf16`.
  **L889 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_bf8_bf16`。
- **L890 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_f32_fp4`.
  **L890 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_f32_fp4`。
- **L891 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_fp4_f32`.
  **L891 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_fp4_f32`。
- **L892 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_f16_fp4`.
  **L892 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_f16_fp4`。
- **L893 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_bf16_fp4`.
  **L893 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_bf16_fp4`。
- **L894 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_f32_fp6`.
  **L894 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_f32_fp6`。
- **L895 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_f32_bf6`.
  **L895 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_f32_bf6`。
- **L896 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_f16_fp6`.
  **L896 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_f16_fp6`。
- **L897 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_bf16_fp6`.
  **L897 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_bf16_fp6`。
- **L898 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_f16_bf6`.
  **L898 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_f16_bf6`。
- **L899 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk32_bf16_bf6`.
  **L899 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk32_bf16_bf6`。
- **L900 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_f16_fp8`.
  **L900 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_f16_fp8`。
- **L901 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_bf16_fp8`.
  **L901 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_bf16_fp8`。
- **L902 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_f16_bf8`.
  **L902 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_f16_bf8`。
- **L903 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_bf16_bf8`.
  **L903 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_bf16_bf8`。
- **L904 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_fp4_f16`.
  **L904 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_fp4_f16`。
- **L905 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk_fp4_bf16`.
  **L905 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk_fp4_bf16`。
- **L906 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk_fp4_f16`.
  **L906 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk_fp4_f16`。
- **L907 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk_fp4_bf16`.
  **L907 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk_fp4_bf16`。
- **L908 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk_fp4_f32`.
  **L908 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk_fp4_f32`。
- **L909 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_bf8_bf16`.
  **L909 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_bf8_bf16`。
- **L910 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_bf8_f16`.
  **L910 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_bf8_f16`。
- **L911 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_bf8_f32`.
  **L911 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_bf8_f32`。
- **L912 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_fp8_bf16`.
  **L912 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_fp8_bf16`。

### Lines 913-936

````tablegen
def __builtin_amdgcn_cvt_scalef32_sr_fp8_f16 : AMDGPUBuiltin<"int(int, _Float16, unsigned int, float, _Constant int)", [Const], "fp8-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_fp8_f32 : AMDGPUBuiltin<"int(int, float, unsigned int, float, _Constant int)", [Const], "fp8-cvt-scale-insts">;

def __builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_bf16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, __bf16>, unsigned int, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_f16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, _Float16>, unsigned int, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_f32 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, float>, unsigned int, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_bf16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, __bf16>, unsigned int, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_f16 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, _Float16>, unsigned int, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_f32 : AMDGPUBuiltin<"_ExtVector<6, unsigned int>(_ExtVector<32, float>, unsigned int, float)", [Const], "f16bf16-to-fp6bf6-cvt-scale-insts">;
def __builtin_amdgcn_bitop3_b32 : AMDGPUBuiltin<"int(int, int, int, _Constant unsigned int)", [Const], "bitop3-insts">;
def __builtin_amdgcn_bitop3_b16 : AMDGPUBuiltin<"short(short, short, short, _Constant unsigned int)", [Const], "bitop3-insts">;

def __builtin_amdgcn_cvt_sr_bf16_f32 : AMDGPUBuiltin<"_ExtVector<2, __bf16>(_ExtVector<2, __bf16>, float, unsigned int, _Constant bool)", [Const], "f32-to-f16bf16-cvt-sr-insts">;
def __builtin_amdgcn_cvt_sr_f16_f32 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(_ExtVector<2, _Float16>, float, unsigned int, _Constant bool)", [Const], "f32-to-f16bf16-cvt-sr-insts">;

//===----------------------------------------------------------------------===//
// GFX1250+ only builtins.
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_s_cluster_barrier : AMDGPUBuiltin<"void()", [], "gfx1250-insts">;

def __builtin_amdgcn_flat_prefetch : AMDGPUBuiltin<"void(void const address_space<0> *, _Constant int)", [Const], "vmem-pref-insts">;
def __builtin_amdgcn_global_prefetch : AMDGPUBuiltin<"void(void const address_space<1> *, _Constant int)", [Const], "vmem-pref-insts">;

def __builtin_amdgcn_global_load_monitor_b32 : AMDGPUBuiltin<"int(int address_space<1> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
````
- **L913 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_fp8_f16`.
  **L913 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_fp8_f16`。
- **L914 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_fp8_f32`.
  **L914 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_fp8_f32`。
- **L915 EN**: Blank line separating nearby declarations or logic blocks.
  **L915 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L916 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_bf16`.
  **L916 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_bf16`。
- **L917 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_f16`.
  **L917 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_f16`。
- **L918 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_f32`.
  **L918 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk32_bf6_f32`。
- **L919 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_bf16`.
  **L919 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_bf16`。
- **L920 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_f16`.
  **L920 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_f16`。
- **L921 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_f32`.
  **L921 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk32_fp6_f32`。
- **L922 EN**: Declares TableGen def record `__builtin_amdgcn_bitop3_b32`.
  **L922 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_bitop3_b32`。
- **L923 EN**: Declares TableGen def record `__builtin_amdgcn_bitop3_b16`.
  **L923 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_bitop3_b16`。
- **L924 EN**: Blank line separating nearby declarations or logic blocks.
  **L924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L925 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_bf16_f32`.
  **L925 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_bf16_f32`。
- **L926 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_f16_f32`.
  **L926 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_f16_f32`。
- **L927 EN**: Blank line separating nearby declarations or logic blocks.
  **L927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L928 EN**: Banner comment marking a file or section boundary.
  **L928 CN**: 横幅注释，用于标记文件或章节边界。
- **L929 EN**: Comment explains nearby logic, constraints, or intent: `GFX1250+ only builtins.`.
  **L929 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX1250+ only builtins.`。
- **L930 EN**: Banner comment marking a file or section boundary.
  **L930 CN**: 横幅注释，用于标记文件或章节边界。
- **L931 EN**: Declares TableGen def record `__builtin_amdgcn_s_cluster_barrier`.
  **L931 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_cluster_barrier`。
- **L932 EN**: Blank line separating nearby declarations or logic blocks.
  **L932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L933 EN**: Declares TableGen def record `__builtin_amdgcn_flat_prefetch`.
  **L933 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_prefetch`。
- **L934 EN**: Declares TableGen def record `__builtin_amdgcn_global_prefetch`.
  **L934 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_prefetch`。
- **L935 EN**: Blank line separating nearby declarations or logic blocks.
  **L935 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L936 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_monitor_b32`.
  **L936 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_monitor_b32`。

### Lines 937-960

````tablegen
def __builtin_amdgcn_global_load_monitor_b64 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<1> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_load_monitor_b128 : AMDGPUBuiltin<"_ExtVector<4, int>(_ExtVector<4, int> address_space<1> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_flat_load_monitor_b32 : AMDGPUBuiltin<"int(int address_space<0> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_flat_load_monitor_b64 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<0> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_flat_load_monitor_b128 : AMDGPUBuiltin<"_ExtVector<4, int>(_ExtVector<4, int> address_space<0> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cluster_load_b32 : AMDGPUBuiltin<"int(int address_space<1> *, _Constant int, int)", [Const], "mcast-load-insts,wavefrontsize32">;
def __builtin_amdgcn_cluster_load_b64 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<1> *, _Constant int, int)", [Const], "mcast-load-insts,wavefrontsize32">;
def __builtin_amdgcn_cluster_load_b128 : AMDGPUBuiltin<"_ExtVector<4, int>(_ExtVector<4, int> address_space<1> *, _Constant int, int)", [Const], "mcast-load-insts,wavefrontsize32">;
def __builtin_amdgcn_cluster_load_async_to_lds_b8 : AMDGPUBuiltin<"void(char address_space<1> *, char address_space<3> *, _Constant int, _Constant int, int)", [Const], "mcast-load-insts,wavefrontsize32">;
def __builtin_amdgcn_cluster_load_async_to_lds_b32 : AMDGPUBuiltin<"void(int address_space<1> *, int address_space<3> *, _Constant int, _Constant int, int)", [Const], "mcast-load-insts,wavefrontsize32">;
def __builtin_amdgcn_cluster_load_async_to_lds_b64 : AMDGPUBuiltin<"void(_ExtVector<2, int> address_space<1> *, _ExtVector<2, int> address_space<3> *, _Constant int, _Constant int, int)", [Const], "mcast-load-insts,wavefrontsize32">;
def __builtin_amdgcn_cluster_load_async_to_lds_b128 : AMDGPUBuiltin<"void(_ExtVector<4, int> address_space<1> *, _ExtVector<4, int> address_space<3> *, _Constant int, _Constant int, int)", [Const], "mcast-load-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_async_to_lds_b8 : AMDGPUBuiltin<"void(char address_space<1> *, char address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_load_async_to_lds_b32 : AMDGPUBuiltin<"void(int address_space<1> *, int address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_load_async_to_lds_b64 : AMDGPUBuiltin<"void(_ExtVector<2, int> address_space<1> *, _ExtVector<2, int> address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_load_async_to_lds_b128 : AMDGPUBuiltin<"void(_ExtVector<4, int> address_space<1> *, _ExtVector<4, int> address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_store_async_from_lds_b8 : AMDGPUBuiltin<"void(char address_space<1> *, char address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_store_async_from_lds_b32 : AMDGPUBuiltin<"void(int address_space<1> *, int address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_store_async_from_lds_b64 : AMDGPUBuiltin<"void(_ExtVector<2, int> address_space<1> *, _ExtVector<2, int> address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_global_store_async_from_lds_b128 : AMDGPUBuiltin<"void(_ExtVector<4, int> address_space<1> *, _ExtVector<4, int> address_space<3> *, _Constant int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_ds_atomic_async_barrier_arrive_b64 : AMDGPUBuiltin<"void(long int address_space<3> *)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_ds_atomic_barrier_arrive_rtn_b64 : AMDGPUBuiltin<"long int(long int address_space<3> *, long int)", [Const], "gfx1250-insts">;

def __builtin_amdgcn_tensor_load_to_lds : AMDGPUBuiltin<"void(_ExtVector<4, unsigned int>, _ExtVector<8, int>, _ExtVector<4, int>, _ExtVector<4, int>, _ExtVector<8, int>, _Constant int)", [Const], "gfx1250-insts">;
````
- **L937 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_monitor_b64`.
  **L937 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_monitor_b64`。
- **L938 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_monitor_b128`.
  **L938 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_monitor_b128`。
- **L939 EN**: Declares TableGen def record `__builtin_amdgcn_flat_load_monitor_b32`.
  **L939 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_load_monitor_b32`。
- **L940 EN**: Declares TableGen def record `__builtin_amdgcn_flat_load_monitor_b64`.
  **L940 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_load_monitor_b64`。
- **L941 EN**: Declares TableGen def record `__builtin_amdgcn_flat_load_monitor_b128`.
  **L941 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_flat_load_monitor_b128`。
- **L942 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_load_b32`.
  **L942 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_load_b32`。
- **L943 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_load_b64`.
  **L943 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_load_b64`。
- **L944 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_load_b128`.
  **L944 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_load_b128`。
- **L945 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_load_async_to_lds_b8`.
  **L945 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_load_async_to_lds_b8`。
- **L946 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_load_async_to_lds_b32`.
  **L946 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_load_async_to_lds_b32`。
- **L947 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_load_async_to_lds_b64`.
  **L947 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_load_async_to_lds_b64`。
- **L948 EN**: Declares TableGen def record `__builtin_amdgcn_cluster_load_async_to_lds_b128`.
  **L948 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cluster_load_async_to_lds_b128`。
- **L949 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_async_to_lds_b8`.
  **L949 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_async_to_lds_b8`。
- **L950 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_async_to_lds_b32`.
  **L950 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_async_to_lds_b32`。
- **L951 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_async_to_lds_b64`.
  **L951 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_async_to_lds_b64`。
- **L952 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_async_to_lds_b128`.
  **L952 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_async_to_lds_b128`。
- **L953 EN**: Declares TableGen def record `__builtin_amdgcn_global_store_async_from_lds_b8`.
  **L953 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_store_async_from_lds_b8`。
- **L954 EN**: Declares TableGen def record `__builtin_amdgcn_global_store_async_from_lds_b32`.
  **L954 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_store_async_from_lds_b32`。
- **L955 EN**: Declares TableGen def record `__builtin_amdgcn_global_store_async_from_lds_b64`.
  **L955 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_store_async_from_lds_b64`。
- **L956 EN**: Declares TableGen def record `__builtin_amdgcn_global_store_async_from_lds_b128`.
  **L956 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_store_async_from_lds_b128`。
- **L957 EN**: Declares TableGen def record `__builtin_amdgcn_ds_atomic_async_barrier_arrive_b64`.
  **L957 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_atomic_async_barrier_arrive_b64`。
- **L958 EN**: Declares TableGen def record `__builtin_amdgcn_ds_atomic_barrier_arrive_rtn_b64`.
  **L958 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_atomic_barrier_arrive_rtn_b64`。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Declares TableGen def record `__builtin_amdgcn_tensor_load_to_lds`.
  **L960 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_tensor_load_to_lds`。

### Lines 961-984

````tablegen
def __builtin_amdgcn_tensor_store_from_lds : AMDGPUBuiltin<"void(_ExtVector<4, unsigned int>, _ExtVector<8, int>, _ExtVector<4, int>, _ExtVector<4, int>, _ExtVector<8, int>, _Constant int)", [Const], "gfx1250-insts">;


def __builtin_amdgcn_global_load_tr4_b64_v2i32 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<1> *)", [Const], "transpose-load-f4f6-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr8_b64_v2i32 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<1> *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr6_b96_v3i32 : AMDGPUBuiltin<"_ExtVector<3, int>(_ExtVector<3, int> address_space<1> *)", [Const], "transpose-load-f4f6-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr16_b128_v8i16 : AMDGPUBuiltin<"_ExtVector<8, short>(_ExtVector<8, short> address_space<1> *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr16_b128_v8f16 : AMDGPUBuiltin<"_ExtVector<8, __fp16>(_ExtVector<8, __fp16> address_space<1> *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_global_load_tr16_b128_v8bf16 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_ExtVector<8, __bf16> address_space<1> *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_ds_load_tr4_b64_v2i32 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<3> *)", [Const], "transpose-load-f4f6-insts,wavefrontsize32">;
def __builtin_amdgcn_ds_load_tr8_b64_v2i32 : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> address_space<3> *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_ds_load_tr6_b96_v3i32 : AMDGPUBuiltin<"_ExtVector<3, int>(_ExtVector<3, int> address_space<3> *)", [Const], "transpose-load-f4f6-insts,wavefrontsize32">;
def __builtin_amdgcn_ds_load_tr16_b128_v8i16 : AMDGPUBuiltin<"_ExtVector<8, short>(_ExtVector<8, short> address_space<3> *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_ds_load_tr16_b128_v8f16 : AMDGPUBuiltin<"_ExtVector<8, __fp16>(_ExtVector<8, __fp16> address_space<3> *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_ds_load_tr16_b128_v8bf16 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_ExtVector<8, __bf16> address_space<3> *)", [Const], "gfx1250-insts,wavefrontsize32">;

def __builtin_amdgcn_s_setprio_inc_wg : AMDGPUBuiltin<"void(_Constant short)", [], "setprio-inc-wg-inst">;
def __builtin_amdgcn_s_monitor_sleep : AMDGPUBuiltin<"void(_Constant short)", [], "gfx1250-insts">;
def __builtin_amdgcn_s_wakeup_barrier : AMDGPUBuiltin<"void(void *)", [], "s-wakeup-barrier-inst">;

def __builtin_amdgcn_s_wait_asynccnt : AMDGPUBuiltin<"void(_Constant unsigned short)", [], "gfx1250-insts">;
def __builtin_amdgcn_s_wait_tensorcnt : AMDGPUBuiltin<"void(_Constant unsigned short)", [], "gfx1250-insts">;

def __builtin_amdgcn_tanhf : AMDGPUBuiltin<"float(float)", [Const], "tanh-insts">;
````
- **L961 EN**: Declares TableGen def record `__builtin_amdgcn_tensor_store_from_lds`.
  **L961 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_tensor_store_from_lds`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Blank line separating nearby declarations or logic blocks.
  **L963 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L964 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr4_b64_v2i32`.
  **L964 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr4_b64_v2i32`。
- **L965 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr8_b64_v2i32`.
  **L965 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr8_b64_v2i32`。
- **L966 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr6_b96_v3i32`.
  **L966 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr6_b96_v3i32`。
- **L967 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr16_b128_v8i16`.
  **L967 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr16_b128_v8i16`。
- **L968 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr16_b128_v8f16`.
  **L968 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr16_b128_v8f16`。
- **L969 EN**: Declares TableGen def record `__builtin_amdgcn_global_load_tr16_b128_v8bf16`.
  **L969 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_global_load_tr16_b128_v8bf16`。
- **L970 EN**: Declares TableGen def record `__builtin_amdgcn_ds_load_tr4_b64_v2i32`.
  **L970 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_load_tr4_b64_v2i32`。
- **L971 EN**: Declares TableGen def record `__builtin_amdgcn_ds_load_tr8_b64_v2i32`.
  **L971 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_load_tr8_b64_v2i32`。
- **L972 EN**: Declares TableGen def record `__builtin_amdgcn_ds_load_tr6_b96_v3i32`.
  **L972 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_load_tr6_b96_v3i32`。
- **L973 EN**: Declares TableGen def record `__builtin_amdgcn_ds_load_tr16_b128_v8i16`.
  **L973 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_load_tr16_b128_v8i16`。
- **L974 EN**: Declares TableGen def record `__builtin_amdgcn_ds_load_tr16_b128_v8f16`.
  **L974 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_load_tr16_b128_v8f16`。
- **L975 EN**: Declares TableGen def record `__builtin_amdgcn_ds_load_tr16_b128_v8bf16`.
  **L975 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_ds_load_tr16_b128_v8bf16`。
- **L976 EN**: Blank line separating nearby declarations or logic blocks.
  **L976 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L977 EN**: Declares TableGen def record `__builtin_amdgcn_s_setprio_inc_wg`.
  **L977 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_setprio_inc_wg`。
- **L978 EN**: Declares TableGen def record `__builtin_amdgcn_s_monitor_sleep`.
  **L978 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_monitor_sleep`。
- **L979 EN**: Declares TableGen def record `__builtin_amdgcn_s_wakeup_barrier`.
  **L979 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_wakeup_barrier`。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L981 EN**: Declares TableGen def record `__builtin_amdgcn_s_wait_asynccnt`.
  **L981 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_wait_asynccnt`。
- **L982 EN**: Declares TableGen def record `__builtin_amdgcn_s_wait_tensorcnt`.
  **L982 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_s_wait_tensorcnt`。
- **L983 EN**: Blank line separating nearby declarations or logic blocks.
  **L983 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L984 EN**: Declares TableGen def record `__builtin_amdgcn_tanhf`.
  **L984 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_tanhf`。

### Lines 985-1008

````tablegen
def __builtin_amdgcn_tanhh : AMDGPUBuiltin<"__fp16(__fp16)", [Const], "tanh-insts">;
def __builtin_amdgcn_tanh_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;
def __builtin_amdgcn_rcp_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;
def __builtin_amdgcn_sqrt_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;
def __builtin_amdgcn_rsq_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;
def __builtin_amdgcn_log_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;
def __builtin_amdgcn_exp2_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;
def __builtin_amdgcn_sin_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;
def __builtin_amdgcn_cos_bf16 : AMDGPUBuiltin<"__bf16(__bf16)", [Const], "bf16-trans-insts">;

def __builtin_amdgcn_cvt_sr_pk_bf16_f32 : AMDGPUBuiltin<"_ExtVector<2, __bf16>(float, float, int)", [Const], "bf16-cvt-insts">;
def __builtin_amdgcn_cvt_sr_pk_f16_f32 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(float, float, int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_f16_fp8 : AMDGPUBuiltin<"_Float16(int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_f16_bf8 : AMDGPUBuiltin<"_Float16(int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_pk_f16_fp8 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(short)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_pk_f16_bf8 : AMDGPUBuiltin<"_ExtVector<2, _Float16>(short)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_pk_fp8_f16 : AMDGPUBuiltin<"short(_ExtVector<2, _Float16>)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_pk_bf8_f16 : AMDGPUBuiltin<"short(_ExtVector<2, _Float16>)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_sr_fp8_f16 : AMDGPUBuiltin<"int(_Float16, int, unsigned int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_sr_bf8_f16 : AMDGPUBuiltin<"int(_Float16, int, unsigned int, _Constant int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_f16_fp8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<2, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_bf16_fp8 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_ExtVector<2, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_f16_bf8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<2, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_bf16_bf8 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_ExtVector<2, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
````
- **L985 EN**: Declares TableGen def record `__builtin_amdgcn_tanhh`.
  **L985 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_tanhh`。
- **L986 EN**: Declares TableGen def record `__builtin_amdgcn_tanh_bf16`.
  **L986 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_tanh_bf16`。
- **L987 EN**: Declares TableGen def record `__builtin_amdgcn_rcp_bf16`.
  **L987 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rcp_bf16`。
- **L988 EN**: Declares TableGen def record `__builtin_amdgcn_sqrt_bf16`.
  **L988 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sqrt_bf16`。
- **L989 EN**: Declares TableGen def record `__builtin_amdgcn_rsq_bf16`.
  **L989 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_rsq_bf16`。
- **L990 EN**: Declares TableGen def record `__builtin_amdgcn_log_bf16`.
  **L990 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_log_bf16`。
- **L991 EN**: Declares TableGen def record `__builtin_amdgcn_exp2_bf16`.
  **L991 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_exp2_bf16`。
- **L992 EN**: Declares TableGen def record `__builtin_amdgcn_sin_bf16`.
  **L992 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sin_bf16`。
- **L993 EN**: Declares TableGen def record `__builtin_amdgcn_cos_bf16`.
  **L993 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cos_bf16`。
- **L994 EN**: Blank line separating nearby declarations or logic blocks.
  **L994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L995 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_pk_bf16_f32`.
  **L995 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_pk_bf16_f32`。
- **L996 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_pk_f16_f32`.
  **L996 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_pk_f16_f32`。
- **L997 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_f16_fp8`.
  **L997 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_f16_fp8`。
- **L998 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_f16_bf8`.
  **L998 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_f16_bf8`。
- **L999 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_f16_fp8`.
  **L999 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_f16_fp8`。
- **L1000 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_f16_bf8`.
  **L1000 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_f16_bf8`。
- **L1001 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_fp8_f16`.
  **L1001 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_fp8_f16`。
- **L1002 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_bf8_f16`.
  **L1002 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_bf8_f16`。
- **L1003 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_fp8_f16`.
  **L1003 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_fp8_f16`。
- **L1004 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_bf8_f16`.
  **L1004 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_bf8_f16`。
- **L1005 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_f16_fp8`.
  **L1005 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_f16_fp8`。
- **L1006 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_bf16_fp8`.
  **L1006 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_bf16_fp8`。
- **L1007 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_f16_bf8`.
  **L1007 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_f16_bf8`。
- **L1008 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_bf16_bf8`.
  **L1008 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_bf16_bf8`。

### Lines 1009-1032

````tablegen
def __builtin_amdgcn_cvt_scale_pk8_f16_fp4 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(unsigned int, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_bf16_fp4 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(unsigned int, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_f32_fp8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_f32_bf8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<2, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk8_f32_fp4 : AMDGPUBuiltin<"_ExtVector<8, float>(unsigned int, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk16_f16_fp6 : AMDGPUBuiltin<"_ExtVector<16, _Float16>(_ExtVector<3, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk16_bf16_fp6 : AMDGPUBuiltin<"_ExtVector<16, __bf16>(_ExtVector<3, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk16_f16_bf6 : AMDGPUBuiltin<"_ExtVector<16, _Float16>(_ExtVector<3, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk16_bf16_bf6 : AMDGPUBuiltin<"_ExtVector<16, __bf16>(_ExtVector<3, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk16_f32_fp6 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<3, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scale_pk16_f32_bf6 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<3, unsigned int>, unsigned int, _Constant unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_fp8_bf16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, __bf16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_bf8_bf16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, __bf16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_fp8_f16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, _Float16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_bf8_f16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, _Float16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_fp8_f32 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, float>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_bf8_f32 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, float>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_fp4_f32 : AMDGPUBuiltin<"unsigned int(_ExtVector<8, float>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_fp4_f16 : AMDGPUBuiltin<"unsigned int(_ExtVector<8, _Float16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk8_fp4_bf16 : AMDGPUBuiltin<"unsigned int(_ExtVector<8, __bf16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk16_fp6_f32 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, float>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk16_bf6_f32 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, float>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk16_fp6_f16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, _Float16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk16_bf6_f16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, _Float16>, float)", [Const], "gfx1250-insts">;
````
- **L1009 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_f16_fp4`.
  **L1009 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_f16_fp4`。
- **L1010 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_bf16_fp4`.
  **L1010 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_bf16_fp4`。
- **L1011 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_f32_fp8`.
  **L1011 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_f32_fp8`。
- **L1012 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_f32_bf8`.
  **L1012 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_f32_bf8`。
- **L1013 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk8_f32_fp4`.
  **L1013 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk8_f32_fp4`。
- **L1014 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk16_f16_fp6`.
  **L1014 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk16_f16_fp6`。
- **L1015 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk16_bf16_fp6`.
  **L1015 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk16_bf16_fp6`。
- **L1016 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk16_f16_bf6`.
  **L1016 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk16_f16_bf6`。
- **L1017 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk16_bf16_bf6`.
  **L1017 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk16_bf16_bf6`。
- **L1018 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk16_f32_fp6`.
  **L1018 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk16_f32_fp6`。
- **L1019 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scale_pk16_f32_bf6`.
  **L1019 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scale_pk16_f32_bf6`。
- **L1020 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_fp8_bf16`.
  **L1020 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_fp8_bf16`。
- **L1021 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_bf8_bf16`.
  **L1021 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_bf8_bf16`。
- **L1022 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_fp8_f16`.
  **L1022 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_fp8_f16`。
- **L1023 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_bf8_f16`.
  **L1023 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_bf8_f16`。
- **L1024 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_fp8_f32`.
  **L1024 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_fp8_f32`。
- **L1025 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_bf8_f32`.
  **L1025 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_bf8_f32`。
- **L1026 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_fp4_f32`.
  **L1026 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_fp4_f32`。
- **L1027 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_fp4_f16`.
  **L1027 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_fp4_f16`。
- **L1028 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk8_fp4_bf16`.
  **L1028 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk8_fp4_bf16`。
- **L1029 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk16_fp6_f32`.
  **L1029 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk16_fp6_f32`。
- **L1030 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk16_bf6_f32`.
  **L1030 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk16_bf6_f32`。
- **L1031 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk16_fp6_f16`.
  **L1031 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk16_fp6_f16`。
- **L1032 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk16_bf6_f16`.
  **L1032 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk16_bf6_f16`。

### Lines 1033-1056

````tablegen
def __builtin_amdgcn_cvt_scalef32_pk16_fp6_bf16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, __bf16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_pk16_bf6_bf16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, __bf16>, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_bf16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, __bf16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_bf16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, __bf16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_f16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, _Float16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_f16 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, _Float16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_f32 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, float>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_f32 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<8, float>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_f32 : AMDGPUBuiltin<"unsigned int(_ExtVector<8, float>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_f16 : AMDGPUBuiltin<"unsigned int(_ExtVector<8, _Float16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_bf16 : AMDGPUBuiltin<"unsigned int(_ExtVector<8, __bf16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_bf16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, __bf16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_f16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, _Float16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_f32 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, float>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_bf16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, __bf16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_f16 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, _Float16>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_f32 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(_ExtVector<16, float>, unsigned int, float)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_cvt_pk_fp8_f32_e5m3 : AMDGPUBuiltin<"int(float, float, int, _Constant bool)", [Const], "fp8e5m3-insts">;
def __builtin_amdgcn_cvt_sr_fp8_f32_e5m3 : AMDGPUBuiltin<"int(float, int, int, _Constant int)", [Const], "fp8e5m3-insts">;
def __builtin_amdgcn_sat_pk4_i4_i8 : AMDGPUBuiltin<"unsigned short(unsigned int)", [Const], "gfx1250-insts">;
def __builtin_amdgcn_sat_pk4_u4_u8 : AMDGPUBuiltin<"unsigned short(unsigned int)", [Const], "gfx1250-insts">;

def __builtin_amdgcn_permlane_bcast : AMDGPUBuiltin<"int(int, int, int)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_permlane_up : AMDGPUBuiltin<"int(int, int, int)", [Const], "gfx1250-insts,wavefrontsize32">;
````
- **L1033 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk16_fp6_bf16`.
  **L1033 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk16_fp6_bf16`。
- **L1034 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_pk16_bf6_bf16`.
  **L1034 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_pk16_bf6_bf16`。
- **L1035 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_bf16`.
  **L1035 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_bf16`。
- **L1036 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_bf16`.
  **L1036 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_bf16`。
- **L1037 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_f16`.
  **L1037 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_f16`。
- **L1038 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_f16`.
  **L1038 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_f16`。
- **L1039 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_f32`.
  **L1039 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp8_f32`。
- **L1040 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_f32`.
  **L1040 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_bf8_f32`。
- **L1041 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_f32`.
  **L1041 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_f32`。
- **L1042 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_f16`.
  **L1042 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_f16`。
- **L1043 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_bf16`.
  **L1043 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk8_fp4_bf16`。
- **L1044 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_bf16`.
  **L1044 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_bf16`。
- **L1045 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_f16`.
  **L1045 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_f16`。
- **L1046 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_f32`.
  **L1046 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk16_bf6_f32`。
- **L1047 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_bf16`.
  **L1047 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_bf16`。
- **L1048 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_f16`.
  **L1048 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_f16`。
- **L1049 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_f32`.
  **L1049 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_scalef32_sr_pk16_fp6_f32`。
- **L1050 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_pk_fp8_f32_e5m3`.
  **L1050 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_pk_fp8_f32_e5m3`。
- **L1051 EN**: Declares TableGen def record `__builtin_amdgcn_cvt_sr_fp8_f32_e5m3`.
  **L1051 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cvt_sr_fp8_f32_e5m3`。
- **L1052 EN**: Declares TableGen def record `__builtin_amdgcn_sat_pk4_i4_i8`.
  **L1052 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sat_pk4_i4_i8`。
- **L1053 EN**: Declares TableGen def record `__builtin_amdgcn_sat_pk4_u4_u8`.
  **L1053 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_sat_pk4_u4_u8`。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1055 EN**: Declares TableGen def record `__builtin_amdgcn_permlane_bcast`.
  **L1055 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane_bcast`。
- **L1056 EN**: Declares TableGen def record `__builtin_amdgcn_permlane_up`.
  **L1056 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane_up`。

### Lines 1057-1080

````tablegen
def __builtin_amdgcn_permlane_down : AMDGPUBuiltin<"int(int, int, int)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_permlane_xor : AMDGPUBuiltin<"int(int, int, int)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_permlane_idx_gen : AMDGPUBuiltin<"int(int, int)", [Const], "gfx1250-insts,wavefrontsize32">;

def __builtin_amdgcn_perm_pk16_b4_u4 : AMDGPUBuiltin<"_ExtVector<2, unsigned int>(unsigned int, unsigned int, _ExtVector<2, unsigned int>)", [Const], "tensor-cvt-lut-insts">;
def __builtin_amdgcn_perm_pk16_b6_u4 : AMDGPUBuiltin<"_ExtVector<3, unsigned int>(unsigned int, unsigned long int, _ExtVector<2, unsigned int>)", [Const], "tensor-cvt-lut-insts">;
def __builtin_amdgcn_perm_pk16_b8_u4 : AMDGPUBuiltin<"_ExtVector<4, unsigned int>(unsigned long int, unsigned long int, _ExtVector<2, unsigned int>)", [Const], "tensor-cvt-lut-insts">;

def __builtin_amdgcn_add_max_i32 : AMDGPUBuiltin<"int(int, int, int, _Constant bool)", [Const], "add-min-max-insts">;
def __builtin_amdgcn_add_max_u32 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, _Constant bool)", [Const], "add-min-max-insts">;
def __builtin_amdgcn_add_min_i32 : AMDGPUBuiltin<"int(int, int, int, _Constant bool)", [Const], "add-min-max-insts">;
def __builtin_amdgcn_add_min_u32 : AMDGPUBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int, _Constant bool)", [Const], "add-min-max-insts">;
def __builtin_amdgcn_pk_add_max_i16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, _ExtVector<2, short>, _ExtVector<2, short>, _Constant bool)", [Const], "pk-add-min-max-insts">;
def __builtin_amdgcn_pk_add_max_u16 : AMDGPUBuiltin<"_ExtVector<2, unsigned short>(_ExtVector<2, unsigned short>, _ExtVector<2, unsigned short>, _ExtVector<2, unsigned short>, _Constant bool)", [Const], "pk-add-min-max-insts">;
def __builtin_amdgcn_pk_add_min_i16 : AMDGPUBuiltin<"_ExtVector<2, short>(_ExtVector<2, short>, _ExtVector<2, short>, _ExtVector<2, short>, _Constant bool)", [Const], "pk-add-min-max-insts">;
def __builtin_amdgcn_pk_add_min_u16 : AMDGPUBuiltin<"_ExtVector<2, unsigned short>(_ExtVector<2, unsigned short>, _ExtVector<2, unsigned short>, _ExtVector<2, unsigned short>, _Constant bool)", [Const], "pk-add-min-max-insts">;

// GFX1250 WMMA builtins
def __builtin_amdgcn_wmma_f32_16x16x4_f32 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant bool, _ExtVector<2, float>, _Constant bool, _ExtVector<2, float>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_f32_16x16x4_f32_GFX1250];
  let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x32_bf16 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant bool, _ExtVector<16, __bf16>, _Constant bool, _ExtVector<16, __bf16>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_f32_16x16x32_GFX1250];
````
- **L1057 EN**: Declares TableGen def record `__builtin_amdgcn_permlane_down`.
  **L1057 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane_down`。
- **L1058 EN**: Declares TableGen def record `__builtin_amdgcn_permlane_xor`.
  **L1058 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane_xor`。
- **L1059 EN**: Declares TableGen def record `__builtin_amdgcn_permlane_idx_gen`.
  **L1059 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_permlane_idx_gen`。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1061 EN**: Declares TableGen def record `__builtin_amdgcn_perm_pk16_b4_u4`.
  **L1061 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_perm_pk16_b4_u4`。
- **L1062 EN**: Declares TableGen def record `__builtin_amdgcn_perm_pk16_b6_u4`.
  **L1062 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_perm_pk16_b6_u4`。
- **L1063 EN**: Declares TableGen def record `__builtin_amdgcn_perm_pk16_b8_u4`.
  **L1063 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_perm_pk16_b8_u4`。
- **L1064 EN**: Blank line separating nearby declarations or logic blocks.
  **L1064 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1065 EN**: Declares TableGen def record `__builtin_amdgcn_add_max_i32`.
  **L1065 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_add_max_i32`。
- **L1066 EN**: Declares TableGen def record `__builtin_amdgcn_add_max_u32`.
  **L1066 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_add_max_u32`。
- **L1067 EN**: Declares TableGen def record `__builtin_amdgcn_add_min_i32`.
  **L1067 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_add_min_i32`。
- **L1068 EN**: Declares TableGen def record `__builtin_amdgcn_add_min_u32`.
  **L1068 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_add_min_u32`。
- **L1069 EN**: Declares TableGen def record `__builtin_amdgcn_pk_add_max_i16`.
  **L1069 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_pk_add_max_i16`。
- **L1070 EN**: Declares TableGen def record `__builtin_amdgcn_pk_add_max_u16`.
  **L1070 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_pk_add_max_u16`。
- **L1071 EN**: Declares TableGen def record `__builtin_amdgcn_pk_add_min_i16`.
  **L1071 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_pk_add_min_i16`。
- **L1072 EN**: Declares TableGen def record `__builtin_amdgcn_pk_add_min_u16`.
  **L1072 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_pk_add_min_u16`。
- **L1073 EN**: Blank line separating nearby declarations or logic blocks.
  **L1073 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1074 EN**: Comment explains nearby logic, constraints, or intent: `GFX1250 WMMA builtins`.
  **L1074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX1250 WMMA builtins`。
- **L1075 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x4_f32`.
  **L1075 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x4_f32`。
- **L1076 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_f32_16x16x4_f32_GFX1250];`.
  **L1076 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_f32_16x16x4_f32_GFX1250];`。
- **L1077 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1077 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1078 EN**: Closes the current lexical scope or compound statement.
  **L1078 CN**: 结束当前词法作用域或复合语句块。
- **L1079 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x32_bf16`.
  **L1079 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x32_bf16`。
- **L1080 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_f32_16x16x32_GFX1250];`.
  **L1080 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_f32_16x16x32_GFX1250];`。

### Lines 1081-1104

````tablegen
  let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_bf16_16x16x32_bf16 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_Constant bool, _ExtVector<16, __bf16>, _Constant bool, _ExtVector<16, __bf16>, _Constant short, _ExtVector<8, __bf16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_half_16x16x32_GFX1250];
  let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_bf16f32_16x16x32_bf16 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_Constant bool, _ExtVector<16, __bf16>, _Constant bool, _ExtVector<16, __bf16>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_bf16f32_16x16x32_GFX1250];
  let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x64_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x64_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x64_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x64_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
````
- **L1081 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1081 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1082 EN**: Closes the current lexical scope or compound statement.
  **L1082 CN**: 结束当前词法作用域或复合语句块。
- **L1083 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16_16x16x32_bf16`.
  **L1083 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16_16x16x32_bf16`。
- **L1084 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_half_16x16x32_GFX1250];`.
  **L1084 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_half_16x16x32_GFX1250];`。
- **L1085 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1085 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1086 EN**: Closes the current lexical scope or compound statement.
  **L1086 CN**: 结束当前词法作用域或复合语句块。
- **L1087 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_bf16f32_16x16x32_bf16`.
  **L1087 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_bf16f32_16x16x32_bf16`。
- **L1088 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_bf16f32_16x16x32_GFX1250];`.
  **L1088 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_bf16f32_16x16x32_GFX1250];`。
- **L1089 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1089 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1090 EN**: Closes the current lexical scope or compound statement.
  **L1090 CN**: 结束当前词法作用域或复合语句块。
- **L1091 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x64_fp8_fp8`.
  **L1091 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x64_fp8_fp8`。
- **L1092 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1092 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。
- **L1093 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1093 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x64_fp8_bf8`.
  **L1095 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x64_fp8_bf8`。
- **L1096 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1096 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。
- **L1097 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1097 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1098 EN**: Closes the current lexical scope or compound statement.
  **L1098 CN**: 结束当前词法作用域或复合语句块。
- **L1099 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x64_bf8_fp8`.
  **L1099 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x64_bf8_fp8`。
- **L1100 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1100 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。
- **L1101 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1101 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x64_bf8_bf8`.
  **L1103 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x64_bf8_bf8`。
- **L1104 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1104 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。

### Lines 1105-1128

````tablegen
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x64_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x64_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x64_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x64_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<8, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_i32_16x16x64_iu8 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<8, int>, _Constant bool, _ExtVector<8, int>, _ExtVector<8, int>, _Constant bool, _Constant bool, ...)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_i32_16x16x64_iu8_GFX1250];
  let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x128_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
````
- **L1105 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1105 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1106 EN**: Closes the current lexical scope or compound statement.
  **L1106 CN**: 结束当前词法作用域或复合语句块。
- **L1107 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x64_fp8_fp8`.
  **L1107 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x64_fp8_fp8`。
- **L1108 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1108 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。
- **L1109 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1109 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1110 EN**: Closes the current lexical scope or compound statement.
  **L1110 CN**: 结束当前词法作用域或复合语句块。
- **L1111 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x64_fp8_bf8`.
  **L1111 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x64_fp8_bf8`。
- **L1112 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1112 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。
- **L1113 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1113 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1114 EN**: Closes the current lexical scope or compound statement.
  **L1114 CN**: 结束当前词法作用域或复合语句块。
- **L1115 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x64_bf8_fp8`.
  **L1115 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x64_bf8_fp8`。
- **L1116 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1116 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。
- **L1117 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1117 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1118 EN**: Closes the current lexical scope or compound statement.
  **L1118 CN**: 结束当前词法作用域或复合语句块。
- **L1119 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x64_bf8_bf8`.
  **L1119 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x64_bf8_bf8`。
- **L1120 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`.
  **L1120 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x64_GFX1250];`。
- **L1121 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1121 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1122 EN**: Closes the current lexical scope or compound statement.
  **L1122 CN**: 结束当前词法作用域或复合语句块。
- **L1123 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_i32_16x16x64_iu8`.
  **L1123 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_i32_16x16x64_iu8`。
- **L1124 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_i32_16x16x64_iu8_GFX1250];`.
  **L1124 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_i32_16x16x64_iu8_GFX1250];`。
- **L1125 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1125 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_sign", "a", "b_sign", "b", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1126 EN**: Closes the current lexical scope or compound statement.
  **L1126 CN**: 结束当前词法作用域或复合语句块。
- **L1127 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x128_fp8_fp8`.
  **L1127 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x128_fp8_fp8`。
- **L1128 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1128 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。

### Lines 1129-1152

````tablegen
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x128_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x128_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x128_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x128_f8f6f4 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant int, _ExtVector<16, int>, _Constant int, _ExtVector<16, int>, _Constant short, _ExtVector<8, float>)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_f8f6f4_GFX1250];
  let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c"];
}
def __builtin_amdgcn_wmma_f32_16x16x128_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x128_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
````
- **L1129 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1129 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1130 EN**: Closes the current lexical scope or compound statement.
  **L1130 CN**: 结束当前词法作用域或复合语句块。
- **L1131 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x128_fp8_bf8`.
  **L1131 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x128_fp8_bf8`。
- **L1132 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1132 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。
- **L1133 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1133 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1134 EN**: Closes the current lexical scope or compound statement.
  **L1134 CN**: 结束当前词法作用域或复合语句块。
- **L1135 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x128_bf8_fp8`.
  **L1135 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x128_bf8_fp8`。
- **L1136 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1136 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。
- **L1137 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1137 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1138 EN**: Closes the current lexical scope or compound statement.
  **L1138 CN**: 结束当前词法作用域或复合语句块。
- **L1139 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x128_bf8_bf8`.
  **L1139 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x128_bf8_bf8`。
- **L1140 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1140 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。
- **L1141 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1141 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1142 EN**: Closes the current lexical scope or compound statement.
  **L1142 CN**: 结束当前词法作用域或复合语句块。
- **L1143 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x128_f8f6f4`.
  **L1143 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x128_f8f6f4`。
- **L1144 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_f8f6f4_GFX1250];`.
  **L1144 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_f8f6f4_GFX1250];`。
- **L1145 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c"];`.
  **L1145 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c"];`。
- **L1146 EN**: Closes the current lexical scope or compound statement.
  **L1146 CN**: 结束当前词法作用域或复合语句块。
- **L1147 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x128_fp8_fp8`.
  **L1147 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x128_fp8_fp8`。
- **L1148 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1148 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。
- **L1149 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1149 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1150 EN**: Closes the current lexical scope or compound statement.
  **L1150 CN**: 结束当前词法作用域或复合语句块。
- **L1151 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x128_fp8_bf8`.
  **L1151 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x128_fp8_bf8`。
- **L1152 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1152 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。

### Lines 1153-1176

````tablegen
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x128_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x128_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<16, int>, _ExtVector<16, int>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_scale_f32_16x16x128_f8f6f4 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant int, _ExtVector<16, int>, _Constant int, _ExtVector<16, int>, _Constant short, _ExtVector<8, float>, _Constant int, _Constant int, int, _Constant int, _Constant int, int, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_scale_GFX1250];
  let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant int, _ExtVector<16, int>, _Constant int, _ExtVector<16, int>, _Constant short, _ExtVector<8, float>, _Constant int, _Constant int, long int, _Constant int, _Constant int, long int, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_scale16_GFX1250];
  let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_16x16x32_f16 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant bool, _ExtVector<16, _Float16>, _Constant bool, _ExtVector<16, _Float16>, _Constant short, _ExtVector<8, float>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_f32_16x16x32_GFX1250];
  let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f16_16x16x32_f16 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_Constant bool, _ExtVector<16, _Float16>, _Constant bool, _ExtVector<16, _Float16>, _Constant short, _ExtVector<8, _Float16>, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_half_16x16x32_GFX1250];
````
- **L1153 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1153 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1154 EN**: Closes the current lexical scope or compound statement.
  **L1154 CN**: 结束当前词法作用域或复合语句块。
- **L1155 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x128_bf8_fp8`.
  **L1155 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x128_bf8_fp8`。
- **L1156 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1156 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。
- **L1157 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1157 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1158 EN**: Closes the current lexical scope or compound statement.
  **L1158 CN**: 结束当前词法作用域或复合语句块。
- **L1159 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x128_bf8_bf8`.
  **L1159 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x128_bf8_bf8`。
- **L1160 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`.
  **L1160 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_fp8_16x16x128_GFX1250];`。
- **L1161 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1161 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1162 EN**: Closes the current lexical scope or compound statement.
  **L1162 CN**: 结束当前词法作用域或复合语句块。
- **L1163 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_scale_f32_16x16x128_f8f6f4`.
  **L1163 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_scale_f32_16x16x128_f8f6f4`。
- **L1164 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_scale_GFX1250];`.
  **L1164 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_scale_GFX1250];`。
- **L1165 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1165 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1166 EN**: Closes the current lexical scope or compound statement.
  **L1166 CN**: 结束当前词法作用域或复合语句块。
- **L1167 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4`.
  **L1167 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_scale16_f32_16x16x128_f8f6f4`。
- **L1168 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_scale16_GFX1250];`.
  **L1168 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_scale16_GFX1250];`。
- **L1169 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1169 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["matrix_a_fmt", "a", "matrix_b_fmt", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1170 EN**: Closes the current lexical scope or compound statement.
  **L1170 CN**: 结束当前词法作用域或复合语句块。
- **L1171 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_16x16x32_f16`.
  **L1171 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_16x16x32_f16`。
- **L1172 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_f32_16x16x32_GFX1250];`.
  **L1172 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_f32_16x16x32_GFX1250];`。
- **L1173 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1173 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1174 EN**: Closes the current lexical scope or compound statement.
  **L1174 CN**: 结束当前词法作用域或复合语句块。
- **L1175 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f16_16x16x32_f16`.
  **L1175 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f16_16x16x32_f16`。
- **L1176 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_half_16x16x32_GFX1250];`.
  **L1176 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_half_16x16x32_GFX1250];`。

### Lines 1177-1200

````tablegen
  let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_f32_32x16x128_f4 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<16, int>, _ExtVector<8, int>, _Constant short, _ExtVector<16, float>)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_f4_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c"];
}
def __builtin_amdgcn_wmma_scale_f32_32x16x128_f4 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<16, int>, _ExtVector<8, int>, _Constant short, _ExtVector<16, float>, _Constant int, _Constant int, int, _Constant int, _Constant int, int, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_scale_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_wmma_scale16_f32_32x16x128_f4 : AMDGPUBuiltin<"_ExtVector<16, float>(_ExtVector<16, int>, _ExtVector<8, int>, _Constant short, _ExtVector<16, float>, _Constant int, _Constant int, long int, _Constant int, _Constant int, long int, _Constant bool, _Constant bool)", [Const], "gfx1250-insts,wavefrontsize32"> {
  let Documentation = [DocWMMA_scale16_GFX1250];
  let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];
}
def __builtin_amdgcn_swmmac_f32_16x16x64_bf16 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant bool, _ExtVector<16, __bf16>, _Constant bool, _ExtVector<32, __bf16>, _ExtVector<8, float>, int, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_bf16_16x16x64_bf16 : AMDGPUBuiltin<"_ExtVector<8, __bf16>(_Constant bool, _ExtVector<16, __bf16>, _Constant bool, _ExtVector<32, __bf16>, _ExtVector<8, __bf16>, int, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_bf16f32_16x16x64_bf16 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant bool, _ExtVector<16, __bf16>, _Constant bool, _ExtVector<32, __bf16>, _ExtVector<8, float>, int, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x128_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, float>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x128_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, float>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x128_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, float>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x128_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<8, float>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, float>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f16_16x16x128_fp8_fp8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, _Float16>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f16_16x16x128_fp8_bf8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, _Float16>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f16_16x16x128_bf8_fp8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, _Float16>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
````
- **L1177 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1177 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a_neg", "a", "b_neg", "b", "c_mod", "c", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1178 EN**: Closes the current lexical scope or compound statement.
  **L1178 CN**: 结束当前词法作用域或复合语句块。
- **L1179 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_f32_32x16x128_f4`.
  **L1179 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_f32_32x16x128_f4`。
- **L1180 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_f4_GFX1250];`.
  **L1180 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_f4_GFX1250];`。
- **L1181 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c"];`.
  **L1181 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c"];`。
- **L1182 EN**: Closes the current lexical scope or compound statement.
  **L1182 CN**: 结束当前词法作用域或复合语句块。
- **L1183 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_scale_f32_32x16x128_f4`.
  **L1183 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_scale_f32_32x16x128_f4`。
- **L1184 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_scale_GFX1250];`.
  **L1184 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_scale_GFX1250];`。
- **L1185 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1185 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1186 EN**: Closes the current lexical scope or compound statement.
  **L1186 CN**: 结束当前词法作用域或复合语句块。
- **L1187 EN**: Declares TableGen def record `__builtin_amdgcn_wmma_scale16_f32_32x16x128_f4`.
  **L1187 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_wmma_scale16_f32_32x16x128_f4`。
- **L1188 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Documentation = [DocWMMA_scale16_GFX1250];`.
  **L1188 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Documentation = [DocWMMA_scale16_GFX1250];`。
- **L1189 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`.
  **L1189 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ArgNames = ["a", "b", "c_mod", "c", "matrix_a_scale", "matrix_a_scale_fmt", "matrix_a_scale_exp", "matrix_b_scale", "matrix_b_scale_fmt", "matrix_b_scale_exp", "matrix_a_reuse", "matrix_b_reuse"];`。
- **L1190 EN**: Closes the current lexical scope or compound statement.
  **L1190 CN**: 结束当前词法作用域或复合语句块。
- **L1191 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x64_bf16`.
  **L1191 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x64_bf16`。
- **L1192 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_bf16_16x16x64_bf16`.
  **L1192 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_bf16_16x16x64_bf16`。
- **L1193 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_bf16f32_16x16x64_bf16`.
  **L1193 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_bf16f32_16x16x64_bf16`。
- **L1194 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x128_fp8_fp8`.
  **L1194 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x128_fp8_fp8`。
- **L1195 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x128_fp8_bf8`.
  **L1195 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x128_fp8_bf8`。
- **L1196 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x128_bf8_fp8`.
  **L1196 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x128_bf8_fp8`。
- **L1197 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x128_bf8_bf8`.
  **L1197 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x128_bf8_bf8`。
- **L1198 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f16_16x16x128_fp8_fp8`.
  **L1198 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f16_16x16x128_fp8_fp8`。
- **L1199 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f16_16x16x128_fp8_bf8`.
  **L1199 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f16_16x16x128_fp8_bf8`。
- **L1200 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f16_16x16x128_bf8_fp8`.
  **L1200 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f16_16x16x128_bf8_fp8`。

### Lines 1201-1224

````tablegen
def __builtin_amdgcn_swmmac_f16_16x16x128_bf8_bf8 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_ExtVector<8, int>, _ExtVector<16, int>, _ExtVector<8, _Float16>, _ExtVector<2, int>, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_i32_16x16x128_iu8 : AMDGPUBuiltin<"_ExtVector<8, int>(_Constant bool, _ExtVector<8, int>, _Constant bool, _ExtVector<16, int>, _ExtVector<8, int>, _ExtVector<2, int>, _Constant bool, _Constant bool, ...)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f32_16x16x64_f16 : AMDGPUBuiltin<"_ExtVector<8, float>(_Constant bool, _ExtVector<16, _Float16>, _Constant bool, _ExtVector<32, _Float16>, _ExtVector<8, float>, int, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_swmmac_f16_16x16x64_f16 : AMDGPUBuiltin<"_ExtVector<8, _Float16>(_Constant bool, _ExtVector<16, _Float16>, _Constant bool, _ExtVector<32, _Float16>, _ExtVector<8, _Float16>, int, _Constant bool, _Constant bool)", [Const], "swmmac-gfx1250-insts,wavefrontsize32">;

// GFX12.5 128B cooperative atomics
def __builtin_amdgcn_cooperative_atomic_load_32x4B : AMDGPUBuiltin<"int(int *, _Constant int, char const *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_cooperative_atomic_store_32x4B : AMDGPUBuiltin<"void(int *, int, _Constant int, char const *)", [Const], "gfx1250-insts,wavefrontsize32">;

def __builtin_amdgcn_cooperative_atomic_load_16x8B : AMDGPUBuiltin<"_ExtVector<2, int>(_ExtVector<2, int> *, _Constant int, char const *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_cooperative_atomic_store_16x8B : AMDGPUBuiltin<"void(_ExtVector<2, int> *, _ExtVector<2, int>, _Constant int, char const *)", [Const], "gfx1250-insts,wavefrontsize32">;

def __builtin_amdgcn_cooperative_atomic_load_8x16B : AMDGPUBuiltin<"_ExtVector<4, int>(_ExtVector<4, int> *, _Constant int, char const *)", [Const], "gfx1250-insts,wavefrontsize32">;
def __builtin_amdgcn_cooperative_atomic_store_8x16B : AMDGPUBuiltin<"void(_ExtVector<4, int> *, _ExtVector<4, int>, _Constant int, char const *)", [Const], "gfx1250-insts,wavefrontsize32">;

//===----------------------------------------------------------------------===//
// Image builtins
//===----------------------------------------------------------------------===//
def __builtin_amdgcn_image_load_1d_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_1d_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_1darray_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_1darray_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_2d_f32_i32 : AMDGPUBuiltin<"float(int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_2d_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
````
- **L1201 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f16_16x16x128_bf8_bf8`.
  **L1201 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f16_16x16x128_bf8_bf8`。
- **L1202 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_i32_16x16x128_iu8`.
  **L1202 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_i32_16x16x128_iu8`。
- **L1203 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f32_16x16x64_f16`.
  **L1203 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f32_16x16x64_f16`。
- **L1204 EN**: Declares TableGen def record `__builtin_amdgcn_swmmac_f16_16x16x64_f16`.
  **L1204 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_swmmac_f16_16x16x64_f16`。
- **L1205 EN**: Blank line separating nearby declarations or logic blocks.
  **L1205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1206 EN**: Comment explains nearby logic, constraints, or intent: `GFX12.5 128B cooperative atomics`.
  **L1206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`GFX12.5 128B cooperative atomics`。
- **L1207 EN**: Declares TableGen def record `__builtin_amdgcn_cooperative_atomic_load_32x4B`.
  **L1207 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cooperative_atomic_load_32x4B`。
- **L1208 EN**: Declares TableGen def record `__builtin_amdgcn_cooperative_atomic_store_32x4B`.
  **L1208 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cooperative_atomic_store_32x4B`。
- **L1209 EN**: Blank line separating nearby declarations or logic blocks.
  **L1209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1210 EN**: Declares TableGen def record `__builtin_amdgcn_cooperative_atomic_load_16x8B`.
  **L1210 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cooperative_atomic_load_16x8B`。
- **L1211 EN**: Declares TableGen def record `__builtin_amdgcn_cooperative_atomic_store_16x8B`.
  **L1211 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cooperative_atomic_store_16x8B`。
- **L1212 EN**: Blank line separating nearby declarations or logic blocks.
  **L1212 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1213 EN**: Declares TableGen def record `__builtin_amdgcn_cooperative_atomic_load_8x16B`.
  **L1213 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cooperative_atomic_load_8x16B`。
- **L1214 EN**: Declares TableGen def record `__builtin_amdgcn_cooperative_atomic_store_8x16B`.
  **L1214 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_cooperative_atomic_store_8x16B`。
- **L1215 EN**: Blank line separating nearby declarations or logic blocks.
  **L1215 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1216 EN**: Banner comment marking a file or section boundary.
  **L1216 CN**: 横幅注释，用于标记文件或章节边界。
- **L1217 EN**: Comment explains nearby logic, constraints, or intent: `Image builtins`.
  **L1217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Image builtins`。
- **L1218 EN**: Banner comment marking a file or section boundary.
  **L1218 CN**: 横幅注释，用于标记文件或章节边界。
- **L1219 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_1d_v4f32_i32`.
  **L1219 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_1d_v4f32_i32`。
- **L1220 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_1d_v4f16_i32`.
  **L1220 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_1d_v4f16_i32`。
- **L1221 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_1darray_v4f32_i32`.
  **L1221 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_1darray_v4f32_i32`。
- **L1222 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_1darray_v4f16_i32`.
  **L1222 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_1darray_v4f16_i32`。
- **L1223 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_2d_f32_i32`.
  **L1223 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_2d_f32_i32`。
- **L1224 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_2d_v4f32_i32`.
  **L1224 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_2d_v4f32_i32`。

### Lines 1225-1248

````tablegen
def __builtin_amdgcn_image_load_2d_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_2darray_f32_i32 : AMDGPUBuiltin<"float(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_2darray_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_2darray_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_3d_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_3d_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_cube_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_cube_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_1d_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_1d_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_1darray_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_1darray_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_2d_f32_i32 : AMDGPUBuiltin<"float(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_2d_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_2d_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_2darray_f32_i32 : AMDGPUBuiltin<"float(int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_2darray_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_2darray_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_3d_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_3d_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_cube_v4f32_i32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_load_mip_cube_v4f16_i32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_1d_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_1d_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
````
- **L1225 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_2d_v4f16_i32`.
  **L1225 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_2d_v4f16_i32`。
- **L1226 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_2darray_f32_i32`.
  **L1226 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_2darray_f32_i32`。
- **L1227 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_2darray_v4f32_i32`.
  **L1227 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_2darray_v4f32_i32`。
- **L1228 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_2darray_v4f16_i32`.
  **L1228 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_2darray_v4f16_i32`。
- **L1229 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_3d_v4f32_i32`.
  **L1229 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_3d_v4f32_i32`。
- **L1230 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_3d_v4f16_i32`.
  **L1230 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_3d_v4f16_i32`。
- **L1231 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_cube_v4f32_i32`.
  **L1231 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_cube_v4f32_i32`。
- **L1232 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_cube_v4f16_i32`.
  **L1232 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_cube_v4f16_i32`。
- **L1233 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_1d_v4f32_i32`.
  **L1233 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_1d_v4f32_i32`。
- **L1234 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_1d_v4f16_i32`.
  **L1234 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_1d_v4f16_i32`。
- **L1235 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_1darray_v4f32_i32`.
  **L1235 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_1darray_v4f32_i32`。
- **L1236 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_1darray_v4f16_i32`.
  **L1236 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_1darray_v4f16_i32`。
- **L1237 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_2d_f32_i32`.
  **L1237 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_2d_f32_i32`。
- **L1238 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_2d_v4f32_i32`.
  **L1238 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_2d_v4f32_i32`。
- **L1239 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_2d_v4f16_i32`.
  **L1239 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_2d_v4f16_i32`。
- **L1240 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_2darray_f32_i32`.
  **L1240 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_2darray_f32_i32`。
- **L1241 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_2darray_v4f32_i32`.
  **L1241 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_2darray_v4f32_i32`。
- **L1242 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_2darray_v4f16_i32`.
  **L1242 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_2darray_v4f16_i32`。
- **L1243 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_3d_v4f32_i32`.
  **L1243 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_3d_v4f32_i32`。
- **L1244 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_3d_v4f16_i32`.
  **L1244 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_3d_v4f16_i32`。
- **L1245 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_cube_v4f32_i32`.
  **L1245 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_cube_v4f32_i32`。
- **L1246 EN**: Declares TableGen def record `__builtin_amdgcn_image_load_mip_cube_v4f16_i32`.
  **L1246 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_load_mip_cube_v4f16_i32`。
- **L1247 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_1d_v4f32_i32`.
  **L1247 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_1d_v4f32_i32`。
- **L1248 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_1d_v4f16_i32`.
  **L1248 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_1d_v4f16_i32`。

### Lines 1249-1272

````tablegen
def __builtin_amdgcn_image_store_1darray_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_1darray_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_2d_f32_i32 : AMDGPUBuiltin<"void(float, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_2d_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_2d_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_2darray_f32_i32 : AMDGPUBuiltin<"void(float, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_2darray_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_2darray_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_3d_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_3d_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_cube_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_cube_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_1d_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_1d_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_1darray_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_1darray_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_2d_f32_i32 : AMDGPUBuiltin<"void(float, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_2d_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_2d_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_2darray_f32_i32 : AMDGPUBuiltin<"void(float, int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_2darray_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_2darray_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_3d_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_3d_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
````
- **L1249 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_1darray_v4f32_i32`.
  **L1249 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_1darray_v4f32_i32`。
- **L1250 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_1darray_v4f16_i32`.
  **L1250 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_1darray_v4f16_i32`。
- **L1251 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_2d_f32_i32`.
  **L1251 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_2d_f32_i32`。
- **L1252 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_2d_v4f32_i32`.
  **L1252 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_2d_v4f32_i32`。
- **L1253 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_2d_v4f16_i32`.
  **L1253 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_2d_v4f16_i32`。
- **L1254 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_2darray_f32_i32`.
  **L1254 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_2darray_f32_i32`。
- **L1255 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_2darray_v4f32_i32`.
  **L1255 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_2darray_v4f32_i32`。
- **L1256 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_2darray_v4f16_i32`.
  **L1256 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_2darray_v4f16_i32`。
- **L1257 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_3d_v4f32_i32`.
  **L1257 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_3d_v4f32_i32`。
- **L1258 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_3d_v4f16_i32`.
  **L1258 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_3d_v4f16_i32`。
- **L1259 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_cube_v4f32_i32`.
  **L1259 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_cube_v4f32_i32`。
- **L1260 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_cube_v4f16_i32`.
  **L1260 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_cube_v4f16_i32`。
- **L1261 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_1d_v4f32_i32`.
  **L1261 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_1d_v4f32_i32`。
- **L1262 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_1d_v4f16_i32`.
  **L1262 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_1d_v4f16_i32`。
- **L1263 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_1darray_v4f32_i32`.
  **L1263 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_1darray_v4f32_i32`。
- **L1264 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_1darray_v4f16_i32`.
  **L1264 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_1darray_v4f16_i32`。
- **L1265 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_2d_f32_i32`.
  **L1265 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_2d_f32_i32`。
- **L1266 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_2d_v4f32_i32`.
  **L1266 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_2d_v4f32_i32`。
- **L1267 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_2d_v4f16_i32`.
  **L1267 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_2d_v4f16_i32`。
- **L1268 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_2darray_f32_i32`.
  **L1268 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_2darray_f32_i32`。
- **L1269 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_2darray_v4f32_i32`.
  **L1269 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_2darray_v4f32_i32`。
- **L1270 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_2darray_v4f16_i32`.
  **L1270 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_2darray_v4f16_i32`。
- **L1271 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_3d_v4f32_i32`.
  **L1271 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_3d_v4f32_i32`。
- **L1272 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_3d_v4f16_i32`.
  **L1272 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_3d_v4f16_i32`。

### Lines 1273-1296

````tablegen
def __builtin_amdgcn_image_store_mip_cube_v4f32_i32 : AMDGPUBuiltin<"void(_ExtVector<4, float>, int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_store_mip_cube_v4f16_i32 : AMDGPUBuiltin<"void(_ExtVector<4, _Float16>, int, int, int, int, int, __amdgpu_texture_t, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_1d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_1d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_1darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_1darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_2d_f32_f32 : AMDGPUBuiltin<"float(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_2d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_2d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_2darray_f32_f32 : AMDGPUBuiltin<"float(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_2darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_2darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_3d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_3d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_cube_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_cube_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "image-insts">;
def __builtin_amdgcn_image_sample_lz_1d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_1d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_1darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_1darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_2d_f32_f32 : AMDGPUBuiltin<"float(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_2d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_2d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_2darray_f32_f32 : AMDGPUBuiltin<"float(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
````
- **L1273 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_cube_v4f32_i32`.
  **L1273 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_cube_v4f32_i32`。
- **L1274 EN**: Declares TableGen def record `__builtin_amdgcn_image_store_mip_cube_v4f16_i32`.
  **L1274 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_store_mip_cube_v4f16_i32`。
- **L1275 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_1d_v4f32_f32`.
  **L1275 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_1d_v4f32_f32`。
- **L1276 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_1d_v4f16_f32`.
  **L1276 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_1d_v4f16_f32`。
- **L1277 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_1darray_v4f32_f32`.
  **L1277 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_1darray_v4f32_f32`。
- **L1278 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_1darray_v4f16_f32`.
  **L1278 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_1darray_v4f16_f32`。
- **L1279 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_2d_f32_f32`.
  **L1279 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_2d_f32_f32`。
- **L1280 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_2d_v4f32_f32`.
  **L1280 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_2d_v4f32_f32`。
- **L1281 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_2d_v4f16_f32`.
  **L1281 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_2d_v4f16_f32`。
- **L1282 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_2darray_f32_f32`.
  **L1282 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_2darray_f32_f32`。
- **L1283 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_2darray_v4f32_f32`.
  **L1283 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_2darray_v4f32_f32`。
- **L1284 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_2darray_v4f16_f32`.
  **L1284 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_2darray_v4f16_f32`。
- **L1285 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_3d_v4f32_f32`.
  **L1285 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_3d_v4f32_f32`。
- **L1286 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_3d_v4f16_f32`.
  **L1286 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_3d_v4f16_f32`。
- **L1287 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_cube_v4f32_f32`.
  **L1287 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_cube_v4f32_f32`。
- **L1288 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_cube_v4f16_f32`.
  **L1288 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_cube_v4f16_f32`。
- **L1289 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_1d_v4f32_f32`.
  **L1289 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_1d_v4f32_f32`。
- **L1290 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_1d_v4f16_f32`.
  **L1290 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_1d_v4f16_f32`。
- **L1291 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_1darray_v4f32_f32`.
  **L1291 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_1darray_v4f32_f32`。
- **L1292 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_1darray_v4f16_f32`.
  **L1292 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_1darray_v4f16_f32`。
- **L1293 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_2d_f32_f32`.
  **L1293 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_2d_f32_f32`。
- **L1294 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_2d_v4f32_f32`.
  **L1294 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_2d_v4f32_f32`。
- **L1295 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_2d_v4f16_f32`.
  **L1295 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_2d_v4f16_f32`。
- **L1296 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_2darray_f32_f32`.
  **L1296 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_2darray_f32_f32`。

### Lines 1297-1320

````tablegen
def __builtin_amdgcn_image_sample_lz_2darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_2darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_3d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_3d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_cube_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_lz_cube_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_1d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_1d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_1darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_1darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_2d_f32_f32 : AMDGPUBuiltin<"float(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_2d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_2d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_2darray_f32_f32 : AMDGPUBuiltin<"float(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_2darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_2darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_3d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_3d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_cube_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_l_cube_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_1d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_1d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_1darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_1darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
````
- **L1297 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_2darray_v4f32_f32`.
  **L1297 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_2darray_v4f32_f32`。
- **L1298 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_2darray_v4f16_f32`.
  **L1298 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_2darray_v4f16_f32`。
- **L1299 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_3d_v4f32_f32`.
  **L1299 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_3d_v4f32_f32`。
- **L1300 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_3d_v4f16_f32`.
  **L1300 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_3d_v4f16_f32`。
- **L1301 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_cube_v4f32_f32`.
  **L1301 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_cube_v4f32_f32`。
- **L1302 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_lz_cube_v4f16_f32`.
  **L1302 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_lz_cube_v4f16_f32`。
- **L1303 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_1d_v4f32_f32`.
  **L1303 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_1d_v4f32_f32`。
- **L1304 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_1d_v4f16_f32`.
  **L1304 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_1d_v4f16_f32`。
- **L1305 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_1darray_v4f32_f32`.
  **L1305 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_1darray_v4f32_f32`。
- **L1306 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_1darray_v4f16_f32`.
  **L1306 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_1darray_v4f16_f32`。
- **L1307 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_2d_f32_f32`.
  **L1307 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_2d_f32_f32`。
- **L1308 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_2d_v4f32_f32`.
  **L1308 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_2d_v4f32_f32`。
- **L1309 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_2d_v4f16_f32`.
  **L1309 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_2d_v4f16_f32`。
- **L1310 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_2darray_f32_f32`.
  **L1310 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_2darray_f32_f32`。
- **L1311 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_2darray_v4f32_f32`.
  **L1311 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_2darray_v4f32_f32`。
- **L1312 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_2darray_v4f16_f32`.
  **L1312 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_2darray_v4f16_f32`。
- **L1313 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_3d_v4f32_f32`.
  **L1313 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_3d_v4f32_f32`。
- **L1314 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_3d_v4f16_f32`.
  **L1314 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_3d_v4f16_f32`。
- **L1315 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_cube_v4f32_f32`.
  **L1315 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_cube_v4f32_f32`。
- **L1316 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_l_cube_v4f16_f32`.
  **L1316 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_l_cube_v4f16_f32`。
- **L1317 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_1d_v4f32_f32`.
  **L1317 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_1d_v4f32_f32`。
- **L1318 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_1d_v4f16_f32`.
  **L1318 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_1d_v4f16_f32`。
- **L1319 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_1darray_v4f32_f32`.
  **L1319 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_1darray_v4f32_f32`。
- **L1320 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_1darray_v4f16_f32`.
  **L1320 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_1darray_v4f16_f32`。

### Lines 1321-1329

````tablegen
def __builtin_amdgcn_image_sample_d_2d_f32_f32 : AMDGPUBuiltin<"float(int, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_2d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_2d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_2darray_f32_f32 : AMDGPUBuiltin<"float(int, float, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_2darray_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_2darray_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_3d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, float, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_sample_d_3d_v4f16_f32 : AMDGPUBuiltin<"_ExtVector<4, _Float16>(int, float, float, float, float, float, float, float, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
def __builtin_amdgcn_image_gather4_lz_2d_v4f32_f32 : AMDGPUBuiltin<"_ExtVector<4, float>(int, float, float, __amdgpu_texture_t, _ExtVector<4, int>, bool, int, int)", [Const], "extended-image-insts">;
````
- **L1321 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_2d_f32_f32`.
  **L1321 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_2d_f32_f32`。
- **L1322 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_2d_v4f32_f32`.
  **L1322 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_2d_v4f32_f32`。
- **L1323 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_2d_v4f16_f32`.
  **L1323 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_2d_v4f16_f32`。
- **L1324 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_2darray_f32_f32`.
  **L1324 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_2darray_f32_f32`。
- **L1325 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_2darray_v4f32_f32`.
  **L1325 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_2darray_v4f32_f32`。
- **L1326 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_2darray_v4f16_f32`.
  **L1326 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_2darray_v4f16_f32`。
- **L1327 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_3d_v4f32_f32`.
  **L1327 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_3d_v4f32_f32`。
- **L1328 EN**: Declares TableGen def record `__builtin_amdgcn_image_sample_d_3d_v4f16_f32`.
  **L1328 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_sample_d_3d_v4f16_f32`。
- **L1329 EN**: Declares TableGen def record `__builtin_amdgcn_image_gather4_lz_2d_v4f32_f32`.
  **L1329 CN**: 声明 TableGen def 记录 `__builtin_amdgcn_image_gather4_lz_2d_v4f32_f32`。

## Key Concepts / 关键概念

- **Compiler configuration / 编译器配置**
  - **EN**: Holds low-level enums, constants, feature switches, and descriptive metadata used across Clang.
  - **CN**: 保存 Clang 各处复用的底层枚举、常量、特性开关与描述性元数据。
- **Target and language metadata / 目标与语言元数据**
  - **EN**: Models ABI, address-space, builtin, and target-feature information shared by many subsystems.
  - **CN**: 建模多个子系统共享的 ABI、地址空间、builtin 与目标特性信息。
- **Generated definition tables / 生成式定义表**
  - **EN**: Many Basic headers use `.td` or `.def` files to describe families of diagnostics, attributes, or builtins declaratively.
  - **CN**: 许多 Basic 头文件使用 `.td` 或 `.def` 文件以声明式方式描述诊断、属性或 builtin 家族。
- **TableGen declarative modeling / TableGen 声明式建模**
  - **EN**: Uses TableGen records, inheritance, and generators to describe families of compiler entities compactly.
  - **CN**: 使用 TableGen 记录、继承与生成器紧凑地描述一组编译器实体。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。
- **Attribute metadata / 属性元数据**
  - **EN**: Captures attribute spellings, subjects, semantic flags, and generated helper data.
  - **CN**: 刻画属性的拼写、适用对象、语义标志与生成的辅助数据。
- **ABI contracts / ABI 契约**
  - **EN**: Records ABI-relevant enums and conventions used to represent constructors, destructors, or calling details.
  - **CN**: 记录用于表示构造函数、析构函数或调用细节的 ABI 相关枚举与约定。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **AMDGPU target support / AMDGPU 目标支持**
  - **EN**: Describes AMDGPU-specific builtins or type metadata.
  - **CN**: 描述 AMDGPU 专用 builtin 或类型元数据。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `AMDGPUBuiltin`
- **Functions or callables / 函数或可调用对象**: `listconcat`, `int`, `short`, `uint64_t`, `void`, `uint32_t`, `double`, `float`, `bool`, `__fp16>`, `short>`, `int>`
- **TableGen records / TableGen 记录**: `AMDGPUBuiltin`, `__builtin_amdgcn_dispatch_ptr`, `__builtin_amdgcn_kernarg_segment_ptr`, `__builtin_amdgcn_implicitarg_ptr`, `__builtin_amdgcn_queue_ptr`, `__builtin_amdgcn_workgroup_id_x`, `__builtin_amdgcn_workgroup_id_y`, `__builtin_amdgcn_workgroup_id_z`, `__builtin_amdgcn_cluster_id_x`, `__builtin_amdgcn_cluster_id_y`, `__builtin_amdgcn_cluster_id_z`, `__builtin_amdgcn_cluster_workgroup_id_x`, `__builtin_amdgcn_cluster_workgroup_id_y`, `__builtin_amdgcn_cluster_workgroup_id_z`, `__builtin_amdgcn_cluster_workgroup_flat_id`, `__builtin_amdgcn_cluster_workgroup_max_id_x`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
