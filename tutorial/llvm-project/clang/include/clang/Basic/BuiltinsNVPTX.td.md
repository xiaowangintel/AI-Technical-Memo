# BuiltinsNVPTX.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsNVPTX.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: NVPTX Builtin function defs *- C++.
- **Purpose (CN)**: 声明与 `BuiltinsNVPTX` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1269

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//===--- BuiltinsNVPTX.td - NVPTX Builtin function defs ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the PTX-specific builtin function database.
//
//===----------------------------------------------------------------------===//

include "clang/Basic/BuiltinsBase.td"

class SMFeatures {
  string Features;
}

// Some builtins are only supported on a subset of 'a' (accelerated) GPUs
class SMa<list<int> gpus> : SMFeatures {
  let Features = !interleave(!foreach(gpu, gpus, "sm_" # gpu # "a"), "|");
}


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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the PTX-specific builtin function database.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the PTX-specific builtin function database.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L13 EN**: Includes TableGen file `"clang/Basic/BuiltinsBase.td"` so later records can reuse shared definitions.
  **L13 CN**: 引入 TableGen 文件 `"clang/Basic/BuiltinsBase.td"`，以便后续记录复用共享定义。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Declares TableGen class record `SMFeatures`.
  **L15 CN**: 声明 TableGen class 记录 `SMFeatures`。
- **L16 EN**: Adds a standalone statement or declaration: `string Features;`.
  **L16 CN**: 添加一条独立语句或声明：`string Features;`。
- **L17 EN**: Closes the current lexical scope or compound statement.
  **L17 CN**: 结束当前词法作用域或复合语句块。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Comment explains nearby logic, constraints, or intent: `Some builtins are only supported on a subset of 'a' (accelerated) GPUs`.
  **L19 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Some builtins are only supported on a subset of 'a' (accelerated) GPUs`。
- **L20 EN**: Declares TableGen class record `SMa`.
  **L20 CN**: 声明 TableGen class 记录 `SMa`。
- **L21 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = !interleave(!foreach(gpu, gpus, "sm_" # gpu # "a"), "|");`.
  **L21 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = !interleave(!foreach(gpu, gpus, "sm_" # gpu # "a"), "|");`。
- **L22 EN**: Closes the current lexical scope or compound statement.
  **L22 CN**: 结束当前词法作用域或复合语句块。
- **L23 EN**: Blank line separating nearby declarations or logic blocks.
  **L23 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L24 EN**: Blank line separating nearby declarations or logic blocks.
  **L24 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 25-48

````tablegen
multiclass SM_Base<int sm, list<int> gpus> {
  defvar family = !div(sm, 10);

  defvar gpu_bases = !foreach(g, gpus, "sm_" # g);

  defvar gpu_fs = !foldl([]<string>, gpus, acc, g,
    !if(!ge(!div(g, 10), 10),
      !listconcat(acc, ["sm_" # g # "f"]),
      acc
    )
  );

  defvar gpu_as = !foldl([]<string>, gpus, acc, g,
    !if(!ge(!div(g, 10), 9),
      !listconcat(acc, ["sm_" # g # "a"]),
      acc
    )
  );

  defvar same_family_newer_fs_as = !foldl([]<string>, gpus, acc, g,
    !if(!and(!ne(g, sm), !eq(!div(g, 10), family)),
      !listconcat(acc, ["sm_" # g # "f", "sm_" # g # "a"]),
      acc
    )
````
- **L25 EN**: Declares TableGen multiclass record `SM_Base`.
  **L25 CN**: 声明 TableGen multiclass 记录 `SM_Base`。
- **L26 EN**: Declares TableGen defvar record `family = !div`.
  **L26 CN**: 声明 TableGen defvar 记录 `family = !div`。
- **L27 EN**: Blank line separating nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L28 EN**: Declares TableGen defvar record `gpu_bases = !foreach`.
  **L28 CN**: 声明 TableGen defvar 记录 `gpu_bases = !foreach`。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L30 EN**: Declares TableGen defvar record `gpu_fs = !foldl`.
  **L30 CN**: 声明 TableGen defvar 记录 `gpu_fs = !foldl`。
- **L31 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!ge(!div(g, 10), 10),`.
  **L31 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!ge(!div(g, 10), 10),`。
- **L32 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(acc, ["sm_" # g # "f"]),`.
  **L32 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(acc, ["sm_" # g # "f"]),`。
- **L33 EN**: Continues the surrounding expression or declaration: `acc`.
  **L33 CN**: 继续构造周围的表达式或声明：`acc`。
- **L34 EN**: Continues the surrounding expression or declaration: `)`.
  **L34 CN**: 继续构造周围的表达式或声明：`)`。
- **L35 EN**: Adds a standalone statement or declaration: `);`.
  **L35 CN**: 添加一条独立语句或声明：`);`。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Declares TableGen defvar record `gpu_as = !foldl`.
  **L37 CN**: 声明 TableGen defvar 记录 `gpu_as = !foldl`。
- **L38 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!ge(!div(g, 10), 9),`.
  **L38 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!ge(!div(g, 10), 9),`。
- **L39 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(acc, ["sm_" # g # "a"]),`.
  **L39 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(acc, ["sm_" # g # "a"]),`。
- **L40 EN**: Continues the surrounding expression or declaration: `acc`.
  **L40 CN**: 继续构造周围的表达式或声明：`acc`。
- **L41 EN**: Continues the surrounding expression or declaration: `)`.
  **L41 CN**: 继续构造周围的表达式或声明：`)`。
- **L42 EN**: Adds a standalone statement or declaration: `);`.
  **L42 CN**: 添加一条独立语句或声明：`);`。
- **L43 EN**: Blank line separating nearby declarations or logic blocks.
  **L43 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L44 EN**: Declares TableGen defvar record `same_family_newer_fs_as = !foldl`.
  **L44 CN**: 声明 TableGen defvar 记录 `same_family_newer_fs_as = !foldl`。
- **L45 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!and(!ne(g, sm), !eq(!div(g, 10), family)),`.
  **L45 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!and(!ne(g, sm), !eq(!div(g, 10), family)),`。
- **L46 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(acc, ["sm_" # g # "f", "sm_" # g # "a"]),`.
  **L46 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(acc, ["sm_" # g # "f", "sm_" # g # "a"]),`。
- **L47 EN**: Continues the surrounding expression or declaration: `acc`.
  **L47 CN**: 继续构造周围的表达式或声明：`acc`。
- **L48 EN**: Continues the surrounding expression or declaration: `)`.
  **L48 CN**: 继续构造周围的表达式或声明：`)`。

### Lines 49-72

````tablegen
  );
  defvar f_features = !if(!ge(family, 10),
    !listconcat(["sm_" # sm # "f", "sm_" # sm # "a"], same_family_newer_fs_as),
    []<string>
  );

  if !ge(family, 9) then {
    def a : SMFeatures {
      let Features = "sm_" # sm # "a";
    }
  }

  if !not(!empty(f_features)) then {
    def f : SMFeatures {
      let Features = !interleave(f_features, "|");
    }
  }

  def "" : SMFeatures {
    let Features = !interleave(!listconcat(gpu_bases, gpu_fs, gpu_as), "|");
  }
}

multiclass SM_Instantiate<list<int> gpu_list> {
````
- **L49 EN**: Adds a standalone statement or declaration: `);`.
  **L49 CN**: 添加一条独立语句或声明：`);`。
- **L50 EN**: Declares TableGen defvar record `f_features = !if`.
  **L50 CN**: 声明 TableGen defvar 记录 `f_features = !if`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(["sm_" # sm # "f", "sm_" # sm # "a"], same_family_newer_fs_as),`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(["sm_" # sm # "f", "sm_" # sm # "a"], same_family_newer_fs_as),`。
- **L52 EN**: Continues the surrounding expression or declaration: `[]<string>`.
  **L52 CN**: 继续构造周围的表达式或声明：`[]<string>`。
- **L53 EN**: Adds a standalone statement or declaration: `);`.
  **L53 CN**: 添加一条独立语句或声明：`);`。
- **L54 EN**: Blank line separating nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L55 EN**: Continues logic associated with callable symbol `ge`.
  **L55 CN**: 继续与可调用符号 `ge` 相关的逻辑。
- **L56 EN**: Declares TableGen def record `a`.
  **L56 CN**: 声明 TableGen def 记录 `a`。
- **L57 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = "sm_" # sm # "a";`.
  **L57 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = "sm_" # sm # "a";`。
- **L58 EN**: Closes the current lexical scope or compound statement.
  **L58 CN**: 结束当前词法作用域或复合语句块。
- **L59 EN**: Closes the current lexical scope or compound statement.
  **L59 CN**: 结束当前词法作用域或复合语句块。
- **L60 EN**: Blank line separating nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L61 EN**: Continues logic associated with callable symbol `not`.
  **L61 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L62 EN**: Declares TableGen def record `f`.
  **L62 CN**: 声明 TableGen def 记录 `f`。
- **L63 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = !interleave(f_features, "|");`.
  **L63 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = !interleave(f_features, "|");`。
- **L64 EN**: Closes the current lexical scope or compound statement.
  **L64 CN**: 结束当前词法作用域或复合语句块。
- **L65 EN**: Closes the current lexical scope or compound statement.
  **L65 CN**: 结束当前词法作用域或复合语句块。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Declares TableGen def record `""`.
  **L67 CN**: 声明 TableGen def 记录 `""`。
- **L68 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = !interleave(!listconcat(gpu_bases, gpu_fs, gpu_as), "|");`.
  **L68 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = !interleave(!listconcat(gpu_bases, gpu_fs, gpu_as), "|");`。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Closes the current lexical scope or compound statement.
  **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Declares TableGen multiclass record `SM_Instantiate`.
  **L72 CN**: 声明 TableGen multiclass 记录 `SM_Instantiate`。

### Lines 73-96

````tablegen
  foreach i = !range(0, !size(gpu_list)) in {
    defvar sm = gpu_list[i];
    // Generate the list of compatible GPUs in ascending order (base SM first).
    defvar gpus = !foreach(idx, !range(0, !add(i, 1)), gpu_list[!sub(i, idx)]);

    // Define base GPU and its variants dynamically
    defm _#sm : SM_Base<sm, gpus>;
  }
}

defm SM : SM_Instantiate<[121, 120, 110, 103, 101, 100, 90, 89, 88, 87, 86, 80, 75, 72, 70, 62, 61, 60, 53]>;

class PTXFeatures {
  string Features;
}

multiclass PTX_InstantiateLoop<list<int> ptx_list> {
  foreach i = !range(0, !size(ptx_list)) in {
    defvar ptx = ptx_list[i];
    // Generate the list of compatible PTX versions in ascending order (base PTX first).
    defvar ptxs = !foreach(idx, !range(0, !add(i, 1)), "ptx" # ptx_list[!sub(i, idx)]);
    def NAME#ptx : PTXFeatures {
      let Features = !interleave(ptxs, "|");
    }
````
- **L73 EN**: Starts a TableGen iteration used to generate repeated records: `foreach i = !range(0, !size(gpu_list)) in {`.
  **L73 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach i = !range(0, !size(gpu_list)) in {`。
- **L74 EN**: Declares TableGen defvar record `sm = gpu_list[i];`.
  **L74 CN**: 声明 TableGen defvar 记录 `sm = gpu_list[i];`。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `Generate the list of compatible GPUs in ascending order (base SM first).`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generate the list of compatible GPUs in ascending order (base SM first).`。
- **L76 EN**: Declares TableGen defvar record `gpus = !foreach`.
  **L76 CN**: 声明 TableGen defvar 记录 `gpus = !foreach`。
- **L77 EN**: Blank line separating nearby declarations or logic blocks.
  **L77 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L78 EN**: Comment explains nearby logic, constraints, or intent: `Define base GPU and its variants dynamically`.
  **L78 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Define base GPU and its variants dynamically`。
- **L79 EN**: Declares TableGen defm record `_#sm`.
  **L79 CN**: 声明 TableGen defm 记录 `_#sm`。
- **L80 EN**: Closes the current lexical scope or compound statement.
  **L80 CN**: 结束当前词法作用域或复合语句块。
- **L81 EN**: Closes the current lexical scope or compound statement.
  **L81 CN**: 结束当前词法作用域或复合语句块。
- **L82 EN**: Blank line separating nearby declarations or logic blocks.
  **L82 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L83 EN**: Declares TableGen defm record `SM`.
  **L83 CN**: 声明 TableGen defm 记录 `SM`。
- **L84 EN**: Blank line separating nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L85 EN**: Declares TableGen class record `PTXFeatures`.
  **L85 CN**: 声明 TableGen class 记录 `PTXFeatures`。
- **L86 EN**: Adds a standalone statement or declaration: `string Features;`.
  **L86 CN**: 添加一条独立语句或声明：`string Features;`。
- **L87 EN**: Closes the current lexical scope or compound statement.
  **L87 CN**: 结束当前词法作用域或复合语句块。
- **L88 EN**: Blank line separating nearby declarations or logic blocks.
  **L88 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L89 EN**: Declares TableGen multiclass record `PTX_InstantiateLoop`.
  **L89 CN**: 声明 TableGen multiclass 记录 `PTX_InstantiateLoop`。
- **L90 EN**: Starts a TableGen iteration used to generate repeated records: `foreach i = !range(0, !size(ptx_list)) in {`.
  **L90 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach i = !range(0, !size(ptx_list)) in {`。
- **L91 EN**: Declares TableGen defvar record `ptx = ptx_list[i];`.
  **L91 CN**: 声明 TableGen defvar 记录 `ptx = ptx_list[i];`。
- **L92 EN**: Comment explains nearby logic, constraints, or intent: `Generate the list of compatible PTX versions in ascending order (base PTX first).`.
  **L92 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Generate the list of compatible PTX versions in ascending order (base PTX first).`。
- **L93 EN**: Declares TableGen defvar record `ptxs = !foreach`.
  **L93 CN**: 声明 TableGen defvar 记录 `ptxs = !foreach`。
- **L94 EN**: Declares TableGen def record `NAME#ptx`.
  **L94 CN**: 声明 TableGen def 记录 `NAME#ptx`。
- **L95 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = !interleave(ptxs, "|");`.
  **L95 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = !interleave(ptxs, "|");`。
- **L96 EN**: Closes the current lexical scope or compound statement.
  **L96 CN**: 结束当前词法作用域或复合语句块。

### Lines 97-120

````tablegen
  }
}

defm PTX : PTX_InstantiateLoop<[90, 88, 87, 86, 85, 84, 83, 82, 81, 80, 78, 77, 76, 75, 74, 73, 72, 71, 70, 65, 64, 63, 62, 61, 60, 42]>;

class NVPTXBuiltin<string prototype> : TargetBuiltin {
  let Spellings = [NAME];
  let Prototype = prototype;
}

class NVPTXBuiltinSM<string prototype, SMFeatures sm> : NVPTXBuiltin<prototype> {
  let Features = sm.Features;
}

class NVPTXBuiltinPTX<string prototype, PTXFeatures ptx> : NVPTXBuiltin<prototype> {
  let Features = ptx.Features;
}

class NVPTXBuiltinSMAndPTX<string prototype, SMFeatures sm, PTXFeatures ptx> : NVPTXBuiltin<prototype> {
  let Features = !strconcat("(", sm.Features, "),(", ptx.Features, ")");
}

// Special Registers

````
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Blank line separating nearby declarations or logic blocks.
  **L99 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L100 EN**: Declares TableGen defm record `PTX`.
  **L100 CN**: 声明 TableGen defm 记录 `PTX`。
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L102 EN**: Declares TableGen class record `NVPTXBuiltin`.
  **L102 CN**: 声明 TableGen class 记录 `NVPTXBuiltin`。
- **L103 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Spellings = [NAME];`.
  **L103 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Spellings = [NAME];`。
- **L104 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Prototype = prototype;`.
  **L104 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Prototype = prototype;`。
- **L105 EN**: Closes the current lexical scope or compound statement.
  **L105 CN**: 结束当前词法作用域或复合语句块。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L107 EN**: Declares TableGen class record `NVPTXBuiltinSM`.
  **L107 CN**: 声明 TableGen class 记录 `NVPTXBuiltinSM`。
- **L108 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = sm.Features;`.
  **L108 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = sm.Features;`。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Blank line separating nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L111 EN**: Declares TableGen class record `NVPTXBuiltinPTX`.
  **L111 CN**: 声明 TableGen class 记录 `NVPTXBuiltinPTX`。
- **L112 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = ptx.Features;`.
  **L112 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = ptx.Features;`。
- **L113 EN**: Closes the current lexical scope or compound statement.
  **L113 CN**: 结束当前词法作用域或复合语句块。
- **L114 EN**: Blank line separating nearby declarations or logic blocks.
  **L114 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L115 EN**: Declares TableGen class record `NVPTXBuiltinSMAndPTX`.
  **L115 CN**: 声明 TableGen class 记录 `NVPTXBuiltinSMAndPTX`。
- **L116 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Features = !strconcat("(", sm.Features, "),(", ptx.Features, ")");`.
  **L116 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Features = !strconcat("(", sm.Features, "),(", ptx.Features, ")");`。
- **L117 EN**: Closes the current lexical scope or compound statement.
  **L117 CN**: 结束当前词法作用域或复合语句块。
- **L118 EN**: Blank line separating nearby declarations or logic blocks.
  **L118 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L119 EN**: Comment explains nearby logic, constraints, or intent: `Special Registers`.
  **L119 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Special Registers`。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 121-144

````tablegen
let Attributes = [NoThrow, Const] in {
  def __nvvm_read_ptx_sreg_tid_x : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_tid_y : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_tid_z : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_tid_w : NVPTXBuiltin<"int()">;

  def __nvvm_read_ptx_sreg_ntid_x : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_ntid_y : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_ntid_z : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_ntid_w : NVPTXBuiltin<"int()">;

  def __nvvm_read_ptx_sreg_ctaid_x : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_ctaid_y : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_ctaid_z : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_ctaid_w : NVPTXBuiltin<"int()">;

  def __nvvm_read_ptx_sreg_nctaid_x : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_nctaid_y : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_nctaid_z : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_nctaid_w : NVPTXBuiltin<"int()">;

  def __nvvm_read_ptx_sreg_clusterid_x : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_clusterid_y : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_clusterid_z : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
````
- **L121 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in {`.
  **L121 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in {`。
- **L122 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_tid_x`.
  **L122 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_tid_x`。
- **L123 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_tid_y`.
  **L123 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_tid_y`。
- **L124 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_tid_z`.
  **L124 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_tid_z`。
- **L125 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_tid_w`.
  **L125 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_tid_w`。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L127 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ntid_x`.
  **L127 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ntid_x`。
- **L128 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ntid_y`.
  **L128 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ntid_y`。
- **L129 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ntid_z`.
  **L129 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ntid_z`。
- **L130 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ntid_w`.
  **L130 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ntid_w`。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L132 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ctaid_x`.
  **L132 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ctaid_x`。
- **L133 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ctaid_y`.
  **L133 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ctaid_y`。
- **L134 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ctaid_z`.
  **L134 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ctaid_z`。
- **L135 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_ctaid_w`.
  **L135 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_ctaid_w`。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L137 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nctaid_x`.
  **L137 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nctaid_x`。
- **L138 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nctaid_y`.
  **L138 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nctaid_y`。
- **L139 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nctaid_z`.
  **L139 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nctaid_z`。
- **L140 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nctaid_w`.
  **L140 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nctaid_w`。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L142 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_clusterid_x`.
  **L142 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_clusterid_x`。
- **L143 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_clusterid_y`.
  **L143 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_clusterid_y`。
- **L144 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_clusterid_z`.
  **L144 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_clusterid_z`。

### Lines 145-168

````tablegen
  def __nvvm_read_ptx_sreg_clusterid_w : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;

  def __nvvm_read_ptx_sreg_nclusterid_x : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_nclusterid_y : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_nclusterid_z : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_nclusterid_w : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;

  def __nvvm_read_ptx_sreg_cluster_ctaid_x : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_cluster_ctaid_y : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_cluster_ctaid_z : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_cluster_ctaid_w : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;

  def __nvvm_read_ptx_sreg_cluster_nctaid_x : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_cluster_nctaid_y : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_cluster_nctaid_z : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_cluster_nctaid_w : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;

  def __nvvm_read_ptx_sreg_cluster_ctarank : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;
  def __nvvm_read_ptx_sreg_cluster_nctarank : NVPTXBuiltinSMAndPTX<"int()", SM_90, PTX78>;

  def __nvvm_is_explicit_cluster : NVPTXBuiltinSMAndPTX<"bool()", SM_90, PTX78>;

  def __nvvm_read_ptx_sreg_laneid : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_warpid : NVPTXBuiltin<"int()">;
````
- **L145 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_clusterid_w`.
  **L145 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_clusterid_w`。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nclusterid_x`.
  **L147 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nclusterid_x`。
- **L148 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nclusterid_y`.
  **L148 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nclusterid_y`。
- **L149 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nclusterid_z`.
  **L149 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nclusterid_z`。
- **L150 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nclusterid_w`.
  **L150 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nclusterid_w`。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L152 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_ctaid_x`.
  **L152 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_ctaid_x`。
- **L153 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_ctaid_y`.
  **L153 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_ctaid_y`。
- **L154 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_ctaid_z`.
  **L154 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_ctaid_z`。
- **L155 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_ctaid_w`.
  **L155 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_ctaid_w`。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_nctaid_x`.
  **L157 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_nctaid_x`。
- **L158 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_nctaid_y`.
  **L158 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_nctaid_y`。
- **L159 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_nctaid_z`.
  **L159 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_nctaid_z`。
- **L160 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_nctaid_w`.
  **L160 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_nctaid_w`。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_ctarank`.
  **L162 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_ctarank`。
- **L163 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_cluster_nctarank`.
  **L163 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_cluster_nctarank`。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Declares TableGen def record `__nvvm_is_explicit_cluster`.
  **L165 CN**: 声明 TableGen def 记录 `__nvvm_is_explicit_cluster`。
- **L166 EN**: Blank line separating nearby declarations or logic blocks.
  **L166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L167 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_laneid`.
  **L167 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_laneid`。
- **L168 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_warpid`.
  **L168 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_warpid`。

### Lines 169-192

````tablegen
  def __nvvm_read_ptx_sreg_nwarpid : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_warpsize : NVPTXBuiltin<"int()">;

  def __nvvm_read_ptx_sreg_smid : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_nsmid : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_gridid : NVPTXBuiltin<"int()">;

  def __nvvm_read_ptx_sreg_lanemask_eq : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_lanemask_le : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_lanemask_lt : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_lanemask_ge : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_lanemask_gt : NVPTXBuiltin<"int()">;
}

let Attributes = [NoThrow] in {
  def __nvvm_read_ptx_sreg_clock : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_clock64 : NVPTXBuiltin<"long long int()">;
  def __nvvm_read_ptx_sreg_globaltimer : NVPTXBuiltin<"long long int()">;

  def __nvvm_read_ptx_sreg_pm0 : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_pm1 : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_pm2 : NVPTXBuiltin<"int()">;
  def __nvvm_read_ptx_sreg_pm3 : NVPTXBuiltin<"int()">;
}
````
- **L169 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nwarpid`.
  **L169 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nwarpid`。
- **L170 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_warpsize`.
  **L170 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_warpsize`。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_smid`.
  **L172 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_smid`。
- **L173 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_nsmid`.
  **L173 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_nsmid`。
- **L174 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_gridid`.
  **L174 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_gridid`。
- **L175 EN**: Blank line separating nearby declarations or logic blocks.
  **L175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L176 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_lanemask_eq`.
  **L176 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_lanemask_eq`。
- **L177 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_lanemask_le`.
  **L177 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_lanemask_le`。
- **L178 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_lanemask_lt`.
  **L178 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_lanemask_lt`。
- **L179 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_lanemask_ge`.
  **L179 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_lanemask_ge`。
- **L180 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_lanemask_gt`.
  **L180 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_lanemask_gt`。
- **L181 EN**: Closes the current lexical scope or compound statement.
  **L181 CN**: 结束当前词法作用域或复合语句块。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L183 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L183 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L184 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_clock`.
  **L184 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_clock`。
- **L185 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_clock64`.
  **L185 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_clock64`。
- **L186 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_globaltimer`.
  **L186 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_globaltimer`。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L188 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_pm0`.
  **L188 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_pm0`。
- **L189 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_pm1`.
  **L189 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_pm1`。
- **L190 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_pm2`.
  **L190 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_pm2`。
- **L191 EN**: Declares TableGen def record `__nvvm_read_ptx_sreg_pm3`.
  **L191 CN**: 声明 TableGen def 记录 `__nvvm_read_ptx_sreg_pm3`。
- **L192 EN**: Closes the current lexical scope or compound statement.
  **L192 CN**: 结束当前词法作用域或复合语句块。

### Lines 193-216

````tablegen

// MISC

def __nvvm_prmt : NVPTXBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)">;
let Attributes = [NoReturn] in {
  def __nvvm_exit : NVPTXBuiltin<"void()">;
  def __nvvm_reflect : NVPTXBuiltin<"unsigned int(char const *)">;
}
let Attributes = [NoThrow] in {
  def __nvvm_nanosleep : NVPTXBuiltinSMAndPTX<"void(unsigned int)", SM_70, PTX63>;
  def __nvvm_pm_event_mask : NVPTXBuiltin<"void(_Constant unsigned short)">;
}

// Min Max

def __nvvm_fmin_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmin_ftz_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmin_nan_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmin_ftz_nan_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmin_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmin_ftz_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmin_nan_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmin_ftz_nan_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmin_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
````
- **L193 EN**: Blank line separating nearby declarations or logic blocks.
  **L193 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L194 EN**: Comment explains nearby logic, constraints, or intent: `MISC`.
  **L194 CN**: 注释解释附近代码的逻辑、约束或设计意图：`MISC`。
- **L195 EN**: Blank line separating nearby declarations or logic blocks.
  **L195 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L196 EN**: Declares TableGen def record `__nvvm_prmt`.
  **L196 CN**: 声明 TableGen def 记录 `__nvvm_prmt`。
- **L197 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoReturn] in {`.
  **L197 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoReturn] in {`。
- **L198 EN**: Declares TableGen def record `__nvvm_exit`.
  **L198 CN**: 声明 TableGen def 记录 `__nvvm_exit`。
- **L199 EN**: Declares TableGen def record `__nvvm_reflect`.
  **L199 CN**: 声明 TableGen def 记录 `__nvvm_reflect`。
- **L200 EN**: Closes the current lexical scope or compound statement.
  **L200 CN**: 结束当前词法作用域或复合语句块。
- **L201 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L201 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L202 EN**: Declares TableGen def record `__nvvm_nanosleep`.
  **L202 CN**: 声明 TableGen def 记录 `__nvvm_nanosleep`。
- **L203 EN**: Declares TableGen def record `__nvvm_pm_event_mask`.
  **L203 CN**: 声明 TableGen def 记录 `__nvvm_pm_event_mask`。
- **L204 EN**: Closes the current lexical scope or compound statement.
  **L204 CN**: 结束当前词法作用域或复合语句块。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L206 EN**: Comment explains nearby logic, constraints, or intent: `Min Max`.
  **L206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Min Max`。
- **L207 EN**: Blank line separating nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L208 EN**: Declares TableGen def record `__nvvm_fmin_f16`.
  **L208 CN**: 声明 TableGen def 记录 `__nvvm_fmin_f16`。
- **L209 EN**: Declares TableGen def record `__nvvm_fmin_ftz_f16`.
  **L209 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_f16`。
- **L210 EN**: Declares TableGen def record `__nvvm_fmin_nan_f16`.
  **L210 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_f16`。
- **L211 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_f16`.
  **L211 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_f16`。
- **L212 EN**: Declares TableGen def record `__nvvm_fmin_xorsign_abs_f16`.
  **L212 CN**: 声明 TableGen def 记录 `__nvvm_fmin_xorsign_abs_f16`。
- **L213 EN**: Declares TableGen def record `__nvvm_fmin_ftz_xorsign_abs_f16`.
  **L213 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_xorsign_abs_f16`。
- **L214 EN**: Declares TableGen def record `__nvvm_fmin_nan_xorsign_abs_f16`.
  **L214 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_xorsign_abs_f16`。
- **L215 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_xorsign_abs_f16`.
  **L215 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_xorsign_abs_f16`。
- **L216 EN**: Declares TableGen def record `__nvvm_fmin_f16x2`.
  **L216 CN**: 声明 TableGen def 记录 `__nvvm_fmin_f16x2`。

### Lines 217-240

````tablegen
def __nvvm_fmin_ftz_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fmin_nan_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fmin_ftz_nan_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fmin_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmin_ftz_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmin_nan_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmin_ftz_nan_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmin_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fmin_ftz_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fmin_nan_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fmin_ftz_nan_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fmin_xorsign_abs_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_86, PTX72>;
def __nvvm_fmin_nan_xorsign_abs_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_86, PTX72>;
def __nvvm_fmin_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmin_ftz_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmin_nan_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmin_ftz_nan_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmin_xorsign_abs_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_86, PTX72>;
def __nvvm_fmin_nan_xorsign_abs_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_86, PTX72>;
def __nvvm_fmin_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_fmin_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_fmin_nan_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_80, PTX70>;
def __nvvm_fmin_ftz_nan_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_80, PTX70>;
def __nvvm_fmin_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
````
- **L217 EN**: Declares TableGen def record `__nvvm_fmin_ftz_f16x2`.
  **L217 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_f16x2`。
- **L218 EN**: Declares TableGen def record `__nvvm_fmin_nan_f16x2`.
  **L218 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_f16x2`。
- **L219 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_f16x2`.
  **L219 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_f16x2`。
- **L220 EN**: Declares TableGen def record `__nvvm_fmin_xorsign_abs_f16x2`.
  **L220 CN**: 声明 TableGen def 记录 `__nvvm_fmin_xorsign_abs_f16x2`。
- **L221 EN**: Declares TableGen def record `__nvvm_fmin_ftz_xorsign_abs_f16x2`.
  **L221 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_xorsign_abs_f16x2`。
- **L222 EN**: Declares TableGen def record `__nvvm_fmin_nan_xorsign_abs_f16x2`.
  **L222 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_xorsign_abs_f16x2`。
- **L223 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_xorsign_abs_f16x2`.
  **L223 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_xorsign_abs_f16x2`。
- **L224 EN**: Declares TableGen def record `__nvvm_fmin_bf16`.
  **L224 CN**: 声明 TableGen def 记录 `__nvvm_fmin_bf16`。
- **L225 EN**: Declares TableGen def record `__nvvm_fmin_ftz_bf16`.
  **L225 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_bf16`。
- **L226 EN**: Declares TableGen def record `__nvvm_fmin_nan_bf16`.
  **L226 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_bf16`。
- **L227 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_bf16`.
  **L227 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_bf16`。
- **L228 EN**: Declares TableGen def record `__nvvm_fmin_xorsign_abs_bf16`.
  **L228 CN**: 声明 TableGen def 记录 `__nvvm_fmin_xorsign_abs_bf16`。
- **L229 EN**: Declares TableGen def record `__nvvm_fmin_nan_xorsign_abs_bf16`.
  **L229 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_xorsign_abs_bf16`。
- **L230 EN**: Declares TableGen def record `__nvvm_fmin_bf16x2`.
  **L230 CN**: 声明 TableGen def 记录 `__nvvm_fmin_bf16x2`。
- **L231 EN**: Declares TableGen def record `__nvvm_fmin_ftz_bf16x2`.
  **L231 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_bf16x2`。
- **L232 EN**: Declares TableGen def record `__nvvm_fmin_nan_bf16x2`.
  **L232 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_bf16x2`。
- **L233 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_bf16x2`.
  **L233 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_bf16x2`。
- **L234 EN**: Declares TableGen def record `__nvvm_fmin_xorsign_abs_bf16x2`.
  **L234 CN**: 声明 TableGen def 记录 `__nvvm_fmin_xorsign_abs_bf16x2`。
- **L235 EN**: Declares TableGen def record `__nvvm_fmin_nan_xorsign_abs_bf16x2`.
  **L235 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_xorsign_abs_bf16x2`。
- **L236 EN**: Declares TableGen def record `__nvvm_fmin_f`.
  **L236 CN**: 声明 TableGen def 记录 `__nvvm_fmin_f`。
- **L237 EN**: Declares TableGen def record `__nvvm_fmin_ftz_f`.
  **L237 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_f`。
- **L238 EN**: Declares TableGen def record `__nvvm_fmin_nan_f`.
  **L238 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_f`。
- **L239 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_f`.
  **L239 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_f`。
- **L240 EN**: Declares TableGen def record `__nvvm_fmin_xorsign_abs_f`.
  **L240 CN**: 声明 TableGen def 记录 `__nvvm_fmin_xorsign_abs_f`。

### Lines 241-264

````tablegen
def __nvvm_fmin_ftz_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
def __nvvm_fmin_nan_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
def __nvvm_fmin_ftz_nan_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
def __nvvm_fmin_d : NVPTXBuiltin<"double(double, double)">;

def __nvvm_fmax_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmax_ftz_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmax_nan_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmax_ftz_nan_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fmax_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmax_ftz_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmax_nan_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmax_ftz_nan_xorsign_abs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_86, PTX72>;
def __nvvm_fmax_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fmax_ftz_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fmax_nan_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fmax_ftz_nan_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fmax_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmax_ftz_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmax_nan_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmax_ftz_nan_xorsign_abs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_86, PTX72>;
def __nvvm_fmax_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fmax_ftz_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fmax_nan_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
````
- **L241 EN**: Declares TableGen def record `__nvvm_fmin_ftz_xorsign_abs_f`.
  **L241 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_xorsign_abs_f`。
- **L242 EN**: Declares TableGen def record `__nvvm_fmin_nan_xorsign_abs_f`.
  **L242 CN**: 声明 TableGen def 记录 `__nvvm_fmin_nan_xorsign_abs_f`。
- **L243 EN**: Declares TableGen def record `__nvvm_fmin_ftz_nan_xorsign_abs_f`.
  **L243 CN**: 声明 TableGen def 记录 `__nvvm_fmin_ftz_nan_xorsign_abs_f`。
- **L244 EN**: Declares TableGen def record `__nvvm_fmin_d`.
  **L244 CN**: 声明 TableGen def 记录 `__nvvm_fmin_d`。
- **L245 EN**: Blank line separating nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L246 EN**: Declares TableGen def record `__nvvm_fmax_f16`.
  **L246 CN**: 声明 TableGen def 记录 `__nvvm_fmax_f16`。
- **L247 EN**: Declares TableGen def record `__nvvm_fmax_ftz_f16`.
  **L247 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_f16`。
- **L248 EN**: Declares TableGen def record `__nvvm_fmax_nan_f16`.
  **L248 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_f16`。
- **L249 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_f16`.
  **L249 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_f16`。
- **L250 EN**: Declares TableGen def record `__nvvm_fmax_xorsign_abs_f16`.
  **L250 CN**: 声明 TableGen def 记录 `__nvvm_fmax_xorsign_abs_f16`。
- **L251 EN**: Declares TableGen def record `__nvvm_fmax_ftz_xorsign_abs_f16`.
  **L251 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_xorsign_abs_f16`。
- **L252 EN**: Declares TableGen def record `__nvvm_fmax_nan_xorsign_abs_f16`.
  **L252 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_xorsign_abs_f16`。
- **L253 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_xorsign_abs_f16`.
  **L253 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_xorsign_abs_f16`。
- **L254 EN**: Declares TableGen def record `__nvvm_fmax_f16x2`.
  **L254 CN**: 声明 TableGen def 记录 `__nvvm_fmax_f16x2`。
- **L255 EN**: Declares TableGen def record `__nvvm_fmax_ftz_f16x2`.
  **L255 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_f16x2`。
- **L256 EN**: Declares TableGen def record `__nvvm_fmax_nan_f16x2`.
  **L256 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_f16x2`。
- **L257 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_f16x2`.
  **L257 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_f16x2`。
- **L258 EN**: Declares TableGen def record `__nvvm_fmax_xorsign_abs_f16x2`.
  **L258 CN**: 声明 TableGen def 记录 `__nvvm_fmax_xorsign_abs_f16x2`。
- **L259 EN**: Declares TableGen def record `__nvvm_fmax_ftz_xorsign_abs_f16x2`.
  **L259 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_xorsign_abs_f16x2`。
- **L260 EN**: Declares TableGen def record `__nvvm_fmax_nan_xorsign_abs_f16x2`.
  **L260 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_xorsign_abs_f16x2`。
- **L261 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_xorsign_abs_f16x2`.
  **L261 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_xorsign_abs_f16x2`。
- **L262 EN**: Declares TableGen def record `__nvvm_fmax_bf16`.
  **L262 CN**: 声明 TableGen def 记录 `__nvvm_fmax_bf16`。
- **L263 EN**: Declares TableGen def record `__nvvm_fmax_ftz_bf16`.
  **L263 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_bf16`。
- **L264 EN**: Declares TableGen def record `__nvvm_fmax_nan_bf16`.
  **L264 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_bf16`。

### Lines 265-288

````tablegen
def __nvvm_fmax_ftz_nan_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fmax_xorsign_abs_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_86, PTX72>;
def __nvvm_fmax_nan_xorsign_abs_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16)", SM_86, PTX72>;
def __nvvm_fmax_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmax_ftz_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmax_nan_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmax_ftz_nan_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fmax_xorsign_abs_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_86, PTX72>;
def __nvvm_fmax_nan_xorsign_abs_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>)", SM_86, PTX72>;
def __nvvm_fmax_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_fmax_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_fmax_nan_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_80, PTX70>;
def __nvvm_fmax_ftz_nan_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_80, PTX70>;
def __nvvm_fmax_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
def __nvvm_fmax_ftz_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
def __nvvm_fmax_nan_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
def __nvvm_fmax_ftz_nan_xorsign_abs_f : NVPTXBuiltinSMAndPTX<"float(float, float)", SM_86, PTX72>;
def __nvvm_fmax_d : NVPTXBuiltin<"double(double, double)">;

// Multiplication

def __nvvm_mulhi_i : NVPTXBuiltin<"int(int, int)">;
def __nvvm_mulhi_ui : NVPTXBuiltin<"unsigned int(unsigned int, unsigned int)">;
def __nvvm_mulhi_ll : NVPTXBuiltin<"long long int(long long int, long long int)">;
````
- **L265 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_bf16`.
  **L265 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_bf16`。
- **L266 EN**: Declares TableGen def record `__nvvm_fmax_xorsign_abs_bf16`.
  **L266 CN**: 声明 TableGen def 记录 `__nvvm_fmax_xorsign_abs_bf16`。
- **L267 EN**: Declares TableGen def record `__nvvm_fmax_nan_xorsign_abs_bf16`.
  **L267 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_xorsign_abs_bf16`。
- **L268 EN**: Declares TableGen def record `__nvvm_fmax_bf16x2`.
  **L268 CN**: 声明 TableGen def 记录 `__nvvm_fmax_bf16x2`。
- **L269 EN**: Declares TableGen def record `__nvvm_fmax_ftz_bf16x2`.
  **L269 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_bf16x2`。
- **L270 EN**: Declares TableGen def record `__nvvm_fmax_nan_bf16x2`.
  **L270 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_bf16x2`。
- **L271 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_bf16x2`.
  **L271 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_bf16x2`。
- **L272 EN**: Declares TableGen def record `__nvvm_fmax_xorsign_abs_bf16x2`.
  **L272 CN**: 声明 TableGen def 记录 `__nvvm_fmax_xorsign_abs_bf16x2`。
- **L273 EN**: Declares TableGen def record `__nvvm_fmax_nan_xorsign_abs_bf16x2`.
  **L273 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_xorsign_abs_bf16x2`。
- **L274 EN**: Declares TableGen def record `__nvvm_fmax_f`.
  **L274 CN**: 声明 TableGen def 记录 `__nvvm_fmax_f`。
- **L275 EN**: Declares TableGen def record `__nvvm_fmax_ftz_f`.
  **L275 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_f`。
- **L276 EN**: Declares TableGen def record `__nvvm_fmax_nan_f`.
  **L276 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_f`。
- **L277 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_f`.
  **L277 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_f`。
- **L278 EN**: Declares TableGen def record `__nvvm_fmax_xorsign_abs_f`.
  **L278 CN**: 声明 TableGen def 记录 `__nvvm_fmax_xorsign_abs_f`。
- **L279 EN**: Declares TableGen def record `__nvvm_fmax_ftz_xorsign_abs_f`.
  **L279 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_xorsign_abs_f`。
- **L280 EN**: Declares TableGen def record `__nvvm_fmax_nan_xorsign_abs_f`.
  **L280 CN**: 声明 TableGen def 记录 `__nvvm_fmax_nan_xorsign_abs_f`。
- **L281 EN**: Declares TableGen def record `__nvvm_fmax_ftz_nan_xorsign_abs_f`.
  **L281 CN**: 声明 TableGen def 记录 `__nvvm_fmax_ftz_nan_xorsign_abs_f`。
- **L282 EN**: Declares TableGen def record `__nvvm_fmax_d`.
  **L282 CN**: 声明 TableGen def 记录 `__nvvm_fmax_d`。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L284 EN**: Comment explains nearby logic, constraints, or intent: `Multiplication`.
  **L284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Multiplication`。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L286 EN**: Declares TableGen def record `__nvvm_mulhi_i`.
  **L286 CN**: 声明 TableGen def 记录 `__nvvm_mulhi_i`。
- **L287 EN**: Declares TableGen def record `__nvvm_mulhi_ui`.
  **L287 CN**: 声明 TableGen def 记录 `__nvvm_mulhi_ui`。
- **L288 EN**: Declares TableGen def record `__nvvm_mulhi_ll`.
  **L288 CN**: 声明 TableGen def 记录 `__nvvm_mulhi_ll`。

### Lines 289-312

````tablegen
def __nvvm_mulhi_ull : NVPTXBuiltin<"unsigned long long int(unsigned long long int, unsigned long long int)">;

def __nvvm_mul_rn_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_mul_rn_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_mul_rz_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_mul_rz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_mul_rm_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_mul_rm_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_mul_rp_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_mul_rp_f : NVPTXBuiltin<"float(float, float)">;

def __nvvm_mul_rn_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_mul_rz_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_mul_rm_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_mul_rp_d : NVPTXBuiltin<"double(double, double)">;

def __nvvm_mul24_i : NVPTXBuiltin<"int(int, int)">;
def __nvvm_mul24_ui : NVPTXBuiltin<"unsigned int(unsigned int, unsigned int)">;

// Div

def __nvvm_div_approx_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_approx_f : NVPTXBuiltin<"float(float, float)">;

````
- **L289 EN**: Declares TableGen def record `__nvvm_mulhi_ull`.
  **L289 CN**: 声明 TableGen def 记录 `__nvvm_mulhi_ull`。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Declares TableGen def record `__nvvm_mul_rn_ftz_f`.
  **L291 CN**: 声明 TableGen def 记录 `__nvvm_mul_rn_ftz_f`。
- **L292 EN**: Declares TableGen def record `__nvvm_mul_rn_f`.
  **L292 CN**: 声明 TableGen def 记录 `__nvvm_mul_rn_f`。
- **L293 EN**: Declares TableGen def record `__nvvm_mul_rz_ftz_f`.
  **L293 CN**: 声明 TableGen def 记录 `__nvvm_mul_rz_ftz_f`。
- **L294 EN**: Declares TableGen def record `__nvvm_mul_rz_f`.
  **L294 CN**: 声明 TableGen def 记录 `__nvvm_mul_rz_f`。
- **L295 EN**: Declares TableGen def record `__nvvm_mul_rm_ftz_f`.
  **L295 CN**: 声明 TableGen def 记录 `__nvvm_mul_rm_ftz_f`。
- **L296 EN**: Declares TableGen def record `__nvvm_mul_rm_f`.
  **L296 CN**: 声明 TableGen def 记录 `__nvvm_mul_rm_f`。
- **L297 EN**: Declares TableGen def record `__nvvm_mul_rp_ftz_f`.
  **L297 CN**: 声明 TableGen def 记录 `__nvvm_mul_rp_ftz_f`。
- **L298 EN**: Declares TableGen def record `__nvvm_mul_rp_f`.
  **L298 CN**: 声明 TableGen def 记录 `__nvvm_mul_rp_f`。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L300 EN**: Declares TableGen def record `__nvvm_mul_rn_d`.
  **L300 CN**: 声明 TableGen def 记录 `__nvvm_mul_rn_d`。
- **L301 EN**: Declares TableGen def record `__nvvm_mul_rz_d`.
  **L301 CN**: 声明 TableGen def 记录 `__nvvm_mul_rz_d`。
- **L302 EN**: Declares TableGen def record `__nvvm_mul_rm_d`.
  **L302 CN**: 声明 TableGen def 记录 `__nvvm_mul_rm_d`。
- **L303 EN**: Declares TableGen def record `__nvvm_mul_rp_d`.
  **L303 CN**: 声明 TableGen def 记录 `__nvvm_mul_rp_d`。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L305 EN**: Declares TableGen def record `__nvvm_mul24_i`.
  **L305 CN**: 声明 TableGen def 记录 `__nvvm_mul24_i`。
- **L306 EN**: Declares TableGen def record `__nvvm_mul24_ui`.
  **L306 CN**: 声明 TableGen def 记录 `__nvvm_mul24_ui`。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L308 EN**: Comment explains nearby logic, constraints, or intent: `Div`.
  **L308 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Div`。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Declares TableGen def record `__nvvm_div_approx_ftz_f`.
  **L310 CN**: 声明 TableGen def 记录 `__nvvm_div_approx_ftz_f`。
- **L311 EN**: Declares TableGen def record `__nvvm_div_approx_f`.
  **L311 CN**: 声明 TableGen def 记录 `__nvvm_div_approx_f`。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 313-336

````tablegen
def __nvvm_div_rn_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_rn_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_rz_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_rz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_rm_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_rm_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_rp_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_div_rp_f : NVPTXBuiltin<"float(float, float)">;

def __nvvm_div_rn_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_div_rz_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_div_rm_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_div_rp_d : NVPTXBuiltin<"double(double, double)">;

// Sad

def __nvvm_sad_i : NVPTXBuiltin<"int(int, int, int)">;
def __nvvm_sad_ui : NVPTXBuiltin<"unsigned int(unsigned int, unsigned int, unsigned int)">;

// Floor, Ceil

def __nvvm_floor_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_floor_f : NVPTXBuiltin<"float(float)">;
def __nvvm_floor_d : NVPTXBuiltin<"double(double)">;
````
- **L313 EN**: Declares TableGen def record `__nvvm_div_rn_ftz_f`.
  **L313 CN**: 声明 TableGen def 记录 `__nvvm_div_rn_ftz_f`。
- **L314 EN**: Declares TableGen def record `__nvvm_div_rn_f`.
  **L314 CN**: 声明 TableGen def 记录 `__nvvm_div_rn_f`。
- **L315 EN**: Declares TableGen def record `__nvvm_div_rz_ftz_f`.
  **L315 CN**: 声明 TableGen def 记录 `__nvvm_div_rz_ftz_f`。
- **L316 EN**: Declares TableGen def record `__nvvm_div_rz_f`.
  **L316 CN**: 声明 TableGen def 记录 `__nvvm_div_rz_f`。
- **L317 EN**: Declares TableGen def record `__nvvm_div_rm_ftz_f`.
  **L317 CN**: 声明 TableGen def 记录 `__nvvm_div_rm_ftz_f`。
- **L318 EN**: Declares TableGen def record `__nvvm_div_rm_f`.
  **L318 CN**: 声明 TableGen def 记录 `__nvvm_div_rm_f`。
- **L319 EN**: Declares TableGen def record `__nvvm_div_rp_ftz_f`.
  **L319 CN**: 声明 TableGen def 记录 `__nvvm_div_rp_ftz_f`。
- **L320 EN**: Declares TableGen def record `__nvvm_div_rp_f`.
  **L320 CN**: 声明 TableGen def 记录 `__nvvm_div_rp_f`。
- **L321 EN**: Blank line separating nearby declarations or logic blocks.
  **L321 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L322 EN**: Declares TableGen def record `__nvvm_div_rn_d`.
  **L322 CN**: 声明 TableGen def 记录 `__nvvm_div_rn_d`。
- **L323 EN**: Declares TableGen def record `__nvvm_div_rz_d`.
  **L323 CN**: 声明 TableGen def 记录 `__nvvm_div_rz_d`。
- **L324 EN**: Declares TableGen def record `__nvvm_div_rm_d`.
  **L324 CN**: 声明 TableGen def 记录 `__nvvm_div_rm_d`。
- **L325 EN**: Declares TableGen def record `__nvvm_div_rp_d`.
  **L325 CN**: 声明 TableGen def 记录 `__nvvm_div_rp_d`。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L327 EN**: Comment explains nearby logic, constraints, or intent: `Sad`.
  **L327 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sad`。
- **L328 EN**: Blank line separating nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L329 EN**: Declares TableGen def record `__nvvm_sad_i`.
  **L329 CN**: 声明 TableGen def 记录 `__nvvm_sad_i`。
- **L330 EN**: Declares TableGen def record `__nvvm_sad_ui`.
  **L330 CN**: 声明 TableGen def 记录 `__nvvm_sad_ui`。
- **L331 EN**: Blank line separating nearby declarations or logic blocks.
  **L331 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L332 EN**: Comment explains nearby logic, constraints, or intent: `Floor, Ceil`.
  **L332 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Floor, Ceil`。
- **L333 EN**: Blank line separating nearby declarations or logic blocks.
  **L333 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L334 EN**: Declares TableGen def record `__nvvm_floor_ftz_f`.
  **L334 CN**: 声明 TableGen def 记录 `__nvvm_floor_ftz_f`。
- **L335 EN**: Declares TableGen def record `__nvvm_floor_f`.
  **L335 CN**: 声明 TableGen def 记录 `__nvvm_floor_f`。
- **L336 EN**: Declares TableGen def record `__nvvm_floor_d`.
  **L336 CN**: 声明 TableGen def 记录 `__nvvm_floor_d`。

### Lines 337-360

````tablegen

def __nvvm_ceil_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_ceil_f : NVPTXBuiltin<"float(float)">;
def __nvvm_ceil_d : NVPTXBuiltin<"double(double)">;

// Abs

def __nvvm_fabs_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_fabs_f : NVPTXBuiltin<"float(float)">;
def __nvvm_fabs_d : NVPTXBuiltin<"double(double)">;

def __nvvm_fabs_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16)", SM_53, PTX65>;
def __nvvm_fabs_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>)", SM_53, PTX65>;
def __nvvm_fabs_ftz_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16)", SM_53, PTX65>;
def __nvvm_fabs_ftz_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>)", SM_53, PTX65>;

// Round

def __nvvm_round_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_round_f : NVPTXBuiltin<"float(float)">;
def __nvvm_round_d : NVPTXBuiltin<"double(double)">;

// Trunc

````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L338 EN**: Declares TableGen def record `__nvvm_ceil_ftz_f`.
  **L338 CN**: 声明 TableGen def 记录 `__nvvm_ceil_ftz_f`。
- **L339 EN**: Declares TableGen def record `__nvvm_ceil_f`.
  **L339 CN**: 声明 TableGen def 记录 `__nvvm_ceil_f`。
- **L340 EN**: Declares TableGen def record `__nvvm_ceil_d`.
  **L340 CN**: 声明 TableGen def 记录 `__nvvm_ceil_d`。
- **L341 EN**: Blank line separating nearby declarations or logic blocks.
  **L341 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L342 EN**: Comment explains nearby logic, constraints, or intent: `Abs`.
  **L342 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Abs`。
- **L343 EN**: Blank line separating nearby declarations or logic blocks.
  **L343 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L344 EN**: Declares TableGen def record `__nvvm_fabs_ftz_f`.
  **L344 CN**: 声明 TableGen def 记录 `__nvvm_fabs_ftz_f`。
- **L345 EN**: Declares TableGen def record `__nvvm_fabs_f`.
  **L345 CN**: 声明 TableGen def 记录 `__nvvm_fabs_f`。
- **L346 EN**: Declares TableGen def record `__nvvm_fabs_d`.
  **L346 CN**: 声明 TableGen def 记录 `__nvvm_fabs_d`。
- **L347 EN**: Blank line separating nearby declarations or logic blocks.
  **L347 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L348 EN**: Declares TableGen def record `__nvvm_fabs_f16`.
  **L348 CN**: 声明 TableGen def 记录 `__nvvm_fabs_f16`。
- **L349 EN**: Declares TableGen def record `__nvvm_fabs_f16x2`.
  **L349 CN**: 声明 TableGen def 记录 `__nvvm_fabs_f16x2`。
- **L350 EN**: Declares TableGen def record `__nvvm_fabs_ftz_f16`.
  **L350 CN**: 声明 TableGen def 记录 `__nvvm_fabs_ftz_f16`。
- **L351 EN**: Declares TableGen def record `__nvvm_fabs_ftz_f16x2`.
  **L351 CN**: 声明 TableGen def 记录 `__nvvm_fabs_ftz_f16x2`。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Comment explains nearby logic, constraints, or intent: `Round`.
  **L353 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Round`。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Declares TableGen def record `__nvvm_round_ftz_f`.
  **L355 CN**: 声明 TableGen def 记录 `__nvvm_round_ftz_f`。
- **L356 EN**: Declares TableGen def record `__nvvm_round_f`.
  **L356 CN**: 声明 TableGen def 记录 `__nvvm_round_f`。
- **L357 EN**: Declares TableGen def record `__nvvm_round_d`.
  **L357 CN**: 声明 TableGen def 记录 `__nvvm_round_d`。
- **L358 EN**: Blank line separating nearby declarations or logic blocks.
  **L358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L359 EN**: Comment explains nearby logic, constraints, or intent: `Trunc`.
  **L359 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Trunc`。
- **L360 EN**: Blank line separating nearby declarations or logic blocks.
  **L360 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 361-384

````tablegen
def __nvvm_trunc_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_trunc_f : NVPTXBuiltin<"float(float)">;
def __nvvm_trunc_d : NVPTXBuiltin<"double(double)">;

// Saturate

def __nvvm_saturate_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_saturate_f : NVPTXBuiltin<"float(float)">;
def __nvvm_saturate_d : NVPTXBuiltin<"double(double)">;

// Exp2, Log2

def __nvvm_ex2_approx_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_ex2_approx_f : NVPTXBuiltin<"float(float)">;
def __nvvm_ex2_approx_d : NVPTXBuiltin<"double(double)">;
def __nvvm_ex2_approx_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16)", SM_75, PTX70>;
def __nvvm_ex2_approx_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>)", SM_75, PTX70>;

def __nvvm_lg2_approx_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_lg2_approx_f : NVPTXBuiltin<"float(float)">;
def __nvvm_lg2_approx_d : NVPTXBuiltin<"double(double)">;

// Sin, Cos

````
- **L361 EN**: Declares TableGen def record `__nvvm_trunc_ftz_f`.
  **L361 CN**: 声明 TableGen def 记录 `__nvvm_trunc_ftz_f`。
- **L362 EN**: Declares TableGen def record `__nvvm_trunc_f`.
  **L362 CN**: 声明 TableGen def 记录 `__nvvm_trunc_f`。
- **L363 EN**: Declares TableGen def record `__nvvm_trunc_d`.
  **L363 CN**: 声明 TableGen def 记录 `__nvvm_trunc_d`。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L365 EN**: Comment explains nearby logic, constraints, or intent: `Saturate`.
  **L365 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Saturate`。
- **L366 EN**: Blank line separating nearby declarations or logic blocks.
  **L366 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L367 EN**: Declares TableGen def record `__nvvm_saturate_ftz_f`.
  **L367 CN**: 声明 TableGen def 记录 `__nvvm_saturate_ftz_f`。
- **L368 EN**: Declares TableGen def record `__nvvm_saturate_f`.
  **L368 CN**: 声明 TableGen def 记录 `__nvvm_saturate_f`。
- **L369 EN**: Declares TableGen def record `__nvvm_saturate_d`.
  **L369 CN**: 声明 TableGen def 记录 `__nvvm_saturate_d`。
- **L370 EN**: Blank line separating nearby declarations or logic blocks.
  **L370 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L371 EN**: Comment explains nearby logic, constraints, or intent: `Exp2, Log2`.
  **L371 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Exp2, Log2`。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L373 EN**: Declares TableGen def record `__nvvm_ex2_approx_ftz_f`.
  **L373 CN**: 声明 TableGen def 记录 `__nvvm_ex2_approx_ftz_f`。
- **L374 EN**: Declares TableGen def record `__nvvm_ex2_approx_f`.
  **L374 CN**: 声明 TableGen def 记录 `__nvvm_ex2_approx_f`。
- **L375 EN**: Declares TableGen def record `__nvvm_ex2_approx_d`.
  **L375 CN**: 声明 TableGen def 记录 `__nvvm_ex2_approx_d`。
- **L376 EN**: Declares TableGen def record `__nvvm_ex2_approx_f16`.
  **L376 CN**: 声明 TableGen def 记录 `__nvvm_ex2_approx_f16`。
- **L377 EN**: Declares TableGen def record `__nvvm_ex2_approx_f16x2`.
  **L377 CN**: 声明 TableGen def 记录 `__nvvm_ex2_approx_f16x2`。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L379 EN**: Declares TableGen def record `__nvvm_lg2_approx_ftz_f`.
  **L379 CN**: 声明 TableGen def 记录 `__nvvm_lg2_approx_ftz_f`。
- **L380 EN**: Declares TableGen def record `__nvvm_lg2_approx_f`.
  **L380 CN**: 声明 TableGen def 记录 `__nvvm_lg2_approx_f`。
- **L381 EN**: Declares TableGen def record `__nvvm_lg2_approx_d`.
  **L381 CN**: 声明 TableGen def 记录 `__nvvm_lg2_approx_d`。
- **L382 EN**: Blank line separating nearby declarations or logic blocks.
  **L382 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L383 EN**: Comment explains nearby logic, constraints, or intent: `Sin, Cos`.
  **L383 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sin, Cos`。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````tablegen
def __nvvm_sin_approx_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sin_approx_f : NVPTXBuiltin<"float(float)">;

def __nvvm_cos_approx_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_cos_approx_f : NVPTXBuiltin<"float(float)">;

// Fma

def __nvvm_fma_rn_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_53, PTX42>;
def __nvvm_fma_rn_ftz_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_53, PTX42>;
def __nvvm_fma_rn_sat_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_53, PTX42>;
def __nvvm_fma_rn_ftz_sat_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_53, PTX42>;
def __nvvm_fma_rn_relu_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fma_rn_ftz_relu_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_80, PTX70>;
def __nvvm_fma_rn_oob_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_90, PTX81>;
def __nvvm_fma_rn_oob_relu_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16, __fp16)", SM_90, PTX81>;
def __nvvm_fma_rn_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;
def __nvvm_fma_rn_ftz_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;
def __nvvm_fma_rn_sat_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;
def __nvvm_fma_rn_ftz_sat_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;
def __nvvm_fma_rn_relu_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fma_rn_ftz_relu_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_80, PTX70>;
def __nvvm_fma_rn_oob_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_90, PTX81>;
def __nvvm_fma_rn_oob_relu_f16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>, _Vector<2, __fp16>)", SM_90, PTX81>;
````
- **L385 EN**: Declares TableGen def record `__nvvm_sin_approx_ftz_f`.
  **L385 CN**: 声明 TableGen def 记录 `__nvvm_sin_approx_ftz_f`。
- **L386 EN**: Declares TableGen def record `__nvvm_sin_approx_f`.
  **L386 CN**: 声明 TableGen def 记录 `__nvvm_sin_approx_f`。
- **L387 EN**: Blank line separating nearby declarations or logic blocks.
  **L387 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L388 EN**: Declares TableGen def record `__nvvm_cos_approx_ftz_f`.
  **L388 CN**: 声明 TableGen def 记录 `__nvvm_cos_approx_ftz_f`。
- **L389 EN**: Declares TableGen def record `__nvvm_cos_approx_f`.
  **L389 CN**: 声明 TableGen def 记录 `__nvvm_cos_approx_f`。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Comment explains nearby logic, constraints, or intent: `Fma`.
  **L391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Fma`。
- **L392 EN**: Blank line separating nearby declarations or logic blocks.
  **L392 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L393 EN**: Declares TableGen def record `__nvvm_fma_rn_f16`.
  **L393 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_f16`。
- **L394 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_f16`.
  **L394 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_f16`。
- **L395 EN**: Declares TableGen def record `__nvvm_fma_rn_sat_f16`.
  **L395 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_sat_f16`。
- **L396 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_sat_f16`.
  **L396 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_sat_f16`。
- **L397 EN**: Declares TableGen def record `__nvvm_fma_rn_relu_f16`.
  **L397 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_relu_f16`。
- **L398 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_relu_f16`.
  **L398 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_relu_f16`。
- **L399 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_f16`.
  **L399 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_f16`。
- **L400 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_relu_f16`.
  **L400 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_relu_f16`。
- **L401 EN**: Declares TableGen def record `__nvvm_fma_rn_f16x2`.
  **L401 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_f16x2`。
- **L402 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_f16x2`.
  **L402 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_f16x2`。
- **L403 EN**: Declares TableGen def record `__nvvm_fma_rn_sat_f16x2`.
  **L403 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_sat_f16x2`。
- **L404 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_sat_f16x2`.
  **L404 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_sat_f16x2`。
- **L405 EN**: Declares TableGen def record `__nvvm_fma_rn_relu_f16x2`.
  **L405 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_relu_f16x2`。
- **L406 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_relu_f16x2`.
  **L406 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_relu_f16x2`。
- **L407 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_f16x2`.
  **L407 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_f16x2`。
- **L408 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_relu_f16x2`.
  **L408 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_relu_f16x2`。

### Lines 409-432

````tablegen
def __nvvm_fma_rn_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fma_rn_relu_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16, __bf16)", SM_80, PTX70>;
def __nvvm_fma_rn_oob_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16, __bf16)", SM_90, PTX81>;
def __nvvm_fma_rn_oob_relu_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16, __bf16, __bf16)", SM_90, PTX81>;
def __nvvm_fma_rn_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fma_rn_relu_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>, _Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_fma_rn_oob_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>, _Vector<2, __bf16>)", SM_90, PTX81>;
def __nvvm_fma_rn_oob_relu_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>, _Vector<2, __bf16>, _Vector<2, __bf16>)", SM_90, PTX81>;
def __nvvm_fma_rn_ftz_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rn_ftz_sat_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rn_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rn_sat_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rz_ftz_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rz_ftz_sat_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rz_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rz_sat_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rm_ftz_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rm_ftz_sat_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rm_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rm_sat_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rp_ftz_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rp_ftz_sat_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rp_f : NVPTXBuiltin<"float(float, float, float)">;
def __nvvm_fma_rp_sat_f : NVPTXBuiltin<"float(float, float, float)">;
````
- **L409 EN**: Declares TableGen def record `__nvvm_fma_rn_bf16`.
  **L409 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_bf16`。
- **L410 EN**: Declares TableGen def record `__nvvm_fma_rn_relu_bf16`.
  **L410 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_relu_bf16`。
- **L411 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_bf16`.
  **L411 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_bf16`。
- **L412 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_relu_bf16`.
  **L412 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_relu_bf16`。
- **L413 EN**: Declares TableGen def record `__nvvm_fma_rn_bf16x2`.
  **L413 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_bf16x2`。
- **L414 EN**: Declares TableGen def record `__nvvm_fma_rn_relu_bf16x2`.
  **L414 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_relu_bf16x2`。
- **L415 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_bf16x2`.
  **L415 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_bf16x2`。
- **L416 EN**: Declares TableGen def record `__nvvm_fma_rn_oob_relu_bf16x2`.
  **L416 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_oob_relu_bf16x2`。
- **L417 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_f`.
  **L417 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_f`。
- **L418 EN**: Declares TableGen def record `__nvvm_fma_rn_ftz_sat_f`.
  **L418 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_ftz_sat_f`。
- **L419 EN**: Declares TableGen def record `__nvvm_fma_rn_f`.
  **L419 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_f`。
- **L420 EN**: Declares TableGen def record `__nvvm_fma_rn_sat_f`.
  **L420 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_sat_f`。
- **L421 EN**: Declares TableGen def record `__nvvm_fma_rz_ftz_f`.
  **L421 CN**: 声明 TableGen def 记录 `__nvvm_fma_rz_ftz_f`。
- **L422 EN**: Declares TableGen def record `__nvvm_fma_rz_ftz_sat_f`.
  **L422 CN**: 声明 TableGen def 记录 `__nvvm_fma_rz_ftz_sat_f`。
- **L423 EN**: Declares TableGen def record `__nvvm_fma_rz_f`.
  **L423 CN**: 声明 TableGen def 记录 `__nvvm_fma_rz_f`。
- **L424 EN**: Declares TableGen def record `__nvvm_fma_rz_sat_f`.
  **L424 CN**: 声明 TableGen def 记录 `__nvvm_fma_rz_sat_f`。
- **L425 EN**: Declares TableGen def record `__nvvm_fma_rm_ftz_f`.
  **L425 CN**: 声明 TableGen def 记录 `__nvvm_fma_rm_ftz_f`。
- **L426 EN**: Declares TableGen def record `__nvvm_fma_rm_ftz_sat_f`.
  **L426 CN**: 声明 TableGen def 记录 `__nvvm_fma_rm_ftz_sat_f`。
- **L427 EN**: Declares TableGen def record `__nvvm_fma_rm_f`.
  **L427 CN**: 声明 TableGen def 记录 `__nvvm_fma_rm_f`。
- **L428 EN**: Declares TableGen def record `__nvvm_fma_rm_sat_f`.
  **L428 CN**: 声明 TableGen def 记录 `__nvvm_fma_rm_sat_f`。
- **L429 EN**: Declares TableGen def record `__nvvm_fma_rp_ftz_f`.
  **L429 CN**: 声明 TableGen def 记录 `__nvvm_fma_rp_ftz_f`。
- **L430 EN**: Declares TableGen def record `__nvvm_fma_rp_ftz_sat_f`.
  **L430 CN**: 声明 TableGen def 记录 `__nvvm_fma_rp_ftz_sat_f`。
- **L431 EN**: Declares TableGen def record `__nvvm_fma_rp_f`.
  **L431 CN**: 声明 TableGen def 记录 `__nvvm_fma_rp_f`。
- **L432 EN**: Declares TableGen def record `__nvvm_fma_rp_sat_f`.
  **L432 CN**: 声明 TableGen def 记录 `__nvvm_fma_rp_sat_f`。

### Lines 433-456

````tablegen
def __nvvm_fma_rn_d : NVPTXBuiltin<"double(double, double, double)">;
def __nvvm_fma_rz_d : NVPTXBuiltin<"double(double, double, double)">;
def __nvvm_fma_rm_d : NVPTXBuiltin<"double(double, double, double)">;
def __nvvm_fma_rp_d : NVPTXBuiltin<"double(double, double, double)">;

// Rcp

def __nvvm_rcp_rn_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_rn_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_rz_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_rz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_rm_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_rm_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_rp_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_rp_f : NVPTXBuiltin<"float(float)">;

def __nvvm_rcp_rn_d : NVPTXBuiltin<"double(double)">;
def __nvvm_rcp_rz_d : NVPTXBuiltin<"double(double)">;
def __nvvm_rcp_rm_d : NVPTXBuiltin<"double(double)">;
def __nvvm_rcp_rp_d : NVPTXBuiltin<"double(double)">;

def __nvvm_rcp_approx_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rcp_approx_ftz_d : NVPTXBuiltin<"double(double)">;

````
- **L433 EN**: Declares TableGen def record `__nvvm_fma_rn_d`.
  **L433 CN**: 声明 TableGen def 记录 `__nvvm_fma_rn_d`。
- **L434 EN**: Declares TableGen def record `__nvvm_fma_rz_d`.
  **L434 CN**: 声明 TableGen def 记录 `__nvvm_fma_rz_d`。
- **L435 EN**: Declares TableGen def record `__nvvm_fma_rm_d`.
  **L435 CN**: 声明 TableGen def 记录 `__nvvm_fma_rm_d`。
- **L436 EN**: Declares TableGen def record `__nvvm_fma_rp_d`.
  **L436 CN**: 声明 TableGen def 记录 `__nvvm_fma_rp_d`。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `Rcp`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rcp`。
- **L439 EN**: Blank line separating nearby declarations or logic blocks.
  **L439 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L440 EN**: Declares TableGen def record `__nvvm_rcp_rn_ftz_f`.
  **L440 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rn_ftz_f`。
- **L441 EN**: Declares TableGen def record `__nvvm_rcp_rn_f`.
  **L441 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rn_f`。
- **L442 EN**: Declares TableGen def record `__nvvm_rcp_rz_ftz_f`.
  **L442 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rz_ftz_f`。
- **L443 EN**: Declares TableGen def record `__nvvm_rcp_rz_f`.
  **L443 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rz_f`。
- **L444 EN**: Declares TableGen def record `__nvvm_rcp_rm_ftz_f`.
  **L444 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rm_ftz_f`。
- **L445 EN**: Declares TableGen def record `__nvvm_rcp_rm_f`.
  **L445 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rm_f`。
- **L446 EN**: Declares TableGen def record `__nvvm_rcp_rp_ftz_f`.
  **L446 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rp_ftz_f`。
- **L447 EN**: Declares TableGen def record `__nvvm_rcp_rp_f`.
  **L447 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rp_f`。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L449 EN**: Declares TableGen def record `__nvvm_rcp_rn_d`.
  **L449 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rn_d`。
- **L450 EN**: Declares TableGen def record `__nvvm_rcp_rz_d`.
  **L450 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rz_d`。
- **L451 EN**: Declares TableGen def record `__nvvm_rcp_rm_d`.
  **L451 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rm_d`。
- **L452 EN**: Declares TableGen def record `__nvvm_rcp_rp_d`.
  **L452 CN**: 声明 TableGen def 记录 `__nvvm_rcp_rp_d`。
- **L453 EN**: Blank line separating nearby declarations or logic blocks.
  **L453 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L454 EN**: Declares TableGen def record `__nvvm_rcp_approx_ftz_f`.
  **L454 CN**: 声明 TableGen def 记录 `__nvvm_rcp_approx_ftz_f`。
- **L455 EN**: Declares TableGen def record `__nvvm_rcp_approx_ftz_d`.
  **L455 CN**: 声明 TableGen def 记录 `__nvvm_rcp_approx_ftz_d`。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 457-480

````tablegen
// Sqrt

def __nvvm_sqrt_rn_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_rn_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_rz_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_rz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_rm_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_rm_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_rp_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_rp_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_approx_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_sqrt_approx_f : NVPTXBuiltin<"float(float)">;

def __nvvm_sqrt_rn_d : NVPTXBuiltin<"double(double)">;
def __nvvm_sqrt_rz_d : NVPTXBuiltin<"double(double)">;
def __nvvm_sqrt_rm_d : NVPTXBuiltin<"double(double)">;
def __nvvm_sqrt_rp_d : NVPTXBuiltin<"double(double)">;

// Rsqrt

def __nvvm_rsqrt_approx_ftz_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rsqrt_approx_f : NVPTXBuiltin<"float(float)">;
def __nvvm_rsqrt_approx_d : NVPTXBuiltin<"double(double)">;

````
- **L457 EN**: Comment explains nearby logic, constraints, or intent: `Sqrt`.
  **L457 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sqrt`。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L459 EN**: Declares TableGen def record `__nvvm_sqrt_rn_ftz_f`.
  **L459 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rn_ftz_f`。
- **L460 EN**: Declares TableGen def record `__nvvm_sqrt_rn_f`.
  **L460 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rn_f`。
- **L461 EN**: Declares TableGen def record `__nvvm_sqrt_rz_ftz_f`.
  **L461 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rz_ftz_f`。
- **L462 EN**: Declares TableGen def record `__nvvm_sqrt_rz_f`.
  **L462 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rz_f`。
- **L463 EN**: Declares TableGen def record `__nvvm_sqrt_rm_ftz_f`.
  **L463 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rm_ftz_f`。
- **L464 EN**: Declares TableGen def record `__nvvm_sqrt_rm_f`.
  **L464 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rm_f`。
- **L465 EN**: Declares TableGen def record `__nvvm_sqrt_rp_ftz_f`.
  **L465 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rp_ftz_f`。
- **L466 EN**: Declares TableGen def record `__nvvm_sqrt_rp_f`.
  **L466 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rp_f`。
- **L467 EN**: Declares TableGen def record `__nvvm_sqrt_approx_ftz_f`.
  **L467 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_approx_ftz_f`。
- **L468 EN**: Declares TableGen def record `__nvvm_sqrt_approx_f`.
  **L468 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_approx_f`。
- **L469 EN**: Blank line separating nearby declarations or logic blocks.
  **L469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L470 EN**: Declares TableGen def record `__nvvm_sqrt_rn_d`.
  **L470 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rn_d`。
- **L471 EN**: Declares TableGen def record `__nvvm_sqrt_rz_d`.
  **L471 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rz_d`。
- **L472 EN**: Declares TableGen def record `__nvvm_sqrt_rm_d`.
  **L472 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rm_d`。
- **L473 EN**: Declares TableGen def record `__nvvm_sqrt_rp_d`.
  **L473 CN**: 声明 TableGen def 记录 `__nvvm_sqrt_rp_d`。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Comment explains nearby logic, constraints, or intent: `Rsqrt`.
  **L475 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Rsqrt`。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L477 EN**: Declares TableGen def record `__nvvm_rsqrt_approx_ftz_f`.
  **L477 CN**: 声明 TableGen def 记录 `__nvvm_rsqrt_approx_ftz_f`。
- **L478 EN**: Declares TableGen def record `__nvvm_rsqrt_approx_f`.
  **L478 CN**: 声明 TableGen def 记录 `__nvvm_rsqrt_approx_f`。
- **L479 EN**: Declares TableGen def record `__nvvm_rsqrt_approx_d`.
  **L479 CN**: 声明 TableGen def 记录 `__nvvm_rsqrt_approx_d`。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 481-504

````tablegen
// Add

def __nvvm_add_rn_sat_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_53, PTX42>;
def __nvvm_add_rn_ftz_sat_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_53, PTX42>;
def __nvvm_add_rn_sat_v2f16 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;
def __nvvm_add_rn_ftz_sat_v2f16 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;

def __nvvm_add_rn_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rn_ftz_sat_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rn_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rn_sat_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rz_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rz_ftz_sat_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rz_sat_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rm_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rm_ftz_sat_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rm_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rm_sat_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rp_ftz_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rp_ftz_sat_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rp_f : NVPTXBuiltin<"float(float, float)">;
def __nvvm_add_rp_sat_f : NVPTXBuiltin<"float(float, float)">;

````
- **L481 EN**: Comment explains nearby logic, constraints, or intent: `Add`.
  **L481 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Add`。
- **L482 EN**: Blank line separating nearby declarations or logic blocks.
  **L482 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L483 EN**: Declares TableGen def record `__nvvm_add_rn_sat_f16`.
  **L483 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_sat_f16`。
- **L484 EN**: Declares TableGen def record `__nvvm_add_rn_ftz_sat_f16`.
  **L484 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_ftz_sat_f16`。
- **L485 EN**: Declares TableGen def record `__nvvm_add_rn_sat_v2f16`.
  **L485 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_sat_v2f16`。
- **L486 EN**: Declares TableGen def record `__nvvm_add_rn_ftz_sat_v2f16`.
  **L486 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_ftz_sat_v2f16`。
- **L487 EN**: Blank line separating nearby declarations or logic blocks.
  **L487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L488 EN**: Declares TableGen def record `__nvvm_add_rn_ftz_f`.
  **L488 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_ftz_f`。
- **L489 EN**: Declares TableGen def record `__nvvm_add_rn_ftz_sat_f`.
  **L489 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_ftz_sat_f`。
- **L490 EN**: Declares TableGen def record `__nvvm_add_rn_f`.
  **L490 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_f`。
- **L491 EN**: Declares TableGen def record `__nvvm_add_rn_sat_f`.
  **L491 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_sat_f`。
- **L492 EN**: Declares TableGen def record `__nvvm_add_rz_ftz_f`.
  **L492 CN**: 声明 TableGen def 记录 `__nvvm_add_rz_ftz_f`。
- **L493 EN**: Declares TableGen def record `__nvvm_add_rz_ftz_sat_f`.
  **L493 CN**: 声明 TableGen def 记录 `__nvvm_add_rz_ftz_sat_f`。
- **L494 EN**: Declares TableGen def record `__nvvm_add_rz_f`.
  **L494 CN**: 声明 TableGen def 记录 `__nvvm_add_rz_f`。
- **L495 EN**: Declares TableGen def record `__nvvm_add_rz_sat_f`.
  **L495 CN**: 声明 TableGen def 记录 `__nvvm_add_rz_sat_f`。
- **L496 EN**: Declares TableGen def record `__nvvm_add_rm_ftz_f`.
  **L496 CN**: 声明 TableGen def 记录 `__nvvm_add_rm_ftz_f`。
- **L497 EN**: Declares TableGen def record `__nvvm_add_rm_ftz_sat_f`.
  **L497 CN**: 声明 TableGen def 记录 `__nvvm_add_rm_ftz_sat_f`。
- **L498 EN**: Declares TableGen def record `__nvvm_add_rm_f`.
  **L498 CN**: 声明 TableGen def 记录 `__nvvm_add_rm_f`。
- **L499 EN**: Declares TableGen def record `__nvvm_add_rm_sat_f`.
  **L499 CN**: 声明 TableGen def 记录 `__nvvm_add_rm_sat_f`。
- **L500 EN**: Declares TableGen def record `__nvvm_add_rp_ftz_f`.
  **L500 CN**: 声明 TableGen def 记录 `__nvvm_add_rp_ftz_f`。
- **L501 EN**: Declares TableGen def record `__nvvm_add_rp_ftz_sat_f`.
  **L501 CN**: 声明 TableGen def 记录 `__nvvm_add_rp_ftz_sat_f`。
- **L502 EN**: Declares TableGen def record `__nvvm_add_rp_f`.
  **L502 CN**: 声明 TableGen def 记录 `__nvvm_add_rp_f`。
- **L503 EN**: Declares TableGen def record `__nvvm_add_rp_sat_f`.
  **L503 CN**: 声明 TableGen def 记录 `__nvvm_add_rp_sat_f`。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 505-528

````tablegen
def __nvvm_add_rn_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_add_rz_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_add_rm_d : NVPTXBuiltin<"double(double, double)">;
def __nvvm_add_rp_d : NVPTXBuiltin<"double(double, double)">;

// Mul

def __nvvm_mul_rn_sat_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_53, PTX42>;
def __nvvm_mul_rn_ftz_sat_f16 : NVPTXBuiltinSMAndPTX<"__fp16(__fp16, __fp16)", SM_53, PTX42>;
def __nvvm_mul_rn_sat_v2f16 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;
def __nvvm_mul_rn_ftz_sat_v2f16 : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(_Vector<2, __fp16>, _Vector<2, __fp16>)", SM_53, PTX42>;

// Convert

def __nvvm_d2f_rn_ftz : NVPTXBuiltin<"float(double)">;
def __nvvm_d2f_rn : NVPTXBuiltin<"float(double)">;
def __nvvm_d2f_rz_ftz : NVPTXBuiltin<"float(double)">;
def __nvvm_d2f_rz : NVPTXBuiltin<"float(double)">;
def __nvvm_d2f_rm_ftz : NVPTXBuiltin<"float(double)">;
def __nvvm_d2f_rm : NVPTXBuiltin<"float(double)">;
def __nvvm_d2f_rp_ftz : NVPTXBuiltin<"float(double)">;
def __nvvm_d2f_rp : NVPTXBuiltin<"float(double)">;

def __nvvm_d2i_rn : NVPTXBuiltin<"int(double)">;
````
- **L505 EN**: Declares TableGen def record `__nvvm_add_rn_d`.
  **L505 CN**: 声明 TableGen def 记录 `__nvvm_add_rn_d`。
- **L506 EN**: Declares TableGen def record `__nvvm_add_rz_d`.
  **L506 CN**: 声明 TableGen def 记录 `__nvvm_add_rz_d`。
- **L507 EN**: Declares TableGen def record `__nvvm_add_rm_d`.
  **L507 CN**: 声明 TableGen def 记录 `__nvvm_add_rm_d`。
- **L508 EN**: Declares TableGen def record `__nvvm_add_rp_d`.
  **L508 CN**: 声明 TableGen def 记录 `__nvvm_add_rp_d`。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L510 EN**: Comment explains nearby logic, constraints, or intent: `Mul`.
  **L510 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Mul`。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L512 EN**: Declares TableGen def record `__nvvm_mul_rn_sat_f16`.
  **L512 CN**: 声明 TableGen def 记录 `__nvvm_mul_rn_sat_f16`。
- **L513 EN**: Declares TableGen def record `__nvvm_mul_rn_ftz_sat_f16`.
  **L513 CN**: 声明 TableGen def 记录 `__nvvm_mul_rn_ftz_sat_f16`。
- **L514 EN**: Declares TableGen def record `__nvvm_mul_rn_sat_v2f16`.
  **L514 CN**: 声明 TableGen def 记录 `__nvvm_mul_rn_sat_v2f16`。
- **L515 EN**: Declares TableGen def record `__nvvm_mul_rn_ftz_sat_v2f16`.
  **L515 CN**: 声明 TableGen def 记录 `__nvvm_mul_rn_ftz_sat_v2f16`。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L517 EN**: Comment explains nearby logic, constraints, or intent: `Convert`.
  **L517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Convert`。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L519 EN**: Declares TableGen def record `__nvvm_d2f_rn_ftz`.
  **L519 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rn_ftz`。
- **L520 EN**: Declares TableGen def record `__nvvm_d2f_rn`.
  **L520 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rn`。
- **L521 EN**: Declares TableGen def record `__nvvm_d2f_rz_ftz`.
  **L521 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rz_ftz`。
- **L522 EN**: Declares TableGen def record `__nvvm_d2f_rz`.
  **L522 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rz`。
- **L523 EN**: Declares TableGen def record `__nvvm_d2f_rm_ftz`.
  **L523 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rm_ftz`。
- **L524 EN**: Declares TableGen def record `__nvvm_d2f_rm`.
  **L524 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rm`。
- **L525 EN**: Declares TableGen def record `__nvvm_d2f_rp_ftz`.
  **L525 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rp_ftz`。
- **L526 EN**: Declares TableGen def record `__nvvm_d2f_rp`.
  **L526 CN**: 声明 TableGen def 记录 `__nvvm_d2f_rp`。
- **L527 EN**: Blank line separating nearby declarations or logic blocks.
  **L527 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L528 EN**: Declares TableGen def record `__nvvm_d2i_rn`.
  **L528 CN**: 声明 TableGen def 记录 `__nvvm_d2i_rn`。

### Lines 529-552

````tablegen
def __nvvm_d2i_rz : NVPTXBuiltin<"int(double)">;
def __nvvm_d2i_rm : NVPTXBuiltin<"int(double)">;
def __nvvm_d2i_rp : NVPTXBuiltin<"int(double)">;

def __nvvm_d2ui_rn : NVPTXBuiltin<"unsigned int(double)">;
def __nvvm_d2ui_rz : NVPTXBuiltin<"unsigned int(double)">;
def __nvvm_d2ui_rm : NVPTXBuiltin<"unsigned int(double)">;
def __nvvm_d2ui_rp : NVPTXBuiltin<"unsigned int(double)">;

def __nvvm_i2d_rn : NVPTXBuiltin<"double(int)">;
def __nvvm_i2d_rz : NVPTXBuiltin<"double(int)">;
def __nvvm_i2d_rm : NVPTXBuiltin<"double(int)">;
def __nvvm_i2d_rp : NVPTXBuiltin<"double(int)">;

def __nvvm_ui2d_rn : NVPTXBuiltin<"double(unsigned int)">;
def __nvvm_ui2d_rz : NVPTXBuiltin<"double(unsigned int)">;
def __nvvm_ui2d_rm : NVPTXBuiltin<"double(unsigned int)">;
def __nvvm_ui2d_rp : NVPTXBuiltin<"double(unsigned int)">;

def __nvvm_f2i_rn_ftz : NVPTXBuiltin<"int(float)">;
def __nvvm_f2i_rn : NVPTXBuiltin<"int(float)">;
def __nvvm_f2i_rz_ftz : NVPTXBuiltin<"int(float)">;
def __nvvm_f2i_rz : NVPTXBuiltin<"int(float)">;
def __nvvm_f2i_rm_ftz : NVPTXBuiltin<"int(float)">;
````
- **L529 EN**: Declares TableGen def record `__nvvm_d2i_rz`.
  **L529 CN**: 声明 TableGen def 记录 `__nvvm_d2i_rz`。
- **L530 EN**: Declares TableGen def record `__nvvm_d2i_rm`.
  **L530 CN**: 声明 TableGen def 记录 `__nvvm_d2i_rm`。
- **L531 EN**: Declares TableGen def record `__nvvm_d2i_rp`.
  **L531 CN**: 声明 TableGen def 记录 `__nvvm_d2i_rp`。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L533 EN**: Declares TableGen def record `__nvvm_d2ui_rn`.
  **L533 CN**: 声明 TableGen def 记录 `__nvvm_d2ui_rn`。
- **L534 EN**: Declares TableGen def record `__nvvm_d2ui_rz`.
  **L534 CN**: 声明 TableGen def 记录 `__nvvm_d2ui_rz`。
- **L535 EN**: Declares TableGen def record `__nvvm_d2ui_rm`.
  **L535 CN**: 声明 TableGen def 记录 `__nvvm_d2ui_rm`。
- **L536 EN**: Declares TableGen def record `__nvvm_d2ui_rp`.
  **L536 CN**: 声明 TableGen def 记录 `__nvvm_d2ui_rp`。
- **L537 EN**: Blank line separating nearby declarations or logic blocks.
  **L537 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L538 EN**: Declares TableGen def record `__nvvm_i2d_rn`.
  **L538 CN**: 声明 TableGen def 记录 `__nvvm_i2d_rn`。
- **L539 EN**: Declares TableGen def record `__nvvm_i2d_rz`.
  **L539 CN**: 声明 TableGen def 记录 `__nvvm_i2d_rz`。
- **L540 EN**: Declares TableGen def record `__nvvm_i2d_rm`.
  **L540 CN**: 声明 TableGen def 记录 `__nvvm_i2d_rm`。
- **L541 EN**: Declares TableGen def record `__nvvm_i2d_rp`.
  **L541 CN**: 声明 TableGen def 记录 `__nvvm_i2d_rp`。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L543 EN**: Declares TableGen def record `__nvvm_ui2d_rn`.
  **L543 CN**: 声明 TableGen def 记录 `__nvvm_ui2d_rn`。
- **L544 EN**: Declares TableGen def record `__nvvm_ui2d_rz`.
  **L544 CN**: 声明 TableGen def 记录 `__nvvm_ui2d_rz`。
- **L545 EN**: Declares TableGen def record `__nvvm_ui2d_rm`.
  **L545 CN**: 声明 TableGen def 记录 `__nvvm_ui2d_rm`。
- **L546 EN**: Declares TableGen def record `__nvvm_ui2d_rp`.
  **L546 CN**: 声明 TableGen def 记录 `__nvvm_ui2d_rp`。
- **L547 EN**: Blank line separating nearby declarations or logic blocks.
  **L547 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L548 EN**: Declares TableGen def record `__nvvm_f2i_rn_ftz`.
  **L548 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rn_ftz`。
- **L549 EN**: Declares TableGen def record `__nvvm_f2i_rn`.
  **L549 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rn`。
- **L550 EN**: Declares TableGen def record `__nvvm_f2i_rz_ftz`.
  **L550 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rz_ftz`。
- **L551 EN**: Declares TableGen def record `__nvvm_f2i_rz`.
  **L551 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rz`。
- **L552 EN**: Declares TableGen def record `__nvvm_f2i_rm_ftz`.
  **L552 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rm_ftz`。

### Lines 553-576

````tablegen
def __nvvm_f2i_rm : NVPTXBuiltin<"int(float)">;
def __nvvm_f2i_rp_ftz : NVPTXBuiltin<"int(float)">;
def __nvvm_f2i_rp : NVPTXBuiltin<"int(float)">;

def __nvvm_f2ui_rn_ftz : NVPTXBuiltin<"unsigned int(float)">;
def __nvvm_f2ui_rn : NVPTXBuiltin<"unsigned int(float)">;
def __nvvm_f2ui_rz_ftz : NVPTXBuiltin<"unsigned int(float)">;
def __nvvm_f2ui_rz : NVPTXBuiltin<"unsigned int(float)">;
def __nvvm_f2ui_rm_ftz : NVPTXBuiltin<"unsigned int(float)">;
def __nvvm_f2ui_rm : NVPTXBuiltin<"unsigned int(float)">;
def __nvvm_f2ui_rp_ftz : NVPTXBuiltin<"unsigned int(float)">;
def __nvvm_f2ui_rp : NVPTXBuiltin<"unsigned int(float)">;

def __nvvm_i2f_rn : NVPTXBuiltin<"float(int)">;
def __nvvm_i2f_rz : NVPTXBuiltin<"float(int)">;
def __nvvm_i2f_rm : NVPTXBuiltin<"float(int)">;
def __nvvm_i2f_rp : NVPTXBuiltin<"float(int)">;

def __nvvm_ui2f_rn : NVPTXBuiltin<"float(unsigned int)">;
def __nvvm_ui2f_rz : NVPTXBuiltin<"float(unsigned int)">;
def __nvvm_ui2f_rm : NVPTXBuiltin<"float(unsigned int)">;
def __nvvm_ui2f_rp : NVPTXBuiltin<"float(unsigned int)">;

def __nvvm_lohi_i2d : NVPTXBuiltin<"double(int, int)">;
````
- **L553 EN**: Declares TableGen def record `__nvvm_f2i_rm`.
  **L553 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rm`。
- **L554 EN**: Declares TableGen def record `__nvvm_f2i_rp_ftz`.
  **L554 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rp_ftz`。
- **L555 EN**: Declares TableGen def record `__nvvm_f2i_rp`.
  **L555 CN**: 声明 TableGen def 记录 `__nvvm_f2i_rp`。
- **L556 EN**: Blank line separating nearby declarations or logic blocks.
  **L556 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L557 EN**: Declares TableGen def record `__nvvm_f2ui_rn_ftz`.
  **L557 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rn_ftz`。
- **L558 EN**: Declares TableGen def record `__nvvm_f2ui_rn`.
  **L558 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rn`。
- **L559 EN**: Declares TableGen def record `__nvvm_f2ui_rz_ftz`.
  **L559 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rz_ftz`。
- **L560 EN**: Declares TableGen def record `__nvvm_f2ui_rz`.
  **L560 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rz`。
- **L561 EN**: Declares TableGen def record `__nvvm_f2ui_rm_ftz`.
  **L561 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rm_ftz`。
- **L562 EN**: Declares TableGen def record `__nvvm_f2ui_rm`.
  **L562 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rm`。
- **L563 EN**: Declares TableGen def record `__nvvm_f2ui_rp_ftz`.
  **L563 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rp_ftz`。
- **L564 EN**: Declares TableGen def record `__nvvm_f2ui_rp`.
  **L564 CN**: 声明 TableGen def 记录 `__nvvm_f2ui_rp`。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L566 EN**: Declares TableGen def record `__nvvm_i2f_rn`.
  **L566 CN**: 声明 TableGen def 记录 `__nvvm_i2f_rn`。
- **L567 EN**: Declares TableGen def record `__nvvm_i2f_rz`.
  **L567 CN**: 声明 TableGen def 记录 `__nvvm_i2f_rz`。
- **L568 EN**: Declares TableGen def record `__nvvm_i2f_rm`.
  **L568 CN**: 声明 TableGen def 记录 `__nvvm_i2f_rm`。
- **L569 EN**: Declares TableGen def record `__nvvm_i2f_rp`.
  **L569 CN**: 声明 TableGen def 记录 `__nvvm_i2f_rp`。
- **L570 EN**: Blank line separating nearby declarations or logic blocks.
  **L570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L571 EN**: Declares TableGen def record `__nvvm_ui2f_rn`.
  **L571 CN**: 声明 TableGen def 记录 `__nvvm_ui2f_rn`。
- **L572 EN**: Declares TableGen def record `__nvvm_ui2f_rz`.
  **L572 CN**: 声明 TableGen def 记录 `__nvvm_ui2f_rz`。
- **L573 EN**: Declares TableGen def record `__nvvm_ui2f_rm`.
  **L573 CN**: 声明 TableGen def 记录 `__nvvm_ui2f_rm`。
- **L574 EN**: Declares TableGen def record `__nvvm_ui2f_rp`.
  **L574 CN**: 声明 TableGen def 记录 `__nvvm_ui2f_rp`。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L576 EN**: Declares TableGen def record `__nvvm_lohi_i2d`.
  **L576 CN**: 声明 TableGen def 记录 `__nvvm_lohi_i2d`。

### Lines 577-600

````tablegen

def __nvvm_d2i_lo : NVPTXBuiltin<"int(double)">;
def __nvvm_d2i_hi : NVPTXBuiltin<"int(double)">;

def __nvvm_f2ll_rn_ftz : NVPTXBuiltin<"long long int(float)">;
def __nvvm_f2ll_rn : NVPTXBuiltin<"long long int(float)">;
def __nvvm_f2ll_rz_ftz : NVPTXBuiltin<"long long int(float)">;
def __nvvm_f2ll_rz : NVPTXBuiltin<"long long int(float)">;
def __nvvm_f2ll_rm_ftz : NVPTXBuiltin<"long long int(float)">;
def __nvvm_f2ll_rm : NVPTXBuiltin<"long long int(float)">;
def __nvvm_f2ll_rp_ftz : NVPTXBuiltin<"long long int(float)">;
def __nvvm_f2ll_rp : NVPTXBuiltin<"long long int(float)">;

def __nvvm_f2ull_rn_ftz : NVPTXBuiltin<"unsigned long long int(float)">;
def __nvvm_f2ull_rn : NVPTXBuiltin<"unsigned long long int(float)">;
def __nvvm_f2ull_rz_ftz : NVPTXBuiltin<"unsigned long long int(float)">;
def __nvvm_f2ull_rz : NVPTXBuiltin<"unsigned long long int(float)">;
def __nvvm_f2ull_rm_ftz : NVPTXBuiltin<"unsigned long long int(float)">;
def __nvvm_f2ull_rm : NVPTXBuiltin<"unsigned long long int(float)">;
def __nvvm_f2ull_rp_ftz : NVPTXBuiltin<"unsigned long long int(float)">;
def __nvvm_f2ull_rp : NVPTXBuiltin<"unsigned long long int(float)">;

def __nvvm_d2ll_rn : NVPTXBuiltin<"long long int(double)">;
def __nvvm_d2ll_rz : NVPTXBuiltin<"long long int(double)">;
````
- **L577 EN**: Blank line separating nearby declarations or logic blocks.
  **L577 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L578 EN**: Declares TableGen def record `__nvvm_d2i_lo`.
  **L578 CN**: 声明 TableGen def 记录 `__nvvm_d2i_lo`。
- **L579 EN**: Declares TableGen def record `__nvvm_d2i_hi`.
  **L579 CN**: 声明 TableGen def 记录 `__nvvm_d2i_hi`。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L581 EN**: Declares TableGen def record `__nvvm_f2ll_rn_ftz`.
  **L581 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rn_ftz`。
- **L582 EN**: Declares TableGen def record `__nvvm_f2ll_rn`.
  **L582 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rn`。
- **L583 EN**: Declares TableGen def record `__nvvm_f2ll_rz_ftz`.
  **L583 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rz_ftz`。
- **L584 EN**: Declares TableGen def record `__nvvm_f2ll_rz`.
  **L584 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rz`。
- **L585 EN**: Declares TableGen def record `__nvvm_f2ll_rm_ftz`.
  **L585 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rm_ftz`。
- **L586 EN**: Declares TableGen def record `__nvvm_f2ll_rm`.
  **L586 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rm`。
- **L587 EN**: Declares TableGen def record `__nvvm_f2ll_rp_ftz`.
  **L587 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rp_ftz`。
- **L588 EN**: Declares TableGen def record `__nvvm_f2ll_rp`.
  **L588 CN**: 声明 TableGen def 记录 `__nvvm_f2ll_rp`。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L590 EN**: Declares TableGen def record `__nvvm_f2ull_rn_ftz`.
  **L590 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rn_ftz`。
- **L591 EN**: Declares TableGen def record `__nvvm_f2ull_rn`.
  **L591 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rn`。
- **L592 EN**: Declares TableGen def record `__nvvm_f2ull_rz_ftz`.
  **L592 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rz_ftz`。
- **L593 EN**: Declares TableGen def record `__nvvm_f2ull_rz`.
  **L593 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rz`。
- **L594 EN**: Declares TableGen def record `__nvvm_f2ull_rm_ftz`.
  **L594 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rm_ftz`。
- **L595 EN**: Declares TableGen def record `__nvvm_f2ull_rm`.
  **L595 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rm`。
- **L596 EN**: Declares TableGen def record `__nvvm_f2ull_rp_ftz`.
  **L596 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rp_ftz`。
- **L597 EN**: Declares TableGen def record `__nvvm_f2ull_rp`.
  **L597 CN**: 声明 TableGen def 记录 `__nvvm_f2ull_rp`。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L599 EN**: Declares TableGen def record `__nvvm_d2ll_rn`.
  **L599 CN**: 声明 TableGen def 记录 `__nvvm_d2ll_rn`。
- **L600 EN**: Declares TableGen def record `__nvvm_d2ll_rz`.
  **L600 CN**: 声明 TableGen def 记录 `__nvvm_d2ll_rz`。

### Lines 601-624

````tablegen
def __nvvm_d2ll_rm : NVPTXBuiltin<"long long int(double)">;
def __nvvm_d2ll_rp : NVPTXBuiltin<"long long int(double)">;

def __nvvm_d2ull_rn : NVPTXBuiltin<"unsigned long long int(double)">;
def __nvvm_d2ull_rz : NVPTXBuiltin<"unsigned long long int(double)">;
def __nvvm_d2ull_rm : NVPTXBuiltin<"unsigned long long int(double)">;
def __nvvm_d2ull_rp : NVPTXBuiltin<"unsigned long long int(double)">;

def __nvvm_ll2f_rn : NVPTXBuiltin<"float(long long int)">;
def __nvvm_ll2f_rz : NVPTXBuiltin<"float(long long int)">;
def __nvvm_ll2f_rm : NVPTXBuiltin<"float(long long int)">;
def __nvvm_ll2f_rp : NVPTXBuiltin<"float(long long int)">;

def __nvvm_ull2f_rn : NVPTXBuiltin<"float(unsigned long long int)">;
def __nvvm_ull2f_rz : NVPTXBuiltin<"float(unsigned long long int)">;
def __nvvm_ull2f_rm : NVPTXBuiltin<"float(unsigned long long int)">;
def __nvvm_ull2f_rp : NVPTXBuiltin<"float(unsigned long long int)">;

def __nvvm_ll2d_rn : NVPTXBuiltin<"double(long long int)">;
def __nvvm_ll2d_rz : NVPTXBuiltin<"double(long long int)">;
def __nvvm_ll2d_rm : NVPTXBuiltin<"double(long long int)">;
def __nvvm_ll2d_rp : NVPTXBuiltin<"double(long long int)">;

def __nvvm_ull2d_rn : NVPTXBuiltin<"double(unsigned long long int)">;
````
- **L601 EN**: Declares TableGen def record `__nvvm_d2ll_rm`.
  **L601 CN**: 声明 TableGen def 记录 `__nvvm_d2ll_rm`。
- **L602 EN**: Declares TableGen def record `__nvvm_d2ll_rp`.
  **L602 CN**: 声明 TableGen def 记录 `__nvvm_d2ll_rp`。
- **L603 EN**: Blank line separating nearby declarations or logic blocks.
  **L603 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L604 EN**: Declares TableGen def record `__nvvm_d2ull_rn`.
  **L604 CN**: 声明 TableGen def 记录 `__nvvm_d2ull_rn`。
- **L605 EN**: Declares TableGen def record `__nvvm_d2ull_rz`.
  **L605 CN**: 声明 TableGen def 记录 `__nvvm_d2ull_rz`。
- **L606 EN**: Declares TableGen def record `__nvvm_d2ull_rm`.
  **L606 CN**: 声明 TableGen def 记录 `__nvvm_d2ull_rm`。
- **L607 EN**: Declares TableGen def record `__nvvm_d2ull_rp`.
  **L607 CN**: 声明 TableGen def 记录 `__nvvm_d2ull_rp`。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Declares TableGen def record `__nvvm_ll2f_rn`.
  **L609 CN**: 声明 TableGen def 记录 `__nvvm_ll2f_rn`。
- **L610 EN**: Declares TableGen def record `__nvvm_ll2f_rz`.
  **L610 CN**: 声明 TableGen def 记录 `__nvvm_ll2f_rz`。
- **L611 EN**: Declares TableGen def record `__nvvm_ll2f_rm`.
  **L611 CN**: 声明 TableGen def 记录 `__nvvm_ll2f_rm`。
- **L612 EN**: Declares TableGen def record `__nvvm_ll2f_rp`.
  **L612 CN**: 声明 TableGen def 记录 `__nvvm_ll2f_rp`。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Declares TableGen def record `__nvvm_ull2f_rn`.
  **L614 CN**: 声明 TableGen def 记录 `__nvvm_ull2f_rn`。
- **L615 EN**: Declares TableGen def record `__nvvm_ull2f_rz`.
  **L615 CN**: 声明 TableGen def 记录 `__nvvm_ull2f_rz`。
- **L616 EN**: Declares TableGen def record `__nvvm_ull2f_rm`.
  **L616 CN**: 声明 TableGen def 记录 `__nvvm_ull2f_rm`。
- **L617 EN**: Declares TableGen def record `__nvvm_ull2f_rp`.
  **L617 CN**: 声明 TableGen def 记录 `__nvvm_ull2f_rp`。
- **L618 EN**: Blank line separating nearby declarations or logic blocks.
  **L618 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L619 EN**: Declares TableGen def record `__nvvm_ll2d_rn`.
  **L619 CN**: 声明 TableGen def 记录 `__nvvm_ll2d_rn`。
- **L620 EN**: Declares TableGen def record `__nvvm_ll2d_rz`.
  **L620 CN**: 声明 TableGen def 记录 `__nvvm_ll2d_rz`。
- **L621 EN**: Declares TableGen def record `__nvvm_ll2d_rm`.
  **L621 CN**: 声明 TableGen def 记录 `__nvvm_ll2d_rm`。
- **L622 EN**: Declares TableGen def record `__nvvm_ll2d_rp`.
  **L622 CN**: 声明 TableGen def 记录 `__nvvm_ll2d_rp`。
- **L623 EN**: Blank line separating nearby declarations or logic blocks.
  **L623 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L624 EN**: Declares TableGen def record `__nvvm_ull2d_rn`.
  **L624 CN**: 声明 TableGen def 记录 `__nvvm_ull2d_rn`。

### Lines 625-648

````tablegen
def __nvvm_ull2d_rz : NVPTXBuiltin<"double(unsigned long long int)">;
def __nvvm_ull2d_rm : NVPTXBuiltin<"double(unsigned long long int)">;
def __nvvm_ull2d_rp : NVPTXBuiltin<"double(unsigned long long int)">;

def __nvvm_f2h_rn_ftz : NVPTXBuiltin<"unsigned short(float)">;
def __nvvm_f2h_rn : NVPTXBuiltin<"unsigned short(float)">;

def __nvvm_ff2bf16x2_rn : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2bf16x2_rn_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2bf16x2_rz : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2bf16x2_rz_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2bf16x2_rn_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2bf16x2_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2bf16x2_rz_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2bf16x2_rz_relu_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2bf16x2_rs :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
def __nvvm_ff2bf16x2_rs_relu :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
def __nvvm_ff2bf16x2_rs_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
````
- **L625 EN**: Declares TableGen def record `__nvvm_ull2d_rz`.
  **L625 CN**: 声明 TableGen def 记录 `__nvvm_ull2d_rz`。
- **L626 EN**: Declares TableGen def record `__nvvm_ull2d_rm`.
  **L626 CN**: 声明 TableGen def 记录 `__nvvm_ull2d_rm`。
- **L627 EN**: Declares TableGen def record `__nvvm_ull2d_rp`.
  **L627 CN**: 声明 TableGen def 记录 `__nvvm_ull2d_rp`。
- **L628 EN**: Blank line separating nearby declarations or logic blocks.
  **L628 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L629 EN**: Declares TableGen def record `__nvvm_f2h_rn_ftz`.
  **L629 CN**: 声明 TableGen def 记录 `__nvvm_f2h_rn_ftz`。
- **L630 EN**: Declares TableGen def record `__nvvm_f2h_rn`.
  **L630 CN**: 声明 TableGen def 记录 `__nvvm_f2h_rn`。
- **L631 EN**: Blank line separating nearby declarations or logic blocks.
  **L631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L632 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rn`.
  **L632 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rn`。
- **L633 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rn_relu`.
  **L633 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rn_relu`。
- **L634 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rz`.
  **L634 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rz`。
- **L635 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rz_relu`.
  **L635 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rz_relu`。
- **L636 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rn_satfinite`.
  **L636 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rn_satfinite`。
- **L637 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rn_relu_satfinite`.
  **L637 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rn_relu_satfinite`。
- **L638 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rz_satfinite`.
  **L638 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rz_satfinite`。
- **L639 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rz_relu_satfinite`.
  **L639 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rz_relu_satfinite`。
- **L640 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rs`.
  **L640 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rs`。
- **L641 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`.
  **L641 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`。
- **L642 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L642 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L643 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rs_relu`.
  **L643 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rs_relu`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`。
- **L645 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L645 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L646 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rs_satfinite`.
  **L646 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rs_satfinite`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`。
- **L648 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L648 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。

### Lines 649-672

````tablegen
def __nvvm_ff2bf16x2_rs_relu_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;

def __nvvm_ff2f16x2_rn : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2f16x2_rn_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2f16x2_rz : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2f16x2_rz_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX70>;
def __nvvm_ff2f16x2_rn_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2f16x2_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2f16x2_rz_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2f16x2_rz_relu_satfinite : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float)", SM_80, PTX81>;
def __nvvm_ff2f16x2_rs :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
def __nvvm_ff2f16x2_rs_relu :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
def __nvvm_ff2f16x2_rs_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
def __nvvm_ff2f16x2_rs_relu_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
````
- **L649 EN**: Declares TableGen def record `__nvvm_ff2bf16x2_rs_relu_satfinite`.
  **L649 CN**: 声明 TableGen def 记录 `__nvvm_ff2bf16x2_rs_relu_satfinite`。
- **L650 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`.
  **L650 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(float, float, uint32_t)",`。
- **L651 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L651 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L653 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rn`.
  **L653 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rn`。
- **L654 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rn_relu`.
  **L654 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rn_relu`。
- **L655 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rz`.
  **L655 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rz`。
- **L656 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rz_relu`.
  **L656 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rz_relu`。
- **L657 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rn_satfinite`.
  **L657 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rn_satfinite`。
- **L658 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rn_relu_satfinite`.
  **L658 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rn_relu_satfinite`。
- **L659 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rz_satfinite`.
  **L659 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rz_satfinite`。
- **L660 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rz_relu_satfinite`.
  **L660 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rz_relu_satfinite`。
- **L661 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rs`.
  **L661 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rs`。
- **L662 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`.
  **L662 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`。
- **L663 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L663 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L664 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rs_relu`.
  **L664 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rs_relu`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`。
- **L666 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L666 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L667 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rs_satfinite`.
  **L667 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rs_satfinite`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`。
- **L669 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L669 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L670 EN**: Declares TableGen def record `__nvvm_ff2f16x2_rs_relu_satfinite`.
  **L670 CN**: 声明 TableGen def 记录 `__nvvm_ff2f16x2_rs_relu_satfinite`。
- **L671 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`.
  **L671 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(float, float, uint32_t)",`。
- **L672 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L672 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。

### Lines 673-696

````tablegen

def __nvvm_f2bf16_rn : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX70>;
def __nvvm_f2bf16_rn_relu : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX70>;
def __nvvm_f2bf16_rz : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX70>;
def __nvvm_f2bf16_rz_relu : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX70>;
def __nvvm_f2bf16_rn_satfinite : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX81>;
def __nvvm_f2bf16_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX81>;
def __nvvm_f2bf16_rz_satfinite : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX81>;
def __nvvm_f2bf16_rz_relu_satfinite : NVPTXBuiltinSMAndPTX<"__bf16(float)", SM_80, PTX81>;

def __nvvm_f2f16_rn : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX70>;
def __nvvm_f2f16_rn_relu : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX70>;
def __nvvm_f2f16_rz : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX70>;
def __nvvm_f2f16_rz_relu : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX70>;
def __nvvm_f2f16_rn_satfinite : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX81>;
def __nvvm_f2f16_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX81>;
def __nvvm_f2f16_rz_satfinite : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX81>;
def __nvvm_f2f16_rz_relu_satfinite : NVPTXBuiltinSMAndPTX<"__fp16(float)", SM_80, PTX81>;

def __nvvm_f2tf32_rna : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_80, PTX70>;
def __nvvm_f2tf32_rna_satfinite : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_80, PTX81>;
def __nvvm_f2tf32_rn : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_90, PTX78>;
def __nvvm_f2tf32_rn_relu : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_90, PTX78>;
def __nvvm_f2tf32_rn_satfinite : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_100, PTX86>;
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Declares TableGen def record `__nvvm_f2bf16_rn`.
  **L674 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rn`。
- **L675 EN**: Declares TableGen def record `__nvvm_f2bf16_rn_relu`.
  **L675 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rn_relu`。
- **L676 EN**: Declares TableGen def record `__nvvm_f2bf16_rz`.
  **L676 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rz`。
- **L677 EN**: Declares TableGen def record `__nvvm_f2bf16_rz_relu`.
  **L677 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rz_relu`。
- **L678 EN**: Declares TableGen def record `__nvvm_f2bf16_rn_satfinite`.
  **L678 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rn_satfinite`。
- **L679 EN**: Declares TableGen def record `__nvvm_f2bf16_rn_relu_satfinite`.
  **L679 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rn_relu_satfinite`。
- **L680 EN**: Declares TableGen def record `__nvvm_f2bf16_rz_satfinite`.
  **L680 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rz_satfinite`。
- **L681 EN**: Declares TableGen def record `__nvvm_f2bf16_rz_relu_satfinite`.
  **L681 CN**: 声明 TableGen def 记录 `__nvvm_f2bf16_rz_relu_satfinite`。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L683 EN**: Declares TableGen def record `__nvvm_f2f16_rn`.
  **L683 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rn`。
- **L684 EN**: Declares TableGen def record `__nvvm_f2f16_rn_relu`.
  **L684 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rn_relu`。
- **L685 EN**: Declares TableGen def record `__nvvm_f2f16_rz`.
  **L685 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rz`。
- **L686 EN**: Declares TableGen def record `__nvvm_f2f16_rz_relu`.
  **L686 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rz_relu`。
- **L687 EN**: Declares TableGen def record `__nvvm_f2f16_rn_satfinite`.
  **L687 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rn_satfinite`。
- **L688 EN**: Declares TableGen def record `__nvvm_f2f16_rn_relu_satfinite`.
  **L688 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rn_relu_satfinite`。
- **L689 EN**: Declares TableGen def record `__nvvm_f2f16_rz_satfinite`.
  **L689 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rz_satfinite`。
- **L690 EN**: Declares TableGen def record `__nvvm_f2f16_rz_relu_satfinite`.
  **L690 CN**: 声明 TableGen def 记录 `__nvvm_f2f16_rz_relu_satfinite`。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Declares TableGen def record `__nvvm_f2tf32_rna`.
  **L692 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rna`。
- **L693 EN**: Declares TableGen def record `__nvvm_f2tf32_rna_satfinite`.
  **L693 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rna_satfinite`。
- **L694 EN**: Declares TableGen def record `__nvvm_f2tf32_rn`.
  **L694 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rn`。
- **L695 EN**: Declares TableGen def record `__nvvm_f2tf32_rn_relu`.
  **L695 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rn_relu`。
- **L696 EN**: Declares TableGen def record `__nvvm_f2tf32_rn_satfinite`.
  **L696 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rn_satfinite`。

### Lines 697-720

````tablegen
def __nvvm_f2tf32_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_100, PTX86>;
def __nvvm_f2tf32_rz : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_90, PTX78>;
def __nvvm_f2tf32_rz_relu : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_90, PTX78>;
def __nvvm_f2tf32_rz_satfinite : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_100, PTX86>;
def __nvvm_f2tf32_rz_relu_satfinite : NVPTXBuiltinSMAndPTX<"int32_t(float)", SM_100, PTX86>;

def __nvvm_ff_to_e4m3x2_rn : NVPTXBuiltinSMAndPTX<"short(float, float)", SM_89, PTX81>;
def __nvvm_ff_to_e4m3x2_rn_relu : NVPTXBuiltinSMAndPTX<"short(float, float)", SM_89, PTX81>;
def __nvvm_ff_to_e5m2x2_rn : NVPTXBuiltinSMAndPTX<"short(float, float)", SM_89, PTX81>;
def __nvvm_ff_to_e5m2x2_rn_relu : NVPTXBuiltinSMAndPTX<"short(float, float)", SM_89, PTX81>;

def __nvvm_f16x2_to_e4m3x2_rn : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __fp16>)", SM_89, PTX81>;
def __nvvm_f16x2_to_e4m3x2_rn_relu : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __fp16>)", SM_89, PTX81>;
def __nvvm_f16x2_to_e5m2x2_rn : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __fp16>)", SM_89, PTX81>;
def __nvvm_f16x2_to_e5m2x2_rn_relu : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __fp16>)", SM_89, PTX81>;

def __nvvm_e4m3x2_to_f16x2_rn : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SM_89, PTX81>;
def __nvvm_e4m3x2_to_f16x2_rn_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SM_89, PTX81>;
def __nvvm_e5m2x2_to_f16x2_rn : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SM_89, PTX81>;
def __nvvm_e5m2x2_to_f16x2_rn_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SM_89, PTX81>;

def __nvvm_f32x4_to_e4m3x4_rs_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;
````
- **L697 EN**: Declares TableGen def record `__nvvm_f2tf32_rn_relu_satfinite`.
  **L697 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rn_relu_satfinite`。
- **L698 EN**: Declares TableGen def record `__nvvm_f2tf32_rz`.
  **L698 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rz`。
- **L699 EN**: Declares TableGen def record `__nvvm_f2tf32_rz_relu`.
  **L699 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rz_relu`。
- **L700 EN**: Declares TableGen def record `__nvvm_f2tf32_rz_satfinite`.
  **L700 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rz_satfinite`。
- **L701 EN**: Declares TableGen def record `__nvvm_f2tf32_rz_relu_satfinite`.
  **L701 CN**: 声明 TableGen def 记录 `__nvvm_f2tf32_rz_relu_satfinite`。
- **L702 EN**: Blank line separating nearby declarations or logic blocks.
  **L702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L703 EN**: Declares TableGen def record `__nvvm_ff_to_e4m3x2_rn`.
  **L703 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e4m3x2_rn`。
- **L704 EN**: Declares TableGen def record `__nvvm_ff_to_e4m3x2_rn_relu`.
  **L704 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e4m3x2_rn_relu`。
- **L705 EN**: Declares TableGen def record `__nvvm_ff_to_e5m2x2_rn`.
  **L705 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e5m2x2_rn`。
- **L706 EN**: Declares TableGen def record `__nvvm_ff_to_e5m2x2_rn_relu`.
  **L706 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e5m2x2_rn_relu`。
- **L707 EN**: Blank line separating nearby declarations or logic blocks.
  **L707 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L708 EN**: Declares TableGen def record `__nvvm_f16x2_to_e4m3x2_rn`.
  **L708 CN**: 声明 TableGen def 记录 `__nvvm_f16x2_to_e4m3x2_rn`。
- **L709 EN**: Declares TableGen def record `__nvvm_f16x2_to_e4m3x2_rn_relu`.
  **L709 CN**: 声明 TableGen def 记录 `__nvvm_f16x2_to_e4m3x2_rn_relu`。
- **L710 EN**: Declares TableGen def record `__nvvm_f16x2_to_e5m2x2_rn`.
  **L710 CN**: 声明 TableGen def 记录 `__nvvm_f16x2_to_e5m2x2_rn`。
- **L711 EN**: Declares TableGen def record `__nvvm_f16x2_to_e5m2x2_rn_relu`.
  **L711 CN**: 声明 TableGen def 记录 `__nvvm_f16x2_to_e5m2x2_rn_relu`。
- **L712 EN**: Blank line separating nearby declarations or logic blocks.
  **L712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L713 EN**: Declares TableGen def record `__nvvm_e4m3x2_to_f16x2_rn`.
  **L713 CN**: 声明 TableGen def 记录 `__nvvm_e4m3x2_to_f16x2_rn`。
- **L714 EN**: Declares TableGen def record `__nvvm_e4m3x2_to_f16x2_rn_relu`.
  **L714 CN**: 声明 TableGen def 记录 `__nvvm_e4m3x2_to_f16x2_rn_relu`。
- **L715 EN**: Declares TableGen def record `__nvvm_e5m2x2_to_f16x2_rn`.
  **L715 CN**: 声明 TableGen def 记录 `__nvvm_e5m2x2_to_f16x2_rn`。
- **L716 EN**: Declares TableGen def record `__nvvm_e5m2x2_to_f16x2_rn_relu`.
  **L716 CN**: 声明 TableGen def 记录 `__nvvm_e5m2x2_to_f16x2_rn_relu`。
- **L717 EN**: Blank line separating nearby declarations or logic blocks.
  **L717 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L718 EN**: Declares TableGen def record `__nvvm_f32x4_to_e4m3x4_rs_satfinite`.
  **L718 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e4m3x4_rs_satfinite`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L720 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L720 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。

### Lines 721-744

````tablegen
def __nvvm_f32x4_to_e4m3x4_rs_relu_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;
def __nvvm_f32x4_to_e5m2x4_rs_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;
def __nvvm_f32x4_to_e5m2x4_rs_relu_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;

def __nvvm_ff_to_e2m3x2_rn_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_ff_to_e2m3x2_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_ff_to_e3m2x2_rn_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_ff_to_e3m2x2_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;

def __nvvm_e2m3x2_to_f16x2_rn : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_e2m3x2_to_f16x2_rn_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_e3m2x2_to_f16x2_rn : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_e3m2x2_to_f16x2_rn_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SMa<[100, 101, 120]>, PTX86>;

def __nvvm_f32x4_to_e2m3x4_rs_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;
def __nvvm_f32x4_to_e2m3x4_rs_relu_satfinite :
````
- **L721 EN**: Declares TableGen def record `__nvvm_f32x4_to_e4m3x4_rs_relu_satfinite`.
  **L721 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e4m3x4_rs_relu_satfinite`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L723 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L723 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L724 EN**: Declares TableGen def record `__nvvm_f32x4_to_e5m2x4_rs_satfinite`.
  **L724 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e5m2x4_rs_satfinite`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L726 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L726 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L727 EN**: Declares TableGen def record `__nvvm_f32x4_to_e5m2x4_rs_relu_satfinite`.
  **L727 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e5m2x4_rs_relu_satfinite`。
- **L728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L728 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L729 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L729 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L730 EN**: Blank line separating nearby declarations or logic blocks.
  **L730 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L731 EN**: Declares TableGen def record `__nvvm_ff_to_e2m3x2_rn_satfinite`.
  **L731 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e2m3x2_rn_satfinite`。
- **L732 EN**: Declares TableGen def record `__nvvm_ff_to_e2m3x2_rn_relu_satfinite`.
  **L732 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e2m3x2_rn_relu_satfinite`。
- **L733 EN**: Declares TableGen def record `__nvvm_ff_to_e3m2x2_rn_satfinite`.
  **L733 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e3m2x2_rn_satfinite`。
- **L734 EN**: Declares TableGen def record `__nvvm_ff_to_e3m2x2_rn_relu_satfinite`.
  **L734 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e3m2x2_rn_relu_satfinite`。
- **L735 EN**: Blank line separating nearby declarations or logic blocks.
  **L735 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L736 EN**: Declares TableGen def record `__nvvm_e2m3x2_to_f16x2_rn`.
  **L736 CN**: 声明 TableGen def 记录 `__nvvm_e2m3x2_to_f16x2_rn`。
- **L737 EN**: Declares TableGen def record `__nvvm_e2m3x2_to_f16x2_rn_relu`.
  **L737 CN**: 声明 TableGen def 记录 `__nvvm_e2m3x2_to_f16x2_rn_relu`。
- **L738 EN**: Declares TableGen def record `__nvvm_e3m2x2_to_f16x2_rn`.
  **L738 CN**: 声明 TableGen def 记录 `__nvvm_e3m2x2_to_f16x2_rn`。
- **L739 EN**: Declares TableGen def record `__nvvm_e3m2x2_to_f16x2_rn_relu`.
  **L739 CN**: 声明 TableGen def 记录 `__nvvm_e3m2x2_to_f16x2_rn_relu`。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Declares TableGen def record `__nvvm_f32x4_to_e2m3x4_rs_satfinite`.
  **L741 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e2m3x4_rs_satfinite`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L743 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L743 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L744 EN**: Declares TableGen def record `__nvvm_f32x4_to_e2m3x4_rs_relu_satfinite`.
  **L744 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e2m3x4_rs_relu_satfinite`。

### Lines 745-768

````tablegen
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;
def __nvvm_f32x4_to_e3m2x4_rs_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;
def __nvvm_f32x4_to_e3m2x4_rs_relu_satfinite :
  NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)", 
                       SMa<[100, 103]>, PTX87>;

def __nvvm_ff_to_e2m1x2_rn_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_ff_to_e2m1x2_rn_relu_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;

def __nvvm_e2m1x2_to_f16x2_rn : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_e2m1x2_to_f16x2_rn_relu : NVPTXBuiltinSMAndPTX<"_Vector<2, __fp16>(short)", SMa<[100, 101, 120]>, PTX86>;

def __nvvm_f32x4_to_e2m1x4_rs_satfinite :
  NVPTXBuiltinSMAndPTX<"short(_Vector<4, float>, uint32_t)",
                       SMa<[100, 103]>, PTX87>;
def __nvvm_f32x4_to_e2m1x4_rs_relu_satfinite :
  NVPTXBuiltinSMAndPTX<"short(_Vector<4, float>, uint32_t)",
                       SMa<[100, 103]>, PTX87>;

def __nvvm_ff_to_ue8m0x2_rz : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_ff_to_ue8m0x2_rz_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;
````
- **L745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L745 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L746 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L746 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L747 EN**: Declares TableGen def record `__nvvm_f32x4_to_e3m2x4_rs_satfinite`.
  **L747 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e3m2x4_rs_satfinite`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L749 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L749 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L750 EN**: Declares TableGen def record `__nvvm_f32x4_to_e3m2x4_rs_relu_satfinite`.
  **L750 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e3m2x4_rs_relu_satfinite`。
- **L751 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`.
  **L751 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"_Vector<4, char>(_Vector<4, float>, uint32_t)",`。
- **L752 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L752 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L753 EN**: Blank line separating nearby declarations or logic blocks.
  **L753 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L754 EN**: Declares TableGen def record `__nvvm_ff_to_e2m1x2_rn_satfinite`.
  **L754 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e2m1x2_rn_satfinite`。
- **L755 EN**: Declares TableGen def record `__nvvm_ff_to_e2m1x2_rn_relu_satfinite`.
  **L755 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_e2m1x2_rn_relu_satfinite`。
- **L756 EN**: Blank line separating nearby declarations or logic blocks.
  **L756 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L757 EN**: Declares TableGen def record `__nvvm_e2m1x2_to_f16x2_rn`.
  **L757 CN**: 声明 TableGen def 记录 `__nvvm_e2m1x2_to_f16x2_rn`。
- **L758 EN**: Declares TableGen def record `__nvvm_e2m1x2_to_f16x2_rn_relu`.
  **L758 CN**: 声明 TableGen def 记录 `__nvvm_e2m1x2_to_f16x2_rn_relu`。
- **L759 EN**: Blank line separating nearby declarations or logic blocks.
  **L759 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L760 EN**: Declares TableGen def record `__nvvm_f32x4_to_e2m1x4_rs_satfinite`.
  **L760 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e2m1x4_rs_satfinite`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"short(_Vector<4, float>, uint32_t)",`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"short(_Vector<4, float>, uint32_t)",`。
- **L762 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L762 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L763 EN**: Declares TableGen def record `__nvvm_f32x4_to_e2m1x4_rs_relu_satfinite`.
  **L763 CN**: 声明 TableGen def 记录 `__nvvm_f32x4_to_e2m1x4_rs_relu_satfinite`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVPTXBuiltinSMAndPTX<"short(_Vector<4, float>, uint32_t)",`.
  **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVPTXBuiltinSMAndPTX<"short(_Vector<4, float>, uint32_t)",`。
- **L765 EN**: Adds a standalone statement or declaration: `SMa<[100, 103]>, PTX87>;`.
  **L765 CN**: 添加一条独立语句或声明：`SMa<[100, 103]>, PTX87>;`。
- **L766 EN**: Blank line separating nearby declarations or logic blocks.
  **L766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L767 EN**: Declares TableGen def record `__nvvm_ff_to_ue8m0x2_rz`.
  **L767 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_ue8m0x2_rz`。
- **L768 EN**: Declares TableGen def record `__nvvm_ff_to_ue8m0x2_rz_satfinite`.
  **L768 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_ue8m0x2_rz_satfinite`。

### Lines 769-792

````tablegen
def __nvvm_ff_to_ue8m0x2_rp : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_ff_to_ue8m0x2_rp_satfinite : NVPTXBuiltinSMAndPTX<"short(float, float)", SMa<[100, 101, 120]>, PTX86>;

def __nvvm_bf16x2_to_ue8m0x2_rz : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __bf16>)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_bf16x2_to_ue8m0x2_rz_satfinite : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __bf16>)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_bf16x2_to_ue8m0x2_rp : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __bf16>)", SMa<[100, 101, 120]>, PTX86>;
def __nvvm_bf16x2_to_ue8m0x2_rp_satfinite : NVPTXBuiltinSMAndPTX<"short(_Vector<2, __bf16>)", SMa<[100, 101, 120]>, PTX86>;

def __nvvm_ue8m0x2_to_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(short)", SMa<[100, 101, 120]>, PTX86>;

// FNS
let Attributes = [NoThrow] in {
  def __nvvm_fns : NVPTXBuiltinPTX<"unsigned int(unsigned int, unsigned int, int)", PTX60>;
}

// Sync

def __syncthreads : NVPTXBuiltin<"void()">;
def __nvvm_bar0_popc : NVPTXBuiltin<"int(int)">;
def __nvvm_bar0_and : NVPTXBuiltin<"int(int)">;
def __nvvm_bar0_or : NVPTXBuiltin<"int(int)">;
let Attributes = [NoThrow] in {
  def __nvvm_bar_sync : NVPTXBuiltin<"void(int)">;
  def __nvvm_bar_warp_sync : NVPTXBuiltinPTX<"void(unsigned int)", PTX60>;
````
- **L769 EN**: Declares TableGen def record `__nvvm_ff_to_ue8m0x2_rp`.
  **L769 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_ue8m0x2_rp`。
- **L770 EN**: Declares TableGen def record `__nvvm_ff_to_ue8m0x2_rp_satfinite`.
  **L770 CN**: 声明 TableGen def 记录 `__nvvm_ff_to_ue8m0x2_rp_satfinite`。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L772 EN**: Declares TableGen def record `__nvvm_bf16x2_to_ue8m0x2_rz`.
  **L772 CN**: 声明 TableGen def 记录 `__nvvm_bf16x2_to_ue8m0x2_rz`。
- **L773 EN**: Declares TableGen def record `__nvvm_bf16x2_to_ue8m0x2_rz_satfinite`.
  **L773 CN**: 声明 TableGen def 记录 `__nvvm_bf16x2_to_ue8m0x2_rz_satfinite`。
- **L774 EN**: Declares TableGen def record `__nvvm_bf16x2_to_ue8m0x2_rp`.
  **L774 CN**: 声明 TableGen def 记录 `__nvvm_bf16x2_to_ue8m0x2_rp`。
- **L775 EN**: Declares TableGen def record `__nvvm_bf16x2_to_ue8m0x2_rp_satfinite`.
  **L775 CN**: 声明 TableGen def 记录 `__nvvm_bf16x2_to_ue8m0x2_rp_satfinite`。
- **L776 EN**: Blank line separating nearby declarations or logic blocks.
  **L776 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L777 EN**: Declares TableGen def record `__nvvm_ue8m0x2_to_bf16x2`.
  **L777 CN**: 声明 TableGen def 记录 `__nvvm_ue8m0x2_to_bf16x2`。
- **L778 EN**: Blank line separating nearby declarations or logic blocks.
  **L778 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L779 EN**: Comment explains nearby logic, constraints, or intent: `FNS`.
  **L779 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FNS`。
- **L780 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L780 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L781 EN**: Declares TableGen def record `__nvvm_fns`.
  **L781 CN**: 声明 TableGen def 记录 `__nvvm_fns`。
- **L782 EN**: Closes the current lexical scope or compound statement.
  **L782 CN**: 结束当前词法作用域或复合语句块。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L784 EN**: Comment explains nearby logic, constraints, or intent: `Sync`.
  **L784 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Sync`。
- **L785 EN**: Blank line separating nearby declarations or logic blocks.
  **L785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L786 EN**: Declares TableGen def record `__syncthreads`.
  **L786 CN**: 声明 TableGen def 记录 `__syncthreads`。
- **L787 EN**: Declares TableGen def record `__nvvm_bar0_popc`.
  **L787 CN**: 声明 TableGen def 记录 `__nvvm_bar0_popc`。
- **L788 EN**: Declares TableGen def record `__nvvm_bar0_and`.
  **L788 CN**: 声明 TableGen def 记录 `__nvvm_bar0_and`。
- **L789 EN**: Declares TableGen def record `__nvvm_bar0_or`.
  **L789 CN**: 声明 TableGen def 记录 `__nvvm_bar0_or`。
- **L790 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L790 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L791 EN**: Declares TableGen def record `__nvvm_bar_sync`.
  **L791 CN**: 声明 TableGen def 记录 `__nvvm_bar_sync`。
- **L792 EN**: Declares TableGen def record `__nvvm_bar_warp_sync`.
  **L792 CN**: 声明 TableGen def 记录 `__nvvm_bar_warp_sync`。

### Lines 793-816

````tablegen
  def __nvvm_barrier_sync : NVPTXBuiltinPTX<"void(unsigned int)", PTX60>;
  def __nvvm_barrier_sync_cnt : NVPTXBuiltinPTX<"void(unsigned int, unsigned int)", PTX60>;

  def __nvvm_barrier_cluster_arrive : NVPTXBuiltinSMAndPTX<"void()", SM_90, PTX78>;
  def __nvvm_barrier_cluster_arrive_relaxed : NVPTXBuiltinSMAndPTX<"void()", SM_90, PTX80>;
  def __nvvm_barrier_cluster_wait : NVPTXBuiltinSMAndPTX<"void()", SM_90, PTX78>;
  def __nvvm_fence_sc_cluster : NVPTXBuiltinSMAndPTX<"void()", SM_90, PTX78>;
}

// Shuffle

def __nvvm_shfl_down_i32 : NVPTXBuiltin<"int(int, int, int)">;
def __nvvm_shfl_down_f32 : NVPTXBuiltin<"float(float, int, int)">;
def __nvvm_shfl_up_i32 : NVPTXBuiltin<"int(int, int, int)">;
def __nvvm_shfl_up_f32 : NVPTXBuiltin<"float(float, int, int)">;
def __nvvm_shfl_bfly_i32 : NVPTXBuiltin<"int(int, int, int)">;
def __nvvm_shfl_bfly_f32 : NVPTXBuiltin<"float(float, int, int)">;
def __nvvm_shfl_idx_i32 : NVPTXBuiltin<"int(int, int, int)">;
def __nvvm_shfl_idx_f32 : NVPTXBuiltin<"float(float, int, int)">;

def __nvvm_shfl_sync_down_i32 : NVPTXBuiltinPTX<"int(unsigned int, int, int, int)", PTX60>;
def __nvvm_shfl_sync_down_f32 : NVPTXBuiltinPTX<"float(unsigned int, float, int, int)", PTX60>;
def __nvvm_shfl_sync_up_i32 : NVPTXBuiltinPTX<"int(unsigned int, int, int, int)", PTX60>;
def __nvvm_shfl_sync_up_f32 : NVPTXBuiltinPTX<"float(unsigned int, float, int, int)", PTX60>;
````
- **L793 EN**: Declares TableGen def record `__nvvm_barrier_sync`.
  **L793 CN**: 声明 TableGen def 记录 `__nvvm_barrier_sync`。
- **L794 EN**: Declares TableGen def record `__nvvm_barrier_sync_cnt`.
  **L794 CN**: 声明 TableGen def 记录 `__nvvm_barrier_sync_cnt`。
- **L795 EN**: Blank line separating nearby declarations or logic blocks.
  **L795 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L796 EN**: Declares TableGen def record `__nvvm_barrier_cluster_arrive`.
  **L796 CN**: 声明 TableGen def 记录 `__nvvm_barrier_cluster_arrive`。
- **L797 EN**: Declares TableGen def record `__nvvm_barrier_cluster_arrive_relaxed`.
  **L797 CN**: 声明 TableGen def 记录 `__nvvm_barrier_cluster_arrive_relaxed`。
- **L798 EN**: Declares TableGen def record `__nvvm_barrier_cluster_wait`.
  **L798 CN**: 声明 TableGen def 记录 `__nvvm_barrier_cluster_wait`。
- **L799 EN**: Declares TableGen def record `__nvvm_fence_sc_cluster`.
  **L799 CN**: 声明 TableGen def 记录 `__nvvm_fence_sc_cluster`。
- **L800 EN**: Closes the current lexical scope or compound statement.
  **L800 CN**: 结束当前词法作用域或复合语句块。
- **L801 EN**: Blank line separating nearby declarations or logic blocks.
  **L801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L802 EN**: Comment explains nearby logic, constraints, or intent: `Shuffle`.
  **L802 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Shuffle`。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L804 EN**: Declares TableGen def record `__nvvm_shfl_down_i32`.
  **L804 CN**: 声明 TableGen def 记录 `__nvvm_shfl_down_i32`。
- **L805 EN**: Declares TableGen def record `__nvvm_shfl_down_f32`.
  **L805 CN**: 声明 TableGen def 记录 `__nvvm_shfl_down_f32`。
- **L806 EN**: Declares TableGen def record `__nvvm_shfl_up_i32`.
  **L806 CN**: 声明 TableGen def 记录 `__nvvm_shfl_up_i32`。
- **L807 EN**: Declares TableGen def record `__nvvm_shfl_up_f32`.
  **L807 CN**: 声明 TableGen def 记录 `__nvvm_shfl_up_f32`。
- **L808 EN**: Declares TableGen def record `__nvvm_shfl_bfly_i32`.
  **L808 CN**: 声明 TableGen def 记录 `__nvvm_shfl_bfly_i32`。
- **L809 EN**: Declares TableGen def record `__nvvm_shfl_bfly_f32`.
  **L809 CN**: 声明 TableGen def 记录 `__nvvm_shfl_bfly_f32`。
- **L810 EN**: Declares TableGen def record `__nvvm_shfl_idx_i32`.
  **L810 CN**: 声明 TableGen def 记录 `__nvvm_shfl_idx_i32`。
- **L811 EN**: Declares TableGen def record `__nvvm_shfl_idx_f32`.
  **L811 CN**: 声明 TableGen def 记录 `__nvvm_shfl_idx_f32`。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Declares TableGen def record `__nvvm_shfl_sync_down_i32`.
  **L813 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_down_i32`。
- **L814 EN**: Declares TableGen def record `__nvvm_shfl_sync_down_f32`.
  **L814 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_down_f32`。
- **L815 EN**: Declares TableGen def record `__nvvm_shfl_sync_up_i32`.
  **L815 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_up_i32`。
- **L816 EN**: Declares TableGen def record `__nvvm_shfl_sync_up_f32`.
  **L816 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_up_f32`。

### Lines 817-840

````tablegen
def __nvvm_shfl_sync_bfly_i32 : NVPTXBuiltinPTX<"int(unsigned int, int, int, int)", PTX60>;
def __nvvm_shfl_sync_bfly_f32 : NVPTXBuiltinPTX<"float(unsigned int, float, int, int)", PTX60>;
def __nvvm_shfl_sync_idx_i32 : NVPTXBuiltinPTX<"int(unsigned int, int, int, int)", PTX60>;
def __nvvm_shfl_sync_idx_f32 : NVPTXBuiltinPTX<"float(unsigned int, float, int, int)", PTX60>;

// Vote
def __nvvm_vote_all : NVPTXBuiltin<"bool(bool)">;
def __nvvm_vote_any : NVPTXBuiltin<"bool(bool)">;
def __nvvm_vote_uni : NVPTXBuiltin<"bool(bool)">;
def __nvvm_vote_ballot : NVPTXBuiltin<"unsigned int(bool)">;

def __nvvm_vote_all_sync : NVPTXBuiltinPTX<"bool(unsigned int, bool)", PTX60>;
def __nvvm_vote_any_sync : NVPTXBuiltinPTX<"bool(unsigned int, bool)", PTX60>;
def __nvvm_vote_uni_sync : NVPTXBuiltinPTX<"bool(unsigned int, bool)", PTX60>;
def __nvvm_vote_ballot_sync : NVPTXBuiltinPTX<"unsigned int(unsigned int, bool)", PTX60>;

// Mask
let Attributes = [NoThrow] in {
  def __nvvm_activemask : NVPTXBuiltinPTX<"unsigned int()", PTX62>;
}

// Match
def __nvvm_match_any_sync_i32 : NVPTXBuiltinSMAndPTX<"unsigned int(unsigned int, unsigned int)", SM_70, PTX60>;
def __nvvm_match_any_sync_i64 : NVPTXBuiltinSMAndPTX<"unsigned int(unsigned int, int64_t)", SM_70, PTX60>;
````
- **L817 EN**: Declares TableGen def record `__nvvm_shfl_sync_bfly_i32`.
  **L817 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_bfly_i32`。
- **L818 EN**: Declares TableGen def record `__nvvm_shfl_sync_bfly_f32`.
  **L818 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_bfly_f32`。
- **L819 EN**: Declares TableGen def record `__nvvm_shfl_sync_idx_i32`.
  **L819 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_idx_i32`。
- **L820 EN**: Declares TableGen def record `__nvvm_shfl_sync_idx_f32`.
  **L820 CN**: 声明 TableGen def 记录 `__nvvm_shfl_sync_idx_f32`。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L822 EN**: Comment explains nearby logic, constraints, or intent: `Vote`.
  **L822 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vote`。
- **L823 EN**: Declares TableGen def record `__nvvm_vote_all`.
  **L823 CN**: 声明 TableGen def 记录 `__nvvm_vote_all`。
- **L824 EN**: Declares TableGen def record `__nvvm_vote_any`.
  **L824 CN**: 声明 TableGen def 记录 `__nvvm_vote_any`。
- **L825 EN**: Declares TableGen def record `__nvvm_vote_uni`.
  **L825 CN**: 声明 TableGen def 记录 `__nvvm_vote_uni`。
- **L826 EN**: Declares TableGen def record `__nvvm_vote_ballot`.
  **L826 CN**: 声明 TableGen def 记录 `__nvvm_vote_ballot`。
- **L827 EN**: Blank line separating nearby declarations or logic blocks.
  **L827 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L828 EN**: Declares TableGen def record `__nvvm_vote_all_sync`.
  **L828 CN**: 声明 TableGen def 记录 `__nvvm_vote_all_sync`。
- **L829 EN**: Declares TableGen def record `__nvvm_vote_any_sync`.
  **L829 CN**: 声明 TableGen def 记录 `__nvvm_vote_any_sync`。
- **L830 EN**: Declares TableGen def record `__nvvm_vote_uni_sync`.
  **L830 CN**: 声明 TableGen def 记录 `__nvvm_vote_uni_sync`。
- **L831 EN**: Declares TableGen def record `__nvvm_vote_ballot_sync`.
  **L831 CN**: 声明 TableGen def 记录 `__nvvm_vote_ballot_sync`。
- **L832 EN**: Blank line separating nearby declarations or logic blocks.
  **L832 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L833 EN**: Comment explains nearby logic, constraints, or intent: `Mask`.
  **L833 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Mask`。
- **L834 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L834 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L835 EN**: Declares TableGen def record `__nvvm_activemask`.
  **L835 CN**: 声明 TableGen def 记录 `__nvvm_activemask`。
- **L836 EN**: Closes the current lexical scope or compound statement.
  **L836 CN**: 结束当前词法作用域或复合语句块。
- **L837 EN**: Blank line separating nearby declarations or logic blocks.
  **L837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L838 EN**: Comment explains nearby logic, constraints, or intent: `Match`.
  **L838 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Match`。
- **L839 EN**: Declares TableGen def record `__nvvm_match_any_sync_i32`.
  **L839 CN**: 声明 TableGen def 记录 `__nvvm_match_any_sync_i32`。
- **L840 EN**: Declares TableGen def record `__nvvm_match_any_sync_i64`.
  **L840 CN**: 声明 TableGen def 记录 `__nvvm_match_any_sync_i64`。

### Lines 841-864

````tablegen
// These return a pair {value, predicate}, which requires custom lowering.
def __nvvm_match_all_sync_i32p : NVPTXBuiltinSMAndPTX<"unsigned int(unsigned int, unsigned int, int *)", SM_70, PTX60>;
def __nvvm_match_all_sync_i64p : NVPTXBuiltinSMAndPTX<"unsigned int(unsigned int, int64_t, int *)", SM_70, PTX60>;

// Redux
def __nvvm_redux_sync_add : NVPTXBuiltinSMAndPTX<"int(int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_min : NVPTXBuiltinSMAndPTX<"int(int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_max : NVPTXBuiltinSMAndPTX<"int(int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_umin : NVPTXBuiltinSMAndPTX<"unsigned int(unsigned int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_umax : NVPTXBuiltinSMAndPTX<"unsigned int(unsigned int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_and : NVPTXBuiltinSMAndPTX<"int(int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_xor : NVPTXBuiltinSMAndPTX<"int(int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_or : NVPTXBuiltinSMAndPTX<"int(int, int)", SM_80, PTX70>;
def __nvvm_redux_sync_fmin : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;
def __nvvm_redux_sync_fmin_abs : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;
def __nvvm_redux_sync_fmin_NaN : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;
def __nvvm_redux_sync_fmin_abs_NaN : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;
def __nvvm_redux_sync_fmax : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;
def __nvvm_redux_sync_fmax_abs : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;
def __nvvm_redux_sync_fmax_NaN : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;
def __nvvm_redux_sync_fmax_abs_NaN : NVPTXBuiltinSMAndPTX<"float(float, int)", SM_100a, PTX86>;

// Membar

````
- **L841 EN**: Comment explains nearby logic, constraints, or intent: `These return a pair {value, predicate}, which requires custom lowering.`.
  **L841 CN**: 注释解释附近代码的逻辑、约束或设计意图：`These return a pair {value, predicate}, which requires custom lowering.`。
- **L842 EN**: Declares TableGen def record `__nvvm_match_all_sync_i32p`.
  **L842 CN**: 声明 TableGen def 记录 `__nvvm_match_all_sync_i32p`。
- **L843 EN**: Declares TableGen def record `__nvvm_match_all_sync_i64p`.
  **L843 CN**: 声明 TableGen def 记录 `__nvvm_match_all_sync_i64p`。
- **L844 EN**: Blank line separating nearby declarations or logic blocks.
  **L844 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L845 EN**: Comment explains nearby logic, constraints, or intent: `Redux`.
  **L845 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Redux`。
- **L846 EN**: Declares TableGen def record `__nvvm_redux_sync_add`.
  **L846 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_add`。
- **L847 EN**: Declares TableGen def record `__nvvm_redux_sync_min`.
  **L847 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_min`。
- **L848 EN**: Declares TableGen def record `__nvvm_redux_sync_max`.
  **L848 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_max`。
- **L849 EN**: Declares TableGen def record `__nvvm_redux_sync_umin`.
  **L849 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_umin`。
- **L850 EN**: Declares TableGen def record `__nvvm_redux_sync_umax`.
  **L850 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_umax`。
- **L851 EN**: Declares TableGen def record `__nvvm_redux_sync_and`.
  **L851 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_and`。
- **L852 EN**: Declares TableGen def record `__nvvm_redux_sync_xor`.
  **L852 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_xor`。
- **L853 EN**: Declares TableGen def record `__nvvm_redux_sync_or`.
  **L853 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_or`。
- **L854 EN**: Declares TableGen def record `__nvvm_redux_sync_fmin`.
  **L854 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmin`。
- **L855 EN**: Declares TableGen def record `__nvvm_redux_sync_fmin_abs`.
  **L855 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmin_abs`。
- **L856 EN**: Declares TableGen def record `__nvvm_redux_sync_fmin_NaN`.
  **L856 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmin_NaN`。
- **L857 EN**: Declares TableGen def record `__nvvm_redux_sync_fmin_abs_NaN`.
  **L857 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmin_abs_NaN`。
- **L858 EN**: Declares TableGen def record `__nvvm_redux_sync_fmax`.
  **L858 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmax`。
- **L859 EN**: Declares TableGen def record `__nvvm_redux_sync_fmax_abs`.
  **L859 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmax_abs`。
- **L860 EN**: Declares TableGen def record `__nvvm_redux_sync_fmax_NaN`.
  **L860 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmax_NaN`。
- **L861 EN**: Declares TableGen def record `__nvvm_redux_sync_fmax_abs_NaN`.
  **L861 CN**: 声明 TableGen def 记录 `__nvvm_redux_sync_fmax_abs_NaN`。
- **L862 EN**: Blank line separating nearby declarations or logic blocks.
  **L862 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L863 EN**: Comment explains nearby logic, constraints, or intent: `Membar`.
  **L863 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Membar`。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 865-888

````tablegen
def __nvvm_membar_cta : NVPTXBuiltin<"void()">;
def __nvvm_membar_gl : NVPTXBuiltin<"void()">;
def __nvvm_membar_sys : NVPTXBuiltin<"void()">;

// mbarrier

def __nvvm_mbarrier_init : NVPTXBuiltinSMAndPTX<"void(int64_t *, int)", SM_80, PTX70>;
def __nvvm_mbarrier_init_shared : NVPTXBuiltinSMAndPTX<"void(int64_t address_space<3> *, int)", SM_80, PTX70>;

def __nvvm_mbarrier_inval : NVPTXBuiltinSMAndPTX<"void(int64_t *)", SM_80, PTX70>;
def __nvvm_mbarrier_inval_shared : NVPTXBuiltinSMAndPTX<"void(int64_t address_space<3> *)", SM_80, PTX70>;

def __nvvm_mbarrier_arrive : NVPTXBuiltinSMAndPTX<"int64_t(int64_t *)", SM_80, PTX70>;
def __nvvm_mbarrier_arrive_shared : NVPTXBuiltinSMAndPTX<"int64_t(int64_t address_space<3> *)", SM_80, PTX70>;
def __nvvm_mbarrier_arrive_noComplete : NVPTXBuiltinSMAndPTX<"int64_t(int64_t *, int)", SM_80, PTX70>;
def __nvvm_mbarrier_arrive_noComplete_shared : NVPTXBuiltinSMAndPTX<"int64_t(int64_t address_space<3> *, int)", SM_80, PTX70>;

def __nvvm_mbarrier_arrive_drop : NVPTXBuiltinSMAndPTX<"int64_t(int64_t *)", SM_80, PTX70>;
def __nvvm_mbarrier_arrive_drop_shared : NVPTXBuiltinSMAndPTX<"int64_t(int64_t address_space<3> *)", SM_80, PTX70>;
def __nvvm_mbarrier_arrive_drop_noComplete : NVPTXBuiltinSMAndPTX<"int64_t(int64_t *, int)", SM_80, PTX70>;
def __nvvm_mbarrier_arrive_drop_noComplete_shared : NVPTXBuiltinSMAndPTX<"int64_t(int64_t address_space<3> *, int)", SM_80, PTX70>;

def __nvvm_mbarrier_test_wait : NVPTXBuiltinSMAndPTX<"bool(int64_t *, int64_t)", SM_80, PTX70>;
def __nvvm_mbarrier_test_wait_shared : NVPTXBuiltinSMAndPTX<"bool(int64_t address_space<3> *, int64_t)", SM_80, PTX70>;
````
- **L865 EN**: Declares TableGen def record `__nvvm_membar_cta`.
  **L865 CN**: 声明 TableGen def 记录 `__nvvm_membar_cta`。
- **L866 EN**: Declares TableGen def record `__nvvm_membar_gl`.
  **L866 CN**: 声明 TableGen def 记录 `__nvvm_membar_gl`。
- **L867 EN**: Declares TableGen def record `__nvvm_membar_sys`.
  **L867 CN**: 声明 TableGen def 记录 `__nvvm_membar_sys`。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L869 EN**: Comment explains nearby logic, constraints, or intent: `mbarrier`.
  **L869 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mbarrier`。
- **L870 EN**: Blank line separating nearby declarations or logic blocks.
  **L870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L871 EN**: Declares TableGen def record `__nvvm_mbarrier_init`.
  **L871 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_init`。
- **L872 EN**: Declares TableGen def record `__nvvm_mbarrier_init_shared`.
  **L872 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_init_shared`。
- **L873 EN**: Blank line separating nearby declarations or logic blocks.
  **L873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L874 EN**: Declares TableGen def record `__nvvm_mbarrier_inval`.
  **L874 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_inval`。
- **L875 EN**: Declares TableGen def record `__nvvm_mbarrier_inval_shared`.
  **L875 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_inval_shared`。
- **L876 EN**: Blank line separating nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L877 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive`.
  **L877 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive`。
- **L878 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive_shared`.
  **L878 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive_shared`。
- **L879 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive_noComplete`.
  **L879 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive_noComplete`。
- **L880 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive_noComplete_shared`.
  **L880 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive_noComplete_shared`。
- **L881 EN**: Blank line separating nearby declarations or logic blocks.
  **L881 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L882 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive_drop`.
  **L882 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive_drop`。
- **L883 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive_drop_shared`.
  **L883 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive_drop_shared`。
- **L884 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive_drop_noComplete`.
  **L884 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive_drop_noComplete`。
- **L885 EN**: Declares TableGen def record `__nvvm_mbarrier_arrive_drop_noComplete_shared`.
  **L885 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_arrive_drop_noComplete_shared`。
- **L886 EN**: Blank line separating nearby declarations or logic blocks.
  **L886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L887 EN**: Declares TableGen def record `__nvvm_mbarrier_test_wait`.
  **L887 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_test_wait`。
- **L888 EN**: Declares TableGen def record `__nvvm_mbarrier_test_wait_shared`.
  **L888 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_test_wait_shared`。

### Lines 889-912

````tablegen

def __nvvm_mbarrier_pending_count : NVPTXBuiltinSMAndPTX<"int(int64_t)", SM_80, PTX70>;

// Memcpy, Memset

def __nvvm_memcpy : NVPTXBuiltin<"void(unsigned char *, unsigned char *, size_t, int)">;
def __nvvm_memset : NVPTXBuiltin<"void(unsigned char *, unsigned char, size_t, int)">;

// Image

def __builtin_ptx_read_image2Dfi_ : NVPTXBuiltin<"_Vector<4, float>(int, int, int, int)">;
def __builtin_ptx_read_image2Dff_ : NVPTXBuiltin<"_Vector<4, float>(int, int, float, float)">;
def __builtin_ptx_read_image2Dii_ : NVPTXBuiltin<"_Vector<4, int>(int, int, int, int)">;
def __builtin_ptx_read_image2Dif_ : NVPTXBuiltin<"_Vector<4, int>(int, int, float, float)">;

def __builtin_ptx_read_image3Dfi_ : NVPTXBuiltin<"_Vector<4, float>(int, int, int, int, int, int)">;
def __builtin_ptx_read_image3Dff_ : NVPTXBuiltin<"_Vector<4, float>(int, int, float, float, float, float)">;
def __builtin_ptx_read_image3Dii_ : NVPTXBuiltin<"_Vector<4, int>(int, int, int, int, int, int)">;
def __builtin_ptx_read_image3Dif_ : NVPTXBuiltin<"_Vector<4, int>(int, int, float, float, float, float)">;

def __builtin_ptx_write_image2Df_ : NVPTXBuiltin<"void(int, int, int, float, float, float, float)">;
def __builtin_ptx_write_image2Di_ : NVPTXBuiltin<"void(int, int, int, int, int, int, int)">;
def __builtin_ptx_write_image2Dui_ : NVPTXBuiltin<"void(int, int, int, unsigned int, unsigned int, unsigned int, unsigned int)">;
def __builtin_ptx_get_image_depthi_ : NVPTXBuiltin<"int(int)">;
````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L890 EN**: Declares TableGen def record `__nvvm_mbarrier_pending_count`.
  **L890 CN**: 声明 TableGen def 记录 `__nvvm_mbarrier_pending_count`。
- **L891 EN**: Blank line separating nearby declarations or logic blocks.
  **L891 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L892 EN**: Comment explains nearby logic, constraints, or intent: `Memcpy, Memset`.
  **L892 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Memcpy, Memset`。
- **L893 EN**: Blank line separating nearby declarations or logic blocks.
  **L893 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L894 EN**: Declares TableGen def record `__nvvm_memcpy`.
  **L894 CN**: 声明 TableGen def 记录 `__nvvm_memcpy`。
- **L895 EN**: Declares TableGen def record `__nvvm_memset`.
  **L895 CN**: 声明 TableGen def 记录 `__nvvm_memset`。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Comment explains nearby logic, constraints, or intent: `Image`.
  **L897 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Image`。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L899 EN**: Declares TableGen def record `__builtin_ptx_read_image2Dfi_`.
  **L899 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image2Dfi_`。
- **L900 EN**: Declares TableGen def record `__builtin_ptx_read_image2Dff_`.
  **L900 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image2Dff_`。
- **L901 EN**: Declares TableGen def record `__builtin_ptx_read_image2Dii_`.
  **L901 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image2Dii_`。
- **L902 EN**: Declares TableGen def record `__builtin_ptx_read_image2Dif_`.
  **L902 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image2Dif_`。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L904 EN**: Declares TableGen def record `__builtin_ptx_read_image3Dfi_`.
  **L904 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image3Dfi_`。
- **L905 EN**: Declares TableGen def record `__builtin_ptx_read_image3Dff_`.
  **L905 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image3Dff_`。
- **L906 EN**: Declares TableGen def record `__builtin_ptx_read_image3Dii_`.
  **L906 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image3Dii_`。
- **L907 EN**: Declares TableGen def record `__builtin_ptx_read_image3Dif_`.
  **L907 CN**: 声明 TableGen def 记录 `__builtin_ptx_read_image3Dif_`。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Declares TableGen def record `__builtin_ptx_write_image2Df_`.
  **L909 CN**: 声明 TableGen def 记录 `__builtin_ptx_write_image2Df_`。
- **L910 EN**: Declares TableGen def record `__builtin_ptx_write_image2Di_`.
  **L910 CN**: 声明 TableGen def 记录 `__builtin_ptx_write_image2Di_`。
- **L911 EN**: Declares TableGen def record `__builtin_ptx_write_image2Dui_`.
  **L911 CN**: 声明 TableGen def 记录 `__builtin_ptx_write_image2Dui_`。
- **L912 EN**: Declares TableGen def record `__builtin_ptx_get_image_depthi_`.
  **L912 CN**: 声明 TableGen def 记录 `__builtin_ptx_get_image_depthi_`。

### Lines 913-936

````tablegen
def __builtin_ptx_get_image_heighti_ : NVPTXBuiltin<"int(int)">;
def __builtin_ptx_get_image_widthi_ : NVPTXBuiltin<"int(int)">;
def __builtin_ptx_get_image_channel_data_typei_ : NVPTXBuiltin<"int(int)">;
def __builtin_ptx_get_image_channel_orderi_ : NVPTXBuiltin<"int(int)">;

// Atomic
//
// We need the atom intrinsics because
// - they are used in converging analysis
// - they are used in address space analysis and optimization
// So it does not hurt to expose them as builtins.
//
let Attributes = [NoThrow] in {
  def __nvvm_atom_add_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_cta_add_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_sys_add_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_add_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_cta_add_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_sys_add_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_add_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;
  def __nvvm_atom_cta_add_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_sys_add_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_add_gen_f : NVPTXBuiltin<"float(float volatile *, float)">;
  def __nvvm_atom_cta_add_gen_f : NVPTXBuiltinSM<"float(float volatile *, float)", SM_60>;
````
- **L913 EN**: Declares TableGen def record `__builtin_ptx_get_image_heighti_`.
  **L913 CN**: 声明 TableGen def 记录 `__builtin_ptx_get_image_heighti_`。
- **L914 EN**: Declares TableGen def record `__builtin_ptx_get_image_widthi_`.
  **L914 CN**: 声明 TableGen def 记录 `__builtin_ptx_get_image_widthi_`。
- **L915 EN**: Declares TableGen def record `__builtin_ptx_get_image_channel_data_typei_`.
  **L915 CN**: 声明 TableGen def 记录 `__builtin_ptx_get_image_channel_data_typei_`。
- **L916 EN**: Declares TableGen def record `__builtin_ptx_get_image_channel_orderi_`.
  **L916 CN**: 声明 TableGen def 记录 `__builtin_ptx_get_image_channel_orderi_`。
- **L917 EN**: Blank line separating nearby declarations or logic blocks.
  **L917 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L918 EN**: Comment explains nearby logic, constraints, or intent: `Atomic`.
  **L918 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Atomic`。
- **L919 EN**: Separator comment used for visual grouping.
  **L919 CN**: 用于视觉分组的分隔注释。
- **L920 EN**: Comment explains nearby logic, constraints, or intent: `We need the atom intrinsics because`.
  **L920 CN**: 注释解释附近代码的逻辑、约束或设计意图：`We need the atom intrinsics because`。
- **L921 EN**: Comment explains nearby logic, constraints, or intent: `they are used in converging analysis`.
  **L921 CN**: 注释解释附近代码的逻辑、约束或设计意图：`they are used in converging analysis`。
- **L922 EN**: Comment explains nearby logic, constraints, or intent: `they are used in address space analysis and optimization`.
  **L922 CN**: 注释解释附近代码的逻辑、约束或设计意图：`they are used in address space analysis and optimization`。
- **L923 EN**: Comment explains nearby logic, constraints, or intent: `So it does not hurt to expose them as builtins.`.
  **L923 CN**: 注释解释附近代码的逻辑、约束或设计意图：`So it does not hurt to expose them as builtins.`。
- **L924 EN**: Separator comment used for visual grouping.
  **L924 CN**: 用于视觉分组的分隔注释。
- **L925 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L925 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L926 EN**: Declares TableGen def record `__nvvm_atom_add_gen_i`.
  **L926 CN**: 声明 TableGen def 记录 `__nvvm_atom_add_gen_i`。
- **L927 EN**: Declares TableGen def record `__nvvm_atom_cta_add_gen_i`.
  **L927 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_add_gen_i`。
- **L928 EN**: Declares TableGen def record `__nvvm_atom_sys_add_gen_i`.
  **L928 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_add_gen_i`。
- **L929 EN**: Declares TableGen def record `__nvvm_atom_add_gen_l`.
  **L929 CN**: 声明 TableGen def 记录 `__nvvm_atom_add_gen_l`。
- **L930 EN**: Declares TableGen def record `__nvvm_atom_cta_add_gen_l`.
  **L930 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_add_gen_l`。
- **L931 EN**: Declares TableGen def record `__nvvm_atom_sys_add_gen_l`.
  **L931 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_add_gen_l`。
- **L932 EN**: Declares TableGen def record `__nvvm_atom_add_gen_ll`.
  **L932 CN**: 声明 TableGen def 记录 `__nvvm_atom_add_gen_ll`。
- **L933 EN**: Declares TableGen def record `__nvvm_atom_cta_add_gen_ll`.
  **L933 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_add_gen_ll`。
- **L934 EN**: Declares TableGen def record `__nvvm_atom_sys_add_gen_ll`.
  **L934 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_add_gen_ll`。
- **L935 EN**: Declares TableGen def record `__nvvm_atom_add_gen_f`.
  **L935 CN**: 声明 TableGen def 记录 `__nvvm_atom_add_gen_f`。
- **L936 EN**: Declares TableGen def record `__nvvm_atom_cta_add_gen_f`.
  **L936 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_add_gen_f`。

### Lines 937-960

````tablegen
  def __nvvm_atom_sys_add_gen_f : NVPTXBuiltinSM<"float(float volatile *, float)", SM_60>;
  def __nvvm_atom_add_gen_d : NVPTXBuiltinSM<"double(double volatile *, double)", SM_60>;
  def __nvvm_atom_cta_add_gen_d : NVPTXBuiltinSM<"double(double volatile *, double)", SM_60>;
  def __nvvm_atom_sys_add_gen_d : NVPTXBuiltinSM<"double(double volatile *, double)", SM_60>;

  def __nvvm_atom_sub_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_sub_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_sub_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;

  def __nvvm_atom_xchg_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_cta_xchg_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_sys_xchg_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_xchg_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_cta_xchg_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_sys_xchg_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_xchg_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;
  def __nvvm_atom_cta_xchg_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_sys_xchg_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;

  def __nvvm_atom_max_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_cta_max_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_sys_max_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_max_gen_ui : NVPTXBuiltin<"unsigned int(unsigned int volatile *, unsigned int)">;
  def __nvvm_atom_cta_max_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;
````
- **L937 EN**: Declares TableGen def record `__nvvm_atom_sys_add_gen_f`.
  **L937 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_add_gen_f`。
- **L938 EN**: Declares TableGen def record `__nvvm_atom_add_gen_d`.
  **L938 CN**: 声明 TableGen def 记录 `__nvvm_atom_add_gen_d`。
- **L939 EN**: Declares TableGen def record `__nvvm_atom_cta_add_gen_d`.
  **L939 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_add_gen_d`。
- **L940 EN**: Declares TableGen def record `__nvvm_atom_sys_add_gen_d`.
  **L940 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_add_gen_d`。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L942 EN**: Declares TableGen def record `__nvvm_atom_sub_gen_i`.
  **L942 CN**: 声明 TableGen def 记录 `__nvvm_atom_sub_gen_i`。
- **L943 EN**: Declares TableGen def record `__nvvm_atom_sub_gen_l`.
  **L943 CN**: 声明 TableGen def 记录 `__nvvm_atom_sub_gen_l`。
- **L944 EN**: Declares TableGen def record `__nvvm_atom_sub_gen_ll`.
  **L944 CN**: 声明 TableGen def 记录 `__nvvm_atom_sub_gen_ll`。
- **L945 EN**: Blank line separating nearby declarations or logic blocks.
  **L945 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L946 EN**: Declares TableGen def record `__nvvm_atom_xchg_gen_i`.
  **L946 CN**: 声明 TableGen def 记录 `__nvvm_atom_xchg_gen_i`。
- **L947 EN**: Declares TableGen def record `__nvvm_atom_cta_xchg_gen_i`.
  **L947 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_xchg_gen_i`。
- **L948 EN**: Declares TableGen def record `__nvvm_atom_sys_xchg_gen_i`.
  **L948 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_xchg_gen_i`。
- **L949 EN**: Declares TableGen def record `__nvvm_atom_xchg_gen_l`.
  **L949 CN**: 声明 TableGen def 记录 `__nvvm_atom_xchg_gen_l`。
- **L950 EN**: Declares TableGen def record `__nvvm_atom_cta_xchg_gen_l`.
  **L950 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_xchg_gen_l`。
- **L951 EN**: Declares TableGen def record `__nvvm_atom_sys_xchg_gen_l`.
  **L951 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_xchg_gen_l`。
- **L952 EN**: Declares TableGen def record `__nvvm_atom_xchg_gen_ll`.
  **L952 CN**: 声明 TableGen def 记录 `__nvvm_atom_xchg_gen_ll`。
- **L953 EN**: Declares TableGen def record `__nvvm_atom_cta_xchg_gen_ll`.
  **L953 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_xchg_gen_ll`。
- **L954 EN**: Declares TableGen def record `__nvvm_atom_sys_xchg_gen_ll`.
  **L954 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_xchg_gen_ll`。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L956 EN**: Declares TableGen def record `__nvvm_atom_max_gen_i`.
  **L956 CN**: 声明 TableGen def 记录 `__nvvm_atom_max_gen_i`。
- **L957 EN**: Declares TableGen def record `__nvvm_atom_cta_max_gen_i`.
  **L957 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_max_gen_i`。
- **L958 EN**: Declares TableGen def record `__nvvm_atom_sys_max_gen_i`.
  **L958 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_max_gen_i`。
- **L959 EN**: Declares TableGen def record `__nvvm_atom_max_gen_ui`.
  **L959 CN**: 声明 TableGen def 记录 `__nvvm_atom_max_gen_ui`。
- **L960 EN**: Declares TableGen def record `__nvvm_atom_cta_max_gen_ui`.
  **L960 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_max_gen_ui`。

### Lines 961-984

````tablegen
  def __nvvm_atom_sys_max_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;
  def __nvvm_atom_max_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_cta_max_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_sys_max_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_max_gen_ul : NVPTXBuiltin<"unsigned long int(unsigned long int volatile *, unsigned long int)">;
  def __nvvm_atom_cta_max_gen_ul : NVPTXBuiltinSM<"unsigned long int(unsigned long int volatile *, unsigned long int)", SM_60>;
  def __nvvm_atom_sys_max_gen_ul : NVPTXBuiltinSM<"unsigned long int(unsigned long int volatile *, unsigned long int)", SM_60>;
  def __nvvm_atom_max_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;
  def __nvvm_atom_cta_max_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_sys_max_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_max_gen_ull : NVPTXBuiltin<"unsigned long long int(unsigned long long int volatile *, unsigned long long int)">;
  def __nvvm_atom_cta_max_gen_ull : NVPTXBuiltinSM<"unsigned long long int(unsigned long long int volatile *, unsigned long long int)", SM_60>;
  def __nvvm_atom_sys_max_gen_ull : NVPTXBuiltinSM<"unsigned long long int(unsigned long long int volatile *, unsigned long long int)", SM_60>;

  def __nvvm_atom_min_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_cta_min_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_sys_min_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_min_gen_ui : NVPTXBuiltin<"unsigned int(unsigned int volatile *, unsigned int)">;
  def __nvvm_atom_cta_min_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;
  def __nvvm_atom_sys_min_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;
  def __nvvm_atom_min_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_cta_min_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_sys_min_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_min_gen_ul : NVPTXBuiltin<"unsigned long int(unsigned long int volatile *, unsigned long int)">;
````
- **L961 EN**: Declares TableGen def record `__nvvm_atom_sys_max_gen_ui`.
  **L961 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_max_gen_ui`。
- **L962 EN**: Declares TableGen def record `__nvvm_atom_max_gen_l`.
  **L962 CN**: 声明 TableGen def 记录 `__nvvm_atom_max_gen_l`。
- **L963 EN**: Declares TableGen def record `__nvvm_atom_cta_max_gen_l`.
  **L963 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_max_gen_l`。
- **L964 EN**: Declares TableGen def record `__nvvm_atom_sys_max_gen_l`.
  **L964 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_max_gen_l`。
- **L965 EN**: Declares TableGen def record `__nvvm_atom_max_gen_ul`.
  **L965 CN**: 声明 TableGen def 记录 `__nvvm_atom_max_gen_ul`。
- **L966 EN**: Declares TableGen def record `__nvvm_atom_cta_max_gen_ul`.
  **L966 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_max_gen_ul`。
- **L967 EN**: Declares TableGen def record `__nvvm_atom_sys_max_gen_ul`.
  **L967 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_max_gen_ul`。
- **L968 EN**: Declares TableGen def record `__nvvm_atom_max_gen_ll`.
  **L968 CN**: 声明 TableGen def 记录 `__nvvm_atom_max_gen_ll`。
- **L969 EN**: Declares TableGen def record `__nvvm_atom_cta_max_gen_ll`.
  **L969 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_max_gen_ll`。
- **L970 EN**: Declares TableGen def record `__nvvm_atom_sys_max_gen_ll`.
  **L970 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_max_gen_ll`。
- **L971 EN**: Declares TableGen def record `__nvvm_atom_max_gen_ull`.
  **L971 CN**: 声明 TableGen def 记录 `__nvvm_atom_max_gen_ull`。
- **L972 EN**: Declares TableGen def record `__nvvm_atom_cta_max_gen_ull`.
  **L972 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_max_gen_ull`。
- **L973 EN**: Declares TableGen def record `__nvvm_atom_sys_max_gen_ull`.
  **L973 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_max_gen_ull`。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Declares TableGen def record `__nvvm_atom_min_gen_i`.
  **L975 CN**: 声明 TableGen def 记录 `__nvvm_atom_min_gen_i`。
- **L976 EN**: Declares TableGen def record `__nvvm_atom_cta_min_gen_i`.
  **L976 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_min_gen_i`。
- **L977 EN**: Declares TableGen def record `__nvvm_atom_sys_min_gen_i`.
  **L977 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_min_gen_i`。
- **L978 EN**: Declares TableGen def record `__nvvm_atom_min_gen_ui`.
  **L978 CN**: 声明 TableGen def 记录 `__nvvm_atom_min_gen_ui`。
- **L979 EN**: Declares TableGen def record `__nvvm_atom_cta_min_gen_ui`.
  **L979 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_min_gen_ui`。
- **L980 EN**: Declares TableGen def record `__nvvm_atom_sys_min_gen_ui`.
  **L980 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_min_gen_ui`。
- **L981 EN**: Declares TableGen def record `__nvvm_atom_min_gen_l`.
  **L981 CN**: 声明 TableGen def 记录 `__nvvm_atom_min_gen_l`。
- **L982 EN**: Declares TableGen def record `__nvvm_atom_cta_min_gen_l`.
  **L982 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_min_gen_l`。
- **L983 EN**: Declares TableGen def record `__nvvm_atom_sys_min_gen_l`.
  **L983 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_min_gen_l`。
- **L984 EN**: Declares TableGen def record `__nvvm_atom_min_gen_ul`.
  **L984 CN**: 声明 TableGen def 记录 `__nvvm_atom_min_gen_ul`。

### Lines 985-1008

````tablegen
  def __nvvm_atom_cta_min_gen_ul : NVPTXBuiltinSM<"unsigned long int(unsigned long int volatile *, unsigned long int)", SM_60>;
  def __nvvm_atom_sys_min_gen_ul : NVPTXBuiltinSM<"unsigned long int(unsigned long int volatile *, unsigned long int)", SM_60>;
  def __nvvm_atom_min_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;
  def __nvvm_atom_cta_min_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_sys_min_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_min_gen_ull : NVPTXBuiltin<"unsigned long long int(unsigned long long int volatile *, unsigned long long int)">;
  def __nvvm_atom_cta_min_gen_ull : NVPTXBuiltinSM<"unsigned long long int(unsigned long long int volatile *, unsigned long long int)", SM_60>;
  def __nvvm_atom_sys_min_gen_ull : NVPTXBuiltinSM<"unsigned long long int(unsigned long long int volatile *, unsigned long long int)", SM_60>;

  def __nvvm_atom_inc_gen_ui : NVPTXBuiltin<"unsigned int(unsigned int volatile *, unsigned int)">;
  def __nvvm_atom_cta_inc_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;
  def __nvvm_atom_sys_inc_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;
  def __nvvm_atom_dec_gen_ui : NVPTXBuiltin<"unsigned int(unsigned int volatile *, unsigned int)">;
  def __nvvm_atom_cta_dec_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;
  def __nvvm_atom_sys_dec_gen_ui : NVPTXBuiltinSM<"unsigned int(unsigned int volatile *, unsigned int)", SM_60>;

  def __nvvm_atom_and_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_cta_and_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_sys_and_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_and_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_cta_and_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_sys_and_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_and_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;
  def __nvvm_atom_cta_and_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
````
- **L985 EN**: Declares TableGen def record `__nvvm_atom_cta_min_gen_ul`.
  **L985 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_min_gen_ul`。
- **L986 EN**: Declares TableGen def record `__nvvm_atom_sys_min_gen_ul`.
  **L986 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_min_gen_ul`。
- **L987 EN**: Declares TableGen def record `__nvvm_atom_min_gen_ll`.
  **L987 CN**: 声明 TableGen def 记录 `__nvvm_atom_min_gen_ll`。
- **L988 EN**: Declares TableGen def record `__nvvm_atom_cta_min_gen_ll`.
  **L988 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_min_gen_ll`。
- **L989 EN**: Declares TableGen def record `__nvvm_atom_sys_min_gen_ll`.
  **L989 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_min_gen_ll`。
- **L990 EN**: Declares TableGen def record `__nvvm_atom_min_gen_ull`.
  **L990 CN**: 声明 TableGen def 记录 `__nvvm_atom_min_gen_ull`。
- **L991 EN**: Declares TableGen def record `__nvvm_atom_cta_min_gen_ull`.
  **L991 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_min_gen_ull`。
- **L992 EN**: Declares TableGen def record `__nvvm_atom_sys_min_gen_ull`.
  **L992 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_min_gen_ull`。
- **L993 EN**: Blank line separating nearby declarations or logic blocks.
  **L993 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L994 EN**: Declares TableGen def record `__nvvm_atom_inc_gen_ui`.
  **L994 CN**: 声明 TableGen def 记录 `__nvvm_atom_inc_gen_ui`。
- **L995 EN**: Declares TableGen def record `__nvvm_atom_cta_inc_gen_ui`.
  **L995 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_inc_gen_ui`。
- **L996 EN**: Declares TableGen def record `__nvvm_atom_sys_inc_gen_ui`.
  **L996 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_inc_gen_ui`。
- **L997 EN**: Declares TableGen def record `__nvvm_atom_dec_gen_ui`.
  **L997 CN**: 声明 TableGen def 记录 `__nvvm_atom_dec_gen_ui`。
- **L998 EN**: Declares TableGen def record `__nvvm_atom_cta_dec_gen_ui`.
  **L998 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_dec_gen_ui`。
- **L999 EN**: Declares TableGen def record `__nvvm_atom_sys_dec_gen_ui`.
  **L999 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_dec_gen_ui`。
- **L1000 EN**: Blank line separating nearby declarations or logic blocks.
  **L1000 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1001 EN**: Declares TableGen def record `__nvvm_atom_and_gen_i`.
  **L1001 CN**: 声明 TableGen def 记录 `__nvvm_atom_and_gen_i`。
- **L1002 EN**: Declares TableGen def record `__nvvm_atom_cta_and_gen_i`.
  **L1002 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_and_gen_i`。
- **L1003 EN**: Declares TableGen def record `__nvvm_atom_sys_and_gen_i`.
  **L1003 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_and_gen_i`。
- **L1004 EN**: Declares TableGen def record `__nvvm_atom_and_gen_l`.
  **L1004 CN**: 声明 TableGen def 记录 `__nvvm_atom_and_gen_l`。
- **L1005 EN**: Declares TableGen def record `__nvvm_atom_cta_and_gen_l`.
  **L1005 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_and_gen_l`。
- **L1006 EN**: Declares TableGen def record `__nvvm_atom_sys_and_gen_l`.
  **L1006 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_and_gen_l`。
- **L1007 EN**: Declares TableGen def record `__nvvm_atom_and_gen_ll`.
  **L1007 CN**: 声明 TableGen def 记录 `__nvvm_atom_and_gen_ll`。
- **L1008 EN**: Declares TableGen def record `__nvvm_atom_cta_and_gen_ll`.
  **L1008 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_and_gen_ll`。

### Lines 1009-1032

````tablegen
  def __nvvm_atom_sys_and_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;

  def __nvvm_atom_or_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_cta_or_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_sys_or_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_or_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_cta_or_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_sys_or_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_or_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;
  def __nvvm_atom_cta_or_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_sys_or_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;

  def __nvvm_atom_xor_gen_i : NVPTXBuiltin<"int(int volatile *, int)">;
  def __nvvm_atom_cta_xor_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_sys_xor_gen_i : NVPTXBuiltinSM<"int(int volatile *, int)", SM_60>;
  def __nvvm_atom_xor_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int)">;
  def __nvvm_atom_cta_xor_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_sys_xor_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int)", SM_60>;
  def __nvvm_atom_xor_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int)">;
  def __nvvm_atom_cta_xor_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;
  def __nvvm_atom_sys_xor_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int)", SM_60>;

  def __nvvm_atom_cas_gen_us : NVPTXBuiltinSM<"unsigned short(unsigned short volatile *, unsigned short, unsigned short)", SM_70>;
  def __nvvm_atom_cta_cas_gen_us : NVPTXBuiltinSM<"unsigned short(unsigned short volatile *, unsigned short, unsigned short)", SM_70>;
````
- **L1009 EN**: Declares TableGen def record `__nvvm_atom_sys_and_gen_ll`.
  **L1009 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_and_gen_ll`。
- **L1010 EN**: Blank line separating nearby declarations or logic blocks.
  **L1010 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1011 EN**: Declares TableGen def record `__nvvm_atom_or_gen_i`.
  **L1011 CN**: 声明 TableGen def 记录 `__nvvm_atom_or_gen_i`。
- **L1012 EN**: Declares TableGen def record `__nvvm_atom_cta_or_gen_i`.
  **L1012 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_or_gen_i`。
- **L1013 EN**: Declares TableGen def record `__nvvm_atom_sys_or_gen_i`.
  **L1013 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_or_gen_i`。
- **L1014 EN**: Declares TableGen def record `__nvvm_atom_or_gen_l`.
  **L1014 CN**: 声明 TableGen def 记录 `__nvvm_atom_or_gen_l`。
- **L1015 EN**: Declares TableGen def record `__nvvm_atom_cta_or_gen_l`.
  **L1015 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_or_gen_l`。
- **L1016 EN**: Declares TableGen def record `__nvvm_atom_sys_or_gen_l`.
  **L1016 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_or_gen_l`。
- **L1017 EN**: Declares TableGen def record `__nvvm_atom_or_gen_ll`.
  **L1017 CN**: 声明 TableGen def 记录 `__nvvm_atom_or_gen_ll`。
- **L1018 EN**: Declares TableGen def record `__nvvm_atom_cta_or_gen_ll`.
  **L1018 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_or_gen_ll`。
- **L1019 EN**: Declares TableGen def record `__nvvm_atom_sys_or_gen_ll`.
  **L1019 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_or_gen_ll`。
- **L1020 EN**: Blank line separating nearby declarations or logic blocks.
  **L1020 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1021 EN**: Declares TableGen def record `__nvvm_atom_xor_gen_i`.
  **L1021 CN**: 声明 TableGen def 记录 `__nvvm_atom_xor_gen_i`。
- **L1022 EN**: Declares TableGen def record `__nvvm_atom_cta_xor_gen_i`.
  **L1022 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_xor_gen_i`。
- **L1023 EN**: Declares TableGen def record `__nvvm_atom_sys_xor_gen_i`.
  **L1023 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_xor_gen_i`。
- **L1024 EN**: Declares TableGen def record `__nvvm_atom_xor_gen_l`.
  **L1024 CN**: 声明 TableGen def 记录 `__nvvm_atom_xor_gen_l`。
- **L1025 EN**: Declares TableGen def record `__nvvm_atom_cta_xor_gen_l`.
  **L1025 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_xor_gen_l`。
- **L1026 EN**: Declares TableGen def record `__nvvm_atom_sys_xor_gen_l`.
  **L1026 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_xor_gen_l`。
- **L1027 EN**: Declares TableGen def record `__nvvm_atom_xor_gen_ll`.
  **L1027 CN**: 声明 TableGen def 记录 `__nvvm_atom_xor_gen_ll`。
- **L1028 EN**: Declares TableGen def record `__nvvm_atom_cta_xor_gen_ll`.
  **L1028 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_xor_gen_ll`。
- **L1029 EN**: Declares TableGen def record `__nvvm_atom_sys_xor_gen_ll`.
  **L1029 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_xor_gen_ll`。
- **L1030 EN**: Blank line separating nearby declarations or logic blocks.
  **L1030 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1031 EN**: Declares TableGen def record `__nvvm_atom_cas_gen_us`.
  **L1031 CN**: 声明 TableGen def 记录 `__nvvm_atom_cas_gen_us`。
- **L1032 EN**: Declares TableGen def record `__nvvm_atom_cta_cas_gen_us`.
  **L1032 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_cas_gen_us`。

### Lines 1033-1056

````tablegen
  def __nvvm_atom_sys_cas_gen_us : NVPTXBuiltinSM<"unsigned short(unsigned short volatile *, unsigned short, unsigned short)", SM_70>;
  def __nvvm_atom_cas_gen_i : NVPTXBuiltin<"int(int volatile *, int, int)">;
  def __nvvm_atom_cta_cas_gen_i : NVPTXBuiltinSM<"int(int volatile *, int, int)", SM_60>;
  def __nvvm_atom_sys_cas_gen_i : NVPTXBuiltinSM<"int(int volatile *, int, int)", SM_60>;
  def __nvvm_atom_cas_gen_l : NVPTXBuiltin<"long int(long int volatile *, long int, long int)">;
  def __nvvm_atom_cta_cas_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int, long int)", SM_60>;
  def __nvvm_atom_sys_cas_gen_l : NVPTXBuiltinSM<"long int(long int volatile *, long int, long int)", SM_60>;
  def __nvvm_atom_cas_gen_ll : NVPTXBuiltin<"long long int(long long int volatile *, long long int, long long int)">;
  def __nvvm_atom_cta_cas_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int, long long int)", SM_60>;
  def __nvvm_atom_sys_cas_gen_ll : NVPTXBuiltinSM<"long long int(long long int volatile *, long long int, long long int)", SM_60>;
}

// Compiler Error Warn
let Attributes = [NoThrow] in {
  def __nvvm_compiler_error : NVPTXBuiltin<"void(char const address_space<4> *)">;
  def __nvvm_compiler_warn : NVPTXBuiltin<"void(char const address_space<4> *)">;
}

def __nvvm_ldu_c : NVPTXBuiltin<"char(char const *)">;
def __nvvm_ldu_sc : NVPTXBuiltin<"signed char(signed char const *)">;
def __nvvm_ldu_s : NVPTXBuiltin<"short(short const *)">;
def __nvvm_ldu_i : NVPTXBuiltin<"int(int const *)">;
def __nvvm_ldu_l : NVPTXBuiltin<"long int(long int const *)">;
def __nvvm_ldu_ll : NVPTXBuiltin<"long long int(long long int const *)">;
````
- **L1033 EN**: Declares TableGen def record `__nvvm_atom_sys_cas_gen_us`.
  **L1033 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_cas_gen_us`。
- **L1034 EN**: Declares TableGen def record `__nvvm_atom_cas_gen_i`.
  **L1034 CN**: 声明 TableGen def 记录 `__nvvm_atom_cas_gen_i`。
- **L1035 EN**: Declares TableGen def record `__nvvm_atom_cta_cas_gen_i`.
  **L1035 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_cas_gen_i`。
- **L1036 EN**: Declares TableGen def record `__nvvm_atom_sys_cas_gen_i`.
  **L1036 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_cas_gen_i`。
- **L1037 EN**: Declares TableGen def record `__nvvm_atom_cas_gen_l`.
  **L1037 CN**: 声明 TableGen def 记录 `__nvvm_atom_cas_gen_l`。
- **L1038 EN**: Declares TableGen def record `__nvvm_atom_cta_cas_gen_l`.
  **L1038 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_cas_gen_l`。
- **L1039 EN**: Declares TableGen def record `__nvvm_atom_sys_cas_gen_l`.
  **L1039 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_cas_gen_l`。
- **L1040 EN**: Declares TableGen def record `__nvvm_atom_cas_gen_ll`.
  **L1040 CN**: 声明 TableGen def 记录 `__nvvm_atom_cas_gen_ll`。
- **L1041 EN**: Declares TableGen def record `__nvvm_atom_cta_cas_gen_ll`.
  **L1041 CN**: 声明 TableGen def 记录 `__nvvm_atom_cta_cas_gen_ll`。
- **L1042 EN**: Declares TableGen def record `__nvvm_atom_sys_cas_gen_ll`.
  **L1042 CN**: 声明 TableGen def 记录 `__nvvm_atom_sys_cas_gen_ll`。
- **L1043 EN**: Closes the current lexical scope or compound statement.
  **L1043 CN**: 结束当前词法作用域或复合语句块。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1045 EN**: Comment explains nearby logic, constraints, or intent: `Compiler Error Warn`.
  **L1045 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Compiler Error Warn`。
- **L1046 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow] in {`.
  **L1046 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow] in {`。
- **L1047 EN**: Declares TableGen def record `__nvvm_compiler_error`.
  **L1047 CN**: 声明 TableGen def 记录 `__nvvm_compiler_error`。
- **L1048 EN**: Declares TableGen def record `__nvvm_compiler_warn`.
  **L1048 CN**: 声明 TableGen def 记录 `__nvvm_compiler_warn`。
- **L1049 EN**: Closes the current lexical scope or compound statement.
  **L1049 CN**: 结束当前词法作用域或复合语句块。
- **L1050 EN**: Blank line separating nearby declarations or logic blocks.
  **L1050 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1051 EN**: Declares TableGen def record `__nvvm_ldu_c`.
  **L1051 CN**: 声明 TableGen def 记录 `__nvvm_ldu_c`。
- **L1052 EN**: Declares TableGen def record `__nvvm_ldu_sc`.
  **L1052 CN**: 声明 TableGen def 记录 `__nvvm_ldu_sc`。
- **L1053 EN**: Declares TableGen def record `__nvvm_ldu_s`.
  **L1053 CN**: 声明 TableGen def 记录 `__nvvm_ldu_s`。
- **L1054 EN**: Declares TableGen def record `__nvvm_ldu_i`.
  **L1054 CN**: 声明 TableGen def 记录 `__nvvm_ldu_i`。
- **L1055 EN**: Declares TableGen def record `__nvvm_ldu_l`.
  **L1055 CN**: 声明 TableGen def 记录 `__nvvm_ldu_l`。
- **L1056 EN**: Declares TableGen def record `__nvvm_ldu_ll`.
  **L1056 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ll`。

### Lines 1057-1080

````tablegen

def __nvvm_ldu_uc : NVPTXBuiltin<"unsigned char(unsigned char const *)">;
def __nvvm_ldu_us : NVPTXBuiltin<"unsigned short(unsigned short const *)">;
def __nvvm_ldu_ui : NVPTXBuiltin<"unsigned int(unsigned int const *)">;
def __nvvm_ldu_ul : NVPTXBuiltin<"unsigned long int(unsigned long int const *)">;
def __nvvm_ldu_ull : NVPTXBuiltin<"unsigned long long int(unsigned long long int const *)">;

def __nvvm_ldu_h : NVPTXBuiltin<"__fp16(__fp16 const *)">;
def __nvvm_ldu_f : NVPTXBuiltin<"float(float const *)">;
def __nvvm_ldu_d : NVPTXBuiltin<"double(double const *)">;

def __nvvm_ldu_c2 : NVPTXBuiltin<"_ExtVector<2, char>(_ExtVector<2, char const *>)">;
def __nvvm_ldu_sc2 : NVPTXBuiltin<"_ExtVector<2, signed char>(_ExtVector<2, signed char const *>)">;
def __nvvm_ldu_c4 : NVPTXBuiltin<"_ExtVector<4, char>(_ExtVector<4, char const *>)">;
def __nvvm_ldu_sc4 : NVPTXBuiltin<"_ExtVector<4, signed char>(_ExtVector<4, signed char const *>)">;
def __nvvm_ldu_s2 : NVPTXBuiltin<"_ExtVector<2, short>(_ExtVector<2, short const *>)">;
def __nvvm_ldu_s4 : NVPTXBuiltin<"_ExtVector<4, short>(_ExtVector<4, short const *>)">;
def __nvvm_ldu_i2 : NVPTXBuiltin<"_ExtVector<2, int>(_ExtVector<2, int const *>)">;
def __nvvm_ldu_i4 : NVPTXBuiltin<"_ExtVector<4, int>(_ExtVector<4, int const *>)">;
def __nvvm_ldu_l2 : NVPTXBuiltin<"_ExtVector<2, long int>(_ExtVector<2, long int const *>)">;
def __nvvm_ldu_ll2 : NVPTXBuiltin<"_ExtVector<2, long long int>(_ExtVector<2, long long int const *>)">;

def __nvvm_ldu_uc2 : NVPTXBuiltin<"_ExtVector<2, unsigned char>(_ExtVector<2, unsigned char const *>)">;
def __nvvm_ldu_uc4 : NVPTXBuiltin<"_ExtVector<4, unsigned char>(_ExtVector<4, unsigned char const *>)">;
````
- **L1057 EN**: Blank line separating nearby declarations or logic blocks.
  **L1057 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1058 EN**: Declares TableGen def record `__nvvm_ldu_uc`.
  **L1058 CN**: 声明 TableGen def 记录 `__nvvm_ldu_uc`。
- **L1059 EN**: Declares TableGen def record `__nvvm_ldu_us`.
  **L1059 CN**: 声明 TableGen def 记录 `__nvvm_ldu_us`。
- **L1060 EN**: Declares TableGen def record `__nvvm_ldu_ui`.
  **L1060 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ui`。
- **L1061 EN**: Declares TableGen def record `__nvvm_ldu_ul`.
  **L1061 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ul`。
- **L1062 EN**: Declares TableGen def record `__nvvm_ldu_ull`.
  **L1062 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ull`。
- **L1063 EN**: Blank line separating nearby declarations or logic blocks.
  **L1063 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1064 EN**: Declares TableGen def record `__nvvm_ldu_h`.
  **L1064 CN**: 声明 TableGen def 记录 `__nvvm_ldu_h`。
- **L1065 EN**: Declares TableGen def record `__nvvm_ldu_f`.
  **L1065 CN**: 声明 TableGen def 记录 `__nvvm_ldu_f`。
- **L1066 EN**: Declares TableGen def record `__nvvm_ldu_d`.
  **L1066 CN**: 声明 TableGen def 记录 `__nvvm_ldu_d`。
- **L1067 EN**: Blank line separating nearby declarations or logic blocks.
  **L1067 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1068 EN**: Declares TableGen def record `__nvvm_ldu_c2`.
  **L1068 CN**: 声明 TableGen def 记录 `__nvvm_ldu_c2`。
- **L1069 EN**: Declares TableGen def record `__nvvm_ldu_sc2`.
  **L1069 CN**: 声明 TableGen def 记录 `__nvvm_ldu_sc2`。
- **L1070 EN**: Declares TableGen def record `__nvvm_ldu_c4`.
  **L1070 CN**: 声明 TableGen def 记录 `__nvvm_ldu_c4`。
- **L1071 EN**: Declares TableGen def record `__nvvm_ldu_sc4`.
  **L1071 CN**: 声明 TableGen def 记录 `__nvvm_ldu_sc4`。
- **L1072 EN**: Declares TableGen def record `__nvvm_ldu_s2`.
  **L1072 CN**: 声明 TableGen def 记录 `__nvvm_ldu_s2`。
- **L1073 EN**: Declares TableGen def record `__nvvm_ldu_s4`.
  **L1073 CN**: 声明 TableGen def 记录 `__nvvm_ldu_s4`。
- **L1074 EN**: Declares TableGen def record `__nvvm_ldu_i2`.
  **L1074 CN**: 声明 TableGen def 记录 `__nvvm_ldu_i2`。
- **L1075 EN**: Declares TableGen def record `__nvvm_ldu_i4`.
  **L1075 CN**: 声明 TableGen def 记录 `__nvvm_ldu_i4`。
- **L1076 EN**: Declares TableGen def record `__nvvm_ldu_l2`.
  **L1076 CN**: 声明 TableGen def 记录 `__nvvm_ldu_l2`。
- **L1077 EN**: Declares TableGen def record `__nvvm_ldu_ll2`.
  **L1077 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ll2`。
- **L1078 EN**: Blank line separating nearby declarations or logic blocks.
  **L1078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1079 EN**: Declares TableGen def record `__nvvm_ldu_uc2`.
  **L1079 CN**: 声明 TableGen def 记录 `__nvvm_ldu_uc2`。
- **L1080 EN**: Declares TableGen def record `__nvvm_ldu_uc4`.
  **L1080 CN**: 声明 TableGen def 记录 `__nvvm_ldu_uc4`。

### Lines 1081-1104

````tablegen
def __nvvm_ldu_us2 : NVPTXBuiltin<"_ExtVector<2, unsigned short>(_ExtVector<2, unsigned short const *>)">;
def __nvvm_ldu_us4 : NVPTXBuiltin<"_ExtVector<4, unsigned short>(_ExtVector<4, unsigned short const *>)">;
def __nvvm_ldu_ui2 : NVPTXBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<2, unsigned int const *>)">;
def __nvvm_ldu_ui4 : NVPTXBuiltin<"_ExtVector<4, unsigned int>(_ExtVector<4, unsigned int const *>)">;
def __nvvm_ldu_ul2 : NVPTXBuiltin<"_ExtVector<2, unsigned long int>(_ExtVector<2, unsigned long int const *>)">;
def __nvvm_ldu_ull2 : NVPTXBuiltin<"_ExtVector<2, unsigned long long int>(_ExtVector<2, unsigned long long int const *>)">;

def __nvvm_ldu_h2 : NVPTXBuiltin<"_ExtVector<2, __fp16>(_ExtVector<2, __fp16 const *>)">;
def __nvvm_ldu_f2 : NVPTXBuiltin<"_ExtVector<2, float>(_ExtVector<2, float const *>)">;
def __nvvm_ldu_f4 : NVPTXBuiltin<"_ExtVector<4, float>(_ExtVector<4, float const *>)">;
def __nvvm_ldu_d2 : NVPTXBuiltin<"_ExtVector<2, double>(_ExtVector<2, double const *>)">;

def __nvvm_ldg_c : NVPTXBuiltin<"char(char const *)">;
def __nvvm_ldg_sc : NVPTXBuiltin<"signed char(signed char const *)">;
def __nvvm_ldg_s : NVPTXBuiltin<"short(short const *)">;
def __nvvm_ldg_i : NVPTXBuiltin<"int(int const *)">;
def __nvvm_ldg_l : NVPTXBuiltin<"long int(long int const *)">;
def __nvvm_ldg_ll : NVPTXBuiltin<"long long int(long long int const *)">;

def __nvvm_ldg_uc : NVPTXBuiltin<"unsigned char(unsigned char const *)">;
def __nvvm_ldg_us : NVPTXBuiltin<"unsigned short(unsigned short const *)">;
def __nvvm_ldg_ui : NVPTXBuiltin<"unsigned int(unsigned int const *)">;
def __nvvm_ldg_ul : NVPTXBuiltin<"unsigned long int(unsigned long int const *)">;
def __nvvm_ldg_ull : NVPTXBuiltin<"unsigned long long int(unsigned long long int const *)">;
````
- **L1081 EN**: Declares TableGen def record `__nvvm_ldu_us2`.
  **L1081 CN**: 声明 TableGen def 记录 `__nvvm_ldu_us2`。
- **L1082 EN**: Declares TableGen def record `__nvvm_ldu_us4`.
  **L1082 CN**: 声明 TableGen def 记录 `__nvvm_ldu_us4`。
- **L1083 EN**: Declares TableGen def record `__nvvm_ldu_ui2`.
  **L1083 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ui2`。
- **L1084 EN**: Declares TableGen def record `__nvvm_ldu_ui4`.
  **L1084 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ui4`。
- **L1085 EN**: Declares TableGen def record `__nvvm_ldu_ul2`.
  **L1085 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ul2`。
- **L1086 EN**: Declares TableGen def record `__nvvm_ldu_ull2`.
  **L1086 CN**: 声明 TableGen def 记录 `__nvvm_ldu_ull2`。
- **L1087 EN**: Blank line separating nearby declarations or logic blocks.
  **L1087 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1088 EN**: Declares TableGen def record `__nvvm_ldu_h2`.
  **L1088 CN**: 声明 TableGen def 记录 `__nvvm_ldu_h2`。
- **L1089 EN**: Declares TableGen def record `__nvvm_ldu_f2`.
  **L1089 CN**: 声明 TableGen def 记录 `__nvvm_ldu_f2`。
- **L1090 EN**: Declares TableGen def record `__nvvm_ldu_f4`.
  **L1090 CN**: 声明 TableGen def 记录 `__nvvm_ldu_f4`。
- **L1091 EN**: Declares TableGen def record `__nvvm_ldu_d2`.
  **L1091 CN**: 声明 TableGen def 记录 `__nvvm_ldu_d2`。
- **L1092 EN**: Blank line separating nearby declarations or logic blocks.
  **L1092 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1093 EN**: Declares TableGen def record `__nvvm_ldg_c`.
  **L1093 CN**: 声明 TableGen def 记录 `__nvvm_ldg_c`。
- **L1094 EN**: Declares TableGen def record `__nvvm_ldg_sc`.
  **L1094 CN**: 声明 TableGen def 记录 `__nvvm_ldg_sc`。
- **L1095 EN**: Declares TableGen def record `__nvvm_ldg_s`.
  **L1095 CN**: 声明 TableGen def 记录 `__nvvm_ldg_s`。
- **L1096 EN**: Declares TableGen def record `__nvvm_ldg_i`.
  **L1096 CN**: 声明 TableGen def 记录 `__nvvm_ldg_i`。
- **L1097 EN**: Declares TableGen def record `__nvvm_ldg_l`.
  **L1097 CN**: 声明 TableGen def 记录 `__nvvm_ldg_l`。
- **L1098 EN**: Declares TableGen def record `__nvvm_ldg_ll`.
  **L1098 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ll`。
- **L1099 EN**: Blank line separating nearby declarations or logic blocks.
  **L1099 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1100 EN**: Declares TableGen def record `__nvvm_ldg_uc`.
  **L1100 CN**: 声明 TableGen def 记录 `__nvvm_ldg_uc`。
- **L1101 EN**: Declares TableGen def record `__nvvm_ldg_us`.
  **L1101 CN**: 声明 TableGen def 记录 `__nvvm_ldg_us`。
- **L1102 EN**: Declares TableGen def record `__nvvm_ldg_ui`.
  **L1102 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ui`。
- **L1103 EN**: Declares TableGen def record `__nvvm_ldg_ul`.
  **L1103 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ul`。
- **L1104 EN**: Declares TableGen def record `__nvvm_ldg_ull`.
  **L1104 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ull`。

### Lines 1105-1128

````tablegen

def __nvvm_ldg_h : NVPTXBuiltin<"__fp16(__fp16 const *)">;
def __nvvm_ldg_f : NVPTXBuiltin<"float(float const *)">;
def __nvvm_ldg_d : NVPTXBuiltin<"double(double const *)">;

def __nvvm_ldg_c2 : NVPTXBuiltin<"_ExtVector<2, char>(_ExtVector<2, char const *>)">;
def __nvvm_ldg_sc2 : NVPTXBuiltin<"_ExtVector<2, signed char>(_ExtVector<2, signed char const *>)">;
def __nvvm_ldg_c4 : NVPTXBuiltin<"_ExtVector<4, char>(_ExtVector<4, char const *>)">;
def __nvvm_ldg_sc4 : NVPTXBuiltin<"_ExtVector<4, signed char>(_ExtVector<4, signed char const *>)">;
def __nvvm_ldg_s2 : NVPTXBuiltin<"_ExtVector<2, short>(_ExtVector<2, short const *>)">;
def __nvvm_ldg_s4 : NVPTXBuiltin<"_ExtVector<4, short>(_ExtVector<4, short const *>)">;
def __nvvm_ldg_i2 : NVPTXBuiltin<"_ExtVector<2, int>(_ExtVector<2, int const *>)">;
def __nvvm_ldg_i4 : NVPTXBuiltin<"_ExtVector<4, int>(_ExtVector<4, int const *>)">;
def __nvvm_ldg_l2 : NVPTXBuiltin<"_ExtVector<2, long int>(_ExtVector<2, long int const *>)">;
def __nvvm_ldg_ll2 : NVPTXBuiltin<"_ExtVector<2, long long int>(_ExtVector<2, long long int const *>)">;

def __nvvm_ldg_uc2 : NVPTXBuiltin<"_ExtVector<2, unsigned char>(_ExtVector<2, unsigned char const *>)">;
def __nvvm_ldg_uc4 : NVPTXBuiltin<"_ExtVector<4, unsigned char>(_ExtVector<4, unsigned char const *>)">;
def __nvvm_ldg_us2 : NVPTXBuiltin<"_ExtVector<2, unsigned short>(_ExtVector<2, unsigned short const *>)">;
def __nvvm_ldg_us4 : NVPTXBuiltin<"_ExtVector<4, unsigned short>(_ExtVector<4, unsigned short const *>)">;
def __nvvm_ldg_ui2 : NVPTXBuiltin<"_ExtVector<2, unsigned int>(_ExtVector<2, unsigned int const *>)">;
def __nvvm_ldg_ui4 : NVPTXBuiltin<"_ExtVector<4, unsigned int>(_ExtVector<4, unsigned int const *>)">;
def __nvvm_ldg_ul2 : NVPTXBuiltin<"_ExtVector<2, unsigned long int>(_ExtVector<2, unsigned long int const *>)">;
def __nvvm_ldg_ull2 : NVPTXBuiltin<"_ExtVector<2, unsigned long long int>(_ExtVector<2, unsigned long long int const *>)">;
````
- **L1105 EN**: Blank line separating nearby declarations or logic blocks.
  **L1105 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1106 EN**: Declares TableGen def record `__nvvm_ldg_h`.
  **L1106 CN**: 声明 TableGen def 记录 `__nvvm_ldg_h`。
- **L1107 EN**: Declares TableGen def record `__nvvm_ldg_f`.
  **L1107 CN**: 声明 TableGen def 记录 `__nvvm_ldg_f`。
- **L1108 EN**: Declares TableGen def record `__nvvm_ldg_d`.
  **L1108 CN**: 声明 TableGen def 记录 `__nvvm_ldg_d`。
- **L1109 EN**: Blank line separating nearby declarations or logic blocks.
  **L1109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1110 EN**: Declares TableGen def record `__nvvm_ldg_c2`.
  **L1110 CN**: 声明 TableGen def 记录 `__nvvm_ldg_c2`。
- **L1111 EN**: Declares TableGen def record `__nvvm_ldg_sc2`.
  **L1111 CN**: 声明 TableGen def 记录 `__nvvm_ldg_sc2`。
- **L1112 EN**: Declares TableGen def record `__nvvm_ldg_c4`.
  **L1112 CN**: 声明 TableGen def 记录 `__nvvm_ldg_c4`。
- **L1113 EN**: Declares TableGen def record `__nvvm_ldg_sc4`.
  **L1113 CN**: 声明 TableGen def 记录 `__nvvm_ldg_sc4`。
- **L1114 EN**: Declares TableGen def record `__nvvm_ldg_s2`.
  **L1114 CN**: 声明 TableGen def 记录 `__nvvm_ldg_s2`。
- **L1115 EN**: Declares TableGen def record `__nvvm_ldg_s4`.
  **L1115 CN**: 声明 TableGen def 记录 `__nvvm_ldg_s4`。
- **L1116 EN**: Declares TableGen def record `__nvvm_ldg_i2`.
  **L1116 CN**: 声明 TableGen def 记录 `__nvvm_ldg_i2`。
- **L1117 EN**: Declares TableGen def record `__nvvm_ldg_i4`.
  **L1117 CN**: 声明 TableGen def 记录 `__nvvm_ldg_i4`。
- **L1118 EN**: Declares TableGen def record `__nvvm_ldg_l2`.
  **L1118 CN**: 声明 TableGen def 记录 `__nvvm_ldg_l2`。
- **L1119 EN**: Declares TableGen def record `__nvvm_ldg_ll2`.
  **L1119 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ll2`。
- **L1120 EN**: Blank line separating nearby declarations or logic blocks.
  **L1120 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1121 EN**: Declares TableGen def record `__nvvm_ldg_uc2`.
  **L1121 CN**: 声明 TableGen def 记录 `__nvvm_ldg_uc2`。
- **L1122 EN**: Declares TableGen def record `__nvvm_ldg_uc4`.
  **L1122 CN**: 声明 TableGen def 记录 `__nvvm_ldg_uc4`。
- **L1123 EN**: Declares TableGen def record `__nvvm_ldg_us2`.
  **L1123 CN**: 声明 TableGen def 记录 `__nvvm_ldg_us2`。
- **L1124 EN**: Declares TableGen def record `__nvvm_ldg_us4`.
  **L1124 CN**: 声明 TableGen def 记录 `__nvvm_ldg_us4`。
- **L1125 EN**: Declares TableGen def record `__nvvm_ldg_ui2`.
  **L1125 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ui2`。
- **L1126 EN**: Declares TableGen def record `__nvvm_ldg_ui4`.
  **L1126 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ui4`。
- **L1127 EN**: Declares TableGen def record `__nvvm_ldg_ul2`.
  **L1127 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ul2`。
- **L1128 EN**: Declares TableGen def record `__nvvm_ldg_ull2`.
  **L1128 CN**: 声明 TableGen def 记录 `__nvvm_ldg_ull2`。

### Lines 1129-1152

````tablegen

def __nvvm_ldg_h2 : NVPTXBuiltin<"_ExtVector<2, __fp16>(_ExtVector<2, __fp16 const *>)">;
def __nvvm_ldg_f2 : NVPTXBuiltin<"_ExtVector<2, float>(_ExtVector<2, float const *>)">;
def __nvvm_ldg_f4 : NVPTXBuiltin<"_ExtVector<4, float>(_ExtVector<4, float const *>)">;
def __nvvm_ldg_d2 : NVPTXBuiltin<"_ExtVector<2, double>(_ExtVector<2, double const *>)">;

// Address space predicates.
let Attributes = [NoThrow, Const] in {
  def __nvvm_isspacep_const : NVPTXBuiltin<"bool(void const *)">;
  def __nvvm_isspacep_global : NVPTXBuiltin<"bool(void const *)">;
  def __nvvm_isspacep_local : NVPTXBuiltin<"bool(void const *)">;
  def __nvvm_isspacep_shared : NVPTXBuiltin<"bool(void const *)">;
  def __nvvm_isspacep_shared_cluster : NVPTXBuiltinSMAndPTX<"bool(void const *)", SM_90, PTX78>;
}

// Builtins to support WMMA instructions on sm_70
def __hmma_m16n16k16_ld_a : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_ld_b : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_ld_c_f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_ld_c_f32 : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_st_c_f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_st_c_f32 : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_70, PTX60>;

def __hmma_m32n8k16_ld_a : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
````
- **L1129 EN**: Blank line separating nearby declarations or logic blocks.
  **L1129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1130 EN**: Declares TableGen def record `__nvvm_ldg_h2`.
  **L1130 CN**: 声明 TableGen def 记录 `__nvvm_ldg_h2`。
- **L1131 EN**: Declares TableGen def record `__nvvm_ldg_f2`.
  **L1131 CN**: 声明 TableGen def 记录 `__nvvm_ldg_f2`。
- **L1132 EN**: Declares TableGen def record `__nvvm_ldg_f4`.
  **L1132 CN**: 声明 TableGen def 记录 `__nvvm_ldg_f4`。
- **L1133 EN**: Declares TableGen def record `__nvvm_ldg_d2`.
  **L1133 CN**: 声明 TableGen def 记录 `__nvvm_ldg_d2`。
- **L1134 EN**: Blank line separating nearby declarations or logic blocks.
  **L1134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1135 EN**: Comment explains nearby logic, constraints, or intent: `Address space predicates.`.
  **L1135 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Address space predicates.`。
- **L1136 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Attributes = [NoThrow, Const] in {`.
  **L1136 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Attributes = [NoThrow, Const] in {`。
- **L1137 EN**: Declares TableGen def record `__nvvm_isspacep_const`.
  **L1137 CN**: 声明 TableGen def 记录 `__nvvm_isspacep_const`。
- **L1138 EN**: Declares TableGen def record `__nvvm_isspacep_global`.
  **L1138 CN**: 声明 TableGen def 记录 `__nvvm_isspacep_global`。
- **L1139 EN**: Declares TableGen def record `__nvvm_isspacep_local`.
  **L1139 CN**: 声明 TableGen def 记录 `__nvvm_isspacep_local`。
- **L1140 EN**: Declares TableGen def record `__nvvm_isspacep_shared`.
  **L1140 CN**: 声明 TableGen def 记录 `__nvvm_isspacep_shared`。
- **L1141 EN**: Declares TableGen def record `__nvvm_isspacep_shared_cluster`.
  **L1141 CN**: 声明 TableGen def 记录 `__nvvm_isspacep_shared_cluster`。
- **L1142 EN**: Closes the current lexical scope or compound statement.
  **L1142 CN**: 结束当前词法作用域或复合语句块。
- **L1143 EN**: Blank line separating nearby declarations or logic blocks.
  **L1143 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1144 EN**: Comment explains nearby logic, constraints, or intent: `Builtins to support WMMA instructions on sm_70`.
  **L1144 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Builtins to support WMMA instructions on sm_70`。
- **L1145 EN**: Declares TableGen def record `__hmma_m16n16k16_ld_a`.
  **L1145 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_ld_a`。
- **L1146 EN**: Declares TableGen def record `__hmma_m16n16k16_ld_b`.
  **L1146 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_ld_b`。
- **L1147 EN**: Declares TableGen def record `__hmma_m16n16k16_ld_c_f16`.
  **L1147 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_ld_c_f16`。
- **L1148 EN**: Declares TableGen def record `__hmma_m16n16k16_ld_c_f32`.
  **L1148 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_ld_c_f32`。
- **L1149 EN**: Declares TableGen def record `__hmma_m16n16k16_st_c_f16`.
  **L1149 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_st_c_f16`。
- **L1150 EN**: Declares TableGen def record `__hmma_m16n16k16_st_c_f32`.
  **L1150 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_st_c_f32`。
- **L1151 EN**: Blank line separating nearby declarations or logic blocks.
  **L1151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1152 EN**: Declares TableGen def record `__hmma_m32n8k16_ld_a`.
  **L1152 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_ld_a`。

### Lines 1153-1176

````tablegen
def __hmma_m32n8k16_ld_b : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m32n8k16_ld_c_f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m32n8k16_ld_c_f32 : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m32n8k16_st_c_f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m32n8k16_st_c_f32 : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_70, PTX61>;

def __hmma_m8n32k16_ld_a : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m8n32k16_ld_b : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m8n32k16_ld_c_f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m8n32k16_ld_c_f32 : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m8n32k16_st_c_f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_70, PTX61>;
def __hmma_m8n32k16_st_c_f32 : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_70, PTX61>;

def __hmma_m16n16k16_mma_f16f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_mma_f32f16 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_mma_f32f32 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_70, PTX60>;
def __hmma_m16n16k16_mma_f16f32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_70, PTX60>;

def __hmma_m32n8k16_mma_f16f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_70, PTX61>;
def __hmma_m32n8k16_mma_f32f16 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_70, PTX61>;
def __hmma_m32n8k16_mma_f32f32 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_70, PTX61>;
def __hmma_m32n8k16_mma_f16f32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_70, PTX61>;

def __hmma_m8n32k16_mma_f16f16 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_70, PTX61>;
````
- **L1153 EN**: Declares TableGen def record `__hmma_m32n8k16_ld_b`.
  **L1153 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_ld_b`。
- **L1154 EN**: Declares TableGen def record `__hmma_m32n8k16_ld_c_f16`.
  **L1154 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_ld_c_f16`。
- **L1155 EN**: Declares TableGen def record `__hmma_m32n8k16_ld_c_f32`.
  **L1155 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_ld_c_f32`。
- **L1156 EN**: Declares TableGen def record `__hmma_m32n8k16_st_c_f16`.
  **L1156 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_st_c_f16`。
- **L1157 EN**: Declares TableGen def record `__hmma_m32n8k16_st_c_f32`.
  **L1157 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_st_c_f32`。
- **L1158 EN**: Blank line separating nearby declarations or logic blocks.
  **L1158 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1159 EN**: Declares TableGen def record `__hmma_m8n32k16_ld_a`.
  **L1159 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_ld_a`。
- **L1160 EN**: Declares TableGen def record `__hmma_m8n32k16_ld_b`.
  **L1160 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_ld_b`。
- **L1161 EN**: Declares TableGen def record `__hmma_m8n32k16_ld_c_f16`.
  **L1161 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_ld_c_f16`。
- **L1162 EN**: Declares TableGen def record `__hmma_m8n32k16_ld_c_f32`.
  **L1162 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_ld_c_f32`。
- **L1163 EN**: Declares TableGen def record `__hmma_m8n32k16_st_c_f16`.
  **L1163 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_st_c_f16`。
- **L1164 EN**: Declares TableGen def record `__hmma_m8n32k16_st_c_f32`.
  **L1164 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_st_c_f32`。
- **L1165 EN**: Blank line separating nearby declarations or logic blocks.
  **L1165 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1166 EN**: Declares TableGen def record `__hmma_m16n16k16_mma_f16f16`.
  **L1166 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_mma_f16f16`。
- **L1167 EN**: Declares TableGen def record `__hmma_m16n16k16_mma_f32f16`.
  **L1167 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_mma_f32f16`。
- **L1168 EN**: Declares TableGen def record `__hmma_m16n16k16_mma_f32f32`.
  **L1168 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_mma_f32f32`。
- **L1169 EN**: Declares TableGen def record `__hmma_m16n16k16_mma_f16f32`.
  **L1169 CN**: 声明 TableGen def 记录 `__hmma_m16n16k16_mma_f16f32`。
- **L1170 EN**: Blank line separating nearby declarations or logic blocks.
  **L1170 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1171 EN**: Declares TableGen def record `__hmma_m32n8k16_mma_f16f16`.
  **L1171 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_mma_f16f16`。
- **L1172 EN**: Declares TableGen def record `__hmma_m32n8k16_mma_f32f16`.
  **L1172 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_mma_f32f16`。
- **L1173 EN**: Declares TableGen def record `__hmma_m32n8k16_mma_f32f32`.
  **L1173 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_mma_f32f32`。
- **L1174 EN**: Declares TableGen def record `__hmma_m32n8k16_mma_f16f32`.
  **L1174 CN**: 声明 TableGen def 记录 `__hmma_m32n8k16_mma_f16f32`。
- **L1175 EN**: Blank line separating nearby declarations or logic blocks.
  **L1175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1176 EN**: Declares TableGen def record `__hmma_m8n32k16_mma_f16f16`.
  **L1176 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_mma_f16f16`。

### Lines 1177-1200

````tablegen
def __hmma_m8n32k16_mma_f32f16 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_70, PTX61>;
def __hmma_m8n32k16_mma_f32f32 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_70, PTX61>;
def __hmma_m8n32k16_mma_f16f32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_70, PTX61>;

// Builtins to support integer and sub-integer WMMA instructions on sm_72/sm_75
def __bmma_m8n8k128_ld_a_b1 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __bmma_m8n8k128_ld_b_b1 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __bmma_m8n8k128_ld_c : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __bmma_m8n8k128_mma_and_popc_b1 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int)", SM_80, PTX71>;
def __bmma_m8n8k128_mma_xor_popc_b1 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int)", SM_75, PTX63>;
def __bmma_m8n8k128_st_c_i32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __imma_m16n16k16_ld_a_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m16n16k16_ld_a_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m16n16k16_ld_b_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m16n16k16_ld_b_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m16n16k16_ld_c : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m16n16k16_mma_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_72, PTX63>;
def __imma_m16n16k16_mma_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_72, PTX63>;
def __imma_m16n16k16_st_c_i32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m32n8k16_ld_a_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m32n8k16_ld_a_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m32n8k16_ld_b_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m32n8k16_ld_b_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m32n8k16_ld_c : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
````
- **L1177 EN**: Declares TableGen def record `__hmma_m8n32k16_mma_f32f16`.
  **L1177 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_mma_f32f16`。
- **L1178 EN**: Declares TableGen def record `__hmma_m8n32k16_mma_f32f32`.
  **L1178 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_mma_f32f32`。
- **L1179 EN**: Declares TableGen def record `__hmma_m8n32k16_mma_f16f32`.
  **L1179 CN**: 声明 TableGen def 记录 `__hmma_m8n32k16_mma_f16f32`。
- **L1180 EN**: Blank line separating nearby declarations or logic blocks.
  **L1180 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1181 EN**: Comment explains nearby logic, constraints, or intent: `Builtins to support integer and sub-integer WMMA instructions on sm_72/sm_75`.
  **L1181 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Builtins to support integer and sub-integer WMMA instructions on sm_72/sm_75`。
- **L1182 EN**: Declares TableGen def record `__bmma_m8n8k128_ld_a_b1`.
  **L1182 CN**: 声明 TableGen def 记录 `__bmma_m8n8k128_ld_a_b1`。
- **L1183 EN**: Declares TableGen def record `__bmma_m8n8k128_ld_b_b1`.
  **L1183 CN**: 声明 TableGen def 记录 `__bmma_m8n8k128_ld_b_b1`。
- **L1184 EN**: Declares TableGen def record `__bmma_m8n8k128_ld_c`.
  **L1184 CN**: 声明 TableGen def 记录 `__bmma_m8n8k128_ld_c`。
- **L1185 EN**: Declares TableGen def record `__bmma_m8n8k128_mma_and_popc_b1`.
  **L1185 CN**: 声明 TableGen def 记录 `__bmma_m8n8k128_mma_and_popc_b1`。
- **L1186 EN**: Declares TableGen def record `__bmma_m8n8k128_mma_xor_popc_b1`.
  **L1186 CN**: 声明 TableGen def 记录 `__bmma_m8n8k128_mma_xor_popc_b1`。
- **L1187 EN**: Declares TableGen def record `__bmma_m8n8k128_st_c_i32`.
  **L1187 CN**: 声明 TableGen def 记录 `__bmma_m8n8k128_st_c_i32`。
- **L1188 EN**: Declares TableGen def record `__imma_m16n16k16_ld_a_s8`.
  **L1188 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_ld_a_s8`。
- **L1189 EN**: Declares TableGen def record `__imma_m16n16k16_ld_a_u8`.
  **L1189 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_ld_a_u8`。
- **L1190 EN**: Declares TableGen def record `__imma_m16n16k16_ld_b_s8`.
  **L1190 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_ld_b_s8`。
- **L1191 EN**: Declares TableGen def record `__imma_m16n16k16_ld_b_u8`.
  **L1191 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_ld_b_u8`。
- **L1192 EN**: Declares TableGen def record `__imma_m16n16k16_ld_c`.
  **L1192 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_ld_c`。
- **L1193 EN**: Declares TableGen def record `__imma_m16n16k16_mma_s8`.
  **L1193 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_mma_s8`。
- **L1194 EN**: Declares TableGen def record `__imma_m16n16k16_mma_u8`.
  **L1194 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_mma_u8`。
- **L1195 EN**: Declares TableGen def record `__imma_m16n16k16_st_c_i32`.
  **L1195 CN**: 声明 TableGen def 记录 `__imma_m16n16k16_st_c_i32`。
- **L1196 EN**: Declares TableGen def record `__imma_m32n8k16_ld_a_s8`.
  **L1196 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_ld_a_s8`。
- **L1197 EN**: Declares TableGen def record `__imma_m32n8k16_ld_a_u8`.
  **L1197 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_ld_a_u8`。
- **L1198 EN**: Declares TableGen def record `__imma_m32n8k16_ld_b_s8`.
  **L1198 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_ld_b_s8`。
- **L1199 EN**: Declares TableGen def record `__imma_m32n8k16_ld_b_u8`.
  **L1199 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_ld_b_u8`。
- **L1200 EN**: Declares TableGen def record `__imma_m32n8k16_ld_c`.
  **L1200 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_ld_c`。

### Lines 1201-1224

````tablegen
def __imma_m32n8k16_mma_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_72, PTX63>;
def __imma_m32n8k16_mma_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_72, PTX63>;
def __imma_m32n8k16_st_c_i32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_ld_a_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_ld_a_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_ld_b_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_ld_b_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_ld_c : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_mma_s8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_mma_u8 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_72, PTX63>;
def __imma_m8n32k16_st_c_i32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_72, PTX63>;
def __imma_m8n8k32_ld_a_s4 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __imma_m8n8k32_ld_a_u4 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __imma_m8n8k32_ld_b_s4 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __imma_m8n8k32_ld_b_u4 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __imma_m8n8k32_ld_c : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;
def __imma_m8n8k32_mma_s4 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_75, PTX63>;
def __imma_m8n8k32_mma_u4 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, int const *, int const *, _Constant int, _Constant int)", SM_75, PTX63>;
def __imma_m8n8k32_st_c_i32 : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_75, PTX63>;

// Builtins to support double and alternate float WMMA instructions on sm_80
def __dmma_m8n8k4_ld_a : NVPTXBuiltinSMAndPTX<"void(double *, double const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __dmma_m8n8k4_ld_b : NVPTXBuiltinSMAndPTX<"void(double *, double const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __dmma_m8n8k4_ld_c : NVPTXBuiltinSMAndPTX<"void(double *, double const *, unsigned int, _Constant int)", SM_80, PTX70>;
````
- **L1201 EN**: Declares TableGen def record `__imma_m32n8k16_mma_s8`.
  **L1201 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_mma_s8`。
- **L1202 EN**: Declares TableGen def record `__imma_m32n8k16_mma_u8`.
  **L1202 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_mma_u8`。
- **L1203 EN**: Declares TableGen def record `__imma_m32n8k16_st_c_i32`.
  **L1203 CN**: 声明 TableGen def 记录 `__imma_m32n8k16_st_c_i32`。
- **L1204 EN**: Declares TableGen def record `__imma_m8n32k16_ld_a_s8`.
  **L1204 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_ld_a_s8`。
- **L1205 EN**: Declares TableGen def record `__imma_m8n32k16_ld_a_u8`.
  **L1205 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_ld_a_u8`。
- **L1206 EN**: Declares TableGen def record `__imma_m8n32k16_ld_b_s8`.
  **L1206 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_ld_b_s8`。
- **L1207 EN**: Declares TableGen def record `__imma_m8n32k16_ld_b_u8`.
  **L1207 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_ld_b_u8`。
- **L1208 EN**: Declares TableGen def record `__imma_m8n32k16_ld_c`.
  **L1208 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_ld_c`。
- **L1209 EN**: Declares TableGen def record `__imma_m8n32k16_mma_s8`.
  **L1209 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_mma_s8`。
- **L1210 EN**: Declares TableGen def record `__imma_m8n32k16_mma_u8`.
  **L1210 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_mma_u8`。
- **L1211 EN**: Declares TableGen def record `__imma_m8n32k16_st_c_i32`.
  **L1211 CN**: 声明 TableGen def 记录 `__imma_m8n32k16_st_c_i32`。
- **L1212 EN**: Declares TableGen def record `__imma_m8n8k32_ld_a_s4`.
  **L1212 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_ld_a_s4`。
- **L1213 EN**: Declares TableGen def record `__imma_m8n8k32_ld_a_u4`.
  **L1213 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_ld_a_u4`。
- **L1214 EN**: Declares TableGen def record `__imma_m8n8k32_ld_b_s4`.
  **L1214 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_ld_b_s4`。
- **L1215 EN**: Declares TableGen def record `__imma_m8n8k32_ld_b_u4`.
  **L1215 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_ld_b_u4`。
- **L1216 EN**: Declares TableGen def record `__imma_m8n8k32_ld_c`.
  **L1216 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_ld_c`。
- **L1217 EN**: Declares TableGen def record `__imma_m8n8k32_mma_s4`.
  **L1217 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_mma_s4`。
- **L1218 EN**: Declares TableGen def record `__imma_m8n8k32_mma_u4`.
  **L1218 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_mma_u4`。
- **L1219 EN**: Declares TableGen def record `__imma_m8n8k32_st_c_i32`.
  **L1219 CN**: 声明 TableGen def 记录 `__imma_m8n8k32_st_c_i32`。
- **L1220 EN**: Blank line separating nearby declarations or logic blocks.
  **L1220 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1221 EN**: Comment explains nearby logic, constraints, or intent: `Builtins to support double and alternate float WMMA instructions on sm_80`.
  **L1221 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Builtins to support double and alternate float WMMA instructions on sm_80`。
- **L1222 EN**: Declares TableGen def record `__dmma_m8n8k4_ld_a`.
  **L1222 CN**: 声明 TableGen def 记录 `__dmma_m8n8k4_ld_a`。
- **L1223 EN**: Declares TableGen def record `__dmma_m8n8k4_ld_b`.
  **L1223 CN**: 声明 TableGen def 记录 `__dmma_m8n8k4_ld_b`。
- **L1224 EN**: Declares TableGen def record `__dmma_m8n8k4_ld_c`.
  **L1224 CN**: 声明 TableGen def 记录 `__dmma_m8n8k4_ld_c`。

### Lines 1225-1248

````tablegen
def __dmma_m8n8k4_st_c_f64 : NVPTXBuiltinSMAndPTX<"void(double *, double const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __dmma_m8n8k4_mma_f64 : NVPTXBuiltinSMAndPTX<"void(double *, double const *, double const *, double const *, _Constant int, _Constant int)", SM_80, PTX70>;

def __mma_bf16_m16n16k16_ld_a : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m16n16k16_ld_b : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m16n16k16_mma_f32 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m8n32k16_ld_a : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m8n32k16_ld_b : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m8n32k16_mma_f32 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m32n8k16_ld_a : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m32n8k16_ld_b : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_bf16_m32n8k16_mma_f32 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_80, PTX70>;

def __mma_tf32_m16n16k8_ld_a : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_tf32_m16n16k8_ld_b : NVPTXBuiltinSMAndPTX<"void(int *, int const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_tf32_m16n16k8_ld_c : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_m16n16k8_st_c_f32 : NVPTXBuiltinSMAndPTX<"void(float *, float const *, unsigned int, _Constant int)", SM_80, PTX70>;
def __mma_tf32_m16n16k8_mma_f32 : NVPTXBuiltinSMAndPTX<"void(float *, int const *, int const *, float const *, _Constant int, _Constant int)", SM_80, PTX70>;

// Async Copy
def __nvvm_cp_async_mbarrier_arrive : NVPTXBuiltinSMAndPTX<"void(int64_t *)", SM_80, PTX70>;
def __nvvm_cp_async_mbarrier_arrive_shared : NVPTXBuiltinSMAndPTX<"void(int64_t address_space<3> *)", SM_80, PTX70>;
def __nvvm_cp_async_mbarrier_arrive_noinc : NVPTXBuiltinSMAndPTX<"void(int64_t *)", SM_80, PTX70>;
def __nvvm_cp_async_mbarrier_arrive_noinc_shared : NVPTXBuiltinSMAndPTX<"void(int64_t address_space<3> *)", SM_80, PTX70>;
````
- **L1225 EN**: Declares TableGen def record `__dmma_m8n8k4_st_c_f64`.
  **L1225 CN**: 声明 TableGen def 记录 `__dmma_m8n8k4_st_c_f64`。
- **L1226 EN**: Declares TableGen def record `__dmma_m8n8k4_mma_f64`.
  **L1226 CN**: 声明 TableGen def 记录 `__dmma_m8n8k4_mma_f64`。
- **L1227 EN**: Blank line separating nearby declarations or logic blocks.
  **L1227 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1228 EN**: Declares TableGen def record `__mma_bf16_m16n16k16_ld_a`.
  **L1228 CN**: 声明 TableGen def 记录 `__mma_bf16_m16n16k16_ld_a`。
- **L1229 EN**: Declares TableGen def record `__mma_bf16_m16n16k16_ld_b`.
  **L1229 CN**: 声明 TableGen def 记录 `__mma_bf16_m16n16k16_ld_b`。
- **L1230 EN**: Declares TableGen def record `__mma_bf16_m16n16k16_mma_f32`.
  **L1230 CN**: 声明 TableGen def 记录 `__mma_bf16_m16n16k16_mma_f32`。
- **L1231 EN**: Declares TableGen def record `__mma_bf16_m8n32k16_ld_a`.
  **L1231 CN**: 声明 TableGen def 记录 `__mma_bf16_m8n32k16_ld_a`。
- **L1232 EN**: Declares TableGen def record `__mma_bf16_m8n32k16_ld_b`.
  **L1232 CN**: 声明 TableGen def 记录 `__mma_bf16_m8n32k16_ld_b`。
- **L1233 EN**: Declares TableGen def record `__mma_bf16_m8n32k16_mma_f32`.
  **L1233 CN**: 声明 TableGen def 记录 `__mma_bf16_m8n32k16_mma_f32`。
- **L1234 EN**: Declares TableGen def record `__mma_bf16_m32n8k16_ld_a`.
  **L1234 CN**: 声明 TableGen def 记录 `__mma_bf16_m32n8k16_ld_a`。
- **L1235 EN**: Declares TableGen def record `__mma_bf16_m32n8k16_ld_b`.
  **L1235 CN**: 声明 TableGen def 记录 `__mma_bf16_m32n8k16_ld_b`。
- **L1236 EN**: Declares TableGen def record `__mma_bf16_m32n8k16_mma_f32`.
  **L1236 CN**: 声明 TableGen def 记录 `__mma_bf16_m32n8k16_mma_f32`。
- **L1237 EN**: Blank line separating nearby declarations or logic blocks.
  **L1237 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1238 EN**: Declares TableGen def record `__mma_tf32_m16n16k8_ld_a`.
  **L1238 CN**: 声明 TableGen def 记录 `__mma_tf32_m16n16k8_ld_a`。
- **L1239 EN**: Declares TableGen def record `__mma_tf32_m16n16k8_ld_b`.
  **L1239 CN**: 声明 TableGen def 记录 `__mma_tf32_m16n16k8_ld_b`。
- **L1240 EN**: Declares TableGen def record `__mma_tf32_m16n16k8_ld_c`.
  **L1240 CN**: 声明 TableGen def 记录 `__mma_tf32_m16n16k8_ld_c`。
- **L1241 EN**: Declares TableGen def record `__mma_m16n16k8_st_c_f32`.
  **L1241 CN**: 声明 TableGen def 记录 `__mma_m16n16k8_st_c_f32`。
- **L1242 EN**: Declares TableGen def record `__mma_tf32_m16n16k8_mma_f32`.
  **L1242 CN**: 声明 TableGen def 记录 `__mma_tf32_m16n16k8_mma_f32`。
- **L1243 EN**: Blank line separating nearby declarations or logic blocks.
  **L1243 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1244 EN**: Comment explains nearby logic, constraints, or intent: `Async Copy`.
  **L1244 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Async Copy`。
- **L1245 EN**: Declares TableGen def record `__nvvm_cp_async_mbarrier_arrive`.
  **L1245 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_mbarrier_arrive`。
- **L1246 EN**: Declares TableGen def record `__nvvm_cp_async_mbarrier_arrive_shared`.
  **L1246 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_mbarrier_arrive_shared`。
- **L1247 EN**: Declares TableGen def record `__nvvm_cp_async_mbarrier_arrive_noinc`.
  **L1247 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_mbarrier_arrive_noinc`。
- **L1248 EN**: Declares TableGen def record `__nvvm_cp_async_mbarrier_arrive_noinc_shared`.
  **L1248 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_mbarrier_arrive_noinc_shared`。

### Lines 1249-1269

````tablegen

def __nvvm_cp_async_ca_shared_global_4 : NVPTXBuiltinSMAndPTX<"void(void address_space<3> *, void const address_space<1> *, ...)", SM_80, PTX70>;
def __nvvm_cp_async_ca_shared_global_8 : NVPTXBuiltinSMAndPTX<"void(void address_space<3> *, void const address_space<1> *, ...)", SM_80, PTX70>;
def __nvvm_cp_async_ca_shared_global_16 : NVPTXBuiltinSMAndPTX<"void(void address_space<3> *, void const address_space<1> *, ...)", SM_80, PTX70>;
def __nvvm_cp_async_cg_shared_global_16 : NVPTXBuiltinSMAndPTX<"void(void address_space<3> *, void const address_space<1> *, ...)", SM_80, PTX70>;

def __nvvm_cp_async_commit_group : NVPTXBuiltinSMAndPTX<"void()", SM_80, PTX70>;
def __nvvm_cp_async_wait_group : NVPTXBuiltinSMAndPTX<"void(_Constant int)", SM_80, PTX70>;
def __nvvm_cp_async_wait_all : NVPTXBuiltinSMAndPTX<"void()", SM_80, PTX70>;


// bf16, bf16x2 abs, neg
def __nvvm_abs_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16)", SM_80, PTX70>;
def __nvvm_abs_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>)", SM_80, PTX70>;
def __nvvm_neg_bf16 : NVPTXBuiltinSMAndPTX<"__bf16(__bf16)", SM_80, PTX70>;
def __nvvm_neg_bf16x2 : NVPTXBuiltinSMAndPTX<"_Vector<2, __bf16>(_Vector<2, __bf16>)", SM_80, PTX70>;

def __nvvm_mapa : NVPTXBuiltinSMAndPTX<"void *(void *, int)", SM_90, PTX78>;
def __nvvm_mapa_shared_cluster : NVPTXBuiltinSMAndPTX<"void address_space<3> *(void address_space<3> *, int)", SM_90, PTX78>;
def __nvvm_getctarank : NVPTXBuiltinSMAndPTX<"int(void *)", SM_90, PTX78>;
def __nvvm_getctarank_shared_cluster : NVPTXBuiltinSMAndPTX<"int(void address_space<3> *)", SM_90, PTX78>;
````
- **L1249 EN**: Blank line separating nearby declarations or logic blocks.
  **L1249 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1250 EN**: Declares TableGen def record `__nvvm_cp_async_ca_shared_global_4`.
  **L1250 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_ca_shared_global_4`。
- **L1251 EN**: Declares TableGen def record `__nvvm_cp_async_ca_shared_global_8`.
  **L1251 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_ca_shared_global_8`。
- **L1252 EN**: Declares TableGen def record `__nvvm_cp_async_ca_shared_global_16`.
  **L1252 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_ca_shared_global_16`。
- **L1253 EN**: Declares TableGen def record `__nvvm_cp_async_cg_shared_global_16`.
  **L1253 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_cg_shared_global_16`。
- **L1254 EN**: Blank line separating nearby declarations or logic blocks.
  **L1254 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1255 EN**: Declares TableGen def record `__nvvm_cp_async_commit_group`.
  **L1255 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_commit_group`。
- **L1256 EN**: Declares TableGen def record `__nvvm_cp_async_wait_group`.
  **L1256 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_wait_group`。
- **L1257 EN**: Declares TableGen def record `__nvvm_cp_async_wait_all`.
  **L1257 CN**: 声明 TableGen def 记录 `__nvvm_cp_async_wait_all`。
- **L1258 EN**: Blank line separating nearby declarations or logic blocks.
  **L1258 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1259 EN**: Blank line separating nearby declarations or logic blocks.
  **L1259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1260 EN**: Comment explains nearby logic, constraints, or intent: `bf16, bf16x2 abs, neg`.
  **L1260 CN**: 注释解释附近代码的逻辑、约束或设计意图：`bf16, bf16x2 abs, neg`。
- **L1261 EN**: Declares TableGen def record `__nvvm_abs_bf16`.
  **L1261 CN**: 声明 TableGen def 记录 `__nvvm_abs_bf16`。
- **L1262 EN**: Declares TableGen def record `__nvvm_abs_bf16x2`.
  **L1262 CN**: 声明 TableGen def 记录 `__nvvm_abs_bf16x2`。
- **L1263 EN**: Declares TableGen def record `__nvvm_neg_bf16`.
  **L1263 CN**: 声明 TableGen def 记录 `__nvvm_neg_bf16`。
- **L1264 EN**: Declares TableGen def record `__nvvm_neg_bf16x2`.
  **L1264 CN**: 声明 TableGen def 记录 `__nvvm_neg_bf16x2`。
- **L1265 EN**: Blank line separating nearby declarations or logic blocks.
  **L1265 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1266 EN**: Declares TableGen def record `__nvvm_mapa`.
  **L1266 CN**: 声明 TableGen def 记录 `__nvvm_mapa`。
- **L1267 EN**: Declares TableGen def record `__nvvm_mapa_shared_cluster`.
  **L1267 CN**: 声明 TableGen def 记录 `__nvvm_mapa_shared_cluster`。
- **L1268 EN**: Declares TableGen def record `__nvvm_getctarank`.
  **L1268 CN**: 声明 TableGen def 记录 `__nvvm_getctarank`。
- **L1269 EN**: Declares TableGen def record `__nvvm_getctarank_shared_cluster`.
  **L1269 CN**: 声明 TableGen def 记录 `__nvvm_getctarank_shared_cluster`。

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
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `SMFeatures`, `SMa`, `PTXFeatures`, `NVPTXBuiltin`, `NVPTXBuiltinSM`, `NVPTXBuiltinPTX`, `NVPTXBuiltinSMAndPTX`
- **Functions or callables / 函数或可调用对象**: `interleave`, `div`, `foreach`, `listconcat`, `ge`, `not`, `range`, `order`, `strconcat`, `int`, `bool`, `void`
- **TableGen records / TableGen 记录**: `SMFeatures`, `SMa`, `SM_Base`, `family`, `gpu_bases`, `gpu_fs`, `gpu_as`, `same_family_newer_fs_as`, `f_features`, `a`, `f`, `""`, `SM_Instantiate`, `sm`, `gpus`, `_#sm`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
