# riscv_vector.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/riscv_vector.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: RISC-V V-ext Builtin function list.
- **Purpose (CN)**: 声明与 `riscv_vector` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 2159

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

````tablegen
//==--- riscv_vector.td - RISC-V V-ext Builtin function list --------------===//
//
//  Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
//  See https://llvm.org/LICENSE.txt for license information.
//  SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the builtins for RISC-V V-extension. See:
//
//     https://github.com/riscv/rvv-intrinsic-doc
//
//===----------------------------------------------------------------------===//

include "riscv_vector_common.td"

defvar TypeList = ["c","s","i","l","x","f","d","y"];
defvar EEWList = [["8", "(Log2EEW:3)"],
                  ["16", "(Log2EEW:4)"],
                  ["32", "(Log2EEW:5)"],
                  ["64", "(Log2EEW:6)"]];

class IsFloat<string type> {
  bit val = !or(!eq(type, "x"), !eq(type, "f"), !eq(type, "d"), !eq(type, "y"));
}

let SupportOverloading = false,
    MaskedPolicyScheme = NonePolicy in {
````
- **L1 EN**: Comment explains nearby logic, constraints, or intent: `riscv_vector.td - RISC-V V-ext Builtin function list`.
  **L1 CN**: 注释解释附近代码的逻辑、约束或设计意图：`riscv_vector.td - RISC-V V-ext Builtin function list`。
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the builtins for RISC-V V-extension. See:`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the builtins for RISC-V V-extension. See:`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `https://github.com/riscv/rvv-intrinsic-doc`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`https://github.com/riscv/rvv-intrinsic-doc`。
- **L12 EN**: Separator comment used for visual grouping.
  **L12 CN**: 用于视觉分组的分隔注释。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Includes TableGen file `"riscv_vector_common.td"` so later records can reuse shared definitions.
  **L15 CN**: 引入 TableGen 文件 `"riscv_vector_common.td"`，以便后续记录复用共享定义。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L17 EN**: Declares TableGen defvar record `TypeList = ["c","s","i","l","x","f","d","y"];`.
  **L17 CN**: 声明 TableGen defvar 记录 `TypeList = ["c","s","i","l","x","f","d","y"];`。
- **L18 EN**: Declares TableGen defvar record `EEWList = [["8", "`.
  **L18 CN**: 声明 TableGen defvar 记录 `EEWList = [["8", "`。
- **L19 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["16", "(Log2EEW:4)"],`.
  **L19 CN**: 继续一个多行参数列表、初始化器或聚合项：`["16", "(Log2EEW:4)"],`。
- **L20 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["32", "(Log2EEW:5)"],`.
  **L20 CN**: 继续一个多行参数列表、初始化器或聚合项：`["32", "(Log2EEW:5)"],`。
- **L21 EN**: Executes a call or declaration centered on `"`.
  **L21 CN**: 执行以 `"` 为核心的调用或声明。
- **L22 EN**: Blank line separating nearby declarations or logic blocks.
  **L22 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L23 EN**: Declares TableGen class record `IsFloat`.
  **L23 CN**: 声明 TableGen class 记录 `IsFloat`。
- **L24 EN**: Initializes variable `val` from the expression on the right-hand side.
  **L24 CN**: 使用右侧表达式初始化变量 `val`。
- **L25 EN**: Closes the current lexical scope or compound statement.
  **L25 CN**: 结束当前词法作用域或复合语句块。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SupportOverloading = false,`.
  **L27 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SupportOverloading = false,`。
- **L28 EN**: Continues the surrounding expression or declaration: `MaskedPolicyScheme = NonePolicy in {`.
  **L28 CN**: 继续构造周围的表达式或声明：`MaskedPolicyScheme = NonePolicy in {`。

### Lines 29-56

````tablegen
  class RVVVLEMaskBuiltin : RVVOutOp0Builtin<"m", "mPCUe", "c"> {
    let Name = "vlm_v";
    let IRName = "vlm";
    let HasMasked = false;
  }
}

let SupportOverloading = false,
    UnMaskedPolicyScheme = HasPassthruOperand in {
  multiclass RVVVLEBuiltin<list<string> types> {
    let Name = NAME # "_v",
        IRName = "vle",
        MaskedIRName ="vle_mask" in {
      foreach type = types in {
        def : RVVOutOp0Builtin<"v", "vPCe", type>;
        if !not(IsFloat<type>.val) then {
          def : RVVOutOp0Builtin<"Uv", "UvPCUe", type>;
        }
      }
    }
  }
}

multiclass RVVVLEFFBuiltin<list<string> types> {
  let Name = NAME # "_v",
      IRName = "vleff",
      MaskedIRName = "vleff_mask",
      SupportOverloading = false,
````
- **L29 EN**: Declares TableGen class record `RVVVLEMaskBuiltin`.
  **L29 CN**: 声明 TableGen class 记录 `RVVVLEMaskBuiltin`。
- **L30 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vlm_v";`.
  **L30 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vlm_v";`。
- **L31 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IRName = "vlm";`.
  **L31 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IRName = "vlm";`。
- **L32 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false;`.
  **L32 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false;`。
- **L33 EN**: Closes the current lexical scope or compound statement.
  **L33 CN**: 结束当前词法作用域或复合语句块。
- **L34 EN**: Closes the current lexical scope or compound statement.
  **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L36 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SupportOverloading = false,`.
  **L36 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SupportOverloading = false,`。
- **L37 EN**: Continues the surrounding expression or declaration: `UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L37 CN**: 继续构造周围的表达式或声明：`UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L38 EN**: Declares TableGen multiclass record `RVVVLEBuiltin`.
  **L38 CN**: 声明 TableGen multiclass 记录 `RVVVLEBuiltin`。
- **L39 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_v",`.
  **L39 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_v",`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vle",`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vle",`。
- **L41 EN**: Continues the surrounding expression or declaration: `MaskedIRName ="vle_mask" in {`.
  **L41 CN**: 继续构造周围的表达式或声明：`MaskedIRName ="vle_mask" in {`。
- **L42 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = types in {`.
  **L42 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = types in {`。
- **L43 EN**: Declares TableGen def record `def`.
  **L43 CN**: 声明 TableGen def 记录 `def`。
- **L44 EN**: Continues logic associated with callable symbol `not`.
  **L44 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L45 EN**: Declares TableGen def record `def`.
  **L45 CN**: 声明 TableGen def 记录 `def`。
- **L46 EN**: Closes the current lexical scope or compound statement.
  **L46 CN**: 结束当前词法作用域或复合语句块。
- **L47 EN**: Closes the current lexical scope or compound statement.
  **L47 CN**: 结束当前词法作用域或复合语句块。
- **L48 EN**: Closes the current lexical scope or compound statement.
  **L48 CN**: 结束当前词法作用域或复合语句块。
- **L49 EN**: Closes the current lexical scope or compound statement.
  **L49 CN**: 结束当前词法作用域或复合语句块。
- **L50 EN**: Closes the current lexical scope or compound statement.
  **L50 CN**: 结束当前词法作用域或复合语句块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Declares TableGen multiclass record `RVVVLEFFBuiltin`.
  **L52 CN**: 声明 TableGen multiclass 记录 `RVVVLEFFBuiltin`。
- **L53 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_v",`.
  **L53 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_v",`。
- **L54 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vleff",`.
  **L54 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vleff",`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vleff_mask",`.
  **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vleff_mask",`。
- **L56 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SupportOverloading = false,`.
  **L56 CN**: 继续一个多行参数列表、初始化器或聚合项：`SupportOverloading = false,`。

### Lines 57-84

````tablegen
      UnMaskedPolicyScheme = HasPassthruOperand,
      ManualCodegen = [{
        return emitRVVVLEFFBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                   PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
    foreach type = types in {
      def : RVVBuiltin<"v", "vPCePz", type>;
      // Skip floating types for unsigned versions.
      if !not(IsFloat<type>.val) then {
        def : RVVBuiltin<"Uv", "UvPCUePz", type>;
      }
    }
  }
}

multiclass RVVVLSEBuiltin<list<string> types> {
  let Name = NAME # "_v",
      IRName = "vlse",
      MaskedIRName ="vlse_mask",
      SupportOverloading = false,
      UnMaskedPolicyScheme = HasPassthruOperand in {
    foreach type = types in {
      def : RVVOutOp0Builtin<"v", "vPCet", type>;
      if !not(IsFloat<type>.val) then {
        def : RVVOutOp0Builtin<"Uv", "UvPCUet", type>;
      }
    }
  }
````
- **L57 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L57 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L58 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L58 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L59 EN**: Returns from the current function with `emitRVVVLEFFBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L59 CN**: 以 `emitRVVVLEFFBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L60 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L60 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L61 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L61 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L62 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = types in {`.
  **L62 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = types in {`。
- **L63 EN**: Declares TableGen def record `def`.
  **L63 CN**: 声明 TableGen def 记录 `def`。
- **L64 EN**: Comment explains nearby logic, constraints, or intent: `Skip floating types for unsigned versions.`.
  **L64 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Skip floating types for unsigned versions.`。
- **L65 EN**: Continues logic associated with callable symbol `not`.
  **L65 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L66 EN**: Declares TableGen def record `def`.
  **L66 CN**: 声明 TableGen def 记录 `def`。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Closes the current lexical scope or compound statement.
  **L68 CN**: 结束当前词法作用域或复合语句块。
- **L69 EN**: Closes the current lexical scope or compound statement.
  **L69 CN**: 结束当前词法作用域或复合语句块。
- **L70 EN**: Closes the current lexical scope or compound statement.
  **L70 CN**: 结束当前词法作用域或复合语句块。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Declares TableGen multiclass record `RVVVLSEBuiltin`.
  **L72 CN**: 声明 TableGen multiclass 记录 `RVVVLSEBuiltin`。
- **L73 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_v",`.
  **L73 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_v",`。
- **L74 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vlse",`.
  **L74 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vlse",`。
- **L75 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName ="vlse_mask",`.
  **L75 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName ="vlse_mask",`。
- **L76 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SupportOverloading = false,`.
  **L76 CN**: 继续一个多行参数列表、初始化器或聚合项：`SupportOverloading = false,`。
- **L77 EN**: Continues the surrounding expression or declaration: `UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L77 CN**: 继续构造周围的表达式或声明：`UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L78 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = types in {`.
  **L78 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = types in {`。
- **L79 EN**: Declares TableGen def record `def`.
  **L79 CN**: 声明 TableGen def 记录 `def`。
- **L80 EN**: Continues logic associated with callable symbol `not`.
  **L80 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L81 EN**: Declares TableGen def record `def`.
  **L81 CN**: 声明 TableGen def 记录 `def`。
- **L82 EN**: Closes the current lexical scope or compound statement.
  **L82 CN**: 结束当前词法作用域或复合语句块。
- **L83 EN**: Closes the current lexical scope or compound statement.
  **L83 CN**: 结束当前词法作用域或复合语句块。
- **L84 EN**: Closes the current lexical scope or compound statement.
  **L84 CN**: 结束当前词法作用域或复合语句块。

### Lines 85-112

````tablegen
}

multiclass RVVIndexedLoad<string op> {
  let UnMaskedPolicyScheme = HasPassthruOperand in {
    foreach type = TypeList in {
      foreach eew_list = EEWList[0-2] in {
        defvar eew = eew_list[0];
        defvar eew_type = eew_list[1];
        let Name = op # eew # "_v", IRName = op, MaskedIRName = op # "_mask" in {
          def: RVVOutOp0Op1Builtin<"v", "vPCe" # eew_type # "Uv", type>;
            if !not(IsFloat<type>.val) then {
              def: RVVOutOp0Op1Builtin<"Uv", "UvPCUe" # eew_type # "Uv", type>;
            }
        }
      }
      defvar eew64 = "64";
      defvar eew64_type = "(Log2EEW:6)";
      let Name = op # eew64 # "_v", IRName = op, MaskedIRName = op # "_mask",
          RequiredFeatures = ["64bit"] in {
          def: RVVOutOp0Op1Builtin<"v", "vPCe" # eew64_type # "Uv", type>;
            if !not(IsFloat<type>.val) then {
              def: RVVOutOp0Op1Builtin<"Uv", "UvPCUe" # eew64_type # "Uv", type>;
            }
        }
    }
  }
}

````
- **L85 EN**: Closes the current lexical scope or compound statement.
  **L85 CN**: 结束当前词法作用域或复合语句块。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L87 EN**: Declares TableGen multiclass record `RVVIndexedLoad`.
  **L87 CN**: 声明 TableGen multiclass 记录 `RVVIndexedLoad`。
- **L88 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L88 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L89 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L89 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L90 EN**: Starts a TableGen iteration used to generate repeated records: `foreach eew_list = EEWList[0-2] in {`.
  **L90 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach eew_list = EEWList[0-2] in {`。
- **L91 EN**: Declares TableGen defvar record `eew = eew_list[0];`.
  **L91 CN**: 声明 TableGen defvar 记录 `eew = eew_list[0];`。
- **L92 EN**: Declares TableGen defvar record `eew_type = eew_list[1];`.
  **L92 CN**: 声明 TableGen defvar 记录 `eew_type = eew_list[1];`。
- **L93 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # eew # "_v", IRName = op, MaskedIRName = op # "_mask" in {`.
  **L93 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # eew # "_v", IRName = op, MaskedIRName = op # "_mask" in {`。
- **L94 EN**: Declares TableGen def record `def`.
  **L94 CN**: 声明 TableGen def 记录 `def`。
- **L95 EN**: Continues logic associated with callable symbol `not`.
  **L95 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L96 EN**: Declares TableGen def record `def`.
  **L96 CN**: 声明 TableGen def 记录 `def`。
- **L97 EN**: Closes the current lexical scope or compound statement.
  **L97 CN**: 结束当前词法作用域或复合语句块。
- **L98 EN**: Closes the current lexical scope or compound statement.
  **L98 CN**: 结束当前词法作用域或复合语句块。
- **L99 EN**: Closes the current lexical scope or compound statement.
  **L99 CN**: 结束当前词法作用域或复合语句块。
- **L100 EN**: Declares TableGen defvar record `eew64 = "64";`.
  **L100 CN**: 声明 TableGen defvar 记录 `eew64 = "64";`。
- **L101 EN**: Declares TableGen defvar record `eew64_type = "`.
  **L101 CN**: 声明 TableGen defvar 记录 `eew64_type = "`。
- **L102 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # eew64 # "_v", IRName = op, MaskedIRName = op # "_mask",`.
  **L102 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # eew64 # "_v", IRName = op, MaskedIRName = op # "_mask",`。
- **L103 EN**: Continues the surrounding expression or declaration: `RequiredFeatures = ["64bit"] in {`.
  **L103 CN**: 继续构造周围的表达式或声明：`RequiredFeatures = ["64bit"] in {`。
- **L104 EN**: Declares TableGen def record `def`.
  **L104 CN**: 声明 TableGen def 记录 `def`。
- **L105 EN**: Continues logic associated with callable symbol `not`.
  **L105 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L106 EN**: Declares TableGen def record `def`.
  **L106 CN**: 声明 TableGen def 记录 `def`。
- **L107 EN**: Closes the current lexical scope or compound statement.
  **L107 CN**: 结束当前词法作用域或复合语句块。
- **L108 EN**: Closes the current lexical scope or compound statement.
  **L108 CN**: 结束当前词法作用域或复合语句块。
- **L109 EN**: Closes the current lexical scope or compound statement.
  **L109 CN**: 结束当前词法作用域或复合语句块。
- **L110 EN**: Closes the current lexical scope or compound statement.
  **L110 CN**: 结束当前词法作用域或复合语句块。
- **L111 EN**: Closes the current lexical scope or compound statement.
  **L111 CN**: 结束当前词法作用域或复合语句块。
- **L112 EN**: Blank line separating nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 113-140

````tablegen
let HasMaskedOffOperand = false,
    MaskedPolicyScheme = NonePolicy,
    ManualCodegen = [{
      return emitRVVVSEMaskBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                   PolicyAttrs, IsMasked, SegInstSEW);
    }] in {
  class RVVVSEMaskBuiltin : RVVBuiltin<"m", "0PUem", "c"> {
    let Name = "vsm_v";
    let IRName = "vsm";
    let HasMasked = false;
  }
  multiclass RVVVSEBuiltin<list<string> types> {
    let Name = NAME # "_v",
        IRName = "vse",
        MaskedIRName = "vse_mask" in {
      foreach type = types in {
        def : RVVBuiltin<"v", "0Pev", type>;
        if !not(IsFloat<type>.val) then {
          def : RVVBuiltin<"Uv", "0PUeUv", type>;
        }
      }
    }
  }
}

multiclass RVVVSSEBuiltin<list<string> types> {
  let Name = NAME # "_v",
      IRName = "vsse",
````
- **L113 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = false,`.
  **L113 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = false,`。
- **L114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L114 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L115 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L115 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L116 EN**: Returns from the current function with `emitRVVVSEMaskBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L116 CN**: 以 `emitRVVVSEMaskBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L117 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L117 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L118 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L118 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L119 EN**: Declares TableGen class record `RVVVSEMaskBuiltin`.
  **L119 CN**: 声明 TableGen class 记录 `RVVVSEMaskBuiltin`。
- **L120 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vsm_v";`.
  **L120 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vsm_v";`。
- **L121 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IRName = "vsm";`.
  **L121 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IRName = "vsm";`。
- **L122 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false;`.
  **L122 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false;`。
- **L123 EN**: Closes the current lexical scope or compound statement.
  **L123 CN**: 结束当前词法作用域或复合语句块。
- **L124 EN**: Declares TableGen multiclass record `RVVVSEBuiltin`.
  **L124 CN**: 声明 TableGen multiclass 记录 `RVVVSEBuiltin`。
- **L125 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_v",`.
  **L125 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_v",`。
- **L126 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vse",`.
  **L126 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vse",`。
- **L127 EN**: Continues the surrounding expression or declaration: `MaskedIRName = "vse_mask" in {`.
  **L127 CN**: 继续构造周围的表达式或声明：`MaskedIRName = "vse_mask" in {`。
- **L128 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = types in {`.
  **L128 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = types in {`。
- **L129 EN**: Declares TableGen def record `def`.
  **L129 CN**: 声明 TableGen def 记录 `def`。
- **L130 EN**: Continues logic associated with callable symbol `not`.
  **L130 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L131 EN**: Declares TableGen def record `def`.
  **L131 CN**: 声明 TableGen def 记录 `def`。
- **L132 EN**: Closes the current lexical scope or compound statement.
  **L132 CN**: 结束当前词法作用域或复合语句块。
- **L133 EN**: Closes the current lexical scope or compound statement.
  **L133 CN**: 结束当前词法作用域或复合语句块。
- **L134 EN**: Closes the current lexical scope or compound statement.
  **L134 CN**: 结束当前词法作用域或复合语句块。
- **L135 EN**: Closes the current lexical scope or compound statement.
  **L135 CN**: 结束当前词法作用域或复合语句块。
- **L136 EN**: Closes the current lexical scope or compound statement.
  **L136 CN**: 结束当前词法作用域或复合语句块。
- **L137 EN**: Blank line separating nearby declarations or logic blocks.
  **L137 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L138 EN**: Declares TableGen multiclass record `RVVVSSEBuiltin`.
  **L138 CN**: 声明 TableGen multiclass 记录 `RVVVSSEBuiltin`。
- **L139 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME # "_v",`.
  **L139 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME # "_v",`。
- **L140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vsse",`.
  **L140 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vsse",`。

### Lines 141-168

````tablegen
      MaskedIRName = "vsse_mask",
      HasMaskedOffOperand = false,
      MaskedPolicyScheme = NonePolicy,
      ManualCodegen = [{
        return emitRVVVSSEBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                  PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
    foreach type = types in {
      def : RVVBuiltin<"v", "0Petv", type>;
      if !not(IsFloat<type>.val) then {
        def : RVVBuiltin<"Uv", "0PUetUv", type>;
      }
    }
  }
}

multiclass RVVIndexedStore<string op> {
  let HasMaskedOffOperand = false,
      MaskedPolicyScheme = NonePolicy,
      ManualCodegen = [{
        return emitRVVIndexedStoreBuiltin(this, E, ReturnValue, ResultType, ID,
                                          Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
      foreach type = TypeList in {
        foreach eew_list = EEWList[0-2] in {
          defvar eew = eew_list[0];
          defvar eew_type = eew_list[1];
          let Name = op # eew  # "_v", IRName = op, MaskedIRName = op # "_mask" in {
````
- **L141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vsse_mask",`.
  **L141 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vsse_mask",`。
- **L142 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasMaskedOffOperand = false,`.
  **L142 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasMaskedOffOperand = false,`。
- **L143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L143 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L144 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L144 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L145 EN**: Returns from the current function with `emitRVVVSSEBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L145 CN**: 以 `emitRVVVSSEBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L146 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L146 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L147 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L147 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L148 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = types in {`.
  **L148 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = types in {`。
- **L149 EN**: Declares TableGen def record `def`.
  **L149 CN**: 声明 TableGen def 记录 `def`。
- **L150 EN**: Continues logic associated with callable symbol `not`.
  **L150 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L151 EN**: Declares TableGen def record `def`.
  **L151 CN**: 声明 TableGen def 记录 `def`。
- **L152 EN**: Closes the current lexical scope or compound statement.
  **L152 CN**: 结束当前词法作用域或复合语句块。
- **L153 EN**: Closes the current lexical scope or compound statement.
  **L153 CN**: 结束当前词法作用域或复合语句块。
- **L154 EN**: Closes the current lexical scope or compound statement.
  **L154 CN**: 结束当前词法作用域或复合语句块。
- **L155 EN**: Closes the current lexical scope or compound statement.
  **L155 CN**: 结束当前词法作用域或复合语句块。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Declares TableGen multiclass record `RVVIndexedStore`.
  **L157 CN**: 声明 TableGen multiclass 记录 `RVVIndexedStore`。
- **L158 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = false,`.
  **L158 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = false,`。
- **L159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L159 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L160 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L160 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L161 EN**: Returns from the current function with `emitRVVIndexedStoreBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L161 CN**: 以 `emitRVVIndexedStoreBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L162 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L162 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L163 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L163 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L164 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L164 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L165 EN**: Starts a TableGen iteration used to generate repeated records: `foreach eew_list = EEWList[0-2] in {`.
  **L165 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach eew_list = EEWList[0-2] in {`。
- **L166 EN**: Declares TableGen defvar record `eew = eew_list[0];`.
  **L166 CN**: 声明 TableGen defvar 记录 `eew = eew_list[0];`。
- **L167 EN**: Declares TableGen defvar record `eew_type = eew_list[1];`.
  **L167 CN**: 声明 TableGen defvar 记录 `eew_type = eew_list[1];`。
- **L168 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # eew  # "_v", IRName = op, MaskedIRName = op # "_mask" in {`.
  **L168 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # eew  # "_v", IRName = op, MaskedIRName = op # "_mask" in {`。

### Lines 169-196

````tablegen
            def : RVVBuiltin<"v", "0Pe" # eew_type # "Uvv", type>;
            if !not(IsFloat<type>.val) then {
              def : RVVBuiltin<"Uv", "0PUe" # eew_type # "UvUv", type>;
            }
          }
        }
        defvar eew64 = "64";
        defvar eew64_type = "(Log2EEW:6)";
        let Name = op # eew64  # "_v", IRName = op, MaskedIRName = op # "_mask",
            RequiredFeatures = ["64bit"] in {
          def : RVVBuiltin<"v", "0Pe" # eew64_type # "Uvv", type>;
          if !not(IsFloat<type>.val) then {
            def : RVVBuiltin<"Uv", "0PUe" # eew64_type # "UvUv", type>;
          }
        }
      }
  }
}

defvar NFList = [2, 3, 4, 5, 6, 7, 8];
/*
A segment load builtin has different variants.

Therefore a segment unit-stride load builtin can have 4 variants,
1. When unmasked and the policies are all specified as agnostic:
(Address0, ..., Address{NF - 1}, Ptr, VL)
2. When masked and the policies are all specified as agnostic:
(Address0, ..., Address{NF - 1}, Mask, Ptr, VL)
````
- **L169 EN**: Declares TableGen def record `def`.
  **L169 CN**: 声明 TableGen def 记录 `def`。
- **L170 EN**: Continues logic associated with callable symbol `not`.
  **L170 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L171 EN**: Declares TableGen def record `def`.
  **L171 CN**: 声明 TableGen def 记录 `def`。
- **L172 EN**: Closes the current lexical scope or compound statement.
  **L172 CN**: 结束当前词法作用域或复合语句块。
- **L173 EN**: Closes the current lexical scope or compound statement.
  **L173 CN**: 结束当前词法作用域或复合语句块。
- **L174 EN**: Closes the current lexical scope or compound statement.
  **L174 CN**: 结束当前词法作用域或复合语句块。
- **L175 EN**: Declares TableGen defvar record `eew64 = "64";`.
  **L175 CN**: 声明 TableGen defvar 记录 `eew64 = "64";`。
- **L176 EN**: Declares TableGen defvar record `eew64_type = "`.
  **L176 CN**: 声明 TableGen defvar 记录 `eew64_type = "`。
- **L177 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # eew64  # "_v", IRName = op, MaskedIRName = op # "_mask",`.
  **L177 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # eew64  # "_v", IRName = op, MaskedIRName = op # "_mask",`。
- **L178 EN**: Continues the surrounding expression or declaration: `RequiredFeatures = ["64bit"] in {`.
  **L178 CN**: 继续构造周围的表达式或声明：`RequiredFeatures = ["64bit"] in {`。
- **L179 EN**: Declares TableGen def record `def`.
  **L179 CN**: 声明 TableGen def 记录 `def`。
- **L180 EN**: Continues logic associated with callable symbol `not`.
  **L180 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L181 EN**: Declares TableGen def record `def`.
  **L181 CN**: 声明 TableGen def 记录 `def`。
- **L182 EN**: Closes the current lexical scope or compound statement.
  **L182 CN**: 结束当前词法作用域或复合语句块。
- **L183 EN**: Closes the current lexical scope or compound statement.
  **L183 CN**: 结束当前词法作用域或复合语句块。
- **L184 EN**: Closes the current lexical scope or compound statement.
  **L184 CN**: 结束当前词法作用域或复合语句块。
- **L185 EN**: Closes the current lexical scope or compound statement.
  **L185 CN**: 结束当前词法作用域或复合语句块。
- **L186 EN**: Closes the current lexical scope or compound statement.
  **L186 CN**: 结束当前词法作用域或复合语句块。
- **L187 EN**: Blank line separating nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L188 EN**: Declares TableGen defvar record `NFList = [2, 3, 4, 5, 6, 7, 8];`.
  **L188 CN**: 声明 TableGen defvar 记录 `NFList = [2, 3, 4, 5, 6, 7, 8];`。
- **L189 EN**: Separator comment used for visual grouping.
  **L189 CN**: 用于视觉分组的分隔注释。
- **L190 EN**: Continues the surrounding expression or declaration: `A segment load builtin has different variants.`.
  **L190 CN**: 继续构造周围的表达式或声明：`A segment load builtin has different variants.`。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Therefore a segment unit-stride load builtin can have 4 variants,`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`Therefore a segment unit-stride load builtin can have 4 variants,`。
- **L193 EN**: Continues the surrounding expression or declaration: `1. When unmasked and the policies are all specified as agnostic:`.
  **L193 CN**: 继续构造周围的表达式或声明：`1. When unmasked and the policies are all specified as agnostic:`。
- **L194 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Ptr, VL)`.
  **L194 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Ptr, VL)`。
- **L195 EN**: Continues the surrounding expression or declaration: `2. When masked and the policies are all specified as agnostic:`.
  **L195 CN**: 继续构造周围的表达式或声明：`2. When masked and the policies are all specified as agnostic:`。
- **L196 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Mask, Ptr, VL)`.
  **L196 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Mask, Ptr, VL)`。

### Lines 197-224

````tablegen
3. When unmasked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, VL)
4. When masked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, VL)

Other variants of segment load builtin share the same structure, but they
have their own extra parameter.

The segment unit-stride fault-only-first load builtin has a 'NewVL'
operand after the 'Ptr' operand.
1. When unmasked and the policies are all specified as agnostic:
(Address0, ..., Address{NF - 1}, Ptr, NewVL, VL)
2. When masked and the policies are all specified as agnostic:
(Address0, ..., Address{NF - 1}, Mask, Ptr, NewVL, VL)
3. When unmasked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, NewVL, VL)
4. When masked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, NewVL, VL)

The segment strided load builtin has a 'Stride' operand after the 'Ptr'
operand.
1. When unmasked and the policies are all specified as agnostic:
(Address0, ..., Address{NF - 1}, Ptr, Stride, VL)
2. When masked and the policies are all specified as agnostic:
````
- **L197 EN**: Continues the surrounding expression or declaration: `3. When unmasked and one of the policies is specified as undisturbed:`.
  **L197 CN**: 继续构造周围的表达式或声明：`3. When unmasked and one of the policies is specified as undisturbed:`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L199 EN**: Continues the surrounding expression or declaration: `Ptr, VL)`.
  **L199 CN**: 继续构造周围的表达式或声明：`Ptr, VL)`。
- **L200 EN**: Continues the surrounding expression or declaration: `4. When masked and one of the policies is specified as undisturbed:`.
  **L200 CN**: 继续构造周围的表达式或声明：`4. When masked and one of the policies is specified as undisturbed:`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L202 EN**: Continues the surrounding expression or declaration: `Ptr, VL)`.
  **L202 CN**: 继续构造周围的表达式或声明：`Ptr, VL)`。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L204 EN**: Continues the surrounding expression or declaration: `Other variants of segment load builtin share the same structure, but they`.
  **L204 CN**: 继续构造周围的表达式或声明：`Other variants of segment load builtin share the same structure, but they`。
- **L205 EN**: Continues the surrounding expression or declaration: `have their own extra parameter.`.
  **L205 CN**: 继续构造周围的表达式或声明：`have their own extra parameter.`。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Continues the surrounding expression or declaration: `The segment unit-stride fault-only-first load builtin has a 'NewVL'`.
  **L207 CN**: 继续构造周围的表达式或声明：`The segment unit-stride fault-only-first load builtin has a 'NewVL'`。
- **L208 EN**: Continues the surrounding expression or declaration: `operand after the 'Ptr' operand.`.
  **L208 CN**: 继续构造周围的表达式或声明：`operand after the 'Ptr' operand.`。
- **L209 EN**: Continues the surrounding expression or declaration: `1. When unmasked and the policies are all specified as agnostic:`.
  **L209 CN**: 继续构造周围的表达式或声明：`1. When unmasked and the policies are all specified as agnostic:`。
- **L210 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Ptr, NewVL, VL)`.
  **L210 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Ptr, NewVL, VL)`。
- **L211 EN**: Continues the surrounding expression or declaration: `2. When masked and the policies are all specified as agnostic:`.
  **L211 CN**: 继续构造周围的表达式或声明：`2. When masked and the policies are all specified as agnostic:`。
- **L212 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Mask, Ptr, NewVL, VL)`.
  **L212 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Mask, Ptr, NewVL, VL)`。
- **L213 EN**: Continues the surrounding expression or declaration: `3. When unmasked and one of the policies is specified as undisturbed:`.
  **L213 CN**: 继续构造周围的表达式或声明：`3. When unmasked and one of the policies is specified as undisturbed:`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L215 EN**: Continues the surrounding expression or declaration: `Ptr, NewVL, VL)`.
  **L215 CN**: 继续构造周围的表达式或声明：`Ptr, NewVL, VL)`。
- **L216 EN**: Continues the surrounding expression or declaration: `4. When masked and one of the policies is specified as undisturbed:`.
  **L216 CN**: 继续构造周围的表达式或声明：`4. When masked and one of the policies is specified as undisturbed:`。
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L218 EN**: Continues the surrounding expression or declaration: `Ptr, NewVL, VL)`.
  **L218 CN**: 继续构造周围的表达式或声明：`Ptr, NewVL, VL)`。
- **L219 EN**: Blank line separating nearby declarations or logic blocks.
  **L219 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L220 EN**: Continues the surrounding expression or declaration: `The segment strided load builtin has a 'Stride' operand after the 'Ptr'`.
  **L220 CN**: 继续构造周围的表达式或声明：`The segment strided load builtin has a 'Stride' operand after the 'Ptr'`。
- **L221 EN**: Continues the surrounding expression or declaration: `operand.`.
  **L221 CN**: 继续构造周围的表达式或声明：`operand.`。
- **L222 EN**: Continues the surrounding expression or declaration: `1. When unmasked and the policies are all specified as agnostic:`.
  **L222 CN**: 继续构造周围的表达式或声明：`1. When unmasked and the policies are all specified as agnostic:`。
- **L223 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Ptr, Stride, VL)`.
  **L223 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Ptr, Stride, VL)`。
- **L224 EN**: Continues the surrounding expression or declaration: `2. When masked and the policies are all specified as agnostic:`.
  **L224 CN**: 继续构造周围的表达式或声明：`2. When masked and the policies are all specified as agnostic:`。

### Lines 225-252

````tablegen
(Address0, ..., Address{NF - 1}, Mask, Ptr, Stride, VL)
3. When unmasked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, Stride, VL)
4. When masked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, Stride, VL)

The segment indexed load builtin has a 'Idx' operand after the 'Ptr' operand.
1. When unmasked and the policies are all specified as agnostic:
(Address0, ..., Address{NF - 1}, Ptr, Idx, VL)
2. When masked and the policies are all specified as agnostic:
(Address0, ..., Address{NF - 1}, Mask, Ptr, Idx, VL)
3. When unmasked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, Idx, VL)
4. When masked and one of the policies is specified as undisturbed:
(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},
  Ptr, Idx, VL)

Segment load intrinsics has different variants similar to their builtins.

Segment unit-stride load intrinsic,
  Masked: (Vector0, ..., Vector{NF - 1}, Ptr, Mask, VL, Policy)
  Unmasked: (Vector0, ..., Vector{NF - 1}, Ptr, VL)
Segment unit-stride fault-only-first load intrinsic,
  Masked: (Vector0, ..., Vector{NF - 1}, Ptr, Mask, VL, Policy)
  Unmasked: (Vector0, ..., Vector{NF - 1}, Ptr, VL)
````
- **L225 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Mask, Ptr, Stride, VL)`.
  **L225 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Mask, Ptr, Stride, VL)`。
- **L226 EN**: Continues the surrounding expression or declaration: `3. When unmasked and one of the policies is specified as undisturbed:`.
  **L226 CN**: 继续构造周围的表达式或声明：`3. When unmasked and one of the policies is specified as undisturbed:`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L228 EN**: Continues the surrounding expression or declaration: `Ptr, Stride, VL)`.
  **L228 CN**: 继续构造周围的表达式或声明：`Ptr, Stride, VL)`。
- **L229 EN**: Continues the surrounding expression or declaration: `4. When masked and one of the policies is specified as undisturbed:`.
  **L229 CN**: 继续构造周围的表达式或声明：`4. When masked and one of the policies is specified as undisturbed:`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L231 EN**: Continues the surrounding expression or declaration: `Ptr, Stride, VL)`.
  **L231 CN**: 继续构造周围的表达式或声明：`Ptr, Stride, VL)`。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Continues the surrounding expression or declaration: `The segment indexed load builtin has a 'Idx' operand after the 'Ptr' operand.`.
  **L233 CN**: 继续构造周围的表达式或声明：`The segment indexed load builtin has a 'Idx' operand after the 'Ptr' operand.`。
- **L234 EN**: Continues the surrounding expression or declaration: `1. When unmasked and the policies are all specified as agnostic:`.
  **L234 CN**: 继续构造周围的表达式或声明：`1. When unmasked and the policies are all specified as agnostic:`。
- **L235 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Ptr, Idx, VL)`.
  **L235 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Ptr, Idx, VL)`。
- **L236 EN**: Continues the surrounding expression or declaration: `2. When masked and the policies are all specified as agnostic:`.
  **L236 CN**: 继续构造周围的表达式或声明：`2. When masked and the policies are all specified as agnostic:`。
- **L237 EN**: Continues the surrounding expression or declaration: `(Address0, ..., Address{NF - 1}, Mask, Ptr, Idx, VL)`.
  **L237 CN**: 继续构造周围的表达式或声明：`(Address0, ..., Address{NF - 1}, Mask, Ptr, Idx, VL)`。
- **L238 EN**: Continues the surrounding expression or declaration: `3. When unmasked and one of the policies is specified as undisturbed:`.
  **L238 CN**: 继续构造周围的表达式或声明：`3. When unmasked and one of the policies is specified as undisturbed:`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L240 EN**: Continues the surrounding expression or declaration: `Ptr, Idx, VL)`.
  **L240 CN**: 继续构造周围的表达式或声明：`Ptr, Idx, VL)`。
- **L241 EN**: Continues the surrounding expression or declaration: `4. When masked and one of the policies is specified as undisturbed:`.
  **L241 CN**: 继续构造周围的表达式或声明：`4. When masked and one of the policies is specified as undisturbed:`。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Address0, ..., Address{NF - 1}, Mask, Maskedoff0, ..., Maskedoff{NF - 1},`。
- **L243 EN**: Continues the surrounding expression or declaration: `Ptr, Idx, VL)`.
  **L243 CN**: 继续构造周围的表达式或声明：`Ptr, Idx, VL)`。
- **L244 EN**: Blank line separating nearby declarations or logic blocks.
  **L244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L245 EN**: Continues the surrounding expression or declaration: `Segment load intrinsics has different variants similar to their builtins.`.
  **L245 CN**: 继续构造周围的表达式或声明：`Segment load intrinsics has different variants similar to their builtins.`。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Segment unit-stride load intrinsic,`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`Segment unit-stride load intrinsic,`。
- **L248 EN**: Continues logic associated with callable symbol `Masked:`.
  **L248 CN**: 继续与可调用符号 `Masked:` 相关的逻辑。
- **L249 EN**: Continues logic associated with callable symbol `Unmasked:`.
  **L249 CN**: 继续与可调用符号 `Unmasked:` 相关的逻辑。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Segment unit-stride fault-only-first load intrinsic,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`Segment unit-stride fault-only-first load intrinsic,`。
- **L251 EN**: Continues logic associated with callable symbol `Masked:`.
  **L251 CN**: 继续与可调用符号 `Masked:` 相关的逻辑。
- **L252 EN**: Continues logic associated with callable symbol `Unmasked:`.
  **L252 CN**: 继续与可调用符号 `Unmasked:` 相关的逻辑。

### Lines 253-280

````tablegen
Segment strided load intrinsic,
  Masked: (Vector0, ..., Vector{NF - 1}, Ptr, Stride, Mask, VL, Policy)
  Unmasked: (Vector0, ..., Vector{NF - 1}, Ptr, Stride, VL)
Segment indexed load intrinsic,
  Masked: (Vector0, ..., Vector{NF - 1}, Ptr, Index, Mask, VL, Policy)
  Unmasked: (Vector0, ..., Vector{NF - 1}, Ptr, Index, VL)

The Vector(s) is poison when the policy behavior allows us to not care
about any masked-off elements.
*/

class PVString<int nf, bit signed> {
  string S =
    !cond(!eq(nf, 2): !if(signed, "PvPv", "PUvPUv"),
          !eq(nf, 3): !if(signed, "PvPvPv", "PUvPUvPUv"),
          !eq(nf, 4): !if(signed, "PvPvPvPv", "PUvPUvPUvPUv"),
          !eq(nf, 5): !if(signed, "PvPvPvPvPv", "PUvPUvPUvPUvPUv"),
          !eq(nf, 6): !if(signed, "PvPvPvPvPvPv", "PUvPUvPUvPUvPUvPUv"),
          !eq(nf, 7): !if(signed, "PvPvPvPvPvPvPv", "PUvPUvPUvPUvPUvPUvPUv"),
          !eq(nf, 8): !if(signed, "PvPvPvPvPvPvPvPv", "PUvPUvPUvPUvPUvPUvPUvPUv"));
}

class VString<int nf, bit signed> {
  string S = !cond(!eq(nf, 2): !if(signed, "vv", "UvUv"),
                   !eq(nf, 3): !if(signed, "vvv", "UvUvUv"),
                   !eq(nf, 4): !if(signed, "vvvv", "UvUvUvUv"),
                   !eq(nf, 5): !if(signed, "vvvvv", "UvUvUvUvUv"),
                   !eq(nf, 6): !if(signed, "vvvvvv", "UvUvUvUvUvUv"),
````
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Segment strided load intrinsic,`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`Segment strided load intrinsic,`。
- **L254 EN**: Continues logic associated with callable symbol `Masked:`.
  **L254 CN**: 继续与可调用符号 `Masked:` 相关的逻辑。
- **L255 EN**: Continues logic associated with callable symbol `Unmasked:`.
  **L255 CN**: 继续与可调用符号 `Unmasked:` 相关的逻辑。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Segment indexed load intrinsic,`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`Segment indexed load intrinsic,`。
- **L257 EN**: Continues logic associated with callable symbol `Masked:`.
  **L257 CN**: 继续与可调用符号 `Masked:` 相关的逻辑。
- **L258 EN**: Continues logic associated with callable symbol `Unmasked:`.
  **L258 CN**: 继续与可调用符号 `Unmasked:` 相关的逻辑。
- **L259 EN**: Blank line separating nearby declarations or logic blocks.
  **L259 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L260 EN**: Continues logic associated with callable symbol `Vector`.
  **L260 CN**: 继续与可调用符号 `Vector` 相关的逻辑。
- **L261 EN**: Continues the surrounding expression or declaration: `about any masked-off elements.`.
  **L261 CN**: 继续构造周围的表达式或声明：`about any masked-off elements.`。
- **L262 EN**: Separator comment used for visual grouping.
  **L262 CN**: 用于视觉分组的分隔注释。
- **L263 EN**: Blank line separating nearby declarations or logic blocks.
  **L263 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L264 EN**: Declares TableGen class record `PVString`.
  **L264 CN**: 声明 TableGen class 记录 `PVString`。
- **L265 EN**: Continues the surrounding expression or declaration: `string S =`.
  **L265 CN**: 继续构造周围的表达式或声明：`string S =`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!cond(!eq(nf, 2): !if(signed, "PvPv", "PUvPUv"),`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`!cond(!eq(nf, 2): !if(signed, "PvPv", "PUvPUv"),`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 3): !if(signed, "PvPvPv", "PUvPUvPUv"),`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 3): !if(signed, "PvPvPv", "PUvPUvPUv"),`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 4): !if(signed, "PvPvPvPv", "PUvPUvPUvPUv"),`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 4): !if(signed, "PvPvPvPv", "PUvPUvPUvPUv"),`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 5): !if(signed, "PvPvPvPvPv", "PUvPUvPUvPUvPUv"),`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 5): !if(signed, "PvPvPvPvPv", "PUvPUvPUvPUvPUv"),`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 6): !if(signed, "PvPvPvPvPvPv", "PUvPUvPUvPUvPUvPUv"),`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 6): !if(signed, "PvPvPvPvPvPv", "PUvPUvPUvPUvPUvPUv"),`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 7): !if(signed, "PvPvPvPvPvPvPv", "PUvPUvPUvPUvPUvPUvPUv"),`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 7): !if(signed, "PvPvPvPvPvPvPv", "PUvPUvPUvPUvPUvPUvPUv"),`。
- **L272 EN**: Executes a call or declaration centered on `!eq`.
  **L272 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L273 EN**: Closes the current lexical scope or compound statement.
  **L273 CN**: 结束当前词法作用域或复合语句块。
- **L274 EN**: Blank line separating nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L275 EN**: Declares TableGen class record `VString`.
  **L275 CN**: 声明 TableGen class 记录 `VString`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `string S = !cond(!eq(nf, 2): !if(signed, "vv", "UvUv"),`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`string S = !cond(!eq(nf, 2): !if(signed, "vv", "UvUv"),`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 3): !if(signed, "vvv", "UvUvUv"),`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 3): !if(signed, "vvv", "UvUvUv"),`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 4): !if(signed, "vvvv", "UvUvUvUv"),`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 4): !if(signed, "vvvv", "UvUvUvUv"),`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 5): !if(signed, "vvvvv", "UvUvUvUvUv"),`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 5): !if(signed, "vvvvv", "UvUvUvUvUv"),`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 6): !if(signed, "vvvvvv", "UvUvUvUvUvUv"),`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 6): !if(signed, "vvvvvv", "UvUvUvUvUvUv"),`。

### Lines 281-308

````tablegen
                   !eq(nf, 7): !if(signed, "vvvvvvv", "UvUvUvUvUvUvUv"),
                   !eq(nf, 8): !if(signed, "vvvvvvvv", "UvUvUvUvUvUvUvUv"));
}


class FixedVString<int fixed_lmul, int num, string vec> {
  string V = "(LFixedLog2LMUL:" # fixed_lmul # ")" # vec;
  string S = !interleave(!listsplat(V, num), "");
}

multiclass RVVNonTupleVCreateBuiltin<int dst_lmul, list<int> src_lmul_list> {
  defvar dst_v = FixedVString<dst_lmul, 1, "v">.V;
  defvar dst_uv = FixedVString<dst_lmul, 1, "Uv">.V;
  foreach src_lmul = src_lmul_list in {
    defvar num = !shl(1, !sub(dst_lmul, src_lmul));

    defvar src_v = FixedVString<src_lmul, num, "v">.V;
    defvar src_s = FixedVString<src_lmul, num, "v">.S;
    def vcreate # src_v # dst_v : RVVBuiltin<src_v # dst_v,
                                             dst_v # src_s,
                                             "csilxfdy">;

    defvar src_uv = FixedVString<src_lmul, num, "Uv">.V;
    defvar src_us = FixedVString<src_lmul, num, "Uv">.S;
    def vcreate_u # src_uv # dst_uv : RVVBuiltin<src_uv # dst_uv,
                                                 dst_uv # src_us,
                                                 "csil">;
  }
````
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(nf, 7): !if(signed, "vvvvvvv", "UvUvUvUvUvUvUv"),`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(nf, 7): !if(signed, "vvvvvvv", "UvUvUvUvUvUvUv"),`。
- **L282 EN**: Executes a call or declaration centered on `!eq`.
  **L282 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L283 EN**: Closes the current lexical scope or compound statement.
  **L283 CN**: 结束当前词法作用域或复合语句块。
- **L284 EN**: Blank line separating nearby declarations or logic blocks.
  **L284 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L286 EN**: Declares TableGen class record `FixedVString`.
  **L286 CN**: 声明 TableGen class 记录 `FixedVString`。
- **L287 EN**: Initializes variable `V` from the expression on the right-hand side.
  **L287 CN**: 使用右侧表达式初始化变量 `V`。
- **L288 EN**: Initializes variable `S` from the expression on the right-hand side.
  **L288 CN**: 使用右侧表达式初始化变量 `S`。
- **L289 EN**: Closes the current lexical scope or compound statement.
  **L289 CN**: 结束当前词法作用域或复合语句块。
- **L290 EN**: Blank line separating nearby declarations or logic blocks.
  **L290 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L291 EN**: Declares TableGen multiclass record `RVVNonTupleVCreateBuiltin`.
  **L291 CN**: 声明 TableGen multiclass 记录 `RVVNonTupleVCreateBuiltin`。
- **L292 EN**: Declares TableGen defvar record `dst_v = FixedVString`.
  **L292 CN**: 声明 TableGen defvar 记录 `dst_v = FixedVString`。
- **L293 EN**: Declares TableGen defvar record `dst_uv = FixedVString`.
  **L293 CN**: 声明 TableGen defvar 记录 `dst_uv = FixedVString`。
- **L294 EN**: Starts a TableGen iteration used to generate repeated records: `foreach src_lmul = src_lmul_list in {`.
  **L294 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach src_lmul = src_lmul_list in {`。
- **L295 EN**: Declares TableGen defvar record `num = !shl`.
  **L295 CN**: 声明 TableGen defvar 记录 `num = !shl`。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Declares TableGen defvar record `src_v = FixedVString`.
  **L297 CN**: 声明 TableGen defvar 记录 `src_v = FixedVString`。
- **L298 EN**: Declares TableGen defvar record `src_s = FixedVString`.
  **L298 CN**: 声明 TableGen defvar 记录 `src_s = FixedVString`。
- **L299 EN**: Declares TableGen def record `vcreate # src_v # dst_v`.
  **L299 CN**: 声明 TableGen def 记录 `vcreate # src_v # dst_v`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dst_v # src_s,`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`dst_v # src_s,`。
- **L301 EN**: Adds a standalone statement or declaration: `"csilxfdy">;`.
  **L301 CN**: 添加一条独立语句或声明：`"csilxfdy">;`。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Declares TableGen defvar record `src_uv = FixedVString`.
  **L303 CN**: 声明 TableGen defvar 记录 `src_uv = FixedVString`。
- **L304 EN**: Declares TableGen defvar record `src_us = FixedVString`.
  **L304 CN**: 声明 TableGen defvar 记录 `src_us = FixedVString`。
- **L305 EN**: Declares TableGen def record `vcreate_u # src_uv # dst_uv`.
  **L305 CN**: 声明 TableGen def 记录 `vcreate_u # src_uv # dst_uv`。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dst_uv # src_us,`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`dst_uv # src_us,`。
- **L307 EN**: Adds a standalone statement or declaration: `"csil">;`.
  **L307 CN**: 添加一条独立语句或声明：`"csil">;`。
- **L308 EN**: Closes the current lexical scope or compound statement.
  **L308 CN**: 结束当前词法作用域或复合语句块。

### Lines 309-336

````tablegen
}

multiclass RVVPseudoUnaryBuiltin<string IR, string type_range> {
  let Name = NAME,
      IRName = IR,
      MaskedIRName = IR # "_mask",
      UnMaskedPolicyScheme = HasPassthruOperand,
      ManualCodegen = [{
        return emitRVVPseudoUnaryBuiltin(this, E, ReturnValue, ResultType, ID,
                                         Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
        def : RVVBuiltin<"v", "vv", type_range>;
  }
}

multiclass RVVPseudoVNotBuiltin<string IR, string type_range> {
  let Name = NAME,
      IRName = IR,
      MaskedIRName = IR # "_mask",
      UnMaskedPolicyScheme = HasPassthruOperand,
      ManualCodegen = [{
        return emitRVVPseudoVNotBuiltin(this, E, ReturnValue, ResultType, ID,
                                        Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
        def : RVVBuiltin<"v", "vv", type_range>;
        def : RVVBuiltin<"Uv", "UvUv", type_range>;
  }
}
````
- **L309 EN**: Closes the current lexical scope or compound statement.
  **L309 CN**: 结束当前词法作用域或复合语句块。
- **L310 EN**: Blank line separating nearby declarations or logic blocks.
  **L310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L311 EN**: Declares TableGen multiclass record `RVVPseudoUnaryBuiltin`.
  **L311 CN**: 声明 TableGen multiclass 记录 `RVVPseudoUnaryBuiltin`。
- **L312 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME,`.
  **L312 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME,`。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = IR,`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = IR,`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = IR # "_mask",`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = IR # "_mask",`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L316 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L316 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L317 EN**: Returns from the current function with `emitRVVPseudoUnaryBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L317 CN**: 以 `emitRVVPseudoUnaryBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L318 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L318 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L319 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L319 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L320 EN**: Declares TableGen def record `def`.
  **L320 CN**: 声明 TableGen def 记录 `def`。
- **L321 EN**: Closes the current lexical scope or compound statement.
  **L321 CN**: 结束当前词法作用域或复合语句块。
- **L322 EN**: Closes the current lexical scope or compound statement.
  **L322 CN**: 结束当前词法作用域或复合语句块。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Declares TableGen multiclass record `RVVPseudoVNotBuiltin`.
  **L324 CN**: 声明 TableGen multiclass 记录 `RVVPseudoVNotBuiltin`。
- **L325 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME,`.
  **L325 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME,`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = IR,`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = IR,`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = IR # "_mask",`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = IR # "_mask",`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L329 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L329 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L330 EN**: Returns from the current function with `emitRVVPseudoVNotBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L330 CN**: 以 `emitRVVPseudoVNotBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L331 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L331 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L332 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L332 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L333 EN**: Declares TableGen def record `def`.
  **L333 CN**: 声明 TableGen def 记录 `def`。
- **L334 EN**: Declares TableGen def record `def`.
  **L334 CN**: 声明 TableGen def 记录 `def`。
- **L335 EN**: Closes the current lexical scope or compound statement.
  **L335 CN**: 结束当前词法作用域或复合语句块。
- **L336 EN**: Closes the current lexical scope or compound statement.
  **L336 CN**: 结束当前词法作用域或复合语句块。

### Lines 337-364

````tablegen

multiclass RVVPseudoMaskBuiltin<string IR, string type_range> {
  let Name = NAME,
      IRName = IR,
      HasMasked = false,
      ManualCodegen = [{
        return emitRVVPseudoMaskBuiltin(this, E, ReturnValue, ResultType, ID,
                                        Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
        def : RVVBuiltin<"m", "mm", type_range>;
  }
}

multiclass RVVPseudoVFUnaryBuiltin<string IR, string type_range> {
  let Name = NAME,
      IRName = IR,
      MaskedIRName = IR # "_mask",
      UnMaskedPolicyScheme = HasPassthruOperand,
      ManualCodegen = [{
        return emitRVVPseudoVFUnaryBuiltin(this, E, ReturnValue, ResultType, ID,
                                           Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
        def : RVVBuiltin<"v", "vv", type_range>;
  }
}

multiclass RVVPseudoVWCVTBuiltin<string IR, string MName, string type_range,
                                 list<list<string>> suffixes_prototypes> {
````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L338 EN**: Declares TableGen multiclass record `RVVPseudoMaskBuiltin`.
  **L338 CN**: 声明 TableGen multiclass 记录 `RVVPseudoMaskBuiltin`。
- **L339 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME,`.
  **L339 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME,`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = IR,`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = IR,`。
- **L341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasMasked = false,`.
  **L341 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasMasked = false,`。
- **L342 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L342 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L343 EN**: Returns from the current function with `emitRVVPseudoMaskBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L343 CN**: 以 `emitRVVPseudoMaskBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L344 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L344 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L345 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L345 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L346 EN**: Declares TableGen def record `def`.
  **L346 CN**: 声明 TableGen def 记录 `def`。
- **L347 EN**: Closes the current lexical scope or compound statement.
  **L347 CN**: 结束当前词法作用域或复合语句块。
- **L348 EN**: Closes the current lexical scope or compound statement.
  **L348 CN**: 结束当前词法作用域或复合语句块。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L350 EN**: Declares TableGen multiclass record `RVVPseudoVFUnaryBuiltin`.
  **L350 CN**: 声明 TableGen multiclass 记录 `RVVPseudoVFUnaryBuiltin`。
- **L351 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME,`.
  **L351 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME,`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = IR,`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = IR,`。
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = IR # "_mask",`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = IR # "_mask",`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L355 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L355 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L356 EN**: Returns from the current function with `emitRVVPseudoVFUnaryBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L356 CN**: 以 `emitRVVPseudoVFUnaryBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L357 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L357 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L358 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L358 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L359 EN**: Declares TableGen def record `def`.
  **L359 CN**: 声明 TableGen def 记录 `def`。
- **L360 EN**: Closes the current lexical scope or compound statement.
  **L360 CN**: 结束当前词法作用域或复合语句块。
- **L361 EN**: Closes the current lexical scope or compound statement.
  **L361 CN**: 结束当前词法作用域或复合语句块。
- **L362 EN**: Blank line separating nearby declarations or logic blocks.
  **L362 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L363 EN**: Declares TableGen multiclass record `RVVPseudoVWCVTBuiltin`.
  **L363 CN**: 声明 TableGen multiclass 记录 `RVVPseudoVWCVTBuiltin`。
- **L364 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes> {`.
  **L364 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes> {`。

### Lines 365-392

````tablegen
  let Name = NAME,
      OverloadedName = MName,
      IRName = IR,
      MaskedIRName = IR # "_mask",
      UnMaskedPolicyScheme = HasPassthruOperand,
      ManualCodegen = [{
        return emitRVVPseudoVWCVTBuiltin(this, E, ReturnValue, ResultType, ID,
                                         Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
        foreach s_p = suffixes_prototypes in {
          def : RVVBuiltin<s_p[0], s_p[1], type_range>;
        }
  }
}

multiclass RVVPseudoVNCVTBuiltin<string IR, string MName, string type_range,
                                 list<list<string>> suffixes_prototypes> {
  let Name = NAME,
      OverloadedName = MName,
      IRName = IR,
      MaskedIRName = IR # "_mask",
      UnMaskedPolicyScheme = HasPassthruOperand,
      ManualCodegen = [{
        return emitRVVPseudoVNCVTBuiltin(this, E, ReturnValue, ResultType, ID,
                                         Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
        foreach s_p = suffixes_prototypes in {
          def : RVVBuiltin<s_p[0], s_p[1], type_range>;
````
- **L365 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME,`.
  **L365 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME,`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OverloadedName = MName,`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`OverloadedName = MName,`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = IR,`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = IR,`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = IR # "_mask",`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = IR # "_mask",`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L370 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L370 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L371 EN**: Returns from the current function with `emitRVVPseudoVWCVTBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L371 CN**: 以 `emitRVVPseudoVWCVTBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L372 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L372 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L373 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L373 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L374 EN**: Starts a TableGen iteration used to generate repeated records: `foreach s_p = suffixes_prototypes in {`.
  **L374 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach s_p = suffixes_prototypes in {`。
- **L375 EN**: Declares TableGen def record `def`.
  **L375 CN**: 声明 TableGen def 记录 `def`。
- **L376 EN**: Closes the current lexical scope or compound statement.
  **L376 CN**: 结束当前词法作用域或复合语句块。
- **L377 EN**: Closes the current lexical scope or compound statement.
  **L377 CN**: 结束当前词法作用域或复合语句块。
- **L378 EN**: Closes the current lexical scope or compound statement.
  **L378 CN**: 结束当前词法作用域或复合语句块。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L380 EN**: Declares TableGen multiclass record `RVVPseudoVNCVTBuiltin`.
  **L380 CN**: 声明 TableGen multiclass 记录 `RVVPseudoVNCVTBuiltin`。
- **L381 EN**: Continues the surrounding expression or declaration: `list<list<string>> suffixes_prototypes> {`.
  **L381 CN**: 继续构造周围的表达式或声明：`list<list<string>> suffixes_prototypes> {`。
- **L382 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = NAME,`.
  **L382 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = NAME,`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OverloadedName = MName,`.
  **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`OverloadedName = MName,`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = IR,`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = IR,`。
- **L385 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = IR # "_mask",`.
  **L385 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = IR # "_mask",`。
- **L386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L386 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L387 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L387 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L388 EN**: Returns from the current function with `emitRVVPseudoVNCVTBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L388 CN**: 以 `emitRVVPseudoVNCVTBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L389 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L389 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L390 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L390 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L391 EN**: Starts a TableGen iteration used to generate repeated records: `foreach s_p = suffixes_prototypes in {`.
  **L391 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach s_p = suffixes_prototypes in {`。
- **L392 EN**: Declares TableGen def record `def`.
  **L392 CN**: 声明 TableGen def 记录 `def`。

### Lines 393-420

````tablegen
        }
  }
}

let HeaderCode =
[{
#define __riscv_vlenb() __builtin_rvv_vlenb()
}] in
def vlenb_macro: RVVHeader;

let HasBuiltinAlias = false, HasVL = false, HasMasked = false,
    UnMaskedPolicyScheme = NonePolicy, MaskedPolicyScheme = NonePolicy,
    Log2LMUL = [0], IRName = "",
    ManualCodegen = [{
      return emitRVVVlenbBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                 PolicyAttrs, IsMasked, SegInstSEW);
    }] in
{
  def vlenb : RVVBuiltin<"", "u", "i">;
}

// 6. Configuration-Setting Instructions
// 6.1. vsetvli/vsetvl instructions

// vsetvl/vsetvlmax are a macro because they require constant integers in SEW
// and LMUL.
let HeaderCode =
[{
````
- **L393 EN**: Closes the current lexical scope or compound statement.
  **L393 CN**: 结束当前词法作用域或复合语句块。
- **L394 EN**: Closes the current lexical scope or compound statement.
  **L394 CN**: 结束当前词法作用域或复合语句块。
- **L395 EN**: Closes the current lexical scope or compound statement.
  **L395 CN**: 结束当前词法作用域或复合语句块。
- **L396 EN**: Blank line separating nearby declarations or logic blocks.
  **L396 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L397 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HeaderCode =`.
  **L397 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HeaderCode =`。
- **L398 EN**: Continues the surrounding expression or declaration: `[{`.
  **L398 CN**: 继续构造周围的表达式或声明：`[{`。
- **L399 EN**: Defines macro `__riscv_vlenb()` for conditional compilation, shorthand, or table-driven expansion.
  **L399 CN**: 定义宏 `__riscv_vlenb()`，用于条件编译、简写或表驱动展开。
- **L400 EN**: Continues the surrounding expression or declaration: `}] in`.
  **L400 CN**: 继续构造周围的表达式或声明：`}] in`。
- **L401 EN**: Declares TableGen def record `vlenb_macro`.
  **L401 CN**: 声明 TableGen def 记录 `vlenb_macro`。
- **L402 EN**: Blank line separating nearby declarations or logic blocks.
  **L402 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L403 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasBuiltinAlias = false, HasVL = false, HasMasked = false,`.
  **L403 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasBuiltinAlias = false, HasVL = false, HasMasked = false,`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = NonePolicy, MaskedPolicyScheme = NonePolicy,`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = NonePolicy, MaskedPolicyScheme = NonePolicy,`。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Log2LMUL = [0], IRName = "",`.
  **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`Log2LMUL = [0], IRName = "",`。
- **L406 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L406 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L407 EN**: Returns from the current function with `emitRVVVlenbBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L407 CN**: 以 `emitRVVVlenbBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L408 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L408 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L409 EN**: Continues the surrounding expression or declaration: `}] in`.
  **L409 CN**: 继续构造周围的表达式或声明：`}] in`。
- **L410 EN**: Opens a new lexical scope or compound statement.
  **L410 CN**: 打开一个新的词法作用域或复合语句块。
- **L411 EN**: Declares TableGen def record `vlenb`.
  **L411 CN**: 声明 TableGen def 记录 `vlenb`。
- **L412 EN**: Closes the current lexical scope or compound statement.
  **L412 CN**: 结束当前词法作用域或复合语句块。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L414 EN**: Comment explains nearby logic, constraints, or intent: `6. Configuration-Setting Instructions`.
  **L414 CN**: 注释解释附近代码的逻辑、约束或设计意图：`6. Configuration-Setting Instructions`。
- **L415 EN**: Comment explains nearby logic, constraints, or intent: `6.1. vsetvli/vsetvl instructions`.
  **L415 CN**: 注释解释附近代码的逻辑、约束或设计意图：`6.1. vsetvli/vsetvl instructions`。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L417 EN**: Comment explains nearby logic, constraints, or intent: `vsetvl/vsetvlmax are a macro because they require constant integers in SEW`.
  **L417 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vsetvl/vsetvlmax are a macro because they require constant integers in SEW`。
- **L418 EN**: Comment explains nearby logic, constraints, or intent: `and LMUL.`.
  **L418 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and LMUL.`。
- **L419 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HeaderCode =`.
  **L419 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HeaderCode =`。
- **L420 EN**: Continues the surrounding expression or declaration: `[{`.
  **L420 CN**: 继续构造周围的表达式或声明：`[{`。

### Lines 421-448

````tablegen
#define __riscv_vsetvl_e8mf4(avl) __builtin_rvv_vsetvli((size_t)(avl), 0, 6)
#define __riscv_vsetvl_e8mf2(avl) __builtin_rvv_vsetvli((size_t)(avl), 0, 7)
#define __riscv_vsetvl_e8m1(avl) __builtin_rvv_vsetvli((size_t)(avl), 0, 0)
#define __riscv_vsetvl_e8m2(avl) __builtin_rvv_vsetvli((size_t)(avl), 0, 1)
#define __riscv_vsetvl_e8m4(avl) __builtin_rvv_vsetvli((size_t)(avl), 0, 2)
#define __riscv_vsetvl_e8m8(avl) __builtin_rvv_vsetvli((size_t)(avl), 0, 3)

#define __riscv_vsetvl_e16mf2(avl) __builtin_rvv_vsetvli((size_t)(avl), 1, 7)
#define __riscv_vsetvl_e16m1(avl) __builtin_rvv_vsetvli((size_t)(avl), 1, 0)
#define __riscv_vsetvl_e16m2(avl) __builtin_rvv_vsetvli((size_t)(avl), 1, 1)
#define __riscv_vsetvl_e16m4(avl) __builtin_rvv_vsetvli((size_t)(avl), 1, 2)
#define __riscv_vsetvl_e16m8(avl) __builtin_rvv_vsetvli((size_t)(avl), 1, 3)

#define __riscv_vsetvl_e32m1(avl) __builtin_rvv_vsetvli((size_t)(avl), 2, 0)
#define __riscv_vsetvl_e32m2(avl) __builtin_rvv_vsetvli((size_t)(avl), 2, 1)
#define __riscv_vsetvl_e32m4(avl) __builtin_rvv_vsetvli((size_t)(avl), 2, 2)
#define __riscv_vsetvl_e32m8(avl) __builtin_rvv_vsetvli((size_t)(avl), 2, 3)

#define __riscv_vsetvl_e8mf8(avl) __builtin_rvv_vsetvli((size_t)(avl), 0, 5)
#define __riscv_vsetvl_e16mf4(avl) __builtin_rvv_vsetvli((size_t)(avl), 1, 6)
#define __riscv_vsetvl_e32mf2(avl) __builtin_rvv_vsetvli((size_t)(avl), 2, 7)

#define __riscv_vsetvl_e64m1(avl) __builtin_rvv_vsetvli((size_t)(avl), 3, 0)
#define __riscv_vsetvl_e64m2(avl) __builtin_rvv_vsetvli((size_t)(avl), 3, 1)
#define __riscv_vsetvl_e64m4(avl) __builtin_rvv_vsetvli((size_t)(avl), 3, 2)
#define __riscv_vsetvl_e64m8(avl) __builtin_rvv_vsetvli((size_t)(avl), 3, 3)

#define __riscv_vsetvlmax_e8mf4() __builtin_rvv_vsetvlimax(0, 6)
````
- **L421 EN**: Defines macro `__riscv_vsetvl_e8mf4(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L421 CN**: 定义宏 `__riscv_vsetvl_e8mf4(avl)`，用于条件编译、简写或表驱动展开。
- **L422 EN**: Defines macro `__riscv_vsetvl_e8mf2(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L422 CN**: 定义宏 `__riscv_vsetvl_e8mf2(avl)`，用于条件编译、简写或表驱动展开。
- **L423 EN**: Defines macro `__riscv_vsetvl_e8m1(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L423 CN**: 定义宏 `__riscv_vsetvl_e8m1(avl)`，用于条件编译、简写或表驱动展开。
- **L424 EN**: Defines macro `__riscv_vsetvl_e8m2(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L424 CN**: 定义宏 `__riscv_vsetvl_e8m2(avl)`，用于条件编译、简写或表驱动展开。
- **L425 EN**: Defines macro `__riscv_vsetvl_e8m4(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L425 CN**: 定义宏 `__riscv_vsetvl_e8m4(avl)`，用于条件编译、简写或表驱动展开。
- **L426 EN**: Defines macro `__riscv_vsetvl_e8m8(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L426 CN**: 定义宏 `__riscv_vsetvl_e8m8(avl)`，用于条件编译、简写或表驱动展开。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L428 EN**: Defines macro `__riscv_vsetvl_e16mf2(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L428 CN**: 定义宏 `__riscv_vsetvl_e16mf2(avl)`，用于条件编译、简写或表驱动展开。
- **L429 EN**: Defines macro `__riscv_vsetvl_e16m1(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L429 CN**: 定义宏 `__riscv_vsetvl_e16m1(avl)`，用于条件编译、简写或表驱动展开。
- **L430 EN**: Defines macro `__riscv_vsetvl_e16m2(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L430 CN**: 定义宏 `__riscv_vsetvl_e16m2(avl)`，用于条件编译、简写或表驱动展开。
- **L431 EN**: Defines macro `__riscv_vsetvl_e16m4(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L431 CN**: 定义宏 `__riscv_vsetvl_e16m4(avl)`，用于条件编译、简写或表驱动展开。
- **L432 EN**: Defines macro `__riscv_vsetvl_e16m8(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L432 CN**: 定义宏 `__riscv_vsetvl_e16m8(avl)`，用于条件编译、简写或表驱动展开。
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L434 EN**: Defines macro `__riscv_vsetvl_e32m1(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L434 CN**: 定义宏 `__riscv_vsetvl_e32m1(avl)`，用于条件编译、简写或表驱动展开。
- **L435 EN**: Defines macro `__riscv_vsetvl_e32m2(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L435 CN**: 定义宏 `__riscv_vsetvl_e32m2(avl)`，用于条件编译、简写或表驱动展开。
- **L436 EN**: Defines macro `__riscv_vsetvl_e32m4(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L436 CN**: 定义宏 `__riscv_vsetvl_e32m4(avl)`，用于条件编译、简写或表驱动展开。
- **L437 EN**: Defines macro `__riscv_vsetvl_e32m8(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L437 CN**: 定义宏 `__riscv_vsetvl_e32m8(avl)`，用于条件编译、简写或表驱动展开。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L439 EN**: Defines macro `__riscv_vsetvl_e8mf8(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L439 CN**: 定义宏 `__riscv_vsetvl_e8mf8(avl)`，用于条件编译、简写或表驱动展开。
- **L440 EN**: Defines macro `__riscv_vsetvl_e16mf4(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L440 CN**: 定义宏 `__riscv_vsetvl_e16mf4(avl)`，用于条件编译、简写或表驱动展开。
- **L441 EN**: Defines macro `__riscv_vsetvl_e32mf2(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L441 CN**: 定义宏 `__riscv_vsetvl_e32mf2(avl)`，用于条件编译、简写或表驱动展开。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Defines macro `__riscv_vsetvl_e64m1(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L443 CN**: 定义宏 `__riscv_vsetvl_e64m1(avl)`，用于条件编译、简写或表驱动展开。
- **L444 EN**: Defines macro `__riscv_vsetvl_e64m2(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L444 CN**: 定义宏 `__riscv_vsetvl_e64m2(avl)`，用于条件编译、简写或表驱动展开。
- **L445 EN**: Defines macro `__riscv_vsetvl_e64m4(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L445 CN**: 定义宏 `__riscv_vsetvl_e64m4(avl)`，用于条件编译、简写或表驱动展开。
- **L446 EN**: Defines macro `__riscv_vsetvl_e64m8(avl)` for conditional compilation, shorthand, or table-driven expansion.
  **L446 CN**: 定义宏 `__riscv_vsetvl_e64m8(avl)`，用于条件编译、简写或表驱动展开。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Defines macro `__riscv_vsetvlmax_e8mf4()` for conditional compilation, shorthand, or table-driven expansion.
  **L448 CN**: 定义宏 `__riscv_vsetvlmax_e8mf4()`，用于条件编译、简写或表驱动展开。

### Lines 449-476

````tablegen
#define __riscv_vsetvlmax_e8mf2() __builtin_rvv_vsetvlimax(0, 7)
#define __riscv_vsetvlmax_e8m1() __builtin_rvv_vsetvlimax(0, 0)
#define __riscv_vsetvlmax_e8m2() __builtin_rvv_vsetvlimax(0, 1)
#define __riscv_vsetvlmax_e8m4() __builtin_rvv_vsetvlimax(0, 2)
#define __riscv_vsetvlmax_e8m8() __builtin_rvv_vsetvlimax(0, 3)

#define __riscv_vsetvlmax_e16mf2() __builtin_rvv_vsetvlimax(1, 7)
#define __riscv_vsetvlmax_e16m1() __builtin_rvv_vsetvlimax(1, 0)
#define __riscv_vsetvlmax_e16m2() __builtin_rvv_vsetvlimax(1, 1)
#define __riscv_vsetvlmax_e16m4() __builtin_rvv_vsetvlimax(1, 2)
#define __riscv_vsetvlmax_e16m8() __builtin_rvv_vsetvlimax(1, 3)

#define __riscv_vsetvlmax_e32m1() __builtin_rvv_vsetvlimax(2, 0)
#define __riscv_vsetvlmax_e32m2() __builtin_rvv_vsetvlimax(2, 1)
#define __riscv_vsetvlmax_e32m4() __builtin_rvv_vsetvlimax(2, 2)
#define __riscv_vsetvlmax_e32m8() __builtin_rvv_vsetvlimax(2, 3)

#define __riscv_vsetvlmax_e8mf8() __builtin_rvv_vsetvlimax(0, 5)
#define __riscv_vsetvlmax_e16mf4() __builtin_rvv_vsetvlimax(1, 6)
#define __riscv_vsetvlmax_e32mf2() __builtin_rvv_vsetvlimax(2, 7)

#define __riscv_vsetvlmax_e64m1() __builtin_rvv_vsetvlimax(3, 0)
#define __riscv_vsetvlmax_e64m2() __builtin_rvv_vsetvlimax(3, 1)
#define __riscv_vsetvlmax_e64m4() __builtin_rvv_vsetvlimax(3, 2)
#define __riscv_vsetvlmax_e64m8() __builtin_rvv_vsetvlimax(3, 3)

}] in
def vsetvl_macro: RVVHeader;
````
- **L449 EN**: Defines macro `__riscv_vsetvlmax_e8mf2()` for conditional compilation, shorthand, or table-driven expansion.
  **L449 CN**: 定义宏 `__riscv_vsetvlmax_e8mf2()`，用于条件编译、简写或表驱动展开。
- **L450 EN**: Defines macro `__riscv_vsetvlmax_e8m1()` for conditional compilation, shorthand, or table-driven expansion.
  **L450 CN**: 定义宏 `__riscv_vsetvlmax_e8m1()`，用于条件编译、简写或表驱动展开。
- **L451 EN**: Defines macro `__riscv_vsetvlmax_e8m2()` for conditional compilation, shorthand, or table-driven expansion.
  **L451 CN**: 定义宏 `__riscv_vsetvlmax_e8m2()`，用于条件编译、简写或表驱动展开。
- **L452 EN**: Defines macro `__riscv_vsetvlmax_e8m4()` for conditional compilation, shorthand, or table-driven expansion.
  **L452 CN**: 定义宏 `__riscv_vsetvlmax_e8m4()`，用于条件编译、简写或表驱动展开。
- **L453 EN**: Defines macro `__riscv_vsetvlmax_e8m8()` for conditional compilation, shorthand, or table-driven expansion.
  **L453 CN**: 定义宏 `__riscv_vsetvlmax_e8m8()`，用于条件编译、简写或表驱动展开。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Defines macro `__riscv_vsetvlmax_e16mf2()` for conditional compilation, shorthand, or table-driven expansion.
  **L455 CN**: 定义宏 `__riscv_vsetvlmax_e16mf2()`，用于条件编译、简写或表驱动展开。
- **L456 EN**: Defines macro `__riscv_vsetvlmax_e16m1()` for conditional compilation, shorthand, or table-driven expansion.
  **L456 CN**: 定义宏 `__riscv_vsetvlmax_e16m1()`，用于条件编译、简写或表驱动展开。
- **L457 EN**: Defines macro `__riscv_vsetvlmax_e16m2()` for conditional compilation, shorthand, or table-driven expansion.
  **L457 CN**: 定义宏 `__riscv_vsetvlmax_e16m2()`，用于条件编译、简写或表驱动展开。
- **L458 EN**: Defines macro `__riscv_vsetvlmax_e16m4()` for conditional compilation, shorthand, or table-driven expansion.
  **L458 CN**: 定义宏 `__riscv_vsetvlmax_e16m4()`，用于条件编译、简写或表驱动展开。
- **L459 EN**: Defines macro `__riscv_vsetvlmax_e16m8()` for conditional compilation, shorthand, or table-driven expansion.
  **L459 CN**: 定义宏 `__riscv_vsetvlmax_e16m8()`，用于条件编译、简写或表驱动展开。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L461 EN**: Defines macro `__riscv_vsetvlmax_e32m1()` for conditional compilation, shorthand, or table-driven expansion.
  **L461 CN**: 定义宏 `__riscv_vsetvlmax_e32m1()`，用于条件编译、简写或表驱动展开。
- **L462 EN**: Defines macro `__riscv_vsetvlmax_e32m2()` for conditional compilation, shorthand, or table-driven expansion.
  **L462 CN**: 定义宏 `__riscv_vsetvlmax_e32m2()`，用于条件编译、简写或表驱动展开。
- **L463 EN**: Defines macro `__riscv_vsetvlmax_e32m4()` for conditional compilation, shorthand, or table-driven expansion.
  **L463 CN**: 定义宏 `__riscv_vsetvlmax_e32m4()`，用于条件编译、简写或表驱动展开。
- **L464 EN**: Defines macro `__riscv_vsetvlmax_e32m8()` for conditional compilation, shorthand, or table-driven expansion.
  **L464 CN**: 定义宏 `__riscv_vsetvlmax_e32m8()`，用于条件编译、简写或表驱动展开。
- **L465 EN**: Blank line separating nearby declarations or logic blocks.
  **L465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L466 EN**: Defines macro `__riscv_vsetvlmax_e8mf8()` for conditional compilation, shorthand, or table-driven expansion.
  **L466 CN**: 定义宏 `__riscv_vsetvlmax_e8mf8()`，用于条件编译、简写或表驱动展开。
- **L467 EN**: Defines macro `__riscv_vsetvlmax_e16mf4()` for conditional compilation, shorthand, or table-driven expansion.
  **L467 CN**: 定义宏 `__riscv_vsetvlmax_e16mf4()`，用于条件编译、简写或表驱动展开。
- **L468 EN**: Defines macro `__riscv_vsetvlmax_e32mf2()` for conditional compilation, shorthand, or table-driven expansion.
  **L468 CN**: 定义宏 `__riscv_vsetvlmax_e32mf2()`，用于条件编译、简写或表驱动展开。
- **L469 EN**: Blank line separating nearby declarations or logic blocks.
  **L469 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L470 EN**: Defines macro `__riscv_vsetvlmax_e64m1()` for conditional compilation, shorthand, or table-driven expansion.
  **L470 CN**: 定义宏 `__riscv_vsetvlmax_e64m1()`，用于条件编译、简写或表驱动展开。
- **L471 EN**: Defines macro `__riscv_vsetvlmax_e64m2()` for conditional compilation, shorthand, or table-driven expansion.
  **L471 CN**: 定义宏 `__riscv_vsetvlmax_e64m2()`，用于条件编译、简写或表驱动展开。
- **L472 EN**: Defines macro `__riscv_vsetvlmax_e64m4()` for conditional compilation, shorthand, or table-driven expansion.
  **L472 CN**: 定义宏 `__riscv_vsetvlmax_e64m4()`，用于条件编译、简写或表驱动展开。
- **L473 EN**: Defines macro `__riscv_vsetvlmax_e64m8()` for conditional compilation, shorthand, or table-driven expansion.
  **L473 CN**: 定义宏 `__riscv_vsetvlmax_e64m8()`，用于条件编译、简写或表驱动展开。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Continues the surrounding expression or declaration: `}] in`.
  **L475 CN**: 继续构造周围的表达式或声明：`}] in`。
- **L476 EN**: Declares TableGen def record `vsetvl_macro`.
  **L476 CN**: 声明 TableGen def 记录 `vsetvl_macro`。

### Lines 477-504

````tablegen

let HasBuiltinAlias = false,
    HasVL = false,
    HasMasked = false,
    MaskedPolicyScheme = NonePolicy,
    Log2LMUL = [0],
    ManualCodegen = [{
      return emitRVVVsetvliBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                   PolicyAttrs, IsMasked, SegInstSEW);
    }] in // Set XLEN type
{
  def vsetvli : RVVBuiltin<"", "zzKzKz", "i">;
  def vsetvlimax : RVVBuiltin<"", "zKzKz", "i">;
}

// 7. Vector Loads and Stores
// 7.4. Vector Unit-Stride Instructions
def vlm: RVVVLEMaskBuiltin;
defm vle8: RVVVLEBuiltin<["c"]>;
defm vle16: RVVVLEBuiltin<["s","x","y"]>;
defm vle32: RVVVLEBuiltin<["i","f"]>;
defm vle64: RVVVLEBuiltin<["l","d"]>;

def vsm : RVVVSEMaskBuiltin;
defm vse8 : RVVVSEBuiltin<["c"]>;
defm vse16: RVVVSEBuiltin<["s","x","y"]>;
defm vse32: RVVVSEBuiltin<["i","f"]>;
defm vse64: RVVVSEBuiltin<["l","d"]>;
````
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L478 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasBuiltinAlias = false,`.
  **L478 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasBuiltinAlias = false,`。
- **L479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasVL = false,`.
  **L479 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasVL = false,`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasMasked = false,`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasMasked = false,`。
- **L481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L481 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Log2LMUL = [0],`.
  **L482 CN**: 继续一个多行参数列表、初始化器或聚合项：`Log2LMUL = [0],`。
- **L483 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L483 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L484 EN**: Returns from the current function with `emitRVVVsetvliBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L484 CN**: 以 `emitRVVVsetvliBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L485 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L485 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L486 EN**: Continues the surrounding expression or declaration: `}] in // Set XLEN type`.
  **L486 CN**: 继续构造周围的表达式或声明：`}] in // Set XLEN type`。
- **L487 EN**: Opens a new lexical scope or compound statement.
  **L487 CN**: 打开一个新的词法作用域或复合语句块。
- **L488 EN**: Declares TableGen def record `vsetvli`.
  **L488 CN**: 声明 TableGen def 记录 `vsetvli`。
- **L489 EN**: Declares TableGen def record `vsetvlimax`.
  **L489 CN**: 声明 TableGen def 记录 `vsetvlimax`。
- **L490 EN**: Closes the current lexical scope or compound statement.
  **L490 CN**: 结束当前词法作用域或复合语句块。
- **L491 EN**: Blank line separating nearby declarations or logic blocks.
  **L491 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L492 EN**: Comment explains nearby logic, constraints, or intent: `7. Vector Loads and Stores`.
  **L492 CN**: 注释解释附近代码的逻辑、约束或设计意图：`7. Vector Loads and Stores`。
- **L493 EN**: Comment explains nearby logic, constraints, or intent: `7.4. Vector Unit-Stride Instructions`.
  **L493 CN**: 注释解释附近代码的逻辑、约束或设计意图：`7.4. Vector Unit-Stride Instructions`。
- **L494 EN**: Declares TableGen def record `vlm`.
  **L494 CN**: 声明 TableGen def 记录 `vlm`。
- **L495 EN**: Declares TableGen defm record `vle8`.
  **L495 CN**: 声明 TableGen defm 记录 `vle8`。
- **L496 EN**: Declares TableGen defm record `vle16`.
  **L496 CN**: 声明 TableGen defm 记录 `vle16`。
- **L497 EN**: Declares TableGen defm record `vle32`.
  **L497 CN**: 声明 TableGen defm 记录 `vle32`。
- **L498 EN**: Declares TableGen defm record `vle64`.
  **L498 CN**: 声明 TableGen defm 记录 `vle64`。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L500 EN**: Declares TableGen def record `vsm`.
  **L500 CN**: 声明 TableGen def 记录 `vsm`。
- **L501 EN**: Declares TableGen defm record `vse8`.
  **L501 CN**: 声明 TableGen defm 记录 `vse8`。
- **L502 EN**: Declares TableGen defm record `vse16`.
  **L502 CN**: 声明 TableGen defm 记录 `vse16`。
- **L503 EN**: Declares TableGen defm record `vse32`.
  **L503 CN**: 声明 TableGen defm 记录 `vse32`。
- **L504 EN**: Declares TableGen defm record `vse64`.
  **L504 CN**: 声明 TableGen defm 记录 `vse64`。

### Lines 505-532

````tablegen

// 7.5. Vector Strided Instructions
defm vlse8: RVVVLSEBuiltin<["c"]>;
defm vlse16: RVVVLSEBuiltin<["s","x","y"]>;
defm vlse32: RVVVLSEBuiltin<["i","f"]>;
defm vlse64: RVVVLSEBuiltin<["l","d"]>;

defm vsse8 : RVVVSSEBuiltin<["c"]>;
defm vsse16: RVVVSSEBuiltin<["s","x","y"]>;
defm vsse32: RVVVSSEBuiltin<["i","f"]>;
defm vsse64: RVVVSSEBuiltin<["l","d"]>;

// 7.6. Vector Indexed Instructions
defm : RVVIndexedLoad<"vluxei">;
defm : RVVIndexedLoad<"vloxei">;

defm : RVVIndexedStore<"vsuxei">;
defm : RVVIndexedStore<"vsoxei">;

// 7.7. Unit-stride Fault-Only-First Loads
defm vle8ff: RVVVLEFFBuiltin<["c"]>;
defm vle16ff: RVVVLEFFBuiltin<["s","x","y"]>;
defm vle32ff: RVVVLEFFBuiltin<["i", "f"]>;
defm vle64ff: RVVVLEFFBuiltin<["l", "d"]>;

multiclass RVVUnitStridedSegLoadTuple<string op> {
  foreach type = TypeList in {
    defvar eew = !cond(!eq(type, "c") : "8",
````
- **L505 EN**: Blank line separating nearby declarations or logic blocks.
  **L505 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L506 EN**: Comment explains nearby logic, constraints, or intent: `7.5. Vector Strided Instructions`.
  **L506 CN**: 注释解释附近代码的逻辑、约束或设计意图：`7.5. Vector Strided Instructions`。
- **L507 EN**: Declares TableGen defm record `vlse8`.
  **L507 CN**: 声明 TableGen defm 记录 `vlse8`。
- **L508 EN**: Declares TableGen defm record `vlse16`.
  **L508 CN**: 声明 TableGen defm 记录 `vlse16`。
- **L509 EN**: Declares TableGen defm record `vlse32`.
  **L509 CN**: 声明 TableGen defm 记录 `vlse32`。
- **L510 EN**: Declares TableGen defm record `vlse64`.
  **L510 CN**: 声明 TableGen defm 记录 `vlse64`。
- **L511 EN**: Blank line separating nearby declarations or logic blocks.
  **L511 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L512 EN**: Declares TableGen defm record `vsse8`.
  **L512 CN**: 声明 TableGen defm 记录 `vsse8`。
- **L513 EN**: Declares TableGen defm record `vsse16`.
  **L513 CN**: 声明 TableGen defm 记录 `vsse16`。
- **L514 EN**: Declares TableGen defm record `vsse32`.
  **L514 CN**: 声明 TableGen defm 记录 `vsse32`。
- **L515 EN**: Declares TableGen defm record `vsse64`.
  **L515 CN**: 声明 TableGen defm 记录 `vsse64`。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L517 EN**: Comment explains nearby logic, constraints, or intent: `7.6. Vector Indexed Instructions`.
  **L517 CN**: 注释解释附近代码的逻辑、约束或设计意图：`7.6. Vector Indexed Instructions`。
- **L518 EN**: Declares TableGen defm record `defm`.
  **L518 CN**: 声明 TableGen defm 记录 `defm`。
- **L519 EN**: Declares TableGen defm record `defm`.
  **L519 CN**: 声明 TableGen defm 记录 `defm`。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L521 EN**: Declares TableGen defm record `defm`.
  **L521 CN**: 声明 TableGen defm 记录 `defm`。
- **L522 EN**: Declares TableGen defm record `defm`.
  **L522 CN**: 声明 TableGen defm 记录 `defm`。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L524 EN**: Comment explains nearby logic, constraints, or intent: `7.7. Unit-stride Fault-Only-First Loads`.
  **L524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`7.7. Unit-stride Fault-Only-First Loads`。
- **L525 EN**: Declares TableGen defm record `vle8ff`.
  **L525 CN**: 声明 TableGen defm 记录 `vle8ff`。
- **L526 EN**: Declares TableGen defm record `vle16ff`.
  **L526 CN**: 声明 TableGen defm 记录 `vle16ff`。
- **L527 EN**: Declares TableGen defm record `vle32ff`.
  **L527 CN**: 声明 TableGen defm 记录 `vle32ff`。
- **L528 EN**: Declares TableGen defm record `vle64ff`.
  **L528 CN**: 声明 TableGen defm 记录 `vle64ff`。
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Declares TableGen multiclass record `RVVUnitStridedSegLoadTuple`.
  **L530 CN**: 声明 TableGen multiclass 记录 `RVVUnitStridedSegLoadTuple`。
- **L531 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L531 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L532 EN**: Declares TableGen defvar record `eew = !cond`.
  **L532 CN**: 声明 TableGen defvar 记录 `eew = !cond`。

### Lines 533-560

````tablegen
                       !eq(type, "s") : "16",
                       !eq(type, "i") : "32",
                       !eq(type, "l") : "64",
                       !eq(type, "x") : "16",
                       !eq(type, "f") : "32",
                       !eq(type, "d") : "64",
                       !eq(type, "y") : "16");
      foreach nf = NFList in {
        let Name = op # nf # "e" # eew # "_v",
            IRName = op # nf,
            MaskedIRName = op # nf # "_mask",
            NF = nf,
            ManualCodegen = [{
              return emitRVVUnitStridedSegLoadTupleBuiltin(
                  this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs,
                  IsMasked, SegInstSEW);
            }] in {
        defvar T = "(Tuple:" # nf # ")";
        def : RVVBuiltin<T # "v", T # "vPCe", type>;
        if !not(IsFloat<type>.val) then {
          def : RVVBuiltin<T # "Uv", T # "UvPCUe", type>;
        }
      }
    }
  }
}

multiclass RVVUnitStridedSegStoreTuple<string op> {
````
- **L533 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "s") : "16",`.
  **L533 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "s") : "16",`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "i") : "32",`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "i") : "32",`。
- **L535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "l") : "64",`.
  **L535 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "l") : "64",`。
- **L536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "x") : "16",`.
  **L536 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "x") : "16",`。
- **L537 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "f") : "32",`.
  **L537 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "f") : "32",`。
- **L538 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "d") : "64",`.
  **L538 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "d") : "64",`。
- **L539 EN**: Executes a call or declaration centered on `!eq`.
  **L539 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L540 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L540 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L541 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # nf # "e" # eew # "_v",`.
  **L541 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # nf # "e" # eew # "_v",`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = op # nf,`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = op # nf,`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = op # nf # "_mask",`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = op # nf # "_mask",`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NF = nf,`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`NF = nf,`。
- **L545 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L545 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L546 EN**: Returns from the current function with `emitRVVUnitStridedSegLoadTupleBuiltin(`.
  **L546 CN**: 以 `emitRVVUnitStridedSegLoadTupleBuiltin(` 从当前函数返回。
- **L547 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs,`.
  **L547 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs,`。
- **L548 EN**: Adds a standalone statement or declaration: `IsMasked, SegInstSEW);`.
  **L548 CN**: 添加一条独立语句或声明：`IsMasked, SegInstSEW);`。
- **L549 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L549 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L550 EN**: Declares TableGen defvar record `T = "`.
  **L550 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L551 EN**: Declares TableGen def record `def`.
  **L551 CN**: 声明 TableGen def 记录 `def`。
- **L552 EN**: Continues logic associated with callable symbol `not`.
  **L552 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L553 EN**: Declares TableGen def record `def`.
  **L553 CN**: 声明 TableGen def 记录 `def`。
- **L554 EN**: Closes the current lexical scope or compound statement.
  **L554 CN**: 结束当前词法作用域或复合语句块。
- **L555 EN**: Closes the current lexical scope or compound statement.
  **L555 CN**: 结束当前词法作用域或复合语句块。
- **L556 EN**: Closes the current lexical scope or compound statement.
  **L556 CN**: 结束当前词法作用域或复合语句块。
- **L557 EN**: Closes the current lexical scope or compound statement.
  **L557 CN**: 结束当前词法作用域或复合语句块。
- **L558 EN**: Closes the current lexical scope or compound statement.
  **L558 CN**: 结束当前词法作用域或复合语句块。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L560 EN**: Declares TableGen multiclass record `RVVUnitStridedSegStoreTuple`.
  **L560 CN**: 声明 TableGen multiclass 记录 `RVVUnitStridedSegStoreTuple`。

### Lines 561-588

````tablegen
  foreach type = TypeList in {
    defvar eew = !cond(!eq(type, "c") : "8",
                       !eq(type, "s") : "16",
                       !eq(type, "i") : "32",
                       !eq(type, "l") : "64",
                       !eq(type, "x") : "16",
                       !eq(type, "f") : "32",
                       !eq(type, "d") : "64",
                       !eq(type, "y") : "16");
      foreach nf = NFList in {
      let Name = op # nf # "e" # eew # "_v",
          IRName = op # nf,
          MaskedIRName = op # nf # "_mask",
          NF = nf,
          HasMaskedOffOperand = false,
          ManualCodegen = [{
            return emitRVVUnitStridedSegStoreTupleBuiltin(
                this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs,
                IsMasked, SegInstSEW);
          }] in {
        defvar T = "(Tuple:" # nf # ")";
        def : RVVBuiltin<T # "v", "0Pe" # T # "v", type>;
        if !not(IsFloat<type>.val) then {
          def : RVVBuiltin<T # "Uv", "0PUe" # T # "Uv", type>;
        }
      }
    }
  }
````
- **L561 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L561 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L562 EN**: Declares TableGen defvar record `eew = !cond`.
  **L562 CN**: 声明 TableGen defvar 记录 `eew = !cond`。
- **L563 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "s") : "16",`.
  **L563 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "s") : "16",`。
- **L564 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "i") : "32",`.
  **L564 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "i") : "32",`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "l") : "64",`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "l") : "64",`。
- **L566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "x") : "16",`.
  **L566 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "x") : "16",`。
- **L567 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "f") : "32",`.
  **L567 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "f") : "32",`。
- **L568 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "d") : "64",`.
  **L568 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "d") : "64",`。
- **L569 EN**: Executes a call or declaration centered on `!eq`.
  **L569 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L570 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L570 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L571 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # nf # "e" # eew # "_v",`.
  **L571 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # nf # "e" # eew # "_v",`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = op # nf,`.
  **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = op # nf,`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = op # nf # "_mask",`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = op # nf # "_mask",`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NF = nf,`.
  **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`NF = nf,`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasMaskedOffOperand = false,`.
  **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasMaskedOffOperand = false,`。
- **L576 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L576 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L577 EN**: Returns from the current function with `emitRVVUnitStridedSegStoreTupleBuiltin(`.
  **L577 CN**: 以 `emitRVVUnitStridedSegStoreTupleBuiltin(` 从当前函数返回。
- **L578 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs,`.
  **L578 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs,`。
- **L579 EN**: Adds a standalone statement or declaration: `IsMasked, SegInstSEW);`.
  **L579 CN**: 添加一条独立语句或声明：`IsMasked, SegInstSEW);`。
- **L580 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L580 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L581 EN**: Declares TableGen defvar record `T = "`.
  **L581 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L582 EN**: Declares TableGen def record `def`.
  **L582 CN**: 声明 TableGen def 记录 `def`。
- **L583 EN**: Continues logic associated with callable symbol `not`.
  **L583 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L584 EN**: Declares TableGen def record `def`.
  **L584 CN**: 声明 TableGen def 记录 `def`。
- **L585 EN**: Closes the current lexical scope or compound statement.
  **L585 CN**: 结束当前词法作用域或复合语句块。
- **L586 EN**: Closes the current lexical scope or compound statement.
  **L586 CN**: 结束当前词法作用域或复合语句块。
- **L587 EN**: Closes the current lexical scope or compound statement.
  **L587 CN**: 结束当前词法作用域或复合语句块。
- **L588 EN**: Closes the current lexical scope or compound statement.
  **L588 CN**: 结束当前词法作用域或复合语句块。

### Lines 589-616

````tablegen
}

multiclass RVVUnitStridedSegLoadFFTuple<string op> {
  foreach type = TypeList in {
    defvar eew = !cond(!eq(type, "c") : "8",
                       !eq(type, "s") : "16",
                       !eq(type, "i") : "32",
                       !eq(type, "l") : "64",
                       !eq(type, "x") : "16",
                       !eq(type, "f") : "32",
                       !eq(type, "d") : "64",
                       !eq(type, "y") : "16");
      foreach nf = NFList in {
        let Name = op # nf # "e" # eew # "ff_v",
            IRName = op # nf # "ff",
            MaskedIRName = op # nf # "ff_mask",
            NF = nf,
            ManualCodegen = [{
      return emitRVVUnitStridedSegLoadFFTupleBuiltin(
          this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,
          SegInstSEW);
    }] in {
        defvar T = "(Tuple:" # nf # ")";
        def : RVVBuiltin<T # "v", T # "vPCePz", type>;
        if !not(IsFloat<type>.val) then {
          def : RVVBuiltin<T # "Uv", T # "UvPCUePz", type>;
        }
      }
````
- **L589 EN**: Closes the current lexical scope or compound statement.
  **L589 CN**: 结束当前词法作用域或复合语句块。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Declares TableGen multiclass record `RVVUnitStridedSegLoadFFTuple`.
  **L591 CN**: 声明 TableGen multiclass 记录 `RVVUnitStridedSegLoadFFTuple`。
- **L592 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L592 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L593 EN**: Declares TableGen defvar record `eew = !cond`.
  **L593 CN**: 声明 TableGen defvar 记录 `eew = !cond`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "s") : "16",`.
  **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "s") : "16",`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "i") : "32",`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "i") : "32",`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "l") : "64",`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "l") : "64",`。
- **L597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "x") : "16",`.
  **L597 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "x") : "16",`。
- **L598 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "f") : "32",`.
  **L598 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "f") : "32",`。
- **L599 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "d") : "64",`.
  **L599 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "d") : "64",`。
- **L600 EN**: Executes a call or declaration centered on `!eq`.
  **L600 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L601 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L601 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L602 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # nf # "e" # eew # "ff_v",`.
  **L602 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # nf # "e" # eew # "ff_v",`。
- **L603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = op # nf # "ff",`.
  **L603 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = op # nf # "ff",`。
- **L604 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = op # nf # "ff_mask",`.
  **L604 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = op # nf # "ff_mask",`。
- **L605 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NF = nf,`.
  **L605 CN**: 继续一个多行参数列表、初始化器或聚合项：`NF = nf,`。
- **L606 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L606 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L607 EN**: Returns from the current function with `emitRVVUnitStridedSegLoadFFTupleBuiltin(`.
  **L607 CN**: 以 `emitRVVUnitStridedSegLoadFFTupleBuiltin(` 从当前函数返回。
- **L608 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`.
  **L608 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`。
- **L609 EN**: Adds a standalone statement or declaration: `SegInstSEW);`.
  **L609 CN**: 添加一条独立语句或声明：`SegInstSEW);`。
- **L610 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L610 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L611 EN**: Declares TableGen defvar record `T = "`.
  **L611 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L612 EN**: Declares TableGen def record `def`.
  **L612 CN**: 声明 TableGen def 记录 `def`。
- **L613 EN**: Continues logic associated with callable symbol `not`.
  **L613 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L614 EN**: Declares TableGen def record `def`.
  **L614 CN**: 声明 TableGen def 记录 `def`。
- **L615 EN**: Closes the current lexical scope or compound statement.
  **L615 CN**: 结束当前词法作用域或复合语句块。
- **L616 EN**: Closes the current lexical scope or compound statement.
  **L616 CN**: 结束当前词法作用域或复合语句块。

### Lines 617-644

````tablegen
    }
  }
}

multiclass RVVStridedSegLoadTuple<string op> {
  foreach type = TypeList in {
    defvar eew = !cond(!eq(type, "c") : "8",
                       !eq(type, "s") : "16",
                       !eq(type, "i") : "32",
                       !eq(type, "l") : "64",
                       !eq(type, "x") : "16",
                       !eq(type, "f") : "32",
                       !eq(type, "d") : "64",
                       !eq(type, "y") : "16");
      foreach nf = NFList in {
        let Name = op # nf # "e" # eew # "_v",
            IRName = op # nf,
            MaskedIRName = op # nf # "_mask",
            NF = nf,
            ManualCodegen = [{
      return emitRVVStridedSegLoadTupleBuiltin(
          this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,
          SegInstSEW);
    }] in {
        defvar T = "(Tuple:" # nf # ")";
        def : RVVBuiltin<T # "v", T # "vPCet", type>;
        if !not(IsFloat<type>.val) then {
          def : RVVBuiltin<T # "Uv", T # "UvPCUet", type>;
````
- **L617 EN**: Closes the current lexical scope or compound statement.
  **L617 CN**: 结束当前词法作用域或复合语句块。
- **L618 EN**: Closes the current lexical scope or compound statement.
  **L618 CN**: 结束当前词法作用域或复合语句块。
- **L619 EN**: Closes the current lexical scope or compound statement.
  **L619 CN**: 结束当前词法作用域或复合语句块。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Declares TableGen multiclass record `RVVStridedSegLoadTuple`.
  **L621 CN**: 声明 TableGen multiclass 记录 `RVVStridedSegLoadTuple`。
- **L622 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L622 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L623 EN**: Declares TableGen defvar record `eew = !cond`.
  **L623 CN**: 声明 TableGen defvar 记录 `eew = !cond`。
- **L624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "s") : "16",`.
  **L624 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "s") : "16",`。
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "i") : "32",`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "i") : "32",`。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "l") : "64",`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "l") : "64",`。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "x") : "16",`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "x") : "16",`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "f") : "32",`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "f") : "32",`。
- **L629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "d") : "64",`.
  **L629 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "d") : "64",`。
- **L630 EN**: Executes a call or declaration centered on `!eq`.
  **L630 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L631 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L631 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L632 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # nf # "e" # eew # "_v",`.
  **L632 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # nf # "e" # eew # "_v",`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = op # nf,`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = op # nf,`。
- **L634 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = op # nf # "_mask",`.
  **L634 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = op # nf # "_mask",`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NF = nf,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`NF = nf,`。
- **L636 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L636 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L637 EN**: Returns from the current function with `emitRVVStridedSegLoadTupleBuiltin(`.
  **L637 CN**: 以 `emitRVVStridedSegLoadTupleBuiltin(` 从当前函数返回。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`.
  **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`。
- **L639 EN**: Adds a standalone statement or declaration: `SegInstSEW);`.
  **L639 CN**: 添加一条独立语句或声明：`SegInstSEW);`。
- **L640 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L640 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L641 EN**: Declares TableGen defvar record `T = "`.
  **L641 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L642 EN**: Declares TableGen def record `def`.
  **L642 CN**: 声明 TableGen def 记录 `def`。
- **L643 EN**: Continues logic associated with callable symbol `not`.
  **L643 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L644 EN**: Declares TableGen def record `def`.
  **L644 CN**: 声明 TableGen def 记录 `def`。

### Lines 645-672

````tablegen
        }
      }
    }
  }
}

multiclass RVVStridedSegStoreTuple<string op> {
  foreach type = TypeList in {
    defvar eew = !cond(!eq(type, "c") : "8",
                       !eq(type, "s") : "16",
                       !eq(type, "i") : "32",
                       !eq(type, "l") : "64",
                       !eq(type, "x") : "16",
                       !eq(type, "f") : "32",
                       !eq(type, "d") : "64",
                       !eq(type, "y") : "16");
      foreach nf = NFList in {
        let Name = op # nf # "e" # eew # "_v",
            IRName = op # nf,
            MaskedIRName = op # nf # "_mask",
            NF = nf,
            HasMaskedOffOperand = false,
            MaskedPolicyScheme = NonePolicy,
            ManualCodegen = [{
      return emitRVVStridedSegStoreTupleBuiltin(
          this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,
          SegInstSEW);
            }] in {
````
- **L645 EN**: Closes the current lexical scope or compound statement.
  **L645 CN**: 结束当前词法作用域或复合语句块。
- **L646 EN**: Closes the current lexical scope or compound statement.
  **L646 CN**: 结束当前词法作用域或复合语句块。
- **L647 EN**: Closes the current lexical scope or compound statement.
  **L647 CN**: 结束当前词法作用域或复合语句块。
- **L648 EN**: Closes the current lexical scope or compound statement.
  **L648 CN**: 结束当前词法作用域或复合语句块。
- **L649 EN**: Closes the current lexical scope or compound statement.
  **L649 CN**: 结束当前词法作用域或复合语句块。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Declares TableGen multiclass record `RVVStridedSegStoreTuple`.
  **L651 CN**: 声明 TableGen multiclass 记录 `RVVStridedSegStoreTuple`。
- **L652 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L652 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L653 EN**: Declares TableGen defvar record `eew = !cond`.
  **L653 CN**: 声明 TableGen defvar 记录 `eew = !cond`。
- **L654 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "s") : "16",`.
  **L654 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "s") : "16",`。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "i") : "32",`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "i") : "32",`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "l") : "64",`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "l") : "64",`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "x") : "16",`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "x") : "16",`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "f") : "32",`.
  **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "f") : "32",`。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type, "d") : "64",`.
  **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type, "d") : "64",`。
- **L660 EN**: Executes a call or declaration centered on `!eq`.
  **L660 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L661 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L661 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L662 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # nf # "e" # eew # "_v",`.
  **L662 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # nf # "e" # eew # "_v",`。
- **L663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = op # nf,`.
  **L663 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = op # nf,`。
- **L664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = op # nf # "_mask",`.
  **L664 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = op # nf # "_mask",`。
- **L665 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NF = nf,`.
  **L665 CN**: 继续一个多行参数列表、初始化器或聚合项：`NF = nf,`。
- **L666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasMaskedOffOperand = false,`.
  **L666 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasMaskedOffOperand = false,`。
- **L667 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L667 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L668 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L668 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L669 EN**: Returns from the current function with `emitRVVStridedSegStoreTupleBuiltin(`.
  **L669 CN**: 以 `emitRVVStridedSegStoreTupleBuiltin(` 从当前函数返回。
- **L670 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`.
  **L670 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`。
- **L671 EN**: Adds a standalone statement or declaration: `SegInstSEW);`.
  **L671 CN**: 添加一条独立语句或声明：`SegInstSEW);`。
- **L672 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L672 CN**: 继续构造周围的表达式或声明：`}] in {`。

### Lines 673-700

````tablegen
        defvar T = "(Tuple:" # nf # ")";
        def : RVVBuiltin<T # "v", "0Pet" # T # "v", type>;
        if !not(IsFloat<type>.val) then {
          def : RVVBuiltin<T # "Uv", "0PUet" # T # "Uv", type>;
        }
      }
    }
  }
}

multiclass RVVIndexedSegLoadTuple<string op> {
  foreach type = TypeList in {
    foreach eew_info = EEWList in {
      defvar eew = eew_info[0];
      defvar eew_type = eew_info[1];
      foreach nf = NFList in {
        let Name = op # nf # "ei" # eew # "_v",
            IRName = op # nf,
            MaskedIRName = op # nf # "_mask",
            NF = nf,
            ManualCodegen = [{
      return emitRVVIndexedSegLoadTupleBuiltin(
          this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,
          SegInstSEW);
    }] in {
          defvar T = "(Tuple:" # nf # ")";
          def : RVVBuiltin<T # "v", T # "vPCe" # eew_type # "Uv", type>;
          if !not(IsFloat<type>.val) then {
````
- **L673 EN**: Declares TableGen defvar record `T = "`.
  **L673 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L674 EN**: Declares TableGen def record `def`.
  **L674 CN**: 声明 TableGen def 记录 `def`。
- **L675 EN**: Continues logic associated with callable symbol `not`.
  **L675 CN**: 继续与可调用符号 `not` 相关的逻辑。
- **L676 EN**: Declares TableGen def record `def`.
  **L676 CN**: 声明 TableGen def 记录 `def`。
- **L677 EN**: Closes the current lexical scope or compound statement.
  **L677 CN**: 结束当前词法作用域或复合语句块。
- **L678 EN**: Closes the current lexical scope or compound statement.
  **L678 CN**: 结束当前词法作用域或复合语句块。
- **L679 EN**: Closes the current lexical scope or compound statement.
  **L679 CN**: 结束当前词法作用域或复合语句块。
- **L680 EN**: Closes the current lexical scope or compound statement.
  **L680 CN**: 结束当前词法作用域或复合语句块。
- **L681 EN**: Closes the current lexical scope or compound statement.
  **L681 CN**: 结束当前词法作用域或复合语句块。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L683 EN**: Declares TableGen multiclass record `RVVIndexedSegLoadTuple`.
  **L683 CN**: 声明 TableGen multiclass 记录 `RVVIndexedSegLoadTuple`。
- **L684 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L684 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L685 EN**: Starts a TableGen iteration used to generate repeated records: `foreach eew_info = EEWList in {`.
  **L685 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach eew_info = EEWList in {`。
- **L686 EN**: Declares TableGen defvar record `eew = eew_info[0];`.
  **L686 CN**: 声明 TableGen defvar 记录 `eew = eew_info[0];`。
- **L687 EN**: Declares TableGen defvar record `eew_type = eew_info[1];`.
  **L687 CN**: 声明 TableGen defvar 记录 `eew_type = eew_info[1];`。
- **L688 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L688 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L689 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # nf # "ei" # eew # "_v",`.
  **L689 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # nf # "ei" # eew # "_v",`。
- **L690 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = op # nf,`.
  **L690 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = op # nf,`。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = op # nf # "_mask",`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = op # nf # "_mask",`。
- **L692 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NF = nf,`.
  **L692 CN**: 继续一个多行参数列表、初始化器或聚合项：`NF = nf,`。
- **L693 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L693 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L694 EN**: Returns from the current function with `emitRVVIndexedSegLoadTupleBuiltin(`.
  **L694 CN**: 以 `emitRVVIndexedSegLoadTupleBuiltin(` 从当前函数返回。
- **L695 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`.
  **L695 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`。
- **L696 EN**: Adds a standalone statement or declaration: `SegInstSEW);`.
  **L696 CN**: 添加一条独立语句或声明：`SegInstSEW);`。
- **L697 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L697 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L698 EN**: Declares TableGen defvar record `T = "`.
  **L698 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L699 EN**: Declares TableGen def record `def`.
  **L699 CN**: 声明 TableGen def 记录 `def`。
- **L700 EN**: Continues logic associated with callable symbol `not`.
  **L700 CN**: 继续与可调用符号 `not` 相关的逻辑。

### Lines 701-728

````tablegen
            def : RVVBuiltin<T # "Uv", T # "UvPCUe" # eew_type # "Uv", type>;
          }
        }
      }
    }
  }
}

multiclass RVVIndexedSegStoreTuple<string op> {
  foreach type = TypeList in {
    foreach eew_info = EEWList in {
      defvar eew = eew_info[0];
      defvar eew_type = eew_info[1];
      foreach nf = NFList in {
        let Name = op # nf # "ei" # eew # "_v",
            IRName = op # nf,
            MaskedIRName = op # nf # "_mask",
            NF = nf,
            HasMaskedOffOperand = false,
            MaskedPolicyScheme = NonePolicy,
            ManualCodegen = [{
      return emitRVVIndexedSegStoreTupleBuiltin(
          this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,
          SegInstSEW);
            }] in {
          defvar T = "(Tuple:" # nf # ")";
          def : RVVBuiltin<T # "v", "0Pe" # eew_type # "Uv" # T # "v", type>;
          if !not(IsFloat<type>.val) then {
````
- **L701 EN**: Declares TableGen def record `def`.
  **L701 CN**: 声明 TableGen def 记录 `def`。
- **L702 EN**: Closes the current lexical scope or compound statement.
  **L702 CN**: 结束当前词法作用域或复合语句块。
- **L703 EN**: Closes the current lexical scope or compound statement.
  **L703 CN**: 结束当前词法作用域或复合语句块。
- **L704 EN**: Closes the current lexical scope or compound statement.
  **L704 CN**: 结束当前词法作用域或复合语句块。
- **L705 EN**: Closes the current lexical scope or compound statement.
  **L705 CN**: 结束当前词法作用域或复合语句块。
- **L706 EN**: Closes the current lexical scope or compound statement.
  **L706 CN**: 结束当前词法作用域或复合语句块。
- **L707 EN**: Closes the current lexical scope or compound statement.
  **L707 CN**: 结束当前词法作用域或复合语句块。
- **L708 EN**: Blank line separating nearby declarations or logic blocks.
  **L708 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L709 EN**: Declares TableGen multiclass record `RVVIndexedSegStoreTuple`.
  **L709 CN**: 声明 TableGen multiclass 记录 `RVVIndexedSegStoreTuple`。
- **L710 EN**: Starts a TableGen iteration used to generate repeated records: `foreach type = TypeList in {`.
  **L710 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach type = TypeList in {`。
- **L711 EN**: Starts a TableGen iteration used to generate repeated records: `foreach eew_info = EEWList in {`.
  **L711 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach eew_info = EEWList in {`。
- **L712 EN**: Declares TableGen defvar record `eew = eew_info[0];`.
  **L712 CN**: 声明 TableGen defvar 记录 `eew = eew_info[0];`。
- **L713 EN**: Declares TableGen defvar record `eew_type = eew_info[1];`.
  **L713 CN**: 声明 TableGen defvar 记录 `eew_type = eew_info[1];`。
- **L714 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L714 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L715 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = op # nf # "ei" # eew # "_v",`.
  **L715 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = op # nf # "ei" # eew # "_v",`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = op # nf,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = op # nf,`。
- **L717 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = op # nf # "_mask",`.
  **L717 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = op # nf # "_mask",`。
- **L718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NF = nf,`.
  **L718 CN**: 继续一个多行参数列表、初始化器或聚合项：`NF = nf,`。
- **L719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasMaskedOffOperand = false,`.
  **L719 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasMaskedOffOperand = false,`。
- **L720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L720 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L721 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L721 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L722 EN**: Returns from the current function with `emitRVVIndexedSegStoreTupleBuiltin(`.
  **L722 CN**: 以 `emitRVVIndexedSegStoreTupleBuiltin(` 从当前函数返回。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`.
  **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`。
- **L724 EN**: Adds a standalone statement or declaration: `SegInstSEW);`.
  **L724 CN**: 添加一条独立语句或声明：`SegInstSEW);`。
- **L725 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L725 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L726 EN**: Declares TableGen defvar record `T = "`.
  **L726 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L727 EN**: Declares TableGen def record `def`.
  **L727 CN**: 声明 TableGen def 记录 `def`。
- **L728 EN**: Continues logic associated with callable symbol `not`.
  **L728 CN**: 继续与可调用符号 `not` 相关的逻辑。

### Lines 729-756

````tablegen
            def : RVVBuiltin<T # "Uv", "0PUe" # eew_type # "Uv" # T # "Uv", type>;
          }
        }
      }
    }
  }
}

// 7.8 Vector Load/Store Segment Instructions
let UnMaskedPolicyScheme = HasPassthruOperand,
    IsTuple = true in {
  defm : RVVUnitStridedSegLoadTuple<"vlseg">;
  defm : RVVUnitStridedSegLoadFFTuple<"vlseg">;
  defm : RVVStridedSegLoadTuple<"vlsseg">;
  defm : RVVIndexedSegLoadTuple<"vluxseg">;
  defm : RVVIndexedSegLoadTuple<"vloxseg">;
}

let UnMaskedPolicyScheme = NonePolicy,
    MaskedPolicyScheme = NonePolicy,
    IsTuple = true in {
defm : RVVUnitStridedSegStoreTuple<"vsseg">;
defm : RVVStridedSegStoreTuple<"vssseg">;
defm : RVVIndexedSegStoreTuple<"vsuxseg">;
defm : RVVIndexedSegStoreTuple<"vsoxseg">;
}

// 11. Vector Integer Arithmetic Instructions
````
- **L729 EN**: Declares TableGen def record `def`.
  **L729 CN**: 声明 TableGen def 记录 `def`。
- **L730 EN**: Closes the current lexical scope or compound statement.
  **L730 CN**: 结束当前词法作用域或复合语句块。
- **L731 EN**: Closes the current lexical scope or compound statement.
  **L731 CN**: 结束当前词法作用域或复合语句块。
- **L732 EN**: Closes the current lexical scope or compound statement.
  **L732 CN**: 结束当前词法作用域或复合语句块。
- **L733 EN**: Closes the current lexical scope or compound statement.
  **L733 CN**: 结束当前词法作用域或复合语句块。
- **L734 EN**: Closes the current lexical scope or compound statement.
  **L734 CN**: 结束当前词法作用域或复合语句块。
- **L735 EN**: Closes the current lexical scope or compound statement.
  **L735 CN**: 结束当前词法作用域或复合语句块。
- **L736 EN**: Blank line separating nearby declarations or logic blocks.
  **L736 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L737 EN**: Comment explains nearby logic, constraints, or intent: `7.8 Vector Load/Store Segment Instructions`.
  **L737 CN**: 注释解释附近代码的逻辑、约束或设计意图：`7.8 Vector Load/Store Segment Instructions`。
- **L738 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L738 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L739 EN**: Continues the surrounding expression or declaration: `IsTuple = true in {`.
  **L739 CN**: 继续构造周围的表达式或声明：`IsTuple = true in {`。
- **L740 EN**: Declares TableGen defm record `defm`.
  **L740 CN**: 声明 TableGen defm 记录 `defm`。
- **L741 EN**: Declares TableGen defm record `defm`.
  **L741 CN**: 声明 TableGen defm 记录 `defm`。
- **L742 EN**: Declares TableGen defm record `defm`.
  **L742 CN**: 声明 TableGen defm 记录 `defm`。
- **L743 EN**: Declares TableGen defm record `defm`.
  **L743 CN**: 声明 TableGen defm 记录 `defm`。
- **L744 EN**: Declares TableGen defm record `defm`.
  **L744 CN**: 声明 TableGen defm 记录 `defm`。
- **L745 EN**: Closes the current lexical scope or compound statement.
  **L745 CN**: 结束当前词法作用域或复合语句块。
- **L746 EN**: Blank line separating nearby declarations or logic blocks.
  **L746 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L747 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = NonePolicy,`.
  **L747 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = NonePolicy,`。
- **L748 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L748 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L749 EN**: Continues the surrounding expression or declaration: `IsTuple = true in {`.
  **L749 CN**: 继续构造周围的表达式或声明：`IsTuple = true in {`。
- **L750 EN**: Declares TableGen defm record `defm`.
  **L750 CN**: 声明 TableGen defm 记录 `defm`。
- **L751 EN**: Declares TableGen defm record `defm`.
  **L751 CN**: 声明 TableGen defm 记录 `defm`。
- **L752 EN**: Declares TableGen defm record `defm`.
  **L752 CN**: 声明 TableGen defm 记录 `defm`。
- **L753 EN**: Declares TableGen defm record `defm`.
  **L753 CN**: 声明 TableGen defm 记录 `defm`。
- **L754 EN**: Closes the current lexical scope or compound statement.
  **L754 CN**: 结束当前词法作用域或复合语句块。
- **L755 EN**: Blank line separating nearby declarations or logic blocks.
  **L755 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L756 EN**: Comment explains nearby logic, constraints, or intent: `11. Vector Integer Arithmetic Instructions`.
  **L756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11. Vector Integer Arithmetic Instructions`。

### Lines 757-784

````tablegen
// 11.1. Vector Single-Width Integer Add and Subtract
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vadd : RVVIntBinBuiltinSet;
defm vsub : RVVIntBinBuiltinSet;
defm vrsub : RVVOutOp1BuiltinSet<"vrsub", "csil",
                                 [["vx", "v", "vve"],
                                  ["vx", "Uv", "UvUvUe"]]>;
}
defm vneg_v : RVVPseudoUnaryBuiltin<"vrsub", "csil">;

// 11.2. Vector Widening Integer Add/Subtract
// Widening unsigned integer add/subtract, 2*SEW = SEW +/- SEW
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vwaddu : RVVUnsignedWidenBinBuiltinSet;
defm vwsubu : RVVUnsignedWidenBinBuiltinSet;
// Widening signed integer add/subtract, 2*SEW = SEW +/- SEW
defm vwadd : RVVSignedWidenBinBuiltinSet;
defm vwsub : RVVSignedWidenBinBuiltinSet;
// Widening unsigned integer add/subtract, 2*SEW = 2*SEW +/- SEW
defm vwaddu : RVVUnsignedWidenOp0BinBuiltinSet;
defm vwsubu : RVVUnsignedWidenOp0BinBuiltinSet;
// Widening signed integer add/subtract, 2*SEW = 2*SEW +/- SEW
defm vwadd : RVVSignedWidenOp0BinBuiltinSet;
defm vwsub : RVVSignedWidenOp0BinBuiltinSet;
}
defm vwcvtu_x_x_v : RVVPseudoVWCVTBuiltin<"vwaddu", "vwcvtu_x", "csi",
                                          [["Uw", "UwUv"]]>;
defm vwcvt_x_x_v : RVVPseudoVWCVTBuiltin<"vwadd", "vwcvt_x", "csi",
````
- **L757 EN**: Comment explains nearby logic, constraints, or intent: `11.1. Vector Single-Width Integer Add and Subtract`.
  **L757 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.1. Vector Single-Width Integer Add and Subtract`。
- **L758 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L758 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L759 EN**: Declares TableGen defm record `vadd`.
  **L759 CN**: 声明 TableGen defm 记录 `vadd`。
- **L760 EN**: Declares TableGen defm record `vsub`.
  **L760 CN**: 声明 TableGen defm 记录 `vsub`。
- **L761 EN**: Declares TableGen defm record `vrsub`.
  **L761 CN**: 声明 TableGen defm 记录 `vrsub`。
- **L762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vx", "v", "vve"],`.
  **L762 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vx", "v", "vve"],`。
- **L763 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvUe"]]>;`.
  **L763 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvUe"]]>;`。
- **L764 EN**: Closes the current lexical scope or compound statement.
  **L764 CN**: 结束当前词法作用域或复合语句块。
- **L765 EN**: Declares TableGen defm record `vneg_v`.
  **L765 CN**: 声明 TableGen defm 记录 `vneg_v`。
- **L766 EN**: Blank line separating nearby declarations or logic blocks.
  **L766 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L767 EN**: Comment explains nearby logic, constraints, or intent: `11.2. Vector Widening Integer Add/Subtract`.
  **L767 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.2. Vector Widening Integer Add/Subtract`。
- **L768 EN**: Comment explains nearby logic, constraints, or intent: `Widening unsigned integer add/subtract, 2*SEW SEW +/- SEW`.
  **L768 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening unsigned integer add/subtract, 2*SEW SEW +/- SEW`。
- **L769 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L769 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L770 EN**: Declares TableGen defm record `vwaddu`.
  **L770 CN**: 声明 TableGen defm 记录 `vwaddu`。
- **L771 EN**: Declares TableGen defm record `vwsubu`.
  **L771 CN**: 声明 TableGen defm 记录 `vwsubu`。
- **L772 EN**: Comment explains nearby logic, constraints, or intent: `Widening signed integer add/subtract, 2*SEW SEW +/- SEW`.
  **L772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening signed integer add/subtract, 2*SEW SEW +/- SEW`。
- **L773 EN**: Declares TableGen defm record `vwadd`.
  **L773 CN**: 声明 TableGen defm 记录 `vwadd`。
- **L774 EN**: Declares TableGen defm record `vwsub`.
  **L774 CN**: 声明 TableGen defm 记录 `vwsub`。
- **L775 EN**: Comment explains nearby logic, constraints, or intent: `Widening unsigned integer add/subtract, 2*SEW 2*SEW +/- SEW`.
  **L775 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening unsigned integer add/subtract, 2*SEW 2*SEW +/- SEW`。
- **L776 EN**: Declares TableGen defm record `vwaddu`.
  **L776 CN**: 声明 TableGen defm 记录 `vwaddu`。
- **L777 EN**: Declares TableGen defm record `vwsubu`.
  **L777 CN**: 声明 TableGen defm 记录 `vwsubu`。
- **L778 EN**: Comment explains nearby logic, constraints, or intent: `Widening signed integer add/subtract, 2*SEW 2*SEW +/- SEW`.
  **L778 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening signed integer add/subtract, 2*SEW 2*SEW +/- SEW`。
- **L779 EN**: Declares TableGen defm record `vwadd`.
  **L779 CN**: 声明 TableGen defm 记录 `vwadd`。
- **L780 EN**: Declares TableGen defm record `vwsub`.
  **L780 CN**: 声明 TableGen defm 记录 `vwsub`。
- **L781 EN**: Closes the current lexical scope or compound statement.
  **L781 CN**: 结束当前词法作用域或复合语句块。
- **L782 EN**: Declares TableGen defm record `vwcvtu_x_x_v`.
  **L782 CN**: 声明 TableGen defm 记录 `vwcvtu_x_x_v`。
- **L783 EN**: Adds a standalone statement or declaration: `[["Uw", "UwUv"]]>;`.
  **L783 CN**: 添加一条独立语句或声明：`[["Uw", "UwUv"]]>;`。
- **L784 EN**: Declares TableGen defm record `vwcvt_x_x_v`.
  **L784 CN**: 声明 TableGen defm 记录 `vwcvt_x_x_v`。

### Lines 785-812

````tablegen
                                         [["w", "wv"]]>;

// 11.3. Vector Integer Extension
let UnMaskedPolicyScheme = HasPassthruOperand in {
let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
  def vsext_vf2 : RVVIntExt<"vsext", "w", "wv", "csi">;
  def vzext_vf2 : RVVIntExt<"vzext", "Uw", "UwUv", "csi">;
}
let Log2LMUL = [-3, -2, -1, 0, 1] in {
  def vsext_vf4 : RVVIntExt<"vsext", "q", "qv", "cs">;
  def vzext_vf4 : RVVIntExt<"vzext", "Uq", "UqUv", "cs">;
}
let Log2LMUL = [-3, -2, -1, 0] in {
  def vsext_vf8 : RVVIntExt<"vsext", "o", "ov", "c">;
  def vzext_vf8 : RVVIntExt<"vzext", "Uo", "UoUv", "c">;
}
}

// 11.4. Vector Integer Add-with-Carry / Subtract-with-Borrow Instructions
let HasMasked = false, MaskedPolicyScheme = NonePolicy in {
  let UnMaskedPolicyScheme = HasPassthruOperand in {
    defm vadc : RVVCarryinBuiltinSet;
    defm vsbc : RVVCarryinBuiltinSet;
  }
  defm vmadc : RVVCarryOutInBuiltinSet<"vmadc_carry_in">;
  defm vmadc : RVVIntMaskOutBuiltinSet;
  defm vmsbc : RVVCarryOutInBuiltinSet<"vmsbc_borrow_in">;
  defm vmsbc : RVVIntMaskOutBuiltinSet;
````
- **L785 EN**: Adds a standalone statement or declaration: `[["w", "wv"]]>;`.
  **L785 CN**: 添加一条独立语句或声明：`[["w", "wv"]]>;`。
- **L786 EN**: Blank line separating nearby declarations or logic blocks.
  **L786 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L787 EN**: Comment explains nearby logic, constraints, or intent: `11.3. Vector Integer Extension`.
  **L787 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.3. Vector Integer Extension`。
- **L788 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L788 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L789 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L789 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L790 EN**: Declares TableGen def record `vsext_vf2`.
  **L790 CN**: 声明 TableGen def 记录 `vsext_vf2`。
- **L791 EN**: Declares TableGen def record `vzext_vf2`.
  **L791 CN**: 声明 TableGen def 记录 `vzext_vf2`。
- **L792 EN**: Closes the current lexical scope or compound statement.
  **L792 CN**: 结束当前词法作用域或复合语句块。
- **L793 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1] in {`.
  **L793 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1] in {`。
- **L794 EN**: Declares TableGen def record `vsext_vf4`.
  **L794 CN**: 声明 TableGen def 记录 `vsext_vf4`。
- **L795 EN**: Declares TableGen def record `vzext_vf4`.
  **L795 CN**: 声明 TableGen def 记录 `vzext_vf4`。
- **L796 EN**: Closes the current lexical scope or compound statement.
  **L796 CN**: 结束当前词法作用域或复合语句块。
- **L797 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0] in {`.
  **L797 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0] in {`。
- **L798 EN**: Declares TableGen def record `vsext_vf8`.
  **L798 CN**: 声明 TableGen def 记录 `vsext_vf8`。
- **L799 EN**: Declares TableGen def record `vzext_vf8`.
  **L799 CN**: 声明 TableGen def 记录 `vzext_vf8`。
- **L800 EN**: Closes the current lexical scope or compound statement.
  **L800 CN**: 结束当前词法作用域或复合语句块。
- **L801 EN**: Closes the current lexical scope or compound statement.
  **L801 CN**: 结束当前词法作用域或复合语句块。
- **L802 EN**: Blank line separating nearby declarations or logic blocks.
  **L802 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L803 EN**: Comment explains nearby logic, constraints, or intent: `11.4. Vector Integer Add-with-Carry / Subtract-with-Borrow Instructions`.
  **L803 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.4. Vector Integer Add-with-Carry / Subtract-with-Borrow Instructions`。
- **L804 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false, MaskedPolicyScheme = NonePolicy in {`.
  **L804 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false, MaskedPolicyScheme = NonePolicy in {`。
- **L805 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L805 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L806 EN**: Declares TableGen defm record `vadc`.
  **L806 CN**: 声明 TableGen defm 记录 `vadc`。
- **L807 EN**: Declares TableGen defm record `vsbc`.
  **L807 CN**: 声明 TableGen defm 记录 `vsbc`。
- **L808 EN**: Closes the current lexical scope or compound statement.
  **L808 CN**: 结束当前词法作用域或复合语句块。
- **L809 EN**: Declares TableGen defm record `vmadc`.
  **L809 CN**: 声明 TableGen defm 记录 `vmadc`。
- **L810 EN**: Declares TableGen defm record `vmadc`.
  **L810 CN**: 声明 TableGen defm 记录 `vmadc`。
- **L811 EN**: Declares TableGen defm record `vmsbc`.
  **L811 CN**: 声明 TableGen defm 记录 `vmsbc`。
- **L812 EN**: Declares TableGen defm record `vmsbc`.
  **L812 CN**: 声明 TableGen defm 记录 `vmsbc`。

### Lines 813-840

````tablegen
}

// 11.5. Vector Bitwise Logical Instructions
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vand : RVVIntBinBuiltinSet;
defm vxor : RVVIntBinBuiltinSet;
defm vor : RVVIntBinBuiltinSet;
}
defm vnot_v : RVVPseudoVNotBuiltin<"vxor", "csil">;

// 11.6. Vector Single-Width Shift Instructions
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vsll : RVVShiftBuiltinSet;
defm vsrl : RVVUnsignedShiftBuiltinSet;
defm vsra : RVVSignedShiftBuiltinSet;

// 11.7. Vector Narrowing Integer Right Shift Instructions
defm vnsrl : RVVUnsignedNShiftBuiltinSet;
defm vnsra : RVVSignedNShiftBuiltinSet;
}
defm vncvt_x_x_w : RVVPseudoVNCVTBuiltin<"vnsrl", "vncvt_x", "csi",
                                         [["v", "vw"],
                                          ["Uv", "UvUw"]]>;

// 11.8. Vector Integer Compare Instructions
let MaskedPolicyScheme = HasPassthruOperand,
    HasTailPolicy = false in {
defm vmseq : RVVIntMaskOutBuiltinSet;
````
- **L813 EN**: Closes the current lexical scope or compound statement.
  **L813 CN**: 结束当前词法作用域或复合语句块。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Comment explains nearby logic, constraints, or intent: `11.5. Vector Bitwise Logical Instructions`.
  **L815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.5. Vector Bitwise Logical Instructions`。
- **L816 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L816 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L817 EN**: Declares TableGen defm record `vand`.
  **L817 CN**: 声明 TableGen defm 记录 `vand`。
- **L818 EN**: Declares TableGen defm record `vxor`.
  **L818 CN**: 声明 TableGen defm 记录 `vxor`。
- **L819 EN**: Declares TableGen defm record `vor`.
  **L819 CN**: 声明 TableGen defm 记录 `vor`。
- **L820 EN**: Closes the current lexical scope or compound statement.
  **L820 CN**: 结束当前词法作用域或复合语句块。
- **L821 EN**: Declares TableGen defm record `vnot_v`.
  **L821 CN**: 声明 TableGen defm 记录 `vnot_v`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Comment explains nearby logic, constraints, or intent: `11.6. Vector Single-Width Shift Instructions`.
  **L823 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.6. Vector Single-Width Shift Instructions`。
- **L824 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L824 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L825 EN**: Declares TableGen defm record `vsll`.
  **L825 CN**: 声明 TableGen defm 记录 `vsll`。
- **L826 EN**: Declares TableGen defm record `vsrl`.
  **L826 CN**: 声明 TableGen defm 记录 `vsrl`。
- **L827 EN**: Declares TableGen defm record `vsra`.
  **L827 CN**: 声明 TableGen defm 记录 `vsra`。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Comment explains nearby logic, constraints, or intent: `11.7. Vector Narrowing Integer Right Shift Instructions`.
  **L829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.7. Vector Narrowing Integer Right Shift Instructions`。
- **L830 EN**: Declares TableGen defm record `vnsrl`.
  **L830 CN**: 声明 TableGen defm 记录 `vnsrl`。
- **L831 EN**: Declares TableGen defm record `vnsra`.
  **L831 CN**: 声明 TableGen defm 记录 `vnsra`。
- **L832 EN**: Closes the current lexical scope or compound statement.
  **L832 CN**: 结束当前词法作用域或复合语句块。
- **L833 EN**: Declares TableGen defm record `vncvt_x_x_w`.
  **L833 CN**: 声明 TableGen defm 记录 `vncvt_x_x_w`。
- **L834 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["v", "vw"],`.
  **L834 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["v", "vw"],`。
- **L835 EN**: Adds a standalone statement or declaration: `["Uv", "UvUw"]]>;`.
  **L835 CN**: 添加一条独立语句或声明：`["Uv", "UvUw"]]>;`。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L837 EN**: Comment explains nearby logic, constraints, or intent: `11.8. Vector Integer Compare Instructions`.
  **L837 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.8. Vector Integer Compare Instructions`。
- **L838 EN**: Assigns a TableGen property that affects following records or inherited fields: `let MaskedPolicyScheme = HasPassthruOperand,`.
  **L838 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let MaskedPolicyScheme = HasPassthruOperand,`。
- **L839 EN**: Continues the surrounding expression or declaration: `HasTailPolicy = false in {`.
  **L839 CN**: 继续构造周围的表达式或声明：`HasTailPolicy = false in {`。
- **L840 EN**: Declares TableGen defm record `vmseq`.
  **L840 CN**: 声明 TableGen defm 记录 `vmseq`。

### Lines 841-868

````tablegen
defm vmsne : RVVIntMaskOutBuiltinSet;
defm vmsltu : RVVUnsignedMaskOutBuiltinSet;
defm vmslt : RVVSignedMaskOutBuiltinSet;
defm vmsleu : RVVUnsignedMaskOutBuiltinSet;
defm vmsle : RVVSignedMaskOutBuiltinSet;
defm vmsgtu : RVVUnsignedMaskOutBuiltinSet;
defm vmsgt : RVVSignedMaskOutBuiltinSet;
defm vmsgeu : RVVUnsignedMaskOutBuiltinSet;
defm vmsge : RVVSignedMaskOutBuiltinSet;
}

// 11.9. Vector Integer Min/Max Instructions
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vminu : RVVUnsignedBinBuiltinSet;
defm vmin : RVVSignedBinBuiltinSet;
defm vmaxu : RVVUnsignedBinBuiltinSet;
defm vmax : RVVSignedBinBuiltinSet;

// 11.10. Vector Single-Width Integer Multiply Instructions
defm vmul : RVVIntBinBuiltinSet;
defm vmulh : RVVSignedBinBuiltinSet;
defm vmulhu : RVVUnsignedBinBuiltinSet;
defm vmulhsu : RVVOutOp1BuiltinSet<"vmulhsu", "csil",
                                   [["vv", "v", "vvUv"],
                                    ["vx", "v", "vvUe"]]>;

// 11.11. Vector Integer Divide Instructions
defm vdivu : RVVUnsignedBinBuiltinSet;
````
- **L841 EN**: Declares TableGen defm record `vmsne`.
  **L841 CN**: 声明 TableGen defm 记录 `vmsne`。
- **L842 EN**: Declares TableGen defm record `vmsltu`.
  **L842 CN**: 声明 TableGen defm 记录 `vmsltu`。
- **L843 EN**: Declares TableGen defm record `vmslt`.
  **L843 CN**: 声明 TableGen defm 记录 `vmslt`。
- **L844 EN**: Declares TableGen defm record `vmsleu`.
  **L844 CN**: 声明 TableGen defm 记录 `vmsleu`。
- **L845 EN**: Declares TableGen defm record `vmsle`.
  **L845 CN**: 声明 TableGen defm 记录 `vmsle`。
- **L846 EN**: Declares TableGen defm record `vmsgtu`.
  **L846 CN**: 声明 TableGen defm 记录 `vmsgtu`。
- **L847 EN**: Declares TableGen defm record `vmsgt`.
  **L847 CN**: 声明 TableGen defm 记录 `vmsgt`。
- **L848 EN**: Declares TableGen defm record `vmsgeu`.
  **L848 CN**: 声明 TableGen defm 记录 `vmsgeu`。
- **L849 EN**: Declares TableGen defm record `vmsge`.
  **L849 CN**: 声明 TableGen defm 记录 `vmsge`。
- **L850 EN**: Closes the current lexical scope or compound statement.
  **L850 CN**: 结束当前词法作用域或复合语句块。
- **L851 EN**: Blank line separating nearby declarations or logic blocks.
  **L851 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L852 EN**: Comment explains nearby logic, constraints, or intent: `11.9. Vector Integer Min/Max Instructions`.
  **L852 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.9. Vector Integer Min/Max Instructions`。
- **L853 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L853 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L854 EN**: Declares TableGen defm record `vminu`.
  **L854 CN**: 声明 TableGen defm 记录 `vminu`。
- **L855 EN**: Declares TableGen defm record `vmin`.
  **L855 CN**: 声明 TableGen defm 记录 `vmin`。
- **L856 EN**: Declares TableGen defm record `vmaxu`.
  **L856 CN**: 声明 TableGen defm 记录 `vmaxu`。
- **L857 EN**: Declares TableGen defm record `vmax`.
  **L857 CN**: 声明 TableGen defm 记录 `vmax`。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L859 EN**: Comment explains nearby logic, constraints, or intent: `11.10. Vector Single-Width Integer Multiply Instructions`.
  **L859 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.10. Vector Single-Width Integer Multiply Instructions`。
- **L860 EN**: Declares TableGen defm record `vmul`.
  **L860 CN**: 声明 TableGen defm 记录 `vmul`。
- **L861 EN**: Declares TableGen defm record `vmulh`.
  **L861 CN**: 声明 TableGen defm 记录 `vmulh`。
- **L862 EN**: Declares TableGen defm record `vmulhu`.
  **L862 CN**: 声明 TableGen defm 记录 `vmulhu`。
- **L863 EN**: Declares TableGen defm record `vmulhsu`.
  **L863 CN**: 声明 TableGen defm 记录 `vmulhsu`。
- **L864 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "v", "vvUv"],`.
  **L864 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "v", "vvUv"],`。
- **L865 EN**: Adds a standalone statement or declaration: `["vx", "v", "vvUe"]]>;`.
  **L865 CN**: 添加一条独立语句或声明：`["vx", "v", "vvUe"]]>;`。
- **L866 EN**: Blank line separating nearby declarations or logic blocks.
  **L866 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L867 EN**: Comment explains nearby logic, constraints, or intent: `11.11. Vector Integer Divide Instructions`.
  **L867 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.11. Vector Integer Divide Instructions`。
- **L868 EN**: Declares TableGen defm record `vdivu`.
  **L868 CN**: 声明 TableGen defm 记录 `vdivu`。

### Lines 869-896

````tablegen
defm vdiv : RVVSignedBinBuiltinSet;
defm vremu : RVVUnsignedBinBuiltinSet;
defm vrem : RVVSignedBinBuiltinSet;
}

// 11.12. Vector Widening Integer Multiply Instructions
let Log2LMUL = [-3, -2, -1, 0, 1, 2], UnMaskedPolicyScheme = HasPassthruOperand in {
defm vwmul : RVVOutOp0Op1BuiltinSet<"vwmul", "csi",
                                    [["vv", "w", "wvv"],
                                     ["vx", "w", "wve"]]>;
defm vwmulu : RVVOutOp0Op1BuiltinSet<"vwmulu", "csi",
                                     [["vv", "Uw", "UwUvUv"],
                                      ["vx", "Uw", "UwUvUe"]]>;
defm vwmulsu : RVVOutOp0Op1BuiltinSet<"vwmulsu", "csi",
                                      [["vv", "w", "wvUv"],
                                       ["vx", "w", "wvUe"]]>;
}

// 11.13. Vector Single-Width Integer Multiply-Add Instructions
let UnMaskedPolicyScheme = HasPolicyOperand in {
defm vmacc  : RVVIntTerBuiltinSet;
defm vnmsac : RVVIntTerBuiltinSet;
defm vmadd  : RVVIntTerBuiltinSet;
defm vnmsub : RVVIntTerBuiltinSet;

// 11.14. Vector Widening Integer Multiply-Add Instructions
let HasMaskedOffOperand = false,
    Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
````
- **L869 EN**: Declares TableGen defm record `vdiv`.
  **L869 CN**: 声明 TableGen defm 记录 `vdiv`。
- **L870 EN**: Declares TableGen defm record `vremu`.
  **L870 CN**: 声明 TableGen defm 记录 `vremu`。
- **L871 EN**: Declares TableGen defm record `vrem`.
  **L871 CN**: 声明 TableGen defm 记录 `vrem`。
- **L872 EN**: Closes the current lexical scope or compound statement.
  **L872 CN**: 结束当前词法作用域或复合语句块。
- **L873 EN**: Blank line separating nearby declarations or logic blocks.
  **L873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L874 EN**: Comment explains nearby logic, constraints, or intent: `11.12. Vector Widening Integer Multiply Instructions`.
  **L874 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.12. Vector Widening Integer Multiply Instructions`。
- **L875 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2], UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L875 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2], UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L876 EN**: Declares TableGen defm record `vwmul`.
  **L876 CN**: 声明 TableGen defm 记录 `vwmul`。
- **L877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvv"],`.
  **L877 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvv"],`。
- **L878 EN**: Adds a standalone statement or declaration: `["vx", "w", "wve"]]>;`.
  **L878 CN**: 添加一条独立语句或声明：`["vx", "w", "wve"]]>;`。
- **L879 EN**: Declares TableGen defm record `vwmulu`.
  **L879 CN**: 声明 TableGen defm 记录 `vwmulu`。
- **L880 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uw", "UwUvUv"],`.
  **L880 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uw", "UwUvUv"],`。
- **L881 EN**: Adds a standalone statement or declaration: `["vx", "Uw", "UwUvUe"]]>;`.
  **L881 CN**: 添加一条独立语句或声明：`["vx", "Uw", "UwUvUe"]]>;`。
- **L882 EN**: Declares TableGen defm record `vwmulsu`.
  **L882 CN**: 声明 TableGen defm 记录 `vwmulsu`。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvUv"],`.
  **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvUv"],`。
- **L884 EN**: Adds a standalone statement or declaration: `["vx", "w", "wvUe"]]>;`.
  **L884 CN**: 添加一条独立语句或声明：`["vx", "w", "wvUe"]]>;`。
- **L885 EN**: Closes the current lexical scope or compound statement.
  **L885 CN**: 结束当前词法作用域或复合语句块。
- **L886 EN**: Blank line separating nearby declarations or logic blocks.
  **L886 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L887 EN**: Comment explains nearby logic, constraints, or intent: `11.13. Vector Single-Width Integer Multiply-Add Instructions`.
  **L887 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.13. Vector Single-Width Integer Multiply-Add Instructions`。
- **L888 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPolicyOperand in {`.
  **L888 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPolicyOperand in {`。
- **L889 EN**: Declares TableGen defm record `vmacc`.
  **L889 CN**: 声明 TableGen defm 记录 `vmacc`。
- **L890 EN**: Declares TableGen defm record `vnmsac`.
  **L890 CN**: 声明 TableGen defm 记录 `vnmsac`。
- **L891 EN**: Declares TableGen defm record `vmadd`.
  **L891 CN**: 声明 TableGen defm 记录 `vmadd`。
- **L892 EN**: Declares TableGen defm record `vnmsub`.
  **L892 CN**: 声明 TableGen defm 记录 `vnmsub`。
- **L893 EN**: Blank line separating nearby declarations or logic blocks.
  **L893 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L894 EN**: Comment explains nearby logic, constraints, or intent: `11.14. Vector Widening Integer Multiply-Add Instructions`.
  **L894 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.14. Vector Widening Integer Multiply-Add Instructions`。
- **L895 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = false,`.
  **L895 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = false,`。
- **L896 EN**: Continues the surrounding expression or declaration: `Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L896 CN**: 继续构造周围的表达式或声明：`Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。

### Lines 897-924

````tablegen
defm vwmaccu : RVVOutOp1Op2BuiltinSet<"vwmaccu", "csi",
                                      [["vv", "Uw", "UwUwUvUv"],
                                       ["vx", "Uw", "UwUwUeUv"]]>;
defm vwmacc : RVVOutOp1Op2BuiltinSet<"vwmacc", "csi",
                                     [["vv", "w", "wwvv"],
                                      ["vx", "w", "wwev"]]>;
defm vwmaccsu : RVVOutOp1Op2BuiltinSet<"vwmaccsu", "csi",
                                       [["vv", "w", "wwvUv"],
                                        ["vx", "w", "wweUv"]]>;
defm vwmaccus : RVVOutOp1Op2BuiltinSet<"vwmaccus", "csi",
                                       [["vx", "w", "wwUev"]]>;
}
}

// 11.15. Vector Integer Merge Instructions
// C/C++ Operand: (mask, op1, op2, vl), Intrinsic: (passthru, op1, op2, mask, vl)
let HasMasked = false,
    UnMaskedPolicyScheme = HasPassthruOperand,
    MaskedPolicyScheme = NonePolicy,
    ManualCodegen = [{
      // insert poison passthru
      if (PolicyAttrs & RVV_VTA)
        Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
      IntrinsicTypes = {ResultType, Ops[2]->getType(), Ops.back()->getType()};
    }] in {
  defm vmerge : RVVOutOp1BuiltinSet<"vmerge", "csil",
                                    [["vvm", "v", "vvvm"],
                                     ["vxm", "v", "vvem"],
````
- **L897 EN**: Declares TableGen defm record `vwmaccu`.
  **L897 CN**: 声明 TableGen defm 记录 `vwmaccu`。
- **L898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uw", "UwUwUvUv"],`.
  **L898 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uw", "UwUwUvUv"],`。
- **L899 EN**: Adds a standalone statement or declaration: `["vx", "Uw", "UwUwUeUv"]]>;`.
  **L899 CN**: 添加一条独立语句或声明：`["vx", "Uw", "UwUwUeUv"]]>;`。
- **L900 EN**: Declares TableGen defm record `vwmacc`.
  **L900 CN**: 声明 TableGen defm 记录 `vwmacc`。
- **L901 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wwvv"],`.
  **L901 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wwvv"],`。
- **L902 EN**: Adds a standalone statement or declaration: `["vx", "w", "wwev"]]>;`.
  **L902 CN**: 添加一条独立语句或声明：`["vx", "w", "wwev"]]>;`。
- **L903 EN**: Declares TableGen defm record `vwmaccsu`.
  **L903 CN**: 声明 TableGen defm 记录 `vwmaccsu`。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wwvUv"],`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wwvUv"],`。
- **L905 EN**: Adds a standalone statement or declaration: `["vx", "w", "wweUv"]]>;`.
  **L905 CN**: 添加一条独立语句或声明：`["vx", "w", "wweUv"]]>;`。
- **L906 EN**: Declares TableGen defm record `vwmaccus`.
  **L906 CN**: 声明 TableGen defm 记录 `vwmaccus`。
- **L907 EN**: Adds a standalone statement or declaration: `[["vx", "w", "wwUev"]]>;`.
  **L907 CN**: 添加一条独立语句或声明：`[["vx", "w", "wwUev"]]>;`。
- **L908 EN**: Closes the current lexical scope or compound statement.
  **L908 CN**: 结束当前词法作用域或复合语句块。
- **L909 EN**: Closes the current lexical scope or compound statement.
  **L909 CN**: 结束当前词法作用域或复合语句块。
- **L910 EN**: Blank line separating nearby declarations or logic blocks.
  **L910 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L911 EN**: Comment explains nearby logic, constraints, or intent: `11.15. Vector Integer Merge Instructions`.
  **L911 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.15. Vector Integer Merge Instructions`。
- **L912 EN**: Comment explains nearby logic, constraints, or intent: `C/C++ Operand: (mask, op1, op2, vl), Intrinsic: (passthru, op1, op2, mask, vl)`.
  **L912 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C/C++ Operand: (mask, op1, op2, vl), Intrinsic: (passthru, op1, op2, mask, vl)`。
- **L913 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false,`.
  **L913 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false,`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L915 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L915 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L916 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L916 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L917 EN**: Comment explains nearby logic, constraints, or intent: `insert poison passthru`.
  **L917 CN**: 注释解释附近代码的逻辑、约束或设计意图：`insert poison passthru`。
- **L918 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L918 CN**: 开始 `if` 控制流语句并计算其条件。
- **L919 EN**: Executes a call or declaration centered on `Ops.insert`.
  **L919 CN**: 执行以 `Ops.insert` 为核心的调用或声明。
- **L920 EN**: Executes a call or declaration centered on `Ops[2]->getType`.
  **L920 CN**: 执行以 `Ops[2]->getType` 为核心的调用或声明。
- **L921 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L921 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L922 EN**: Declares TableGen defm record `vmerge`.
  **L922 CN**: 声明 TableGen defm 记录 `vmerge`。
- **L923 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vvm", "v", "vvvm"],`.
  **L923 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vvm", "v", "vvvm"],`。
- **L924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vxm", "v", "vvem"],`.
  **L924 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vxm", "v", "vvem"],`。

### Lines 925-952

````tablegen
                                     ["vvm", "Uv", "UvUvUvm"],
                                     ["vxm", "Uv", "UvUvUem"]]>;
}

// 11.16. Vector Integer Move Instructions
let HasMasked = false,
    UnMaskedPolicyScheme = HasPassthruOperand,
    MaskedPolicyScheme = NonePolicy,
    OverloadedName = "vmv_v" in {
    defm vmv_v : RVVOutBuiltinSet<"vmv_v_v", "csil",
                                   [["v", "Uv", "UvUv"]]>;
    defm vmv_v : RVVOutBuiltinSet<"vmv_v_v", "csilxfdy",
                                   [["v", "v", "vv"]]>;
  let SupportOverloading = false in
    defm vmv_v : RVVOutBuiltinSet<"vmv_v_x", "csil",
                                   [["x", "v", "ve"],
                                    ["x", "Uv", "UvUe"]]>;
}

// 12. Vector Fixed-Point Arithmetic Instructions
let HeaderCode =
[{
enum __RISCV_VXRM {
  __RISCV_VXRM_RNU = 0,
  __RISCV_VXRM_RNE = 1,
  __RISCV_VXRM_RDN = 2,
  __RISCV_VXRM_ROD = 3,
};
````
- **L925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["vvm", "Uv", "UvUvUvm"],`.
  **L925 CN**: 继续一个多行参数列表、初始化器或聚合项：`["vvm", "Uv", "UvUvUvm"],`。
- **L926 EN**: Adds a standalone statement or declaration: `["vxm", "Uv", "UvUvUem"]]>;`.
  **L926 CN**: 添加一条独立语句或声明：`["vxm", "Uv", "UvUvUem"]]>;`。
- **L927 EN**: Closes the current lexical scope or compound statement.
  **L927 CN**: 结束当前词法作用域或复合语句块。
- **L928 EN**: Blank line separating nearby declarations or logic blocks.
  **L928 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L929 EN**: Comment explains nearby logic, constraints, or intent: `11.16. Vector Integer Move Instructions`.
  **L929 CN**: 注释解释附近代码的逻辑、约束或设计意图：`11.16. Vector Integer Move Instructions`。
- **L930 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false,`.
  **L930 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false,`。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L933 EN**: Continues the surrounding expression or declaration: `OverloadedName = "vmv_v" in {`.
  **L933 CN**: 继续构造周围的表达式或声明：`OverloadedName = "vmv_v" in {`。
- **L934 EN**: Declares TableGen defm record `vmv_v`.
  **L934 CN**: 声明 TableGen defm 记录 `vmv_v`。
- **L935 EN**: Adds a standalone statement or declaration: `[["v", "Uv", "UvUv"]]>;`.
  **L935 CN**: 添加一条独立语句或声明：`[["v", "Uv", "UvUv"]]>;`。
- **L936 EN**: Declares TableGen defm record `vmv_v`.
  **L936 CN**: 声明 TableGen defm 记录 `vmv_v`。
- **L937 EN**: Adds a standalone statement or declaration: `[["v", "v", "vv"]]>;`.
  **L937 CN**: 添加一条独立语句或声明：`[["v", "v", "vv"]]>;`。
- **L938 EN**: Assigns a TableGen property that affects following records or inherited fields: `let SupportOverloading = false in`.
  **L938 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let SupportOverloading = false in`。
- **L939 EN**: Declares TableGen defm record `vmv_v`.
  **L939 CN**: 声明 TableGen defm 记录 `vmv_v`。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["x", "v", "ve"],`.
  **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["x", "v", "ve"],`。
- **L941 EN**: Adds a standalone statement or declaration: `["x", "Uv", "UvUe"]]>;`.
  **L941 CN**: 添加一条独立语句或声明：`["x", "Uv", "UvUe"]]>;`。
- **L942 EN**: Closes the current lexical scope or compound statement.
  **L942 CN**: 结束当前词法作用域或复合语句块。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Comment explains nearby logic, constraints, or intent: `12. Vector Fixed-Point Arithmetic Instructions`.
  **L944 CN**: 注释解释附近代码的逻辑、约束或设计意图：`12. Vector Fixed-Point Arithmetic Instructions`。
- **L945 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HeaderCode =`.
  **L945 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HeaderCode =`。
- **L946 EN**: Continues the surrounding expression or declaration: `[{`.
  **L946 CN**: 继续构造周围的表达式或声明：`[{`。
- **L947 EN**: Declares enum `__RISCV_VXRM`.
  **L947 CN**: 声明 enum `__RISCV_VXRM`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_VXRM_RNU = 0,`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_VXRM_RNU = 0,`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_VXRM_RNE = 1,`.
  **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_VXRM_RNE = 1,`。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_VXRM_RDN = 2,`.
  **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_VXRM_RDN = 2,`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_VXRM_ROD = 3,`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_VXRM_ROD = 3,`。
- **L952 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L952 CN**: 结束当前声明作用域，例如结构体、枚举或类。

### Lines 953-980

````tablegen
}] in
def vxrm_enum : RVVHeader;

// 12.1. Vector Single-Width Saturating Add and Subtract
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vsaddu : RVVUnsignedBinBuiltinSet;
defm vsadd : RVVSignedBinBuiltinSet;
defm vssubu : RVVUnsignedBinBuiltinSet;
defm vssub : RVVSignedBinBuiltinSet;

let ManualCodegen = [{
  {
    return emitRVVAveragingBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                   PolicyAttrs, IsMasked, SegInstSEW);
  }
}] in {
  // 12.2. Vector Single-Width Averaging Add and Subtract
  defm vaaddu : RVVUnsignedBinBuiltinSetRoundingMode;
  defm vaadd : RVVSignedBinBuiltinSetRoundingMode;
  defm vasubu : RVVUnsignedBinBuiltinSetRoundingMode;
  defm vasub : RVVSignedBinBuiltinSetRoundingMode;

  // 12.3. Vector Single-Width Fractional Multiply with Rounding and Saturation
  defm vsmul : RVVSignedBinBuiltinSetRoundingMode;

  // 12.4. Vector Single-Width Scaling Shift Instructions
  defm vssrl : RVVUnsignedShiftBuiltinSetRoundingMode;
  defm vssra : RVVSignedShiftBuiltinSetRoundingMode;
````
- **L953 EN**: Continues the surrounding expression or declaration: `}] in`.
  **L953 CN**: 继续构造周围的表达式或声明：`}] in`。
- **L954 EN**: Declares TableGen def record `vxrm_enum`.
  **L954 CN**: 声明 TableGen def 记录 `vxrm_enum`。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L956 EN**: Comment explains nearby logic, constraints, or intent: `12.1. Vector Single-Width Saturating Add and Subtract`.
  **L956 CN**: 注释解释附近代码的逻辑、约束或设计意图：`12.1. Vector Single-Width Saturating Add and Subtract`。
- **L957 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L957 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L958 EN**: Declares TableGen defm record `vsaddu`.
  **L958 CN**: 声明 TableGen defm 记录 `vsaddu`。
- **L959 EN**: Declares TableGen defm record `vsadd`.
  **L959 CN**: 声明 TableGen defm 记录 `vsadd`。
- **L960 EN**: Declares TableGen defm record `vssubu`.
  **L960 CN**: 声明 TableGen defm 记录 `vssubu`。
- **L961 EN**: Declares TableGen defm record `vssub`.
  **L961 CN**: 声明 TableGen defm 记录 `vssub`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L963 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。
- **L964 EN**: Opens a new lexical scope or compound statement.
  **L964 CN**: 打开一个新的词法作用域或复合语句块。
- **L965 EN**: Returns from the current function with `emitRVVAveragingBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L965 CN**: 以 `emitRVVAveragingBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L966 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L966 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L967 EN**: Closes the current lexical scope or compound statement.
  **L967 CN**: 结束当前词法作用域或复合语句块。
- **L968 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L968 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L969 EN**: Comment explains nearby logic, constraints, or intent: `12.2. Vector Single-Width Averaging Add and Subtract`.
  **L969 CN**: 注释解释附近代码的逻辑、约束或设计意图：`12.2. Vector Single-Width Averaging Add and Subtract`。
- **L970 EN**: Declares TableGen defm record `vaaddu`.
  **L970 CN**: 声明 TableGen defm 记录 `vaaddu`。
- **L971 EN**: Declares TableGen defm record `vaadd`.
  **L971 CN**: 声明 TableGen defm 记录 `vaadd`。
- **L972 EN**: Declares TableGen defm record `vasubu`.
  **L972 CN**: 声明 TableGen defm 记录 `vasubu`。
- **L973 EN**: Declares TableGen defm record `vasub`.
  **L973 CN**: 声明 TableGen defm 记录 `vasub`。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Comment explains nearby logic, constraints, or intent: `12.3. Vector Single-Width Fractional Multiply with Rounding and Saturation`.
  **L975 CN**: 注释解释附近代码的逻辑、约束或设计意图：`12.3. Vector Single-Width Fractional Multiply with Rounding and Saturation`。
- **L976 EN**: Declares TableGen defm record `vsmul`.
  **L976 CN**: 声明 TableGen defm 记录 `vsmul`。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L978 EN**: Comment explains nearby logic, constraints, or intent: `12.4. Vector Single-Width Scaling Shift Instructions`.
  **L978 CN**: 注释解释附近代码的逻辑、约束或设计意图：`12.4. Vector Single-Width Scaling Shift Instructions`。
- **L979 EN**: Declares TableGen defm record `vssrl`.
  **L979 CN**: 声明 TableGen defm 记录 `vssrl`。
- **L980 EN**: Declares TableGen defm record `vssra`.
  **L980 CN**: 声明 TableGen defm 记录 `vssra`。

### Lines 981-1008

````tablegen
}

let ManualCodegen = [{
  {
    return emitRVVNarrowingClipBuiltin(this, E, ReturnValue, ResultType, ID,
                                       Ops, PolicyAttrs, IsMasked, SegInstSEW);
  }
}] in {
  // 12.5. Vector Narrowing Fixed-Point Clip Instructions
  defm vnclipu : RVVUnsignedNShiftBuiltinSetRoundingMode;
  defm vnclip : RVVSignedNShiftBuiltinSetRoundingMode;
}
}

// 13. Vector Floating-Point Instructions
let HeaderCode =
[{
enum __RISCV_FRM {
  __RISCV_FRM_RNE = 0,
  __RISCV_FRM_RTZ = 1,
  __RISCV_FRM_RDN = 2,
  __RISCV_FRM_RUP = 3,
  __RISCV_FRM_RMM = 4,
};
}] in def frm_enum : RVVHeader;

let UnMaskedPolicyScheme = HasPassthruOperand in {
let ManualCodegen = [{
````
- **L981 EN**: Closes the current lexical scope or compound statement.
  **L981 CN**: 结束当前词法作用域或复合语句块。
- **L982 EN**: Blank line separating nearby declarations or logic blocks.
  **L982 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L983 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L983 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。
- **L984 EN**: Opens a new lexical scope or compound statement.
  **L984 CN**: 打开一个新的词法作用域或复合语句块。
- **L985 EN**: Returns from the current function with `emitRVVNarrowingClipBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L985 CN**: 以 `emitRVVNarrowingClipBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L986 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L986 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L987 EN**: Closes the current lexical scope or compound statement.
  **L987 CN**: 结束当前词法作用域或复合语句块。
- **L988 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L988 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L989 EN**: Comment explains nearby logic, constraints, or intent: `12.5. Vector Narrowing Fixed-Point Clip Instructions`.
  **L989 CN**: 注释解释附近代码的逻辑、约束或设计意图：`12.5. Vector Narrowing Fixed-Point Clip Instructions`。
- **L990 EN**: Declares TableGen defm record `vnclipu`.
  **L990 CN**: 声明 TableGen defm 记录 `vnclipu`。
- **L991 EN**: Declares TableGen defm record `vnclip`.
  **L991 CN**: 声明 TableGen defm 记录 `vnclip`。
- **L992 EN**: Closes the current lexical scope or compound statement.
  **L992 CN**: 结束当前词法作用域或复合语句块。
- **L993 EN**: Closes the current lexical scope or compound statement.
  **L993 CN**: 结束当前词法作用域或复合语句块。
- **L994 EN**: Blank line separating nearby declarations or logic blocks.
  **L994 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L995 EN**: Comment explains nearby logic, constraints, or intent: `13. Vector Floating-Point Instructions`.
  **L995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13. Vector Floating-Point Instructions`。
- **L996 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HeaderCode =`.
  **L996 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HeaderCode =`。
- **L997 EN**: Continues the surrounding expression or declaration: `[{`.
  **L997 CN**: 继续构造周围的表达式或声明：`[{`。
- **L998 EN**: Declares enum `__RISCV_FRM`.
  **L998 CN**: 声明 enum `__RISCV_FRM`。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_FRM_RNE = 0,`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_FRM_RNE = 0,`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_FRM_RTZ = 1,`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_FRM_RTZ = 1,`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_FRM_RDN = 2,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_FRM_RDN = 2,`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_FRM_RUP = 3,`.
  **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_FRM_RUP = 3,`。
- **L1003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `__RISCV_FRM_RMM = 4,`.
  **L1003 CN**: 继续一个多行参数列表、初始化器或聚合项：`__RISCV_FRM_RMM = 4,`。
- **L1004 EN**: Closes the current declaration scope such as a struct, enum, or class.
  **L1004 CN**: 结束当前声明作用域，例如结构体、枚举或类。
- **L1005 EN**: Adds a standalone statement or declaration: `}] in def frm_enum : RVVHeader;`.
  **L1005 CN**: 添加一条独立语句或声明：`}] in def frm_enum : RVVHeader;`。
- **L1006 EN**: Blank line separating nearby declarations or logic blocks.
  **L1006 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1007 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L1007 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L1008 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L1008 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。

### Lines 1009-1036

````tablegen
  {
    return emitRVVFloatingPointBuiltin(this, E, ReturnValue, ResultType, ID,
                                       Ops, PolicyAttrs, IsMasked, SegInstSEW);
  }
}] in {
  let HasFRMRoundModeOp = true in {
    // 13.2. Vector Single-Width Floating-Point Add/Subtract Instructions
    defm vfadd  : RVVFloatingBinBuiltinSetRoundingMode<HasBF=1>;
    defm vfsub  : RVVFloatingBinBuiltinSetRoundingMode<HasBF=1>;
    defm vfrsub : RVVFloatingBinVFBuiltinSetRoundingMode<HasBF=1>;

    // 13.3. Vector Widening Floating-Point Add/Subtract Instructions
    // Widening FP add/subtract, 2*SEW = 2*SEW +/- SEW
    defm vfwadd : RVVFloatingWidenOp0BinBuiltinSetRoundingMode;
    defm vfwsub : RVVFloatingWidenOp0BinBuiltinSetRoundingMode;

    // 13.4. Vector Single-Width Floating-Point Multiply/Divide Instructions
    defm vfmul  : RVVFloatingBinBuiltinSetRoundingMode<HasBF=1>;
    defm vfdiv  : RVVFloatingBinBuiltinSetRoundingMode;
    defm vfrdiv : RVVFloatingBinVFBuiltinSetRoundingMode;
  }
  // 13.2. Vector Single-Width Floating-Point Add/Subtract Instructions
  defm vfadd  : RVVFloatingBinBuiltinSet<HasBF=1>;
  defm vfsub  : RVVFloatingBinBuiltinSet<HasBF=1>;
  defm vfrsub : RVVFloatingBinVFBuiltinSet<HasBF=1>;

  // 13.3. Vector Widening Floating-Point Add/Subtract Instructions
  // Widening FP add/subtract, 2*SEW = 2*SEW +/- SEW
````
- **L1009 EN**: Opens a new lexical scope or compound statement.
  **L1009 CN**: 打开一个新的词法作用域或复合语句块。
- **L1010 EN**: Returns from the current function with `emitRVVFloatingPointBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L1010 CN**: 以 `emitRVVFloatingPointBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L1011 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1011 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1012 EN**: Closes the current lexical scope or compound statement.
  **L1012 CN**: 结束当前词法作用域或复合语句块。
- **L1013 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1013 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1014 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasFRMRoundModeOp = true in {`.
  **L1014 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasFRMRoundModeOp = true in {`。
- **L1015 EN**: Comment explains nearby logic, constraints, or intent: `13.2. Vector Single-Width Floating-Point Add/Subtract Instructions`.
  **L1015 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.2. Vector Single-Width Floating-Point Add/Subtract Instructions`。
- **L1016 EN**: Declares TableGen defm record `vfadd`.
  **L1016 CN**: 声明 TableGen defm 记录 `vfadd`。
- **L1017 EN**: Declares TableGen defm record `vfsub`.
  **L1017 CN**: 声明 TableGen defm 记录 `vfsub`。
- **L1018 EN**: Declares TableGen defm record `vfrsub`.
  **L1018 CN**: 声明 TableGen defm 记录 `vfrsub`。
- **L1019 EN**: Blank line separating nearby declarations or logic blocks.
  **L1019 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1020 EN**: Comment explains nearby logic, constraints, or intent: `13.3. Vector Widening Floating-Point Add/Subtract Instructions`.
  **L1020 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.3. Vector Widening Floating-Point Add/Subtract Instructions`。
- **L1021 EN**: Comment explains nearby logic, constraints, or intent: `Widening FP add/subtract, 2*SEW 2*SEW +/- SEW`.
  **L1021 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening FP add/subtract, 2*SEW 2*SEW +/- SEW`。
- **L1022 EN**: Declares TableGen defm record `vfwadd`.
  **L1022 CN**: 声明 TableGen defm 记录 `vfwadd`。
- **L1023 EN**: Declares TableGen defm record `vfwsub`.
  **L1023 CN**: 声明 TableGen defm 记录 `vfwsub`。
- **L1024 EN**: Blank line separating nearby declarations or logic blocks.
  **L1024 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1025 EN**: Comment explains nearby logic, constraints, or intent: `13.4. Vector Single-Width Floating-Point Multiply/Divide Instructions`.
  **L1025 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.4. Vector Single-Width Floating-Point Multiply/Divide Instructions`。
- **L1026 EN**: Declares TableGen defm record `vfmul`.
  **L1026 CN**: 声明 TableGen defm 记录 `vfmul`。
- **L1027 EN**: Declares TableGen defm record `vfdiv`.
  **L1027 CN**: 声明 TableGen defm 记录 `vfdiv`。
- **L1028 EN**: Declares TableGen defm record `vfrdiv`.
  **L1028 CN**: 声明 TableGen defm 记录 `vfrdiv`。
- **L1029 EN**: Closes the current lexical scope or compound statement.
  **L1029 CN**: 结束当前词法作用域或复合语句块。
- **L1030 EN**: Comment explains nearby logic, constraints, or intent: `13.2. Vector Single-Width Floating-Point Add/Subtract Instructions`.
  **L1030 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.2. Vector Single-Width Floating-Point Add/Subtract Instructions`。
- **L1031 EN**: Declares TableGen defm record `vfadd`.
  **L1031 CN**: 声明 TableGen defm 记录 `vfadd`。
- **L1032 EN**: Declares TableGen defm record `vfsub`.
  **L1032 CN**: 声明 TableGen defm 记录 `vfsub`。
- **L1033 EN**: Declares TableGen defm record `vfrsub`.
  **L1033 CN**: 声明 TableGen defm 记录 `vfrsub`。
- **L1034 EN**: Blank line separating nearby declarations or logic blocks.
  **L1034 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1035 EN**: Comment explains nearby logic, constraints, or intent: `13.3. Vector Widening Floating-Point Add/Subtract Instructions`.
  **L1035 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.3. Vector Widening Floating-Point Add/Subtract Instructions`。
- **L1036 EN**: Comment explains nearby logic, constraints, or intent: `Widening FP add/subtract, 2*SEW 2*SEW +/- SEW`.
  **L1036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening FP add/subtract, 2*SEW 2*SEW +/- SEW`。

### Lines 1037-1064

````tablegen
  defm vfwadd : RVVFloatingWidenOp0BinBuiltinSet;
  defm vfwsub : RVVFloatingWidenOp0BinBuiltinSet;

  // 13.4. Vector Single-Width Floating-Point Multiply/Divide Instructions
  defm vfmul  : RVVFloatingBinBuiltinSet<HasBF=1>;
  defm vfdiv  : RVVFloatingBinBuiltinSet;
  defm vfrdiv : RVVFloatingBinVFBuiltinSet;
}

let ManualCodegen = [{
  {
    return emitRVVWideningFloatingPointBuiltin(
        this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,
        SegInstSEW);
  }
}] in {
  let HasFRMRoundModeOp = true in {
    // 13.3. Vector Widening Floating-Point Add/Subtract Instructions
    // Widening FP add/subtract, 2*SEW = SEW +/- SEW
    defm vfwadd : RVVFloatingWidenBinBuiltinSetRoundingMode;
    defm vfwsub : RVVFloatingWidenBinBuiltinSetRoundingMode;

    // 13.5. Vector Widening Floating-Point Multiply
    let Log2LMUL = [-2, -1, 0, 1, 2] in {
      defm vfwmul : RVVOutOp0Op1BuiltinSet<"vfwmul", "f",
                                           [["vv", "w", "wvvu"],
                                            ["vf", "w", "wveu"]]>;
      let RequiredFeatures = ["zvfh"] in
````
- **L1037 EN**: Declares TableGen defm record `vfwadd`.
  **L1037 CN**: 声明 TableGen defm 记录 `vfwadd`。
- **L1038 EN**: Declares TableGen defm record `vfwsub`.
  **L1038 CN**: 声明 TableGen defm 记录 `vfwsub`。
- **L1039 EN**: Blank line separating nearby declarations or logic blocks.
  **L1039 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1040 EN**: Comment explains nearby logic, constraints, or intent: `13.4. Vector Single-Width Floating-Point Multiply/Divide Instructions`.
  **L1040 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.4. Vector Single-Width Floating-Point Multiply/Divide Instructions`。
- **L1041 EN**: Declares TableGen defm record `vfmul`.
  **L1041 CN**: 声明 TableGen defm 记录 `vfmul`。
- **L1042 EN**: Declares TableGen defm record `vfdiv`.
  **L1042 CN**: 声明 TableGen defm 记录 `vfdiv`。
- **L1043 EN**: Declares TableGen defm record `vfrdiv`.
  **L1043 CN**: 声明 TableGen defm 记录 `vfrdiv`。
- **L1044 EN**: Closes the current lexical scope or compound statement.
  **L1044 CN**: 结束当前词法作用域或复合语句块。
- **L1045 EN**: Blank line separating nearby declarations or logic blocks.
  **L1045 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1046 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L1046 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。
- **L1047 EN**: Opens a new lexical scope or compound statement.
  **L1047 CN**: 打开一个新的词法作用域或复合语句块。
- **L1048 EN**: Returns from the current function with `emitRVVWideningFloatingPointBuiltin(`.
  **L1048 CN**: 以 `emitRVVWideningFloatingPointBuiltin(` 从当前函数返回。
- **L1049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`.
  **L1049 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`。
- **L1050 EN**: Adds a standalone statement or declaration: `SegInstSEW);`.
  **L1050 CN**: 添加一条独立语句或声明：`SegInstSEW);`。
- **L1051 EN**: Closes the current lexical scope or compound statement.
  **L1051 CN**: 结束当前词法作用域或复合语句块。
- **L1052 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1052 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1053 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasFRMRoundModeOp = true in {`.
  **L1053 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasFRMRoundModeOp = true in {`。
- **L1054 EN**: Comment explains nearby logic, constraints, or intent: `13.3. Vector Widening Floating-Point Add/Subtract Instructions`.
  **L1054 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.3. Vector Widening Floating-Point Add/Subtract Instructions`。
- **L1055 EN**: Comment explains nearby logic, constraints, or intent: `Widening FP add/subtract, 2*SEW SEW +/- SEW`.
  **L1055 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening FP add/subtract, 2*SEW SEW +/- SEW`。
- **L1056 EN**: Declares TableGen defm record `vfwadd`.
  **L1056 CN**: 声明 TableGen defm 记录 `vfwadd`。
- **L1057 EN**: Declares TableGen defm record `vfwsub`.
  **L1057 CN**: 声明 TableGen defm 记录 `vfwsub`。
- **L1058 EN**: Blank line separating nearby declarations or logic blocks.
  **L1058 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1059 EN**: Comment explains nearby logic, constraints, or intent: `13.5. Vector Widening Floating-Point Multiply`.
  **L1059 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.5. Vector Widening Floating-Point Multiply`。
- **L1060 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-2, -1, 0, 1, 2] in {`.
  **L1060 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-2, -1, 0, 1, 2] in {`。
- **L1061 EN**: Declares TableGen defm record `vfwmul`.
  **L1061 CN**: 声明 TableGen defm 记录 `vfwmul`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvvu"],`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvvu"],`。
- **L1063 EN**: Adds a standalone statement or declaration: `["vf", "w", "wveu"]]>;`.
  **L1063 CN**: 添加一条独立语句或声明：`["vf", "w", "wveu"]]>;`。
- **L1064 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1064 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。

### Lines 1065-1092

````tablegen
        defm vfwmul : RVVOutOp0Op1BuiltinSet<"vfwmul", "x",
                                             [["vv", "w", "wvvu"],
                                              ["vf", "w", "wveu"]]>;
      let RequiredFeatures = ["zvfbfa"] in
        defm vfwmul : RVVOutOp0Op1BuiltinSet<"vfwmul", "y",
                                             [["vv", "vw", "wvvu"],
                                              ["vf", "vw", "wveu"]]>;
    }
  }
  // 13.3. Vector Widening Floating-Point Add/Subtract Instructions
  // Widening FP add/subtract, 2*SEW = SEW +/- SEW
  defm vfwadd : RVVFloatingWidenBinBuiltinSet;
  defm vfwsub : RVVFloatingWidenBinBuiltinSet;

  // 13.5. Vector Widening Floating-Point Multiply
  let Log2LMUL = [-2, -1, 0, 1, 2] in {
    defm vfwmul : RVVOutOp0Op1BuiltinSet<"vfwmul", "f",
                                         [["vv", "w", "wvv"],
                                          ["vf", "w", "wve"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm vfwmul : RVVOutOp0Op1BuiltinSet<"vfwmul", "x",
                                           [["vv", "w", "wvv"],
                                            ["vf", "w", "wve"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm vfwmul : RVVOutOp0Op1BuiltinSet<"vfwmul", "y",
                                           [["vv", "vw", "wvv"],
                                            ["vf", "vw", "wve"]]>;
  }
````
- **L1065 EN**: Declares TableGen defm record `vfwmul`.
  **L1065 CN**: 声明 TableGen defm 记录 `vfwmul`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvvu"],`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvvu"],`。
- **L1067 EN**: Adds a standalone statement or declaration: `["vf", "w", "wveu"]]>;`.
  **L1067 CN**: 添加一条独立语句或声明：`["vf", "w", "wveu"]]>;`。
- **L1068 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1068 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1069 EN**: Declares TableGen defm record `vfwmul`.
  **L1069 CN**: 声明 TableGen defm 记录 `vfwmul`。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vw", "wvvu"],`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vw", "wvvu"],`。
- **L1071 EN**: Adds a standalone statement or declaration: `["vf", "vw", "wveu"]]>;`.
  **L1071 CN**: 添加一条独立语句或声明：`["vf", "vw", "wveu"]]>;`。
- **L1072 EN**: Closes the current lexical scope or compound statement.
  **L1072 CN**: 结束当前词法作用域或复合语句块。
- **L1073 EN**: Closes the current lexical scope or compound statement.
  **L1073 CN**: 结束当前词法作用域或复合语句块。
- **L1074 EN**: Comment explains nearby logic, constraints, or intent: `13.3. Vector Widening Floating-Point Add/Subtract Instructions`.
  **L1074 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.3. Vector Widening Floating-Point Add/Subtract Instructions`。
- **L1075 EN**: Comment explains nearby logic, constraints, or intent: `Widening FP add/subtract, 2*SEW SEW +/- SEW`.
  **L1075 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Widening FP add/subtract, 2*SEW SEW +/- SEW`。
- **L1076 EN**: Declares TableGen defm record `vfwadd`.
  **L1076 CN**: 声明 TableGen defm 记录 `vfwadd`。
- **L1077 EN**: Declares TableGen defm record `vfwsub`.
  **L1077 CN**: 声明 TableGen defm 记录 `vfwsub`。
- **L1078 EN**: Blank line separating nearby declarations or logic blocks.
  **L1078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1079 EN**: Comment explains nearby logic, constraints, or intent: `13.5. Vector Widening Floating-Point Multiply`.
  **L1079 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.5. Vector Widening Floating-Point Multiply`。
- **L1080 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-2, -1, 0, 1, 2] in {`.
  **L1080 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-2, -1, 0, 1, 2] in {`。
- **L1081 EN**: Declares TableGen defm record `vfwmul`.
  **L1081 CN**: 声明 TableGen defm 记录 `vfwmul`。
- **L1082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvv"],`.
  **L1082 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvv"],`。
- **L1083 EN**: Adds a standalone statement or declaration: `["vf", "w", "wve"]]>;`.
  **L1083 CN**: 添加一条独立语句或声明：`["vf", "w", "wve"]]>;`。
- **L1084 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1084 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1085 EN**: Declares TableGen defm record `vfwmul`.
  **L1085 CN**: 声明 TableGen defm 记录 `vfwmul`。
- **L1086 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "w", "wvv"],`.
  **L1086 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "w", "wvv"],`。
- **L1087 EN**: Adds a standalone statement or declaration: `["vf", "w", "wve"]]>;`.
  **L1087 CN**: 添加一条独立语句或声明：`["vf", "w", "wve"]]>;`。
- **L1088 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1088 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1089 EN**: Declares TableGen defm record `vfwmul`.
  **L1089 CN**: 声明 TableGen defm 记录 `vfwmul`。
- **L1090 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "vw", "wvv"],`.
  **L1090 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "vw", "wvv"],`。
- **L1091 EN**: Adds a standalone statement or declaration: `["vf", "vw", "wve"]]>;`.
  **L1091 CN**: 添加一条独立语句或声明：`["vf", "vw", "wve"]]>;`。
- **L1092 EN**: Closes the current lexical scope or compound statement.
  **L1092 CN**: 结束当前词法作用域或复合语句块。

### Lines 1093-1120

````tablegen
}
}


let UnMaskedPolicyScheme = HasPolicyOperand in {
let ManualCodegen = [{
  {
    return emitRVVFMABuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                             PolicyAttrs, IsMasked, SegInstSEW);
  }
}] in {
  let HasFRMRoundModeOp = 1 in {
    // 13.6. Vector Single-Width Floating-Point Fused Multiply-Add Instructions
    defm vfmacc  : RVVFloatingTerBuiltinSetRoundingMode;
    defm vfnmacc : RVVFloatingTerBuiltinSetRoundingMode;
    defm vfmsac  : RVVFloatingTerBuiltinSetRoundingMode;
    defm vfnmsac : RVVFloatingTerBuiltinSetRoundingMode;
    defm vfmadd  : RVVFloatingTerBuiltinSetRoundingMode;
    defm vfnmadd : RVVFloatingTerBuiltinSetRoundingMode;
    defm vfmsub  : RVVFloatingTerBuiltinSetRoundingMode;
    defm vfnmsub : RVVFloatingTerBuiltinSetRoundingMode;
  }
  // 13.6. Vector Single-Width Floating-Point Fused Multiply-Add Instructions
  defm vfmacc  : RVVFloatingTerBuiltinSet;
  defm vfnmacc : RVVFloatingTerBuiltinSet;
  defm vfmsac  : RVVFloatingTerBuiltinSet;
  defm vfnmsac : RVVFloatingTerBuiltinSet;
  defm vfmadd  : RVVFloatingTerBuiltinSet;
````
- **L1093 EN**: Closes the current lexical scope or compound statement.
  **L1093 CN**: 结束当前词法作用域或复合语句块。
- **L1094 EN**: Closes the current lexical scope or compound statement.
  **L1094 CN**: 结束当前词法作用域或复合语句块。
- **L1095 EN**: Blank line separating nearby declarations or logic blocks.
  **L1095 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1096 EN**: Blank line separating nearby declarations or logic blocks.
  **L1096 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1097 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPolicyOperand in {`.
  **L1097 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPolicyOperand in {`。
- **L1098 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L1098 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。
- **L1099 EN**: Opens a new lexical scope or compound statement.
  **L1099 CN**: 打开一个新的词法作用域或复合语句块。
- **L1100 EN**: Returns from the current function with `emitRVVFMABuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L1100 CN**: 以 `emitRVVFMABuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L1101 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1101 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1102 EN**: Closes the current lexical scope or compound statement.
  **L1102 CN**: 结束当前词法作用域或复合语句块。
- **L1103 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1103 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1104 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasFRMRoundModeOp = 1 in {`.
  **L1104 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasFRMRoundModeOp = 1 in {`。
- **L1105 EN**: Comment explains nearby logic, constraints, or intent: `13.6. Vector Single-Width Floating-Point Fused Multiply-Add Instructions`.
  **L1105 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.6. Vector Single-Width Floating-Point Fused Multiply-Add Instructions`。
- **L1106 EN**: Declares TableGen defm record `vfmacc`.
  **L1106 CN**: 声明 TableGen defm 记录 `vfmacc`。
- **L1107 EN**: Declares TableGen defm record `vfnmacc`.
  **L1107 CN**: 声明 TableGen defm 记录 `vfnmacc`。
- **L1108 EN**: Declares TableGen defm record `vfmsac`.
  **L1108 CN**: 声明 TableGen defm 记录 `vfmsac`。
- **L1109 EN**: Declares TableGen defm record `vfnmsac`.
  **L1109 CN**: 声明 TableGen defm 记录 `vfnmsac`。
- **L1110 EN**: Declares TableGen defm record `vfmadd`.
  **L1110 CN**: 声明 TableGen defm 记录 `vfmadd`。
- **L1111 EN**: Declares TableGen defm record `vfnmadd`.
  **L1111 CN**: 声明 TableGen defm 记录 `vfnmadd`。
- **L1112 EN**: Declares TableGen defm record `vfmsub`.
  **L1112 CN**: 声明 TableGen defm 记录 `vfmsub`。
- **L1113 EN**: Declares TableGen defm record `vfnmsub`.
  **L1113 CN**: 声明 TableGen defm 记录 `vfnmsub`。
- **L1114 EN**: Closes the current lexical scope or compound statement.
  **L1114 CN**: 结束当前词法作用域或复合语句块。
- **L1115 EN**: Comment explains nearby logic, constraints, or intent: `13.6. Vector Single-Width Floating-Point Fused Multiply-Add Instructions`.
  **L1115 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.6. Vector Single-Width Floating-Point Fused Multiply-Add Instructions`。
- **L1116 EN**: Declares TableGen defm record `vfmacc`.
  **L1116 CN**: 声明 TableGen defm 记录 `vfmacc`。
- **L1117 EN**: Declares TableGen defm record `vfnmacc`.
  **L1117 CN**: 声明 TableGen defm 记录 `vfnmacc`。
- **L1118 EN**: Declares TableGen defm record `vfmsac`.
  **L1118 CN**: 声明 TableGen defm 记录 `vfmsac`。
- **L1119 EN**: Declares TableGen defm record `vfnmsac`.
  **L1119 CN**: 声明 TableGen defm 记录 `vfnmsac`。
- **L1120 EN**: Declares TableGen defm record `vfmadd`.
  **L1120 CN**: 声明 TableGen defm 记录 `vfmadd`。

### Lines 1121-1148

````tablegen
  defm vfnmadd : RVVFloatingTerBuiltinSet;
  defm vfmsub  : RVVFloatingTerBuiltinSet;
  defm vfnmsub : RVVFloatingTerBuiltinSet;
}

let ManualCodegen = [{
  {
    return emitRVVWideningFMABuiltin(this, E, ReturnValue, ResultType, ID,
                                     Ops, PolicyAttrs, IsMasked, SegInstSEW);
  }
}] in {
  let HasFRMRoundModeOp = 1 in {
    // 13.7. Vector Widening Floating-Point Fused Multiply-Add Instructions
    defm vfwmacc  : RVVFloatingWidenTerBuiltinSetRoundingMode;
    defm vfwnmacc : RVVFloatingWidenTerBuiltinSetRoundingMode;
    defm vfwmsac  : RVVFloatingWidenTerBuiltinSetRoundingMode;
    defm vfwnmsac : RVVFloatingWidenTerBuiltinSetRoundingMode;

    // Vector BF16 widening multiply-accumulate
    let Log2LMUL = [-2, -1, 0, 1, 2],
        RequiredFeatures = ["zvfbfwma"],
        HasMaskedOffOperand = false in
    defm vfwmaccbf16 : RVVOutOp1Op2BuiltinSet<"vfwmaccbf16", "y",
                                              [["vv", "Fw", "FwFwvvu"],
                                               ["vf", "Fw", "FwFwevu"]]>;
  }
  // 13.7. Vector Widening Floating-Point Fused Multiply-Add Instructions
  defm vfwmacc  : RVVFloatingWidenTerBuiltinSet;
````
- **L1121 EN**: Declares TableGen defm record `vfnmadd`.
  **L1121 CN**: 声明 TableGen defm 记录 `vfnmadd`。
- **L1122 EN**: Declares TableGen defm record `vfmsub`.
  **L1122 CN**: 声明 TableGen defm 记录 `vfmsub`。
- **L1123 EN**: Declares TableGen defm record `vfnmsub`.
  **L1123 CN**: 声明 TableGen defm 记录 `vfnmsub`。
- **L1124 EN**: Closes the current lexical scope or compound statement.
  **L1124 CN**: 结束当前词法作用域或复合语句块。
- **L1125 EN**: Blank line separating nearby declarations or logic blocks.
  **L1125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1126 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L1126 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。
- **L1127 EN**: Opens a new lexical scope or compound statement.
  **L1127 CN**: 打开一个新的词法作用域或复合语句块。
- **L1128 EN**: Returns from the current function with `emitRVVWideningFMABuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L1128 CN**: 以 `emitRVVWideningFMABuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L1129 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1129 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1130 EN**: Closes the current lexical scope or compound statement.
  **L1130 CN**: 结束当前词法作用域或复合语句块。
- **L1131 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1131 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1132 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasFRMRoundModeOp = 1 in {`.
  **L1132 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasFRMRoundModeOp = 1 in {`。
- **L1133 EN**: Comment explains nearby logic, constraints, or intent: `13.7. Vector Widening Floating-Point Fused Multiply-Add Instructions`.
  **L1133 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.7. Vector Widening Floating-Point Fused Multiply-Add Instructions`。
- **L1134 EN**: Declares TableGen defm record `vfwmacc`.
  **L1134 CN**: 声明 TableGen defm 记录 `vfwmacc`。
- **L1135 EN**: Declares TableGen defm record `vfwnmacc`.
  **L1135 CN**: 声明 TableGen defm 记录 `vfwnmacc`。
- **L1136 EN**: Declares TableGen defm record `vfwmsac`.
  **L1136 CN**: 声明 TableGen defm 记录 `vfwmsac`。
- **L1137 EN**: Declares TableGen defm record `vfwnmsac`.
  **L1137 CN**: 声明 TableGen defm 记录 `vfwnmsac`。
- **L1138 EN**: Blank line separating nearby declarations or logic blocks.
  **L1138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1139 EN**: Comment explains nearby logic, constraints, or intent: `Vector BF16 widening multiply-accumulate`.
  **L1139 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector BF16 widening multiply-accumulate`。
- **L1140 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-2, -1, 0, 1, 2],`.
  **L1140 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-2, -1, 0, 1, 2],`。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RequiredFeatures = ["zvfbfwma"],`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`RequiredFeatures = ["zvfbfwma"],`。
- **L1142 EN**: Continues the surrounding expression or declaration: `HasMaskedOffOperand = false in`.
  **L1142 CN**: 继续构造周围的表达式或声明：`HasMaskedOffOperand = false in`。
- **L1143 EN**: Declares TableGen defm record `vfwmaccbf16`.
  **L1143 CN**: 声明 TableGen defm 记录 `vfwmaccbf16`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Fw", "FwFwvvu"],`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Fw", "FwFwvvu"],`。
- **L1145 EN**: Adds a standalone statement or declaration: `["vf", "Fw", "FwFwevu"]]>;`.
  **L1145 CN**: 添加一条独立语句或声明：`["vf", "Fw", "FwFwevu"]]>;`。
- **L1146 EN**: Closes the current lexical scope or compound statement.
  **L1146 CN**: 结束当前词法作用域或复合语句块。
- **L1147 EN**: Comment explains nearby logic, constraints, or intent: `13.7. Vector Widening Floating-Point Fused Multiply-Add Instructions`.
  **L1147 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.7. Vector Widening Floating-Point Fused Multiply-Add Instructions`。
- **L1148 EN**: Declares TableGen defm record `vfwmacc`.
  **L1148 CN**: 声明 TableGen defm 记录 `vfwmacc`。

### Lines 1149-1176

````tablegen
  defm vfwnmacc : RVVFloatingWidenTerBuiltinSet;
  defm vfwmsac  : RVVFloatingWidenTerBuiltinSet;
  defm vfwnmsac : RVVFloatingWidenTerBuiltinSet;

  // Vector BF16 widening multiply-accumulate
  let Log2LMUL = [-2, -1, 0, 1, 2],
      RequiredFeatures = ["zvfbfwma"],
      HasMaskedOffOperand = false in
  defm vfwmaccbf16 : RVVOutOp1Op2BuiltinSet<"vfwmaccbf16", "y",
                                            [["vv", "Fw", "FwFwvv"],
                                             ["vf", "Fw", "FwFwev"]]>;
}

}

let UnMaskedPolicyScheme = HasPassthruOperand in {
let ManualCodegen = [{
  {
    return emitRVVFloatingUnaryBuiltin(this, E, ReturnValue, ResultType, ID,
                                       Ops, PolicyAttrs, IsMasked, SegInstSEW);
  }
}] in {
  let HasFRMRoundModeOp = 1 in {
    // 13.8. Vector Floating-Point Square-Root Instruction
    defm vfsqrt : RVVOutBuiltinSet<"vfsqrt", "fd", [["v", "v", "vvu"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm vfsqrt : RVVOutBuiltinSet<"vfsqrt", "x", [["v", "v", "vvu"]]>;

````
- **L1149 EN**: Declares TableGen defm record `vfwnmacc`.
  **L1149 CN**: 声明 TableGen defm 记录 `vfwnmacc`。
- **L1150 EN**: Declares TableGen defm record `vfwmsac`.
  **L1150 CN**: 声明 TableGen defm 记录 `vfwmsac`。
- **L1151 EN**: Declares TableGen defm record `vfwnmsac`.
  **L1151 CN**: 声明 TableGen defm 记录 `vfwnmsac`。
- **L1152 EN**: Blank line separating nearby declarations or logic blocks.
  **L1152 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1153 EN**: Comment explains nearby logic, constraints, or intent: `Vector BF16 widening multiply-accumulate`.
  **L1153 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector BF16 widening multiply-accumulate`。
- **L1154 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-2, -1, 0, 1, 2],`.
  **L1154 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-2, -1, 0, 1, 2],`。
- **L1155 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RequiredFeatures = ["zvfbfwma"],`.
  **L1155 CN**: 继续一个多行参数列表、初始化器或聚合项：`RequiredFeatures = ["zvfbfwma"],`。
- **L1156 EN**: Continues the surrounding expression or declaration: `HasMaskedOffOperand = false in`.
  **L1156 CN**: 继续构造周围的表达式或声明：`HasMaskedOffOperand = false in`。
- **L1157 EN**: Declares TableGen defm record `vfwmaccbf16`.
  **L1157 CN**: 声明 TableGen defm 记录 `vfwmaccbf16`。
- **L1158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Fw", "FwFwvv"],`.
  **L1158 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Fw", "FwFwvv"],`。
- **L1159 EN**: Adds a standalone statement or declaration: `["vf", "Fw", "FwFwev"]]>;`.
  **L1159 CN**: 添加一条独立语句或声明：`["vf", "Fw", "FwFwev"]]>;`。
- **L1160 EN**: Closes the current lexical scope or compound statement.
  **L1160 CN**: 结束当前词法作用域或复合语句块。
- **L1161 EN**: Blank line separating nearby declarations or logic blocks.
  **L1161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1162 EN**: Closes the current lexical scope or compound statement.
  **L1162 CN**: 结束当前词法作用域或复合语句块。
- **L1163 EN**: Blank line separating nearby declarations or logic blocks.
  **L1163 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1164 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L1164 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L1165 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L1165 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。
- **L1166 EN**: Opens a new lexical scope or compound statement.
  **L1166 CN**: 打开一个新的词法作用域或复合语句块。
- **L1167 EN**: Returns from the current function with `emitRVVFloatingUnaryBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L1167 CN**: 以 `emitRVVFloatingUnaryBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L1168 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1168 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1169 EN**: Closes the current lexical scope or compound statement.
  **L1169 CN**: 结束当前词法作用域或复合语句块。
- **L1170 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1170 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1171 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasFRMRoundModeOp = 1 in {`.
  **L1171 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasFRMRoundModeOp = 1 in {`。
- **L1172 EN**: Comment explains nearby logic, constraints, or intent: `13.8. Vector Floating-Point Square-Root Instruction`.
  **L1172 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.8. Vector Floating-Point Square-Root Instruction`。
- **L1173 EN**: Declares TableGen defm record `vfsqrt`.
  **L1173 CN**: 声明 TableGen defm 记录 `vfsqrt`。
- **L1174 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1174 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1175 EN**: Declares TableGen defm record `vfsqrt`.
  **L1175 CN**: 声明 TableGen defm 记录 `vfsqrt`。
- **L1176 EN**: Blank line separating nearby declarations or logic blocks.
  **L1176 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1177-1204

````tablegen
    // 13.10. Vector Floating-Point Reciprocal Estimate Instruction
    defm vfrec7 : RVVOutBuiltinSet<"vfrec7", "fd", [["v", "v", "vvu"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm vfrec7 : RVVOutBuiltinSet<"vfrec7", "x", [["v", "v", "vvu"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm vfrec7 : RVVOutBuiltinSet<"vfrec7", "y", [["v", "v", "vvu"]]>;
  }
  // 13.8. Vector Floating-Point Square-Root Instruction
  defm vfsqrt : RVVOutBuiltinSet<"vfsqrt", "fd", [["v", "v", "vv"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm vfsqrt : RVVOutBuiltinSet<"vfsqrt", "x", [["v", "v", "vv"]]>;

  // 13.10. Vector Floating-Point Reciprocal Estimate Instruction
  defm vfrec7 : RVVOutBuiltinSet<"vfrec7", "fd", [["v", "v", "vv"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm vfrec7 : RVVOutBuiltinSet<"vfrec7", "x", [["v", "v", "vv"]]>;
  let RequiredFeatures = ["zvfbfa"] in
    defm vfrec7 : RVVOutBuiltinSet<"vfrec7", "y", [["v", "v", "vv"]]>;
}

// 13.9. Vector Floating-Point Reciprocal Square-Root Estimate Instruction
defm vfrsqrt7 : RVVOutBuiltinSet<"vfrsqrt7", "fd", [["v", "v", "vv"]]>;
let RequiredFeatures = ["zvfh"] in
  defm vfrsqrt7 : RVVOutBuiltinSet<"vfrsqrt7", "x", [["v", "v", "vv"]]>;
let RequiredFeatures = ["zvfbfa"] in
  defm vfrsqrt7 : RVVOutBuiltinSet<"vfrsqrt7", "y", [["v", "v", "vv"]]>;


````
- **L1177 EN**: Comment explains nearby logic, constraints, or intent: `13.10. Vector Floating-Point Reciprocal Estimate Instruction`.
  **L1177 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.10. Vector Floating-Point Reciprocal Estimate Instruction`。
- **L1178 EN**: Declares TableGen defm record `vfrec7`.
  **L1178 CN**: 声明 TableGen defm 记录 `vfrec7`。
- **L1179 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1179 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1180 EN**: Declares TableGen defm record `vfrec7`.
  **L1180 CN**: 声明 TableGen defm 记录 `vfrec7`。
- **L1181 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1181 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1182 EN**: Declares TableGen defm record `vfrec7`.
  **L1182 CN**: 声明 TableGen defm 记录 `vfrec7`。
- **L1183 EN**: Closes the current lexical scope or compound statement.
  **L1183 CN**: 结束当前词法作用域或复合语句块。
- **L1184 EN**: Comment explains nearby logic, constraints, or intent: `13.8. Vector Floating-Point Square-Root Instruction`.
  **L1184 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.8. Vector Floating-Point Square-Root Instruction`。
- **L1185 EN**: Declares TableGen defm record `vfsqrt`.
  **L1185 CN**: 声明 TableGen defm 记录 `vfsqrt`。
- **L1186 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1186 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1187 EN**: Declares TableGen defm record `vfsqrt`.
  **L1187 CN**: 声明 TableGen defm 记录 `vfsqrt`。
- **L1188 EN**: Blank line separating nearby declarations or logic blocks.
  **L1188 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1189 EN**: Comment explains nearby logic, constraints, or intent: `13.10. Vector Floating-Point Reciprocal Estimate Instruction`.
  **L1189 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.10. Vector Floating-Point Reciprocal Estimate Instruction`。
- **L1190 EN**: Declares TableGen defm record `vfrec7`.
  **L1190 CN**: 声明 TableGen defm 记录 `vfrec7`。
- **L1191 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1191 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1192 EN**: Declares TableGen defm record `vfrec7`.
  **L1192 CN**: 声明 TableGen defm 记录 `vfrec7`。
- **L1193 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1193 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1194 EN**: Declares TableGen defm record `vfrec7`.
  **L1194 CN**: 声明 TableGen defm 记录 `vfrec7`。
- **L1195 EN**: Closes the current lexical scope or compound statement.
  **L1195 CN**: 结束当前词法作用域或复合语句块。
- **L1196 EN**: Blank line separating nearby declarations or logic blocks.
  **L1196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1197 EN**: Comment explains nearby logic, constraints, or intent: `13.9. Vector Floating-Point Reciprocal Square-Root Estimate Instruction`.
  **L1197 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.9. Vector Floating-Point Reciprocal Square-Root Estimate Instruction`。
- **L1198 EN**: Declares TableGen defm record `vfrsqrt7`.
  **L1198 CN**: 声明 TableGen defm 记录 `vfrsqrt7`。
- **L1199 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1199 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1200 EN**: Declares TableGen defm record `vfrsqrt7`.
  **L1200 CN**: 声明 TableGen defm 记录 `vfrsqrt7`。
- **L1201 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1201 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1202 EN**: Declares TableGen defm record `vfrsqrt7`.
  **L1202 CN**: 声明 TableGen defm 记录 `vfrsqrt7`。
- **L1203 EN**: Blank line separating nearby declarations or logic blocks.
  **L1203 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1204 EN**: Blank line separating nearby declarations or logic blocks.
  **L1204 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1205-1232

````tablegen
// 13.11. Vector Floating-Point MIN/MAX Instructions
defm vfmin : RVVFloatingBinBuiltinSet<HasBF=1>;
defm vfmax : RVVFloatingBinBuiltinSet<HasBF=1>;

// 13.12. Vector Floating-Point Sign-Injection Instructions
defm vfsgnj  : RVVFloatingBinBuiltinSet<HasBF=1>;
defm vfsgnjn : RVVFloatingBinBuiltinSet<HasBF=1>;
defm vfsgnjx : RVVFloatingBinBuiltinSet<HasBF=1>;
}
defm vfneg_v : RVVPseudoVFUnaryBuiltin<"vfsgnjn", "fd">;
let RequiredFeatures = ["zvfh"] in
  defm vfneg_v : RVVPseudoVFUnaryBuiltin<"vfsgnjn", "x">;
let RequiredFeatures = ["zvfbfa"] in
  defm vfneg_v : RVVPseudoVFUnaryBuiltin<"vfsgnjn", "y">;
defm vfabs_v : RVVPseudoVFUnaryBuiltin<"vfsgnjx", "fd">;
let RequiredFeatures = ["zvfh"] in
  defm vfabs_v : RVVPseudoVFUnaryBuiltin<"vfsgnjx", "x">;
let RequiredFeatures = ["zvfbfa"] in
  defm vfabs_v : RVVPseudoVFUnaryBuiltin<"vfsgnjx", "y">;

// 13.13. Vector Floating-Point Compare Instructions
let MaskedPolicyScheme = HasPassthruOperand,
    HasTailPolicy = false in {
defm vmfeq : RVVFloatingMaskOutBuiltinSet;
defm vmfne : RVVFloatingMaskOutBuiltinSet;
defm vmflt : RVVFloatingMaskOutBuiltinSet;
defm vmfle : RVVFloatingMaskOutBuiltinSet;
defm vmfgt : RVVFloatingMaskOutBuiltinSet;
````
- **L1205 EN**: Comment explains nearby logic, constraints, or intent: `13.11. Vector Floating-Point MIN/MAX Instructions`.
  **L1205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.11. Vector Floating-Point MIN/MAX Instructions`。
- **L1206 EN**: Declares TableGen defm record `vfmin`.
  **L1206 CN**: 声明 TableGen defm 记录 `vfmin`。
- **L1207 EN**: Declares TableGen defm record `vfmax`.
  **L1207 CN**: 声明 TableGen defm 记录 `vfmax`。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1209 EN**: Comment explains nearby logic, constraints, or intent: `13.12. Vector Floating-Point Sign-Injection Instructions`.
  **L1209 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.12. Vector Floating-Point Sign-Injection Instructions`。
- **L1210 EN**: Declares TableGen defm record `vfsgnj`.
  **L1210 CN**: 声明 TableGen defm 记录 `vfsgnj`。
- **L1211 EN**: Declares TableGen defm record `vfsgnjn`.
  **L1211 CN**: 声明 TableGen defm 记录 `vfsgnjn`。
- **L1212 EN**: Declares TableGen defm record `vfsgnjx`.
  **L1212 CN**: 声明 TableGen defm 记录 `vfsgnjx`。
- **L1213 EN**: Closes the current lexical scope or compound statement.
  **L1213 CN**: 结束当前词法作用域或复合语句块。
- **L1214 EN**: Declares TableGen defm record `vfneg_v`.
  **L1214 CN**: 声明 TableGen defm 记录 `vfneg_v`。
- **L1215 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1215 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1216 EN**: Declares TableGen defm record `vfneg_v`.
  **L1216 CN**: 声明 TableGen defm 记录 `vfneg_v`。
- **L1217 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1217 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1218 EN**: Declares TableGen defm record `vfneg_v`.
  **L1218 CN**: 声明 TableGen defm 记录 `vfneg_v`。
- **L1219 EN**: Declares TableGen defm record `vfabs_v`.
  **L1219 CN**: 声明 TableGen defm 记录 `vfabs_v`。
- **L1220 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1220 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1221 EN**: Declares TableGen defm record `vfabs_v`.
  **L1221 CN**: 声明 TableGen defm 记录 `vfabs_v`。
- **L1222 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1222 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1223 EN**: Declares TableGen defm record `vfabs_v`.
  **L1223 CN**: 声明 TableGen defm 记录 `vfabs_v`。
- **L1224 EN**: Blank line separating nearby declarations or logic blocks.
  **L1224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1225 EN**: Comment explains nearby logic, constraints, or intent: `13.13. Vector Floating-Point Compare Instructions`.
  **L1225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.13. Vector Floating-Point Compare Instructions`。
- **L1226 EN**: Assigns a TableGen property that affects following records or inherited fields: `let MaskedPolicyScheme = HasPassthruOperand,`.
  **L1226 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let MaskedPolicyScheme = HasPassthruOperand,`。
- **L1227 EN**: Continues the surrounding expression or declaration: `HasTailPolicy = false in {`.
  **L1227 CN**: 继续构造周围的表达式或声明：`HasTailPolicy = false in {`。
- **L1228 EN**: Declares TableGen defm record `vmfeq`.
  **L1228 CN**: 声明 TableGen defm 记录 `vmfeq`。
- **L1229 EN**: Declares TableGen defm record `vmfne`.
  **L1229 CN**: 声明 TableGen defm 记录 `vmfne`。
- **L1230 EN**: Declares TableGen defm record `vmflt`.
  **L1230 CN**: 声明 TableGen defm 记录 `vmflt`。
- **L1231 EN**: Declares TableGen defm record `vmfle`.
  **L1231 CN**: 声明 TableGen defm 记录 `vmfle`。
- **L1232 EN**: Declares TableGen defm record `vmfgt`.
  **L1232 CN**: 声明 TableGen defm 记录 `vmfgt`。

### Lines 1233-1260

````tablegen
defm vmfge : RVVFloatingMaskOutBuiltinSet;
}

// 13.14. Vector Floating-Point Classify Instruction
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vfclass : RVVOp0BuiltinSet<"vfclass", "fd", [["v", "Uv", "Uvv"]]>;
let RequiredFeatures = ["zvfh"] in
  defm vfclass : RVVOp0BuiltinSet<"vfclass", "x", [["v", "Uv", "Uvv"]]>;
let RequiredFeatures = ["zvfbfa"] in
  defm vfclass : RVVOp0BuiltinSet<"vfclass", "y", [["v", "vUv", "Uvv"]]>;
}

// 13.15. Vector Floating-Point Merge Instruction
// C/C++ Operand: (mask, op1, op2, vl), Builtin: (op1, op2, mask, vl)
let HasMasked = false,
    UnMaskedPolicyScheme = HasPassthruOperand,
    MaskedPolicyScheme = NonePolicy,
    ManualCodegen = [{
      // insert poison passthru
      if (PolicyAttrs & RVV_VTA)
        Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
      IntrinsicTypes = {ResultType, Ops[2]->getType(), Ops.back()->getType()};
    }] in {
  defm vmerge : RVVOutOp1BuiltinSet<"vmerge", "xfdy",
                                    [["vvm", "v", "vvvm"]]>;
  defm vfmerge : RVVOutOp1BuiltinSet<"vfmerge", "fd",
                                     [["vfm", "v", "vvem"]]>;
  let RequiredFeatures = ["zvfh"] in
````
- **L1233 EN**: Declares TableGen defm record `vmfge`.
  **L1233 CN**: 声明 TableGen defm 记录 `vmfge`。
- **L1234 EN**: Closes the current lexical scope or compound statement.
  **L1234 CN**: 结束当前词法作用域或复合语句块。
- **L1235 EN**: Blank line separating nearby declarations or logic blocks.
  **L1235 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1236 EN**: Comment explains nearby logic, constraints, or intent: `13.14. Vector Floating-Point Classify Instruction`.
  **L1236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.14. Vector Floating-Point Classify Instruction`。
- **L1237 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L1237 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L1238 EN**: Declares TableGen defm record `vfclass`.
  **L1238 CN**: 声明 TableGen defm 记录 `vfclass`。
- **L1239 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1239 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1240 EN**: Declares TableGen defm record `vfclass`.
  **L1240 CN**: 声明 TableGen defm 记录 `vfclass`。
- **L1241 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1241 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1242 EN**: Declares TableGen defm record `vfclass`.
  **L1242 CN**: 声明 TableGen defm 记录 `vfclass`。
- **L1243 EN**: Closes the current lexical scope or compound statement.
  **L1243 CN**: 结束当前词法作用域或复合语句块。
- **L1244 EN**: Blank line separating nearby declarations or logic blocks.
  **L1244 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1245 EN**: Comment explains nearby logic, constraints, or intent: `13.15. Vector Floating-Point Merge Instruction`.
  **L1245 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.15. Vector Floating-Point Merge Instruction`。
- **L1246 EN**: Comment explains nearby logic, constraints, or intent: `C/C++ Operand: (mask, op1, op2, vl), Builtin: (op1, op2, mask, vl)`.
  **L1246 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C/C++ Operand: (mask, op1, op2, vl), Builtin: (op1, op2, mask, vl)`。
- **L1247 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false,`.
  **L1247 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false,`。
- **L1248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L1248 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L1249 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L1249 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L1250 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1250 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1251 EN**: Comment explains nearby logic, constraints, or intent: `insert poison passthru`.
  **L1251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`insert poison passthru`。
- **L1252 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1252 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1253 EN**: Executes a call or declaration centered on `Ops.insert`.
  **L1253 CN**: 执行以 `Ops.insert` 为核心的调用或声明。
- **L1254 EN**: Executes a call or declaration centered on `Ops[2]->getType`.
  **L1254 CN**: 执行以 `Ops[2]->getType` 为核心的调用或声明。
- **L1255 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1255 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1256 EN**: Declares TableGen defm record `vmerge`.
  **L1256 CN**: 声明 TableGen defm 记录 `vmerge`。
- **L1257 EN**: Adds a standalone statement or declaration: `[["vvm", "v", "vvvm"]]>;`.
  **L1257 CN**: 添加一条独立语句或声明：`[["vvm", "v", "vvvm"]]>;`。
- **L1258 EN**: Declares TableGen defm record `vfmerge`.
  **L1258 CN**: 声明 TableGen defm 记录 `vfmerge`。
- **L1259 EN**: Adds a standalone statement or declaration: `[["vfm", "v", "vvem"]]>;`.
  **L1259 CN**: 添加一条独立语句或声明：`[["vfm", "v", "vvem"]]>;`。
- **L1260 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1260 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。

### Lines 1261-1288

````tablegen
    defm vfmerge : RVVOutOp1BuiltinSet<"vfmerge", "x",
                                       [["vfm", "v", "vvem"]]>;
  let RequiredFeatures = ["zvfbfa"] in
    defm vfmerge : RVVOutOp1BuiltinSet<"vfmerge", "y",
                                       [["vfm", "v", "vvem"]]>;
}

// 13.16. Vector Floating-Point Move Instruction
let HasMasked = false,
    UnMaskedPolicyScheme = HasPassthruOperand,
    SupportOverloading = false,
    MaskedPolicyScheme = NonePolicy,
    OverloadedName = "vfmv_v" in {
  defm vfmv_v : RVVOutBuiltinSet<"vfmv_v_f", "fd",
                                 [["f", "v", "ve"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm vfmv_v : RVVOutBuiltinSet<"vfmv_v_f", "x",
                                   [["f", "v", "ve"]]>;
  let RequiredFeatures = ["zvfbfa"] in
    defm vfmv_v : RVVOutBuiltinSet<"vfmv_v_f", "y",
                                   [["f", "v", "ve"]]>;
}

// 13.17. Single-Width Floating-Point/Integer Type-Convert Instructions
let UnMaskedPolicyScheme = HasPassthruOperand in {
let OverloadedName = "vfcvt_rtz_xu" in {
  defm : RVVConvBuiltinSet<"vfcvt_rtz_xu_f_v", "fd", [["Uv", "Uvv"]]>;
  let RequiredFeatures = ["zvfh"] in
````
- **L1261 EN**: Declares TableGen defm record `vfmerge`.
  **L1261 CN**: 声明 TableGen defm 记录 `vfmerge`。
- **L1262 EN**: Adds a standalone statement or declaration: `[["vfm", "v", "vvem"]]>;`.
  **L1262 CN**: 添加一条独立语句或声明：`[["vfm", "v", "vvem"]]>;`。
- **L1263 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1263 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1264 EN**: Declares TableGen defm record `vfmerge`.
  **L1264 CN**: 声明 TableGen defm 记录 `vfmerge`。
- **L1265 EN**: Adds a standalone statement or declaration: `[["vfm", "v", "vvem"]]>;`.
  **L1265 CN**: 添加一条独立语句或声明：`[["vfm", "v", "vvem"]]>;`。
- **L1266 EN**: Closes the current lexical scope or compound statement.
  **L1266 CN**: 结束当前词法作用域或复合语句块。
- **L1267 EN**: Blank line separating nearby declarations or logic blocks.
  **L1267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1268 EN**: Comment explains nearby logic, constraints, or intent: `13.16. Vector Floating-Point Move Instruction`.
  **L1268 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.16. Vector Floating-Point Move Instruction`。
- **L1269 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false,`.
  **L1269 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false,`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SupportOverloading = false,`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`SupportOverloading = false,`。
- **L1272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L1272 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L1273 EN**: Continues the surrounding expression or declaration: `OverloadedName = "vfmv_v" in {`.
  **L1273 CN**: 继续构造周围的表达式或声明：`OverloadedName = "vfmv_v" in {`。
- **L1274 EN**: Declares TableGen defm record `vfmv_v`.
  **L1274 CN**: 声明 TableGen defm 记录 `vfmv_v`。
- **L1275 EN**: Adds a standalone statement or declaration: `[["f", "v", "ve"]]>;`.
  **L1275 CN**: 添加一条独立语句或声明：`[["f", "v", "ve"]]>;`。
- **L1276 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1276 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1277 EN**: Declares TableGen defm record `vfmv_v`.
  **L1277 CN**: 声明 TableGen defm 记录 `vfmv_v`。
- **L1278 EN**: Adds a standalone statement or declaration: `[["f", "v", "ve"]]>;`.
  **L1278 CN**: 添加一条独立语句或声明：`[["f", "v", "ve"]]>;`。
- **L1279 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1279 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1280 EN**: Declares TableGen defm record `vfmv_v`.
  **L1280 CN**: 声明 TableGen defm 记录 `vfmv_v`。
- **L1281 EN**: Adds a standalone statement or declaration: `[["f", "v", "ve"]]>;`.
  **L1281 CN**: 添加一条独立语句或声明：`[["f", "v", "ve"]]>;`。
- **L1282 EN**: Closes the current lexical scope or compound statement.
  **L1282 CN**: 结束当前词法作用域或复合语句块。
- **L1283 EN**: Blank line separating nearby declarations or logic blocks.
  **L1283 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1284 EN**: Comment explains nearby logic, constraints, or intent: `13.17. Single-Width Floating-Point/Integer Type-Convert Instructions`.
  **L1284 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.17. Single-Width Floating-Point/Integer Type-Convert Instructions`。
- **L1285 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L1285 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L1286 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_rtz_xu" in {`.
  **L1286 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_rtz_xu" in {`。
- **L1287 EN**: Declares TableGen defm record `defm`.
  **L1287 CN**: 声明 TableGen defm 记录 `defm`。
- **L1288 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1288 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。

### Lines 1289-1316

````tablegen
    defm : RVVConvBuiltinSet<"vfcvt_rtz_xu_f_v", "x", [["Uv", "Uvv"]]>;
}
let OverloadedName = "vfcvt_rtz_x" in {
  defm : RVVConvBuiltinSet<"vfcvt_rtz_x_f_v", "fd", [["Iv", "Ivv"]]>;
  let RequiredFeatures = ["zvfh"] in
    defm : RVVConvBuiltinSet<"vfcvt_rtz_x_f_v", "x", [["Iv", "Ivv"]]>;
}

// 13.18. Widening Floating-Point/Integer Type-Convert Instructions
let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
  let OverloadedName = "vfwcvt_rtz_xu" in {
    defm : RVVConvBuiltinSet<"vfwcvt_rtz_xu_f_v", "f", [["Uw", "Uwv"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm : RVVConvBuiltinSet<"vfwcvt_rtz_xu_f_v", "x", [["Uw", "Uwv"]]>;
  }
  let OverloadedName = "vfwcvt_rtz_x" in {
    defm : RVVConvBuiltinSet<"vfwcvt_rtz_x_f_v", "f", [["Iw", "Iwv"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm : RVVConvBuiltinSet<"vfwcvt_rtz_x_f_v", "x", [["Iw", "Iwv"]]>;
  }
  let OverloadedName = "vfwcvt_f" in {
    defm : RVVConvBuiltinSet<"vfwcvt_f_xu_v", "si", [["Fw", "FwUv"]]>;
    defm : RVVConvBuiltinSet<"vfwcvt_f_x_v", "si", [["Fw", "Fwv"]]>;
    let RequiredFeatures = ["zvfh"] in {
      defm : RVVConvBuiltinSet<"vfwcvt_f_xu_v", "c", [["Fw", "FwUv"]]>;
      defm : RVVConvBuiltinSet<"vfwcvt_f_x_v", "c", [["Fw", "Fwv"]]>;
    }
  }
````
- **L1289 EN**: Declares TableGen defm record `defm`.
  **L1289 CN**: 声明 TableGen defm 记录 `defm`。
- **L1290 EN**: Closes the current lexical scope or compound statement.
  **L1290 CN**: 结束当前词法作用域或复合语句块。
- **L1291 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_rtz_x" in {`.
  **L1291 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_rtz_x" in {`。
- **L1292 EN**: Declares TableGen defm record `defm`.
  **L1292 CN**: 声明 TableGen defm 记录 `defm`。
- **L1293 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1293 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1294 EN**: Declares TableGen defm record `defm`.
  **L1294 CN**: 声明 TableGen defm 记录 `defm`。
- **L1295 EN**: Closes the current lexical scope or compound statement.
  **L1295 CN**: 结束当前词法作用域或复合语句块。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1297 EN**: Comment explains nearby logic, constraints, or intent: `13.18. Widening Floating-Point/Integer Type-Convert Instructions`.
  **L1297 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.18. Widening Floating-Point/Integer Type-Convert Instructions`。
- **L1298 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1298 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L1299 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_rtz_xu" in {`.
  **L1299 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_rtz_xu" in {`。
- **L1300 EN**: Declares TableGen defm record `defm`.
  **L1300 CN**: 声明 TableGen defm 记录 `defm`。
- **L1301 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1301 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1302 EN**: Declares TableGen defm record `defm`.
  **L1302 CN**: 声明 TableGen defm 记录 `defm`。
- **L1303 EN**: Closes the current lexical scope or compound statement.
  **L1303 CN**: 结束当前词法作用域或复合语句块。
- **L1304 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_rtz_x" in {`.
  **L1304 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_rtz_x" in {`。
- **L1305 EN**: Declares TableGen defm record `defm`.
  **L1305 CN**: 声明 TableGen defm 记录 `defm`。
- **L1306 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1306 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1307 EN**: Declares TableGen defm record `defm`.
  **L1307 CN**: 声明 TableGen defm 记录 `defm`。
- **L1308 EN**: Closes the current lexical scope or compound statement.
  **L1308 CN**: 结束当前词法作用域或复合语句块。
- **L1309 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_f" in {`.
  **L1309 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_f" in {`。
- **L1310 EN**: Declares TableGen defm record `defm`.
  **L1310 CN**: 声明 TableGen defm 记录 `defm`。
- **L1311 EN**: Declares TableGen defm record `defm`.
  **L1311 CN**: 声明 TableGen defm 记录 `defm`。
- **L1312 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in {`.
  **L1312 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in {`。
- **L1313 EN**: Declares TableGen defm record `defm`.
  **L1313 CN**: 声明 TableGen defm 记录 `defm`。
- **L1314 EN**: Declares TableGen defm record `defm`.
  **L1314 CN**: 声明 TableGen defm 记录 `defm`。
- **L1315 EN**: Closes the current lexical scope or compound statement.
  **L1315 CN**: 结束当前词法作用域或复合语句块。
- **L1316 EN**: Closes the current lexical scope or compound statement.
  **L1316 CN**: 结束当前词法作用域或复合语句块。

### Lines 1317-1344

````tablegen
  let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfwcvt_f_bf16" in {
    defm : RVVConvBuiltinSet<"vfwcvt_f_xu_v", "c", [["Yw", "YwUv"]]>;
    defm : RVVConvBuiltinSet<"vfwcvt_f_x_v", "c", [["Yw", "Ywv"]]>;
  }
  let OverloadedName = "vfwcvt_f" in {
    defm : RVVConvBuiltinSet<"vfwcvt_f_f_v", "f", [["w", "wv"]]>;
    let RequiredFeatures = ["zvfhmin"] in
      defm : RVVConvBuiltinSet<"vfwcvt_f_f_v", "x", [["w", "wv"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm : RVVConvBuiltinSet<"vfwcvt_f_f_v", "y", [["vw", "wv"]]>;
  }
}

// 13.19. Narrowing Floating-Point/Integer Type-Convert Instructions
let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
  let OverloadedName = "vfncvt_rtz_xu" in {
    defm : RVVConvBuiltinSet<"vfncvt_rtz_xu_f_w", "si", [["Uv", "UvFw"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm : RVVConvBuiltinSet<"vfncvt_rtz_xu_f_w", "c", [["Uv", "UvFw"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm : RVVConvBuiltinSet<"vfncvt_rtz_xu_f_w", "c", [["YwUv", "UvYw"]]>;
  }
  let OverloadedName = "vfncvt_rtz_x" in {
    defm : RVVConvBuiltinSet<"vfncvt_rtz_x_f_w", "si", [["Iv", "IvFw"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm : RVVConvBuiltinSet<"vfncvt_rtz_x_f_w", "c", [["Iv", "IvFw"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm : RVVConvBuiltinSet<"vfncvt_rtz_x_f_w", "c", [["YwIv", "IvYw"]]>;
````
- **L1317 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfwcvt_f_bf16" in {`.
  **L1317 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfwcvt_f_bf16" in {`。
- **L1318 EN**: Declares TableGen defm record `defm`.
  **L1318 CN**: 声明 TableGen defm 记录 `defm`。
- **L1319 EN**: Declares TableGen defm record `defm`.
  **L1319 CN**: 声明 TableGen defm 记录 `defm`。
- **L1320 EN**: Closes the current lexical scope or compound statement.
  **L1320 CN**: 结束当前词法作用域或复合语句块。
- **L1321 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_f" in {`.
  **L1321 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_f" in {`。
- **L1322 EN**: Declares TableGen defm record `defm`.
  **L1322 CN**: 声明 TableGen defm 记录 `defm`。
- **L1323 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfhmin"] in`.
  **L1323 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfhmin"] in`。
- **L1324 EN**: Declares TableGen defm record `defm`.
  **L1324 CN**: 声明 TableGen defm 记录 `defm`。
- **L1325 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1325 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1326 EN**: Declares TableGen defm record `defm`.
  **L1326 CN**: 声明 TableGen defm 记录 `defm`。
- **L1327 EN**: Closes the current lexical scope or compound statement.
  **L1327 CN**: 结束当前词法作用域或复合语句块。
- **L1328 EN**: Closes the current lexical scope or compound statement.
  **L1328 CN**: 结束当前词法作用域或复合语句块。
- **L1329 EN**: Blank line separating nearby declarations or logic blocks.
  **L1329 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1330 EN**: Comment explains nearby logic, constraints, or intent: `13.19. Narrowing Floating-Point/Integer Type-Convert Instructions`.
  **L1330 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.19. Narrowing Floating-Point/Integer Type-Convert Instructions`。
- **L1331 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1331 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L1332 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_rtz_xu" in {`.
  **L1332 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_rtz_xu" in {`。
- **L1333 EN**: Declares TableGen defm record `defm`.
  **L1333 CN**: 声明 TableGen defm 记录 `defm`。
- **L1334 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1334 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1335 EN**: Declares TableGen defm record `defm`.
  **L1335 CN**: 声明 TableGen defm 记录 `defm`。
- **L1336 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1336 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1337 EN**: Declares TableGen defm record `defm`.
  **L1337 CN**: 声明 TableGen defm 记录 `defm`。
- **L1338 EN**: Closes the current lexical scope or compound statement.
  **L1338 CN**: 结束当前词法作用域或复合语句块。
- **L1339 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_rtz_x" in {`.
  **L1339 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_rtz_x" in {`。
- **L1340 EN**: Declares TableGen defm record `defm`.
  **L1340 CN**: 声明 TableGen defm 记录 `defm`。
- **L1341 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1341 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1342 EN**: Declares TableGen defm record `defm`.
  **L1342 CN**: 声明 TableGen defm 记录 `defm`。
- **L1343 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1343 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1344 EN**: Declares TableGen defm record `defm`.
  **L1344 CN**: 声明 TableGen defm 记录 `defm`。

### Lines 1345-1372

````tablegen
  }
  let OverloadedName = "vfncvt_rod_f" in {
    defm : RVVConvBuiltinSet<"vfncvt_rod_f_f_w", "f", [["v", "vw"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm : RVVConvBuiltinSet<"vfncvt_rod_f_f_w", "x", [["v", "vw"]]>;
  }
  let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_rod_f_bf16" in
    defm : RVVConvBuiltinSet<"vfncvt_rod_f_f_w", "y", [["v", "vw"]]>;
}

// Zvfbfmin - Vector convert BF16 to FP32
let RequiredFeatures = ["zvfbfmin"], Log2LMUL = [-2, -1, 0, 1, 2] in
def vfwcvtbf16_f_f_v : RVVConvBuiltin<"Fw", "Fwv", "y", "vfwcvtbf16_f">;

// Zvfofp8min
let Log2LMUL = [-3, -2, -1, 0, 1, 2],
    RequiredFeatures = ["zvfofp8min"],
    UnMaskedPolicyScheme = HasPassthruOperand in {
  let OverloadedName = "vfwcvt_f_bf16" in
    defm : RVVConvBuiltinSet<"vfwcvt_f_f_v", "a", [["vw", "wv"]]>;
  let OverloadedName = "vfwcvt_f_bf16",
      IRName = "vfwcvt_f_f_v_alt",
      MaskedIRName = "vfwcvt_f_f_v_alt_mask",
      AltFmt = 1 in
    defm : RVVConvBuiltinSet<"vfwcvt_f_f_v", "b", [["vw", "wv"]]>;
}

let ManualCodegen = [{
````
- **L1345 EN**: Closes the current lexical scope or compound statement.
  **L1345 CN**: 结束当前词法作用域或复合语句块。
- **L1346 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_rod_f" in {`.
  **L1346 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_rod_f" in {`。
- **L1347 EN**: Declares TableGen defm record `defm`.
  **L1347 CN**: 声明 TableGen defm 记录 `defm`。
- **L1348 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1348 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1349 EN**: Declares TableGen defm record `defm`.
  **L1349 CN**: 声明 TableGen defm 记录 `defm`。
- **L1350 EN**: Closes the current lexical scope or compound statement.
  **L1350 CN**: 结束当前词法作用域或复合语句块。
- **L1351 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_rod_f_bf16" in`.
  **L1351 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_rod_f_bf16" in`。
- **L1352 EN**: Declares TableGen defm record `defm`.
  **L1352 CN**: 声明 TableGen defm 记录 `defm`。
- **L1353 EN**: Closes the current lexical scope or compound statement.
  **L1353 CN**: 结束当前词法作用域或复合语句块。
- **L1354 EN**: Blank line separating nearby declarations or logic blocks.
  **L1354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1355 EN**: Comment explains nearby logic, constraints, or intent: `Zvfbfmin - Vector convert BF16 to FP32`.
  **L1355 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvfbfmin - Vector convert BF16 to FP32`。
- **L1356 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfmin"], Log2LMUL = [-2, -1, 0, 1, 2] in`.
  **L1356 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfmin"], Log2LMUL = [-2, -1, 0, 1, 2] in`。
- **L1357 EN**: Declares TableGen def record `vfwcvtbf16_f_f_v`.
  **L1357 CN**: 声明 TableGen def 记录 `vfwcvtbf16_f_f_v`。
- **L1358 EN**: Blank line separating nearby declarations or logic blocks.
  **L1358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1359 EN**: Comment explains nearby logic, constraints, or intent: `Zvfofp8min`.
  **L1359 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvfofp8min`。
- **L1360 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2],`.
  **L1360 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2],`。
- **L1361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RequiredFeatures = ["zvfofp8min"],`.
  **L1361 CN**: 继续一个多行参数列表、初始化器或聚合项：`RequiredFeatures = ["zvfofp8min"],`。
- **L1362 EN**: Continues the surrounding expression or declaration: `UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L1362 CN**: 继续构造周围的表达式或声明：`UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L1363 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_f_bf16" in`.
  **L1363 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_f_bf16" in`。
- **L1364 EN**: Declares TableGen defm record `defm`.
  **L1364 CN**: 声明 TableGen defm 记录 `defm`。
- **L1365 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_f_bf16",`.
  **L1365 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_f_bf16",`。
- **L1366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfwcvt_f_f_v_alt",`.
  **L1366 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfwcvt_f_f_v_alt",`。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfwcvt_f_f_v_alt_mask",`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfwcvt_f_f_v_alt_mask",`。
- **L1368 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1368 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1369 EN**: Declares TableGen defm record `defm`.
  **L1369 CN**: 声明 TableGen defm 记录 `defm`。
- **L1370 EN**: Closes the current lexical scope or compound statement.
  **L1370 CN**: 结束当前词法作用域或复合语句块。
- **L1371 EN**: Blank line separating nearby declarations or logic blocks.
  **L1371 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1372 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L1372 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。

### Lines 1373-1400

````tablegen
  {
    return emitRVVFloatingConvBuiltin(this, E, ReturnValue, ResultType, ID,
                                      Ops, PolicyAttrs, IsMasked, SegInstSEW);
  }
}] in {
  let HasFRMRoundModeOp = 1 in {
    // 14.17. Single-Width Floating-Point/Integer Type-Convert Instructions
    let OverloadedName = "vfcvt_x" in {
      defm : RVVConvBuiltinSet<"vfcvt_x_f_v", "fd", [["Iv", "Ivvu"]]>;
      let RequiredFeatures = ["zvfh"] in
        defm : RVVConvBuiltinSet<"vfcvt_x_f_v", "x", [["Iv", "Ivvu"]]>;
    }
    let OverloadedName = "vfcvt_xu" in {
      defm : RVVConvBuiltinSet<"vfcvt_xu_f_v", "fd", [["Uv", "Uvvu"]]>;
      let RequiredFeatures = ["zvfh"] in
        defm : RVVConvBuiltinSet<"vfcvt_xu_f_v", "x", [["Uv", "Uvvu"]]>;
    }
    let OverloadedName = "vfcvt_f" in {
      defm : RVVConvBuiltinSet<"vfcvt_f_x_v", "fd", [["v", "vIvu"]]>;
      defm : RVVConvBuiltinSet<"vfcvt_f_xu_v", "fd", [["v", "vUvu"]]>;
      let RequiredFeatures = ["zvfh"] in {
        defm : RVVConvBuiltinSet<"vfcvt_f_x_v", "x", [["v", "vIvu"]]>;
        defm : RVVConvBuiltinSet<"vfcvt_f_xu_v", "x", [["v", "vUvu"]]>;
      }
    }

    // 13.18. Widening Floating-Point/Integer Type-Convert Instructions
    let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
````
- **L1373 EN**: Opens a new lexical scope or compound statement.
  **L1373 CN**: 打开一个新的词法作用域或复合语句块。
- **L1374 EN**: Returns from the current function with `emitRVVFloatingConvBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L1374 CN**: 以 `emitRVVFloatingConvBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L1375 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1375 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1376 EN**: Closes the current lexical scope or compound statement.
  **L1376 CN**: 结束当前词法作用域或复合语句块。
- **L1377 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1377 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1378 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasFRMRoundModeOp = 1 in {`.
  **L1378 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasFRMRoundModeOp = 1 in {`。
- **L1379 EN**: Comment explains nearby logic, constraints, or intent: `14.17. Single-Width Floating-Point/Integer Type-Convert Instructions`.
  **L1379 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.17. Single-Width Floating-Point/Integer Type-Convert Instructions`。
- **L1380 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_x" in {`.
  **L1380 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_x" in {`。
- **L1381 EN**: Declares TableGen defm record `defm`.
  **L1381 CN**: 声明 TableGen defm 记录 `defm`。
- **L1382 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1382 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1383 EN**: Declares TableGen defm record `defm`.
  **L1383 CN**: 声明 TableGen defm 记录 `defm`。
- **L1384 EN**: Closes the current lexical scope or compound statement.
  **L1384 CN**: 结束当前词法作用域或复合语句块。
- **L1385 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_xu" in {`.
  **L1385 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_xu" in {`。
- **L1386 EN**: Declares TableGen defm record `defm`.
  **L1386 CN**: 声明 TableGen defm 记录 `defm`。
- **L1387 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1387 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1388 EN**: Declares TableGen defm record `defm`.
  **L1388 CN**: 声明 TableGen defm 记录 `defm`。
- **L1389 EN**: Closes the current lexical scope or compound statement.
  **L1389 CN**: 结束当前词法作用域或复合语句块。
- **L1390 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_f" in {`.
  **L1390 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_f" in {`。
- **L1391 EN**: Declares TableGen defm record `defm`.
  **L1391 CN**: 声明 TableGen defm 记录 `defm`。
- **L1392 EN**: Declares TableGen defm record `defm`.
  **L1392 CN**: 声明 TableGen defm 记录 `defm`。
- **L1393 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in {`.
  **L1393 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in {`。
- **L1394 EN**: Declares TableGen defm record `defm`.
  **L1394 CN**: 声明 TableGen defm 记录 `defm`。
- **L1395 EN**: Declares TableGen defm record `defm`.
  **L1395 CN**: 声明 TableGen defm 记录 `defm`。
- **L1396 EN**: Closes the current lexical scope or compound statement.
  **L1396 CN**: 结束当前词法作用域或复合语句块。
- **L1397 EN**: Closes the current lexical scope or compound statement.
  **L1397 CN**: 结束当前词法作用域或复合语句块。
- **L1398 EN**: Blank line separating nearby declarations or logic blocks.
  **L1398 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1399 EN**: Comment explains nearby logic, constraints, or intent: `13.18. Widening Floating-Point/Integer Type-Convert Instructions`.
  **L1399 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.18. Widening Floating-Point/Integer Type-Convert Instructions`。
- **L1400 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1400 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。

### Lines 1401-1428

````tablegen
      let OverloadedName = "vfwcvt_x" in {
        defm : RVVConvBuiltinSet<"vfwcvt_x_f_v", "f", [["Iw", "Iwvu"]]>;
        let RequiredFeatures = ["zvfh"] in
          defm : RVVConvBuiltinSet<"vfwcvt_x_f_v", "x", [["Iw", "Iwvu"]]>;
      }
      let OverloadedName = "vfwcvt_xu" in {
        defm : RVVConvBuiltinSet<"vfwcvt_xu_f_v", "f", [["Uw", "Uwvu"]]>;
        let RequiredFeatures = ["zvfh"] in
          defm : RVVConvBuiltinSet<"vfwcvt_xu_f_v", "x", [["Uw", "Uwvu"]]>;
      }
    }
    // 13.19. Narrowing Floating-Point/Integer Type-Convert Instructions
    let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
      let OverloadedName = "vfncvt_x" in {
        defm : RVVConvBuiltinSet<"vfncvt_x_f_w", "si", [["Iv", "IvFwu"]]>;
        let RequiredFeatures = ["zvfh"] in
          defm : RVVConvBuiltinSet<"vfncvt_x_f_w", "c", [["Iv", "IvFwu"]]>;
        let RequiredFeatures = ["zvfbfa"] in
          defm : RVVConvBuiltinSet<"vfncvt_x_f_w", "c", [["YwIv", "IvYwu"]]>;
      }
      let OverloadedName = "vfncvt_xu" in {
        defm : RVVConvBuiltinSet<"vfncvt_xu_f_w", "si", [["Uv", "UvFwu"]]>;
        let RequiredFeatures = ["zvfh"] in
          defm : RVVConvBuiltinSet<"vfncvt_xu_f_w", "c", [["Uv", "UvFwu"]]>;
        let RequiredFeatures = ["zvfbfa"] in
          defm : RVVConvBuiltinSet<"vfncvt_xu_f_w", "c", [["YwUv", "UvYwu"]]>;
      }
      let OverloadedName = "vfncvt_f" in {
````
- **L1401 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_x" in {`.
  **L1401 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_x" in {`。
- **L1402 EN**: Declares TableGen defm record `defm`.
  **L1402 CN**: 声明 TableGen defm 记录 `defm`。
- **L1403 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1403 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1404 EN**: Declares TableGen defm record `defm`.
  **L1404 CN**: 声明 TableGen defm 记录 `defm`。
- **L1405 EN**: Closes the current lexical scope or compound statement.
  **L1405 CN**: 结束当前词法作用域或复合语句块。
- **L1406 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_xu" in {`.
  **L1406 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_xu" in {`。
- **L1407 EN**: Declares TableGen defm record `defm`.
  **L1407 CN**: 声明 TableGen defm 记录 `defm`。
- **L1408 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1408 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1409 EN**: Declares TableGen defm record `defm`.
  **L1409 CN**: 声明 TableGen defm 记录 `defm`。
- **L1410 EN**: Closes the current lexical scope or compound statement.
  **L1410 CN**: 结束当前词法作用域或复合语句块。
- **L1411 EN**: Closes the current lexical scope or compound statement.
  **L1411 CN**: 结束当前词法作用域或复合语句块。
- **L1412 EN**: Comment explains nearby logic, constraints, or intent: `13.19. Narrowing Floating-Point/Integer Type-Convert Instructions`.
  **L1412 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.19. Narrowing Floating-Point/Integer Type-Convert Instructions`。
- **L1413 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1413 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L1414 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_x" in {`.
  **L1414 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_x" in {`。
- **L1415 EN**: Declares TableGen defm record `defm`.
  **L1415 CN**: 声明 TableGen defm 记录 `defm`。
- **L1416 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1416 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1417 EN**: Declares TableGen defm record `defm`.
  **L1417 CN**: 声明 TableGen defm 记录 `defm`。
- **L1418 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1418 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1419 EN**: Declares TableGen defm record `defm`.
  **L1419 CN**: 声明 TableGen defm 记录 `defm`。
- **L1420 EN**: Closes the current lexical scope or compound statement.
  **L1420 CN**: 结束当前词法作用域或复合语句块。
- **L1421 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_xu" in {`.
  **L1421 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_xu" in {`。
- **L1422 EN**: Declares TableGen defm record `defm`.
  **L1422 CN**: 声明 TableGen defm 记录 `defm`。
- **L1423 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1423 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1424 EN**: Declares TableGen defm record `defm`.
  **L1424 CN**: 声明 TableGen defm 记录 `defm`。
- **L1425 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1425 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1426 EN**: Declares TableGen defm record `defm`.
  **L1426 CN**: 声明 TableGen defm 记录 `defm`。
- **L1427 EN**: Closes the current lexical scope or compound statement.
  **L1427 CN**: 结束当前词法作用域或复合语句块。
- **L1428 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f" in {`.
  **L1428 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f" in {`。

### Lines 1429-1456

````tablegen
        defm : RVVConvBuiltinSet<"vfncvt_f_x_w", "f", [["v", "vIwu"]]>;
        defm : RVVConvBuiltinSet<"vfncvt_f_xu_w", "f", [["v", "vUwu"]]>;
        let RequiredFeatures = ["zvfh"] in {
          defm : RVVConvBuiltinSet<"vfncvt_f_x_w", "x", [["v", "vIwu"]]>;
          defm : RVVConvBuiltinSet<"vfncvt_f_xu_w", "x", [["v", "vUwu"]]>;
        }
      }
      let OverloadedName = "vfncvt_f" in {
        defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "f", [["v", "vwu"]]>;
        let RequiredFeatures = ["zvfhmin"] in
          defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "x", [["v", "vwu"]]>;
      }
      let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_f_bf16" in
        defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "y", [["v", "vwu"]]>;
    }

    // Zvfbfmin - Vector convert FP32 to BF16
    let RequiredFeatures = ["zvfbfmin"],
        Log2LMUL = [-2, -1, 0, 1, 2],
        OverloadedName = "vfncvtbf16_f" in
    defm : RVVConvBuiltinSet<"vfncvtbf16_f_f_w", "y", [["v", "vFwu"]]>;

    // Zvfofp8min
    let RequiredFeatures = ["zvfofp8min"] in {
      let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
        let OverloadedName = "vfncvt_f_f8e4m3" in
          defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "a", [["wv", "vwu"]]>;
        let OverloadedName = "vfncvt_f_f8e5m2",
````
- **L1429 EN**: Declares TableGen defm record `defm`.
  **L1429 CN**: 声明 TableGen defm 记录 `defm`。
- **L1430 EN**: Declares TableGen defm record `defm`.
  **L1430 CN**: 声明 TableGen defm 记录 `defm`。
- **L1431 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in {`.
  **L1431 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in {`。
- **L1432 EN**: Declares TableGen defm record `defm`.
  **L1432 CN**: 声明 TableGen defm 记录 `defm`。
- **L1433 EN**: Declares TableGen defm record `defm`.
  **L1433 CN**: 声明 TableGen defm 记录 `defm`。
- **L1434 EN**: Closes the current lexical scope or compound statement.
  **L1434 CN**: 结束当前词法作用域或复合语句块。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f" in {`.
  **L1436 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f" in {`。
- **L1437 EN**: Declares TableGen defm record `defm`.
  **L1437 CN**: 声明 TableGen defm 记录 `defm`。
- **L1438 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfhmin"] in`.
  **L1438 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfhmin"] in`。
- **L1439 EN**: Declares TableGen defm record `defm`.
  **L1439 CN**: 声明 TableGen defm 记录 `defm`。
- **L1440 EN**: Closes the current lexical scope or compound statement.
  **L1440 CN**: 结束当前词法作用域或复合语句块。
- **L1441 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_f_bf16" in`.
  **L1441 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_f_bf16" in`。
- **L1442 EN**: Declares TableGen defm record `defm`.
  **L1442 CN**: 声明 TableGen defm 记录 `defm`。
- **L1443 EN**: Closes the current lexical scope or compound statement.
  **L1443 CN**: 结束当前词法作用域或复合语句块。
- **L1444 EN**: Blank line separating nearby declarations or logic blocks.
  **L1444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1445 EN**: Comment explains nearby logic, constraints, or intent: `Zvfbfmin - Vector convert FP32 to BF16`.
  **L1445 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvfbfmin - Vector convert FP32 to BF16`。
- **L1446 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfmin"],`.
  **L1446 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfmin"],`。
- **L1447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Log2LMUL = [-2, -1, 0, 1, 2],`.
  **L1447 CN**: 继续一个多行参数列表、初始化器或聚合项：`Log2LMUL = [-2, -1, 0, 1, 2],`。
- **L1448 EN**: Continues the surrounding expression or declaration: `OverloadedName = "vfncvtbf16_f" in`.
  **L1448 CN**: 继续构造周围的表达式或声明：`OverloadedName = "vfncvtbf16_f" in`。
- **L1449 EN**: Declares TableGen defm record `defm`.
  **L1449 CN**: 声明 TableGen defm 记录 `defm`。
- **L1450 EN**: Blank line separating nearby declarations or logic blocks.
  **L1450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1451 EN**: Comment explains nearby logic, constraints, or intent: `Zvfofp8min`.
  **L1451 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvfofp8min`。
- **L1452 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfofp8min"] in {`.
  **L1452 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfofp8min"] in {`。
- **L1453 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1453 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L1454 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e4m3" in`.
  **L1454 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e4m3" in`。
- **L1455 EN**: Declares TableGen defm record `defm`.
  **L1455 CN**: 声明 TableGen defm 记录 `defm`。
- **L1456 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e5m2",`.
  **L1456 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e5m2",`。

### Lines 1457-1484

````tablegen
            IRName = "vfncvt_f_f_w_alt",
            MaskedIRName = "vfncvt_f_f_w_alt_mask",
            AltFmt = 1 in
          defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "b", [["wv", "vwu"]]>;
        let OverloadedName = "vfncvt_sat_f_f8e4m3" in
          defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_w", "a", [["wv", "vwu"]]>;
        let OverloadedName = "vfncvt_sat_f_f8e5m2",
            IRName = "vfncvt_sat_f_f_w_alt",
            MaskedIRName = "vfncvt_sat_f_f_w_alt_mask",
            AltFmt = 1 in
          defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_w", "b", [["wv", "vwu"]]>;
      }
      let Log2LMUL = [-3, -2, -1, 0, 1] in {
        let OverloadedName = "vfncvt_f_f8e4m3" in
          defm : RVVConvBuiltinSet<"vfncvt_f_f_q", "a", [["v", "vqu"]]>;
        let OverloadedName = "vfncvt_f_f8e5m2",
            IRName = "vfncvt_f_f_q_alt",
            MaskedIRName = "vfncvt_f_f_q_alt_mask",
            AltFmt = 1 in
          defm : RVVConvBuiltinSet<"vfncvt_f_f_q", "b", [["v", "vqu"]]>;
        let OverloadedName = "vfncvt_sat_f_f8e4m3" in
          defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_q", "a", [["v", "vqu"]]>;
        let OverloadedName = "vfncvt_sat_f_f8e5m2",
            IRName = "vfncvt_sat_f_f_q_alt",
            MaskedIRName = "vfncvt_sat_f_f_q_alt_mask",
            AltFmt = 1 in
          defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_q", "b", [["v", "vqu"]]>;
      }
````
- **L1457 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_f_f_w_alt",`.
  **L1457 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_f_f_w_alt",`。
- **L1458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_f_f_w_alt_mask",`.
  **L1458 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_f_f_w_alt_mask",`。
- **L1459 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1459 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1460 EN**: Declares TableGen defm record `defm`.
  **L1460 CN**: 声明 TableGen defm 记录 `defm`。
- **L1461 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e4m3" in`.
  **L1461 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e4m3" in`。
- **L1462 EN**: Declares TableGen defm record `defm`.
  **L1462 CN**: 声明 TableGen defm 记录 `defm`。
- **L1463 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e5m2",`.
  **L1463 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e5m2",`。
- **L1464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_sat_f_f_w_alt",`.
  **L1464 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_sat_f_f_w_alt",`。
- **L1465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_sat_f_f_w_alt_mask",`.
  **L1465 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_sat_f_f_w_alt_mask",`。
- **L1466 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1466 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1467 EN**: Declares TableGen defm record `defm`.
  **L1467 CN**: 声明 TableGen defm 记录 `defm`。
- **L1468 EN**: Closes the current lexical scope or compound statement.
  **L1468 CN**: 结束当前词法作用域或复合语句块。
- **L1469 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1] in {`.
  **L1469 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1] in {`。
- **L1470 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e4m3" in`.
  **L1470 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e4m3" in`。
- **L1471 EN**: Declares TableGen defm record `defm`.
  **L1471 CN**: 声明 TableGen defm 记录 `defm`。
- **L1472 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e5m2",`.
  **L1472 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e5m2",`。
- **L1473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_f_f_q_alt",`.
  **L1473 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_f_f_q_alt",`。
- **L1474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_f_f_q_alt_mask",`.
  **L1474 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_f_f_q_alt_mask",`。
- **L1475 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1475 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1476 EN**: Declares TableGen defm record `defm`.
  **L1476 CN**: 声明 TableGen defm 记录 `defm`。
- **L1477 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e4m3" in`.
  **L1477 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e4m3" in`。
- **L1478 EN**: Declares TableGen defm record `defm`.
  **L1478 CN**: 声明 TableGen defm 记录 `defm`。
- **L1479 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e5m2",`.
  **L1479 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e5m2",`。
- **L1480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_sat_f_f_q_alt",`.
  **L1480 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_sat_f_f_q_alt",`。
- **L1481 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_sat_f_f_q_alt_mask",`.
  **L1481 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_sat_f_f_q_alt_mask",`。
- **L1482 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1482 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1483 EN**: Declares TableGen defm record `defm`.
  **L1483 CN**: 声明 TableGen defm 记录 `defm`。
- **L1484 EN**: Closes the current lexical scope or compound statement.
  **L1484 CN**: 结束当前词法作用域或复合语句块。

### Lines 1485-1512

````tablegen
    }
  }

  // 13.17. Single-Width Floating-Point/Integer Type-Convert Instructions
  let OverloadedName = "vfcvt_x" in {
    defm : RVVConvBuiltinSet<"vfcvt_x_f_v", "fd", [["Iv", "Ivv"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm : RVVConvBuiltinSet<"vfcvt_x_f_v", "x", [["Iv", "Ivv"]]>;
  }
  let OverloadedName = "vfcvt_xu" in {
    defm : RVVConvBuiltinSet<"vfcvt_xu_f_v", "fd", [["Uv", "Uvv"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm : RVVConvBuiltinSet<"vfcvt_xu_f_v", "x", [["Uv", "Uvv"]]>;
  }
  let OverloadedName = "vfcvt_f" in {
    defm : RVVConvBuiltinSet<"vfcvt_f_x_v", "fd", [["v", "vIv"]]>;
    defm : RVVConvBuiltinSet<"vfcvt_f_xu_v", "fd", [["v", "vUv"]]>;
    let RequiredFeatures = ["zvfh"] in {
      defm : RVVConvBuiltinSet<"vfcvt_f_x_v", "x", [["v", "vIv"]]>;
      defm : RVVConvBuiltinSet<"vfcvt_f_xu_v", "x", [["v", "vUv"]]>;
    }
  }

  // 13.18. Widening Floating-Point/Integer Type-Convert Instructions
  let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
    let OverloadedName = "vfwcvt_x" in {
      defm : RVVConvBuiltinSet<"vfwcvt_x_f_v", "f", [["Iw", "Iwv"]]>;
      let RequiredFeatures = ["zvfh"] in
````
- **L1485 EN**: Closes the current lexical scope or compound statement.
  **L1485 CN**: 结束当前词法作用域或复合语句块。
- **L1486 EN**: Closes the current lexical scope or compound statement.
  **L1486 CN**: 结束当前词法作用域或复合语句块。
- **L1487 EN**: Blank line separating nearby declarations or logic blocks.
  **L1487 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1488 EN**: Comment explains nearby logic, constraints, or intent: `13.17. Single-Width Floating-Point/Integer Type-Convert Instructions`.
  **L1488 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.17. Single-Width Floating-Point/Integer Type-Convert Instructions`。
- **L1489 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_x" in {`.
  **L1489 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_x" in {`。
- **L1490 EN**: Declares TableGen defm record `defm`.
  **L1490 CN**: 声明 TableGen defm 记录 `defm`。
- **L1491 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1491 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1492 EN**: Declares TableGen defm record `defm`.
  **L1492 CN**: 声明 TableGen defm 记录 `defm`。
- **L1493 EN**: Closes the current lexical scope or compound statement.
  **L1493 CN**: 结束当前词法作用域或复合语句块。
- **L1494 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_xu" in {`.
  **L1494 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_xu" in {`。
- **L1495 EN**: Declares TableGen defm record `defm`.
  **L1495 CN**: 声明 TableGen defm 记录 `defm`。
- **L1496 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1496 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1497 EN**: Declares TableGen defm record `defm`.
  **L1497 CN**: 声明 TableGen defm 记录 `defm`。
- **L1498 EN**: Closes the current lexical scope or compound statement.
  **L1498 CN**: 结束当前词法作用域或复合语句块。
- **L1499 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfcvt_f" in {`.
  **L1499 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfcvt_f" in {`。
- **L1500 EN**: Declares TableGen defm record `defm`.
  **L1500 CN**: 声明 TableGen defm 记录 `defm`。
- **L1501 EN**: Declares TableGen defm record `defm`.
  **L1501 CN**: 声明 TableGen defm 记录 `defm`。
- **L1502 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in {`.
  **L1502 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in {`。
- **L1503 EN**: Declares TableGen defm record `defm`.
  **L1503 CN**: 声明 TableGen defm 记录 `defm`。
- **L1504 EN**: Declares TableGen defm record `defm`.
  **L1504 CN**: 声明 TableGen defm 记录 `defm`。
- **L1505 EN**: Closes the current lexical scope or compound statement.
  **L1505 CN**: 结束当前词法作用域或复合语句块。
- **L1506 EN**: Closes the current lexical scope or compound statement.
  **L1506 CN**: 结束当前词法作用域或复合语句块。
- **L1507 EN**: Blank line separating nearby declarations or logic blocks.
  **L1507 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1508 EN**: Comment explains nearby logic, constraints, or intent: `13.18. Widening Floating-Point/Integer Type-Convert Instructions`.
  **L1508 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.18. Widening Floating-Point/Integer Type-Convert Instructions`。
- **L1509 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1509 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L1510 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_x" in {`.
  **L1510 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_x" in {`。
- **L1511 EN**: Declares TableGen defm record `defm`.
  **L1511 CN**: 声明 TableGen defm 记录 `defm`。
- **L1512 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1512 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。

### Lines 1513-1540

````tablegen
        defm : RVVConvBuiltinSet<"vfwcvt_x_f_v", "x", [["Iw", "Iwv"]]>;
    }
    let OverloadedName = "vfwcvt_xu" in {
      defm : RVVConvBuiltinSet<"vfwcvt_xu_f_v", "f", [["Uw", "Uwv"]]>;
      let RequiredFeatures = ["zvfh"] in
        defm : RVVConvBuiltinSet<"vfwcvt_xu_f_v", "x", [["Uw", "Uwv"]]>;
    }
  }
  // 13.19. Narrowing Floating-Point/Integer Type-Convert Instructions
  let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
    let OverloadedName = "vfncvt_x" in {
      defm : RVVConvBuiltinSet<"vfncvt_x_f_w", "si", [["Iv", "IvFw"]]>;
      let RequiredFeatures = ["zvfh"] in
        defm : RVVConvBuiltinSet<"vfncvt_x_f_w", "c", [["Iv", "IvFw"]]>;
      let RequiredFeatures = ["zvfbfa"] in
        defm : RVVConvBuiltinSet<"vfncvt_x_f_w", "c", [["YwIv", "IvYw"]]>;
    }
    let OverloadedName = "vfncvt_xu" in {
      defm : RVVConvBuiltinSet<"vfncvt_xu_f_w", "si", [["Uv", "UvFw"]]>;
      let RequiredFeatures = ["zvfh"] in
        defm : RVVConvBuiltinSet<"vfncvt_xu_f_w", "c", [["Uv", "UvFw"]]>;
      let RequiredFeatures = ["zvfbfa"] in
        defm : RVVConvBuiltinSet<"vfncvt_xu_f_w", "c", [["YwUv", "UvYw"]]>;
    }
    let OverloadedName = "vfncvt_f" in {
      defm : RVVConvBuiltinSet<"vfncvt_f_x_w", "f", [["v", "vIw"]]>;
      defm : RVVConvBuiltinSet<"vfncvt_f_xu_w", "f", [["v", "vUw"]]>;
      let RequiredFeatures = ["zvfh"] in {
````
- **L1513 EN**: Declares TableGen defm record `defm`.
  **L1513 CN**: 声明 TableGen defm 记录 `defm`。
- **L1514 EN**: Closes the current lexical scope or compound statement.
  **L1514 CN**: 结束当前词法作用域或复合语句块。
- **L1515 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfwcvt_xu" in {`.
  **L1515 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfwcvt_xu" in {`。
- **L1516 EN**: Declares TableGen defm record `defm`.
  **L1516 CN**: 声明 TableGen defm 记录 `defm`。
- **L1517 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1517 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1518 EN**: Declares TableGen defm record `defm`.
  **L1518 CN**: 声明 TableGen defm 记录 `defm`。
- **L1519 EN**: Closes the current lexical scope or compound statement.
  **L1519 CN**: 结束当前词法作用域或复合语句块。
- **L1520 EN**: Closes the current lexical scope or compound statement.
  **L1520 CN**: 结束当前词法作用域或复合语句块。
- **L1521 EN**: Comment explains nearby logic, constraints, or intent: `13.19. Narrowing Floating-Point/Integer Type-Convert Instructions`.
  **L1521 CN**: 注释解释附近代码的逻辑、约束或设计意图：`13.19. Narrowing Floating-Point/Integer Type-Convert Instructions`。
- **L1522 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1522 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L1523 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_x" in {`.
  **L1523 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_x" in {`。
- **L1524 EN**: Declares TableGen defm record `defm`.
  **L1524 CN**: 声明 TableGen defm 记录 `defm`。
- **L1525 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1525 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1526 EN**: Declares TableGen defm record `defm`.
  **L1526 CN**: 声明 TableGen defm 记录 `defm`。
- **L1527 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1527 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1528 EN**: Declares TableGen defm record `defm`.
  **L1528 CN**: 声明 TableGen defm 记录 `defm`。
- **L1529 EN**: Closes the current lexical scope or compound statement.
  **L1529 CN**: 结束当前词法作用域或复合语句块。
- **L1530 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_xu" in {`.
  **L1530 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_xu" in {`。
- **L1531 EN**: Declares TableGen defm record `defm`.
  **L1531 CN**: 声明 TableGen defm 记录 `defm`。
- **L1532 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1532 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1533 EN**: Declares TableGen defm record `defm`.
  **L1533 CN**: 声明 TableGen defm 记录 `defm`。
- **L1534 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1534 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1535 EN**: Declares TableGen defm record `defm`.
  **L1535 CN**: 声明 TableGen defm 记录 `defm`。
- **L1536 EN**: Closes the current lexical scope or compound statement.
  **L1536 CN**: 结束当前词法作用域或复合语句块。
- **L1537 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f" in {`.
  **L1537 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f" in {`。
- **L1538 EN**: Declares TableGen defm record `defm`.
  **L1538 CN**: 声明 TableGen defm 记录 `defm`。
- **L1539 EN**: Declares TableGen defm record `defm`.
  **L1539 CN**: 声明 TableGen defm 记录 `defm`。
- **L1540 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in {`.
  **L1540 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in {`。

### Lines 1541-1568

````tablegen
        defm : RVVConvBuiltinSet<"vfncvt_f_x_w", "x", [["v", "vIw"]]>;
        defm : RVVConvBuiltinSet<"vfncvt_f_xu_w", "x", [["v", "vUw"]]>;
      }
    }
    let OverloadedName = "vfncvt_f" in {
      defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "f", [["v", "vw"]]>;
      let RequiredFeatures = ["zvfhmin"] in
        defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "x", [["v", "vw"]]>;
    }
    let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_f_bf16" in
      defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "y", [["v", "vw"]]>;
  }

  // Zvfbfmin - Vector convert FP32 to BF16
  let Log2LMUL = [-2, -1, 0, 1, 2],
      OverloadedName = "vfncvtbf16_f" in
  defm : RVVConvBuiltinSet<"vfncvtbf16_f_f_w", "y", [["v", "vFw"]]>;

  // Zvfofp8min
  let RequiredFeatures = ["zvfofp8min"] in {
    let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {
      let OverloadedName = "vfncvt_f_f8e4m3" in
        defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "a", [["wv", "vw"]]>;
      let OverloadedName = "vfncvt_f_f8e5m2",
          IRName = "vfncvt_f_f_w_alt",
          MaskedIRName = "vfncvt_f_f_w_alt_mask",
          AltFmt = 1 in
        defm : RVVConvBuiltinSet<"vfncvt_f_f_w", "b", [["wv", "vw"]]>;
````
- **L1541 EN**: Declares TableGen defm record `defm`.
  **L1541 CN**: 声明 TableGen defm 记录 `defm`。
- **L1542 EN**: Declares TableGen defm record `defm`.
  **L1542 CN**: 声明 TableGen defm 记录 `defm`。
- **L1543 EN**: Closes the current lexical scope or compound statement.
  **L1543 CN**: 结束当前词法作用域或复合语句块。
- **L1544 EN**: Closes the current lexical scope or compound statement.
  **L1544 CN**: 结束当前词法作用域或复合语句块。
- **L1545 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f" in {`.
  **L1545 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f" in {`。
- **L1546 EN**: Declares TableGen defm record `defm`.
  **L1546 CN**: 声明 TableGen defm 记录 `defm`。
- **L1547 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfhmin"] in`.
  **L1547 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfhmin"] in`。
- **L1548 EN**: Declares TableGen defm record `defm`.
  **L1548 CN**: 声明 TableGen defm 记录 `defm`。
- **L1549 EN**: Closes the current lexical scope or compound statement.
  **L1549 CN**: 结束当前词法作用域或复合语句块。
- **L1550 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_f_bf16" in`.
  **L1550 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"], OverloadedName = "vfncvt_f_bf16" in`。
- **L1551 EN**: Declares TableGen defm record `defm`.
  **L1551 CN**: 声明 TableGen defm 记录 `defm`。
- **L1552 EN**: Closes the current lexical scope or compound statement.
  **L1552 CN**: 结束当前词法作用域或复合语句块。
- **L1553 EN**: Blank line separating nearby declarations or logic blocks.
  **L1553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1554 EN**: Comment explains nearby logic, constraints, or intent: `Zvfbfmin - Vector convert FP32 to BF16`.
  **L1554 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvfbfmin - Vector convert FP32 to BF16`。
- **L1555 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-2, -1, 0, 1, 2],`.
  **L1555 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-2, -1, 0, 1, 2],`。
- **L1556 EN**: Continues the surrounding expression or declaration: `OverloadedName = "vfncvtbf16_f" in`.
  **L1556 CN**: 继续构造周围的表达式或声明：`OverloadedName = "vfncvtbf16_f" in`。
- **L1557 EN**: Declares TableGen defm record `defm`.
  **L1557 CN**: 声明 TableGen defm 记录 `defm`。
- **L1558 EN**: Blank line separating nearby declarations or logic blocks.
  **L1558 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1559 EN**: Comment explains nearby logic, constraints, or intent: `Zvfofp8min`.
  **L1559 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvfofp8min`。
- **L1560 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfofp8min"] in {`.
  **L1560 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfofp8min"] in {`。
- **L1561 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`.
  **L1561 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1, 2] in {`。
- **L1562 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e4m3" in`.
  **L1562 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e4m3" in`。
- **L1563 EN**: Declares TableGen defm record `defm`.
  **L1563 CN**: 声明 TableGen defm 记录 `defm`。
- **L1564 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e5m2",`.
  **L1564 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e5m2",`。
- **L1565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_f_f_w_alt",`.
  **L1565 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_f_f_w_alt",`。
- **L1566 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_f_f_w_alt_mask",`.
  **L1566 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_f_f_w_alt_mask",`。
- **L1567 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1567 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1568 EN**: Declares TableGen defm record `defm`.
  **L1568 CN**: 声明 TableGen defm 记录 `defm`。

### Lines 1569-1596

````tablegen
      let OverloadedName = "vfncvt_sat_f_f8e4m3" in
        defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_w", "a", [["wv", "vw"]]>;
      let OverloadedName = "vfncvt_sat_f_f8e5m2",
          IRName = "vfncvt_sat_f_f_w_alt",
          MaskedIRName = "vfncvt_sat_f_f_w_alt_mask",
          AltFmt = 1 in
        defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_w", "b", [["wv", "vw"]]>;
    }
    let Log2LMUL = [-3, -2, -1, 0, 1] in {
      // The non-policy overloaded version will have conflict with zvfhmin, so we
      // need another naming. The reason policy version doesn't have this issue is
      // because of passthru operand which is f16 in zvfhmin and u8 in this case.
      let OverloadedName = "vfncvt_f_f8e4m3" in
        defm : RVVConvBuiltinSet<"vfncvt_f_f_q", "a", [["v", "vq"]]>;
      let OverloadedName = "vfncvt_f_f8e5m2",
          IRName = "vfncvt_f_f_q_alt",
          MaskedIRName = "vfncvt_f_f_q_alt_mask",
          AltFmt = 1 in
        defm : RVVConvBuiltinSet<"vfncvt_f_f_q", "b", [["v", "vq"]]>;
      let OverloadedName = "vfncvt_sat_f_f8e4m3" in
        defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_q", "a", [["v", "vq"]]>;
      let OverloadedName = "vfncvt_sat_f_f8e5m2",
          IRName = "vfncvt_sat_f_f_q_alt",
          MaskedIRName = "vfncvt_sat_f_f_q_alt_mask",
          AltFmt = 1 in
        defm : RVVConvBuiltinSet<"vfncvt_sat_f_f_q", "b", [["v", "vq"]]>;
    }
  }
````
- **L1569 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e4m3" in`.
  **L1569 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e4m3" in`。
- **L1570 EN**: Declares TableGen defm record `defm`.
  **L1570 CN**: 声明 TableGen defm 记录 `defm`。
- **L1571 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e5m2",`.
  **L1571 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e5m2",`。
- **L1572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_sat_f_f_w_alt",`.
  **L1572 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_sat_f_f_w_alt",`。
- **L1573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_sat_f_f_w_alt_mask",`.
  **L1573 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_sat_f_f_w_alt_mask",`。
- **L1574 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1574 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1575 EN**: Declares TableGen defm record `defm`.
  **L1575 CN**: 声明 TableGen defm 记录 `defm`。
- **L1576 EN**: Closes the current lexical scope or compound statement.
  **L1576 CN**: 结束当前词法作用域或复合语句块。
- **L1577 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3, -2, -1, 0, 1] in {`.
  **L1577 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3, -2, -1, 0, 1] in {`。
- **L1578 EN**: Comment explains nearby logic, constraints, or intent: `The non-policy overloaded version will have conflict with zvfhmin, so we`.
  **L1578 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The non-policy overloaded version will have conflict with zvfhmin, so we`。
- **L1579 EN**: Comment explains nearby logic, constraints, or intent: `need another naming. The reason policy version doesn't have this issue is`.
  **L1579 CN**: 注释解释附近代码的逻辑、约束或设计意图：`need another naming. The reason policy version doesn't have this issue is`。
- **L1580 EN**: Comment explains nearby logic, constraints, or intent: `because of passthru operand which is f16 in zvfhmin and u8 in this case.`.
  **L1580 CN**: 注释解释附近代码的逻辑、约束或设计意图：`because of passthru operand which is f16 in zvfhmin and u8 in this case.`。
- **L1581 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e4m3" in`.
  **L1581 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e4m3" in`。
- **L1582 EN**: Declares TableGen defm record `defm`.
  **L1582 CN**: 声明 TableGen defm 记录 `defm`。
- **L1583 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_f_f8e5m2",`.
  **L1583 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_f_f8e5m2",`。
- **L1584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_f_f_q_alt",`.
  **L1584 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_f_f_q_alt",`。
- **L1585 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_f_f_q_alt_mask",`.
  **L1585 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_f_f_q_alt_mask",`。
- **L1586 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1586 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1587 EN**: Declares TableGen defm record `defm`.
  **L1587 CN**: 声明 TableGen defm 记录 `defm`。
- **L1588 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e4m3" in`.
  **L1588 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e4m3" in`。
- **L1589 EN**: Declares TableGen defm record `defm`.
  **L1589 CN**: 声明 TableGen defm 记录 `defm`。
- **L1590 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfncvt_sat_f_f8e5m2",`.
  **L1590 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfncvt_sat_f_f8e5m2",`。
- **L1591 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRName = "vfncvt_sat_f_f_q_alt",`.
  **L1591 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRName = "vfncvt_sat_f_f_q_alt",`。
- **L1592 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedIRName = "vfncvt_sat_f_f_q_alt_mask",`.
  **L1592 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedIRName = "vfncvt_sat_f_f_q_alt_mask",`。
- **L1593 EN**: Continues the surrounding expression or declaration: `AltFmt = 1 in`.
  **L1593 CN**: 继续构造周围的表达式或声明：`AltFmt = 1 in`。
- **L1594 EN**: Declares TableGen defm record `defm`.
  **L1594 CN**: 声明 TableGen defm 记录 `defm`。
- **L1595 EN**: Closes the current lexical scope or compound statement.
  **L1595 CN**: 结束当前词法作用域或复合语句块。
- **L1596 EN**: Closes the current lexical scope or compound statement.
  **L1596 CN**: 结束当前词法作用域或复合语句块。

### Lines 1597-1624

````tablegen
}
}

// 14. Vector Reduction Operations
// 14.1. Vector Single-Width Integer Reduction Instructions
let UnMaskedPolicyScheme = HasPassthruOperand,
    MaskedPolicyScheme = HasPassthruOperand,
    HasMaskPolicy = false in {
defm vredsum : RVVIntReductionBuiltinSet;
defm vredmaxu : RVVUnsignedReductionBuiltin;
defm vredmax : RVVSignedReductionBuiltin;
defm vredminu : RVVUnsignedReductionBuiltin;
defm vredmin : RVVSignedReductionBuiltin;
defm vredand : RVVIntReductionBuiltinSet;
defm vredor : RVVIntReductionBuiltinSet;
defm vredxor : RVVIntReductionBuiltinSet;

// 14.2. Vector Widening Integer Reduction Instructions
// Vector Widening Integer Reduction Operations
let HasMaskedOffOperand = true in {
  defm vwredsum : RVVOutOp0BuiltinSet<"vwredsum", "csi",
                                      [["vs", "vSw", "SwvSw"]]>;
  defm vwredsumu : RVVOutOp0BuiltinSet<"vwredsumu", "csi",
                                       [["vs", "UvUSw", "USwUvUSw"]]>;
}

// 14.3. Vector Single-Width Floating-Point Reduction Instructions
defm vfredmax : RVVFloatingReductionBuiltin;
````
- **L1597 EN**: Closes the current lexical scope or compound statement.
  **L1597 CN**: 结束当前词法作用域或复合语句块。
- **L1598 EN**: Closes the current lexical scope or compound statement.
  **L1598 CN**: 结束当前词法作用域或复合语句块。
- **L1599 EN**: Blank line separating nearby declarations or logic blocks.
  **L1599 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1600 EN**: Comment explains nearby logic, constraints, or intent: `14. Vector Reduction Operations`.
  **L1600 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14. Vector Reduction Operations`。
- **L1601 EN**: Comment explains nearby logic, constraints, or intent: `14.1. Vector Single-Width Integer Reduction Instructions`.
  **L1601 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.1. Vector Single-Width Integer Reduction Instructions`。
- **L1602 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L1602 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L1603 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = HasPassthruOperand,`.
  **L1603 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = HasPassthruOperand,`。
- **L1604 EN**: Continues the surrounding expression or declaration: `HasMaskPolicy = false in {`.
  **L1604 CN**: 继续构造周围的表达式或声明：`HasMaskPolicy = false in {`。
- **L1605 EN**: Declares TableGen defm record `vredsum`.
  **L1605 CN**: 声明 TableGen defm 记录 `vredsum`。
- **L1606 EN**: Declares TableGen defm record `vredmaxu`.
  **L1606 CN**: 声明 TableGen defm 记录 `vredmaxu`。
- **L1607 EN**: Declares TableGen defm record `vredmax`.
  **L1607 CN**: 声明 TableGen defm 记录 `vredmax`。
- **L1608 EN**: Declares TableGen defm record `vredminu`.
  **L1608 CN**: 声明 TableGen defm 记录 `vredminu`。
- **L1609 EN**: Declares TableGen defm record `vredmin`.
  **L1609 CN**: 声明 TableGen defm 记录 `vredmin`。
- **L1610 EN**: Declares TableGen defm record `vredand`.
  **L1610 CN**: 声明 TableGen defm 记录 `vredand`。
- **L1611 EN**: Declares TableGen defm record `vredor`.
  **L1611 CN**: 声明 TableGen defm 记录 `vredor`。
- **L1612 EN**: Declares TableGen defm record `vredxor`.
  **L1612 CN**: 声明 TableGen defm 记录 `vredxor`。
- **L1613 EN**: Blank line separating nearby declarations or logic blocks.
  **L1613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1614 EN**: Comment explains nearby logic, constraints, or intent: `14.2. Vector Widening Integer Reduction Instructions`.
  **L1614 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.2. Vector Widening Integer Reduction Instructions`。
- **L1615 EN**: Comment explains nearby logic, constraints, or intent: `Vector Widening Integer Reduction Operations`.
  **L1615 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector Widening Integer Reduction Operations`。
- **L1616 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMaskedOffOperand = true in {`.
  **L1616 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMaskedOffOperand = true in {`。
- **L1617 EN**: Declares TableGen defm record `vwredsum`.
  **L1617 CN**: 声明 TableGen defm 记录 `vwredsum`。
- **L1618 EN**: Adds a standalone statement or declaration: `[["vs", "vSw", "SwvSw"]]>;`.
  **L1618 CN**: 添加一条独立语句或声明：`[["vs", "vSw", "SwvSw"]]>;`。
- **L1619 EN**: Declares TableGen defm record `vwredsumu`.
  **L1619 CN**: 声明 TableGen defm 记录 `vwredsumu`。
- **L1620 EN**: Adds a standalone statement or declaration: `[["vs", "UvUSw", "USwUvUSw"]]>;`.
  **L1620 CN**: 添加一条独立语句或声明：`[["vs", "UvUSw", "USwUvUSw"]]>;`。
- **L1621 EN**: Closes the current lexical scope or compound statement.
  **L1621 CN**: 结束当前词法作用域或复合语句块。
- **L1622 EN**: Blank line separating nearby declarations or logic blocks.
  **L1622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1623 EN**: Comment explains nearby logic, constraints, or intent: `14.3. Vector Single-Width Floating-Point Reduction Instructions`.
  **L1623 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.3. Vector Single-Width Floating-Point Reduction Instructions`。
- **L1624 EN**: Declares TableGen defm record `vfredmax`.
  **L1624 CN**: 声明 TableGen defm 记录 `vfredmax`。

### Lines 1625-1652

````tablegen
defm vfredmin : RVVFloatingReductionBuiltin;
let ManualCodegen = [{
  {
    return emitRVVFloatingReductionBuiltin(
        this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,
        SegInstSEW);
  }
}] in {
  let HasFRMRoundModeOp = 1 in {
    // 14.3. Vector Single-Width Floating-Point Reduction Instructions
    defm vfredusum : RVVFloatingReductionBuiltinRoundingMode;
    defm vfredosum : RVVFloatingReductionBuiltinRoundingMode;

    // 14.4. Vector Widening Floating-Point Reduction Instructions
    defm vfwredusum : RVVFloatingWidenReductionBuiltinRoundingMode;
    defm vfwredosum : RVVFloatingWidenReductionBuiltinRoundingMode;
  }
  // 14.3. Vector Single-Width Floating-Point Reduction Instructions
  defm vfredusum : RVVFloatingReductionBuiltin;
  defm vfredosum : RVVFloatingReductionBuiltin;

  // 14.4. Vector Widening Floating-Point Reduction Instructions
  defm vfwredusum : RVVFloatingWidenReductionBuiltin;
  defm vfwredosum : RVVFloatingWidenReductionBuiltin;
}
}

// 15. Vector Mask Instructions
````
- **L1625 EN**: Declares TableGen defm record `vfredmin`.
  **L1625 CN**: 声明 TableGen defm 记录 `vfredmin`。
- **L1626 EN**: Assigns a TableGen property that affects following records or inherited fields: `let ManualCodegen = [{`.
  **L1626 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let ManualCodegen = [{`。
- **L1627 EN**: Opens a new lexical scope or compound statement.
  **L1627 CN**: 打开一个新的词法作用域或复合语句块。
- **L1628 EN**: Returns from the current function with `emitRVVFloatingReductionBuiltin(`.
  **L1628 CN**: 以 `emitRVVFloatingReductionBuiltin(` 从当前函数返回。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`this, E, ReturnValue, ResultType, ID, Ops, PolicyAttrs, IsMasked,`。
- **L1630 EN**: Adds a standalone statement or declaration: `SegInstSEW);`.
  **L1630 CN**: 添加一条独立语句或声明：`SegInstSEW);`。
- **L1631 EN**: Closes the current lexical scope or compound statement.
  **L1631 CN**: 结束当前词法作用域或复合语句块。
- **L1632 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1632 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1633 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasFRMRoundModeOp = 1 in {`.
  **L1633 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasFRMRoundModeOp = 1 in {`。
- **L1634 EN**: Comment explains nearby logic, constraints, or intent: `14.3. Vector Single-Width Floating-Point Reduction Instructions`.
  **L1634 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.3. Vector Single-Width Floating-Point Reduction Instructions`。
- **L1635 EN**: Declares TableGen defm record `vfredusum`.
  **L1635 CN**: 声明 TableGen defm 记录 `vfredusum`。
- **L1636 EN**: Declares TableGen defm record `vfredosum`.
  **L1636 CN**: 声明 TableGen defm 记录 `vfredosum`。
- **L1637 EN**: Blank line separating nearby declarations or logic blocks.
  **L1637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1638 EN**: Comment explains nearby logic, constraints, or intent: `14.4. Vector Widening Floating-Point Reduction Instructions`.
  **L1638 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.4. Vector Widening Floating-Point Reduction Instructions`。
- **L1639 EN**: Declares TableGen defm record `vfwredusum`.
  **L1639 CN**: 声明 TableGen defm 记录 `vfwredusum`。
- **L1640 EN**: Declares TableGen defm record `vfwredosum`.
  **L1640 CN**: 声明 TableGen defm 记录 `vfwredosum`。
- **L1641 EN**: Closes the current lexical scope or compound statement.
  **L1641 CN**: 结束当前词法作用域或复合语句块。
- **L1642 EN**: Comment explains nearby logic, constraints, or intent: `14.3. Vector Single-Width Floating-Point Reduction Instructions`.
  **L1642 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.3. Vector Single-Width Floating-Point Reduction Instructions`。
- **L1643 EN**: Declares TableGen defm record `vfredusum`.
  **L1643 CN**: 声明 TableGen defm 记录 `vfredusum`。
- **L1644 EN**: Declares TableGen defm record `vfredosum`.
  **L1644 CN**: 声明 TableGen defm 记录 `vfredosum`。
- **L1645 EN**: Blank line separating nearby declarations or logic blocks.
  **L1645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1646 EN**: Comment explains nearby logic, constraints, or intent: `14.4. Vector Widening Floating-Point Reduction Instructions`.
  **L1646 CN**: 注释解释附近代码的逻辑、约束或设计意图：`14.4. Vector Widening Floating-Point Reduction Instructions`。
- **L1647 EN**: Declares TableGen defm record `vfwredusum`.
  **L1647 CN**: 声明 TableGen defm 记录 `vfwredusum`。
- **L1648 EN**: Declares TableGen defm record `vfwredosum`.
  **L1648 CN**: 声明 TableGen defm 记录 `vfwredosum`。
- **L1649 EN**: Closes the current lexical scope or compound statement.
  **L1649 CN**: 结束当前词法作用域或复合语句块。
- **L1650 EN**: Closes the current lexical scope or compound statement.
  **L1650 CN**: 结束当前词法作用域或复合语句块。
- **L1651 EN**: Blank line separating nearby declarations or logic blocks.
  **L1651 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1652 EN**: Comment explains nearby logic, constraints, or intent: `15. Vector Mask Instructions`.
  **L1652 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15. Vector Mask Instructions`。

### Lines 1653-1680

````tablegen
// 15.1. Vector Mask-Register Logical Instructions
def vmand    : RVVMaskBinBuiltin;
def vmnand   : RVVMaskBinBuiltin;
def vmandn   : RVVMaskBinBuiltin;
def vmxor    : RVVMaskBinBuiltin;
def vmor     : RVVMaskBinBuiltin;
def vmnor    : RVVMaskBinBuiltin;
def vmorn    : RVVMaskBinBuiltin;
def vmxnor   : RVVMaskBinBuiltin;
// pseudoinstructions
def vmclr    : RVVMaskNullaryBuiltin;
def vmset    : RVVMaskNullaryBuiltin;
defm vmmv_m : RVVPseudoMaskBuiltin<"vmand", "c">;
defm vmnot_m : RVVPseudoMaskBuiltin<"vmnand", "c">;

let MaskedPolicyScheme = NonePolicy in {
// 15.2. Vector count population in mask vcpop.m
def vcpop : RVVMaskOp0Builtin<"um">;

// 15.3. vfirst find-first-set mask bit
def vfirst : RVVMaskOp0Builtin<"lm">;
}

let MaskedPolicyScheme = HasPassthruOperand,
    HasTailPolicy = false in {
// 15.4. vmsbf.m set-before-first mask bit
def vmsbf : RVVMaskUnaryBuiltin;

````
- **L1653 EN**: Comment explains nearby logic, constraints, or intent: `15.1. Vector Mask-Register Logical Instructions`.
  **L1653 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.1. Vector Mask-Register Logical Instructions`。
- **L1654 EN**: Declares TableGen def record `vmand`.
  **L1654 CN**: 声明 TableGen def 记录 `vmand`。
- **L1655 EN**: Declares TableGen def record `vmnand`.
  **L1655 CN**: 声明 TableGen def 记录 `vmnand`。
- **L1656 EN**: Declares TableGen def record `vmandn`.
  **L1656 CN**: 声明 TableGen def 记录 `vmandn`。
- **L1657 EN**: Declares TableGen def record `vmxor`.
  **L1657 CN**: 声明 TableGen def 记录 `vmxor`。
- **L1658 EN**: Declares TableGen def record `vmor`.
  **L1658 CN**: 声明 TableGen def 记录 `vmor`。
- **L1659 EN**: Declares TableGen def record `vmnor`.
  **L1659 CN**: 声明 TableGen def 记录 `vmnor`。
- **L1660 EN**: Declares TableGen def record `vmorn`.
  **L1660 CN**: 声明 TableGen def 记录 `vmorn`。
- **L1661 EN**: Declares TableGen def record `vmxnor`.
  **L1661 CN**: 声明 TableGen def 记录 `vmxnor`。
- **L1662 EN**: Comment explains nearby logic, constraints, or intent: `pseudoinstructions`.
  **L1662 CN**: 注释解释附近代码的逻辑、约束或设计意图：`pseudoinstructions`。
- **L1663 EN**: Declares TableGen def record `vmclr`.
  **L1663 CN**: 声明 TableGen def 记录 `vmclr`。
- **L1664 EN**: Declares TableGen def record `vmset`.
  **L1664 CN**: 声明 TableGen def 记录 `vmset`。
- **L1665 EN**: Declares TableGen defm record `vmmv_m`.
  **L1665 CN**: 声明 TableGen defm 记录 `vmmv_m`。
- **L1666 EN**: Declares TableGen defm record `vmnot_m`.
  **L1666 CN**: 声明 TableGen defm 记录 `vmnot_m`。
- **L1667 EN**: Blank line separating nearby declarations or logic blocks.
  **L1667 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1668 EN**: Assigns a TableGen property that affects following records or inherited fields: `let MaskedPolicyScheme = NonePolicy in {`.
  **L1668 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let MaskedPolicyScheme = NonePolicy in {`。
- **L1669 EN**: Comment explains nearby logic, constraints, or intent: `15.2. Vector count population in mask vcpop.m`.
  **L1669 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.2. Vector count population in mask vcpop.m`。
- **L1670 EN**: Declares TableGen def record `vcpop`.
  **L1670 CN**: 声明 TableGen def 记录 `vcpop`。
- **L1671 EN**: Blank line separating nearby declarations or logic blocks.
  **L1671 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1672 EN**: Comment explains nearby logic, constraints, or intent: `15.3. vfirst find-first-set mask bit`.
  **L1672 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.3. vfirst find-first-set mask bit`。
- **L1673 EN**: Declares TableGen def record `vfirst`.
  **L1673 CN**: 声明 TableGen def 记录 `vfirst`。
- **L1674 EN**: Closes the current lexical scope or compound statement.
  **L1674 CN**: 结束当前词法作用域或复合语句块。
- **L1675 EN**: Blank line separating nearby declarations or logic blocks.
  **L1675 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1676 EN**: Assigns a TableGen property that affects following records or inherited fields: `let MaskedPolicyScheme = HasPassthruOperand,`.
  **L1676 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let MaskedPolicyScheme = HasPassthruOperand,`。
- **L1677 EN**: Continues the surrounding expression or declaration: `HasTailPolicy = false in {`.
  **L1677 CN**: 继续构造周围的表达式或声明：`HasTailPolicy = false in {`。
- **L1678 EN**: Comment explains nearby logic, constraints, or intent: `15.4. vmsbf.m set-before-first mask bit`.
  **L1678 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.4. vmsbf.m set-before-first mask bit`。
- **L1679 EN**: Declares TableGen def record `vmsbf`.
  **L1679 CN**: 声明 TableGen def 记录 `vmsbf`。
- **L1680 EN**: Blank line separating nearby declarations or logic blocks.
  **L1680 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1681-1708

````tablegen
// 15.5. vmsif.m set-including-first mask bit
def vmsif : RVVMaskUnaryBuiltin;

// 15.6. vmsof.m set-only-first mask bit
def vmsof : RVVMaskUnaryBuiltin;
}

let UnMaskedPolicyScheme = HasPassthruOperand, SupportOverloading = false in {
  // 15.8. Vector Iota Instruction
  defm viota : RVVOutBuiltinSet<"viota", "csil", [["m", "Uv", "Uvm"]]>;

  // 15.9. Vector Element Index Instruction
  defm vid : RVVOutBuiltinSet<"vid", "csil", [["v", "v", "v"],
                                              ["v", "Uv", "Uv"]]>;
}

// 16. Vector Permutation Instructions
// 16.1. Integer Scalar Move Instructions
let HasMasked = false, MaskedPolicyScheme = NonePolicy in {
  let HasVL = false, OverloadedName = "vmv_x" in
    defm vmv_x : RVVOp0BuiltinSet<"vmv_x_s", "csil",
                                   [["s", "ve", "ev"],
                                    ["s", "UvUe", "UeUv"]]>;
  let OverloadedName = "vmv_s",
      UnMaskedPolicyScheme = HasPassthruOperand,
      SupportOverloading = false in
    defm vmv_s : RVVOutBuiltinSet<"vmv_s_x", "csil",
                                   [["x", "v", "ve"],
````
- **L1681 EN**: Comment explains nearby logic, constraints, or intent: `15.5. vmsif.m set-including-first mask bit`.
  **L1681 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.5. vmsif.m set-including-first mask bit`。
- **L1682 EN**: Declares TableGen def record `vmsif`.
  **L1682 CN**: 声明 TableGen def 记录 `vmsif`。
- **L1683 EN**: Blank line separating nearby declarations or logic blocks.
  **L1683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1684 EN**: Comment explains nearby logic, constraints, or intent: `15.6. vmsof.m set-only-first mask bit`.
  **L1684 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.6. vmsof.m set-only-first mask bit`。
- **L1685 EN**: Declares TableGen def record `vmsof`.
  **L1685 CN**: 声明 TableGen def 记录 `vmsof`。
- **L1686 EN**: Closes the current lexical scope or compound statement.
  **L1686 CN**: 结束当前词法作用域或复合语句块。
- **L1687 EN**: Blank line separating nearby declarations or logic blocks.
  **L1687 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1688 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand, SupportOverloading = false in {`.
  **L1688 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand, SupportOverloading = false in {`。
- **L1689 EN**: Comment explains nearby logic, constraints, or intent: `15.8. Vector Iota Instruction`.
  **L1689 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.8. Vector Iota Instruction`。
- **L1690 EN**: Declares TableGen defm record `viota`.
  **L1690 CN**: 声明 TableGen defm 记录 `viota`。
- **L1691 EN**: Blank line separating nearby declarations or logic blocks.
  **L1691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1692 EN**: Comment explains nearby logic, constraints, or intent: `15.9. Vector Element Index Instruction`.
  **L1692 CN**: 注释解释附近代码的逻辑、约束或设计意图：`15.9. Vector Element Index Instruction`。
- **L1693 EN**: Declares TableGen defm record `vid`.
  **L1693 CN**: 声明 TableGen defm 记录 `vid`。
- **L1694 EN**: Adds a standalone statement or declaration: `["v", "Uv", "Uv"]]>;`.
  **L1694 CN**: 添加一条独立语句或声明：`["v", "Uv", "Uv"]]>;`。
- **L1695 EN**: Closes the current lexical scope or compound statement.
  **L1695 CN**: 结束当前词法作用域或复合语句块。
- **L1696 EN**: Blank line separating nearby declarations or logic blocks.
  **L1696 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1697 EN**: Comment explains nearby logic, constraints, or intent: `16. Vector Permutation Instructions`.
  **L1697 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16. Vector Permutation Instructions`。
- **L1698 EN**: Comment explains nearby logic, constraints, or intent: `16.1. Integer Scalar Move Instructions`.
  **L1698 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.1. Integer Scalar Move Instructions`。
- **L1699 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false, MaskedPolicyScheme = NonePolicy in {`.
  **L1699 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false, MaskedPolicyScheme = NonePolicy in {`。
- **L1700 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasVL = false, OverloadedName = "vmv_x" in`.
  **L1700 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasVL = false, OverloadedName = "vmv_x" in`。
- **L1701 EN**: Declares TableGen defm record `vmv_x`.
  **L1701 CN**: 声明 TableGen defm 记录 `vmv_x`。
- **L1702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["s", "ve", "ev"],`.
  **L1702 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["s", "ve", "ev"],`。
- **L1703 EN**: Adds a standalone statement or declaration: `["s", "UvUe", "UeUv"]]>;`.
  **L1703 CN**: 添加一条独立语句或声明：`["s", "UvUe", "UeUv"]]>;`。
- **L1704 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vmv_s",`.
  **L1704 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vmv_s",`。
- **L1705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L1705 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L1706 EN**: Continues the surrounding expression or declaration: `SupportOverloading = false in`.
  **L1706 CN**: 继续构造周围的表达式或声明：`SupportOverloading = false in`。
- **L1707 EN**: Declares TableGen defm record `vmv_s`.
  **L1707 CN**: 声明 TableGen defm 记录 `vmv_s`。
- **L1708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["x", "v", "ve"],`.
  **L1708 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["x", "v", "ve"],`。

### Lines 1709-1736

````tablegen
                                    ["x", "Uv", "UvUe"]]>;
}

// 16.2. Floating-Point Scalar Move Instructions
let HasMasked = false, MaskedPolicyScheme = NonePolicy in {
  let HasVL = false, OverloadedName = "vfmv_f" in {
    defm vfmv_f : RVVOp0BuiltinSet<"vfmv_f_s", "fd",
                                   [["s", "ve", "ev"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm vfmv_f : RVVOp0BuiltinSet<"vfmv_f_s", "x",
                                     [["s", "ve", "ev"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm vfmv_f : RVVOp0BuiltinSet<"vfmv_f_s", "y",
                                     [["s", "ve", "ev"]]>;
  }
  let OverloadedName = "vfmv_s",
      UnMaskedPolicyScheme = HasPassthruOperand,
      SupportOverloading = false in {
    defm vfmv_s : RVVOutBuiltinSet<"vfmv_s_f", "fd",
                                   [["f", "v", "ve"],
                                    ["x", "Uv", "UvUe"]]>;
    let RequiredFeatures = ["zvfh"] in
      defm vfmv_s : RVVOutBuiltinSet<"vfmv_s_f", "x",
                                     [["f", "v", "ve"],
                                      ["x", "Uv", "UvUe"]]>;
    let RequiredFeatures = ["zvfbfa"] in
      defm vfmv_s : RVVOutBuiltinSet<"vfmv_s_f", "y",
                                     [["f", "v", "ve"]]>;
````
- **L1709 EN**: Adds a standalone statement or declaration: `["x", "Uv", "UvUe"]]>;`.
  **L1709 CN**: 添加一条独立语句或声明：`["x", "Uv", "UvUe"]]>;`。
- **L1710 EN**: Closes the current lexical scope or compound statement.
  **L1710 CN**: 结束当前词法作用域或复合语句块。
- **L1711 EN**: Blank line separating nearby declarations or logic blocks.
  **L1711 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1712 EN**: Comment explains nearby logic, constraints, or intent: `16.2. Floating-Point Scalar Move Instructions`.
  **L1712 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.2. Floating-Point Scalar Move Instructions`。
- **L1713 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false, MaskedPolicyScheme = NonePolicy in {`.
  **L1713 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false, MaskedPolicyScheme = NonePolicy in {`。
- **L1714 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasVL = false, OverloadedName = "vfmv_f" in {`.
  **L1714 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasVL = false, OverloadedName = "vfmv_f" in {`。
- **L1715 EN**: Declares TableGen defm record `vfmv_f`.
  **L1715 CN**: 声明 TableGen defm 记录 `vfmv_f`。
- **L1716 EN**: Adds a standalone statement or declaration: `[["s", "ve", "ev"]]>;`.
  **L1716 CN**: 添加一条独立语句或声明：`[["s", "ve", "ev"]]>;`。
- **L1717 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1717 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1718 EN**: Declares TableGen defm record `vfmv_f`.
  **L1718 CN**: 声明 TableGen defm 记录 `vfmv_f`。
- **L1719 EN**: Adds a standalone statement or declaration: `[["s", "ve", "ev"]]>;`.
  **L1719 CN**: 添加一条独立语句或声明：`[["s", "ve", "ev"]]>;`。
- **L1720 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1720 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1721 EN**: Declares TableGen defm record `vfmv_f`.
  **L1721 CN**: 声明 TableGen defm 记录 `vfmv_f`。
- **L1722 EN**: Adds a standalone statement or declaration: `[["s", "ve", "ev"]]>;`.
  **L1722 CN**: 添加一条独立语句或声明：`[["s", "ve", "ev"]]>;`。
- **L1723 EN**: Closes the current lexical scope or compound statement.
  **L1723 CN**: 结束当前词法作用域或复合语句块。
- **L1724 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vfmv_s",`.
  **L1724 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vfmv_s",`。
- **L1725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L1725 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L1726 EN**: Continues the surrounding expression or declaration: `SupportOverloading = false in {`.
  **L1726 CN**: 继续构造周围的表达式或声明：`SupportOverloading = false in {`。
- **L1727 EN**: Declares TableGen defm record `vfmv_s`.
  **L1727 CN**: 声明 TableGen defm 记录 `vfmv_s`。
- **L1728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["f", "v", "ve"],`.
  **L1728 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["f", "v", "ve"],`。
- **L1729 EN**: Adds a standalone statement or declaration: `["x", "Uv", "UvUe"]]>;`.
  **L1729 CN**: 添加一条独立语句或声明：`["x", "Uv", "UvUe"]]>;`。
- **L1730 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfh"] in`.
  **L1730 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfh"] in`。
- **L1731 EN**: Declares TableGen defm record `vfmv_s`.
  **L1731 CN**: 声明 TableGen defm 记录 `vfmv_s`。
- **L1732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["f", "v", "ve"],`.
  **L1732 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["f", "v", "ve"],`。
- **L1733 EN**: Adds a standalone statement or declaration: `["x", "Uv", "UvUe"]]>;`.
  **L1733 CN**: 添加一条独立语句或声明：`["x", "Uv", "UvUe"]]>;`。
- **L1734 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfbfa"] in`.
  **L1734 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfbfa"] in`。
- **L1735 EN**: Declares TableGen defm record `vfmv_s`.
  **L1735 CN**: 声明 TableGen defm 记录 `vfmv_s`。
- **L1736 EN**: Adds a standalone statement or declaration: `[["f", "v", "ve"]]>;`.
  **L1736 CN**: 添加一条独立语句或声明：`[["f", "v", "ve"]]>;`。

### Lines 1737-1764

````tablegen
  }
}

// 16.3. Vector Slide Instructions
// 16.3.1. Vector Slideup Instructions
defm vslideup   : RVVSlideUpBuiltinSet;
// 16.3.2. Vector Slidedown Instructions
defm vslidedown : RVVSlideDownBuiltinSet;

// 16.3.3. Vector Slide1up Instructions
let UnMaskedPolicyScheme = HasPassthruOperand in {
defm vslide1up : RVVSlideOneBuiltinSet;
defm vfslide1up : RVVFloatingBinVFBuiltinSet<HasBF=1>;

// 16.3.4. Vector Slide1down Instruction
defm vslide1down : RVVSlideOneBuiltinSet;
defm vfslide1down : RVVFloatingBinVFBuiltinSet<HasBF=1>;

// 16.4. Vector Register Gather Instructions
// signed and floating type
defm vrgather : RVVOutBuiltinSet<"vrgather_vv", "csilxfdy",
                                 [["vv", "v", "vvUv"]]>;
defm vrgather : RVVOutBuiltinSet<"vrgather_vx", "csilxfdy",
                                 [["vx", "v", "vvz"]]>;
defm vrgatherei16 : RVVOutBuiltinSet<"vrgatherei16_vv", "csilxfdy",
                                     [["vv", "v", "vv(Log2EEW:4)Uv"]]>;
// unsigned type
defm vrgather : RVVOutBuiltinSet<"vrgather_vv", "csil",
````
- **L1737 EN**: Closes the current lexical scope or compound statement.
  **L1737 CN**: 结束当前词法作用域或复合语句块。
- **L1738 EN**: Closes the current lexical scope or compound statement.
  **L1738 CN**: 结束当前词法作用域或复合语句块。
- **L1739 EN**: Blank line separating nearby declarations or logic blocks.
  **L1739 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1740 EN**: Comment explains nearby logic, constraints, or intent: `16.3. Vector Slide Instructions`.
  **L1740 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.3. Vector Slide Instructions`。
- **L1741 EN**: Comment explains nearby logic, constraints, or intent: `16.3.1. Vector Slideup Instructions`.
  **L1741 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.3.1. Vector Slideup Instructions`。
- **L1742 EN**: Declares TableGen defm record `vslideup`.
  **L1742 CN**: 声明 TableGen defm 记录 `vslideup`。
- **L1743 EN**: Comment explains nearby logic, constraints, or intent: `16.3.2. Vector Slidedown Instructions`.
  **L1743 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.3.2. Vector Slidedown Instructions`。
- **L1744 EN**: Declares TableGen defm record `vslidedown`.
  **L1744 CN**: 声明 TableGen defm 记录 `vslidedown`。
- **L1745 EN**: Blank line separating nearby declarations or logic blocks.
  **L1745 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1746 EN**: Comment explains nearby logic, constraints, or intent: `16.3.3. Vector Slide1up Instructions`.
  **L1746 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.3.3. Vector Slide1up Instructions`。
- **L1747 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L1747 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L1748 EN**: Declares TableGen defm record `vslide1up`.
  **L1748 CN**: 声明 TableGen defm 记录 `vslide1up`。
- **L1749 EN**: Declares TableGen defm record `vfslide1up`.
  **L1749 CN**: 声明 TableGen defm 记录 `vfslide1up`。
- **L1750 EN**: Blank line separating nearby declarations or logic blocks.
  **L1750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1751 EN**: Comment explains nearby logic, constraints, or intent: `16.3.4. Vector Slide1down Instruction`.
  **L1751 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.3.4. Vector Slide1down Instruction`。
- **L1752 EN**: Declares TableGen defm record `vslide1down`.
  **L1752 CN**: 声明 TableGen defm 记录 `vslide1down`。
- **L1753 EN**: Declares TableGen defm record `vfslide1down`.
  **L1753 CN**: 声明 TableGen defm 记录 `vfslide1down`。
- **L1754 EN**: Blank line separating nearby declarations or logic blocks.
  **L1754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1755 EN**: Comment explains nearby logic, constraints, or intent: `16.4. Vector Register Gather Instructions`.
  **L1755 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.4. Vector Register Gather Instructions`。
- **L1756 EN**: Comment explains nearby logic, constraints, or intent: `signed and floating type`.
  **L1756 CN**: 注释解释附近代码的逻辑、约束或设计意图：`signed and floating type`。
- **L1757 EN**: Declares TableGen defm record `vrgather`.
  **L1757 CN**: 声明 TableGen defm 记录 `vrgather`。
- **L1758 EN**: Adds a standalone statement or declaration: `[["vv", "v", "vvUv"]]>;`.
  **L1758 CN**: 添加一条独立语句或声明：`[["vv", "v", "vvUv"]]>;`。
- **L1759 EN**: Declares TableGen defm record `vrgather`.
  **L1759 CN**: 声明 TableGen defm 记录 `vrgather`。
- **L1760 EN**: Adds a standalone statement or declaration: `[["vx", "v", "vvz"]]>;`.
  **L1760 CN**: 添加一条独立语句或声明：`[["vx", "v", "vvz"]]>;`。
- **L1761 EN**: Declares TableGen defm record `vrgatherei16`.
  **L1761 CN**: 声明 TableGen defm 记录 `vrgatherei16`。
- **L1762 EN**: Executes a call or declaration centered on `"vv`.
  **L1762 CN**: 执行以 `"vv` 为核心的调用或声明。
- **L1763 EN**: Comment explains nearby logic, constraints, or intent: `unsigned type`.
  **L1763 CN**: 注释解释附近代码的逻辑、约束或设计意图：`unsigned type`。
- **L1764 EN**: Declares TableGen defm record `vrgather`.
  **L1764 CN**: 声明 TableGen defm 记录 `vrgather`。

### Lines 1765-1792

````tablegen
                                 [["vv", "Uv", "UvUvUv"]]>;
defm vrgather : RVVOutBuiltinSet<"vrgather_vx", "csil",
                                 [["vx", "Uv", "UvUvz"]]>;
defm vrgatherei16 : RVVOutBuiltinSet<"vrgatherei16_vv", "csil",
                                     [["vv", "Uv", "UvUv(Log2EEW:4)Uv"]]>;
}

// 16.5. Vector Compress Instruction
let HasMasked = false,
    UnMaskedPolicyScheme = HasPassthruOperand,
    MaskedPolicyScheme = NonePolicy,
    ManualCodegen = [{
      // insert poison passthru
      if (PolicyAttrs & RVV_VTA)
        Ops.insert(Ops.begin(), llvm::PoisonValue::get(ResultType));
      IntrinsicTypes = {ResultType, Ops.back()->getType()};
    }] in {
  // signed and floating type
  defm vcompress : RVVOutBuiltinSet<"vcompress", "csilxfdy",
                                    [["vm", "v", "vvm"]]>;
  // unsigned type
  defm vcompress : RVVOutBuiltinSet<"vcompress", "csil",
                                    [["vm", "Uv", "UvUvm"]]>;
}

// Miscellaneous
let HasMasked = false, HasVL = false, IRName = "" in {
  let Name = "vreinterpret_v", MaskedPolicyScheme = NonePolicy,
````
- **L1765 EN**: Adds a standalone statement or declaration: `[["vv", "Uv", "UvUvUv"]]>;`.
  **L1765 CN**: 添加一条独立语句或声明：`[["vv", "Uv", "UvUvUv"]]>;`。
- **L1766 EN**: Declares TableGen defm record `vrgather`.
  **L1766 CN**: 声明 TableGen defm 记录 `vrgather`。
- **L1767 EN**: Adds a standalone statement or declaration: `[["vx", "Uv", "UvUvz"]]>;`.
  **L1767 CN**: 添加一条独立语句或声明：`[["vx", "Uv", "UvUvz"]]>;`。
- **L1768 EN**: Declares TableGen defm record `vrgatherei16`.
  **L1768 CN**: 声明 TableGen defm 记录 `vrgatherei16`。
- **L1769 EN**: Executes a call or declaration centered on `"UvUv`.
  **L1769 CN**: 执行以 `"UvUv` 为核心的调用或声明。
- **L1770 EN**: Closes the current lexical scope or compound statement.
  **L1770 CN**: 结束当前词法作用域或复合语句块。
- **L1771 EN**: Blank line separating nearby declarations or logic blocks.
  **L1771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1772 EN**: Comment explains nearby logic, constraints, or intent: `16.5. Vector Compress Instruction`.
  **L1772 CN**: 注释解释附近代码的逻辑、约束或设计意图：`16.5. Vector Compress Instruction`。
- **L1773 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false,`.
  **L1773 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false,`。
- **L1774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = HasPassthruOperand,`.
  **L1774 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = HasPassthruOperand,`。
- **L1775 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L1775 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L1776 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1776 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1777 EN**: Comment explains nearby logic, constraints, or intent: `insert poison passthru`.
  **L1777 CN**: 注释解释附近代码的逻辑、约束或设计意图：`insert poison passthru`。
- **L1778 EN**: Begins a `if` control-flow statement and evaluates its condition.
  **L1778 CN**: 开始 `if` 控制流语句并计算其条件。
- **L1779 EN**: Executes a call or declaration centered on `Ops.insert`.
  **L1779 CN**: 执行以 `Ops.insert` 为核心的调用或声明。
- **L1780 EN**: Executes a call or declaration centered on `Ops.back`.
  **L1780 CN**: 执行以 `Ops.back` 为核心的调用或声明。
- **L1781 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1781 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1782 EN**: Comment explains nearby logic, constraints, or intent: `signed and floating type`.
  **L1782 CN**: 注释解释附近代码的逻辑、约束或设计意图：`signed and floating type`。
- **L1783 EN**: Declares TableGen defm record `vcompress`.
  **L1783 CN**: 声明 TableGen defm 记录 `vcompress`。
- **L1784 EN**: Adds a standalone statement or declaration: `[["vm", "v", "vvm"]]>;`.
  **L1784 CN**: 添加一条独立语句或声明：`[["vm", "v", "vvm"]]>;`。
- **L1785 EN**: Comment explains nearby logic, constraints, or intent: `unsigned type`.
  **L1785 CN**: 注释解释附近代码的逻辑、约束或设计意图：`unsigned type`。
- **L1786 EN**: Declares TableGen defm record `vcompress`.
  **L1786 CN**: 声明 TableGen defm 记录 `vcompress`。
- **L1787 EN**: Adds a standalone statement or declaration: `[["vm", "Uv", "UvUvm"]]>;`.
  **L1787 CN**: 添加一条独立语句或声明：`[["vm", "Uv", "UvUvm"]]>;`。
- **L1788 EN**: Closes the current lexical scope or compound statement.
  **L1788 CN**: 结束当前词法作用域或复合语句块。
- **L1789 EN**: Blank line separating nearby declarations or logic blocks.
  **L1789 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1790 EN**: Comment explains nearby logic, constraints, or intent: `Miscellaneous`.
  **L1790 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Miscellaneous`。
- **L1791 EN**: Assigns a TableGen property that affects following records or inherited fields: `let HasMasked = false, HasVL = false, IRName = "" in {`.
  **L1791 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let HasMasked = false, HasVL = false, IRName = "" in {`。
- **L1792 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vreinterpret_v", MaskedPolicyScheme = NonePolicy,`.
  **L1792 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vreinterpret_v", MaskedPolicyScheme = NonePolicy,`。

### Lines 1793-1820

````tablegen
      ManualCodegen = [{
        return emitRVVReinterpretBuiltin(this, E, ReturnValue, ResultType, ID,
                                         Ops, PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
    // Reinterpret between different type under the same SEW and LMUL
    def vreinterpret_i_u : RVVBuiltin<"Uvv", "vUv", "csil", "v">;
    def vreinterpret_i_f : RVVBuiltin<"Fvv", "vFv", "sil", "v">;
    def vreinterpret_u_i : RVVBuiltin<"vUv", "Uvv", "csil", "Uv">;
    def vreinterpret_u_f : RVVBuiltin<"FvUv", "UvFv", "sil", "Uv">;
    def vreinterpret_f_i : RVVBuiltin<"vFv", "Fvv", "sil", "Fv">;
    def vreinterpret_f_u : RVVBuiltin<"UvFv", "FvUv", "sil", "Fv">;
    def vreinterpret_i_bf16 : RVVBuiltin<"vIv", "Ivv", "y", "Iv">;
    def vreinterpret_u_bf16 : RVVBuiltin<"vUv", "Uvv", "y", "Uv">;
    def vreinterpret_bf16_i : RVVBuiltin<"Ivv", "vIv", "y", "v">;
    def vreinterpret_bf16_u : RVVBuiltin<"Uvv", "vUv", "y", "v">;
    let RequiredFeatures = ["zvfofp8min"] in {
      def vreinterpret_u_f8e4m3 : RVVBuiltin<"vUv", "Uvv", "a", "Uv">;
      def vreinterpret_f8e4m3_u : RVVBuiltin<"Uvv", "vUv", "a", "v">;
      def vreinterpret_u_f8e5m2 : RVVBuiltin<"vUv", "Uvv", "b", "Uv">;
      def vreinterpret_f8e5m2_u : RVVBuiltin<"Uvv", "vUv", "b", "v">;
    }

    // Reinterpret between different SEW under the same LMUL
    foreach dst_sew = ["(FixedSEW:8)", "(FixedSEW:16)", "(FixedSEW:32)",
                       "(FixedSEW:64)"] in {
      def vreinterpret_i_ # dst_sew : RVVBuiltin<"v" # dst_sew # "v",
                                                 dst_sew # "vv", "csil", dst_sew # "v">;
      def vreinterpret_u_ # dst_sew : RVVBuiltin<"Uv" # dst_sew # "Uv",
````
- **L1793 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1793 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1794 EN**: Returns from the current function with `emitRVVReinterpretBuiltin(this, E, ReturnValue, ResultType, ID,`.
  **L1794 CN**: 以 `emitRVVReinterpretBuiltin(this, E, ReturnValue, ResultType, ID,` 从当前函数返回。
- **L1795 EN**: Adds a standalone statement or declaration: `Ops, PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1795 CN**: 添加一条独立语句或声明：`Ops, PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1796 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1796 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1797 EN**: Comment explains nearby logic, constraints, or intent: `Reinterpret between different type under the same SEW and LMUL`.
  **L1797 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reinterpret between different type under the same SEW and LMUL`。
- **L1798 EN**: Declares TableGen def record `vreinterpret_i_u`.
  **L1798 CN**: 声明 TableGen def 记录 `vreinterpret_i_u`。
- **L1799 EN**: Declares TableGen def record `vreinterpret_i_f`.
  **L1799 CN**: 声明 TableGen def 记录 `vreinterpret_i_f`。
- **L1800 EN**: Declares TableGen def record `vreinterpret_u_i`.
  **L1800 CN**: 声明 TableGen def 记录 `vreinterpret_u_i`。
- **L1801 EN**: Declares TableGen def record `vreinterpret_u_f`.
  **L1801 CN**: 声明 TableGen def 记录 `vreinterpret_u_f`。
- **L1802 EN**: Declares TableGen def record `vreinterpret_f_i`.
  **L1802 CN**: 声明 TableGen def 记录 `vreinterpret_f_i`。
- **L1803 EN**: Declares TableGen def record `vreinterpret_f_u`.
  **L1803 CN**: 声明 TableGen def 记录 `vreinterpret_f_u`。
- **L1804 EN**: Declares TableGen def record `vreinterpret_i_bf16`.
  **L1804 CN**: 声明 TableGen def 记录 `vreinterpret_i_bf16`。
- **L1805 EN**: Declares TableGen def record `vreinterpret_u_bf16`.
  **L1805 CN**: 声明 TableGen def 记录 `vreinterpret_u_bf16`。
- **L1806 EN**: Declares TableGen def record `vreinterpret_bf16_i`.
  **L1806 CN**: 声明 TableGen def 记录 `vreinterpret_bf16_i`。
- **L1807 EN**: Declares TableGen def record `vreinterpret_bf16_u`.
  **L1807 CN**: 声明 TableGen def 记录 `vreinterpret_bf16_u`。
- **L1808 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvfofp8min"] in {`.
  **L1808 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvfofp8min"] in {`。
- **L1809 EN**: Declares TableGen def record `vreinterpret_u_f8e4m3`.
  **L1809 CN**: 声明 TableGen def 记录 `vreinterpret_u_f8e4m3`。
- **L1810 EN**: Declares TableGen def record `vreinterpret_f8e4m3_u`.
  **L1810 CN**: 声明 TableGen def 记录 `vreinterpret_f8e4m3_u`。
- **L1811 EN**: Declares TableGen def record `vreinterpret_u_f8e5m2`.
  **L1811 CN**: 声明 TableGen def 记录 `vreinterpret_u_f8e5m2`。
- **L1812 EN**: Declares TableGen def record `vreinterpret_f8e5m2_u`.
  **L1812 CN**: 声明 TableGen def 记录 `vreinterpret_f8e5m2_u`。
- **L1813 EN**: Closes the current lexical scope or compound statement.
  **L1813 CN**: 结束当前词法作用域或复合语句块。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1815 EN**: Comment explains nearby logic, constraints, or intent: `Reinterpret between different SEW under the same LMUL`.
  **L1815 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reinterpret between different SEW under the same LMUL`。
- **L1816 EN**: Starts a TableGen iteration used to generate repeated records: `foreach dst_sew = ["(FixedSEW:8)", "(FixedSEW:16)", "(FixedSEW:32)",`.
  **L1816 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach dst_sew = ["(FixedSEW:8)", "(FixedSEW:16)", "(FixedSEW:32)",`。
- **L1817 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `"(FixedSEW:64)"] in {`.
  **L1817 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`"(FixedSEW:64)"] in {`。
- **L1818 EN**: Declares TableGen def record `vreinterpret_i_ # dst_sew`.
  **L1818 CN**: 声明 TableGen def 记录 `vreinterpret_i_ # dst_sew`。
- **L1819 EN**: Adds a standalone statement or declaration: `dst_sew # "vv", "csil", dst_sew # "v">;`.
  **L1819 CN**: 添加一条独立语句或声明：`dst_sew # "vv", "csil", dst_sew # "v">;`。
- **L1820 EN**: Declares TableGen def record `vreinterpret_u_ # dst_sew`.
  **L1820 CN**: 声明 TableGen def 记录 `vreinterpret_u_ # dst_sew`。

### Lines 1821-1848

````tablegen
                                                 dst_sew # "UvUv", "csil", dst_sew # "Uv">;
    }

    // Existing users of FixedSEW - the reinterpretation between different SEW
    // and same LMUL has the implicit assumption that if FixedSEW is set to the
    // given element width, then the type will be identified as invalid, thus
    // skipping definition of reinterpret of SEW=8 to SEW=8. However this blocks
    // our usage here of defining all possible combinations of a fixed SEW to
    // any boolean. So we need to separately define SEW=8 here.
    // Reinterpret from LMUL=1 integer type to vector boolean type
    def vreintrepret_m1_b8_signed :
        RVVBuiltin<"Svm",
                    "mSv",
                    "c", "m">;
    def vreintrepret_m1_b8_usigned :
        RVVBuiltin<"USvm",
                    "mUSv",
                    "c", "m">;

    // Reinterpret from vector boolean type to LMUL=1 integer type
    def vreintrepret_b8_m1_signed :
        RVVBuiltin<"mSv",
                    "Svm",
                    "c", "Sv">;
    def vreintrepret_b8_m1_usigned :
        RVVBuiltin<"mUSv",
                    "USvm",
                    "c", "USv">;
````
- **L1821 EN**: Adds a standalone statement or declaration: `dst_sew # "UvUv", "csil", dst_sew # "Uv">;`.
  **L1821 CN**: 添加一条独立语句或声明：`dst_sew # "UvUv", "csil", dst_sew # "Uv">;`。
- **L1822 EN**: Closes the current lexical scope or compound statement.
  **L1822 CN**: 结束当前词法作用域或复合语句块。
- **L1823 EN**: Blank line separating nearby declarations or logic blocks.
  **L1823 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1824 EN**: Comment explains nearby logic, constraints, or intent: `Existing users of FixedSEW - the reinterpretation between different SEW`.
  **L1824 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Existing users of FixedSEW - the reinterpretation between different SEW`。
- **L1825 EN**: Comment explains nearby logic, constraints, or intent: `and same LMUL has the implicit assumption that if FixedSEW is set to the`.
  **L1825 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and same LMUL has the implicit assumption that if FixedSEW is set to the`。
- **L1826 EN**: Comment explains nearby logic, constraints, or intent: `given element width, then the type will be identified as invalid, thus`.
  **L1826 CN**: 注释解释附近代码的逻辑、约束或设计意图：`given element width, then the type will be identified as invalid, thus`。
- **L1827 EN**: Comment explains nearby logic, constraints, or intent: `skipping definition of reinterpret of SEW 8 to SEW 8. However this blocks`.
  **L1827 CN**: 注释解释附近代码的逻辑、约束或设计意图：`skipping definition of reinterpret of SEW 8 to SEW 8. However this blocks`。
- **L1828 EN**: Comment explains nearby logic, constraints, or intent: `our usage here of defining all possible combinations of a fixed SEW to`.
  **L1828 CN**: 注释解释附近代码的逻辑、约束或设计意图：`our usage here of defining all possible combinations of a fixed SEW to`。
- **L1829 EN**: Comment explains nearby logic, constraints, or intent: `any boolean. So we need to separately define SEW 8 here.`.
  **L1829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`any boolean. So we need to separately define SEW 8 here.`。
- **L1830 EN**: Comment explains nearby logic, constraints, or intent: `Reinterpret from LMUL 1 integer type to vector boolean type`.
  **L1830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reinterpret from LMUL 1 integer type to vector boolean type`。
- **L1831 EN**: Declares TableGen def record `vreintrepret_m1_b8_signed`.
  **L1831 CN**: 声明 TableGen def 记录 `vreintrepret_m1_b8_signed`。
- **L1832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"Svm",`.
  **L1832 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"Svm",`。
- **L1833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"mSv",`.
  **L1833 CN**: 继续一个多行参数列表、初始化器或聚合项：`"mSv",`。
- **L1834 EN**: Adds a standalone statement or declaration: `"c", "m">;`.
  **L1834 CN**: 添加一条独立语句或声明：`"c", "m">;`。
- **L1835 EN**: Declares TableGen def record `vreintrepret_m1_b8_usigned`.
  **L1835 CN**: 声明 TableGen def 记录 `vreintrepret_m1_b8_usigned`。
- **L1836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"USvm",`.
  **L1836 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"USvm",`。
- **L1837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"mUSv",`.
  **L1837 CN**: 继续一个多行参数列表、初始化器或聚合项：`"mUSv",`。
- **L1838 EN**: Adds a standalone statement or declaration: `"c", "m">;`.
  **L1838 CN**: 添加一条独立语句或声明：`"c", "m">;`。
- **L1839 EN**: Blank line separating nearby declarations or logic blocks.
  **L1839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1840 EN**: Comment explains nearby logic, constraints, or intent: `Reinterpret from vector boolean type to LMUL 1 integer type`.
  **L1840 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reinterpret from vector boolean type to LMUL 1 integer type`。
- **L1841 EN**: Declares TableGen def record `vreintrepret_b8_m1_signed`.
  **L1841 CN**: 声明 TableGen def 记录 `vreintrepret_b8_m1_signed`。
- **L1842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"mSv",`.
  **L1842 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"mSv",`。
- **L1843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"Svm",`.
  **L1843 CN**: 继续一个多行参数列表、初始化器或聚合项：`"Svm",`。
- **L1844 EN**: Adds a standalone statement or declaration: `"c", "Sv">;`.
  **L1844 CN**: 添加一条独立语句或声明：`"c", "Sv">;`。
- **L1845 EN**: Declares TableGen def record `vreintrepret_b8_m1_usigned`.
  **L1845 CN**: 声明 TableGen def 记录 `vreintrepret_b8_m1_usigned`。
- **L1846 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"mUSv",`.
  **L1846 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"mUSv",`。
- **L1847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"USvm",`.
  **L1847 CN**: 继续一个多行参数列表、初始化器或聚合项：`"USvm",`。
- **L1848 EN**: Adds a standalone statement or declaration: `"c", "USv">;`.
  **L1848 CN**: 添加一条独立语句或声明：`"c", "USv">;`。

### Lines 1849-1876

````tablegen

    foreach dst_sew = ["16", "32", "64"] in {
      // Reinterpret from LMUL=1 integer type to vector boolean type
      def vreinterpret_m1_b # dst_sew # _signed:
        RVVBuiltin<"(FixedSEW:" # dst_sew # ")Svm",
                    "m(FixedSEW:" # dst_sew # ")Sv",
                    "c", "m">;
      def vreinterpret_m1_b # dst_sew # _unsigned:
        RVVBuiltin<"(FixedSEW:" # dst_sew # ")USvm",
                    "m(FixedSEW:" # dst_sew # ")USv",
                    "c", "m">;
      // Reinterpret from vector boolean type to LMUL=1 integer type
      def vreinterpret_b # dst_sew # _m1_signed:
        RVVBuiltin<"m(FixedSEW:" # dst_sew # ")Sv",
                    "(FixedSEW:" # dst_sew # ")Svm",
                    "c", "(FixedSEW:" # dst_sew # ")Sv">;
      def vreinterpret_b # dst_sew # _m1_unsigned:
        RVVBuiltin<"m(FixedSEW:" # dst_sew # ")USv",
                    "(FixedSEW:" # dst_sew # ")USvm",
                    "c", "(FixedSEW:" # dst_sew # ")USv">;
    }
  }

  let Name = "vundefined", SupportOverloading = false,
      MaskedPolicyScheme = NonePolicy,
      ManualCodegen = [{
        return llvm::PoisonValue::get(ResultType);
      }] in {
````
- **L1849 EN**: Blank line separating nearby declarations or logic blocks.
  **L1849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1850 EN**: Starts a TableGen iteration used to generate repeated records: `foreach dst_sew = ["16", "32", "64"] in {`.
  **L1850 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach dst_sew = ["16", "32", "64"] in {`。
- **L1851 EN**: Comment explains nearby logic, constraints, or intent: `Reinterpret from LMUL 1 integer type to vector boolean type`.
  **L1851 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reinterpret from LMUL 1 integer type to vector boolean type`。
- **L1852 EN**: Declares TableGen def record `vreinterpret_m1_b # dst_sew # _signed`.
  **L1852 CN**: 声明 TableGen def 记录 `vreinterpret_m1_b # dst_sew # _signed`。
- **L1853 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"(FixedSEW:" # dst_sew # ")Svm",`.
  **L1853 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"(FixedSEW:" # dst_sew # ")Svm",`。
- **L1854 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"m(FixedSEW:" # dst_sew # ")Sv",`.
  **L1854 CN**: 继续一个多行参数列表、初始化器或聚合项：`"m(FixedSEW:" # dst_sew # ")Sv",`。
- **L1855 EN**: Adds a standalone statement or declaration: `"c", "m">;`.
  **L1855 CN**: 添加一条独立语句或声明：`"c", "m">;`。
- **L1856 EN**: Declares TableGen def record `vreinterpret_m1_b # dst_sew # _unsigned`.
  **L1856 CN**: 声明 TableGen def 记录 `vreinterpret_m1_b # dst_sew # _unsigned`。
- **L1857 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"(FixedSEW:" # dst_sew # ")USvm",`.
  **L1857 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"(FixedSEW:" # dst_sew # ")USvm",`。
- **L1858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"m(FixedSEW:" # dst_sew # ")USv",`.
  **L1858 CN**: 继续一个多行参数列表、初始化器或聚合项：`"m(FixedSEW:" # dst_sew # ")USv",`。
- **L1859 EN**: Adds a standalone statement or declaration: `"c", "m">;`.
  **L1859 CN**: 添加一条独立语句或声明：`"c", "m">;`。
- **L1860 EN**: Comment explains nearby logic, constraints, or intent: `Reinterpret from vector boolean type to LMUL 1 integer type`.
  **L1860 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Reinterpret from vector boolean type to LMUL 1 integer type`。
- **L1861 EN**: Declares TableGen def record `vreinterpret_b # dst_sew # _m1_signed`.
  **L1861 CN**: 声明 TableGen def 记录 `vreinterpret_b # dst_sew # _m1_signed`。
- **L1862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"m(FixedSEW:" # dst_sew # ")Sv",`.
  **L1862 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"m(FixedSEW:" # dst_sew # ")Sv",`。
- **L1863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"(FixedSEW:" # dst_sew # ")Svm",`.
  **L1863 CN**: 继续一个多行参数列表、初始化器或聚合项：`"(FixedSEW:" # dst_sew # ")Svm",`。
- **L1864 EN**: Executes a call or declaration centered on `"`.
  **L1864 CN**: 执行以 `"` 为核心的调用或声明。
- **L1865 EN**: Declares TableGen def record `vreinterpret_b # dst_sew # _m1_unsigned`.
  **L1865 CN**: 声明 TableGen def 记录 `vreinterpret_b # dst_sew # _m1_unsigned`。
- **L1866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RVVBuiltin<"m(FixedSEW:" # dst_sew # ")USv",`.
  **L1866 CN**: 继续一个多行参数列表、初始化器或聚合项：`RVVBuiltin<"m(FixedSEW:" # dst_sew # ")USv",`。
- **L1867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"(FixedSEW:" # dst_sew # ")USvm",`.
  **L1867 CN**: 继续一个多行参数列表、初始化器或聚合项：`"(FixedSEW:" # dst_sew # ")USvm",`。
- **L1868 EN**: Executes a call or declaration centered on `"`.
  **L1868 CN**: 执行以 `"` 为核心的调用或声明。
- **L1869 EN**: Closes the current lexical scope or compound statement.
  **L1869 CN**: 结束当前词法作用域或复合语句块。
- **L1870 EN**: Closes the current lexical scope or compound statement.
  **L1870 CN**: 结束当前词法作用域或复合语句块。
- **L1871 EN**: Blank line separating nearby declarations or logic blocks.
  **L1871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1872 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vundefined", SupportOverloading = false,`.
  **L1872 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vundefined", SupportOverloading = false,`。
- **L1873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L1873 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L1874 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1874 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1875 EN**: Returns from the current function with `llvm::PoisonValue::get(ResultType)`.
  **L1875 CN**: 以 `llvm::PoisonValue::get(ResultType)` 从当前函数返回。
- **L1876 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1876 CN**: 继续构造周围的表达式或声明：`}] in {`。

### Lines 1877-1904

````tablegen
    def vundefined : RVVBuiltin<"v", "v", "csilxfdy">;
    def vundefined_u : RVVBuiltin<"Uv", "Uv", "csil">;

    foreach nf = NFList in {
      let NF = nf in {
        defvar T = "(Tuple:" # nf # ")";
        def : RVVBuiltin<T # "v", T # "v", "csilxfdy">;
        def : RVVBuiltin<T # "Uv", T # "Uv", "csil">;
      }
    }

  }

  // LMUL truncation
  // C/C++ Operand: VecTy, IR Operand: VecTy, Index
  let Name = "vlmul_trunc_v", OverloadedName = "vlmul_trunc",
      MaskedPolicyScheme = NonePolicy,
      ManualCodegen = [{ {
        return Builder.CreateExtractVector(ResultType, Ops[0],
                                           ConstantInt::get(Int64Ty, 0));
      } }] in {
    foreach dst_lmul = ["(SFixedLog2LMUL:-3)", "(SFixedLog2LMUL:-2)", "(SFixedLog2LMUL:-1)",
                        "(SFixedLog2LMUL:0)", "(SFixedLog2LMUL:1)", "(SFixedLog2LMUL:2)"] in {
      def vlmul_trunc # dst_lmul : RVVBuiltin<"v" # dst_lmul # "v",
                                              dst_lmul # "vv", "csilxfdy", dst_lmul # "v">;
      def vlmul_trunc_u # dst_lmul : RVVBuiltin<"Uv" # dst_lmul # "Uv",
                                                dst_lmul # "UvUv", "csil", dst_lmul # "Uv">;
    }
````
- **L1877 EN**: Declares TableGen def record `vundefined`.
  **L1877 CN**: 声明 TableGen def 记录 `vundefined`。
- **L1878 EN**: Declares TableGen def record `vundefined_u`.
  **L1878 CN**: 声明 TableGen def 记录 `vundefined_u`。
- **L1879 EN**: Blank line separating nearby declarations or logic blocks.
  **L1879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1880 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L1880 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L1881 EN**: Assigns a TableGen property that affects following records or inherited fields: `let NF = nf in {`.
  **L1881 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let NF = nf in {`。
- **L1882 EN**: Declares TableGen defvar record `T = "`.
  **L1882 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L1883 EN**: Declares TableGen def record `def`.
  **L1883 CN**: 声明 TableGen def 记录 `def`。
- **L1884 EN**: Declares TableGen def record `def`.
  **L1884 CN**: 声明 TableGen def 记录 `def`。
- **L1885 EN**: Closes the current lexical scope or compound statement.
  **L1885 CN**: 结束当前词法作用域或复合语句块。
- **L1886 EN**: Closes the current lexical scope or compound statement.
  **L1886 CN**: 结束当前词法作用域或复合语句块。
- **L1887 EN**: Blank line separating nearby declarations or logic blocks.
  **L1887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1888 EN**: Closes the current lexical scope or compound statement.
  **L1888 CN**: 结束当前词法作用域或复合语句块。
- **L1889 EN**: Blank line separating nearby declarations or logic blocks.
  **L1889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1890 EN**: Comment explains nearby logic, constraints, or intent: `LMUL truncation`.
  **L1890 CN**: 注释解释附近代码的逻辑、约束或设计意图：`LMUL truncation`。
- **L1891 EN**: Comment explains nearby logic, constraints, or intent: `C/C++ Operand: VecTy, IR Operand: VecTy, Index`.
  **L1891 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C/C++ Operand: VecTy, IR Operand: VecTy, Index`。
- **L1892 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vlmul_trunc_v", OverloadedName = "vlmul_trunc",`.
  **L1892 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vlmul_trunc_v", OverloadedName = "vlmul_trunc",`。
- **L1893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L1893 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L1894 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{ {`.
  **L1894 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{ {`。
- **L1895 EN**: Returns from the current function with `Builder.CreateExtractVector(ResultType, Ops[0],`.
  **L1895 CN**: 以 `Builder.CreateExtractVector(ResultType, Ops[0],` 从当前函数返回。
- **L1896 EN**: Executes a call or declaration centered on `ConstantInt::get`.
  **L1896 CN**: 执行以 `ConstantInt::get` 为核心的调用或声明。
- **L1897 EN**: Continues the surrounding expression or declaration: `} }] in {`.
  **L1897 CN**: 继续构造周围的表达式或声明：`} }] in {`。
- **L1898 EN**: Starts a TableGen iteration used to generate repeated records: `foreach dst_lmul = ["(SFixedLog2LMUL:-3)", "(SFixedLog2LMUL:-2)", "(SFixedLog2LMUL:-1)",`.
  **L1898 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach dst_lmul = ["(SFixedLog2LMUL:-3)", "(SFixedLog2LMUL:-2)", "(SFixedLog2LMUL:-1)",`。
- **L1899 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `"(SFixedLog2LMUL:0)", "(SFixedLog2LMUL:1)", "(SFixedLog2LMUL:2)"] in {`.
  **L1899 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`"(SFixedLog2LMUL:0)", "(SFixedLog2LMUL:1)", "(SFixedLog2LMUL:2)"] in {`。
- **L1900 EN**: Declares TableGen def record `vlmul_trunc # dst_lmul`.
  **L1900 CN**: 声明 TableGen def 记录 `vlmul_trunc # dst_lmul`。
- **L1901 EN**: Adds a standalone statement or declaration: `dst_lmul # "vv", "csilxfdy", dst_lmul # "v">;`.
  **L1901 CN**: 添加一条独立语句或声明：`dst_lmul # "vv", "csilxfdy", dst_lmul # "v">;`。
- **L1902 EN**: Declares TableGen def record `vlmul_trunc_u # dst_lmul`.
  **L1902 CN**: 声明 TableGen def 记录 `vlmul_trunc_u # dst_lmul`。
- **L1903 EN**: Adds a standalone statement or declaration: `dst_lmul # "UvUv", "csil", dst_lmul # "Uv">;`.
  **L1903 CN**: 添加一条独立语句或声明：`dst_lmul # "UvUv", "csil", dst_lmul # "Uv">;`。
- **L1904 EN**: Closes the current lexical scope or compound statement.
  **L1904 CN**: 结束当前词法作用域或复合语句块。

### Lines 1905-1932

````tablegen
  }

  // LMUL extension
  // C/C++ Operand: SubVecTy, IR Operand: VecTy, SubVecTy, Index
  let Name = "vlmul_ext_v", OverloadedName = "vlmul_ext",
      MaskedPolicyScheme = NonePolicy,
      ManualCodegen = [{
        return Builder.CreateInsertVector(ResultType,
                                          llvm::PoisonValue::get(ResultType),
                                          Ops[0], ConstantInt::get(Int64Ty, 0));
      }] in {
    foreach dst_lmul = ["(LFixedLog2LMUL:-2)", "(LFixedLog2LMUL:-1)", "(LFixedLog2LMUL:-0)",
                        "(LFixedLog2LMUL:1)", "(LFixedLog2LMUL:2)", "(LFixedLog2LMUL:3)"] in {
      def vlmul_ext # dst_lmul : RVVBuiltin<"v" # dst_lmul # "v",
                                            dst_lmul # "vv", "csilxfdy", dst_lmul # "v">;
      def vlmul_ext_u # dst_lmul : RVVBuiltin<"Uv" # dst_lmul # "Uv",
                                              dst_lmul # "UvUv", "csil", dst_lmul # "Uv">;
    }
  }

  let Name = "vget_v", MaskedPolicyScheme = NonePolicy,
      ManualCodegen = [{
        return emitRVVGetBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                 PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
    foreach dst_lmul = ["(SFixedLog2LMUL:0)", "(SFixedLog2LMUL:1)", "(SFixedLog2LMUL:2)"] in {
      def : RVVBuiltin<"v" # dst_lmul # "v", dst_lmul # "vvKz", "csilxfdy", dst_lmul # "v">;
      def : RVVBuiltin<"Uv" # dst_lmul # "Uv", dst_lmul # "UvUvKz", "csil", dst_lmul # "Uv">;
````
- **L1905 EN**: Closes the current lexical scope or compound statement.
  **L1905 CN**: 结束当前词法作用域或复合语句块。
- **L1906 EN**: Blank line separating nearby declarations or logic blocks.
  **L1906 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1907 EN**: Comment explains nearby logic, constraints, or intent: `LMUL extension`.
  **L1907 CN**: 注释解释附近代码的逻辑、约束或设计意图：`LMUL extension`。
- **L1908 EN**: Comment explains nearby logic, constraints, or intent: `C/C++ Operand: SubVecTy, IR Operand: VecTy, SubVecTy, Index`.
  **L1908 CN**: 注释解释附近代码的逻辑、约束或设计意图：`C/C++ Operand: SubVecTy, IR Operand: VecTy, SubVecTy, Index`。
- **L1909 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vlmul_ext_v", OverloadedName = "vlmul_ext",`.
  **L1909 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vlmul_ext_v", OverloadedName = "vlmul_ext",`。
- **L1910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L1910 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L1911 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1911 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1912 EN**: Returns from the current function with `Builder.CreateInsertVector(ResultType,`.
  **L1912 CN**: 以 `Builder.CreateInsertVector(ResultType,` 从当前函数返回。
- **L1913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm::PoisonValue::get(ResultType),`.
  **L1913 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm::PoisonValue::get(ResultType),`。
- **L1914 EN**: Executes a call or declaration centered on `ConstantInt::get`.
  **L1914 CN**: 执行以 `ConstantInt::get` 为核心的调用或声明。
- **L1915 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1915 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1916 EN**: Starts a TableGen iteration used to generate repeated records: `foreach dst_lmul = ["(LFixedLog2LMUL:-2)", "(LFixedLog2LMUL:-1)", "(LFixedLog2LMUL:-0)",`.
  **L1916 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach dst_lmul = ["(LFixedLog2LMUL:-2)", "(LFixedLog2LMUL:-1)", "(LFixedLog2LMUL:-0)",`。
- **L1917 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `"(LFixedLog2LMUL:1)", "(LFixedLog2LMUL:2)", "(LFixedLog2LMUL:3)"] in {`.
  **L1917 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`"(LFixedLog2LMUL:1)", "(LFixedLog2LMUL:2)", "(LFixedLog2LMUL:3)"] in {`。
- **L1918 EN**: Declares TableGen def record `vlmul_ext # dst_lmul`.
  **L1918 CN**: 声明 TableGen def 记录 `vlmul_ext # dst_lmul`。
- **L1919 EN**: Adds a standalone statement or declaration: `dst_lmul # "vv", "csilxfdy", dst_lmul # "v">;`.
  **L1919 CN**: 添加一条独立语句或声明：`dst_lmul # "vv", "csilxfdy", dst_lmul # "v">;`。
- **L1920 EN**: Declares TableGen def record `vlmul_ext_u # dst_lmul`.
  **L1920 CN**: 声明 TableGen def 记录 `vlmul_ext_u # dst_lmul`。
- **L1921 EN**: Adds a standalone statement or declaration: `dst_lmul # "UvUv", "csil", dst_lmul # "Uv">;`.
  **L1921 CN**: 添加一条独立语句或声明：`dst_lmul # "UvUv", "csil", dst_lmul # "Uv">;`。
- **L1922 EN**: Closes the current lexical scope or compound statement.
  **L1922 CN**: 结束当前词法作用域或复合语句块。
- **L1923 EN**: Closes the current lexical scope or compound statement.
  **L1923 CN**: 结束当前词法作用域或复合语句块。
- **L1924 EN**: Blank line separating nearby declarations or logic blocks.
  **L1924 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1925 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vget_v", MaskedPolicyScheme = NonePolicy,`.
  **L1925 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vget_v", MaskedPolicyScheme = NonePolicy,`。
- **L1926 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1926 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1927 EN**: Returns from the current function with `emitRVVGetBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L1927 CN**: 以 `emitRVVGetBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L1928 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1928 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1929 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1929 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1930 EN**: Starts a TableGen iteration used to generate repeated records: `foreach dst_lmul = ["(SFixedLog2LMUL:0)", "(SFixedLog2LMUL:1)", "(SFixedLog2LMUL:2)"] in {`.
  **L1930 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach dst_lmul = ["(SFixedLog2LMUL:0)", "(SFixedLog2LMUL:1)", "(SFixedLog2LMUL:2)"] in {`。
- **L1931 EN**: Declares TableGen def record `def`.
  **L1931 CN**: 声明 TableGen def 记录 `def`。
- **L1932 EN**: Declares TableGen def record `def`.
  **L1932 CN**: 声明 TableGen def 记录 `def`。

### Lines 1933-1960

````tablegen
    }
    foreach nf = NFList in {
      defvar T = "(Tuple:" # nf # ")";
      def : RVVBuiltin<T # "vv", "v" # T # "vKz", "csilxfdy", "v">;
      def : RVVBuiltin<T # "UvUv", "Uv" # T # "UvKz", "csil", "Uv">;
    }
  }

  let Name = "vset_v", MaskedPolicyScheme = NonePolicy,
      ManualCodegen = [{
        return emitRVVSetBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                 PolicyAttrs, IsMasked, SegInstSEW);
      }] in {
    foreach dst_lmul = ["(LFixedLog2LMUL:1)", "(LFixedLog2LMUL:2)", "(LFixedLog2LMUL:3)"] in {
      def : RVVBuiltin<"v" # dst_lmul # "v", dst_lmul # "v" # dst_lmul # "vKzv", "csilxfdy">;
      def : RVVBuiltin<"Uv" # dst_lmul # "Uv", dst_lmul # "Uv" # dst_lmul #"UvKzUv", "csil">;
    }
    foreach nf = NFList in {
      defvar T = "(Tuple:" # nf # ")";
      def : RVVBuiltin<"v" # T # "v", T # "v" # T # "vKzv", "csilxfdy">;
      def : RVVBuiltin<"Uv" # T # "Uv", T # "Uv" # T # "UvKzUv", "csil">;
    }
  }

  let Name = "vcreate_v",
      UnMaskedPolicyScheme = NonePolicy,
      MaskedPolicyScheme = NonePolicy,
      SupportOverloading = false,
````
- **L1933 EN**: Closes the current lexical scope or compound statement.
  **L1933 CN**: 结束当前词法作用域或复合语句块。
- **L1934 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L1934 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L1935 EN**: Declares TableGen defvar record `T = "`.
  **L1935 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L1936 EN**: Declares TableGen def record `def`.
  **L1936 CN**: 声明 TableGen def 记录 `def`。
- **L1937 EN**: Declares TableGen def record `def`.
  **L1937 CN**: 声明 TableGen def 记录 `def`。
- **L1938 EN**: Closes the current lexical scope or compound statement.
  **L1938 CN**: 结束当前词法作用域或复合语句块。
- **L1939 EN**: Closes the current lexical scope or compound statement.
  **L1939 CN**: 结束当前词法作用域或复合语句块。
- **L1940 EN**: Blank line separating nearby declarations or logic blocks.
  **L1940 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1941 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vset_v", MaskedPolicyScheme = NonePolicy,`.
  **L1941 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vset_v", MaskedPolicyScheme = NonePolicy,`。
- **L1942 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1942 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1943 EN**: Returns from the current function with `emitRVVSetBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L1943 CN**: 以 `emitRVVSetBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L1944 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1944 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1945 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1945 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1946 EN**: Starts a TableGen iteration used to generate repeated records: `foreach dst_lmul = ["(LFixedLog2LMUL:1)", "(LFixedLog2LMUL:2)", "(LFixedLog2LMUL:3)"] in {`.
  **L1946 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach dst_lmul = ["(LFixedLog2LMUL:1)", "(LFixedLog2LMUL:2)", "(LFixedLog2LMUL:3)"] in {`。
- **L1947 EN**: Declares TableGen def record `def`.
  **L1947 CN**: 声明 TableGen def 记录 `def`。
- **L1948 EN**: Declares TableGen def record `def`.
  **L1948 CN**: 声明 TableGen def 记录 `def`。
- **L1949 EN**: Closes the current lexical scope or compound statement.
  **L1949 CN**: 结束当前词法作用域或复合语句块。
- **L1950 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L1950 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L1951 EN**: Declares TableGen defvar record `T = "`.
  **L1951 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L1952 EN**: Declares TableGen def record `def`.
  **L1952 CN**: 声明 TableGen def 记录 `def`。
- **L1953 EN**: Declares TableGen def record `def`.
  **L1953 CN**: 声明 TableGen def 记录 `def`。
- **L1954 EN**: Closes the current lexical scope or compound statement.
  **L1954 CN**: 结束当前词法作用域或复合语句块。
- **L1955 EN**: Closes the current lexical scope or compound statement.
  **L1955 CN**: 结束当前词法作用域或复合语句块。
- **L1956 EN**: Blank line separating nearby declarations or logic blocks.
  **L1956 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1957 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Name = "vcreate_v",`.
  **L1957 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Name = "vcreate_v",`。
- **L1958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnMaskedPolicyScheme = NonePolicy,`.
  **L1958 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnMaskedPolicyScheme = NonePolicy,`。
- **L1959 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `MaskedPolicyScheme = NonePolicy,`.
  **L1959 CN**: 继续一个多行参数列表、初始化器或聚合项：`MaskedPolicyScheme = NonePolicy,`。
- **L1960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `SupportOverloading = false,`.
  **L1960 CN**: 继续一个多行参数列表、初始化器或聚合项：`SupportOverloading = false,`。

### Lines 1961-1988

````tablegen
      ManualCodegen = [{
        return emitRVVCreateBuiltin(this, E, ReturnValue, ResultType, ID, Ops,
                                    PolicyAttrs, IsMasked, SegInstSEW);
      }] in {

    // Since the vcreate_v uses LFixedLog2LMUL, setting the Log2LMUL to [-3] can
    // avoid creating the intrinsics which contain the same name and prototype.
    let Log2LMUL = [-3] in {
      defm : RVVNonTupleVCreateBuiltin<1, [0]>;
      defm : RVVNonTupleVCreateBuiltin<2, [0, 1]>;
      defm : RVVNonTupleVCreateBuiltin<3, [0, 1, 2]>;
    }

    foreach nf = NFList in {
      let NF = nf in {
        defvar T = "(Tuple:" # nf # ")";
        defvar V = VString<nf, /*signed=*/true>.S;
        defvar UV = VString<nf, /*signed=*/false>.S;
        def : RVVBuiltin<T # "v", T # "v" # V, "csilxfdy">;
        def : RVVBuiltin<T # "Uv", T # "Uv" # UV, "csil">;
      }
    }
  }
}

multiclass RVVOutBuiltinSetZvbb {
  let OverloadedName = NAME in
    defm "" : RVVOutBuiltinSet<NAME, "csil", [["v", "v", "vv"],
````
- **L1961 EN**: Continues the surrounding expression or declaration: `ManualCodegen = [{`.
  **L1961 CN**: 继续构造周围的表达式或声明：`ManualCodegen = [{`。
- **L1962 EN**: Returns from the current function with `emitRVVCreateBuiltin(this, E, ReturnValue, ResultType, ID, Ops,`.
  **L1962 CN**: 以 `emitRVVCreateBuiltin(this, E, ReturnValue, ResultType, ID, Ops,` 从当前函数返回。
- **L1963 EN**: Adds a standalone statement or declaration: `PolicyAttrs, IsMasked, SegInstSEW);`.
  **L1963 CN**: 添加一条独立语句或声明：`PolicyAttrs, IsMasked, SegInstSEW);`。
- **L1964 EN**: Continues the surrounding expression or declaration: `}] in {`.
  **L1964 CN**: 继续构造周围的表达式或声明：`}] in {`。
- **L1965 EN**: Blank line separating nearby declarations or logic blocks.
  **L1965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1966 EN**: Comment explains nearby logic, constraints, or intent: `Since the vcreate_v uses LFixedLog2LMUL, setting the Log2LMUL to [-3] can`.
  **L1966 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Since the vcreate_v uses LFixedLog2LMUL, setting the Log2LMUL to [-3] can`。
- **L1967 EN**: Comment explains nearby logic, constraints, or intent: `avoid creating the intrinsics which contain the same name and prototype.`.
  **L1967 CN**: 注释解释附近代码的逻辑、约束或设计意图：`avoid creating the intrinsics which contain the same name and prototype.`。
- **L1968 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Log2LMUL = [-3] in {`.
  **L1968 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Log2LMUL = [-3] in {`。
- **L1969 EN**: Declares TableGen defm record `defm`.
  **L1969 CN**: 声明 TableGen defm 记录 `defm`。
- **L1970 EN**: Declares TableGen defm record `defm`.
  **L1970 CN**: 声明 TableGen defm 记录 `defm`。
- **L1971 EN**: Declares TableGen defm record `defm`.
  **L1971 CN**: 声明 TableGen defm 记录 `defm`。
- **L1972 EN**: Closes the current lexical scope or compound statement.
  **L1972 CN**: 结束当前词法作用域或复合语句块。
- **L1973 EN**: Blank line separating nearby declarations or logic blocks.
  **L1973 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1974 EN**: Starts a TableGen iteration used to generate repeated records: `foreach nf = NFList in {`.
  **L1974 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach nf = NFList in {`。
- **L1975 EN**: Assigns a TableGen property that affects following records or inherited fields: `let NF = nf in {`.
  **L1975 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let NF = nf in {`。
- **L1976 EN**: Declares TableGen defvar record `T = "`.
  **L1976 CN**: 声明 TableGen defvar 记录 `T = "`。
- **L1977 EN**: Declares TableGen defvar record `V = VString`.
  **L1977 CN**: 声明 TableGen defvar 记录 `V = VString`。
- **L1978 EN**: Declares TableGen defvar record `UV = VString`.
  **L1978 CN**: 声明 TableGen defvar 记录 `UV = VString`。
- **L1979 EN**: Declares TableGen def record `def`.
  **L1979 CN**: 声明 TableGen def 记录 `def`。
- **L1980 EN**: Declares TableGen def record `def`.
  **L1980 CN**: 声明 TableGen def 记录 `def`。
- **L1981 EN**: Closes the current lexical scope or compound statement.
  **L1981 CN**: 结束当前词法作用域或复合语句块。
- **L1982 EN**: Closes the current lexical scope or compound statement.
  **L1982 CN**: 结束当前词法作用域或复合语句块。
- **L1983 EN**: Closes the current lexical scope or compound statement.
  **L1983 CN**: 结束当前词法作用域或复合语句块。
- **L1984 EN**: Closes the current lexical scope or compound statement.
  **L1984 CN**: 结束当前词法作用域或复合语句块。
- **L1985 EN**: Blank line separating nearby declarations or logic blocks.
  **L1985 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1986 EN**: Declares TableGen multiclass record `RVVOutBuiltinSetZvbb`.
  **L1986 CN**: 声明 TableGen multiclass 记录 `RVVOutBuiltinSetZvbb`。
- **L1987 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = NAME in`.
  **L1987 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = NAME in`。
- **L1988 EN**: Declares TableGen defm record `""`.
  **L1988 CN**: 声明 TableGen defm 记录 `""`。

### Lines 1989-2016

````tablegen
                                              ["v", "Uv", "UvUv"]]>;
}

multiclass RVVOutBuiltinSetZvk<bit HasVV = 1, bit HasVS = 1> {
  // vaesz only has 'vs' and vgmul only has 'vv' and they do not have ambiguous
  // prototypes like other zvkned instructions (e.g. vaesdf), so we don't
  // need to encode the operand mnemonics into its intrinsic function name.
  if HasVV then {
    defvar name = NAME # !if(!eq(NAME, "vgmul"), "", "_vv");
    let OverloadedName = name in
      defm "" : RVVOutBuiltinSet<NAME # "_vv", "i",
                                 [["vv", "Uv", "UvUvUv"]]>;
  }

  if HasVS then {
    foreach vs2_lmul = ["(SEFixedLog2LMUL:-1)", "(SEFixedLog2LMUL:0)",
                        "(SEFixedLog2LMUL:1)", "(SEFixedLog2LMUL:2)"] in {
    defvar name = NAME # !if(!eq(NAME, "vaesz"), "", "_vs");
    let OverloadedName = name, IRName = NAME # "_vs", Name = NAME # "_vs",
        IntrinsicTypes = [-1, 1] in
      def NAME # vs2_lmul
          : RVVBuiltin<vs2_lmul # "UvUv", "UvUv" # vs2_lmul # "Uv", "i">;
    }
  }
}

multiclass RVVOutOp2BuiltinSetVVZvk<string type_range = "i">
    : RVVOutOp2BuiltinSet<NAME, type_range, [["vv", "Uv", "UvUvUvUv"]]>;
````
- **L1989 EN**: Adds a standalone statement or declaration: `["v", "Uv", "UvUv"]]>;`.
  **L1989 CN**: 添加一条独立语句或声明：`["v", "Uv", "UvUv"]]>;`。
- **L1990 EN**: Closes the current lexical scope or compound statement.
  **L1990 CN**: 结束当前词法作用域或复合语句块。
- **L1991 EN**: Blank line separating nearby declarations or logic blocks.
  **L1991 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1992 EN**: Declares TableGen multiclass record `RVVOutBuiltinSetZvk`.
  **L1992 CN**: 声明 TableGen multiclass 记录 `RVVOutBuiltinSetZvk`。
- **L1993 EN**: Comment explains nearby logic, constraints, or intent: `vaesz only has 'vs' and vgmul only has 'vv' and they do not have ambiguous`.
  **L1993 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vaesz only has 'vs' and vgmul only has 'vv' and they do not have ambiguous`。
- **L1994 EN**: Comment explains nearby logic, constraints, or intent: `prototypes like other zvkned instructions (e.g. vaesdf), so we don't`.
  **L1994 CN**: 注释解释附近代码的逻辑、约束或设计意图：`prototypes like other zvkned instructions (e.g. vaesdf), so we don't`。
- **L1995 EN**: Comment explains nearby logic, constraints, or intent: `need to encode the operand mnemonics into its intrinsic function name.`.
  **L1995 CN**: 注释解释附近代码的逻辑、约束或设计意图：`need to encode the operand mnemonics into its intrinsic function name.`。
- **L1996 EN**: Continues the surrounding expression or declaration: `if HasVV then {`.
  **L1996 CN**: 继续构造周围的表达式或声明：`if HasVV then {`。
- **L1997 EN**: Declares TableGen defvar record `name = NAME # !if`.
  **L1997 CN**: 声明 TableGen defvar 记录 `name = NAME # !if`。
- **L1998 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = name in`.
  **L1998 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = name in`。
- **L1999 EN**: Declares TableGen defm record `""`.
  **L1999 CN**: 声明 TableGen defm 记录 `""`。
- **L2000 EN**: Adds a standalone statement or declaration: `[["vv", "Uv", "UvUvUv"]]>;`.
  **L2000 CN**: 添加一条独立语句或声明：`[["vv", "Uv", "UvUvUv"]]>;`。
- **L2001 EN**: Closes the current lexical scope or compound statement.
  **L2001 CN**: 结束当前词法作用域或复合语句块。
- **L2002 EN**: Blank line separating nearby declarations or logic blocks.
  **L2002 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2003 EN**: Continues the surrounding expression or declaration: `if HasVS then {`.
  **L2003 CN**: 继续构造周围的表达式或声明：`if HasVS then {`。
- **L2004 EN**: Starts a TableGen iteration used to generate repeated records: `foreach vs2_lmul = ["(SEFixedLog2LMUL:-1)", "(SEFixedLog2LMUL:0)",`.
  **L2004 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach vs2_lmul = ["(SEFixedLog2LMUL:-1)", "(SEFixedLog2LMUL:0)",`。
- **L2005 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `"(SEFixedLog2LMUL:1)", "(SEFixedLog2LMUL:2)"] in {`.
  **L2005 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`"(SEFixedLog2LMUL:1)", "(SEFixedLog2LMUL:2)"] in {`。
- **L2006 EN**: Declares TableGen defvar record `name = NAME # !if`.
  **L2006 CN**: 声明 TableGen defvar 记录 `name = NAME # !if`。
- **L2007 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = name, IRName = NAME # "_vs", Name = NAME # "_vs",`.
  **L2007 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = name, IRName = NAME # "_vs", Name = NAME # "_vs",`。
- **L2008 EN**: Continues the surrounding expression or declaration: `IntrinsicTypes = [-1, 1] in`.
  **L2008 CN**: 继续构造周围的表达式或声明：`IntrinsicTypes = [-1, 1] in`。
- **L2009 EN**: Declares TableGen def record `NAME # vs2_lmul`.
  **L2009 CN**: 声明 TableGen def 记录 `NAME # vs2_lmul`。
- **L2010 EN**: Adds a standalone statement or declaration: `: RVVBuiltin<vs2_lmul # "UvUv", "UvUv" # vs2_lmul # "Uv", "i">;`.
  **L2010 CN**: 添加一条独立语句或声明：`: RVVBuiltin<vs2_lmul # "UvUv", "UvUv" # vs2_lmul # "Uv", "i">;`。
- **L2011 EN**: Closes the current lexical scope or compound statement.
  **L2011 CN**: 结束当前词法作用域或复合语句块。
- **L2012 EN**: Closes the current lexical scope or compound statement.
  **L2012 CN**: 结束当前词法作用域或复合语句块。
- **L2013 EN**: Closes the current lexical scope or compound statement.
  **L2013 CN**: 结束当前词法作用域或复合语句块。
- **L2014 EN**: Blank line separating nearby declarations or logic blocks.
  **L2014 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2015 EN**: Declares TableGen multiclass record `RVVOutOp2BuiltinSetVVZvk`.
  **L2015 CN**: 声明 TableGen multiclass 记录 `RVVOutOp2BuiltinSetVVZvk`。
- **L2016 EN**: Adds a standalone statement or declaration: `: RVVOutOp2BuiltinSet<NAME, type_range, [["vv", "Uv", "UvUvUvUv"]]>;`.
  **L2016 CN**: 添加一条独立语句或声明：`: RVVOutOp2BuiltinSet<NAME, type_range, [["vv", "Uv", "UvUvUvUv"]]>;`。

### Lines 2017-2044

````tablegen

multiclass RVVOutOp2BuiltinSetVIZvk<string type_range = "i">
    : RVVOutOp2BuiltinSet<NAME, type_range, [["vi", "Uv", "UvUvUvKz"]]>;

multiclass RVVSignedWidenBinBuiltinSetVwsll
    : RVVWidenBuiltinSet<NAME, "csi",
                         [["vv", "Uw", "UwUvUv"],
                          ["vx", "Uw", "UwUvz"]]>;

let UnMaskedPolicyScheme = HasPassthruOperand in {
  // zvkb
  let RequiredFeatures = ["zvkb"] in {
    defm vandn   : RVVUnsignedBinBuiltinSet;
    defm vbrev8  : RVVOutBuiltinSetZvbb;
    defm vrev8   : RVVOutBuiltinSetZvbb;
    defm vrol    : RVVUnsignedShiftBuiltinSet;
    defm vror    : RVVUnsignedShiftBuiltinSet;
  }

  // zvbb
  let RequiredFeatures = ["zvbb"] in {
    defm vbrev   : RVVOutBuiltinSetZvbb;
    defm vclz    : RVVOutBuiltinSetZvbb;
    defm vctz    : RVVOutBuiltinSetZvbb;
    let IRName = "vcpopv", MaskedIRName = "vcpopv_mask" in
    defm vcpop   : RVVOutBuiltinSetZvbb;
    let OverloadedName = "vwsll" in
    defm vwsll   : RVVSignedWidenBinBuiltinSetVwsll;
````
- **L2017 EN**: Blank line separating nearby declarations or logic blocks.
  **L2017 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2018 EN**: Declares TableGen multiclass record `RVVOutOp2BuiltinSetVIZvk`.
  **L2018 CN**: 声明 TableGen multiclass 记录 `RVVOutOp2BuiltinSetVIZvk`。
- **L2019 EN**: Adds a standalone statement or declaration: `: RVVOutOp2BuiltinSet<NAME, type_range, [["vi", "Uv", "UvUvUvKz"]]>;`.
  **L2019 CN**: 添加一条独立语句或声明：`: RVVOutOp2BuiltinSet<NAME, type_range, [["vi", "Uv", "UvUvUvKz"]]>;`。
- **L2020 EN**: Blank line separating nearby declarations or logic blocks.
  **L2020 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2021 EN**: Declares TableGen multiclass record `RVVSignedWidenBinBuiltinSetVwsll`.
  **L2021 CN**: 声明 TableGen multiclass 记录 `RVVSignedWidenBinBuiltinSetVwsll`。
- **L2022 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVWidenBuiltinSet<NAME, "csi",`.
  **L2022 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVWidenBuiltinSet<NAME, "csi",`。
- **L2023 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[["vv", "Uw", "UwUvUv"],`.
  **L2023 CN**: 继续一个多行参数列表、初始化器或聚合项：`[["vv", "Uw", "UwUvUv"],`。
- **L2024 EN**: Adds a standalone statement or declaration: `["vx", "Uw", "UwUvz"]]>;`.
  **L2024 CN**: 添加一条独立语句或声明：`["vx", "Uw", "UwUvz"]]>;`。
- **L2025 EN**: Blank line separating nearby declarations or logic blocks.
  **L2025 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2026 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L2026 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L2027 EN**: Comment explains nearby logic, constraints, or intent: `zvkb`.
  **L2027 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvkb`。
- **L2028 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvkb"] in {`.
  **L2028 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvkb"] in {`。
- **L2029 EN**: Declares TableGen defm record `vandn`.
  **L2029 CN**: 声明 TableGen defm 记录 `vandn`。
- **L2030 EN**: Declares TableGen defm record `vbrev8`.
  **L2030 CN**: 声明 TableGen defm 记录 `vbrev8`。
- **L2031 EN**: Declares TableGen defm record `vrev8`.
  **L2031 CN**: 声明 TableGen defm 记录 `vrev8`。
- **L2032 EN**: Declares TableGen defm record `vrol`.
  **L2032 CN**: 声明 TableGen defm 记录 `vrol`。
- **L2033 EN**: Declares TableGen defm record `vror`.
  **L2033 CN**: 声明 TableGen defm 记录 `vror`。
- **L2034 EN**: Closes the current lexical scope or compound statement.
  **L2034 CN**: 结束当前词法作用域或复合语句块。
- **L2035 EN**: Blank line separating nearby declarations or logic blocks.
  **L2035 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2036 EN**: Comment explains nearby logic, constraints, or intent: `zvbb`.
  **L2036 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvbb`。
- **L2037 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvbb"] in {`.
  **L2037 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvbb"] in {`。
- **L2038 EN**: Declares TableGen defm record `vbrev`.
  **L2038 CN**: 声明 TableGen defm 记录 `vbrev`。
- **L2039 EN**: Declares TableGen defm record `vclz`.
  **L2039 CN**: 声明 TableGen defm 记录 `vclz`。
- **L2040 EN**: Declares TableGen defm record `vctz`.
  **L2040 CN**: 声明 TableGen defm 记录 `vctz`。
- **L2041 EN**: Assigns a TableGen property that affects following records or inherited fields: `let IRName = "vcpopv", MaskedIRName = "vcpopv_mask" in`.
  **L2041 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let IRName = "vcpopv", MaskedIRName = "vcpopv_mask" in`。
- **L2042 EN**: Declares TableGen defm record `vcpop`.
  **L2042 CN**: 声明 TableGen defm 记录 `vcpop`。
- **L2043 EN**: Assigns a TableGen property that affects following records or inherited fields: `let OverloadedName = "vwsll" in`.
  **L2043 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let OverloadedName = "vwsll" in`。
- **L2044 EN**: Declares TableGen defm record `vwsll`.
  **L2044 CN**: 声明 TableGen defm 记录 `vwsll`。

### Lines 2045-2072

````tablegen
  }

  // zvbc
  let RequiredFeatures = ["zvbc"] in {
    defm vclmul  : RVVInt64BinBuiltinSet;
    defm vclmulh : RVVInt64BinBuiltinSet;
  }

  // zvabd
  let RequiredFeatures = ["zvabd"] in {
    defm vabs : RVVOutBuiltinSet<"vabs", "csil", [["v", "Uv", "Uvv"]]>;
    defm vabd : RVVOutBuiltinSet<"vabd", "cs", [["vv", "Uv", "Uvvv"]]>;
    defm vabdu : RVVOutBuiltinSet<"vabdu", "cs", [["vv", "Uv", "UvUvUv"]]>;
  }
}

let UnMaskedPolicyScheme = HasPolicyOperand, HasMaskedOffOperand = false in {
  let RequiredFeatures = ["zvabd"] in {
    defm vwabda : RVVOutOp1Op2BuiltinSet<"vwabda", "cs",
                                         [["vv", "Uw", "UwUwvv"]]>;
    defm vwabdau : RVVOutOp1Op2BuiltinSet<"vwabdau", "cs",
                                          [["vv", "Uw", "UwUwUvUv"]]>;
  }
}

let UnMaskedPolicyScheme = HasPolicyOperand, HasMasked = false in {
  // zvkg
  let RequiredFeatures = ["zvkg"] in {
````
- **L2045 EN**: Closes the current lexical scope or compound statement.
  **L2045 CN**: 结束当前词法作用域或复合语句块。
- **L2046 EN**: Blank line separating nearby declarations or logic blocks.
  **L2046 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2047 EN**: Comment explains nearby logic, constraints, or intent: `zvbc`.
  **L2047 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvbc`。
- **L2048 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvbc"] in {`.
  **L2048 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvbc"] in {`。
- **L2049 EN**: Declares TableGen defm record `vclmul`.
  **L2049 CN**: 声明 TableGen defm 记录 `vclmul`。
- **L2050 EN**: Declares TableGen defm record `vclmulh`.
  **L2050 CN**: 声明 TableGen defm 记录 `vclmulh`。
- **L2051 EN**: Closes the current lexical scope or compound statement.
  **L2051 CN**: 结束当前词法作用域或复合语句块。
- **L2052 EN**: Blank line separating nearby declarations or logic blocks.
  **L2052 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2053 EN**: Comment explains nearby logic, constraints, or intent: `zvabd`.
  **L2053 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvabd`。
- **L2054 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvabd"] in {`.
  **L2054 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvabd"] in {`。
- **L2055 EN**: Declares TableGen defm record `vabs`.
  **L2055 CN**: 声明 TableGen defm 记录 `vabs`。
- **L2056 EN**: Declares TableGen defm record `vabd`.
  **L2056 CN**: 声明 TableGen defm 记录 `vabd`。
- **L2057 EN**: Declares TableGen defm record `vabdu`.
  **L2057 CN**: 声明 TableGen defm 记录 `vabdu`。
- **L2058 EN**: Closes the current lexical scope or compound statement.
  **L2058 CN**: 结束当前词法作用域或复合语句块。
- **L2059 EN**: Closes the current lexical scope or compound statement.
  **L2059 CN**: 结束当前词法作用域或复合语句块。
- **L2060 EN**: Blank line separating nearby declarations or logic blocks.
  **L2060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2061 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPolicyOperand, HasMaskedOffOperand = false in {`.
  **L2061 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPolicyOperand, HasMaskedOffOperand = false in {`。
- **L2062 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvabd"] in {`.
  **L2062 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvabd"] in {`。
- **L2063 EN**: Declares TableGen defm record `vwabda`.
  **L2063 CN**: 声明 TableGen defm 记录 `vwabda`。
- **L2064 EN**: Adds a standalone statement or declaration: `[["vv", "Uw", "UwUwvv"]]>;`.
  **L2064 CN**: 添加一条独立语句或声明：`[["vv", "Uw", "UwUwvv"]]>;`。
- **L2065 EN**: Declares TableGen defm record `vwabdau`.
  **L2065 CN**: 声明 TableGen defm 记录 `vwabdau`。
- **L2066 EN**: Adds a standalone statement or declaration: `[["vv", "Uw", "UwUwUvUv"]]>;`.
  **L2066 CN**: 添加一条独立语句或声明：`[["vv", "Uw", "UwUwUvUv"]]>;`。
- **L2067 EN**: Closes the current lexical scope or compound statement.
  **L2067 CN**: 结束当前词法作用域或复合语句块。
- **L2068 EN**: Closes the current lexical scope or compound statement.
  **L2068 CN**: 结束当前词法作用域或复合语句块。
- **L2069 EN**: Blank line separating nearby declarations or logic blocks.
  **L2069 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2070 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPolicyOperand, HasMasked = false in {`.
  **L2070 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPolicyOperand, HasMasked = false in {`。
- **L2071 EN**: Comment explains nearby logic, constraints, or intent: `zvkg`.
  **L2071 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvkg`。
- **L2072 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvkg"] in {`.
  **L2072 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvkg"] in {`。

### Lines 2073-2100

````tablegen
    defm vghsh   : RVVOutOp2BuiltinSetVVZvk;
    defm vgmul   : RVVOutBuiltinSetZvk<HasVV=1, HasVS=0>;
  }

  // zvkned
  let RequiredFeatures = ["zvkned"] in {
    defm vaesdf  : RVVOutBuiltinSetZvk;
    defm vaesdm  : RVVOutBuiltinSetZvk;
    defm vaesef  : RVVOutBuiltinSetZvk;
    defm vaesem  : RVVOutBuiltinSetZvk;
    let UnMaskedPolicyScheme = HasPassthruOperand in
    defm vaeskf1 : RVVOutOp1BuiltinSet<"vaeskf1", "i", [["vi", "Uv", "UvUvKz"]]>;
    defm vaeskf2 : RVVOutOp2BuiltinSetVIZvk;
    defm vaesz   : RVVOutBuiltinSetZvk<HasVV=0>;
  }

  // zvknha
  let RequiredFeatures = ["zvknha"] in {
    defm vsha2ch : RVVOutOp2BuiltinSetVVZvk<"i">;
    defm vsha2cl : RVVOutOp2BuiltinSetVVZvk<"i">;
    defm vsha2ms : RVVOutOp2BuiltinSetVVZvk<"i">;
  }

  // zvknhb
  let RequiredFeatures = ["zvknhb"] in {
    defm vsha2ch : RVVOutOp2BuiltinSetVVZvk<"l">;
    defm vsha2cl : RVVOutOp2BuiltinSetVVZvk<"l">;
    defm vsha2ms : RVVOutOp2BuiltinSetVVZvk<"l">;
````
- **L2073 EN**: Declares TableGen defm record `vghsh`.
  **L2073 CN**: 声明 TableGen defm 记录 `vghsh`。
- **L2074 EN**: Declares TableGen defm record `vgmul`.
  **L2074 CN**: 声明 TableGen defm 记录 `vgmul`。
- **L2075 EN**: Closes the current lexical scope or compound statement.
  **L2075 CN**: 结束当前词法作用域或复合语句块。
- **L2076 EN**: Blank line separating nearby declarations or logic blocks.
  **L2076 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2077 EN**: Comment explains nearby logic, constraints, or intent: `zvkned`.
  **L2077 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvkned`。
- **L2078 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvkned"] in {`.
  **L2078 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvkned"] in {`。
- **L2079 EN**: Declares TableGen defm record `vaesdf`.
  **L2079 CN**: 声明 TableGen defm 记录 `vaesdf`。
- **L2080 EN**: Declares TableGen defm record `vaesdm`.
  **L2080 CN**: 声明 TableGen defm 记录 `vaesdm`。
- **L2081 EN**: Declares TableGen defm record `vaesef`.
  **L2081 CN**: 声明 TableGen defm 记录 `vaesef`。
- **L2082 EN**: Declares TableGen defm record `vaesem`.
  **L2082 CN**: 声明 TableGen defm 记录 `vaesem`。
- **L2083 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in`.
  **L2083 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in`。
- **L2084 EN**: Declares TableGen defm record `vaeskf1`.
  **L2084 CN**: 声明 TableGen defm 记录 `vaeskf1`。
- **L2085 EN**: Declares TableGen defm record `vaeskf2`.
  **L2085 CN**: 声明 TableGen defm 记录 `vaeskf2`。
- **L2086 EN**: Declares TableGen defm record `vaesz`.
  **L2086 CN**: 声明 TableGen defm 记录 `vaesz`。
- **L2087 EN**: Closes the current lexical scope or compound statement.
  **L2087 CN**: 结束当前词法作用域或复合语句块。
- **L2088 EN**: Blank line separating nearby declarations or logic blocks.
  **L2088 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2089 EN**: Comment explains nearby logic, constraints, or intent: `zvknha`.
  **L2089 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvknha`。
- **L2090 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvknha"] in {`.
  **L2090 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvknha"] in {`。
- **L2091 EN**: Declares TableGen defm record `vsha2ch`.
  **L2091 CN**: 声明 TableGen defm 记录 `vsha2ch`。
- **L2092 EN**: Declares TableGen defm record `vsha2cl`.
  **L2092 CN**: 声明 TableGen defm 记录 `vsha2cl`。
- **L2093 EN**: Declares TableGen defm record `vsha2ms`.
  **L2093 CN**: 声明 TableGen defm 记录 `vsha2ms`。
- **L2094 EN**: Closes the current lexical scope or compound statement.
  **L2094 CN**: 结束当前词法作用域或复合语句块。
- **L2095 EN**: Blank line separating nearby declarations or logic blocks.
  **L2095 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2096 EN**: Comment explains nearby logic, constraints, or intent: `zvknhb`.
  **L2096 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvknhb`。
- **L2097 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvknhb"] in {`.
  **L2097 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvknhb"] in {`。
- **L2098 EN**: Declares TableGen defm record `vsha2ch`.
  **L2098 CN**: 声明 TableGen defm 记录 `vsha2ch`。
- **L2099 EN**: Declares TableGen defm record `vsha2cl`.
  **L2099 CN**: 声明 TableGen defm 记录 `vsha2cl`。
- **L2100 EN**: Declares TableGen defm record `vsha2ms`.
  **L2100 CN**: 声明 TableGen defm 记录 `vsha2ms`。

### Lines 2101-2128

````tablegen
  }

  // zvksed
  let RequiredFeatures = ["zvksed"] in {
    let UnMaskedPolicyScheme = HasPassthruOperand in
    defm vsm4k   : RVVOutOp1BuiltinSet<"vsm4k", "i", [["vi", "Uv", "UvUvKz"]]>;
    defm vsm4r   : RVVOutBuiltinSetZvk;
  }

  // zvksh
  let RequiredFeatures = ["zvksh"] in {
    defm vsm3c   : RVVOutOp2BuiltinSetVIZvk;
    let UnMaskedPolicyScheme = HasPassthruOperand in
    defm vsm3me  : RVVOutOp1BuiltinSet<"vsm3me", "i", [["vv", "Uv", "UvUvUv"]]>;
  }
}

// Zvdot4a8i
multiclass RVVVDOT4AQBuiltinSet<list<list<string>> suffixes_prototypes> {
  let UnMaskedPolicyScheme = HasPolicyOperand,
      HasMaskedOffOperand = false,
      OverloadedName = NAME,
      Log2LMUL = [-1, 0, 1, 2, 3] in {
    defm NAME : RVVOutOp2BuiltinSet<NAME, "i", suffixes_prototypes>;
  }
}

// Only SEW=32 is defined for zvdot4a8i so far, and since inputs are in fact
````
- **L2101 EN**: Closes the current lexical scope or compound statement.
  **L2101 CN**: 结束当前词法作用域或复合语句块。
- **L2102 EN**: Blank line separating nearby declarations or logic blocks.
  **L2102 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2103 EN**: Comment explains nearby logic, constraints, or intent: `zvksed`.
  **L2103 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvksed`。
- **L2104 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvksed"] in {`.
  **L2104 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvksed"] in {`。
- **L2105 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in`.
  **L2105 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in`。
- **L2106 EN**: Declares TableGen defm record `vsm4k`.
  **L2106 CN**: 声明 TableGen defm 记录 `vsm4k`。
- **L2107 EN**: Declares TableGen defm record `vsm4r`.
  **L2107 CN**: 声明 TableGen defm 记录 `vsm4r`。
- **L2108 EN**: Closes the current lexical scope or compound statement.
  **L2108 CN**: 结束当前词法作用域或复合语句块。
- **L2109 EN**: Blank line separating nearby declarations or logic blocks.
  **L2109 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2110 EN**: Comment explains nearby logic, constraints, or intent: `zvksh`.
  **L2110 CN**: 注释解释附近代码的逻辑、约束或设计意图：`zvksh`。
- **L2111 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvksh"] in {`.
  **L2111 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvksh"] in {`。
- **L2112 EN**: Declares TableGen defm record `vsm3c`.
  **L2112 CN**: 声明 TableGen defm 记录 `vsm3c`。
- **L2113 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in`.
  **L2113 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in`。
- **L2114 EN**: Declares TableGen defm record `vsm3me`.
  **L2114 CN**: 声明 TableGen defm 记录 `vsm3me`。
- **L2115 EN**: Closes the current lexical scope or compound statement.
  **L2115 CN**: 结束当前词法作用域或复合语句块。
- **L2116 EN**: Closes the current lexical scope or compound statement.
  **L2116 CN**: 结束当前词法作用域或复合语句块。
- **L2117 EN**: Blank line separating nearby declarations or logic blocks.
  **L2117 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2118 EN**: Comment explains nearby logic, constraints, or intent: `Zvdot4a8i`.
  **L2118 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvdot4a8i`。
- **L2119 EN**: Declares TableGen multiclass record `RVVVDOT4AQBuiltinSet`.
  **L2119 CN**: 声明 TableGen multiclass 记录 `RVVVDOT4AQBuiltinSet`。
- **L2120 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPolicyOperand,`.
  **L2120 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPolicyOperand,`。
- **L2121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HasMaskedOffOperand = false,`.
  **L2121 CN**: 继续一个多行参数列表、初始化器或聚合项：`HasMaskedOffOperand = false,`。
- **L2122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `OverloadedName = NAME,`.
  **L2122 CN**: 继续一个多行参数列表、初始化器或聚合项：`OverloadedName = NAME,`。
- **L2123 EN**: Continues the surrounding expression or declaration: `Log2LMUL = [-1, 0, 1, 2, 3] in {`.
  **L2123 CN**: 继续构造周围的表达式或声明：`Log2LMUL = [-1, 0, 1, 2, 3] in {`。
- **L2124 EN**: Declares TableGen defm record `NAME`.
  **L2124 CN**: 声明 TableGen defm 记录 `NAME`。
- **L2125 EN**: Closes the current lexical scope or compound statement.
  **L2125 CN**: 结束当前词法作用域或复合语句块。
- **L2126 EN**: Closes the current lexical scope or compound statement.
  **L2126 CN**: 结束当前词法作用域或复合语句块。
- **L2127 EN**: Blank line separating nearby declarations or logic blocks.
  **L2127 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2128 EN**: Comment explains nearby logic, constraints, or intent: `Only SEW 32 is defined for zvdot4a8i so far, and since inputs are in fact`.
  **L2128 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Only SEW 32 is defined for zvdot4a8i so far, and since inputs are in fact`。

### Lines 2129-2156

````tablegen
// four 8-bit integer bundles, we use unsigned type to represent all of them
let RequiredFeatures = ["zvdot4a8i"] in {
  defm vdot4a
      : RVVVDOT4AQBuiltinSet<[["vv", "v", "vvUvUv"],
                              ["vx", "v", "vvUvUe"]]>;
  defm vdot4au
      : RVVVDOT4AQBuiltinSet<[["vv", "Uv", "UvUvUvUv"],
                              ["vx", "Uv", "UvUvUvUe"]]>;
  defm vdot4asu
      : RVVVDOT4AQBuiltinSet<[["vv", "v", "vvUvUv"],
                              ["vx", "v", "vvUvUe"]]>;
  defm vdot4aus : RVVVDOT4AQBuiltinSet<[["vx", "v", "vvUvUe"]]>;
}

// Zvzip
let UnMaskedPolicyScheme = HasPassthruOperand in {
  let RequiredFeatures = ["zvzip"] in {
    // Signed and floating type
    defm vzip : RVVOutOp0BuiltinSet<"vzip", "csilxfdy", [["vv", "d", "dvv"]]>;
    defm vunzipe : RVVOutOp0BuiltinSet<"vunzipe", "csilxfdy", [["v", "v", "vd"]]>;
    defm vunzipo : RVVOutOp0BuiltinSet<"vunzipo", "csilxfdy", [["v", "v", "vd"]]>;
    defm vpaire : RVVOutBuiltinSet<"vpaire", "csilxfdy", [["vv", "v", "vvv"]]>;
    defm vpairo : RVVOutBuiltinSet<"vpairo", "csilxfdy", [["vv", "v", "vvv"]]>;
    // Unsigned
    defm vzip : RVVOutOp0BuiltinSet<"vzip", "csil", [["vv", "Ud", "UdUvUv"]]>;
    defm vunzipe : RVVOutOp0BuiltinSet<"vunzipe", "csil", [["v", "Uv", "UvUd"]]>;
    defm vunzipo : RVVOutOp0BuiltinSet<"vunzipo", "csil", [["v", "Uv", "UvUd"]]>;
    defm vpaire : RVVOutBuiltinSet<"vpaire", "csil", [["vv", "Uv", "UvUvUv"]]>;
````
- **L2129 EN**: Comment explains nearby logic, constraints, or intent: `four 8-bit integer bundles, we use unsigned type to represent all of them`.
  **L2129 CN**: 注释解释附近代码的逻辑、约束或设计意图：`four 8-bit integer bundles, we use unsigned type to represent all of them`。
- **L2130 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvdot4a8i"] in {`.
  **L2130 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvdot4a8i"] in {`。
- **L2131 EN**: Declares TableGen defm record `vdot4a`.
  **L2131 CN**: 声明 TableGen defm 记录 `vdot4a`。
- **L2132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVVDOT4AQBuiltinSet<[["vv", "v", "vvUvUv"],`.
  **L2132 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVVDOT4AQBuiltinSet<[["vv", "v", "vvUvUv"],`。
- **L2133 EN**: Adds a standalone statement or declaration: `["vx", "v", "vvUvUe"]]>;`.
  **L2133 CN**: 添加一条独立语句或声明：`["vx", "v", "vvUvUe"]]>;`。
- **L2134 EN**: Declares TableGen defm record `vdot4au`.
  **L2134 CN**: 声明 TableGen defm 记录 `vdot4au`。
- **L2135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVVDOT4AQBuiltinSet<[["vv", "Uv", "UvUvUvUv"],`.
  **L2135 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVVDOT4AQBuiltinSet<[["vv", "Uv", "UvUvUvUv"],`。
- **L2136 EN**: Adds a standalone statement or declaration: `["vx", "Uv", "UvUvUvUe"]]>;`.
  **L2136 CN**: 添加一条独立语句或声明：`["vx", "Uv", "UvUvUvUe"]]>;`。
- **L2137 EN**: Declares TableGen defm record `vdot4asu`.
  **L2137 CN**: 声明 TableGen defm 记录 `vdot4asu`。
- **L2138 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: RVVVDOT4AQBuiltinSet<[["vv", "v", "vvUvUv"],`.
  **L2138 CN**: 继续一个多行参数列表、初始化器或聚合项：`: RVVVDOT4AQBuiltinSet<[["vv", "v", "vvUvUv"],`。
- **L2139 EN**: Adds a standalone statement or declaration: `["vx", "v", "vvUvUe"]]>;`.
  **L2139 CN**: 添加一条独立语句或声明：`["vx", "v", "vvUvUe"]]>;`。
- **L2140 EN**: Declares TableGen defm record `vdot4aus`.
  **L2140 CN**: 声明 TableGen defm 记录 `vdot4aus`。
- **L2141 EN**: Closes the current lexical scope or compound statement.
  **L2141 CN**: 结束当前词法作用域或复合语句块。
- **L2142 EN**: Blank line separating nearby declarations or logic blocks.
  **L2142 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L2143 EN**: Comment explains nearby logic, constraints, or intent: `Zvzip`.
  **L2143 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Zvzip`。
- **L2144 EN**: Assigns a TableGen property that affects following records or inherited fields: `let UnMaskedPolicyScheme = HasPassthruOperand in {`.
  **L2144 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let UnMaskedPolicyScheme = HasPassthruOperand in {`。
- **L2145 EN**: Assigns a TableGen property that affects following records or inherited fields: `let RequiredFeatures = ["zvzip"] in {`.
  **L2145 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let RequiredFeatures = ["zvzip"] in {`。
- **L2146 EN**: Comment explains nearby logic, constraints, or intent: `Signed and floating type`.
  **L2146 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Signed and floating type`。
- **L2147 EN**: Declares TableGen defm record `vzip`.
  **L2147 CN**: 声明 TableGen defm 记录 `vzip`。
- **L2148 EN**: Declares TableGen defm record `vunzipe`.
  **L2148 CN**: 声明 TableGen defm 记录 `vunzipe`。
- **L2149 EN**: Declares TableGen defm record `vunzipo`.
  **L2149 CN**: 声明 TableGen defm 记录 `vunzipo`。
- **L2150 EN**: Declares TableGen defm record `vpaire`.
  **L2150 CN**: 声明 TableGen defm 记录 `vpaire`。
- **L2151 EN**: Declares TableGen defm record `vpairo`.
  **L2151 CN**: 声明 TableGen defm 记录 `vpairo`。
- **L2152 EN**: Comment explains nearby logic, constraints, or intent: `Unsigned`.
  **L2152 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Unsigned`。
- **L2153 EN**: Declares TableGen defm record `vzip`.
  **L2153 CN**: 声明 TableGen defm 记录 `vzip`。
- **L2154 EN**: Declares TableGen defm record `vunzipe`.
  **L2154 CN**: 声明 TableGen defm 记录 `vunzipe`。
- **L2155 EN**: Declares TableGen defm record `vunzipo`.
  **L2155 CN**: 声明 TableGen defm 记录 `vunzipo`。
- **L2156 EN**: Declares TableGen defm record `vpaire`.
  **L2156 CN**: 声明 TableGen defm 记录 `vpaire`。

### Lines 2157-2159

````tablegen
    defm vpairo : RVVOutBuiltinSet<"vpairo", "csil", [["vv", "Uv", "UvUvUv"]]>;
  }
}
````
- **L2157 EN**: Declares TableGen defm record `vpairo`.
  **L2157 CN**: 声明 TableGen defm 记录 `vpairo`。
- **L2158 EN**: Closes the current lexical scope or compound statement.
  **L2158 CN**: 结束当前词法作用域或复合语句块。
- **L2159 EN**: Closes the current lexical scope or compound statement.
  **L2159 CN**: 结束当前词法作用域或复合语句块。

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
- **Indexing and tooling / 索引与工具支持**
  - **EN**: Exposes APIs used by source indexing, diagnostics retrieval, and IDE tooling.
  - **CN**: 暴露源码索引、诊断提取与 IDE 工具使用的 API。
- **RISC-V target support / RISC-V 目标支持**
  - **EN**: Captures RISC-V-specific builtin, vector, or target-feature information.
  - **CN**: 刻画 RISC-V 专用 builtin、向量或目标特性信息。
- **Macro-driven generation / 宏驱动生成**
  - **EN**: Relies on macros or generated include fragments to keep large definition sets synchronized.
  - **CN**: 依赖宏或生成式 include 片段来同步大规模定义集合。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: `__riscv_vlenb()`, `__riscv_vsetvl_e8mf4(avl)`, `__riscv_vsetvl_e8mf2(avl)`, `__riscv_vsetvl_e8m1(avl)`, `__riscv_vsetvl_e8m2(avl)`, `__riscv_vsetvl_e8m4(avl)`, `__riscv_vsetvl_e8m8(avl)`, `__riscv_vsetvl_e16mf2(avl)`, `__riscv_vsetvl_e16m1(avl)`, `__riscv_vsetvl_e16m2(avl)`, `__riscv_vsetvl_e16m4(avl)`, `__riscv_vsetvl_e16m8(avl)`, `__riscv_vsetvl_e32m1(avl)`, `__riscv_vsetvl_e32m2(avl)`, `__riscv_vsetvl_e32m4(avl)`, `__riscv_vsetvl_e32m8(avl)`, `__riscv_vsetvl_e8mf8(avl)`, `__riscv_vsetvl_e16mf4(avl)`, `__riscv_vsetvl_e32mf2(avl)`, `__riscv_vsetvl_e64m1(avl)`, `__riscv_vsetvl_e64m2(avl)`, `__riscv_vsetvl_e64m4(avl)`, `__riscv_vsetvl_e64m8(avl)`, `__riscv_vsetvlmax_e8mf4()`
- **Types / 类型**: `IsFloat`, `RVVVLEMaskBuiltin`, `RVVVSEMaskBuiltin`, `PVString`, `VString`, `FixedVString`, `__RISCV_VXRM`, `__RISCV_FRM`
- **Functions or callables / 函数或可调用对象**: `or`, `not`, `Masked:`, `Unmasked:`, `Vector`, `cond`, `eq`, `interleave`, `shl`, `Operand:`, `insert`, `getType`
- **TableGen records / TableGen 记录**: `TypeList`, `EEWList`, `IsFloat`, `RVVVLEMaskBuiltin`, `RVVVLEBuiltin`, `RVVVLEFFBuiltin`, `RVVVLSEBuiltin`, `RVVIndexedLoad`, `eew`, `eew_type`, `eew64`, `eew64_type`, `RVVVSEMaskBuiltin`, `RVVVSEBuiltin`, `RVVVSSEBuiltin`, `RVVIndexedStore`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
