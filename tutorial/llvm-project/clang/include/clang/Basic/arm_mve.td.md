# arm_mve.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/arm_mve.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: ACLE intrinsic functions for MVE architecture.
- **Purpose (CN)**: 声明与 `arm_mve` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1599

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//===- arm_mve.td - ACLE intrinsic functions for MVE architecture ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the set of ACLE-specified source-level intrinsic
// functions wrapping the MVE vector instruction set and scalar shift
// operations.
//
// Refer to comments in arm_mve_defs.td for the infrastructure used in
// here, and to MveEmitter.cpp for how those are used in turn to
// generate code.
//
//===----------------------------------------------------------------------===//

include "arm_mve_defs.td"

let params = T.Usual in
foreach n = [ 2, 4 ] in {
  def "vst"#n#"q": Intrinsic<Void, (args Ptr<Scalar>, MultiVector<n>),
                             (CustomCodegen<"VST24"> n:$NumVectors,
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the set of ACLE-specified source-level intrinsic`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the set of ACLE-specified source-level intrinsic`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `functions wrapping the MVE vector instruction set and scalar shift`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`functions wrapping the MVE vector instruction set and scalar shift`。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `operations.`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operations.`。
- **L12 EN**: Separator comment used for visual grouping.
  **L12 CN**: 用于视觉分组的分隔注释。
- **L13 EN**: Comment explains nearby logic, constraints, or intent: `Refer to comments in arm_mve_defs.td for the infrastructure used in`.
  **L13 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Refer to comments in arm_mve_defs.td for the infrastructure used in`。
- **L14 EN**: Comment explains nearby logic, constraints, or intent: `here, and to MveEmitter.cpp for how those are used in turn to`.
  **L14 CN**: 注释解释附近代码的逻辑、约束或设计意图：`here, and to MveEmitter.cpp for how those are used in turn to`。
- **L15 EN**: Comment explains nearby logic, constraints, or intent: `generate code.`.
  **L15 CN**: 注释解释附近代码的逻辑、约束或设计意图：`generate code.`。
- **L16 EN**: Separator comment used for visual grouping.
  **L16 CN**: 用于视觉分组的分隔注释。
- **L17 EN**: Banner comment marking a file or section boundary.
  **L17 CN**: 横幅注释，用于标记文件或章节边界。
- **L18 EN**: Blank line separating nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L19 EN**: Includes TableGen file `"arm_mve_defs.td"` so later records can reuse shared definitions.
  **L19 CN**: 引入 TableGen 文件 `"arm_mve_defs.td"`，以便后续记录复用共享定义。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L21 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Usual in`.
  **L21 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Usual in`。
- **L22 EN**: Starts a TableGen iteration used to generate repeated records: `foreach n = [ 2, 4 ] in {`.
  **L22 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach n = [ 2, 4 ] in {`。
- **L23 EN**: Declares TableGen def record `"vst"#n#"q"`.
  **L23 CN**: 声明 TableGen def 记录 `"vst"#n#"q"`。
- **L24 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(CustomCodegen<"VST24"> n:$NumVectors,`.
  **L24 CN**: 继续一个多行参数列表、初始化器或聚合项：`(CustomCodegen<"VST24"> n:$NumVectors,`。

### Lines 25-48

````tablegen
                              "Intrinsic::arm_mve_vst"#n#"q":$IRIntr)>;
  def "vld"#n#"q": Intrinsic<MultiVector<n>, (args CPtr<Scalar>),
                             (CustomCodegen<"VLD24"> n:$NumVectors,
                              "Intrinsic::arm_mve_vld"#n#"q":$IRIntr)>;
}

multiclass bit_op_fp<IRBuilder bitop> {
def "": Intrinsic<Vector, (args Vector:$a, Vector:$b),
   (bitcast (bitop (bitcast $a, UVector), (bitcast $b, UVector)), Vector)>;
}

multiclass bit_op_fp_with_inv<IRBuilder bitop> {
def "": Intrinsic<Vector, (args Vector:$a, Vector:$b),
   (bitcast (bitop (bitcast $a, UVector), (not (bitcast $b, UVector))), Vector)>;
}

let params = T.Signed in {
def vqaddq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                              (IRIntBase<"sadd_sat", [Vector]> $a, $b)>;
def vqsubq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                              (IRIntBase<"ssub_sat", [Vector]> $a, $b)>;
let pnt = PNT_NType in {
  def vqaddq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                  (IRIntBase<"sadd_sat", [Vector]> $a, (splat $b))>;
````
- **L25 EN**: Adds a standalone statement or declaration: `"Intrinsic::arm_mve_vst"#n#"q":$IRIntr)>;`.
  **L25 CN**: 添加一条独立语句或声明：`"Intrinsic::arm_mve_vst"#n#"q":$IRIntr)>;`。
- **L26 EN**: Declares TableGen def record `"vld"#n#"q"`.
  **L26 CN**: 声明 TableGen def 记录 `"vld"#n#"q"`。
- **L27 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(CustomCodegen<"VLD24"> n:$NumVectors,`.
  **L27 CN**: 继续一个多行参数列表、初始化器或聚合项：`(CustomCodegen<"VLD24"> n:$NumVectors,`。
- **L28 EN**: Adds a standalone statement or declaration: `"Intrinsic::arm_mve_vld"#n#"q":$IRIntr)>;`.
  **L28 CN**: 添加一条独立语句或声明：`"Intrinsic::arm_mve_vld"#n#"q":$IRIntr)>;`。
- **L29 EN**: Closes the current lexical scope or compound statement.
  **L29 CN**: 结束当前词法作用域或复合语句块。
- **L30 EN**: Blank line separating nearby declarations or logic blocks.
  **L30 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L31 EN**: Declares TableGen multiclass record `bit_op_fp`.
  **L31 CN**: 声明 TableGen multiclass 记录 `bit_op_fp`。
- **L32 EN**: Declares TableGen def record `""`.
  **L32 CN**: 声明 TableGen def 记录 `""`。
- **L33 EN**: Executes a call or declaration centered on `statement`.
  **L33 CN**: 执行以 `statement` 为核心的调用或声明。
- **L34 EN**: Closes the current lexical scope or compound statement.
  **L34 CN**: 结束当前词法作用域或复合语句块。
- **L35 EN**: Blank line separating nearby declarations or logic blocks.
  **L35 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L36 EN**: Declares TableGen multiclass record `bit_op_fp_with_inv`.
  **L36 CN**: 声明 TableGen multiclass 记录 `bit_op_fp_with_inv`。
- **L37 EN**: Declares TableGen def record `""`.
  **L37 CN**: 声明 TableGen def 记录 `""`。
- **L38 EN**: Executes a call or declaration centered on `statement`.
  **L38 CN**: 执行以 `statement` 为核心的调用或声明。
- **L39 EN**: Closes the current lexical scope or compound statement.
  **L39 CN**: 结束当前词法作用域或复合语句块。
- **L40 EN**: Blank line separating nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L41 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L41 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L42 EN**: Declares TableGen def record `vqaddq`.
  **L42 CN**: 声明 TableGen def 记录 `vqaddq`。
- **L43 EN**: Executes a call or declaration centered on `statement`.
  **L43 CN**: 执行以 `statement` 为核心的调用或声明。
- **L44 EN**: Declares TableGen def record `vqsubq`.
  **L44 CN**: 声明 TableGen def 记录 `vqsubq`。
- **L45 EN**: Executes a call or declaration centered on `statement`.
  **L45 CN**: 执行以 `statement` 为核心的调用或声明。
- **L46 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L46 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L47 EN**: Declares TableGen def record `vqaddq_n`.
  **L47 CN**: 声明 TableGen def 记录 `vqaddq_n`。
- **L48 EN**: Executes a call or declaration centered on `statement`.
  **L48 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 49-72

````tablegen
  def vqsubq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                  (IRIntBase<"ssub_sat", [Vector]> $a, (splat $b))>;
}
}
let params = T.Unsigned in {
def vqaddq_u: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                (IRIntBase<"uadd_sat", [Vector]> $a, $b)>,
              NameOverride<"vqaddq">;
def vqsubq_u: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                (IRIntBase<"usub_sat", [Vector]> $a, $b)>,
              NameOverride<"vqsubq">;
let pnt = PNT_NType in {
  def vqaddq_u_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                    (IRIntBase<"uadd_sat", [Vector]> $a, (splat $b))>,
                  NameOverride<"vqaddq_n">;
  def vqsubq_u_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                    (IRIntBase<"usub_sat", [Vector]> $a, (splat $b))>,
                  NameOverride<"vqsubq_n">;
}
}

// Some intrinsics below are implemented not as IR fragments, but as
// special-purpose IR intrinsics. This is because such a general form
// (such as NEON uses) required a variable-width vector size, and we are
````
- **L49 EN**: Declares TableGen def record `vqsubq_n`.
  **L49 CN**: 声明 TableGen def 记录 `vqsubq_n`。
- **L50 EN**: Executes a call or declaration centered on `statement`.
  **L50 CN**: 执行以 `statement` 为核心的调用或声明。
- **L51 EN**: Closes the current lexical scope or compound statement.
  **L51 CN**: 结束当前词法作用域或复合语句块。
- **L52 EN**: Closes the current lexical scope or compound statement.
  **L52 CN**: 结束当前词法作用域或复合语句块。
- **L53 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Unsigned in {`.
  **L53 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Unsigned in {`。
- **L54 EN**: Declares TableGen def record `vqaddq_u`.
  **L54 CN**: 声明 TableGen def 记录 `vqaddq_u`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRIntBase<"uadd_sat", [Vector]> $a, $b)>,`.
  **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRIntBase<"uadd_sat", [Vector]> $a, $b)>,`。
- **L56 EN**: Adds a standalone statement or declaration: `NameOverride<"vqaddq">;`.
  **L56 CN**: 添加一条独立语句或声明：`NameOverride<"vqaddq">;`。
- **L57 EN**: Declares TableGen def record `vqsubq_u`.
  **L57 CN**: 声明 TableGen def 记录 `vqsubq_u`。
- **L58 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRIntBase<"usub_sat", [Vector]> $a, $b)>,`.
  **L58 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRIntBase<"usub_sat", [Vector]> $a, $b)>,`。
- **L59 EN**: Adds a standalone statement or declaration: `NameOverride<"vqsubq">;`.
  **L59 CN**: 添加一条独立语句或声明：`NameOverride<"vqsubq">;`。
- **L60 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L60 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L61 EN**: Declares TableGen def record `vqaddq_u_n`.
  **L61 CN**: 声明 TableGen def 记录 `vqaddq_u_n`。
- **L62 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRIntBase<"uadd_sat", [Vector]> $a, (splat $b))>,`.
  **L62 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRIntBase<"uadd_sat", [Vector]> $a, (splat $b))>,`。
- **L63 EN**: Adds a standalone statement or declaration: `NameOverride<"vqaddq_n">;`.
  **L63 CN**: 添加一条独立语句或声明：`NameOverride<"vqaddq_n">;`。
- **L64 EN**: Declares TableGen def record `vqsubq_u_n`.
  **L64 CN**: 声明 TableGen def 记录 `vqsubq_u_n`。
- **L65 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRIntBase<"usub_sat", [Vector]> $a, (splat $b))>,`.
  **L65 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRIntBase<"usub_sat", [Vector]> $a, (splat $b))>,`。
- **L66 EN**: Adds a standalone statement or declaration: `NameOverride<"vqsubq_n">;`.
  **L66 CN**: 添加一条独立语句或声明：`NameOverride<"vqsubq_n">;`。
- **L67 EN**: Closes the current lexical scope or compound statement.
  **L67 CN**: 结束当前词法作用域或复合语句块。
- **L68 EN**: Closes the current lexical scope or compound statement.
  **L68 CN**: 结束当前词法作用域或复合语句块。
- **L69 EN**: Blank line separating nearby declarations or logic blocks.
  **L69 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L70 EN**: Comment explains nearby logic, constraints, or intent: `Some intrinsics below are implemented not as IR fragments, but as`.
  **L70 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Some intrinsics below are implemented not as IR fragments, but as`。
- **L71 EN**: Comment explains nearby logic, constraints, or intent: `special-purpose IR intrinsics. This is because such a general form`.
  **L71 CN**: 注释解释附近代码的逻辑、约束或设计意图：`special-purpose IR intrinsics. This is because such a general form`。
- **L72 EN**: Comment explains nearby logic, constraints, or intent: `(such as NEON uses) required a variable-width vector size, and we are`.
  **L72 CN**: 注释解释附近代码的逻辑、约束或设计意图：`(such as NEON uses) required a variable-width vector size, and we are`。

### Lines 73-96

````tablegen
// restricted to 128 bit. Although we can possibly get clever with lane
// operations, the consequent IR representation would be very hard to
// write sensibly. In particular, doubling a vector's width would be a
// mess. Other intrinsics just don't translate nicely into IR.
let params = T.Int in {
def vaddq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (add $a, $b)>;
def vhaddq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                              (IRInt<"vhadd", [Vector]> $a, $b, (unsignedflag Scalar))>;
def vrhaddq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                               (IRInt<"vrhadd", [Vector]> $a, $b, (unsignedflag Scalar))>;
def vandq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (and $a, $b)>;
def vbicq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (and $a, (not $b))>;
def veorq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (xor $a, $b)>;
def vornq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (or $a, (not $b))>;
def vorrq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (or $a, $b)>;
def vsubq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (sub $a, $b)>;
def vhsubq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                              (IRInt<"vhsub", [Vector]> $a, $b, (unsignedflag Scalar))>;
def vmulq: Intrinsic<Vector, (args Vector:$a, Vector:$b), (mul $a, $b)>;
def vmulhq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                              (IRInt<"vmulh", [Vector]> $a, $b, (unsignedflag Scalar))>;
def vrmulhq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                               (IRInt<"vrmulh", [Vector]> $a, $b, (unsignedflag Scalar))>;
def vmullbq_int: Intrinsic<DblVector, (args Vector:$a, Vector:$b),
````
- **L73 EN**: Comment explains nearby logic, constraints, or intent: `restricted to 128 bit. Although we can possibly get clever with lane`.
  **L73 CN**: 注释解释附近代码的逻辑、约束或设计意图：`restricted to 128 bit. Although we can possibly get clever with lane`。
- **L74 EN**: Comment explains nearby logic, constraints, or intent: `operations, the consequent IR representation would be very hard to`.
  **L74 CN**: 注释解释附近代码的逻辑、约束或设计意图：`operations, the consequent IR representation would be very hard to`。
- **L75 EN**: Comment explains nearby logic, constraints, or intent: `write sensibly. In particular, doubling a vector's width would be a`.
  **L75 CN**: 注释解释附近代码的逻辑、约束或设计意图：`write sensibly. In particular, doubling a vector's width would be a`。
- **L76 EN**: Comment explains nearby logic, constraints, or intent: `mess. Other intrinsics just don't translate nicely into IR.`.
  **L76 CN**: 注释解释附近代码的逻辑、约束或设计意图：`mess. Other intrinsics just don't translate nicely into IR.`。
- **L77 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L77 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L78 EN**: Declares TableGen def record `vaddq`.
  **L78 CN**: 声明 TableGen def 记录 `vaddq`。
- **L79 EN**: Declares TableGen def record `vhaddq`.
  **L79 CN**: 声明 TableGen def 记录 `vhaddq`。
- **L80 EN**: Executes a call or declaration centered on `statement`.
  **L80 CN**: 执行以 `statement` 为核心的调用或声明。
- **L81 EN**: Declares TableGen def record `vrhaddq`.
  **L81 CN**: 声明 TableGen def 记录 `vrhaddq`。
- **L82 EN**: Executes a call or declaration centered on `statement`.
  **L82 CN**: 执行以 `statement` 为核心的调用或声明。
- **L83 EN**: Declares TableGen def record `vandq`.
  **L83 CN**: 声明 TableGen def 记录 `vandq`。
- **L84 EN**: Declares TableGen def record `vbicq`.
  **L84 CN**: 声明 TableGen def 记录 `vbicq`。
- **L85 EN**: Declares TableGen def record `veorq`.
  **L85 CN**: 声明 TableGen def 记录 `veorq`。
- **L86 EN**: Declares TableGen def record `vornq`.
  **L86 CN**: 声明 TableGen def 记录 `vornq`。
- **L87 EN**: Declares TableGen def record `vorrq`.
  **L87 CN**: 声明 TableGen def 记录 `vorrq`。
- **L88 EN**: Declares TableGen def record `vsubq`.
  **L88 CN**: 声明 TableGen def 记录 `vsubq`。
- **L89 EN**: Declares TableGen def record `vhsubq`.
  **L89 CN**: 声明 TableGen def 记录 `vhsubq`。
- **L90 EN**: Executes a call or declaration centered on `statement`.
  **L90 CN**: 执行以 `statement` 为核心的调用或声明。
- **L91 EN**: Declares TableGen def record `vmulq`.
  **L91 CN**: 声明 TableGen def 记录 `vmulq`。
- **L92 EN**: Declares TableGen def record `vmulhq`.
  **L92 CN**: 声明 TableGen def 记录 `vmulhq`。
- **L93 EN**: Executes a call or declaration centered on `statement`.
  **L93 CN**: 执行以 `statement` 为核心的调用或声明。
- **L94 EN**: Declares TableGen def record `vrmulhq`.
  **L94 CN**: 声明 TableGen def 记录 `vrmulhq`。
- **L95 EN**: Executes a call or declaration centered on `statement`.
  **L95 CN**: 执行以 `statement` 为核心的调用或声明。
- **L96 EN**: Declares TableGen def record `vmullbq_int`.
  **L96 CN**: 声明 TableGen def 记录 `vmullbq_int`。

### Lines 97-120

````tablegen
                                      (IRInt<"vmull", [DblVector, Vector]>
                                       $a, $b, (unsignedflag Scalar), 0)>;
def vmulltq_int: Intrinsic<DblVector, (args Vector:$a, Vector:$b),
                                      (IRInt<"vmull", [DblVector, Vector]>
                                       $a, $b, (unsignedflag Scalar), 1)>;
let pnt = PNT_NType in {
  def vaddq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                 (add $a, (splat $b))>;
  def vsubq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                 (sub $a, (splat $b))>;
  def vmulq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                 (mul $a, (splat $b))>;
  def vhaddq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                  (IRInt<"vhadd", [Vector]> $a, (splat $b),
                                  (unsignedflag Scalar))>;
  def vhsubq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                  (IRInt<"vhsub", [Vector]> $a, (splat $b),
                                  (unsignedflag Scalar))>;
}
}
let params = T.Signed in {
def vqdmulhq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                (IRInt<"vqdmulh", [Vector]> $a, $b)>;
def vqrdmulhq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
````
- **L97 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmull", [DblVector, Vector]>`.
  **L97 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmull", [DblVector, Vector]>`。
- **L98 EN**: Executes a call or declaration centered on `$b,`.
  **L98 CN**: 执行以 `$b,` 为核心的调用或声明。
- **L99 EN**: Declares TableGen def record `vmulltq_int`.
  **L99 CN**: 声明 TableGen def 记录 `vmulltq_int`。
- **L100 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmull", [DblVector, Vector]>`.
  **L100 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmull", [DblVector, Vector]>`。
- **L101 EN**: Executes a call or declaration centered on `$b,`.
  **L101 CN**: 执行以 `$b,` 为核心的调用或声明。
- **L102 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L102 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L103 EN**: Declares TableGen def record `vaddq_n`.
  **L103 CN**: 声明 TableGen def 记录 `vaddq_n`。
- **L104 EN**: Executes a call or declaration centered on `statement`.
  **L104 CN**: 执行以 `statement` 为核心的调用或声明。
- **L105 EN**: Declares TableGen def record `vsubq_n`.
  **L105 CN**: 声明 TableGen def 记录 `vsubq_n`。
- **L106 EN**: Executes a call or declaration centered on `statement`.
  **L106 CN**: 执行以 `statement` 为核心的调用或声明。
- **L107 EN**: Declares TableGen def record `vmulq_n`.
  **L107 CN**: 声明 TableGen def 记录 `vmulq_n`。
- **L108 EN**: Executes a call or declaration centered on `statement`.
  **L108 CN**: 执行以 `statement` 为核心的调用或声明。
- **L109 EN**: Declares TableGen def record `vhaddq_n`.
  **L109 CN**: 声明 TableGen def 记录 `vhaddq_n`。
- **L110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vhadd", [Vector]> $a, (splat $b),`.
  **L110 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vhadd", [Vector]> $a, (splat $b),`。
- **L111 EN**: Executes a call or declaration centered on `statement`.
  **L111 CN**: 执行以 `statement` 为核心的调用或声明。
- **L112 EN**: Declares TableGen def record `vhsubq_n`.
  **L112 CN**: 声明 TableGen def 记录 `vhsubq_n`。
- **L113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vhsub", [Vector]> $a, (splat $b),`.
  **L113 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vhsub", [Vector]> $a, (splat $b),`。
- **L114 EN**: Executes a call or declaration centered on `statement`.
  **L114 CN**: 执行以 `statement` 为核心的调用或声明。
- **L115 EN**: Closes the current lexical scope or compound statement.
  **L115 CN**: 结束当前词法作用域或复合语句块。
- **L116 EN**: Closes the current lexical scope or compound statement.
  **L116 CN**: 结束当前词法作用域或复合语句块。
- **L117 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L117 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L118 EN**: Declares TableGen def record `vqdmulhq`.
  **L118 CN**: 声明 TableGen def 记录 `vqdmulhq`。
- **L119 EN**: Executes a call or declaration centered on `statement`.
  **L119 CN**: 执行以 `statement` 为核心的调用或声明。
- **L120 EN**: Declares TableGen def record `vqrdmulhq`.
  **L120 CN**: 声明 TableGen def 记录 `vqrdmulhq`。

### Lines 121-144

````tablegen
                                 (IRInt<"vqrdmulh", [Vector]> $a, $b)>;
let pnt = PNT_NType in {
  def vqdmulhq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                    (IRInt<"vqdmulh", [Vector]> $a, (splat $b))>;
  def vqrdmulhq_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                    (IRInt<"vqrdmulh", [Vector]> $a, (splat $b))>;
}
}

let params = T.Poly, overrideKindLetter = "p" in {
def vmullbq_poly: Intrinsic<DblVector, (args Vector:$a, Vector:$b),
                                       (IRInt<"vmull_poly", [DblVector, Vector]>
                                        $a, $b, 0)>;
def vmulltq_poly: Intrinsic<DblVector, (args Vector:$a, Vector:$b),
                                       (IRInt<"vmull_poly", [DblVector, Vector]>
                                        $a, $b, 1)>;
}

let params = T.Float in {
def vaddqf: Intrinsic<Vector, (args Vector:$a, Vector:$b), (fadd $a, $b)>,
            NameOverride<"vaddq">;
defm vandqf: bit_op_fp<and>, NameOverride<"vandq">;
defm vbicqf: bit_op_fp_with_inv<and>, NameOverride<"vbicq">;
defm veorqf: bit_op_fp<xor>, NameOverride<"veorq">;
````
- **L121 EN**: Executes a call or declaration centered on `statement`.
  **L121 CN**: 执行以 `statement` 为核心的调用或声明。
- **L122 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L122 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L123 EN**: Declares TableGen def record `vqdmulhq_n`.
  **L123 CN**: 声明 TableGen def 记录 `vqdmulhq_n`。
- **L124 EN**: Executes a call or declaration centered on `statement`.
  **L124 CN**: 执行以 `statement` 为核心的调用或声明。
- **L125 EN**: Declares TableGen def record `vqrdmulhq_n`.
  **L125 CN**: 声明 TableGen def 记录 `vqrdmulhq_n`。
- **L126 EN**: Executes a call or declaration centered on `statement`.
  **L126 CN**: 执行以 `statement` 为核心的调用或声明。
- **L127 EN**: Closes the current lexical scope or compound statement.
  **L127 CN**: 结束当前词法作用域或复合语句块。
- **L128 EN**: Closes the current lexical scope or compound statement.
  **L128 CN**: 结束当前词法作用域或复合语句块。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L130 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Poly, overrideKindLetter = "p" in {`.
  **L130 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Poly, overrideKindLetter = "p" in {`。
- **L131 EN**: Declares TableGen def record `vmullbq_poly`.
  **L131 CN**: 声明 TableGen def 记录 `vmullbq_poly`。
- **L132 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmull_poly", [DblVector, Vector]>`.
  **L132 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmull_poly", [DblVector, Vector]>`。
- **L133 EN**: Adds a standalone statement or declaration: `$a, $b, 0)>;`.
  **L133 CN**: 添加一条独立语句或声明：`$a, $b, 0)>;`。
- **L134 EN**: Declares TableGen def record `vmulltq_poly`.
  **L134 CN**: 声明 TableGen def 记录 `vmulltq_poly`。
- **L135 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmull_poly", [DblVector, Vector]>`.
  **L135 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmull_poly", [DblVector, Vector]>`。
- **L136 EN**: Adds a standalone statement or declaration: `$a, $b, 1)>;`.
  **L136 CN**: 添加一条独立语句或声明：`$a, $b, 1)>;`。
- **L137 EN**: Closes the current lexical scope or compound statement.
  **L137 CN**: 结束当前词法作用域或复合语句块。
- **L138 EN**: Blank line separating nearby declarations or logic blocks.
  **L138 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L139 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L139 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L140 EN**: Declares TableGen def record `vaddqf`.
  **L140 CN**: 声明 TableGen def 记录 `vaddqf`。
- **L141 EN**: Adds a standalone statement or declaration: `NameOverride<"vaddq">;`.
  **L141 CN**: 添加一条独立语句或声明：`NameOverride<"vaddq">;`。
- **L142 EN**: Declares TableGen defm record `vandqf`.
  **L142 CN**: 声明 TableGen defm 记录 `vandqf`。
- **L143 EN**: Declares TableGen defm record `vbicqf`.
  **L143 CN**: 声明 TableGen defm 记录 `vbicqf`。
- **L144 EN**: Declares TableGen defm record `veorqf`.
  **L144 CN**: 声明 TableGen defm 记录 `veorqf`。

### Lines 145-168

````tablegen
defm vornqf: bit_op_fp_with_inv<or>, NameOverride<"vornq">;
defm vorrqf: bit_op_fp<or>, NameOverride<"vorrq">;
def vsubqf: Intrinsic<Vector, (args Vector:$a, Vector:$b), (fsub $a, $b)>,
            NameOverride<"vsubq">;
def vmulqf: Intrinsic<Vector, (args Vector:$a, Vector:$b), (fmul $a, $b)>,
            NameOverride<"vmulq">;

let pnt = PNT_NType in {
  def vaddqf_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                  (fadd $a, (splat $b))>,
                NameOverride<"vaddq_n">;
  def vsubqf_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                  (fsub $a, (splat $b))>,
                NameOverride<"vsubq_n">;
  def vmulqf_n: Intrinsic<Vector, (args Vector:$a, unpromoted<Scalar>:$b),
                                  (fmul $a, (splat $b))>,
                NameOverride<"vmulq_n">;
}
}

multiclass FMA<bit add> {
  // FMS instructions are defined in the ArmARM as if they negate the
  // second multiply input.
  defvar m2_cg = !if(add, (id $m2), (fneg $m2));
````
- **L145 EN**: Declares TableGen defm record `vornqf`.
  **L145 CN**: 声明 TableGen defm 记录 `vornqf`。
- **L146 EN**: Declares TableGen defm record `vorrqf`.
  **L146 CN**: 声明 TableGen defm 记录 `vorrqf`。
- **L147 EN**: Declares TableGen def record `vsubqf`.
  **L147 CN**: 声明 TableGen def 记录 `vsubqf`。
- **L148 EN**: Adds a standalone statement or declaration: `NameOverride<"vsubq">;`.
  **L148 CN**: 添加一条独立语句或声明：`NameOverride<"vsubq">;`。
- **L149 EN**: Declares TableGen def record `vmulqf`.
  **L149 CN**: 声明 TableGen def 记录 `vmulqf`。
- **L150 EN**: Adds a standalone statement or declaration: `NameOverride<"vmulq">;`.
  **L150 CN**: 添加一条独立语句或声明：`NameOverride<"vmulq">;`。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L152 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L152 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L153 EN**: Declares TableGen def record `vaddqf_n`.
  **L153 CN**: 声明 TableGen def 记录 `vaddqf_n`。
- **L154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(fadd $a, (splat $b))>,`.
  **L154 CN**: 继续一个多行参数列表、初始化器或聚合项：`(fadd $a, (splat $b))>,`。
- **L155 EN**: Adds a standalone statement or declaration: `NameOverride<"vaddq_n">;`.
  **L155 CN**: 添加一条独立语句或声明：`NameOverride<"vaddq_n">;`。
- **L156 EN**: Declares TableGen def record `vsubqf_n`.
  **L156 CN**: 声明 TableGen def 记录 `vsubqf_n`。
- **L157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(fsub $a, (splat $b))>,`.
  **L157 CN**: 继续一个多行参数列表、初始化器或聚合项：`(fsub $a, (splat $b))>,`。
- **L158 EN**: Adds a standalone statement or declaration: `NameOverride<"vsubq_n">;`.
  **L158 CN**: 添加一条独立语句或声明：`NameOverride<"vsubq_n">;`。
- **L159 EN**: Declares TableGen def record `vmulqf_n`.
  **L159 CN**: 声明 TableGen def 记录 `vmulqf_n`。
- **L160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(fmul $a, (splat $b))>,`.
  **L160 CN**: 继续一个多行参数列表、初始化器或聚合项：`(fmul $a, (splat $b))>,`。
- **L161 EN**: Adds a standalone statement or declaration: `NameOverride<"vmulq_n">;`.
  **L161 CN**: 添加一条独立语句或声明：`NameOverride<"vmulq_n">;`。
- **L162 EN**: Closes the current lexical scope or compound statement.
  **L162 CN**: 结束当前词法作用域或复合语句块。
- **L163 EN**: Closes the current lexical scope or compound statement.
  **L163 CN**: 结束当前词法作用域或复合语句块。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L165 EN**: Declares TableGen multiclass record `FMA`.
  **L165 CN**: 声明 TableGen multiclass 记录 `FMA`。
- **L166 EN**: Comment explains nearby logic, constraints, or intent: `FMS instructions are defined in the ArmARM as if they negate the`.
  **L166 CN**: 注释解释附近代码的逻辑、约束或设计意图：`FMS instructions are defined in the ArmARM as if they negate the`。
- **L167 EN**: Comment explains nearby logic, constraints, or intent: `second multiply input.`.
  **L167 CN**: 注释解释附近代码的逻辑、约束或设计意图：`second multiply input.`。
- **L168 EN**: Declares TableGen defvar record `m2_cg = !if`.
  **L168 CN**: 声明 TableGen defvar 记录 `m2_cg = !if`。

### Lines 169-192

````tablegen

  defvar fma = strictFPAlt<IRIntBase<"fma", [Vector]>,
                           IRInt<"fma", [Vector]>>;
  defvar unpred_cg = (fma $m1, m2_cg, $addend);
  defvar pred_cg   = (IRInt<"fma_predicated", [Vector, Predicate]>
                          $m1, m2_cg, $addend, $pred);

  def q: Intrinsic<Vector, (args Vector:$addend, Vector:$m1, Vector:$m2),
                   unpred_cg>;

  def q_m: Intrinsic<Vector, (args Vector:$addend, Vector:$m1, Vector:$m2,
                                   Predicate:$pred), pred_cg>;

  // Only FMA has the vector/scalar variants, not FMS
  if add then let pnt = PNT_NType in {

    def q_n: Intrinsic<Vector, (args Vector:$addend, Vector:$m1,
                                     unpromoted<Scalar>:$m2_s),
                     (seq (splat $m2_s):$m2, unpred_cg)>;
    def sq_n: Intrinsic<Vector, (args Vector:$m1, Vector:$m2,
                                      unpromoted<Scalar>:$addend_s),
                        (seq (splat $addend_s):$addend, unpred_cg)>;
    def q_m_n: Intrinsic<Vector, (args Vector:$addend, Vector:$m1,
                                       unpromoted<Scalar>:$m2_s,
````
- **L169 EN**: Blank line separating nearby declarations or logic blocks.
  **L169 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L170 EN**: Declares TableGen defvar record `fma = strictFPAlt`.
  **L170 CN**: 声明 TableGen defvar 记录 `fma = strictFPAlt`。
- **L171 EN**: Adds a standalone statement or declaration: `IRInt<"fma", [Vector]>>;`.
  **L171 CN**: 添加一条独立语句或声明：`IRInt<"fma", [Vector]>>;`。
- **L172 EN**: Declares TableGen defvar record `unpred_cg =`.
  **L172 CN**: 声明 TableGen defvar 记录 `unpred_cg =`。
- **L173 EN**: Declares TableGen defvar record `pred_cg   =`.
  **L173 CN**: 声明 TableGen defvar 记录 `pred_cg   =`。
- **L174 EN**: Adds a standalone statement or declaration: `$m1, m2_cg, $addend, $pred);`.
  **L174 CN**: 添加一条独立语句或声明：`$m1, m2_cg, $addend, $pred);`。
- **L175 EN**: Blank line separating nearby declarations or logic blocks.
  **L175 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L176 EN**: Declares TableGen def record `q`.
  **L176 CN**: 声明 TableGen def 记录 `q`。
- **L177 EN**: Adds a standalone statement or declaration: `unpred_cg>;`.
  **L177 CN**: 添加一条独立语句或声明：`unpred_cg>;`。
- **L178 EN**: Blank line separating nearby declarations or logic blocks.
  **L178 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L179 EN**: Declares TableGen def record `q_m`.
  **L179 CN**: 声明 TableGen def 记录 `q_m`。
- **L180 EN**: Adds a standalone statement or declaration: `Predicate:$pred), pred_cg>;`.
  **L180 CN**: 添加一条独立语句或声明：`Predicate:$pred), pred_cg>;`。
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L182 EN**: Comment explains nearby logic, constraints, or intent: `Only FMA has the vector/scalar variants, not FMS`.
  **L182 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Only FMA has the vector/scalar variants, not FMS`。
- **L183 EN**: Continues the surrounding expression or declaration: `if add then let pnt = PNT_NType in {`.
  **L183 CN**: 继续构造周围的表达式或声明：`if add then let pnt = PNT_NType in {`。
- **L184 EN**: Blank line separating nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L185 EN**: Declares TableGen def record `q_n`.
  **L185 CN**: 声明 TableGen def 记录 `q_n`。
- **L186 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unpromoted<Scalar>:$m2_s),`.
  **L186 CN**: 继续一个多行参数列表、初始化器或聚合项：`unpromoted<Scalar>:$m2_s),`。
- **L187 EN**: Executes a call or declaration centered on `statement`.
  **L187 CN**: 执行以 `statement` 为核心的调用或声明。
- **L188 EN**: Declares TableGen def record `sq_n`.
  **L188 CN**: 声明 TableGen def 记录 `sq_n`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unpromoted<Scalar>:$addend_s),`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`unpromoted<Scalar>:$addend_s),`。
- **L190 EN**: Executes a call or declaration centered on `statement`.
  **L190 CN**: 执行以 `statement` 为核心的调用或声明。
- **L191 EN**: Declares TableGen def record `q_m_n`.
  **L191 CN**: 声明 TableGen def 记录 `q_m_n`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unpromoted<Scalar>:$m2_s,`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`unpromoted<Scalar>:$m2_s,`。

### Lines 193-216

````tablegen
                                       Predicate:$pred),
                         (seq (splat $m2_s):$m2, pred_cg)>;
    def sq_m_n: Intrinsic<Vector, (args Vector:$m1, Vector:$m2,
                                        unpromoted<Scalar>:$addend_s,
                                        Predicate:$pred),
                          (select $pred, (seq (splat $addend_s):$addend, unpred_cg), $m1)>;
  }
}

let params = T.Float in {
  defm vfma: FMA<1>;
  defm vfms: FMA<0>;
}

let params = T.Int, pnt = PNT_NType in {
  def vmlaq_n: Intrinsic<
    Vector, (args Vector:$addend, Vector:$m1, unpromoted<Scalar>:$m2_s),
    (add (mul $m1, (splat $m2_s)), $addend)>;
  def vmlasq_n: Intrinsic<
    Vector, (args Vector:$m1, Vector:$m2, unpromoted<Scalar>:$addend_s),
    (add (mul $m1, $m2), (splat $addend_s))>;

  def vmlaq_m_n: Intrinsic<
    Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s, Predicate:$pred),
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L194 EN**: Executes a call or declaration centered on `statement`.
  **L194 CN**: 执行以 `statement` 为核心的调用或声明。
- **L195 EN**: Declares TableGen def record `sq_m_n`.
  **L195 CN**: 声明 TableGen def 记录 `sq_m_n`。
- **L196 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unpromoted<Scalar>:$addend_s,`.
  **L196 CN**: 继续一个多行参数列表、初始化器或聚合项：`unpromoted<Scalar>:$addend_s,`。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L198 EN**: Executes a call or declaration centered on `statement`.
  **L198 CN**: 执行以 `statement` 为核心的调用或声明。
- **L199 EN**: Closes the current lexical scope or compound statement.
  **L199 CN**: 结束当前词法作用域或复合语句块。
- **L200 EN**: Closes the current lexical scope or compound statement.
  **L200 CN**: 结束当前词法作用域或复合语句块。
- **L201 EN**: Blank line separating nearby declarations or logic blocks.
  **L201 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L202 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L202 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L203 EN**: Declares TableGen defm record `vfma`.
  **L203 CN**: 声明 TableGen defm 记录 `vfma`。
- **L204 EN**: Declares TableGen defm record `vfms`.
  **L204 CN**: 声明 TableGen defm 记录 `vfms`。
- **L205 EN**: Closes the current lexical scope or compound statement.
  **L205 CN**: 结束当前词法作用域或复合语句块。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int, pnt = PNT_NType in {`.
  **L207 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int, pnt = PNT_NType in {`。
- **L208 EN**: Declares TableGen def record `vmlaq_n`.
  **L208 CN**: 声明 TableGen def 记录 `vmlaq_n`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$addend, Vector:$m1, unpromoted<Scalar>:$m2_s),`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$addend, Vector:$m1, unpromoted<Scalar>:$m2_s),`。
- **L210 EN**: Executes a call or declaration centered on `statement`.
  **L210 CN**: 执行以 `statement` 为核心的调用或声明。
- **L211 EN**: Declares TableGen def record `vmlasq_n`.
  **L211 CN**: 声明 TableGen def 记录 `vmlasq_n`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$m1, Vector:$m2, unpromoted<Scalar>:$addend_s),`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$m1, Vector:$m2, unpromoted<Scalar>:$addend_s),`。
- **L213 EN**: Executes a call or declaration centered on `statement`.
  **L213 CN**: 执行以 `statement` 为核心的调用或声明。
- **L214 EN**: Blank line separating nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L215 EN**: Declares TableGen def record `vmlaq_m_n`.
  **L215 CN**: 声明 TableGen def 记录 `vmlaq_m_n`。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s, Predicate:$pred),`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s, Predicate:$pred),`。

### Lines 217-240

````tablegen
    (IRInt<"vmla_n_predicated", [Vector, Predicate]>
        $addend, $m1, $m2_s, $pred)>;
  def vmlasq_m_n: Intrinsic<
    Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s, Predicate:$pred),
    (IRInt<"vmlas_n_predicated", [Vector, Predicate]>
        $m1, $m2, $addend_s, $pred)>;
}

multiclass VQDMLA {
  def hq_n: Intrinsic<
    Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s),
    (IRInt<NAME # "h", [Vector]> $addend, $m1, $m2_s)>;
  def shq_n: Intrinsic<
    Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s),
    (IRInt<NAME # "sh", [Vector]> $m1, $m2, $addend_s)>;

  def hq_m_n: Intrinsic<
    Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s, Predicate:$pred),
    (IRInt<NAME # "h_predicated", [Vector, Predicate]>
         $addend, $m1, $m2_s, $pred)>;
  def shq_m_n: Intrinsic<
    Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s, Predicate:$pred),
    (IRInt<NAME # "sh_predicated", [Vector, Predicate]>
         $m1, $m2, $addend_s, $pred)>;
````
- **L217 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmla_n_predicated", [Vector, Predicate]>`.
  **L217 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmla_n_predicated", [Vector, Predicate]>`。
- **L218 EN**: Adds a standalone statement or declaration: `$addend, $m1, $m2_s, $pred)>;`.
  **L218 CN**: 添加一条独立语句或声明：`$addend, $m1, $m2_s, $pred)>;`。
- **L219 EN**: Declares TableGen def record `vmlasq_m_n`.
  **L219 CN**: 声明 TableGen def 记录 `vmlasq_m_n`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s, Predicate:$pred),`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s, Predicate:$pred),`。
- **L221 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmlas_n_predicated", [Vector, Predicate]>`.
  **L221 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmlas_n_predicated", [Vector, Predicate]>`。
- **L222 EN**: Adds a standalone statement or declaration: `$m1, $m2, $addend_s, $pred)>;`.
  **L222 CN**: 添加一条独立语句或声明：`$m1, $m2, $addend_s, $pred)>;`。
- **L223 EN**: Closes the current lexical scope or compound statement.
  **L223 CN**: 结束当前词法作用域或复合语句块。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L225 EN**: Declares TableGen multiclass record `VQDMLA`.
  **L225 CN**: 声明 TableGen multiclass 记录 `VQDMLA`。
- **L226 EN**: Declares TableGen def record `hq_n`.
  **L226 CN**: 声明 TableGen def 记录 `hq_n`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s),`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s),`。
- **L228 EN**: Executes a call or declaration centered on `statement`.
  **L228 CN**: 执行以 `statement` 为核心的调用或声明。
- **L229 EN**: Declares TableGen def record `shq_n`.
  **L229 CN**: 声明 TableGen def 记录 `shq_n`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s),`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s),`。
- **L231 EN**: Executes a call or declaration centered on `statement`.
  **L231 CN**: 执行以 `statement` 为核心的调用或声明。
- **L232 EN**: Blank line separating nearby declarations or logic blocks.
  **L232 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L233 EN**: Declares TableGen def record `hq_m_n`.
  **L233 CN**: 声明 TableGen def 记录 `hq_m_n`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s, Predicate:$pred),`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$addend, Vector:$m1, Scalar:$m2_s, Predicate:$pred),`。
- **L235 EN**: Continues the surrounding expression or declaration: `(IRInt<NAME # "h_predicated", [Vector, Predicate]>`.
  **L235 CN**: 继续构造周围的表达式或声明：`(IRInt<NAME # "h_predicated", [Vector, Predicate]>`。
- **L236 EN**: Adds a standalone statement or declaration: `$addend, $m1, $m2_s, $pred)>;`.
  **L236 CN**: 添加一条独立语句或声明：`$addend, $m1, $m2_s, $pred)>;`。
- **L237 EN**: Declares TableGen def record `shq_m_n`.
  **L237 CN**: 声明 TableGen def 记录 `shq_m_n`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s, Predicate:$pred),`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$m1, Vector:$m2, Scalar:$addend_s, Predicate:$pred),`。
- **L239 EN**: Continues the surrounding expression or declaration: `(IRInt<NAME # "sh_predicated", [Vector, Predicate]>`.
  **L239 CN**: 继续构造周围的表达式或声明：`(IRInt<NAME # "sh_predicated", [Vector, Predicate]>`。
- **L240 EN**: Adds a standalone statement or declaration: `$m1, $m2, $addend_s, $pred)>;`.
  **L240 CN**: 添加一条独立语句或声明：`$m1, $m2, $addend_s, $pred)>;`。

### Lines 241-264

````tablegen
}

let params = T.Signed, pnt = PNT_NType in {
  defm vqdmla: VQDMLA;
  defm vqrdmla: VQDMLA;
}

multiclass VQDMLAD<int exchange, int round, int subtract> {
  def "": Intrinsic<Vector, (args Vector:$a, Vector:$b, Vector:$c),
       (IRInt<"vqdmlad", [Vector]> $a, $b, $c,
            (u32 exchange), (u32 round), (u32 subtract))>;
  def _m: Intrinsic<Vector, (args Vector:$a, Vector:$b, Vector:$c,
                                  Predicate:$pred),
       (IRInt<"vqdmlad_predicated", [Vector, Predicate]> $a, $b, $c,
            (u32 exchange), (u32 round), (u32 subtract), $pred)>;
}
let params = T.Signed in {
  defm vqdmladhq:   VQDMLAD<0, 0, 0>;
  defm vqdmladhxq:  VQDMLAD<1, 0, 0>;
  defm vqdmlsdhq:   VQDMLAD<0, 0, 1>;
  defm vqdmlsdhxq:  VQDMLAD<1, 0, 1>;
  defm vqrdmladhq:  VQDMLAD<0, 1, 0>;
  defm vqrdmladhxq: VQDMLAD<1, 1, 0>;
  defm vqrdmlsdhq:  VQDMLAD<0, 1, 1>;
````
- **L241 EN**: Closes the current lexical scope or compound statement.
  **L241 CN**: 结束当前词法作用域或复合语句块。
- **L242 EN**: Blank line separating nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L243 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed, pnt = PNT_NType in {`.
  **L243 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed, pnt = PNT_NType in {`。
- **L244 EN**: Declares TableGen defm record `vqdmla`.
  **L244 CN**: 声明 TableGen defm 记录 `vqdmla`。
- **L245 EN**: Declares TableGen defm record `vqrdmla`.
  **L245 CN**: 声明 TableGen defm 记录 `vqrdmla`。
- **L246 EN**: Closes the current lexical scope or compound statement.
  **L246 CN**: 结束当前词法作用域或复合语句块。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L248 EN**: Declares TableGen multiclass record `VQDMLAD`.
  **L248 CN**: 声明 TableGen multiclass 记录 `VQDMLAD`。
- **L249 EN**: Declares TableGen def record `""`.
  **L249 CN**: 声明 TableGen def 记录 `""`。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vqdmlad", [Vector]> $a, $b, $c,`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vqdmlad", [Vector]> $a, $b, $c,`。
- **L251 EN**: Executes a call or declaration centered on `statement`.
  **L251 CN**: 执行以 `statement` 为核心的调用或声明。
- **L252 EN**: Declares TableGen def record `_m`.
  **L252 CN**: 声明 TableGen def 记录 `_m`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vqdmlad_predicated", [Vector, Predicate]> $a, $b, $c,`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vqdmlad_predicated", [Vector, Predicate]> $a, $b, $c,`。
- **L255 EN**: Executes a call or declaration centered on `statement`.
  **L255 CN**: 执行以 `statement` 为核心的调用或声明。
- **L256 EN**: Closes the current lexical scope or compound statement.
  **L256 CN**: 结束当前词法作用域或复合语句块。
- **L257 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L257 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L258 EN**: Declares TableGen defm record `vqdmladhq`.
  **L258 CN**: 声明 TableGen defm 记录 `vqdmladhq`。
- **L259 EN**: Declares TableGen defm record `vqdmladhxq`.
  **L259 CN**: 声明 TableGen defm 记录 `vqdmladhxq`。
- **L260 EN**: Declares TableGen defm record `vqdmlsdhq`.
  **L260 CN**: 声明 TableGen defm 记录 `vqdmlsdhq`。
- **L261 EN**: Declares TableGen defm record `vqdmlsdhxq`.
  **L261 CN**: 声明 TableGen defm 记录 `vqdmlsdhxq`。
- **L262 EN**: Declares TableGen defm record `vqrdmladhq`.
  **L262 CN**: 声明 TableGen defm 记录 `vqrdmladhq`。
- **L263 EN**: Declares TableGen defm record `vqrdmladhxq`.
  **L263 CN**: 声明 TableGen defm 记录 `vqrdmladhxq`。
- **L264 EN**: Declares TableGen defm record `vqrdmlsdhq`.
  **L264 CN**: 声明 TableGen defm 记录 `vqrdmlsdhq`。

### Lines 265-288

````tablegen
  defm vqrdmlsdhxq: VQDMLAD<1, 1, 1>;
}

let params = !listconcat(T.Int16, T.Int32) in {
  let pnt = PNT_None in {
    def vmvnq_n: Intrinsic<Vector, (args imm_simd_vmvn:$imm),
                           (not (splat (Scalar $imm)))>;
  }
  defm vmvnq: IntrinsicMX<Vector, (args imm_simd_vmvn:$imm, Predicate:$pred),
                     (select $pred, (not (splat (Scalar $imm))), $inactive),
                     1, "_n", PNT_NType, PNT_None>;
  let pnt = PNT_NType in {
    def vbicq_n: Intrinsic<Vector, (args Vector:$v, imm_simd_restrictive:$imm),
                           (and $v, (not (splat (Scalar $imm))))>;
    def vorrq_n: Intrinsic<Vector, (args Vector:$v, imm_simd_restrictive:$imm),
                           (or $v, (splat (Scalar $imm)))>;
  }
  def vbicq_m_n: Intrinsic<
    Vector, (args Vector:$v, imm_simd_restrictive:$imm, Predicate:$pred),
    (select $pred, (and $v, (not (splat (Scalar $imm)))), $v)>;
  def vorrq_m_n: Intrinsic<
    Vector, (args Vector:$v, imm_simd_restrictive:$imm, Predicate:$pred),
    (select $pred, (or $v, (splat (Scalar $imm))), $v)>;
}
````
- **L265 EN**: Declares TableGen defm record `vqrdmlsdhxq`.
  **L265 CN**: 声明 TableGen defm 记录 `vqrdmlsdhxq`。
- **L266 EN**: Closes the current lexical scope or compound statement.
  **L266 CN**: 结束当前词法作用域或复合语句块。
- **L267 EN**: Blank line separating nearby declarations or logic blocks.
  **L267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L268 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = !listconcat(T.Int16, T.Int32) in {`.
  **L268 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = !listconcat(T.Int16, T.Int32) in {`。
- **L269 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_None in {`.
  **L269 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_None in {`。
- **L270 EN**: Declares TableGen def record `vmvnq_n`.
  **L270 CN**: 声明 TableGen def 记录 `vmvnq_n`。
- **L271 EN**: Executes a call or declaration centered on `statement`.
  **L271 CN**: 执行以 `statement` 为核心的调用或声明。
- **L272 EN**: Closes the current lexical scope or compound statement.
  **L272 CN**: 结束当前词法作用域或复合语句块。
- **L273 EN**: Declares TableGen defm record `vmvnq`.
  **L273 CN**: 声明 TableGen defm 记录 `vmvnq`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(select $pred, (not (splat (Scalar $imm))), $inactive),`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`(select $pred, (not (splat (Scalar $imm))), $inactive),`。
- **L275 EN**: Adds a standalone statement or declaration: `1, "_n", PNT_NType, PNT_None>;`.
  **L275 CN**: 添加一条独立语句或声明：`1, "_n", PNT_NType, PNT_None>;`。
- **L276 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L276 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L277 EN**: Declares TableGen def record `vbicq_n`.
  **L277 CN**: 声明 TableGen def 记录 `vbicq_n`。
- **L278 EN**: Executes a call or declaration centered on `statement`.
  **L278 CN**: 执行以 `statement` 为核心的调用或声明。
- **L279 EN**: Declares TableGen def record `vorrq_n`.
  **L279 CN**: 声明 TableGen def 记录 `vorrq_n`。
- **L280 EN**: Executes a call or declaration centered on `statement`.
  **L280 CN**: 执行以 `statement` 为核心的调用或声明。
- **L281 EN**: Closes the current lexical scope or compound statement.
  **L281 CN**: 结束当前词法作用域或复合语句块。
- **L282 EN**: Declares TableGen def record `vbicq_m_n`.
  **L282 CN**: 声明 TableGen def 记录 `vbicq_m_n`。
- **L283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$v, imm_simd_restrictive:$imm, Predicate:$pred),`.
  **L283 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$v, imm_simd_restrictive:$imm, Predicate:$pred),`。
- **L284 EN**: Executes a call or declaration centered on `statement`.
  **L284 CN**: 执行以 `statement` 为核心的调用或声明。
- **L285 EN**: Declares TableGen def record `vorrq_m_n`.
  **L285 CN**: 声明 TableGen def 记录 `vorrq_m_n`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$v, imm_simd_restrictive:$imm, Predicate:$pred),`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$v, imm_simd_restrictive:$imm, Predicate:$pred),`。
- **L287 EN**: Executes a call or declaration centered on `statement`.
  **L287 CN**: 执行以 `statement` 为核心的调用或声明。
- **L288 EN**: Closes the current lexical scope or compound statement.
  **L288 CN**: 结束当前词法作用域或复合语句块。

### Lines 289-312

````tablegen

let params = T.Usual in {
  let pnt = PNT_None in
    def vdupq_n: Intrinsic<Vector, (args unpromoted<Scalar>:$s), (splat $s)>;

  defm vdupq: IntrinsicMX<
      Vector, (args unpromoted<Scalar>:$s, Predicate:$pred),
      (select $pred, (splat $s), $inactive), 1, "_n", PNT_NType, PNT_None>;
}

multiclass vxdup_mc<dag paramsIn, dag paramsOut> {
  defvar UnpredInt     = IRInt<NAME, [Vector]>;
  defvar PredInt       = IRInt<NAME # "_predicated", [Vector, Predicate]>;
  defvar UnpredIntCall = !con((UnpredInt $base), paramsOut);
  defvar PredIntCall   = !con((PredInt $inactive, $base), paramsOut, (? $pred));

  // Straightforward case with neither writeback nor predication
  let pnt = PNT_N in
    def q_n: Intrinsic<Vector, !con((args u32:$base), paramsIn),
                       (xval UnpredIntCall, 0)>;

  // Predicated form without writeback
  defm q: IntrinsicMX<
      Vector, !con((args u32:$base), paramsIn, (? Predicate:$pred)),
````
- **L289 EN**: Blank line separating nearby declarations or logic blocks.
  **L289 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L290 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Usual in {`.
  **L290 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Usual in {`。
- **L291 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_None in`.
  **L291 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_None in`。
- **L292 EN**: Declares TableGen def record `vdupq_n`.
  **L292 CN**: 声明 TableGen def 记录 `vdupq_n`。
- **L293 EN**: Blank line separating nearby declarations or logic blocks.
  **L293 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L294 EN**: Declares TableGen defm record `vdupq`.
  **L294 CN**: 声明 TableGen defm 记录 `vdupq`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args unpromoted<Scalar>:$s, Predicate:$pred),`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args unpromoted<Scalar>:$s, Predicate:$pred),`。
- **L296 EN**: Executes a call or declaration centered on `statement`.
  **L296 CN**: 执行以 `statement` 为核心的调用或声明。
- **L297 EN**: Closes the current lexical scope or compound statement.
  **L297 CN**: 结束当前词法作用域或复合语句块。
- **L298 EN**: Blank line separating nearby declarations or logic blocks.
  **L298 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L299 EN**: Declares TableGen multiclass record `vxdup_mc`.
  **L299 CN**: 声明 TableGen multiclass 记录 `vxdup_mc`。
- **L300 EN**: Declares TableGen defvar record `UnpredInt     = IRInt`.
  **L300 CN**: 声明 TableGen defvar 记录 `UnpredInt     = IRInt`。
- **L301 EN**: Declares TableGen defvar record `PredInt       = IRInt`.
  **L301 CN**: 声明 TableGen defvar 记录 `PredInt       = IRInt`。
- **L302 EN**: Declares TableGen defvar record `UnpredIntCall = !con`.
  **L302 CN**: 声明 TableGen defvar 记录 `UnpredIntCall = !con`。
- **L303 EN**: Declares TableGen defvar record `PredIntCall   = !con`.
  **L303 CN**: 声明 TableGen defvar 记录 `PredIntCall   = !con`。
- **L304 EN**: Blank line separating nearby declarations or logic blocks.
  **L304 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L305 EN**: Comment explains nearby logic, constraints, or intent: `Straightforward case with neither writeback nor predication`.
  **L305 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Straightforward case with neither writeback nor predication`。
- **L306 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_N in`.
  **L306 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_N in`。
- **L307 EN**: Declares TableGen def record `q_n`.
  **L307 CN**: 声明 TableGen def 记录 `q_n`。
- **L308 EN**: Executes a call or declaration centered on `statement`.
  **L308 CN**: 执行以 `statement` 为核心的调用或声明。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L310 EN**: Comment explains nearby logic, constraints, or intent: `Predicated form without writeback`.
  **L310 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicated form without writeback`。
- **L311 EN**: Declares TableGen defm record `q`.
  **L311 CN**: 声明 TableGen defm 记录 `q`。
- **L312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, !con((args u32:$base), paramsIn, (? Predicate:$pred)),`.
  **L312 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, !con((args u32:$base), paramsIn, (? Predicate:$pred)),`。

### Lines 313-336

````tablegen
      (xval PredIntCall, 0), 1, "_n", PNT_NType, PNT_N>;

  // Writeback without predication
  let pnt = PNT_WB in
    def q_wb: Intrinsic<
      Vector, !con((args Ptr<u32>:$baseaddr), paramsIn),
      (seq (load $baseaddr):$base,
           UnpredIntCall:$pair,
           (store (xval $pair, 1), $baseaddr),
           (xval $pair, 0))>;

  // Both writeback and predicated
  defm q: IntrinsicMX<
      Vector, !con((args Ptr<u32>:$baseaddr), paramsIn, (? Predicate:$pred)),
      (seq (load $baseaddr):$base,
           PredIntCall:$pair,
           (store (xval $pair, 1), $baseaddr),
           (xval $pair, 0)), 1, "_wb", PNT_WBType, PNT_WB>;
}

let params = T.Unsigned in {
  defm vidup:  vxdup_mc<(?             imm_1248:$step), (?         $step)>;
  defm vddup:  vxdup_mc<(?             imm_1248:$step), (?         $step)>;
  defm viwdup: vxdup_mc<(? u32:$limit, imm_1248:$step), (? $limit, $step)>;
````
- **L313 EN**: Executes a call or declaration centered on `statement`.
  **L313 CN**: 执行以 `statement` 为核心的调用或声明。
- **L314 EN**: Blank line separating nearby declarations or logic blocks.
  **L314 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L315 EN**: Comment explains nearby logic, constraints, or intent: `Writeback without predication`.
  **L315 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Writeback without predication`。
- **L316 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_WB in`.
  **L316 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_WB in`。
- **L317 EN**: Declares TableGen def record `q_wb`.
  **L317 CN**: 声明 TableGen def 记录 `q_wb`。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, !con((args Ptr<u32>:$baseaddr), paramsIn),`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, !con((args Ptr<u32>:$baseaddr), paramsIn),`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (load $baseaddr):$base,`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (load $baseaddr):$base,`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `UnpredIntCall:$pair,`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`UnpredIntCall:$pair,`。
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (xval $pair, 1), $baseaddr),`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (xval $pair, 1), $baseaddr),`。
- **L322 EN**: Executes a call or declaration centered on `statement`.
  **L322 CN**: 执行以 `statement` 为核心的调用或声明。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L324 EN**: Comment explains nearby logic, constraints, or intent: `Both writeback and predicated`.
  **L324 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Both writeback and predicated`。
- **L325 EN**: Declares TableGen defm record `q`.
  **L325 CN**: 声明 TableGen defm 记录 `q`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, !con((args Ptr<u32>:$baseaddr), paramsIn, (? Predicate:$pred)),`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, !con((args Ptr<u32>:$baseaddr), paramsIn, (? Predicate:$pred)),`。
- **L327 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (load $baseaddr):$base,`.
  **L327 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (load $baseaddr):$base,`。
- **L328 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PredIntCall:$pair,`.
  **L328 CN**: 继续一个多行参数列表、初始化器或聚合项：`PredIntCall:$pair,`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (xval $pair, 1), $baseaddr),`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (xval $pair, 1), $baseaddr),`。
- **L330 EN**: Executes a call or declaration centered on `statement`.
  **L330 CN**: 执行以 `statement` 为核心的调用或声明。
- **L331 EN**: Closes the current lexical scope or compound statement.
  **L331 CN**: 结束当前词法作用域或复合语句块。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Unsigned in {`.
  **L333 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Unsigned in {`。
- **L334 EN**: Declares TableGen defm record `vidup`.
  **L334 CN**: 声明 TableGen defm 记录 `vidup`。
- **L335 EN**: Declares TableGen defm record `vddup`.
  **L335 CN**: 声明 TableGen defm 记录 `vddup`。
- **L336 EN**: Declares TableGen defm record `viwdup`.
  **L336 CN**: 声明 TableGen defm 记录 `viwdup`。

### Lines 337-360

````tablegen
  defm vdwdup: vxdup_mc<(? u32:$limit, imm_1248:$step), (? $limit, $step)>;
}

let params = T.Int in {
  def vmvnq: Intrinsic<Vector, (args Vector:$a),
                       (xor $a, (uint_max Vector))>;
  defm vmvnq: IntrinsicMX<Vector, (args Vector:$a, Predicate:$pred),
    (IRInt<"mvn_predicated", [Vector, Predicate]> $a, $pred, $inactive)>;
  def vclzq: Intrinsic<Vector, (args Vector:$a),
                       (IRIntBase<"ctlz", [Vector]> $a, (i1 0))>;
  defm vclzq: IntrinsicMX<Vector, (args Vector:$a, Predicate:$pred),
    (IRInt<"clz_predicated", [Vector, Predicate]> $a, $pred, $inactive)>;
}
let params = T.Signed in {
  def vclsq: Intrinsic<Vector, (args Vector:$a), (IRInt<"vcls", [Vector]> $a)>;
  defm vclsq: IntrinsicMX<Vector, (args Vector:$a, Predicate:$pred),
    (IRInt<"cls_predicated", [Vector, Predicate]> $a, $pred, $inactive)>;

  def vnegq: Intrinsic<Vector, (args Vector:$a),
                       (sub (zeroinit Vector), $a)>;
  def vabsq: Intrinsic<Vector, (args Vector:$a),
                       (select (icmp_slt $a, (zeroinit Vector)),
                               (sub (zeroinit Vector), $a), $a)>;
  def vqnegq: Intrinsic<Vector, (args Vector:$a),
````
- **L337 EN**: Declares TableGen defm record `vdwdup`.
  **L337 CN**: 声明 TableGen defm 记录 `vdwdup`。
- **L338 EN**: Closes the current lexical scope or compound statement.
  **L338 CN**: 结束当前词法作用域或复合语句块。
- **L339 EN**: Blank line separating nearby declarations or logic blocks.
  **L339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L340 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L340 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L341 EN**: Declares TableGen def record `vmvnq`.
  **L341 CN**: 声明 TableGen def 记录 `vmvnq`。
- **L342 EN**: Executes a call or declaration centered on `statement`.
  **L342 CN**: 执行以 `statement` 为核心的调用或声明。
- **L343 EN**: Declares TableGen defm record `vmvnq`.
  **L343 CN**: 声明 TableGen defm 记录 `vmvnq`。
- **L344 EN**: Executes a call or declaration centered on `statement`.
  **L344 CN**: 执行以 `statement` 为核心的调用或声明。
- **L345 EN**: Declares TableGen def record `vclzq`.
  **L345 CN**: 声明 TableGen def 记录 `vclzq`。
- **L346 EN**: Executes a call or declaration centered on `statement`.
  **L346 CN**: 执行以 `statement` 为核心的调用或声明。
- **L347 EN**: Declares TableGen defm record `vclzq`.
  **L347 CN**: 声明 TableGen defm 记录 `vclzq`。
- **L348 EN**: Executes a call or declaration centered on `statement`.
  **L348 CN**: 执行以 `statement` 为核心的调用或声明。
- **L349 EN**: Closes the current lexical scope or compound statement.
  **L349 CN**: 结束当前词法作用域或复合语句块。
- **L350 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L350 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L351 EN**: Declares TableGen def record `vclsq`.
  **L351 CN**: 声明 TableGen def 记录 `vclsq`。
- **L352 EN**: Declares TableGen defm record `vclsq`.
  **L352 CN**: 声明 TableGen defm 记录 `vclsq`。
- **L353 EN**: Executes a call or declaration centered on `statement`.
  **L353 CN**: 执行以 `statement` 为核心的调用或声明。
- **L354 EN**: Blank line separating nearby declarations or logic blocks.
  **L354 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L355 EN**: Declares TableGen def record `vnegq`.
  **L355 CN**: 声明 TableGen def 记录 `vnegq`。
- **L356 EN**: Executes a call or declaration centered on `statement`.
  **L356 CN**: 执行以 `statement` 为核心的调用或声明。
- **L357 EN**: Declares TableGen def record `vabsq`.
  **L357 CN**: 声明 TableGen def 记录 `vabsq`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(select (icmp_slt $a, (zeroinit Vector)),`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`(select (icmp_slt $a, (zeroinit Vector)),`。
- **L359 EN**: Executes a call or declaration centered on `statement`.
  **L359 CN**: 执行以 `statement` 为核心的调用或声明。
- **L360 EN**: Declares TableGen def record `vqnegq`.
  **L360 CN**: 声明 TableGen def 记录 `vqnegq`。

### Lines 361-384

````tablegen
                        (select (icmp_eq $a, (int_min Vector)),
                                (int_max Vector),
                                (sub (zeroinit Vector), $a))>;
  def vqabsq: Intrinsic<Vector, (args Vector:$a),
                        (select (icmp_sgt $a, (zeroinit Vector)), $a,
                                (select (icmp_eq $a, (int_min Vector)),
                                        (int_max Vector),
                                        (sub (zeroinit Vector), $a)))>;

  foreach name = ["qneg", "qabs"] in {
    defm v#name#q: IntrinsicMX<Vector, (args Vector:$a, Predicate:$pred),
      (IRInt<name#"_predicated", [Vector, Predicate]> $a, $pred, $inactive),
      0 /* no _x variant for saturating intrinsics */>;
  }
}
let params = !listconcat(T.Signed, T.Float) in {
  foreach name = ["neg", "abs"] in {
    defm v#name#q: IntrinsicMX<Vector, (args Vector:$a, Predicate:$pred),
      (IRInt<name#"_predicated", [Vector, Predicate]> $a, $pred, $inactive)>;
  }
}
let params = T.Float in {
  def vnegq_f: Intrinsic<Vector, (args Vector:$a), (fneg $a)>,
               NameOverride<"vnegq">;
````
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(select (icmp_eq $a, (int_min Vector)),`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`(select (icmp_eq $a, (int_min Vector)),`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(int_max Vector),`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`(int_max Vector),`。
- **L363 EN**: Executes a call or declaration centered on `statement`.
  **L363 CN**: 执行以 `statement` 为核心的调用或声明。
- **L364 EN**: Declares TableGen def record `vqabsq`.
  **L364 CN**: 声明 TableGen def 记录 `vqabsq`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(select (icmp_sgt $a, (zeroinit Vector)), $a,`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`(select (icmp_sgt $a, (zeroinit Vector)), $a,`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(select (icmp_eq $a, (int_min Vector)),`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`(select (icmp_eq $a, (int_min Vector)),`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(int_max Vector),`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`(int_max Vector),`。
- **L368 EN**: Executes a call or declaration centered on `statement`.
  **L368 CN**: 执行以 `statement` 为核心的调用或声明。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L370 EN**: Starts a TableGen iteration used to generate repeated records: `foreach name = ["qneg", "qabs"] in {`.
  **L370 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach name = ["qneg", "qabs"] in {`。
- **L371 EN**: Declares TableGen defm record `v#name#q`.
  **L371 CN**: 声明 TableGen defm 记录 `v#name#q`。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<name#"_predicated", [Vector, Predicate]> $a, $pred, $inactive),`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<name#"_predicated", [Vector, Predicate]> $a, $pred, $inactive),`。
- **L373 EN**: Adds a standalone statement or declaration: `0 /* no _x variant for saturating intrinsics */>;`.
  **L373 CN**: 添加一条独立语句或声明：`0 /* no _x variant for saturating intrinsics */>;`。
- **L374 EN**: Closes the current lexical scope or compound statement.
  **L374 CN**: 结束当前词法作用域或复合语句块。
- **L375 EN**: Closes the current lexical scope or compound statement.
  **L375 CN**: 结束当前词法作用域或复合语句块。
- **L376 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = !listconcat(T.Signed, T.Float) in {`.
  **L376 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = !listconcat(T.Signed, T.Float) in {`。
- **L377 EN**: Starts a TableGen iteration used to generate repeated records: `foreach name = ["neg", "abs"] in {`.
  **L377 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach name = ["neg", "abs"] in {`。
- **L378 EN**: Declares TableGen defm record `v#name#q`.
  **L378 CN**: 声明 TableGen defm 记录 `v#name#q`。
- **L379 EN**: Executes a call or declaration centered on `statement`.
  **L379 CN**: 执行以 `statement` 为核心的调用或声明。
- **L380 EN**: Closes the current lexical scope or compound statement.
  **L380 CN**: 结束当前词法作用域或复合语句块。
- **L381 EN**: Closes the current lexical scope or compound statement.
  **L381 CN**: 结束当前词法作用域或复合语句块。
- **L382 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L382 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L383 EN**: Declares TableGen def record `vnegq_f`.
  **L383 CN**: 声明 TableGen def 记录 `vnegq_f`。
- **L384 EN**: Adds a standalone statement or declaration: `NameOverride<"vnegq">;`.
  **L384 CN**: 添加一条独立语句或声明：`NameOverride<"vnegq">;`。

### Lines 385-408

````tablegen
  def vabsq_f: Intrinsic<Vector, (args Vector:$a),
               (IRIntBase<"fabs", [Vector]> $a)>, NameOverride<"vabsq">;
}

// The bitcasting below is not overcomplicating the IR because while
// Vector and UVector may be different vector types at the C level i.e.
// vectors of same size signed/unsigned ints. Once they're lowered
// to IR, they are just bit vectors with no sign at all, so the
// bitcasts will be automatically elided by IRBuilder.
multiclass predicated_bit_op_fp<string int_op> {
def "": Intrinsic<Vector, (args Vector:$inactive, Vector:$a, Vector:$b,
                                Predicate:$pred),
    (bitcast (IRInt<int_op, [UVector, Predicate]>
                    (bitcast $a, UVector),
                    (bitcast $b, UVector),
                    $pred,
                    (bitcast $inactive, UVector)), Vector)>;
}

// Plain intrinsics
let params = T.Usual in {
def vabdq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                     (IRInt<"vabd", [Vector]> $a, $b, (unsignedflag Scalar))>;
}
````
- **L385 EN**: Declares TableGen def record `vabsq_f`.
  **L385 CN**: 声明 TableGen def 记录 `vabsq_f`。
- **L386 EN**: Executes a call or declaration centered on `statement`.
  **L386 CN**: 执行以 `statement` 为核心的调用或声明。
- **L387 EN**: Closes the current lexical scope or compound statement.
  **L387 CN**: 结束当前词法作用域或复合语句块。
- **L388 EN**: Blank line separating nearby declarations or logic blocks.
  **L388 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L389 EN**: Comment explains nearby logic, constraints, or intent: `The bitcasting below is not overcomplicating the IR because while`.
  **L389 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The bitcasting below is not overcomplicating the IR because while`。
- **L390 EN**: Comment explains nearby logic, constraints, or intent: `Vector and UVector may be different vector types at the C level i.e.`.
  **L390 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Vector and UVector may be different vector types at the C level i.e.`。
- **L391 EN**: Comment explains nearby logic, constraints, or intent: `vectors of same size signed/unsigned ints. Once they're lowered`.
  **L391 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vectors of same size signed/unsigned ints. Once they're lowered`。
- **L392 EN**: Comment explains nearby logic, constraints, or intent: `to IR, they are just bit vectors with no sign at all, so the`.
  **L392 CN**: 注释解释附近代码的逻辑、约束或设计意图：`to IR, they are just bit vectors with no sign at all, so the`。
- **L393 EN**: Comment explains nearby logic, constraints, or intent: `bitcasts will be automatically elided by IRBuilder.`.
  **L393 CN**: 注释解释附近代码的逻辑、约束或设计意图：`bitcasts will be automatically elided by IRBuilder.`。
- **L394 EN**: Declares TableGen multiclass record `predicated_bit_op_fp`.
  **L394 CN**: 声明 TableGen multiclass 记录 `predicated_bit_op_fp`。
- **L395 EN**: Declares TableGen def record `""`.
  **L395 CN**: 声明 TableGen def 记录 `""`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L397 EN**: Continues logic associated with callable symbol `bitcast`.
  **L397 CN**: 继续与可调用符号 `bitcast` 相关的逻辑。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast $a, UVector),`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast $a, UVector),`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast $b, UVector),`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast $b, UVector),`。
- **L400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$pred,`.
  **L400 CN**: 继续一个多行参数列表、初始化器或聚合项：`$pred,`。
- **L401 EN**: Executes a call or declaration centered on `statement`.
  **L401 CN**: 执行以 `statement` 为核心的调用或声明。
- **L402 EN**: Closes the current lexical scope or compound statement.
  **L402 CN**: 结束当前词法作用域或复合语句块。
- **L403 EN**: Blank line separating nearby declarations or logic blocks.
  **L403 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L404 EN**: Comment explains nearby logic, constraints, or intent: `Plain intrinsics`.
  **L404 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Plain intrinsics`。
- **L405 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Usual in {`.
  **L405 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Usual in {`。
- **L406 EN**: Declares TableGen def record `vabdq`.
  **L406 CN**: 声明 TableGen def 记录 `vabdq`。
- **L407 EN**: Executes a call or declaration centered on `statement`.
  **L407 CN**: 执行以 `statement` 为核心的调用或声明。
- **L408 EN**: Closes the current lexical scope or compound statement.
  **L408 CN**: 结束当前词法作用域或复合语句块。

### Lines 409-432

````tablegen

multiclass VectorVectorArithmetic<string operation, dag extraArgs = (?),
                                  bit wantXVariant = 1> {
  defm "" : IntrinsicMX<
      Vector, (args Vector:$a, Vector:$b, Predicate:$pred),
      !con((IRInt<operation, [Vector, Predicate]> $a, $b),
           extraArgs, (? $pred, $inactive)), wantXVariant>;
}

multiclass VectorScalarArithmetic<string operation, string basename,
                                  dag extraArgs = (?),
                                  bit wantXVariant = 1> {
  defm "" : IntrinsicMXNameOverride<
      Vector, (args Vector:$a, unpromoted<Scalar>:$b, Predicate:$pred),
      !con((IRInt<operation, [Vector, Predicate]> $a, (splat $b)),
           extraArgs, (? $pred, $inactive)), basename, wantXVariant, "_n",
           PNT_NType, PNT_NType>;
}

multiclass VectorVectorArithmeticBitcast<string operation> {
  defm "" : IntrinsicMX<Vector, (args Vector:$a, Vector:$b,
                                 Predicate:$pred),
                                (bitcast (IRInt<operation, [UVector, Predicate]>
                                          (bitcast $a, UVector),
````
- **L409 EN**: Blank line separating nearby declarations or logic blocks.
  **L409 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L410 EN**: Declares TableGen multiclass record `VectorVectorArithmetic`.
  **L410 CN**: 声明 TableGen multiclass 记录 `VectorVectorArithmetic`。
- **L411 EN**: Continues the surrounding expression or declaration: `bit wantXVariant = 1> {`.
  **L411 CN**: 继续构造周围的表达式或声明：`bit wantXVariant = 1> {`。
- **L412 EN**: Declares TableGen defm record `""`.
  **L412 CN**: 声明 TableGen defm 记录 `""`。
- **L413 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$a, Vector:$b, Predicate:$pred),`.
  **L413 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$a, Vector:$b, Predicate:$pred),`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!con((IRInt<operation, [Vector, Predicate]> $a, $b),`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`!con((IRInt<operation, [Vector, Predicate]> $a, $b),`。
- **L415 EN**: Executes a call or declaration centered on `extraArgs,`.
  **L415 CN**: 执行以 `extraArgs,` 为核心的调用或声明。
- **L416 EN**: Closes the current lexical scope or compound statement.
  **L416 CN**: 结束当前词法作用域或复合语句块。
- **L417 EN**: Blank line separating nearby declarations or logic blocks.
  **L417 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L418 EN**: Declares TableGen multiclass record `VectorScalarArithmetic`.
  **L418 CN**: 声明 TableGen multiclass 记录 `VectorScalarArithmetic`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dag extraArgs = (?),`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`dag extraArgs = (?),`。
- **L420 EN**: Continues the surrounding expression or declaration: `bit wantXVariant = 1> {`.
  **L420 CN**: 继续构造周围的表达式或声明：`bit wantXVariant = 1> {`。
- **L421 EN**: Declares TableGen defm record `""`.
  **L421 CN**: 声明 TableGen defm 记录 `""`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$a, unpromoted<Scalar>:$b, Predicate:$pred),`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$a, unpromoted<Scalar>:$b, Predicate:$pred),`。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!con((IRInt<operation, [Vector, Predicate]> $a, (splat $b)),`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`!con((IRInt<operation, [Vector, Predicate]> $a, (splat $b)),`。
- **L424 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `extraArgs, (? $pred, $inactive)), basename, wantXVariant, "_n",`.
  **L424 CN**: 继续一个多行参数列表、初始化器或聚合项：`extraArgs, (? $pred, $inactive)), basename, wantXVariant, "_n",`。
- **L425 EN**: Adds a standalone statement or declaration: `PNT_NType, PNT_NType>;`.
  **L425 CN**: 添加一条独立语句或声明：`PNT_NType, PNT_NType>;`。
- **L426 EN**: Closes the current lexical scope or compound statement.
  **L426 CN**: 结束当前词法作用域或复合语句块。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L428 EN**: Declares TableGen multiclass record `VectorVectorArithmeticBitcast`.
  **L428 CN**: 声明 TableGen multiclass 记录 `VectorVectorArithmeticBitcast`。
- **L429 EN**: Declares TableGen defm record `""`.
  **L429 CN**: 声明 TableGen defm 记录 `""`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L431 EN**: Continues logic associated with callable symbol `bitcast`.
  **L431 CN**: 继续与可调用符号 `bitcast` 相关的逻辑。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast $a, UVector),`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast $a, UVector),`。

### Lines 433-456

````tablegen
                                          (bitcast $b, UVector),
                                          $pred,
                                          (bitcast $inactive, UVector)), Vector)>;
}

// Predicated intrinsics
let params = T.Usual in {
  defm vabdq : VectorVectorArithmetic<"abd_predicated", (? (unsignedflag Scalar))>;
  defm vaddq : VectorVectorArithmetic<"add_predicated">;
  defm vsubq : VectorVectorArithmetic<"sub_predicated">;
  defm vmulq : VectorVectorArithmetic<"mul_predicated">;
  defm vandq : VectorVectorArithmeticBitcast<"and_predicated">;
  defm vbicq : VectorVectorArithmeticBitcast<"bic_predicated">;
  defm veorq : VectorVectorArithmeticBitcast<"eor_predicated">;
  defm vornq : VectorVectorArithmeticBitcast<"orn_predicated">;
  defm vorrq : VectorVectorArithmeticBitcast<"orr_predicated">;

  defm : VectorScalarArithmetic<"add_predicated", "vaddq">;
  defm : VectorScalarArithmetic<"sub_predicated", "vsubq">;
  defm : VectorScalarArithmetic<"mul_predicated", "vmulq">;
}

multiclass DblVectorVectorArithmetic<string operation, dag extraArgs = (?),
                                     bit wantXVariant = 1> {
````
- **L433 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast $b, UVector),`.
  **L433 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast $b, UVector),`。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$pred,`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`$pred,`。
- **L435 EN**: Executes a call or declaration centered on `statement`.
  **L435 CN**: 执行以 `statement` 为核心的调用或声明。
- **L436 EN**: Closes the current lexical scope or compound statement.
  **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Comment explains nearby logic, constraints, or intent: `Predicated intrinsics`.
  **L438 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicated intrinsics`。
- **L439 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Usual in {`.
  **L439 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Usual in {`。
- **L440 EN**: Declares TableGen defm record `vabdq`.
  **L440 CN**: 声明 TableGen defm 记录 `vabdq`。
- **L441 EN**: Declares TableGen defm record `vaddq`.
  **L441 CN**: 声明 TableGen defm 记录 `vaddq`。
- **L442 EN**: Declares TableGen defm record `vsubq`.
  **L442 CN**: 声明 TableGen defm 记录 `vsubq`。
- **L443 EN**: Declares TableGen defm record `vmulq`.
  **L443 CN**: 声明 TableGen defm 记录 `vmulq`。
- **L444 EN**: Declares TableGen defm record `vandq`.
  **L444 CN**: 声明 TableGen defm 记录 `vandq`。
- **L445 EN**: Declares TableGen defm record `vbicq`.
  **L445 CN**: 声明 TableGen defm 记录 `vbicq`。
- **L446 EN**: Declares TableGen defm record `veorq`.
  **L446 CN**: 声明 TableGen defm 记录 `veorq`。
- **L447 EN**: Declares TableGen defm record `vornq`.
  **L447 CN**: 声明 TableGen defm 记录 `vornq`。
- **L448 EN**: Declares TableGen defm record `vorrq`.
  **L448 CN**: 声明 TableGen defm 记录 `vorrq`。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L450 EN**: Declares TableGen defm record `defm`.
  **L450 CN**: 声明 TableGen defm 记录 `defm`。
- **L451 EN**: Declares TableGen defm record `defm`.
  **L451 CN**: 声明 TableGen defm 记录 `defm`。
- **L452 EN**: Declares TableGen defm record `defm`.
  **L452 CN**: 声明 TableGen defm 记录 `defm`。
- **L453 EN**: Closes the current lexical scope or compound statement.
  **L453 CN**: 结束当前词法作用域或复合语句块。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Declares TableGen multiclass record `DblVectorVectorArithmetic`.
  **L455 CN**: 声明 TableGen multiclass 记录 `DblVectorVectorArithmetic`。
- **L456 EN**: Continues the surrounding expression or declaration: `bit wantXVariant = 1> {`.
  **L456 CN**: 继续构造周围的表达式或声明：`bit wantXVariant = 1> {`。

### Lines 457-480

````tablegen
  defm "" : IntrinsicMX<
      DblVector, (args Vector:$a, Vector:$b, DblPredicate:$pred),
      !con((IRInt<operation, [DblVector, Vector, DblPredicate]> $a, $b),
           extraArgs, (? $pred, $inactive)), wantXVariant>;
}

multiclass DblVectorScalarArithmetic<string operation, string basename,
                                     dag extraArgs = (?),
                                     bit wantXVariant = 1> {
  defm "" : IntrinsicMXNameOverride<
      DblVector, (args Vector:$a, unpromoted<Scalar>:$b, DblPredicate:$pred),
      !con((IRInt<operation, [DblVector, Vector, DblPredicate]> $a, (splat $b)),
           extraArgs, (? $pred, $inactive)), basename, wantXVariant, "_n",
           PNT_NType, PNT_NType>;
}

// Predicated intrinsics - Int types only
let params = T.Int in {
  defm vminq : VectorVectorArithmetic<"min_predicated", (? (unsignedflag Scalar))>;
  defm vmaxq : VectorVectorArithmetic<"max_predicated", (? (unsignedflag Scalar))>;
  defm vmulhq : VectorVectorArithmetic<"mulh_predicated", (? (unsignedflag Scalar))>;
  defm vrmulhq : VectorVectorArithmetic<"rmulh_predicated", (? (unsignedflag Scalar))>;
  defm vqaddq : VectorVectorArithmetic<"qadd_predicated", (? (unsignedflag Scalar)), 0>;
  defm vhaddq : VectorVectorArithmetic<"hadd_predicated", (? (unsignedflag Scalar))>;
````
- **L457 EN**: Declares TableGen defm record `""`.
  **L457 CN**: 声明 TableGen defm 记录 `""`。
- **L458 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DblVector, (args Vector:$a, Vector:$b, DblPredicate:$pred),`.
  **L458 CN**: 继续一个多行参数列表、初始化器或聚合项：`DblVector, (args Vector:$a, Vector:$b, DblPredicate:$pred),`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!con((IRInt<operation, [DblVector, Vector, DblPredicate]> $a, $b),`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`!con((IRInt<operation, [DblVector, Vector, DblPredicate]> $a, $b),`。
- **L460 EN**: Executes a call or declaration centered on `extraArgs,`.
  **L460 CN**: 执行以 `extraArgs,` 为核心的调用或声明。
- **L461 EN**: Closes the current lexical scope or compound statement.
  **L461 CN**: 结束当前词法作用域或复合语句块。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L463 EN**: Declares TableGen multiclass record `DblVectorScalarArithmetic`.
  **L463 CN**: 声明 TableGen multiclass 记录 `DblVectorScalarArithmetic`。
- **L464 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dag extraArgs = (?),`.
  **L464 CN**: 继续一个多行参数列表、初始化器或聚合项：`dag extraArgs = (?),`。
- **L465 EN**: Continues the surrounding expression or declaration: `bit wantXVariant = 1> {`.
  **L465 CN**: 继续构造周围的表达式或声明：`bit wantXVariant = 1> {`。
- **L466 EN**: Declares TableGen defm record `""`.
  **L466 CN**: 声明 TableGen defm 记录 `""`。
- **L467 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DblVector, (args Vector:$a, unpromoted<Scalar>:$b, DblPredicate:$pred),`.
  **L467 CN**: 继续一个多行参数列表、初始化器或聚合项：`DblVector, (args Vector:$a, unpromoted<Scalar>:$b, DblPredicate:$pred),`。
- **L468 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!con((IRInt<operation, [DblVector, Vector, DblPredicate]> $a, (splat $b)),`.
  **L468 CN**: 继续一个多行参数列表、初始化器或聚合项：`!con((IRInt<operation, [DblVector, Vector, DblPredicate]> $a, (splat $b)),`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `extraArgs, (? $pred, $inactive)), basename, wantXVariant, "_n",`.
  **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`extraArgs, (? $pred, $inactive)), basename, wantXVariant, "_n",`。
- **L470 EN**: Adds a standalone statement or declaration: `PNT_NType, PNT_NType>;`.
  **L470 CN**: 添加一条独立语句或声明：`PNT_NType, PNT_NType>;`。
- **L471 EN**: Closes the current lexical scope or compound statement.
  **L471 CN**: 结束当前词法作用域或复合语句块。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L473 EN**: Comment explains nearby logic, constraints, or intent: `Predicated intrinsics - Int types only`.
  **L473 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicated intrinsics - Int types only`。
- **L474 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L474 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L475 EN**: Declares TableGen defm record `vminq`.
  **L475 CN**: 声明 TableGen defm 记录 `vminq`。
- **L476 EN**: Declares TableGen defm record `vmaxq`.
  **L476 CN**: 声明 TableGen defm 记录 `vmaxq`。
- **L477 EN**: Declares TableGen defm record `vmulhq`.
  **L477 CN**: 声明 TableGen defm 记录 `vmulhq`。
- **L478 EN**: Declares TableGen defm record `vrmulhq`.
  **L478 CN**: 声明 TableGen defm 记录 `vrmulhq`。
- **L479 EN**: Declares TableGen defm record `vqaddq`.
  **L479 CN**: 声明 TableGen defm 记录 `vqaddq`。
- **L480 EN**: Declares TableGen defm record `vhaddq`.
  **L480 CN**: 声明 TableGen defm 记录 `vhaddq`。

### Lines 481-504

````tablegen
  defm vrhaddq : VectorVectorArithmetic<"rhadd_predicated", (? (unsignedflag Scalar))>;
  defm vqsubq : VectorVectorArithmetic<"qsub_predicated", (? (unsignedflag Scalar)), 0>;
  defm vhsubq : VectorVectorArithmetic<"hsub_predicated", (? (unsignedflag Scalar))>;
  defm vmullbq_int : DblVectorVectorArithmetic<"mull_int_predicated", (? (unsignedflag Scalar), (u32 0))>;
  defm vmulltq_int : DblVectorVectorArithmetic<"mull_int_predicated", (? (unsignedflag Scalar), (u32 1))>;

  defm : VectorScalarArithmetic<"qadd_predicated", "vqaddq", (? (unsignedflag Scalar)), 0>;
  defm : VectorScalarArithmetic<"hadd_predicated", "vhaddq", (? (unsignedflag Scalar))>;
  defm : VectorScalarArithmetic<"qsub_predicated", "vqsubq", (? (unsignedflag Scalar)), 0>;
  defm : VectorScalarArithmetic<"hsub_predicated", "vhsubq", (? (unsignedflag Scalar))>;
}
let params = T.Signed in {
  defm vqdmulhq : VectorVectorArithmetic<"qdmulh_predicated", (?), 0>;
  defm vqrdmulhq : VectorVectorArithmetic<"qrdmulh_predicated", (?), 0>;
  def vminaq_m: Intrinsic<UVector, (args UVector:$a, Vector:$b, Predicate:$pred),
                                   (IRInt<"vmina_predicated", [UVector,Predicate]> $a, $b, $pred)>;
  def vmaxaq_m: Intrinsic<UVector, (args UVector:$a, Vector:$b, Predicate:$pred),
                                   (IRInt<"vmaxa_predicated", [UVector,Predicate]> $a, $b, $pred)>;

  defm : VectorScalarArithmetic<"qdmulh_predicated", "vqdmulhq", (?), 0>;
  defm : VectorScalarArithmetic<"qrdmulh_predicated", "vqrdmulhq", (?), 0>;
}

let params = T.Poly, overrideKindLetter = "p" in {
````
- **L481 EN**: Declares TableGen defm record `vrhaddq`.
  **L481 CN**: 声明 TableGen defm 记录 `vrhaddq`。
- **L482 EN**: Declares TableGen defm record `vqsubq`.
  **L482 CN**: 声明 TableGen defm 记录 `vqsubq`。
- **L483 EN**: Declares TableGen defm record `vhsubq`.
  **L483 CN**: 声明 TableGen defm 记录 `vhsubq`。
- **L484 EN**: Declares TableGen defm record `vmullbq_int`.
  **L484 CN**: 声明 TableGen defm 记录 `vmullbq_int`。
- **L485 EN**: Declares TableGen defm record `vmulltq_int`.
  **L485 CN**: 声明 TableGen defm 记录 `vmulltq_int`。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L487 EN**: Declares TableGen defm record `defm`.
  **L487 CN**: 声明 TableGen defm 记录 `defm`。
- **L488 EN**: Declares TableGen defm record `defm`.
  **L488 CN**: 声明 TableGen defm 记录 `defm`。
- **L489 EN**: Declares TableGen defm record `defm`.
  **L489 CN**: 声明 TableGen defm 记录 `defm`。
- **L490 EN**: Declares TableGen defm record `defm`.
  **L490 CN**: 声明 TableGen defm 记录 `defm`。
- **L491 EN**: Closes the current lexical scope or compound statement.
  **L491 CN**: 结束当前词法作用域或复合语句块。
- **L492 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L492 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L493 EN**: Declares TableGen defm record `vqdmulhq`.
  **L493 CN**: 声明 TableGen defm 记录 `vqdmulhq`。
- **L494 EN**: Declares TableGen defm record `vqrdmulhq`.
  **L494 CN**: 声明 TableGen defm 记录 `vqrdmulhq`。
- **L495 EN**: Declares TableGen def record `vminaq_m`.
  **L495 CN**: 声明 TableGen def 记录 `vminaq_m`。
- **L496 EN**: Executes a call or declaration centered on `statement`.
  **L496 CN**: 执行以 `statement` 为核心的调用或声明。
- **L497 EN**: Declares TableGen def record `vmaxaq_m`.
  **L497 CN**: 声明 TableGen def 记录 `vmaxaq_m`。
- **L498 EN**: Executes a call or declaration centered on `statement`.
  **L498 CN**: 执行以 `statement` 为核心的调用或声明。
- **L499 EN**: Blank line separating nearby declarations or logic blocks.
  **L499 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L500 EN**: Declares TableGen defm record `defm`.
  **L500 CN**: 声明 TableGen defm 记录 `defm`。
- **L501 EN**: Declares TableGen defm record `defm`.
  **L501 CN**: 声明 TableGen defm 记录 `defm`。
- **L502 EN**: Closes the current lexical scope or compound statement.
  **L502 CN**: 结束当前词法作用域或复合语句块。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Poly, overrideKindLetter = "p" in {`.
  **L504 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Poly, overrideKindLetter = "p" in {`。

### Lines 505-528

````tablegen
  defm vmullbq_poly : DblVectorVectorArithmetic<"mull_poly_predicated", (? (u32 0))>;
  defm vmulltq_poly : DblVectorVectorArithmetic<"mull_poly_predicated", (? (u32 1))>;
}

let params = [s16, s32] in {
  def  vqdmullbq:  Intrinsic<DblVector, (args Vector:$a, Vector:$b),
                             (IRInt<"vqdmull", [DblVector, Vector]> $a, $b, 0)>;
  def  vqdmulltq:  Intrinsic<DblVector, (args Vector:$a, Vector:$b),
                             (IRInt<"vqdmull", [DblVector, Vector]> $a, $b, 1)>;
  defm vqdmullbq:  DblVectorVectorArithmetic<"vqdmull_predicated", (? (u32 0)), 0>;
  defm vqdmulltq:  DblVectorVectorArithmetic<"vqdmull_predicated", (? (u32 1)), 0>;

  let pnt = PNT_NType in {
    def vqdmullbq_n: Intrinsic<DblVector, (args Vector:$a, unpromoted<Scalar>:$b),
                               (IRInt<"vqdmull", [DblVector, Vector]>
                                $a, (splat $b), 0)>;
    def vqdmulltq_n: Intrinsic<DblVector, (args Vector:$a, unpromoted<Scalar>:$b),
                               (IRInt<"vqdmull", [DblVector, Vector]>
                                $a, (splat $b), 1)>;
  }
  defm vqdmullbq_n: DblVectorScalarArithmetic<"vqdmull_predicated",
                                              "vqdmullbq", (? (u32 0)), 0>;
  defm vqdmulltq_n: DblVectorScalarArithmetic<"vqdmull_predicated",
                                              "vqdmulltq", (? (u32 1)), 0>;
````
- **L505 EN**: Declares TableGen defm record `vmullbq_poly`.
  **L505 CN**: 声明 TableGen defm 记录 `vmullbq_poly`。
- **L506 EN**: Declares TableGen defm record `vmulltq_poly`.
  **L506 CN**: 声明 TableGen defm 记录 `vmulltq_poly`。
- **L507 EN**: Closes the current lexical scope or compound statement.
  **L507 CN**: 结束当前词法作用域或复合语句块。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L509 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, s32] in {`.
  **L509 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, s32] in {`。
- **L510 EN**: Declares TableGen def record `vqdmullbq`.
  **L510 CN**: 声明 TableGen def 记录 `vqdmullbq`。
- **L511 EN**: Executes a call or declaration centered on `statement`.
  **L511 CN**: 执行以 `statement` 为核心的调用或声明。
- **L512 EN**: Declares TableGen def record `vqdmulltq`.
  **L512 CN**: 声明 TableGen def 记录 `vqdmulltq`。
- **L513 EN**: Executes a call or declaration centered on `statement`.
  **L513 CN**: 执行以 `statement` 为核心的调用或声明。
- **L514 EN**: Declares TableGen defm record `vqdmullbq`.
  **L514 CN**: 声明 TableGen defm 记录 `vqdmullbq`。
- **L515 EN**: Declares TableGen defm record `vqdmulltq`.
  **L515 CN**: 声明 TableGen defm 记录 `vqdmulltq`。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L517 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L517 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L518 EN**: Declares TableGen def record `vqdmullbq_n`.
  **L518 CN**: 声明 TableGen def 记录 `vqdmullbq_n`。
- **L519 EN**: Continues the surrounding expression or declaration: `(IRInt<"vqdmull", [DblVector, Vector]>`.
  **L519 CN**: 继续构造周围的表达式或声明：`(IRInt<"vqdmull", [DblVector, Vector]>`。
- **L520 EN**: Executes a call or declaration centered on `$a,`.
  **L520 CN**: 执行以 `$a,` 为核心的调用或声明。
- **L521 EN**: Declares TableGen def record `vqdmulltq_n`.
  **L521 CN**: 声明 TableGen def 记录 `vqdmulltq_n`。
- **L522 EN**: Continues the surrounding expression or declaration: `(IRInt<"vqdmull", [DblVector, Vector]>`.
  **L522 CN**: 继续构造周围的表达式或声明：`(IRInt<"vqdmull", [DblVector, Vector]>`。
- **L523 EN**: Executes a call or declaration centered on `$a,`.
  **L523 CN**: 执行以 `$a,` 为核心的调用或声明。
- **L524 EN**: Closes the current lexical scope or compound statement.
  **L524 CN**: 结束当前词法作用域或复合语句块。
- **L525 EN**: Declares TableGen defm record `vqdmullbq_n`.
  **L525 CN**: 声明 TableGen defm 记录 `vqdmullbq_n`。
- **L526 EN**: Executes a call or declaration centered on `"vqdmullbq",`.
  **L526 CN**: 执行以 `"vqdmullbq",` 为核心的调用或声明。
- **L527 EN**: Declares TableGen defm record `vqdmulltq_n`.
  **L527 CN**: 声明 TableGen defm 记录 `vqdmulltq_n`。
- **L528 EN**: Executes a call or declaration centered on `"vqdmulltq",`.
  **L528 CN**: 执行以 `"vqdmulltq",` 为核心的调用或声明。

### Lines 529-552

````tablegen
}

// Predicated intrinsics - Float types only
let params = T.Float in {
  defm vminnmq : VectorVectorArithmetic<"min_predicated", (? (u32 0))>;
  defm vmaxnmq : VectorVectorArithmetic<"max_predicated", (? (u32 0))>;
  def vminnmaq_m: Intrinsic<Vector, (args Vector:$a, Vector:$b, Predicate:$pred),
                                    (IRInt<"vminnma_predicated", [Vector,Predicate]> $a, $b, $pred)>;
  def vmaxnmaq_m: Intrinsic<Vector, (args Vector:$a, Vector:$b, Predicate:$pred),
                                    (IRInt<"vmaxnma_predicated", [Vector,Predicate]> $a, $b, $pred)>;
}

multiclass Reduction<Type Accumulator, string basename, list<Type> basetypes,
                     bit needSign = 0,
                     dag postCG = (seq (id $ret)),
                     dag accArg = (args Accumulator:$prev),
                     dag preCG = (seq)> {
  defvar intArgsBase   = (? $prev, $vec);
  defvar intArgsUnpred = !con(intArgsBase,
                              !if(needSign, (? (unsignedflag Scalar)), (?)));
  defvar intArgsPred   = !con(intArgsUnpred, (? $pred));
  defvar intUnpred     = !setdagop(intArgsUnpred, IRInt<basename, basetypes>);
  defvar intPred       = !setdagop(intArgsPred, IRInt<
            basename#"_predicated", !listconcat(basetypes, [Predicate])>);
````
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L531 EN**: Comment explains nearby logic, constraints, or intent: `Predicated intrinsics - Float types only`.
  **L531 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Predicated intrinsics - Float types only`。
- **L532 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L532 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L533 EN**: Declares TableGen defm record `vminnmq`.
  **L533 CN**: 声明 TableGen defm 记录 `vminnmq`。
- **L534 EN**: Declares TableGen defm record `vmaxnmq`.
  **L534 CN**: 声明 TableGen defm 记录 `vmaxnmq`。
- **L535 EN**: Declares TableGen def record `vminnmaq_m`.
  **L535 CN**: 声明 TableGen def 记录 `vminnmaq_m`。
- **L536 EN**: Executes a call or declaration centered on `statement`.
  **L536 CN**: 执行以 `statement` 为核心的调用或声明。
- **L537 EN**: Declares TableGen def record `vmaxnmaq_m`.
  **L537 CN**: 声明 TableGen def 记录 `vmaxnmaq_m`。
- **L538 EN**: Executes a call or declaration centered on `statement`.
  **L538 CN**: 执行以 `statement` 为核心的调用或声明。
- **L539 EN**: Closes the current lexical scope or compound statement.
  **L539 CN**: 结束当前词法作用域或复合语句块。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L541 EN**: Declares TableGen multiclass record `Reduction`.
  **L541 CN**: 声明 TableGen multiclass 记录 `Reduction`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bit needSign = 0,`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`bit needSign = 0,`。
- **L543 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dag postCG = (seq (id $ret)),`.
  **L543 CN**: 继续一个多行参数列表、初始化器或聚合项：`dag postCG = (seq (id $ret)),`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `dag accArg = (args Accumulator:$prev),`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`dag accArg = (args Accumulator:$prev),`。
- **L545 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `dag preCG = (seq)> {`.
  **L545 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`dag preCG = (seq)> {`。
- **L546 EN**: Declares TableGen defvar record `intArgsBase   =`.
  **L546 CN**: 声明 TableGen defvar 记录 `intArgsBase   =`。
- **L547 EN**: Declares TableGen defvar record `intArgsUnpred = !con`.
  **L547 CN**: 声明 TableGen defvar 记录 `intArgsUnpred = !con`。
- **L548 EN**: Executes a call or declaration centered on `!if`.
  **L548 CN**: 执行以 `!if` 为核心的调用或声明。
- **L549 EN**: Declares TableGen defvar record `intArgsPred   = !con`.
  **L549 CN**: 声明 TableGen defvar 记录 `intArgsPred   = !con`。
- **L550 EN**: Declares TableGen defvar record `intUnpred     = !setdagop`.
  **L550 CN**: 声明 TableGen defvar 记录 `intUnpred     = !setdagop`。
- **L551 EN**: Declares TableGen defvar record `intPred       = !setdagop`.
  **L551 CN**: 声明 TableGen defvar 记录 `intPred       = !setdagop`。
- **L552 EN**: Executes a call or declaration centered on `!listconcat`.
  **L552 CN**: 执行以 `!listconcat` 为核心的调用或声明。

### Lines 553-576

````tablegen

  def "": Intrinsic<
    Accumulator, !con(accArg, (args Vector:$vec)),
    !con(preCG, (seq intUnpred:$ret), postCG)>;
  def _p: Intrinsic<
    Accumulator, !con(accArg, (args Vector:$vec, Predicate:$pred)),
    !con(preCG, (seq intPred:$ret), postCG)>;
}

let params = T.Int in {
defm vminvq: Reduction<Scalar, "minv", [Vector], 1, (seq (Scalar $ret))>;
defm vmaxvq: Reduction<Scalar, "maxv", [Vector], 1, (seq (Scalar $ret))>;
}

let params = T.Signed in {
defm vminavq: Reduction<UScalar, "minav", [Vector], 0, (seq (UScalar $ret))>;
defm vmaxavq: Reduction<UScalar, "maxav", [Vector], 0, (seq (UScalar $ret))>;
}

let params = T.Float in {
defm vminnmvq: Reduction<Scalar, "minnmv", [Scalar, Vector]>;
defm vmaxnmvq: Reduction<Scalar, "maxnmv", [Scalar, Vector]>;
defm vminnmavq: Reduction<Scalar, "minnmav", [Scalar, Vector]>;
defm vmaxnmavq: Reduction<Scalar, "maxnmav", [Scalar, Vector]>;
````
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L554 EN**: Declares TableGen def record `""`.
  **L554 CN**: 声明 TableGen def 记录 `""`。
- **L555 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Accumulator, !con(accArg, (args Vector:$vec)),`.
  **L555 CN**: 继续一个多行参数列表、初始化器或聚合项：`Accumulator, !con(accArg, (args Vector:$vec)),`。
- **L556 EN**: Executes a call or declaration centered on `!con`.
  **L556 CN**: 执行以 `!con` 为核心的调用或声明。
- **L557 EN**: Declares TableGen def record `_p`.
  **L557 CN**: 声明 TableGen def 记录 `_p`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Accumulator, !con(accArg, (args Vector:$vec, Predicate:$pred)),`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`Accumulator, !con(accArg, (args Vector:$vec, Predicate:$pred)),`。
- **L559 EN**: Executes a call or declaration centered on `!con`.
  **L559 CN**: 执行以 `!con` 为核心的调用或声明。
- **L560 EN**: Closes the current lexical scope or compound statement.
  **L560 CN**: 结束当前词法作用域或复合语句块。
- **L561 EN**: Blank line separating nearby declarations or logic blocks.
  **L561 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L562 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L562 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L563 EN**: Declares TableGen defm record `vminvq`.
  **L563 CN**: 声明 TableGen defm 记录 `vminvq`。
- **L564 EN**: Declares TableGen defm record `vmaxvq`.
  **L564 CN**: 声明 TableGen defm 记录 `vmaxvq`。
- **L565 EN**: Closes the current lexical scope or compound statement.
  **L565 CN**: 结束当前词法作用域或复合语句块。
- **L566 EN**: Blank line separating nearby declarations or logic blocks.
  **L566 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L567 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L567 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L568 EN**: Declares TableGen defm record `vminavq`.
  **L568 CN**: 声明 TableGen defm 记录 `vminavq`。
- **L569 EN**: Declares TableGen defm record `vmaxavq`.
  **L569 CN**: 声明 TableGen defm 记录 `vmaxavq`。
- **L570 EN**: Closes the current lexical scope or compound statement.
  **L570 CN**: 结束当前词法作用域或复合语句块。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L572 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L573 EN**: Declares TableGen defm record `vminnmvq`.
  **L573 CN**: 声明 TableGen defm 记录 `vminnmvq`。
- **L574 EN**: Declares TableGen defm record `vmaxnmvq`.
  **L574 CN**: 声明 TableGen defm 记录 `vmaxnmvq`。
- **L575 EN**: Declares TableGen defm record `vminnmavq`.
  **L575 CN**: 声明 TableGen defm 记录 `vminnmavq`。
- **L576 EN**: Declares TableGen defm record `vmaxnmavq`.
  **L576 CN**: 声明 TableGen defm 记录 `vmaxnmavq`。

### Lines 577-600

````tablegen
}

foreach half = [ "b", "t" ] in {
  defvar halfconst = !ne(half, "b");

  let params = [f32], pnt = PNT_None in {
    def vcvt#half#q_f16: Intrinsic<
      VecOf<f16>, (args VecOf<f16>:$inactive, Vector:$a),
      (IRInt<"vcvt_narrow"> $inactive, $a, halfconst)>;
    def vcvt#half#q_m_f16: Intrinsic<
      VecOf<f16>, (args VecOf<f16>:$inactive, Vector:$a, PredOf<f32>:$pred),
      (IRInt<"vcvt_narrow_predicated"> $inactive, $a, halfconst, $pred)>;
  } // params = [f32], pnt = PNT_None

  let params = [f16], pnt = PNT_None in {
    def vcvt#half#q_f32: Intrinsic<VecOf<f32>, (args Vector:$a),
      (IRInt<"vcvt_widen"> $a, halfconst)>;
    defm vcvt#half#q: IntrinsicMX<
      VecOf<f32>, (args Vector:$a, PredOf<f32>:$pred),
      (IRInt<"vcvt_widen_predicated"> $inactive, $a, halfconst, $pred),
      1, "_f32">;
  } // params = [f16], pnt = PNT_None
} // loop over half = "b", "t"

````
- **L577 EN**: Closes the current lexical scope or compound statement.
  **L577 CN**: 结束当前词法作用域或复合语句块。
- **L578 EN**: Blank line separating nearby declarations or logic blocks.
  **L578 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L579 EN**: Starts a TableGen iteration used to generate repeated records: `foreach half = [ "b", "t" ] in {`.
  **L579 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach half = [ "b", "t" ] in {`。
- **L580 EN**: Declares TableGen defvar record `halfconst = !ne`.
  **L580 CN**: 声明 TableGen defvar 记录 `halfconst = !ne`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [f32], pnt = PNT_None in {`.
  **L582 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [f32], pnt = PNT_None in {`。
- **L583 EN**: Declares TableGen def record `vcvt#half#q_f16`.
  **L583 CN**: 声明 TableGen def 记录 `vcvt#half#q_f16`。
- **L584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VecOf<f16>, (args VecOf<f16>:$inactive, Vector:$a),`.
  **L584 CN**: 继续一个多行参数列表、初始化器或聚合项：`VecOf<f16>, (args VecOf<f16>:$inactive, Vector:$a),`。
- **L585 EN**: Executes a call or declaration centered on `statement`.
  **L585 CN**: 执行以 `statement` 为核心的调用或声明。
- **L586 EN**: Declares TableGen def record `vcvt#half#q_m_f16`.
  **L586 CN**: 声明 TableGen def 记录 `vcvt#half#q_m_f16`。
- **L587 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VecOf<f16>, (args VecOf<f16>:$inactive, Vector:$a, PredOf<f32>:$pred),`.
  **L587 CN**: 继续一个多行参数列表、初始化器或聚合项：`VecOf<f16>, (args VecOf<f16>:$inactive, Vector:$a, PredOf<f32>:$pred),`。
- **L588 EN**: Executes a call or declaration centered on `statement`.
  **L588 CN**: 执行以 `statement` 为核心的调用或声明。
- **L589 EN**: Continues the surrounding expression or declaration: `} // params = [f32], pnt = PNT_None`.
  **L589 CN**: 继续构造周围的表达式或声明：`} // params = [f32], pnt = PNT_None`。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [f16], pnt = PNT_None in {`.
  **L591 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [f16], pnt = PNT_None in {`。
- **L592 EN**: Declares TableGen def record `vcvt#half#q_f32`.
  **L592 CN**: 声明 TableGen def 记录 `vcvt#half#q_f32`。
- **L593 EN**: Executes a call or declaration centered on `statement`.
  **L593 CN**: 执行以 `statement` 为核心的调用或声明。
- **L594 EN**: Declares TableGen defm record `vcvt#half#q`.
  **L594 CN**: 声明 TableGen defm 记录 `vcvt#half#q`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VecOf<f32>, (args Vector:$a, PredOf<f32>:$pred),`.
  **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`VecOf<f32>, (args Vector:$a, PredOf<f32>:$pred),`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vcvt_widen_predicated"> $inactive, $a, halfconst, $pred),`.
  **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vcvt_widen_predicated"> $inactive, $a, halfconst, $pred),`。
- **L597 EN**: Adds a standalone statement or declaration: `1, "_f32">;`.
  **L597 CN**: 添加一条独立语句或声明：`1, "_f32">;`。
- **L598 EN**: Continues the surrounding expression or declaration: `} // params = [f16], pnt = PNT_None`.
  **L598 CN**: 继续构造周围的表达式或声明：`} // params = [f16], pnt = PNT_None`。
- **L599 EN**: Continues the surrounding expression or declaration: `} // loop over half = "b", "t"`.
  **L599 CN**: 继续构造周围的表达式或声明：`} // loop over half = "b", "t"`。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 601-624

````tablegen
multiclass float_int_conversions<Type FScalar, Type IScalar, Builder ftoi, Builder itof> {
  defvar FVector = VecOf<FScalar>;
  defvar IVector = VecOf<IScalar>;

  let params = [IScalar] in {
    let pnt = PNT_2Type in {
      def : Intrinsic<FVector, (args IVector:$a), (itof $a, FVector)>,
            NameOverride<"vcvtq_" # FScalar>;
    }
    defm vcvtq: IntrinsicMX<FVector, (args IVector:$a, Predicate:$pred),
        (IRInt<"vcvt_fp_int_predicated", [FVector, IVector, Predicate]>
            $a, (unsignedflag IScalar), $pred, $inactive),
        1, "_" # FScalar, PNT_2Type, PNT_2Type>;
  }
  let params = [FScalar] in {
    let pnt = PNT_None in {
      def : Intrinsic<IVector, (args FVector:$a), (ftoi $a, IVector)>,
            NameOverride<"vcvtq_" # IScalar>;

      foreach suffix = ["a","n","p","m"] in
        def : Intrinsic<IVector, (args FVector:$a),
                        (IRInt<"vcvt"#suffix, [IVector, FVector]>
                            (unsignedflag IScalar), $a)>,
              NameOverride<"vcvt"#suffix#"q_" # IScalar>;
````
- **L601 EN**: Declares TableGen multiclass record `float_int_conversions`.
  **L601 CN**: 声明 TableGen multiclass 记录 `float_int_conversions`。
- **L602 EN**: Declares TableGen defvar record `FVector = VecOf`.
  **L602 CN**: 声明 TableGen defvar 记录 `FVector = VecOf`。
- **L603 EN**: Declares TableGen defvar record `IVector = VecOf`.
  **L603 CN**: 声明 TableGen defvar 记录 `IVector = VecOf`。
- **L604 EN**: Blank line separating nearby declarations or logic blocks.
  **L604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L605 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [IScalar] in {`.
  **L605 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [IScalar] in {`。
- **L606 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_2Type in {`.
  **L606 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_2Type in {`。
- **L607 EN**: Declares TableGen def record `def`.
  **L607 CN**: 声明 TableGen def 记录 `def`。
- **L608 EN**: Adds a standalone statement or declaration: `NameOverride<"vcvtq_" # FScalar>;`.
  **L608 CN**: 添加一条独立语句或声明：`NameOverride<"vcvtq_" # FScalar>;`。
- **L609 EN**: Closes the current lexical scope or compound statement.
  **L609 CN**: 结束当前词法作用域或复合语句块。
- **L610 EN**: Declares TableGen defm record `vcvtq`.
  **L610 CN**: 声明 TableGen defm 记录 `vcvtq`。
- **L611 EN**: Continues the surrounding expression or declaration: `(IRInt<"vcvt_fp_int_predicated", [FVector, IVector, Predicate]>`.
  **L611 CN**: 继续构造周围的表达式或声明：`(IRInt<"vcvt_fp_int_predicated", [FVector, IVector, Predicate]>`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$a, (unsignedflag IScalar), $pred, $inactive),`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`$a, (unsignedflag IScalar), $pred, $inactive),`。
- **L613 EN**: Adds a standalone statement or declaration: `1, "_" # FScalar, PNT_2Type, PNT_2Type>;`.
  **L613 CN**: 添加一条独立语句或声明：`1, "_" # FScalar, PNT_2Type, PNT_2Type>;`。
- **L614 EN**: Closes the current lexical scope or compound statement.
  **L614 CN**: 结束当前词法作用域或复合语句块。
- **L615 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [FScalar] in {`.
  **L615 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [FScalar] in {`。
- **L616 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_None in {`.
  **L616 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_None in {`。
- **L617 EN**: Declares TableGen def record `def`.
  **L617 CN**: 声明 TableGen def 记录 `def`。
- **L618 EN**: Adds a standalone statement or declaration: `NameOverride<"vcvtq_" # IScalar>;`.
  **L618 CN**: 添加一条独立语句或声明：`NameOverride<"vcvtq_" # IScalar>;`。
- **L619 EN**: Blank line separating nearby declarations or logic blocks.
  **L619 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L620 EN**: Starts a TableGen iteration used to generate repeated records: `foreach suffix = ["a","n","p","m"] in`.
  **L620 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach suffix = ["a","n","p","m"] in`。
- **L621 EN**: Declares TableGen def record `def`.
  **L621 CN**: 声明 TableGen def 记录 `def`。
- **L622 EN**: Continues the surrounding expression or declaration: `(IRInt<"vcvt"#suffix, [IVector, FVector]>`.
  **L622 CN**: 继续构造周围的表达式或声明：`(IRInt<"vcvt"#suffix, [IVector, FVector]>`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag IScalar), $a)>,`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag IScalar), $a)>,`。
- **L624 EN**: Adds a standalone statement or declaration: `NameOverride<"vcvt"#suffix#"q_" # IScalar>;`.
  **L624 CN**: 添加一条独立语句或声明：`NameOverride<"vcvt"#suffix#"q_" # IScalar>;`。

### Lines 625-648

````tablegen
    }
    defm vcvtq: IntrinsicMX<IVector, (args FVector:$a, Predicate:$pred),
        (IRInt<"vcvt_fp_int_predicated", [IVector, FVector, Predicate]>
            $a, (unsignedflag IScalar), $pred, $inactive),
        1, "_" # IScalar, PNT_2Type, PNT_None>;

    foreach suffix = ["a","n","p","m"] in {
      defm "vcvt"#suffix#"q" : IntrinsicMX<
          IVector, (args FVector:$a, Predicate:$pred),
          (IRInt<"vcvt"#suffix#"_predicated", [IVector, FVector, Predicate]>
              (unsignedflag IScalar), $inactive, $a, $pred),
          1, "_" # IScalar, PNT_2Type, PNT_None>;
    }
  }
}

defm "" : float_int_conversions<f32, u32, fptoui, uitofp>;
defm "" : float_int_conversions<f16, u16, fptoui, uitofp>;
defm "" : float_int_conversions<f32, s32, fptosi, sitofp>;
defm "" : float_int_conversions<f16, s16, fptosi, sitofp>;

multiclass vmovl<bit top> {
  let params = [s8, u8, s16, u16] in {
    def "": Intrinsic<DblVector, (args Vector:$a),
````
- **L625 EN**: Closes the current lexical scope or compound statement.
  **L625 CN**: 结束当前词法作用域或复合语句块。
- **L626 EN**: Declares TableGen defm record `vcvtq`.
  **L626 CN**: 声明 TableGen defm 记录 `vcvtq`。
- **L627 EN**: Continues the surrounding expression or declaration: `(IRInt<"vcvt_fp_int_predicated", [IVector, FVector, Predicate]>`.
  **L627 CN**: 继续构造周围的表达式或声明：`(IRInt<"vcvt_fp_int_predicated", [IVector, FVector, Predicate]>`。
- **L628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$a, (unsignedflag IScalar), $pred, $inactive),`.
  **L628 CN**: 继续一个多行参数列表、初始化器或聚合项：`$a, (unsignedflag IScalar), $pred, $inactive),`。
- **L629 EN**: Adds a standalone statement or declaration: `1, "_" # IScalar, PNT_2Type, PNT_None>;`.
  **L629 CN**: 添加一条独立语句或声明：`1, "_" # IScalar, PNT_2Type, PNT_None>;`。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L631 EN**: Starts a TableGen iteration used to generate repeated records: `foreach suffix = ["a","n","p","m"] in {`.
  **L631 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach suffix = ["a","n","p","m"] in {`。
- **L632 EN**: Declares TableGen defm record `"vcvt"#suffix#"q"`.
  **L632 CN**: 声明 TableGen defm 记录 `"vcvt"#suffix#"q"`。
- **L633 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IVector, (args FVector:$a, Predicate:$pred),`.
  **L633 CN**: 继续一个多行参数列表、初始化器或聚合项：`IVector, (args FVector:$a, Predicate:$pred),`。
- **L634 EN**: Continues the surrounding expression or declaration: `(IRInt<"vcvt"#suffix#"_predicated", [IVector, FVector, Predicate]>`.
  **L634 CN**: 继续构造周围的表达式或声明：`(IRInt<"vcvt"#suffix#"_predicated", [IVector, FVector, Predicate]>`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag IScalar), $inactive, $a, $pred),`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag IScalar), $inactive, $a, $pred),`。
- **L636 EN**: Adds a standalone statement or declaration: `1, "_" # IScalar, PNT_2Type, PNT_None>;`.
  **L636 CN**: 添加一条独立语句或声明：`1, "_" # IScalar, PNT_2Type, PNT_None>;`。
- **L637 EN**: Closes the current lexical scope or compound statement.
  **L637 CN**: 结束当前词法作用域或复合语句块。
- **L638 EN**: Closes the current lexical scope or compound statement.
  **L638 CN**: 结束当前词法作用域或复合语句块。
- **L639 EN**: Closes the current lexical scope or compound statement.
  **L639 CN**: 结束当前词法作用域或复合语句块。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L641 EN**: Declares TableGen defm record `""`.
  **L641 CN**: 声明 TableGen defm 记录 `""`。
- **L642 EN**: Declares TableGen defm record `""`.
  **L642 CN**: 声明 TableGen defm 记录 `""`。
- **L643 EN**: Declares TableGen defm record `""`.
  **L643 CN**: 声明 TableGen defm 记录 `""`。
- **L644 EN**: Declares TableGen defm record `""`.
  **L644 CN**: 声明 TableGen defm 记录 `""`。
- **L645 EN**: Blank line separating nearby declarations or logic blocks.
  **L645 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L646 EN**: Declares TableGen multiclass record `vmovl`.
  **L646 CN**: 声明 TableGen multiclass 记录 `vmovl`。
- **L647 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s8, u8, s16, u16] in {`.
  **L647 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s8, u8, s16, u16] in {`。
- **L648 EN**: Declares TableGen def record `""`.
  **L648 CN**: 声明 TableGen def 记录 `""`。

### Lines 649-672

````tablegen
      (extend (unzip $a, top), DblVector, (unsignedflag Scalar))>;
    defm "": IntrinsicMX<DblVector, (args Vector:$a, DblPredicate:$pred),
        (IRInt<"vmovl_predicated", [DblVector, Vector, DblPredicate]>
         $a, (unsignedflag Scalar), top, $pred, $inactive)>;
  }
}

defm vmovlbq: vmovl<0>;
defm vmovltq: vmovl<1>;

multiclass vmovn<bit top, dag wide_result> {
  let params = [s16, u16, s32, u32] in {
    def "": Intrinsic<HalfVector, (args HalfVector:$inactive, Vector:$a),
      (trunc wide_result, HalfVector)>;
    def _m: Intrinsic<HalfVector, (args HalfVector:$inactive, Vector:$a,
                                        Predicate:$pred),
      (IRInt<"vmovn_predicated", [HalfVector, Vector, Predicate]>
          $inactive, $a, top, $pred)>;
  }
}

defm vmovntq: vmovn<1, (zip (vreinterpret $inactive, Vector), $a)>;
defm vmovnbq: vmovn<0,
   (zip $a, (vreinterpret (vrev $inactive, (bitsize Scalar)), Vector))>;
````
- **L649 EN**: Executes a call or declaration centered on `statement`.
  **L649 CN**: 执行以 `statement` 为核心的调用或声明。
- **L650 EN**: Declares TableGen defm record `""`.
  **L650 CN**: 声明 TableGen defm 记录 `""`。
- **L651 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmovl_predicated", [DblVector, Vector, DblPredicate]>`.
  **L651 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmovl_predicated", [DblVector, Vector, DblPredicate]>`。
- **L652 EN**: Executes a call or declaration centered on `$a,`.
  **L652 CN**: 执行以 `$a,` 为核心的调用或声明。
- **L653 EN**: Closes the current lexical scope or compound statement.
  **L653 CN**: 结束当前词法作用域或复合语句块。
- **L654 EN**: Closes the current lexical scope or compound statement.
  **L654 CN**: 结束当前词法作用域或复合语句块。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L656 EN**: Declares TableGen defm record `vmovlbq`.
  **L656 CN**: 声明 TableGen defm 记录 `vmovlbq`。
- **L657 EN**: Declares TableGen defm record `vmovltq`.
  **L657 CN**: 声明 TableGen defm 记录 `vmovltq`。
- **L658 EN**: Blank line separating nearby declarations or logic blocks.
  **L658 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L659 EN**: Declares TableGen multiclass record `vmovn`.
  **L659 CN**: 声明 TableGen multiclass 记录 `vmovn`。
- **L660 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, u16, s32, u32] in {`.
  **L660 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, u16, s32, u32] in {`。
- **L661 EN**: Declares TableGen def record `""`.
  **L661 CN**: 声明 TableGen def 记录 `""`。
- **L662 EN**: Executes a call or declaration centered on `statement`.
  **L662 CN**: 执行以 `statement` 为核心的调用或声明。
- **L663 EN**: Declares TableGen def record `_m`.
  **L663 CN**: 声明 TableGen def 记录 `_m`。
- **L664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L664 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L665 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmovn_predicated", [HalfVector, Vector, Predicate]>`.
  **L665 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmovn_predicated", [HalfVector, Vector, Predicate]>`。
- **L666 EN**: Adds a standalone statement or declaration: `$inactive, $a, top, $pred)>;`.
  **L666 CN**: 添加一条独立语句或声明：`$inactive, $a, top, $pred)>;`。
- **L667 EN**: Closes the current lexical scope or compound statement.
  **L667 CN**: 结束当前词法作用域或复合语句块。
- **L668 EN**: Closes the current lexical scope or compound statement.
  **L668 CN**: 结束当前词法作用域或复合语句块。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L670 EN**: Declares TableGen defm record `vmovntq`.
  **L670 CN**: 声明 TableGen defm 记录 `vmovntq`。
- **L671 EN**: Declares TableGen defm record `vmovnbq`.
  **L671 CN**: 声明 TableGen defm 记录 `vmovnbq`。
- **L672 EN**: Executes a call or declaration centered on `statement`.
  **L672 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 673-696

````tablegen

multiclass vqmovn<bit top, Type RetScalar> {
  defvar RetVector = VecOf<RetScalar>;

  let params = [s16, u16, s32, u32] in {
    def : Intrinsic<
      RetVector, (args RetVector:$inactive, Vector:$a),
      (IRInt<"vqmovn", [RetVector, Vector]>
          $inactive, $a, (unsignedflag RetScalar), (unsignedflag Scalar), top)>,
      NameOverride<NAME>;
    def: Intrinsic<
      RetVector, (args RetVector:$inactive, Vector:$a, Predicate:$pred),
      (IRInt<"vqmovn_predicated", [RetVector, Vector, Predicate]>
          $inactive, $a, (unsignedflag RetScalar), (unsignedflag Scalar),
          top, $pred)>,
      NameOverride<NAME # "_m">;
  }
}

let params = [s16, s32, u16, u32] in {
  defm vqmovntq: vqmovn<1, HalfScalar>;
  defm vqmovnbq: vqmovn<0, HalfScalar>;
}
let params = [s16, s32] in {
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Declares TableGen multiclass record `vqmovn`.
  **L674 CN**: 声明 TableGen multiclass 记录 `vqmovn`。
- **L675 EN**: Declares TableGen defvar record `RetVector = VecOf`.
  **L675 CN**: 声明 TableGen defvar 记录 `RetVector = VecOf`。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L677 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, u16, s32, u32] in {`.
  **L677 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, u16, s32, u32] in {`。
- **L678 EN**: Declares TableGen def record `def`.
  **L678 CN**: 声明 TableGen def 记录 `def`。
- **L679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RetVector, (args RetVector:$inactive, Vector:$a),`.
  **L679 CN**: 继续一个多行参数列表、初始化器或聚合项：`RetVector, (args RetVector:$inactive, Vector:$a),`。
- **L680 EN**: Continues the surrounding expression or declaration: `(IRInt<"vqmovn", [RetVector, Vector]>`.
  **L680 CN**: 继续构造周围的表达式或声明：`(IRInt<"vqmovn", [RetVector, Vector]>`。
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$inactive, $a, (unsignedflag RetScalar), (unsignedflag Scalar), top)>,`.
  **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`$inactive, $a, (unsignedflag RetScalar), (unsignedflag Scalar), top)>,`。
- **L682 EN**: Adds a standalone statement or declaration: `NameOverride<NAME>;`.
  **L682 CN**: 添加一条独立语句或声明：`NameOverride<NAME>;`。
- **L683 EN**: Declares TableGen def record `def`.
  **L683 CN**: 声明 TableGen def 记录 `def`。
- **L684 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `RetVector, (args RetVector:$inactive, Vector:$a, Predicate:$pred),`.
  **L684 CN**: 继续一个多行参数列表、初始化器或聚合项：`RetVector, (args RetVector:$inactive, Vector:$a, Predicate:$pred),`。
- **L685 EN**: Continues the surrounding expression or declaration: `(IRInt<"vqmovn_predicated", [RetVector, Vector, Predicate]>`.
  **L685 CN**: 继续构造周围的表达式或声明：`(IRInt<"vqmovn_predicated", [RetVector, Vector, Predicate]>`。
- **L686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$inactive, $a, (unsignedflag RetScalar), (unsignedflag Scalar),`.
  **L686 CN**: 继续一个多行参数列表、初始化器或聚合项：`$inactive, $a, (unsignedflag RetScalar), (unsignedflag Scalar),`。
- **L687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `top, $pred)>,`.
  **L687 CN**: 继续一个多行参数列表、初始化器或聚合项：`top, $pred)>,`。
- **L688 EN**: Adds a standalone statement or declaration: `NameOverride<NAME # "_m">;`.
  **L688 CN**: 添加一条独立语句或声明：`NameOverride<NAME # "_m">;`。
- **L689 EN**: Closes the current lexical scope or compound statement.
  **L689 CN**: 结束当前词法作用域或复合语句块。
- **L690 EN**: Closes the current lexical scope or compound statement.
  **L690 CN**: 结束当前词法作用域或复合语句块。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, s32, u16, u32] in {`.
  **L692 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, s32, u16, u32] in {`。
- **L693 EN**: Declares TableGen defm record `vqmovntq`.
  **L693 CN**: 声明 TableGen defm 记录 `vqmovntq`。
- **L694 EN**: Declares TableGen defm record `vqmovnbq`.
  **L694 CN**: 声明 TableGen defm 记录 `vqmovnbq`。
- **L695 EN**: Closes the current lexical scope or compound statement.
  **L695 CN**: 结束当前词法作用域或复合语句块。
- **L696 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, s32] in {`.
  **L696 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, s32] in {`。

### Lines 697-720

````tablegen
  defm vqmovuntq: vqmovn<1, UHalfScalar>;
  defm vqmovunbq: vqmovn<0, UHalfScalar>;
}

multiclass vrnd<IRIntBase ir_int, string suffix> {
  let params = T.Float in {
    def "": Intrinsic<Vector, (args Vector:$a),
                              (strictFPAlt<ir_int, IRInt<"vrint"#suffix, [Vector]>> $a)>;
    defm "": IntrinsicMX<Vector, (args Vector:$a, Predicate:$pred),
        (IRInt<"vrint"#suffix#"_predicated", [Vector, Predicate]>
            $a, $pred, $inactive)>;
  }
}

defm vrndq:  vrnd<IRIntBase<"trunc", [Vector]>, "z">;
defm vrndmq: vrnd<IRIntBase<"floor", [Vector]>, "m">;
defm vrndpq: vrnd<IRIntBase<"ceil",  [Vector]>, "p">;
defm vrndaq: vrnd<IRIntBase<"round", [Vector]>, "a">;
defm vrndxq: vrnd<IRIntBase<"rint",  [Vector]>, "x">;
defm vrndnq: vrnd<IRIntBase<"roundeven", [Vector]>, "n">;

multiclass compare_with_pred<string condname, dag arguments,
                             dag cmp, string suffix> {
  // Make the predicated and unpredicated versions of a single comparison.
````
- **L697 EN**: Declares TableGen defm record `vqmovuntq`.
  **L697 CN**: 声明 TableGen defm 记录 `vqmovuntq`。
- **L698 EN**: Declares TableGen defm record `vqmovunbq`.
  **L698 CN**: 声明 TableGen defm 记录 `vqmovunbq`。
- **L699 EN**: Closes the current lexical scope or compound statement.
  **L699 CN**: 结束当前词法作用域或复合语句块。
- **L700 EN**: Blank line separating nearby declarations or logic blocks.
  **L700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L701 EN**: Declares TableGen multiclass record `vrnd`.
  **L701 CN**: 声明 TableGen multiclass 记录 `vrnd`。
- **L702 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L702 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L703 EN**: Declares TableGen def record `""`.
  **L703 CN**: 声明 TableGen def 记录 `""`。
- **L704 EN**: Executes a call or declaration centered on `statement`.
  **L704 CN**: 执行以 `statement` 为核心的调用或声明。
- **L705 EN**: Declares TableGen defm record `""`.
  **L705 CN**: 声明 TableGen defm 记录 `""`。
- **L706 EN**: Continues the surrounding expression or declaration: `(IRInt<"vrint"#suffix#"_predicated", [Vector, Predicate]>`.
  **L706 CN**: 继续构造周围的表达式或声明：`(IRInt<"vrint"#suffix#"_predicated", [Vector, Predicate]>`。
- **L707 EN**: Adds a standalone statement or declaration: `$a, $pred, $inactive)>;`.
  **L707 CN**: 添加一条独立语句或声明：`$a, $pred, $inactive)>;`。
- **L708 EN**: Closes the current lexical scope or compound statement.
  **L708 CN**: 结束当前词法作用域或复合语句块。
- **L709 EN**: Closes the current lexical scope or compound statement.
  **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Blank line separating nearby declarations or logic blocks.
  **L710 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L711 EN**: Declares TableGen defm record `vrndq`.
  **L711 CN**: 声明 TableGen defm 记录 `vrndq`。
- **L712 EN**: Declares TableGen defm record `vrndmq`.
  **L712 CN**: 声明 TableGen defm 记录 `vrndmq`。
- **L713 EN**: Declares TableGen defm record `vrndpq`.
  **L713 CN**: 声明 TableGen defm 记录 `vrndpq`。
- **L714 EN**: Declares TableGen defm record `vrndaq`.
  **L714 CN**: 声明 TableGen defm 记录 `vrndaq`。
- **L715 EN**: Declares TableGen defm record `vrndxq`.
  **L715 CN**: 声明 TableGen defm 记录 `vrndxq`。
- **L716 EN**: Declares TableGen defm record `vrndnq`.
  **L716 CN**: 声明 TableGen defm 记录 `vrndnq`。
- **L717 EN**: Blank line separating nearby declarations or logic blocks.
  **L717 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L718 EN**: Declares TableGen multiclass record `compare_with_pred`.
  **L718 CN**: 声明 TableGen multiclass 记录 `compare_with_pred`。
- **L719 EN**: Continues the surrounding expression or declaration: `dag cmp, string suffix> {`.
  **L719 CN**: 继续构造周围的表达式或声明：`dag cmp, string suffix> {`。
- **L720 EN**: Comment explains nearby logic, constraints, or intent: `Make the predicated and unpredicated versions of a single comparison.`.
  **L720 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Make the predicated and unpredicated versions of a single comparison.`。

### Lines 721-744

````tablegen
  def: Intrinsic<Predicate, arguments,
                 (u16 (IRInt<"pred_v2i", [Predicate]> cmp))>,
       NameOverride<"vcmp" # condname # "q" # suffix>;
  def: Intrinsic<Predicate, !con(arguments, (args Predicate:$inpred)),
                 (u16 (IRInt<"pred_v2i", [Predicate]> (and $inpred, cmp)))>,
       NameOverride<"vcmp" # condname # "q_m" # suffix>;
}

multiclass compare<string condname, Builder cmpop> {
  // Make all four variants of a comparison: the vector/vector and
  // vector/scalar forms, each using compare_with_pred to make a
  // predicated and unpredicated version.
  defm: compare_with_pred<condname, (args Vector:$va, Vector:$vb),
                          (cmpop $va, $vb), "">;
  let pnt = PNT_NType in {
    defm: compare_with_pred<condname, (args Vector:$va, unpromoted<Scalar>:$sb),
                            (cmpop $va, (splat $sb)), "_n">;
  }
}
let params = T.Int in {
  defm: compare<"eq", icmp_eq>;
  defm: compare<"ne", icmp_ne>;
}
let params = T.Signed in {
````
- **L721 EN**: Declares TableGen def record `def`.
  **L721 CN**: 声明 TableGen def 记录 `def`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(u16 (IRInt<"pred_v2i", [Predicate]> cmp))>,`.
  **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`(u16 (IRInt<"pred_v2i", [Predicate]> cmp))>,`。
- **L723 EN**: Adds a standalone statement or declaration: `NameOverride<"vcmp" # condname # "q" # suffix>;`.
  **L723 CN**: 添加一条独立语句或声明：`NameOverride<"vcmp" # condname # "q" # suffix>;`。
- **L724 EN**: Declares TableGen def record `def`.
  **L724 CN**: 声明 TableGen def 记录 `def`。
- **L725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(u16 (IRInt<"pred_v2i", [Predicate]> (and $inpred, cmp)))>,`.
  **L725 CN**: 继续一个多行参数列表、初始化器或聚合项：`(u16 (IRInt<"pred_v2i", [Predicate]> (and $inpred, cmp)))>,`。
- **L726 EN**: Adds a standalone statement or declaration: `NameOverride<"vcmp" # condname # "q_m" # suffix>;`.
  **L726 CN**: 添加一条独立语句或声明：`NameOverride<"vcmp" # condname # "q_m" # suffix>;`。
- **L727 EN**: Closes the current lexical scope or compound statement.
  **L727 CN**: 结束当前词法作用域或复合语句块。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L729 EN**: Declares TableGen multiclass record `compare`.
  **L729 CN**: 声明 TableGen multiclass 记录 `compare`。
- **L730 EN**: Comment explains nearby logic, constraints, or intent: `Make all four variants of a comparison: the vector/vector and`.
  **L730 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Make all four variants of a comparison: the vector/vector and`。
- **L731 EN**: Comment explains nearby logic, constraints, or intent: `vector/scalar forms, each using compare_with_pred to make a`.
  **L731 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vector/scalar forms, each using compare_with_pred to make a`。
- **L732 EN**: Comment explains nearby logic, constraints, or intent: `predicated and unpredicated version.`.
  **L732 CN**: 注释解释附近代码的逻辑、约束或设计意图：`predicated and unpredicated version.`。
- **L733 EN**: Declares TableGen defm record `defm`.
  **L733 CN**: 声明 TableGen defm 记录 `defm`。
- **L734 EN**: Executes a call or declaration centered on `statement`.
  **L734 CN**: 执行以 `statement` 为核心的调用或声明。
- **L735 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L735 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L736 EN**: Declares TableGen defm record `defm`.
  **L736 CN**: 声明 TableGen defm 记录 `defm`。
- **L737 EN**: Executes a call or declaration centered on `statement`.
  **L737 CN**: 执行以 `statement` 为核心的调用或声明。
- **L738 EN**: Closes the current lexical scope or compound statement.
  **L738 CN**: 结束当前词法作用域或复合语句块。
- **L739 EN**: Closes the current lexical scope or compound statement.
  **L739 CN**: 结束当前词法作用域或复合语句块。
- **L740 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L740 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L741 EN**: Declares TableGen defm record `defm`.
  **L741 CN**: 声明 TableGen defm 记录 `defm`。
- **L742 EN**: Declares TableGen defm record `defm`.
  **L742 CN**: 声明 TableGen defm 记录 `defm`。
- **L743 EN**: Closes the current lexical scope or compound statement.
  **L743 CN**: 结束当前词法作用域或复合语句块。
- **L744 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L744 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。

### Lines 745-768

````tablegen
  defm: compare<"gt", icmp_sgt>;
  defm: compare<"ge", icmp_sge>;
  defm: compare<"lt", icmp_slt>;
  defm: compare<"le", icmp_sle>;
}
let params = T.Unsigned in {
  defm: compare<"hi", icmp_ugt>;
  defm: compare<"cs", icmp_uge>;
}
let params = T.Float in {
  defm: compare<"eq", fcmp_eq>;
  defm: compare<"ne", fcmp_ne>;
  defm: compare<"gt", fcmp_gt>;
  defm: compare<"ge", fcmp_ge>;
  defm: compare<"lt", fcmp_ult>;
  defm: compare<"le", fcmp_ule>;
}

let params = T.Signed in {
  def vminq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                               (select (icmp_sle $a, $b), $a, $b)>;
  def vmaxq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                               (select (icmp_sge $a, $b), $a, $b)>;
  def vminaq: Intrinsic<UVector, (args UVector:$a, Vector:$b),
````
- **L745 EN**: Declares TableGen defm record `defm`.
  **L745 CN**: 声明 TableGen defm 记录 `defm`。
- **L746 EN**: Declares TableGen defm record `defm`.
  **L746 CN**: 声明 TableGen defm 记录 `defm`。
- **L747 EN**: Declares TableGen defm record `defm`.
  **L747 CN**: 声明 TableGen defm 记录 `defm`。
- **L748 EN**: Declares TableGen defm record `defm`.
  **L748 CN**: 声明 TableGen defm 记录 `defm`。
- **L749 EN**: Closes the current lexical scope or compound statement.
  **L749 CN**: 结束当前词法作用域或复合语句块。
- **L750 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Unsigned in {`.
  **L750 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Unsigned in {`。
- **L751 EN**: Declares TableGen defm record `defm`.
  **L751 CN**: 声明 TableGen defm 记录 `defm`。
- **L752 EN**: Declares TableGen defm record `defm`.
  **L752 CN**: 声明 TableGen defm 记录 `defm`。
- **L753 EN**: Closes the current lexical scope or compound statement.
  **L753 CN**: 结束当前词法作用域或复合语句块。
- **L754 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L754 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L755 EN**: Declares TableGen defm record `defm`.
  **L755 CN**: 声明 TableGen defm 记录 `defm`。
- **L756 EN**: Declares TableGen defm record `defm`.
  **L756 CN**: 声明 TableGen defm 记录 `defm`。
- **L757 EN**: Declares TableGen defm record `defm`.
  **L757 CN**: 声明 TableGen defm 记录 `defm`。
- **L758 EN**: Declares TableGen defm record `defm`.
  **L758 CN**: 声明 TableGen defm 记录 `defm`。
- **L759 EN**: Declares TableGen defm record `defm`.
  **L759 CN**: 声明 TableGen defm 记录 `defm`。
- **L760 EN**: Declares TableGen defm record `defm`.
  **L760 CN**: 声明 TableGen defm 记录 `defm`。
- **L761 EN**: Closes the current lexical scope or compound statement.
  **L761 CN**: 结束当前词法作用域或复合语句块。
- **L762 EN**: Blank line separating nearby declarations or logic blocks.
  **L762 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L763 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L763 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L764 EN**: Declares TableGen def record `vminq`.
  **L764 CN**: 声明 TableGen def 记录 `vminq`。
- **L765 EN**: Executes a call or declaration centered on `statement`.
  **L765 CN**: 执行以 `statement` 为核心的调用或声明。
- **L766 EN**: Declares TableGen def record `vmaxq`.
  **L766 CN**: 声明 TableGen def 记录 `vmaxq`。
- **L767 EN**: Executes a call or declaration centered on `statement`.
  **L767 CN**: 执行以 `statement` 为核心的调用或声明。
- **L768 EN**: Declares TableGen def record `vminaq`.
  **L768 CN**: 声明 TableGen def 记录 `vminaq`。

### Lines 769-792

````tablegen
                                 (seq (select (icmp_slt $b, (zeroinit Vector)),
                                              (sub (zeroinit Vector), $b), $b):$absb,
                                      (select (icmp_ule $a, $absb), $a, $absb))>;
  def vmaxaq: Intrinsic<UVector, (args UVector:$a, Vector:$b),
                                 (seq (select (icmp_slt $b, (zeroinit Vector)),
                                              (sub (zeroinit Vector), $b), $b):$absb,
                                      (select (icmp_uge $a, $absb), $a, $absb))>;
}
let params = T.Unsigned in {
  def vminqu: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                (select (icmp_ule $a, $b), $a, $b)>,
              NameOverride<"vminq">;
  def vmaxqu: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                (select (icmp_uge $a, $b), $a, $b)>,
              NameOverride<"vmaxq">;
}
let params = T.Float in {
  def vminnmq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                 (fminnm $a, $b)>;
  def vmaxnmq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                 (fmaxnm $a, $b)>;
  def vminnmaq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                  (fminnm
                                   (IRIntBase<"fabs", [Vector]> $a),
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (select (icmp_slt $b, (zeroinit Vector)),`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (select (icmp_slt $b, (zeroinit Vector)),`。
- **L770 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(sub (zeroinit Vector), $b), $b):$absb,`.
  **L770 CN**: 继续一个多行参数列表、初始化器或聚合项：`(sub (zeroinit Vector), $b), $b):$absb,`。
- **L771 EN**: Executes a call or declaration centered on `statement`.
  **L771 CN**: 执行以 `statement` 为核心的调用或声明。
- **L772 EN**: Declares TableGen def record `vmaxaq`.
  **L772 CN**: 声明 TableGen def 记录 `vmaxaq`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (select (icmp_slt $b, (zeroinit Vector)),`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (select (icmp_slt $b, (zeroinit Vector)),`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(sub (zeroinit Vector), $b), $b):$absb,`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`(sub (zeroinit Vector), $b), $b):$absb,`。
- **L775 EN**: Executes a call or declaration centered on `statement`.
  **L775 CN**: 执行以 `statement` 为核心的调用或声明。
- **L776 EN**: Closes the current lexical scope or compound statement.
  **L776 CN**: 结束当前词法作用域或复合语句块。
- **L777 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Unsigned in {`.
  **L777 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Unsigned in {`。
- **L778 EN**: Declares TableGen def record `vminqu`.
  **L778 CN**: 声明 TableGen def 记录 `vminqu`。
- **L779 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(select (icmp_ule $a, $b), $a, $b)>,`.
  **L779 CN**: 继续一个多行参数列表、初始化器或聚合项：`(select (icmp_ule $a, $b), $a, $b)>,`。
- **L780 EN**: Adds a standalone statement or declaration: `NameOverride<"vminq">;`.
  **L780 CN**: 添加一条独立语句或声明：`NameOverride<"vminq">;`。
- **L781 EN**: Declares TableGen def record `vmaxqu`.
  **L781 CN**: 声明 TableGen def 记录 `vmaxqu`。
- **L782 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(select (icmp_uge $a, $b), $a, $b)>,`.
  **L782 CN**: 继续一个多行参数列表、初始化器或聚合项：`(select (icmp_uge $a, $b), $a, $b)>,`。
- **L783 EN**: Adds a standalone statement or declaration: `NameOverride<"vmaxq">;`.
  **L783 CN**: 添加一条独立语句或声明：`NameOverride<"vmaxq">;`。
- **L784 EN**: Closes the current lexical scope or compound statement.
  **L784 CN**: 结束当前词法作用域或复合语句块。
- **L785 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L785 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L786 EN**: Declares TableGen def record `vminnmq`.
  **L786 CN**: 声明 TableGen def 记录 `vminnmq`。
- **L787 EN**: Executes a call or declaration centered on `statement`.
  **L787 CN**: 执行以 `statement` 为核心的调用或声明。
- **L788 EN**: Declares TableGen def record `vmaxnmq`.
  **L788 CN**: 声明 TableGen def 记录 `vmaxnmq`。
- **L789 EN**: Executes a call or declaration centered on `statement`.
  **L789 CN**: 执行以 `statement` 为核心的调用或声明。
- **L790 EN**: Declares TableGen def record `vminnmaq`.
  **L790 CN**: 声明 TableGen def 记录 `vminnmaq`。
- **L791 EN**: Continues the surrounding expression or declaration: `(fminnm`.
  **L791 CN**: 继续构造周围的表达式或声明：`(fminnm`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRIntBase<"fabs", [Vector]> $a),`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRIntBase<"fabs", [Vector]> $a),`。

### Lines 793-816

````tablegen
                                   (IRIntBase<"fabs", [Vector]> $b))>;
  def vmaxnmaq: Intrinsic<Vector, (args Vector:$a, Vector:$b),
                                  (fmaxnm
                                   (IRIntBase<"fabs", [Vector]> $a),
                                   (IRIntBase<"fabs", [Vector]> $b))>;
}

def vpselq: Intrinsic<Vector, (args Vector:$t, Vector:$f, Predicate:$pred),
                      (select $pred, $t, $f)> { let params = T.Usual; }
def vpselq_64: Intrinsic<
    Vector, (args Vector:$t, Vector:$f, PredOf<u32>:$pred),
            (bitcast (select $pred, (bitcast $t, VecOf<u32>),
                                    (bitcast $f, VecOf<u32>)), Vector)>,
    NameOverride<"vpselq"> { let params = T.All64; }

let params = [Void], pnt = PNT_None in {

  multiclass vctp<Type pred, string intname> {
    def "": Intrinsic<pred, (args u32:$val),
        (u16 (IRInt<"pred_v2i", [pred]> (IRIntBase<intname> $val)))>;
    def _m: Intrinsic<pred, (args u32:$val, pred:$inpred),
        (u16 (IRInt<"pred_v2i", [pred]> (and $inpred,
                                         (IRIntBase<intname> $val))))>;
  }
````
- **L793 EN**: Executes a call or declaration centered on `statement`.
  **L793 CN**: 执行以 `statement` 为核心的调用或声明。
- **L794 EN**: Declares TableGen def record `vmaxnmaq`.
  **L794 CN**: 声明 TableGen def 记录 `vmaxnmaq`。
- **L795 EN**: Continues the surrounding expression or declaration: `(fmaxnm`.
  **L795 CN**: 继续构造周围的表达式或声明：`(fmaxnm`。
- **L796 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRIntBase<"fabs", [Vector]> $a),`.
  **L796 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRIntBase<"fabs", [Vector]> $a),`。
- **L797 EN**: Executes a call or declaration centered on `statement`.
  **L797 CN**: 执行以 `statement` 为核心的调用或声明。
- **L798 EN**: Closes the current lexical scope or compound statement.
  **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Blank line separating nearby declarations or logic blocks.
  **L799 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L800 EN**: Declares TableGen def record `vpselq`.
  **L800 CN**: 声明 TableGen def 记录 `vpselq`。
- **L801 EN**: Continues the surrounding expression or declaration: `(select $pred, $t, $f)> { let params = T.Usual; }`.
  **L801 CN**: 继续构造周围的表达式或声明：`(select $pred, $t, $f)> { let params = T.Usual; }`。
- **L802 EN**: Declares TableGen def record `vpselq_64`.
  **L802 CN**: 声明 TableGen def 记录 `vpselq_64`。
- **L803 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$t, Vector:$f, PredOf<u32>:$pred),`.
  **L803 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$t, Vector:$f, PredOf<u32>:$pred),`。
- **L804 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast (select $pred, (bitcast $t, VecOf<u32>),`.
  **L804 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast (select $pred, (bitcast $t, VecOf<u32>),`。
- **L805 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(bitcast $f, VecOf<u32>)), Vector)>,`.
  **L805 CN**: 继续一个多行参数列表、初始化器或聚合项：`(bitcast $f, VecOf<u32>)), Vector)>,`。
- **L806 EN**: Continues the surrounding expression or declaration: `NameOverride<"vpselq"> { let params = T.All64; }`.
  **L806 CN**: 继续构造周围的表达式或声明：`NameOverride<"vpselq"> { let params = T.All64; }`。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [Void], pnt = PNT_None in {`.
  **L808 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [Void], pnt = PNT_None in {`。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L810 EN**: Declares TableGen multiclass record `vctp`.
  **L810 CN**: 声明 TableGen multiclass 记录 `vctp`。
- **L811 EN**: Declares TableGen def record `""`.
  **L811 CN**: 声明 TableGen def 记录 `""`。
- **L812 EN**: Executes a call or declaration centered on `statement`.
  **L812 CN**: 执行以 `statement` 为核心的调用或声明。
- **L813 EN**: Declares TableGen def record `_m`.
  **L813 CN**: 声明 TableGen def 记录 `_m`。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(u16 (IRInt<"pred_v2i", [pred]> (and $inpred,`.
  **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`(u16 (IRInt<"pred_v2i", [pred]> (and $inpred,`。
- **L815 EN**: Executes a call or declaration centered on `statement`.
  **L815 CN**: 执行以 `statement` 为核心的调用或声明。
- **L816 EN**: Closes the current lexical scope or compound statement.
  **L816 CN**: 结束当前词法作用域或复合语句块。

### Lines 817-840

````tablegen
  defm vctp8q:  vctp<PredOf<u8>,  "arm_mve_vctp8">;
  defm vctp16q: vctp<PredOf<u16>, "arm_mve_vctp16">;
  defm vctp32q: vctp<PredOf<u32>, "arm_mve_vctp32">;
  defm vctp64q: vctp<PredOf<u64>, "arm_mve_vctp64">;

  def vpnot: Intrinsic<PredOf<u8>, (args unpromoted<PredOf<u8>>:$pred),
                       (xor $pred, (u16 65535))>;

}

multiclass contiguous_load<string mnemonic, PrimitiveType memtype,
                           list<Type> same_size, list<Type> wider> {
  // Intrinsics named with explicit memory and element sizes that match:
  // vldrbq_?8, vldrhq_?16, vldrwq_?32.
  let params = same_size, pnt = PNT_None in {
    def: Intrinsic<Vector, (args CPtr<CopyKind<same_size[0], Scalar>>:$addr),
                   (load (address (CPtr<Vector> $addr), !srl(memtype.size,3)))>,
         NameOverride<mnemonic>;
    def: Intrinsic<Vector, (args CPtr<CopyKind<same_size[0], Scalar>>:$addr,
                                 Predicate:$pred),
                   (masked_load Vector, (CPtr<Vector> $addr),
                       !srl(memtype.size,3), $pred, (zeroinit Vector))>,
         NameOverride<mnemonic # "_z">;
  }
````
- **L817 EN**: Declares TableGen defm record `vctp8q`.
  **L817 CN**: 声明 TableGen defm 记录 `vctp8q`。
- **L818 EN**: Declares TableGen defm record `vctp16q`.
  **L818 CN**: 声明 TableGen defm 记录 `vctp16q`。
- **L819 EN**: Declares TableGen defm record `vctp32q`.
  **L819 CN**: 声明 TableGen defm 记录 `vctp32q`。
- **L820 EN**: Declares TableGen defm record `vctp64q`.
  **L820 CN**: 声明 TableGen defm 记录 `vctp64q`。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L822 EN**: Declares TableGen def record `vpnot`.
  **L822 CN**: 声明 TableGen def 记录 `vpnot`。
- **L823 EN**: Executes a call or declaration centered on `statement`.
  **L823 CN**: 执行以 `statement` 为核心的调用或声明。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L825 EN**: Closes the current lexical scope or compound statement.
  **L825 CN**: 结束当前词法作用域或复合语句块。
- **L826 EN**: Blank line separating nearby declarations or logic blocks.
  **L826 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L827 EN**: Declares TableGen multiclass record `contiguous_load`.
  **L827 CN**: 声明 TableGen multiclass 记录 `contiguous_load`。
- **L828 EN**: Continues the surrounding expression or declaration: `list<Type> same_size, list<Type> wider> {`.
  **L828 CN**: 继续构造周围的表达式或声明：`list<Type> same_size, list<Type> wider> {`。
- **L829 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsics named with explicit memory and element sizes that match:`.
  **L829 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsics named with explicit memory and element sizes that match:`。
- **L830 EN**: Comment explains nearby logic, constraints, or intent: `vldrbq_?8, vldrhq_?16, vldrwq_?32.`.
  **L830 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vldrbq_?8, vldrhq_?16, vldrwq_?32.`。
- **L831 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = same_size, pnt = PNT_None in {`.
  **L831 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = same_size, pnt = PNT_None in {`。
- **L832 EN**: Declares TableGen def record `def`.
  **L832 CN**: 声明 TableGen def 记录 `def`。
- **L833 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(load (address (CPtr<Vector> $addr), !srl(memtype.size,3)))>,`.
  **L833 CN**: 继续一个多行参数列表、初始化器或聚合项：`(load (address (CPtr<Vector> $addr), !srl(memtype.size,3)))>,`。
- **L834 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic>;`.
  **L834 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic>;`。
- **L835 EN**: Declares TableGen def record `def`.
  **L835 CN**: 声明 TableGen def 记录 `def`。
- **L836 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L836 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(masked_load Vector, (CPtr<Vector> $addr),`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`(masked_load Vector, (CPtr<Vector> $addr),`。
- **L838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!srl(memtype.size,3), $pred, (zeroinit Vector))>,`.
  **L838 CN**: 继续一个多行参数列表、初始化器或聚合项：`!srl(memtype.size,3), $pred, (zeroinit Vector))>,`。
- **L839 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic # "_z">;`.
  **L839 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic # "_z">;`。
- **L840 EN**: Closes the current lexical scope or compound statement.
  **L840 CN**: 结束当前词法作用域或复合语句块。

### Lines 841-864

````tablegen

  // Synonyms for the above, with the generic name vld1q that just means
  // 'memory and element sizes match', and allows convenient polymorphism with
  // the memory and element types covariant.
  let params = same_size in {
    def: Intrinsic<Vector, (args CPtr<CopyKind<same_size[0], Scalar>>:$addr),
                   (load (address (CPtr<Vector> $addr), !srl(memtype.size,3)))>,
         NameOverride<"vld1q">;
    def: Intrinsic<Vector, (args CPtr<CopyKind<same_size[0], Scalar>>:$addr,
                                 Predicate:$pred),
                   (masked_load Vector, (CPtr<Vector> $addr),
                        !srl(memtype.size,3), $pred, (zeroinit Vector))>,
         NameOverride<"vld1q_z">;
  }

  // Intrinsics with the memory size narrower than the vector element, so that
  // they load less than 128 bits of memory and sign/zero extend each loaded
  // value into a wider vector lane.
  let params = wider, pnt = PNT_None in {
    def: Intrinsic<Vector, (args CPtr<CopyKind<same_size[0], Scalar>>:$addr),
                   (extend (load (address (CPtr<NarrowedVecOf<memtype,Vector>>
                                           $addr), !srl(memtype.size,3))),
                           Vector, (unsignedflag Scalar))>,
         NameOverride<mnemonic>;
````
- **L841 EN**: Blank line separating nearby declarations or logic blocks.
  **L841 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L842 EN**: Comment explains nearby logic, constraints, or intent: `Synonyms for the above, with the generic name vld1q that just means`.
  **L842 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Synonyms for the above, with the generic name vld1q that just means`。
- **L843 EN**: Comment explains nearby logic, constraints, or intent: `'memory and element sizes match', and allows convenient polymorphism with`.
  **L843 CN**: 注释解释附近代码的逻辑、约束或设计意图：`'memory and element sizes match', and allows convenient polymorphism with`。
- **L844 EN**: Comment explains nearby logic, constraints, or intent: `the memory and element types covariant.`.
  **L844 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the memory and element types covariant.`。
- **L845 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = same_size in {`.
  **L845 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = same_size in {`。
- **L846 EN**: Declares TableGen def record `def`.
  **L846 CN**: 声明 TableGen def 记录 `def`。
- **L847 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(load (address (CPtr<Vector> $addr), !srl(memtype.size,3)))>,`.
  **L847 CN**: 继续一个多行参数列表、初始化器或聚合项：`(load (address (CPtr<Vector> $addr), !srl(memtype.size,3)))>,`。
- **L848 EN**: Adds a standalone statement or declaration: `NameOverride<"vld1q">;`.
  **L848 CN**: 添加一条独立语句或声明：`NameOverride<"vld1q">;`。
- **L849 EN**: Declares TableGen def record `def`.
  **L849 CN**: 声明 TableGen def 记录 `def`。
- **L850 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L850 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L851 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(masked_load Vector, (CPtr<Vector> $addr),`.
  **L851 CN**: 继续一个多行参数列表、初始化器或聚合项：`(masked_load Vector, (CPtr<Vector> $addr),`。
- **L852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!srl(memtype.size,3), $pred, (zeroinit Vector))>,`.
  **L852 CN**: 继续一个多行参数列表、初始化器或聚合项：`!srl(memtype.size,3), $pred, (zeroinit Vector))>,`。
- **L853 EN**: Adds a standalone statement or declaration: `NameOverride<"vld1q_z">;`.
  **L853 CN**: 添加一条独立语句或声明：`NameOverride<"vld1q_z">;`。
- **L854 EN**: Closes the current lexical scope or compound statement.
  **L854 CN**: 结束当前词法作用域或复合语句块。
- **L855 EN**: Blank line separating nearby declarations or logic blocks.
  **L855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L856 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsics with the memory size narrower than the vector element, so that`.
  **L856 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsics with the memory size narrower than the vector element, so that`。
- **L857 EN**: Comment explains nearby logic, constraints, or intent: `they load less than 128 bits of memory and sign/zero extend each loaded`.
  **L857 CN**: 注释解释附近代码的逻辑、约束或设计意图：`they load less than 128 bits of memory and sign/zero extend each loaded`。
- **L858 EN**: Comment explains nearby logic, constraints, or intent: `value into a wider vector lane.`.
  **L858 CN**: 注释解释附近代码的逻辑、约束或设计意图：`value into a wider vector lane.`。
- **L859 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = wider, pnt = PNT_None in {`.
  **L859 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = wider, pnt = PNT_None in {`。
- **L860 EN**: Declares TableGen def record `def`.
  **L860 CN**: 声明 TableGen def 记录 `def`。
- **L861 EN**: Continues logic associated with callable symbol `extend`.
  **L861 CN**: 继续与可调用符号 `extend` 相关的逻辑。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$addr), !srl(memtype.size,3))),`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`$addr), !srl(memtype.size,3))),`。
- **L863 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (unsignedflag Scalar))>,`.
  **L863 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (unsignedflag Scalar))>,`。
- **L864 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic>;`.
  **L864 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic>;`。

### Lines 865-888

````tablegen
    def: Intrinsic<Vector, (args CPtr<CopyKind<same_size[0], Scalar>>:$addr,
                                 Predicate:$pred),
                   (extend (masked_load NarrowedVecOf<memtype,Vector>,
                                (CPtr<NarrowedVecOf<memtype,Vector>> $addr),
                                !srl(memtype.size,3), $pred,
                                (zeroinit NarrowedVecOf<memtype,Vector>)),
                           Vector, (unsignedflag Scalar))>,
         NameOverride<mnemonic # "_z">;
  }
}

defm: contiguous_load<"vldrbq", u8, T.All8, !listconcat(T.Int16, T.Int32)>;
defm: contiguous_load<"vldrhq", u16, T.All16, T.Int32>;
defm: contiguous_load<"vldrwq", u32, T.All32, []>;

multiclass contiguous_store<string mnemonic, PrimitiveType memtype,
                           list<Type> same_size, list<Type> wider> {
  // Intrinsics named with explicit memory and element sizes that match:
  // vstrbq_?8, vstrhq_?16, vstrwq_?32.
  let params = same_size in {
    def: Intrinsic<Void, (args Ptr<CopyKind<same_size[0], Scalar>>:$addr,
                               Vector:$value),
                   (store $value,
                          (address (Ptr<Vector> $addr), !srl(memtype.size,3)))>,
````
- **L865 EN**: Declares TableGen def record `def`.
  **L865 CN**: 声明 TableGen def 记录 `def`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(extend (masked_load NarrowedVecOf<memtype,Vector>,`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`(extend (masked_load NarrowedVecOf<memtype,Vector>,`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(CPtr<NarrowedVecOf<memtype,Vector>> $addr),`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`(CPtr<NarrowedVecOf<memtype,Vector>> $addr),`。
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!srl(memtype.size,3), $pred,`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`!srl(memtype.size,3), $pred,`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(zeroinit NarrowedVecOf<memtype,Vector>)),`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`(zeroinit NarrowedVecOf<memtype,Vector>)),`。
- **L871 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (unsignedflag Scalar))>,`.
  **L871 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (unsignedflag Scalar))>,`。
- **L872 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic # "_z">;`.
  **L872 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic # "_z">;`。
- **L873 EN**: Closes the current lexical scope or compound statement.
  **L873 CN**: 结束当前词法作用域或复合语句块。
- **L874 EN**: Closes the current lexical scope or compound statement.
  **L874 CN**: 结束当前词法作用域或复合语句块。
- **L875 EN**: Blank line separating nearby declarations or logic blocks.
  **L875 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L876 EN**: Declares TableGen defm record `defm`.
  **L876 CN**: 声明 TableGen defm 记录 `defm`。
- **L877 EN**: Declares TableGen defm record `defm`.
  **L877 CN**: 声明 TableGen defm 记录 `defm`。
- **L878 EN**: Declares TableGen defm record `defm`.
  **L878 CN**: 声明 TableGen defm 记录 `defm`。
- **L879 EN**: Blank line separating nearby declarations or logic blocks.
  **L879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L880 EN**: Declares TableGen multiclass record `contiguous_store`.
  **L880 CN**: 声明 TableGen multiclass 记录 `contiguous_store`。
- **L881 EN**: Continues the surrounding expression or declaration: `list<Type> same_size, list<Type> wider> {`.
  **L881 CN**: 继续构造周围的表达式或声明：`list<Type> same_size, list<Type> wider> {`。
- **L882 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsics named with explicit memory and element sizes that match:`.
  **L882 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsics named with explicit memory and element sizes that match:`。
- **L883 EN**: Comment explains nearby logic, constraints, or intent: `vstrbq_?8, vstrhq_?16, vstrwq_?32.`.
  **L883 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vstrbq_?8, vstrhq_?16, vstrwq_?32.`。
- **L884 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = same_size in {`.
  **L884 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = same_size in {`。
- **L885 EN**: Declares TableGen def record `def`.
  **L885 CN**: 声明 TableGen def 记录 `def`。
- **L886 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$value),`.
  **L886 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$value),`。
- **L887 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store $value,`.
  **L887 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store $value,`。
- **L888 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(address (Ptr<Vector> $addr), !srl(memtype.size,3)))>,`.
  **L888 CN**: 继续一个多行参数列表、初始化器或聚合项：`(address (Ptr<Vector> $addr), !srl(memtype.size,3)))>,`。

### Lines 889-912

````tablegen
         NameOverride<mnemonic>;
    def: Intrinsic<Void, (args Ptr<CopyKind<same_size[0], Scalar>>:$addr,
                               Vector:$value, Predicate:$pred),
                   (masked_store $value, (Ptr<Vector> $addr),
                        !srl(memtype.size,3), $pred)>,
         NameOverride<mnemonic # "_p">;
  }

  // Synonyms for the above, with the generic name vst1q that just means
  // 'memory and element sizes match', and allows convenient polymorphism with
  // the memory and element types covariant.
  let params = same_size in {
    def: Intrinsic<Void, (args Ptr<CopyKind<same_size[0], Scalar>>:$addr,
                               Vector:$value),
                   (store $value,
                          (address (Ptr<Vector> $addr), !srl(memtype.size,3)))>,
         NameOverride<"vst1q">;
    def: Intrinsic<Void, (args Ptr<CopyKind<same_size[0], Scalar>>:$addr,
                               Vector:$value, Predicate:$pred),
                   (masked_store $value, (Ptr<Vector> $addr),
                        !srl(memtype.size,3), $pred)>,
         NameOverride<"vst1q_p">;
  }

````
- **L889 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic>;`.
  **L889 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic>;`。
- **L890 EN**: Declares TableGen def record `def`.
  **L890 CN**: 声明 TableGen def 记录 `def`。
- **L891 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$value, Predicate:$pred),`.
  **L891 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$value, Predicate:$pred),`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(masked_store $value, (Ptr<Vector> $addr),`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`(masked_store $value, (Ptr<Vector> $addr),`。
- **L893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!srl(memtype.size,3), $pred)>,`.
  **L893 CN**: 继续一个多行参数列表、初始化器或聚合项：`!srl(memtype.size,3), $pred)>,`。
- **L894 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic # "_p">;`.
  **L894 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic # "_p">;`。
- **L895 EN**: Closes the current lexical scope or compound statement.
  **L895 CN**: 结束当前词法作用域或复合语句块。
- **L896 EN**: Blank line separating nearby declarations or logic blocks.
  **L896 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L897 EN**: Comment explains nearby logic, constraints, or intent: `Synonyms for the above, with the generic name vst1q that just means`.
  **L897 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Synonyms for the above, with the generic name vst1q that just means`。
- **L898 EN**: Comment explains nearby logic, constraints, or intent: `'memory and element sizes match', and allows convenient polymorphism with`.
  **L898 CN**: 注释解释附近代码的逻辑、约束或设计意图：`'memory and element sizes match', and allows convenient polymorphism with`。
- **L899 EN**: Comment explains nearby logic, constraints, or intent: `the memory and element types covariant.`.
  **L899 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the memory and element types covariant.`。
- **L900 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = same_size in {`.
  **L900 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = same_size in {`。
- **L901 EN**: Declares TableGen def record `def`.
  **L901 CN**: 声明 TableGen def 记录 `def`。
- **L902 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$value),`.
  **L902 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$value),`。
- **L903 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store $value,`.
  **L903 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store $value,`。
- **L904 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(address (Ptr<Vector> $addr), !srl(memtype.size,3)))>,`.
  **L904 CN**: 继续一个多行参数列表、初始化器或聚合项：`(address (Ptr<Vector> $addr), !srl(memtype.size,3)))>,`。
- **L905 EN**: Adds a standalone statement or declaration: `NameOverride<"vst1q">;`.
  **L905 CN**: 添加一条独立语句或声明：`NameOverride<"vst1q">;`。
- **L906 EN**: Declares TableGen def record `def`.
  **L906 CN**: 声明 TableGen def 记录 `def`。
- **L907 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$value, Predicate:$pred),`.
  **L907 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$value, Predicate:$pred),`。
- **L908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(masked_store $value, (Ptr<Vector> $addr),`.
  **L908 CN**: 继续一个多行参数列表、初始化器或聚合项：`(masked_store $value, (Ptr<Vector> $addr),`。
- **L909 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!srl(memtype.size,3), $pred)>,`.
  **L909 CN**: 继续一个多行参数列表、初始化器或聚合项：`!srl(memtype.size,3), $pred)>,`。
- **L910 EN**: Adds a standalone statement or declaration: `NameOverride<"vst1q_p">;`.
  **L910 CN**: 添加一条独立语句或声明：`NameOverride<"vst1q_p">;`。
- **L911 EN**: Closes the current lexical scope or compound statement.
  **L911 CN**: 结束当前词法作用域或复合语句块。
- **L912 EN**: Blank line separating nearby declarations or logic blocks.
  **L912 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 913-936

````tablegen
  // Intrinsics with the memory size narrower than the vector element, so that
  // they store less than 128 bits of memory, truncating each vector lane into
  // a narrower value to store.
  let params = wider in {
    def: Intrinsic<Void, (args Ptr<CopyKind<same_size[0], Scalar>>:$addr,
                               Vector:$value),
                   (store (trunc $value, NarrowedVecOf<memtype,Vector>),
                          (address (Ptr<NarrowedVecOf<memtype,Vector>> $addr),
                                   !srl(memtype.size,3)))>,
         NameOverride<mnemonic>;
    def: Intrinsic<Void, (args Ptr<CopyKind<same_size[0], Scalar>>:$addr,
                               Vector:$value, Predicate:$pred),
                   (masked_store
                        (trunc $value, NarrowedVecOf<memtype,Vector>),
                        (Ptr<NarrowedVecOf<memtype,Vector>> $addr),
                        !srl(memtype.size,3), $pred)>,
         NameOverride<mnemonic # "_p">;
  }
}

defm: contiguous_store<"vstrbq", u8, T.All8, !listconcat(T.Int16, T.Int32)>;
defm: contiguous_store<"vstrhq", u16, T.All16, T.Int32>;
defm: contiguous_store<"vstrwq", u32, T.All32, []>;

````
- **L913 EN**: Comment explains nearby logic, constraints, or intent: `Intrinsics with the memory size narrower than the vector element, so that`.
  **L913 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Intrinsics with the memory size narrower than the vector element, so that`。
- **L914 EN**: Comment explains nearby logic, constraints, or intent: `they store less than 128 bits of memory, truncating each vector lane into`.
  **L914 CN**: 注释解释附近代码的逻辑、约束或设计意图：`they store less than 128 bits of memory, truncating each vector lane into`。
- **L915 EN**: Comment explains nearby logic, constraints, or intent: `a narrower value to store.`.
  **L915 CN**: 注释解释附近代码的逻辑、约束或设计意图：`a narrower value to store.`。
- **L916 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = wider in {`.
  **L916 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = wider in {`。
- **L917 EN**: Declares TableGen def record `def`.
  **L917 CN**: 声明 TableGen def 记录 `def`。
- **L918 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$value),`.
  **L918 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$value),`。
- **L919 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (trunc $value, NarrowedVecOf<memtype,Vector>),`.
  **L919 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (trunc $value, NarrowedVecOf<memtype,Vector>),`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(address (Ptr<NarrowedVecOf<memtype,Vector>> $addr),`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`(address (Ptr<NarrowedVecOf<memtype,Vector>> $addr),`。
- **L921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!srl(memtype.size,3)))>,`.
  **L921 CN**: 继续一个多行参数列表、初始化器或聚合项：`!srl(memtype.size,3)))>,`。
- **L922 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic>;`.
  **L922 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic>;`。
- **L923 EN**: Declares TableGen def record `def`.
  **L923 CN**: 声明 TableGen def 记录 `def`。
- **L924 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$value, Predicate:$pred),`.
  **L924 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$value, Predicate:$pred),`。
- **L925 EN**: Continues the surrounding expression or declaration: `(masked_store`.
  **L925 CN**: 继续构造周围的表达式或声明：`(masked_store`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(trunc $value, NarrowedVecOf<memtype,Vector>),`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`(trunc $value, NarrowedVecOf<memtype,Vector>),`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(Ptr<NarrowedVecOf<memtype,Vector>> $addr),`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`(Ptr<NarrowedVecOf<memtype,Vector>> $addr),`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!srl(memtype.size,3), $pred)>,`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`!srl(memtype.size,3), $pred)>,`。
- **L929 EN**: Adds a standalone statement or declaration: `NameOverride<mnemonic # "_p">;`.
  **L929 CN**: 添加一条独立语句或声明：`NameOverride<mnemonic # "_p">;`。
- **L930 EN**: Closes the current lexical scope or compound statement.
  **L930 CN**: 结束当前词法作用域或复合语句块。
- **L931 EN**: Closes the current lexical scope or compound statement.
  **L931 CN**: 结束当前词法作用域或复合语句块。
- **L932 EN**: Blank line separating nearby declarations or logic blocks.
  **L932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L933 EN**: Declares TableGen defm record `defm`.
  **L933 CN**: 声明 TableGen defm 记录 `defm`。
- **L934 EN**: Declares TableGen defm record `defm`.
  **L934 CN**: 声明 TableGen defm 记录 `defm`。
- **L935 EN**: Declares TableGen defm record `defm`.
  **L935 CN**: 声明 TableGen defm 记录 `defm`。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 937-960

````tablegen
multiclass gather_base<list<Type> types, int size> {
  let params = types, pnt = PNT_None in {
    def _gather_base: Intrinsic<
      Vector, (args UVector:$addr, imm_mem7bit<size>:$offset),
      (IRInt<"vldr_gather_base", [Vector, UVector]> $addr, $offset)>;

    def _gather_base_z: Intrinsic<
      Vector, (args UVector:$addr, imm_mem7bit<size>:$offset, Predicate:$pred),
      (IRInt<"vldr_gather_base_predicated", [Vector, UVector, Predicate]>
          $addr, $offset, $pred)>;

    def _gather_base_wb: Intrinsic<
      Vector, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset),
      (seq (IRInt<"vldr_gather_base_wb", [Vector, UVector]>
               (load $addr), $offset):$pair,
           (store (xval $pair, 1), $addr),
           (xval $pair, 0))>;

    def _gather_base_wb_z: Intrinsic<
      Vector, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset,
                    Predicate:$pred),
      (seq (IRInt<"vldr_gather_base_wb_predicated",
                  [Vector, UVector, Predicate]>
               (load $addr), $offset, $pred):$pair,
````
- **L937 EN**: Declares TableGen multiclass record `gather_base`.
  **L937 CN**: 声明 TableGen multiclass 记录 `gather_base`。
- **L938 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = types, pnt = PNT_None in {`.
  **L938 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = types, pnt = PNT_None in {`。
- **L939 EN**: Declares TableGen def record `_gather_base`.
  **L939 CN**: 声明 TableGen def 记录 `_gather_base`。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args UVector:$addr, imm_mem7bit<size>:$offset),`.
  **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args UVector:$addr, imm_mem7bit<size>:$offset),`。
- **L941 EN**: Executes a call or declaration centered on `statement`.
  **L941 CN**: 执行以 `statement` 为核心的调用或声明。
- **L942 EN**: Blank line separating nearby declarations or logic blocks.
  **L942 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L943 EN**: Declares TableGen def record `_gather_base_z`.
  **L943 CN**: 声明 TableGen def 记录 `_gather_base_z`。
- **L944 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args UVector:$addr, imm_mem7bit<size>:$offset, Predicate:$pred),`.
  **L944 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args UVector:$addr, imm_mem7bit<size>:$offset, Predicate:$pred),`。
- **L945 EN**: Continues the surrounding expression or declaration: `(IRInt<"vldr_gather_base_predicated", [Vector, UVector, Predicate]>`.
  **L945 CN**: 继续构造周围的表达式或声明：`(IRInt<"vldr_gather_base_predicated", [Vector, UVector, Predicate]>`。
- **L946 EN**: Adds a standalone statement or declaration: `$addr, $offset, $pred)>;`.
  **L946 CN**: 添加一条独立语句或声明：`$addr, $offset, $pred)>;`。
- **L947 EN**: Blank line separating nearby declarations or logic blocks.
  **L947 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L948 EN**: Declares TableGen def record `_gather_base_wb`.
  **L948 CN**: 声明 TableGen def 记录 `_gather_base_wb`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset),`.
  **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset),`。
- **L950 EN**: Continues logic associated with callable symbol `seq`.
  **L950 CN**: 继续与可调用符号 `seq` 相关的逻辑。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(load $addr), $offset):$pair,`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`(load $addr), $offset):$pair,`。
- **L952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (xval $pair, 1), $addr),`.
  **L952 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (xval $pair, 1), $addr),`。
- **L953 EN**: Executes a call or declaration centered on `statement`.
  **L953 CN**: 执行以 `statement` 为核心的调用或声明。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L955 EN**: Declares TableGen def record `_gather_base_wb_z`.
  **L955 CN**: 声明 TableGen def 记录 `_gather_base_wb_z`。
- **L956 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset,`.
  **L956 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset,`。
- **L957 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L957 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L958 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (IRInt<"vldr_gather_base_wb_predicated",`.
  **L958 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (IRInt<"vldr_gather_base_wb_predicated",`。
- **L959 EN**: Continues the surrounding expression or declaration: `[Vector, UVector, Predicate]>`.
  **L959 CN**: 继续构造周围的表达式或声明：`[Vector, UVector, Predicate]>`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(load $addr), $offset, $pred):$pair,`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`(load $addr), $offset, $pred):$pair,`。

### Lines 961-984

````tablegen
           (store (xval $pair, 1), $addr),
           (xval $pair, 0))>;
  }
}

defm vldrwq: gather_base<T.All32, 4>;
defm vldrdq: gather_base<T.All64, 8>;

multiclass scatter_base<list<Type> types, int size> {
  let params = types in {
    def _scatter_base: Intrinsic<
      Void, (args UVector:$addr, imm_mem7bit<size>:$offset, Vector:$data),
      (IRInt<"vstr_scatter_base", [UVector, Vector]> $addr, $offset, $data)>;

    def _scatter_base_p: Intrinsic<
      Void, (args UVector:$addr, imm_mem7bit<size>:$offset, Vector:$data,
                  Predicate:$pred),
      (IRInt<"vstr_scatter_base_predicated", [UVector, Vector, Predicate]>
          $addr, $offset, $data, $pred)>;

    def _scatter_base_wb: Intrinsic<
      Void, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset, Vector:$data),
      (seq (IRInt<"vstr_scatter_base_wb", [UVector, Vector]>
                 (load $addr), $offset, $data):$wbaddr,
````
- **L961 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (xval $pair, 1), $addr),`.
  **L961 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (xval $pair, 1), $addr),`。
- **L962 EN**: Executes a call or declaration centered on `statement`.
  **L962 CN**: 执行以 `statement` 为核心的调用或声明。
- **L963 EN**: Closes the current lexical scope or compound statement.
  **L963 CN**: 结束当前词法作用域或复合语句块。
- **L964 EN**: Closes the current lexical scope or compound statement.
  **L964 CN**: 结束当前词法作用域或复合语句块。
- **L965 EN**: Blank line separating nearby declarations or logic blocks.
  **L965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L966 EN**: Declares TableGen defm record `vldrwq`.
  **L966 CN**: 声明 TableGen defm 记录 `vldrwq`。
- **L967 EN**: Declares TableGen defm record `vldrdq`.
  **L967 CN**: 声明 TableGen defm 记录 `vldrdq`。
- **L968 EN**: Blank line separating nearby declarations or logic blocks.
  **L968 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L969 EN**: Declares TableGen multiclass record `scatter_base`.
  **L969 CN**: 声明 TableGen multiclass 记录 `scatter_base`。
- **L970 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = types in {`.
  **L970 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = types in {`。
- **L971 EN**: Declares TableGen def record `_scatter_base`.
  **L971 CN**: 声明 TableGen def 记录 `_scatter_base`。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args UVector:$addr, imm_mem7bit<size>:$offset, Vector:$data),`.
  **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args UVector:$addr, imm_mem7bit<size>:$offset, Vector:$data),`。
- **L973 EN**: Executes a call or declaration centered on `statement`.
  **L973 CN**: 执行以 `statement` 为核心的调用或声明。
- **L974 EN**: Blank line separating nearby declarations or logic blocks.
  **L974 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L975 EN**: Declares TableGen def record `_scatter_base_p`.
  **L975 CN**: 声明 TableGen def 记录 `_scatter_base_p`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args UVector:$addr, imm_mem7bit<size>:$offset, Vector:$data,`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args UVector:$addr, imm_mem7bit<size>:$offset, Vector:$data,`。
- **L977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L977 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L978 EN**: Continues the surrounding expression or declaration: `(IRInt<"vstr_scatter_base_predicated", [UVector, Vector, Predicate]>`.
  **L978 CN**: 继续构造周围的表达式或声明：`(IRInt<"vstr_scatter_base_predicated", [UVector, Vector, Predicate]>`。
- **L979 EN**: Adds a standalone statement or declaration: `$addr, $offset, $data, $pred)>;`.
  **L979 CN**: 添加一条独立语句或声明：`$addr, $offset, $data, $pred)>;`。
- **L980 EN**: Blank line separating nearby declarations or logic blocks.
  **L980 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L981 EN**: Declares TableGen def record `_scatter_base_wb`.
  **L981 CN**: 声明 TableGen def 记录 `_scatter_base_wb`。
- **L982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset, Vector:$data),`.
  **L982 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset, Vector:$data),`。
- **L983 EN**: Continues logic associated with callable symbol `seq`.
  **L983 CN**: 继续与可调用符号 `seq` 相关的逻辑。
- **L984 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(load $addr), $offset, $data):$wbaddr,`.
  **L984 CN**: 继续一个多行参数列表、初始化器或聚合项：`(load $addr), $offset, $data):$wbaddr,`。

### Lines 985-1008

````tablegen
           (store $wbaddr, $addr))>;

    def _scatter_base_wb_p: Intrinsic<
      Void, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset,
                    Vector:$data, Predicate:$pred),
      (seq (IRInt<"vstr_scatter_base_wb_predicated",
                  [UVector, Vector, Predicate]>
               (load $addr), $offset, $data, $pred):$wbaddr,
           (store $wbaddr, $addr))>;
  }
}

defm vstrwq: scatter_base<T.All32, 4>;
defm vstrdq: scatter_base<T.All64, 8>;

multiclass gather_offset_unshifted<list<Type> types, PrimitiveType memtype> {
  let params = types in {
    def _gather_offset: Intrinsic<
      Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets),
      (IRInt<"vldr_gather_offset",
             [Vector, CPtr<CopyKind<memtype, Scalar>>, UVector]>
          $base, $offsets, memtype.size, 0, (unsignedflag Scalar))>;
    def _gather_offset_z: Intrinsic<
      Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,
````
- **L985 EN**: Executes a call or declaration centered on `statement`.
  **L985 CN**: 执行以 `statement` 为核心的调用或声明。
- **L986 EN**: Blank line separating nearby declarations or logic blocks.
  **L986 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L987 EN**: Declares TableGen def record `_scatter_base_wb_p`.
  **L987 CN**: 声明 TableGen def 记录 `_scatter_base_wb_p`。
- **L988 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset,`.
  **L988 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args Ptr<UVector>:$addr, imm_mem7bit<size>:$offset,`。
- **L989 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$data, Predicate:$pred),`.
  **L989 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$data, Predicate:$pred),`。
- **L990 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (IRInt<"vstr_scatter_base_wb_predicated",`.
  **L990 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (IRInt<"vstr_scatter_base_wb_predicated",`。
- **L991 EN**: Continues the surrounding expression or declaration: `[UVector, Vector, Predicate]>`.
  **L991 CN**: 继续构造周围的表达式或声明：`[UVector, Vector, Predicate]>`。
- **L992 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(load $addr), $offset, $data, $pred):$wbaddr,`.
  **L992 CN**: 继续一个多行参数列表、初始化器或聚合项：`(load $addr), $offset, $data, $pred):$wbaddr,`。
- **L993 EN**: Executes a call or declaration centered on `statement`.
  **L993 CN**: 执行以 `statement` 为核心的调用或声明。
- **L994 EN**: Closes the current lexical scope or compound statement.
  **L994 CN**: 结束当前词法作用域或复合语句块。
- **L995 EN**: Closes the current lexical scope or compound statement.
  **L995 CN**: 结束当前词法作用域或复合语句块。
- **L996 EN**: Blank line separating nearby declarations or logic blocks.
  **L996 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L997 EN**: Declares TableGen defm record `vstrwq`.
  **L997 CN**: 声明 TableGen defm 记录 `vstrwq`。
- **L998 EN**: Declares TableGen defm record `vstrdq`.
  **L998 CN**: 声明 TableGen defm 记录 `vstrdq`。
- **L999 EN**: Blank line separating nearby declarations or logic blocks.
  **L999 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1000 EN**: Declares TableGen multiclass record `gather_offset_unshifted`.
  **L1000 CN**: 声明 TableGen multiclass 记录 `gather_offset_unshifted`。
- **L1001 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = types in {`.
  **L1001 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = types in {`。
- **L1002 EN**: Declares TableGen def record `_gather_offset`.
  **L1002 CN**: 声明 TableGen def 记录 `_gather_offset`。
- **L1003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets),`.
  **L1003 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets),`。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vldr_gather_offset",`.
  **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vldr_gather_offset",`。
- **L1005 EN**: Continues the surrounding expression or declaration: `[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector]>`.
  **L1005 CN**: 继续构造周围的表达式或声明：`[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector]>`。
- **L1006 EN**: Executes a call or declaration centered on `0,`.
  **L1006 CN**: 执行以 `0,` 为核心的调用或声明。
- **L1007 EN**: Declares TableGen def record `_gather_offset_z`.
  **L1007 CN**: 声明 TableGen def 记录 `_gather_offset_z`。
- **L1008 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`.
  **L1008 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`。

### Lines 1009-1032

````tablegen
                    Predicate:$pred),
      (IRInt<"vldr_gather_offset_predicated",
             [Vector, CPtr<CopyKind<memtype, Scalar>>, UVector, Predicate]>
          $base, $offsets, memtype.size, 0, (unsignedflag Scalar), $pred)>;
  }
}

multiclass gather_offset_shifted<list<Type> types, PrimitiveType memtype,
                                 int shift> {
  let params = types in {
    def _gather_shifted_offset: Intrinsic<
      Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets),
      (IRInt<"vldr_gather_offset",
             [Vector, CPtr<CopyKind<memtype, Scalar>>, UVector]>
          $base, $offsets, memtype.size, shift, (unsignedflag Scalar))>;
    def _gather_shifted_offset_z: Intrinsic<
      Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,
                    Predicate:$pred),
      (IRInt<"vldr_gather_offset_predicated",
             [Vector, CPtr<CopyKind<memtype, Scalar>>, UVector, Predicate]>
          $base, $offsets, memtype.size, shift, (unsignedflag Scalar), $pred)>;
  }
}

````
- **L1009 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1009 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vldr_gather_offset_predicated",`.
  **L1010 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vldr_gather_offset_predicated",`。
- **L1011 EN**: Continues the surrounding expression or declaration: `[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector, Predicate]>`.
  **L1011 CN**: 继续构造周围的表达式或声明：`[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector, Predicate]>`。
- **L1012 EN**: Executes a call or declaration centered on `0,`.
  **L1012 CN**: 执行以 `0,` 为核心的调用或声明。
- **L1013 EN**: Closes the current lexical scope or compound statement.
  **L1013 CN**: 结束当前词法作用域或复合语句块。
- **L1014 EN**: Closes the current lexical scope or compound statement.
  **L1014 CN**: 结束当前词法作用域或复合语句块。
- **L1015 EN**: Blank line separating nearby declarations or logic blocks.
  **L1015 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1016 EN**: Declares TableGen multiclass record `gather_offset_shifted`.
  **L1016 CN**: 声明 TableGen multiclass 记录 `gather_offset_shifted`。
- **L1017 EN**: Continues the surrounding expression or declaration: `int shift> {`.
  **L1017 CN**: 继续构造周围的表达式或声明：`int shift> {`。
- **L1018 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = types in {`.
  **L1018 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = types in {`。
- **L1019 EN**: Declares TableGen def record `_gather_shifted_offset`.
  **L1019 CN**: 声明 TableGen def 记录 `_gather_shifted_offset`。
- **L1020 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets),`.
  **L1020 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets),`。
- **L1021 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vldr_gather_offset",`.
  **L1021 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vldr_gather_offset",`。
- **L1022 EN**: Continues the surrounding expression or declaration: `[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector]>`.
  **L1022 CN**: 继续构造周围的表达式或声明：`[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector]>`。
- **L1023 EN**: Executes a call or declaration centered on `shift,`.
  **L1023 CN**: 执行以 `shift,` 为核心的调用或声明。
- **L1024 EN**: Declares TableGen def record `_gather_shifted_offset_z`.
  **L1024 CN**: 声明 TableGen def 记录 `_gather_shifted_offset_z`。
- **L1025 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`.
  **L1025 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args CPtr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`。
- **L1026 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1026 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1027 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vldr_gather_offset_predicated",`.
  **L1027 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vldr_gather_offset_predicated",`。
- **L1028 EN**: Continues the surrounding expression or declaration: `[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector, Predicate]>`.
  **L1028 CN**: 继续构造周围的表达式或声明：`[Vector, CPtr<CopyKind<memtype, Scalar>>, UVector, Predicate]>`。
- **L1029 EN**: Executes a call or declaration centered on `shift,`.
  **L1029 CN**: 执行以 `shift,` 为核心的调用或声明。
- **L1030 EN**: Closes the current lexical scope or compound statement.
  **L1030 CN**: 结束当前词法作用域或复合语句块。
- **L1031 EN**: Closes the current lexical scope or compound statement.
  **L1031 CN**: 结束当前词法作用域或复合语句块。
- **L1032 EN**: Blank line separating nearby declarations or logic blocks.
  **L1032 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1033-1056

````tablegen
multiclass gather_offset_both<list<Type> types, PrimitiveType memtype,
                              int shift> {
  defm "": gather_offset_unshifted<types, memtype>;
  defm "": gather_offset_shifted<types, memtype, shift>;
}

defm vldrbq: gather_offset_unshifted<!listconcat(T.All8, T.Int16, T.Int32), u8>;
defm vldrhq: gather_offset_both<!listconcat(T.All16, T.Int32), u16, 1>;
defm vldrwq: gather_offset_both<T.All32, u32, 2>;
defm vldrdq: gather_offset_both<T.Int64, u64, 3>;

multiclass scatter_offset_unshifted<list<Type> types, PrimitiveType memtype> {
  let params = types in {
    def _scatter_offset: Intrinsic<
      Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,
                  Vector:$data),
      (IRInt<"vstr_scatter_offset",
             [Ptr<CopyKind<memtype, Scalar>>, UVector, Vector]>
          $base, $offsets, $data, memtype.size, 0)>;
    def _scatter_offset_p: Intrinsic<
      Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,
                  Vector:$data, Predicate:$pred),
      (IRInt<"vstr_scatter_offset_predicated",
             [Ptr<CopyKind<memtype, Scalar>>, UVector, Vector, Predicate]>
````
- **L1033 EN**: Declares TableGen multiclass record `gather_offset_both`.
  **L1033 CN**: 声明 TableGen multiclass 记录 `gather_offset_both`。
- **L1034 EN**: Continues the surrounding expression or declaration: `int shift> {`.
  **L1034 CN**: 继续构造周围的表达式或声明：`int shift> {`。
- **L1035 EN**: Declares TableGen defm record `""`.
  **L1035 CN**: 声明 TableGen defm 记录 `""`。
- **L1036 EN**: Declares TableGen defm record `""`.
  **L1036 CN**: 声明 TableGen defm 记录 `""`。
- **L1037 EN**: Closes the current lexical scope or compound statement.
  **L1037 CN**: 结束当前词法作用域或复合语句块。
- **L1038 EN**: Blank line separating nearby declarations or logic blocks.
  **L1038 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1039 EN**: Declares TableGen defm record `vldrbq`.
  **L1039 CN**: 声明 TableGen defm 记录 `vldrbq`。
- **L1040 EN**: Declares TableGen defm record `vldrhq`.
  **L1040 CN**: 声明 TableGen defm 记录 `vldrhq`。
- **L1041 EN**: Declares TableGen defm record `vldrwq`.
  **L1041 CN**: 声明 TableGen defm 记录 `vldrwq`。
- **L1042 EN**: Declares TableGen defm record `vldrdq`.
  **L1042 CN**: 声明 TableGen defm 记录 `vldrdq`。
- **L1043 EN**: Blank line separating nearby declarations or logic blocks.
  **L1043 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1044 EN**: Declares TableGen multiclass record `scatter_offset_unshifted`.
  **L1044 CN**: 声明 TableGen multiclass 记录 `scatter_offset_unshifted`。
- **L1045 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = types in {`.
  **L1045 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = types in {`。
- **L1046 EN**: Declares TableGen def record `_scatter_offset`.
  **L1046 CN**: 声明 TableGen def 记录 `_scatter_offset`。
- **L1047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`.
  **L1047 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`。
- **L1048 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$data),`.
  **L1048 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$data),`。
- **L1049 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vstr_scatter_offset",`.
  **L1049 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vstr_scatter_offset",`。
- **L1050 EN**: Continues the surrounding expression or declaration: `[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector]>`.
  **L1050 CN**: 继续构造周围的表达式或声明：`[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector]>`。
- **L1051 EN**: Adds a standalone statement or declaration: `$base, $offsets, $data, memtype.size, 0)>;`.
  **L1051 CN**: 添加一条独立语句或声明：`$base, $offsets, $data, memtype.size, 0)>;`。
- **L1052 EN**: Declares TableGen def record `_scatter_offset_p`.
  **L1052 CN**: 声明 TableGen def 记录 `_scatter_offset_p`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`。
- **L1054 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$data, Predicate:$pred),`.
  **L1054 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$data, Predicate:$pred),`。
- **L1055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vstr_scatter_offset_predicated",`.
  **L1055 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vstr_scatter_offset_predicated",`。
- **L1056 EN**: Continues the surrounding expression or declaration: `[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector, Predicate]>`.
  **L1056 CN**: 继续构造周围的表达式或声明：`[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector, Predicate]>`。

### Lines 1057-1080

````tablegen
          $base, $offsets, $data, memtype.size, 0, $pred)>;
  }
}

multiclass scatter_offset_shifted<list<Type> types, PrimitiveType memtype,
                                  int shift> {
  let params = types in {
    def _scatter_shifted_offset: Intrinsic<
      Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,
                  Vector:$data),
      (IRInt<"vstr_scatter_offset",
             [Ptr<CopyKind<memtype, Scalar>>, UVector, Vector]>
          $base, $offsets, $data, memtype.size, shift)>;
    def _scatter_shifted_offset_p: Intrinsic<
      Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,
                  Vector:$data, Predicate:$pred),
      (IRInt<"vstr_scatter_offset_predicated",
             [Ptr<CopyKind<memtype, Scalar>>, UVector, Vector, Predicate]>
          $base, $offsets, $data, memtype.size, shift, $pred)>;
  }
}

multiclass scatter_offset_both<list<Type> types, PrimitiveType memtype,
                               int shift> {
````
- **L1057 EN**: Adds a standalone statement or declaration: `$base, $offsets, $data, memtype.size, 0, $pred)>;`.
  **L1057 CN**: 添加一条独立语句或声明：`$base, $offsets, $data, memtype.size, 0, $pred)>;`。
- **L1058 EN**: Closes the current lexical scope or compound statement.
  **L1058 CN**: 结束当前词法作用域或复合语句块。
- **L1059 EN**: Closes the current lexical scope or compound statement.
  **L1059 CN**: 结束当前词法作用域或复合语句块。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1061 EN**: Declares TableGen multiclass record `scatter_offset_shifted`.
  **L1061 CN**: 声明 TableGen multiclass 记录 `scatter_offset_shifted`。
- **L1062 EN**: Continues the surrounding expression or declaration: `int shift> {`.
  **L1062 CN**: 继续构造周围的表达式或声明：`int shift> {`。
- **L1063 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = types in {`.
  **L1063 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = types in {`。
- **L1064 EN**: Declares TableGen def record `_scatter_shifted_offset`.
  **L1064 CN**: 声明 TableGen def 记录 `_scatter_shifted_offset`。
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$data),`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$data),`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vstr_scatter_offset",`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vstr_scatter_offset",`。
- **L1068 EN**: Continues the surrounding expression or declaration: `[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector]>`.
  **L1068 CN**: 继续构造周围的表达式或声明：`[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector]>`。
- **L1069 EN**: Adds a standalone statement or declaration: `$base, $offsets, $data, memtype.size, shift)>;`.
  **L1069 CN**: 添加一条独立语句或声明：`$base, $offsets, $data, memtype.size, shift)>;`。
- **L1070 EN**: Declares TableGen def record `_scatter_shifted_offset_p`.
  **L1070 CN**: 声明 TableGen def 记录 `_scatter_shifted_offset_p`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`Void, (args Ptr<CopyKind<memtype, Scalar>>:$base, UVector:$offsets,`。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector:$data, Predicate:$pred),`.
  **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector:$data, Predicate:$pred),`。
- **L1073 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vstr_scatter_offset_predicated",`.
  **L1073 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vstr_scatter_offset_predicated",`。
- **L1074 EN**: Continues the surrounding expression or declaration: `[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector, Predicate]>`.
  **L1074 CN**: 继续构造周围的表达式或声明：`[Ptr<CopyKind<memtype, Scalar>>, UVector, Vector, Predicate]>`。
- **L1075 EN**: Adds a standalone statement or declaration: `$base, $offsets, $data, memtype.size, shift, $pred)>;`.
  **L1075 CN**: 添加一条独立语句或声明：`$base, $offsets, $data, memtype.size, shift, $pred)>;`。
- **L1076 EN**: Closes the current lexical scope or compound statement.
  **L1076 CN**: 结束当前词法作用域或复合语句块。
- **L1077 EN**: Closes the current lexical scope or compound statement.
  **L1077 CN**: 结束当前词法作用域或复合语句块。
- **L1078 EN**: Blank line separating nearby declarations or logic blocks.
  **L1078 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1079 EN**: Declares TableGen multiclass record `scatter_offset_both`.
  **L1079 CN**: 声明 TableGen multiclass 记录 `scatter_offset_both`。
- **L1080 EN**: Continues the surrounding expression or declaration: `int shift> {`.
  **L1080 CN**: 继续构造周围的表达式或声明：`int shift> {`。

### Lines 1081-1104

````tablegen
  defm "": scatter_offset_unshifted<types, memtype>;
  defm "": scatter_offset_shifted<types, memtype, shift>;
}

defm vstrbq: scatter_offset_unshifted<!listconcat(T.All8,T.Int16,T.Int32), u8>;
defm vstrhq: scatter_offset_both<!listconcat(T.All16, T.Int32), u16, 1>;
defm vstrwq: scatter_offset_both<T.All32, u32, 2>;
defm vstrdq: scatter_offset_both<T.Int64, u64, 3>;

let params = T.Int in {
  def vshlq_n: Intrinsic<Vector, (args Vector:$v, imm_0toNm1:$sh),
                         (shl $v, (splat (Scalar $sh)))>;
  defm vshlq: IntrinsicMX<Vector, (args Vector:$v, imm_0toNm1:$sh,
                                        Predicate:$pred),
      (IRInt<"shl_imm_predicated", [Vector, Predicate]>
           $v, $sh, $pred, $inactive), 1, "_n">;

  let pnt = PNT_NType in {
    def vshrq_n: Intrinsic<Vector, (args Vector:$v, imm_1toN:$sh),
                            (immshr $v, $sh, (unsignedflag Scalar))>;
    defm vshrq: IntrinsicMX<Vector, (args Vector:$v, imm_1toN:$sh,
                                          Predicate:$pred),
        (IRInt<"shr_imm_predicated", [Vector, Predicate]>
             $v, $sh, (unsignedflag Scalar), $pred, $inactive), 1, "_n">;
````
- **L1081 EN**: Declares TableGen defm record `""`.
  **L1081 CN**: 声明 TableGen defm 记录 `""`。
- **L1082 EN**: Declares TableGen defm record `""`.
  **L1082 CN**: 声明 TableGen defm 记录 `""`。
- **L1083 EN**: Closes the current lexical scope or compound statement.
  **L1083 CN**: 结束当前词法作用域或复合语句块。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1085 EN**: Declares TableGen defm record `vstrbq`.
  **L1085 CN**: 声明 TableGen defm 记录 `vstrbq`。
- **L1086 EN**: Declares TableGen defm record `vstrhq`.
  **L1086 CN**: 声明 TableGen defm 记录 `vstrhq`。
- **L1087 EN**: Declares TableGen defm record `vstrwq`.
  **L1087 CN**: 声明 TableGen defm 记录 `vstrwq`。
- **L1088 EN**: Declares TableGen defm record `vstrdq`.
  **L1088 CN**: 声明 TableGen defm 记录 `vstrdq`。
- **L1089 EN**: Blank line separating nearby declarations or logic blocks.
  **L1089 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1090 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L1090 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L1091 EN**: Declares TableGen def record `vshlq_n`.
  **L1091 CN**: 声明 TableGen def 记录 `vshlq_n`。
- **L1092 EN**: Executes a call or declaration centered on `statement`.
  **L1092 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1093 EN**: Declares TableGen defm record `vshlq`.
  **L1093 CN**: 声明 TableGen defm 记录 `vshlq`。
- **L1094 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1094 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1095 EN**: Continues the surrounding expression or declaration: `(IRInt<"shl_imm_predicated", [Vector, Predicate]>`.
  **L1095 CN**: 继续构造周围的表达式或声明：`(IRInt<"shl_imm_predicated", [Vector, Predicate]>`。
- **L1096 EN**: Adds a standalone statement or declaration: `$v, $sh, $pred, $inactive), 1, "_n">;`.
  **L1096 CN**: 添加一条独立语句或声明：`$v, $sh, $pred, $inactive), 1, "_n">;`。
- **L1097 EN**: Blank line separating nearby declarations or logic blocks.
  **L1097 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1098 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L1098 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L1099 EN**: Declares TableGen def record `vshrq_n`.
  **L1099 CN**: 声明 TableGen def 记录 `vshrq_n`。
- **L1100 EN**: Executes a call or declaration centered on `statement`.
  **L1100 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1101 EN**: Declares TableGen defm record `vshrq`.
  **L1101 CN**: 声明 TableGen defm 记录 `vshrq`。
- **L1102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1102 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1103 EN**: Continues the surrounding expression or declaration: `(IRInt<"shr_imm_predicated", [Vector, Predicate]>`.
  **L1103 CN**: 继续构造周围的表达式或声明：`(IRInt<"shr_imm_predicated", [Vector, Predicate]>`。
- **L1104 EN**: Executes a call or declaration centered on `$sh,`.
  **L1104 CN**: 执行以 `$sh,` 为核心的调用或声明。

### Lines 1105-1128

````tablegen
  }
}

let params = T.Int in {
  def vqshlq_n: Intrinsic<Vector, (args Vector:$v, imm_0toNm1:$sh),
       (IRInt<"vqshl_imm", [Vector]> $v, $sh, (unsignedflag Scalar))>;
  def vqshlq_m_n: Intrinsic<Vector, (args Vector:$inactive, Vector:$v,
                                          imm_0toNm1:$sh, Predicate:$pred),
       (IRInt<"vqshl_imm_predicated", [Vector, Predicate]>
            $v, $sh, (unsignedflag Scalar), $pred, $inactive)>;

  let pnt = PNT_NType in {
    def vrshrq_n: Intrinsic<Vector, (args Vector:$v, imm_1toN:$sh),
         (IRInt<"vrshr_imm", [Vector]> $v, $sh, (unsignedflag Scalar))>;
    defm vrshrq: IntrinsicMX<Vector, (args Vector:$v, imm_1toN:$sh,
                                           Predicate:$pred),
         (IRInt<"vrshr_imm_predicated", [Vector, Predicate]>
              $v, $sh, (unsignedflag Scalar), $pred, $inactive), 1, "_n">;
  }
}

let params = T.Signed, pnt = PNT_NType in {
  def vqshluq_n: Intrinsic<UVector, (args Vector:$v, imm_0toNm1:$sh),
       (IRInt<"vqshlu_imm", [Vector]> $v, $sh)>;
````
- **L1105 EN**: Closes the current lexical scope or compound statement.
  **L1105 CN**: 结束当前词法作用域或复合语句块。
- **L1106 EN**: Closes the current lexical scope or compound statement.
  **L1106 CN**: 结束当前词法作用域或复合语句块。
- **L1107 EN**: Blank line separating nearby declarations or logic blocks.
  **L1107 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1108 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L1108 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L1109 EN**: Declares TableGen def record `vqshlq_n`.
  **L1109 CN**: 声明 TableGen def 记录 `vqshlq_n`。
- **L1110 EN**: Executes a call or declaration centered on `statement`.
  **L1110 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1111 EN**: Declares TableGen def record `vqshlq_m_n`.
  **L1111 CN**: 声明 TableGen def 记录 `vqshlq_m_n`。
- **L1112 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `imm_0toNm1:$sh, Predicate:$pred),`.
  **L1112 CN**: 继续一个多行参数列表、初始化器或聚合项：`imm_0toNm1:$sh, Predicate:$pred),`。
- **L1113 EN**: Continues the surrounding expression or declaration: `(IRInt<"vqshl_imm_predicated", [Vector, Predicate]>`.
  **L1113 CN**: 继续构造周围的表达式或声明：`(IRInt<"vqshl_imm_predicated", [Vector, Predicate]>`。
- **L1114 EN**: Executes a call or declaration centered on `$sh,`.
  **L1114 CN**: 执行以 `$sh,` 为核心的调用或声明。
- **L1115 EN**: Blank line separating nearby declarations or logic blocks.
  **L1115 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1116 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in {`.
  **L1116 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in {`。
- **L1117 EN**: Declares TableGen def record `vrshrq_n`.
  **L1117 CN**: 声明 TableGen def 记录 `vrshrq_n`。
- **L1118 EN**: Executes a call or declaration centered on `statement`.
  **L1118 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1119 EN**: Declares TableGen defm record `vrshrq`.
  **L1119 CN**: 声明 TableGen defm 记录 `vrshrq`。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1121 EN**: Continues the surrounding expression or declaration: `(IRInt<"vrshr_imm_predicated", [Vector, Predicate]>`.
  **L1121 CN**: 继续构造周围的表达式或声明：`(IRInt<"vrshr_imm_predicated", [Vector, Predicate]>`。
- **L1122 EN**: Executes a call or declaration centered on `$sh,`.
  **L1122 CN**: 执行以 `$sh,` 为核心的调用或声明。
- **L1123 EN**: Closes the current lexical scope or compound statement.
  **L1123 CN**: 结束当前词法作用域或复合语句块。
- **L1124 EN**: Closes the current lexical scope or compound statement.
  **L1124 CN**: 结束当前词法作用域或复合语句块。
- **L1125 EN**: Blank line separating nearby declarations or logic blocks.
  **L1125 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1126 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed, pnt = PNT_NType in {`.
  **L1126 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed, pnt = PNT_NType in {`。
- **L1127 EN**: Declares TableGen def record `vqshluq_n`.
  **L1127 CN**: 声明 TableGen def 记录 `vqshluq_n`。
- **L1128 EN**: Executes a call or declaration centered on `statement`.
  **L1128 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1129-1152

````tablegen
  def vqshluq_m_n: Intrinsic<UVector, (args UVector:$inactive, Vector:$v,
                                            imm_0toNm1:$sh, Predicate:$pred),
       (IRInt<"vqshlu_imm_predicated", [Vector, Predicate]>
            $v, $sh, $pred, $inactive)>;
}

multiclass vshll_imm<int top> {
  let params = !listconcat(T.Int8, T.Int16), pnt = PNT_NType in {
    def _n: Intrinsic<DblVector, (args Vector:$v, imm_1toN:$sh),
        (IRInt<"vshll_imm", [DblVector, Vector]>
            $v, $sh, (unsignedflag Scalar), top)>;
    defm "": IntrinsicMX<DblVector, (args Vector:$v, imm_1toN:$sh,
                                          DblPredicate:$pred),
        (IRInt<"vshll_imm_predicated", [DblVector, Vector, DblPredicate]>
            $v, $sh, (unsignedflag Scalar), top, $pred, $inactive), 1, "_n">;
  }
}
defm vshllbq : vshll_imm<0>;
defm vshlltq : vshll_imm<1>;

multiclass DyadicImmShift<Type outtype, Immediate imm, string intname = NAME,
                          dag extraargs = (?)> {
  defvar intparams = !if(!eq(outtype, Vector), [Vector], [outtype, Vector]);

````
- **L1129 EN**: Declares TableGen def record `vqshluq_m_n`.
  **L1129 CN**: 声明 TableGen def 记录 `vqshluq_m_n`。
- **L1130 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `imm_0toNm1:$sh, Predicate:$pred),`.
  **L1130 CN**: 继续一个多行参数列表、初始化器或聚合项：`imm_0toNm1:$sh, Predicate:$pred),`。
- **L1131 EN**: Continues the surrounding expression or declaration: `(IRInt<"vqshlu_imm_predicated", [Vector, Predicate]>`.
  **L1131 CN**: 继续构造周围的表达式或声明：`(IRInt<"vqshlu_imm_predicated", [Vector, Predicate]>`。
- **L1132 EN**: Adds a standalone statement or declaration: `$v, $sh, $pred, $inactive)>;`.
  **L1132 CN**: 添加一条独立语句或声明：`$v, $sh, $pred, $inactive)>;`。
- **L1133 EN**: Closes the current lexical scope or compound statement.
  **L1133 CN**: 结束当前词法作用域或复合语句块。
- **L1134 EN**: Blank line separating nearby declarations or logic blocks.
  **L1134 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1135 EN**: Declares TableGen multiclass record `vshll_imm`.
  **L1135 CN**: 声明 TableGen multiclass 记录 `vshll_imm`。
- **L1136 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = !listconcat(T.Int8, T.Int16), pnt = PNT_NType in {`.
  **L1136 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = !listconcat(T.Int8, T.Int16), pnt = PNT_NType in {`。
- **L1137 EN**: Declares TableGen def record `_n`.
  **L1137 CN**: 声明 TableGen def 记录 `_n`。
- **L1138 EN**: Continues the surrounding expression or declaration: `(IRInt<"vshll_imm", [DblVector, Vector]>`.
  **L1138 CN**: 继续构造周围的表达式或声明：`(IRInt<"vshll_imm", [DblVector, Vector]>`。
- **L1139 EN**: Executes a call or declaration centered on `$sh,`.
  **L1139 CN**: 执行以 `$sh,` 为核心的调用或声明。
- **L1140 EN**: Declares TableGen defm record `""`.
  **L1140 CN**: 声明 TableGen defm 记录 `""`。
- **L1141 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DblPredicate:$pred),`.
  **L1141 CN**: 继续一个多行参数列表、初始化器或聚合项：`DblPredicate:$pred),`。
- **L1142 EN**: Continues the surrounding expression or declaration: `(IRInt<"vshll_imm_predicated", [DblVector, Vector, DblPredicate]>`.
  **L1142 CN**: 继续构造周围的表达式或声明：`(IRInt<"vshll_imm_predicated", [DblVector, Vector, DblPredicate]>`。
- **L1143 EN**: Executes a call or declaration centered on `$sh,`.
  **L1143 CN**: 执行以 `$sh,` 为核心的调用或声明。
- **L1144 EN**: Closes the current lexical scope or compound statement.
  **L1144 CN**: 结束当前词法作用域或复合语句块。
- **L1145 EN**: Closes the current lexical scope or compound statement.
  **L1145 CN**: 结束当前词法作用域或复合语句块。
- **L1146 EN**: Declares TableGen defm record `vshllbq`.
  **L1146 CN**: 声明 TableGen defm 记录 `vshllbq`。
- **L1147 EN**: Declares TableGen defm record `vshlltq`.
  **L1147 CN**: 声明 TableGen defm 记录 `vshlltq`。
- **L1148 EN**: Blank line separating nearby declarations or logic blocks.
  **L1148 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1149 EN**: Declares TableGen multiclass record `DyadicImmShift`.
  **L1149 CN**: 声明 TableGen multiclass 记录 `DyadicImmShift`。
- **L1150 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `dag extraargs = (?)> {`.
  **L1150 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`dag extraargs = (?)> {`。
- **L1151 EN**: Declares TableGen defvar record `intparams = !if`.
  **L1151 CN**: 声明 TableGen defvar 记录 `intparams = !if`。
- **L1152 EN**: Blank line separating nearby declarations or logic blocks.
  **L1152 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1153-1176

````tablegen
  def q_n: Intrinsic<
      outtype, (args outtype:$a, Vector:$b, imm:$sh),
      !con((IRInt<intname, intparams> $a, $b, $sh), extraargs)>;

  def q_m_n: Intrinsic<
      outtype, (args outtype:$a, Vector:$b, imm:$sh, Predicate:$pred),
      !con((IRInt<intname # "_predicated", intparams # [Predicate]>
               $a, $b, $sh), extraargs, (? $pred))>;
}

multiclass VSHRN<Type outtype, Immediate imm, dag extraargs> {
  defm b: DyadicImmShift<outtype, imm, "vshrn", !con(extraargs, (? 0))>;
  defm t: DyadicImmShift<outtype, imm, "vshrn", !con(extraargs, (? 1))>;
}

let params = [s16, s32, u16, u32], pnt = PNT_NType in {
  defvar U = (unsignedflag Scalar);
  defm vshrn   : VSHRN<HalfVector, imm_1toHalfN, (? 0,0,U,U)>;
  defm vqshrn  : VSHRN<HalfVector, imm_1toHalfN, (? 1,0,U,U)>;
  defm vrshrn  : VSHRN<HalfVector, imm_1toHalfN, (? 0,1,U,U)>;
  defm vqrshrn : VSHRN<HalfVector, imm_1toHalfN, (? 1,1,U,U)>;
}
let params = [s16, s32], pnt = PNT_NType in {
  defm vqshrun  : VSHRN<UHalfVector, imm_1toHalfN, (? 1,0,1,0)>;
````
- **L1153 EN**: Declares TableGen def record `q_n`.
  **L1153 CN**: 声明 TableGen def 记录 `q_n`。
- **L1154 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `outtype, (args outtype:$a, Vector:$b, imm:$sh),`.
  **L1154 CN**: 继续一个多行参数列表、初始化器或聚合项：`outtype, (args outtype:$a, Vector:$b, imm:$sh),`。
- **L1155 EN**: Executes a call or declaration centered on `!con`.
  **L1155 CN**: 执行以 `!con` 为核心的调用或声明。
- **L1156 EN**: Blank line separating nearby declarations or logic blocks.
  **L1156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1157 EN**: Declares TableGen def record `q_m_n`.
  **L1157 CN**: 声明 TableGen def 记录 `q_m_n`。
- **L1158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `outtype, (args outtype:$a, Vector:$b, imm:$sh, Predicate:$pred),`.
  **L1158 CN**: 继续一个多行参数列表、初始化器或聚合项：`outtype, (args outtype:$a, Vector:$b, imm:$sh, Predicate:$pred),`。
- **L1159 EN**: Continues logic associated with callable symbol `con`.
  **L1159 CN**: 继续与可调用符号 `con` 相关的逻辑。
- **L1160 EN**: Executes a call or declaration centered on `extraargs,`.
  **L1160 CN**: 执行以 `extraargs,` 为核心的调用或声明。
- **L1161 EN**: Closes the current lexical scope or compound statement.
  **L1161 CN**: 结束当前词法作用域或复合语句块。
- **L1162 EN**: Blank line separating nearby declarations or logic blocks.
  **L1162 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1163 EN**: Declares TableGen multiclass record `VSHRN`.
  **L1163 CN**: 声明 TableGen multiclass 记录 `VSHRN`。
- **L1164 EN**: Declares TableGen defm record `b`.
  **L1164 CN**: 声明 TableGen defm 记录 `b`。
- **L1165 EN**: Declares TableGen defm record `t`.
  **L1165 CN**: 声明 TableGen defm 记录 `t`。
- **L1166 EN**: Closes the current lexical scope or compound statement.
  **L1166 CN**: 结束当前词法作用域或复合语句块。
- **L1167 EN**: Blank line separating nearby declarations or logic blocks.
  **L1167 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1168 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, s32, u16, u32], pnt = PNT_NType in {`.
  **L1168 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, s32, u16, u32], pnt = PNT_NType in {`。
- **L1169 EN**: Declares TableGen defvar record `U =`.
  **L1169 CN**: 声明 TableGen defvar 记录 `U =`。
- **L1170 EN**: Declares TableGen defm record `vshrn`.
  **L1170 CN**: 声明 TableGen defm 记录 `vshrn`。
- **L1171 EN**: Declares TableGen defm record `vqshrn`.
  **L1171 CN**: 声明 TableGen defm 记录 `vqshrn`。
- **L1172 EN**: Declares TableGen defm record `vrshrn`.
  **L1172 CN**: 声明 TableGen defm 记录 `vrshrn`。
- **L1173 EN**: Declares TableGen defm record `vqrshrn`.
  **L1173 CN**: 声明 TableGen defm 记录 `vqrshrn`。
- **L1174 EN**: Closes the current lexical scope or compound statement.
  **L1174 CN**: 结束当前词法作用域或复合语句块。
- **L1175 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, s32], pnt = PNT_NType in {`.
  **L1175 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, s32], pnt = PNT_NType in {`。
- **L1176 EN**: Declares TableGen defm record `vqshrun`.
  **L1176 CN**: 声明 TableGen defm 记录 `vqshrun`。

### Lines 1177-1200

````tablegen
  defm vqrshrun : VSHRN<UHalfVector, imm_1toHalfN, (? 1,1,1,0)>;
}
let params = T.Int, pnt = PNT_NType in {
  defm vsli : DyadicImmShift<Vector, imm_0toNm1>;
  defm vsri : DyadicImmShift<Vector, imm_1toN>;
}

multiclass VSHL_non_imm<string scalarSuffix, int q, int r,
                        PolymorphicNameType pnt_scalar_unpred = PNT_Type> {
  let pnt = pnt_scalar_unpred in {
    def scalarSuffix: Intrinsic<
      Vector, (args Vector:$in, s32:$sh),
      (IRInt<"vshl_scalar", [Vector]> $in, $sh,
           q, r, (unsignedflag Scalar))>;
  }
  def "_m" # scalarSuffix: Intrinsic<
    Vector, (args Vector:$in, s32:$sh, Predicate:$pred),
    (IRInt<"vshl_scalar_predicated", [Vector, Predicate]> $in, $sh,
         q, r, (unsignedflag Scalar), $pred)>;

  def "": Intrinsic<
    Vector, (args Vector:$in, SVector:$sh),
    (IRInt<"vshl_vector", [Vector, SVector]> $in, $sh,
         q, r, (unsignedflag Scalar))>;
````
- **L1177 EN**: Declares TableGen defm record `vqrshrun`.
  **L1177 CN**: 声明 TableGen defm 记录 `vqrshrun`。
- **L1178 EN**: Closes the current lexical scope or compound statement.
  **L1178 CN**: 结束当前词法作用域或复合语句块。
- **L1179 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int, pnt = PNT_NType in {`.
  **L1179 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int, pnt = PNT_NType in {`。
- **L1180 EN**: Declares TableGen defm record `vsli`.
  **L1180 CN**: 声明 TableGen defm 记录 `vsli`。
- **L1181 EN**: Declares TableGen defm record `vsri`.
  **L1181 CN**: 声明 TableGen defm 记录 `vsri`。
- **L1182 EN**: Closes the current lexical scope or compound statement.
  **L1182 CN**: 结束当前词法作用域或复合语句块。
- **L1183 EN**: Blank line separating nearby declarations or logic blocks.
  **L1183 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1184 EN**: Declares TableGen multiclass record `VSHL_non_imm`.
  **L1184 CN**: 声明 TableGen multiclass 记录 `VSHL_non_imm`。
- **L1185 EN**: Continues the surrounding expression or declaration: `PolymorphicNameType pnt_scalar_unpred = PNT_Type> {`.
  **L1185 CN**: 继续构造周围的表达式或声明：`PolymorphicNameType pnt_scalar_unpred = PNT_Type> {`。
- **L1186 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = pnt_scalar_unpred in {`.
  **L1186 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = pnt_scalar_unpred in {`。
- **L1187 EN**: Declares TableGen def record `scalarSuffix`.
  **L1187 CN**: 声明 TableGen def 记录 `scalarSuffix`。
- **L1188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$in, s32:$sh),`.
  **L1188 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$in, s32:$sh),`。
- **L1189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vshl_scalar", [Vector]> $in, $sh,`.
  **L1189 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vshl_scalar", [Vector]> $in, $sh,`。
- **L1190 EN**: Executes a call or declaration centered on `r,`.
  **L1190 CN**: 执行以 `r,` 为核心的调用或声明。
- **L1191 EN**: Closes the current lexical scope or compound statement.
  **L1191 CN**: 结束当前词法作用域或复合语句块。
- **L1192 EN**: Declares TableGen def record `"_m" # scalarSuffix`.
  **L1192 CN**: 声明 TableGen def 记录 `"_m" # scalarSuffix`。
- **L1193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$in, s32:$sh, Predicate:$pred),`.
  **L1193 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$in, s32:$sh, Predicate:$pred),`。
- **L1194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vshl_scalar_predicated", [Vector, Predicate]> $in, $sh,`.
  **L1194 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vshl_scalar_predicated", [Vector, Predicate]> $in, $sh,`。
- **L1195 EN**: Executes a call or declaration centered on `r,`.
  **L1195 CN**: 执行以 `r,` 为核心的调用或声明。
- **L1196 EN**: Blank line separating nearby declarations or logic blocks.
  **L1196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1197 EN**: Declares TableGen def record `""`.
  **L1197 CN**: 声明 TableGen def 记录 `""`。
- **L1198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$in, SVector:$sh),`.
  **L1198 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$in, SVector:$sh),`。
- **L1199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vshl_vector", [Vector, SVector]> $in, $sh,`.
  **L1199 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vshl_vector", [Vector, SVector]> $in, $sh,`。
- **L1200 EN**: Executes a call or declaration centered on `r,`.
  **L1200 CN**: 执行以 `r,` 为核心的调用或声明。

### Lines 1201-1224

````tablegen
  defm "": IntrinsicMX<
    Vector, (args Vector:$in, SVector:$sh, Predicate:$pred),
    (IRInt<"vshl_vector_predicated", [Vector, SVector, Predicate]> $in, $sh,
         q, r, (unsignedflag Scalar), $pred, $inactive),
    // The saturating shift intrinsics don't have an x variant, so we
    // set wantXVariant to 1 iff q == 0
    !eq(q, 0)>;
}

let params = T.Int in {
  defm vshlq   : VSHL_non_imm<"_r", 0, 0>;
  defm vqshlq  : VSHL_non_imm<"_r", 1, 0>;
  defm vrshlq  : VSHL_non_imm<"_n", 0, 1, PNT_NType>;
  defm vqrshlq : VSHL_non_imm<"_n", 1, 1, PNT_NType>;
}

// Base class for the scalar shift intrinsics.
class ScalarShift<Type argtype, dag shiftCountArg, dag shiftCodeGen>:
  Intrinsic<argtype, !con((args argtype:$value), shiftCountArg), shiftCodeGen> {
  let params = [Void];
  let pnt = PNT_None;
}

// Subclass that includes the machinery to take a 64-bit input apart
````
- **L1201 EN**: Declares TableGen defm record `""`.
  **L1201 CN**: 声明 TableGen defm 记录 `""`。
- **L1202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$in, SVector:$sh, Predicate:$pred),`.
  **L1202 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$in, SVector:$sh, Predicate:$pred),`。
- **L1203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vshl_vector_predicated", [Vector, SVector, Predicate]> $in, $sh,`.
  **L1203 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vshl_vector_predicated", [Vector, SVector, Predicate]> $in, $sh,`。
- **L1204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `q, r, (unsignedflag Scalar), $pred, $inactive),`.
  **L1204 CN**: 继续一个多行参数列表、初始化器或聚合项：`q, r, (unsignedflag Scalar), $pred, $inactive),`。
- **L1205 EN**: Comment explains nearby logic, constraints, or intent: `The saturating shift intrinsics don't have an x variant, so we`.
  **L1205 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The saturating shift intrinsics don't have an x variant, so we`。
- **L1206 EN**: Comment explains nearby logic, constraints, or intent: `set wantXVariant to 1 iff q 0`.
  **L1206 CN**: 注释解释附近代码的逻辑、约束或设计意图：`set wantXVariant to 1 iff q 0`。
- **L1207 EN**: Executes a call or declaration centered on `!eq`.
  **L1207 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L1208 EN**: Closes the current lexical scope or compound statement.
  **L1208 CN**: 结束当前词法作用域或复合语句块。
- **L1209 EN**: Blank line separating nearby declarations or logic blocks.
  **L1209 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1210 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L1210 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L1211 EN**: Declares TableGen defm record `vshlq`.
  **L1211 CN**: 声明 TableGen defm 记录 `vshlq`。
- **L1212 EN**: Declares TableGen defm record `vqshlq`.
  **L1212 CN**: 声明 TableGen defm 记录 `vqshlq`。
- **L1213 EN**: Declares TableGen defm record `vrshlq`.
  **L1213 CN**: 声明 TableGen defm 记录 `vrshlq`。
- **L1214 EN**: Declares TableGen defm record `vqrshlq`.
  **L1214 CN**: 声明 TableGen defm 记录 `vqrshlq`。
- **L1215 EN**: Closes the current lexical scope or compound statement.
  **L1215 CN**: 结束当前词法作用域或复合语句块。
- **L1216 EN**: Blank line separating nearby declarations or logic blocks.
  **L1216 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1217 EN**: Comment explains nearby logic, constraints, or intent: `Base class for the scalar shift intrinsics.`.
  **L1217 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Base class for the scalar shift intrinsics.`。
- **L1218 EN**: Declares TableGen class record `ScalarShift`.
  **L1218 CN**: 声明 TableGen class 记录 `ScalarShift`。
- **L1219 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `Intrinsic<argtype, !con((args argtype:$value), shiftCountArg), shiftCodeGen> {`.
  **L1219 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`Intrinsic<argtype, !con((args argtype:$value), shiftCountArg), shiftCodeGen> {`。
- **L1220 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [Void];`.
  **L1220 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [Void];`。
- **L1221 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_None;`.
  **L1221 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_None;`。
- **L1222 EN**: Closes the current lexical scope or compound statement.
  **L1222 CN**: 结束当前词法作用域或复合语句块。
- **L1223 EN**: Blank line separating nearby declarations or logic blocks.
  **L1223 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1224 EN**: Comment explains nearby logic, constraints, or intent: `Subclass that includes the machinery to take a 64-bit input apart`.
  **L1224 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Subclass that includes the machinery to take a 64-bit input apart`。

### Lines 1225-1248

````tablegen
// into halves, retrieve the two halves of a shifted output as a pair,
// and glue the pieces of the pair back into an i64 for output.
class LongScalarShift<Type argtype, dag shiftCountArg, dag shiftCodeGen>:
   ScalarShift<argtype, shiftCountArg,
               (seq (u32 (lshr $value, (argtype 32))):$hi,
                    (u32 $value):$lo,
                    shiftCodeGen:$pair,
                    (or (shl (u64 (xval $pair, 1)), (u64 32)),
                             (u64 (xval $pair, 0))))>;

// The family of saturating/rounding scalar shifts that take an
// immediate shift count. They come in matched 32- and 64-bit pairs.
multiclass ScalarSaturatingShiftImm<Type arg32, Type arg64> {
  def "": ScalarShift<arg32, (args imm_1to32:$sh),
                      (IRInt<NAME> $value, $sh)>;
  def l:  LongScalarShift<arg64, (args imm_1to32:$sh),
                          (IRInt<NAME # "l"> $lo, $hi, $sh)>;
}
defm uqshl: ScalarSaturatingShiftImm<u32, u64>;
defm urshr: ScalarSaturatingShiftImm<u32, u64>;
defm sqshl: ScalarSaturatingShiftImm<s32, s64>;
defm srshr: ScalarSaturatingShiftImm<s32, s64>;

// The family of saturating/rounding scalar shifts that take a
````
- **L1225 EN**: Comment explains nearby logic, constraints, or intent: `into halves, retrieve the two halves of a shifted output as a pair,`.
  **L1225 CN**: 注释解释附近代码的逻辑、约束或设计意图：`into halves, retrieve the two halves of a shifted output as a pair,`。
- **L1226 EN**: Comment explains nearby logic, constraints, or intent: `and glue the pieces of the pair back into an i64 for output.`.
  **L1226 CN**: 注释解释附近代码的逻辑、约束或设计意图：`and glue the pieces of the pair back into an i64 for output.`。
- **L1227 EN**: Declares TableGen class record `LongScalarShift`.
  **L1227 CN**: 声明 TableGen class 记录 `LongScalarShift`。
- **L1228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ScalarShift<argtype, shiftCountArg,`.
  **L1228 CN**: 继续一个多行参数列表、初始化器或聚合项：`ScalarShift<argtype, shiftCountArg,`。
- **L1229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (u32 (lshr $value, (argtype 32))):$hi,`.
  **L1229 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (u32 (lshr $value, (argtype 32))):$hi,`。
- **L1230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(u32 $value):$lo,`.
  **L1230 CN**: 继续一个多行参数列表、初始化器或聚合项：`(u32 $value):$lo,`。
- **L1231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `shiftCodeGen:$pair,`.
  **L1231 CN**: 继续一个多行参数列表、初始化器或聚合项：`shiftCodeGen:$pair,`。
- **L1232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(or (shl (u64 (xval $pair, 1)), (u64 32)),`.
  **L1232 CN**: 继续一个多行参数列表、初始化器或聚合项：`(or (shl (u64 (xval $pair, 1)), (u64 32)),`。
- **L1233 EN**: Executes a call or declaration centered on `statement`.
  **L1233 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1234 EN**: Blank line separating nearby declarations or logic blocks.
  **L1234 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1235 EN**: Comment explains nearby logic, constraints, or intent: `The family of saturating/rounding scalar shifts that take an`.
  **L1235 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The family of saturating/rounding scalar shifts that take an`。
- **L1236 EN**: Comment explains nearby logic, constraints, or intent: `immediate shift count. They come in matched 32- and 64-bit pairs.`.
  **L1236 CN**: 注释解释附近代码的逻辑、约束或设计意图：`immediate shift count. They come in matched 32- and 64-bit pairs.`。
- **L1237 EN**: Declares TableGen multiclass record `ScalarSaturatingShiftImm`.
  **L1237 CN**: 声明 TableGen multiclass 记录 `ScalarSaturatingShiftImm`。
- **L1238 EN**: Declares TableGen def record `""`.
  **L1238 CN**: 声明 TableGen def 记录 `""`。
- **L1239 EN**: Executes a call or declaration centered on `statement`.
  **L1239 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1240 EN**: Declares TableGen def record `l`.
  **L1240 CN**: 声明 TableGen def 记录 `l`。
- **L1241 EN**: Executes a call or declaration centered on `statement`.
  **L1241 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1242 EN**: Closes the current lexical scope or compound statement.
  **L1242 CN**: 结束当前词法作用域或复合语句块。
- **L1243 EN**: Declares TableGen defm record `uqshl`.
  **L1243 CN**: 声明 TableGen defm 记录 `uqshl`。
- **L1244 EN**: Declares TableGen defm record `urshr`.
  **L1244 CN**: 声明 TableGen defm 记录 `urshr`。
- **L1245 EN**: Declares TableGen defm record `sqshl`.
  **L1245 CN**: 声明 TableGen defm 记录 `sqshl`。
- **L1246 EN**: Declares TableGen defm record `srshr`.
  **L1246 CN**: 声明 TableGen defm 记录 `srshr`。
- **L1247 EN**: Blank line separating nearby declarations or logic blocks.
  **L1247 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1248 EN**: Comment explains nearby logic, constraints, or intent: `The family of saturating/rounding scalar shifts that take a`.
  **L1248 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The family of saturating/rounding scalar shifts that take a`。

### Lines 1249-1272

````tablegen
// register shift count. They also have 32- and 64-bit forms, but the
// 64-bit form also has a version that saturates to 48 bits, so the IR
// intrinsic takes an extra saturation-type operand.
multiclass ScalarSaturatingShiftReg<Type arg32, Type arg64> {
  def "":          ScalarShift<arg32, (args s32:$sh),
                               (IRInt<NAME> $value, $sh)>;
  def l:       LongScalarShift<arg64, (args s32:$sh),
                               (IRInt<NAME # "l"> $lo, $hi, $sh, 64)>;
  def l_sat48: LongScalarShift<arg64, (args s32:$sh),
                               (IRInt<NAME # "l"> $lo, $hi, $sh, 48)>;
}
defm uqrshl: ScalarSaturatingShiftReg<u32, u64>;
defm sqrshr: ScalarSaturatingShiftReg<s32, s64>;

// The intrinsics for LSLL and ASRL come in 64-bit versions only, with
// no saturation count.
def lsll: LongScalarShift<u64, (args s32:$sh), (IRInt<"lsll"> $lo, $hi, $sh)>;
def asrl: LongScalarShift<s64, (args s32:$sh), (IRInt<"asrl"> $lo, $hi, $sh)>;

multiclass vadcsbc<dag initial_carry_in> {
  def q: Intrinsic<Vector, (args Vector:$a, Vector:$b, Ptr<uint>:$carry),
      (seq (IRInt<NAME, [Vector]> $a, $b, (shl (load $carry), 29)):$pair,
           (store (and 1, (lshr (xval $pair, 1), 29)), $carry),
           (xval $pair, 0))>;
````
- **L1249 EN**: Comment explains nearby logic, constraints, or intent: `register shift count. They also have 32- and 64-bit forms, but the`.
  **L1249 CN**: 注释解释附近代码的逻辑、约束或设计意图：`register shift count. They also have 32- and 64-bit forms, but the`。
- **L1250 EN**: Comment explains nearby logic, constraints, or intent: `64-bit form also has a version that saturates to 48 bits, so the IR`.
  **L1250 CN**: 注释解释附近代码的逻辑、约束或设计意图：`64-bit form also has a version that saturates to 48 bits, so the IR`。
- **L1251 EN**: Comment explains nearby logic, constraints, or intent: `intrinsic takes an extra saturation-type operand.`.
  **L1251 CN**: 注释解释附近代码的逻辑、约束或设计意图：`intrinsic takes an extra saturation-type operand.`。
- **L1252 EN**: Declares TableGen multiclass record `ScalarSaturatingShiftReg`.
  **L1252 CN**: 声明 TableGen multiclass 记录 `ScalarSaturatingShiftReg`。
- **L1253 EN**: Declares TableGen def record `""`.
  **L1253 CN**: 声明 TableGen def 记录 `""`。
- **L1254 EN**: Executes a call or declaration centered on `statement`.
  **L1254 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1255 EN**: Declares TableGen def record `l`.
  **L1255 CN**: 声明 TableGen def 记录 `l`。
- **L1256 EN**: Executes a call or declaration centered on `statement`.
  **L1256 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1257 EN**: Declares TableGen def record `l_sat48`.
  **L1257 CN**: 声明 TableGen def 记录 `l_sat48`。
- **L1258 EN**: Executes a call or declaration centered on `statement`.
  **L1258 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1259 EN**: Closes the current lexical scope or compound statement.
  **L1259 CN**: 结束当前词法作用域或复合语句块。
- **L1260 EN**: Declares TableGen defm record `uqrshl`.
  **L1260 CN**: 声明 TableGen defm 记录 `uqrshl`。
- **L1261 EN**: Declares TableGen defm record `sqrshr`.
  **L1261 CN**: 声明 TableGen defm 记录 `sqrshr`。
- **L1262 EN**: Blank line separating nearby declarations or logic blocks.
  **L1262 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1263 EN**: Comment explains nearby logic, constraints, or intent: `The intrinsics for LSLL and ASRL come in 64-bit versions only, with`.
  **L1263 CN**: 注释解释附近代码的逻辑、约束或设计意图：`The intrinsics for LSLL and ASRL come in 64-bit versions only, with`。
- **L1264 EN**: Comment explains nearby logic, constraints, or intent: `no saturation count.`.
  **L1264 CN**: 注释解释附近代码的逻辑、约束或设计意图：`no saturation count.`。
- **L1265 EN**: Declares TableGen def record `lsll`.
  **L1265 CN**: 声明 TableGen def 记录 `lsll`。
- **L1266 EN**: Declares TableGen def record `asrl`.
  **L1266 CN**: 声明 TableGen def 记录 `asrl`。
- **L1267 EN**: Blank line separating nearby declarations or logic blocks.
  **L1267 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1268 EN**: Declares TableGen multiclass record `vadcsbc`.
  **L1268 CN**: 声明 TableGen multiclass 记录 `vadcsbc`。
- **L1269 EN**: Declares TableGen def record `q`.
  **L1269 CN**: 声明 TableGen def 记录 `q`。
- **L1270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (IRInt<NAME, [Vector]> $a, $b, (shl (load $carry), 29)):$pair,`.
  **L1270 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (IRInt<NAME, [Vector]> $a, $b, (shl (load $carry), 29)):$pair,`。
- **L1271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`.
  **L1271 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`。
- **L1272 EN**: Executes a call or declaration centered on `statement`.
  **L1272 CN**: 执行以 `statement` 为核心的调用或声明。

### Lines 1273-1296

````tablegen
  def iq: Intrinsic<Vector, (args Vector:$a, Vector:$b, Ptr<uint>:$carry),
      (seq (IRInt<NAME, [Vector]> $a, $b, initial_carry_in):$pair,
           (store (and 1, (lshr (xval $pair, 1), 29)), $carry),
           (xval $pair, 0))>;
  def q_m: Intrinsic<Vector, (args Vector:$inactive, Vector:$a, Vector:$b,
                                   Ptr<uint>:$carry, Predicate:$pred),
      (seq (IRInt<NAME # "_predicated", [Vector, Predicate]> $inactive, $a, $b,
               (shl (load $carry), 29), $pred):$pair,
           (store (and 1, (lshr (xval $pair, 1), 29)), $carry),
           (xval $pair, 0))>;
  def iq_m: Intrinsic<Vector, (args Vector:$inactive, Vector:$a, Vector:$b,
                                    Ptr<uint>:$carry, Predicate:$pred),
      (seq (IRInt<NAME # "_predicated", [Vector, Predicate]> $inactive, $a, $b,
               initial_carry_in, $pred):$pair,
           (store (and 1, (lshr (xval $pair, 1), 29)), $carry),
           (xval $pair, 0))>;
}
let params = T.Int32 in {
  defm vadc: vadcsbc<(u32 0)>;
  defm vsbc: vadcsbc<(shl 1, 29)>;
}

let params = T.Int in {
  def vshlcq: Intrinsic<
````
- **L1273 EN**: Declares TableGen def record `iq`.
  **L1273 CN**: 声明 TableGen def 记录 `iq`。
- **L1274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (IRInt<NAME, [Vector]> $a, $b, initial_carry_in):$pair,`.
  **L1274 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (IRInt<NAME, [Vector]> $a, $b, initial_carry_in):$pair,`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`。
- **L1276 EN**: Executes a call or declaration centered on `statement`.
  **L1276 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1277 EN**: Declares TableGen def record `q_m`.
  **L1277 CN**: 声明 TableGen def 记录 `q_m`。
- **L1278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ptr<uint>:$carry, Predicate:$pred),`.
  **L1278 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ptr<uint>:$carry, Predicate:$pred),`。
- **L1279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (IRInt<NAME # "_predicated", [Vector, Predicate]> $inactive, $a, $b,`.
  **L1279 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (IRInt<NAME # "_predicated", [Vector, Predicate]> $inactive, $a, $b,`。
- **L1280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(shl (load $carry), 29), $pred):$pair,`.
  **L1280 CN**: 继续一个多行参数列表、初始化器或聚合项：`(shl (load $carry), 29), $pred):$pair,`。
- **L1281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`.
  **L1281 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`。
- **L1282 EN**: Executes a call or declaration centered on `statement`.
  **L1282 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1283 EN**: Declares TableGen def record `iq_m`.
  **L1283 CN**: 声明 TableGen def 记录 `iq_m`。
- **L1284 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Ptr<uint>:$carry, Predicate:$pred),`.
  **L1284 CN**: 继续一个多行参数列表、初始化器或聚合项：`Ptr<uint>:$carry, Predicate:$pred),`。
- **L1285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (IRInt<NAME # "_predicated", [Vector, Predicate]> $inactive, $a, $b,`.
  **L1285 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (IRInt<NAME # "_predicated", [Vector, Predicate]> $inactive, $a, $b,`。
- **L1286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `initial_carry_in, $pred):$pair,`.
  **L1286 CN**: 继续一个多行参数列表、初始化器或聚合项：`initial_carry_in, $pred):$pair,`。
- **L1287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`.
  **L1287 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (and 1, (lshr (xval $pair, 1), 29)), $carry),`。
- **L1288 EN**: Executes a call or declaration centered on `statement`.
  **L1288 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1289 EN**: Closes the current lexical scope or compound statement.
  **L1289 CN**: 结束当前词法作用域或复合语句块。
- **L1290 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int32 in {`.
  **L1290 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int32 in {`。
- **L1291 EN**: Declares TableGen defm record `vadc`.
  **L1291 CN**: 声明 TableGen defm 记录 `vadc`。
- **L1292 EN**: Declares TableGen defm record `vsbc`.
  **L1292 CN**: 声明 TableGen defm 记录 `vsbc`。
- **L1293 EN**: Closes the current lexical scope or compound statement.
  **L1293 CN**: 结束当前词法作用域或复合语句块。
- **L1294 EN**: Blank line separating nearby declarations or logic blocks.
  **L1294 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1295 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L1295 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L1296 EN**: Declares TableGen def record `vshlcq`.
  **L1296 CN**: 声明 TableGen def 记录 `vshlcq`。

### Lines 1297-1320

````tablegen
    Vector, (args Vector:$v, Ptr<u32>:$ps, imm_1to32:$imm),
    (seq (load $ps):$s,
         (IRInt<"vshlc", [Vector]> $v, $s, $imm):$pair,
         (store (xval $pair, 0), $ps),
         (xval $pair, 1))>;
  def vshlcq_m: Intrinsic<
    Vector, (args Vector:$v, Ptr<u32>:$ps, imm_1to32:$imm, Predicate:$pred),
    (seq (load $ps):$s,
         (IRInt<"vshlc_predicated", [Vector, Predicate]>
              $v, $s, $imm, $pred):$pair,
         (store (xval $pair, 0), $ps),
         (xval $pair, 1))>;
}

multiclass VectorComplexAddPred<dag not_halving, dag angle> {
  def "" : Intrinsic<Vector, (args Vector:$a, Vector:$b),
     (IRInt<"vcaddq", [Vector]> not_halving, angle, $a, $b)>;
  defm "" : IntrinsicMX<Vector, (args Vector:$a, Vector:$b, Predicate:$pred),
     (IRInt<"vcaddq_predicated", [Vector, Predicate]>
       not_halving, angle, $inactive, $a, $b, $pred)>;
}

multiclass VectorComplexMulPred<dag angle> {
  def "" : Intrinsic<Vector, (args Vector:$a, Vector:$b),
````
- **L1297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$v, Ptr<u32>:$ps, imm_1to32:$imm),`.
  **L1297 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$v, Ptr<u32>:$ps, imm_1to32:$imm),`。
- **L1298 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (load $ps):$s,`.
  **L1298 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (load $ps):$s,`。
- **L1299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vshlc", [Vector]> $v, $s, $imm):$pair,`.
  **L1299 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vshlc", [Vector]> $v, $s, $imm):$pair,`。
- **L1300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (xval $pair, 0), $ps),`.
  **L1300 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (xval $pair, 0), $ps),`。
- **L1301 EN**: Executes a call or declaration centered on `statement`.
  **L1301 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1302 EN**: Declares TableGen def record `vshlcq_m`.
  **L1302 CN**: 声明 TableGen def 记录 `vshlcq_m`。
- **L1303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Vector, (args Vector:$v, Ptr<u32>:$ps, imm_1to32:$imm, Predicate:$pred),`.
  **L1303 CN**: 继续一个多行参数列表、初始化器或聚合项：`Vector, (args Vector:$v, Ptr<u32>:$ps, imm_1to32:$imm, Predicate:$pred),`。
- **L1304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (load $ps):$s,`.
  **L1304 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (load $ps):$s,`。
- **L1305 EN**: Continues the surrounding expression or declaration: `(IRInt<"vshlc_predicated", [Vector, Predicate]>`.
  **L1305 CN**: 继续构造周围的表达式或声明：`(IRInt<"vshlc_predicated", [Vector, Predicate]>`。
- **L1306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$v, $s, $imm, $pred):$pair,`.
  **L1306 CN**: 继续一个多行参数列表、初始化器或聚合项：`$v, $s, $imm, $pred):$pair,`。
- **L1307 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(store (xval $pair, 0), $ps),`.
  **L1307 CN**: 继续一个多行参数列表、初始化器或聚合项：`(store (xval $pair, 0), $ps),`。
- **L1308 EN**: Executes a call or declaration centered on `statement`.
  **L1308 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1309 EN**: Closes the current lexical scope or compound statement.
  **L1309 CN**: 结束当前词法作用域或复合语句块。
- **L1310 EN**: Blank line separating nearby declarations or logic blocks.
  **L1310 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1311 EN**: Declares TableGen multiclass record `VectorComplexAddPred`.
  **L1311 CN**: 声明 TableGen multiclass 记录 `VectorComplexAddPred`。
- **L1312 EN**: Declares TableGen def record `""`.
  **L1312 CN**: 声明 TableGen def 记录 `""`。
- **L1313 EN**: Executes a call or declaration centered on `statement`.
  **L1313 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1314 EN**: Declares TableGen defm record `""`.
  **L1314 CN**: 声明 TableGen defm 记录 `""`。
- **L1315 EN**: Continues the surrounding expression or declaration: `(IRInt<"vcaddq_predicated", [Vector, Predicate]>`.
  **L1315 CN**: 继续构造周围的表达式或声明：`(IRInt<"vcaddq_predicated", [Vector, Predicate]>`。
- **L1316 EN**: Adds a standalone statement or declaration: `not_halving, angle, $inactive, $a, $b, $pred)>;`.
  **L1316 CN**: 添加一条独立语句或声明：`not_halving, angle, $inactive, $a, $b, $pred)>;`。
- **L1317 EN**: Closes the current lexical scope or compound statement.
  **L1317 CN**: 结束当前词法作用域或复合语句块。
- **L1318 EN**: Blank line separating nearby declarations or logic blocks.
  **L1318 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1319 EN**: Declares TableGen multiclass record `VectorComplexMulPred`.
  **L1319 CN**: 声明 TableGen multiclass 记录 `VectorComplexMulPred`。
- **L1320 EN**: Declares TableGen def record `""`.
  **L1320 CN**: 声明 TableGen def 记录 `""`。

### Lines 1321-1344

````tablegen
    (IRInt<"vcmulq", [Vector]> angle, $a, $b)>;
  defm "" : IntrinsicMX<Vector, (args Vector:$a, Vector:$b, Predicate:$pred),
    (IRInt<"vcmulq_predicated", [Vector, Predicate]> angle, $inactive, $a, $b,
      $pred)>;
}

multiclass VectorComplexMLAPred<dag angle> {
  def "" : Intrinsic<Vector, (args Vector:$a, Vector:$b, Vector:$c),
    (IRInt<"vcmlaq", [Vector]> angle, $a, $b, $c)>;
  def _m : Intrinsic<Vector, (args Vector:$a, Vector:$b, Vector:$c,
                                   Predicate:$pred),
    (IRInt<"vcmlaq_predicated", [Vector, Predicate]> angle, $a, $b, $c, $pred)>;
}

multiclass VectorComplexAddAngle<dag not_halving> {
  defm _rot90 : VectorComplexAddPred<not_halving, (u32 0)>;
  defm _rot270 : VectorComplexAddPred<not_halving, (u32 1)>;
}

multiclass VectorComplexMulAngle {
  defm ""      : VectorComplexMulPred<(u32 0)>;
  defm _rot90  : VectorComplexMulPred<(u32 1)>;
  defm _rot180 : VectorComplexMulPred<(u32 2)>;
  defm _rot270 : VectorComplexMulPred<(u32 3)>;
````
- **L1321 EN**: Executes a call or declaration centered on `statement`.
  **L1321 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1322 EN**: Declares TableGen defm record `""`.
  **L1322 CN**: 声明 TableGen defm 记录 `""`。
- **L1323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(IRInt<"vcmulq_predicated", [Vector, Predicate]> angle, $inactive, $a, $b,`.
  **L1323 CN**: 继续一个多行参数列表、初始化器或聚合项：`(IRInt<"vcmulq_predicated", [Vector, Predicate]> angle, $inactive, $a, $b,`。
- **L1324 EN**: Adds a standalone statement or declaration: `$pred)>;`.
  **L1324 CN**: 添加一条独立语句或声明：`$pred)>;`。
- **L1325 EN**: Closes the current lexical scope or compound statement.
  **L1325 CN**: 结束当前词法作用域或复合语句块。
- **L1326 EN**: Blank line separating nearby declarations or logic blocks.
  **L1326 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1327 EN**: Declares TableGen multiclass record `VectorComplexMLAPred`.
  **L1327 CN**: 声明 TableGen multiclass 记录 `VectorComplexMLAPred`。
- **L1328 EN**: Declares TableGen def record `""`.
  **L1328 CN**: 声明 TableGen def 记录 `""`。
- **L1329 EN**: Executes a call or declaration centered on `statement`.
  **L1329 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1330 EN**: Declares TableGen def record `_m`.
  **L1330 CN**: 声明 TableGen def 记录 `_m`。
- **L1331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1331 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1332 EN**: Executes a call or declaration centered on `statement`.
  **L1332 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1333 EN**: Closes the current lexical scope or compound statement.
  **L1333 CN**: 结束当前词法作用域或复合语句块。
- **L1334 EN**: Blank line separating nearby declarations or logic blocks.
  **L1334 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1335 EN**: Declares TableGen multiclass record `VectorComplexAddAngle`.
  **L1335 CN**: 声明 TableGen multiclass 记录 `VectorComplexAddAngle`。
- **L1336 EN**: Declares TableGen defm record `_rot90`.
  **L1336 CN**: 声明 TableGen defm 记录 `_rot90`。
- **L1337 EN**: Declares TableGen defm record `_rot270`.
  **L1337 CN**: 声明 TableGen defm 记录 `_rot270`。
- **L1338 EN**: Closes the current lexical scope or compound statement.
  **L1338 CN**: 结束当前词法作用域或复合语句块。
- **L1339 EN**: Blank line separating nearby declarations or logic blocks.
  **L1339 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1340 EN**: Declares TableGen multiclass record `VectorComplexMulAngle`.
  **L1340 CN**: 声明 TableGen multiclass 记录 `VectorComplexMulAngle`。
- **L1341 EN**: Declares TableGen defm record `""`.
  **L1341 CN**: 声明 TableGen defm 记录 `""`。
- **L1342 EN**: Declares TableGen defm record `_rot90`.
  **L1342 CN**: 声明 TableGen defm 记录 `_rot90`。
- **L1343 EN**: Declares TableGen defm record `_rot180`.
  **L1343 CN**: 声明 TableGen defm 记录 `_rot180`。
- **L1344 EN**: Declares TableGen defm record `_rot270`.
  **L1344 CN**: 声明 TableGen defm 记录 `_rot270`。

### Lines 1345-1368

````tablegen
}

multiclass VectorComplexMLAAngle {
  defm ""      : VectorComplexMLAPred<(u32 0)>;
  defm _rot90  : VectorComplexMLAPred<(u32 1)>;
  defm _rot180 : VectorComplexMLAPred<(u32 2)>;
  defm _rot270 : VectorComplexMLAPred<(u32 3)>;
}

let params = T.Usual in
defm vcaddq : VectorComplexAddAngle<(u32 1)>;

let params = T.Signed in
defm vhcaddq : VectorComplexAddAngle<(u32 0)>;

let params = T.Float in {
defm vcmulq : VectorComplexMulAngle;
defm vcmlaq : VectorComplexMLAAngle;
}

multiclass MVEBinaryVectorHoriz32<dag subtract, dag exchange, string xsuffix> {
  def xsuffix#"q"
    : Intrinsic<Scalar32, (args Vector:$a, Vector:$b),
                          (IRInt<"vmldava", [Vector]>
````
- **L1345 EN**: Closes the current lexical scope or compound statement.
  **L1345 CN**: 结束当前词法作用域或复合语句块。
- **L1346 EN**: Blank line separating nearby declarations or logic blocks.
  **L1346 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1347 EN**: Declares TableGen multiclass record `VectorComplexMLAAngle`.
  **L1347 CN**: 声明 TableGen multiclass 记录 `VectorComplexMLAAngle`。
- **L1348 EN**: Declares TableGen defm record `""`.
  **L1348 CN**: 声明 TableGen defm 记录 `""`。
- **L1349 EN**: Declares TableGen defm record `_rot90`.
  **L1349 CN**: 声明 TableGen defm 记录 `_rot90`。
- **L1350 EN**: Declares TableGen defm record `_rot180`.
  **L1350 CN**: 声明 TableGen defm 记录 `_rot180`。
- **L1351 EN**: Declares TableGen defm record `_rot270`.
  **L1351 CN**: 声明 TableGen defm 记录 `_rot270`。
- **L1352 EN**: Closes the current lexical scope or compound statement.
  **L1352 CN**: 结束当前词法作用域或复合语句块。
- **L1353 EN**: Blank line separating nearby declarations or logic blocks.
  **L1353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1354 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Usual in`.
  **L1354 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Usual in`。
- **L1355 EN**: Declares TableGen defm record `vcaddq`.
  **L1355 CN**: 声明 TableGen defm 记录 `vcaddq`。
- **L1356 EN**: Blank line separating nearby declarations or logic blocks.
  **L1356 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1357 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in`.
  **L1357 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in`。
- **L1358 EN**: Declares TableGen defm record `vhcaddq`.
  **L1358 CN**: 声明 TableGen defm 记录 `vhcaddq`。
- **L1359 EN**: Blank line separating nearby declarations or logic blocks.
  **L1359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1360 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Float in {`.
  **L1360 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Float in {`。
- **L1361 EN**: Declares TableGen defm record `vcmulq`.
  **L1361 CN**: 声明 TableGen defm 记录 `vcmulq`。
- **L1362 EN**: Declares TableGen defm record `vcmlaq`.
  **L1362 CN**: 声明 TableGen defm 记录 `vcmlaq`。
- **L1363 EN**: Closes the current lexical scope or compound statement.
  **L1363 CN**: 结束当前词法作用域或复合语句块。
- **L1364 EN**: Blank line separating nearby declarations or logic blocks.
  **L1364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1365 EN**: Declares TableGen multiclass record `MVEBinaryVectorHoriz32`.
  **L1365 CN**: 声明 TableGen multiclass 记录 `MVEBinaryVectorHoriz32`。
- **L1366 EN**: Declares TableGen def record `xsuffix#"q"`.
  **L1366 CN**: 声明 TableGen def 记录 `xsuffix#"q"`。
- **L1367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<Scalar32, (args Vector:$a, Vector:$b),`.
  **L1367 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<Scalar32, (args Vector:$a, Vector:$b),`。
- **L1368 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmldava", [Vector]>`.
  **L1368 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmldava", [Vector]>`。

### Lines 1369-1392

````tablegen
                           (unsignedflag Scalar), subtract, exchange,
                           (zeroinit Scalar32), $a, $b)>;
  def xsuffix#"q_p"
    : Intrinsic<Scalar32, (args Vector:$a, Vector:$b, Predicate:$pred),
                          (IRInt<"vmldava_predicated", [Vector, Predicate]>
                           (unsignedflag Scalar), subtract, exchange,
                           (zeroinit Scalar32), $a, $b, $pred)>;

  def "a"#xsuffix#"q"
    : Intrinsic<Scalar32, (args Scalar32:$a, Vector:$b, Vector:$c),
                          (IRInt<"vmldava", [Vector]>
                           (unsignedflag Scalar), subtract, exchange,
                           $a, $b, $c)>;
  def "a"#xsuffix#"q_p"
    : Intrinsic<Scalar32, (args Scalar32:$a, Vector:$b, Vector:$c,
                                Predicate:$pred),
                          (IRInt<"vmldava_predicated", [Vector, Predicate]>
                           (unsignedflag Scalar), subtract, exchange,
                           $a, $b, $c, $pred)>;
}

class IntrSplit64<Type resty, dag args, dag codegen>
  : Intrinsic<resty, args,
              (seq (u32 (lshr $a, (u64 32))):$hi,
````
- **L1369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1369 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1370 EN**: Executes a call or declaration centered on `statement`.
  **L1370 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1371 EN**: Declares TableGen def record `xsuffix#"q_p"`.
  **L1371 CN**: 声明 TableGen def 记录 `xsuffix#"q_p"`。
- **L1372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<Scalar32, (args Vector:$a, Vector:$b, Predicate:$pred),`.
  **L1372 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<Scalar32, (args Vector:$a, Vector:$b, Predicate:$pred),`。
- **L1373 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmldava_predicated", [Vector, Predicate]>`.
  **L1373 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmldava_predicated", [Vector, Predicate]>`。
- **L1374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1374 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1375 EN**: Executes a call or declaration centered on `statement`.
  **L1375 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1376 EN**: Blank line separating nearby declarations or logic blocks.
  **L1376 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1377 EN**: Declares TableGen def record `"a"#xsuffix#"q"`.
  **L1377 CN**: 声明 TableGen def 记录 `"a"#xsuffix#"q"`。
- **L1378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<Scalar32, (args Scalar32:$a, Vector:$b, Vector:$c),`.
  **L1378 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<Scalar32, (args Scalar32:$a, Vector:$b, Vector:$c),`。
- **L1379 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmldava", [Vector]>`.
  **L1379 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmldava", [Vector]>`。
- **L1380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1380 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1381 EN**: Adds a standalone statement or declaration: `$a, $b, $c)>;`.
  **L1381 CN**: 添加一条独立语句或声明：`$a, $b, $c)>;`。
- **L1382 EN**: Declares TableGen def record `"a"#xsuffix#"q_p"`.
  **L1382 CN**: 声明 TableGen def 记录 `"a"#xsuffix#"q_p"`。
- **L1383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<Scalar32, (args Scalar32:$a, Vector:$b, Vector:$c,`.
  **L1383 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<Scalar32, (args Scalar32:$a, Vector:$b, Vector:$c,`。
- **L1384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1384 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1385 EN**: Continues the surrounding expression or declaration: `(IRInt<"vmldava_predicated", [Vector, Predicate]>`.
  **L1385 CN**: 继续构造周围的表达式或声明：`(IRInt<"vmldava_predicated", [Vector, Predicate]>`。
- **L1386 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1386 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1387 EN**: Adds a standalone statement or declaration: `$a, $b, $c, $pred)>;`.
  **L1387 CN**: 添加一条独立语句或声明：`$a, $b, $c, $pred)>;`。
- **L1388 EN**: Closes the current lexical scope or compound statement.
  **L1388 CN**: 结束当前词法作用域或复合语句块。
- **L1389 EN**: Blank line separating nearby declarations or logic blocks.
  **L1389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1390 EN**: Declares TableGen class record `IntrSplit64`.
  **L1390 CN**: 声明 TableGen class 记录 `IntrSplit64`。
- **L1391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<resty, args,`.
  **L1391 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<resty, args,`。
- **L1392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (u32 (lshr $a, (u64 32))):$hi,`.
  **L1392 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (u32 (lshr $a, (u64 32))):$hi,`。

### Lines 1393-1416

````tablegen
                   (u32 $a):$lo,
                   codegen:$pair,
                   (or (shl (u64 (xval $pair, 1)), (u64 32)),
                            (u64 (xval $pair, 0))))>;

class IntrSplit64ZeroInit<Type resty, dag args, dag codegen>
  : Intrinsic<resty, args,
              (seq (zeroinit u32):$hi,
                   (zeroinit u32):$lo,
                   codegen:$pair,
                   (or (shl (u64 (xval $pair, 1)), (u64 32)),
                            (u64 (xval $pair, 0))))>;

multiclass MVEBinaryVectorHoriz64Base<dag subtract, dag exchange,
                                      string xsuffix, string irname> {
  def xsuffix#"q"
    : IntrSplit64ZeroInit<Scalar64, (args Vector:$a, Vector:$b),
                          (IRInt<irname, [Vector]>
                           (unsignedflag Scalar), subtract, exchange,
                           $lo, $hi, $a, $b)>;
  def xsuffix#"q_p"
    : IntrSplit64ZeroInit<Scalar64, (args Vector:$a, Vector:$b,
                                          Predicate:$pred),
                          (IRInt<irname#"_predicated", [Vector, Predicate]>
````
- **L1393 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(u32 $a):$lo,`.
  **L1393 CN**: 继续一个多行参数列表、初始化器或聚合项：`(u32 $a):$lo,`。
- **L1394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `codegen:$pair,`.
  **L1394 CN**: 继续一个多行参数列表、初始化器或聚合项：`codegen:$pair,`。
- **L1395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(or (shl (u64 (xval $pair, 1)), (u64 32)),`.
  **L1395 CN**: 继续一个多行参数列表、初始化器或聚合项：`(or (shl (u64 (xval $pair, 1)), (u64 32)),`。
- **L1396 EN**: Executes a call or declaration centered on `statement`.
  **L1396 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1397 EN**: Blank line separating nearby declarations or logic blocks.
  **L1397 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1398 EN**: Declares TableGen class record `IntrSplit64ZeroInit`.
  **L1398 CN**: 声明 TableGen class 记录 `IntrSplit64ZeroInit`。
- **L1399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<resty, args,`.
  **L1399 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<resty, args,`。
- **L1400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(seq (zeroinit u32):$hi,`.
  **L1400 CN**: 继续一个多行参数列表、初始化器或聚合项：`(seq (zeroinit u32):$hi,`。
- **L1401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(zeroinit u32):$lo,`.
  **L1401 CN**: 继续一个多行参数列表、初始化器或聚合项：`(zeroinit u32):$lo,`。
- **L1402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `codegen:$pair,`.
  **L1402 CN**: 继续一个多行参数列表、初始化器或聚合项：`codegen:$pair,`。
- **L1403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(or (shl (u64 (xval $pair, 1)), (u64 32)),`.
  **L1403 CN**: 继续一个多行参数列表、初始化器或聚合项：`(or (shl (u64 (xval $pair, 1)), (u64 32)),`。
- **L1404 EN**: Executes a call or declaration centered on `statement`.
  **L1404 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1405 EN**: Blank line separating nearby declarations or logic blocks.
  **L1405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1406 EN**: Declares TableGen multiclass record `MVEBinaryVectorHoriz64Base`.
  **L1406 CN**: 声明 TableGen multiclass 记录 `MVEBinaryVectorHoriz64Base`。
- **L1407 EN**: Continues the surrounding expression or declaration: `string xsuffix, string irname> {`.
  **L1407 CN**: 继续构造周围的表达式或声明：`string xsuffix, string irname> {`。
- **L1408 EN**: Declares TableGen def record `xsuffix#"q"`.
  **L1408 CN**: 声明 TableGen def 记录 `xsuffix#"q"`。
- **L1409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: IntrSplit64ZeroInit<Scalar64, (args Vector:$a, Vector:$b),`.
  **L1409 CN**: 继续一个多行参数列表、初始化器或聚合项：`: IntrSplit64ZeroInit<Scalar64, (args Vector:$a, Vector:$b),`。
- **L1410 EN**: Continues the surrounding expression or declaration: `(IRInt<irname, [Vector]>`.
  **L1410 CN**: 继续构造周围的表达式或声明：`(IRInt<irname, [Vector]>`。
- **L1411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1411 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1412 EN**: Adds a standalone statement or declaration: `$lo, $hi, $a, $b)>;`.
  **L1412 CN**: 添加一条独立语句或声明：`$lo, $hi, $a, $b)>;`。
- **L1413 EN**: Declares TableGen def record `xsuffix#"q_p"`.
  **L1413 CN**: 声明 TableGen def 记录 `xsuffix#"q_p"`。
- **L1414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: IntrSplit64ZeroInit<Scalar64, (args Vector:$a, Vector:$b,`.
  **L1414 CN**: 继续一个多行参数列表、初始化器或聚合项：`: IntrSplit64ZeroInit<Scalar64, (args Vector:$a, Vector:$b,`。
- **L1415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1415 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1416 EN**: Continues the surrounding expression or declaration: `(IRInt<irname#"_predicated", [Vector, Predicate]>`.
  **L1416 CN**: 继续构造周围的表达式或声明：`(IRInt<irname#"_predicated", [Vector, Predicate]>`。

### Lines 1417-1440

````tablegen
                           (unsignedflag Scalar), subtract, exchange,
                           $lo, $hi, $a, $b, $pred)>;

  def "a"#xsuffix#"q"
    : IntrSplit64<Scalar64, (args Scalar64:$a, Vector:$b, Vector:$c),
                          (IRInt<irname, [Vector]>
                           (unsignedflag Scalar), subtract, exchange,
                           $lo, $hi, $b, $c)>;
  def "a"#xsuffix#"q_p"
    : IntrSplit64<Scalar64, (args Scalar64:$a, Vector:$b, Vector:$c,
                                  Predicate:$pred),
                          (IRInt<irname#"_predicated", [Vector, Predicate]>
                           (unsignedflag Scalar), subtract, exchange,
                           $lo, $hi, $b, $c, $pred)>;
}

multiclass MVEBinaryVectorHoriz64<dag subtract, dag exchange, string xsuffix> {
  defm "" : MVEBinaryVectorHoriz64Base<subtract, exchange, xsuffix, "vmlldava">;
}

multiclass MVEBinaryVectorHoriz64R<dag subtract, dag exchange, string xsuffix> {
  defm "" : MVEBinaryVectorHoriz64Base<subtract, exchange, xsuffix,
                                       "vrmlldavha">;
}
````
- **L1417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1417 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1418 EN**: Adds a standalone statement or declaration: `$lo, $hi, $a, $b, $pred)>;`.
  **L1418 CN**: 添加一条独立语句或声明：`$lo, $hi, $a, $b, $pred)>;`。
- **L1419 EN**: Blank line separating nearby declarations or logic blocks.
  **L1419 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1420 EN**: Declares TableGen def record `"a"#xsuffix#"q"`.
  **L1420 CN**: 声明 TableGen def 记录 `"a"#xsuffix#"q"`。
- **L1421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: IntrSplit64<Scalar64, (args Scalar64:$a, Vector:$b, Vector:$c),`.
  **L1421 CN**: 继续一个多行参数列表、初始化器或聚合项：`: IntrSplit64<Scalar64, (args Scalar64:$a, Vector:$b, Vector:$c),`。
- **L1422 EN**: Continues the surrounding expression or declaration: `(IRInt<irname, [Vector]>`.
  **L1422 CN**: 继续构造周围的表达式或声明：`(IRInt<irname, [Vector]>`。
- **L1423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1423 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1424 EN**: Adds a standalone statement or declaration: `$lo, $hi, $b, $c)>;`.
  **L1424 CN**: 添加一条独立语句或声明：`$lo, $hi, $b, $c)>;`。
- **L1425 EN**: Declares TableGen def record `"a"#xsuffix#"q_p"`.
  **L1425 CN**: 声明 TableGen def 记录 `"a"#xsuffix#"q_p"`。
- **L1426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: IntrSplit64<Scalar64, (args Scalar64:$a, Vector:$b, Vector:$c,`.
  **L1426 CN**: 继续一个多行参数列表、初始化器或聚合项：`: IntrSplit64<Scalar64, (args Scalar64:$a, Vector:$b, Vector:$c,`。
- **L1427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1427 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1428 EN**: Continues the surrounding expression or declaration: `(IRInt<irname#"_predicated", [Vector, Predicate]>`.
  **L1428 CN**: 继续构造周围的表达式或声明：`(IRInt<irname#"_predicated", [Vector, Predicate]>`。
- **L1429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(unsignedflag Scalar), subtract, exchange,`.
  **L1429 CN**: 继续一个多行参数列表、初始化器或聚合项：`(unsignedflag Scalar), subtract, exchange,`。
- **L1430 EN**: Adds a standalone statement or declaration: `$lo, $hi, $b, $c, $pred)>;`.
  **L1430 CN**: 添加一条独立语句或声明：`$lo, $hi, $b, $c, $pred)>;`。
- **L1431 EN**: Closes the current lexical scope or compound statement.
  **L1431 CN**: 结束当前词法作用域或复合语句块。
- **L1432 EN**: Blank line separating nearby declarations or logic blocks.
  **L1432 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1433 EN**: Declares TableGen multiclass record `MVEBinaryVectorHoriz64`.
  **L1433 CN**: 声明 TableGen multiclass 记录 `MVEBinaryVectorHoriz64`。
- **L1434 EN**: Declares TableGen defm record `""`.
  **L1434 CN**: 声明 TableGen defm 记录 `""`。
- **L1435 EN**: Closes the current lexical scope or compound statement.
  **L1435 CN**: 结束当前词法作用域或复合语句块。
- **L1436 EN**: Blank line separating nearby declarations or logic blocks.
  **L1436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1437 EN**: Declares TableGen multiclass record `MVEBinaryVectorHoriz64R`.
  **L1437 CN**: 声明 TableGen multiclass 记录 `MVEBinaryVectorHoriz64R`。
- **L1438 EN**: Declares TableGen defm record `""`.
  **L1438 CN**: 声明 TableGen defm 记录 `""`。
- **L1439 EN**: Adds a standalone statement or declaration: `"vrmlldavha">;`.
  **L1439 CN**: 添加一条独立语句或声明：`"vrmlldavha">;`。
- **L1440 EN**: Closes the current lexical scope or compound statement.
  **L1440 CN**: 结束当前词法作用域或复合语句块。

### Lines 1441-1464

````tablegen

multiclass VADDV<bit acc, bit pred, string intbase, Type Scalar> {
  defvar accArg = !if(acc, (args Scalar:$acc), (args));
  defvar predArg = !if(pred, (args Predicate:$pred), (args));
  defvar intrinsic = !if(pred,
      IRInt<intbase # "_predicated", [Vector, Predicate]>,
      IRInt<intbase, [Vector]>);
  defvar intCG = !con((intrinsic $v, (unsignedflag Scalar)),
                      !if(pred, (? $pred), (?)));
  defvar accCG = !if(acc, (add intCG, $acc), intCG);

  def "": Intrinsic<Scalar, !con(accArg, (args Vector:$v), predArg), accCG>;
}

let params = T.Int in {
defm vaddvq    : VADDV<0, 0, "addv", Scalar32>;
defm vaddvaq   : VADDV<1, 0, "addv", Scalar32>;
defm vaddvq_p  : VADDV<0, 1, "addv", Scalar32>;
defm vaddvaq_p : VADDV<1, 1, "addv", Scalar32>;
}

let params = [s32, u32] in {
defm vaddlvq    : VADDV<0, 0, "addlv", Scalar64>;
defm vaddlvaq   : VADDV<1, 0, "addlv", Scalar64>;
````
- **L1441 EN**: Blank line separating nearby declarations or logic blocks.
  **L1441 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1442 EN**: Declares TableGen multiclass record `VADDV`.
  **L1442 CN**: 声明 TableGen multiclass 记录 `VADDV`。
- **L1443 EN**: Declares TableGen defvar record `accArg = !if`.
  **L1443 CN**: 声明 TableGen defvar 记录 `accArg = !if`。
- **L1444 EN**: Declares TableGen defvar record `predArg = !if`.
  **L1444 CN**: 声明 TableGen defvar 记录 `predArg = !if`。
- **L1445 EN**: Declares TableGen defvar record `intrinsic = !if`.
  **L1445 CN**: 声明 TableGen defvar 记录 `intrinsic = !if`。
- **L1446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IRInt<intbase # "_predicated", [Vector, Predicate]>,`.
  **L1446 CN**: 继续一个多行参数列表、初始化器或聚合项：`IRInt<intbase # "_predicated", [Vector, Predicate]>,`。
- **L1447 EN**: Adds a standalone statement or declaration: `IRInt<intbase, [Vector]>);`.
  **L1447 CN**: 添加一条独立语句或声明：`IRInt<intbase, [Vector]>);`。
- **L1448 EN**: Declares TableGen defvar record `intCG = !con`.
  **L1448 CN**: 声明 TableGen defvar 记录 `intCG = !con`。
- **L1449 EN**: Executes a call or declaration centered on `!if`.
  **L1449 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1450 EN**: Declares TableGen defvar record `accCG = !if`.
  **L1450 CN**: 声明 TableGen defvar 记录 `accCG = !if`。
- **L1451 EN**: Blank line separating nearby declarations or logic blocks.
  **L1451 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1452 EN**: Declares TableGen def record `""`.
  **L1452 CN**: 声明 TableGen def 记录 `""`。
- **L1453 EN**: Closes the current lexical scope or compound statement.
  **L1453 CN**: 结束当前词法作用域或复合语句块。
- **L1454 EN**: Blank line separating nearby declarations or logic blocks.
  **L1454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1455 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L1455 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L1456 EN**: Declares TableGen defm record `vaddvq`.
  **L1456 CN**: 声明 TableGen defm 记录 `vaddvq`。
- **L1457 EN**: Declares TableGen defm record `vaddvaq`.
  **L1457 CN**: 声明 TableGen defm 记录 `vaddvaq`。
- **L1458 EN**: Declares TableGen defm record `vaddvq_p`.
  **L1458 CN**: 声明 TableGen defm 记录 `vaddvq_p`。
- **L1459 EN**: Declares TableGen defm record `vaddvaq_p`.
  **L1459 CN**: 声明 TableGen defm 记录 `vaddvaq_p`。
- **L1460 EN**: Closes the current lexical scope or compound statement.
  **L1460 CN**: 结束当前词法作用域或复合语句块。
- **L1461 EN**: Blank line separating nearby declarations or logic blocks.
  **L1461 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1462 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s32, u32] in {`.
  **L1462 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s32, u32] in {`。
- **L1463 EN**: Declares TableGen defm record `vaddlvq`.
  **L1463 CN**: 声明 TableGen defm 记录 `vaddlvq`。
- **L1464 EN**: Declares TableGen defm record `vaddlvaq`.
  **L1464 CN**: 声明 TableGen defm 记录 `vaddlvaq`。

### Lines 1465-1488

````tablegen
defm vaddlvq_p  : VADDV<0, 1, "addlv", Scalar64>;
defm vaddlvaq_p : VADDV<1, 1, "addlv", Scalar64>;
}

let params = T.Int in {
def vabavq : Intrinsic<u32, (args u32:$a, Vector:$b, Vector:$c),
    (IRInt<"vabav", [Vector]> (unsignedflag Scalar), $a, $b, $c)>;
def vabavq_p : Intrinsic<u32, (args u32:$a, Vector:$b, Vector:$c,
                                    Predicate:$pred),
    (IRInt<"vabav_predicated", [Vector, Predicate]>
                               (unsignedflag Scalar), $a, $b, $c, $pred)>;

defm vmladav  : MVEBinaryVectorHoriz32<V.False, V.False, "">;
}

let params = T.Signed in {
defm vmladav : MVEBinaryVectorHoriz32<V.False, V.True, "x">;
defm vmlsdav : MVEBinaryVectorHoriz32<V.True, V.False, "">;
defm vmlsdav : MVEBinaryVectorHoriz32<V.True, V.True, "x">;
}

let params = [u16, s16, u32, s32] in
defm vmlaldav : MVEBinaryVectorHoriz64<V.False, V.False, "">;

````
- **L1465 EN**: Declares TableGen defm record `vaddlvq_p`.
  **L1465 CN**: 声明 TableGen defm 记录 `vaddlvq_p`。
- **L1466 EN**: Declares TableGen defm record `vaddlvaq_p`.
  **L1466 CN**: 声明 TableGen defm 记录 `vaddlvaq_p`。
- **L1467 EN**: Closes the current lexical scope or compound statement.
  **L1467 CN**: 结束当前词法作用域或复合语句块。
- **L1468 EN**: Blank line separating nearby declarations or logic blocks.
  **L1468 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1469 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int in {`.
  **L1469 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int in {`。
- **L1470 EN**: Declares TableGen def record `vabavq`.
  **L1470 CN**: 声明 TableGen def 记录 `vabavq`。
- **L1471 EN**: Executes a call or declaration centered on `statement`.
  **L1471 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1472 EN**: Declares TableGen def record `vabavq_p`.
  **L1472 CN**: 声明 TableGen def 记录 `vabavq_p`。
- **L1473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Predicate:$pred),`.
  **L1473 CN**: 继续一个多行参数列表、初始化器或聚合项：`Predicate:$pred),`。
- **L1474 EN**: Continues the surrounding expression or declaration: `(IRInt<"vabav_predicated", [Vector, Predicate]>`.
  **L1474 CN**: 继续构造周围的表达式或声明：`(IRInt<"vabav_predicated", [Vector, Predicate]>`。
- **L1475 EN**: Executes a call or declaration centered on `statement`.
  **L1475 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1476 EN**: Blank line separating nearby declarations or logic blocks.
  **L1476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1477 EN**: Declares TableGen defm record `vmladav`.
  **L1477 CN**: 声明 TableGen defm 记录 `vmladav`。
- **L1478 EN**: Closes the current lexical scope or compound statement.
  **L1478 CN**: 结束当前词法作用域或复合语句块。
- **L1479 EN**: Blank line separating nearby declarations or logic blocks.
  **L1479 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1480 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Signed in {`.
  **L1480 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Signed in {`。
- **L1481 EN**: Declares TableGen defm record `vmladav`.
  **L1481 CN**: 声明 TableGen defm 记录 `vmladav`。
- **L1482 EN**: Declares TableGen defm record `vmlsdav`.
  **L1482 CN**: 声明 TableGen defm 记录 `vmlsdav`。
- **L1483 EN**: Declares TableGen defm record `vmlsdav`.
  **L1483 CN**: 声明 TableGen defm 记录 `vmlsdav`。
- **L1484 EN**: Closes the current lexical scope or compound statement.
  **L1484 CN**: 结束当前词法作用域或复合语句块。
- **L1485 EN**: Blank line separating nearby declarations or logic blocks.
  **L1485 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1486 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [u16, s16, u32, s32] in`.
  **L1486 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [u16, s16, u32, s32] in`。
- **L1487 EN**: Declares TableGen defm record `vmlaldav`.
  **L1487 CN**: 声明 TableGen defm 记录 `vmlaldav`。
- **L1488 EN**: Blank line separating nearby declarations or logic blocks.
  **L1488 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 1489-1512

````tablegen
let params = [s16, s32] in {
defm vmlaldav : MVEBinaryVectorHoriz64<V.False, V.True, "x">;
defm vmlsldav : MVEBinaryVectorHoriz64<V.True, V.False, "">;
defm vmlsldav : MVEBinaryVectorHoriz64<V.True, V.True, "x">;
}

let params = T.Int32 in
defm vrmlaldavh : MVEBinaryVectorHoriz64R<V.False, V.False, "">;

let params = [s32] in {
defm vrmlaldavh : MVEBinaryVectorHoriz64R<V.False, V.True, "x">;
defm vrmlsldavh : MVEBinaryVectorHoriz64R<V.True, V.False, "">;
defm vrmlsldavh : MVEBinaryVectorHoriz64R<V.True, V.True, "x">;
}

multiclass vrev_predicated<int revsize> {
  defm "" : IntrinsicMX<Vector, (args Vector:$a, Predicate:$pred),
      (IRInt<"vrev_predicated", [Vector, Predicate]>
          $a, revsize, $pred, $inactive)>;
}

let params = T.All8 in {
  def vrev16q : Intrinsic<Vector, (args Vector:$a), (vrev $a, 16)>;
  defm vrev16q: vrev_predicated<16>;
````
- **L1489 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s16, s32] in {`.
  **L1489 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s16, s32] in {`。
- **L1490 EN**: Declares TableGen defm record `vmlaldav`.
  **L1490 CN**: 声明 TableGen defm 记录 `vmlaldav`。
- **L1491 EN**: Declares TableGen defm record `vmlsldav`.
  **L1491 CN**: 声明 TableGen defm 记录 `vmlsldav`。
- **L1492 EN**: Declares TableGen defm record `vmlsldav`.
  **L1492 CN**: 声明 TableGen defm 记录 `vmlsldav`。
- **L1493 EN**: Closes the current lexical scope or compound statement.
  **L1493 CN**: 结束当前词法作用域或复合语句块。
- **L1494 EN**: Blank line separating nearby declarations or logic blocks.
  **L1494 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1495 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Int32 in`.
  **L1495 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Int32 in`。
- **L1496 EN**: Declares TableGen defm record `vrmlaldavh`.
  **L1496 CN**: 声明 TableGen defm 记录 `vrmlaldavh`。
- **L1497 EN**: Blank line separating nearby declarations or logic blocks.
  **L1497 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1498 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = [s32] in {`.
  **L1498 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = [s32] in {`。
- **L1499 EN**: Declares TableGen defm record `vrmlaldavh`.
  **L1499 CN**: 声明 TableGen defm 记录 `vrmlaldavh`。
- **L1500 EN**: Declares TableGen defm record `vrmlsldavh`.
  **L1500 CN**: 声明 TableGen defm 记录 `vrmlsldavh`。
- **L1501 EN**: Declares TableGen defm record `vrmlsldavh`.
  **L1501 CN**: 声明 TableGen defm 记录 `vrmlsldavh`。
- **L1502 EN**: Closes the current lexical scope or compound statement.
  **L1502 CN**: 结束当前词法作用域或复合语句块。
- **L1503 EN**: Blank line separating nearby declarations or logic blocks.
  **L1503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1504 EN**: Declares TableGen multiclass record `vrev_predicated`.
  **L1504 CN**: 声明 TableGen multiclass 记录 `vrev_predicated`。
- **L1505 EN**: Declares TableGen defm record `""`.
  **L1505 CN**: 声明 TableGen defm 记录 `""`。
- **L1506 EN**: Continues the surrounding expression or declaration: `(IRInt<"vrev_predicated", [Vector, Predicate]>`.
  **L1506 CN**: 继续构造周围的表达式或声明：`(IRInt<"vrev_predicated", [Vector, Predicate]>`。
- **L1507 EN**: Adds a standalone statement or declaration: `$a, revsize, $pred, $inactive)>;`.
  **L1507 CN**: 添加一条独立语句或声明：`$a, revsize, $pred, $inactive)>;`。
- **L1508 EN**: Closes the current lexical scope or compound statement.
  **L1508 CN**: 结束当前词法作用域或复合语句块。
- **L1509 EN**: Blank line separating nearby declarations or logic blocks.
  **L1509 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1510 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.All8 in {`.
  **L1510 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.All8 in {`。
- **L1511 EN**: Declares TableGen def record `vrev16q`.
  **L1511 CN**: 声明 TableGen def 记录 `vrev16q`。
- **L1512 EN**: Declares TableGen defm record `vrev16q`.
  **L1512 CN**: 声明 TableGen defm 记录 `vrev16q`。

### Lines 1513-1536

````tablegen
}
let params = !listconcat(T.All8, T.All16) in {
  def vrev32q : Intrinsic<Vector, (args Vector:$a), (vrev $a, 32)>;
  defm vrev32q: vrev_predicated<32>;
}
let params = T.Usual in {
  def vrev64q : Intrinsic<Vector, (args Vector:$a), (vrev $a, 64)>;
  defm vrev64q: vrev_predicated<64>;
}

foreach desttype = T.All in {
  // We want a vreinterpretq between every pair of supported vector types
  // _except_ that there shouldn't be one from a type to itself.
  let params = !filter(srctype, T.All, !ne(srctype, desttype)) in {
    def "vreinterpretq_" # desttype: Intrinsic<
        VecOf<desttype>, (args Vector:$x), (vreinterpret $x, VecOf<desttype>)>;
  }
}

let params = T.All in {
  let pnt = PNT_None in {
    def vcreateq: Intrinsic<Vector, (args u64:$a, u64:$b),
        (vreinterpret (ielt_const (ielt_const (poison VecOf<u64>), $a, 0),
                             $b, 1), Vector)>;
````
- **L1513 EN**: Closes the current lexical scope or compound statement.
  **L1513 CN**: 结束当前词法作用域或复合语句块。
- **L1514 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = !listconcat(T.All8, T.All16) in {`.
  **L1514 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = !listconcat(T.All8, T.All16) in {`。
- **L1515 EN**: Declares TableGen def record `vrev32q`.
  **L1515 CN**: 声明 TableGen def 记录 `vrev32q`。
- **L1516 EN**: Declares TableGen defm record `vrev32q`.
  **L1516 CN**: 声明 TableGen defm 记录 `vrev32q`。
- **L1517 EN**: Closes the current lexical scope or compound statement.
  **L1517 CN**: 结束当前词法作用域或复合语句块。
- **L1518 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Usual in {`.
  **L1518 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Usual in {`。
- **L1519 EN**: Declares TableGen def record `vrev64q`.
  **L1519 CN**: 声明 TableGen def 记录 `vrev64q`。
- **L1520 EN**: Declares TableGen defm record `vrev64q`.
  **L1520 CN**: 声明 TableGen defm 记录 `vrev64q`。
- **L1521 EN**: Closes the current lexical scope or compound statement.
  **L1521 CN**: 结束当前词法作用域或复合语句块。
- **L1522 EN**: Blank line separating nearby declarations or logic blocks.
  **L1522 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1523 EN**: Starts a TableGen iteration used to generate repeated records: `foreach desttype = T.All in {`.
  **L1523 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach desttype = T.All in {`。
- **L1524 EN**: Comment explains nearby logic, constraints, or intent: `We want a vreinterpretq between every pair of supported vector types`.
  **L1524 CN**: 注释解释附近代码的逻辑、约束或设计意图：`We want a vreinterpretq between every pair of supported vector types`。
- **L1525 EN**: Comment explains nearby logic, constraints, or intent: `_except_ that there shouldn't be one from a type to itself.`.
  **L1525 CN**: 注释解释附近代码的逻辑、约束或设计意图：`_except_ that there shouldn't be one from a type to itself.`。
- **L1526 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = !filter(srctype, T.All, !ne(srctype, desttype)) in {`.
  **L1526 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = !filter(srctype, T.All, !ne(srctype, desttype)) in {`。
- **L1527 EN**: Declares TableGen def record `"vreinterpretq_" # desttype`.
  **L1527 CN**: 声明 TableGen def 记录 `"vreinterpretq_" # desttype`。
- **L1528 EN**: Executes a call or declaration centered on `VecOf<desttype>,`.
  **L1528 CN**: 执行以 `VecOf<desttype>,` 为核心的调用或声明。
- **L1529 EN**: Closes the current lexical scope or compound statement.
  **L1529 CN**: 结束当前词法作用域或复合语句块。
- **L1530 EN**: Closes the current lexical scope or compound statement.
  **L1530 CN**: 结束当前词法作用域或复合语句块。
- **L1531 EN**: Blank line separating nearby declarations or logic blocks.
  **L1531 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1532 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.All in {`.
  **L1532 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.All in {`。
- **L1533 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_None in {`.
  **L1533 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_None in {`。
- **L1534 EN**: Declares TableGen def record `vcreateq`.
  **L1534 CN**: 声明 TableGen def 记录 `vcreateq`。
- **L1535 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(vreinterpret (ielt_const (ielt_const (poison VecOf<u64>), $a, 0),`.
  **L1535 CN**: 继续一个多行参数列表、初始化器或聚合项：`(vreinterpret (ielt_const (ielt_const (poison VecOf<u64>), $a, 0),`。
- **L1536 EN**: Adds a standalone statement or declaration: `$b, 1), Vector)>;`.
  **L1536 CN**: 添加一条独立语句或声明：`$b, 1), Vector)>;`。

### Lines 1537-1560

````tablegen
    def vuninitializedq: Intrinsic<Vector, (args), (undef Vector)>;
  }

  // This is the polymorphic form of vuninitializedq, which takes no type
  // suffix, but takes an _unevaluated_ vector parameter and returns an
  // uninitialized vector of the same vector type.
  //
  // This intrinsic has no _non_-polymorphic form exposed to the user. But each
  // separately typed version of it still has to have its own clang builtin id,
  // which can't be called vuninitializedq_u32 or similar because that would
  // collide with the explicit nullary versions above. So I'm calling them
  // vuninitializedq_polymorphic_u32 (and so on) for builtin id purposes; that
  // full name never appears in the header file due to the polymorphicOnly
  // flag, and the _polymorphic suffix is omitted from the shortened name by
  // the custom PolymorphicNameType here.
  let polymorphicOnly = 1, nonEvaluating = 1,
      pnt = PolymorphicNameType<1, "polymorphic"> in {
    def vuninitializedq_polymorphic: Intrinsic<
        Vector, (args Vector), (undef Vector)>;
  }

  def vgetq_lane: Intrinsic<Scalar, (args Vector:$v, imm_lane:$lane),
                            (xelt_var $v, $lane)>;
  def vsetq_lane: Intrinsic<Vector, (args unpromoted<Scalar>:$e, Vector:$v, imm_lane:$lane),
````
- **L1537 EN**: Declares TableGen def record `vuninitializedq`.
  **L1537 CN**: 声明 TableGen def 记录 `vuninitializedq`。
- **L1538 EN**: Closes the current lexical scope or compound statement.
  **L1538 CN**: 结束当前词法作用域或复合语句块。
- **L1539 EN**: Blank line separating nearby declarations or logic blocks.
  **L1539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1540 EN**: Comment explains nearby logic, constraints, or intent: `This is the polymorphic form of vuninitializedq, which takes no type`.
  **L1540 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This is the polymorphic form of vuninitializedq, which takes no type`。
- **L1541 EN**: Comment explains nearby logic, constraints, or intent: `suffix, but takes an _unevaluated_ vector parameter and returns an`.
  **L1541 CN**: 注释解释附近代码的逻辑、约束或设计意图：`suffix, but takes an _unevaluated_ vector parameter and returns an`。
- **L1542 EN**: Comment explains nearby logic, constraints, or intent: `uninitialized vector of the same vector type.`.
  **L1542 CN**: 注释解释附近代码的逻辑、约束或设计意图：`uninitialized vector of the same vector type.`。
- **L1543 EN**: Separator comment used for visual grouping.
  **L1543 CN**: 用于视觉分组的分隔注释。
- **L1544 EN**: Comment explains nearby logic, constraints, or intent: `This intrinsic has no _non_-polymorphic form exposed to the user. But each`.
  **L1544 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This intrinsic has no _non_-polymorphic form exposed to the user. But each`。
- **L1545 EN**: Comment explains nearby logic, constraints, or intent: `separately typed version of it still has to have its own clang builtin id,`.
  **L1545 CN**: 注释解释附近代码的逻辑、约束或设计意图：`separately typed version of it still has to have its own clang builtin id,`。
- **L1546 EN**: Comment explains nearby logic, constraints, or intent: `which can't be called vuninitializedq_u32 or similar because that would`.
  **L1546 CN**: 注释解释附近代码的逻辑、约束或设计意图：`which can't be called vuninitializedq_u32 or similar because that would`。
- **L1547 EN**: Comment explains nearby logic, constraints, or intent: `collide with the explicit nullary versions above. So I'm calling them`.
  **L1547 CN**: 注释解释附近代码的逻辑、约束或设计意图：`collide with the explicit nullary versions above. So I'm calling them`。
- **L1548 EN**: Comment explains nearby logic, constraints, or intent: `vuninitializedq_polymorphic_u32 (and so on) for builtin id purposes; that`.
  **L1548 CN**: 注释解释附近代码的逻辑、约束或设计意图：`vuninitializedq_polymorphic_u32 (and so on) for builtin id purposes; that`。
- **L1549 EN**: Comment explains nearby logic, constraints, or intent: `full name never appears in the header file due to the polymorphicOnly`.
  **L1549 CN**: 注释解释附近代码的逻辑、约束或设计意图：`full name never appears in the header file due to the polymorphicOnly`。
- **L1550 EN**: Comment explains nearby logic, constraints, or intent: `flag, and the _polymorphic suffix is omitted from the shortened name by`.
  **L1550 CN**: 注释解释附近代码的逻辑、约束或设计意图：`flag, and the _polymorphic suffix is omitted from the shortened name by`。
- **L1551 EN**: Comment explains nearby logic, constraints, or intent: `the custom PolymorphicNameType here.`.
  **L1551 CN**: 注释解释附近代码的逻辑、约束或设计意图：`the custom PolymorphicNameType here.`。
- **L1552 EN**: Assigns a TableGen property that affects following records or inherited fields: `let polymorphicOnly = 1, nonEvaluating = 1,`.
  **L1552 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let polymorphicOnly = 1, nonEvaluating = 1,`。
- **L1553 EN**: Continues the surrounding expression or declaration: `pnt = PolymorphicNameType<1, "polymorphic"> in {`.
  **L1553 CN**: 继续构造周围的表达式或声明：`pnt = PolymorphicNameType<1, "polymorphic"> in {`。
- **L1554 EN**: Declares TableGen def record `vuninitializedq_polymorphic`.
  **L1554 CN**: 声明 TableGen def 记录 `vuninitializedq_polymorphic`。
- **L1555 EN**: Executes a call or declaration centered on `Vector,`.
  **L1555 CN**: 执行以 `Vector,` 为核心的调用或声明。
- **L1556 EN**: Closes the current lexical scope or compound statement.
  **L1556 CN**: 结束当前词法作用域或复合语句块。
- **L1557 EN**: Blank line separating nearby declarations or logic blocks.
  **L1557 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1558 EN**: Declares TableGen def record `vgetq_lane`.
  **L1558 CN**: 声明 TableGen def 记录 `vgetq_lane`。
- **L1559 EN**: Executes a call or declaration centered on `statement`.
  **L1559 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1560 EN**: Declares TableGen def record `vsetq_lane`.
  **L1560 CN**: 声明 TableGen def 记录 `vsetq_lane`。

### Lines 1561-1584

````tablegen
                            (ielt_var $v, $e, $lane)>;
}

foreach desttype = !listconcat(T.Int16, T.Int32, T.Float) in {
  defvar is_dest_float = !eq(desttype.kind, "f");
  defvar is_dest_unsigned = !eq(desttype.kind, "u");
  // First immediate operand of the LLVM intrinsic
  defvar unsigned_flag = !cond(is_dest_float: (unsignedflag Scalar),
                               is_dest_unsigned: V.True,
                               true: V.False);
  // For float->int conversions _n and _x_n intrinsics are not polymorphic
  // because the signedness of the destination type cannot be inferred.
  defvar pnt_nx = !if(is_dest_float, PNT_2Type, PNT_None);

  let params = !if(is_dest_float,
                   !if(!eq(desttype.size, 16), T.Int16, T.Int32),
                   !if(!eq(desttype.size, 16), [f16], [f32])) in {
    let pnt = pnt_nx in
      def "vcvtq_n_"#desttype : Intrinsic<VecOf<desttype>,
        (args Vector:$a, imm_1toN:$b),
        (IRInt<"vcvt_fix", [VecOf<desttype>, Vector]> unsigned_flag, $a, $b)>;

    defm "vcvtq" : IntrinsicMX<VecOf<desttype>,
      (args Vector:$a, imm_1toN:$b, Predicate:$p),
````
- **L1561 EN**: Executes a call or declaration centered on `statement`.
  **L1561 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1562 EN**: Closes the current lexical scope or compound statement.
  **L1562 CN**: 结束当前词法作用域或复合语句块。
- **L1563 EN**: Blank line separating nearby declarations or logic blocks.
  **L1563 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1564 EN**: Starts a TableGen iteration used to generate repeated records: `foreach desttype = !listconcat(T.Int16, T.Int32, T.Float) in {`.
  **L1564 CN**: 开始一个用于生成重复记录的 TableGen 迭代：`foreach desttype = !listconcat(T.Int16, T.Int32, T.Float) in {`。
- **L1565 EN**: Declares TableGen defvar record `is_dest_float = !eq`.
  **L1565 CN**: 声明 TableGen defvar 记录 `is_dest_float = !eq`。
- **L1566 EN**: Declares TableGen defvar record `is_dest_unsigned = !eq`.
  **L1566 CN**: 声明 TableGen defvar 记录 `is_dest_unsigned = !eq`。
- **L1567 EN**: Comment explains nearby logic, constraints, or intent: `First immediate operand of the LLVM intrinsic`.
  **L1567 CN**: 注释解释附近代码的逻辑、约束或设计意图：`First immediate operand of the LLVM intrinsic`。
- **L1568 EN**: Declares TableGen defvar record `unsigned_flag = !cond`.
  **L1568 CN**: 声明 TableGen defvar 记录 `unsigned_flag = !cond`。
- **L1569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `is_dest_unsigned: V.True,`.
  **L1569 CN**: 继续一个多行参数列表、初始化器或聚合项：`is_dest_unsigned: V.True,`。
- **L1570 EN**: Adds a standalone statement or declaration: `true: V.False);`.
  **L1570 CN**: 添加一条独立语句或声明：`true: V.False);`。
- **L1571 EN**: Comment explains nearby logic, constraints, or intent: `For float->int conversions _n and _x_n intrinsics are not polymorphic`.
  **L1571 CN**: 注释解释附近代码的逻辑、约束或设计意图：`For float->int conversions _n and _x_n intrinsics are not polymorphic`。
- **L1572 EN**: Comment explains nearby logic, constraints, or intent: `because the signedness of the destination type cannot be inferred.`.
  **L1572 CN**: 注释解释附近代码的逻辑、约束或设计意图：`because the signedness of the destination type cannot be inferred.`。
- **L1573 EN**: Declares TableGen defvar record `pnt_nx = !if`.
  **L1573 CN**: 声明 TableGen defvar 记录 `pnt_nx = !if`。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1575 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = !if(is_dest_float,`.
  **L1575 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = !if(is_dest_float,`。
- **L1576 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(desttype.size, 16), T.Int16, T.Int32),`.
  **L1576 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(desttype.size, 16), T.Int16, T.Int32),`。
- **L1577 EN**: Starts a function, inline wrapper, lambda, or structured helper scope: `!if(!eq(desttype.size, 16), [f16], [f32])) in {`.
  **L1577 CN**: 开始一个函数、内联包装层、lambda 或结构化辅助作用域：`!if(!eq(desttype.size, 16), [f16], [f32])) in {`。
- **L1578 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = pnt_nx in`.
  **L1578 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = pnt_nx in`。
- **L1579 EN**: Declares TableGen def record `"vcvtq_n_"#desttype`.
  **L1579 CN**: 声明 TableGen def 记录 `"vcvtq_n_"#desttype`。
- **L1580 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(args Vector:$a, imm_1toN:$b),`.
  **L1580 CN**: 继续一个多行参数列表、初始化器或聚合项：`(args Vector:$a, imm_1toN:$b),`。
- **L1581 EN**: Executes a call or declaration centered on `statement`.
  **L1581 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1582 EN**: Blank line separating nearby declarations or logic blocks.
  **L1582 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1583 EN**: Declares TableGen defm record `"vcvtq"`.
  **L1583 CN**: 声明 TableGen defm 记录 `"vcvtq"`。
- **L1584 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `(args Vector:$a, imm_1toN:$b, Predicate:$p),`.
  **L1584 CN**: 继续一个多行参数列表、初始化器或聚合项：`(args Vector:$a, imm_1toN:$b, Predicate:$p),`。

### Lines 1585-1599

````tablegen
      (IRInt<"vcvt_fix_predicated", [VecOf<desttype>, Vector, Predicate]>
             unsigned_flag, $inactive, $a, $b, $p),
      1, "_n_"#desttype, PNT_2Type, pnt_nx>;
  }
}

let params = T.Usual in {
let pnt = PNT_NType in
def vbrsrq_n: Intrinsic<Vector, (args Vector:$a, s32:$b),
                        (IRInt<"vbrsr", [Vector]> $a, $b)>;
defm vbrsrq : IntrinsicMX<Vector, (args Vector:$a, s32:$b, Predicate:$pred),
                         (IRInt<"vbrsr_predicated", [Vector, Predicate]>
                         $inactive, $a, $b, $pred), 1, "_n",
                         PNT_NType, PNT_NType>;
}
````
- **L1585 EN**: Continues the surrounding expression or declaration: `(IRInt<"vcvt_fix_predicated", [VecOf<desttype>, Vector, Predicate]>`.
  **L1585 CN**: 继续构造周围的表达式或声明：`(IRInt<"vcvt_fix_predicated", [VecOf<desttype>, Vector, Predicate]>`。
- **L1586 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `unsigned_flag, $inactive, $a, $b, $p),`.
  **L1586 CN**: 继续一个多行参数列表、初始化器或聚合项：`unsigned_flag, $inactive, $a, $b, $p),`。
- **L1587 EN**: Adds a standalone statement or declaration: `1, "_n_"#desttype, PNT_2Type, pnt_nx>;`.
  **L1587 CN**: 添加一条独立语句或声明：`1, "_n_"#desttype, PNT_2Type, pnt_nx>;`。
- **L1588 EN**: Closes the current lexical scope or compound statement.
  **L1588 CN**: 结束当前词法作用域或复合语句块。
- **L1589 EN**: Closes the current lexical scope or compound statement.
  **L1589 CN**: 结束当前词法作用域或复合语句块。
- **L1590 EN**: Blank line separating nearby declarations or logic blocks.
  **L1590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L1591 EN**: Assigns a TableGen property that affects following records or inherited fields: `let params = T.Usual in {`.
  **L1591 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let params = T.Usual in {`。
- **L1592 EN**: Assigns a TableGen property that affects following records or inherited fields: `let pnt = PNT_NType in`.
  **L1592 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let pnt = PNT_NType in`。
- **L1593 EN**: Declares TableGen def record `vbrsrq_n`.
  **L1593 CN**: 声明 TableGen def 记录 `vbrsrq_n`。
- **L1594 EN**: Executes a call or declaration centered on `statement`.
  **L1594 CN**: 执行以 `statement` 为核心的调用或声明。
- **L1595 EN**: Declares TableGen defm record `vbrsrq`.
  **L1595 CN**: 声明 TableGen defm 记录 `vbrsrq`。
- **L1596 EN**: Continues the surrounding expression or declaration: `(IRInt<"vbrsr_predicated", [Vector, Predicate]>`.
  **L1596 CN**: 继续构造周围的表达式或声明：`(IRInt<"vbrsr_predicated", [Vector, Predicate]>`。
- **L1597 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `$inactive, $a, $b, $pred), 1, "_n",`.
  **L1597 CN**: 继续一个多行参数列表、初始化器或聚合项：`$inactive, $a, $b, $pred), 1, "_n",`。
- **L1598 EN**: Adds a standalone statement or declaration: `PNT_NType, PNT_NType>;`.
  **L1598 CN**: 添加一条独立语句或声明：`PNT_NType, PNT_NType>;`。
- **L1599 EN**: Closes the current lexical scope or compound statement.
  **L1599 CN**: 结束当前词法作用域或复合语句块。

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
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Source-location handling / 源码位置处理**
  - **EN**: Tracks files, offsets, and source ranges needed by diagnostics and tooling.
  - **CN**: 跟踪诊断与工具所需的文件、偏移与源码范围。
- **Arm target support / Arm 目标支持**
  - **EN**: Encodes Arm-specific language extensions, intrinsics, or target metadata.
  - **CN**: 编码 Arm 专用语言扩展、intrinsic 或目标元数据。
- **Record families / 记录族**
  - **EN**: Groups related TableGen records so backends can derive structured generated output.
  - **CN**: 将相关 TableGen 记录组织成族，以便后端导出结构化的生成结果。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: `for`, `ScalarShift`, `LongScalarShift`, `IntrSplit64`, `IntrSplit64ZeroInit`
- **Functions or callables / 函数或可调用对象**: `bitcast`, `seq`, `add`, `listconcat`, `not`, `splat`, `con`, `store`, `vxdup_mc<`, `sub`, `select`, `setdagop`
- **TableGen records / TableGen 记录**: `"vst"#n#"q"`, `"vld"#n#"q"`, `bit_op_fp`, `""`, `bit_op_fp_with_inv`, `vqaddq`, `vqsubq`, `vqaddq_n`, `vqsubq_n`, `vqaddq_u`, `vqsubq_u`, `vqaddq_u_n`, `vqsubq_u_n`, `vaddq`, `vhaddq`, `vrhaddq`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
