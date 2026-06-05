# VPIntrinsics.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/VPIntrinsics.def`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file contains descriptions of the various Vector Predication intrinsics. This is used as a central place for enumerating the different instructions and should eventually be the place to put comments about the instructions.
- **Purpose (CN)**: 该定义片段文件位于 `llvm/include/llvm/IR`，主要为 `VPIntrinsics` 提供宏驱动的定义片段和枚举项。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//===-- IR/VPIntrinsics.def - Describes llvm.vp.* Intrinsics -*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains descriptions of the various Vector Predication intrinsics.
// This is used as a central place for enumerating the different instructions
// and should eventually be the place to put comments about the instructions.
//
//===----------------------------------------------------------------------===//

// NOTE: NO INCLUDE GUARD DESIRED!

// Provide definitions of macros so that users of this file do not have to
// define everything to use it...
//
// Register a VP intrinsic and begin its property scope.
// All VP intrinsic scopes are top level, ie it is illegal to place a
// BEGIN_REGISTER_VP_INTRINSIC within a VP intrinsic scope.
// \p VPID     The VP intrinsic id.
// \p MASKPOS  The mask operand position.
````
- **L1 EN**: Banner comment marking a file or section boundary.
  **L1 CN**: 横幅注释，用于标记文件或章节边界。
- **L2 EN**: Separator comment used for visual grouping.
  **L2 CN**: 用于视觉分组的分隔注释。
- **L3 EN**: Comment explains nearby logic, invariants, or intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L3 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4 EN**: Comment explains nearby logic, invariants, or intent: `See https://llvm.org/LICENSE.txt for license information.`.
  **L4 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5 EN**: Comment explains nearby logic, invariants, or intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L5 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6 EN**: Separator comment used for visual grouping.
  **L6 CN**: 用于视觉分组的分隔注释。
- **L7 EN**: Banner comment marking a file or section boundary.
  **L7 CN**: 横幅注释，用于标记文件或章节边界。
- **L8 EN**: Separator comment used for visual grouping.
  **L8 CN**: 用于视觉分组的分隔注释。
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file contains descriptions of the various Vector Predication intrinsics.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file contains descriptions of the various Vector Predication intrinsics.`。
- **L10 EN**: Comment explains nearby logic, invariants, or intent: `This is used as a central place for enumerating the different instructions`.
  **L10 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This is used as a central place for enumerating the different instructions`。
- **L11 EN**: Comment explains nearby logic, invariants, or intent: `and should eventually be the place to put comments about the instructions.`.
  **L11 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and should eventually be the place to put comments about the instructions.`。
- **L12 EN**: Separator comment used for visual grouping.
  **L12 CN**: 用于视觉分组的分隔注释。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L15 EN**: Comment highlights an implementation note: `NOTE: NO INCLUDE GUARD DESIRED!`.
  **L15 CN**: 注释强调了一条实现说明：`NOTE: NO INCLUDE GUARD DESIRED!`。
- **L16 EN**: Blank line separating nearby declarations or logic blocks.
  **L16 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L17 EN**: Comment explains nearby logic, invariants, or intent: `Provide definitions of macros so that users of this file do not have to`.
  **L17 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Provide definitions of macros so that users of this file do not have to`。
- **L18 EN**: Comment explains nearby logic, invariants, or intent: `define everything to use it...`.
  **L18 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`define everything to use it...`。
- **L19 EN**: Separator comment used for visual grouping.
  **L19 CN**: 用于视觉分组的分隔注释。
- **L20 EN**: Comment explains nearby logic, invariants, or intent: `Register a VP intrinsic and begin its property scope.`.
  **L20 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Register a VP intrinsic and begin its property scope.`。
- **L21 EN**: Comment explains nearby logic, invariants, or intent: `All VP intrinsic scopes are top level, ie it is illegal to place a`.
  **L21 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All VP intrinsic scopes are top level, ie it is illegal to place a`。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `BEGIN_REGISTER_VP_INTRINSIC within a VP intrinsic scope.`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BEGIN_REGISTER_VP_INTRINSIC within a VP intrinsic scope.`。
- **L23 EN**: Comment explains nearby logic, invariants, or intent: `\p VPID     The VP intrinsic id.`.
  **L23 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p VPID     The VP intrinsic id.`。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `\p MASKPOS  The mask operand position.`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p MASKPOS  The mask operand position.`。

### Lines 25-48

````cpp
// \p EVLPOS   The explicit vector length operand position.
#ifndef BEGIN_REGISTER_VP_INTRINSIC
#define BEGIN_REGISTER_VP_INTRINSIC(VPID, MASKPOS, EVLPOS)
#endif

// End the property scope of a VP intrinsic.
#ifndef END_REGISTER_VP_INTRINSIC
#define END_REGISTER_VP_INTRINSIC(VPID)
#endif

// Register a new VP SDNode and begin its property scope.
// When the SDNode scope is nested within a VP intrinsic scope, it is
// implicitly registered as the canonical SDNode for this VP intrinsic. There
// is one VP intrinsic that maps directly to one SDNode that goes by the
// same name.  Since the operands are also the same, we open the property
// scopes for both the VPIntrinsic and the SDNode at once.
// \p VPSD     The SelectionDAG Node id (eg VP_ADD).
// \p LEGALPOS The operand position of the SDNode that is used for legalizing.
//             If LEGALPOS < 0, then the return type given by
//             TheNode->getValueType(-1-LEGALPOS) is used.
// \p TDNAME   The name of the TableGen definition of this SDNode.
// \p MASKPOS  The mask operand position.
// \p EVLPOS   The explicit vector length operand position.
#ifndef BEGIN_REGISTER_VP_SDNODE
````
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `\p EVLPOS   The explicit vector length operand position.`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p EVLPOS   The explicit vector length operand position.`。
- **L26 EN**: Starts a preprocessor conditional block: `#ifndef BEGIN_REGISTER_VP_INTRINSIC`.
  **L26 CN**: 开始一个预处理条件块：`#ifndef BEGIN_REGISTER_VP_INTRINSIC`。
- **L27 EN**: Defines macro `BEGIN_REGISTER_VP_INTRINSIC(VPID,` for conditional compilation, local shorthand, or diagnostics.
  **L27 CN**: 定义宏 `BEGIN_REGISTER_VP_INTRINSIC(VPID,`，供条件编译、本地简写或诊断使用。
- **L28 EN**: Closes the current preprocessor conditional block.
  **L28 CN**: 结束当前预处理条件块。
- **L29 EN**: Blank line separating nearby declarations or logic blocks.
  **L29 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `End the property scope of a VP intrinsic.`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`End the property scope of a VP intrinsic.`。
- **L31 EN**: Starts a preprocessor conditional block: `#ifndef END_REGISTER_VP_INTRINSIC`.
  **L31 CN**: 开始一个预处理条件块：`#ifndef END_REGISTER_VP_INTRINSIC`。
- **L32 EN**: Defines macro `END_REGISTER_VP_INTRINSIC(VPID)` for conditional compilation, local shorthand, or diagnostics.
  **L32 CN**: 定义宏 `END_REGISTER_VP_INTRINSIC(VPID)`，供条件编译、本地简写或诊断使用。
- **L33 EN**: Closes the current preprocessor conditional block.
  **L33 CN**: 结束当前预处理条件块。
- **L34 EN**: Blank line separating nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Comment explains nearby logic, invariants, or intent: `Register a new VP SDNode and begin its property scope.`.
  **L35 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Register a new VP SDNode and begin its property scope.`。
- **L36 EN**: Comment explains nearby logic, invariants, or intent: `When the SDNode scope is nested within a VP intrinsic scope, it is`.
  **L36 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`When the SDNode scope is nested within a VP intrinsic scope, it is`。
- **L37 EN**: Comment explains nearby logic, invariants, or intent: `implicitly registered as the canonical SDNode for this VP intrinsic. There`.
  **L37 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`implicitly registered as the canonical SDNode for this VP intrinsic. There`。
- **L38 EN**: Comment explains nearby logic, invariants, or intent: `is one VP intrinsic that maps directly to one SDNode that goes by the`.
  **L38 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`is one VP intrinsic that maps directly to one SDNode that goes by the`。
- **L39 EN**: Comment explains nearby logic, invariants, or intent: `same name.  Since the operands are also the same, we open the property`.
  **L39 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`same name.  Since the operands are also the same, we open the property`。
- **L40 EN**: Comment explains nearby logic, invariants, or intent: `scopes for both the VPIntrinsic and the SDNode at once.`.
  **L40 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scopes for both the VPIntrinsic and the SDNode at once.`。
- **L41 EN**: Comment explains nearby logic, invariants, or intent: `\p VPSD     The SelectionDAG Node id (eg VP_ADD).`.
  **L41 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p VPSD     The SelectionDAG Node id (eg VP_ADD).`。
- **L42 EN**: Comment explains nearby logic, invariants, or intent: `\p LEGALPOS The operand position of the SDNode that is used for legalizing.`.
  **L42 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p LEGALPOS The operand position of the SDNode that is used for legalizing.`。
- **L43 EN**: Comment explains nearby logic, invariants, or intent: `If LEGALPOS < 0, then the return type given by`.
  **L43 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If LEGALPOS < 0, then the return type given by`。
- **L44 EN**: Comment explains nearby logic, invariants, or intent: `TheNode->getValueType(-1-LEGALPOS) is used.`.
  **L44 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TheNode->getValueType(-1-LEGALPOS) is used.`。
- **L45 EN**: Comment explains nearby logic, invariants, or intent: `\p TDNAME   The name of the TableGen definition of this SDNode.`.
  **L45 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p TDNAME   The name of the TableGen definition of this SDNode.`。
- **L46 EN**: Comment explains nearby logic, invariants, or intent: `\p MASKPOS  The mask operand position.`.
  **L46 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p MASKPOS  The mask operand position.`。
- **L47 EN**: Comment explains nearby logic, invariants, or intent: `\p EVLPOS   The explicit vector length operand position.`.
  **L47 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p EVLPOS   The explicit vector length operand position.`。
- **L48 EN**: Starts a preprocessor conditional block: `#ifndef BEGIN_REGISTER_VP_SDNODE`.
  **L48 CN**: 开始一个预处理条件块：`#ifndef BEGIN_REGISTER_VP_SDNODE`。

### Lines 49-72

````cpp
#define BEGIN_REGISTER_VP_SDNODE(VPSD, LEGALPOS, TDNAME, MASKPOS, EVLPOS)
#endif

// End the property scope of a new VP SDNode.
#ifndef END_REGISTER_VP_SDNODE
#define END_REGISTER_VP_SDNODE(VPSD)
#endif

// Helper macro to set up the mapping from VP intrinsic to ISD opcode.
// Note: More than one VP intrinsic may map to one ISD opcode.
#ifndef HELPER_MAP_VPID_TO_VPSD
#define HELPER_MAP_VPID_TO_VPSD(VPID, VPSD)
#endif

// Helper macros for the common "1:1 - Intrinsic : SDNode" case.
//
// There is one VP intrinsic that maps directly to one SDNode that goes by the
// same name.  Since the operands are also the same, we open the property
// scopes for both the VPIntrinsic and the SDNode at once.
//
// \p VPID     The canonical name (eg `vp_add`, which at the same time is the
//             name of the intrinsic and the TableGen def of the SDNode).
// \p MASKPOS  The mask operand position.
// \p EVLPOS   The explicit vector length operand position.
````
- **L49 EN**: Defines macro `BEGIN_REGISTER_VP_SDNODE(VPSD,` for conditional compilation, local shorthand, or diagnostics.
  **L49 CN**: 定义宏 `BEGIN_REGISTER_VP_SDNODE(VPSD,`，供条件编译、本地简写或诊断使用。
- **L50 EN**: Closes the current preprocessor conditional block.
  **L50 CN**: 结束当前预处理条件块。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Comment explains nearby logic, invariants, or intent: `End the property scope of a new VP SDNode.`.
  **L52 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`End the property scope of a new VP SDNode.`。
- **L53 EN**: Starts a preprocessor conditional block: `#ifndef END_REGISTER_VP_SDNODE`.
  **L53 CN**: 开始一个预处理条件块：`#ifndef END_REGISTER_VP_SDNODE`。
- **L54 EN**: Defines macro `END_REGISTER_VP_SDNODE(VPSD)` for conditional compilation, local shorthand, or diagnostics.
  **L54 CN**: 定义宏 `END_REGISTER_VP_SDNODE(VPSD)`，供条件编译、本地简写或诊断使用。
- **L55 EN**: Closes the current preprocessor conditional block.
  **L55 CN**: 结束当前预处理条件块。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L57 EN**: Comment explains nearby logic, invariants, or intent: `Helper macro to set up the mapping from VP intrinsic to ISD opcode.`.
  **L57 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper macro to set up the mapping from VP intrinsic to ISD opcode.`。
- **L58 EN**: Comment explains nearby logic, invariants, or intent: `Note: More than one VP intrinsic may map to one ISD opcode.`.
  **L58 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: More than one VP intrinsic may map to one ISD opcode.`。
- **L59 EN**: Starts a preprocessor conditional block: `#ifndef HELPER_MAP_VPID_TO_VPSD`.
  **L59 CN**: 开始一个预处理条件块：`#ifndef HELPER_MAP_VPID_TO_VPSD`。
- **L60 EN**: Defines macro `HELPER_MAP_VPID_TO_VPSD(VPID,` for conditional compilation, local shorthand, or diagnostics.
  **L60 CN**: 定义宏 `HELPER_MAP_VPID_TO_VPSD(VPID,`，供条件编译、本地简写或诊断使用。
- **L61 EN**: Closes the current preprocessor conditional block.
  **L61 CN**: 结束当前预处理条件块。
- **L62 EN**: Blank line separating nearby declarations or logic blocks.
  **L62 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L63 EN**: Comment explains nearby logic, invariants, or intent: `Helper macros for the common "1:1 - Intrinsic : SDNode" case.`.
  **L63 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper macros for the common "1:1 - Intrinsic : SDNode" case.`。
- **L64 EN**: Separator comment used for visual grouping.
  **L64 CN**: 用于视觉分组的分隔注释。
- **L65 EN**: Comment explains nearby logic, invariants, or intent: `There is one VP intrinsic that maps directly to one SDNode that goes by the`.
  **L65 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`There is one VP intrinsic that maps directly to one SDNode that goes by the`。
- **L66 EN**: Comment explains nearby logic, invariants, or intent: `same name.  Since the operands are also the same, we open the property`.
  **L66 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`same name.  Since the operands are also the same, we open the property`。
- **L67 EN**: Comment explains nearby logic, invariants, or intent: `scopes for both the VPIntrinsic and the SDNode at once.`.
  **L67 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scopes for both the VPIntrinsic and the SDNode at once.`。
- **L68 EN**: Separator comment used for visual grouping.
  **L68 CN**: 用于视觉分组的分隔注释。
- **L69 EN**: Comment explains nearby logic, invariants, or intent: `\p VPID     The canonical name (eg `vp_add`, which at the same time is the`.
  **L69 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p VPID     The canonical name (eg `vp_add`, which at the same time is the`。
- **L70 EN**: Comment explains nearby logic, invariants, or intent: `name of the intrinsic and the TableGen def of the SDNode).`.
  **L70 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`name of the intrinsic and the TableGen def of the SDNode).`。
- **L71 EN**: Comment explains nearby logic, invariants, or intent: `\p MASKPOS  The mask operand position.`.
  **L71 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p MASKPOS  The mask operand position.`。
- **L72 EN**: Comment explains nearby logic, invariants, or intent: `\p EVLPOS   The explicit vector length operand position.`.
  **L72 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p EVLPOS   The explicit vector length operand position.`。

### Lines 73-96

````cpp
// \p VPSD     The SelectionDAG Node id (eg VP_ADD).
// \p LEGALPOS The operand position of the SDNode that is used for legalizing
//             this SDNode. This can be `-1`, in which case the return type of
//             the SDNode is used.
#define BEGIN_REGISTER_VP(VPID, MASKPOS, EVLPOS, VPSD, LEGALPOS)               \
  BEGIN_REGISTER_VP_INTRINSIC(VPID, MASKPOS, EVLPOS)                           \
  BEGIN_REGISTER_VP_SDNODE(VPSD, LEGALPOS, VPID, MASKPOS, EVLPOS)              \
  HELPER_MAP_VPID_TO_VPSD(VPID, VPSD)

#define END_REGISTER_VP(VPID, VPSD)                                            \
  END_REGISTER_VP_INTRINSIC(VPID)                                              \
  END_REGISTER_VP_SDNODE(VPSD)

// The following macros attach properties to the scope they are placed in. This
// assigns the property to the VP Intrinsic and/or SDNode that belongs to the
// scope.
//
// Property Macros {

// The intrinsic and/or SDNode has the same function as this LLVM IR Opcode.
// \p OPC      The opcode of the instruction with the same function.
#ifndef VP_PROPERTY_FUNCTIONAL_OPC
#define VP_PROPERTY_FUNCTIONAL_OPC(OPC)
#endif
````
- **L73 EN**: Comment explains nearby logic, invariants, or intent: `\p VPSD     The SelectionDAG Node id (eg VP_ADD).`.
  **L73 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p VPSD     The SelectionDAG Node id (eg VP_ADD).`。
- **L74 EN**: Comment explains nearby logic, invariants, or intent: `\p LEGALPOS The operand position of the SDNode that is used for legalizing`.
  **L74 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p LEGALPOS The operand position of the SDNode that is used for legalizing`。
- **L75 EN**: Comment explains nearby logic, invariants, or intent: `this SDNode. This can be `-1`, in which case the return type of`.
  **L75 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`this SDNode. This can be `-1`, in which case the return type of`。
- **L76 EN**: Comment explains nearby logic, invariants, or intent: `the SDNode is used.`.
  **L76 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the SDNode is used.`。
- **L77 EN**: Defines macro `BEGIN_REGISTER_VP(VPID,` for conditional compilation, local shorthand, or diagnostics.
  **L77 CN**: 定义宏 `BEGIN_REGISTER_VP(VPID,`，供条件编译、本地简写或诊断使用。
- **L78 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L78 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L79 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L79 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L80 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L80 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L82 EN**: Defines macro `END_REGISTER_VP(VPID,` for conditional compilation, local shorthand, or diagnostics.
  **L82 CN**: 定义宏 `END_REGISTER_VP(VPID,`，供条件编译、本地简写或诊断使用。
- **L83 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_INTRINSIC`.
  **L83 CN**: 继续与可调用符号 `END_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L84 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L84 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L85 EN**: Blank line separating nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L86 EN**: Comment explains nearby logic, invariants, or intent: `The following macros attach properties to the scope they are placed in. This`.
  **L86 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following macros attach properties to the scope they are placed in. This`。
- **L87 EN**: Comment explains nearby logic, invariants, or intent: `assigns the property to the VP Intrinsic and/or SDNode that belongs to the`.
  **L87 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`assigns the property to the VP Intrinsic and/or SDNode that belongs to the`。
- **L88 EN**: Comment explains nearby logic, invariants, or intent: `scope.`.
  **L88 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`scope.`。
- **L89 EN**: Separator comment used for visual grouping.
  **L89 CN**: 用于视觉分组的分隔注释。
- **L90 EN**: Comment explains nearby logic, invariants, or intent: `Property Macros {`.
  **L90 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Property Macros {`。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L92 EN**: Comment explains nearby logic, invariants, or intent: `The intrinsic and/or SDNode has the same function as this LLVM IR Opcode.`.
  **L92 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The intrinsic and/or SDNode has the same function as this LLVM IR Opcode.`。
- **L93 EN**: Comment explains nearby logic, invariants, or intent: `\p OPC      The opcode of the instruction with the same function.`.
  **L93 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p OPC      The opcode of the instruction with the same function.`。
- **L94 EN**: Starts a preprocessor conditional block: `#ifndef VP_PROPERTY_FUNCTIONAL_OPC`.
  **L94 CN**: 开始一个预处理条件块：`#ifndef VP_PROPERTY_FUNCTIONAL_OPC`。
- **L95 EN**: Defines macro `VP_PROPERTY_FUNCTIONAL_OPC(OPC)` for conditional compilation, local shorthand, or diagnostics.
  **L95 CN**: 定义宏 `VP_PROPERTY_FUNCTIONAL_OPC(OPC)`，供条件编译、本地简写或诊断使用。
- **L96 EN**: Closes the current preprocessor conditional block.
  **L96 CN**: 结束当前预处理条件块。

### Lines 97-120

````cpp

// If operation can have rounding or fp exceptions, maps to corresponding
// constrained fp intrinsic.
#ifndef VP_PROPERTY_CONSTRAINEDFP
#define VP_PROPERTY_CONSTRAINEDFP(INTRINID)
#endif

// The intrinsic and/or SDNode has the same function as this ISD Opcode.
// \p SDOPC      The opcode of the instruction with the same function.
#ifndef VP_PROPERTY_FUNCTIONAL_SDOPC
#define VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)
#endif

// Map this VP intrinsic to its canonical functional intrinsic.
// \p INTRIN     The non-VP intrinsics with the same function.
#ifndef VP_PROPERTY_FUNCTIONAL_INTRINSIC
#define VP_PROPERTY_FUNCTIONAL_INTRINSIC(INTRIN)
#endif

// This VP Intrinsic has no functionally-equivalent non-VP opcode or intrinsic.
#ifndef VP_PROPERTY_NO_FUNCTIONAL
#define VP_PROPERTY_NO_FUNCTIONAL
#endif

````
- **L97 EN**: Blank line separating nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Comment explains nearby logic, invariants, or intent: `If operation can have rounding or fp exceptions, maps to corresponding`.
  **L98 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`If operation can have rounding or fp exceptions, maps to corresponding`。
- **L99 EN**: Comment explains nearby logic, invariants, or intent: `constrained fp intrinsic.`.
  **L99 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`constrained fp intrinsic.`。
- **L100 EN**: Starts a preprocessor conditional block: `#ifndef VP_PROPERTY_CONSTRAINEDFP`.
  **L100 CN**: 开始一个预处理条件块：`#ifndef VP_PROPERTY_CONSTRAINEDFP`。
- **L101 EN**: Defines macro `VP_PROPERTY_CONSTRAINEDFP(INTRINID)` for conditional compilation, local shorthand, or diagnostics.
  **L101 CN**: 定义宏 `VP_PROPERTY_CONSTRAINEDFP(INTRINID)`，供条件编译、本地简写或诊断使用。
- **L102 EN**: Closes the current preprocessor conditional block.
  **L102 CN**: 结束当前预处理条件块。
- **L103 EN**: Blank line separating nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L104 EN**: Comment explains nearby logic, invariants, or intent: `The intrinsic and/or SDNode has the same function as this ISD Opcode.`.
  **L104 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The intrinsic and/or SDNode has the same function as this ISD Opcode.`。
- **L105 EN**: Comment explains nearby logic, invariants, or intent: `\p SDOPC      The opcode of the instruction with the same function.`.
  **L105 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p SDOPC      The opcode of the instruction with the same function.`。
- **L106 EN**: Starts a preprocessor conditional block: `#ifndef VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L106 CN**: 开始一个预处理条件块：`#ifndef VP_PROPERTY_FUNCTIONAL_SDOPC`。
- **L107 EN**: Defines macro `VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)` for conditional compilation, local shorthand, or diagnostics.
  **L107 CN**: 定义宏 `VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)`，供条件编译、本地简写或诊断使用。
- **L108 EN**: Closes the current preprocessor conditional block.
  **L108 CN**: 结束当前预处理条件块。
- **L109 EN**: Blank line separating nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `Map this VP intrinsic to its canonical functional intrinsic.`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Map this VP intrinsic to its canonical functional intrinsic.`。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `\p INTRIN     The non-VP intrinsics with the same function.`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`\p INTRIN     The non-VP intrinsics with the same function.`。
- **L112 EN**: Starts a preprocessor conditional block: `#ifndef VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L112 CN**: 开始一个预处理条件块：`#ifndef VP_PROPERTY_FUNCTIONAL_INTRINSIC`。
- **L113 EN**: Defines macro `VP_PROPERTY_FUNCTIONAL_INTRINSIC(INTRIN)` for conditional compilation, local shorthand, or diagnostics.
  **L113 CN**: 定义宏 `VP_PROPERTY_FUNCTIONAL_INTRINSIC(INTRIN)`，供条件编译、本地简写或诊断使用。
- **L114 EN**: Closes the current preprocessor conditional block.
  **L114 CN**: 结束当前预处理条件块。
- **L115 EN**: Blank line separating nearby declarations or logic blocks.
  **L115 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L116 EN**: Comment explains nearby logic, invariants, or intent: `This VP Intrinsic has no functionally-equivalent non-VP opcode or intrinsic.`.
  **L116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This VP Intrinsic has no functionally-equivalent non-VP opcode or intrinsic.`。
- **L117 EN**: Starts a preprocessor conditional block: `#ifndef VP_PROPERTY_NO_FUNCTIONAL`.
  **L117 CN**: 开始一个预处理条件块：`#ifndef VP_PROPERTY_NO_FUNCTIONAL`。
- **L118 EN**: Defines macro `VP_PROPERTY_NO_FUNCTIONAL` for conditional compilation, local shorthand, or diagnostics.
  **L118 CN**: 定义宏 `VP_PROPERTY_NO_FUNCTIONAL`，供条件编译、本地简写或诊断使用。
- **L119 EN**: Closes the current preprocessor conditional block.
  **L119 CN**: 结束当前预处理条件块。
- **L120 EN**: Blank line separating nearby declarations or logic blocks.
  **L120 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 121-144

````cpp
// A property to infer VP binary-op SDNode opcodes automatically.
#ifndef VP_PROPERTY_BINARYOP
#define VP_PROPERTY_BINARYOP
#endif

/// } Property Macros

///// Integer Arithmetic {

// Specialized helper macro for integer binary operators (%x, %y, %mask, %evl).
#ifdef HELPER_REGISTER_BINARY_INT_VP
#error                                                                         \
    "The internal helper macro HELPER_REGISTER_BINARY_INT_VP is already defined!"
#endif
#define HELPER_REGISTER_BINARY_INT_VP(VPID, VPSD, IROPC, SDOPC)                \
  BEGIN_REGISTER_VP(VPID, 2, 3, VPSD, -1)                                      \
  VP_PROPERTY_FUNCTIONAL_OPC(IROPC)                                            \
  VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)                                          \
  VP_PROPERTY_BINARYOP                                                         \
  END_REGISTER_VP(VPID, VPSD)

// llvm.vp.add(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_add, VP_ADD, Add, ADD)

````
- **L121 EN**: Comment explains nearby logic, invariants, or intent: `A property to infer VP binary-op SDNode opcodes automatically.`.
  **L121 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`A property to infer VP binary-op SDNode opcodes automatically.`。
- **L122 EN**: Starts a preprocessor conditional block: `#ifndef VP_PROPERTY_BINARYOP`.
  **L122 CN**: 开始一个预处理条件块：`#ifndef VP_PROPERTY_BINARYOP`。
- **L123 EN**: Defines macro `VP_PROPERTY_BINARYOP` for conditional compilation, local shorthand, or diagnostics.
  **L123 CN**: 定义宏 `VP_PROPERTY_BINARYOP`，供条件编译、本地简写或诊断使用。
- **L124 EN**: Closes the current preprocessor conditional block.
  **L124 CN**: 结束当前预处理条件块。
- **L125 EN**: Blank line separating nearby declarations or logic blocks.
  **L125 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L126 EN**: Comment explains nearby logic, invariants, or intent: `} Property Macros`.
  **L126 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Property Macros`。
- **L127 EN**: Blank line separating nearby declarations or logic blocks.
  **L127 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L128 EN**: Comment explains nearby logic, invariants, or intent: `Integer Arithmetic {`.
  **L128 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Integer Arithmetic {`。
- **L129 EN**: Blank line separating nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L130 EN**: Comment explains nearby logic, invariants, or intent: `Specialized helper macro for integer binary operators (%x, %y, %mask, %evl).`.
  **L130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Specialized helper macro for integer binary operators (%x, %y, %mask, %evl).`。
- **L131 EN**: Starts a preprocessor conditional block: `#ifdef HELPER_REGISTER_BINARY_INT_VP`.
  **L131 CN**: 开始一个预处理条件块：`#ifdef HELPER_REGISTER_BINARY_INT_VP`。
- **L132 EN**: Continues the surrounding expression or declaration: `#error                                                                         \`.
  **L132 CN**: 继续构造周围的表达式或声明：`#error                                                                         \`。
- **L133 EN**: Continues the surrounding expression or declaration: `"The internal helper macro HELPER_REGISTER_BINARY_INT_VP is already defined!"`.
  **L133 CN**: 继续构造周围的表达式或声明：`"The internal helper macro HELPER_REGISTER_BINARY_INT_VP is already defined!"`。
- **L134 EN**: Closes the current preprocessor conditional block.
  **L134 CN**: 结束当前预处理条件块。
- **L135 EN**: Defines macro `HELPER_REGISTER_BINARY_INT_VP(VPID,` for conditional compilation, local shorthand, or diagnostics.
  **L135 CN**: 定义宏 `HELPER_REGISTER_BINARY_INT_VP(VPID,`，供条件编译、本地简写或诊断使用。
- **L136 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L136 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L137 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L137 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L138 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L138 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L139 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP                                                         \`.
  **L139 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP                                                         \`。
- **L140 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L140 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L142 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.add(x,y,mask,vlen)`.
  **L142 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.add(x,y,mask,vlen)`。
- **L143 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L143 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L144 EN**: Blank line separating nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 145-168

````cpp
// llvm.vp.and(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_and, VP_AND, And, AND)

// llvm.vp.ashr(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_ashr, VP_SRA, AShr, SRA)

// llvm.vp.lshr(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_lshr, VP_SRL, LShr, SRL)

// llvm.vp.mul(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_mul, VP_MUL, Mul, MUL)

// llvm.vp.or(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_or, VP_OR, Or, OR)

// llvm.vp.sdiv(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_sdiv, VP_SDIV, SDiv, SDIV)

// llvm.vp.shl(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_shl, VP_SHL, Shl, SHL)

// llvm.vp.srem(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_srem, VP_SREM, SRem, SREM)

````
- **L145 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.and(x,y,mask,vlen)`.
  **L145 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.and(x,y,mask,vlen)`。
- **L146 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L146 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.ashr(x,y,mask,vlen)`.
  **L148 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.ashr(x,y,mask,vlen)`。
- **L149 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L149 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L150 EN**: Blank line separating nearby declarations or logic blocks.
  **L150 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L151 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.lshr(x,y,mask,vlen)`.
  **L151 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.lshr(x,y,mask,vlen)`。
- **L152 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L152 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L154 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.mul(x,y,mask,vlen)`.
  **L154 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.mul(x,y,mask,vlen)`。
- **L155 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L155 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L157 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.or(x,y,mask,vlen)`.
  **L157 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.or(x,y,mask,vlen)`。
- **L158 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L158 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L159 EN**: Blank line separating nearby declarations or logic blocks.
  **L159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L160 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.sdiv(x,y,mask,vlen)`.
  **L160 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.sdiv(x,y,mask,vlen)`。
- **L161 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L161 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L162 EN**: Blank line separating nearby declarations or logic blocks.
  **L162 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L163 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.shl(x,y,mask,vlen)`.
  **L163 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.shl(x,y,mask,vlen)`。
- **L164 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L164 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L165 EN**: Blank line separating nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.srem(x,y,mask,vlen)`.
  **L166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.srem(x,y,mask,vlen)`。
- **L167 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L167 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L168 EN**: Blank line separating nearby declarations or logic blocks.
  **L168 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 169-192

````cpp
// llvm.vp.sub(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_sub, VP_SUB, Sub, SUB)

// llvm.vp.udiv(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_udiv, VP_UDIV, UDiv, UDIV)

// llvm.vp.urem(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_urem, VP_UREM, URem, UREM)

// llvm.vp.xor(x,y,mask,vlen)
HELPER_REGISTER_BINARY_INT_VP(vp_xor, VP_XOR, Xor, XOR)

#undef HELPER_REGISTER_BINARY_INT_VP

// llvm.vp.smin(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_smin, 2, 3, VP_SMIN, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(SMIN)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(smin)
END_REGISTER_VP(vp_smin, VP_SMIN)

// llvm.vp.smax(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_smax, 2, 3, VP_SMAX, -1)
VP_PROPERTY_BINARYOP
````
- **L169 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.sub(x,y,mask,vlen)`.
  **L169 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.sub(x,y,mask,vlen)`。
- **L170 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L170 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L172 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.udiv(x,y,mask,vlen)`.
  **L172 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.udiv(x,y,mask,vlen)`。
- **L173 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L173 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L174 EN**: Blank line separating nearby declarations or logic blocks.
  **L174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L175 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.urem(x,y,mask,vlen)`.
  **L175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.urem(x,y,mask,vlen)`。
- **L176 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L176 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L177 EN**: Blank line separating nearby declarations or logic blocks.
  **L177 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L178 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.xor(x,y,mask,vlen)`.
  **L178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.xor(x,y,mask,vlen)`。
- **L179 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_INT_VP`.
  **L179 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_INT_VP` 相关的逻辑。
- **L180 EN**: Blank line separating nearby declarations or logic blocks.
  **L180 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L181 EN**: Undefines a macro to limit its scope: `#undef HELPER_REGISTER_BINARY_INT_VP`.
  **L181 CN**: 取消宏定义以限制其作用域：`#undef HELPER_REGISTER_BINARY_INT_VP`。
- **L182 EN**: Blank line separating nearby declarations or logic blocks.
  **L182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L183 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.smin(x,y,mask,vlen)`.
  **L183 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.smin(x,y,mask,vlen)`。
- **L184 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L184 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L185 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L185 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L186 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L186 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L187 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L187 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L188 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L188 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L189 EN**: Blank line separating nearby declarations or logic blocks.
  **L189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L190 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.smax(x,y,mask,vlen)`.
  **L190 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.smax(x,y,mask,vlen)`。
- **L191 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L191 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L192 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L192 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。

### Lines 193-216

````cpp
VP_PROPERTY_FUNCTIONAL_SDOPC(SMAX)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(smax)
END_REGISTER_VP(vp_smax, VP_SMAX)

// llvm.vp.umin(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_umin, 2, 3, VP_UMIN, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(UMIN)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(umin)
END_REGISTER_VP(vp_umin, VP_UMIN)

// llvm.vp.umax(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_umax, 2, 3, VP_UMAX, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(UMAX)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(umax)
END_REGISTER_VP(vp_umax, VP_UMAX)

// llvm.vp.abs(x,is_int_min_poison,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_abs, 2, 3)
BEGIN_REGISTER_VP_SDNODE(VP_ABS, -1, vp_abs, 1, 2)
HELPER_MAP_VPID_TO_VPSD(vp_abs, VP_ABS)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(abs)
VP_PROPERTY_FUNCTIONAL_SDOPC(ABS)
````
- **L193 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L193 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L194 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L194 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L195 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L195 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L197 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.umin(x,y,mask,vlen)`.
  **L197 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.umin(x,y,mask,vlen)`。
- **L198 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L198 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L199 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L199 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L200 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L200 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L201 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L201 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L202 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L202 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L203 EN**: Blank line separating nearby declarations or logic blocks.
  **L203 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L204 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.umax(x,y,mask,vlen)`.
  **L204 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.umax(x,y,mask,vlen)`。
- **L205 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L205 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L206 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L206 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L207 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L207 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L208 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L208 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L209 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L209 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.abs(x,is_int_min_poison,mask,vlen)`.
  **L211 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.abs(x,is_int_min_poison,mask,vlen)`。
- **L212 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L212 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L213 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L213 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L214 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L214 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L215 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L215 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L216 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L216 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。

### Lines 217-240

````cpp
END_REGISTER_VP(vp_abs, VP_ABS)

// llvm.vp.bswap(x,mask,vlen)
BEGIN_REGISTER_VP(vp_bswap, 1, 2, VP_BSWAP, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(bswap)
VP_PROPERTY_FUNCTIONAL_SDOPC(BSWAP)
END_REGISTER_VP(vp_bswap, VP_BSWAP)

// llvm.vp.bitreverse(x,mask,vlen)
BEGIN_REGISTER_VP(vp_bitreverse, 1, 2, VP_BITREVERSE, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(bitreverse)
VP_PROPERTY_FUNCTIONAL_SDOPC(BITREVERSE)
END_REGISTER_VP(vp_bitreverse, VP_BITREVERSE)

// llvm.vp.ctpop(x,mask,vlen)
BEGIN_REGISTER_VP(vp_ctpop, 1, 2, VP_CTPOP, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(ctpop)
VP_PROPERTY_FUNCTIONAL_SDOPC(CTPOP)
END_REGISTER_VP(vp_ctpop, VP_CTPOP)

// llvm.vp.ctlz(x,is_zero_poison,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_ctlz, 2, 3)
BEGIN_REGISTER_VP_SDNODE(VP_CTLZ, -1, vp_ctlz, 1, 2)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(ctlz)
````
- **L217 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L217 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L218 EN**: Blank line separating nearby declarations or logic blocks.
  **L218 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L219 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.bswap(x,mask,vlen)`.
  **L219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.bswap(x,mask,vlen)`。
- **L220 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L220 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L221 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L221 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L222 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L222 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L223 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L223 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L224 EN**: Blank line separating nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L225 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.bitreverse(x,mask,vlen)`.
  **L225 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.bitreverse(x,mask,vlen)`。
- **L226 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L226 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L227 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L227 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L228 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L228 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L229 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L229 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L230 EN**: Blank line separating nearby declarations or logic blocks.
  **L230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L231 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.ctpop(x,mask,vlen)`.
  **L231 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.ctpop(x,mask,vlen)`。
- **L232 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L232 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L233 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L233 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L234 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L234 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L235 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L235 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L237 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.ctlz(x,is_zero_poison,mask,vlen)`.
  **L237 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.ctlz(x,is_zero_poison,mask,vlen)`。
- **L238 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L238 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L239 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L239 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L240 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L240 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。

### Lines 241-264

````cpp
VP_PROPERTY_FUNCTIONAL_SDOPC(CTLZ)
END_REGISTER_VP_SDNODE(VP_CTLZ)
BEGIN_REGISTER_VP_SDNODE(VP_CTLZ_ZERO_POISON, -1, vp_ctlz_zero_poison, 1, 2)
VP_PROPERTY_FUNCTIONAL_SDOPC(CTLZ_ZERO_POISON)
END_REGISTER_VP_SDNODE(VP_CTLZ_ZERO_POISON)
END_REGISTER_VP_INTRINSIC(vp_ctlz)

// llvm.vp.cttz(x,is_zero_poison,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_cttz, 2, 3)
BEGIN_REGISTER_VP_SDNODE(VP_CTTZ, -1, vp_cttz, 1, 2)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(cttz)
VP_PROPERTY_FUNCTIONAL_SDOPC(CTTZ)
END_REGISTER_VP_SDNODE(VP_CTTZ)
BEGIN_REGISTER_VP_SDNODE(VP_CTTZ_ZERO_POISON, -1, vp_cttz_zero_poison, 1, 2)
END_REGISTER_VP_SDNODE(VP_CTTZ_ZERO_POISON)
END_REGISTER_VP_INTRINSIC(vp_cttz)

// llvm.vp.cttz.elts(x,is_zero_poison,mask,vl)
BEGIN_REGISTER_VP_INTRINSIC(vp_cttz_elts, 2, 3)
VP_PROPERTY_NO_FUNCTIONAL
BEGIN_REGISTER_VP_SDNODE(VP_CTTZ_ELTS, 0, vp_cttz_elts, 1, 2)
END_REGISTER_VP_SDNODE(VP_CTTZ_ELTS)
BEGIN_REGISTER_VP_SDNODE(VP_CTTZ_ELTS_ZERO_POISON, 0, vp_cttz_elts_zero_poison, 1, 2)
END_REGISTER_VP_SDNODE(VP_CTTZ_ELTS_ZERO_POISON)
````
- **L241 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L241 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L242 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L242 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L243 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L243 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L244 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L244 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L245 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L245 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L246 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_INTRINSIC`.
  **L246 CN**: 继续与可调用符号 `END_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L247 EN**: Blank line separating nearby declarations or logic blocks.
  **L247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L248 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.cttz(x,is_zero_poison,mask,vlen)`.
  **L248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.cttz(x,is_zero_poison,mask,vlen)`。
- **L249 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L249 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L250 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L250 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L251 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L251 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L252 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L252 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L253 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L253 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L254 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L254 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L255 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L255 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L256 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_INTRINSIC`.
  **L256 CN**: 继续与可调用符号 `END_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L257 EN**: Blank line separating nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L258 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.cttz.elts(x,is_zero_poison,mask,vl)`.
  **L258 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.cttz.elts(x,is_zero_poison,mask,vl)`。
- **L259 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L259 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L260 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_NO_FUNCTIONAL`.
  **L260 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_NO_FUNCTIONAL`。
- **L261 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L261 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L262 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L262 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L263 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L263 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L264 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L264 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。

### Lines 265-288

````cpp
END_REGISTER_VP_INTRINSIC(vp_cttz_elts)

// llvm.vp.fshl(x,y,z,mask,vlen)
BEGIN_REGISTER_VP(vp_fshl, 3, 4, VP_FSHL, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(fshl)
VP_PROPERTY_FUNCTIONAL_SDOPC(FSHL)
END_REGISTER_VP(vp_fshl, VP_FSHL)

// llvm.vp.fshr(x,y,z,mask,vlen)
BEGIN_REGISTER_VP(vp_fshr, 3, 4, VP_FSHR, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(fshr)
VP_PROPERTY_FUNCTIONAL_SDOPC(FSHR)
END_REGISTER_VP(vp_fshr, VP_FSHR)

// llvm.vp.sadd.sat(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_sadd_sat, 2, 3, VP_SADDSAT, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_INTRINSIC(sadd_sat)
VP_PROPERTY_FUNCTIONAL_SDOPC(SADDSAT)
END_REGISTER_VP(vp_sadd_sat, VP_SADDSAT)

// llvm.vp.uadd.sat(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_uadd_sat, 2, 3, VP_UADDSAT, -1)
VP_PROPERTY_BINARYOP
````
- **L265 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_INTRINSIC`.
  **L265 CN**: 继续与可调用符号 `END_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L267 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fshl(x,y,z,mask,vlen)`.
  **L267 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fshl(x,y,z,mask,vlen)`。
- **L268 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L268 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L269 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L269 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L270 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L270 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L271 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L271 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L272 EN**: Blank line separating nearby declarations or logic blocks.
  **L272 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L273 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fshr(x,y,z,mask,vlen)`.
  **L273 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fshr(x,y,z,mask,vlen)`。
- **L274 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L274 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L275 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L275 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L276 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L276 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L277 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L277 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L278 EN**: Blank line separating nearby declarations or logic blocks.
  **L278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L279 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.sadd.sat(x,y,mask,vlen)`.
  **L279 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.sadd.sat(x,y,mask,vlen)`。
- **L280 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L280 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L281 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L281 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L282 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L282 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L283 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L283 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L284 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L284 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L285 EN**: Blank line separating nearby declarations or logic blocks.
  **L285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L286 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.uadd.sat(x,y,mask,vlen)`.
  **L286 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.uadd.sat(x,y,mask,vlen)`。
- **L287 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L287 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L288 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L288 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。

### Lines 289-312

````cpp
VP_PROPERTY_FUNCTIONAL_INTRINSIC(uadd_sat)
VP_PROPERTY_FUNCTIONAL_SDOPC(UADDSAT)
END_REGISTER_VP(vp_uadd_sat, VP_UADDSAT)

// llvm.vp.ssub.sat(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_ssub_sat, 2, 3, VP_SSUBSAT, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_INTRINSIC(ssub_sat)
VP_PROPERTY_FUNCTIONAL_SDOPC(SSUBSAT)
END_REGISTER_VP(vp_ssub_sat, VP_SSUBSAT)

// llvm.vp.usub.sat(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_usub_sat, 2, 3, VP_USUBSAT, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_INTRINSIC(usub_sat)
VP_PROPERTY_FUNCTIONAL_SDOPC(USUBSAT)
END_REGISTER_VP(vp_usub_sat, VP_USUBSAT)
///// } Integer Arithmetic

///// Floating-Point Arithmetic {

// Specialized helper macro for floating-point binary operators
// <operation>(%x, %y, %mask, %evl).
#ifdef HELPER_REGISTER_BINARY_FP_VP
````
- **L289 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L289 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L290 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L290 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L291 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L291 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L293 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.ssub.sat(x,y,mask,vlen)`.
  **L293 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.ssub.sat(x,y,mask,vlen)`。
- **L294 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L294 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L295 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L295 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L296 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L296 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L297 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L297 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L298 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L298 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L299 EN**: Blank line separating nearby declarations or logic blocks.
  **L299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L300 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.usub.sat(x,y,mask,vlen)`.
  **L300 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.usub.sat(x,y,mask,vlen)`。
- **L301 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L301 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L302 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L302 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L303 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L303 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L304 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L304 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L305 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L305 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L306 EN**: Comment explains nearby logic, invariants, or intent: `} Integer Arithmetic`.
  **L306 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Integer Arithmetic`。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L308 EN**: Comment explains nearby logic, invariants, or intent: `Floating-Point Arithmetic {`.
  **L308 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floating-Point Arithmetic {`。
- **L309 EN**: Blank line separating nearby declarations or logic blocks.
  **L309 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L310 EN**: Comment explains nearby logic, invariants, or intent: `Specialized helper macro for floating-point binary operators`.
  **L310 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Specialized helper macro for floating-point binary operators`。
- **L311 EN**: Comment explains nearby logic, invariants, or intent: `<operation>(%x, %y, %mask, %evl).`.
  **L311 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<operation>(%x, %y, %mask, %evl).`。
- **L312 EN**: Starts a preprocessor conditional block: `#ifdef HELPER_REGISTER_BINARY_FP_VP`.
  **L312 CN**: 开始一个预处理条件块：`#ifdef HELPER_REGISTER_BINARY_FP_VP`。

### Lines 313-336

````cpp
#error                                                                         \
    "The internal helper macro HELPER_REGISTER_BINARY_FP_VP is already defined!"
#endif
#define HELPER_REGISTER_BINARY_FP_VP(OPSUFFIX, VPSD, IROPC, SDOPC)             \
  BEGIN_REGISTER_VP(vp_##OPSUFFIX, 2, 3, VPSD, -1)                             \
  VP_PROPERTY_FUNCTIONAL_OPC(IROPC)                                            \
  VP_PROPERTY_CONSTRAINEDFP(experimental_constrained_##OPSUFFIX)         \
  VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)                                          \
  VP_PROPERTY_BINARYOP                                                         \
  END_REGISTER_VP(vp_##OPSUFFIX, VPSD)

// llvm.vp.fadd(x,y,mask,vlen)
HELPER_REGISTER_BINARY_FP_VP(fadd, VP_FADD, FAdd, FADD)

// llvm.vp.fsub(x,y,mask,vlen)
HELPER_REGISTER_BINARY_FP_VP(fsub, VP_FSUB, FSub, FSUB)

// llvm.vp.fmul(x,y,mask,vlen)
HELPER_REGISTER_BINARY_FP_VP(fmul, VP_FMUL, FMul, FMUL)

// llvm.vp.fdiv(x,y,mask,vlen)
HELPER_REGISTER_BINARY_FP_VP(fdiv, VP_FDIV, FDiv, FDIV)

// llvm.vp.frem(x,y,mask,vlen)
````
- **L313 EN**: Continues the surrounding expression or declaration: `#error                                                                         \`.
  **L313 CN**: 继续构造周围的表达式或声明：`#error                                                                         \`。
- **L314 EN**: Continues the surrounding expression or declaration: `"The internal helper macro HELPER_REGISTER_BINARY_FP_VP is already defined!"`.
  **L314 CN**: 继续构造周围的表达式或声明：`"The internal helper macro HELPER_REGISTER_BINARY_FP_VP is already defined!"`。
- **L315 EN**: Closes the current preprocessor conditional block.
  **L315 CN**: 结束当前预处理条件块。
- **L316 EN**: Defines macro `HELPER_REGISTER_BINARY_FP_VP(OPSUFFIX,` for conditional compilation, local shorthand, or diagnostics.
  **L316 CN**: 定义宏 `HELPER_REGISTER_BINARY_FP_VP(OPSUFFIX,`，供条件编译、本地简写或诊断使用。
- **L317 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L317 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L318 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L318 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L319 EN**: Continues logic associated with callable symbol `VP_PROPERTY_CONSTRAINEDFP`.
  **L319 CN**: 继续与可调用符号 `VP_PROPERTY_CONSTRAINEDFP` 相关的逻辑。
- **L320 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L320 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L321 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP                                                         \`.
  **L321 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP                                                         \`。
- **L322 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L322 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L323 EN**: Blank line separating nearby declarations or logic blocks.
  **L323 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L324 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fadd(x,y,mask,vlen)`.
  **L324 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fadd(x,y,mask,vlen)`。
- **L325 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_FP_VP`.
  **L325 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_FP_VP` 相关的逻辑。
- **L326 EN**: Blank line separating nearby declarations or logic blocks.
  **L326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L327 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fsub(x,y,mask,vlen)`.
  **L327 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fsub(x,y,mask,vlen)`。
- **L328 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_FP_VP`.
  **L328 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_FP_VP` 相关的逻辑。
- **L329 EN**: Blank line separating nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fmul(x,y,mask,vlen)`.
  **L330 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fmul(x,y,mask,vlen)`。
- **L331 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_FP_VP`.
  **L331 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_FP_VP` 相关的逻辑。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L333 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fdiv(x,y,mask,vlen)`.
  **L333 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fdiv(x,y,mask,vlen)`。
- **L334 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_FP_VP`.
  **L334 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_FP_VP` 相关的逻辑。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L336 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.frem(x,y,mask,vlen)`.
  **L336 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.frem(x,y,mask,vlen)`。

### Lines 337-360

````cpp
HELPER_REGISTER_BINARY_FP_VP(frem, VP_FREM, FRem, FREM)

#undef HELPER_REGISTER_BINARY_FP_VP

// llvm.vp.fneg(x,mask,vlen)
BEGIN_REGISTER_VP(vp_fneg, 1, 2, VP_FNEG, -1)
VP_PROPERTY_FUNCTIONAL_OPC(FNeg)
VP_PROPERTY_FUNCTIONAL_SDOPC(FNEG)
END_REGISTER_VP(vp_fneg, VP_FNEG)

// llvm.vp.fabs(x,mask,vlen)
BEGIN_REGISTER_VP(vp_fabs, 1, 2, VP_FABS, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(fabs)
VP_PROPERTY_FUNCTIONAL_SDOPC(FABS)
END_REGISTER_VP(vp_fabs, VP_FABS)

// llvm.vp.sqrt(x,mask,vlen)
BEGIN_REGISTER_VP(vp_sqrt, 1, 2, VP_SQRT, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(sqrt)
VP_PROPERTY_FUNCTIONAL_SDOPC(FSQRT)
END_REGISTER_VP(vp_sqrt, VP_SQRT)

// llvm.vp.fma(x,y,z,mask,vlen)
BEGIN_REGISTER_VP(vp_fma, 3, 4, VP_FMA, -1)
````
- **L337 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_BINARY_FP_VP`.
  **L337 CN**: 继续与可调用符号 `HELPER_REGISTER_BINARY_FP_VP` 相关的逻辑。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L339 EN**: Undefines a macro to limit its scope: `#undef HELPER_REGISTER_BINARY_FP_VP`.
  **L339 CN**: 取消宏定义以限制其作用域：`#undef HELPER_REGISTER_BINARY_FP_VP`。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L341 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fneg(x,mask,vlen)`.
  **L341 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fneg(x,mask,vlen)`。
- **L342 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L342 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L343 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L343 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L344 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L344 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L345 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L345 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L346 EN**: Blank line separating nearby declarations or logic blocks.
  **L346 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L347 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fabs(x,mask,vlen)`.
  **L347 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fabs(x,mask,vlen)`。
- **L348 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L348 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L349 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L349 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L350 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L350 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L351 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L351 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L353 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.sqrt(x,mask,vlen)`.
  **L353 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.sqrt(x,mask,vlen)`。
- **L354 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L354 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L355 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L355 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L356 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L356 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L357 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L357 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L358 EN**: Blank line separating nearby declarations or logic blocks.
  **L358 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L359 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fma(x,y,z,mask,vlen)`.
  **L359 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fma(x,y,z,mask,vlen)`。
- **L360 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L360 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。

### Lines 361-384

````cpp
VP_PROPERTY_CONSTRAINEDFP(experimental_constrained_fma)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(fma)
VP_PROPERTY_FUNCTIONAL_SDOPC(FMA)
END_REGISTER_VP(vp_fma, VP_FMA)

// llvm.vp.fmuladd(x,y,z,mask,vlen)
BEGIN_REGISTER_VP(vp_fmuladd, 3, 4, VP_FMULADD, -1)
VP_PROPERTY_CONSTRAINEDFP(experimental_constrained_fmuladd)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(fmuladd)
VP_PROPERTY_FUNCTIONAL_SDOPC(FMAD)
END_REGISTER_VP(vp_fmuladd, VP_FMULADD)

// llvm.vp.copysign(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_copysign, 2, 3, VP_FCOPYSIGN, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(FCOPYSIGN)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(copysign)
END_REGISTER_VP(vp_copysign, VP_FCOPYSIGN)

// llvm.vp.minnum(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_minnum, 2, 3, VP_FMINNUM, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(FMINNUM)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(minnum)
````
- **L361 EN**: Continues logic associated with callable symbol `VP_PROPERTY_CONSTRAINEDFP`.
  **L361 CN**: 继续与可调用符号 `VP_PROPERTY_CONSTRAINEDFP` 相关的逻辑。
- **L362 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L362 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L363 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L363 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L364 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L364 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L365 EN**: Blank line separating nearby declarations or logic blocks.
  **L365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L366 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fmuladd(x,y,z,mask,vlen)`.
  **L366 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fmuladd(x,y,z,mask,vlen)`。
- **L367 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L367 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L368 EN**: Continues logic associated with callable symbol `VP_PROPERTY_CONSTRAINEDFP`.
  **L368 CN**: 继续与可调用符号 `VP_PROPERTY_CONSTRAINEDFP` 相关的逻辑。
- **L369 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L369 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L370 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L370 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L371 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L371 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L372 EN**: Blank line separating nearby declarations or logic blocks.
  **L372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L373 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.copysign(x,y,mask,vlen)`.
  **L373 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.copysign(x,y,mask,vlen)`。
- **L374 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L374 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L375 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L375 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L376 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L376 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L377 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L377 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L378 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L378 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L380 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.minnum(x,y,mask,vlen)`.
  **L380 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.minnum(x,y,mask,vlen)`。
- **L381 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L381 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L382 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L382 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L383 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L383 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L384 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L384 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。

### Lines 385-408

````cpp
END_REGISTER_VP(vp_minnum, VP_FMINNUM)

// llvm.vp.maxnum(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_maxnum, 2, 3, VP_FMAXNUM, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(FMAXNUM)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(maxnum)
END_REGISTER_VP(vp_maxnum, VP_FMAXNUM)

// llvm.vp.minimum(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_minimum, 2, 3, VP_FMINIMUM, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(FMINIMUM)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(minimum)
END_REGISTER_VP(vp_minimum, VP_FMINIMUM)

// llvm.vp.maximum(x,y,mask,vlen)
BEGIN_REGISTER_VP(vp_maximum, 2, 3, VP_FMAXIMUM, -1)
VP_PROPERTY_BINARYOP
VP_PROPERTY_FUNCTIONAL_SDOPC(FMAXIMUM)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(maximum)
END_REGISTER_VP(vp_maximum, VP_FMAXIMUM)

// llvm.vp.ceil(x,mask,vlen)
````
- **L385 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L385 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L386 EN**: Blank line separating nearby declarations or logic blocks.
  **L386 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L387 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.maxnum(x,y,mask,vlen)`.
  **L387 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.maxnum(x,y,mask,vlen)`。
- **L388 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L388 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L389 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L389 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L390 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L390 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L391 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L391 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L392 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L392 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L393 EN**: Blank line separating nearby declarations or logic blocks.
  **L393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L394 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.minimum(x,y,mask,vlen)`.
  **L394 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.minimum(x,y,mask,vlen)`。
- **L395 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L395 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L396 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L396 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L397 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L397 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L398 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L398 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L399 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L399 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L401 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.maximum(x,y,mask,vlen)`.
  **L401 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.maximum(x,y,mask,vlen)`。
- **L402 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L402 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L403 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_BINARYOP`.
  **L403 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_BINARYOP`。
- **L404 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L404 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L405 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L405 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L406 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L406 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L407 EN**: Blank line separating nearby declarations or logic blocks.
  **L407 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L408 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.ceil(x,mask,vlen)`.
  **L408 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.ceil(x,mask,vlen)`。

### Lines 409-432

````cpp
BEGIN_REGISTER_VP(vp_ceil, 1, 2, VP_FCEIL, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(ceil)
VP_PROPERTY_FUNCTIONAL_SDOPC(FCEIL)
END_REGISTER_VP(vp_ceil, VP_FCEIL)

// llvm.vp.floor(x,mask,vlen)
BEGIN_REGISTER_VP(vp_floor, 1, 2, VP_FFLOOR, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(floor)
VP_PROPERTY_FUNCTIONAL_SDOPC(FFLOOR)
END_REGISTER_VP(vp_floor, VP_FFLOOR)

// llvm.vp.round(x,mask,vlen)
BEGIN_REGISTER_VP(vp_round, 1, 2, VP_FROUND, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(round)
VP_PROPERTY_FUNCTIONAL_SDOPC(FROUND)
END_REGISTER_VP(vp_round, VP_FROUND)

// llvm.vp.roundeven(x,mask,vlen)
BEGIN_REGISTER_VP(vp_roundeven, 1, 2, VP_FROUNDEVEN, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(roundeven)
VP_PROPERTY_FUNCTIONAL_SDOPC(FROUNDEVEN)
END_REGISTER_VP(vp_roundeven, VP_FROUNDEVEN)

// llvm.vp.roundtozero(x,mask,vlen)
````
- **L409 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L409 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L410 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L410 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L411 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L411 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L412 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L412 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L413 EN**: Blank line separating nearby declarations or logic blocks.
  **L413 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L414 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.floor(x,mask,vlen)`.
  **L414 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.floor(x,mask,vlen)`。
- **L415 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L415 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L416 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L416 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L417 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L417 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L418 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L418 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L419 EN**: Blank line separating nearby declarations or logic blocks.
  **L419 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L420 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.round(x,mask,vlen)`.
  **L420 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.round(x,mask,vlen)`。
- **L421 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L421 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L422 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L422 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L423 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L423 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L424 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L424 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L425 EN**: Blank line separating nearby declarations or logic blocks.
  **L425 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L426 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.roundeven(x,mask,vlen)`.
  **L426 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.roundeven(x,mask,vlen)`。
- **L427 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L427 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L428 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L428 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L429 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L429 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L430 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L430 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L432 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.roundtozero(x,mask,vlen)`.
  **L432 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.roundtozero(x,mask,vlen)`。

### Lines 433-456

````cpp
BEGIN_REGISTER_VP(vp_roundtozero, 1, 2, VP_FROUNDTOZERO, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(trunc)
VP_PROPERTY_FUNCTIONAL_SDOPC(FTRUNC)
END_REGISTER_VP(vp_roundtozero, VP_FROUNDTOZERO)

// llvm.vp.rint(x,mask,vlen)
BEGIN_REGISTER_VP(vp_rint, 1, 2, VP_FRINT, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(rint)
VP_PROPERTY_FUNCTIONAL_SDOPC(FRINT)
END_REGISTER_VP(vp_rint, VP_FRINT)

// llvm.vp.nearbyint(x,mask,vlen)
BEGIN_REGISTER_VP(vp_nearbyint, 1, 2, VP_FNEARBYINT, -1)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(nearbyint)
VP_PROPERTY_FUNCTIONAL_SDOPC(FNEARBYINT)
END_REGISTER_VP(vp_nearbyint, VP_FNEARBYINT)

// llvm.vp.lrint(x,mask,vlen)
BEGIN_REGISTER_VP(vp_lrint, 1, 2, VP_LRINT, 0)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(lrint)
VP_PROPERTY_FUNCTIONAL_SDOPC(LRINT)
END_REGISTER_VP(vp_lrint, VP_LRINT)

// llvm.vp.llrint(x,mask,vlen)
````
- **L433 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L433 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L434 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L434 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L435 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L435 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L436 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L436 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L438 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.rint(x,mask,vlen)`.
  **L438 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.rint(x,mask,vlen)`。
- **L439 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L439 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L440 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L440 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L441 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L441 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L442 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L442 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L444 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.nearbyint(x,mask,vlen)`.
  **L444 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.nearbyint(x,mask,vlen)`。
- **L445 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L445 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L446 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L446 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L447 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L447 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L448 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L448 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L450 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.lrint(x,mask,vlen)`.
  **L450 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.lrint(x,mask,vlen)`。
- **L451 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L451 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L452 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L452 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L453 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L453 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L454 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L454 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L455 EN**: Blank line separating nearby declarations or logic blocks.
  **L455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L456 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.llrint(x,mask,vlen)`.
  **L456 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.llrint(x,mask,vlen)`。

### Lines 457-480

````cpp
BEGIN_REGISTER_VP(vp_llrint, 1, 2, VP_LLRINT, 0)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(llrint)
VP_PROPERTY_FUNCTIONAL_SDOPC(LLRINT)
END_REGISTER_VP(vp_llrint, VP_LLRINT)

///// } Floating-Point Arithmetic

///// Type Casts {
// Specialized helper macro for type conversions.
// <operation>(%x, %mask, %evl).
#ifdef HELPER_REGISTER_FP_CAST_VP
#error                                                                         \
    "The internal helper macro HELPER_REGISTER_FP_CAST_VP is already defined!"
#endif
#define HELPER_REGISTER_FP_CAST_VP(OPSUFFIX, VPSD, IROPC, SDOPC)               \
  BEGIN_REGISTER_VP(vp_##OPSUFFIX, 1, 2, VPSD, -1)                             \
  VP_PROPERTY_FUNCTIONAL_OPC(IROPC)                                            \
  VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)                                          \
  VP_PROPERTY_CONSTRAINEDFP(experimental_constrained_##OPSUFFIX)  \
  END_REGISTER_VP(vp_##OPSUFFIX, VPSD)

// llvm.vp.fptoui(x,mask,vlen)
HELPER_REGISTER_FP_CAST_VP(fptoui, VP_FP_TO_UINT, FPToUI, FP_TO_UINT)

````
- **L457 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L457 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L458 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L458 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L459 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L459 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L460 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L460 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L461 EN**: Blank line separating nearby declarations or logic blocks.
  **L461 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L462 EN**: Comment explains nearby logic, invariants, or intent: `} Floating-Point Arithmetic`.
  **L462 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Floating-Point Arithmetic`。
- **L463 EN**: Blank line separating nearby declarations or logic blocks.
  **L463 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L464 EN**: Comment explains nearby logic, invariants, or intent: `Type Casts {`.
  **L464 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Type Casts {`。
- **L465 EN**: Comment explains nearby logic, invariants, or intent: `Specialized helper macro for type conversions.`.
  **L465 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Specialized helper macro for type conversions.`。
- **L466 EN**: Comment explains nearby logic, invariants, or intent: `<operation>(%x, %mask, %evl).`.
  **L466 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<operation>(%x, %mask, %evl).`。
- **L467 EN**: Starts a preprocessor conditional block: `#ifdef HELPER_REGISTER_FP_CAST_VP`.
  **L467 CN**: 开始一个预处理条件块：`#ifdef HELPER_REGISTER_FP_CAST_VP`。
- **L468 EN**: Continues the surrounding expression or declaration: `#error                                                                         \`.
  **L468 CN**: 继续构造周围的表达式或声明：`#error                                                                         \`。
- **L469 EN**: Continues the surrounding expression or declaration: `"The internal helper macro HELPER_REGISTER_FP_CAST_VP is already defined!"`.
  **L469 CN**: 继续构造周围的表达式或声明：`"The internal helper macro HELPER_REGISTER_FP_CAST_VP is already defined!"`。
- **L470 EN**: Closes the current preprocessor conditional block.
  **L470 CN**: 结束当前预处理条件块。
- **L471 EN**: Defines macro `HELPER_REGISTER_FP_CAST_VP(OPSUFFIX,` for conditional compilation, local shorthand, or diagnostics.
  **L471 CN**: 定义宏 `HELPER_REGISTER_FP_CAST_VP(OPSUFFIX,`，供条件编译、本地简写或诊断使用。
- **L472 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L472 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L473 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L473 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L474 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L474 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L475 EN**: Continues logic associated with callable symbol `VP_PROPERTY_CONSTRAINEDFP`.
  **L475 CN**: 继续与可调用符号 `VP_PROPERTY_CONSTRAINEDFP` 相关的逻辑。
- **L476 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L476 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L477 EN**: Blank line separating nearby declarations or logic blocks.
  **L477 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L478 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fptoui(x,mask,vlen)`.
  **L478 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fptoui(x,mask,vlen)`。
- **L479 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_FP_CAST_VP`.
  **L479 CN**: 继续与可调用符号 `HELPER_REGISTER_FP_CAST_VP` 相关的逻辑。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 481-504

````cpp
// llvm.vp.fptosi(x,mask,vlen)
HELPER_REGISTER_FP_CAST_VP(fptosi, VP_FP_TO_SINT, FPToSI, FP_TO_SINT)

// llvm.vp.uitofp(x,mask,vlen)
HELPER_REGISTER_FP_CAST_VP(uitofp, VP_UINT_TO_FP, UIToFP, UINT_TO_FP)

// llvm.vp.sitofp(x,mask,vlen)
HELPER_REGISTER_FP_CAST_VP(sitofp, VP_SINT_TO_FP, SIToFP, SINT_TO_FP)

// llvm.vp.fptrunc(x,mask,vlen)
HELPER_REGISTER_FP_CAST_VP(fptrunc, VP_FP_ROUND, FPTrunc, FP_ROUND)

// llvm.vp.fpext(x,mask,vlen)
HELPER_REGISTER_FP_CAST_VP(fpext, VP_FP_EXTEND, FPExt, FP_EXTEND)

#undef HELPER_REGISTER_FP_CAST_VP

// Specialized helper macro for integer type conversions.
// <operation>(%x, %mask, %evl).
#ifdef HELPER_REGISTER_INT_CAST_VP
#error                                                                         \
    "The internal helper macro HELPER_REGISTER_INT_CAST_VP is already defined!"
#endif
#define HELPER_REGISTER_INT_CAST_VP(OPSUFFIX, VPSD, IROPC, SDOPC)              \
````
- **L481 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fptosi(x,mask,vlen)`.
  **L481 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fptosi(x,mask,vlen)`。
- **L482 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_FP_CAST_VP`.
  **L482 CN**: 继续与可调用符号 `HELPER_REGISTER_FP_CAST_VP` 相关的逻辑。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L484 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.uitofp(x,mask,vlen)`.
  **L484 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.uitofp(x,mask,vlen)`。
- **L485 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_FP_CAST_VP`.
  **L485 CN**: 继续与可调用符号 `HELPER_REGISTER_FP_CAST_VP` 相关的逻辑。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L487 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.sitofp(x,mask,vlen)`.
  **L487 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.sitofp(x,mask,vlen)`。
- **L488 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_FP_CAST_VP`.
  **L488 CN**: 继续与可调用符号 `HELPER_REGISTER_FP_CAST_VP` 相关的逻辑。
- **L489 EN**: Blank line separating nearby declarations or logic blocks.
  **L489 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L490 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fptrunc(x,mask,vlen)`.
  **L490 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fptrunc(x,mask,vlen)`。
- **L491 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_FP_CAST_VP`.
  **L491 CN**: 继续与可调用符号 `HELPER_REGISTER_FP_CAST_VP` 相关的逻辑。
- **L492 EN**: Blank line separating nearby declarations or logic blocks.
  **L492 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L493 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fpext(x,mask,vlen)`.
  **L493 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fpext(x,mask,vlen)`。
- **L494 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_FP_CAST_VP`.
  **L494 CN**: 继续与可调用符号 `HELPER_REGISTER_FP_CAST_VP` 相关的逻辑。
- **L495 EN**: Blank line separating nearby declarations or logic blocks.
  **L495 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L496 EN**: Undefines a macro to limit its scope: `#undef HELPER_REGISTER_FP_CAST_VP`.
  **L496 CN**: 取消宏定义以限制其作用域：`#undef HELPER_REGISTER_FP_CAST_VP`。
- **L497 EN**: Blank line separating nearby declarations or logic blocks.
  **L497 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L498 EN**: Comment explains nearby logic, invariants, or intent: `Specialized helper macro for integer type conversions.`.
  **L498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Specialized helper macro for integer type conversions.`。
- **L499 EN**: Comment explains nearby logic, invariants, or intent: `<operation>(%x, %mask, %evl).`.
  **L499 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`<operation>(%x, %mask, %evl).`。
- **L500 EN**: Starts a preprocessor conditional block: `#ifdef HELPER_REGISTER_INT_CAST_VP`.
  **L500 CN**: 开始一个预处理条件块：`#ifdef HELPER_REGISTER_INT_CAST_VP`。
- **L501 EN**: Continues the surrounding expression or declaration: `#error                                                                         \`.
  **L501 CN**: 继续构造周围的表达式或声明：`#error                                                                         \`。
- **L502 EN**: Continues the surrounding expression or declaration: `"The internal helper macro HELPER_REGISTER_INT_CAST_VP is already defined!"`.
  **L502 CN**: 继续构造周围的表达式或声明：`"The internal helper macro HELPER_REGISTER_INT_CAST_VP is already defined!"`。
- **L503 EN**: Closes the current preprocessor conditional block.
  **L503 CN**: 结束当前预处理条件块。
- **L504 EN**: Defines macro `HELPER_REGISTER_INT_CAST_VP(OPSUFFIX,` for conditional compilation, local shorthand, or diagnostics.
  **L504 CN**: 定义宏 `HELPER_REGISTER_INT_CAST_VP(OPSUFFIX,`，供条件编译、本地简写或诊断使用。

### Lines 505-528

````cpp
  BEGIN_REGISTER_VP(vp_##OPSUFFIX, 1, 2, VPSD, -1)                             \
  VP_PROPERTY_FUNCTIONAL_OPC(IROPC)                                            \
  VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)                                          \
  END_REGISTER_VP(vp_##OPSUFFIX, VPSD)

// llvm.vp.trunc(x,mask,vlen)
HELPER_REGISTER_INT_CAST_VP(trunc, VP_TRUNCATE, Trunc, TRUNCATE)

// llvm.vp.zext(x,mask,vlen)
HELPER_REGISTER_INT_CAST_VP(zext, VP_ZERO_EXTEND, ZExt, ZERO_EXTEND)

// llvm.vp.sext(x,mask,vlen)
HELPER_REGISTER_INT_CAST_VP(sext, VP_SIGN_EXTEND, SExt, SIGN_EXTEND)

// llvm.vp.ptrtoint(x,mask,vlen)
BEGIN_REGISTER_VP(vp_ptrtoint, 1, 2, VP_PTRTOINT, -1)
VP_PROPERTY_FUNCTIONAL_OPC(PtrToInt)
END_REGISTER_VP(vp_ptrtoint, VP_PTRTOINT)

// llvm.vp.inttoptr(x,mask,vlen)
BEGIN_REGISTER_VP(vp_inttoptr, 1, 2, VP_INTTOPTR, -1)
VP_PROPERTY_FUNCTIONAL_OPC(IntToPtr)
END_REGISTER_VP(vp_inttoptr, VP_INTTOPTR)

````
- **L505 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L505 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L506 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L506 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L507 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L507 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L508 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L508 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L510 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.trunc(x,mask,vlen)`.
  **L510 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.trunc(x,mask,vlen)`。
- **L511 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_INT_CAST_VP`.
  **L511 CN**: 继续与可调用符号 `HELPER_REGISTER_INT_CAST_VP` 相关的逻辑。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L513 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.zext(x,mask,vlen)`.
  **L513 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.zext(x,mask,vlen)`。
- **L514 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_INT_CAST_VP`.
  **L514 CN**: 继续与可调用符号 `HELPER_REGISTER_INT_CAST_VP` 相关的逻辑。
- **L515 EN**: Blank line separating nearby declarations or logic blocks.
  **L515 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L516 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.sext(x,mask,vlen)`.
  **L516 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.sext(x,mask,vlen)`。
- **L517 EN**: Continues logic associated with callable symbol `HELPER_REGISTER_INT_CAST_VP`.
  **L517 CN**: 继续与可调用符号 `HELPER_REGISTER_INT_CAST_VP` 相关的逻辑。
- **L518 EN**: Blank line separating nearby declarations or logic blocks.
  **L518 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L519 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.ptrtoint(x,mask,vlen)`.
  **L519 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.ptrtoint(x,mask,vlen)`。
- **L520 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L520 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L521 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L521 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L522 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L522 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L523 EN**: Blank line separating nearby declarations or logic blocks.
  **L523 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L524 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.inttoptr(x,mask,vlen)`.
  **L524 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.inttoptr(x,mask,vlen)`。
- **L525 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L525 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L526 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L526 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L527 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L527 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L528 EN**: Blank line separating nearby declarations or logic blocks.
  **L528 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 529-552

````cpp
#undef HELPER_REGISTER_INT_CAST_VP

///// } Type Casts

///// Comparisons {

// VP_SETCC (ISel only)
BEGIN_REGISTER_VP_SDNODE(VP_SETCC, 0, vp_setcc, 3, 4)
END_REGISTER_VP_SDNODE(VP_SETCC)

// llvm.vp.fcmp(x,y,cc,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_fcmp, 3, 4)
HELPER_MAP_VPID_TO_VPSD(vp_fcmp, VP_SETCC)
VP_PROPERTY_FUNCTIONAL_OPC(FCmp)
VP_PROPERTY_CONSTRAINEDFP(experimental_constrained_fcmp)
END_REGISTER_VP_INTRINSIC(vp_fcmp)

// llvm.vp.icmp(x,y,cc,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_icmp, 3, 4)
HELPER_MAP_VPID_TO_VPSD(vp_icmp, VP_SETCC)
VP_PROPERTY_FUNCTIONAL_OPC(ICmp)
END_REGISTER_VP_INTRINSIC(vp_icmp)

///// } Comparisons
````
- **L529 EN**: Undefines a macro to limit its scope: `#undef HELPER_REGISTER_INT_CAST_VP`.
  **L529 CN**: 取消宏定义以限制其作用域：`#undef HELPER_REGISTER_INT_CAST_VP`。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L531 EN**: Comment explains nearby logic, invariants, or intent: `} Type Casts`.
  **L531 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Type Casts`。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L533 EN**: Comment explains nearby logic, invariants, or intent: `Comparisons {`.
  **L533 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Comparisons {`。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L535 EN**: Comment explains nearby logic, invariants, or intent: `VP_SETCC (ISel only)`.
  **L535 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VP_SETCC (ISel only)`。
- **L536 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L536 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L537 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L537 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L538 EN**: Blank line separating nearby declarations or logic blocks.
  **L538 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L539 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.fcmp(x,y,cc,mask,vlen)`.
  **L539 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.fcmp(x,y,cc,mask,vlen)`。
- **L540 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L540 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L541 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L541 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L542 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L542 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L543 EN**: Continues logic associated with callable symbol `VP_PROPERTY_CONSTRAINEDFP`.
  **L543 CN**: 继续与可调用符号 `VP_PROPERTY_CONSTRAINEDFP` 相关的逻辑。
- **L544 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_INTRINSIC`.
  **L544 CN**: 继续与可调用符号 `END_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L546 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.icmp(x,y,cc,mask,vlen)`.
  **L546 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.icmp(x,y,cc,mask,vlen)`。
- **L547 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L547 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L548 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L548 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L549 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L549 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L550 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_INTRINSIC`.
  **L550 CN**: 继续与可调用符号 `END_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L551 EN**: Blank line separating nearby declarations or logic blocks.
  **L551 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L552 EN**: Comment explains nearby logic, invariants, or intent: `} Comparisons`.
  **L552 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Comparisons`。

### Lines 553-576

````cpp

// llvm.vp.is.fpclass(on_true,on_false,mask,vlen)
BEGIN_REGISTER_VP(vp_is_fpclass, 2, 3, VP_IS_FPCLASS, 0)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(is_fpclass)
END_REGISTER_VP(vp_is_fpclass, VP_IS_FPCLASS)

///// Memory Operations {
// llvm.vp.store(val,ptr,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_store, 2, 3)
// chain = VP_STORE chain,val,base,offset,mask,evl
BEGIN_REGISTER_VP_SDNODE(VP_STORE, 1, vp_store, 4, 5)
HELPER_MAP_VPID_TO_VPSD(vp_store, VP_STORE)
VP_PROPERTY_FUNCTIONAL_OPC(Store)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(masked_store)
END_REGISTER_VP(vp_store, VP_STORE)

// llvm.experimental.vp.strided.store(val,ptr,stride,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(experimental_vp_strided_store, 3, 4)
// chain = EXPERIMENTAL_VP_STRIDED_STORE chain,val,base,offset,stride,mask,evl
VP_PROPERTY_NO_FUNCTIONAL
BEGIN_REGISTER_VP_SDNODE(EXPERIMENTAL_VP_STRIDED_STORE, 1, experimental_vp_strided_store, 5, 6)
HELPER_MAP_VPID_TO_VPSD(experimental_vp_strided_store, EXPERIMENTAL_VP_STRIDED_STORE)
END_REGISTER_VP(experimental_vp_strided_store, EXPERIMENTAL_VP_STRIDED_STORE)

````
- **L553 EN**: Blank line separating nearby declarations or logic blocks.
  **L553 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L554 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.is.fpclass(on_true,on_false,mask,vlen)`.
  **L554 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.is.fpclass(on_true,on_false,mask,vlen)`。
- **L555 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L555 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L556 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L556 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L557 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L557 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L558 EN**: Blank line separating nearby declarations or logic blocks.
  **L558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L559 EN**: Comment explains nearby logic, invariants, or intent: `Memory Operations {`.
  **L559 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Memory Operations {`。
- **L560 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.store(val,ptr,mask,vlen)`.
  **L560 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.store(val,ptr,mask,vlen)`。
- **L561 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L561 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L562 EN**: Comment explains nearby logic, invariants, or intent: `chain = VP_STORE chain,val,base,offset,mask,evl`.
  **L562 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`chain = VP_STORE chain,val,base,offset,mask,evl`。
- **L563 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L563 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L564 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L564 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L565 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L565 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L566 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L566 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L567 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L567 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L569 EN**: Comment explains nearby logic, invariants, or intent: `llvm.experimental.vp.strided.store(val,ptr,stride,mask,vlen)`.
  **L569 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.experimental.vp.strided.store(val,ptr,stride,mask,vlen)`。
- **L570 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L570 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L571 EN**: Comment explains nearby logic, invariants, or intent: `chain = EXPERIMENTAL_VP_STRIDED_STORE chain,val,base,offset,stride,mask,evl`.
  **L571 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`chain = EXPERIMENTAL_VP_STRIDED_STORE chain,val,base,offset,stride,mask,evl`。
- **L572 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_NO_FUNCTIONAL`.
  **L572 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_NO_FUNCTIONAL`。
- **L573 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L573 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L574 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L574 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L575 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L575 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L576 EN**: Blank line separating nearby declarations or logic blocks.
  **L576 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 577-600

````cpp
// llvm.vp.scatter(ptr,val,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_scatter, 2, 3)
// chain = VP_SCATTER chain,val,base,indices,scale,mask,evl
BEGIN_REGISTER_VP_SDNODE(VP_SCATTER, 1, vp_scatter, 5, 6)
HELPER_MAP_VPID_TO_VPSD(vp_scatter, VP_SCATTER)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(masked_scatter)
END_REGISTER_VP(vp_scatter, VP_SCATTER)

// llvm.vp.load(ptr,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_load, 1, 2)
// val,chain = VP_LOAD chain,base,offset,mask,evl
BEGIN_REGISTER_VP_SDNODE(VP_LOAD, -1, vp_load, 3, 4)
HELPER_MAP_VPID_TO_VPSD(vp_load, VP_LOAD)
VP_PROPERTY_FUNCTIONAL_OPC(Load)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(masked_load)
END_REGISTER_VP(vp_load, VP_LOAD)

BEGIN_REGISTER_VP_INTRINSIC(vp_load_ff, 1, 2)
// val,chain = VP_LOAD_FF chain,base,mask,evl
BEGIN_REGISTER_VP_SDNODE(VP_LOAD_FF, -1, vp_load_ff, 2, 3)
HELPER_MAP_VPID_TO_VPSD(vp_load_ff, VP_LOAD_FF)
VP_PROPERTY_NO_FUNCTIONAL
END_REGISTER_VP(vp_load_ff, VP_LOAD_FF)
// llvm.experimental.vp.strided.load(ptr,stride,mask,vlen)
````
- **L577 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.scatter(ptr,val,mask,vlen)`.
  **L577 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.scatter(ptr,val,mask,vlen)`。
- **L578 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L578 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L579 EN**: Comment explains nearby logic, invariants, or intent: `chain = VP_SCATTER chain,val,base,indices,scale,mask,evl`.
  **L579 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`chain = VP_SCATTER chain,val,base,indices,scale,mask,evl`。
- **L580 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L580 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L581 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L581 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L582 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L582 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L583 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L583 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L585 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.load(ptr,mask,vlen)`.
  **L585 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.load(ptr,mask,vlen)`。
- **L586 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L586 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L587 EN**: Comment explains nearby logic, invariants, or intent: `val,chain = VP_LOAD chain,base,offset,mask,evl`.
  **L587 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`val,chain = VP_LOAD chain,base,offset,mask,evl`。
- **L588 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L588 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L589 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L589 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L590 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L590 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L591 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L591 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L592 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L592 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L593 EN**: Blank line separating nearby declarations or logic blocks.
  **L593 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L594 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L594 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L595 EN**: Comment explains nearby logic, invariants, or intent: `val,chain = VP_LOAD_FF chain,base,mask,evl`.
  **L595 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`val,chain = VP_LOAD_FF chain,base,mask,evl`。
- **L596 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L596 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L597 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L597 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L598 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_NO_FUNCTIONAL`.
  **L598 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_NO_FUNCTIONAL`。
- **L599 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L599 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L600 EN**: Comment explains nearby logic, invariants, or intent: `llvm.experimental.vp.strided.load(ptr,stride,mask,vlen)`.
  **L600 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.experimental.vp.strided.load(ptr,stride,mask,vlen)`。

### Lines 601-624

````cpp
BEGIN_REGISTER_VP_INTRINSIC(experimental_vp_strided_load, 2, 3)
// chain = EXPERIMENTAL_VP_STRIDED_LOAD chain,base,offset,stride,mask,evl
VP_PROPERTY_NO_FUNCTIONAL
BEGIN_REGISTER_VP_SDNODE(EXPERIMENTAL_VP_STRIDED_LOAD, -1, experimental_vp_strided_load, 4, 5)
HELPER_MAP_VPID_TO_VPSD(experimental_vp_strided_load, EXPERIMENTAL_VP_STRIDED_LOAD)
END_REGISTER_VP(experimental_vp_strided_load, EXPERIMENTAL_VP_STRIDED_LOAD)

// llvm.vp.gather(ptr,mask,vlen)
BEGIN_REGISTER_VP_INTRINSIC(vp_gather, 1, 2)
// val,chain = VP_GATHER chain,base,indices,scale,mask,evl
BEGIN_REGISTER_VP_SDNODE(VP_GATHER, -1, vp_gather, 4, 5)
HELPER_MAP_VPID_TO_VPSD(vp_gather, VP_GATHER)
VP_PROPERTY_FUNCTIONAL_INTRINSIC(masked_gather)
END_REGISTER_VP(vp_gather, VP_GATHER)

///// } Memory Operations

///// Reductions {

// Specialized helper macro for VP reductions (%start, %x, %mask, %evl).
#ifdef HELPER_REGISTER_REDUCTION_VP
#error                                                                         \
    "The internal helper macro HELPER_REGISTER_REDUCTION_VP is already defined!"
#endif
````
- **L601 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L601 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L602 EN**: Comment explains nearby logic, invariants, or intent: `chain = EXPERIMENTAL_VP_STRIDED_LOAD chain,base,offset,stride,mask,evl`.
  **L602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`chain = EXPERIMENTAL_VP_STRIDED_LOAD chain,base,offset,stride,mask,evl`。
- **L603 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_NO_FUNCTIONAL`.
  **L603 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_NO_FUNCTIONAL`。
- **L604 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L604 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L605 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L605 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L606 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L606 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L608 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.gather(ptr,mask,vlen)`.
  **L608 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.gather(ptr,mask,vlen)`。
- **L609 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L609 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L610 EN**: Comment explains nearby logic, invariants, or intent: `val,chain = VP_GATHER chain,base,indices,scale,mask,evl`.
  **L610 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`val,chain = VP_GATHER chain,base,indices,scale,mask,evl`。
- **L611 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L611 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L612 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L612 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L613 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L613 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L614 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L614 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L615 EN**: Blank line separating nearby declarations or logic blocks.
  **L615 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L616 EN**: Comment explains nearby logic, invariants, or intent: `} Memory Operations`.
  **L616 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Memory Operations`。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L618 EN**: Comment explains nearby logic, invariants, or intent: `Reductions {`.
  **L618 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reductions {`。
- **L619 EN**: Blank line separating nearby declarations or logic blocks.
  **L619 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L620 EN**: Comment explains nearby logic, invariants, or intent: `Specialized helper macro for VP reductions (%start, %x, %mask, %evl).`.
  **L620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Specialized helper macro for VP reductions (%start, %x, %mask, %evl).`。
- **L621 EN**: Starts a preprocessor conditional block: `#ifdef HELPER_REGISTER_REDUCTION_VP`.
  **L621 CN**: 开始一个预处理条件块：`#ifdef HELPER_REGISTER_REDUCTION_VP`。
- **L622 EN**: Continues the surrounding expression or declaration: `#error                                                                         \`.
  **L622 CN**: 继续构造周围的表达式或声明：`#error                                                                         \`。
- **L623 EN**: Continues the surrounding expression or declaration: `"The internal helper macro HELPER_REGISTER_REDUCTION_VP is already defined!"`.
  **L623 CN**: 继续构造周围的表达式或声明：`"The internal helper macro HELPER_REGISTER_REDUCTION_VP is already defined!"`。
- **L624 EN**: Closes the current preprocessor conditional block.
  **L624 CN**: 结束当前预处理条件块。

### Lines 625-648

````cpp
#define HELPER_REGISTER_REDUCTION_VP(VPID, VPSD, INTRIN, SDOPC)                \
  BEGIN_REGISTER_VP(VPID, 2, 3, VPSD, 1)                                       \
  VP_PROPERTY_FUNCTIONAL_INTRINSIC(INTRIN)                                     \
  VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)                                          \
  END_REGISTER_VP(VPID, VPSD)

// llvm.vp.reduce.add(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_add, VP_REDUCE_ADD,
                             vector_reduce_add, VECREDUCE_ADD)

// llvm.vp.reduce.mul(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_mul, VP_REDUCE_MUL,
                             vector_reduce_mul, VECREDUCE_MUL)

// llvm.vp.reduce.and(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_and, VP_REDUCE_AND,
                             vector_reduce_and, VECREDUCE_AND)

// llvm.vp.reduce.or(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_or, VP_REDUCE_OR,
                             vector_reduce_or, VECREDUCE_OR)

// llvm.vp.reduce.xor(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_xor, VP_REDUCE_XOR,
````
- **L625 EN**: Defines macro `HELPER_REGISTER_REDUCTION_VP(VPID,` for conditional compilation, local shorthand, or diagnostics.
  **L625 CN**: 定义宏 `HELPER_REGISTER_REDUCTION_VP(VPID,`，供条件编译、本地简写或诊断使用。
- **L626 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L626 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L627 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L627 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L628 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L628 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L629 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L629 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L630 EN**: Blank line separating nearby declarations or logic blocks.
  **L630 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L631 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.add(start,x,mask,vlen)`.
  **L631 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.add(start,x,mask,vlen)`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_add, VP_REDUCE_ADD,`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_add, VP_REDUCE_ADD,`。
- **L633 EN**: Continues the surrounding expression or declaration: `vector_reduce_add, VECREDUCE_ADD)`.
  **L633 CN**: 继续构造周围的表达式或声明：`vector_reduce_add, VECREDUCE_ADD)`。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L635 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.mul(start,x,mask,vlen)`.
  **L635 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.mul(start,x,mask,vlen)`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_mul, VP_REDUCE_MUL,`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_mul, VP_REDUCE_MUL,`。
- **L637 EN**: Continues the surrounding expression or declaration: `vector_reduce_mul, VECREDUCE_MUL)`.
  **L637 CN**: 继续构造周围的表达式或声明：`vector_reduce_mul, VECREDUCE_MUL)`。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L639 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.and(start,x,mask,vlen)`.
  **L639 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.and(start,x,mask,vlen)`。
- **L640 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_and, VP_REDUCE_AND,`.
  **L640 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_and, VP_REDUCE_AND,`。
- **L641 EN**: Continues the surrounding expression or declaration: `vector_reduce_and, VECREDUCE_AND)`.
  **L641 CN**: 继续构造周围的表达式或声明：`vector_reduce_and, VECREDUCE_AND)`。
- **L642 EN**: Blank line separating nearby declarations or logic blocks.
  **L642 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L643 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.or(start,x,mask,vlen)`.
  **L643 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.or(start,x,mask,vlen)`。
- **L644 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_or, VP_REDUCE_OR,`.
  **L644 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_or, VP_REDUCE_OR,`。
- **L645 EN**: Continues the surrounding expression or declaration: `vector_reduce_or, VECREDUCE_OR)`.
  **L645 CN**: 继续构造周围的表达式或声明：`vector_reduce_or, VECREDUCE_OR)`。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L647 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.xor(start,x,mask,vlen)`.
  **L647 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.xor(start,x,mask,vlen)`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_xor, VP_REDUCE_XOR,`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_xor, VP_REDUCE_XOR,`。

### Lines 649-672

````cpp
                             vector_reduce_xor, VECREDUCE_XOR)

// llvm.vp.reduce.smax(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_smax, VP_REDUCE_SMAX,
                             vector_reduce_smax, VECREDUCE_SMAX)

// llvm.vp.reduce.smin(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_smin, VP_REDUCE_SMIN,
                             vector_reduce_smin, VECREDUCE_SMIN)

// llvm.vp.reduce.umax(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_umax, VP_REDUCE_UMAX,
                             vector_reduce_umax, VECREDUCE_UMAX)

// llvm.vp.reduce.umin(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_umin, VP_REDUCE_UMIN,
                             vector_reduce_umin, VECREDUCE_UMIN)

// llvm.vp.reduce.fmax(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmax, VP_REDUCE_FMAX,
                             vector_reduce_fmax, VECREDUCE_FMAX)

// llvm.vp.reduce.fmin(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmin, VP_REDUCE_FMIN,
````
- **L649 EN**: Continues the surrounding expression or declaration: `vector_reduce_xor, VECREDUCE_XOR)`.
  **L649 CN**: 继续构造周围的表达式或声明：`vector_reduce_xor, VECREDUCE_XOR)`。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L651 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.smax(start,x,mask,vlen)`.
  **L651 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.smax(start,x,mask,vlen)`。
- **L652 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_smax, VP_REDUCE_SMAX,`.
  **L652 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_smax, VP_REDUCE_SMAX,`。
- **L653 EN**: Continues the surrounding expression or declaration: `vector_reduce_smax, VECREDUCE_SMAX)`.
  **L653 CN**: 继续构造周围的表达式或声明：`vector_reduce_smax, VECREDUCE_SMAX)`。
- **L654 EN**: Blank line separating nearby declarations or logic blocks.
  **L654 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L655 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.smin(start,x,mask,vlen)`.
  **L655 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.smin(start,x,mask,vlen)`。
- **L656 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_smin, VP_REDUCE_SMIN,`.
  **L656 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_smin, VP_REDUCE_SMIN,`。
- **L657 EN**: Continues the surrounding expression or declaration: `vector_reduce_smin, VECREDUCE_SMIN)`.
  **L657 CN**: 继续构造周围的表达式或声明：`vector_reduce_smin, VECREDUCE_SMIN)`。
- **L658 EN**: Blank line separating nearby declarations or logic blocks.
  **L658 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L659 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.umax(start,x,mask,vlen)`.
  **L659 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.umax(start,x,mask,vlen)`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_umax, VP_REDUCE_UMAX,`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_umax, VP_REDUCE_UMAX,`。
- **L661 EN**: Continues the surrounding expression or declaration: `vector_reduce_umax, VECREDUCE_UMAX)`.
  **L661 CN**: 继续构造周围的表达式或声明：`vector_reduce_umax, VECREDUCE_UMAX)`。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L663 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.umin(start,x,mask,vlen)`.
  **L663 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.umin(start,x,mask,vlen)`。
- **L664 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_umin, VP_REDUCE_UMIN,`.
  **L664 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_umin, VP_REDUCE_UMIN,`。
- **L665 EN**: Continues the surrounding expression or declaration: `vector_reduce_umin, VECREDUCE_UMIN)`.
  **L665 CN**: 继续构造周围的表达式或声明：`vector_reduce_umin, VECREDUCE_UMIN)`。
- **L666 EN**: Blank line separating nearby declarations or logic blocks.
  **L666 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L667 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.fmax(start,x,mask,vlen)`.
  **L667 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.fmax(start,x,mask,vlen)`。
- **L668 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmax, VP_REDUCE_FMAX,`.
  **L668 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmax, VP_REDUCE_FMAX,`。
- **L669 EN**: Continues the surrounding expression or declaration: `vector_reduce_fmax, VECREDUCE_FMAX)`.
  **L669 CN**: 继续构造周围的表达式或声明：`vector_reduce_fmax, VECREDUCE_FMAX)`。
- **L670 EN**: Blank line separating nearby declarations or logic blocks.
  **L670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L671 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.fmin(start,x,mask,vlen)`.
  **L671 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.fmin(start,x,mask,vlen)`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmin, VP_REDUCE_FMIN,`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmin, VP_REDUCE_FMIN,`。

### Lines 673-696

````cpp
                             vector_reduce_fmin, VECREDUCE_FMIN)

// llvm.vp.reduce.fmaximum(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmaximum, VP_REDUCE_FMAXIMUM,
                             vector_reduce_fmaximum, VECREDUCE_FMAXIMUM)

// llvm.vp.reduce.fminimum(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_VP(vp_reduce_fminimum, VP_REDUCE_FMINIMUM,
                             vector_reduce_fminimum, VECREDUCE_FMINIMUM)

#undef HELPER_REGISTER_REDUCTION_VP

// Specialized helper macro for VP reductions as above but with two forms:
// sequential and reassociative. These manifest as the presence of 'reassoc'
// fast-math flags in the IR and as two distinct ISD opcodes in the
// SelectionDAG.
// Note we by default map from the VP intrinsic to the SEQ ISD opcode, which
// can then be relaxed to the non-SEQ ISD opcode if the 'reassoc' flag is set.
#ifdef HELPER_REGISTER_REDUCTION_SEQ_VP
#error                                                                         \
    "The internal helper macro HELPER_REGISTER_REDUCTION_SEQ_VP is already defined!"
#endif
#define HELPER_REGISTER_REDUCTION_SEQ_VP(VPID, VPSD, SEQ_VPSD, SDOPC, SEQ_SDOPC, INTRIN) \
  BEGIN_REGISTER_VP_INTRINSIC(VPID, 2, 3)                                      \
````
- **L673 EN**: Continues the surrounding expression or declaration: `vector_reduce_fmin, VECREDUCE_FMIN)`.
  **L673 CN**: 继续构造周围的表达式或声明：`vector_reduce_fmin, VECREDUCE_FMIN)`。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L675 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.fmaximum(start,x,mask,vlen)`.
  **L675 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.fmaximum(start,x,mask,vlen)`。
- **L676 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmaximum, VP_REDUCE_FMAXIMUM,`.
  **L676 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_fmaximum, VP_REDUCE_FMAXIMUM,`。
- **L677 EN**: Continues the surrounding expression or declaration: `vector_reduce_fmaximum, VECREDUCE_FMAXIMUM)`.
  **L677 CN**: 继续构造周围的表达式或声明：`vector_reduce_fmaximum, VECREDUCE_FMAXIMUM)`。
- **L678 EN**: Blank line separating nearby declarations or logic blocks.
  **L678 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L679 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.fminimum(start,x,mask,vlen)`.
  **L679 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.fminimum(start,x,mask,vlen)`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_VP(vp_reduce_fminimum, VP_REDUCE_FMINIMUM,`.
  **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_VP(vp_reduce_fminimum, VP_REDUCE_FMINIMUM,`。
- **L681 EN**: Continues the surrounding expression or declaration: `vector_reduce_fminimum, VECREDUCE_FMINIMUM)`.
  **L681 CN**: 继续构造周围的表达式或声明：`vector_reduce_fminimum, VECREDUCE_FMINIMUM)`。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L683 EN**: Undefines a macro to limit its scope: `#undef HELPER_REGISTER_REDUCTION_VP`.
  **L683 CN**: 取消宏定义以限制其作用域：`#undef HELPER_REGISTER_REDUCTION_VP`。
- **L684 EN**: Blank line separating nearby declarations or logic blocks.
  **L684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L685 EN**: Comment explains nearby logic, invariants, or intent: `Specialized helper macro for VP reductions as above but with two forms:`.
  **L685 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Specialized helper macro for VP reductions as above but with two forms:`。
- **L686 EN**: Comment explains nearby logic, invariants, or intent: `sequential and reassociative. These manifest as the presence of 'reassoc'`.
  **L686 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sequential and reassociative. These manifest as the presence of 'reassoc'`。
- **L687 EN**: Comment explains nearby logic, invariants, or intent: `fast-math flags in the IR and as two distinct ISD opcodes in the`.
  **L687 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`fast-math flags in the IR and as two distinct ISD opcodes in the`。
- **L688 EN**: Comment explains nearby logic, invariants, or intent: `SelectionDAG.`.
  **L688 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SelectionDAG.`。
- **L689 EN**: Comment explains nearby logic, invariants, or intent: `Note we by default map from the VP intrinsic to the SEQ ISD opcode, which`.
  **L689 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note we by default map from the VP intrinsic to the SEQ ISD opcode, which`。
- **L690 EN**: Comment explains nearby logic, invariants, or intent: `can then be relaxed to the non-SEQ ISD opcode if the 'reassoc' flag is set.`.
  **L690 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`can then be relaxed to the non-SEQ ISD opcode if the 'reassoc' flag is set.`。
- **L691 EN**: Starts a preprocessor conditional block: `#ifdef HELPER_REGISTER_REDUCTION_SEQ_VP`.
  **L691 CN**: 开始一个预处理条件块：`#ifdef HELPER_REGISTER_REDUCTION_SEQ_VP`。
- **L692 EN**: Continues the surrounding expression or declaration: `#error                                                                         \`.
  **L692 CN**: 继续构造周围的表达式或声明：`#error                                                                         \`。
- **L693 EN**: Continues the surrounding expression or declaration: `"The internal helper macro HELPER_REGISTER_REDUCTION_SEQ_VP is already defined!"`.
  **L693 CN**: 继续构造周围的表达式或声明：`"The internal helper macro HELPER_REGISTER_REDUCTION_SEQ_VP is already defined!"`。
- **L694 EN**: Closes the current preprocessor conditional block.
  **L694 CN**: 结束当前预处理条件块。
- **L695 EN**: Defines macro `HELPER_REGISTER_REDUCTION_SEQ_VP(VPID,` for conditional compilation, local shorthand, or diagnostics.
  **L695 CN**: 定义宏 `HELPER_REGISTER_REDUCTION_SEQ_VP(VPID,`，供条件编译、本地简写或诊断使用。
- **L696 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_INTRINSIC`.
  **L696 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_INTRINSIC` 相关的逻辑。

### Lines 697-720

````cpp
  BEGIN_REGISTER_VP_SDNODE(VPSD, 1, VPID, 2, 3)                                \
  VP_PROPERTY_FUNCTIONAL_SDOPC(SDOPC)                                          \
  END_REGISTER_VP_SDNODE(VPSD)                                                 \
  BEGIN_REGISTER_VP_SDNODE(SEQ_VPSD, 1, VPID, 2, 3)                            \
  HELPER_MAP_VPID_TO_VPSD(VPID, SEQ_VPSD)                                      \
  VP_PROPERTY_FUNCTIONAL_SDOPC(SEQ_SDOPC)                                      \
  END_REGISTER_VP_SDNODE(SEQ_VPSD)                                             \
  VP_PROPERTY_FUNCTIONAL_INTRINSIC(INTRIN)                                     \
  END_REGISTER_VP_INTRINSIC(VPID)

// llvm.vp.reduce.fadd(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_SEQ_VP(vp_reduce_fadd, VP_REDUCE_FADD,
                                 VP_REDUCE_SEQ_FADD, VECREDUCE_FADD,
                                 VECREDUCE_SEQ_FADD, vector_reduce_fadd)

// llvm.vp.reduce.fmul(start,x,mask,vlen)
HELPER_REGISTER_REDUCTION_SEQ_VP(vp_reduce_fmul, VP_REDUCE_FMUL,
                                 VP_REDUCE_SEQ_FMUL, VECREDUCE_FMUL,
                                 VECREDUCE_SEQ_FMUL, vector_reduce_fmul)

#undef HELPER_REGISTER_REDUCTION_SEQ_VP

///// } Reduction

````
- **L697 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L697 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L698 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L698 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L699 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L699 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L700 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP_SDNODE`.
  **L700 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP_SDNODE` 相关的逻辑。
- **L701 EN**: Continues logic associated with callable symbol `HELPER_MAP_VPID_TO_VPSD`.
  **L701 CN**: 继续与可调用符号 `HELPER_MAP_VPID_TO_VPSD` 相关的逻辑。
- **L702 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L702 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L703 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_SDNODE`.
  **L703 CN**: 继续与可调用符号 `END_REGISTER_VP_SDNODE` 相关的逻辑。
- **L704 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L704 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_INTRINSIC` 相关的逻辑。
- **L705 EN**: Continues logic associated with callable symbol `END_REGISTER_VP_INTRINSIC`.
  **L705 CN**: 继续与可调用符号 `END_REGISTER_VP_INTRINSIC` 相关的逻辑。
- **L706 EN**: Blank line separating nearby declarations or logic blocks.
  **L706 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L707 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.fadd(start,x,mask,vlen)`.
  **L707 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.fadd(start,x,mask,vlen)`。
- **L708 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_SEQ_VP(vp_reduce_fadd, VP_REDUCE_FADD,`.
  **L708 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_SEQ_VP(vp_reduce_fadd, VP_REDUCE_FADD,`。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VP_REDUCE_SEQ_FADD, VECREDUCE_FADD,`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`VP_REDUCE_SEQ_FADD, VECREDUCE_FADD,`。
- **L710 EN**: Continues the surrounding expression or declaration: `VECREDUCE_SEQ_FADD, vector_reduce_fadd)`.
  **L710 CN**: 继续构造周围的表达式或声明：`VECREDUCE_SEQ_FADD, vector_reduce_fadd)`。
- **L711 EN**: Blank line separating nearby declarations or logic blocks.
  **L711 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L712 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.reduce.fmul(start,x,mask,vlen)`.
  **L712 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.reduce.fmul(start,x,mask,vlen)`。
- **L713 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `HELPER_REGISTER_REDUCTION_SEQ_VP(vp_reduce_fmul, VP_REDUCE_FMUL,`.
  **L713 CN**: 继续一个多行参数列表、初始化器或聚合项：`HELPER_REGISTER_REDUCTION_SEQ_VP(vp_reduce_fmul, VP_REDUCE_FMUL,`。
- **L714 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `VP_REDUCE_SEQ_FMUL, VECREDUCE_FMUL,`.
  **L714 CN**: 继续一个多行参数列表、初始化器或聚合项：`VP_REDUCE_SEQ_FMUL, VECREDUCE_FMUL,`。
- **L715 EN**: Continues the surrounding expression or declaration: `VECREDUCE_SEQ_FMUL, vector_reduce_fmul)`.
  **L715 CN**: 继续构造周围的表达式或声明：`VECREDUCE_SEQ_FMUL, vector_reduce_fmul)`。
- **L716 EN**: Blank line separating nearby declarations or logic blocks.
  **L716 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L717 EN**: Undefines a macro to limit its scope: `#undef HELPER_REGISTER_REDUCTION_SEQ_VP`.
  **L717 CN**: 取消宏定义以限制其作用域：`#undef HELPER_REGISTER_REDUCTION_SEQ_VP`。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L719 EN**: Comment explains nearby logic, invariants, or intent: `} Reduction`.
  **L719 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Reduction`。
- **L720 EN**: Blank line separating nearby declarations or logic blocks.
  **L720 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 721-744

````cpp
///// Shuffles {

// The mask 'cond' operand of llvm.vp.select and llvm.vp.merge are not reported
// as masks with the BEGIN_REGISTER_VP_* macros.  This is because, unlike other
// VP intrinsics, these two have a defined result on lanes where the mask is
// false.
//
// llvm.vp.select(cond,on_true,on_false,vlen)
BEGIN_REGISTER_VP(vp_select, std::nullopt, 3, VP_SELECT, -1)
VP_PROPERTY_FUNCTIONAL_OPC(Select)
VP_PROPERTY_FUNCTIONAL_SDOPC(VSELECT)
END_REGISTER_VP(vp_select, VP_SELECT)

// llvm.vp.merge(cond,on_true,on_false,pivot)
BEGIN_REGISTER_VP(vp_merge, std::nullopt, 3, VP_MERGE, -1)
VP_PROPERTY_NO_FUNCTIONAL
END_REGISTER_VP(vp_merge, VP_MERGE)

BEGIN_REGISTER_VP(experimental_vp_splice, 3, 5, EXPERIMENTAL_VP_SPLICE, -1)
VP_PROPERTY_NO_FUNCTIONAL
END_REGISTER_VP(experimental_vp_splice, EXPERIMENTAL_VP_SPLICE)

// llvm.experimental.vp.reverse(x,mask,vlen)
BEGIN_REGISTER_VP(experimental_vp_reverse, 1, 2,
````
- **L721 EN**: Comment explains nearby logic, invariants, or intent: `Shuffles {`.
  **L721 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Shuffles {`。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L723 EN**: Comment explains nearby logic, invariants, or intent: `The mask 'cond' operand of llvm.vp.select and llvm.vp.merge are not reported`.
  **L723 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The mask 'cond' operand of llvm.vp.select and llvm.vp.merge are not reported`。
- **L724 EN**: Comment explains nearby logic, invariants, or intent: `as masks with the BEGIN_REGISTER_VP_* macros.  This is because, unlike other`.
  **L724 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`as masks with the BEGIN_REGISTER_VP_* macros.  This is because, unlike other`。
- **L725 EN**: Comment explains nearby logic, invariants, or intent: `VP intrinsics, these two have a defined result on lanes where the mask is`.
  **L725 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VP intrinsics, these two have a defined result on lanes where the mask is`。
- **L726 EN**: Comment explains nearby logic, invariants, or intent: `false.`.
  **L726 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`false.`。
- **L727 EN**: Separator comment used for visual grouping.
  **L727 CN**: 用于视觉分组的分隔注释。
- **L728 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.select(cond,on_true,on_false,vlen)`.
  **L728 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.select(cond,on_true,on_false,vlen)`。
- **L729 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L729 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L730 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_OPC`.
  **L730 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_OPC` 相关的逻辑。
- **L731 EN**: Continues logic associated with callable symbol `VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L731 CN**: 继续与可调用符号 `VP_PROPERTY_FUNCTIONAL_SDOPC` 相关的逻辑。
- **L732 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L732 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L734 EN**: Comment explains nearby logic, invariants, or intent: `llvm.vp.merge(cond,on_true,on_false,pivot)`.
  **L734 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.vp.merge(cond,on_true,on_false,pivot)`。
- **L735 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L735 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L736 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_NO_FUNCTIONAL`.
  **L736 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_NO_FUNCTIONAL`。
- **L737 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L737 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L738 EN**: Blank line separating nearby declarations or logic blocks.
  **L738 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L739 EN**: Continues logic associated with callable symbol `BEGIN_REGISTER_VP`.
  **L739 CN**: 继续与可调用符号 `BEGIN_REGISTER_VP` 相关的逻辑。
- **L740 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_NO_FUNCTIONAL`.
  **L740 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_NO_FUNCTIONAL`。
- **L741 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L741 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L742 EN**: Blank line separating nearby declarations or logic blocks.
  **L742 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L743 EN**: Comment explains nearby logic, invariants, or intent: `llvm.experimental.vp.reverse(x,mask,vlen)`.
  **L743 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`llvm.experimental.vp.reverse(x,mask,vlen)`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `BEGIN_REGISTER_VP(experimental_vp_reverse, 1, 2,`.
  **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`BEGIN_REGISTER_VP(experimental_vp_reverse, 1, 2,`。

### Lines 745-763

````cpp
                  EXPERIMENTAL_VP_REVERSE, -1)
VP_PROPERTY_NO_FUNCTIONAL
END_REGISTER_VP(experimental_vp_reverse, EXPERIMENTAL_VP_REVERSE)

///// } Shuffles

#undef BEGIN_REGISTER_VP
#undef BEGIN_REGISTER_VP_INTRINSIC
#undef BEGIN_REGISTER_VP_SDNODE
#undef END_REGISTER_VP
#undef END_REGISTER_VP_INTRINSIC
#undef END_REGISTER_VP_SDNODE
#undef HELPER_MAP_VPID_TO_VPSD
#undef VP_PROPERTY_BINARYOP
#undef VP_PROPERTY_CONSTRAINEDFP
#undef VP_PROPERTY_FUNCTIONAL_INTRINSIC
#undef VP_PROPERTY_FUNCTIONAL_OPC
#undef VP_PROPERTY_FUNCTIONAL_SDOPC
#undef VP_PROPERTY_NO_FUNCTIONAL
````
- **L745 EN**: Continues the surrounding expression or declaration: `EXPERIMENTAL_VP_REVERSE, -1)`.
  **L745 CN**: 继续构造周围的表达式或声明：`EXPERIMENTAL_VP_REVERSE, -1)`。
- **L746 EN**: Continues the surrounding expression or declaration: `VP_PROPERTY_NO_FUNCTIONAL`.
  **L746 CN**: 继续构造周围的表达式或声明：`VP_PROPERTY_NO_FUNCTIONAL`。
- **L747 EN**: Continues logic associated with callable symbol `END_REGISTER_VP`.
  **L747 CN**: 继续与可调用符号 `END_REGISTER_VP` 相关的逻辑。
- **L748 EN**: Blank line separating nearby declarations or logic blocks.
  **L748 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L749 EN**: Comment explains nearby logic, invariants, or intent: `} Shuffles`.
  **L749 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`} Shuffles`。
- **L750 EN**: Blank line separating nearby declarations or logic blocks.
  **L750 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L751 EN**: Undefines a macro to limit its scope: `#undef BEGIN_REGISTER_VP`.
  **L751 CN**: 取消宏定义以限制其作用域：`#undef BEGIN_REGISTER_VP`。
- **L752 EN**: Undefines a macro to limit its scope: `#undef BEGIN_REGISTER_VP_INTRINSIC`.
  **L752 CN**: 取消宏定义以限制其作用域：`#undef BEGIN_REGISTER_VP_INTRINSIC`。
- **L753 EN**: Undefines a macro to limit its scope: `#undef BEGIN_REGISTER_VP_SDNODE`.
  **L753 CN**: 取消宏定义以限制其作用域：`#undef BEGIN_REGISTER_VP_SDNODE`。
- **L754 EN**: Undefines a macro to limit its scope: `#undef END_REGISTER_VP`.
  **L754 CN**: 取消宏定义以限制其作用域：`#undef END_REGISTER_VP`。
- **L755 EN**: Undefines a macro to limit its scope: `#undef END_REGISTER_VP_INTRINSIC`.
  **L755 CN**: 取消宏定义以限制其作用域：`#undef END_REGISTER_VP_INTRINSIC`。
- **L756 EN**: Undefines a macro to limit its scope: `#undef END_REGISTER_VP_SDNODE`.
  **L756 CN**: 取消宏定义以限制其作用域：`#undef END_REGISTER_VP_SDNODE`。
- **L757 EN**: Undefines a macro to limit its scope: `#undef HELPER_MAP_VPID_TO_VPSD`.
  **L757 CN**: 取消宏定义以限制其作用域：`#undef HELPER_MAP_VPID_TO_VPSD`。
- **L758 EN**: Undefines a macro to limit its scope: `#undef VP_PROPERTY_BINARYOP`.
  **L758 CN**: 取消宏定义以限制其作用域：`#undef VP_PROPERTY_BINARYOP`。
- **L759 EN**: Undefines a macro to limit its scope: `#undef VP_PROPERTY_CONSTRAINEDFP`.
  **L759 CN**: 取消宏定义以限制其作用域：`#undef VP_PROPERTY_CONSTRAINEDFP`。
- **L760 EN**: Undefines a macro to limit its scope: `#undef VP_PROPERTY_FUNCTIONAL_INTRINSIC`.
  **L760 CN**: 取消宏定义以限制其作用域：`#undef VP_PROPERTY_FUNCTIONAL_INTRINSIC`。
- **L761 EN**: Undefines a macro to limit its scope: `#undef VP_PROPERTY_FUNCTIONAL_OPC`.
  **L761 CN**: 取消宏定义以限制其作用域：`#undef VP_PROPERTY_FUNCTIONAL_OPC`。
- **L762 EN**: Undefines a macro to limit its scope: `#undef VP_PROPERTY_FUNCTIONAL_SDOPC`.
  **L762 CN**: 取消宏定义以限制其作用域：`#undef VP_PROPERTY_FUNCTIONAL_SDOPC`。
- **L763 EN**: Undefines a macro to limit its scope: `#undef VP_PROPERTY_NO_FUNCTIONAL`.
  **L763 CN**: 取消宏定义以限制其作用域：`#undef VP_PROPERTY_NO_FUNCTIONAL`。

## Key Concepts / 关键概念

- **LLVM public API surface / LLVM 公共 API 表面**
- **Function-level IR management / 函数级 IR 管理**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Typed error propagation / 类型化错误传播**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
