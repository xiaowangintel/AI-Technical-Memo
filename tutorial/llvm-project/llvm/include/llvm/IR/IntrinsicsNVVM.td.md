# IntrinsicsNVVM.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/IR/IntrinsicsNVVM.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This file defines all of the NVVM-specific intrinsics for use with NVPTX.
- **Purpose (CN)**: 该 TableGen 文件位于 `llvm/include/llvm/IR`，主要定义与 `IntrinsicsNVVM` 相关的记录、生成元数据和声明式规则。

## Line-by-Line Analysis / 逐行分析

### Lines 1-32

````tablegen
//===- IntrinsicsNVVM.td - Defines NVVM intrinsics ---------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines all of the NVVM-specific intrinsics for use with NVPTX.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Guidelines on NVPTX Intrinsic design
//===----------------------------------------------------------------------===//
//
// The NVPTX intrinsics are used to model instructions in the PTX ISA.
// While simpler intrinsics can represent certain features effectively,
// more complex instructions like TMA and MMA are not as straightforward
// to model. A single variant of these complex instructions can expand
// into hundreds of intrinsics. Additionally, any expansion in the
// corresponding ISA can exponentially increase these numbers, making it
// difficult to manage them in the IR and backend passes. Therefore,
// a careful design of intrinsic interfaces can ease maintenance and
// contribute to a sustainable, long-term solution.
//
// The default approach is to have a 1:1 match between the intrinsic and
// the instruction where the instruction suffixes map to the intrinsic name
// and the instruction arguments map to the intrinsic arguments or return
// value.
//
// However, when there are too many instruction/intrinsic variants like
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
- **L9 EN**: Comment explains nearby logic, invariants, or intent: `This file defines all of the NVVM-specific intrinsics for use with NVPTX.`.
  **L9 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This file defines all of the NVVM-specific intrinsics for use with NVPTX.`。
- **L10 EN**: Separator comment used for visual grouping.
  **L10 CN**: 用于视觉分组的分隔注释。
- **L11 EN**: Banner comment marking a file or section boundary.
  **L11 CN**: 横幅注释，用于标记文件或章节边界。
- **L12 EN**: Blank line separating nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Comment explains nearby logic, invariants, or intent: `Guidelines on NVPTX Intrinsic design`.
  **L14 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Guidelines on NVPTX Intrinsic design`。
- **L15 EN**: Banner comment marking a file or section boundary.
  **L15 CN**: 横幅注释，用于标记文件或章节边界。
- **L16 EN**: Separator comment used for visual grouping.
  **L16 CN**: 用于视觉分组的分隔注释。
- **L17 EN**: Comment explains nearby logic, invariants, or intent: `The NVPTX intrinsics are used to model instructions in the PTX ISA.`.
  **L17 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The NVPTX intrinsics are used to model instructions in the PTX ISA.`。
- **L18 EN**: Comment explains nearby logic, invariants, or intent: `While simpler intrinsics can represent certain features effectively,`.
  **L18 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`While simpler intrinsics can represent certain features effectively,`。
- **L19 EN**: Comment explains nearby logic, invariants, or intent: `more complex instructions like TMA and MMA are not as straightforward`.
  **L19 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`more complex instructions like TMA and MMA are not as straightforward`。
- **L20 EN**: Comment explains nearby logic, invariants, or intent: `to model. A single variant of these complex instructions can expand`.
  **L20 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to model. A single variant of these complex instructions can expand`。
- **L21 EN**: Comment explains nearby logic, invariants, or intent: `into hundreds of intrinsics. Additionally, any expansion in the`.
  **L21 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`into hundreds of intrinsics. Additionally, any expansion in the`。
- **L22 EN**: Comment explains nearby logic, invariants, or intent: `corresponding ISA can exponentially increase these numbers, making it`.
  **L22 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`corresponding ISA can exponentially increase these numbers, making it`。
- **L23 EN**: Comment explains nearby logic, invariants, or intent: `difficult to manage them in the IR and backend passes. Therefore,`.
  **L23 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`difficult to manage them in the IR and backend passes. Therefore,`。
- **L24 EN**: Comment explains nearby logic, invariants, or intent: `a careful design of intrinsic interfaces can ease maintenance and`.
  **L24 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a careful design of intrinsic interfaces can ease maintenance and`。
- **L25 EN**: Comment explains nearby logic, invariants, or intent: `contribute to a sustainable, long-term solution.`.
  **L25 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`contribute to a sustainable, long-term solution.`。
- **L26 EN**: Separator comment used for visual grouping.
  **L26 CN**: 用于视觉分组的分隔注释。
- **L27 EN**: Comment explains nearby logic, invariants, or intent: `The default approach is to have a 1:1 match between the intrinsic and`.
  **L27 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The default approach is to have a 1:1 match between the intrinsic and`。
- **L28 EN**: Comment explains nearby logic, invariants, or intent: `the instruction where the instruction suffixes map to the intrinsic name`.
  **L28 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the instruction where the instruction suffixes map to the intrinsic name`。
- **L29 EN**: Comment explains nearby logic, invariants, or intent: `and the instruction arguments map to the intrinsic arguments or return`.
  **L29 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and the instruction arguments map to the intrinsic arguments or return`。
- **L30 EN**: Comment explains nearby logic, invariants, or intent: `value.`.
  **L30 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`value.`。
- **L31 EN**: Separator comment used for visual grouping.
  **L31 CN**: 用于视觉分组的分隔注释。
- **L32 EN**: Comment explains nearby logic, invariants, or intent: `However, when there are too many instruction/intrinsic variants like`.
  **L32 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`However, when there are too many instruction/intrinsic variants like`。

### Lines 33-64

````tablegen
// the TMA/MMA family, it is desirable to encode some variants as a
// constant argument, referred to as 'flags'.
// TODO: Add a guideline to quantify the metric on 'how many intrinsics' here.
//
// Below are a set of guidelines that may help in choosing
// an appropriate design for the complex intrinsics:
//
// 1. Each flag argument represents one set of instruction modifiers.
//    These flags are compile-time integer constants.
//
// 2. When an intrinsic uses flags, document it with details of the
//    flag usage in the ``NVPTXUsage.rst`` file.
// 3. Annotate all flag arguments with ImmArg<ArgIdx<>>.
// 4. Place the flag arguments at the end of the (actual)argument list.
//
// 5. Use `i1` for boolean flags and `i8` for others. Usually,
//    the `i8` types represent an `enum` encoding the family of
//    modifiers.
// 6. Note that, the specific variant for non-boolean flags may not be
//    obvious in the IR. So, maintain consistency between the enum value
//    definitions and their usage in the backend.
//    * Provide a meaningful default value in the enums wherever applicable.
//    * TODO: Investigate auto-upgrade capability for intrinsics
//      when only flag value mappings change.
//
// 7. Identify the key features of an intrinsic and distinguish between
//    first-order and supplementary information. Typically, encoding the
//    first-order information in the intrinsic name while using flags
//    for supplementary details improves readability.
//    For example:
//
//    i. For MMA intrinsics, 'dense' vs. 'sparse' is a fundamental feature,
````
- **L33 EN**: Comment explains nearby logic, invariants, or intent: `the TMA/MMA family, it is desirable to encode some variants as a`.
  **L33 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the TMA/MMA family, it is desirable to encode some variants as a`。
- **L34 EN**: Comment explains nearby logic, invariants, or intent: `constant argument, referred to as 'flags'.`.
  **L34 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`constant argument, referred to as 'flags'.`。
- **L35 EN**: Comment records a pending task or caution: `TODO: Add a guideline to quantify the metric on 'how many intrinsics' here.`.
  **L35 CN**: 注释记录了待办事项或注意点：`TODO: Add a guideline to quantify the metric on 'how many intrinsics' here.`。
- **L36 EN**: Separator comment used for visual grouping.
  **L36 CN**: 用于视觉分组的分隔注释。
- **L37 EN**: Comment explains nearby logic, invariants, or intent: `Below are a set of guidelines that may help in choosing`.
  **L37 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Below are a set of guidelines that may help in choosing`。
- **L38 EN**: Comment explains nearby logic, invariants, or intent: `an appropriate design for the complex intrinsics:`.
  **L38 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`an appropriate design for the complex intrinsics:`。
- **L39 EN**: Separator comment used for visual grouping.
  **L39 CN**: 用于视觉分组的分隔注释。
- **L40 EN**: Comment explains nearby logic, invariants, or intent: `1. Each flag argument represents one set of instruction modifiers.`.
  **L40 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`1. Each flag argument represents one set of instruction modifiers.`。
- **L41 EN**: Comment explains nearby logic, invariants, or intent: `These flags are compile-time integer constants.`.
  **L41 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`These flags are compile-time integer constants.`。
- **L42 EN**: Separator comment used for visual grouping.
  **L42 CN**: 用于视觉分组的分隔注释。
- **L43 EN**: Comment explains nearby logic, invariants, or intent: `2. When an intrinsic uses flags, document it with details of the`.
  **L43 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`2. When an intrinsic uses flags, document it with details of the`。
- **L44 EN**: Comment explains nearby logic, invariants, or intent: `flag usage in the ``NVPTXUsage.rst`` file.`.
  **L44 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`flag usage in the ``NVPTXUsage.rst`` file.`。
- **L45 EN**: Comment explains nearby logic, invariants, or intent: `3. Annotate all flag arguments with ImmArg<ArgIdx<>>.`.
  **L45 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`3. Annotate all flag arguments with ImmArg<ArgIdx<>>.`。
- **L46 EN**: Comment explains nearby logic, invariants, or intent: `4. Place the flag arguments at the end of the (actual)argument list.`.
  **L46 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`4. Place the flag arguments at the end of the (actual)argument list.`。
- **L47 EN**: Separator comment used for visual grouping.
  **L47 CN**: 用于视觉分组的分隔注释。
- **L48 EN**: Comment explains nearby logic, invariants, or intent: `5. Use `i1` for boolean flags and `i8` for others. Usually,`.
  **L48 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`5. Use `i1` for boolean flags and `i8` for others. Usually,`。
- **L49 EN**: Comment explains nearby logic, invariants, or intent: `the `i8` types represent an `enum` encoding the family of`.
  **L49 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the `i8` types represent an `enum` encoding the family of`。
- **L50 EN**: Comment explains nearby logic, invariants, or intent: `modifiers.`.
  **L50 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`modifiers.`。
- **L51 EN**: Comment explains nearby logic, invariants, or intent: `6. Note that, the specific variant for non-boolean flags may not be`.
  **L51 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`6. Note that, the specific variant for non-boolean flags may not be`。
- **L52 EN**: Comment explains nearby logic, invariants, or intent: `obvious in the IR. So, maintain consistency between the enum value`.
  **L52 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`obvious in the IR. So, maintain consistency between the enum value`。
- **L53 EN**: Comment explains nearby logic, invariants, or intent: `definitions and their usage in the backend.`.
  **L53 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`definitions and their usage in the backend.`。
- **L54 EN**: Comment explains nearby logic, invariants, or intent: `* Provide a meaningful default value in the enums wherever applicable.`.
  **L54 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* Provide a meaningful default value in the enums wherever applicable.`。
- **L55 EN**: Comment records a pending task or caution: `* TODO: Investigate auto-upgrade capability for intrinsics`.
  **L55 CN**: 注释记录了待办事项或注意点：`* TODO: Investigate auto-upgrade capability for intrinsics`。
- **L56 EN**: Comment explains nearby logic, invariants, or intent: `when only flag value mappings change.`.
  **L56 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`when only flag value mappings change.`。
- **L57 EN**: Separator comment used for visual grouping.
  **L57 CN**: 用于视觉分组的分隔注释。
- **L58 EN**: Comment explains nearby logic, invariants, or intent: `7. Identify the key features of an intrinsic and distinguish between`.
  **L58 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`7. Identify the key features of an intrinsic and distinguish between`。
- **L59 EN**: Comment explains nearby logic, invariants, or intent: `first-order and supplementary information. Typically, encoding the`.
  **L59 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`first-order and supplementary information. Typically, encoding the`。
- **L60 EN**: Comment explains nearby logic, invariants, or intent: `first-order information in the intrinsic name while using flags`.
  **L60 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`first-order information in the intrinsic name while using flags`。
- **L61 EN**: Comment explains nearby logic, invariants, or intent: `for supplementary details improves readability.`.
  **L61 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for supplementary details improves readability.`。
- **L62 EN**: Comment explains nearby logic, invariants, or intent: `For example:`.
  **L62 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For example:`。
- **L63 EN**: Separator comment used for visual grouping.
  **L63 CN**: 用于视觉分组的分隔注释。
- **L64 EN**: Comment explains nearby logic, invariants, or intent: `i. For MMA intrinsics, 'dense' vs. 'sparse' is a fundamental feature,`.
  **L64 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`i. For MMA intrinsics, 'dense' vs. 'sparse' is a fundamental feature,`。

### Lines 65-96

````tablegen
//    whereas an optional scaling applied to matrices is relatively secondary.
//
//    ii. For TMAs, the mode of copy (e.g., 'Tile' or 'Im2col') is a first-order
//    information, while features like an optional cache hint tend to be
//    secondary.
//
// 8. If there are invalid combinations within a set of modifiers, avoid
//    encoding them as flags, as much as possible. This helps reduce the
//    need for error handling of unsupported cases in the backend.
//    For example, some 'cvt' intrinsics support only a subset of the
//    possible rounding modes; so it is preferable not to encode the
//    rounding modes as flags.
// 9. Similarly, when there are invalid combinations across a set of
//    modifiers, avoid encoding them as flags to prevent additional
//    complexity in error handling.
//
// 10. Maintain a consistent design within an intrinsic family, including
//     argument ordering as well as the usage and ordering of flags.
// 11. When designing an intrinsic corresponding to an instruction or its variant,
//     consider the entire instruction family. This may reveal common features
//     that can be modelled consistently across the family.
//
// In summary, strive to balance the aspects mentioned above, to achieve
// a scalable design with maximum readability.
//===----------------------------------------------------------------------===//

// The following intrinsics were once defined here, but are now auto-upgraded
// to target-generic LLVM intrinsics.
//
//   * llvm.nvvm.brev32              --> llvm.bitreverse.i32
//   * llvm.nvvm.brev64              --> llvm.bitreverse.i64
//   * llvm.nvvm.clz.i               --> llvm.ctlz.i32
````
- **L65 EN**: Comment explains nearby logic, invariants, or intent: `whereas an optional scaling applied to matrices is relatively secondary.`.
  **L65 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`whereas an optional scaling applied to matrices is relatively secondary.`。
- **L66 EN**: Separator comment used for visual grouping.
  **L66 CN**: 用于视觉分组的分隔注释。
- **L67 EN**: Comment explains nearby logic, invariants, or intent: `ii. For TMAs, the mode of copy (e.g., 'Tile' or 'Im2col') is a first-order`.
  **L67 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ii. For TMAs, the mode of copy (e.g., 'Tile' or 'Im2col') is a first-order`。
- **L68 EN**: Comment explains nearby logic, invariants, or intent: `information, while features like an optional cache hint tend to be`.
  **L68 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`information, while features like an optional cache hint tend to be`。
- **L69 EN**: Comment explains nearby logic, invariants, or intent: `secondary.`.
  **L69 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`secondary.`。
- **L70 EN**: Separator comment used for visual grouping.
  **L70 CN**: 用于视觉分组的分隔注释。
- **L71 EN**: Comment explains nearby logic, invariants, or intent: `8. If there are invalid combinations within a set of modifiers, avoid`.
  **L71 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`8. If there are invalid combinations within a set of modifiers, avoid`。
- **L72 EN**: Comment explains nearby logic, invariants, or intent: `encoding them as flags, as much as possible. This helps reduce the`.
  **L72 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`encoding them as flags, as much as possible. This helps reduce the`。
- **L73 EN**: Comment explains nearby logic, invariants, or intent: `need for error handling of unsupported cases in the backend.`.
  **L73 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`need for error handling of unsupported cases in the backend.`。
- **L74 EN**: Comment explains nearby logic, invariants, or intent: `For example, some 'cvt' intrinsics support only a subset of the`.
  **L74 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For example, some 'cvt' intrinsics support only a subset of the`。
- **L75 EN**: Comment explains nearby logic, invariants, or intent: `possible rounding modes; so it is preferable not to encode the`.
  **L75 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`possible rounding modes; so it is preferable not to encode the`。
- **L76 EN**: Comment explains nearby logic, invariants, or intent: `rounding modes as flags.`.
  **L76 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`rounding modes as flags.`。
- **L77 EN**: Comment explains nearby logic, invariants, or intent: `9. Similarly, when there are invalid combinations across a set of`.
  **L77 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`9. Similarly, when there are invalid combinations across a set of`。
- **L78 EN**: Comment explains nearby logic, invariants, or intent: `modifiers, avoid encoding them as flags to prevent additional`.
  **L78 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`modifiers, avoid encoding them as flags to prevent additional`。
- **L79 EN**: Comment explains nearby logic, invariants, or intent: `complexity in error handling.`.
  **L79 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`complexity in error handling.`。
- **L80 EN**: Separator comment used for visual grouping.
  **L80 CN**: 用于视觉分组的分隔注释。
- **L81 EN**: Comment explains nearby logic, invariants, or intent: `10. Maintain a consistent design within an intrinsic family, including`.
  **L81 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`10. Maintain a consistent design within an intrinsic family, including`。
- **L82 EN**: Comment explains nearby logic, invariants, or intent: `argument ordering as well as the usage and ordering of flags.`.
  **L82 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`argument ordering as well as the usage and ordering of flags.`。
- **L83 EN**: Comment explains nearby logic, invariants, or intent: `11. When designing an intrinsic corresponding to an instruction or its variant,`.
  **L83 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`11. When designing an intrinsic corresponding to an instruction or its variant,`。
- **L84 EN**: Comment explains nearby logic, invariants, or intent: `consider the entire instruction family. This may reveal common features`.
  **L84 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`consider the entire instruction family. This may reveal common features`。
- **L85 EN**: Comment explains nearby logic, invariants, or intent: `that can be modelled consistently across the family.`.
  **L85 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`that can be modelled consistently across the family.`。
- **L86 EN**: Separator comment used for visual grouping.
  **L86 CN**: 用于视觉分组的分隔注释。
- **L87 EN**: Comment explains nearby logic, invariants, or intent: `In summary, strive to balance the aspects mentioned above, to achieve`.
  **L87 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`In summary, strive to balance the aspects mentioned above, to achieve`。
- **L88 EN**: Comment explains nearby logic, invariants, or intent: `a scalable design with maximum readability.`.
  **L88 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a scalable design with maximum readability.`。
- **L89 EN**: Banner comment marking a file or section boundary.
  **L89 CN**: 横幅注释，用于标记文件或章节边界。
- **L90 EN**: Blank line separating nearby declarations or logic blocks.
  **L90 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L91 EN**: Comment explains nearby logic, invariants, or intent: `The following intrinsics were once defined here, but are now auto-upgraded`.
  **L91 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The following intrinsics were once defined here, but are now auto-upgraded`。
- **L92 EN**: Comment explains nearby logic, invariants, or intent: `to target-generic LLVM intrinsics.`.
  **L92 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to target-generic LLVM intrinsics.`。
- **L93 EN**: Separator comment used for visual grouping.
  **L93 CN**: 用于视觉分组的分隔注释。
- **L94 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.brev32              --> llvm.bitreverse.i32`.
  **L94 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.brev32              --> llvm.bitreverse.i32`。
- **L95 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.brev64              --> llvm.bitreverse.i64`.
  **L95 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.brev64              --> llvm.bitreverse.i64`。
- **L96 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.clz.i               --> llvm.ctlz.i32`.
  **L96 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.clz.i               --> llvm.ctlz.i32`。

### Lines 97-128

````tablegen
//   * llvm.nvvm.clz.ll              --> trunc i64 llvm.ctlz.i64(x) to i32
//   * llvm.nvvm.popc.i              --> llvm.ctpop.i32
//   * llvm.nvvm.popc.ll             --> trunc i64 llvm.ctpop.i64 to i32
//   * llvm.nvvm.abs.i               --> select(x >= -x, x, -x)
//   * llvm.nvvm.abs.ll              --> ibid.
//   * llvm.nvvm.max.i               --> select(x sge y, x, y)
//   * llvm.nvvm.max.ll              --> ibid.
//   * llvm.nvvm.max.ui              --> select(x uge y, x, y)
//   * llvm.nvvm.max.ull             --> ibid.
//   * llvm.nvvm.max.i               --> select(x sle y, x, y)
//   * llvm.nvvm.max.ll              --> ibid.
//   * llvm.nvvm.max.ui              --> select(x ule y, x, y)
//   * llvm.nvvm.max.ull             --> ibid.
//   * llvm.nvvm.h2f                 --> llvm.convert.to.fp16.f32
//   * llvm.nvvm.bitcast.f2i         --> bitcast
//   * llvm.nvvm.bitcast.i2f         --> ibid.
//   * llvm.nvvm.bitcast.d2ll        --> ibid.
//   * llvm.nvvm.bitcast.ll2d        --> ibid.
//   * llvm.nvvm.ptr.gen.to.global   --> addrspacecast
//   * llvm.nvvm.ptr.gen.to.shared   --> ibid.
//   * llvm.nvvm.ptr.gen.to.constant --> ibid.
//   * llvm.nvvm.ptr.gen.to.local    --> ibid.
//   * llvm.nvvm.ptr.gen.to.param    --> ibid.
//   * llvm.nvvm.ptr.global.to.gen   --> ibid.
//   * llvm.nvvm.ptr.shared.to.gen   --> ibid.
//   * llvm.nvvm.ptr.constant.to.gen --> ibid.
//   * llvm.nvvm.ptr.local.to.gen    --> ibid.
//   * llvm.nvvm.ptr.param.to.gen    --> ibid.
//   * llvm.nvvm.ldg.global.i        --> load addrspace(1) !load.invariant
//   * llvm.nvvm.ldg.global.f        --> ibid.
//   * llvm.nvvm.ldg.global.p        --> ibid.
//   * llvm.nvvm.swap.lo.hi.b64      --> llvm.fshl(x, x, 32)
````
- **L97 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.clz.ll              --> trunc i64 llvm.ctlz.i64(x) to i32`.
  **L97 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.clz.ll              --> trunc i64 llvm.ctlz.i64(x) to i32`。
- **L98 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.popc.i              --> llvm.ctpop.i32`.
  **L98 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.popc.i              --> llvm.ctpop.i32`。
- **L99 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.popc.ll             --> trunc i64 llvm.ctpop.i64 to i32`.
  **L99 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.popc.ll             --> trunc i64 llvm.ctpop.i64 to i32`。
- **L100 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.abs.i               --> select(x >= -x, x, -x)`.
  **L100 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.abs.i               --> select(x >= -x, x, -x)`。
- **L101 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.abs.ll              --> ibid.`.
  **L101 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.abs.ll              --> ibid.`。
- **L102 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.i               --> select(x sge y, x, y)`.
  **L102 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.i               --> select(x sge y, x, y)`。
- **L103 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.ll              --> ibid.`.
  **L103 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.ll              --> ibid.`。
- **L104 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.ui              --> select(x uge y, x, y)`.
  **L104 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.ui              --> select(x uge y, x, y)`。
- **L105 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.ull             --> ibid.`.
  **L105 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.ull             --> ibid.`。
- **L106 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.i               --> select(x sle y, x, y)`.
  **L106 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.i               --> select(x sle y, x, y)`。
- **L107 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.ll              --> ibid.`.
  **L107 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.ll              --> ibid.`。
- **L108 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.ui              --> select(x ule y, x, y)`.
  **L108 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.ui              --> select(x ule y, x, y)`。
- **L109 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.max.ull             --> ibid.`.
  **L109 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.max.ull             --> ibid.`。
- **L110 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.h2f                 --> llvm.convert.to.fp16.f32`.
  **L110 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.h2f                 --> llvm.convert.to.fp16.f32`。
- **L111 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.bitcast.f2i         --> bitcast`.
  **L111 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.bitcast.f2i         --> bitcast`。
- **L112 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.bitcast.i2f         --> ibid.`.
  **L112 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.bitcast.i2f         --> ibid.`。
- **L113 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.bitcast.d2ll        --> ibid.`.
  **L113 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.bitcast.d2ll        --> ibid.`。
- **L114 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.bitcast.ll2d        --> ibid.`.
  **L114 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.bitcast.ll2d        --> ibid.`。
- **L115 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.gen.to.global   --> addrspacecast`.
  **L115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.gen.to.global   --> addrspacecast`。
- **L116 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.gen.to.shared   --> ibid.`.
  **L116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.gen.to.shared   --> ibid.`。
- **L117 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.gen.to.constant --> ibid.`.
  **L117 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.gen.to.constant --> ibid.`。
- **L118 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.gen.to.local    --> ibid.`.
  **L118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.gen.to.local    --> ibid.`。
- **L119 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.gen.to.param    --> ibid.`.
  **L119 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.gen.to.param    --> ibid.`。
- **L120 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.global.to.gen   --> ibid.`.
  **L120 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.global.to.gen   --> ibid.`。
- **L121 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.shared.to.gen   --> ibid.`.
  **L121 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.shared.to.gen   --> ibid.`。
- **L122 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.constant.to.gen --> ibid.`.
  **L122 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.constant.to.gen --> ibid.`。
- **L123 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.local.to.gen    --> ibid.`.
  **L123 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.local.to.gen    --> ibid.`。
- **L124 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ptr.param.to.gen    --> ibid.`.
  **L124 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ptr.param.to.gen    --> ibid.`。
- **L125 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ldg.global.i        --> load addrspace(1) !load.invariant`.
  **L125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ldg.global.i        --> load addrspace(1) !load.invariant`。
- **L126 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ldg.global.f        --> ibid.`.
  **L126 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ldg.global.f        --> ibid.`。
- **L127 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.ldg.global.p        --> ibid.`.
  **L127 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.ldg.global.p        --> ibid.`。
- **L128 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.swap.lo.hi.b64      --> llvm.fshl(x, x, 32)`.
  **L128 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.swap.lo.hi.b64      --> llvm.fshl(x, x, 32)`。

### Lines 129-160

````tablegen
//   * llvm.nvvm.atomic.load.inc.32  --> atomicrmw uinc_wrap
//   * llvm.nvvm.atomic.load.dec.32  --> atomicrmw udec_wrap
//   * llvm.nvvm.barrier0            --> llvm.nvvm.barrier.cta.sync.aligned.all(0)
//   * llvm.nvvm.barrier.n           --> llvm.nvvm.barrier.cta.sync.aligned.all(x)
//   * llvm.nvvm.bar.sync            --> llvm.nvvm.barrier.cta.sync.aligned.all(x)
//   * llvm.nvvm.barrier             --> llvm.nvvm.barrier.cta.sync.aligned(x, y)
//   * llvm.nvvm.barrier.sync        --> llvm.nvvm.barrier.cta.sync.all(x)
//   * llvm.nvvm.barrier.sync.cnt    --> llvm.nvvm.barrier.cta.sync(x, y)
//   * llvm.nvvm.barrier0.popc       --> llvm.nvvm.barrier.cta.red.popc.aligned.all(0, c)
//   * llvm.nvvm.barrier0.and        --> llvm.nvvm.barrier.cta.red.and.aligned.all(0, z)
//   * llvm.nvvm.barrier0.or         --> llvm.nvvm.barrier.cta.red.or.aligned.all(0, z)

def llvm_global_ptr_ty  : LLVMQualPointerType<1>;         // (global)ptr
def llvm_shared_ptr_ty  : LLVMQualPointerType<3>;         // (shared)ptr
def llvm_constant_ptr_ty: LLVMQualPointerType<4>;         // (const)ptr
def llvm_local_ptr_ty   : LLVMQualPointerType<5>;         // (local)ptr
def llvm_tmem_ptr_ty    : LLVMQualPointerType<6>;         // (tensor memory)ptr
def llvm_shared_cluster_ptr_ty : LLVMQualPointerType<7>;  // (shared_cluster)ptr

//
// MISC
//

defvar WARP_SIZE = 32;

// Note: the maximum grid size in the x-dimension is the lower value of 65535
// on sm_20. We conservatively use the larger value here as it required for
// sm_30+ and also correct for sm_20.
defvar MAX_GRID_SIZE_X = 0x7fffffff;
defvar MAX_GRID_SIZE_Y = 0xffff;
defvar MAX_GRID_SIZE_Z = 0xffff;

````
- **L129 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.atomic.load.inc.32  --> atomicrmw uinc_wrap`.
  **L129 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.atomic.load.inc.32  --> atomicrmw uinc_wrap`。
- **L130 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.atomic.load.dec.32  --> atomicrmw udec_wrap`.
  **L130 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.atomic.load.dec.32  --> atomicrmw udec_wrap`。
- **L131 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier0            --> llvm.nvvm.barrier.cta.sync.aligned.all(0)`.
  **L131 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier0            --> llvm.nvvm.barrier.cta.sync.aligned.all(0)`。
- **L132 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier.n           --> llvm.nvvm.barrier.cta.sync.aligned.all(x)`.
  **L132 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier.n           --> llvm.nvvm.barrier.cta.sync.aligned.all(x)`。
- **L133 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.bar.sync            --> llvm.nvvm.barrier.cta.sync.aligned.all(x)`.
  **L133 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.bar.sync            --> llvm.nvvm.barrier.cta.sync.aligned.all(x)`。
- **L134 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier             --> llvm.nvvm.barrier.cta.sync.aligned(x, y)`.
  **L134 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier             --> llvm.nvvm.barrier.cta.sync.aligned(x, y)`。
- **L135 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier.sync        --> llvm.nvvm.barrier.cta.sync.all(x)`.
  **L135 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier.sync        --> llvm.nvvm.barrier.cta.sync.all(x)`。
- **L136 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier.sync.cnt    --> llvm.nvvm.barrier.cta.sync(x, y)`.
  **L136 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier.sync.cnt    --> llvm.nvvm.barrier.cta.sync(x, y)`。
- **L137 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier0.popc       --> llvm.nvvm.barrier.cta.red.popc.aligned.all(0, c)`.
  **L137 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier0.popc       --> llvm.nvvm.barrier.cta.red.popc.aligned.all(0, c)`。
- **L138 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier0.and        --> llvm.nvvm.barrier.cta.red.and.aligned.all(0, z)`.
  **L138 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier0.and        --> llvm.nvvm.barrier.cta.red.and.aligned.all(0, z)`。
- **L139 EN**: Comment explains nearby logic, invariants, or intent: `* llvm.nvvm.barrier0.or         --> llvm.nvvm.barrier.cta.red.or.aligned.all(0, z)`.
  **L139 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`* llvm.nvvm.barrier0.or         --> llvm.nvvm.barrier.cta.red.or.aligned.all(0, z)`。
- **L140 EN**: Blank line separating nearby declarations or logic blocks.
  **L140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L141 EN**: Declares TableGen def `llvm_global_ptr_ty`.
  **L141 CN**: 声明 TableGen def `llvm_global_ptr_ty`。
- **L142 EN**: Declares TableGen def `llvm_shared_ptr_ty`.
  **L142 CN**: 声明 TableGen def `llvm_shared_ptr_ty`。
- **L143 EN**: Declares TableGen def `llvm_constant_ptr_ty`.
  **L143 CN**: 声明 TableGen def `llvm_constant_ptr_ty`。
- **L144 EN**: Declares TableGen def `llvm_local_ptr_ty`.
  **L144 CN**: 声明 TableGen def `llvm_local_ptr_ty`。
- **L145 EN**: Declares TableGen def `llvm_tmem_ptr_ty`.
  **L145 CN**: 声明 TableGen def `llvm_tmem_ptr_ty`。
- **L146 EN**: Declares TableGen def `llvm_shared_cluster_ptr_ty`.
  **L146 CN**: 声明 TableGen def `llvm_shared_cluster_ptr_ty`。
- **L147 EN**: Blank line separating nearby declarations or logic blocks.
  **L147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L148 EN**: Separator comment used for visual grouping.
  **L148 CN**: 用于视觉分组的分隔注释。
- **L149 EN**: Comment explains nearby logic, invariants, or intent: `MISC`.
  **L149 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MISC`。
- **L150 EN**: Separator comment used for visual grouping.
  **L150 CN**: 用于视觉分组的分隔注释。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L152 EN**: Initializes variable `WARP_SIZE` from the right-hand expression.
  **L152 CN**: 使用右侧表达式初始化变量 `WARP_SIZE`。
- **L153 EN**: Blank line separating nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L154 EN**: Comment explains nearby logic, invariants, or intent: `Note: the maximum grid size in the x-dimension is the lower value of 65535`.
  **L154 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: the maximum grid size in the x-dimension is the lower value of 65535`。
- **L155 EN**: Comment explains nearby logic, invariants, or intent: `on sm_20. We conservatively use the larger value here as it required for`.
  **L155 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`on sm_20. We conservatively use the larger value here as it required for`。
- **L156 EN**: Comment explains nearby logic, invariants, or intent: `sm_30+ and also correct for sm_20.`.
  **L156 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sm_30+ and also correct for sm_20.`。
- **L157 EN**: Initializes variable `MAX_GRID_SIZE_X` from the right-hand expression.
  **L157 CN**: 使用右侧表达式初始化变量 `MAX_GRID_SIZE_X`。
- **L158 EN**: Initializes variable `MAX_GRID_SIZE_Y` from the right-hand expression.
  **L158 CN**: 使用右侧表达式初始化变量 `MAX_GRID_SIZE_Y`。
- **L159 EN**: Initializes variable `MAX_GRID_SIZE_Z` from the right-hand expression.
  **L159 CN**: 使用右侧表达式初始化变量 `MAX_GRID_SIZE_Z`。
- **L160 EN**: Blank line separating nearby declarations or logic blocks.
  **L160 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 161-192

````tablegen
defvar MAX_BLOCK_SIZE_X = 1024;
defvar MAX_BLOCK_SIZE_Y = 1024;
defvar MAX_BLOCK_SIZE_Z = 64;

// Helper class that concatenates list elements with
// a given separator 'sep' and returns the result.
// Handles empty strings.
class StrJoin<string sep, list<string> str_list> {
  string ret = !foldl("", str_list, a, b,
               !if(!eq(a, ""), b, !if(!eq(b, ""), a, !strconcat(a, sep, b))));
}

// Helper class that represents a 'fragment' of an NVPTX *MMA instruction.
// Geom: m<M>n<N>k<K>. E.g. m8n32k16
// Frag: [a|b|c|d] ([x1|x2|x4] for ldmatrix)
// PtxEltType: PTX type for the element.
class WMMA_REGS<string Geom, string Frag, string PtxEltType, bit IsSparse = false> {
  string geom = Geom;
  string frag = Frag;
  string ptx_elt_type = PtxEltType;
  string gft = Geom#":"#Frag#":"#ptx_elt_type;
  string gf = Geom#":"#Frag;
  string ft = frag#":"#ptx_elt_type;
  bit isSparse = IsSparse;
  list<LLVMType> regs = !if(!eq(isSparse, true),
    !cond(
      // mma sparse ops use other fragments for some arguments
      !eq(gft,"m16n8k16:a:bf16") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k16:a:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k16:b:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k16:c:f16") : !listsplat(llvm_v2f16_ty, 2),
````
- **L161 EN**: Initializes variable `MAX_BLOCK_SIZE_X` from the right-hand expression.
  **L161 CN**: 使用右侧表达式初始化变量 `MAX_BLOCK_SIZE_X`。
- **L162 EN**: Initializes variable `MAX_BLOCK_SIZE_Y` from the right-hand expression.
  **L162 CN**: 使用右侧表达式初始化变量 `MAX_BLOCK_SIZE_Y`。
- **L163 EN**: Initializes variable `MAX_BLOCK_SIZE_Z` from the right-hand expression.
  **L163 CN**: 使用右侧表达式初始化变量 `MAX_BLOCK_SIZE_Z`。
- **L164 EN**: Blank line separating nearby declarations or logic blocks.
  **L164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L165 EN**: Comment explains nearby logic, invariants, or intent: `Helper class that concatenates list elements with`.
  **L165 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper class that concatenates list elements with`。
- **L166 EN**: Comment explains nearby logic, invariants, or intent: `a given separator 'sep' and returns the result.`.
  **L166 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a given separator 'sep' and returns the result.`。
- **L167 EN**: Comment explains nearby logic, invariants, or intent: `Handles empty strings.`.
  **L167 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Handles empty strings.`。
- **L168 EN**: Declares class `StrJoin<string`.
  **L168 CN**: 声明 class `StrJoin<string`。
- **L169 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `string ret = !foldl("", str_list, a, b,`.
  **L169 CN**: 继续一个多行参数列表、初始化器或聚合项：`string ret = !foldl("", str_list, a, b,`。
- **L170 EN**: Executes a call or declaration centered on `!if`.
  **L170 CN**: 执行以 `!if` 为核心的调用或声明。
- **L171 EN**: Closes the current lexical scope or compound statement.
  **L171 CN**: 结束当前词法作用域或复合语句块。
- **L172 EN**: Blank line separating nearby declarations or logic blocks.
  **L172 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L173 EN**: Comment explains nearby logic, invariants, or intent: `Helper class that represents a 'fragment' of an NVPTX *MMA instruction.`.
  **L173 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper class that represents a 'fragment' of an NVPTX *MMA instruction.`。
- **L174 EN**: Comment explains nearby logic, invariants, or intent: `Geom: m<M>n<N>k<K>. E.g. m8n32k16`.
  **L174 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Geom: m<M>n<N>k<K>. E.g. m8n32k16`。
- **L175 EN**: Comment explains nearby logic, invariants, or intent: `Frag: [a|b|c|d] ([x1|x2|x4] for ldmatrix)`.
  **L175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Frag: [a|b|c|d] ([x1|x2|x4] for ldmatrix)`。
- **L176 EN**: Comment explains nearby logic, invariants, or intent: `PtxEltType: PTX type for the element.`.
  **L176 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PtxEltType: PTX type for the element.`。
- **L177 EN**: Declares class `WMMA_REGS<string`.
  **L177 CN**: 声明 class `WMMA_REGS<string`。
- **L178 EN**: Initializes variable `geom` from the right-hand expression.
  **L178 CN**: 使用右侧表达式初始化变量 `geom`。
- **L179 EN**: Initializes variable `frag` from the right-hand expression.
  **L179 CN**: 使用右侧表达式初始化变量 `frag`。
- **L180 EN**: Initializes variable `ptx_elt_type` from the right-hand expression.
  **L180 CN**: 使用右侧表达式初始化变量 `ptx_elt_type`。
- **L181 EN**: Initializes variable `gft` from the right-hand expression.
  **L181 CN**: 使用右侧表达式初始化变量 `gft`。
- **L182 EN**: Initializes variable `gf` from the right-hand expression.
  **L182 CN**: 使用右侧表达式初始化变量 `gf`。
- **L183 EN**: Initializes variable `ft` from the right-hand expression.
  **L183 CN**: 使用右侧表达式初始化变量 `ft`。
- **L184 EN**: Initializes variable `isSparse` from the right-hand expression.
  **L184 CN**: 使用右侧表达式初始化变量 `isSparse`。
- **L185 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> regs = !if(!eq(isSparse, true),`.
  **L185 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> regs = !if(!eq(isSparse, true),`。
- **L186 EN**: Continues logic associated with callable symbol `cond`.
  **L186 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L187 EN**: Comment explains nearby logic, invariants, or intent: `mma sparse ops use other fragments for some arguments`.
  **L187 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma sparse ops use other fragments for some arguments`。
- **L188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:bf16") : !listsplat(llvm_i32_ty, 2),`.
  **L188 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:bf16") : !listsplat(llvm_i32_ty, 2),`。
- **L189 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L189 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L190 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),`.
  **L190 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),`。
- **L191 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L191 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L192 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:c:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L192 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:c:f16") : !listsplat(llvm_v2f16_ty, 2),`。

### Lines 193-224

````tablegen
      !eq(gft,"m16n8k16:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k16:d:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k16:d:f32") : !listsplat(llvm_float_ty, 4),

      !eq(gft,"m16n8k32:a:bf16") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:a:f16") : !listsplat(llvm_v2f16_ty, 4),
      !eq(gft,"m16n8k32:b:bf16") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:b:f16") : !listsplat(llvm_v2f16_ty, 4),
      !eq(gft,"m16n8k32:c:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k32:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k32:d:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k32:d:f32") : !listsplat(llvm_float_ty, 4),

      !eq(gft,"m16n8k16:a:tf32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k16:b:tf32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k16:c:tf32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k16:d:tf32") : !listsplat(llvm_float_ty, 4),

      !eq(gft,"m16n8k8:a:tf32") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k8:b:tf32") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k8:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k8:d:f32") : !listsplat(llvm_float_ty, 4),

      !eq(gft,"m16n8k32:a:u8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:a:s8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:u8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:s8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),

      !eq(gft,"m16n8k64:a:u8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:a:s8") : !listsplat(llvm_i32_ty, 4),
````
- **L193 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L193 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L194 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:d:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L194 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:d:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L195 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L195 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:bf16") : !listsplat(llvm_i32_ty, 4),`.
  **L197 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:bf16") : !listsplat(llvm_i32_ty, 4),`。
- **L198 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:f16") : !listsplat(llvm_v2f16_ty, 4),`.
  **L198 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:f16") : !listsplat(llvm_v2f16_ty, 4),`。
- **L199 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:bf16") : !listsplat(llvm_i32_ty, 4),`.
  **L199 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:bf16") : !listsplat(llvm_i32_ty, 4),`。
- **L200 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:f16") : !listsplat(llvm_v2f16_ty, 4),`.
  **L200 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:f16") : !listsplat(llvm_v2f16_ty, 4),`。
- **L201 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:c:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L201 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:c:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L202 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L202 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L203 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:d:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L203 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:d:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L204 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L204 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L205 EN**: Blank line separating nearby declarations or logic blocks.
  **L205 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:tf32") : !listsplat(llvm_i32_ty, 4),`.
  **L206 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:tf32") : !listsplat(llvm_i32_ty, 4),`。
- **L207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:tf32") : !listsplat(llvm_i32_ty, 4),`.
  **L207 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:tf32") : !listsplat(llvm_i32_ty, 4),`。
- **L208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:c:tf32") : !listsplat(llvm_float_ty, 4),`.
  **L208 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:c:tf32") : !listsplat(llvm_float_ty, 4),`。
- **L209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:d:tf32") : !listsplat(llvm_float_ty, 4),`.
  **L209 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:d:tf32") : !listsplat(llvm_float_ty, 4),`。
- **L210 EN**: Blank line separating nearby declarations or logic blocks.
  **L210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:a:tf32") : !listsplat(llvm_i32_ty, 2),`.
  **L211 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:a:tf32") : !listsplat(llvm_i32_ty, 2),`。
- **L212 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:b:tf32") : !listsplat(llvm_i32_ty, 2),`.
  **L212 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:b:tf32") : !listsplat(llvm_i32_ty, 2),`。
- **L213 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L213 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L214 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L214 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L215 EN**: Blank line separating nearby declarations or logic blocks.
  **L215 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L216 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:u8") : !listsplat(llvm_i32_ty, 2),`.
  **L216 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:u8") : !listsplat(llvm_i32_ty, 2),`。
- **L217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:s8") : !listsplat(llvm_i32_ty, 2),`.
  **L217 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:s8") : !listsplat(llvm_i32_ty, 2),`。
- **L218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:u8") : !listsplat(llvm_i32_ty, 2),`.
  **L218 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:u8") : !listsplat(llvm_i32_ty, 2),`。
- **L219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:s8") : !listsplat(llvm_i32_ty, 2),`.
  **L219 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:s8") : !listsplat(llvm_i32_ty, 2),`。
- **L220 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L220 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L221 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L221 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L222 EN**: Blank line separating nearby declarations or logic blocks.
  **L222 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L223 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:u8") : !listsplat(llvm_i32_ty, 4),`.
  **L223 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:u8") : !listsplat(llvm_i32_ty, 4),`。
- **L224 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:s8") : !listsplat(llvm_i32_ty, 4),`.
  **L224 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:s8") : !listsplat(llvm_i32_ty, 4),`。

### Lines 225-256

````tablegen
      !eq(gft,"m16n8k64:a:e4m3") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:a:e5m2") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:a:e3m2") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:a:e2m3") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:a:e2m1") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:u8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:s8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:e4m3") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:e5m2") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:e3m2") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:e2m3") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:e2m1") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:c:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k64:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k64:d:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k64:d:f32") : !listsplat(llvm_float_ty, 4),

      !eq(gft,"m16n8k64:a:u4") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k64:a:s4") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k64:b:u4") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k64:b:s4") : !listsplat(llvm_i32_ty, 2),

      !eq(gft,"m16n8k64:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:d:s32") : !listsplat(llvm_i32_ty, 4),

      !eq(gft,"m16n8k128:a:u4") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:a:s4") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:a:e2m1") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:b:u4") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:b:s4") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:b:e2m1") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:c:s32") : !listsplat(llvm_i32_ty, 4),
````
- **L225 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:e4m3") : !listsplat(llvm_i32_ty, 4),`.
  **L225 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:e4m3") : !listsplat(llvm_i32_ty, 4),`。
- **L226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:e5m2") : !listsplat(llvm_i32_ty, 4),`.
  **L226 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:e5m2") : !listsplat(llvm_i32_ty, 4),`。
- **L227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:e3m2") : !listsplat(llvm_i32_ty, 4),`.
  **L227 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:e3m2") : !listsplat(llvm_i32_ty, 4),`。
- **L228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:e2m3") : !listsplat(llvm_i32_ty, 4),`.
  **L228 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:e2m3") : !listsplat(llvm_i32_ty, 4),`。
- **L229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:e2m1") : !listsplat(llvm_i32_ty, 4),`.
  **L229 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:e2m1") : !listsplat(llvm_i32_ty, 4),`。
- **L230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:u8") : !listsplat(llvm_i32_ty, 4),`.
  **L230 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:u8") : !listsplat(llvm_i32_ty, 4),`。
- **L231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:s8") : !listsplat(llvm_i32_ty, 4),`.
  **L231 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:s8") : !listsplat(llvm_i32_ty, 4),`。
- **L232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:e4m3") : !listsplat(llvm_i32_ty, 4),`.
  **L232 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:e4m3") : !listsplat(llvm_i32_ty, 4),`。
- **L233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:e5m2") : !listsplat(llvm_i32_ty, 4),`.
  **L233 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:e5m2") : !listsplat(llvm_i32_ty, 4),`。
- **L234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:e3m2") : !listsplat(llvm_i32_ty, 4),`.
  **L234 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:e3m2") : !listsplat(llvm_i32_ty, 4),`。
- **L235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:e2m3") : !listsplat(llvm_i32_ty, 4),`.
  **L235 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:e2m3") : !listsplat(llvm_i32_ty, 4),`。
- **L236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:e2m1") : !listsplat(llvm_i32_ty, 4),`.
  **L236 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:e2m1") : !listsplat(llvm_i32_ty, 4),`。
- **L237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:c:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L237 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:c:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L238 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L239 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:d:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L239 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:d:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L242 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:u4") : !listsplat(llvm_i32_ty, 2),`.
  **L242 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:u4") : !listsplat(llvm_i32_ty, 2),`。
- **L243 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:s4") : !listsplat(llvm_i32_ty, 2),`.
  **L243 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:s4") : !listsplat(llvm_i32_ty, 2),`。
- **L244 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:u4") : !listsplat(llvm_i32_ty, 2),`.
  **L244 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:u4") : !listsplat(llvm_i32_ty, 2),`。
- **L245 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:s4") : !listsplat(llvm_i32_ty, 2),`.
  **L245 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:s4") : !listsplat(llvm_i32_ty, 2),`。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L247 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L247 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L248 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L248 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L249 EN**: Blank line separating nearby declarations or logic blocks.
  **L249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L250 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:a:u4") : !listsplat(llvm_i32_ty, 4),`.
  **L250 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:a:u4") : !listsplat(llvm_i32_ty, 4),`。
- **L251 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:a:s4") : !listsplat(llvm_i32_ty, 4),`.
  **L251 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:a:s4") : !listsplat(llvm_i32_ty, 4),`。
- **L252 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:a:e2m1") : !listsplat(llvm_i32_ty, 4),`.
  **L252 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:a:e2m1") : !listsplat(llvm_i32_ty, 4),`。
- **L253 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:b:u4") : !listsplat(llvm_i32_ty, 4),`.
  **L253 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:b:u4") : !listsplat(llvm_i32_ty, 4),`。
- **L254 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:b:s4") : !listsplat(llvm_i32_ty, 4),`.
  **L254 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:b:s4") : !listsplat(llvm_i32_ty, 4),`。
- **L255 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:b:e2m1") : !listsplat(llvm_i32_ty, 4),`.
  **L255 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:b:e2m1") : !listsplat(llvm_i32_ty, 4),`。
- **L256 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L256 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:c:s32") : !listsplat(llvm_i32_ty, 4),`。

### Lines 257-288

````tablegen
      !eq(gft,"m16n8k128:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k128:d:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:d:f32") : !listsplat(llvm_float_ty, 4),
    ),
    !cond(
      // mma fp ops use smaller fragments than wmma fp ops
      !eq(gft,"m8n8k4:a:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m8n8k4:b:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k8:a:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k8:b:f16") : [llvm_v2f16_ty],
      !eq(gft,"m16n8k8:c:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k8:d:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k8:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k8:d:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k16:a:f16") : !listsplat(llvm_v2f16_ty, 4),
      !eq(gft,"m16n8k16:b:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k16:c:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k16:d:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k16:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k16:d:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k4:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k4:d:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k32:c:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k32:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k32:d:f16") : !listsplat(llvm_v2f16_ty, 2),
      !eq(gft,"m16n8k32:d:f32") : !listsplat(llvm_float_ty, 4),

      // mma.block_scale e2m1 (mxf4, mxf4nvf4) -> f32 @ m16n8k64
      !eq(gft,"m16n8k64:c:f32") : !listsplat(llvm_float_ty, 4),
      !eq(gft,"m16n8k64:d:f32") : !listsplat(llvm_float_ty, 4),

      // wmma fp16 -> fp16/fp32 @  m16n16k16/m8n32k16/m32n8k16
````
- **L257 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L257 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L258 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L258 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L259 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L259 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L260 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `),`.
  **L260 CN**: 继续一个多行参数列表、初始化器或聚合项：`),`。
- **L261 EN**: Continues logic associated with callable symbol `cond`.
  **L261 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L262 EN**: Comment explains nearby logic, invariants, or intent: `mma fp ops use smaller fragments than wmma fp ops`.
  **L262 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma fp ops use smaller fragments than wmma fp ops`。
- **L263 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k4:a:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L263 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k4:a:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L264 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k4:b:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L264 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k4:b:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L265 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:a:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L265 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:a:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L266 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:b:f16") : [llvm_v2f16_ty],`.
  **L266 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:b:f16") : [llvm_v2f16_ty],`。
- **L267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:c:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L267 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:c:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L268 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:d:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L268 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:d:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L269 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L269 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L270 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L270 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L271 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:f16") : !listsplat(llvm_v2f16_ty, 4),`.
  **L271 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:f16") : !listsplat(llvm_v2f16_ty, 4),`。
- **L272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L272 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:c:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L273 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:c:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L274 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:d:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L274 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:d:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L275 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L276 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L276 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L277 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L277 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L278 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L278 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L279 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:c:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L279 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:c:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L280 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L280 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L281 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:d:f16") : !listsplat(llvm_v2f16_ty, 2),`.
  **L281 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:d:f16") : !listsplat(llvm_v2f16_ty, 2),`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L283 EN**: Blank line separating nearby declarations or logic blocks.
  **L283 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L284 EN**: Comment explains nearby logic, invariants, or intent: `mma.block_scale e2m1 (mxf4, mxf4nvf4) -> f32 @ m16n8k64`.
  **L284 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma.block_scale e2m1 (mxf4, mxf4nvf4) -> f32 @ m16n8k64`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:c:f32") : !listsplat(llvm_float_ty, 4),`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:c:f32") : !listsplat(llvm_float_ty, 4),`。
- **L286 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:d:f32") : !listsplat(llvm_float_ty, 4),`.
  **L286 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:d:f32") : !listsplat(llvm_float_ty, 4),`。
- **L287 EN**: Blank line separating nearby declarations or logic blocks.
  **L287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L288 EN**: Comment explains nearby logic, invariants, or intent: `wmma fp16 -> fp16/fp32 @  m16n16k16/m8n32k16/m32n8k16`.
  **L288 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wmma fp16 -> fp16/fp32 @  m16n16k16/m8n32k16/m32n8k16`。

### Lines 289-320

````tablegen
      // All other supported geometries use the same fragment format for f32 and
      // f16, so we only need to consider {fragment, type}.
      !eq(ft,"a:f16") : !listsplat(llvm_v2f16_ty, 8),
      !eq(ft,"b:f16") : !listsplat(llvm_v2f16_ty, 8),
      !eq(ft,"c:f16") : !listsplat(llvm_v2f16_ty, 4),
      !eq(ft,"d:f16") : !listsplat(llvm_v2f16_ty, 4),
      !eq(ft,"c:f32") : !listsplat(llvm_float_ty, 8),
      !eq(ft,"d:f32") : !listsplat(llvm_float_ty, 8),

      // wmma tf32 -> s32 @ m16n16k8
      !eq(gft,"m16n16k8:a:tf32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n16k8:b:tf32") : !listsplat(llvm_i32_ty, 4),

      // mma tf32 -> s32 @ m16n16k8/m16n8k8
      !eq(gft,"m16n8k4:a:tf32") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k4:b:tf32") : [llvm_i32_ty],
      !eq(gft,"m16n8k8:a:tf32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k8:b:tf32") : !listsplat(llvm_i32_ty, 2),

      !eq(gft,"m8n8k4:a:f64") : [llvm_double_ty],
      !eq(gft,"m8n8k4:b:f64") : [llvm_double_ty],
      !eq(gft,"m8n8k4:c:f64") : !listsplat(llvm_double_ty, 2),
      !eq(gft,"m8n8k4:d:f64") : !listsplat(llvm_double_ty, 2),

      !eq(gft,"m16n8k4:a:f64") : !listsplat(llvm_double_ty, 2),
      !eq(gft,"m16n8k4:b:f64") : [llvm_double_ty],
      !eq(gft,"m16n8k4:c:f64") : !listsplat(llvm_double_ty, 4),
      !eq(gft,"m16n8k4:d:f64") : !listsplat(llvm_double_ty, 4),

      !eq(gft,"m16n8k8:a:f64") : !listsplat(llvm_double_ty, 4),
      !eq(gft,"m16n8k8:b:f64") : !listsplat(llvm_double_ty, 2),
      !eq(gft,"m16n8k8:c:f64") : !listsplat(llvm_double_ty, 4),
````
- **L289 EN**: Comment explains nearby logic, invariants, or intent: `All other supported geometries use the same fragment format for f32 and`.
  **L289 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All other supported geometries use the same fragment format for f32 and`。
- **L290 EN**: Comment explains nearby logic, invariants, or intent: `f16, so we only need to consider {fragment, type}.`.
  **L290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f16, so we only need to consider {fragment, type}.`。
- **L291 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(ft,"a:f16") : !listsplat(llvm_v2f16_ty, 8),`.
  **L291 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(ft,"a:f16") : !listsplat(llvm_v2f16_ty, 8),`。
- **L292 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(ft,"b:f16") : !listsplat(llvm_v2f16_ty, 8),`.
  **L292 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(ft,"b:f16") : !listsplat(llvm_v2f16_ty, 8),`。
- **L293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(ft,"c:f16") : !listsplat(llvm_v2f16_ty, 4),`.
  **L293 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(ft,"c:f16") : !listsplat(llvm_v2f16_ty, 4),`。
- **L294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(ft,"d:f16") : !listsplat(llvm_v2f16_ty, 4),`.
  **L294 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(ft,"d:f16") : !listsplat(llvm_v2f16_ty, 4),`。
- **L295 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(ft,"c:f32") : !listsplat(llvm_float_ty, 8),`.
  **L295 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(ft,"c:f32") : !listsplat(llvm_float_ty, 8),`。
- **L296 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(ft,"d:f32") : !listsplat(llvm_float_ty, 8),`.
  **L296 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(ft,"d:f32") : !listsplat(llvm_float_ty, 8),`。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L298 EN**: Comment explains nearby logic, invariants, or intent: `wmma tf32 -> s32 @ m16n16k8`.
  **L298 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wmma tf32 -> s32 @ m16n16k8`。
- **L299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k8:a:tf32") : !listsplat(llvm_i32_ty, 4),`.
  **L299 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k8:a:tf32") : !listsplat(llvm_i32_ty, 4),`。
- **L300 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k8:b:tf32") : !listsplat(llvm_i32_ty, 4),`.
  **L300 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k8:b:tf32") : !listsplat(llvm_i32_ty, 4),`。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L302 EN**: Comment explains nearby logic, invariants, or intent: `mma tf32 -> s32 @ m16n16k8/m16n8k8`.
  **L302 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma tf32 -> s32 @ m16n16k8/m16n8k8`。
- **L303 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:a:tf32") : !listsplat(llvm_i32_ty, 2),`.
  **L303 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:a:tf32") : !listsplat(llvm_i32_ty, 2),`。
- **L304 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:b:tf32") : [llvm_i32_ty],`.
  **L304 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:b:tf32") : [llvm_i32_ty],`。
- **L305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:a:tf32") : !listsplat(llvm_i32_ty, 4),`.
  **L305 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:a:tf32") : !listsplat(llvm_i32_ty, 4),`。
- **L306 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:b:tf32") : !listsplat(llvm_i32_ty, 2),`.
  **L306 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:b:tf32") : !listsplat(llvm_i32_ty, 2),`。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k4:a:f64") : [llvm_double_ty],`.
  **L308 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k4:a:f64") : [llvm_double_ty],`。
- **L309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k4:b:f64") : [llvm_double_ty],`.
  **L309 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k4:b:f64") : [llvm_double_ty],`。
- **L310 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k4:c:f64") : !listsplat(llvm_double_ty, 2),`.
  **L310 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k4:c:f64") : !listsplat(llvm_double_ty, 2),`。
- **L311 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k4:d:f64") : !listsplat(llvm_double_ty, 2),`.
  **L311 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k4:d:f64") : !listsplat(llvm_double_ty, 2),`。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L313 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:a:f64") : !listsplat(llvm_double_ty, 2),`.
  **L313 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:a:f64") : !listsplat(llvm_double_ty, 2),`。
- **L314 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:b:f64") : [llvm_double_ty],`.
  **L314 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:b:f64") : [llvm_double_ty],`。
- **L315 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:c:f64") : !listsplat(llvm_double_ty, 4),`.
  **L315 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:c:f64") : !listsplat(llvm_double_ty, 4),`。
- **L316 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k4:d:f64") : !listsplat(llvm_double_ty, 4),`.
  **L316 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k4:d:f64") : !listsplat(llvm_double_ty, 4),`。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L318 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:a:f64") : !listsplat(llvm_double_ty, 4),`.
  **L318 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:a:f64") : !listsplat(llvm_double_ty, 4),`。
- **L319 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:b:f64") : !listsplat(llvm_double_ty, 2),`.
  **L319 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:b:f64") : !listsplat(llvm_double_ty, 2),`。
- **L320 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:c:f64") : !listsplat(llvm_double_ty, 4),`.
  **L320 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:c:f64") : !listsplat(llvm_double_ty, 4),`。

### Lines 321-352

````tablegen
      !eq(gft,"m16n8k8:d:f64") : !listsplat(llvm_double_ty, 4),

      !eq(gft,"m16n8k16:a:f64") : !listsplat(llvm_double_ty, 8),
      !eq(gft,"m16n8k16:b:f64") : !listsplat(llvm_double_ty, 4),
      !eq(gft,"m16n8k16:c:f64") : !listsplat(llvm_double_ty, 4),
      !eq(gft,"m16n8k16:d:f64") : !listsplat(llvm_double_ty, 4),

      // wmma bf16 -> s32 @ m16n16k16/m8n32k16/m32n8k16
      !eq(gft,"m16n16k16:a:bf16") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n16k16:b:bf16") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m8n32k16:a:bf16") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m8n32k16:b:bf16") : !listsplat(llvm_i32_ty, 8),
      !eq(gft,"m32n8k16:a:bf16") : !listsplat(llvm_i32_ty, 8),
      !eq(gft,"m32n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),

      // mma bf16 -> s32 @ m16n8k16/m16n8k8
      !eq(gft,"m16n8k16:a:bf16") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k8:a:bf16") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k8:b:bf16") : [llvm_i32_ty],

      // wmma u8/s8 -> s32 @ m16n16k16/m8n32k16/m32n8k16
      !eq(gft,"m16n16k16:a:u8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n16k16:a:s8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n16k16:b:u8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n16k16:b:s8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n16k16:c:s32") : !listsplat(llvm_i32_ty, 8),
      !eq(gft,"m16n16k16:d:s32") : !listsplat(llvm_i32_ty, 8),

      !eq(gft,"m8n32k16:a:u8") : [llvm_i32_ty],
      !eq(gft,"m8n32k16:a:s8") : [llvm_i32_ty],
      !eq(gft,"m8n32k16:b:u8") : !listsplat(llvm_i32_ty, 4),
````
- **L321 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:d:f64") : !listsplat(llvm_double_ty, 4),`.
  **L321 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:d:f64") : !listsplat(llvm_double_ty, 4),`。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:f64") : !listsplat(llvm_double_ty, 8),`.
  **L323 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:f64") : !listsplat(llvm_double_ty, 8),`。
- **L324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:f64") : !listsplat(llvm_double_ty, 4),`.
  **L324 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:f64") : !listsplat(llvm_double_ty, 4),`。
- **L325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:c:f64") : !listsplat(llvm_double_ty, 4),`.
  **L325 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:c:f64") : !listsplat(llvm_double_ty, 4),`。
- **L326 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:d:f64") : !listsplat(llvm_double_ty, 4),`.
  **L326 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:d:f64") : !listsplat(llvm_double_ty, 4),`。
- **L327 EN**: Blank line separating nearby declarations or logic blocks.
  **L327 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L328 EN**: Comment explains nearby logic, invariants, or intent: `wmma bf16 -> s32 @ m16n16k16/m8n32k16/m32n8k16`.
  **L328 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wmma bf16 -> s32 @ m16n16k16/m8n32k16/m32n8k16`。
- **L329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:a:bf16") : !listsplat(llvm_i32_ty, 4),`.
  **L329 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:a:bf16") : !listsplat(llvm_i32_ty, 4),`。
- **L330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:b:bf16") : !listsplat(llvm_i32_ty, 4),`.
  **L330 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:b:bf16") : !listsplat(llvm_i32_ty, 4),`。
- **L331 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:a:bf16") : !listsplat(llvm_i32_ty, 2),`.
  **L331 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:a:bf16") : !listsplat(llvm_i32_ty, 2),`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:b:bf16") : !listsplat(llvm_i32_ty, 8),`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:b:bf16") : !listsplat(llvm_i32_ty, 8),`。
- **L333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:a:bf16") : !listsplat(llvm_i32_ty, 8),`.
  **L333 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:a:bf16") : !listsplat(llvm_i32_ty, 8),`。
- **L334 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),`.
  **L334 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),`。
- **L335 EN**: Blank line separating nearby declarations or logic blocks.
  **L335 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L336 EN**: Comment explains nearby logic, invariants, or intent: `mma bf16 -> s32 @ m16n8k16/m16n8k8`.
  **L336 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma bf16 -> s32 @ m16n8k16/m16n8k8`。
- **L337 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:bf16") : !listsplat(llvm_i32_ty, 4),`.
  **L337 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:bf16") : !listsplat(llvm_i32_ty, 4),`。
- **L338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),`.
  **L338 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:bf16") : !listsplat(llvm_i32_ty, 2),`。
- **L339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:a:bf16") : !listsplat(llvm_i32_ty, 2),`.
  **L339 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:a:bf16") : !listsplat(llvm_i32_ty, 2),`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k8:b:bf16") : [llvm_i32_ty],`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k8:b:bf16") : [llvm_i32_ty],`。
- **L341 EN**: Blank line separating nearby declarations or logic blocks.
  **L341 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L342 EN**: Comment explains nearby logic, invariants, or intent: `wmma u8/s8 -> s32 @ m16n16k16/m8n32k16/m32n8k16`.
  **L342 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wmma u8/s8 -> s32 @ m16n16k16/m8n32k16/m32n8k16`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:a:u8") : !listsplat(llvm_i32_ty, 2),`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:a:u8") : !listsplat(llvm_i32_ty, 2),`。
- **L344 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:a:s8") : !listsplat(llvm_i32_ty, 2),`.
  **L344 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:a:s8") : !listsplat(llvm_i32_ty, 2),`。
- **L345 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:b:u8") : !listsplat(llvm_i32_ty, 2),`.
  **L345 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:b:u8") : !listsplat(llvm_i32_ty, 2),`。
- **L346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:b:s8") : !listsplat(llvm_i32_ty, 2),`.
  **L346 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:b:s8") : !listsplat(llvm_i32_ty, 2),`。
- **L347 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:c:s32") : !listsplat(llvm_i32_ty, 8),`.
  **L347 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:c:s32") : !listsplat(llvm_i32_ty, 8),`。
- **L348 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n16k16:d:s32") : !listsplat(llvm_i32_ty, 8),`.
  **L348 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n16k16:d:s32") : !listsplat(llvm_i32_ty, 8),`。
- **L349 EN**: Blank line separating nearby declarations or logic blocks.
  **L349 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L350 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:a:u8") : [llvm_i32_ty],`.
  **L350 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:a:u8") : [llvm_i32_ty],`。
- **L351 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:a:s8") : [llvm_i32_ty],`.
  **L351 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:a:s8") : [llvm_i32_ty],`。
- **L352 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:b:u8") : !listsplat(llvm_i32_ty, 4),`.
  **L352 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:b:u8") : !listsplat(llvm_i32_ty, 4),`。

### Lines 353-384

````tablegen
      !eq(gft,"m8n32k16:b:s8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m8n32k16:c:s32") : !listsplat(llvm_i32_ty, 8),
      !eq(gft,"m8n32k16:d:s32") : !listsplat(llvm_i32_ty, 8),

      !eq(gft,"m32n8k16:a:u8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m32n8k16:a:s8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m32n8k16:b:u8") : [llvm_i32_ty],
      !eq(gft,"m32n8k16:b:s8") : [llvm_i32_ty],
      !eq(gft,"m32n8k16:c:s32") : !listsplat(llvm_i32_ty, 8),
      !eq(gft,"m32n8k16:d:s32") : !listsplat(llvm_i32_ty, 8),

      // mma u8/s8 -> s32 @ m8n8k16/m16n8k16/m16n8k32
      !eq(gft,"m8n8k16:a:u8") : [llvm_i32_ty],
      !eq(gft,"m8n8k16:a:s8") : [llvm_i32_ty],
      !eq(gft,"m8n8k16:b:u8") : [llvm_i32_ty],
      !eq(gft,"m8n8k16:b:s8") : [llvm_i32_ty],
      !eq(gft,"m8n8k16:c:s32") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m8n8k16:d:s32") : !listsplat(llvm_i32_ty, 2),

      !eq(gft,"m16n8k16:a:u8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k16:a:s8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k16:b:u8") : [llvm_i32_ty],
      !eq(gft,"m16n8k16:b:s8") : [llvm_i32_ty],
      !eq(gft,"m16n8k16:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k16:d:s32") : !listsplat(llvm_i32_ty, 4),

      !eq(gft,"m16n8k32:a:u8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:a:s8") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:b:u8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:s8") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),
````
- **L353 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:b:s8") : !listsplat(llvm_i32_ty, 4),`.
  **L353 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:b:s8") : !listsplat(llvm_i32_ty, 4),`。
- **L354 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:c:s32") : !listsplat(llvm_i32_ty, 8),`.
  **L354 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:c:s32") : !listsplat(llvm_i32_ty, 8),`。
- **L355 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n32k16:d:s32") : !listsplat(llvm_i32_ty, 8),`.
  **L355 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n32k16:d:s32") : !listsplat(llvm_i32_ty, 8),`。
- **L356 EN**: Blank line separating nearby declarations or logic blocks.
  **L356 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L357 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:a:u8") : !listsplat(llvm_i32_ty, 4),`.
  **L357 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:a:u8") : !listsplat(llvm_i32_ty, 4),`。
- **L358 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:a:s8") : !listsplat(llvm_i32_ty, 4),`.
  **L358 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:a:s8") : !listsplat(llvm_i32_ty, 4),`。
- **L359 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:b:u8") : [llvm_i32_ty],`.
  **L359 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:b:u8") : [llvm_i32_ty],`。
- **L360 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:b:s8") : [llvm_i32_ty],`.
  **L360 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:b:s8") : [llvm_i32_ty],`。
- **L361 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:c:s32") : !listsplat(llvm_i32_ty, 8),`.
  **L361 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:c:s32") : !listsplat(llvm_i32_ty, 8),`。
- **L362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m32n8k16:d:s32") : !listsplat(llvm_i32_ty, 8),`.
  **L362 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m32n8k16:d:s32") : !listsplat(llvm_i32_ty, 8),`。
- **L363 EN**: Blank line separating nearby declarations or logic blocks.
  **L363 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L364 EN**: Comment explains nearby logic, invariants, or intent: `mma u8/s8 -> s32 @ m8n8k16/m16n8k16/m16n8k32`.
  **L364 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma u8/s8 -> s32 @ m8n8k16/m16n8k16/m16n8k32`。
- **L365 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k16:a:u8") : [llvm_i32_ty],`.
  **L365 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k16:a:u8") : [llvm_i32_ty],`。
- **L366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k16:a:s8") : [llvm_i32_ty],`.
  **L366 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k16:a:s8") : [llvm_i32_ty],`。
- **L367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k16:b:u8") : [llvm_i32_ty],`.
  **L367 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k16:b:u8") : [llvm_i32_ty],`。
- **L368 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k16:b:s8") : [llvm_i32_ty],`.
  **L368 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k16:b:s8") : [llvm_i32_ty],`。
- **L369 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k16:c:s32") : !listsplat(llvm_i32_ty, 2),`.
  **L369 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k16:c:s32") : !listsplat(llvm_i32_ty, 2),`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k16:d:s32") : !listsplat(llvm_i32_ty, 2),`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k16:d:s32") : !listsplat(llvm_i32_ty, 2),`。
- **L371 EN**: Blank line separating nearby declarations or logic blocks.
  **L371 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L372 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:u8") : !listsplat(llvm_i32_ty, 2),`.
  **L372 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:u8") : !listsplat(llvm_i32_ty, 2),`。
- **L373 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:s8") : !listsplat(llvm_i32_ty, 2),`.
  **L373 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:s8") : !listsplat(llvm_i32_ty, 2),`。
- **L374 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:u8") : [llvm_i32_ty],`.
  **L374 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:u8") : [llvm_i32_ty],`。
- **L375 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:s8") : [llvm_i32_ty],`.
  **L375 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:s8") : [llvm_i32_ty],`。
- **L376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L376 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L377 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L378 EN**: Blank line separating nearby declarations or logic blocks.
  **L378 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L379 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:u8") : !listsplat(llvm_i32_ty, 4),`.
  **L379 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:u8") : !listsplat(llvm_i32_ty, 4),`。
- **L380 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:s8") : !listsplat(llvm_i32_ty, 4),`.
  **L380 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:s8") : !listsplat(llvm_i32_ty, 4),`。
- **L381 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:u8") : !listsplat(llvm_i32_ty, 2),`.
  **L381 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:u8") : !listsplat(llvm_i32_ty, 2),`。
- **L382 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:s8") : !listsplat(llvm_i32_ty, 2),`.
  **L382 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:s8") : !listsplat(llvm_i32_ty, 2),`。
- **L383 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L383 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L384 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L384 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),`。

### Lines 385-416

````tablegen

      // wmma/mma u4/s4 -> s32 @ m8n8k32 (u4/s4)
      !eq(gft,"m8n8k32:a:u4") : [llvm_i32_ty],
      !eq(gft,"m8n8k32:a:s4") : [llvm_i32_ty],
      !eq(gft,"m8n8k32:b:u4") : [llvm_i32_ty],
      !eq(gft,"m8n8k32:b:s4") : [llvm_i32_ty],
      !eq(gft,"m8n8k32:c:s32") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m8n8k32:d:s32") : !listsplat(llvm_i32_ty, 2),

      !eq(gft,"m16n8k32:a:u4") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:a:s4") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:u4") : [llvm_i32_ty],
      !eq(gft,"m16n8k32:b:s4") : [llvm_i32_ty],
      !eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),

      !eq(gft,"m16n8k64:a:u4") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:a:s4") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:u4") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k64:b:s4") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k64:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:d:s32") : !listsplat(llvm_i32_ty, 4),

      // mma e4m3/e5m2 -> f16/f32 @ m16n8k16
      !eq(gft,"m16n8k16:a:e4m3") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k16:a:e5m2") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k16:b:e4m3") : [llvm_i32_ty],
      !eq(gft,"m16n8k16:b:e5m2") : [llvm_i32_ty],
      // mma e4m3/e5m2/e3m2/e2m3/e2m1 -> f32 @ m16n8k32
      !eq(gft,"m16n8k32:a:e4m3") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:a:e5m2") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:a:e3m2") : !listsplat(llvm_i32_ty, 4),
````
- **L385 EN**: Blank line separating nearby declarations or logic blocks.
  **L385 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L386 EN**: Comment explains nearby logic, invariants, or intent: `wmma/mma u4/s4 -> s32 @ m8n8k32 (u4/s4)`.
  **L386 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wmma/mma u4/s4 -> s32 @ m8n8k32 (u4/s4)`。
- **L387 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k32:a:u4") : [llvm_i32_ty],`.
  **L387 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k32:a:u4") : [llvm_i32_ty],`。
- **L388 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k32:a:s4") : [llvm_i32_ty],`.
  **L388 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k32:a:s4") : [llvm_i32_ty],`。
- **L389 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k32:b:u4") : [llvm_i32_ty],`.
  **L389 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k32:b:u4") : [llvm_i32_ty],`。
- **L390 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k32:b:s4") : [llvm_i32_ty],`.
  **L390 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k32:b:s4") : [llvm_i32_ty],`。
- **L391 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k32:c:s32") : !listsplat(llvm_i32_ty, 2),`.
  **L391 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k32:c:s32") : !listsplat(llvm_i32_ty, 2),`。
- **L392 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k32:d:s32") : !listsplat(llvm_i32_ty, 2),`.
  **L392 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k32:d:s32") : !listsplat(llvm_i32_ty, 2),`。
- **L393 EN**: Blank line separating nearby declarations or logic blocks.
  **L393 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L394 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:u4") : !listsplat(llvm_i32_ty, 2),`.
  **L394 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:u4") : !listsplat(llvm_i32_ty, 2),`。
- **L395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:s4") : !listsplat(llvm_i32_ty, 2),`.
  **L395 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:s4") : !listsplat(llvm_i32_ty, 2),`。
- **L396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:u4") : [llvm_i32_ty],`.
  **L396 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:u4") : [llvm_i32_ty],`。
- **L397 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:s4") : [llvm_i32_ty],`.
  **L397 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:s4") : [llvm_i32_ty],`。
- **L398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L398 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L399 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:u4") : !listsplat(llvm_i32_ty, 4),`.
  **L401 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:u4") : !listsplat(llvm_i32_ty, 4),`。
- **L402 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:s4") : !listsplat(llvm_i32_ty, 4),`.
  **L402 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:s4") : !listsplat(llvm_i32_ty, 4),`。
- **L403 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:u4") : !listsplat(llvm_i32_ty, 2),`.
  **L403 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:u4") : !listsplat(llvm_i32_ty, 2),`。
- **L404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:s4") : !listsplat(llvm_i32_ty, 2),`.
  **L404 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:s4") : !listsplat(llvm_i32_ty, 2),`。
- **L405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L405 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L406 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L407 EN**: Blank line separating nearby declarations or logic blocks.
  **L407 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L408 EN**: Comment explains nearby logic, invariants, or intent: `mma e4m3/e5m2 -> f16/f32 @ m16n8k16`.
  **L408 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma e4m3/e5m2 -> f16/f32 @ m16n8k16`。
- **L409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:e4m3") : !listsplat(llvm_i32_ty, 2),`.
  **L409 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:e4m3") : !listsplat(llvm_i32_ty, 2),`。
- **L410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:a:e5m2") : !listsplat(llvm_i32_ty, 2),`.
  **L410 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:a:e5m2") : !listsplat(llvm_i32_ty, 2),`。
- **L411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:e4m3") : [llvm_i32_ty],`.
  **L411 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:e4m3") : [llvm_i32_ty],`。
- **L412 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k16:b:e5m2") : [llvm_i32_ty],`.
  **L412 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k16:b:e5m2") : [llvm_i32_ty],`。
- **L413 EN**: Comment explains nearby logic, invariants, or intent: `mma e4m3/e5m2/e3m2/e2m3/e2m1 -> f32 @ m16n8k32`.
  **L413 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma e4m3/e5m2/e3m2/e2m3/e2m1 -> f32 @ m16n8k32`。
- **L414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:e4m3") : !listsplat(llvm_i32_ty, 4),`.
  **L414 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:e4m3") : !listsplat(llvm_i32_ty, 4),`。
- **L415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:e5m2") : !listsplat(llvm_i32_ty, 4),`.
  **L415 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:e5m2") : !listsplat(llvm_i32_ty, 4),`。
- **L416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:e3m2") : !listsplat(llvm_i32_ty, 4),`.
  **L416 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:e3m2") : !listsplat(llvm_i32_ty, 4),`。

### Lines 417-448

````tablegen
      !eq(gft,"m16n8k32:a:e2m3") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:a:e2m1") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k32:b:e4m3") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:e5m2") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:e3m2") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:e2m3") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k32:b:e2m1") : !listsplat(llvm_i32_ty, 2),
      // mma e2m1 -> f32 @m16n8k64
      !eq(gft,"m16n8k64:a:e2m1") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k64:b:e2m1") : !listsplat(llvm_i32_ty, 2),

      // wmma/mma b1 -> s32 @ m8n8k128(b1)
      !eq(gft,"m8n8k128:a:b1") : [llvm_i32_ty],
      !eq(gft,"m8n8k128:b:b1") : [llvm_i32_ty],
      !eq(gft,"m8n8k128:c:s32") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m8n8k128:d:s32") : !listsplat(llvm_i32_ty, 2),

      !eq(gft,"m16n8k128:a:b1") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k128:b:b1") : [llvm_i32_ty],
      !eq(gft,"m16n8k128:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k128:d:s32") : !listsplat(llvm_i32_ty, 4),

      !eq(gft,"m16n8k256:a:b1") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k256:b:b1") : !listsplat(llvm_i32_ty, 2),
      !eq(gft,"m16n8k256:c:s32") : !listsplat(llvm_i32_ty, 4),
      !eq(gft,"m16n8k256:d:s32") : !listsplat(llvm_i32_ty, 4),

      // ldmatrix b16 -> s32 @ m8n8
      !eq(gf,"m8n8:x1") : !listsplat(llvm_i32_ty, 1),
      !eq(gf,"m8n8:x2") : !listsplat(llvm_i32_ty, 2),
      !eq(gf,"m8n8:x4") : !listsplat(llvm_i32_ty, 4),

````
- **L417 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:e2m3") : !listsplat(llvm_i32_ty, 4),`.
  **L417 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:e2m3") : !listsplat(llvm_i32_ty, 4),`。
- **L418 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:a:e2m1") : !listsplat(llvm_i32_ty, 4),`.
  **L418 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:a:e2m1") : !listsplat(llvm_i32_ty, 4),`。
- **L419 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:e4m3") : !listsplat(llvm_i32_ty, 2),`.
  **L419 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:e4m3") : !listsplat(llvm_i32_ty, 2),`。
- **L420 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:e5m2") : !listsplat(llvm_i32_ty, 2),`.
  **L420 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:e5m2") : !listsplat(llvm_i32_ty, 2),`。
- **L421 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:e3m2") : !listsplat(llvm_i32_ty, 2),`.
  **L421 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:e3m2") : !listsplat(llvm_i32_ty, 2),`。
- **L422 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:e2m3") : !listsplat(llvm_i32_ty, 2),`.
  **L422 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:e2m3") : !listsplat(llvm_i32_ty, 2),`。
- **L423 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k32:b:e2m1") : !listsplat(llvm_i32_ty, 2),`.
  **L423 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k32:b:e2m1") : !listsplat(llvm_i32_ty, 2),`。
- **L424 EN**: Comment explains nearby logic, invariants, or intent: `mma e2m1 -> f32 @m16n8k64`.
  **L424 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mma e2m1 -> f32 @m16n8k64`。
- **L425 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:a:e2m1") : !listsplat(llvm_i32_ty, 4),`.
  **L425 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:a:e2m1") : !listsplat(llvm_i32_ty, 4),`。
- **L426 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k64:b:e2m1") : !listsplat(llvm_i32_ty, 2),`.
  **L426 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k64:b:e2m1") : !listsplat(llvm_i32_ty, 2),`。
- **L427 EN**: Blank line separating nearby declarations or logic blocks.
  **L427 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L428 EN**: Comment explains nearby logic, invariants, or intent: `wmma/mma b1 -> s32 @ m8n8k128(b1)`.
  **L428 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wmma/mma b1 -> s32 @ m8n8k128(b1)`。
- **L429 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k128:a:b1") : [llvm_i32_ty],`.
  **L429 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k128:a:b1") : [llvm_i32_ty],`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k128:b:b1") : [llvm_i32_ty],`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k128:b:b1") : [llvm_i32_ty],`。
- **L431 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k128:c:s32") : !listsplat(llvm_i32_ty, 2),`.
  **L431 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k128:c:s32") : !listsplat(llvm_i32_ty, 2),`。
- **L432 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m8n8k128:d:s32") : !listsplat(llvm_i32_ty, 2),`.
  **L432 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m8n8k128:d:s32") : !listsplat(llvm_i32_ty, 2),`。
- **L433 EN**: Blank line separating nearby declarations or logic blocks.
  **L433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L434 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:a:b1") : !listsplat(llvm_i32_ty, 2),`.
  **L434 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:a:b1") : !listsplat(llvm_i32_ty, 2),`。
- **L435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:b:b1") : [llvm_i32_ty],`.
  **L435 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:b:b1") : [llvm_i32_ty],`。
- **L436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L436 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k128:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L437 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k128:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L438 EN**: Blank line separating nearby declarations or logic blocks.
  **L438 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L439 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k256:a:b1") : !listsplat(llvm_i32_ty, 4),`.
  **L439 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k256:a:b1") : !listsplat(llvm_i32_ty, 4),`。
- **L440 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k256:b:b1") : !listsplat(llvm_i32_ty, 2),`.
  **L440 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k256:b:b1") : !listsplat(llvm_i32_ty, 2),`。
- **L441 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k256:c:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L441 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k256:c:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gft,"m16n8k256:d:s32") : !listsplat(llvm_i32_ty, 4),`.
  **L442 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gft,"m16n8k256:d:s32") : !listsplat(llvm_i32_ty, 4),`。
- **L443 EN**: Blank line separating nearby declarations or logic blocks.
  **L443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L444 EN**: Comment explains nearby logic, invariants, or intent: `ldmatrix b16 -> s32 @ m8n8`.
  **L444 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ldmatrix b16 -> s32 @ m8n8`。
- **L445 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m8n8:x1") : !listsplat(llvm_i32_ty, 1),`.
  **L445 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m8n8:x1") : !listsplat(llvm_i32_ty, 1),`。
- **L446 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m8n8:x2") : !listsplat(llvm_i32_ty, 2),`.
  **L446 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m8n8:x2") : !listsplat(llvm_i32_ty, 2),`。
- **L447 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m8n8:x4") : !listsplat(llvm_i32_ty, 4),`.
  **L447 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m8n8:x4") : !listsplat(llvm_i32_ty, 4),`。
- **L448 EN**: Blank line separating nearby declarations or logic blocks.
  **L448 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 449-480

````tablegen
      // ldmatrix b8, b8x16.b6x16_p32, b8x16.b4x16_p64 -> s32 @ m16n16
      !eq(gf,"m16n16:x1") : !listsplat(llvm_i32_ty, 2),
      !eq(gf,"m16n16:x2") : !listsplat(llvm_i32_ty, 4),

      // ldmatrix b8x16.b6x16_p32, b8x16.b4x16_p64 -> s32 @ m8n16
      !eq(gf,"m8n16:x1") : !listsplat(llvm_i32_ty, 1),
      !eq(gf,"m8n16:x2") : !listsplat(llvm_i32_ty, 2),
      !eq(gf,"m8n16:x4") : !listsplat(llvm_i32_ty, 4),

      // stmatrix b8 -> s32 @ m16n8
      !eq(gf,"m16n8:x1") : !listsplat(llvm_i32_ty, 1),
      !eq(gf,"m16n8:x2") : !listsplat(llvm_i32_ty, 2),
      !eq(gf,"m16n8:x4") : !listsplat(llvm_i32_ty, 4),
    )
  );
}

class WMMA_NAME_LDST<string Op, WMMA_REGS Frag, string Layout, int WithStride> {
  string intr_name = "llvm.nvvm.wmma."
                # Frag.geom
                # "." # Op
                # "." # Frag.frag
                # "." # Layout
                # !if(WithStride, ".stride", "")
                # "." # Frag.ptx_elt_type
                ;
  // TODO(tra): record name should ideally use the same field order as the intrinsic.
  // E.g. string record = !subst("llvm", "int",
  //                      !subst(".", "_", llvm));
  string record_name = "int_nvvm_wmma_"
                # Frag.geom
                # "_" # Op
````
- **L449 EN**: Comment explains nearby logic, invariants, or intent: `ldmatrix b8, b8x16.b6x16_p32, b8x16.b4x16_p64 -> s32 @ m16n16`.
  **L449 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ldmatrix b8, b8x16.b6x16_p32, b8x16.b4x16_p64 -> s32 @ m16n16`。
- **L450 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m16n16:x1") : !listsplat(llvm_i32_ty, 2),`.
  **L450 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m16n16:x1") : !listsplat(llvm_i32_ty, 2),`。
- **L451 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m16n16:x2") : !listsplat(llvm_i32_ty, 4),`.
  **L451 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m16n16:x2") : !listsplat(llvm_i32_ty, 4),`。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L453 EN**: Comment explains nearby logic, invariants, or intent: `ldmatrix b8x16.b6x16_p32, b8x16.b4x16_p64 -> s32 @ m8n16`.
  **L453 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ldmatrix b8x16.b6x16_p32, b8x16.b4x16_p64 -> s32 @ m8n16`。
- **L454 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m8n16:x1") : !listsplat(llvm_i32_ty, 1),`.
  **L454 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m8n16:x1") : !listsplat(llvm_i32_ty, 1),`。
- **L455 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m8n16:x2") : !listsplat(llvm_i32_ty, 2),`.
  **L455 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m8n16:x2") : !listsplat(llvm_i32_ty, 2),`。
- **L456 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m8n16:x4") : !listsplat(llvm_i32_ty, 4),`.
  **L456 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m8n16:x4") : !listsplat(llvm_i32_ty, 4),`。
- **L457 EN**: Blank line separating nearby declarations or logic blocks.
  **L457 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L458 EN**: Comment explains nearby logic, invariants, or intent: `stmatrix b8 -> s32 @ m16n8`.
  **L458 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`stmatrix b8 -> s32 @ m16n8`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m16n8:x1") : !listsplat(llvm_i32_ty, 1),`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m16n8:x1") : !listsplat(llvm_i32_ty, 1),`。
- **L460 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m16n8:x2") : !listsplat(llvm_i32_ty, 2),`.
  **L460 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m16n8:x2") : !listsplat(llvm_i32_ty, 2),`。
- **L461 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gf,"m16n8:x4") : !listsplat(llvm_i32_ty, 4),`.
  **L461 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gf,"m16n8:x4") : !listsplat(llvm_i32_ty, 4),`。
- **L462 EN**: Continues the surrounding expression or declaration: `)`.
  **L462 CN**: 继续构造周围的表达式或声明：`)`。
- **L463 EN**: Executes a standalone statement or declaration: `);`.
  **L463 CN**: 执行一条独立语句或声明：`);`。
- **L464 EN**: Closes the current lexical scope or compound statement.
  **L464 CN**: 结束当前词法作用域或复合语句块。
- **L465 EN**: Blank line separating nearby declarations or logic blocks.
  **L465 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L466 EN**: Declares class `WMMA_NAME_LDST<string`.
  **L466 CN**: 声明 class `WMMA_NAME_LDST<string`。
- **L467 EN**: Continues the surrounding expression or declaration: `string intr_name = "llvm.nvvm.wmma."`.
  **L467 CN**: 继续构造周围的表达式或声明：`string intr_name = "llvm.nvvm.wmma."`。
- **L468 EN**: Continues the surrounding expression or declaration: `# Frag.geom`.
  **L468 CN**: 继续构造周围的表达式或声明：`# Frag.geom`。
- **L469 EN**: Continues the surrounding expression or declaration: `# "." # Op`.
  **L469 CN**: 继续构造周围的表达式或声明：`# "." # Op`。
- **L470 EN**: Continues the surrounding expression or declaration: `# "." # Frag.frag`.
  **L470 CN**: 继续构造周围的表达式或声明：`# "." # Frag.frag`。
- **L471 EN**: Continues the surrounding expression or declaration: `# "." # Layout`.
  **L471 CN**: 继续构造周围的表达式或声明：`# "." # Layout`。
- **L472 EN**: Continues the surrounding expression or declaration: `# !if(WithStride, ".stride", "")`.
  **L472 CN**: 继续构造周围的表达式或声明：`# !if(WithStride, ".stride", "")`。
- **L473 EN**: Continues the surrounding expression or declaration: `# "." # Frag.ptx_elt_type`.
  **L473 CN**: 继续构造周围的表达式或声明：`# "." # Frag.ptx_elt_type`。
- **L474 EN**: Executes a standalone statement or declaration: `;`.
  **L474 CN**: 执行一条独立语句或声明：`;`。
- **L475 EN**: Comment records a pending task or caution: `TODO(tra): record name should ideally use the same field order as the intrinsic.`.
  **L475 CN**: 注释记录了待办事项或注意点：`TODO(tra): record name should ideally use the same field order as the intrinsic.`。
- **L476 EN**: Comment explains nearby logic, invariants, or intent: `E.g. string record = !subst("llvm", "int",`.
  **L476 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`E.g. string record = !subst("llvm", "int",`。
- **L477 EN**: Comment explains nearby logic, invariants, or intent: `subst(".", "_", llvm));`.
  **L477 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`subst(".", "_", llvm));`。
- **L478 EN**: Continues the surrounding expression or declaration: `string record_name = "int_nvvm_wmma_"`.
  **L478 CN**: 继续构造周围的表达式或声明：`string record_name = "int_nvvm_wmma_"`。
- **L479 EN**: Continues the surrounding expression or declaration: `# Frag.geom`.
  **L479 CN**: 继续构造周围的表达式或声明：`# Frag.geom`。
- **L480 EN**: Continues the surrounding expression or declaration: `# "_" # Op`.
  **L480 CN**: 继续构造周围的表达式或声明：`# "_" # Op`。

### Lines 481-512

````tablegen
                # "_" # Frag.frag
                # "_" # Frag.ptx_elt_type
                # "_" # Layout
                # !if(WithStride, "_stride", "");
}

class MMA_SIGNATURE<WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {
  list<WMMA_REGS> id_frags = !cond(
     // FP8/F8F6F4 ops are identified by A,B inputs & accomulator & result type.
     !or(!eq(A.ptx_elt_type, "e4m3"),
         !eq(A.ptx_elt_type, "e5m2"),
         !eq(A.ptx_elt_type, "e3m2"),
         !eq(A.ptx_elt_type, "e2m3"),
         !eq(A.ptx_elt_type, "e2m1")): [D, A, B, C],
     // FP16 ops are identified by accumulator & result type.
     !eq(A.ptx_elt_type, "f16") : [D, C],
     // other ops are identified by input types.
     !ne(A.ptx_elt_type, B.ptx_elt_type): [A, B],
     true: [A]
     );
   string ret = !foldl("", id_frags, a, b, !strconcat(a, "_", b.ptx_elt_type));
}

class WMMA_NAME<string ALayout, string BLayout, int Satfinite, string Rnd, string b1op,
                WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {
  string signature = MMA_SIGNATURE<A, B, C, D>.ret;
  string record_name = "int_nvvm_wmma_"
                  # A.geom
                  # "_mma"
                  # !subst(".", "_", b1op)
                  # "_" # ALayout
                  # "_" # BLayout
````
- **L481 EN**: Continues the surrounding expression or declaration: `# "_" # Frag.frag`.
  **L481 CN**: 继续构造周围的表达式或声明：`# "_" # Frag.frag`。
- **L482 EN**: Continues the surrounding expression or declaration: `# "_" # Frag.ptx_elt_type`.
  **L482 CN**: 继续构造周围的表达式或声明：`# "_" # Frag.ptx_elt_type`。
- **L483 EN**: Continues the surrounding expression or declaration: `# "_" # Layout`.
  **L483 CN**: 继续构造周围的表达式或声明：`# "_" # Layout`。
- **L484 EN**: Executes a call or declaration centered on `!if`.
  **L484 CN**: 执行以 `!if` 为核心的调用或声明。
- **L485 EN**: Closes the current lexical scope or compound statement.
  **L485 CN**: 结束当前词法作用域或复合语句块。
- **L486 EN**: Blank line separating nearby declarations or logic blocks.
  **L486 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L487 EN**: Declares class `MMA_SIGNATURE<WMMA_REGS`.
  **L487 CN**: 声明 class `MMA_SIGNATURE<WMMA_REGS`。
- **L488 EN**: Continues logic associated with callable symbol `cond`.
  **L488 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L489 EN**: Comment explains nearby logic, invariants, or intent: `FP8/F8F6F4 ops are identified by A,B inputs & accomulator & result type.`.
  **L489 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP8/F8F6F4 ops are identified by A,B inputs & accomulator & result type.`。
- **L490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(A.ptx_elt_type, "e4m3"),`.
  **L490 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(A.ptx_elt_type, "e4m3"),`。
- **L491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e5m2"),`.
  **L491 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e5m2"),`。
- **L492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e3m2"),`.
  **L492 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e3m2"),`。
- **L493 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e2m3"),`.
  **L493 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e2m3"),`。
- **L494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e2m1")): [D, A, B, C],`.
  **L494 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e2m1")): [D, A, B, C],`。
- **L495 EN**: Comment explains nearby logic, invariants, or intent: `FP16 ops are identified by accumulator & result type.`.
  **L495 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP16 ops are identified by accumulator & result type.`。
- **L496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "f16") : [D, C],`.
  **L496 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "f16") : [D, C],`。
- **L497 EN**: Comment explains nearby logic, invariants, or intent: `other ops are identified by input types.`.
  **L497 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`other ops are identified by input types.`。
- **L498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(A.ptx_elt_type, B.ptx_elt_type): [A, B],`.
  **L498 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(A.ptx_elt_type, B.ptx_elt_type): [A, B],`。
- **L499 EN**: Continues the surrounding expression or declaration: `true: [A]`.
  **L499 CN**: 继续构造周围的表达式或声明：`true: [A]`。
- **L500 EN**: Executes a standalone statement or declaration: `);`.
  **L500 CN**: 执行一条独立语句或声明：`);`。
- **L501 EN**: Initializes variable `ret` from the right-hand expression.
  **L501 CN**: 使用右侧表达式初始化变量 `ret`。
- **L502 EN**: Closes the current lexical scope or compound statement.
  **L502 CN**: 结束当前词法作用域或复合语句块。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L504 EN**: Declares class `WMMA_NAME<string`.
  **L504 CN**: 声明 class `WMMA_NAME<string`。
- **L505 EN**: Continues the surrounding expression or declaration: `WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {`.
  **L505 CN**: 继续构造周围的表达式或声明：`WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {`。
- **L506 EN**: Initializes variable `signature` from the right-hand expression.
  **L506 CN**: 使用右侧表达式初始化变量 `signature`。
- **L507 EN**: Continues the surrounding expression or declaration: `string record_name = "int_nvvm_wmma_"`.
  **L507 CN**: 继续构造周围的表达式或声明：`string record_name = "int_nvvm_wmma_"`。
- **L508 EN**: Continues the surrounding expression or declaration: `# A.geom`.
  **L508 CN**: 继续构造周围的表达式或声明：`# A.geom`。
- **L509 EN**: Continues the surrounding expression or declaration: `# "_mma"`.
  **L509 CN**: 继续构造周围的表达式或声明：`# "_mma"`。
- **L510 EN**: Continues logic associated with callable symbol `subst`.
  **L510 CN**: 继续与可调用符号 `subst` 相关的逻辑。
- **L511 EN**: Continues the surrounding expression or declaration: `# "_" # ALayout`.
  **L511 CN**: 继续构造周围的表达式或声明：`# "_" # ALayout`。
- **L512 EN**: Continues the surrounding expression or declaration: `# "_" # BLayout`.
  **L512 CN**: 继续构造周围的表达式或声明：`# "_" # BLayout`。

### Lines 513-544

````tablegen
                  # !if(!ne(Rnd, ""), !strconcat("_", Rnd), "")
                  # signature
                  # !if(Satfinite, "_satfinite", "");
}

class MMA_NAME<string ALayout, string BLayout, int Satfinite, string b1op, string Kind,
               WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {
  string signature = MMA_SIGNATURE<A, B, C, D>.ret;
  string record_name = "int_nvvm_mma"
                  # !subst(".", "_", b1op)
                  # "_" # A.geom
                  # "_" # ALayout
                  # "_" # BLayout
                  # !if(!ne(Kind, ""), !strconcat("_", !subst("::", "_", Kind)), "")
                  # !if(Satfinite, "_satfinite", "")
                  # signature;
}

class MMA_BLOCK_SCALE_NAME<string Kind, string SType, string ScaleVecSize,
                           WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {
  string signature = MMA_SIGNATURE<A, B, C, D>.ret;
  string record_name = "int_nvvm_mma_block_scale"
                  # "_" # A.geom
                  # "_row_col"
                  # "_" # Kind
                  # !subst(".", "_", ScaleVecSize)
                  # signature
                  # "_" # SType;
}

class MMA_SP_NAME<string Metadata, string Kind, int Satfinite,
                  WMMA_REGS A, WMMA_REGS B,
````
- **L513 EN**: Continues the surrounding expression or declaration: `# !if(!ne(Rnd, ""), !strconcat("_", Rnd), "")`.
  **L513 CN**: 继续构造周围的表达式或声明：`# !if(!ne(Rnd, ""), !strconcat("_", Rnd), "")`。
- **L514 EN**: Continues the surrounding expression or declaration: `# signature`.
  **L514 CN**: 继续构造周围的表达式或声明：`# signature`。
- **L515 EN**: Executes a call or declaration centered on `!if`.
  **L515 CN**: 执行以 `!if` 为核心的调用或声明。
- **L516 EN**: Closes the current lexical scope or compound statement.
  **L516 CN**: 结束当前词法作用域或复合语句块。
- **L517 EN**: Blank line separating nearby declarations or logic blocks.
  **L517 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L518 EN**: Declares class `MMA_NAME<string`.
  **L518 CN**: 声明 class `MMA_NAME<string`。
- **L519 EN**: Continues the surrounding expression or declaration: `WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {`.
  **L519 CN**: 继续构造周围的表达式或声明：`WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {`。
- **L520 EN**: Initializes variable `signature` from the right-hand expression.
  **L520 CN**: 使用右侧表达式初始化变量 `signature`。
- **L521 EN**: Continues the surrounding expression or declaration: `string record_name = "int_nvvm_mma"`.
  **L521 CN**: 继续构造周围的表达式或声明：`string record_name = "int_nvvm_mma"`。
- **L522 EN**: Continues logic associated with callable symbol `subst`.
  **L522 CN**: 继续与可调用符号 `subst` 相关的逻辑。
- **L523 EN**: Continues the surrounding expression or declaration: `# "_" # A.geom`.
  **L523 CN**: 继续构造周围的表达式或声明：`# "_" # A.geom`。
- **L524 EN**: Continues the surrounding expression or declaration: `# "_" # ALayout`.
  **L524 CN**: 继续构造周围的表达式或声明：`# "_" # ALayout`。
- **L525 EN**: Continues the surrounding expression or declaration: `# "_" # BLayout`.
  **L525 CN**: 继续构造周围的表达式或声明：`# "_" # BLayout`。
- **L526 EN**: Continues the surrounding expression or declaration: `# !if(!ne(Kind, ""), !strconcat("_", !subst("::", "_", Kind)), "")`.
  **L526 CN**: 继续构造周围的表达式或声明：`# !if(!ne(Kind, ""), !strconcat("_", !subst("::", "_", Kind)), "")`。
- **L527 EN**: Continues the surrounding expression or declaration: `# !if(Satfinite, "_satfinite", "")`.
  **L527 CN**: 继续构造周围的表达式或声明：`# !if(Satfinite, "_satfinite", "")`。
- **L528 EN**: Executes a standalone statement or declaration: `# signature;`.
  **L528 CN**: 执行一条独立语句或声明：`# signature;`。
- **L529 EN**: Closes the current lexical scope or compound statement.
  **L529 CN**: 结束当前词法作用域或复合语句块。
- **L530 EN**: Blank line separating nearby declarations or logic blocks.
  **L530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L531 EN**: Declares class `MMA_BLOCK_SCALE_NAME<string`.
  **L531 CN**: 声明 class `MMA_BLOCK_SCALE_NAME<string`。
- **L532 EN**: Continues the surrounding expression or declaration: `WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {`.
  **L532 CN**: 继续构造周围的表达式或声明：`WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D> {`。
- **L533 EN**: Initializes variable `signature` from the right-hand expression.
  **L533 CN**: 使用右侧表达式初始化变量 `signature`。
- **L534 EN**: Continues the surrounding expression or declaration: `string record_name = "int_nvvm_mma_block_scale"`.
  **L534 CN**: 继续构造周围的表达式或声明：`string record_name = "int_nvvm_mma_block_scale"`。
- **L535 EN**: Continues the surrounding expression or declaration: `# "_" # A.geom`.
  **L535 CN**: 继续构造周围的表达式或声明：`# "_" # A.geom`。
- **L536 EN**: Continues the surrounding expression or declaration: `# "_row_col"`.
  **L536 CN**: 继续构造周围的表达式或声明：`# "_row_col"`。
- **L537 EN**: Continues the surrounding expression or declaration: `# "_" # Kind`.
  **L537 CN**: 继续构造周围的表达式或声明：`# "_" # Kind`。
- **L538 EN**: Continues logic associated with callable symbol `subst`.
  **L538 CN**: 继续与可调用符号 `subst` 相关的逻辑。
- **L539 EN**: Continues the surrounding expression or declaration: `# signature`.
  **L539 CN**: 继续构造周围的表达式或声明：`# signature`。
- **L540 EN**: Executes a standalone statement or declaration: `# "_" # SType;`.
  **L540 CN**: 执行一条独立语句或声明：`# "_" # SType;`。
- **L541 EN**: Closes the current lexical scope or compound statement.
  **L541 CN**: 结束当前词法作用域或复合语句块。
- **L542 EN**: Blank line separating nearby declarations or logic blocks.
  **L542 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L543 EN**: Declares class `MMA_SP_NAME<string`.
  **L543 CN**: 声明 class `MMA_SP_NAME<string`。
- **L544 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WMMA_REGS A, WMMA_REGS B,`.
  **L544 CN**: 继续一个多行参数列表、初始化器或聚合项：`WMMA_REGS A, WMMA_REGS B,`。

### Lines 545-576

````tablegen
                  WMMA_REGS C, WMMA_REGS D> {
  string signature = MMA_SIGNATURE<A, B, C, D>.ret;
  string record_name = "int_nvvm_mma"
                  # "_" # !subst("::", "_", Metadata)
                  # "_" # A.geom
                  # "_row_col"
                  # !if(!ne(Kind, ""), !strconcat("_", !subst("::", "_", Kind)), "")
                  # !if(Satfinite, "_satfinite", "")
                  # signature;
}

class MMA_SP_BLOCK_SCALE_NAME<string Kind, string SType, string ScaleVecSize,
                              WMMA_REGS A, WMMA_REGS B,
                              WMMA_REGS C, WMMA_REGS D> {
  string signature = MMA_SIGNATURE<A, B, C, D>.ret;
  string record_name = "int_nvvm_mma_sp_ordered_metadata_block_scale"
                  # "_" # A.geom
                  # "_row_col"
                  # "_" # Kind
                  # !subst(".", "_", ScaleVecSize)
                  # signature
                  # "_" # SType;
}

// Helper class that takes an intrinsic name and construct a record name.
// Additionally, sets `intr_name` to be non-empty if the default name assigned
// to this intrinsic will not match the name given.
class IntrinsicName<string name> {
  string record_name = !subst(".", "_",
                  !subst("llvm.", "int_", name));
  // Use explicit intrinsic name if it has an _ in it, else rely on LLVM
  // assigned default name.
````
- **L545 EN**: Continues the surrounding expression or declaration: `WMMA_REGS C, WMMA_REGS D> {`.
  **L545 CN**: 继续构造周围的表达式或声明：`WMMA_REGS C, WMMA_REGS D> {`。
- **L546 EN**: Initializes variable `signature` from the right-hand expression.
  **L546 CN**: 使用右侧表达式初始化变量 `signature`。
- **L547 EN**: Continues the surrounding expression or declaration: `string record_name = "int_nvvm_mma"`.
  **L547 CN**: 继续构造周围的表达式或声明：`string record_name = "int_nvvm_mma"`。
- **L548 EN**: Continues logic associated with callable symbol `subst`.
  **L548 CN**: 继续与可调用符号 `subst` 相关的逻辑。
- **L549 EN**: Continues the surrounding expression or declaration: `# "_" # A.geom`.
  **L549 CN**: 继续构造周围的表达式或声明：`# "_" # A.geom`。
- **L550 EN**: Continues the surrounding expression or declaration: `# "_row_col"`.
  **L550 CN**: 继续构造周围的表达式或声明：`# "_row_col"`。
- **L551 EN**: Continues the surrounding expression or declaration: `# !if(!ne(Kind, ""), !strconcat("_", !subst("::", "_", Kind)), "")`.
  **L551 CN**: 继续构造周围的表达式或声明：`# !if(!ne(Kind, ""), !strconcat("_", !subst("::", "_", Kind)), "")`。
- **L552 EN**: Continues the surrounding expression or declaration: `# !if(Satfinite, "_satfinite", "")`.
  **L552 CN**: 继续构造周围的表达式或声明：`# !if(Satfinite, "_satfinite", "")`。
- **L553 EN**: Executes a standalone statement or declaration: `# signature;`.
  **L553 CN**: 执行一条独立语句或声明：`# signature;`。
- **L554 EN**: Closes the current lexical scope or compound statement.
  **L554 CN**: 结束当前词法作用域或复合语句块。
- **L555 EN**: Blank line separating nearby declarations or logic blocks.
  **L555 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L556 EN**: Declares class `MMA_SP_BLOCK_SCALE_NAME<string`.
  **L556 CN**: 声明 class `MMA_SP_BLOCK_SCALE_NAME<string`。
- **L557 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WMMA_REGS A, WMMA_REGS B,`.
  **L557 CN**: 继续一个多行参数列表、初始化器或聚合项：`WMMA_REGS A, WMMA_REGS B,`。
- **L558 EN**: Continues the surrounding expression or declaration: `WMMA_REGS C, WMMA_REGS D> {`.
  **L558 CN**: 继续构造周围的表达式或声明：`WMMA_REGS C, WMMA_REGS D> {`。
- **L559 EN**: Initializes variable `signature` from the right-hand expression.
  **L559 CN**: 使用右侧表达式初始化变量 `signature`。
- **L560 EN**: Continues the surrounding expression or declaration: `string record_name = "int_nvvm_mma_sp_ordered_metadata_block_scale"`.
  **L560 CN**: 继续构造周围的表达式或声明：`string record_name = "int_nvvm_mma_sp_ordered_metadata_block_scale"`。
- **L561 EN**: Continues the surrounding expression or declaration: `# "_" # A.geom`.
  **L561 CN**: 继续构造周围的表达式或声明：`# "_" # A.geom`。
- **L562 EN**: Continues the surrounding expression or declaration: `# "_row_col"`.
  **L562 CN**: 继续构造周围的表达式或声明：`# "_row_col"`。
- **L563 EN**: Continues the surrounding expression or declaration: `# "_" # Kind`.
  **L563 CN**: 继续构造周围的表达式或声明：`# "_" # Kind`。
- **L564 EN**: Continues logic associated with callable symbol `subst`.
  **L564 CN**: 继续与可调用符号 `subst` 相关的逻辑。
- **L565 EN**: Continues the surrounding expression or declaration: `# signature`.
  **L565 CN**: 继续构造周围的表达式或声明：`# signature`。
- **L566 EN**: Executes a standalone statement or declaration: `# "_" # SType;`.
  **L566 CN**: 执行一条独立语句或声明：`# "_" # SType;`。
- **L567 EN**: Closes the current lexical scope or compound statement.
  **L567 CN**: 结束当前词法作用域或复合语句块。
- **L568 EN**: Blank line separating nearby declarations or logic blocks.
  **L568 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L569 EN**: Comment explains nearby logic, invariants, or intent: `Helper class that takes an intrinsic name and construct a record name.`.
  **L569 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Helper class that takes an intrinsic name and construct a record name.`。
- **L570 EN**: Comment explains nearby logic, invariants, or intent: `Additionally, sets `intr_name` to be non-empty if the default name assigned`.
  **L570 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Additionally, sets `intr_name` to be non-empty if the default name assigned`。
- **L571 EN**: Comment explains nearby logic, invariants, or intent: `to this intrinsic will not match the name given.`.
  **L571 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`to this intrinsic will not match the name given.`。
- **L572 EN**: Declares class `IntrinsicName<string`.
  **L572 CN**: 声明 class `IntrinsicName<string`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `string record_name = !subst(".", "_",`.
  **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`string record_name = !subst(".", "_",`。
- **L574 EN**: Executes a call or declaration centered on `!subst`.
  **L574 CN**: 执行以 `!subst` 为核心的调用或声明。
- **L575 EN**: Comment explains nearby logic, invariants, or intent: `Use explicit intrinsic name if it has an _ in it, else rely on LLVM`.
  **L575 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Use explicit intrinsic name if it has an _ in it, else rely on LLVM`。
- **L576 EN**: Comment explains nearby logic, invariants, or intent: `assigned default name.`.
  **L576 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`assigned default name.`。

### Lines 577-608

````tablegen
  string intr_name = !if(!ne(!find(name, "_"), -1), name, "");
}

class LDMATRIX_NAME<WMMA_REGS Frag, int Trans> {
  defvar name = "llvm.nvvm.ldmatrix.sync.aligned"
                # "." # Frag.geom
                # "." # Frag.frag
                # !if(Trans, ".trans", "")
                # "." # Frag.ptx_elt_type
                ;
  string intr_name = IntrinsicName<name>.intr_name;
  string record_name = IntrinsicName<name>.record_name;
}

class STMATRIX_NAME<WMMA_REGS Frag, int Trans> {
  defvar name = "llvm.nvvm.stmatrix.sync.aligned"
                # "." # Frag.geom
                # "." # Frag.frag
                # !if(Trans, ".trans", "")
                # "." # Frag.ptx_elt_type
                ;
  string intr_name = IntrinsicName<name>.intr_name;
  string record_name = IntrinsicName<name>.record_name;
}

// Generates list of 4-tuples of WMMA_REGS representing a valid MMA op.
//   Geom: list of supported geometries.
//   TypeN: PTX type of the corresponding fragment's element.
//   TypeB and TypeD may be empty if it must match that of TypeA or TypeC.
class MMA_OPS<list<string> Geom, list<string> TypeA, list<string> TypeB,
            list<string> TypeC, list<string> TypeD, bit IsSparse = false> {
  list<list<WMMA_REGS>> ret =
````
- **L577 EN**: Initializes variable `intr_name` from the right-hand expression.
  **L577 CN**: 使用右侧表达式初始化变量 `intr_name`。
- **L578 EN**: Closes the current lexical scope or compound statement.
  **L578 CN**: 结束当前词法作用域或复合语句块。
- **L579 EN**: Blank line separating nearby declarations or logic blocks.
  **L579 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L580 EN**: Declares class `LDMATRIX_NAME<WMMA_REGS`.
  **L580 CN**: 声明 class `LDMATRIX_NAME<WMMA_REGS`。
- **L581 EN**: Continues the surrounding expression or declaration: `defvar name = "llvm.nvvm.ldmatrix.sync.aligned"`.
  **L581 CN**: 继续构造周围的表达式或声明：`defvar name = "llvm.nvvm.ldmatrix.sync.aligned"`。
- **L582 EN**: Continues the surrounding expression or declaration: `# "." # Frag.geom`.
  **L582 CN**: 继续构造周围的表达式或声明：`# "." # Frag.geom`。
- **L583 EN**: Continues the surrounding expression or declaration: `# "." # Frag.frag`.
  **L583 CN**: 继续构造周围的表达式或声明：`# "." # Frag.frag`。
- **L584 EN**: Continues the surrounding expression or declaration: `# !if(Trans, ".trans", "")`.
  **L584 CN**: 继续构造周围的表达式或声明：`# !if(Trans, ".trans", "")`。
- **L585 EN**: Continues the surrounding expression or declaration: `# "." # Frag.ptx_elt_type`.
  **L585 CN**: 继续构造周围的表达式或声明：`# "." # Frag.ptx_elt_type`。
- **L586 EN**: Executes a standalone statement or declaration: `;`.
  **L586 CN**: 执行一条独立语句或声明：`;`。
- **L587 EN**: Initializes variable `intr_name` from the right-hand expression.
  **L587 CN**: 使用右侧表达式初始化变量 `intr_name`。
- **L588 EN**: Initializes variable `record_name` from the right-hand expression.
  **L588 CN**: 使用右侧表达式初始化变量 `record_name`。
- **L589 EN**: Closes the current lexical scope or compound statement.
  **L589 CN**: 结束当前词法作用域或复合语句块。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L591 EN**: Declares class `STMATRIX_NAME<WMMA_REGS`.
  **L591 CN**: 声明 class `STMATRIX_NAME<WMMA_REGS`。
- **L592 EN**: Continues the surrounding expression or declaration: `defvar name = "llvm.nvvm.stmatrix.sync.aligned"`.
  **L592 CN**: 继续构造周围的表达式或声明：`defvar name = "llvm.nvvm.stmatrix.sync.aligned"`。
- **L593 EN**: Continues the surrounding expression or declaration: `# "." # Frag.geom`.
  **L593 CN**: 继续构造周围的表达式或声明：`# "." # Frag.geom`。
- **L594 EN**: Continues the surrounding expression or declaration: `# "." # Frag.frag`.
  **L594 CN**: 继续构造周围的表达式或声明：`# "." # Frag.frag`。
- **L595 EN**: Continues the surrounding expression or declaration: `# !if(Trans, ".trans", "")`.
  **L595 CN**: 继续构造周围的表达式或声明：`# !if(Trans, ".trans", "")`。
- **L596 EN**: Continues the surrounding expression or declaration: `# "." # Frag.ptx_elt_type`.
  **L596 CN**: 继续构造周围的表达式或声明：`# "." # Frag.ptx_elt_type`。
- **L597 EN**: Executes a standalone statement or declaration: `;`.
  **L597 CN**: 执行一条独立语句或声明：`;`。
- **L598 EN**: Initializes variable `intr_name` from the right-hand expression.
  **L598 CN**: 使用右侧表达式初始化变量 `intr_name`。
- **L599 EN**: Initializes variable `record_name` from the right-hand expression.
  **L599 CN**: 使用右侧表达式初始化变量 `record_name`。
- **L600 EN**: Closes the current lexical scope or compound statement.
  **L600 CN**: 结束当前词法作用域或复合语句块。
- **L601 EN**: Blank line separating nearby declarations or logic blocks.
  **L601 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L602 EN**: Comment explains nearby logic, invariants, or intent: `Generates list of 4-tuples of WMMA_REGS representing a valid MMA op.`.
  **L602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generates list of 4-tuples of WMMA_REGS representing a valid MMA op.`。
- **L603 EN**: Comment explains nearby logic, invariants, or intent: `Geom: list of supported geometries.`.
  **L603 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Geom: list of supported geometries.`。
- **L604 EN**: Comment explains nearby logic, invariants, or intent: `TypeN: PTX type of the corresponding fragment's element.`.
  **L604 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TypeN: PTX type of the corresponding fragment's element.`。
- **L605 EN**: Comment explains nearby logic, invariants, or intent: `TypeB and TypeD may be empty if it must match that of TypeA or TypeC.`.
  **L605 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TypeB and TypeD may be empty if it must match that of TypeA or TypeC.`。
- **L606 EN**: Declares class `MMA_OPS<list<string>`.
  **L606 CN**: 声明 class `MMA_OPS<list<string>`。
- **L607 EN**: Continues the surrounding expression or declaration: `list<string> TypeC, list<string> TypeD, bit IsSparse = false> {`.
  **L607 CN**: 继续构造周围的表达式或声明：`list<string> TypeC, list<string> TypeD, bit IsSparse = false> {`。
- **L608 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> ret =`.
  **L608 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> ret =`。

### Lines 609-640

````tablegen
     !foldl([]<list<WMMA_REGS>>, Geom, t1, geom, !listconcat(t1,
     !foldl([]<list<WMMA_REGS>>, TypeA, t2, type_a, !listconcat(t2,
     !foldl([]<list<WMMA_REGS>>, !if(!size(TypeB), TypeB, [type_a]), t3, type_b, !listconcat(t3,
     !foldl([]<list<WMMA_REGS>>, TypeC, t4, type_c, !listconcat(t4,
     !foldl([]<list<WMMA_REGS>>, !if(!size(TypeD), TypeD, [type_c]), t5, type_d, !listconcat(t5,
            [[WMMA_REGS<geom, "a", type_a, IsSparse>,
              WMMA_REGS<geom, "b", type_b, IsSparse>,
              WMMA_REGS<geom, "c", type_c, IsSparse>,
              WMMA_REGS<geom, "d", type_d, IsSparse>]]))))))))));
   // Debugging aid for readable representation of the list above.
   list<list<string>> ops = !foreach(x, ret, [x[0].gft, x[1].gft, x[2].gft, x[3].gft]);
}


class MMA_LDST_OPS<list<string> Geom, list<string> Frags, list<string> Types> {
  list<WMMA_REGS> ret =
     !foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,
     !foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,
     !foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,
            [WMMA_REGS<geom, frag, type>]))))));
   // Debugging aid for readable representation of the list above.
   list<string> ops = !foreach(x, ret, x.gft);
}

class LDMATRIX_OPS<list<string> Geom, list<string> Frags, list<string> Types> {
  list<WMMA_REGS> ret =
     !foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,
     !foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,
     !foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,
            [WMMA_REGS<geom, frag, type>]))))));
   // Debugging aid for readable representation of the list above.
   list<string> ops = !foreach(x, ret, x.gft);
````
- **L609 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<list<WMMA_REGS>>, Geom, t1, geom, !listconcat(t1,`.
  **L609 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<list<WMMA_REGS>>, Geom, t1, geom, !listconcat(t1,`。
- **L610 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<list<WMMA_REGS>>, TypeA, t2, type_a, !listconcat(t2,`.
  **L610 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<list<WMMA_REGS>>, TypeA, t2, type_a, !listconcat(t2,`。
- **L611 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<list<WMMA_REGS>>, !if(!size(TypeB), TypeB, [type_a]), t3, type_b, !listconcat(t3,`.
  **L611 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<list<WMMA_REGS>>, !if(!size(TypeB), TypeB, [type_a]), t3, type_b, !listconcat(t3,`。
- **L612 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<list<WMMA_REGS>>, TypeC, t4, type_c, !listconcat(t4,`.
  **L612 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<list<WMMA_REGS>>, TypeC, t4, type_c, !listconcat(t4,`。
- **L613 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<list<WMMA_REGS>>, !if(!size(TypeD), TypeD, [type_c]), t5, type_d, !listconcat(t5,`.
  **L613 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<list<WMMA_REGS>>, !if(!size(TypeD), TypeD, [type_c]), t5, type_d, !listconcat(t5,`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[[WMMA_REGS<geom, "a", type_a, IsSparse>,`.
  **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`[[WMMA_REGS<geom, "a", type_a, IsSparse>,`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WMMA_REGS<geom, "b", type_b, IsSparse>,`.
  **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`WMMA_REGS<geom, "b", type_b, IsSparse>,`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WMMA_REGS<geom, "c", type_c, IsSparse>,`.
  **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`WMMA_REGS<geom, "c", type_c, IsSparse>,`。
- **L617 EN**: Executes a standalone statement or declaration: `WMMA_REGS<geom, "d", type_d, IsSparse>]]))))))))));`.
  **L617 CN**: 执行一条独立语句或声明：`WMMA_REGS<geom, "d", type_d, IsSparse>]]))))))))));`。
- **L618 EN**: Comment explains nearby logic, invariants, or intent: `Debugging aid for readable representation of the list above.`.
  **L618 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Debugging aid for readable representation of the list above.`。
- **L619 EN**: Initializes variable `ops` from the right-hand expression.
  **L619 CN**: 使用右侧表达式初始化变量 `ops`。
- **L620 EN**: Closes the current lexical scope or compound statement.
  **L620 CN**: 结束当前词法作用域或复合语句块。
- **L621 EN**: Blank line separating nearby declarations or logic blocks.
  **L621 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L623 EN**: Declares class `MMA_LDST_OPS<list<string>`.
  **L623 CN**: 声明 class `MMA_LDST_OPS<list<string>`。
- **L624 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ret =`.
  **L624 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ret =`。
- **L625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,`.
  **L625 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,`。
- **L626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,`.
  **L626 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,`。
- **L627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,`.
  **L627 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,`。
- **L628 EN**: Executes a standalone statement or declaration: `[WMMA_REGS<geom, frag, type>]))))));`.
  **L628 CN**: 执行一条独立语句或声明：`[WMMA_REGS<geom, frag, type>]))))));`。
- **L629 EN**: Comment explains nearby logic, invariants, or intent: `Debugging aid for readable representation of the list above.`.
  **L629 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Debugging aid for readable representation of the list above.`。
- **L630 EN**: Initializes variable `ops` from the right-hand expression.
  **L630 CN**: 使用右侧表达式初始化变量 `ops`。
- **L631 EN**: Closes the current lexical scope or compound statement.
  **L631 CN**: 结束当前词法作用域或复合语句块。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L633 EN**: Declares class `LDMATRIX_OPS<list<string>`.
  **L633 CN**: 声明 class `LDMATRIX_OPS<list<string>`。
- **L634 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ret =`.
  **L634 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ret =`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,`.
  **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,`.
  **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,`。
- **L638 EN**: Executes a standalone statement or declaration: `[WMMA_REGS<geom, frag, type>]))))));`.
  **L638 CN**: 执行一条独立语句或声明：`[WMMA_REGS<geom, frag, type>]))))));`。
- **L639 EN**: Comment explains nearby logic, invariants, or intent: `Debugging aid for readable representation of the list above.`.
  **L639 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Debugging aid for readable representation of the list above.`。
- **L640 EN**: Initializes variable `ops` from the right-hand expression.
  **L640 CN**: 使用右侧表达式初始化变量 `ops`。

### Lines 641-672

````tablegen
}

class STMATRIX_OPS<list<string> Geom, list<string> Frags, list<string> Types> {
  list<WMMA_REGS> ret =
     !foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,
     !foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,
     !foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,
            [WMMA_REGS<geom, frag, type>]))))));
   // Debugging aid for readable representation of the list above.
   list<string> ops = !foreach(x, ret, x.gft);
}

// Creates list of valid combinations of fragments. This is the main list that
// drives generation of corresponding intrinsics and instructions.
class NVVM_MMA_OPS {
  list<list<WMMA_REGS>> tf32_wmma_ops = MMA_OPS<
            ["m16n16k8"],
            ["tf32"], [], ["f32"], []>.ret;
  list<list<WMMA_REGS>> bf16_wmma_ops = MMA_OPS<
            ["m16n16k16", "m32n8k16", "m8n32k16"],
            ["bf16"], [], ["f32"], []>.ret;
  list<list<WMMA_REGS>> f64_wmma_ops = MMA_OPS<
            ["m8n8k4"],
            ["f64"], [], ["f64"], []>.ret;
  list<list<WMMA_REGS>> fp_wmma_ops = MMA_OPS<
            ["m16n16k16", "m32n8k16", "m8n32k16"],
            ["f16"], [], ["f16", "f32"], ["f16", "f32"]>.ret;
  list<list<WMMA_REGS>> int_wmma_ops = MMA_OPS<
            ["m16n16k16", "m32n8k16", "m8n32k16"],
            ["s8", "u8"], [], ["s32"], []>.ret;
  list<list<WMMA_REGS>> subint_wmma_ops = MMA_OPS<
            ["m8n8k32"],
````
- **L641 EN**: Closes the current lexical scope or compound statement.
  **L641 CN**: 结束当前词法作用域或复合语句块。
- **L642 EN**: Blank line separating nearby declarations or logic blocks.
  **L642 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L643 EN**: Declares class `STMATRIX_OPS<list<string>`.
  **L643 CN**: 声明 class `STMATRIX_OPS<list<string>`。
- **L644 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ret =`.
  **L644 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ret =`。
- **L645 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,`.
  **L645 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Geom, t1, geom, !listconcat(t1,`。
- **L646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,`.
  **L646 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Frags, t2, frag, !listconcat(t2,`。
- **L647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,`.
  **L647 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foldl([]<WMMA_REGS>, Types, t3, type, !listconcat(t3,`。
- **L648 EN**: Executes a standalone statement or declaration: `[WMMA_REGS<geom, frag, type>]))))));`.
  **L648 CN**: 执行一条独立语句或声明：`[WMMA_REGS<geom, frag, type>]))))));`。
- **L649 EN**: Comment explains nearby logic, invariants, or intent: `Debugging aid for readable representation of the list above.`.
  **L649 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Debugging aid for readable representation of the list above.`。
- **L650 EN**: Initializes variable `ops` from the right-hand expression.
  **L650 CN**: 使用右侧表达式初始化变量 `ops`。
- **L651 EN**: Closes the current lexical scope or compound statement.
  **L651 CN**: 结束当前词法作用域或复合语句块。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L653 EN**: Comment explains nearby logic, invariants, or intent: `Creates list of valid combinations of fragments. This is the main list that`.
  **L653 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Creates list of valid combinations of fragments. This is the main list that`。
- **L654 EN**: Comment explains nearby logic, invariants, or intent: `drives generation of corresponding intrinsics and instructions.`.
  **L654 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`drives generation of corresponding intrinsics and instructions.`。
- **L655 EN**: Declares class `NVVM_MMA_OPS`.
  **L655 CN**: 声明 class `NVVM_MMA_OPS`。
- **L656 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> tf32_wmma_ops = MMA_OPS<`.
  **L656 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> tf32_wmma_ops = MMA_OPS<`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k8"],`.
  **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k8"],`。
- **L658 EN**: Executes a standalone statement or declaration: `["tf32"], [], ["f32"], []>.ret;`.
  **L658 CN**: 执行一条独立语句或声明：`["tf32"], [], ["f32"], []>.ret;`。
- **L659 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> bf16_wmma_ops = MMA_OPS<`.
  **L659 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> bf16_wmma_ops = MMA_OPS<`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k16", "m32n8k16", "m8n32k16"],`.
  **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k16", "m32n8k16", "m8n32k16"],`。
- **L661 EN**: Executes a standalone statement or declaration: `["bf16"], [], ["f32"], []>.ret;`.
  **L661 CN**: 执行一条独立语句或声明：`["bf16"], [], ["f32"], []>.ret;`。
- **L662 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> f64_wmma_ops = MMA_OPS<`.
  **L662 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> f64_wmma_ops = MMA_OPS<`。
- **L663 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k4"],`.
  **L663 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k4"],`。
- **L664 EN**: Executes a standalone statement or declaration: `["f64"], [], ["f64"], []>.ret;`.
  **L664 CN**: 执行一条独立语句或声明：`["f64"], [], ["f64"], []>.ret;`。
- **L665 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> fp_wmma_ops = MMA_OPS<`.
  **L665 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> fp_wmma_ops = MMA_OPS<`。
- **L666 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k16", "m32n8k16", "m8n32k16"],`.
  **L666 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k16", "m32n8k16", "m8n32k16"],`。
- **L667 EN**: Executes a standalone statement or declaration: `["f16"], [], ["f16", "f32"], ["f16", "f32"]>.ret;`.
  **L667 CN**: 执行一条独立语句或声明：`["f16"], [], ["f16", "f32"], ["f16", "f32"]>.ret;`。
- **L668 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> int_wmma_ops = MMA_OPS<`.
  **L668 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> int_wmma_ops = MMA_OPS<`。
- **L669 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k16", "m32n8k16", "m8n32k16"],`.
  **L669 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k16", "m32n8k16", "m8n32k16"],`。
- **L670 EN**: Executes a standalone statement or declaration: `["s8", "u8"], [], ["s32"], []>.ret;`.
  **L670 CN**: 执行一条独立语句或声明：`["s8", "u8"], [], ["s32"], []>.ret;`。
- **L671 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> subint_wmma_ops = MMA_OPS<`.
  **L671 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> subint_wmma_ops = MMA_OPS<`。
- **L672 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k32"],`.
  **L672 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k32"],`。

### Lines 673-704

````tablegen
            ["s4", "u4"], [], ["s32"], []>.ret;
  list<list<WMMA_REGS>> bit_wmma_ops = MMA_OPS<
            ["m8n8k128"],
            ["b1"], [], ["s32"], []>.ret;
  list<list<WMMA_REGS>> all_wmma_ops = !listconcat(
            tf32_wmma_ops, bf16_wmma_ops, f64_wmma_ops,
            fp_wmma_ops, int_wmma_ops, subint_wmma_ops, bit_wmma_ops);

  list<list<WMMA_REGS>> tf32_mma_ops = MMA_OPS<
            ["m16n8k4", "m16n8k8"],
            ["tf32"], [], ["f32"], []>.ret;
  list<list<WMMA_REGS>> bf16_mma_ops = MMA_OPS<
            ["m16n8k16", "m16n8k8"],
            ["bf16"], [], ["f32"], []>.ret;
  list<list<WMMA_REGS>> f64_mma_ops = MMA_OPS<
            ["m8n8k4", "m16n8k4", "m16n8k8", "m16n8k16"],
            ["f64"], [], ["f64"], []>.ret;
  list<list<WMMA_REGS>> fp_mma_ops = MMA_OPS<
            ["m8n8k4", "m16n8k8", "m16n8k16"],
            ["f16"], [], ["f16", "f32"], ["f16", "f32"]>.ret;
  list<list<WMMA_REGS>> int_mma_ops = MMA_OPS<
            ["m8n8k16", "m16n8k16", "m16n8k32"],
            ["s8", "u8"], ["s8", "u8"], ["s32"], []>.ret;
  // m16n8k32 fp8 variants are intersected with f8f6f4 variants
  // and processed there
  list<list<WMMA_REGS>> fp8_mma_ops = MMA_OPS<
            ["m16n8k16"],
            ["e4m3", "e5m2"], ["e4m3", "e5m2"],
            ["f16", "f32"], ["f16", "f32"]>.ret;
  // it also contains e4m3/e5m2 from fp8 variants
  list<list<WMMA_REGS>> f8f6f4_mma_ops = MMA_OPS<
            ["m16n8k32"],
````
- **L673 EN**: Executes a standalone statement or declaration: `["s4", "u4"], [], ["s32"], []>.ret;`.
  **L673 CN**: 执行一条独立语句或声明：`["s4", "u4"], [], ["s32"], []>.ret;`。
- **L674 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> bit_wmma_ops = MMA_OPS<`.
  **L674 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> bit_wmma_ops = MMA_OPS<`。
- **L675 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k128"],`.
  **L675 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k128"],`。
- **L676 EN**: Executes a standalone statement or declaration: `["b1"], [], ["s32"], []>.ret;`.
  **L676 CN**: 执行一条独立语句或声明：`["b1"], [], ["s32"], []>.ret;`。
- **L677 EN**: Continues logic associated with callable symbol `listconcat`.
  **L677 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L678 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tf32_wmma_ops, bf16_wmma_ops, f64_wmma_ops,`.
  **L678 CN**: 继续一个多行参数列表、初始化器或聚合项：`tf32_wmma_ops, bf16_wmma_ops, f64_wmma_ops,`。
- **L679 EN**: Executes a standalone statement or declaration: `fp_wmma_ops, int_wmma_ops, subint_wmma_ops, bit_wmma_ops);`.
  **L679 CN**: 执行一条独立语句或声明：`fp_wmma_ops, int_wmma_ops, subint_wmma_ops, bit_wmma_ops);`。
- **L680 EN**: Blank line separating nearby declarations or logic blocks.
  **L680 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L681 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> tf32_mma_ops = MMA_OPS<`.
  **L681 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> tf32_mma_ops = MMA_OPS<`。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k4", "m16n8k8"],`.
  **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k4", "m16n8k8"],`。
- **L683 EN**: Executes a standalone statement or declaration: `["tf32"], [], ["f32"], []>.ret;`.
  **L683 CN**: 执行一条独立语句或声明：`["tf32"], [], ["f32"], []>.ret;`。
- **L684 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> bf16_mma_ops = MMA_OPS<`.
  **L684 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> bf16_mma_ops = MMA_OPS<`。
- **L685 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k16", "m16n8k8"],`.
  **L685 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k16", "m16n8k8"],`。
- **L686 EN**: Executes a standalone statement or declaration: `["bf16"], [], ["f32"], []>.ret;`.
  **L686 CN**: 执行一条独立语句或声明：`["bf16"], [], ["f32"], []>.ret;`。
- **L687 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> f64_mma_ops = MMA_OPS<`.
  **L687 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> f64_mma_ops = MMA_OPS<`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k4", "m16n8k4", "m16n8k8", "m16n8k16"],`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k4", "m16n8k4", "m16n8k8", "m16n8k16"],`。
- **L689 EN**: Executes a standalone statement or declaration: `["f64"], [], ["f64"], []>.ret;`.
  **L689 CN**: 执行一条独立语句或声明：`["f64"], [], ["f64"], []>.ret;`。
- **L690 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> fp_mma_ops = MMA_OPS<`.
  **L690 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> fp_mma_ops = MMA_OPS<`。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k4", "m16n8k8", "m16n8k16"],`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k4", "m16n8k8", "m16n8k16"],`。
- **L692 EN**: Executes a standalone statement or declaration: `["f16"], [], ["f16", "f32"], ["f16", "f32"]>.ret;`.
  **L692 CN**: 执行一条独立语句或声明：`["f16"], [], ["f16", "f32"], ["f16", "f32"]>.ret;`。
- **L693 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> int_mma_ops = MMA_OPS<`.
  **L693 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> int_mma_ops = MMA_OPS<`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k16", "m16n8k16", "m16n8k32"],`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k16", "m16n8k16", "m16n8k32"],`。
- **L695 EN**: Executes a standalone statement or declaration: `["s8", "u8"], ["s8", "u8"], ["s32"], []>.ret;`.
  **L695 CN**: 执行一条独立语句或声明：`["s8", "u8"], ["s8", "u8"], ["s32"], []>.ret;`。
- **L696 EN**: Comment explains nearby logic, invariants, or intent: `m16n8k32 fp8 variants are intersected with f8f6f4 variants`.
  **L696 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`m16n8k32 fp8 variants are intersected with f8f6f4 variants`。
- **L697 EN**: Comment explains nearby logic, invariants, or intent: `and processed there`.
  **L697 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`and processed there`。
- **L698 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> fp8_mma_ops = MMA_OPS<`.
  **L698 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> fp8_mma_ops = MMA_OPS<`。
- **L699 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k16"],`.
  **L699 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k16"],`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["e4m3", "e5m2"], ["e4m3", "e5m2"],`.
  **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`["e4m3", "e5m2"], ["e4m3", "e5m2"],`。
- **L701 EN**: Executes a standalone statement or declaration: `["f16", "f32"], ["f16", "f32"]>.ret;`.
  **L701 CN**: 执行一条独立语句或声明：`["f16", "f32"], ["f16", "f32"]>.ret;`。
- **L702 EN**: Comment explains nearby logic, invariants, or intent: `it also contains e4m3/e5m2 from fp8 variants`.
  **L702 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`it also contains e4m3/e5m2 from fp8 variants`。
- **L703 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> f8f6f4_mma_ops = MMA_OPS<`.
  **L703 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> f8f6f4_mma_ops = MMA_OPS<`。
- **L704 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k32"],`.
  **L704 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k32"],`。

### Lines 705-736

````tablegen
            ["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],
            ["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],
            ["f16", "f32"], ["f16", "f32"]>.ret;
  list<list<WMMA_REGS>> subint_mma_ops = MMA_OPS<
            ["m8n8k32", "m16n8k32", "m16n8k64"],
            ["s4", "u4"], ["s4", "u4"], ["s32"], []>.ret;
  list<list<WMMA_REGS>> bit_mma_ops = MMA_OPS<
            ["m8n8k128", "m16n8k128", "m16n8k256"],
            ["b1"], [], ["s32"], []>.ret;
  list<list<WMMA_REGS>> all_mma_ops = !listconcat(
            tf32_mma_ops, bf16_mma_ops, f64_mma_ops,
            fp_mma_ops, fp8_mma_ops, f8f6f4_mma_ops,
            int_mma_ops, subint_mma_ops, bit_mma_ops);

  list<list<WMMA_REGS>> mxf4_mma_ops = MMA_OPS<
    ["m16n8k64"], ["e2m1"], ["e2m1"], ["f32"], ["f32"]
  >.ret;

  list<list<WMMA_REGS>> mxf8f6f4_mma_ops = MMA_OPS<
    ["m16n8k32"], ["e2m1", "e2m3", "e3m2", "e5m2", "e4m3"],
                  ["e2m1", "e2m3", "e3m2", "e5m2", "e4m3"], ["f32"], ["f32"]
  >.ret;

  list<list<WMMA_REGS>> all_mma_block_scale_ops = !listconcat(
            mxf4_mma_ops, mxf8f6f4_mma_ops);

  list<list<WMMA_REGS>> bf16_mma_sp_ops = MMA_OPS<
            ["m16n8k16", "m16n8k32"],
            ["bf16"], [], ["f32"], [], true>.ret;
  list<list<WMMA_REGS>> tf32_mma_sp_ops = MMA_OPS<
            ["m16n8k8", "m16n8k16"],
            ["tf32"], [], ["f32"], [], true>.ret;
````
- **L705 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`.
  **L705 CN**: 继续一个多行参数列表、初始化器或聚合项：`["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`。
- **L706 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`.
  **L706 CN**: 继续一个多行参数列表、初始化器或聚合项：`["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`。
- **L707 EN**: Executes a standalone statement or declaration: `["f16", "f32"], ["f16", "f32"]>.ret;`.
  **L707 CN**: 执行一条独立语句或声明：`["f16", "f32"], ["f16", "f32"]>.ret;`。
- **L708 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> subint_mma_ops = MMA_OPS<`.
  **L708 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> subint_mma_ops = MMA_OPS<`。
- **L709 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k32", "m16n8k32", "m16n8k64"],`.
  **L709 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k32", "m16n8k32", "m16n8k64"],`。
- **L710 EN**: Executes a standalone statement or declaration: `["s4", "u4"], ["s4", "u4"], ["s32"], []>.ret;`.
  **L710 CN**: 执行一条独立语句或声明：`["s4", "u4"], ["s4", "u4"], ["s32"], []>.ret;`。
- **L711 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> bit_mma_ops = MMA_OPS<`.
  **L711 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> bit_mma_ops = MMA_OPS<`。
- **L712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k128", "m16n8k128", "m16n8k256"],`.
  **L712 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k128", "m16n8k128", "m16n8k256"],`。
- **L713 EN**: Executes a standalone statement or declaration: `["b1"], [], ["s32"], []>.ret;`.
  **L713 CN**: 执行一条独立语句或声明：`["b1"], [], ["s32"], []>.ret;`。
- **L714 EN**: Continues logic associated with callable symbol `listconcat`.
  **L714 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `tf32_mma_ops, bf16_mma_ops, f64_mma_ops,`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`tf32_mma_ops, bf16_mma_ops, f64_mma_ops,`。
- **L716 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `fp_mma_ops, fp8_mma_ops, f8f6f4_mma_ops,`.
  **L716 CN**: 继续一个多行参数列表、初始化器或聚合项：`fp_mma_ops, fp8_mma_ops, f8f6f4_mma_ops,`。
- **L717 EN**: Executes a standalone statement or declaration: `int_mma_ops, subint_mma_ops, bit_mma_ops);`.
  **L717 CN**: 执行一条独立语句或声明：`int_mma_ops, subint_mma_ops, bit_mma_ops);`。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L719 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> mxf4_mma_ops = MMA_OPS<`.
  **L719 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> mxf4_mma_ops = MMA_OPS<`。
- **L720 EN**: Continues the surrounding expression or declaration: `["m16n8k64"], ["e2m1"], ["e2m1"], ["f32"], ["f32"]`.
  **L720 CN**: 继续构造周围的表达式或声明：`["m16n8k64"], ["e2m1"], ["e2m1"], ["f32"], ["f32"]`。
- **L721 EN**: Executes a standalone statement or declaration: `>.ret;`.
  **L721 CN**: 执行一条独立语句或声明：`>.ret;`。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L723 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> mxf8f6f4_mma_ops = MMA_OPS<`.
  **L723 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> mxf8f6f4_mma_ops = MMA_OPS<`。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k32"], ["e2m1", "e2m3", "e3m2", "e5m2", "e4m3"],`.
  **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k32"], ["e2m1", "e2m3", "e3m2", "e5m2", "e4m3"],`。
- **L725 EN**: Continues the surrounding expression or declaration: `["e2m1", "e2m3", "e3m2", "e5m2", "e4m3"], ["f32"], ["f32"]`.
  **L725 CN**: 继续构造周围的表达式或声明：`["e2m1", "e2m3", "e3m2", "e5m2", "e4m3"], ["f32"], ["f32"]`。
- **L726 EN**: Executes a standalone statement or declaration: `>.ret;`.
  **L726 CN**: 执行一条独立语句或声明：`>.ret;`。
- **L727 EN**: Blank line separating nearby declarations or logic blocks.
  **L727 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L728 EN**: Continues logic associated with callable symbol `listconcat`.
  **L728 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L729 EN**: Executes a standalone statement or declaration: `mxf4_mma_ops, mxf8f6f4_mma_ops);`.
  **L729 CN**: 执行一条独立语句或声明：`mxf4_mma_ops, mxf8f6f4_mma_ops);`。
- **L730 EN**: Blank line separating nearby declarations or logic blocks.
  **L730 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L731 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> bf16_mma_sp_ops = MMA_OPS<`.
  **L731 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> bf16_mma_sp_ops = MMA_OPS<`。
- **L732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k16", "m16n8k32"],`.
  **L732 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k16", "m16n8k32"],`。
- **L733 EN**: Executes a standalone statement or declaration: `["bf16"], [], ["f32"], [], true>.ret;`.
  **L733 CN**: 执行一条独立语句或声明：`["bf16"], [], ["f32"], [], true>.ret;`。
- **L734 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> tf32_mma_sp_ops = MMA_OPS<`.
  **L734 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> tf32_mma_sp_ops = MMA_OPS<`。
- **L735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k8", "m16n8k16"],`.
  **L735 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k8", "m16n8k16"],`。
- **L736 EN**: Executes a standalone statement or declaration: `["tf32"], [], ["f32"], [], true>.ret;`.
  **L736 CN**: 执行一条独立语句或声明：`["tf32"], [], ["f32"], [], true>.ret;`。

### Lines 737-768

````tablegen
  list<list<WMMA_REGS>> fp_mma_sp_ops = MMA_OPS<
            ["m16n8k16", "m16n8k32"],
            ["f16"], [], ["f16", "f32"], ["f16", "f32"], true>.ret;
  list<list<WMMA_REGS>> fp8_mma_sp_ops = MMA_OPS<
            ["m16n8k64"],
            ["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],
            ["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],
            ["f16", "f32"], ["f16", "f32"], true>.ret;
  list<list<WMMA_REGS>> subint_mma_sp_ops = MMA_OPS<
            ["m16n8k64", "m16n8k128"],
            ["s4", "u4"], ["s4", "u4"], ["s32"], [], true>.ret;
  list<list<WMMA_REGS>> int_mma_sp_ops = MMA_OPS<
            ["m16n8k32", "m16n8k64"],
            ["s8", "u8"], ["s8", "u8"], ["s32"], [], true>.ret;
  list<list<WMMA_REGS>> all_mma_sp_ops = !listconcat(
            bf16_mma_sp_ops, tf32_mma_sp_ops, fp_mma_sp_ops, fp8_mma_sp_ops,
            subint_mma_sp_ops, int_mma_sp_ops);

  // combines available geoms and types for mxf4 and mxf4nvf4 kinds
  list<list<WMMA_REGS>> mxf4xx_mma_sp_ops = MMA_OPS<
            ["m16n8k128"],
            ["e2m1"], ["e2m1"], ["f32"], [], true>.ret;
  list<list<WMMA_REGS>> mxf8f6f4_mma_sp_ops = MMA_OPS<
            ["m16n8k64"],
            ["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],
            ["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],
            ["f32"], [], true>.ret;
  list<list<WMMA_REGS>> all_mma_sp_block_scale_ops = !listconcat(
            mxf4xx_mma_sp_ops, mxf8f6f4_mma_sp_ops);

  list<WMMA_REGS> ldst_ab_ops = MMA_LDST_OPS<
            ["m16n16k16", "m32n8k16", "m8n32k16"],
````
- **L737 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> fp_mma_sp_ops = MMA_OPS<`.
  **L737 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> fp_mma_sp_ops = MMA_OPS<`。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k16", "m16n8k32"],`.
  **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k16", "m16n8k32"],`。
- **L739 EN**: Executes a standalone statement or declaration: `["f16"], [], ["f16", "f32"], ["f16", "f32"], true>.ret;`.
  **L739 CN**: 执行一条独立语句或声明：`["f16"], [], ["f16", "f32"], ["f16", "f32"], true>.ret;`。
- **L740 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> fp8_mma_sp_ops = MMA_OPS<`.
  **L740 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> fp8_mma_sp_ops = MMA_OPS<`。
- **L741 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k64"],`.
  **L741 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k64"],`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`.
  **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`。
- **L744 EN**: Executes a standalone statement or declaration: `["f16", "f32"], ["f16", "f32"], true>.ret;`.
  **L744 CN**: 执行一条独立语句或声明：`["f16", "f32"], ["f16", "f32"], true>.ret;`。
- **L745 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> subint_mma_sp_ops = MMA_OPS<`.
  **L745 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> subint_mma_sp_ops = MMA_OPS<`。
- **L746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k64", "m16n8k128"],`.
  **L746 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k64", "m16n8k128"],`。
- **L747 EN**: Executes a standalone statement or declaration: `["s4", "u4"], ["s4", "u4"], ["s32"], [], true>.ret;`.
  **L747 CN**: 执行一条独立语句或声明：`["s4", "u4"], ["s4", "u4"], ["s32"], [], true>.ret;`。
- **L748 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> int_mma_sp_ops = MMA_OPS<`.
  **L748 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> int_mma_sp_ops = MMA_OPS<`。
- **L749 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k32", "m16n8k64"],`.
  **L749 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k32", "m16n8k64"],`。
- **L750 EN**: Executes a standalone statement or declaration: `["s8", "u8"], ["s8", "u8"], ["s32"], [], true>.ret;`.
  **L750 CN**: 执行一条独立语句或声明：`["s8", "u8"], ["s8", "u8"], ["s32"], [], true>.ret;`。
- **L751 EN**: Continues logic associated with callable symbol `listconcat`.
  **L751 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L752 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bf16_mma_sp_ops, tf32_mma_sp_ops, fp_mma_sp_ops, fp8_mma_sp_ops,`.
  **L752 CN**: 继续一个多行参数列表、初始化器或聚合项：`bf16_mma_sp_ops, tf32_mma_sp_ops, fp_mma_sp_ops, fp8_mma_sp_ops,`。
- **L753 EN**: Executes a standalone statement or declaration: `subint_mma_sp_ops, int_mma_sp_ops);`.
  **L753 CN**: 执行一条独立语句或声明：`subint_mma_sp_ops, int_mma_sp_ops);`。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L755 EN**: Comment explains nearby logic, invariants, or intent: `combines available geoms and types for mxf4 and mxf4nvf4 kinds`.
  **L755 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`combines available geoms and types for mxf4 and mxf4nvf4 kinds`。
- **L756 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> mxf4xx_mma_sp_ops = MMA_OPS<`.
  **L756 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> mxf4xx_mma_sp_ops = MMA_OPS<`。
- **L757 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k128"],`.
  **L757 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k128"],`。
- **L758 EN**: Executes a standalone statement or declaration: `["e2m1"], ["e2m1"], ["f32"], [], true>.ret;`.
  **L758 CN**: 执行一条独立语句或声明：`["e2m1"], ["e2m1"], ["f32"], [], true>.ret;`。
- **L759 EN**: Continues the surrounding expression or declaration: `list<list<WMMA_REGS>> mxf8f6f4_mma_sp_ops = MMA_OPS<`.
  **L759 CN**: 继续构造周围的表达式或声明：`list<list<WMMA_REGS>> mxf8f6f4_mma_sp_ops = MMA_OPS<`。
- **L760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n8k64"],`.
  **L760 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n8k64"],`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`。
- **L762 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`.
  **L762 CN**: 继续一个多行参数列表、初始化器或聚合项：`["e4m3", "e5m2", "e3m2", "e2m3", "e2m1"],`。
- **L763 EN**: Executes a standalone statement or declaration: `["f32"], [], true>.ret;`.
  **L763 CN**: 执行一条独立语句或声明：`["f32"], [], true>.ret;`。
- **L764 EN**: Continues logic associated with callable symbol `listconcat`.
  **L764 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L765 EN**: Executes a standalone statement or declaration: `mxf4xx_mma_sp_ops, mxf8f6f4_mma_sp_ops);`.
  **L765 CN**: 执行一条独立语句或声明：`mxf4xx_mma_sp_ops, mxf8f6f4_mma_sp_ops);`。
- **L766 EN**: Blank line separating nearby declarations or logic blocks.
  **L766 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L767 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_ab_ops = MMA_LDST_OPS<`.
  **L767 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_ab_ops = MMA_LDST_OPS<`。
- **L768 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k16", "m32n8k16", "m8n32k16"],`.
  **L768 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k16", "m32n8k16", "m8n32k16"],`。

### Lines 769-800

````tablegen
            ["a", "b"], ["f16", "u8", "s8", "bf16"]>.ret;
  list<WMMA_REGS> ldst_cd_ops = MMA_LDST_OPS<
            ["m16n16k16", "m32n8k16", "m8n32k16"],
            ["c", "d"], ["f16", "f32", "s32"]>.ret;
  list<WMMA_REGS> ldst_tf32_ab_ops = MMA_LDST_OPS<
            ["m16n16k8"],
            ["a", "b"], ["tf32"]>.ret;
  list<WMMA_REGS> ldst_tf32_cd_ops = MMA_LDST_OPS<
            ["m16n16k8"],
            ["c", "d"], ["f32"]>.ret;
  list<WMMA_REGS> ldst_f64_abcd_ops = MMA_LDST_OPS<
            ["m8n8k4"],
            ["a", "b", "c", "d"], ["f64"]>.ret;
  list<WMMA_REGS> ldst_subint_ab_ops = MMA_LDST_OPS<
            ["m8n8k32"], ["a", "b"], ["s4","u4"]>.ret;
  list<WMMA_REGS> ldst_bit_ab_ops = MMA_LDST_OPS<
            ["m8n8k128"], ["a", "b"], ["b1"]>.ret;
  list<WMMA_REGS> ldst_subint_cd_ops = MMA_LDST_OPS<
            ["m8n8k32", "m8n8k128"],  ["c", "d"], ["s32"]>.ret;
  list<WMMA_REGS> all_ldst_ops = !listconcat(ldst_ab_ops, ldst_cd_ops,
                                             ldst_tf32_ab_ops,
                                             ldst_tf32_cd_ops,
                                             ldst_f64_abcd_ops,
                                             ldst_subint_ab_ops,
                                             ldst_bit_ab_ops,
                                             ldst_subint_cd_ops);
  // Separate A/B/C fragments (loads) from D (stores).
  list<WMMA_REGS> all_ld_ops = !filter(op, all_ldst_ops, !ne(op.frag, "d"));
  list<WMMA_REGS> all_st_ops = !filter(op, all_ldst_ops, !eq(op.frag, "d"));

  list<WMMA_REGS> ldmatrix_b16_ops = LDMATRIX_OPS<
    ["m8n8"], ["x1", "x2", "x4"], ["b16"]>.ret;
````
- **L769 EN**: Executes a standalone statement or declaration: `["a", "b"], ["f16", "u8", "s8", "bf16"]>.ret;`.
  **L769 CN**: 执行一条独立语句或声明：`["a", "b"], ["f16", "u8", "s8", "bf16"]>.ret;`。
- **L770 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_cd_ops = MMA_LDST_OPS<`.
  **L770 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_cd_ops = MMA_LDST_OPS<`。
- **L771 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k16", "m32n8k16", "m8n32k16"],`.
  **L771 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k16", "m32n8k16", "m8n32k16"],`。
- **L772 EN**: Executes a standalone statement or declaration: `["c", "d"], ["f16", "f32", "s32"]>.ret;`.
  **L772 CN**: 执行一条独立语句或声明：`["c", "d"], ["f16", "f32", "s32"]>.ret;`。
- **L773 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_tf32_ab_ops = MMA_LDST_OPS<`.
  **L773 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_tf32_ab_ops = MMA_LDST_OPS<`。
- **L774 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k8"],`.
  **L774 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k8"],`。
- **L775 EN**: Executes a standalone statement or declaration: `["a", "b"], ["tf32"]>.ret;`.
  **L775 CN**: 执行一条独立语句或声明：`["a", "b"], ["tf32"]>.ret;`。
- **L776 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_tf32_cd_ops = MMA_LDST_OPS<`.
  **L776 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_tf32_cd_ops = MMA_LDST_OPS<`。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m16n16k8"],`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m16n16k8"],`。
- **L778 EN**: Executes a standalone statement or declaration: `["c", "d"], ["f32"]>.ret;`.
  **L778 CN**: 执行一条独立语句或声明：`["c", "d"], ["f32"]>.ret;`。
- **L779 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_f64_abcd_ops = MMA_LDST_OPS<`.
  **L779 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_f64_abcd_ops = MMA_LDST_OPS<`。
- **L780 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `["m8n8k4"],`.
  **L780 CN**: 继续一个多行参数列表、初始化器或聚合项：`["m8n8k4"],`。
- **L781 EN**: Executes a standalone statement or declaration: `["a", "b", "c", "d"], ["f64"]>.ret;`.
  **L781 CN**: 执行一条独立语句或声明：`["a", "b", "c", "d"], ["f64"]>.ret;`。
- **L782 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_subint_ab_ops = MMA_LDST_OPS<`.
  **L782 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_subint_ab_ops = MMA_LDST_OPS<`。
- **L783 EN**: Executes a standalone statement or declaration: `["m8n8k32"], ["a", "b"], ["s4","u4"]>.ret;`.
  **L783 CN**: 执行一条独立语句或声明：`["m8n8k32"], ["a", "b"], ["s4","u4"]>.ret;`。
- **L784 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_bit_ab_ops = MMA_LDST_OPS<`.
  **L784 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_bit_ab_ops = MMA_LDST_OPS<`。
- **L785 EN**: Executes a standalone statement or declaration: `["m8n8k128"], ["a", "b"], ["b1"]>.ret;`.
  **L785 CN**: 执行一条独立语句或声明：`["m8n8k128"], ["a", "b"], ["b1"]>.ret;`。
- **L786 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldst_subint_cd_ops = MMA_LDST_OPS<`.
  **L786 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldst_subint_cd_ops = MMA_LDST_OPS<`。
- **L787 EN**: Executes a standalone statement or declaration: `["m8n8k32", "m8n8k128"],  ["c", "d"], ["s32"]>.ret;`.
  **L787 CN**: 执行一条独立语句或声明：`["m8n8k32", "m8n8k128"],  ["c", "d"], ["s32"]>.ret;`。
- **L788 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<WMMA_REGS> all_ldst_ops = !listconcat(ldst_ab_ops, ldst_cd_ops,`.
  **L788 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<WMMA_REGS> all_ldst_ops = !listconcat(ldst_ab_ops, ldst_cd_ops,`。
- **L789 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ldst_tf32_ab_ops,`.
  **L789 CN**: 继续一个多行参数列表、初始化器或聚合项：`ldst_tf32_ab_ops,`。
- **L790 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ldst_tf32_cd_ops,`.
  **L790 CN**: 继续一个多行参数列表、初始化器或聚合项：`ldst_tf32_cd_ops,`。
- **L791 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ldst_f64_abcd_ops,`.
  **L791 CN**: 继续一个多行参数列表、初始化器或聚合项：`ldst_f64_abcd_ops,`。
- **L792 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ldst_subint_ab_ops,`.
  **L792 CN**: 继续一个多行参数列表、初始化器或聚合项：`ldst_subint_ab_ops,`。
- **L793 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ldst_bit_ab_ops,`.
  **L793 CN**: 继续一个多行参数列表、初始化器或聚合项：`ldst_bit_ab_ops,`。
- **L794 EN**: Executes a standalone statement or declaration: `ldst_subint_cd_ops);`.
  **L794 CN**: 执行一条独立语句或声明：`ldst_subint_cd_ops);`。
- **L795 EN**: Comment explains nearby logic, invariants, or intent: `Separate A/B/C fragments (loads) from D (stores).`.
  **L795 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Separate A/B/C fragments (loads) from D (stores).`。
- **L796 EN**: Initializes variable `all_ld_ops` from the right-hand expression.
  **L796 CN**: 使用右侧表达式初始化变量 `all_ld_ops`。
- **L797 EN**: Initializes variable `all_st_ops` from the right-hand expression.
  **L797 CN**: 使用右侧表达式初始化变量 `all_st_ops`。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L799 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldmatrix_b16_ops = LDMATRIX_OPS<`.
  **L799 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldmatrix_b16_ops = LDMATRIX_OPS<`。
- **L800 EN**: Executes a standalone statement or declaration: `["m8n8"], ["x1", "x2", "x4"], ["b16"]>.ret;`.
  **L800 CN**: 执行一条独立语句或声明：`["m8n8"], ["x1", "x2", "x4"], ["b16"]>.ret;`。

### Lines 801-832

````tablegen

  list<WMMA_REGS> ldmatrix_geom_m16n16_ops = LDMATRIX_OPS<
    ["m16n16"], ["x1", "x2"], ["b8", "b8x16.b6x16_p32", "b8x16.b4x16_p64"]>.ret;

  list<WMMA_REGS> ldmatrix_geom_m8n16_ops = LDMATRIX_OPS<
    ["m8n16"], ["x1", "x2", "x4"], ["b8x16.b6x16_p32", "b8x16.b4x16_p64"]>.ret;

  list<WMMA_REGS> stmatrix_b16_ops = STMATRIX_OPS<
    ["m8n8"], ["x1", "x2", "x4"], ["b16"]>.ret;

  list<WMMA_REGS> stmatrix_b8_ops = STMATRIX_OPS<
    ["m16n8"], ["x1", "x2", "x4"], ["b8"]>.ret;

  list<WMMA_REGS> all_ldmatrix_ops = !listconcat(ldmatrix_b16_ops,
                                                 ldmatrix_geom_m16n16_ops,
                                                 ldmatrix_geom_m8n16_ops);

  list<WMMA_REGS> all_stmatrix_ops = !listconcat(stmatrix_b16_ops,
                                                 stmatrix_b8_ops);
}

def NVVM_MMA_OPS : NVVM_MMA_OPS;

// Returns true if this combination of fragment and layout for WMMA load/store
// ops is supported; false otherwise.
// E.g.
// if NVVM_WMMA_LDST_SUPPORTED<...>.ret then
//   def : FOO<>; // The record will only be defined for supported ops.
//
class NVVM_WMMA_LDST_SUPPORTED<WMMA_REGS frag, string layout> {
  string f = frag.frag;
  string t = frag.ptx_elt_type;
````
- **L801 EN**: Blank line separating nearby declarations or logic blocks.
  **L801 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L802 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldmatrix_geom_m16n16_ops = LDMATRIX_OPS<`.
  **L802 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldmatrix_geom_m16n16_ops = LDMATRIX_OPS<`。
- **L803 EN**: Executes a standalone statement or declaration: `["m16n16"], ["x1", "x2"], ["b8", "b8x16.b6x16_p32", "b8x16.b4x16_p64"]>.ret;`.
  **L803 CN**: 执行一条独立语句或声明：`["m16n16"], ["x1", "x2"], ["b8", "b8x16.b6x16_p32", "b8x16.b4x16_p64"]>.ret;`。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L805 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> ldmatrix_geom_m8n16_ops = LDMATRIX_OPS<`.
  **L805 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> ldmatrix_geom_m8n16_ops = LDMATRIX_OPS<`。
- **L806 EN**: Executes a standalone statement or declaration: `["m8n16"], ["x1", "x2", "x4"], ["b8x16.b6x16_p32", "b8x16.b4x16_p64"]>.ret;`.
  **L806 CN**: 执行一条独立语句或声明：`["m8n16"], ["x1", "x2", "x4"], ["b8x16.b6x16_p32", "b8x16.b4x16_p64"]>.ret;`。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L808 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> stmatrix_b16_ops = STMATRIX_OPS<`.
  **L808 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> stmatrix_b16_ops = STMATRIX_OPS<`。
- **L809 EN**: Executes a standalone statement or declaration: `["m8n8"], ["x1", "x2", "x4"], ["b16"]>.ret;`.
  **L809 CN**: 执行一条独立语句或声明：`["m8n8"], ["x1", "x2", "x4"], ["b16"]>.ret;`。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L811 EN**: Continues the surrounding expression or declaration: `list<WMMA_REGS> stmatrix_b8_ops = STMATRIX_OPS<`.
  **L811 CN**: 继续构造周围的表达式或声明：`list<WMMA_REGS> stmatrix_b8_ops = STMATRIX_OPS<`。
- **L812 EN**: Executes a standalone statement or declaration: `["m16n8"], ["x1", "x2", "x4"], ["b8"]>.ret;`.
  **L812 CN**: 执行一条独立语句或声明：`["m16n8"], ["x1", "x2", "x4"], ["b8"]>.ret;`。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<WMMA_REGS> all_ldmatrix_ops = !listconcat(ldmatrix_b16_ops,`.
  **L814 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<WMMA_REGS> all_ldmatrix_ops = !listconcat(ldmatrix_b16_ops,`。
- **L815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ldmatrix_geom_m16n16_ops,`.
  **L815 CN**: 继续一个多行参数列表、初始化器或聚合项：`ldmatrix_geom_m16n16_ops,`。
- **L816 EN**: Executes a standalone statement or declaration: `ldmatrix_geom_m8n16_ops);`.
  **L816 CN**: 执行一条独立语句或声明：`ldmatrix_geom_m8n16_ops);`。
- **L817 EN**: Blank line separating nearby declarations or logic blocks.
  **L817 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L818 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<WMMA_REGS> all_stmatrix_ops = !listconcat(stmatrix_b16_ops,`.
  **L818 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<WMMA_REGS> all_stmatrix_ops = !listconcat(stmatrix_b16_ops,`。
- **L819 EN**: Executes a standalone statement or declaration: `stmatrix_b8_ops);`.
  **L819 CN**: 执行一条独立语句或声明：`stmatrix_b8_ops);`。
- **L820 EN**: Closes the current lexical scope or compound statement.
  **L820 CN**: 结束当前词法作用域或复合语句块。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L822 EN**: Declares TableGen def `NVVM_MMA_OPS`.
  **L822 CN**: 声明 TableGen def `NVVM_MMA_OPS`。
- **L823 EN**: Blank line separating nearby declarations or logic blocks.
  **L823 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L824 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if this combination of fragment and layout for WMMA load/store`.
  **L824 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if this combination of fragment and layout for WMMA load/store`。
- **L825 EN**: Comment explains nearby logic, invariants, or intent: `ops is supported; false otherwise.`.
  **L825 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ops is supported; false otherwise.`。
- **L826 EN**: Comment explains nearby logic, invariants, or intent: `E.g.`.
  **L826 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`E.g.`。
- **L827 EN**: Comment explains nearby logic, invariants, or intent: `if NVVM_WMMA_LDST_SUPPORTED<...>.ret then`.
  **L827 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if NVVM_WMMA_LDST_SUPPORTED<...>.ret then`。
- **L828 EN**: Comment explains nearby logic, invariants, or intent: `def : FOO<>; // The record will only be defined for supported ops.`.
  **L828 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def : FOO<>; // The record will only be defined for supported ops.`。
- **L829 EN**: Separator comment used for visual grouping.
  **L829 CN**: 用于视觉分组的分隔注释。
- **L830 EN**: Declares class `NVVM_WMMA_LDST_SUPPORTED<WMMA_REGS`.
  **L830 CN**: 声明 class `NVVM_WMMA_LDST_SUPPORTED<WMMA_REGS`。
- **L831 EN**: Initializes variable `f` from the right-hand expression.
  **L831 CN**: 使用右侧表达式初始化变量 `f`。
- **L832 EN**: Initializes variable `t` from the right-hand expression.
  **L832 CN**: 使用右侧表达式初始化变量 `t`。

### Lines 833-864

````tablegen

  bit ret = !cond(
    // Sub-int load and store requires A fragment to be of row layout and B
    // fragments to be of column layout.
    !and(!or(!eq(t, "b1"),
             !eq(t, "u4"),
             !eq(t, "s4")),
         !or(!and(!eq(f, "a"),
                  !ne(layout, "row")),
             !and(!eq(f, "b"),
                  !ne(layout, "col")))) : false,
    true: true
  );
}

// Returns true if this combination of layout/satf/rnd for WMMA ops is
// supported; false otherwise.
// E.g.
// if NVVM_WMMA_SUPPORTED<...>.ret then
//   def : FOO<>; // The record will only be defined for supported ops.
//
class NVVM_WMMA_SUPPORTED<list<WMMA_REGS> frags, string layout_a, string layout_b, int satf, string rnd> {
  // WMMA ops check both layouts.
  string layout = layout_a # ":" # layout_b;
  string t = frags[0].ptx_elt_type;

  bit ret = !cond(
    // only f64 wmma functions support rnd options
    // any non f64 type that uses a rnd value is invalid
    !and(!ne(t, "f64"), !ne(rnd, "")) : false,

    // satf is only valid for select types
````
- **L833 EN**: Blank line separating nearby declarations or logic blocks.
  **L833 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L834 EN**: Continues logic associated with callable symbol `cond`.
  **L834 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L835 EN**: Comment explains nearby logic, invariants, or intent: `Sub-int load and store requires A fragment to be of row layout and B`.
  **L835 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sub-int load and store requires A fragment to be of row layout and B`。
- **L836 EN**: Comment explains nearby logic, invariants, or intent: `fragments to be of column layout.`.
  **L836 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`fragments to be of column layout.`。
- **L837 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!or(!eq(t, "b1"),`.
  **L837 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!or(!eq(t, "b1"),`。
- **L838 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(t, "u4"),`.
  **L838 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(t, "u4"),`。
- **L839 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(t, "s4")),`.
  **L839 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(t, "s4")),`。
- **L840 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!and(!eq(f, "a"),`.
  **L840 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!and(!eq(f, "a"),`。
- **L841 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(layout, "row")),`.
  **L841 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(layout, "row")),`。
- **L842 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(f, "b"),`.
  **L842 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(f, "b"),`。
- **L843 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(layout, "col")))) : false,`.
  **L843 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(layout, "col")))) : false,`。
- **L844 EN**: Continues the surrounding expression or declaration: `true: true`.
  **L844 CN**: 继续构造周围的表达式或声明：`true: true`。
- **L845 EN**: Executes a standalone statement or declaration: `);`.
  **L845 CN**: 执行一条独立语句或声明：`);`。
- **L846 EN**: Closes the current lexical scope or compound statement.
  **L846 CN**: 结束当前词法作用域或复合语句块。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L848 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if this combination of layout/satf/rnd for WMMA ops is`.
  **L848 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if this combination of layout/satf/rnd for WMMA ops is`。
- **L849 EN**: Comment explains nearby logic, invariants, or intent: `supported; false otherwise.`.
  **L849 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`supported; false otherwise.`。
- **L850 EN**: Comment explains nearby logic, invariants, or intent: `E.g.`.
  **L850 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`E.g.`。
- **L851 EN**: Comment explains nearby logic, invariants, or intent: `if NVVM_WMMA_SUPPORTED<...>.ret then`.
  **L851 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if NVVM_WMMA_SUPPORTED<...>.ret then`。
- **L852 EN**: Comment explains nearby logic, invariants, or intent: `def : FOO<>; // The record will only be defined for supported ops.`.
  **L852 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def : FOO<>; // The record will only be defined for supported ops.`。
- **L853 EN**: Separator comment used for visual grouping.
  **L853 CN**: 用于视觉分组的分隔注释。
- **L854 EN**: Declares class `NVVM_WMMA_SUPPORTED<list<WMMA_REGS>`.
  **L854 CN**: 声明 class `NVVM_WMMA_SUPPORTED<list<WMMA_REGS>`。
- **L855 EN**: Comment explains nearby logic, invariants, or intent: `WMMA ops check both layouts.`.
  **L855 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA ops check both layouts.`。
- **L856 EN**: Initializes variable `layout` from the right-hand expression.
  **L856 CN**: 使用右侧表达式初始化变量 `layout`。
- **L857 EN**: Initializes variable `t` from the right-hand expression.
  **L857 CN**: 使用右侧表达式初始化变量 `t`。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L859 EN**: Continues logic associated with callable symbol `cond`.
  **L859 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L860 EN**: Comment explains nearby logic, invariants, or intent: `only f64 wmma functions support rnd options`.
  **L860 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`only f64 wmma functions support rnd options`。
- **L861 EN**: Comment explains nearby logic, invariants, or intent: `any non f64 type that uses a rnd value is invalid`.
  **L861 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`any non f64 type that uses a rnd value is invalid`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!ne(t, "f64"), !ne(rnd, "")) : false,`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!ne(t, "f64"), !ne(rnd, "")) : false,`。
- **L863 EN**: Blank line separating nearby declarations or logic blocks.
  **L863 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L864 EN**: Comment explains nearby logic, invariants, or intent: `satf is only valid for select types`.
  **L864 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`satf is only valid for select types`。

### Lines 865-896

````tablegen
    !and(!eq(satf, 1),
         !ne(t, "s8"),
         !ne(t, "u8"),
         !ne(t, "s4"),
         !ne(t, "u4"),
         !ne(t, "f16")): false,

    // Sub-int wmma requires row/column layout
    !and(!or(!eq(t, "s4"),
             !eq(t, "u4"),
             !eq(t, "b1")),
         !ne(layout, "row:col")) : false,
    true: true
  );
}

class NVVM_MMA_B1OPS<list<WMMA_REGS> frags> {
  list<string> ret = !cond(
    !eq(frags[0].ptx_elt_type, "b1") : [".xor.popc", ".and.popc"],
    true: [""]
  );
}

// Returns true if this combination of layout/satf for MMA ops is supported;
// false otherwise.
// E.g.
// if NVVM_MMA_SUPPORTED<...>.ret then
//   def : FOO<>; // The record will only be defined for supported ops.
//
class NVVM_MMA_SUPPORTED<list<WMMA_REGS> frags, string layout_a, string layout_b,
                         string kind, int satf> {
  // MMA ops check both layouts.
````
- **L865 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(satf, 1),`.
  **L865 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(satf, 1),`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(t, "s8"),`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(t, "s8"),`。
- **L867 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(t, "u8"),`.
  **L867 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(t, "u8"),`。
- **L868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(t, "s4"),`.
  **L868 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(t, "s4"),`。
- **L869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(t, "u4"),`.
  **L869 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(t, "u4"),`。
- **L870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(t, "f16")): false,`.
  **L870 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(t, "f16")): false,`。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L872 EN**: Comment explains nearby logic, invariants, or intent: `Sub-int wmma requires row/column layout`.
  **L872 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sub-int wmma requires row/column layout`。
- **L873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!or(!eq(t, "s4"),`.
  **L873 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!or(!eq(t, "s4"),`。
- **L874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(t, "u4"),`.
  **L874 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(t, "u4"),`。
- **L875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(t, "b1")),`.
  **L875 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(t, "b1")),`。
- **L876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(layout, "row:col")) : false,`.
  **L876 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(layout, "row:col")) : false,`。
- **L877 EN**: Continues the surrounding expression or declaration: `true: true`.
  **L877 CN**: 继续构造周围的表达式或声明：`true: true`。
- **L878 EN**: Executes a standalone statement or declaration: `);`.
  **L878 CN**: 执行一条独立语句或声明：`);`。
- **L879 EN**: Closes the current lexical scope or compound statement.
  **L879 CN**: 结束当前词法作用域或复合语句块。
- **L880 EN**: Blank line separating nearby declarations or logic blocks.
  **L880 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L881 EN**: Declares class `NVVM_MMA_B1OPS<list<WMMA_REGS>`.
  **L881 CN**: 声明 class `NVVM_MMA_B1OPS<list<WMMA_REGS>`。
- **L882 EN**: Continues logic associated with callable symbol `cond`.
  **L882 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L883 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(frags[0].ptx_elt_type, "b1") : [".xor.popc", ".and.popc"],`.
  **L883 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(frags[0].ptx_elt_type, "b1") : [".xor.popc", ".and.popc"],`。
- **L884 EN**: Continues the surrounding expression or declaration: `true: [""]`.
  **L884 CN**: 继续构造周围的表达式或声明：`true: [""]`。
- **L885 EN**: Executes a standalone statement or declaration: `);`.
  **L885 CN**: 执行一条独立语句或声明：`);`。
- **L886 EN**: Closes the current lexical scope or compound statement.
  **L886 CN**: 结束当前词法作用域或复合语句块。
- **L887 EN**: Blank line separating nearby declarations or logic blocks.
  **L887 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L888 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if this combination of layout/satf for MMA ops is supported;`.
  **L888 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if this combination of layout/satf for MMA ops is supported;`。
- **L889 EN**: Comment explains nearby logic, invariants, or intent: `false otherwise.`.
  **L889 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`false otherwise.`。
- **L890 EN**: Comment explains nearby logic, invariants, or intent: `E.g.`.
  **L890 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`E.g.`。
- **L891 EN**: Comment explains nearby logic, invariants, or intent: `if NVVM_MMA_SUPPORTED<...>.ret then`.
  **L891 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if NVVM_MMA_SUPPORTED<...>.ret then`。
- **L892 EN**: Comment explains nearby logic, invariants, or intent: `def : FOO<>; // The record will only be defined for supported ops.`.
  **L892 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def : FOO<>; // The record will only be defined for supported ops.`。
- **L893 EN**: Separator comment used for visual grouping.
  **L893 CN**: 用于视觉分组的分隔注释。
- **L894 EN**: Declares class `NVVM_MMA_SUPPORTED<list<WMMA_REGS>`.
  **L894 CN**: 声明 class `NVVM_MMA_SUPPORTED<list<WMMA_REGS>`。
- **L895 EN**: Continues the surrounding expression or declaration: `string kind, int satf> {`.
  **L895 CN**: 继续构造周围的表达式或声明：`string kind, int satf> {`。
- **L896 EN**: Comment explains nearby logic, invariants, or intent: `MMA ops check both layouts.`.
  **L896 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA ops check both layouts.`。

### Lines 897-928

````tablegen
  string layout = layout_a # ":" # layout_b;
  string a_type = frags[0].ptx_elt_type;
  string b_type = frags[1].ptx_elt_type;
  string c_type = frags[2].ptx_elt_type;
  string d_type = frags[3].ptx_elt_type;
  string geom = frags[0].geom;

  // gcd is a shortcut used to identify instructions that depend on
  // geom+frag_c+frag_d.
  string gcd = geom # ":" # c_type # d_type;
  bit ret = !cond(

    // Limit satf to valid types
    !and(!eq(satf, 1),
         !ne(a_type, "s8"),
         !ne(a_type, "u8"),
         !ne(a_type, "s4"),
         !ne(a_type, "u4")): false,

    // m8n8k4 has no C=f32 D=f16 variant.
    !eq(gcd, "m8n8k4:f32f16"): false,

    // only m8n8k4 for f16 does not require row:col layout
    !and(!ne(layout, "row:col"),
         !or(!ne(geom, "m8n8k4"),
             !ne(a_type, "f16"))) : false,

    // m16n8k8 requires A and B to be the same type and C and D to be the same
    // type.
    !and(!eq(geom, "m16n8k8"),
         !or(!ne(a_type, b_type),
             !ne(c_type, d_type))): false,
````
- **L897 EN**: Initializes variable `layout` from the right-hand expression.
  **L897 CN**: 使用右侧表达式初始化变量 `layout`。
- **L898 EN**: Initializes variable `a_type` from the right-hand expression.
  **L898 CN**: 使用右侧表达式初始化变量 `a_type`。
- **L899 EN**: Initializes variable `b_type` from the right-hand expression.
  **L899 CN**: 使用右侧表达式初始化变量 `b_type`。
- **L900 EN**: Initializes variable `c_type` from the right-hand expression.
  **L900 CN**: 使用右侧表达式初始化变量 `c_type`。
- **L901 EN**: Initializes variable `d_type` from the right-hand expression.
  **L901 CN**: 使用右侧表达式初始化变量 `d_type`。
- **L902 EN**: Initializes variable `geom` from the right-hand expression.
  **L902 CN**: 使用右侧表达式初始化变量 `geom`。
- **L903 EN**: Blank line separating nearby declarations or logic blocks.
  **L903 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L904 EN**: Comment explains nearby logic, invariants, or intent: `gcd is a shortcut used to identify instructions that depend on`.
  **L904 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`gcd is a shortcut used to identify instructions that depend on`。
- **L905 EN**: Comment explains nearby logic, invariants, or intent: `geom+frag_c+frag_d.`.
  **L905 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`geom+frag_c+frag_d.`。
- **L906 EN**: Initializes variable `gcd` from the right-hand expression.
  **L906 CN**: 使用右侧表达式初始化变量 `gcd`。
- **L907 EN**: Continues logic associated with callable symbol `cond`.
  **L907 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L909 EN**: Comment explains nearby logic, invariants, or intent: `Limit satf to valid types`.
  **L909 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Limit satf to valid types`。
- **L910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(satf, 1),`.
  **L910 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(satf, 1),`。
- **L911 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "s8"),`.
  **L911 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "s8"),`。
- **L912 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "u8"),`.
  **L912 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "u8"),`。
- **L913 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "s4"),`.
  **L913 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "s4"),`。
- **L914 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "u4")): false,`.
  **L914 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "u4")): false,`。
- **L915 EN**: Blank line separating nearby declarations or logic blocks.
  **L915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L916 EN**: Comment explains nearby logic, invariants, or intent: `m8n8k4 has no C=f32 D=f16 variant.`.
  **L916 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`m8n8k4 has no C=f32 D=f16 variant.`。
- **L917 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(gcd, "m8n8k4:f32f16"): false,`.
  **L917 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(gcd, "m8n8k4:f32f16"): false,`。
- **L918 EN**: Blank line separating nearby declarations or logic blocks.
  **L918 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L919 EN**: Comment explains nearby logic, invariants, or intent: `only m8n8k4 for f16 does not require row:col layout`.
  **L919 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`only m8n8k4 for f16 does not require row:col layout`。
- **L920 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!ne(layout, "row:col"),`.
  **L920 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!ne(layout, "row:col"),`。
- **L921 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!ne(geom, "m8n8k4"),`.
  **L921 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!ne(geom, "m8n8k4"),`。
- **L922 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "f16"))) : false,`.
  **L922 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "f16"))) : false,`。
- **L923 EN**: Blank line separating nearby declarations or logic blocks.
  **L923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L924 EN**: Comment explains nearby logic, invariants, or intent: `m16n8k8 requires A and B to be the same type and C and D to be the same`.
  **L924 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`m16n8k8 requires A and B to be the same type and C and D to be the same`。
- **L925 EN**: Comment explains nearby logic, invariants, or intent: `type.`.
  **L925 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`type.`。
- **L926 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k8"),`.
  **L926 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k8"),`。
- **L927 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!ne(a_type, b_type),`.
  **L927 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!ne(a_type, b_type),`。
- **L928 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(c_type, d_type))): false,`.
  **L928 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(c_type, d_type))): false,`。

### Lines 929-960

````tablegen

    // m16n8k16/m16n8k32 requires C and D to be the same type
    !and(!or(!eq(geom, "m16n8k16"),
             !eq(geom, "m16n8k32")),
         !ne(c_type, d_type)): false,

    // Limit kind to valid types and geometries
    !and(!ne(kind, ""),
         !or(!ne(geom, "m16n8k32"),
             !and(!ne(a_type, "e4m3"),
                  !ne(a_type, "e5m2"),
                  !ne(a_type, "e3m2"),
                  !ne(a_type, "e2m3"),
                  !ne(a_type, "e2m1")))): false,

    // Limit m16n8k16/m16n8k32 with no kind to valid types
    !and(!eq(kind, ""),
         !or(!eq(geom, "m16n8k16"),
             !eq(geom, "m16n8k32")),
             !or(!eq(a_type, "e3m2"),
                 !eq(a_type, "e2m3"),
                 !eq(a_type, "e2m1"),
                 !eq(b_type, "e3m2"),
                 !eq(b_type, "e2m3"),
                 !eq(b_type, "e2m1"))): false,

    // All other are OK.
    true: true
  );
}

class NVVM_MMA_BLOCK_SCALE_SUPPORTED<list<WMMA_REGS> frags, string kind, string stype, string scale_vec_size> {
````
- **L929 EN**: Blank line separating nearby declarations or logic blocks.
  **L929 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L930 EN**: Comment explains nearby logic, invariants, or intent: `m16n8k16/m16n8k32 requires C and D to be the same type`.
  **L930 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`m16n8k16/m16n8k32 requires C and D to be the same type`。
- **L931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!or(!eq(geom, "m16n8k16"),`.
  **L931 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!or(!eq(geom, "m16n8k16"),`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(geom, "m16n8k32")),`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(geom, "m16n8k32")),`。
- **L933 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(c_type, d_type)): false,`.
  **L933 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(c_type, d_type)): false,`。
- **L934 EN**: Blank line separating nearby declarations or logic blocks.
  **L934 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L935 EN**: Comment explains nearby logic, invariants, or intent: `Limit kind to valid types and geometries`.
  **L935 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Limit kind to valid types and geometries`。
- **L936 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!ne(kind, ""),`.
  **L936 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!ne(kind, ""),`。
- **L937 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!ne(geom, "m16n8k32"),`.
  **L937 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!ne(geom, "m16n8k32"),`。
- **L938 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!ne(a_type, "e4m3"),`.
  **L938 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!ne(a_type, "e4m3"),`。
- **L939 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "e5m2"),`.
  **L939 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "e5m2"),`。
- **L940 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "e3m2"),`.
  **L940 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "e3m2"),`。
- **L941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "e2m3"),`.
  **L941 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "e2m3"),`。
- **L942 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, "e2m1")))): false,`.
  **L942 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, "e2m1")))): false,`。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L944 EN**: Comment explains nearby logic, invariants, or intent: `Limit m16n8k16/m16n8k32 with no kind to valid types`.
  **L944 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Limit m16n8k16/m16n8k32 with no kind to valid types`。
- **L945 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(kind, ""),`.
  **L945 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(kind, ""),`。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(geom, "m16n8k16"),`.
  **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(geom, "m16n8k16"),`。
- **L947 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(geom, "m16n8k32")),`.
  **L947 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(geom, "m16n8k32")),`。
- **L948 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(a_type, "e3m2"),`.
  **L948 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(a_type, "e3m2"),`。
- **L949 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "e2m3"),`.
  **L949 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "e2m3"),`。
- **L950 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "e2m1"),`.
  **L950 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "e2m1"),`。
- **L951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(b_type, "e3m2"),`.
  **L951 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(b_type, "e3m2"),`。
- **L952 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(b_type, "e2m3"),`.
  **L952 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(b_type, "e2m3"),`。
- **L953 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(b_type, "e2m1"))): false,`.
  **L953 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(b_type, "e2m1"))): false,`。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L955 EN**: Comment explains nearby logic, invariants, or intent: `All other are OK.`.
  **L955 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All other are OK.`。
- **L956 EN**: Continues the surrounding expression or declaration: `true: true`.
  **L956 CN**: 继续构造周围的表达式或声明：`true: true`。
- **L957 EN**: Executes a standalone statement or declaration: `);`.
  **L957 CN**: 执行一条独立语句或声明：`);`。
- **L958 EN**: Closes the current lexical scope or compound statement.
  **L958 CN**: 结束当前词法作用域或复合语句块。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L960 EN**: Declares class `NVVM_MMA_BLOCK_SCALE_SUPPORTED<list<WMMA_REGS>`.
  **L960 CN**: 声明 class `NVVM_MMA_BLOCK_SCALE_SUPPORTED<list<WMMA_REGS>`。

### Lines 961-992

````tablegen
  string geom = frags[0].geom;

  bit ret = !cond(
    !and(!eq(geom, "m16n8k64"),
         !eq(kind, "mxf4"),
         !or(!eq(scale_vec_size, ""),
             !eq(scale_vec_size, ".scale_2x")),
         !eq(stype, "ue8m0")) : true,
    !and(!eq(geom, "m16n8k64"),
         !eq(kind, "mxf4nvf4"),
         !eq(scale_vec_size, ".scale_2x"),
         !eq(stype, "ue8m0")) : true,
    !and(!eq(geom, "m16n8k64"),
         !eq(kind, "mxf4nvf4"),
         !eq(scale_vec_size, ".scale_4x"),
         !or(!eq(stype, "ue4m3"),
             !eq(stype, "ue8m0"))) : true,
    !and(!eq(geom, "m16n8k32"),
         !eq(kind, "mxf8f6f4"),
         !or(!eq(scale_vec_size, ""),
             !eq(scale_vec_size, ".scale_1x")),
         !eq(stype, "ue8m0")) : true,
    true: false
  );
}

// Returns true if the fragment is valid for ldmatrix ops is supported;
// false otherwise.
// E.g.
// if NVVM_LDMATRIX_SUPPORTED<...>.ret then
//   def : FOO<>; // The record will only be defined for supported ops.
//
````
- **L961 EN**: Initializes variable `geom` from the right-hand expression.
  **L961 CN**: 使用右侧表达式初始化变量 `geom`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L963 EN**: Continues logic associated with callable symbol `cond`.
  **L963 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L964 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k64"),`.
  **L964 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k64"),`。
- **L965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf4"),`.
  **L965 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf4"),`。
- **L966 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(scale_vec_size, ""),`.
  **L966 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(scale_vec_size, ""),`。
- **L967 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_2x")),`.
  **L967 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_2x")),`。
- **L968 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0")) : true,`.
  **L968 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0")) : true,`。
- **L969 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k64"),`.
  **L969 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k64"),`。
- **L970 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf4nvf4"),`.
  **L970 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf4nvf4"),`。
- **L971 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_2x"),`.
  **L971 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_2x"),`。
- **L972 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0")) : true,`.
  **L972 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0")) : true,`。
- **L973 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k64"),`.
  **L973 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k64"),`。
- **L974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf4nvf4"),`.
  **L974 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf4nvf4"),`。
- **L975 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_4x"),`.
  **L975 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_4x"),`。
- **L976 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(stype, "ue4m3"),`.
  **L976 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(stype, "ue4m3"),`。
- **L977 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0"))) : true,`.
  **L977 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0"))) : true,`。
- **L978 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k32"),`.
  **L978 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k32"),`。
- **L979 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf8f6f4"),`.
  **L979 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf8f6f4"),`。
- **L980 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(scale_vec_size, ""),`.
  **L980 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(scale_vec_size, ""),`。
- **L981 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_1x")),`.
  **L981 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_1x")),`。
- **L982 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0")) : true,`.
  **L982 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0")) : true,`。
- **L983 EN**: Continues the surrounding expression or declaration: `true: false`.
  **L983 CN**: 继续构造周围的表达式或声明：`true: false`。
- **L984 EN**: Executes a standalone statement or declaration: `);`.
  **L984 CN**: 执行一条独立语句或声明：`);`。
- **L985 EN**: Closes the current lexical scope or compound statement.
  **L985 CN**: 结束当前词法作用域或复合语句块。
- **L986 EN**: Blank line separating nearby declarations or logic blocks.
  **L986 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L987 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if the fragment is valid for ldmatrix ops is supported;`.
  **L987 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if the fragment is valid for ldmatrix ops is supported;`。
- **L988 EN**: Comment explains nearby logic, invariants, or intent: `false otherwise.`.
  **L988 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`false otherwise.`。
- **L989 EN**: Comment explains nearby logic, invariants, or intent: `E.g.`.
  **L989 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`E.g.`。
- **L990 EN**: Comment explains nearby logic, invariants, or intent: `if NVVM_LDMATRIX_SUPPORTED<...>.ret then`.
  **L990 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if NVVM_LDMATRIX_SUPPORTED<...>.ret then`。
- **L991 EN**: Comment explains nearby logic, invariants, or intent: `def : FOO<>; // The record will only be defined for supported ops.`.
  **L991 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def : FOO<>; // The record will only be defined for supported ops.`。
- **L992 EN**: Separator comment used for visual grouping.
  **L992 CN**: 用于视觉分组的分隔注释。

### Lines 993-1024

````tablegen
class NVVM_LDMATRIX_SUPPORTED<WMMA_REGS frag, bit trans> {
  string g = frag.geom;
  string t = frag.ptx_elt_type;

  bit ret = !cond(
    !and(!eq(g, "m8n8"), !eq(t, "b16")): true,
    !and(!eq(g, "m16n16"), !eq(t, "b8"), !eq(trans, 1)): true,
    !and(!eq(g, "m16n16"), !eq(t, "b8x16.b6x16_p32"), !eq(trans, 1)): true,
    !and(!eq(g, "m16n16"), !eq(t, "b8x16.b4x16_p64"), !eq(trans, 1)): true,
    !and(!eq(g, "m8n16"), !eq(t, "b8"), !eq(trans, 0)): true,
    !and(!eq(g, "m8n16"), !eq(t, "b8x16.b6x16_p32"), !eq(trans, 0)): true,
    !and(!eq(g, "m8n16"), !eq(t, "b8x16.b4x16_p64"), !eq(trans, 0)): true,
    true: false
  );
}

// Returns true if the fragment is valid for stmatrix ops is supported;
// false otherwise.
class NVVM_STMATRIX_SUPPORTED<WMMA_REGS frag, bit trans> {
  string g = frag.geom;
  string t = frag.ptx_elt_type;

  bit ret = !cond(
    !and(!eq(g, "m8n8"), !eq(t, "b16")): true,
    !and(!eq(g, "m16n8"), !eq(t, "b8"), !eq(trans, 1)): true,
    true: false
  );
}


// Returns true if this combination of layout/kind/satf for MMA.SP ops is supported;
// false otherwise.
````
- **L993 EN**: Declares class `NVVM_LDMATRIX_SUPPORTED<WMMA_REGS`.
  **L993 CN**: 声明 class `NVVM_LDMATRIX_SUPPORTED<WMMA_REGS`。
- **L994 EN**: Initializes variable `g` from the right-hand expression.
  **L994 CN**: 使用右侧表达式初始化变量 `g`。
- **L995 EN**: Initializes variable `t` from the right-hand expression.
  **L995 CN**: 使用右侧表达式初始化变量 `t`。
- **L996 EN**: Blank line separating nearby declarations or logic blocks.
  **L996 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L997 EN**: Continues logic associated with callable symbol `cond`.
  **L997 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L998 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m8n8"), !eq(t, "b16")): true,`.
  **L998 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m8n8"), !eq(t, "b16")): true,`。
- **L999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m16n16"), !eq(t, "b8"), !eq(trans, 1)): true,`.
  **L999 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m16n16"), !eq(t, "b8"), !eq(trans, 1)): true,`。
- **L1000 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m16n16"), !eq(t, "b8x16.b6x16_p32"), !eq(trans, 1)): true,`.
  **L1000 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m16n16"), !eq(t, "b8x16.b6x16_p32"), !eq(trans, 1)): true,`。
- **L1001 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m16n16"), !eq(t, "b8x16.b4x16_p64"), !eq(trans, 1)): true,`.
  **L1001 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m16n16"), !eq(t, "b8x16.b4x16_p64"), !eq(trans, 1)): true,`。
- **L1002 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m8n16"), !eq(t, "b8"), !eq(trans, 0)): true,`.
  **L1002 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m8n16"), !eq(t, "b8"), !eq(trans, 0)): true,`。
- **L1003 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m8n16"), !eq(t, "b8x16.b6x16_p32"), !eq(trans, 0)): true,`.
  **L1003 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m8n16"), !eq(t, "b8x16.b6x16_p32"), !eq(trans, 0)): true,`。
- **L1004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m8n16"), !eq(t, "b8x16.b4x16_p64"), !eq(trans, 0)): true,`.
  **L1004 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m8n16"), !eq(t, "b8x16.b4x16_p64"), !eq(trans, 0)): true,`。
- **L1005 EN**: Continues the surrounding expression or declaration: `true: false`.
  **L1005 CN**: 继续构造周围的表达式或声明：`true: false`。
- **L1006 EN**: Executes a standalone statement or declaration: `);`.
  **L1006 CN**: 执行一条独立语句或声明：`);`。
- **L1007 EN**: Closes the current lexical scope or compound statement.
  **L1007 CN**: 结束当前词法作用域或复合语句块。
- **L1008 EN**: Blank line separating nearby declarations or logic blocks.
  **L1008 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1009 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if the fragment is valid for stmatrix ops is supported;`.
  **L1009 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if the fragment is valid for stmatrix ops is supported;`。
- **L1010 EN**: Comment explains nearby logic, invariants, or intent: `false otherwise.`.
  **L1010 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`false otherwise.`。
- **L1011 EN**: Declares class `NVVM_STMATRIX_SUPPORTED<WMMA_REGS`.
  **L1011 CN**: 声明 class `NVVM_STMATRIX_SUPPORTED<WMMA_REGS`。
- **L1012 EN**: Initializes variable `g` from the right-hand expression.
  **L1012 CN**: 使用右侧表达式初始化变量 `g`。
- **L1013 EN**: Initializes variable `t` from the right-hand expression.
  **L1013 CN**: 使用右侧表达式初始化变量 `t`。
- **L1014 EN**: Blank line separating nearby declarations or logic blocks.
  **L1014 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1015 EN**: Continues logic associated with callable symbol `cond`.
  **L1015 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L1016 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m8n8"), !eq(t, "b16")): true,`.
  **L1016 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m8n8"), !eq(t, "b16")): true,`。
- **L1017 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(g, "m16n8"), !eq(t, "b8"), !eq(trans, 1)): true,`.
  **L1017 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(g, "m16n8"), !eq(t, "b8"), !eq(trans, 1)): true,`。
- **L1018 EN**: Continues the surrounding expression or declaration: `true: false`.
  **L1018 CN**: 继续构造周围的表达式或声明：`true: false`。
- **L1019 EN**: Executes a standalone statement or declaration: `);`.
  **L1019 CN**: 执行一条独立语句或声明：`);`。
- **L1020 EN**: Closes the current lexical scope or compound statement.
  **L1020 CN**: 结束当前词法作用域或复合语句块。
- **L1021 EN**: Blank line separating nearby declarations or logic blocks.
  **L1021 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1022 EN**: Blank line separating nearby declarations or logic blocks.
  **L1022 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1023 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if this combination of layout/kind/satf for MMA.SP ops is supported;`.
  **L1023 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if this combination of layout/kind/satf for MMA.SP ops is supported;`。
- **L1024 EN**: Comment explains nearby logic, invariants, or intent: `false otherwise.`.
  **L1024 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`false otherwise.`。

### Lines 1025-1056

````tablegen
// E.g.
// if NVVM_MMA_SP_SUPPORTED<...>.ret then
//   def : FOO<>; // The record will only be defined for supported ops.
//
class NVVM_MMA_SP_SUPPORTED<list<WMMA_REGS> frags, string metadata,
                            string kind, int satf> {
  // MMA.SP ops check both layouts.
  string a_type = frags[0].ptx_elt_type;
  string b_type = frags[1].ptx_elt_type;
  string c_type = frags[2].ptx_elt_type;
  string d_type = frags[3].ptx_elt_type;
  string geom = frags[0].geom;

  bit is_int = !or(!eq(a_type, "s8"),
                   !eq(a_type, "u8"),
                   !eq(a_type, "s4"),
                   !eq(a_type, "u4"));

  bit ret = !cond(

    // Limit satf to valid types
    !and(!eq(satf, 1),
         !eq(is_int, 0)): false,

    // f16/bf16/tf32 requires A and B to be the same type.
    !and(!or(!eq(a_type, "f16"),
             !eq(a_type, "bf16"),
             !eq(a_type, "tf32")),
         !ne(a_type, b_type)): false,

    // m16n8k16, m16n8k32 and m16n8k64 requires C and D to be the same type.
    !and(!or(!eq(geom, "m16n8k16"),
````
- **L1025 EN**: Comment explains nearby logic, invariants, or intent: `E.g.`.
  **L1025 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`E.g.`。
- **L1026 EN**: Comment explains nearby logic, invariants, or intent: `if NVVM_MMA_SP_SUPPORTED<...>.ret then`.
  **L1026 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if NVVM_MMA_SP_SUPPORTED<...>.ret then`。
- **L1027 EN**: Comment explains nearby logic, invariants, or intent: `def : FOO<>; // The record will only be defined for supported ops.`.
  **L1027 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def : FOO<>; // The record will only be defined for supported ops.`。
- **L1028 EN**: Separator comment used for visual grouping.
  **L1028 CN**: 用于视觉分组的分隔注释。
- **L1029 EN**: Declares class `NVVM_MMA_SP_SUPPORTED<list<WMMA_REGS>`.
  **L1029 CN**: 声明 class `NVVM_MMA_SP_SUPPORTED<list<WMMA_REGS>`。
- **L1030 EN**: Continues the surrounding expression or declaration: `string kind, int satf> {`.
  **L1030 CN**: 继续构造周围的表达式或声明：`string kind, int satf> {`。
- **L1031 EN**: Comment explains nearby logic, invariants, or intent: `MMA.SP ops check both layouts.`.
  **L1031 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA.SP ops check both layouts.`。
- **L1032 EN**: Initializes variable `a_type` from the right-hand expression.
  **L1032 CN**: 使用右侧表达式初始化变量 `a_type`。
- **L1033 EN**: Initializes variable `b_type` from the right-hand expression.
  **L1033 CN**: 使用右侧表达式初始化变量 `b_type`。
- **L1034 EN**: Initializes variable `c_type` from the right-hand expression.
  **L1034 CN**: 使用右侧表达式初始化变量 `c_type`。
- **L1035 EN**: Initializes variable `d_type` from the right-hand expression.
  **L1035 CN**: 使用右侧表达式初始化变量 `d_type`。
- **L1036 EN**: Initializes variable `geom` from the right-hand expression.
  **L1036 CN**: 使用右侧表达式初始化变量 `geom`。
- **L1037 EN**: Blank line separating nearby declarations or logic blocks.
  **L1037 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1038 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `bit is_int = !or(!eq(a_type, "s8"),`.
  **L1038 CN**: 继续一个多行参数列表、初始化器或聚合项：`bit is_int = !or(!eq(a_type, "s8"),`。
- **L1039 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "u8"),`.
  **L1039 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "u8"),`。
- **L1040 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "s4"),`.
  **L1040 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "s4"),`。
- **L1041 EN**: Executes a call or declaration centered on `!eq`.
  **L1041 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L1042 EN**: Blank line separating nearby declarations or logic blocks.
  **L1042 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1043 EN**: Continues logic associated with callable symbol `cond`.
  **L1043 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L1044 EN**: Blank line separating nearby declarations or logic blocks.
  **L1044 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1045 EN**: Comment explains nearby logic, invariants, or intent: `Limit satf to valid types`.
  **L1045 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Limit satf to valid types`。
- **L1046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(satf, 1),`.
  **L1046 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(satf, 1),`。
- **L1047 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(is_int, 0)): false,`.
  **L1047 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(is_int, 0)): false,`。
- **L1048 EN**: Blank line separating nearby declarations or logic blocks.
  **L1048 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1049 EN**: Comment explains nearby logic, invariants, or intent: `f16/bf16/tf32 requires A and B to be the same type.`.
  **L1049 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`f16/bf16/tf32 requires A and B to be the same type.`。
- **L1050 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!or(!eq(a_type, "f16"),`.
  **L1050 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!or(!eq(a_type, "f16"),`。
- **L1051 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "bf16"),`.
  **L1051 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "bf16"),`。
- **L1052 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "tf32")),`.
  **L1052 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "tf32")),`。
- **L1053 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(a_type, b_type)): false,`.
  **L1053 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(a_type, b_type)): false,`。
- **L1054 EN**: Blank line separating nearby declarations or logic blocks.
  **L1054 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1055 EN**: Comment explains nearby logic, invariants, or intent: `m16n8k16, m16n8k32 and m16n8k64 requires C and D to be the same type.`.
  **L1055 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`m16n8k16, m16n8k32 and m16n8k64 requires C and D to be the same type.`。
- **L1056 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!or(!eq(geom, "m16n8k16"),`.
  **L1056 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!or(!eq(geom, "m16n8k16"),`。

### Lines 1057-1088

````tablegen
             !eq(geom, "m16n8k32"),
             !eq(geom, "m16n8k64")),
         !ne(c_type, d_type)): false,

    !and(!eq(kind, ""),
         !or(!eq(a_type, "e3m2"),
             !eq(a_type, "e2m3"),
             !eq(a_type, "e2m1"),
             !eq(b_type, "e3m2"),
             !eq(b_type, "e2m3"),
             !eq(b_type, "e2m1"))): false,

    !and(!eq(kind, ""),
         !eq(geom, "m16n8k64"),
         !or(!eq(c_type, "f16"),
             !eq(d_type, "f16"))): false,

    !and(!ne(kind, ""),
         !or(!eq(metadata, "sp"),
             !ne(geom, "m16n8k64"),
             !eq(is_int, 1))): false,

    // All other are OK.
    true: true
  );
}


// Returns true if this combination of kind/scale_vec_size/stype
// for MMA.SP ops is supported;
// false otherwise.
// E.g.
````
- **L1057 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(geom, "m16n8k32"),`.
  **L1057 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(geom, "m16n8k32"),`。
- **L1058 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(geom, "m16n8k64")),`.
  **L1058 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(geom, "m16n8k64")),`。
- **L1059 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(c_type, d_type)): false,`.
  **L1059 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(c_type, d_type)): false,`。
- **L1060 EN**: Blank line separating nearby declarations or logic blocks.
  **L1060 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1061 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(kind, ""),`.
  **L1061 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(kind, ""),`。
- **L1062 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(a_type, "e3m2"),`.
  **L1062 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(a_type, "e3m2"),`。
- **L1063 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "e2m3"),`.
  **L1063 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "e2m3"),`。
- **L1064 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(a_type, "e2m1"),`.
  **L1064 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(a_type, "e2m1"),`。
- **L1065 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(b_type, "e3m2"),`.
  **L1065 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(b_type, "e3m2"),`。
- **L1066 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(b_type, "e2m3"),`.
  **L1066 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(b_type, "e2m3"),`。
- **L1067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(b_type, "e2m1"))): false,`.
  **L1067 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(b_type, "e2m1"))): false,`。
- **L1068 EN**: Blank line separating nearby declarations or logic blocks.
  **L1068 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1069 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(kind, ""),`.
  **L1069 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(kind, ""),`。
- **L1070 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(geom, "m16n8k64"),`.
  **L1070 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(geom, "m16n8k64"),`。
- **L1071 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(c_type, "f16"),`.
  **L1071 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(c_type, "f16"),`。
- **L1072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(d_type, "f16"))): false,`.
  **L1072 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(d_type, "f16"))): false,`。
- **L1073 EN**: Blank line separating nearby declarations or logic blocks.
  **L1073 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1074 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!ne(kind, ""),`.
  **L1074 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!ne(kind, ""),`。
- **L1075 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(metadata, "sp"),`.
  **L1075 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(metadata, "sp"),`。
- **L1076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!ne(geom, "m16n8k64"),`.
  **L1076 CN**: 继续一个多行参数列表、初始化器或聚合项：`!ne(geom, "m16n8k64"),`。
- **L1077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(is_int, 1))): false,`.
  **L1077 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(is_int, 1))): false,`。
- **L1078 EN**: Blank line separating nearby declarations or logic blocks.
  **L1078 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1079 EN**: Comment explains nearby logic, invariants, or intent: `All other are OK.`.
  **L1079 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All other are OK.`。
- **L1080 EN**: Continues the surrounding expression or declaration: `true: true`.
  **L1080 CN**: 继续构造周围的表达式或声明：`true: true`。
- **L1081 EN**: Executes a standalone statement or declaration: `);`.
  **L1081 CN**: 执行一条独立语句或声明：`);`。
- **L1082 EN**: Closes the current lexical scope or compound statement.
  **L1082 CN**: 结束当前词法作用域或复合语句块。
- **L1083 EN**: Blank line separating nearby declarations or logic blocks.
  **L1083 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1084 EN**: Blank line separating nearby declarations or logic blocks.
  **L1084 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1085 EN**: Comment explains nearby logic, invariants, or intent: `Returns true if this combination of kind/scale_vec_size/stype`.
  **L1085 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Returns true if this combination of kind/scale_vec_size/stype`。
- **L1086 EN**: Comment explains nearby logic, invariants, or intent: `for MMA.SP ops is supported;`.
  **L1086 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for MMA.SP ops is supported;`。
- **L1087 EN**: Comment explains nearby logic, invariants, or intent: `false otherwise.`.
  **L1087 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`false otherwise.`。
- **L1088 EN**: Comment explains nearby logic, invariants, or intent: `E.g.`.
  **L1088 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`E.g.`。

### Lines 1089-1120

````tablegen
// if NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<...>.ret then
//   def : FOO<>; // The record will only be defined for supported ops.
//
class NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<list<WMMA_REGS> frags, string kind,
                                        string stype, string scale_vec_size> {
  // MMA.SP ops check both layouts.
  string a_type = frags[0].ptx_elt_type;
  string b_type = frags[1].ptx_elt_type;
  string c_type = frags[2].ptx_elt_type;
  string d_type = frags[3].ptx_elt_type;
  string geom = frags[0].geom;

  bit ret = !cond(
    !and(!eq(geom, "m16n8k128"),
         !eq(kind, "mxf4"),
         !eq(stype, "ue8m0"),
         !or(!eq(scale_vec_size, ""),
             !eq(scale_vec_size, ".scale_2x"))): true,

    !and(!eq(geom, "m16n8k128"),
         !eq(kind, "mxf4nvf4"),
         !eq(stype, "ue8m0"),
         !eq(scale_vec_size, ".scale_2x")): true,

    !and(!eq(geom, "m16n8k128"),
         !eq(kind, "mxf4nvf4"),
         !or(!eq(stype, "ue4m3"),
             !eq(stype, "ue8m0")),
         !eq(scale_vec_size, ".scale_4x")): true,

    !and(!eq(geom, "m16n8k64"),
         !eq(kind, "mxf8f6f4"),
````
- **L1089 EN**: Comment explains nearby logic, invariants, or intent: `if NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<...>.ret then`.
  **L1089 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`if NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<...>.ret then`。
- **L1090 EN**: Comment explains nearby logic, invariants, or intent: `def : FOO<>; // The record will only be defined for supported ops.`.
  **L1090 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`def : FOO<>; // The record will only be defined for supported ops.`。
- **L1091 EN**: Separator comment used for visual grouping.
  **L1091 CN**: 用于视觉分组的分隔注释。
- **L1092 EN**: Declares class `NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<list<WMMA_REGS>`.
  **L1092 CN**: 声明 class `NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<list<WMMA_REGS>`。
- **L1093 EN**: Continues the surrounding expression or declaration: `string stype, string scale_vec_size> {`.
  **L1093 CN**: 继续构造周围的表达式或声明：`string stype, string scale_vec_size> {`。
- **L1094 EN**: Comment explains nearby logic, invariants, or intent: `MMA.SP ops check both layouts.`.
  **L1094 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA.SP ops check both layouts.`。
- **L1095 EN**: Initializes variable `a_type` from the right-hand expression.
  **L1095 CN**: 使用右侧表达式初始化变量 `a_type`。
- **L1096 EN**: Initializes variable `b_type` from the right-hand expression.
  **L1096 CN**: 使用右侧表达式初始化变量 `b_type`。
- **L1097 EN**: Initializes variable `c_type` from the right-hand expression.
  **L1097 CN**: 使用右侧表达式初始化变量 `c_type`。
- **L1098 EN**: Initializes variable `d_type` from the right-hand expression.
  **L1098 CN**: 使用右侧表达式初始化变量 `d_type`。
- **L1099 EN**: Initializes variable `geom` from the right-hand expression.
  **L1099 CN**: 使用右侧表达式初始化变量 `geom`。
- **L1100 EN**: Blank line separating nearby declarations or logic blocks.
  **L1100 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1101 EN**: Continues logic associated with callable symbol `cond`.
  **L1101 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L1102 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k128"),`.
  **L1102 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k128"),`。
- **L1103 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf4"),`.
  **L1103 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf4"),`。
- **L1104 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0"),`.
  **L1104 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0"),`。
- **L1105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(scale_vec_size, ""),`.
  **L1105 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(scale_vec_size, ""),`。
- **L1106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_2x"))): true,`.
  **L1106 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_2x"))): true,`。
- **L1107 EN**: Blank line separating nearby declarations or logic blocks.
  **L1107 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1108 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k128"),`.
  **L1108 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k128"),`。
- **L1109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf4nvf4"),`.
  **L1109 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf4nvf4"),`。
- **L1110 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0"),`.
  **L1110 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0"),`。
- **L1111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_2x")): true,`.
  **L1111 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_2x")): true,`。
- **L1112 EN**: Blank line separating nearby declarations or logic blocks.
  **L1112 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1113 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k128"),`.
  **L1113 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k128"),`。
- **L1114 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf4nvf4"),`.
  **L1114 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf4nvf4"),`。
- **L1115 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(stype, "ue4m3"),`.
  **L1115 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(stype, "ue4m3"),`。
- **L1116 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0")),`.
  **L1116 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0")),`。
- **L1117 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_4x")): true,`.
  **L1117 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_4x")): true,`。
- **L1118 EN**: Blank line separating nearby declarations or logic blocks.
  **L1118 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1119 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(geom, "m16n8k64"),`.
  **L1119 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(geom, "m16n8k64"),`。
- **L1120 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(kind, "mxf8f6f4"),`.
  **L1120 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(kind, "mxf8f6f4"),`。

### Lines 1121-1152

````tablegen
         !eq(stype, "ue8m0"),
         !or(!eq(scale_vec_size, ""),
             !eq(scale_vec_size, ".scale_1x"))): true,

    // All other are NOT OK.
    true: false
  );
}


class SHFL_INFO<bit sync, string mode, string type, bit return_pred> {
  string Suffix = !if(sync, "sync_", "")
                  # mode # "_"
                  # type
                  # !if(return_pred, "p", "");

  string Name = "int_nvvm_shfl_" # Suffix;
  bit withGccBuiltin = !not(return_pred);
  LLVMType OpType = !cond(
    !eq(type,"i32"): llvm_i32_ty,
    !eq(type,"f32"): llvm_float_ty);
  list<LLVMType> RetTy = !if(return_pred, [OpType, llvm_i1_ty], [OpType]);
  list<LLVMType> ArgsTy = !if(sync,
    [llvm_i32_ty, OpType, llvm_i32_ty, llvm_i32_ty],
    [OpType, llvm_i32_ty, llvm_i32_ty]);
}

class NVVM_TCGEN05_LDST_ACCESS_SIZE<string Shape, int Num, string ElemType = "i32"> {
  int shift = !cond(!eq(Shape, "16x128b"): 1,
                    !eq(Shape, "16x256b"): 2,
                    true : 0);

````
- **L1121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(stype, "ue8m0"),`.
  **L1121 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(stype, "ue8m0"),`。
- **L1122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(scale_vec_size, ""),`.
  **L1122 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(scale_vec_size, ""),`。
- **L1123 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_vec_size, ".scale_1x"))): true,`.
  **L1123 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_vec_size, ".scale_1x"))): true,`。
- **L1124 EN**: Blank line separating nearby declarations or logic blocks.
  **L1124 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1125 EN**: Comment explains nearby logic, invariants, or intent: `All other are NOT OK.`.
  **L1125 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`All other are NOT OK.`。
- **L1126 EN**: Continues the surrounding expression or declaration: `true: false`.
  **L1126 CN**: 继续构造周围的表达式或声明：`true: false`。
- **L1127 EN**: Executes a standalone statement or declaration: `);`.
  **L1127 CN**: 执行一条独立语句或声明：`);`。
- **L1128 EN**: Closes the current lexical scope or compound statement.
  **L1128 CN**: 结束当前词法作用域或复合语句块。
- **L1129 EN**: Blank line separating nearby declarations or logic blocks.
  **L1129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1130 EN**: Blank line separating nearby declarations or logic blocks.
  **L1130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1131 EN**: Declares class `SHFL_INFO<bit`.
  **L1131 CN**: 声明 class `SHFL_INFO<bit`。
- **L1132 EN**: Continues the surrounding expression or declaration: `string Suffix = !if(sync, "sync_", "")`.
  **L1132 CN**: 继续构造周围的表达式或声明：`string Suffix = !if(sync, "sync_", "")`。
- **L1133 EN**: Continues the surrounding expression or declaration: `# mode # "_"`.
  **L1133 CN**: 继续构造周围的表达式或声明：`# mode # "_"`。
- **L1134 EN**: Continues the surrounding expression or declaration: `# type`.
  **L1134 CN**: 继续构造周围的表达式或声明：`# type`。
- **L1135 EN**: Executes a call or declaration centered on `!if`.
  **L1135 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1136 EN**: Blank line separating nearby declarations or logic blocks.
  **L1136 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1137 EN**: Initializes variable `Name` from the right-hand expression.
  **L1137 CN**: 使用右侧表达式初始化变量 `Name`。
- **L1138 EN**: Initializes variable `withGccBuiltin` from the right-hand expression.
  **L1138 CN**: 使用右侧表达式初始化变量 `withGccBuiltin`。
- **L1139 EN**: Continues logic associated with callable symbol `cond`.
  **L1139 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L1140 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(type,"i32"): llvm_i32_ty,`.
  **L1140 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(type,"i32"): llvm_i32_ty,`。
- **L1141 EN**: Executes a call or declaration centered on `!eq`.
  **L1141 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L1142 EN**: Initializes variable `RetTy` from the right-hand expression.
  **L1142 CN**: 使用右侧表达式初始化变量 `RetTy`。
- **L1143 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> ArgsTy = !if(sync,`.
  **L1143 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> ArgsTy = !if(sync,`。
- **L1144 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, OpType, llvm_i32_ty, llvm_i32_ty],`.
  **L1144 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, OpType, llvm_i32_ty, llvm_i32_ty],`。
- **L1145 EN**: Executes a standalone statement or declaration: `[OpType, llvm_i32_ty, llvm_i32_ty]);`.
  **L1145 CN**: 执行一条独立语句或声明：`[OpType, llvm_i32_ty, llvm_i32_ty]);`。
- **L1146 EN**: Closes the current lexical scope or compound statement.
  **L1146 CN**: 结束当前词法作用域或复合语句块。
- **L1147 EN**: Blank line separating nearby declarations or logic blocks.
  **L1147 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1148 EN**: Declares class `NVVM_TCGEN05_LDST_ACCESS_SIZE<string`.
  **L1148 CN**: 声明 class `NVVM_TCGEN05_LDST_ACCESS_SIZE<string`。
- **L1149 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int shift = !cond(!eq(Shape, "16x128b"): 1,`.
  **L1149 CN**: 继续一个多行参数列表、初始化器或聚合项：`int shift = !cond(!eq(Shape, "16x128b"): 1,`。
- **L1150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(Shape, "16x256b"): 2,`.
  **L1150 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(Shape, "16x256b"): 2,`。
- **L1151 EN**: Executes a standalone statement or declaration: `true : 0);`.
  **L1151 CN**: 执行一条独立语句或声明：`true : 0);`。
- **L1152 EN**: Blank line separating nearby declarations or logic blocks.
  **L1152 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1153-1184

````tablegen
  int veclen = !shl(1, !add(Num, shift));

  int valid = !le(veclen, 128);
  LLVMType type = !cond(!eq(veclen,   1): LLVMType<!cast<ValueType>(ElemType)>,
                        !eq(veclen,   2): LLVMType<!cast<ValueType>("v"#2#ElemType)>,
                        !eq(veclen,   4): LLVMType<!cast<ValueType>("v"#4#ElemType)>,
                        !eq(veclen,   8): LLVMType<!cast<ValueType>("v"#8#ElemType)>,
                        !eq(veclen,  16): LLVMType<!cast<ValueType>("v"#16#ElemType)>,
                        !eq(veclen,  32): LLVMType<!cast<ValueType>("v"#32#ElemType)>,
                        !eq(veclen,  64): LLVMType<!cast<ValueType>("v"#64#ElemType)>,
                        !eq(veclen, 128): LLVMType<!cast<ValueType>("v"#128#ElemType)>,
                        true : llvm_void_ty);
}

class NVVM_TCGEN05_MMA_BASE<string Space, bit IsSparse, string Kind = ""> {
  LLVMType a_operand_type = !if(!eq(Space, "tensor"),
                                  llvm_tmem_ptr_ty, llvm_i64_ty);
  list<LLVMType> common_args = !listconcat(
                                  [llvm_tmem_ptr_ty, // d
                                   a_operand_type,   // a
                                   llvm_i64_ty,      // b
                                   llvm_i32_ty,      // idesc
                                   llvm_i1_ty],      // enable_input_d
                                  !if(!eq(IsSparse, 1), [llvm_tmem_ptr_ty], [])); // spmetadata
  list<IntrinsicProperty> common_intr_props = !listconcat(
                            [IntrArgMemOnly, WriteOnly<ArgIndex<0>>],
                            !if(!eq(Space, "tensor"), [ReadOnly<ArgIndex<1>>], [])
                          );

  string Prefix = "llvm.nvvm.tcgen05.mma";
  string SpSpaceKindStr = !if(IsSparse, ".sp", "")
                          # "." # Space
````
- **L1153 EN**: Initializes variable `veclen` from the right-hand expression.
  **L1153 CN**: 使用右侧表达式初始化变量 `veclen`。
- **L1154 EN**: Blank line separating nearby declarations or logic blocks.
  **L1154 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1155 EN**: Initializes variable `valid` from the right-hand expression.
  **L1155 CN**: 使用右侧表达式初始化变量 `valid`。
- **L1156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMType type = !cond(!eq(veclen,   1): LLVMType<!cast<ValueType>(ElemType)>,`.
  **L1156 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMType type = !cond(!eq(veclen,   1): LLVMType<!cast<ValueType>(ElemType)>,`。
- **L1157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(veclen,   2): LLVMType<!cast<ValueType>("v"#2#ElemType)>,`.
  **L1157 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(veclen,   2): LLVMType<!cast<ValueType>("v"#2#ElemType)>,`。
- **L1158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(veclen,   4): LLVMType<!cast<ValueType>("v"#4#ElemType)>,`.
  **L1158 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(veclen,   4): LLVMType<!cast<ValueType>("v"#4#ElemType)>,`。
- **L1159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(veclen,   8): LLVMType<!cast<ValueType>("v"#8#ElemType)>,`.
  **L1159 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(veclen,   8): LLVMType<!cast<ValueType>("v"#8#ElemType)>,`。
- **L1160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(veclen,  16): LLVMType<!cast<ValueType>("v"#16#ElemType)>,`.
  **L1160 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(veclen,  16): LLVMType<!cast<ValueType>("v"#16#ElemType)>,`。
- **L1161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(veclen,  32): LLVMType<!cast<ValueType>("v"#32#ElemType)>,`.
  **L1161 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(veclen,  32): LLVMType<!cast<ValueType>("v"#32#ElemType)>,`。
- **L1162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(veclen,  64): LLVMType<!cast<ValueType>("v"#64#ElemType)>,`.
  **L1162 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(veclen,  64): LLVMType<!cast<ValueType>("v"#64#ElemType)>,`。
- **L1163 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(veclen, 128): LLVMType<!cast<ValueType>("v"#128#ElemType)>,`.
  **L1163 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(veclen, 128): LLVMType<!cast<ValueType>("v"#128#ElemType)>,`。
- **L1164 EN**: Executes a standalone statement or declaration: `true : llvm_void_ty);`.
  **L1164 CN**: 执行一条独立语句或声明：`true : llvm_void_ty);`。
- **L1165 EN**: Closes the current lexical scope or compound statement.
  **L1165 CN**: 结束当前词法作用域或复合语句块。
- **L1166 EN**: Blank line separating nearby declarations or logic blocks.
  **L1166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1167 EN**: Declares class `NVVM_TCGEN05_MMA_BASE<string`.
  **L1167 CN**: 声明 class `NVVM_TCGEN05_MMA_BASE<string`。
- **L1168 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `LLVMType a_operand_type = !if(!eq(Space, "tensor"),`.
  **L1168 CN**: 继续一个多行参数列表、初始化器或聚合项：`LLVMType a_operand_type = !if(!eq(Space, "tensor"),`。
- **L1169 EN**: Executes a standalone statement or declaration: `llvm_tmem_ptr_ty, llvm_i64_ty);`.
  **L1169 CN**: 执行一条独立语句或声明：`llvm_tmem_ptr_ty, llvm_i64_ty);`。
- **L1170 EN**: Continues logic associated with callable symbol `listconcat`.
  **L1170 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L1171 EN**: Continues the surrounding expression or declaration: `[llvm_tmem_ptr_ty, // d`.
  **L1171 CN**: 继续构造周围的表达式或声明：`[llvm_tmem_ptr_ty, // d`。
- **L1172 EN**: Continues the surrounding expression or declaration: `a_operand_type,   // a`.
  **L1172 CN**: 继续构造周围的表达式或声明：`a_operand_type,   // a`。
- **L1173 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty,      // b`.
  **L1173 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty,      // b`。
- **L1174 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,      // idesc`.
  **L1174 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,      // idesc`。
- **L1175 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],      // enable_input_d`.
  **L1175 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],      // enable_input_d`。
- **L1176 EN**: Continues the surrounding expression or declaration: `!if(!eq(IsSparse, 1), [llvm_tmem_ptr_ty], [])); // spmetadata`.
  **L1176 CN**: 继续构造周围的表达式或声明：`!if(!eq(IsSparse, 1), [llvm_tmem_ptr_ty], [])); // spmetadata`。
- **L1177 EN**: Continues logic associated with callable symbol `listconcat`.
  **L1177 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L1178 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, WriteOnly<ArgIndex<0>>],`.
  **L1178 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, WriteOnly<ArgIndex<0>>],`。
- **L1179 EN**: Continues the surrounding expression or declaration: `!if(!eq(Space, "tensor"), [ReadOnly<ArgIndex<1>>], [])`.
  **L1179 CN**: 继续构造周围的表达式或声明：`!if(!eq(Space, "tensor"), [ReadOnly<ArgIndex<1>>], [])`。
- **L1180 EN**: Executes a standalone statement or declaration: `);`.
  **L1180 CN**: 执行一条独立语句或声明：`);`。
- **L1181 EN**: Blank line separating nearby declarations or logic blocks.
  **L1181 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1182 EN**: Initializes variable `Prefix` from the right-hand expression.
  **L1182 CN**: 使用右侧表达式初始化变量 `Prefix`。
- **L1183 EN**: Continues the surrounding expression or declaration: `string SpSpaceKindStr = !if(IsSparse, ".sp", "")`.
  **L1183 CN**: 继续构造周围的表达式或声明：`string SpSpaceKindStr = !if(IsSparse, ".sp", "")`。
- **L1184 EN**: Continues the surrounding expression or declaration: `# "." # Space`.
  **L1184 CN**: 继续构造周围的表达式或声明：`# "." # Space`。

### Lines 1185-1216

````tablegen
                          # !if(!ne(Kind, ""), "." # Kind, "");
}

class NVVM_TCGEN05_MMA<bit IsSparse, string Space,
                       bit IsAShift, bit IsScaleInputD>:
        NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {
  string name = Prefix
                # SpSpaceKindStr
                # !if(IsScaleInputD, ".scale_d", "")
                # !if(IsAShift, ".ashift", "");
  string intr_name = IntrinsicName<name>.intr_name;
  string record_name = IntrinsicName<name>.record_name;
}

class NVVM_TCGEN05_MMA_BLOCKSCALE<bit IsSparse, string Space,
                                  string Kind, string ScaleVecSize>:
        NVVM_TCGEN05_MMA_BASE<Space, IsSparse, Kind> {
  string name = Prefix
                # SpSpaceKindStr
                # ".block_scale" # ScaleVecSize;
  string intr_name = IntrinsicName<name>.intr_name;
  string record_name = IntrinsicName<name>.record_name;
}

class NVVM_TCGEN05_MMA_WS<bit IsSparse, string Space, bit IsZeroColMask>:
        NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {
  string name = Prefix # ".ws"
                # SpSpaceKindStr
                # !if(IsZeroColMask, ".zero_col_mask", "");
  string intr_name = IntrinsicName<name>.intr_name;
  string record_name = IntrinsicName<name>.record_name;
}
````
- **L1185 EN**: Executes a call or declaration centered on `!if`.
  **L1185 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1186 EN**: Closes the current lexical scope or compound statement.
  **L1186 CN**: 结束当前词法作用域或复合语句块。
- **L1187 EN**: Blank line separating nearby declarations or logic blocks.
  **L1187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1188 EN**: Declares class `NVVM_TCGEN05_MMA<bit`.
  **L1188 CN**: 声明 class `NVVM_TCGEN05_MMA<bit`。
- **L1189 EN**: Continues the surrounding expression or declaration: `bit IsAShift, bit IsScaleInputD>:`.
  **L1189 CN**: 继续构造周围的表达式或声明：`bit IsAShift, bit IsScaleInputD>:`。
- **L1190 EN**: Continues the surrounding expression or declaration: `NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {`.
  **L1190 CN**: 继续构造周围的表达式或声明：`NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {`。
- **L1191 EN**: Continues the surrounding expression or declaration: `string name = Prefix`.
  **L1191 CN**: 继续构造周围的表达式或声明：`string name = Prefix`。
- **L1192 EN**: Continues the surrounding expression or declaration: `# SpSpaceKindStr`.
  **L1192 CN**: 继续构造周围的表达式或声明：`# SpSpaceKindStr`。
- **L1193 EN**: Continues the surrounding expression or declaration: `# !if(IsScaleInputD, ".scale_d", "")`.
  **L1193 CN**: 继续构造周围的表达式或声明：`# !if(IsScaleInputD, ".scale_d", "")`。
- **L1194 EN**: Executes a call or declaration centered on `!if`.
  **L1194 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1195 EN**: Initializes variable `intr_name` from the right-hand expression.
  **L1195 CN**: 使用右侧表达式初始化变量 `intr_name`。
- **L1196 EN**: Initializes variable `record_name` from the right-hand expression.
  **L1196 CN**: 使用右侧表达式初始化变量 `record_name`。
- **L1197 EN**: Closes the current lexical scope or compound statement.
  **L1197 CN**: 结束当前词法作用域或复合语句块。
- **L1198 EN**: Blank line separating nearby declarations or logic blocks.
  **L1198 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1199 EN**: Declares class `NVVM_TCGEN05_MMA_BLOCKSCALE<bit`.
  **L1199 CN**: 声明 class `NVVM_TCGEN05_MMA_BLOCKSCALE<bit`。
- **L1200 EN**: Continues the surrounding expression or declaration: `string Kind, string ScaleVecSize>:`.
  **L1200 CN**: 继续构造周围的表达式或声明：`string Kind, string ScaleVecSize>:`。
- **L1201 EN**: Continues the surrounding expression or declaration: `NVVM_TCGEN05_MMA_BASE<Space, IsSparse, Kind> {`.
  **L1201 CN**: 继续构造周围的表达式或声明：`NVVM_TCGEN05_MMA_BASE<Space, IsSparse, Kind> {`。
- **L1202 EN**: Continues the surrounding expression or declaration: `string name = Prefix`.
  **L1202 CN**: 继续构造周围的表达式或声明：`string name = Prefix`。
- **L1203 EN**: Continues the surrounding expression or declaration: `# SpSpaceKindStr`.
  **L1203 CN**: 继续构造周围的表达式或声明：`# SpSpaceKindStr`。
- **L1204 EN**: Executes a standalone statement or declaration: `# ".block_scale" # ScaleVecSize;`.
  **L1204 CN**: 执行一条独立语句或声明：`# ".block_scale" # ScaleVecSize;`。
- **L1205 EN**: Initializes variable `intr_name` from the right-hand expression.
  **L1205 CN**: 使用右侧表达式初始化变量 `intr_name`。
- **L1206 EN**: Initializes variable `record_name` from the right-hand expression.
  **L1206 CN**: 使用右侧表达式初始化变量 `record_name`。
- **L1207 EN**: Closes the current lexical scope or compound statement.
  **L1207 CN**: 结束当前词法作用域或复合语句块。
- **L1208 EN**: Blank line separating nearby declarations or logic blocks.
  **L1208 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1209 EN**: Declares class `NVVM_TCGEN05_MMA_WS<bit`.
  **L1209 CN**: 声明 class `NVVM_TCGEN05_MMA_WS<bit`。
- **L1210 EN**: Continues the surrounding expression or declaration: `NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {`.
  **L1210 CN**: 继续构造周围的表达式或声明：`NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {`。
- **L1211 EN**: Continues the surrounding expression or declaration: `string name = Prefix # ".ws"`.
  **L1211 CN**: 继续构造周围的表达式或声明：`string name = Prefix # ".ws"`。
- **L1212 EN**: Continues the surrounding expression or declaration: `# SpSpaceKindStr`.
  **L1212 CN**: 继续构造周围的表达式或声明：`# SpSpaceKindStr`。
- **L1213 EN**: Executes a call or declaration centered on `!if`.
  **L1213 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1214 EN**: Initializes variable `intr_name` from the right-hand expression.
  **L1214 CN**: 使用右侧表达式初始化变量 `intr_name`。
- **L1215 EN**: Initializes variable `record_name` from the right-hand expression.
  **L1215 CN**: 使用右侧表达式初始化变量 `record_name`。
- **L1216 EN**: Closes the current lexical scope or compound statement.
  **L1216 CN**: 结束当前词法作用域或复合语句块。

### Lines 1217-1248

````tablegen

class NVVM_TCGEN05_MMA_DISABLE_OUTPUT_LANE<bit IsSparse, string Space,
                                           int CtaGroup, bit IsAShift,
                                           bit IsScaleInputD>:
        NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {
  string name = Prefix
                # SpSpaceKindStr
                # !if(IsScaleInputD, ".scale_d", "")
                # ".disable_output_lane.cg" # CtaGroup
                # !if(IsAShift, ".ashift", "");
  string intr_name = IntrinsicName<name>.intr_name;
  string record_name = IntrinsicName<name>.record_name;
}

class NVVM_TCGEN05_MMA_BLOCKSCALE_SUPPORTED<string Kind, string ScaleVecSize> {
  bit ret = !cond(
    !and(!eq(Kind, "mxf8f6f4"), !eq(ScaleVecSize, ""))         : true,
    !and(!eq(Kind, "mxf4"),     !eq(ScaleVecSize, ""))         : true,
    !and(!eq(Kind, "mxf4nvf4"), !eq(ScaleVecSize, ".block16")) : true,
    !and(!eq(Kind, "mxf4"),     !eq(ScaleVecSize, ".block32")) : true,
    !and(!eq(Kind, "mxf4nvf4"), !eq(ScaleVecSize, ".block32")) : true,
    !and(!eq(Kind, "mxf8f6f4"), !eq(ScaleVecSize, ".block32")) : true,
    true: false
  );
}

class TexVector<string name, list<LLVMType> types> {
  string Name = name;
  list<LLVMType> Types = types;
}

def TV_I8 : TexVector<"i8", [llvm_i16_ty]>;
````
- **L1217 EN**: Blank line separating nearby declarations or logic blocks.
  **L1217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1218 EN**: Declares class `NVVM_TCGEN05_MMA_DISABLE_OUTPUT_LANE<bit`.
  **L1218 CN**: 声明 class `NVVM_TCGEN05_MMA_DISABLE_OUTPUT_LANE<bit`。
- **L1219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int CtaGroup, bit IsAShift,`.
  **L1219 CN**: 继续一个多行参数列表、初始化器或聚合项：`int CtaGroup, bit IsAShift,`。
- **L1220 EN**: Continues the surrounding expression or declaration: `bit IsScaleInputD>:`.
  **L1220 CN**: 继续构造周围的表达式或声明：`bit IsScaleInputD>:`。
- **L1221 EN**: Continues the surrounding expression or declaration: `NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {`.
  **L1221 CN**: 继续构造周围的表达式或声明：`NVVM_TCGEN05_MMA_BASE<Space, IsSparse> {`。
- **L1222 EN**: Continues the surrounding expression or declaration: `string name = Prefix`.
  **L1222 CN**: 继续构造周围的表达式或声明：`string name = Prefix`。
- **L1223 EN**: Continues the surrounding expression or declaration: `# SpSpaceKindStr`.
  **L1223 CN**: 继续构造周围的表达式或声明：`# SpSpaceKindStr`。
- **L1224 EN**: Continues the surrounding expression or declaration: `# !if(IsScaleInputD, ".scale_d", "")`.
  **L1224 CN**: 继续构造周围的表达式或声明：`# !if(IsScaleInputD, ".scale_d", "")`。
- **L1225 EN**: Continues the surrounding expression or declaration: `# ".disable_output_lane.cg" # CtaGroup`.
  **L1225 CN**: 继续构造周围的表达式或声明：`# ".disable_output_lane.cg" # CtaGroup`。
- **L1226 EN**: Executes a call or declaration centered on `!if`.
  **L1226 CN**: 执行以 `!if` 为核心的调用或声明。
- **L1227 EN**: Initializes variable `intr_name` from the right-hand expression.
  **L1227 CN**: 使用右侧表达式初始化变量 `intr_name`。
- **L1228 EN**: Initializes variable `record_name` from the right-hand expression.
  **L1228 CN**: 使用右侧表达式初始化变量 `record_name`。
- **L1229 EN**: Closes the current lexical scope or compound statement.
  **L1229 CN**: 结束当前词法作用域或复合语句块。
- **L1230 EN**: Blank line separating nearby declarations or logic blocks.
  **L1230 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1231 EN**: Declares class `NVVM_TCGEN05_MMA_BLOCKSCALE_SUPPORTED<string`.
  **L1231 CN**: 声明 class `NVVM_TCGEN05_MMA_BLOCKSCALE_SUPPORTED<string`。
- **L1232 EN**: Continues logic associated with callable symbol `cond`.
  **L1232 CN**: 继续与可调用符号 `cond` 相关的逻辑。
- **L1233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(Kind, "mxf8f6f4"), !eq(ScaleVecSize, ""))         : true,`.
  **L1233 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(Kind, "mxf8f6f4"), !eq(ScaleVecSize, ""))         : true,`。
- **L1234 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(Kind, "mxf4"),     !eq(ScaleVecSize, ""))         : true,`.
  **L1234 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(Kind, "mxf4"),     !eq(ScaleVecSize, ""))         : true,`。
- **L1235 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(Kind, "mxf4nvf4"), !eq(ScaleVecSize, ".block16")) : true,`.
  **L1235 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(Kind, "mxf4nvf4"), !eq(ScaleVecSize, ".block16")) : true,`。
- **L1236 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(Kind, "mxf4"),     !eq(ScaleVecSize, ".block32")) : true,`.
  **L1236 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(Kind, "mxf4"),     !eq(ScaleVecSize, ".block32")) : true,`。
- **L1237 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(Kind, "mxf4nvf4"), !eq(ScaleVecSize, ".block32")) : true,`.
  **L1237 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(Kind, "mxf4nvf4"), !eq(ScaleVecSize, ".block32")) : true,`。
- **L1238 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(Kind, "mxf8f6f4"), !eq(ScaleVecSize, ".block32")) : true,`.
  **L1238 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(Kind, "mxf8f6f4"), !eq(ScaleVecSize, ".block32")) : true,`。
- **L1239 EN**: Continues the surrounding expression or declaration: `true: false`.
  **L1239 CN**: 继续构造周围的表达式或声明：`true: false`。
- **L1240 EN**: Executes a standalone statement or declaration: `);`.
  **L1240 CN**: 执行一条独立语句或声明：`);`。
- **L1241 EN**: Closes the current lexical scope or compound statement.
  **L1241 CN**: 结束当前词法作用域或复合语句块。
- **L1242 EN**: Blank line separating nearby declarations or logic blocks.
  **L1242 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1243 EN**: Declares class `TexVector<string`.
  **L1243 CN**: 声明 class `TexVector<string`。
- **L1244 EN**: Initializes variable `Name` from the right-hand expression.
  **L1244 CN**: 使用右侧表达式初始化变量 `Name`。
- **L1245 EN**: Initializes variable `Types` from the right-hand expression.
  **L1245 CN**: 使用右侧表达式初始化变量 `Types`。
- **L1246 EN**: Closes the current lexical scope or compound statement.
  **L1246 CN**: 结束当前词法作用域或复合语句块。
- **L1247 EN**: Blank line separating nearby declarations or logic blocks.
  **L1247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1248 EN**: Declares TableGen def `TV_I8`.
  **L1248 CN**: 声明 TableGen def `TV_I8`。

### Lines 1249-1280

````tablegen
def TV_I16 : TexVector<"i16", [llvm_i16_ty]>;
def TV_I32 : TexVector<"i32", [llvm_i32_ty]>;
def TV_I64 : TexVector<"i64", [llvm_i64_ty]>;
def TV_V2I8 : TexVector<"v2i8", !listsplat(llvm_i16_ty, 2)>;
def TV_V2I16 : TexVector<"v2i16", !listsplat(llvm_i16_ty, 2)>;
def TV_V2I32 : TexVector<"v2i32", !listsplat(llvm_i32_ty, 2)>;
def TV_V2I64 : TexVector<"v2i64", !listsplat(llvm_i64_ty, 2)>;
def TV_V4I8 : TexVector<"v4i8", !listsplat(llvm_i16_ty, 4)>;
def TV_V4I16 : TexVector<"v4i16", !listsplat(llvm_i16_ty, 4)>;
def TV_V4I32 : TexVector<"v4i32", !listsplat(llvm_i32_ty, 4)>;


def V4F32 : TexVector<"v4f32", !listsplat(llvm_float_ty, 4)>;
def V4S32 : TexVector<"v4s32", !listsplat(llvm_i32_ty, 4)>;
def V4U32 : TexVector<"v4u32", !listsplat(llvm_i32_ty, 4)>;

class NVVMBuiltin :
  ClangBuiltin<!strconcat("__", !substr(NAME, !size("int_")))> {
    assert !eq(!substr(NAME, 0, !size("int_nvvm_")), "int_nvvm_"),
           "NVVMBuiltin must be a NVVM intrinsic starting with 'int_nvvm_'";
}

class NVVMPureIntrinsic<list<LLVMType> ret_types,
                    list<LLVMType> param_types = [],
                    list<IntrinsicProperty> intr_properties = [],
                    string name = ""> :
  DefaultAttrsIntrinsic<ret_types, param_types,
                        intr_properties # [IntrNoMem, IntrSpeculatable], name> {}

let TargetPrefix = "nvvm" in {

  //
````
- **L1249 EN**: Declares TableGen def `TV_I16`.
  **L1249 CN**: 声明 TableGen def `TV_I16`。
- **L1250 EN**: Declares TableGen def `TV_I32`.
  **L1250 CN**: 声明 TableGen def `TV_I32`。
- **L1251 EN**: Declares TableGen def `TV_I64`.
  **L1251 CN**: 声明 TableGen def `TV_I64`。
- **L1252 EN**: Declares TableGen def `TV_V2I8`.
  **L1252 CN**: 声明 TableGen def `TV_V2I8`。
- **L1253 EN**: Declares TableGen def `TV_V2I16`.
  **L1253 CN**: 声明 TableGen def `TV_V2I16`。
- **L1254 EN**: Declares TableGen def `TV_V2I32`.
  **L1254 CN**: 声明 TableGen def `TV_V2I32`。
- **L1255 EN**: Declares TableGen def `TV_V2I64`.
  **L1255 CN**: 声明 TableGen def `TV_V2I64`。
- **L1256 EN**: Declares TableGen def `TV_V4I8`.
  **L1256 CN**: 声明 TableGen def `TV_V4I8`。
- **L1257 EN**: Declares TableGen def `TV_V4I16`.
  **L1257 CN**: 声明 TableGen def `TV_V4I16`。
- **L1258 EN**: Declares TableGen def `TV_V4I32`.
  **L1258 CN**: 声明 TableGen def `TV_V4I32`。
- **L1259 EN**: Blank line separating nearby declarations or logic blocks.
  **L1259 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1260 EN**: Blank line separating nearby declarations or logic blocks.
  **L1260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1261 EN**: Declares TableGen def `V4F32`.
  **L1261 CN**: 声明 TableGen def `V4F32`。
- **L1262 EN**: Declares TableGen def `V4S32`.
  **L1262 CN**: 声明 TableGen def `V4S32`。
- **L1263 EN**: Declares TableGen def `V4U32`.
  **L1263 CN**: 声明 TableGen def `V4U32`。
- **L1264 EN**: Blank line separating nearby declarations or logic blocks.
  **L1264 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1265 EN**: Declares class `NVVMBuiltin`.
  **L1265 CN**: 声明 class `NVVMBuiltin`。
- **L1266 EN**: Starts a function, method, lambda, or structured scope: `ClangBuiltin<!strconcat("__", !substr(NAME, !size("int_")))> {`.
  **L1266 CN**: 开始一个函数、方法、lambda 或结构化作用域：`ClangBuiltin<!strconcat("__", !substr(NAME, !size("int_")))> {`。
- **L1267 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `assert !eq(!substr(NAME, 0, !size("int_nvvm_")), "int_nvvm_"),`.
  **L1267 CN**: 继续一个多行参数列表、初始化器或聚合项：`assert !eq(!substr(NAME, 0, !size("int_nvvm_")), "int_nvvm_"),`。
- **L1268 EN**: Executes a standalone statement or declaration: `"NVVMBuiltin must be a NVVM intrinsic starting with 'int_nvvm_'";`.
  **L1268 CN**: 执行一条独立语句或声明：`"NVVMBuiltin must be a NVVM intrinsic starting with 'int_nvvm_'";`。
- **L1269 EN**: Closes the current lexical scope or compound statement.
  **L1269 CN**: 结束当前词法作用域或复合语句块。
- **L1270 EN**: Blank line separating nearby declarations or logic blocks.
  **L1270 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1271 EN**: Declares class `NVVMPureIntrinsic<list<LLVMType>`.
  **L1271 CN**: 声明 class `NVVMPureIntrinsic<list<LLVMType>`。
- **L1272 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> param_types = [],`.
  **L1272 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> param_types = [],`。
- **L1273 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<IntrinsicProperty> intr_properties = [],`.
  **L1273 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<IntrinsicProperty> intr_properties = [],`。
- **L1274 EN**: Continues the surrounding expression or declaration: `string name = ""> :`.
  **L1274 CN**: 继续构造周围的表达式或声明：`string name = ""> :`。
- **L1275 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<ret_types, param_types,`.
  **L1275 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<ret_types, param_types,`。
- **L1276 EN**: Continues the surrounding expression or declaration: `intr_properties # [IntrNoMem, IntrSpeculatable], name> {}`.
  **L1276 CN**: 继续构造周围的表达式或声明：`intr_properties # [IntrNoMem, IntrSpeculatable], name> {}`。
- **L1277 EN**: Blank line separating nearby declarations or logic blocks.
  **L1277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1278 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1278 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1279 EN**: Blank line separating nearby declarations or logic blocks.
  **L1279 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1280 EN**: Separator comment used for visual grouping.
  **L1280 CN**: 用于视觉分组的分隔注释。

### Lines 1281-1312

````tablegen
  // PRMT - permute
  //
  def int_nvvm_prmt : NVVMBuiltin,
    PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

  foreach mode = ["f4e", "b4e"] in
    def int_nvvm_prmt_ # mode :
        PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

  // Note: these variants also have 2 source operands but only one will ever
  // be used so we eliminate the other operand in the IR (0 is used as the
  // placeholder in the backend).
  foreach mode = ["rc8", "ecl", "ecr", "rc16"] in
    def int_nvvm_prmt_ # mode :
        PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;

  //
  // Nanosleep
  //
  def int_nvvm_nanosleep : NVVMBuiltin,
      DefaultAttrsIntrinsic<[], [llvm_i32_ty],
                            [IntrConvergent, IntrNoMem, IntrHasSideEffects]>;

  //
  // Performance Monitor Events (pm events) intrinsics
  //
  def int_nvvm_pm_event_mask : NVVMBuiltin,
      DefaultAttrsIntrinsic<[], [llvm_i16_ty],
                [IntrConvergent, IntrNoMem, IntrHasSideEffects,
                 ImmArg<ArgIndex<0>>]>;
  //
  // Min Max
````
- **L1281 EN**: Comment explains nearby logic, invariants, or intent: `PRMT - permute`.
  **L1281 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`PRMT - permute`。
- **L1282 EN**: Separator comment used for visual grouping.
  **L1282 CN**: 用于视觉分组的分隔注释。
- **L1283 EN**: Declares TableGen def `int_nvvm_prmt`.
  **L1283 CN**: 声明 TableGen def `int_nvvm_prmt`。
- **L1284 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L1284 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L1285 EN**: Blank line separating nearby declarations or logic blocks.
  **L1285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1286 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1286 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1287 EN**: Declares TableGen def `int_nvvm_prmt_`.
  **L1287 CN**: 声明 TableGen def `int_nvvm_prmt_`。
- **L1288 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L1288 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L1289 EN**: Blank line separating nearby declarations or logic blocks.
  **L1289 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1290 EN**: Comment explains nearby logic, invariants, or intent: `Note: these variants also have 2 source operands but only one will ever`.
  **L1290 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: these variants also have 2 source operands but only one will ever`。
- **L1291 EN**: Comment explains nearby logic, invariants, or intent: `be used so we eliminate the other operand in the IR (0 is used as the`.
  **L1291 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`be used so we eliminate the other operand in the IR (0 is used as the`。
- **L1292 EN**: Comment explains nearby logic, invariants, or intent: `placeholder in the backend).`.
  **L1292 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`placeholder in the backend).`。
- **L1293 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1293 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1294 EN**: Declares TableGen def `int_nvvm_prmt_`.
  **L1294 CN**: 声明 TableGen def `int_nvvm_prmt_`。
- **L1295 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L1295 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L1296 EN**: Blank line separating nearby declarations or logic blocks.
  **L1296 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1297 EN**: Separator comment used for visual grouping.
  **L1297 CN**: 用于视觉分组的分隔注释。
- **L1298 EN**: Comment explains nearby logic, invariants, or intent: `Nanosleep`.
  **L1298 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Nanosleep`。
- **L1299 EN**: Separator comment used for visual grouping.
  **L1299 CN**: 用于视觉分组的分隔注释。
- **L1300 EN**: Declares TableGen def `int_nvvm_nanosleep`.
  **L1300 CN**: 声明 TableGen def `int_nvvm_nanosleep`。
- **L1301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i32_ty],`.
  **L1301 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i32_ty],`。
- **L1302 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrNoMem, IntrHasSideEffects]>;`.
  **L1302 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrNoMem, IntrHasSideEffects]>;`。
- **L1303 EN**: Blank line separating nearby declarations or logic blocks.
  **L1303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1304 EN**: Separator comment used for visual grouping.
  **L1304 CN**: 用于视觉分组的分隔注释。
- **L1305 EN**: Comment explains nearby logic, invariants, or intent: `Performance Monitor Events (pm events) intrinsics`.
  **L1305 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Performance Monitor Events (pm events) intrinsics`。
- **L1306 EN**: Separator comment used for visual grouping.
  **L1306 CN**: 用于视觉分组的分隔注释。
- **L1307 EN**: Declares TableGen def `int_nvvm_pm_event_mask`.
  **L1307 CN**: 声明 TableGen def `int_nvvm_pm_event_mask`。
- **L1308 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_i16_ty],`.
  **L1308 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_i16_ty],`。
- **L1309 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrNoMem, IntrHasSideEffects,`.
  **L1309 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrNoMem, IntrHasSideEffects,`。
- **L1310 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>]>;`.
  **L1310 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>]>;`。
- **L1311 EN**: Separator comment used for visual grouping.
  **L1311 CN**: 用于视觉分组的分隔注释。
- **L1312 EN**: Comment explains nearby logic, invariants, or intent: `Min Max`.
  **L1312 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Min Max`。

### Lines 1313-1344

````tablegen
  //
  let IntrProperties = [IntrNoMem, IntrSpeculatable, Commutative,
                        IntrNoCreateUndefOrPoison] in {
    foreach operation = ["min", "max"] in {
      def int_nvvm_f # operation # _d : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;

      foreach variant = ["", "_xorsign_abs"] in {
        foreach nan = ["", "_nan"] in {
          foreach ftz = ["", "_ftz"] in {
            def int_nvvm_f # operation # ftz # nan # variant # _f : NVVMBuiltin,
              DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;

            def int_nvvm_f # operation # ftz # nan # variant # _f16 :
              DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;

            def int_nvvm_f # operation # ftz # nan # variant # _f16x2 :
              DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;

            def int_nvvm_f # operation # ftz # nan # variant # _bf16 : NVVMBuiltin,
              DefaultAttrsIntrinsic<[llvm_bfloat_ty], [llvm_bfloat_ty, llvm_bfloat_ty]>;

            def int_nvvm_f # operation # ftz # nan # variant # _bf16x2 : NVVMBuiltin,
              DefaultAttrsIntrinsic<[llvm_v2bf16_ty], [llvm_v2bf16_ty, llvm_v2bf16_ty]>;
          } // ftz
        } // nan
      } // variant
    } // operation
  }

  //
  // Multiplication
````
- **L1313 EN**: Separator comment used for visual grouping.
  **L1313 CN**: 用于视觉分组的分隔注释。
- **L1314 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1314 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1315 EN**: Continues the surrounding expression or declaration: `IntrNoCreateUndefOrPoison] in {`.
  **L1315 CN**: 继续构造周围的表达式或声明：`IntrNoCreateUndefOrPoison] in {`。
- **L1316 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1316 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1317 EN**: Declares TableGen def `int_nvvm_f`.
  **L1317 CN**: 声明 TableGen def `int_nvvm_f`。
- **L1318 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`.
  **L1318 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`。
- **L1319 EN**: Blank line separating nearby declarations or logic blocks.
  **L1319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1320 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1320 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1321 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1321 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1322 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1322 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1323 EN**: Declares TableGen def `int_nvvm_f`.
  **L1323 CN**: 声明 TableGen def `int_nvvm_f`。
- **L1324 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1324 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1325 EN**: Blank line separating nearby declarations or logic blocks.
  **L1325 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1326 EN**: Declares TableGen def `int_nvvm_f`.
  **L1326 CN**: 声明 TableGen def `int_nvvm_f`。
- **L1327 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;`.
  **L1327 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;`。
- **L1328 EN**: Blank line separating nearby declarations or logic blocks.
  **L1328 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1329 EN**: Declares TableGen def `int_nvvm_f`.
  **L1329 CN**: 声明 TableGen def `int_nvvm_f`。
- **L1330 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;`.
  **L1330 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;`。
- **L1331 EN**: Blank line separating nearby declarations or logic blocks.
  **L1331 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1332 EN**: Declares TableGen def `int_nvvm_f`.
  **L1332 CN**: 声明 TableGen def `int_nvvm_f`。
- **L1333 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_bfloat_ty], [llvm_bfloat_ty, llvm_bfloat_ty]>;`.
  **L1333 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_bfloat_ty], [llvm_bfloat_ty, llvm_bfloat_ty]>;`。
- **L1334 EN**: Blank line separating nearby declarations or logic blocks.
  **L1334 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1335 EN**: Declares TableGen def `int_nvvm_f`.
  **L1335 CN**: 声明 TableGen def `int_nvvm_f`。
- **L1336 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2bf16_ty], [llvm_v2bf16_ty, llvm_v2bf16_ty]>;`.
  **L1336 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2bf16_ty], [llvm_v2bf16_ty, llvm_v2bf16_ty]>;`。
- **L1337 EN**: Continues the surrounding expression or declaration: `} // ftz`.
  **L1337 CN**: 继续构造周围的表达式或声明：`} // ftz`。
- **L1338 EN**: Continues the surrounding expression or declaration: `} // nan`.
  **L1338 CN**: 继续构造周围的表达式或声明：`} // nan`。
- **L1339 EN**: Continues the surrounding expression or declaration: `} // variant`.
  **L1339 CN**: 继续构造周围的表达式或声明：`} // variant`。
- **L1340 EN**: Continues the surrounding expression or declaration: `} // operation`.
  **L1340 CN**: 继续构造周围的表达式或声明：`} // operation`。
- **L1341 EN**: Closes the current lexical scope or compound statement.
  **L1341 CN**: 结束当前词法作用域或复合语句块。
- **L1342 EN**: Blank line separating nearby declarations or logic blocks.
  **L1342 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1343 EN**: Separator comment used for visual grouping.
  **L1343 CN**: 用于视觉分组的分隔注释。
- **L1344 EN**: Comment explains nearby logic, invariants, or intent: `Multiplication`.
  **L1344 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Multiplication`。

### Lines 1345-1376

````tablegen
  //
  let IntrProperties = [IntrNoMem, IntrSpeculatable, Commutative, 
                        IntrNoCreateUndefOrPoison] in {
    foreach sign = ["", "u"] in {
      def int_nvvm_mulhi_ # sign # s : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_i16_ty], [llvm_i16_ty, llvm_i16_ty]>;

      def int_nvvm_mulhi_ # sign # i : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;

      def int_nvvm_mulhi_ # sign # ll : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty]>;

      def int_nvvm_mul24_ # sign # i : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;
    }

    foreach rnd = ["rn", "rz", "rm", "rp"] in {
      foreach ftz = ["", "_ftz"] in
        def int_nvvm_mul_ # rnd # ftz # _f : NVVMBuiltin,
            DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;

      def int_nvvm_mul_ # rnd # _d : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;
    }
    
    foreach ftz = ["", "_ftz"] in {
      def int_nvvm_mul_rn # ftz # _sat_f16 : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;

      def int_nvvm_mul_rn # ftz # _sat_v2f16 : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;
````
- **L1345 EN**: Separator comment used for visual grouping.
  **L1345 CN**: 用于视觉分组的分隔注释。
- **L1346 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1346 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1347 EN**: Continues the surrounding expression or declaration: `IntrNoCreateUndefOrPoison] in {`.
  **L1347 CN**: 继续构造周围的表达式或声明：`IntrNoCreateUndefOrPoison] in {`。
- **L1348 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1348 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1349 EN**: Declares TableGen def `int_nvvm_mulhi_`.
  **L1349 CN**: 声明 TableGen def `int_nvvm_mulhi_`。
- **L1350 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i16_ty], [llvm_i16_ty, llvm_i16_ty]>;`.
  **L1350 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i16_ty], [llvm_i16_ty, llvm_i16_ty]>;`。
- **L1351 EN**: Blank line separating nearby declarations or logic blocks.
  **L1351 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1352 EN**: Declares TableGen def `int_nvvm_mulhi_`.
  **L1352 CN**: 声明 TableGen def `int_nvvm_mulhi_`。
- **L1353 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L1353 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L1354 EN**: Blank line separating nearby declarations or logic blocks.
  **L1354 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1355 EN**: Declares TableGen def `int_nvvm_mulhi_`.
  **L1355 CN**: 声明 TableGen def `int_nvvm_mulhi_`。
- **L1356 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty]>;`.
  **L1356 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty]>;`。
- **L1357 EN**: Blank line separating nearby declarations or logic blocks.
  **L1357 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1358 EN**: Declares TableGen def `int_nvvm_mul24_`.
  **L1358 CN**: 声明 TableGen def `int_nvvm_mul24_`。
- **L1359 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L1359 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L1360 EN**: Closes the current lexical scope or compound statement.
  **L1360 CN**: 结束当前词法作用域或复合语句块。
- **L1361 EN**: Blank line separating nearby declarations or logic blocks.
  **L1361 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1362 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1362 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1363 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1363 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1364 EN**: Declares TableGen def `int_nvvm_mul_`.
  **L1364 CN**: 声明 TableGen def `int_nvvm_mul_`。
- **L1365 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1365 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1366 EN**: Blank line separating nearby declarations or logic blocks.
  **L1366 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1367 EN**: Declares TableGen def `int_nvvm_mul_`.
  **L1367 CN**: 声明 TableGen def `int_nvvm_mul_`。
- **L1368 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`.
  **L1368 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`。
- **L1369 EN**: Closes the current lexical scope or compound statement.
  **L1369 CN**: 结束当前词法作用域或复合语句块。
- **L1370 EN**: Blank line separating nearby declarations or logic blocks.
  **L1370 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1371 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1371 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1372 EN**: Declares TableGen def `int_nvvm_mul_rn`.
  **L1372 CN**: 声明 TableGen def `int_nvvm_mul_rn`。
- **L1373 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;`.
  **L1373 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;`。
- **L1374 EN**: Blank line separating nearby declarations or logic blocks.
  **L1374 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1375 EN**: Declares TableGen def `int_nvvm_mul_rn`.
  **L1375 CN**: 声明 TableGen def `int_nvvm_mul_rn`。
- **L1376 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;`.
  **L1376 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;`。

### Lines 1377-1408

````tablegen
    } // ftz
  }

  //
  // Div
  //
  let IntrProperties = [IntrNoMem, IntrNoCreateUndefOrPoison] in {
    foreach ftz = ["", "_ftz"] in {
      def int_nvvm_div_approx # ftz # _f : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;

      def int_nvvm_div_full # ftz : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;
    }

    foreach rnd = ["rn", "rz", "rm", "rp"] in {
      foreach ftz = ["", "_ftz"] in
        def int_nvvm_div_ # rnd # ftz # _f : NVVMBuiltin,
            DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;

      def int_nvvm_div_ # rnd # _d : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;
    }
  }

  //
  // Sad - Sum of Absolute Differences
  //
  foreach sign = ["", "u"] in {
    def int_nvvm_sad_ # sign # s : NVVMBuiltin,
        PureIntrinsic<[llvm_i16_ty], [llvm_i16_ty, llvm_i16_ty, llvm_i16_ty]>;

````
- **L1377 EN**: Continues the surrounding expression or declaration: `} // ftz`.
  **L1377 CN**: 继续构造周围的表达式或声明：`} // ftz`。
- **L1378 EN**: Closes the current lexical scope or compound statement.
  **L1378 CN**: 结束当前词法作用域或复合语句块。
- **L1379 EN**: Blank line separating nearby declarations or logic blocks.
  **L1379 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1380 EN**: Separator comment used for visual grouping.
  **L1380 CN**: 用于视觉分组的分隔注释。
- **L1381 EN**: Comment explains nearby logic, invariants, or intent: `Div`.
  **L1381 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Div`。
- **L1382 EN**: Separator comment used for visual grouping.
  **L1382 CN**: 用于视觉分组的分隔注释。
- **L1383 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1383 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1384 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1384 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1385 EN**: Declares TableGen def `int_nvvm_div_approx`.
  **L1385 CN**: 声明 TableGen def `int_nvvm_div_approx`。
- **L1386 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1386 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1387 EN**: Blank line separating nearby declarations or logic blocks.
  **L1387 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1388 EN**: Declares TableGen def `int_nvvm_div_full`.
  **L1388 CN**: 声明 TableGen def `int_nvvm_div_full`。
- **L1389 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1389 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1390 EN**: Closes the current lexical scope or compound statement.
  **L1390 CN**: 结束当前词法作用域或复合语句块。
- **L1391 EN**: Blank line separating nearby declarations or logic blocks.
  **L1391 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1392 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1392 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1393 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1393 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1394 EN**: Declares TableGen def `int_nvvm_div_`.
  **L1394 CN**: 声明 TableGen def `int_nvvm_div_`。
- **L1395 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1395 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1396 EN**: Blank line separating nearby declarations or logic blocks.
  **L1396 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1397 EN**: Declares TableGen def `int_nvvm_div_`.
  **L1397 CN**: 声明 TableGen def `int_nvvm_div_`。
- **L1398 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`.
  **L1398 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`。
- **L1399 EN**: Closes the current lexical scope or compound statement.
  **L1399 CN**: 结束当前词法作用域或复合语句块。
- **L1400 EN**: Closes the current lexical scope or compound statement.
  **L1400 CN**: 结束当前词法作用域或复合语句块。
- **L1401 EN**: Blank line separating nearby declarations or logic blocks.
  **L1401 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1402 EN**: Separator comment used for visual grouping.
  **L1402 CN**: 用于视觉分组的分隔注释。
- **L1403 EN**: Comment explains nearby logic, invariants, or intent: `Sad - Sum of Absolute Differences`.
  **L1403 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sad - Sum of Absolute Differences`。
- **L1404 EN**: Separator comment used for visual grouping.
  **L1404 CN**: 用于视觉分组的分隔注释。
- **L1405 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1405 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1406 EN**: Declares TableGen def `int_nvvm_sad_`.
  **L1406 CN**: 声明 TableGen def `int_nvvm_sad_`。
- **L1407 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_i16_ty, llvm_i16_ty, llvm_i16_ty]>;`.
  **L1407 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_i16_ty, llvm_i16_ty, llvm_i16_ty]>;`。
- **L1408 EN**: Blank line separating nearby declarations or logic blocks.
  **L1408 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1409-1440

````tablegen
    def int_nvvm_sad_ # sign # i : NVVMBuiltin,
        PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

    def int_nvvm_sad_ # sign # ll : NVVMBuiltin,
        PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i64_ty]>;
  }

  //
  // Floor  Ceil
  //
  foreach op = ["floor", "ceil"] in {
    foreach ftz = ["", "_ftz"] in
      def int_nvvm_ # op # ftz # _f : NVVMBuiltin,
          PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;
    def int_nvvm_ # op # _d : NVVMBuiltin,
        PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;
  }

  //
  // Abs
  //
  foreach ftz = ["", "_ftz"] in
    def int_nvvm_fabs # ftz :
      PureIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>]>;

  //
  // Neg bf16, bf16x2
  //
  def int_nvvm_neg_bf16 : NVVMBuiltin,
    PureIntrinsic<[llvm_bfloat_ty], [llvm_bfloat_ty]>;
  def int_nvvm_neg_bf16x2 : NVVMBuiltin,
    PureIntrinsic<[llvm_v2bf16_ty], [llvm_v2bf16_ty]>;
````
- **L1409 EN**: Declares TableGen def `int_nvvm_sad_`.
  **L1409 CN**: 声明 TableGen def `int_nvvm_sad_`。
- **L1410 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L1410 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L1411 EN**: Blank line separating nearby declarations or logic blocks.
  **L1411 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1412 EN**: Declares TableGen def `int_nvvm_sad_`.
  **L1412 CN**: 声明 TableGen def `int_nvvm_sad_`。
- **L1413 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i64_ty]>;`.
  **L1413 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty, llvm_i64_ty]>;`。
- **L1414 EN**: Closes the current lexical scope or compound statement.
  **L1414 CN**: 结束当前词法作用域或复合语句块。
- **L1415 EN**: Blank line separating nearby declarations or logic blocks.
  **L1415 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1416 EN**: Separator comment used for visual grouping.
  **L1416 CN**: 用于视觉分组的分隔注释。
- **L1417 EN**: Comment explains nearby logic, invariants, or intent: `Floor  Ceil`.
  **L1417 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Floor  Ceil`。
- **L1418 EN**: Separator comment used for visual grouping.
  **L1418 CN**: 用于视觉分组的分隔注释。
- **L1419 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1419 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1420 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1420 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1421 EN**: Declares TableGen def `int_nvvm_`.
  **L1421 CN**: 声明 TableGen def `int_nvvm_`。
- **L1422 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1422 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1423 EN**: Declares TableGen def `int_nvvm_`.
  **L1423 CN**: 声明 TableGen def `int_nvvm_`。
- **L1424 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1424 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1425 EN**: Closes the current lexical scope or compound statement.
  **L1425 CN**: 结束当前词法作用域或复合语句块。
- **L1426 EN**: Blank line separating nearby declarations or logic blocks.
  **L1426 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1427 EN**: Separator comment used for visual grouping.
  **L1427 CN**: 用于视觉分组的分隔注释。
- **L1428 EN**: Comment explains nearby logic, invariants, or intent: `Abs`.
  **L1428 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Abs`。
- **L1429 EN**: Separator comment used for visual grouping.
  **L1429 CN**: 用于视觉分组的分隔注释。
- **L1430 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1430 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1431 EN**: Declares TableGen def `int_nvvm_fabs`.
  **L1431 CN**: 声明 TableGen def `int_nvvm_fabs`。
- **L1432 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>]>;`.
  **L1432 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>]>;`。
- **L1433 EN**: Blank line separating nearby declarations or logic blocks.
  **L1433 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1434 EN**: Separator comment used for visual grouping.
  **L1434 CN**: 用于视觉分组的分隔注释。
- **L1435 EN**: Comment explains nearby logic, invariants, or intent: `Neg bf16, bf16x2`.
  **L1435 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Neg bf16, bf16x2`。
- **L1436 EN**: Separator comment used for visual grouping.
  **L1436 CN**: 用于视觉分组的分隔注释。
- **L1437 EN**: Declares TableGen def `int_nvvm_neg_bf16`.
  **L1437 CN**: 声明 TableGen def `int_nvvm_neg_bf16`。
- **L1438 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_bfloat_ty], [llvm_bfloat_ty]>;`.
  **L1438 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_bfloat_ty], [llvm_bfloat_ty]>;`。
- **L1439 EN**: Declares TableGen def `int_nvvm_neg_bf16x2`.
  **L1439 CN**: 声明 TableGen def `int_nvvm_neg_bf16x2`。
- **L1440 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2bf16_ty], [llvm_v2bf16_ty]>;`.
  **L1440 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2bf16_ty], [llvm_v2bf16_ty]>;`。

### Lines 1441-1472

````tablegen

  //
  // Round
  //
  foreach ftz = ["", "_ftz"] in
    def int_nvvm_round # ftz # _f : NVVMBuiltin,
        PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;

  def int_nvvm_round_d : NVVMBuiltin,
      PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;

  //
  // Trunc
  //
  foreach ftz = ["", "_ftz"] in
    def int_nvvm_trunc # ftz # _f : NVVMBuiltin,
        PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;

  def int_nvvm_trunc_d : NVVMBuiltin,
      PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;

  //
  // Saturate
  //
  foreach ftz = ["", "_ftz"] in
    def int_nvvm_saturate # ftz # _f : NVVMBuiltin,
        PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;

  def int_nvvm_saturate_d : NVVMBuiltin,
      PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;

  //
````
- **L1441 EN**: Blank line separating nearby declarations or logic blocks.
  **L1441 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1442 EN**: Separator comment used for visual grouping.
  **L1442 CN**: 用于视觉分组的分隔注释。
- **L1443 EN**: Comment explains nearby logic, invariants, or intent: `Round`.
  **L1443 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Round`。
- **L1444 EN**: Separator comment used for visual grouping.
  **L1444 CN**: 用于视觉分组的分隔注释。
- **L1445 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1445 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1446 EN**: Declares TableGen def `int_nvvm_round`.
  **L1446 CN**: 声明 TableGen def `int_nvvm_round`。
- **L1447 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1447 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1448 EN**: Blank line separating nearby declarations or logic blocks.
  **L1448 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1449 EN**: Declares TableGen def `int_nvvm_round_d`.
  **L1449 CN**: 声明 TableGen def `int_nvvm_round_d`。
- **L1450 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1450 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1451 EN**: Blank line separating nearby declarations or logic blocks.
  **L1451 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1452 EN**: Separator comment used for visual grouping.
  **L1452 CN**: 用于视觉分组的分隔注释。
- **L1453 EN**: Comment explains nearby logic, invariants, or intent: `Trunc`.
  **L1453 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Trunc`。
- **L1454 EN**: Separator comment used for visual grouping.
  **L1454 CN**: 用于视觉分组的分隔注释。
- **L1455 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1455 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1456 EN**: Declares TableGen def `int_nvvm_trunc`.
  **L1456 CN**: 声明 TableGen def `int_nvvm_trunc`。
- **L1457 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1457 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1458 EN**: Blank line separating nearby declarations or logic blocks.
  **L1458 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1459 EN**: Declares TableGen def `int_nvvm_trunc_d`.
  **L1459 CN**: 声明 TableGen def `int_nvvm_trunc_d`。
- **L1460 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1460 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1461 EN**: Blank line separating nearby declarations or logic blocks.
  **L1461 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1462 EN**: Separator comment used for visual grouping.
  **L1462 CN**: 用于视觉分组的分隔注释。
- **L1463 EN**: Comment explains nearby logic, invariants, or intent: `Saturate`.
  **L1463 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Saturate`。
- **L1464 EN**: Separator comment used for visual grouping.
  **L1464 CN**: 用于视觉分组的分隔注释。
- **L1465 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1465 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1466 EN**: Declares TableGen def `int_nvvm_saturate`.
  **L1466 CN**: 声明 TableGen def `int_nvvm_saturate`。
- **L1467 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1467 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1468 EN**: Blank line separating nearby declarations or logic blocks.
  **L1468 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1469 EN**: Declares TableGen def `int_nvvm_saturate_d`.
  **L1469 CN**: 声明 TableGen def `int_nvvm_saturate_d`。
- **L1470 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1470 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1471 EN**: Blank line separating nearby declarations or logic blocks.
  **L1471 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1472 EN**: Separator comment used for visual grouping.
  **L1472 CN**: 用于视觉分组的分隔注释。

### Lines 1473-1504

````tablegen
  // Exp2  Log2
  //
  let IntrProperties = [IntrNoMem] in {
    foreach ftz = ["", "_ftz"] in
      def int_nvvm_ex2_approx # ftz :
          PureIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>]>;

    foreach ftz = ["", "_ftz"] in
      def int_nvvm_lg2_approx # ftz # _f : NVVMBuiltin,
          PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;

    def int_nvvm_lg2_approx_d : NVVMBuiltin,
        PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;
  }

  //
  // Sin  Cos
  //
  foreach op = ["sin", "cos"] in
    foreach ftz = ["", "_ftz"] in
      def int_nvvm_ # op # _approx # ftz # _f : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty],
                                [IntrNoMem, IntrNoCreateUndefOrPoison]>;

  //
  // Fma
  //
  foreach variant = ["", "_sat", "_relu"] in {
    foreach ftz = ["", "_ftz"] in {
      def int_nvvm_fma_rn # ftz # variant # _f16 :
        PureIntrinsic<[llvm_half_ty],
          [llvm_half_ty, llvm_half_ty, llvm_half_ty]>;
````
- **L1473 EN**: Comment explains nearby logic, invariants, or intent: `Exp2  Log2`.
  **L1473 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Exp2  Log2`。
- **L1474 EN**: Separator comment used for visual grouping.
  **L1474 CN**: 用于视觉分组的分隔注释。
- **L1475 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1475 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1476 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1476 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1477 EN**: Declares TableGen def `int_nvvm_ex2_approx`.
  **L1477 CN**: 声明 TableGen def `int_nvvm_ex2_approx`。
- **L1478 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>]>;`.
  **L1478 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_anyfloat_ty], [LLVMMatchType<0>]>;`。
- **L1479 EN**: Blank line separating nearby declarations or logic blocks.
  **L1479 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1480 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1480 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1481 EN**: Declares TableGen def `int_nvvm_lg2_approx`.
  **L1481 CN**: 声明 TableGen def `int_nvvm_lg2_approx`。
- **L1482 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1482 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1483 EN**: Blank line separating nearby declarations or logic blocks.
  **L1483 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1484 EN**: Declares TableGen def `int_nvvm_lg2_approx_d`.
  **L1484 CN**: 声明 TableGen def `int_nvvm_lg2_approx_d`。
- **L1485 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1485 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1486 EN**: Closes the current lexical scope or compound statement.
  **L1486 CN**: 结束当前词法作用域或复合语句块。
- **L1487 EN**: Blank line separating nearby declarations or logic blocks.
  **L1487 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1488 EN**: Separator comment used for visual grouping.
  **L1488 CN**: 用于视觉分组的分隔注释。
- **L1489 EN**: Comment explains nearby logic, invariants, or intent: `Sin  Cos`.
  **L1489 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sin  Cos`。
- **L1490 EN**: Separator comment used for visual grouping.
  **L1490 CN**: 用于视觉分组的分隔注释。
- **L1491 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1491 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1492 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1492 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1493 EN**: Declares TableGen def `int_nvvm_`.
  **L1493 CN**: 声明 TableGen def `int_nvvm_`。
- **L1494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty],`.
  **L1494 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty],`。
- **L1495 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrNoCreateUndefOrPoison]>;`.
  **L1495 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrNoCreateUndefOrPoison]>;`。
- **L1496 EN**: Blank line separating nearby declarations or logic blocks.
  **L1496 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1497 EN**: Separator comment used for visual grouping.
  **L1497 CN**: 用于视觉分组的分隔注释。
- **L1498 EN**: Comment explains nearby logic, invariants, or intent: `Fma`.
  **L1498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Fma`。
- **L1499 EN**: Separator comment used for visual grouping.
  **L1499 CN**: 用于视觉分组的分隔注释。
- **L1500 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1500 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1501 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1501 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1502 EN**: Declares TableGen def `int_nvvm_fma_rn`.
  **L1502 CN**: 声明 TableGen def `int_nvvm_fma_rn`。
- **L1503 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_half_ty],`.
  **L1503 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_half_ty],`。
- **L1504 EN**: Executes a standalone statement or declaration: `[llvm_half_ty, llvm_half_ty, llvm_half_ty]>;`.
  **L1504 CN**: 执行一条独立语句或声明：`[llvm_half_ty, llvm_half_ty, llvm_half_ty]>;`。

### Lines 1505-1536

````tablegen

      def int_nvvm_fma_rn # ftz # variant # _f16x2 :
        PureIntrinsic<[llvm_v2f16_ty],
          [llvm_v2f16_ty, llvm_v2f16_ty, llvm_v2f16_ty]>;
    } // ftz
  } // variant
  
  foreach relu = ["", "_relu"] in { 
    def int_nvvm_fma_rn # relu # _bf16 : NVVMBuiltin,
      PureIntrinsic<[llvm_bfloat_ty],
        [llvm_bfloat_ty, llvm_bfloat_ty, llvm_bfloat_ty]>;

    def int_nvvm_fma_rn # relu # _bf16x2 : NVVMBuiltin,
      PureIntrinsic<[llvm_v2bf16_ty],
        [llvm_v2bf16_ty, llvm_v2bf16_ty, llvm_v2bf16_ty]>;
    
    // oob (out-of-bounds) - clamps the result to 0 if either of the operand is 
    // an OOB NaN value.
    def int_nvvm_fma_rn_oob # relu : PureIntrinsic<[llvm_anyfloat_ty],
        [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;
  } // relu

  foreach rnd = ["_rn", "_rz", "_rm", "_rp"] in {
    foreach ftz = ["", "_ftz"] in {
      foreach sat = ["", "_sat"] in {
        def int_nvvm_fma # rnd # ftz # sat # _f : NVVMBuiltin,
          PureIntrinsic<[llvm_float_ty],
            [llvm_float_ty, llvm_float_ty, llvm_float_ty]>;
      } // sat
    } // ftz
    def int_nvvm_fma # rnd # _d : NVVMBuiltin,
      PureIntrinsic<[llvm_double_ty],
````
- **L1505 EN**: Blank line separating nearby declarations or logic blocks.
  **L1505 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1506 EN**: Declares TableGen def `int_nvvm_fma_rn`.
  **L1506 CN**: 声明 TableGen def `int_nvvm_fma_rn`。
- **L1507 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_v2f16_ty],`.
  **L1507 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_v2f16_ty],`。
- **L1508 EN**: Executes a standalone statement or declaration: `[llvm_v2f16_ty, llvm_v2f16_ty, llvm_v2f16_ty]>;`.
  **L1508 CN**: 执行一条独立语句或声明：`[llvm_v2f16_ty, llvm_v2f16_ty, llvm_v2f16_ty]>;`。
- **L1509 EN**: Continues the surrounding expression or declaration: `} // ftz`.
  **L1509 CN**: 继续构造周围的表达式或声明：`} // ftz`。
- **L1510 EN**: Continues the surrounding expression or declaration: `} // variant`.
  **L1510 CN**: 继续构造周围的表达式或声明：`} // variant`。
- **L1511 EN**: Blank line separating nearby declarations or logic blocks.
  **L1511 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1512 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1512 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1513 EN**: Declares TableGen def `int_nvvm_fma_rn`.
  **L1513 CN**: 声明 TableGen def `int_nvvm_fma_rn`。
- **L1514 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_bfloat_ty],`.
  **L1514 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_bfloat_ty],`。
- **L1515 EN**: Executes a standalone statement or declaration: `[llvm_bfloat_ty, llvm_bfloat_ty, llvm_bfloat_ty]>;`.
  **L1515 CN**: 执行一条独立语句或声明：`[llvm_bfloat_ty, llvm_bfloat_ty, llvm_bfloat_ty]>;`。
- **L1516 EN**: Blank line separating nearby declarations or logic blocks.
  **L1516 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1517 EN**: Declares TableGen def `int_nvvm_fma_rn`.
  **L1517 CN**: 声明 TableGen def `int_nvvm_fma_rn`。
- **L1518 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_v2bf16_ty],`.
  **L1518 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_v2bf16_ty],`。
- **L1519 EN**: Executes a standalone statement or declaration: `[llvm_v2bf16_ty, llvm_v2bf16_ty, llvm_v2bf16_ty]>;`.
  **L1519 CN**: 执行一条独立语句或声明：`[llvm_v2bf16_ty, llvm_v2bf16_ty, llvm_v2bf16_ty]>;`。
- **L1520 EN**: Blank line separating nearby declarations or logic blocks.
  **L1520 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1521 EN**: Comment explains nearby logic, invariants, or intent: `oob (out-of-bounds) - clamps the result to 0 if either of the operand is`.
  **L1521 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`oob (out-of-bounds) - clamps the result to 0 if either of the operand is`。
- **L1522 EN**: Comment explains nearby logic, invariants, or intent: `an OOB NaN value.`.
  **L1522 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`an OOB NaN value.`。
- **L1523 EN**: Declares TableGen def `int_nvvm_fma_rn_oob`.
  **L1523 CN**: 声明 TableGen def `int_nvvm_fma_rn_oob`。
- **L1524 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;`.
  **L1524 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;`。
- **L1525 EN**: Continues the surrounding expression or declaration: `} // relu`.
  **L1525 CN**: 继续构造周围的表达式或声明：`} // relu`。
- **L1526 EN**: Blank line separating nearby declarations or logic blocks.
  **L1526 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1527 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1527 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1528 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1528 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1529 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1529 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1530 EN**: Declares TableGen def `int_nvvm_fma`.
  **L1530 CN**: 声明 TableGen def `int_nvvm_fma`。
- **L1531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_float_ty],`.
  **L1531 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_float_ty],`。
- **L1532 EN**: Executes a standalone statement or declaration: `[llvm_float_ty, llvm_float_ty, llvm_float_ty]>;`.
  **L1532 CN**: 执行一条独立语句或声明：`[llvm_float_ty, llvm_float_ty, llvm_float_ty]>;`。
- **L1533 EN**: Continues the surrounding expression or declaration: `} // sat`.
  **L1533 CN**: 继续构造周围的表达式或声明：`} // sat`。
- **L1534 EN**: Continues the surrounding expression or declaration: `} // ftz`.
  **L1534 CN**: 继续构造周围的表达式或声明：`} // ftz`。
- **L1535 EN**: Declares TableGen def `int_nvvm_fma`.
  **L1535 CN**: 声明 TableGen def `int_nvvm_fma`。
- **L1536 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_double_ty],`.
  **L1536 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_double_ty],`。

### Lines 1537-1568

````tablegen
        [llvm_double_ty, llvm_double_ty, llvm_double_ty]>;
  } // rnd

  //
  // Rcp
  //
  let IntrProperties = [IntrNoMem, IntrNoCreateUndefOrPoison] in {
    foreach rnd = ["rn", "rz", "rm", "rp"] in {
      foreach ftz = ["", "_ftz"] in
        def int_nvvm_rcp_ # rnd # ftz # _f : NVVMBuiltin,
            DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;

      def int_nvvm_rcp_ # rnd # _d : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;
    }

    def int_nvvm_rcp_approx_ftz_f : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;
    def int_nvvm_rcp_approx_ftz_d : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;
  }

  //
  // Sqrt
  //
  let IntrProperties = [IntrNoMem, IntrNoCreateUndefOrPoison] in {
    foreach rnd = ["rn", "rz", "rm", "rp"] in {
      foreach ftz = ["", "_ftz"] in
        def int_nvvm_sqrt_ # rnd # ftz # _f : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;

      def int_nvvm_sqrt_ # rnd # _d : NVVMBuiltin,
````
- **L1537 EN**: Executes a standalone statement or declaration: `[llvm_double_ty, llvm_double_ty, llvm_double_ty]>;`.
  **L1537 CN**: 执行一条独立语句或声明：`[llvm_double_ty, llvm_double_ty, llvm_double_ty]>;`。
- **L1538 EN**: Continues the surrounding expression or declaration: `} // rnd`.
  **L1538 CN**: 继续构造周围的表达式或声明：`} // rnd`。
- **L1539 EN**: Blank line separating nearby declarations or logic blocks.
  **L1539 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1540 EN**: Separator comment used for visual grouping.
  **L1540 CN**: 用于视觉分组的分隔注释。
- **L1541 EN**: Comment explains nearby logic, invariants, or intent: `Rcp`.
  **L1541 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rcp`。
- **L1542 EN**: Separator comment used for visual grouping.
  **L1542 CN**: 用于视觉分组的分隔注释。
- **L1543 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1543 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1544 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1544 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1545 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1545 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1546 EN**: Declares TableGen def `int_nvvm_rcp_`.
  **L1546 CN**: 声明 TableGen def `int_nvvm_rcp_`。
- **L1547 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1547 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1548 EN**: Blank line separating nearby declarations or logic blocks.
  **L1548 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1549 EN**: Declares TableGen def `int_nvvm_rcp_`.
  **L1549 CN**: 声明 TableGen def `int_nvvm_rcp_`。
- **L1550 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1550 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1551 EN**: Closes the current lexical scope or compound statement.
  **L1551 CN**: 结束当前词法作用域或复合语句块。
- **L1552 EN**: Blank line separating nearby declarations or logic blocks.
  **L1552 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1553 EN**: Declares TableGen def `int_nvvm_rcp_approx_ftz_f`.
  **L1553 CN**: 声明 TableGen def `int_nvvm_rcp_approx_ftz_f`。
- **L1554 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1554 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1555 EN**: Declares TableGen def `int_nvvm_rcp_approx_ftz_d`.
  **L1555 CN**: 声明 TableGen def `int_nvvm_rcp_approx_ftz_d`。
- **L1556 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1556 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1557 EN**: Closes the current lexical scope or compound statement.
  **L1557 CN**: 结束当前词法作用域或复合语句块。
- **L1558 EN**: Blank line separating nearby declarations or logic blocks.
  **L1558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1559 EN**: Separator comment used for visual grouping.
  **L1559 CN**: 用于视觉分组的分隔注释。
- **L1560 EN**: Comment explains nearby logic, invariants, or intent: `Sqrt`.
  **L1560 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sqrt`。
- **L1561 EN**: Separator comment used for visual grouping.
  **L1561 CN**: 用于视觉分组的分隔注释。
- **L1562 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1562 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1563 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1563 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1564 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1564 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1565 EN**: Declares TableGen def `int_nvvm_sqrt_`.
  **L1565 CN**: 声明 TableGen def `int_nvvm_sqrt_`。
- **L1566 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1566 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1567 EN**: Blank line separating nearby declarations or logic blocks.
  **L1567 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1568 EN**: Declares TableGen def `int_nvvm_sqrt_`.
  **L1568 CN**: 声明 TableGen def `int_nvvm_sqrt_`。

### Lines 1569-1600

````tablegen
          DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;
    }

    def int_nvvm_sqrt_f : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;

    foreach ftz = ["", "_ftz"] in
      def int_nvvm_sqrt_approx # ftz # _f : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;
  }

  //
  // Rsqrt
  //
  let IntrProperties = [IntrNoMem, IntrNoCreateUndefOrPoison] in {
    foreach ftz = ["", "_ftz"] in {
      def int_nvvm_rsqrt_approx # ftz # _f : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;
      def int_nvvm_rsqrt_approx # ftz # _d : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;
    }
  }

  //
  // Add
  //

  let IntrProperties = [IntrNoMem, IntrSpeculatable, Commutative,
                        IntrNoCreateUndefOrPoison] in {
    foreach rnd = ["_rn", "_rz", "_rm", "_rp"] in {
      foreach ftz = ["", "_ftz"] in {
        foreach sat = ["", "_sat"] in {
````
- **L1569 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1569 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1570 EN**: Closes the current lexical scope or compound statement.
  **L1570 CN**: 结束当前词法作用域或复合语句块。
- **L1571 EN**: Blank line separating nearby declarations or logic blocks.
  **L1571 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1572 EN**: Declares TableGen def `int_nvvm_sqrt_f`.
  **L1572 CN**: 声明 TableGen def `int_nvvm_sqrt_f`。
- **L1573 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1573 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1574 EN**: Blank line separating nearby declarations or logic blocks.
  **L1574 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1575 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1575 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1576 EN**: Declares TableGen def `int_nvvm_sqrt_approx`.
  **L1576 CN**: 声明 TableGen def `int_nvvm_sqrt_approx`。
- **L1577 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1577 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1578 EN**: Closes the current lexical scope or compound statement.
  **L1578 CN**: 结束当前词法作用域或复合语句块。
- **L1579 EN**: Blank line separating nearby declarations or logic blocks.
  **L1579 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1580 EN**: Separator comment used for visual grouping.
  **L1580 CN**: 用于视觉分组的分隔注释。
- **L1581 EN**: Comment explains nearby logic, invariants, or intent: `Rsqrt`.
  **L1581 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Rsqrt`。
- **L1582 EN**: Separator comment used for visual grouping.
  **L1582 CN**: 用于视觉分组的分隔注释。
- **L1583 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1583 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1584 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1584 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1585 EN**: Declares TableGen def `int_nvvm_rsqrt_approx`.
  **L1585 CN**: 声明 TableGen def `int_nvvm_rsqrt_approx`。
- **L1586 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`.
  **L1586 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;`。
- **L1587 EN**: Declares TableGen def `int_nvvm_rsqrt_approx`.
  **L1587 CN**: 声明 TableGen def `int_nvvm_rsqrt_approx`。
- **L1588 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`.
  **L1588 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;`。
- **L1589 EN**: Closes the current lexical scope or compound statement.
  **L1589 CN**: 结束当前词法作用域或复合语句块。
- **L1590 EN**: Closes the current lexical scope or compound statement.
  **L1590 CN**: 结束当前词法作用域或复合语句块。
- **L1591 EN**: Blank line separating nearby declarations or logic blocks.
  **L1591 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1592 EN**: Separator comment used for visual grouping.
  **L1592 CN**: 用于视觉分组的分隔注释。
- **L1593 EN**: Comment explains nearby logic, invariants, or intent: `Add`.
  **L1593 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Add`。
- **L1594 EN**: Separator comment used for visual grouping.
  **L1594 CN**: 用于视觉分组的分隔注释。
- **L1595 EN**: Blank line separating nearby declarations or logic blocks.
  **L1595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1596 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1596 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1597 EN**: Continues the surrounding expression or declaration: `IntrNoCreateUndefOrPoison] in {`.
  **L1597 CN**: 继续构造周围的表达式或声明：`IntrNoCreateUndefOrPoison] in {`。
- **L1598 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1598 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1599 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1599 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1600 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1600 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1601-1632

````tablegen
          def int_nvvm_add # rnd # ftz # sat # _f : NVVMBuiltin,
            DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;
        } // sat
      } // ftz
      def int_nvvm_add # rnd # _d : NVVMBuiltin,
          DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;
    }
    
    foreach ftz = ["", "_ftz"] in {
      def int_nvvm_add_rn # ftz # _sat_f16 : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;

      def int_nvvm_add_rn # ftz # _sat_v2f16 : NVVMBuiltin,
        DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;
        
    } // ftz
  }

  //
  // Dot Product
  //
  foreach a_type = ["s", "u"] in {
    foreach b_type = ["s", "u"] in {
      def int_nvvm_idp4a_ # a_type # _ # b_type :
          PureIntrinsic<[llvm_i32_ty],
              [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;
      def int_nvvm_idp2a_ # a_type # _ # b_type :
          PureIntrinsic<[llvm_i32_ty],
            [llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],
            [ImmArg<ArgIndex<2>>]>;
    }
  }
````
- **L1601 EN**: Declares TableGen def `int_nvvm_add`.
  **L1601 CN**: 声明 TableGen def `int_nvvm_add`。
- **L1602 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1602 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1603 EN**: Continues the surrounding expression or declaration: `} // sat`.
  **L1603 CN**: 继续构造周围的表达式或声明：`} // sat`。
- **L1604 EN**: Continues the surrounding expression or declaration: `} // ftz`.
  **L1604 CN**: 继续构造周围的表达式或声明：`} // ftz`。
- **L1605 EN**: Declares TableGen def `int_nvvm_add`.
  **L1605 CN**: 声明 TableGen def `int_nvvm_add`。
- **L1606 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`.
  **L1606 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty, llvm_double_ty]>;`。
- **L1607 EN**: Closes the current lexical scope or compound statement.
  **L1607 CN**: 结束当前词法作用域或复合语句块。
- **L1608 EN**: Blank line separating nearby declarations or logic blocks.
  **L1608 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1609 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1609 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1610 EN**: Declares TableGen def `int_nvvm_add_rn`.
  **L1610 CN**: 声明 TableGen def `int_nvvm_add_rn`。
- **L1611 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;`.
  **L1611 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_half_ty], [llvm_half_ty, llvm_half_ty]>;`。
- **L1612 EN**: Blank line separating nearby declarations or logic blocks.
  **L1612 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1613 EN**: Declares TableGen def `int_nvvm_add_rn`.
  **L1613 CN**: 声明 TableGen def `int_nvvm_add_rn`。
- **L1614 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;`.
  **L1614 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_v2f16_ty], [llvm_v2f16_ty, llvm_v2f16_ty]>;`。
- **L1615 EN**: Blank line separating nearby declarations or logic blocks.
  **L1615 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1616 EN**: Continues the surrounding expression or declaration: `} // ftz`.
  **L1616 CN**: 继续构造周围的表达式或声明：`} // ftz`。
- **L1617 EN**: Closes the current lexical scope or compound statement.
  **L1617 CN**: 结束当前词法作用域或复合语句块。
- **L1618 EN**: Blank line separating nearby declarations or logic blocks.
  **L1618 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1619 EN**: Separator comment used for visual grouping.
  **L1619 CN**: 用于视觉分组的分隔注释。
- **L1620 EN**: Comment explains nearby logic, invariants, or intent: `Dot Product`.
  **L1620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Dot Product`。
- **L1621 EN**: Separator comment used for visual grouping.
  **L1621 CN**: 用于视觉分组的分隔注释。
- **L1622 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1622 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1623 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1623 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1624 EN**: Declares TableGen def `int_nvvm_idp4a_`.
  **L1624 CN**: 声明 TableGen def `int_nvvm_idp4a_`。
- **L1625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L1625 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L1626 EN**: Executes a standalone statement or declaration: `[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L1626 CN**: 执行一条独立语句或声明：`[llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L1627 EN**: Declares TableGen def `int_nvvm_idp2a_`.
  **L1627 CN**: 声明 TableGen def `int_nvvm_idp2a_`。
- **L1628 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty],`.
  **L1628 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty],`。
- **L1629 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],`.
  **L1629 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i32_ty, llvm_i32_ty, llvm_i1_ty, llvm_i32_ty],`。
- **L1630 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<2>>]>;`.
  **L1630 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<2>>]>;`。
- **L1631 EN**: Closes the current lexical scope or compound statement.
  **L1631 CN**: 结束当前词法作用域或复合语句块。
- **L1632 EN**: Closes the current lexical scope or compound statement.
  **L1632 CN**: 结束当前词法作用域或复合语句块。

### Lines 1633-1664

````tablegen

  //
  // Funnel-shift
  //
  foreach direction = ["l", "r"] in
    def int_nvvm_fsh # direction # _clamp :
      PureIntrinsic<[llvm_anyint_ty],
                    [LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;

  //
  // FLO - Find Leading One
  //
  foreach sign = ["s", "u"] in
    def int_nvvm_flo_ # sign :
      PureIntrinsic<[llvm_i32_ty], [llvm_anyint_ty, llvm_i1_ty],
                    [ImmArg<ArgIndex<1>>]>;

  //
  // szext
  //
  foreach ext = ["sext", "zext"] in
    foreach mode = ["wrap", "clamp"] in
      def int_nvvm_ # ext # _ # mode :
        PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;

  //
  // BMSK - bit mask
  //
  foreach mode = ["wrap", "clamp"] in
    def int_nvvm_bmsk_ # mode :
      PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;

````
- **L1633 EN**: Blank line separating nearby declarations or logic blocks.
  **L1633 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1634 EN**: Separator comment used for visual grouping.
  **L1634 CN**: 用于视觉分组的分隔注释。
- **L1635 EN**: Comment explains nearby logic, invariants, or intent: `Funnel-shift`.
  **L1635 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Funnel-shift`。
- **L1636 EN**: Separator comment used for visual grouping.
  **L1636 CN**: 用于视觉分组的分隔注释。
- **L1637 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1637 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1638 EN**: Declares TableGen def `int_nvvm_fsh`.
  **L1638 CN**: 声明 TableGen def `int_nvvm_fsh`。
- **L1639 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_anyint_ty],`.
  **L1639 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_anyint_ty],`。
- **L1640 EN**: Executes a standalone statement or declaration: `[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;`.
  **L1640 CN**: 执行一条独立语句或声明：`[LLVMMatchType<0>, LLVMMatchType<0>, LLVMMatchType<0>]>;`。
- **L1641 EN**: Blank line separating nearby declarations or logic blocks.
  **L1641 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1642 EN**: Separator comment used for visual grouping.
  **L1642 CN**: 用于视觉分组的分隔注释。
- **L1643 EN**: Comment explains nearby logic, invariants, or intent: `FLO - Find Leading One`.
  **L1643 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FLO - Find Leading One`。
- **L1644 EN**: Separator comment used for visual grouping.
  **L1644 CN**: 用于视觉分组的分隔注释。
- **L1645 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1645 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1646 EN**: Declares TableGen def `int_nvvm_flo_`.
  **L1646 CN**: 声明 TableGen def `int_nvvm_flo_`。
- **L1647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `PureIntrinsic<[llvm_i32_ty], [llvm_anyint_ty, llvm_i1_ty],`.
  **L1647 CN**: 继续一个多行参数列表、初始化器或聚合项：`PureIntrinsic<[llvm_i32_ty], [llvm_anyint_ty, llvm_i1_ty],`。
- **L1648 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]>;`.
  **L1648 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]>;`。
- **L1649 EN**: Blank line separating nearby declarations or logic blocks.
  **L1649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1650 EN**: Separator comment used for visual grouping.
  **L1650 CN**: 用于视觉分组的分隔注释。
- **L1651 EN**: Comment explains nearby logic, invariants, or intent: `szext`.
  **L1651 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`szext`。
- **L1652 EN**: Separator comment used for visual grouping.
  **L1652 CN**: 用于视觉分组的分隔注释。
- **L1653 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1653 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1654 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1654 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1655 EN**: Declares TableGen def `int_nvvm_`.
  **L1655 CN**: 声明 TableGen def `int_nvvm_`。
- **L1656 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L1656 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L1657 EN**: Blank line separating nearby declarations or logic blocks.
  **L1657 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1658 EN**: Separator comment used for visual grouping.
  **L1658 CN**: 用于视觉分组的分隔注释。
- **L1659 EN**: Comment explains nearby logic, invariants, or intent: `BMSK - bit mask`.
  **L1659 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`BMSK - bit mask`。
- **L1660 EN**: Separator comment used for visual grouping.
  **L1660 CN**: 用于视觉分组的分隔注释。
- **L1661 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1661 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1662 EN**: Declares TableGen def `int_nvvm_bmsk_`.
  **L1662 CN**: 声明 TableGen def `int_nvvm_bmsk_`。
- **L1663 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L1663 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L1664 EN**: Blank line separating nearby declarations or logic blocks.
  **L1664 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1665-1696

````tablegen
  //
  // FNS - Find the n-th set bit
  //
  def int_nvvm_fns : NVVMBuiltin,
      PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

  //
  // Convert
  //
  // TODO: All these intrinsics are defined as NVVMPureIntrinsic, this attaches the
  //       IntrSpeculatable property to them. Consider if some of these should
  //       have this attribute removed as they may be too expensive.
  //
  def int_nvvm_lohi_i2d : NVVMBuiltin,
      PureIntrinsic<[llvm_double_ty], [llvm_i32_ty, llvm_i32_ty]>;

  def int_nvvm_d2i_lo : NVVMBuiltin,
      PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;
  def int_nvvm_d2i_hi : NVVMBuiltin,
      PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;

  foreach rnd = ["rn", "rz", "rm", "rp"] in {
    foreach ftz = ["", "_ftz"] in
      def int_nvvm_d2f_ # rnd # ftz : NVVMBuiltin,
          PureIntrinsic<[llvm_float_ty], [llvm_double_ty]>;

    foreach sign = ["", "u"] in {

      def int_nvvm_d2 # sign # i_ # rnd : NVVMBuiltin,
          PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;

      def int_nvvm_ # sign # i2d_ # rnd : NVVMBuiltin,
````
- **L1665 EN**: Separator comment used for visual grouping.
  **L1665 CN**: 用于视觉分组的分隔注释。
- **L1666 EN**: Comment explains nearby logic, invariants, or intent: `FNS - Find the n-th set bit`.
  **L1666 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FNS - Find the n-th set bit`。
- **L1667 EN**: Separator comment used for visual grouping.
  **L1667 CN**: 用于视觉分组的分隔注释。
- **L1668 EN**: Declares TableGen def `int_nvvm_fns`.
  **L1668 CN**: 声明 TableGen def `int_nvvm_fns`。
- **L1669 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L1669 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L1670 EN**: Blank line separating nearby declarations or logic blocks.
  **L1670 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1671 EN**: Separator comment used for visual grouping.
  **L1671 CN**: 用于视觉分组的分隔注释。
- **L1672 EN**: Comment explains nearby logic, invariants, or intent: `Convert`.
  **L1672 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Convert`。
- **L1673 EN**: Separator comment used for visual grouping.
  **L1673 CN**: 用于视觉分组的分隔注释。
- **L1674 EN**: Comment records a pending task or caution: `TODO: All these intrinsics are defined as NVVMPureIntrinsic, this attaches the`.
  **L1674 CN**: 注释记录了待办事项或注意点：`TODO: All these intrinsics are defined as NVVMPureIntrinsic, this attaches the`。
- **L1675 EN**: Comment explains nearby logic, invariants, or intent: `IntrSpeculatable property to them. Consider if some of these should`.
  **L1675 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`IntrSpeculatable property to them. Consider if some of these should`。
- **L1676 EN**: Comment explains nearby logic, invariants, or intent: `have this attribute removed as they may be too expensive.`.
  **L1676 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`have this attribute removed as they may be too expensive.`。
- **L1677 EN**: Separator comment used for visual grouping.
  **L1677 CN**: 用于视觉分组的分隔注释。
- **L1678 EN**: Declares TableGen def `int_nvvm_lohi_i2d`.
  **L1678 CN**: 声明 TableGen def `int_nvvm_lohi_i2d`。
- **L1679 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L1679 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L1680 EN**: Blank line separating nearby declarations or logic blocks.
  **L1680 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1681 EN**: Declares TableGen def `int_nvvm_d2i_lo`.
  **L1681 CN**: 声明 TableGen def `int_nvvm_d2i_lo`。
- **L1682 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;`.
  **L1682 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;`。
- **L1683 EN**: Declares TableGen def `int_nvvm_d2i_hi`.
  **L1683 CN**: 声明 TableGen def `int_nvvm_d2i_hi`。
- **L1684 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;`.
  **L1684 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;`。
- **L1685 EN**: Blank line separating nearby declarations or logic blocks.
  **L1685 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1686 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1686 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1687 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1687 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1688 EN**: Declares TableGen def `int_nvvm_d2f_`.
  **L1688 CN**: 声明 TableGen def `int_nvvm_d2f_`。
- **L1689 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_double_ty]>;`.
  **L1689 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_double_ty]>;`。
- **L1690 EN**: Blank line separating nearby declarations or logic blocks.
  **L1690 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1691 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1691 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1692 EN**: Blank line separating nearby declarations or logic blocks.
  **L1692 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1693 EN**: Declares TableGen def `int_nvvm_d2`.
  **L1693 CN**: 声明 TableGen def `int_nvvm_d2`。
- **L1694 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;`.
  **L1694 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_double_ty]>;`。
- **L1695 EN**: Blank line separating nearby declarations or logic blocks.
  **L1695 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1696 EN**: Declares TableGen def `int_nvvm_`.
  **L1696 CN**: 声明 TableGen def `int_nvvm_`。

### Lines 1697-1728

````tablegen
        PureIntrinsic<[llvm_double_ty], [llvm_i32_ty]>;

      foreach ftz = ["", "_ftz"] in
        def int_nvvm_f2 # sign # i_ # rnd # ftz : NVVMBuiltin,
            PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;

      def int_nvvm_ # sign # i2f_ # rnd : NVVMBuiltin,
          PureIntrinsic<[llvm_float_ty], [llvm_i32_ty]>;

      foreach ftz = ["", "_ftz"] in
        def int_nvvm_f2 # sign # ll_ # rnd # ftz : NVVMBuiltin,
            PureIntrinsic<[llvm_i64_ty], [llvm_float_ty]>;

      def int_nvvm_d2 # sign # ll_ # rnd : NVVMBuiltin,
        PureIntrinsic<[llvm_i64_ty], [llvm_double_ty]>;

      def int_nvvm_ # sign # ll2f_ # rnd : NVVMBuiltin,
          PureIntrinsic<[llvm_float_ty], [llvm_i64_ty]>;

      def int_nvvm_ # sign # ll2d_ # rnd : NVVMBuiltin,
          PureIntrinsic<[llvm_double_ty], [llvm_i64_ty]>;

    } // sign
  } // rnd

  foreach ftz = ["", "_ftz"] in {
    def int_nvvm_f2h_rn # ftz : NVVMBuiltin,
        PureIntrinsic<[llvm_i16_ty], [llvm_float_ty]>;

    def int_nvvm_bf2h_rn # ftz : NVVMBuiltin,
        PureIntrinsic<[llvm_i16_ty], [llvm_bfloat_ty]>;
  }
````
- **L1697 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_i32_ty]>;`.
  **L1697 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_i32_ty]>;`。
- **L1698 EN**: Blank line separating nearby declarations or logic blocks.
  **L1698 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1699 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1699 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1700 EN**: Declares TableGen def `int_nvvm_f2`.
  **L1700 CN**: 声明 TableGen def `int_nvvm_f2`。
- **L1701 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;`.
  **L1701 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;`。
- **L1702 EN**: Blank line separating nearby declarations or logic blocks.
  **L1702 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1703 EN**: Declares TableGen def `int_nvvm_`.
  **L1703 CN**: 声明 TableGen def `int_nvvm_`。
- **L1704 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_i32_ty]>;`.
  **L1704 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_i32_ty]>;`。
- **L1705 EN**: Blank line separating nearby declarations or logic blocks.
  **L1705 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1706 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1706 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1707 EN**: Declares TableGen def `int_nvvm_f2`.
  **L1707 CN**: 声明 TableGen def `int_nvvm_f2`。
- **L1708 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i64_ty], [llvm_float_ty]>;`.
  **L1708 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i64_ty], [llvm_float_ty]>;`。
- **L1709 EN**: Blank line separating nearby declarations or logic blocks.
  **L1709 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1710 EN**: Declares TableGen def `int_nvvm_d2`.
  **L1710 CN**: 声明 TableGen def `int_nvvm_d2`。
- **L1711 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i64_ty], [llvm_double_ty]>;`.
  **L1711 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i64_ty], [llvm_double_ty]>;`。
- **L1712 EN**: Blank line separating nearby declarations or logic blocks.
  **L1712 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1713 EN**: Declares TableGen def `int_nvvm_`.
  **L1713 CN**: 声明 TableGen def `int_nvvm_`。
- **L1714 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_float_ty], [llvm_i64_ty]>;`.
  **L1714 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_float_ty], [llvm_i64_ty]>;`。
- **L1715 EN**: Blank line separating nearby declarations or logic blocks.
  **L1715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1716 EN**: Declares TableGen def `int_nvvm_`.
  **L1716 CN**: 声明 TableGen def `int_nvvm_`。
- **L1717 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_double_ty], [llvm_i64_ty]>;`.
  **L1717 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_double_ty], [llvm_i64_ty]>;`。
- **L1718 EN**: Blank line separating nearby declarations or logic blocks.
  **L1718 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1719 EN**: Continues the surrounding expression or declaration: `} // sign`.
  **L1719 CN**: 继续构造周围的表达式或声明：`} // sign`。
- **L1720 EN**: Continues the surrounding expression or declaration: `} // rnd`.
  **L1720 CN**: 继续构造周围的表达式或声明：`} // rnd`。
- **L1721 EN**: Blank line separating nearby declarations or logic blocks.
  **L1721 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1722 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1722 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1723 EN**: Declares TableGen def `int_nvvm_f2h_rn`.
  **L1723 CN**: 声明 TableGen def `int_nvvm_f2h_rn`。
- **L1724 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_float_ty]>;`.
  **L1724 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_float_ty]>;`。
- **L1725 EN**: Blank line separating nearby declarations or logic blocks.
  **L1725 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1726 EN**: Declares TableGen def `int_nvvm_bf2h_rn`.
  **L1726 CN**: 声明 TableGen def `int_nvvm_bf2h_rn`。
- **L1727 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_bfloat_ty]>;`.
  **L1727 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_bfloat_ty]>;`。
- **L1728 EN**: Closes the current lexical scope or compound statement.
  **L1728 CN**: 结束当前词法作用域或复合语句块。

### Lines 1729-1760

````tablegen

  foreach rnd = ["rn", "rz"] in {
    foreach relu = ["", "_relu"] in {
      foreach satfinite = ["", "_satfinite"] in {
        def int_nvvm_ff2bf16x2_ # rnd # relu # satfinite : NVVMBuiltin,
            PureIntrinsic<[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty]>;

        def int_nvvm_ff2f16x2_ # rnd # relu # satfinite : NVVMBuiltin,
            PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty]>;

        def int_nvvm_f2bf16_ # rnd # relu # satfinite : NVVMBuiltin,
            PureIntrinsic<[llvm_bfloat_ty], [llvm_float_ty]>;

        def int_nvvm_f2f16_ # rnd # relu # satfinite : NVVMBuiltin,
            PureIntrinsic<[llvm_half_ty], [llvm_float_ty]>;
      }
    }
  }

  // RS rounding mode (Stochastic Rounding) conversions for f16x2, bf16x2 types
  // The last i32 operand provides the random bits for the conversion
  foreach relu = ["", "_relu"] in {
    foreach satfinite = ["", "_satfinite"] in {
      def int_nvvm_ff2f16x2_rs # relu # satfinite : NVVMBuiltin,
          PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]>;

      def int_nvvm_ff2bf16x2_rs # relu # satfinite : NVVMBuiltin,
          PureIntrinsic<[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]>;
    }
  }

  foreach satfinite = ["", "_satfinite"] in {
````
- **L1729 EN**: Blank line separating nearby declarations or logic blocks.
  **L1729 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1730 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1730 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1731 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1731 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1732 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1732 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1733 EN**: Declares TableGen def `int_nvvm_ff2bf16x2_`.
  **L1733 CN**: 声明 TableGen def `int_nvvm_ff2bf16x2_`。
- **L1734 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1734 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1735 EN**: Blank line separating nearby declarations or logic blocks.
  **L1735 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1736 EN**: Declares TableGen def `int_nvvm_ff2f16x2_`.
  **L1736 CN**: 声明 TableGen def `int_nvvm_ff2f16x2_`。
- **L1737 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1737 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1738 EN**: Blank line separating nearby declarations or logic blocks.
  **L1738 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1739 EN**: Declares TableGen def `int_nvvm_f2bf16_`.
  **L1739 CN**: 声明 TableGen def `int_nvvm_f2bf16_`。
- **L1740 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_bfloat_ty], [llvm_float_ty]>;`.
  **L1740 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_bfloat_ty], [llvm_float_ty]>;`。
- **L1741 EN**: Blank line separating nearby declarations or logic blocks.
  **L1741 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1742 EN**: Declares TableGen def `int_nvvm_f2f16_`.
  **L1742 CN**: 声明 TableGen def `int_nvvm_f2f16_`。
- **L1743 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_half_ty], [llvm_float_ty]>;`.
  **L1743 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_half_ty], [llvm_float_ty]>;`。
- **L1744 EN**: Closes the current lexical scope or compound statement.
  **L1744 CN**: 结束当前词法作用域或复合语句块。
- **L1745 EN**: Closes the current lexical scope or compound statement.
  **L1745 CN**: 结束当前词法作用域或复合语句块。
- **L1746 EN**: Closes the current lexical scope or compound statement.
  **L1746 CN**: 结束当前词法作用域或复合语句块。
- **L1747 EN**: Blank line separating nearby declarations or logic blocks.
  **L1747 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1748 EN**: Comment explains nearby logic, invariants, or intent: `RS rounding mode (Stochastic Rounding) conversions for f16x2, bf16x2 types`.
  **L1748 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RS rounding mode (Stochastic Rounding) conversions for f16x2, bf16x2 types`。
- **L1749 EN**: Comment explains nearby logic, invariants, or intent: `The last i32 operand provides the random bits for the conversion`.
  **L1749 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The last i32 operand provides the random bits for the conversion`。
- **L1750 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1750 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1751 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1751 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1752 EN**: Declares TableGen def `int_nvvm_ff2f16x2_rs`.
  **L1752 CN**: 声明 TableGen def `int_nvvm_ff2f16x2_rs`。
- **L1753 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]>;`.
  **L1753 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2f16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]>;`。
- **L1754 EN**: Blank line separating nearby declarations or logic blocks.
  **L1754 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1755 EN**: Declares TableGen def `int_nvvm_ff2bf16x2_rs`.
  **L1755 CN**: 声明 TableGen def `int_nvvm_ff2bf16x2_rs`。
- **L1756 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]>;`.
  **L1756 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2bf16_ty], [llvm_float_ty, llvm_float_ty, llvm_i32_ty]>;`。
- **L1757 EN**: Closes the current lexical scope or compound statement.
  **L1757 CN**: 结束当前词法作用域或复合语句块。
- **L1758 EN**: Closes the current lexical scope or compound statement.
  **L1758 CN**: 结束当前词法作用域或复合语句块。
- **L1759 EN**: Blank line separating nearby declarations or logic blocks.
  **L1759 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1760 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1760 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 1761-1792

````tablegen
    def int_nvvm_f2tf32_rna # satfinite : NVVMBuiltin,
        PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;

    foreach rnd = ["rn", "rz"] in
      foreach relu = ["", "_relu"] in
        def int_nvvm_f2tf32_ # rnd # relu # satfinite : NVVMBuiltin,
            PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;
  }

  foreach relu = ["", "_relu"] in {
    def int_nvvm_ff_to_s2f6x2_rn # relu # _satfinite_scale_n2_ue8m0 :
        PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty, llvm_i16_ty]>;

    def int_nvvm_bf16x2_to_s2f6x2_rn # relu # _satfinite_scale_n2_ue8m0 :
        PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty, llvm_i16_ty]>;

    def int_nvvm_s2f6x2_to_bf16x2_rn # relu # _satfinite_scale_n2_ue8m0 :
        PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;

    // No satfinite variants
    def int_nvvm_s2f6x2_to_bf16x2_rn # relu # _scale_n2_ue8m0 :
        PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;
  }

  foreach type = ["e4m3x2", "e5m2x2"] in {
    foreach relu = ["", "_relu"] in {
      def int_nvvm_ff_to_ # type # _rn # relu : NVVMBuiltin,
          PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;

      def int_nvvm_f16x2_to_ # type # _rn # relu : NVVMBuiltin,
          PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;

````
- **L1761 EN**: Declares TableGen def `int_nvvm_f2tf32_rna`.
  **L1761 CN**: 声明 TableGen def `int_nvvm_f2tf32_rna`。
- **L1762 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;`.
  **L1762 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;`。
- **L1763 EN**: Blank line separating nearby declarations or logic blocks.
  **L1763 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1764 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1764 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1765 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1765 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1766 EN**: Declares TableGen def `int_nvvm_f2tf32_`.
  **L1766 CN**: 声明 TableGen def `int_nvvm_f2tf32_`。
- **L1767 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;`.
  **L1767 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i32_ty], [llvm_float_ty]>;`。
- **L1768 EN**: Closes the current lexical scope or compound statement.
  **L1768 CN**: 结束当前词法作用域或复合语句块。
- **L1769 EN**: Blank line separating nearby declarations or logic blocks.
  **L1769 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1770 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1770 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1771 EN**: Declares TableGen def `int_nvvm_ff_to_s2f6x2_rn`.
  **L1771 CN**: 声明 TableGen def `int_nvvm_ff_to_s2f6x2_rn`。
- **L1772 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty, llvm_i16_ty]>;`.
  **L1772 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty, llvm_i16_ty]>;`。
- **L1773 EN**: Blank line separating nearby declarations or logic blocks.
  **L1773 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1774 EN**: Declares TableGen def `int_nvvm_bf16x2_to_s2f6x2_rn`.
  **L1774 CN**: 声明 TableGen def `int_nvvm_bf16x2_to_s2f6x2_rn`。
- **L1775 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty, llvm_i16_ty]>;`.
  **L1775 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty, llvm_i16_ty]>;`。
- **L1776 EN**: Blank line separating nearby declarations or logic blocks.
  **L1776 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1777 EN**: Declares TableGen def `int_nvvm_s2f6x2_to_bf16x2_rn`.
  **L1777 CN**: 声明 TableGen def `int_nvvm_s2f6x2_to_bf16x2_rn`。
- **L1778 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`.
  **L1778 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`。
- **L1779 EN**: Blank line separating nearby declarations or logic blocks.
  **L1779 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1780 EN**: Comment explains nearby logic, invariants, or intent: `No satfinite variants`.
  **L1780 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`No satfinite variants`。
- **L1781 EN**: Declares TableGen def `int_nvvm_s2f6x2_to_bf16x2_rn`.
  **L1781 CN**: 声明 TableGen def `int_nvvm_s2f6x2_to_bf16x2_rn`。
- **L1782 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`.
  **L1782 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`。
- **L1783 EN**: Closes the current lexical scope or compound statement.
  **L1783 CN**: 结束当前词法作用域或复合语句块。
- **L1784 EN**: Blank line separating nearby declarations or logic blocks.
  **L1784 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1785 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1785 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1786 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1786 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1787 EN**: Declares TableGen def `int_nvvm_ff_to_`.
  **L1787 CN**: 声明 TableGen def `int_nvvm_ff_to_`。
- **L1788 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1788 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1789 EN**: Blank line separating nearby declarations or logic blocks.
  **L1789 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1790 EN**: Declares TableGen def `int_nvvm_f16x2_to_`.
  **L1790 CN**: 声明 TableGen def `int_nvvm_f16x2_to_`。
- **L1791 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;`.
  **L1791 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;`。
- **L1792 EN**: Blank line separating nearby declarations or logic blocks.
  **L1792 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1793-1824

````tablegen
      def int_nvvm_ # type # _to_f16x2_rn # relu : NVVMBuiltin,
          PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;
          
      def int_nvvm_bf16x2_to_ # type # _rn # relu # _satfinite
        : PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;
      
      foreach satfinite = ["", "_satfinite"] in {
        def int_nvvm_ # type # _to_bf16x2_rn # relu # satfinite # _scale_n2_ue8m0
            : PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;
      }
    }
  }

  // RS rounding mode (Stochastic Rounding) conversions for f8x4 types
  // The last i32 operand provides the random bits for the conversion
  foreach type = ["e4m3x4", "e5m2x4"] in {
    foreach relu = ["", "_relu"] in {
      def int_nvvm_f32x4_to_ # type # _rs # relu # _satfinite : NVVMBuiltin,
          PureIntrinsic<[llvm_v4i8_ty], [llvm_v4f32_ty, llvm_i32_ty]>;
    }
  }

  // FP4 conversions.
  foreach relu = ["", "_relu"] in {
    def int_nvvm_ff_to_e2m1x2_rn # relu # _satfinite : NVVMBuiltin,
        PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;

    def int_nvvm_e2m1x2_to_f16x2_rn # relu : NVVMBuiltin,
        PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;
        
    def int_nvvm_f16x2_to_e2m1x2_rn # relu # _satfinite
      : PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;
````
- **L1793 EN**: Declares TableGen def `int_nvvm_`.
  **L1793 CN**: 声明 TableGen def `int_nvvm_`。
- **L1794 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;`.
  **L1794 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;`。
- **L1795 EN**: Blank line separating nearby declarations or logic blocks.
  **L1795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1796 EN**: Declares TableGen def `int_nvvm_bf16x2_to_`.
  **L1796 CN**: 声明 TableGen def `int_nvvm_bf16x2_to_`。
- **L1797 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`.
  **L1797 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`。
- **L1798 EN**: Blank line separating nearby declarations or logic blocks.
  **L1798 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1799 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1799 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1800 EN**: Declares TableGen def `int_nvvm_`.
  **L1800 CN**: 声明 TableGen def `int_nvvm_`。
- **L1801 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`.
  **L1801 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`。
- **L1802 EN**: Closes the current lexical scope or compound statement.
  **L1802 CN**: 结束当前词法作用域或复合语句块。
- **L1803 EN**: Closes the current lexical scope or compound statement.
  **L1803 CN**: 结束当前词法作用域或复合语句块。
- **L1804 EN**: Closes the current lexical scope or compound statement.
  **L1804 CN**: 结束当前词法作用域或复合语句块。
- **L1805 EN**: Blank line separating nearby declarations or logic blocks.
  **L1805 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1806 EN**: Comment explains nearby logic, invariants, or intent: `RS rounding mode (Stochastic Rounding) conversions for f8x4 types`.
  **L1806 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RS rounding mode (Stochastic Rounding) conversions for f8x4 types`。
- **L1807 EN**: Comment explains nearby logic, invariants, or intent: `The last i32 operand provides the random bits for the conversion`.
  **L1807 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The last i32 operand provides the random bits for the conversion`。
- **L1808 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1808 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1809 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1809 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1810 EN**: Declares TableGen def `int_nvvm_f32x4_to_`.
  **L1810 CN**: 声明 TableGen def `int_nvvm_f32x4_to_`。
- **L1811 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v4i8_ty], [llvm_v4f32_ty, llvm_i32_ty]>;`.
  **L1811 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v4i8_ty], [llvm_v4f32_ty, llvm_i32_ty]>;`。
- **L1812 EN**: Closes the current lexical scope or compound statement.
  **L1812 CN**: 结束当前词法作用域或复合语句块。
- **L1813 EN**: Closes the current lexical scope or compound statement.
  **L1813 CN**: 结束当前词法作用域或复合语句块。
- **L1814 EN**: Blank line separating nearby declarations or logic blocks.
  **L1814 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1815 EN**: Comment explains nearby logic, invariants, or intent: `FP4 conversions.`.
  **L1815 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP4 conversions.`。
- **L1816 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1816 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1817 EN**: Declares TableGen def `int_nvvm_ff_to_e2m1x2_rn`.
  **L1817 CN**: 声明 TableGen def `int_nvvm_ff_to_e2m1x2_rn`。
- **L1818 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1818 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1819 EN**: Blank line separating nearby declarations or logic blocks.
  **L1819 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1820 EN**: Declares TableGen def `int_nvvm_e2m1x2_to_f16x2_rn`.
  **L1820 CN**: 声明 TableGen def `int_nvvm_e2m1x2_to_f16x2_rn`。
- **L1821 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;`.
  **L1821 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;`。
- **L1822 EN**: Blank line separating nearby declarations or logic blocks.
  **L1822 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1823 EN**: Declares TableGen def `int_nvvm_f16x2_to_e2m1x2_rn`.
  **L1823 CN**: 声明 TableGen def `int_nvvm_f16x2_to_e2m1x2_rn`。
- **L1824 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;`.
  **L1824 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;`。

### Lines 1825-1856

````tablegen

    def int_nvvm_bf16x2_to_e2m1x2_rn # relu # _satfinite
      : PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;
    
    foreach satfinite = ["", "_satfinite"] in {
      def int_nvvm_e2m1x2_to_bf16x2_rn # relu # satfinite # _scale_n2_ue8m0
        : PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;
    }
  }

  // RS rounding mode (Stochastic Rounding) conversions for f4x4 type
  // The last i32 operand provides the random bits for the conversion
  foreach relu = ["", "_relu"] in {
    def int_nvvm_f32x4_to_e2m1x4_rs # relu # _satfinite : NVVMBuiltin,
        PureIntrinsic<[llvm_i16_ty], [llvm_v4f32_ty, llvm_i32_ty]>;
  }

  // FP6 conversions.
  foreach type = ["e2m3x2", "e3m2x2"] in {
    foreach relu = ["", "_relu"] in {
      def int_nvvm_ff_to_ # type # _rn # relu # _satfinite : NVVMBuiltin,
          PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;

      def int_nvvm_ # type # _to_f16x2_rn # relu : NVVMBuiltin,
          PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;
          
      def int_nvvm_f16x2_to_ # type # _rn # relu # _satfinite
        : PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;

      def int_nvvm_bf16x2_to_ # type # _rn # relu # _satfinite
        : PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;
      
````
- **L1825 EN**: Blank line separating nearby declarations or logic blocks.
  **L1825 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1826 EN**: Declares TableGen def `int_nvvm_bf16x2_to_e2m1x2_rn`.
  **L1826 CN**: 声明 TableGen def `int_nvvm_bf16x2_to_e2m1x2_rn`。
- **L1827 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`.
  **L1827 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`。
- **L1828 EN**: Blank line separating nearby declarations or logic blocks.
  **L1828 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1829 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1829 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1830 EN**: Declares TableGen def `int_nvvm_e2m1x2_to_bf16x2_rn`.
  **L1830 CN**: 声明 TableGen def `int_nvvm_e2m1x2_to_bf16x2_rn`。
- **L1831 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`.
  **L1831 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`。
- **L1832 EN**: Closes the current lexical scope or compound statement.
  **L1832 CN**: 结束当前词法作用域或复合语句块。
- **L1833 EN**: Closes the current lexical scope or compound statement.
  **L1833 CN**: 结束当前词法作用域或复合语句块。
- **L1834 EN**: Blank line separating nearby declarations or logic blocks.
  **L1834 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1835 EN**: Comment explains nearby logic, invariants, or intent: `RS rounding mode (Stochastic Rounding) conversions for f4x4 type`.
  **L1835 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RS rounding mode (Stochastic Rounding) conversions for f4x4 type`。
- **L1836 EN**: Comment explains nearby logic, invariants, or intent: `The last i32 operand provides the random bits for the conversion`.
  **L1836 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The last i32 operand provides the random bits for the conversion`。
- **L1837 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1837 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1838 EN**: Declares TableGen def `int_nvvm_f32x4_to_e2m1x4_rs`.
  **L1838 CN**: 声明 TableGen def `int_nvvm_f32x4_to_e2m1x4_rs`。
- **L1839 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_v4f32_ty, llvm_i32_ty]>;`.
  **L1839 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_v4f32_ty, llvm_i32_ty]>;`。
- **L1840 EN**: Closes the current lexical scope or compound statement.
  **L1840 CN**: 结束当前词法作用域或复合语句块。
- **L1841 EN**: Blank line separating nearby declarations or logic blocks.
  **L1841 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1842 EN**: Comment explains nearby logic, invariants, or intent: `FP6 conversions.`.
  **L1842 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`FP6 conversions.`。
- **L1843 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1843 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1844 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1844 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1845 EN**: Declares TableGen def `int_nvvm_ff_to_`.
  **L1845 CN**: 声明 TableGen def `int_nvvm_ff_to_`。
- **L1846 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1846 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1847 EN**: Blank line separating nearby declarations or logic blocks.
  **L1847 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1848 EN**: Declares TableGen def `int_nvvm_`.
  **L1848 CN**: 声明 TableGen def `int_nvvm_`。
- **L1849 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;`.
  **L1849 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2f16_ty], [llvm_i16_ty]>;`。
- **L1850 EN**: Blank line separating nearby declarations or logic blocks.
  **L1850 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1851 EN**: Declares TableGen def `int_nvvm_f16x2_to_`.
  **L1851 CN**: 声明 TableGen def `int_nvvm_f16x2_to_`。
- **L1852 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;`.
  **L1852 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_i16_ty], [llvm_v2f16_ty]>;`。
- **L1853 EN**: Blank line separating nearby declarations or logic blocks.
  **L1853 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1854 EN**: Declares TableGen def `int_nvvm_bf16x2_to_`.
  **L1854 CN**: 声明 TableGen def `int_nvvm_bf16x2_to_`。
- **L1855 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`.
  **L1855 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`。
- **L1856 EN**: Blank line separating nearby declarations or logic blocks.
  **L1856 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1857-1888

````tablegen
      foreach satfinite = ["", "_satfinite"] in {
        def int_nvvm_ # type # _to_bf16x2_rn # relu # satfinite # _scale_n2_ue8m0
            : PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;
      }
    }
  }

  // RS rounding mode (Stochastic Rounding) conversions for f6x4 types
  // The last i32 operand provides the random bits for the conversion
  foreach type = ["e2m3x4", "e3m2x4"] in {
    foreach relu = ["", "_relu"] in {
      def int_nvvm_f32x4_to_ # type # _rs # relu # _satfinite : NVVMBuiltin,
          PureIntrinsic<[llvm_v4i8_ty], [llvm_v4f32_ty, llvm_i32_ty]>;
    }
  }

  // UE8M0x2 conversions.
  foreach rmode = ["_rz", "_rp"] in {
    foreach satmode = ["", "_satfinite"] in {
      defvar suffix = rmode # satmode;
      def int_nvvm_ff_to_ue8m0x2 # suffix : NVVMBuiltin,
          PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;

      def int_nvvm_bf16x2_to_ue8m0x2 # suffix : NVVMBuiltin,
          PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;

    }
  }

  def int_nvvm_ue8m0x2_to_bf16x2 : NVVMBuiltin,
      PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty]>;

````
- **L1857 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1857 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1858 EN**: Declares TableGen def `int_nvvm_`.
  **L1858 CN**: 声明 TableGen def `int_nvvm_`。
- **L1859 EN**: Executes a standalone statement or declaration: `: PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`.
  **L1859 CN**: 执行一条独立语句或声明：`: PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty, llvm_i16_ty]>;`。
- **L1860 EN**: Closes the current lexical scope or compound statement.
  **L1860 CN**: 结束当前词法作用域或复合语句块。
- **L1861 EN**: Closes the current lexical scope or compound statement.
  **L1861 CN**: 结束当前词法作用域或复合语句块。
- **L1862 EN**: Closes the current lexical scope or compound statement.
  **L1862 CN**: 结束当前词法作用域或复合语句块。
- **L1863 EN**: Blank line separating nearby declarations or logic blocks.
  **L1863 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1864 EN**: Comment explains nearby logic, invariants, or intent: `RS rounding mode (Stochastic Rounding) conversions for f6x4 types`.
  **L1864 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`RS rounding mode (Stochastic Rounding) conversions for f6x4 types`。
- **L1865 EN**: Comment explains nearby logic, invariants, or intent: `The last i32 operand provides the random bits for the conversion`.
  **L1865 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The last i32 operand provides the random bits for the conversion`。
- **L1866 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1866 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1867 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1867 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1868 EN**: Declares TableGen def `int_nvvm_f32x4_to_`.
  **L1868 CN**: 声明 TableGen def `int_nvvm_f32x4_to_`。
- **L1869 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v4i8_ty], [llvm_v4f32_ty, llvm_i32_ty]>;`.
  **L1869 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v4i8_ty], [llvm_v4f32_ty, llvm_i32_ty]>;`。
- **L1870 EN**: Closes the current lexical scope or compound statement.
  **L1870 CN**: 结束当前词法作用域或复合语句块。
- **L1871 EN**: Closes the current lexical scope or compound statement.
  **L1871 CN**: 结束当前词法作用域或复合语句块。
- **L1872 EN**: Blank line separating nearby declarations or logic blocks.
  **L1872 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1873 EN**: Comment explains nearby logic, invariants, or intent: `UE8M0x2 conversions.`.
  **L1873 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`UE8M0x2 conversions.`。
- **L1874 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1874 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1875 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1875 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1876 EN**: Initializes variable `suffix` from the right-hand expression.
  **L1876 CN**: 使用右侧表达式初始化变量 `suffix`。
- **L1877 EN**: Declares TableGen def `int_nvvm_ff_to_ue8m0x2`.
  **L1877 CN**: 声明 TableGen def `int_nvvm_ff_to_ue8m0x2`。
- **L1878 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`.
  **L1878 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_float_ty, llvm_float_ty]>;`。
- **L1879 EN**: Blank line separating nearby declarations or logic blocks.
  **L1879 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1880 EN**: Declares TableGen def `int_nvvm_bf16x2_to_ue8m0x2`.
  **L1880 CN**: 声明 TableGen def `int_nvvm_bf16x2_to_ue8m0x2`。
- **L1881 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`.
  **L1881 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_i16_ty], [llvm_v2bf16_ty]>;`。
- **L1882 EN**: Blank line separating nearby declarations or logic blocks.
  **L1882 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1883 EN**: Closes the current lexical scope or compound statement.
  **L1883 CN**: 结束当前词法作用域或复合语句块。
- **L1884 EN**: Closes the current lexical scope or compound statement.
  **L1884 CN**: 结束当前词法作用域或复合语句块。
- **L1885 EN**: Blank line separating nearby declarations or logic blocks.
  **L1885 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1886 EN**: Declares TableGen def `int_nvvm_ue8m0x2_to_bf16x2`.
  **L1886 CN**: 声明 TableGen def `int_nvvm_ue8m0x2_to_bf16x2`。
- **L1887 EN**: Executes a standalone statement or declaration: `PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty]>;`.
  **L1887 CN**: 执行一条独立语句或声明：`PureIntrinsic<[llvm_v2bf16_ty], [llvm_i16_ty]>;`。
- **L1888 EN**: Blank line separating nearby declarations or logic blocks.
  **L1888 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 1889-1920

````tablegen
  //
  // Atomic operations
  //
  class SCOPED_ATOMIC2_impl<LLVMType elty>
        : Intrinsic<[elty],
          [llvm_anyptr_ty, LLVMMatchType<0>],
          [IntrArgMemOnly, IntrNoCallback, NoCapture<ArgIndex<0>>]>;
  class SCOPED_ATOMIC3_impl<LLVMType elty>
        : Intrinsic<[elty],
          [llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>],
          [IntrArgMemOnly, IntrNoCallback, NoCapture<ArgIndex<0>>]>;

  multiclass PTXAtomicWithScope2<LLVMType elty> {
    def _cta : SCOPED_ATOMIC2_impl<elty>;
    def _sys : SCOPED_ATOMIC2_impl<elty>;
  }
  multiclass PTXAtomicWithScope3<LLVMType elty> {
    def _cta : SCOPED_ATOMIC3_impl<elty>;
    def _sys : SCOPED_ATOMIC3_impl<elty>;
  }
  multiclass PTXAtomicWithScope2_fi {
    defm _f : PTXAtomicWithScope2<llvm_anyfloat_ty>;
    defm _i : PTXAtomicWithScope2<llvm_anyint_ty>;
  }
  defm int_nvvm_atomic_add_gen    : PTXAtomicWithScope2_fi;
  defm int_nvvm_atomic_inc_gen_i  : PTXAtomicWithScope2<llvm_anyint_ty>;
  defm int_nvvm_atomic_dec_gen_i  : PTXAtomicWithScope2<llvm_anyint_ty>;
  defm int_nvvm_atomic_exch_gen_i : PTXAtomicWithScope2<llvm_anyint_ty>;
  defm int_nvvm_atomic_xor_gen_i  : PTXAtomicWithScope2<llvm_anyint_ty>;
  defm int_nvvm_atomic_max_gen_i  : PTXAtomicWithScope2<llvm_anyint_ty>;
  defm int_nvvm_atomic_min_gen_i  : PTXAtomicWithScope2<llvm_anyint_ty>;
  defm int_nvvm_atomic_or_gen_i   : PTXAtomicWithScope2<llvm_anyint_ty>;
````
- **L1889 EN**: Separator comment used for visual grouping.
  **L1889 CN**: 用于视觉分组的分隔注释。
- **L1890 EN**: Comment explains nearby logic, invariants, or intent: `Atomic operations`.
  **L1890 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Atomic operations`。
- **L1891 EN**: Separator comment used for visual grouping.
  **L1891 CN**: 用于视觉分组的分隔注释。
- **L1892 EN**: Declares class `SCOPED_ATOMIC2_impl<LLVMType`.
  **L1892 CN**: 声明 class `SCOPED_ATOMIC2_impl<LLVMType`。
- **L1893 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[elty],`.
  **L1893 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[elty],`。
- **L1894 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMMatchType<0>],`.
  **L1894 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMMatchType<0>],`。
- **L1895 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, IntrNoCallback, NoCapture<ArgIndex<0>>]>;`.
  **L1895 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, IntrNoCallback, NoCapture<ArgIndex<0>>]>;`。
- **L1896 EN**: Declares class `SCOPED_ATOMIC3_impl<LLVMType`.
  **L1896 CN**: 声明 class `SCOPED_ATOMIC3_impl<LLVMType`。
- **L1897 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[elty],`.
  **L1897 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[elty],`。
- **L1898 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>],`.
  **L1898 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty, LLVMMatchType<0>, LLVMMatchType<0>],`。
- **L1899 EN**: Executes a standalone statement or declaration: `[IntrArgMemOnly, IntrNoCallback, NoCapture<ArgIndex<0>>]>;`.
  **L1899 CN**: 执行一条独立语句或声明：`[IntrArgMemOnly, IntrNoCallback, NoCapture<ArgIndex<0>>]>;`。
- **L1900 EN**: Blank line separating nearby declarations or logic blocks.
  **L1900 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1901 EN**: Declares TableGen multiclass `PTXAtomicWithScope2`.
  **L1901 CN**: 声明 TableGen multiclass `PTXAtomicWithScope2`。
- **L1902 EN**: Declares TableGen def `_cta`.
  **L1902 CN**: 声明 TableGen def `_cta`。
- **L1903 EN**: Declares TableGen def `_sys`.
  **L1903 CN**: 声明 TableGen def `_sys`。
- **L1904 EN**: Closes the current lexical scope or compound statement.
  **L1904 CN**: 结束当前词法作用域或复合语句块。
- **L1905 EN**: Declares TableGen multiclass `PTXAtomicWithScope3`.
  **L1905 CN**: 声明 TableGen multiclass `PTXAtomicWithScope3`。
- **L1906 EN**: Declares TableGen def `_cta`.
  **L1906 CN**: 声明 TableGen def `_cta`。
- **L1907 EN**: Declares TableGen def `_sys`.
  **L1907 CN**: 声明 TableGen def `_sys`。
- **L1908 EN**: Closes the current lexical scope or compound statement.
  **L1908 CN**: 结束当前词法作用域或复合语句块。
- **L1909 EN**: Declares TableGen multiclass `PTXAtomicWithScope2_fi`.
  **L1909 CN**: 声明 TableGen multiclass `PTXAtomicWithScope2_fi`。
- **L1910 EN**: Declares TableGen defm `_f`.
  **L1910 CN**: 声明 TableGen defm `_f`。
- **L1911 EN**: Declares TableGen defm `_i`.
  **L1911 CN**: 声明 TableGen defm `_i`。
- **L1912 EN**: Closes the current lexical scope or compound statement.
  **L1912 CN**: 结束当前词法作用域或复合语句块。
- **L1913 EN**: Declares TableGen defm `int_nvvm_atomic_add_gen`.
  **L1913 CN**: 声明 TableGen defm `int_nvvm_atomic_add_gen`。
- **L1914 EN**: Declares TableGen defm `int_nvvm_atomic_inc_gen_i`.
  **L1914 CN**: 声明 TableGen defm `int_nvvm_atomic_inc_gen_i`。
- **L1915 EN**: Declares TableGen defm `int_nvvm_atomic_dec_gen_i`.
  **L1915 CN**: 声明 TableGen defm `int_nvvm_atomic_dec_gen_i`。
- **L1916 EN**: Declares TableGen defm `int_nvvm_atomic_exch_gen_i`.
  **L1916 CN**: 声明 TableGen defm `int_nvvm_atomic_exch_gen_i`。
- **L1917 EN**: Declares TableGen defm `int_nvvm_atomic_xor_gen_i`.
  **L1917 CN**: 声明 TableGen defm `int_nvvm_atomic_xor_gen_i`。
- **L1918 EN**: Declares TableGen defm `int_nvvm_atomic_max_gen_i`.
  **L1918 CN**: 声明 TableGen defm `int_nvvm_atomic_max_gen_i`。
- **L1919 EN**: Declares TableGen defm `int_nvvm_atomic_min_gen_i`.
  **L1919 CN**: 声明 TableGen defm `int_nvvm_atomic_min_gen_i`。
- **L1920 EN**: Declares TableGen defm `int_nvvm_atomic_or_gen_i`.
  **L1920 CN**: 声明 TableGen defm `int_nvvm_atomic_or_gen_i`。

### Lines 1921-1952

````tablegen
  defm int_nvvm_atomic_and_gen_i  : PTXAtomicWithScope2<llvm_anyint_ty>;
  defm int_nvvm_atomic_cas_gen_i  : PTXAtomicWithScope3<llvm_anyint_ty>;

  //
  // Bar.Sync
  //
  def int_nvvm_bar_warp_sync : NVVMBuiltin,
      Intrinsic<[], [llvm_i32_ty], [IntrConvergent, IntrNoCallback]>;

  multiclass BarrierCTAIntrinsic<list<LLVMType> ret_types = [],
                                 list<LLVMType> extra_param_types = [],
                                 bit has_all_variant = true> {
    let IntrProperties = [IntrConvergent, IntrNoCallback] in {
      foreach align = ["", "_aligned"] in {
        def align # _count :
          Intrinsic<ret_types, [llvm_i32_ty, llvm_i32_ty] # extra_param_types>;
        if has_all_variant then
          def align # _all :
            Intrinsic<ret_types, [llvm_i32_ty] # extra_param_types>;
      }
    }
  }

  // barrier{.cta}.sync{.aligned}      a{, b};
  // barrier{.cta}.arrive{.aligned}    a, b;
  defm int_nvvm_barrier_cta_sync : BarrierCTAIntrinsic<>;
  defm int_nvvm_barrier_cta_arrive : BarrierCTAIntrinsic<has_all_variant = false>;

  // barrier{.cta}.red.popc{.aligned}.u32  d, a{, b}, {!}c;
  // barrier{.cta}.red.op{.aligned}.pred   p, a{, b}, {!}c;
  // .op = { .and, .or };
  defm int_nvvm_barrier_cta_red_popc : BarrierCTAIntrinsic<[llvm_i32_ty], [llvm_i1_ty]>;
````
- **L1921 EN**: Declares TableGen defm `int_nvvm_atomic_and_gen_i`.
  **L1921 CN**: 声明 TableGen defm `int_nvvm_atomic_and_gen_i`。
- **L1922 EN**: Declares TableGen defm `int_nvvm_atomic_cas_gen_i`.
  **L1922 CN**: 声明 TableGen defm `int_nvvm_atomic_cas_gen_i`。
- **L1923 EN**: Blank line separating nearby declarations or logic blocks.
  **L1923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1924 EN**: Separator comment used for visual grouping.
  **L1924 CN**: 用于视觉分组的分隔注释。
- **L1925 EN**: Comment explains nearby logic, invariants, or intent: `Bar.Sync`.
  **L1925 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bar.Sync`。
- **L1926 EN**: Separator comment used for visual grouping.
  **L1926 CN**: 用于视觉分组的分隔注释。
- **L1927 EN**: Declares TableGen def `int_nvvm_bar_warp_sync`.
  **L1927 CN**: 声明 TableGen def `int_nvvm_bar_warp_sync`。
- **L1928 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [IntrConvergent, IntrNoCallback]>;`.
  **L1928 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [IntrConvergent, IntrNoCallback]>;`。
- **L1929 EN**: Blank line separating nearby declarations or logic blocks.
  **L1929 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1930 EN**: Declares TableGen multiclass `BarrierCTAIntrinsic`.
  **L1930 CN**: 声明 TableGen multiclass `BarrierCTAIntrinsic`。
- **L1931 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> extra_param_types = [],`.
  **L1931 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> extra_param_types = [],`。
- **L1932 EN**: Continues the surrounding expression or declaration: `bit has_all_variant = true> {`.
  **L1932 CN**: 继续构造周围的表达式或声明：`bit has_all_variant = true> {`。
- **L1933 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1933 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1934 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L1934 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1935 EN**: Declares TableGen def `align`.
  **L1935 CN**: 声明 TableGen def `align`。
- **L1936 EN**: Executes a standalone statement or declaration: `Intrinsic<ret_types, [llvm_i32_ty, llvm_i32_ty] # extra_param_types>;`.
  **L1936 CN**: 执行一条独立语句或声明：`Intrinsic<ret_types, [llvm_i32_ty, llvm_i32_ty] # extra_param_types>;`。
- **L1937 EN**: Continues the surrounding expression or declaration: `if has_all_variant then`.
  **L1937 CN**: 继续构造周围的表达式或声明：`if has_all_variant then`。
- **L1938 EN**: Declares TableGen def `align`.
  **L1938 CN**: 声明 TableGen def `align`。
- **L1939 EN**: Executes a standalone statement or declaration: `Intrinsic<ret_types, [llvm_i32_ty] # extra_param_types>;`.
  **L1939 CN**: 执行一条独立语句或声明：`Intrinsic<ret_types, [llvm_i32_ty] # extra_param_types>;`。
- **L1940 EN**: Closes the current lexical scope or compound statement.
  **L1940 CN**: 结束当前词法作用域或复合语句块。
- **L1941 EN**: Closes the current lexical scope or compound statement.
  **L1941 CN**: 结束当前词法作用域或复合语句块。
- **L1942 EN**: Closes the current lexical scope or compound statement.
  **L1942 CN**: 结束当前词法作用域或复合语句块。
- **L1943 EN**: Blank line separating nearby declarations or logic blocks.
  **L1943 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1944 EN**: Comment explains nearby logic, invariants, or intent: `barrier{.cta}.sync{.aligned}      a{, b};`.
  **L1944 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`barrier{.cta}.sync{.aligned}      a{, b};`。
- **L1945 EN**: Comment explains nearby logic, invariants, or intent: `barrier{.cta}.arrive{.aligned}    a, b;`.
  **L1945 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`barrier{.cta}.arrive{.aligned}    a, b;`。
- **L1946 EN**: Declares TableGen defm `int_nvvm_barrier_cta_sync`.
  **L1946 CN**: 声明 TableGen defm `int_nvvm_barrier_cta_sync`。
- **L1947 EN**: Declares TableGen defm `int_nvvm_barrier_cta_arrive`.
  **L1947 CN**: 声明 TableGen defm `int_nvvm_barrier_cta_arrive`。
- **L1948 EN**: Blank line separating nearby declarations or logic blocks.
  **L1948 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1949 EN**: Comment explains nearby logic, invariants, or intent: `barrier{.cta}.red.popc{.aligned}.u32  d, a{, b}, {!}c;`.
  **L1949 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`barrier{.cta}.red.popc{.aligned}.u32  d, a{, b}, {!}c;`。
- **L1950 EN**: Comment explains nearby logic, invariants, or intent: `barrier{.cta}.red.op{.aligned}.pred   p, a{, b}, {!}c;`.
  **L1950 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`barrier{.cta}.red.op{.aligned}.pred   p, a{, b}, {!}c;`。
- **L1951 EN**: Comment explains nearby logic, invariants, or intent: `.op = { .and, .or };`.
  **L1951 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`.op = { .and, .or };`。
- **L1952 EN**: Declares TableGen defm `int_nvvm_barrier_cta_red_popc`.
  **L1952 CN**: 声明 TableGen defm `int_nvvm_barrier_cta_red_popc`。

### Lines 1953-1984

````tablegen
  defm int_nvvm_barrier_cta_red_and : BarrierCTAIntrinsic<[llvm_i1_ty], [llvm_i1_ty]>;
  defm int_nvvm_barrier_cta_red_or : BarrierCTAIntrinsic<[llvm_i1_ty], [llvm_i1_ty]>;

  let IntrProperties = [IntrConvergent, IntrNoCallback] in {
    // barrier.cluster.[wait, arrive, arrive.relaxed]
    def int_nvvm_barrier_cluster_arrive : Intrinsic<[]>;
    def int_nvvm_barrier_cluster_arrive_relaxed : Intrinsic<[]>;
    def int_nvvm_barrier_cluster_wait : Intrinsic<[]>;

    // 'aligned' versions of the above barrier.cluster.* intrinsics
    def int_nvvm_barrier_cluster_arrive_aligned : Intrinsic<[]>;
    def int_nvvm_barrier_cluster_arrive_relaxed_aligned : Intrinsic<[]>;
    def int_nvvm_barrier_cluster_wait_aligned : Intrinsic<[]>;
  }

  //
  // Membar / Fence
  //
  let IntrProperties = [IntrNoCallback] in {
    def int_nvvm_membar_cta : NVVMBuiltin, Intrinsic<[]>;
    def int_nvvm_membar_gl : NVVMBuiltin, Intrinsic<[]>;
    def int_nvvm_membar_sys : NVVMBuiltin, Intrinsic<[]>;
    def int_nvvm_fence_sc_cluster : Intrinsic<[]>;

    // Operation fence
    def int_nvvm_fence_mbarrier_init_release_cluster: Intrinsic<[], [], [],
            "llvm.nvvm.fence.mbarrier_init.release.cluster">;

    // Thread fence
    def int_nvvm_fence_acquire_sync_restrict_space_cluster_scope_cluster :
          Intrinsic<[], [], [],
            "llvm.nvvm.fence.acquire.sync_restrict.space.cluster.scope.cluster">;
````
- **L1953 EN**: Declares TableGen defm `int_nvvm_barrier_cta_red_and`.
  **L1953 CN**: 声明 TableGen defm `int_nvvm_barrier_cta_red_and`。
- **L1954 EN**: Declares TableGen defm `int_nvvm_barrier_cta_red_or`.
  **L1954 CN**: 声明 TableGen defm `int_nvvm_barrier_cta_red_or`。
- **L1955 EN**: Blank line separating nearby declarations or logic blocks.
  **L1955 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1956 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1956 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1957 EN**: Comment explains nearby logic, invariants, or intent: `barrier.cluster.[wait, arrive, arrive.relaxed]`.
  **L1957 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`barrier.cluster.[wait, arrive, arrive.relaxed]`。
- **L1958 EN**: Declares TableGen def `int_nvvm_barrier_cluster_arrive`.
  **L1958 CN**: 声明 TableGen def `int_nvvm_barrier_cluster_arrive`。
- **L1959 EN**: Declares TableGen def `int_nvvm_barrier_cluster_arrive_relaxed`.
  **L1959 CN**: 声明 TableGen def `int_nvvm_barrier_cluster_arrive_relaxed`。
- **L1960 EN**: Declares TableGen def `int_nvvm_barrier_cluster_wait`.
  **L1960 CN**: 声明 TableGen def `int_nvvm_barrier_cluster_wait`。
- **L1961 EN**: Blank line separating nearby declarations or logic blocks.
  **L1961 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1962 EN**: Comment explains nearby logic, invariants, or intent: `'aligned' versions of the above barrier.cluster.* intrinsics`.
  **L1962 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`'aligned' versions of the above barrier.cluster.* intrinsics`。
- **L1963 EN**: Declares TableGen def `int_nvvm_barrier_cluster_arrive_aligned`.
  **L1963 CN**: 声明 TableGen def `int_nvvm_barrier_cluster_arrive_aligned`。
- **L1964 EN**: Declares TableGen def `int_nvvm_barrier_cluster_arrive_relaxed_aligned`.
  **L1964 CN**: 声明 TableGen def `int_nvvm_barrier_cluster_arrive_relaxed_aligned`。
- **L1965 EN**: Declares TableGen def `int_nvvm_barrier_cluster_wait_aligned`.
  **L1965 CN**: 声明 TableGen def `int_nvvm_barrier_cluster_wait_aligned`。
- **L1966 EN**: Closes the current lexical scope or compound statement.
  **L1966 CN**: 结束当前词法作用域或复合语句块。
- **L1967 EN**: Blank line separating nearby declarations or logic blocks.
  **L1967 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1968 EN**: Separator comment used for visual grouping.
  **L1968 CN**: 用于视觉分组的分隔注释。
- **L1969 EN**: Comment explains nearby logic, invariants, or intent: `Membar / Fence`.
  **L1969 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Membar / Fence`。
- **L1970 EN**: Separator comment used for visual grouping.
  **L1970 CN**: 用于视觉分组的分隔注释。
- **L1971 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L1971 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L1972 EN**: Declares TableGen def `int_nvvm_membar_cta`.
  **L1972 CN**: 声明 TableGen def `int_nvvm_membar_cta`。
- **L1973 EN**: Declares TableGen def `int_nvvm_membar_gl`.
  **L1973 CN**: 声明 TableGen def `int_nvvm_membar_gl`。
- **L1974 EN**: Declares TableGen def `int_nvvm_membar_sys`.
  **L1974 CN**: 声明 TableGen def `int_nvvm_membar_sys`。
- **L1975 EN**: Declares TableGen def `int_nvvm_fence_sc_cluster`.
  **L1975 CN**: 声明 TableGen def `int_nvvm_fence_sc_cluster`。
- **L1976 EN**: Blank line separating nearby declarations or logic blocks.
  **L1976 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1977 EN**: Comment explains nearby logic, invariants, or intent: `Operation fence`.
  **L1977 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Operation fence`。
- **L1978 EN**: Declares TableGen def `int_nvvm_fence_mbarrier_init_release_cluster`.
  **L1978 CN**: 声明 TableGen def `int_nvvm_fence_mbarrier_init_release_cluster`。
- **L1979 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.fence.mbarrier_init.release.cluster">;`.
  **L1979 CN**: 执行一条独立语句或声明：`"llvm.nvvm.fence.mbarrier_init.release.cluster">;`。
- **L1980 EN**: Blank line separating nearby declarations or logic blocks.
  **L1980 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1981 EN**: Comment explains nearby logic, invariants, or intent: `Thread fence`.
  **L1981 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Thread fence`。
- **L1982 EN**: Declares TableGen def `int_nvvm_fence_acquire_sync_restrict_space_cluster_scope_cluster`.
  **L1982 CN**: 声明 TableGen def `int_nvvm_fence_acquire_sync_restrict_space_cluster_scope_cluster`。
- **L1983 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [], [],`.
  **L1983 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [], [],`。
- **L1984 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.fence.acquire.sync_restrict.space.cluster.scope.cluster">;`.
  **L1984 CN**: 执行一条独立语句或声明：`"llvm.nvvm.fence.acquire.sync_restrict.space.cluster.scope.cluster">;`。

### Lines 1985-2016

````tablegen

    def int_nvvm_fence_release_sync_restrict_space_cta_scope_cluster :
          Intrinsic<[], [], [],
            "llvm.nvvm.fence.release.sync_restrict.space.cta.scope.cluster">;

  //
  // Proxy fence (uni-directional)
  //

  def int_nvvm_fence_proxy_async_generic_acquire_sync_restrict_space_cluster_scope_cluster :
        Intrinsic<[], [], [],
          "llvm.nvvm.fence.proxy.async_generic.acquire.sync_restrict.space.cluster.scope.cluster">;

  def int_nvvm_fence_proxy_async_generic_release_sync_restrict_space_cta_scope_cluster :
        Intrinsic<[], [], [],
          "llvm.nvvm.fence.proxy.async_generic.release.sync_restrict.space.cta.scope.cluster">;

  foreach scope = ["cta", "cluster", "gpu", "sys"] in {

    def int_nvvm_fence_proxy_tensormap_generic_release_ # scope :
          Intrinsic<[], [], [],
          "llvm.nvvm.fence.proxy.tensormap_generic.release." # scope>;

    // The imm-arg 'size' can only be 128.
    def int_nvvm_fence_proxy_tensormap_generic_acquire_ # scope :
          Intrinsic<[], [llvm_ptr_ty, llvm_i32_ty], [],
                    "llvm.nvvm.fence.proxy.tensormap_generic.acquire." # scope> {
      let IntrProperties = [IntrNoCallback, IntrArgMemOnly,
                            ImmArg<ArgIndex<1>>, Range<ArgIndex<1>, 128, 129>];
    }
  }

````
- **L1985 EN**: Blank line separating nearby declarations or logic blocks.
  **L1985 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1986 EN**: Declares TableGen def `int_nvvm_fence_release_sync_restrict_space_cta_scope_cluster`.
  **L1986 CN**: 声明 TableGen def `int_nvvm_fence_release_sync_restrict_space_cta_scope_cluster`。
- **L1987 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [], [],`.
  **L1987 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [], [],`。
- **L1988 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.fence.release.sync_restrict.space.cta.scope.cluster">;`.
  **L1988 CN**: 执行一条独立语句或声明：`"llvm.nvvm.fence.release.sync_restrict.space.cta.scope.cluster">;`。
- **L1989 EN**: Blank line separating nearby declarations or logic blocks.
  **L1989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1990 EN**: Separator comment used for visual grouping.
  **L1990 CN**: 用于视觉分组的分隔注释。
- **L1991 EN**: Comment explains nearby logic, invariants, or intent: `Proxy fence (uni-directional)`.
  **L1991 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Proxy fence (uni-directional)`。
- **L1992 EN**: Separator comment used for visual grouping.
  **L1992 CN**: 用于视觉分组的分隔注释。
- **L1993 EN**: Blank line separating nearby declarations or logic blocks.
  **L1993 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1994 EN**: Declares TableGen def `int_nvvm_fence_proxy_async_generic_acquire_sync_restrict_space_cluster_scope_cluster`.
  **L1994 CN**: 声明 TableGen def `int_nvvm_fence_proxy_async_generic_acquire_sync_restrict_space_cluster_scope_cluster`。
- **L1995 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [], [],`.
  **L1995 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [], [],`。
- **L1996 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.fence.proxy.async_generic.acquire.sync_restrict.space.cluster.scope.cluster">;`.
  **L1996 CN**: 执行一条独立语句或声明：`"llvm.nvvm.fence.proxy.async_generic.acquire.sync_restrict.space.cluster.scope.cluster">;`。
- **L1997 EN**: Blank line separating nearby declarations or logic blocks.
  **L1997 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L1998 EN**: Declares TableGen def `int_nvvm_fence_proxy_async_generic_release_sync_restrict_space_cta_scope_cluster`.
  **L1998 CN**: 声明 TableGen def `int_nvvm_fence_proxy_async_generic_release_sync_restrict_space_cta_scope_cluster`。
- **L1999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [], [],`.
  **L1999 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [], [],`。
- **L2000 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.fence.proxy.async_generic.release.sync_restrict.space.cta.scope.cluster">;`.
  **L2000 CN**: 执行一条独立语句或声明：`"llvm.nvvm.fence.proxy.async_generic.release.sync_restrict.space.cta.scope.cluster">;`。
- **L2001 EN**: Blank line separating nearby declarations or logic blocks.
  **L2001 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2002 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2002 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2003 EN**: Blank line separating nearby declarations or logic blocks.
  **L2003 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2004 EN**: Declares TableGen def `int_nvvm_fence_proxy_tensormap_generic_release_`.
  **L2004 CN**: 声明 TableGen def `int_nvvm_fence_proxy_tensormap_generic_release_`。
- **L2005 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [], [],`.
  **L2005 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [], [],`。
- **L2006 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.fence.proxy.tensormap_generic.release." # scope>;`.
  **L2006 CN**: 执行一条独立语句或声明：`"llvm.nvvm.fence.proxy.tensormap_generic.release." # scope>;`。
- **L2007 EN**: Blank line separating nearby declarations or logic blocks.
  **L2007 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2008 EN**: Comment explains nearby logic, invariants, or intent: `The imm-arg 'size' can only be 128.`.
  **L2008 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The imm-arg 'size' can only be 128.`。
- **L2009 EN**: Declares TableGen def `int_nvvm_fence_proxy_tensormap_generic_acquire_`.
  **L2009 CN**: 声明 TableGen def `int_nvvm_fence_proxy_tensormap_generic_acquire_`。
- **L2010 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [llvm_ptr_ty, llvm_i32_ty], [],`.
  **L2010 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [llvm_ptr_ty, llvm_i32_ty], [],`。
- **L2011 EN**: Continues the surrounding expression or declaration: `"llvm.nvvm.fence.proxy.tensormap_generic.acquire." # scope> {`.
  **L2011 CN**: 继续构造周围的表达式或声明：`"llvm.nvvm.fence.proxy.tensormap_generic.acquire." # scope> {`。
- **L2012 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2012 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2013 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<1>>, Range<ArgIndex<1>, 128, 129>];`.
  **L2013 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<1>>, Range<ArgIndex<1>, 128, 129>];`。
- **L2014 EN**: Closes the current lexical scope or compound statement.
  **L2014 CN**: 结束当前词法作用域或复合语句块。
- **L2015 EN**: Closes the current lexical scope or compound statement.
  **L2015 CN**: 结束当前词法作用域或复合语句块。
- **L2016 EN**: Blank line separating nearby declarations or logic blocks.
  **L2016 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2017-2048

````tablegen
  //
  // Proxy fence (bi-directional)
  //
    foreach proxykind = ["alias", "async", "async.global", "async.shared_cta",
                          "async.shared_cluster"] in {
      defvar Intr = IntrinsicName<"llvm.nvvm.fence.proxy." # proxykind>;
      def Intr.record_name: Intrinsic<[], [], [], Intr.intr_name>;
    }
  }

//
// Async Copy
//
let IntrProperties = [IntrConvergent, IntrNoCallback] in {
  def int_nvvm_cp_async_mbarrier_arrive : NVVMBuiltin,
      Intrinsic<[], [llvm_ptr_ty]>;
  def int_nvvm_cp_async_mbarrier_arrive_shared : NVVMBuiltin,
      Intrinsic<[], [llvm_shared_ptr_ty]>;
  def int_nvvm_cp_async_mbarrier_arrive_noinc : NVVMBuiltin,
      Intrinsic<[], [llvm_ptr_ty]>;
  def int_nvvm_cp_async_mbarrier_arrive_noinc_shared : NVVMBuiltin,
      Intrinsic<[], [llvm_shared_ptr_ty]>;
}

multiclass CP_ASYNC_SHARED_GLOBAL {
  let IntrProperties = [IntrArgMemOnly, IntrNoCallback, NoAlias<ArgIndex<0>>,
                        NoAlias<ArgIndex<1>>, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>] in {
    def NAME : Intrinsic<[], [llvm_shared_ptr_ty, llvm_global_ptr_ty]>;
    def _s : Intrinsic<[], [llvm_shared_ptr_ty, llvm_global_ptr_ty, llvm_i32_ty]>;
  }
}

````
- **L2017 EN**: Separator comment used for visual grouping.
  **L2017 CN**: 用于视觉分组的分隔注释。
- **L2018 EN**: Comment explains nearby logic, invariants, or intent: `Proxy fence (bi-directional)`.
  **L2018 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Proxy fence (bi-directional)`。
- **L2019 EN**: Separator comment used for visual grouping.
  **L2019 CN**: 用于视觉分组的分隔注释。
- **L2020 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2020 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2021 EN**: Continues the surrounding expression or declaration: `"async.shared_cluster"] in {`.
  **L2021 CN**: 继续构造周围的表达式或声明：`"async.shared_cluster"] in {`。
- **L2022 EN**: Initializes variable `Intr` from the right-hand expression.
  **L2022 CN**: 使用右侧表达式初始化变量 `Intr`。
- **L2023 EN**: Declares TableGen def `Intr.record_name`.
  **L2023 CN**: 声明 TableGen def `Intr.record_name`。
- **L2024 EN**: Closes the current lexical scope or compound statement.
  **L2024 CN**: 结束当前词法作用域或复合语句块。
- **L2025 EN**: Closes the current lexical scope or compound statement.
  **L2025 CN**: 结束当前词法作用域或复合语句块。
- **L2026 EN**: Blank line separating nearby declarations or logic blocks.
  **L2026 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2027 EN**: Separator comment used for visual grouping.
  **L2027 CN**: 用于视觉分组的分隔注释。
- **L2028 EN**: Comment explains nearby logic, invariants, or intent: `Async Copy`.
  **L2028 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Async Copy`。
- **L2029 EN**: Separator comment used for visual grouping.
  **L2029 CN**: 用于视觉分组的分隔注释。
- **L2030 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2030 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2031 EN**: Declares TableGen def `int_nvvm_cp_async_mbarrier_arrive`.
  **L2031 CN**: 声明 TableGen def `int_nvvm_cp_async_mbarrier_arrive`。
- **L2032 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty]>;`.
  **L2032 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty]>;`。
- **L2033 EN**: Declares TableGen def `int_nvvm_cp_async_mbarrier_arrive_shared`.
  **L2033 CN**: 声明 TableGen def `int_nvvm_cp_async_mbarrier_arrive_shared`。
- **L2034 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_shared_ptr_ty]>;`.
  **L2034 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_shared_ptr_ty]>;`。
- **L2035 EN**: Declares TableGen def `int_nvvm_cp_async_mbarrier_arrive_noinc`.
  **L2035 CN**: 声明 TableGen def `int_nvvm_cp_async_mbarrier_arrive_noinc`。
- **L2036 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_ptr_ty]>;`.
  **L2036 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_ptr_ty]>;`。
- **L2037 EN**: Declares TableGen def `int_nvvm_cp_async_mbarrier_arrive_noinc_shared`.
  **L2037 CN**: 声明 TableGen def `int_nvvm_cp_async_mbarrier_arrive_noinc_shared`。
- **L2038 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_shared_ptr_ty]>;`.
  **L2038 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_shared_ptr_ty]>;`。
- **L2039 EN**: Closes the current lexical scope or compound statement.
  **L2039 CN**: 结束当前词法作用域或复合语句块。
- **L2040 EN**: Blank line separating nearby declarations or logic blocks.
  **L2040 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2041 EN**: Declares TableGen multiclass `CP_ASYNC_SHARED_GLOBAL`.
  **L2041 CN**: 声明 TableGen multiclass `CP_ASYNC_SHARED_GLOBAL`。
- **L2042 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2042 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2043 EN**: Continues the surrounding expression or declaration: `NoAlias<ArgIndex<1>>, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>] in {`.
  **L2043 CN**: 继续构造周围的表达式或声明：`NoAlias<ArgIndex<1>>, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>] in {`。
- **L2044 EN**: Declares TableGen def `NAME`.
  **L2044 CN**: 声明 TableGen def `NAME`。
- **L2045 EN**: Declares TableGen def `_s`.
  **L2045 CN**: 声明 TableGen def `_s`。
- **L2046 EN**: Closes the current lexical scope or compound statement.
  **L2046 CN**: 结束当前词法作用域或复合语句块。
- **L2047 EN**: Closes the current lexical scope or compound statement.
  **L2047 CN**: 结束当前词法作用域或复合语句块。
- **L2048 EN**: Blank line separating nearby declarations or logic blocks.
  **L2048 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2049-2080

````tablegen
defm int_nvvm_cp_async_ca_shared_global_4 : CP_ASYNC_SHARED_GLOBAL;
defm int_nvvm_cp_async_ca_shared_global_8 : CP_ASYNC_SHARED_GLOBAL;
defm int_nvvm_cp_async_ca_shared_global_16 : CP_ASYNC_SHARED_GLOBAL;
defm int_nvvm_cp_async_cg_shared_global_16 : CP_ASYNC_SHARED_GLOBAL;

def int_nvvm_cp_async_commit_group : NVVMBuiltin, Intrinsic<[]>;

def int_nvvm_cp_async_wait_group : NVVMBuiltin,
    Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

def int_nvvm_cp_async_wait_all : NVVMBuiltin, Intrinsic<[]>;

// cp.async.bulk variants of the commit/wait group
def int_nvvm_cp_async_bulk_commit_group : Intrinsic<[]>;

def int_nvvm_cp_async_bulk_wait_group :
    Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

def int_nvvm_cp_async_bulk_wait_group_read :
    Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;

// mbarrier
foreach is_shared = [true, false] in {
  defvar mbarrier_ptr_ty = !if(is_shared, llvm_shared_ptr_ty, llvm_ptr_ty);
  defvar shared = !if(is_shared, "_shared", "");

  def int_nvvm_mbarrier_init # shared : NVVMBuiltin,
      Intrinsic<[], [mbarrier_ptr_ty, llvm_i32_ty],
                [IntrConvergent, IntrNoCallback]>;

  def int_nvvm_mbarrier_inval # shared : NVVMBuiltin,
      Intrinsic<[], [mbarrier_ptr_ty],
````
- **L2049 EN**: Declares TableGen defm `int_nvvm_cp_async_ca_shared_global_4`.
  **L2049 CN**: 声明 TableGen defm `int_nvvm_cp_async_ca_shared_global_4`。
- **L2050 EN**: Declares TableGen defm `int_nvvm_cp_async_ca_shared_global_8`.
  **L2050 CN**: 声明 TableGen defm `int_nvvm_cp_async_ca_shared_global_8`。
- **L2051 EN**: Declares TableGen defm `int_nvvm_cp_async_ca_shared_global_16`.
  **L2051 CN**: 声明 TableGen defm `int_nvvm_cp_async_ca_shared_global_16`。
- **L2052 EN**: Declares TableGen defm `int_nvvm_cp_async_cg_shared_global_16`.
  **L2052 CN**: 声明 TableGen defm `int_nvvm_cp_async_cg_shared_global_16`。
- **L2053 EN**: Blank line separating nearby declarations or logic blocks.
  **L2053 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2054 EN**: Declares TableGen def `int_nvvm_cp_async_commit_group`.
  **L2054 CN**: 声明 TableGen def `int_nvvm_cp_async_commit_group`。
- **L2055 EN**: Blank line separating nearby declarations or logic blocks.
  **L2055 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2056 EN**: Declares TableGen def `int_nvvm_cp_async_wait_group`.
  **L2056 CN**: 声明 TableGen def `int_nvvm_cp_async_wait_group`。
- **L2057 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L2057 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L2058 EN**: Blank line separating nearby declarations or logic blocks.
  **L2058 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2059 EN**: Declares TableGen def `int_nvvm_cp_async_wait_all`.
  **L2059 CN**: 声明 TableGen def `int_nvvm_cp_async_wait_all`。
- **L2060 EN**: Blank line separating nearby declarations or logic blocks.
  **L2060 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2061 EN**: Comment explains nearby logic, invariants, or intent: `cp.async.bulk variants of the commit/wait group`.
  **L2061 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`cp.async.bulk variants of the commit/wait group`。
- **L2062 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_commit_group`.
  **L2062 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_commit_group`。
- **L2063 EN**: Blank line separating nearby declarations or logic blocks.
  **L2063 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2064 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_wait_group`.
  **L2064 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_wait_group`。
- **L2065 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L2065 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L2066 EN**: Blank line separating nearby declarations or logic blocks.
  **L2066 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2067 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_wait_group_read`.
  **L2067 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_wait_group_read`。
- **L2068 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`.
  **L2068 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i32_ty], [ImmArg<ArgIndex<0>>]>;`。
- **L2069 EN**: Blank line separating nearby declarations or logic blocks.
  **L2069 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2070 EN**: Comment explains nearby logic, invariants, or intent: `mbarrier`.
  **L2070 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mbarrier`。
- **L2071 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2071 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2072 EN**: Initializes variable `mbarrier_ptr_ty` from the right-hand expression.
  **L2072 CN**: 使用右侧表达式初始化变量 `mbarrier_ptr_ty`。
- **L2073 EN**: Initializes variable `shared` from the right-hand expression.
  **L2073 CN**: 使用右侧表达式初始化变量 `shared`。
- **L2074 EN**: Blank line separating nearby declarations or logic blocks.
  **L2074 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2075 EN**: Declares TableGen def `int_nvvm_mbarrier_init`.
  **L2075 CN**: 声明 TableGen def `int_nvvm_mbarrier_init`。
- **L2076 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [mbarrier_ptr_ty, llvm_i32_ty],`.
  **L2076 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [mbarrier_ptr_ty, llvm_i32_ty],`。
- **L2077 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrNoCallback]>;`.
  **L2077 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrNoCallback]>;`。
- **L2078 EN**: Blank line separating nearby declarations or logic blocks.
  **L2078 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2079 EN**: Declares TableGen def `int_nvvm_mbarrier_inval`.
  **L2079 CN**: 声明 TableGen def `int_nvvm_mbarrier_inval`。
- **L2080 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [mbarrier_ptr_ty],`.
  **L2080 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [mbarrier_ptr_ty],`。

### Lines 2081-2112

````tablegen
                [IntrConvergent, IntrWriteMem, IntrArgMemOnly, IntrNoCallback,
                 WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;

  let IntrProperties = [IntrConvergent, IntrNoCallback] in {
    foreach drop = ["", "_drop"] in {
      def int_nvvm_mbarrier_arrive # drop # shared : NVVMBuiltin,
          Intrinsic<[llvm_i64_ty], [mbarrier_ptr_ty]>;
      def int_nvvm_mbarrier_arrive # drop # _noComplete # shared : NVVMBuiltin,
          Intrinsic<[llvm_i64_ty], [mbarrier_ptr_ty, llvm_i32_ty]>;
    }

    def int_nvvm_mbarrier_test_wait # shared : NVVMBuiltin,
        Intrinsic<[llvm_i1_ty], [mbarrier_ptr_ty, llvm_i64_ty]>;
  }
}

def int_nvvm_mbarrier_pending_count : NVVMBuiltin,
    NVVMPureIntrinsic<[llvm_i32_ty], [llvm_i64_ty]>;

// mbarrier.{expect_tx/complete_tx}
foreach op = ["expect_tx", "complete_tx"] in {
  foreach scope = ["scope_cta", "scope_cluster"] in {
    foreach space = ["space_cta", "space_cluster"] in {
      defvar suffix = StrJoin<"_", [op, scope, space]>.ret;
      defvar mbar_addr_ty = !if(!eq(space, "space_cta"),
                            llvm_shared_ptr_ty, llvm_shared_cluster_ptr_ty);

      def int_nvvm_mbarrier_ # suffix :
          Intrinsic<[], [mbar_addr_ty, llvm_i32_ty],
          [IntrConvergent, IntrArgMemOnly, IntrNoCallback]>;
    } // space
  } // scope
````
- **L2081 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrWriteMem, IntrArgMemOnly, IntrNoCallback,`.
  **L2081 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrWriteMem, IntrArgMemOnly, IntrNoCallback,`。
- **L2082 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;`.
  **L2082 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;`。
- **L2083 EN**: Blank line separating nearby declarations or logic blocks.
  **L2083 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2084 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2084 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2085 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2085 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2086 EN**: Declares TableGen def `int_nvvm_mbarrier_arrive`.
  **L2086 CN**: 声明 TableGen def `int_nvvm_mbarrier_arrive`。
- **L2087 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [mbarrier_ptr_ty]>;`.
  **L2087 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [mbarrier_ptr_ty]>;`。
- **L2088 EN**: Declares TableGen def `int_nvvm_mbarrier_arrive`.
  **L2088 CN**: 声明 TableGen def `int_nvvm_mbarrier_arrive`。
- **L2089 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i64_ty], [mbarrier_ptr_ty, llvm_i32_ty]>;`.
  **L2089 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i64_ty], [mbarrier_ptr_ty, llvm_i32_ty]>;`。
- **L2090 EN**: Closes the current lexical scope or compound statement.
  **L2090 CN**: 结束当前词法作用域或复合语句块。
- **L2091 EN**: Blank line separating nearby declarations or logic blocks.
  **L2091 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2092 EN**: Declares TableGen def `int_nvvm_mbarrier_test_wait`.
  **L2092 CN**: 声明 TableGen def `int_nvvm_mbarrier_test_wait`。
- **L2093 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i1_ty], [mbarrier_ptr_ty, llvm_i64_ty]>;`.
  **L2093 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i1_ty], [mbarrier_ptr_ty, llvm_i64_ty]>;`。
- **L2094 EN**: Closes the current lexical scope or compound statement.
  **L2094 CN**: 结束当前词法作用域或复合语句块。
- **L2095 EN**: Closes the current lexical scope or compound statement.
  **L2095 CN**: 结束当前词法作用域或复合语句块。
- **L2096 EN**: Blank line separating nearby declarations or logic blocks.
  **L2096 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2097 EN**: Declares TableGen def `int_nvvm_mbarrier_pending_count`.
  **L2097 CN**: 声明 TableGen def `int_nvvm_mbarrier_pending_count`。
- **L2098 EN**: Executes a standalone statement or declaration: `NVVMPureIntrinsic<[llvm_i32_ty], [llvm_i64_ty]>;`.
  **L2098 CN**: 执行一条独立语句或声明：`NVVMPureIntrinsic<[llvm_i32_ty], [llvm_i64_ty]>;`。
- **L2099 EN**: Blank line separating nearby declarations or logic blocks.
  **L2099 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2100 EN**: Comment explains nearby logic, invariants, or intent: `mbarrier.{expect_tx/complete_tx}`.
  **L2100 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mbarrier.{expect_tx/complete_tx}`。
- **L2101 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2101 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2102 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2102 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2103 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2103 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2104 EN**: Initializes variable `suffix` from the right-hand expression.
  **L2104 CN**: 使用右侧表达式初始化变量 `suffix`。
- **L2105 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar mbar_addr_ty = !if(!eq(space, "space_cta"),`.
  **L2105 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar mbar_addr_ty = !if(!eq(space, "space_cta"),`。
- **L2106 EN**: Executes a standalone statement or declaration: `llvm_shared_ptr_ty, llvm_shared_cluster_ptr_ty);`.
  **L2106 CN**: 执行一条独立语句或声明：`llvm_shared_ptr_ty, llvm_shared_cluster_ptr_ty);`。
- **L2107 EN**: Blank line separating nearby declarations or logic blocks.
  **L2107 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2108 EN**: Declares TableGen def `int_nvvm_mbarrier_`.
  **L2108 CN**: 声明 TableGen def `int_nvvm_mbarrier_`。
- **L2109 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[], [mbar_addr_ty, llvm_i32_ty],`.
  **L2109 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[], [mbar_addr_ty, llvm_i32_ty],`。
- **L2110 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrArgMemOnly, IntrNoCallback]>;`.
  **L2110 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrArgMemOnly, IntrNoCallback]>;`。
- **L2111 EN**: Continues the surrounding expression or declaration: `} // space`.
  **L2111 CN**: 继续构造周围的表达式或声明：`} // space`。
- **L2112 EN**: Continues the surrounding expression or declaration: `} // scope`.
  **L2112 CN**: 继续构造周围的表达式或声明：`} // scope`。

### Lines 2113-2144

````tablegen
} // op

// mbarrier.arrive and mbarrier.arrive.expect_tx
// mbarrier.arrive_drop and mbarrier.arrive_drop.expect_tx
foreach op = ["arrive", "arrive_expect_tx",
              "arrive_drop", "arrive_drop_expect_tx"] in {
  foreach scope = ["scope_cta", "scope_cluster"] in {
    foreach space = ["space_cta", "space_cluster"] in {
      defvar suffix = StrJoin<"_", [scope, space]>.ret;
      defvar mbar_addr_ty = !if(!eq(space, "space_cta"),
                            llvm_shared_ptr_ty, llvm_shared_cluster_ptr_ty);
      defvar args_ty = [mbar_addr_ty, // mbar_address_ptr
                        llvm_i32_ty]; // tx-count

      // mbarriers in shared_cluster space cannot return any value.
      defvar mbar_ret_ty = !if(!eq(space, "space_cta"),
                              [llvm_i64_ty], []<LLVMType>);

      def int_nvvm_mbarrier_ # op # "_" # suffix:
          Intrinsic<mbar_ret_ty, args_ty,
          [IntrConvergent, IntrNoCallback]>;
      def int_nvvm_mbarrier_ # op # "_relaxed_" # suffix :
          Intrinsic<mbar_ret_ty, args_ty,
          [IntrConvergent, IntrArgMemOnly, IntrNoCallback]>;
    } // space
  } // scope
} // op

// mbarrier.{test_wait and try_wait}
foreach op = ["test_wait", "try_wait"] in {
  foreach scope = ["scope_cta", "scope_cluster"] in {
    foreach parity = [true, false] in {
````
- **L2113 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2113 CN**: 继续构造周围的表达式或声明：`} // op`。
- **L2114 EN**: Blank line separating nearby declarations or logic blocks.
  **L2114 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2115 EN**: Comment explains nearby logic, invariants, or intent: `mbarrier.arrive and mbarrier.arrive.expect_tx`.
  **L2115 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mbarrier.arrive and mbarrier.arrive.expect_tx`。
- **L2116 EN**: Comment explains nearby logic, invariants, or intent: `mbarrier.arrive_drop and mbarrier.arrive_drop.expect_tx`.
  **L2116 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mbarrier.arrive_drop and mbarrier.arrive_drop.expect_tx`。
- **L2117 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2117 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2118 EN**: Continues the surrounding expression or declaration: `"arrive_drop", "arrive_drop_expect_tx"] in {`.
  **L2118 CN**: 继续构造周围的表达式或声明：`"arrive_drop", "arrive_drop_expect_tx"] in {`。
- **L2119 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2119 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2120 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2120 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2121 EN**: Initializes variable `suffix` from the right-hand expression.
  **L2121 CN**: 使用右侧表达式初始化变量 `suffix`。
- **L2122 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar mbar_addr_ty = !if(!eq(space, "space_cta"),`.
  **L2122 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar mbar_addr_ty = !if(!eq(space, "space_cta"),`。
- **L2123 EN**: Executes a standalone statement or declaration: `llvm_shared_ptr_ty, llvm_shared_cluster_ptr_ty);`.
  **L2123 CN**: 执行一条独立语句或声明：`llvm_shared_ptr_ty, llvm_shared_cluster_ptr_ty);`。
- **L2124 EN**: Continues the surrounding expression or declaration: `defvar args_ty = [mbar_addr_ty, // mbar_address_ptr`.
  **L2124 CN**: 继续构造周围的表达式或声明：`defvar args_ty = [mbar_addr_ty, // mbar_address_ptr`。
- **L2125 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty]; // tx-count`.
  **L2125 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty]; // tx-count`。
- **L2126 EN**: Blank line separating nearby declarations or logic blocks.
  **L2126 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2127 EN**: Comment explains nearby logic, invariants, or intent: `mbarriers in shared_cluster space cannot return any value.`.
  **L2127 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mbarriers in shared_cluster space cannot return any value.`。
- **L2128 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar mbar_ret_ty = !if(!eq(space, "space_cta"),`.
  **L2128 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar mbar_ret_ty = !if(!eq(space, "space_cta"),`。
- **L2129 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty], []<LLVMType>);`.
  **L2129 CN**: 执行一条独立语句或声明：`[llvm_i64_ty], []<LLVMType>);`。
- **L2130 EN**: Blank line separating nearby declarations or logic blocks.
  **L2130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2131 EN**: Declares TableGen def `int_nvvm_mbarrier_`.
  **L2131 CN**: 声明 TableGen def `int_nvvm_mbarrier_`。
- **L2132 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<mbar_ret_ty, args_ty,`.
  **L2132 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<mbar_ret_ty, args_ty,`。
- **L2133 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrNoCallback]>;`.
  **L2133 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrNoCallback]>;`。
- **L2134 EN**: Declares TableGen def `int_nvvm_mbarrier_`.
  **L2134 CN**: 声明 TableGen def `int_nvvm_mbarrier_`。
- **L2135 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<mbar_ret_ty, args_ty,`.
  **L2135 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<mbar_ret_ty, args_ty,`。
- **L2136 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrArgMemOnly, IntrNoCallback]>;`.
  **L2136 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrArgMemOnly, IntrNoCallback]>;`。
- **L2137 EN**: Continues the surrounding expression or declaration: `} // space`.
  **L2137 CN**: 继续构造周围的表达式或声明：`} // space`。
- **L2138 EN**: Continues the surrounding expression or declaration: `} // scope`.
  **L2138 CN**: 继续构造周围的表达式或声明：`} // scope`。
- **L2139 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2139 CN**: 继续构造周围的表达式或声明：`} // op`。
- **L2140 EN**: Blank line separating nearby declarations or logic blocks.
  **L2140 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2141 EN**: Comment explains nearby logic, invariants, or intent: `mbarrier.{test_wait and try_wait}`.
  **L2141 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`mbarrier.{test_wait and try_wait}`。
- **L2142 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2142 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2143 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2143 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2144 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2144 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2145-2176

````tablegen
      foreach time_limit = !if(!eq(op, "try_wait"), [true, false], [false]) in {
        defvar base_args = [llvm_shared_ptr_ty]; // mbar_ptr
        defvar parity_args = !if(parity, [llvm_i32_ty], [llvm_i64_ty]);
        defvar tl_args = !if(time_limit, [llvm_i32_ty], []<LLVMType>);
        defvar args = !listconcat(base_args, parity_args, tl_args);
        defvar tmp_op = StrJoin<"_", [op,
                           !if(parity, "parity", ""),
                           !if(time_limit, "tl", "")]>.ret;
        defvar suffix = StrJoin<"_", [scope, "space_cta"]>.ret;

        def int_nvvm_mbarrier_ # tmp_op # "_" # suffix :
            Intrinsic<[llvm_i1_ty], args,
            [IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback]>;
        def int_nvvm_mbarrier_ # tmp_op # "_relaxed_" # suffix :
            Intrinsic<[llvm_i1_ty], args,
            [IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback,
             IntrArgMemOnly, IntrReadMem]>;
      } // tl
    } // parity
  } // scope
} // op

// Generated within nvvm. Use for ldu on sm_20 or later.  Second arg is the
// pointer's alignment.
let IntrProperties = [IntrReadMem, IntrArgMemOnly, IntrNoCallback, IntrWillReturn, NoCapture<ArgIndex<0>>] in {
  def int_nvvm_ldu_global_i : Intrinsic<[llvm_anyint_ty], [llvm_anyptr_ty, llvm_i32_ty]>;
  def int_nvvm_ldu_global_f : Intrinsic<[llvm_anyfloat_ty], [llvm_anyptr_ty, llvm_i32_ty]>;
  def int_nvvm_ldu_global_p : Intrinsic<[llvm_anyptr_ty], [llvm_anyptr_ty, llvm_i32_ty]>;
}

// Represents an explicit hole in the LLVM IR type system. It may be inserted by
// the compiler in cases where a pointer is of the wrong type. In the backend
````
- **L2145 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2145 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2146 EN**: Continues the surrounding expression or declaration: `defvar base_args = [llvm_shared_ptr_ty]; // mbar_ptr`.
  **L2146 CN**: 继续构造周围的表达式或声明：`defvar base_args = [llvm_shared_ptr_ty]; // mbar_ptr`。
- **L2147 EN**: Initializes variable `parity_args` from the right-hand expression.
  **L2147 CN**: 使用右侧表达式初始化变量 `parity_args`。
- **L2148 EN**: Initializes variable `tl_args` from the right-hand expression.
  **L2148 CN**: 使用右侧表达式初始化变量 `tl_args`。
- **L2149 EN**: Initializes variable `args` from the right-hand expression.
  **L2149 CN**: 使用右侧表达式初始化变量 `args`。
- **L2150 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar tmp_op = StrJoin<"_", [op,`.
  **L2150 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar tmp_op = StrJoin<"_", [op,`。
- **L2151 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(parity, "parity", ""),`.
  **L2151 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(parity, "parity", ""),`。
- **L2152 EN**: Executes a call or declaration centered on `!if`.
  **L2152 CN**: 执行以 `!if` 为核心的调用或声明。
- **L2153 EN**: Initializes variable `suffix` from the right-hand expression.
  **L2153 CN**: 使用右侧表达式初始化变量 `suffix`。
- **L2154 EN**: Blank line separating nearby declarations or logic blocks.
  **L2154 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2155 EN**: Declares TableGen def `int_nvvm_mbarrier_`.
  **L2155 CN**: 声明 TableGen def `int_nvvm_mbarrier_`。
- **L2156 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i1_ty], args,`.
  **L2156 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i1_ty], args,`。
- **L2157 EN**: Executes a standalone statement or declaration: `[IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback]>;`.
  **L2157 CN**: 执行一条独立语句或声明：`[IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback]>;`。
- **L2158 EN**: Declares TableGen def `int_nvvm_mbarrier_`.
  **L2158 CN**: 声明 TableGen def `int_nvvm_mbarrier_`。
- **L2159 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i1_ty], args,`.
  **L2159 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i1_ty], args,`。
- **L2160 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback,`.
  **L2160 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, NoCapture<ArgIndex<0>>, IntrNoCallback,`。
- **L2161 EN**: Executes a standalone statement or declaration: `IntrArgMemOnly, IntrReadMem]>;`.
  **L2161 CN**: 执行一条独立语句或声明：`IntrArgMemOnly, IntrReadMem]>;`。
- **L2162 EN**: Continues the surrounding expression or declaration: `} // tl`.
  **L2162 CN**: 继续构造周围的表达式或声明：`} // tl`。
- **L2163 EN**: Continues the surrounding expression or declaration: `} // parity`.
  **L2163 CN**: 继续构造周围的表达式或声明：`} // parity`。
- **L2164 EN**: Continues the surrounding expression or declaration: `} // scope`.
  **L2164 CN**: 继续构造周围的表达式或声明：`} // scope`。
- **L2165 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2165 CN**: 继续构造周围的表达式或声明：`} // op`。
- **L2166 EN**: Blank line separating nearby declarations or logic blocks.
  **L2166 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2167 EN**: Comment explains nearby logic, invariants, or intent: `Generated within nvvm. Use for ldu on sm_20 or later.  Second arg is the`.
  **L2167 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Generated within nvvm. Use for ldu on sm_20 or later.  Second arg is the`。
- **L2168 EN**: Comment explains nearby logic, invariants, or intent: `pointer's alignment.`.
  **L2168 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`pointer's alignment.`。
- **L2169 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2169 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2170 EN**: Declares TableGen def `int_nvvm_ldu_global_i`.
  **L2170 CN**: 声明 TableGen def `int_nvvm_ldu_global_i`。
- **L2171 EN**: Declares TableGen def `int_nvvm_ldu_global_f`.
  **L2171 CN**: 声明 TableGen def `int_nvvm_ldu_global_f`。
- **L2172 EN**: Declares TableGen def `int_nvvm_ldu_global_p`.
  **L2172 CN**: 声明 TableGen def `int_nvvm_ldu_global_p`。
- **L2173 EN**: Closes the current lexical scope or compound statement.
  **L2173 CN**: 结束当前词法作用域或复合语句块。
- **L2174 EN**: Blank line separating nearby declarations or logic blocks.
  **L2174 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2175 EN**: Comment explains nearby logic, invariants, or intent: `Represents an explicit hole in the LLVM IR type system. It may be inserted by`.
  **L2175 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Represents an explicit hole in the LLVM IR type system. It may be inserted by`。
- **L2176 EN**: Comment explains nearby logic, invariants, or intent: `the compiler in cases where a pointer is of the wrong type. In the backend`.
  **L2176 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`the compiler in cases where a pointer is of the wrong type. In the backend`。

### Lines 2177-2208

````tablegen
// this intrinsic will be folded away and not equate to any instruction. It
// should not be used by any frontend and should only be considered well defined
// when added in the following cases:
//
//  - NVPTXLowerArgs: When wrapping a byval pointer argument to a kernel
//    function to convert the address space from generic (0) to param (101).
//    This accounts for the fact that the parameter symbols will occupy this
//    space when lowered during ISel.
//
def int_nvvm_internal_addrspace_wrap :
  NVVMPureIntrinsic<[llvm_anyptr_ty], [llvm_anyptr_ty],
                [NoUndef<ArgIndex<0>>, NoUndef<RetIndex>]>;

// Move intrinsics, used in nvvm internally

let IntrProperties = [IntrNoMem] in {
  def int_nvvm_move_i16 : DefaultAttrsIntrinsic<[llvm_i16_ty], [llvm_i16_ty]>;
  def int_nvvm_move_i32 : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i32_ty]>;
  def int_nvvm_move_i64 : DefaultAttrsIntrinsic<[llvm_i64_ty], [llvm_i64_ty]>;
  def int_nvvm_move_float : DefaultAttrsIntrinsic<[llvm_float_ty], [llvm_float_ty]>;
  def int_nvvm_move_double : DefaultAttrsIntrinsic<[llvm_double_ty], [llvm_double_ty]>;
  def int_nvvm_move_ptr : DefaultAttrsIntrinsic<[llvm_anyptr_ty], [llvm_anyptr_ty]>;
}

// For getting the handle from a texture or surface variable
def int_nvvm_texsurf_handle
  : NVVMPureIntrinsic<[llvm_i64_ty], [llvm_metadata_ty, llvm_anyptr_ty]>;
def int_nvvm_texsurf_handle_internal
  : NVVMPureIntrinsic<[llvm_i64_ty], [llvm_anyptr_ty]>;

/// Error / Warn
def int_nvvm_compiler_error : Intrinsic<[], [llvm_anyptr_ty]>;
````
- **L2177 EN**: Comment explains nearby logic, invariants, or intent: `this intrinsic will be folded away and not equate to any instruction. It`.
  **L2177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`this intrinsic will be folded away and not equate to any instruction. It`。
- **L2178 EN**: Comment explains nearby logic, invariants, or intent: `should not be used by any frontend and should only be considered well defined`.
  **L2178 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`should not be used by any frontend and should only be considered well defined`。
- **L2179 EN**: Comment explains nearby logic, invariants, or intent: `when added in the following cases:`.
  **L2179 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`when added in the following cases:`。
- **L2180 EN**: Separator comment used for visual grouping.
  **L2180 CN**: 用于视觉分组的分隔注释。
- **L2181 EN**: Comment explains nearby logic, invariants, or intent: `- NVPTXLowerArgs: When wrapping a byval pointer argument to a kernel`.
  **L2181 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`- NVPTXLowerArgs: When wrapping a byval pointer argument to a kernel`。
- **L2182 EN**: Comment explains nearby logic, invariants, or intent: `function to convert the address space from generic (0) to param (101).`.
  **L2182 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`function to convert the address space from generic (0) to param (101).`。
- **L2183 EN**: Comment explains nearby logic, invariants, or intent: `This accounts for the fact that the parameter symbols will occupy this`.
  **L2183 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`This accounts for the fact that the parameter symbols will occupy this`。
- **L2184 EN**: Comment explains nearby logic, invariants, or intent: `space when lowered during ISel.`.
  **L2184 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`space when lowered during ISel.`。
- **L2185 EN**: Separator comment used for visual grouping.
  **L2185 CN**: 用于视觉分组的分隔注释。
- **L2186 EN**: Declares TableGen def `int_nvvm_internal_addrspace_wrap`.
  **L2186 CN**: 声明 TableGen def `int_nvvm_internal_addrspace_wrap`。
- **L2187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `NVVMPureIntrinsic<[llvm_anyptr_ty], [llvm_anyptr_ty],`.
  **L2187 CN**: 继续一个多行参数列表、初始化器或聚合项：`NVVMPureIntrinsic<[llvm_anyptr_ty], [llvm_anyptr_ty],`。
- **L2188 EN**: Executes a standalone statement or declaration: `[NoUndef<ArgIndex<0>>, NoUndef<RetIndex>]>;`.
  **L2188 CN**: 执行一条独立语句或声明：`[NoUndef<ArgIndex<0>>, NoUndef<RetIndex>]>;`。
- **L2189 EN**: Blank line separating nearby declarations or logic blocks.
  **L2189 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2190 EN**: Comment explains nearby logic, invariants, or intent: `Move intrinsics, used in nvvm internally`.
  **L2190 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Move intrinsics, used in nvvm internally`。
- **L2191 EN**: Blank line separating nearby declarations or logic blocks.
  **L2191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2192 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2192 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2193 EN**: Declares TableGen def `int_nvvm_move_i16`.
  **L2193 CN**: 声明 TableGen def `int_nvvm_move_i16`。
- **L2194 EN**: Declares TableGen def `int_nvvm_move_i32`.
  **L2194 CN**: 声明 TableGen def `int_nvvm_move_i32`。
- **L2195 EN**: Declares TableGen def `int_nvvm_move_i64`.
  **L2195 CN**: 声明 TableGen def `int_nvvm_move_i64`。
- **L2196 EN**: Declares TableGen def `int_nvvm_move_float`.
  **L2196 CN**: 声明 TableGen def `int_nvvm_move_float`。
- **L2197 EN**: Declares TableGen def `int_nvvm_move_double`.
  **L2197 CN**: 声明 TableGen def `int_nvvm_move_double`。
- **L2198 EN**: Declares TableGen def `int_nvvm_move_ptr`.
  **L2198 CN**: 声明 TableGen def `int_nvvm_move_ptr`。
- **L2199 EN**: Closes the current lexical scope or compound statement.
  **L2199 CN**: 结束当前词法作用域或复合语句块。
- **L2200 EN**: Blank line separating nearby declarations or logic blocks.
  **L2200 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2201 EN**: Comment explains nearby logic, invariants, or intent: `For getting the handle from a texture or surface variable`.
  **L2201 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For getting the handle from a texture or surface variable`。
- **L2202 EN**: Declares TableGen def `int_nvvm_texsurf_handle`.
  **L2202 CN**: 声明 TableGen def `int_nvvm_texsurf_handle`。
- **L2203 EN**: Executes a standalone statement or declaration: `: NVVMPureIntrinsic<[llvm_i64_ty], [llvm_metadata_ty, llvm_anyptr_ty]>;`.
  **L2203 CN**: 执行一条独立语句或声明：`: NVVMPureIntrinsic<[llvm_i64_ty], [llvm_metadata_ty, llvm_anyptr_ty]>;`。
- **L2204 EN**: Declares TableGen def `int_nvvm_texsurf_handle_internal`.
  **L2204 CN**: 声明 TableGen def `int_nvvm_texsurf_handle_internal`。
- **L2205 EN**: Executes a standalone statement or declaration: `: NVVMPureIntrinsic<[llvm_i64_ty], [llvm_anyptr_ty]>;`.
  **L2205 CN**: 执行一条独立语句或声明：`: NVVMPureIntrinsic<[llvm_i64_ty], [llvm_anyptr_ty]>;`。
- **L2206 EN**: Blank line separating nearby declarations or logic blocks.
  **L2206 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2207 EN**: Comment explains nearby logic, invariants, or intent: `Error / Warn`.
  **L2207 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Error / Warn`。
- **L2208 EN**: Declares TableGen def `int_nvvm_compiler_error`.
  **L2208 CN**: 声明 TableGen def `int_nvvm_compiler_error`。

### Lines 2209-2240

````tablegen
def int_nvvm_compiler_warn : Intrinsic<[], [llvm_anyptr_ty]>;

def int_nvvm_reflect : NVVMBuiltin, NVVMPureIntrinsic<[llvm_i32_ty], [llvm_ptr_ty]>;

// isspacep.{const, global, local, shared}
foreach space = ["const", "global", "local", "shared", "shared_cluster"] in
  def int_nvvm_isspacep_ # space : NVVMBuiltin,
    NVVMPureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty], [NoCapture<ArgIndex<0>>]>;

//
// Texture Fetch
//
let IntrProperties = [IntrReadMem, IntrNoCallback, IntrNoFree, IntrWillReturn] in {
  foreach is_unified = [true, false] in {
    defvar mode = !if(is_unified, "_unified", "");
    defvar addr_args = !if(is_unified, [llvm_i64_ty], [llvm_i64_ty, llvm_i64_ty]);

    foreach vec = [V4F32, V4S32, V4U32] in {
      foreach is_array = [true, false] in {
        defvar array = !if(is_array, "_array", "");
        defvar array_args = !if(is_array, [llvm_i32_ty], []<LLVMType>);
        defvar base_args = !listconcat(addr_args, array_args);

        def int_nvvm_tex # mode # _1d # array # _ # vec.Name # _s32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_i32_ty, 1)>;
        def int_nvvm_tex # mode # _1d # array # _ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 1)>;
        def int_nvvm_tex # mode # _1d # array # _level_ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 2)>;
        def int_nvvm_tex # mode # _1d # array # _grad_ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 3)>;

````
- **L2209 EN**: Declares TableGen def `int_nvvm_compiler_warn`.
  **L2209 CN**: 声明 TableGen def `int_nvvm_compiler_warn`。
- **L2210 EN**: Blank line separating nearby declarations or logic blocks.
  **L2210 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2211 EN**: Declares TableGen def `int_nvvm_reflect`.
  **L2211 CN**: 声明 TableGen def `int_nvvm_reflect`。
- **L2212 EN**: Blank line separating nearby declarations or logic blocks.
  **L2212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2213 EN**: Comment explains nearby logic, invariants, or intent: `isspacep.{const, global, local, shared}`.
  **L2213 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`isspacep.{const, global, local, shared}`。
- **L2214 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2214 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2215 EN**: Declares TableGen def `int_nvvm_isspacep_`.
  **L2215 CN**: 声明 TableGen def `int_nvvm_isspacep_`。
- **L2216 EN**: Executes a standalone statement or declaration: `NVVMPureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty], [NoCapture<ArgIndex<0>>]>;`.
  **L2216 CN**: 执行一条独立语句或声明：`NVVMPureIntrinsic<[llvm_i1_ty], [llvm_ptr_ty], [NoCapture<ArgIndex<0>>]>;`。
- **L2217 EN**: Blank line separating nearby declarations or logic blocks.
  **L2217 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2218 EN**: Separator comment used for visual grouping.
  **L2218 CN**: 用于视觉分组的分隔注释。
- **L2219 EN**: Comment explains nearby logic, invariants, or intent: `Texture Fetch`.
  **L2219 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Texture Fetch`。
- **L2220 EN**: Separator comment used for visual grouping.
  **L2220 CN**: 用于视觉分组的分隔注释。
- **L2221 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2221 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2222 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2222 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2223 EN**: Initializes variable `mode` from the right-hand expression.
  **L2223 CN**: 使用右侧表达式初始化变量 `mode`。
- **L2224 EN**: Initializes variable `addr_args` from the right-hand expression.
  **L2224 CN**: 使用右侧表达式初始化变量 `addr_args`。
- **L2225 EN**: Blank line separating nearby declarations or logic blocks.
  **L2225 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2226 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2226 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2227 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2227 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2228 EN**: Initializes variable `array` from the right-hand expression.
  **L2228 CN**: 使用右侧表达式初始化变量 `array`。
- **L2229 EN**: Initializes variable `array_args` from the right-hand expression.
  **L2229 CN**: 使用右侧表达式初始化变量 `array_args`。
- **L2230 EN**: Initializes variable `base_args` from the right-hand expression.
  **L2230 CN**: 使用右侧表达式初始化变量 `base_args`。
- **L2231 EN**: Blank line separating nearby declarations or logic blocks.
  **L2231 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2232 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2232 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2233 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2233 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2234 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2234 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2235 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2235 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2236 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2236 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2237 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2237 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2238 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2238 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2239 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2239 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2240 EN**: Blank line separating nearby declarations or logic blocks.
  **L2240 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 2241-2272

````tablegen
        def int_nvvm_tex # mode # _2d # array # _ # vec.Name # _s32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_i32_ty, 2)>;
        def int_nvvm_tex # mode # _2d # array # _ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 2)>;
        def int_nvvm_tex # mode # _2d # array # _level_ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 3)>;
        def int_nvvm_tex # mode # _2d # array # _grad_ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 6)>;

        if !not(is_array) then {
          def int_nvvm_tex # mode # _3d_ # vec.Name # _s32
            : Intrinsic<vec.Types, base_args # !listsplat(llvm_i32_ty, 3)>;
          def int_nvvm_tex # mode # _3d_ # vec.Name # _f32
            : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 3)>;
          def int_nvvm_tex # mode # _3d_level_ # vec.Name # _f32
            : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 4)>;
          def int_nvvm_tex # mode # _3d_grad_ # vec.Name # _f32
            : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 9)>;
        }

        def int_nvvm_tex # mode # _cube # array # _ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 3)>;
        def int_nvvm_tex # mode # _cube # array # _level_ # vec.Name # _f32
          : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 4)>;

        if is_unified then
          def int_nvvm_tex # mode # _cube # array # _grad_ # vec.Name # _f32
            : Intrinsic<vec.Types, base_args # !listsplat(llvm_float_ty, 9)>;
      } // is_array

      foreach comp = ["r", "g", "b", "a"] in {
        def int_nvvm_tld4 # mode # _ # comp # _2d_ # vec.Name # _f32
````
- **L2241 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2241 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2242 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2242 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2243 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2243 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2244 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2244 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2245 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2245 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2246 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2246 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2247 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2247 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2248 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2248 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2249 EN**: Blank line separating nearby declarations or logic blocks.
  **L2249 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2250 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2250 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2251 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2251 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2252 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2252 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2253 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2253 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2254 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2254 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2255 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2255 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2256 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2256 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2257 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2257 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2258 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2258 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2259 EN**: Closes the current lexical scope or compound statement.
  **L2259 CN**: 结束当前词法作用域或复合语句块。
- **L2260 EN**: Blank line separating nearby declarations or logic blocks.
  **L2260 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2261 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2261 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2262 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2262 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2263 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2263 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2264 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2264 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2265 EN**: Blank line separating nearby declarations or logic blocks.
  **L2265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2266 EN**: Continues the surrounding expression or declaration: `if is_unified then`.
  **L2266 CN**: 继续构造周围的表达式或声明：`if is_unified then`。
- **L2267 EN**: Declares TableGen def `int_nvvm_tex`.
  **L2267 CN**: 声明 TableGen def `int_nvvm_tex`。
- **L2268 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2268 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2269 EN**: Continues the surrounding expression or declaration: `} // is_array`.
  **L2269 CN**: 继续构造周围的表达式或声明：`} // is_array`。
- **L2270 EN**: Blank line separating nearby declarations or logic blocks.
  **L2270 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2271 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2271 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2272 EN**: Declares TableGen def `int_nvvm_tld4`.
  **L2272 CN**: 声明 TableGen def `int_nvvm_tld4`。

### Lines 2273-2304

````tablegen
          : Intrinsic<vec.Types, addr_args # !listsplat(llvm_float_ty, 2)>;
      } // comp
    } // vec
  } // is_unified
} // IntrProperties = [IntrReadMem]

//=== Surface Load
foreach clamp = ["clamp", "trap", "zero"] in {
  foreach vec = [TV_I8, TV_I16, TV_I32, TV_I64,
                TV_V2I8, TV_V2I16, TV_V2I32, TV_V2I64,
                TV_V4I8, TV_V4I16, TV_V4I32] in {

    let IntrProperties = [IntrNoCallback, IntrNoFree, IntrReadMem]
                         # !if(!ne(clamp, "trap"), [IntrWillReturn], []<IntrinsicProperty>) in {

      def int_nvvm_suld_1d_ # vec.Name # _ # clamp
        : Intrinsic<vec.Types,
                    [llvm_i64_ty, llvm_i32_ty]>;

      def int_nvvm_suld_1d_array_ # vec.Name # _ # clamp
        : Intrinsic<vec.Types,
                    [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty]>;

      def int_nvvm_suld_2d_ # vec.Name # _ # clamp
        : Intrinsic<vec.Types,
                    [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty]>;

      def int_nvvm_suld_2d_array_ # vec.Name # _ # clamp
        : Intrinsic<vec.Types,
                    [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;

      def int_nvvm_suld_3d_ # vec.Name # _ # clamp
````
- **L2273 EN**: Executes a call or declaration centered on `!listsplat`.
  **L2273 CN**: 执行以 `!listsplat` 为核心的调用或声明。
- **L2274 EN**: Continues the surrounding expression or declaration: `} // comp`.
  **L2274 CN**: 继续构造周围的表达式或声明：`} // comp`。
- **L2275 EN**: Continues the surrounding expression or declaration: `} // vec`.
  **L2275 CN**: 继续构造周围的表达式或声明：`} // vec`。
- **L2276 EN**: Continues the surrounding expression or declaration: `} // is_unified`.
  **L2276 CN**: 继续构造周围的表达式或声明：`} // is_unified`。
- **L2277 EN**: Continues the surrounding expression or declaration: `} // IntrProperties = [IntrReadMem]`.
  **L2277 CN**: 继续构造周围的表达式或声明：`} // IntrProperties = [IntrReadMem]`。
- **L2278 EN**: Blank line separating nearby declarations or logic blocks.
  **L2278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2279 EN**: Banner comment marking a file or section boundary.
  **L2279 CN**: 横幅注释，用于标记文件或章节边界。
- **L2280 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2280 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2281 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2281 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TV_V2I8, TV_V2I16, TV_V2I32, TV_V2I64,`.
  **L2282 CN**: 继续一个多行参数列表、初始化器或聚合项：`TV_V2I8, TV_V2I16, TV_V2I32, TV_V2I64,`。
- **L2283 EN**: Continues the surrounding expression or declaration: `TV_V4I8, TV_V4I16, TV_V4I32] in {`.
  **L2283 CN**: 继续构造周围的表达式或声明：`TV_V4I8, TV_V4I16, TV_V4I32] in {`。
- **L2284 EN**: Blank line separating nearby declarations or logic blocks.
  **L2284 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2285 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2285 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2286 EN**: Starts a function, method, lambda, or structured scope: `# !if(!ne(clamp, "trap"), [IntrWillReturn], []<IntrinsicProperty>) in {`.
  **L2286 CN**: 开始一个函数、方法、lambda 或结构化作用域：`# !if(!ne(clamp, "trap"), [IntrWillReturn], []<IntrinsicProperty>) in {`。
- **L2287 EN**: Blank line separating nearby declarations or logic blocks.
  **L2287 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2288 EN**: Declares TableGen def `int_nvvm_suld_1d_`.
  **L2288 CN**: 声明 TableGen def `int_nvvm_suld_1d_`。
- **L2289 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<vec.Types,`.
  **L2289 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<vec.Types,`。
- **L2290 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i32_ty]>;`.
  **L2290 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i32_ty]>;`。
- **L2291 EN**: Blank line separating nearby declarations or logic blocks.
  **L2291 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2292 EN**: Declares TableGen def `int_nvvm_suld_1d_array_`.
  **L2292 CN**: 声明 TableGen def `int_nvvm_suld_1d_array_`。
- **L2293 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<vec.Types,`.
  **L2293 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<vec.Types,`。
- **L2294 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2294 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2295 EN**: Blank line separating nearby declarations or logic blocks.
  **L2295 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2296 EN**: Declares TableGen def `int_nvvm_suld_2d_`.
  **L2296 CN**: 声明 TableGen def `int_nvvm_suld_2d_`。
- **L2297 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<vec.Types,`.
  **L2297 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<vec.Types,`。
- **L2298 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2298 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2299 EN**: Blank line separating nearby declarations or logic blocks.
  **L2299 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2300 EN**: Declares TableGen def `int_nvvm_suld_2d_array_`.
  **L2300 CN**: 声明 TableGen def `int_nvvm_suld_2d_array_`。
- **L2301 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<vec.Types,`.
  **L2301 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<vec.Types,`。
- **L2302 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2302 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2303 EN**: Blank line separating nearby declarations or logic blocks.
  **L2303 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2304 EN**: Declares TableGen def `int_nvvm_suld_3d_`.
  **L2304 CN**: 声明 TableGen def `int_nvvm_suld_3d_`。

### Lines 2305-2336

````tablegen
        : Intrinsic<vec.Types,
                    [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;
    }
  } // vec
} // clamp

//===- Texture Query ------------------------------------------------------===//

foreach query = ["channel_order", "channel_data_type", "width", "height",
                 "depth", "array_size", "num_samples", "num_mipmap_levels"] in
  def int_nvvm_txq_ # query : NVVMBuiltin,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty], [IntrNoMem]>;

//===- Surface Query ------------------------------------------------------===//

foreach query = ["channel_order", "channel_data_type", "width", "height",
                 "depth", "array_size"] in
  def int_nvvm_suq_ # query : NVVMBuiltin,
    DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty], [IntrNoMem]>;

//===- Handle Query -------------------------------------------------------===//

foreach type = ["sampler", "surface", "texture"] in
  def int_nvvm_istypep_ # type : NVVMBuiltin,
    DefaultAttrsIntrinsic<[llvm_i1_ty], [llvm_i64_ty], [IntrNoMem]>;

//===- Surface Stores -----------------------------------------------------===//

multiclass SurfaceStoreIntrinsics<string clamp, TexVector vec> {
  let IntrProperties = [IntrNoCallback, IntrNoFree, IntrWriteMem] #
                       !if(!ne(clamp, "trap"), [IntrWillReturn], []<IntrinsicProperty>) in {
    def _1d_ # vec.Name # _ # clamp : NVVMBuiltin,
````
- **L2305 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<vec.Types,`.
  **L2305 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<vec.Types,`。
- **L2306 EN**: Executes a standalone statement or declaration: `[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`.
  **L2306 CN**: 执行一条独立语句或声明：`[llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty]>;`。
- **L2307 EN**: Closes the current lexical scope or compound statement.
  **L2307 CN**: 结束当前词法作用域或复合语句块。
- **L2308 EN**: Continues the surrounding expression or declaration: `} // vec`.
  **L2308 CN**: 继续构造周围的表达式或声明：`} // vec`。
- **L2309 EN**: Continues the surrounding expression or declaration: `} // clamp`.
  **L2309 CN**: 继续构造周围的表达式或声明：`} // clamp`。
- **L2310 EN**: Blank line separating nearby declarations or logic blocks.
  **L2310 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2311 EN**: Banner comment marking a file or section boundary.
  **L2311 CN**: 横幅注释，用于标记文件或章节边界。
- **L2312 EN**: Blank line separating nearby declarations or logic blocks.
  **L2312 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2313 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2313 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2314 EN**: Continues the surrounding expression or declaration: `"depth", "array_size", "num_samples", "num_mipmap_levels"] in`.
  **L2314 CN**: 继续构造周围的表达式或声明：`"depth", "array_size", "num_samples", "num_mipmap_levels"] in`。
- **L2315 EN**: Declares TableGen def `int_nvvm_txq_`.
  **L2315 CN**: 声明 TableGen def `int_nvvm_txq_`。
- **L2316 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L2316 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L2317 EN**: Blank line separating nearby declarations or logic blocks.
  **L2317 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2318 EN**: Banner comment marking a file or section boundary.
  **L2318 CN**: 横幅注释，用于标记文件或章节边界。
- **L2319 EN**: Blank line separating nearby declarations or logic blocks.
  **L2319 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2320 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2320 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2321 EN**: Continues the surrounding expression or declaration: `"depth", "array_size"] in`.
  **L2321 CN**: 继续构造周围的表达式或声明：`"depth", "array_size"] in`。
- **L2322 EN**: Declares TableGen def `int_nvvm_suq_`.
  **L2322 CN**: 声明 TableGen def `int_nvvm_suq_`。
- **L2323 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L2323 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L2324 EN**: Blank line separating nearby declarations or logic blocks.
  **L2324 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2325 EN**: Banner comment marking a file or section boundary.
  **L2325 CN**: 横幅注释，用于标记文件或章节边界。
- **L2326 EN**: Blank line separating nearby declarations or logic blocks.
  **L2326 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2327 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2327 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2328 EN**: Declares TableGen def `int_nvvm_istypep_`.
  **L2328 CN**: 声明 TableGen def `int_nvvm_istypep_`。
- **L2329 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[llvm_i1_ty], [llvm_i64_ty], [IntrNoMem]>;`.
  **L2329 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[llvm_i1_ty], [llvm_i64_ty], [IntrNoMem]>;`。
- **L2330 EN**: Blank line separating nearby declarations or logic blocks.
  **L2330 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2331 EN**: Banner comment marking a file or section boundary.
  **L2331 CN**: 横幅注释，用于标记文件或章节边界。
- **L2332 EN**: Blank line separating nearby declarations or logic blocks.
  **L2332 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2333 EN**: Declares TableGen multiclass `SurfaceStoreIntrinsics`.
  **L2333 CN**: 声明 TableGen multiclass `SurfaceStoreIntrinsics`。
- **L2334 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2334 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2335 EN**: Starts a function, method, lambda, or structured scope: `!if(!ne(clamp, "trap"), [IntrWillReturn], []<IntrinsicProperty>) in {`.
  **L2335 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!if(!ne(clamp, "trap"), [IntrWillReturn], []<IntrinsicProperty>) in {`。
- **L2336 EN**: Declares TableGen def `_1d_`.
  **L2336 CN**: 声明 TableGen def `_1d_`。

### Lines 2337-2368

````tablegen
        Intrinsic<[], [llvm_i64_ty, llvm_i32_ty] # vec.Types>;

    def _1d_array_ # vec.Name # _ # clamp : NVVMBuiltin,
        Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;

    def _2d_ # vec.Name # _ # clamp : NVVMBuiltin,
        Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;

    def _2d_array_ # vec.Name # _ # clamp : NVVMBuiltin,
        Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;

    def _3d_ # vec.Name # _ # clamp : NVVMBuiltin,
        Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;
  }
}

// Unformatted
foreach clamp = ["clamp", "trap", "zero"] in
  foreach vec = [TV_I8, TV_I16, TV_I32, TV_I64,
                 TV_V2I8, TV_V2I16, TV_V2I32, TV_V2I64,
                 TV_V4I8, TV_V4I16, TV_V4I32] in
    defm int_nvvm_sust_b : SurfaceStoreIntrinsics<clamp, vec>;

// Formatted
foreach vec = [TV_I8, TV_I16, TV_I32,
               TV_V2I8, TV_V2I16, TV_V2I32,
               TV_V4I8, TV_V4I16, TV_V4I32] in
  defm int_nvvm_sust_p : SurfaceStoreIntrinsics<"trap", vec>;

//
// Accessing special registers.
//
````
- **L2337 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty, llvm_i32_ty] # vec.Types>;`.
  **L2337 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty, llvm_i32_ty] # vec.Types>;`。
- **L2338 EN**: Blank line separating nearby declarations or logic blocks.
  **L2338 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2339 EN**: Declares TableGen def `_1d_array_`.
  **L2339 CN**: 声明 TableGen def `_1d_array_`。
- **L2340 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`.
  **L2340 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`。
- **L2341 EN**: Blank line separating nearby declarations or logic blocks.
  **L2341 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2342 EN**: Declares TableGen def `_2d_`.
  **L2342 CN**: 声明 TableGen def `_2d_`。
- **L2343 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`.
  **L2343 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`。
- **L2344 EN**: Blank line separating nearby declarations or logic blocks.
  **L2344 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2345 EN**: Declares TableGen def `_2d_array_`.
  **L2345 CN**: 声明 TableGen def `_2d_array_`。
- **L2346 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`.
  **L2346 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`。
- **L2347 EN**: Blank line separating nearby declarations or logic blocks.
  **L2347 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2348 EN**: Declares TableGen def `_3d_`.
  **L2348 CN**: 声明 TableGen def `_3d_`。
- **L2349 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`.
  **L2349 CN**: 执行一条独立语句或声明：`Intrinsic<[], [llvm_i64_ty, llvm_i32_ty, llvm_i32_ty, llvm_i32_ty] # vec.Types>;`。
- **L2350 EN**: Closes the current lexical scope or compound statement.
  **L2350 CN**: 结束当前词法作用域或复合语句块。
- **L2351 EN**: Closes the current lexical scope or compound statement.
  **L2351 CN**: 结束当前词法作用域或复合语句块。
- **L2352 EN**: Blank line separating nearby declarations or logic blocks.
  **L2352 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2353 EN**: Comment explains nearby logic, invariants, or intent: `Unformatted`.
  **L2353 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Unformatted`。
- **L2354 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2354 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2355 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2355 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2356 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TV_V2I8, TV_V2I16, TV_V2I32, TV_V2I64,`.
  **L2356 CN**: 继续一个多行参数列表、初始化器或聚合项：`TV_V2I8, TV_V2I16, TV_V2I32, TV_V2I64,`。
- **L2357 EN**: Continues the surrounding expression or declaration: `TV_V4I8, TV_V4I16, TV_V4I32] in`.
  **L2357 CN**: 继续构造周围的表达式或声明：`TV_V4I8, TV_V4I16, TV_V4I32] in`。
- **L2358 EN**: Declares TableGen defm `int_nvvm_sust_b`.
  **L2358 CN**: 声明 TableGen defm `int_nvvm_sust_b`。
- **L2359 EN**: Blank line separating nearby declarations or logic blocks.
  **L2359 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2360 EN**: Comment explains nearby logic, invariants, or intent: `Formatted`.
  **L2360 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Formatted`。
- **L2361 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2361 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2362 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `TV_V2I8, TV_V2I16, TV_V2I32,`.
  **L2362 CN**: 继续一个多行参数列表、初始化器或聚合项：`TV_V2I8, TV_V2I16, TV_V2I32,`。
- **L2363 EN**: Continues the surrounding expression or declaration: `TV_V4I8, TV_V4I16, TV_V4I32] in`.
  **L2363 CN**: 继续构造周围的表达式或声明：`TV_V4I8, TV_V4I16, TV_V4I32] in`。
- **L2364 EN**: Declares TableGen defm `int_nvvm_sust_p`.
  **L2364 CN**: 声明 TableGen defm `int_nvvm_sust_p`。
- **L2365 EN**: Blank line separating nearby declarations or logic blocks.
  **L2365 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2366 EN**: Separator comment used for visual grouping.
  **L2366 CN**: 用于视觉分组的分隔注释。
- **L2367 EN**: Comment explains nearby logic, invariants, or intent: `Accessing special registers.`.
  **L2367 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Accessing special registers.`。
- **L2368 EN**: Separator comment used for visual grouping.
  **L2368 CN**: 用于视觉分组的分隔注释。

### Lines 2369-2400

````tablegen
class PTXReadSRegIntrinsicNB_r32<list<IntrinsicProperty> properties = [], string name = "">
  : NVVMPureIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>] # properties,
                      !if(!eq(name, ""), "", !strconcat("llvm.nvvm.read.ptx.sreg.", name))>;

class PTXReadSRegIntrinsic_r32<list<IntrinsicProperty> properties = []>
  : PTXReadSRegIntrinsicNB_r32<properties>, NVVMBuiltin;

multiclass PTXReadSRegIntrinsic_v4i32<list<list<IntrinsicProperty>> properties = [[], [], [], []]> {
  assert !eq(!size(properties), 4), "properties must be a list of 4 lists";
  defvar suffixes = ["_x", "_y", "_z", "_w"];
  foreach i = !range(suffixes) in
    def suffixes[i] : PTXReadSRegIntrinsic_r32<properties[i]>;
}

// Same, but without automatic clang builtins. It will be used for
// registers that require particular GPU or PTX version.
multiclass PTXReadSRegIntrinsicNB_v4i32<list<list<IntrinsicProperty>> properties = [[], [], [], []]> {
  assert !eq(!size(properties), 4), "properties must be a list of 4 lists";
  defvar suffixes = ["_x", "_y", "_z", "_w"];
  foreach i = !range(suffixes) in
    def suffixes[i] : PTXReadSRegIntrinsicNB_r32<properties[i]>;
}

// Intrinsics to read registers with non-constant values. E.g. the values that
// do change over the kernel lifetime. Such reads should not be CSE'd.
class PTXReadNCSRegIntrinsic<LLVMType ty>
  : Intrinsic<[ty], [], [IntrInaccessibleMemOnly, IntrNoCallback,
                         IntrNoFree, IntrWillReturn, NoUndef<RetIndex>]>,
    NVVMBuiltin;

defvar MAX_BLOCK_ID_RANGE = [[Range<RetIndex, 0, MAX_BLOCK_SIZE_X>],
                             [Range<RetIndex, 0, MAX_BLOCK_SIZE_Y>],
````
- **L2369 EN**: Declares class `PTXReadSRegIntrinsicNB_r32<list<IntrinsicProperty>`.
  **L2369 CN**: 声明 class `PTXReadSRegIntrinsicNB_r32<list<IntrinsicProperty>`。
- **L2370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: NVVMPureIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>] # properties,`.
  **L2370 CN**: 继续一个多行参数列表、初始化器或聚合项：`: NVVMPureIntrinsic<[llvm_i32_ty], [], [NoUndef<RetIndex>] # properties,`。
- **L2371 EN**: Executes a call or declaration centered on `!if`.
  **L2371 CN**: 执行以 `!if` 为核心的调用或声明。
- **L2372 EN**: Blank line separating nearby declarations or logic blocks.
  **L2372 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2373 EN**: Declares class `PTXReadSRegIntrinsic_r32<list<IntrinsicProperty>`.
  **L2373 CN**: 声明 class `PTXReadSRegIntrinsic_r32<list<IntrinsicProperty>`。
- **L2374 EN**: Executes a standalone statement or declaration: `: PTXReadSRegIntrinsicNB_r32<properties>, NVVMBuiltin;`.
  **L2374 CN**: 执行一条独立语句或声明：`: PTXReadSRegIntrinsicNB_r32<properties>, NVVMBuiltin;`。
- **L2375 EN**: Blank line separating nearby declarations or logic blocks.
  **L2375 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2376 EN**: Declares TableGen multiclass `PTXReadSRegIntrinsic_v4i32`.
  **L2376 CN**: 声明 TableGen multiclass `PTXReadSRegIntrinsic_v4i32`。
- **L2377 EN**: Executes a call or declaration centered on `!eq`.
  **L2377 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L2378 EN**: Initializes variable `suffixes` from the right-hand expression.
  **L2378 CN**: 使用右侧表达式初始化变量 `suffixes`。
- **L2379 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2379 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2380 EN**: Declares TableGen def `suffixes[i]`.
  **L2380 CN**: 声明 TableGen def `suffixes[i]`。
- **L2381 EN**: Closes the current lexical scope or compound statement.
  **L2381 CN**: 结束当前词法作用域或复合语句块。
- **L2382 EN**: Blank line separating nearby declarations or logic blocks.
  **L2382 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2383 EN**: Comment explains nearby logic, invariants, or intent: `Same, but without automatic clang builtins. It will be used for`.
  **L2383 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Same, but without automatic clang builtins. It will be used for`。
- **L2384 EN**: Comment explains nearby logic, invariants, or intent: `registers that require particular GPU or PTX version.`.
  **L2384 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`registers that require particular GPU or PTX version.`。
- **L2385 EN**: Declares TableGen multiclass `PTXReadSRegIntrinsicNB_v4i32`.
  **L2385 CN**: 声明 TableGen multiclass `PTXReadSRegIntrinsicNB_v4i32`。
- **L2386 EN**: Executes a call or declaration centered on `!eq`.
  **L2386 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L2387 EN**: Initializes variable `suffixes` from the right-hand expression.
  **L2387 CN**: 使用右侧表达式初始化变量 `suffixes`。
- **L2388 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2388 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2389 EN**: Declares TableGen def `suffixes[i]`.
  **L2389 CN**: 声明 TableGen def `suffixes[i]`。
- **L2390 EN**: Closes the current lexical scope or compound statement.
  **L2390 CN**: 结束当前词法作用域或复合语句块。
- **L2391 EN**: Blank line separating nearby declarations or logic blocks.
  **L2391 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2392 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsics to read registers with non-constant values. E.g. the values that`.
  **L2392 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsics to read registers with non-constant values. E.g. the values that`。
- **L2393 EN**: Comment explains nearby logic, invariants, or intent: `do change over the kernel lifetime. Such reads should not be CSE'd.`.
  **L2393 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`do change over the kernel lifetime. Such reads should not be CSE'd.`。
- **L2394 EN**: Declares class `PTXReadNCSRegIntrinsic<LLVMType`.
  **L2394 CN**: 声明 class `PTXReadNCSRegIntrinsic<LLVMType`。
- **L2395 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[ty], [], [IntrInaccessibleMemOnly, IntrNoCallback,`.
  **L2395 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[ty], [], [IntrInaccessibleMemOnly, IntrNoCallback,`。
- **L2396 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `IntrNoFree, IntrWillReturn, NoUndef<RetIndex>]>,`.
  **L2396 CN**: 继续一个多行参数列表、初始化器或聚合项：`IntrNoFree, IntrWillReturn, NoUndef<RetIndex>]>,`。
- **L2397 EN**: Executes a standalone statement or declaration: `NVVMBuiltin;`.
  **L2397 CN**: 执行一条独立语句或声明：`NVVMBuiltin;`。
- **L2398 EN**: Blank line separating nearby declarations or logic blocks.
  **L2398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2399 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar MAX_BLOCK_ID_RANGE = [[Range<RetIndex, 0, MAX_BLOCK_SIZE_X>],`.
  **L2399 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar MAX_BLOCK_ID_RANGE = [[Range<RetIndex, 0, MAX_BLOCK_SIZE_X>],`。
- **L2400 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 0, MAX_BLOCK_SIZE_Y>],`.
  **L2400 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 0, MAX_BLOCK_SIZE_Y>],`。

### Lines 2401-2432

````tablegen
                             [Range<RetIndex, 0, MAX_BLOCK_SIZE_Z>],
                             [Range<RetIndex, 0, 1>]];

defvar MAX_BLOCK_NID_RANGE = [[Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_X, 1)>],
                              [Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_Y, 1)>],
                              [Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_Z, 1)>],
                              [Range<RetIndex, 0, 1>]];

defvar MAX_GRID_ID_RANGE = [[Range<RetIndex, 0, MAX_GRID_SIZE_X>],
                            [Range<RetIndex, 0, MAX_GRID_SIZE_Y>],
                            [Range<RetIndex, 0, MAX_GRID_SIZE_Z>],
                            [Range<RetIndex, 0, 1>]];

defvar MAX_GRID_NID_RANGE = [[Range<RetIndex, 1, !add(MAX_GRID_SIZE_X, 1)>],
                             [Range<RetIndex, 1, !add(MAX_GRID_SIZE_Y, 1)>],
                             [Range<RetIndex, 1, !add(MAX_GRID_SIZE_Z, 1)>],
                             [Range<RetIndex, 0, 1>]];

defm int_nvvm_read_ptx_sreg_tid : PTXReadSRegIntrinsic_v4i32<MAX_BLOCK_ID_RANGE>;
defm int_nvvm_read_ptx_sreg_ntid : PTXReadSRegIntrinsic_v4i32<MAX_BLOCK_NID_RANGE>;

def int_nvvm_read_ptx_sreg_laneid
  : PTXReadSRegIntrinsic_r32<[Range<RetIndex, 0, WARP_SIZE>]>;

def int_nvvm_read_ptx_sreg_warpid : PTXReadSRegIntrinsic_r32;
def int_nvvm_read_ptx_sreg_nwarpid : PTXReadSRegIntrinsic_r32;

defm int_nvvm_read_ptx_sreg_ctaid : PTXReadSRegIntrinsic_v4i32<MAX_GRID_ID_RANGE>;
defm int_nvvm_read_ptx_sreg_nctaid : PTXReadSRegIntrinsic_v4i32<MAX_GRID_NID_RANGE>;

def int_nvvm_read_ptx_sreg_smid : PTXReadSRegIntrinsic_r32;
def int_nvvm_read_ptx_sreg_nsmid : PTXReadSRegIntrinsic_r32;
````
- **L2401 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 0, MAX_BLOCK_SIZE_Z>],`.
  **L2401 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 0, MAX_BLOCK_SIZE_Z>],`。
- **L2402 EN**: Executes a standalone statement or declaration: `[Range<RetIndex, 0, 1>]];`.
  **L2402 CN**: 执行一条独立语句或声明：`[Range<RetIndex, 0, 1>]];`。
- **L2403 EN**: Blank line separating nearby declarations or logic blocks.
  **L2403 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2404 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar MAX_BLOCK_NID_RANGE = [[Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_X, 1)>],`.
  **L2404 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar MAX_BLOCK_NID_RANGE = [[Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_X, 1)>],`。
- **L2405 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_Y, 1)>],`.
  **L2405 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_Y, 1)>],`。
- **L2406 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_Z, 1)>],`.
  **L2406 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 1, !add(MAX_BLOCK_SIZE_Z, 1)>],`。
- **L2407 EN**: Executes a standalone statement or declaration: `[Range<RetIndex, 0, 1>]];`.
  **L2407 CN**: 执行一条独立语句或声明：`[Range<RetIndex, 0, 1>]];`。
- **L2408 EN**: Blank line separating nearby declarations or logic blocks.
  **L2408 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar MAX_GRID_ID_RANGE = [[Range<RetIndex, 0, MAX_GRID_SIZE_X>],`.
  **L2409 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar MAX_GRID_ID_RANGE = [[Range<RetIndex, 0, MAX_GRID_SIZE_X>],`。
- **L2410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 0, MAX_GRID_SIZE_Y>],`.
  **L2410 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 0, MAX_GRID_SIZE_Y>],`。
- **L2411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 0, MAX_GRID_SIZE_Z>],`.
  **L2411 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 0, MAX_GRID_SIZE_Z>],`。
- **L2412 EN**: Executes a standalone statement or declaration: `[Range<RetIndex, 0, 1>]];`.
  **L2412 CN**: 执行一条独立语句或声明：`[Range<RetIndex, 0, 1>]];`。
- **L2413 EN**: Blank line separating nearby declarations or logic blocks.
  **L2413 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2414 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar MAX_GRID_NID_RANGE = [[Range<RetIndex, 1, !add(MAX_GRID_SIZE_X, 1)>],`.
  **L2414 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar MAX_GRID_NID_RANGE = [[Range<RetIndex, 1, !add(MAX_GRID_SIZE_X, 1)>],`。
- **L2415 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 1, !add(MAX_GRID_SIZE_Y, 1)>],`.
  **L2415 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 1, !add(MAX_GRID_SIZE_Y, 1)>],`。
- **L2416 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<RetIndex, 1, !add(MAX_GRID_SIZE_Z, 1)>],`.
  **L2416 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<RetIndex, 1, !add(MAX_GRID_SIZE_Z, 1)>],`。
- **L2417 EN**: Executes a standalone statement or declaration: `[Range<RetIndex, 0, 1>]];`.
  **L2417 CN**: 执行一条独立语句或声明：`[Range<RetIndex, 0, 1>]];`。
- **L2418 EN**: Blank line separating nearby declarations or logic blocks.
  **L2418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2419 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_tid`.
  **L2419 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_tid`。
- **L2420 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_ntid`.
  **L2420 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_ntid`。
- **L2421 EN**: Blank line separating nearby declarations or logic blocks.
  **L2421 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2422 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_laneid`.
  **L2422 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_laneid`。
- **L2423 EN**: Executes a standalone statement or declaration: `: PTXReadSRegIntrinsic_r32<[Range<RetIndex, 0, WARP_SIZE>]>;`.
  **L2423 CN**: 执行一条独立语句或声明：`: PTXReadSRegIntrinsic_r32<[Range<RetIndex, 0, WARP_SIZE>]>;`。
- **L2424 EN**: Blank line separating nearby declarations or logic blocks.
  **L2424 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2425 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_warpid`.
  **L2425 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_warpid`。
- **L2426 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_nwarpid`.
  **L2426 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_nwarpid`。
- **L2427 EN**: Blank line separating nearby declarations or logic blocks.
  **L2427 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2428 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_ctaid`.
  **L2428 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_ctaid`。
- **L2429 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_nctaid`.
  **L2429 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_nctaid`。
- **L2430 EN**: Blank line separating nearby declarations or logic blocks.
  **L2430 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2431 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_smid`.
  **L2431 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_smid`。
- **L2432 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_nsmid`.
  **L2432 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_nsmid`。

### Lines 2433-2464

````tablegen
def int_nvvm_read_ptx_sreg_gridid : PTXReadSRegIntrinsic_r32;

def int_nvvm_read_ptx_sreg_lanemask_eq : PTXReadSRegIntrinsic_r32;
def int_nvvm_read_ptx_sreg_lanemask_le : PTXReadSRegIntrinsic_r32;
def int_nvvm_read_ptx_sreg_lanemask_lt : PTXReadSRegIntrinsic_r32;
def int_nvvm_read_ptx_sreg_lanemask_ge : PTXReadSRegIntrinsic_r32;
def int_nvvm_read_ptx_sreg_lanemask_gt : PTXReadSRegIntrinsic_r32;

def int_nvvm_read_ptx_sreg_clock : PTXReadNCSRegIntrinsic<llvm_i32_ty>;
def int_nvvm_read_ptx_sreg_clock64 : PTXReadNCSRegIntrinsic<llvm_i64_ty>;

def int_nvvm_read_ptx_sreg_globaltimer : PTXReadNCSRegIntrinsic<llvm_i64_ty>;
def int_nvvm_read_ptx_sreg_globaltimer_lo : PTXReadNCSRegIntrinsic<llvm_i32_ty>;

foreach i = 0...4 in
  def int_nvvm_read_ptx_sreg_pm # i : PTXReadNCSRegIntrinsic<llvm_i32_ty>;

def int_nvvm_read_ptx_sreg_warpsize
  : PTXReadSRegIntrinsic_r32<[Range<RetIndex, WARP_SIZE, !add(WARP_SIZE, 1)>]>;

foreach i = 0...31 in
  def int_nvvm_read_ptx_sreg_envreg # i : PTXReadSRegIntrinsic_r32;

// sm90+, PTX7.8+

// Note: Since clusters are subdivisions of the grid, we conservatively use the
// maximum grid size as an upper bound for the clusterid and cluster_ctaid. In
// practice, the clusterid will likely be much smaller. The CUDA programming
// guide recommends 8 as a maximum portable value and H100s support 16.

defm int_nvvm_read_ptx_sreg_clusterid : PTXReadSRegIntrinsicNB_v4i32<MAX_GRID_ID_RANGE>;
defm int_nvvm_read_ptx_sreg_nclusterid : PTXReadSRegIntrinsicNB_v4i32<MAX_GRID_NID_RANGE>;
````
- **L2433 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_gridid`.
  **L2433 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_gridid`。
- **L2434 EN**: Blank line separating nearby declarations or logic blocks.
  **L2434 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2435 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_lanemask_eq`.
  **L2435 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_lanemask_eq`。
- **L2436 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_lanemask_le`.
  **L2436 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_lanemask_le`。
- **L2437 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_lanemask_lt`.
  **L2437 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_lanemask_lt`。
- **L2438 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_lanemask_ge`.
  **L2438 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_lanemask_ge`。
- **L2439 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_lanemask_gt`.
  **L2439 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_lanemask_gt`。
- **L2440 EN**: Blank line separating nearby declarations or logic blocks.
  **L2440 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2441 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_clock`.
  **L2441 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_clock`。
- **L2442 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_clock64`.
  **L2442 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_clock64`。
- **L2443 EN**: Blank line separating nearby declarations or logic blocks.
  **L2443 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2444 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_globaltimer`.
  **L2444 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_globaltimer`。
- **L2445 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_globaltimer_lo`.
  **L2445 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_globaltimer_lo`。
- **L2446 EN**: Blank line separating nearby declarations or logic blocks.
  **L2446 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2447 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2447 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2448 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_pm`.
  **L2448 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_pm`。
- **L2449 EN**: Blank line separating nearby declarations or logic blocks.
  **L2449 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2450 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_warpsize`.
  **L2450 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_warpsize`。
- **L2451 EN**: Executes a call or declaration centered on `!add`.
  **L2451 CN**: 执行以 `!add` 为核心的调用或声明。
- **L2452 EN**: Blank line separating nearby declarations or logic blocks.
  **L2452 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2453 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2453 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2454 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_envreg`.
  **L2454 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_envreg`。
- **L2455 EN**: Blank line separating nearby declarations or logic blocks.
  **L2455 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2456 EN**: Comment explains nearby logic, invariants, or intent: `sm90+, PTX7.8+`.
  **L2456 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`sm90+, PTX7.8+`。
- **L2457 EN**: Blank line separating nearby declarations or logic blocks.
  **L2457 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2458 EN**: Comment explains nearby logic, invariants, or intent: `Note: Since clusters are subdivisions of the grid, we conservatively use the`.
  **L2458 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Note: Since clusters are subdivisions of the grid, we conservatively use the`。
- **L2459 EN**: Comment explains nearby logic, invariants, or intent: `maximum grid size as an upper bound for the clusterid and cluster_ctaid. In`.
  **L2459 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`maximum grid size as an upper bound for the clusterid and cluster_ctaid. In`。
- **L2460 EN**: Comment explains nearby logic, invariants, or intent: `practice, the clusterid will likely be much smaller. The CUDA programming`.
  **L2460 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`practice, the clusterid will likely be much smaller. The CUDA programming`。
- **L2461 EN**: Comment explains nearby logic, invariants, or intent: `guide recommends 8 as a maximum portable value and H100s support 16.`.
  **L2461 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`guide recommends 8 as a maximum portable value and H100s support 16.`。
- **L2462 EN**: Blank line separating nearby declarations or logic blocks.
  **L2462 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2463 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_clusterid`.
  **L2463 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_clusterid`。
- **L2464 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_nclusterid`.
  **L2464 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_nclusterid`。

### Lines 2465-2496

````tablegen
defm int_nvvm_read_ptx_sreg_cluster_ctaid : PTXReadSRegIntrinsicNB_v4i32<MAX_GRID_ID_RANGE>;
defm int_nvvm_read_ptx_sreg_cluster_nctaid : PTXReadSRegIntrinsicNB_v4i32<MAX_GRID_NID_RANGE>;

def int_nvvm_read_ptx_sreg_cluster_ctarank : PTXReadSRegIntrinsicNB_r32;
def int_nvvm_read_ptx_sreg_cluster_nctarank : PTXReadSRegIntrinsicNB_r32;

//
// Reserved Shared Memory Intrinsics
//
def int_nvvm_read_ptx_sreg_reserved_smem_offset_begin
  : PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_begin">;
def int_nvvm_read_ptx_sreg_reserved_smem_offset_end
  : PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_end">;
def int_nvvm_read_ptx_sreg_reserved_smem_offset_cap
  : PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_cap">;
def int_nvvm_read_ptx_sreg_reserved_smem_offset_0
  : PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_0">;
def int_nvvm_read_ptx_sreg_reserved_smem_offset_1
  : PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_1">;

def int_nvvm_read_ptx_sreg_total_smem_size :
    PTXReadSRegIntrinsicNB_r32<name = "total_smem_size">;
def int_nvvm_read_ptx_sreg_aggr_smem_size :
    PTXReadSRegIntrinsicNB_r32<name = "aggr_smem_size">;
def int_nvvm_read_ptx_sreg_dynamic_smem_size :
    PTXReadSRegIntrinsicNB_r32<name = "dynamic_smem_size">;

//
// SHUFFLE
//
// Non-sync SHFL variants are pure functions of their SSA operands and the active
// thread mask. Memory properties (IntrReadMem + IntrInaccessibleMemOnly + IntrWillReturn) 
````
- **L2465 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_cluster_ctaid`.
  **L2465 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_cluster_ctaid`。
- **L2466 EN**: Declares TableGen defm `int_nvvm_read_ptx_sreg_cluster_nctaid`.
  **L2466 CN**: 声明 TableGen defm `int_nvvm_read_ptx_sreg_cluster_nctaid`。
- **L2467 EN**: Blank line separating nearby declarations or logic blocks.
  **L2467 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2468 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_cluster_ctarank`.
  **L2468 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_cluster_ctarank`。
- **L2469 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_cluster_nctarank`.
  **L2469 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_cluster_nctarank`。
- **L2470 EN**: Blank line separating nearby declarations or logic blocks.
  **L2470 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2471 EN**: Separator comment used for visual grouping.
  **L2471 CN**: 用于视觉分组的分隔注释。
- **L2472 EN**: Comment explains nearby logic, invariants, or intent: `Reserved Shared Memory Intrinsics`.
  **L2472 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Reserved Shared Memory Intrinsics`。
- **L2473 EN**: Separator comment used for visual grouping.
  **L2473 CN**: 用于视觉分组的分隔注释。
- **L2474 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_begin`.
  **L2474 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_begin`。
- **L2475 EN**: Executes a standalone statement or declaration: `: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_begin">;`.
  **L2475 CN**: 执行一条独立语句或声明：`: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_begin">;`。
- **L2476 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_end`.
  **L2476 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_end`。
- **L2477 EN**: Executes a standalone statement or declaration: `: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_end">;`.
  **L2477 CN**: 执行一条独立语句或声明：`: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_end">;`。
- **L2478 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_cap`.
  **L2478 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_cap`。
- **L2479 EN**: Executes a standalone statement or declaration: `: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_cap">;`.
  **L2479 CN**: 执行一条独立语句或声明：`: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_cap">;`。
- **L2480 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_0`.
  **L2480 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_0`。
- **L2481 EN**: Executes a standalone statement or declaration: `: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_0">;`.
  **L2481 CN**: 执行一条独立语句或声明：`: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_0">;`。
- **L2482 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_1`.
  **L2482 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_reserved_smem_offset_1`。
- **L2483 EN**: Executes a standalone statement or declaration: `: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_1">;`.
  **L2483 CN**: 执行一条独立语句或声明：`: PTXReadSRegIntrinsicNB_r32<name = "reserved_smem_offset_1">;`。
- **L2484 EN**: Blank line separating nearby declarations or logic blocks.
  **L2484 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2485 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_total_smem_size`.
  **L2485 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_total_smem_size`。
- **L2486 EN**: Executes a standalone statement or declaration: `PTXReadSRegIntrinsicNB_r32<name = "total_smem_size">;`.
  **L2486 CN**: 执行一条独立语句或声明：`PTXReadSRegIntrinsicNB_r32<name = "total_smem_size">;`。
- **L2487 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_aggr_smem_size`.
  **L2487 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_aggr_smem_size`。
- **L2488 EN**: Executes a standalone statement or declaration: `PTXReadSRegIntrinsicNB_r32<name = "aggr_smem_size">;`.
  **L2488 CN**: 执行一条独立语句或声明：`PTXReadSRegIntrinsicNB_r32<name = "aggr_smem_size">;`。
- **L2489 EN**: Declares TableGen def `int_nvvm_read_ptx_sreg_dynamic_smem_size`.
  **L2489 CN**: 声明 TableGen def `int_nvvm_read_ptx_sreg_dynamic_smem_size`。
- **L2490 EN**: Executes a standalone statement or declaration: `PTXReadSRegIntrinsicNB_r32<name = "dynamic_smem_size">;`.
  **L2490 CN**: 执行一条独立语句或声明：`PTXReadSRegIntrinsicNB_r32<name = "dynamic_smem_size">;`。
- **L2491 EN**: Blank line separating nearby declarations or logic blocks.
  **L2491 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2492 EN**: Separator comment used for visual grouping.
  **L2492 CN**: 用于视觉分组的分隔注释。
- **L2493 EN**: Comment explains nearby logic, invariants, or intent: `SHUFFLE`.
  **L2493 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`SHUFFLE`。
- **L2494 EN**: Separator comment used for visual grouping.
  **L2494 CN**: 用于视觉分组的分隔注释。
- **L2495 EN**: Comment explains nearby logic, invariants, or intent: `Non-sync SHFL variants are pure functions of their SSA operands and the active`.
  **L2495 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Non-sync SHFL variants are pure functions of their SSA operands and the active`。
- **L2496 EN**: Comment explains nearby logic, invariants, or intent: `thread mask. Memory properties (IntrReadMem + IntrInaccessibleMemOnly + IntrWillReturn)`.
  **L2496 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`thread mask. Memory properties (IntrReadMem + IntrInaccessibleMemOnly + IntrWillReturn)`。

### Lines 2497-2528

````tablegen
// model reading the implicit mask state to ensure correct ordering with other convergent
// operations, preventing unsafe reordering while allowing DCE of truly dead code.
//
// Sync shfl variants have synchronization side effects modeled as
// inaccessible memory accesses to prevent DCE and reordering.
foreach is_sync = [false, true] in {
  defvar CommonIntrinsicProps = [IntrInaccessibleMemOnly, IntrConvergent, IntrNoCallback];
  defvar IntrinsicProps = !if(is_sync, CommonIntrinsicProps,
                              !listconcat(CommonIntrinsicProps, [IntrReadMem, IntrWillReturn]));
  let IntrProperties = IntrinsicProps in {
    foreach mode = ["up", "down", "bfly", "idx"] in {
      foreach type = ["i32", "f32"] in {
        foreach return_pred = [false, true] in {
          defvar i = SHFL_INFO<is_sync, mode, type, return_pred>;
          if i.withGccBuiltin then
            def i.Name : NVVMBuiltin, Intrinsic<i.RetTy, i.ArgsTy>;
          else
            def i.Name : Intrinsic<i.RetTy, i.ArgsTy>;
        }
      }
    }
  }
}

//
// VOTE
//
let IntrProperties = [IntrInaccessibleMemOnly, IntrConvergent, IntrNoCallback] in {
  def int_nvvm_vote_all : NVVMBuiltin, Intrinsic<[llvm_i1_ty], [llvm_i1_ty]>;
  def int_nvvm_vote_any : NVVMBuiltin, Intrinsic<[llvm_i1_ty], [llvm_i1_ty]>;
  def int_nvvm_vote_uni : NVVMBuiltin, Intrinsic<[llvm_i1_ty], [llvm_i1_ty]>;
  def int_nvvm_vote_ballot : NVVMBuiltin, Intrinsic<[llvm_i32_ty], [llvm_i1_ty]>;
````
- **L2497 EN**: Comment explains nearby logic, invariants, or intent: `model reading the implicit mask state to ensure correct ordering with other convergent`.
  **L2497 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`model reading the implicit mask state to ensure correct ordering with other convergent`。
- **L2498 EN**: Comment explains nearby logic, invariants, or intent: `operations, preventing unsafe reordering while allowing DCE of truly dead code.`.
  **L2498 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`operations, preventing unsafe reordering while allowing DCE of truly dead code.`。
- **L2499 EN**: Separator comment used for visual grouping.
  **L2499 CN**: 用于视觉分组的分隔注释。
- **L2500 EN**: Comment explains nearby logic, invariants, or intent: `Sync shfl variants have synchronization side effects modeled as`.
  **L2500 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Sync shfl variants have synchronization side effects modeled as`。
- **L2501 EN**: Comment explains nearby logic, invariants, or intent: `inaccessible memory accesses to prevent DCE and reordering.`.
  **L2501 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`inaccessible memory accesses to prevent DCE and reordering.`。
- **L2502 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2502 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2503 EN**: Initializes variable `CommonIntrinsicProps` from the right-hand expression.
  **L2503 CN**: 使用右侧表达式初始化变量 `CommonIntrinsicProps`。
- **L2504 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar IntrinsicProps = !if(is_sync, CommonIntrinsicProps,`.
  **L2504 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar IntrinsicProps = !if(is_sync, CommonIntrinsicProps,`。
- **L2505 EN**: Executes a call or declaration centered on `!listconcat`.
  **L2505 CN**: 执行以 `!listconcat` 为核心的调用或声明。
- **L2506 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2506 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2507 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2507 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2508 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2508 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2509 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2509 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2510 EN**: Initializes variable `i` from the right-hand expression.
  **L2510 CN**: 使用右侧表达式初始化变量 `i`。
- **L2511 EN**: Continues the surrounding expression or declaration: `if i.withGccBuiltin then`.
  **L2511 CN**: 继续构造周围的表达式或声明：`if i.withGccBuiltin then`。
- **L2512 EN**: Declares TableGen def `i.Name`.
  **L2512 CN**: 声明 TableGen def `i.Name`。
- **L2513 EN**: Starts the alternative branch of the preceding conditional.
  **L2513 CN**: 开始前一个条件语句的备选分支。
- **L2514 EN**: Declares TableGen def `i.Name`.
  **L2514 CN**: 声明 TableGen def `i.Name`。
- **L2515 EN**: Closes the current lexical scope or compound statement.
  **L2515 CN**: 结束当前词法作用域或复合语句块。
- **L2516 EN**: Closes the current lexical scope or compound statement.
  **L2516 CN**: 结束当前词法作用域或复合语句块。
- **L2517 EN**: Closes the current lexical scope or compound statement.
  **L2517 CN**: 结束当前词法作用域或复合语句块。
- **L2518 EN**: Closes the current lexical scope or compound statement.
  **L2518 CN**: 结束当前词法作用域或复合语句块。
- **L2519 EN**: Closes the current lexical scope or compound statement.
  **L2519 CN**: 结束当前词法作用域或复合语句块。
- **L2520 EN**: Blank line separating nearby declarations or logic blocks.
  **L2520 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2521 EN**: Separator comment used for visual grouping.
  **L2521 CN**: 用于视觉分组的分隔注释。
- **L2522 EN**: Comment explains nearby logic, invariants, or intent: `VOTE`.
  **L2522 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VOTE`。
- **L2523 EN**: Separator comment used for visual grouping.
  **L2523 CN**: 用于视觉分组的分隔注释。
- **L2524 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2524 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2525 EN**: Declares TableGen def `int_nvvm_vote_all`.
  **L2525 CN**: 声明 TableGen def `int_nvvm_vote_all`。
- **L2526 EN**: Declares TableGen def `int_nvvm_vote_any`.
  **L2526 CN**: 声明 TableGen def `int_nvvm_vote_any`。
- **L2527 EN**: Declares TableGen def `int_nvvm_vote_uni`.
  **L2527 CN**: 声明 TableGen def `int_nvvm_vote_uni`。
- **L2528 EN**: Declares TableGen def `int_nvvm_vote_ballot`.
  **L2528 CN**: 声明 TableGen def `int_nvvm_vote_ballot`。

### Lines 2529-2560

````tablegen
}

//
// VOTE.SYNC
//
let IntrProperties = [IntrInaccessibleMemOnly, IntrConvergent, IntrNoCallback] in {
  def int_nvvm_vote_all_sync : NVVMBuiltin, Intrinsic<[llvm_i1_ty], [llvm_i32_ty, llvm_i1_ty]>;
  def int_nvvm_vote_any_sync : NVVMBuiltin, Intrinsic<[llvm_i1_ty], [llvm_i32_ty, llvm_i1_ty]>;
  def int_nvvm_vote_uni_sync : NVVMBuiltin, Intrinsic<[llvm_i1_ty], [llvm_i32_ty, llvm_i1_ty]>;
  def int_nvvm_vote_ballot_sync : NVVMBuiltin, Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i1_ty]>;
}

//
// ACTIVEMASK
//
def int_nvvm_activemask : NVVMBuiltin,
  Intrinsic<[llvm_i32_ty], [],
            [IntrInaccessibleMemOnly, IntrConvergent, IntrNoCallback, IntrHasSideEffects]>;

//
// MATCH.SYNC
//
let IntrProperties = [IntrInaccessibleMemOnly, IntrConvergent, IntrNoCallback] in {
  // match.any.sync.b32 mask, value
  def int_nvvm_match_any_sync_i32 : NVVMBuiltin,
    Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;
  // match.any.sync.b64 mask, value
  def int_nvvm_match_any_sync_i64 : NVVMBuiltin,
    Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i64_ty]>;

  // match.all instruction have two variants -- one returns a single value, another
  // returns a pair {value, predicate}. We currently only implement the latter as
````
- **L2529 EN**: Closes the current lexical scope or compound statement.
  **L2529 CN**: 结束当前词法作用域或复合语句块。
- **L2530 EN**: Blank line separating nearby declarations or logic blocks.
  **L2530 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2531 EN**: Separator comment used for visual grouping.
  **L2531 CN**: 用于视觉分组的分隔注释。
- **L2532 EN**: Comment explains nearby logic, invariants, or intent: `VOTE.SYNC`.
  **L2532 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`VOTE.SYNC`。
- **L2533 EN**: Separator comment used for visual grouping.
  **L2533 CN**: 用于视觉分组的分隔注释。
- **L2534 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2534 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2535 EN**: Declares TableGen def `int_nvvm_vote_all_sync`.
  **L2535 CN**: 声明 TableGen def `int_nvvm_vote_all_sync`。
- **L2536 EN**: Declares TableGen def `int_nvvm_vote_any_sync`.
  **L2536 CN**: 声明 TableGen def `int_nvvm_vote_any_sync`。
- **L2537 EN**: Declares TableGen def `int_nvvm_vote_uni_sync`.
  **L2537 CN**: 声明 TableGen def `int_nvvm_vote_uni_sync`。
- **L2538 EN**: Declares TableGen def `int_nvvm_vote_ballot_sync`.
  **L2538 CN**: 声明 TableGen def `int_nvvm_vote_ballot_sync`。
- **L2539 EN**: Closes the current lexical scope or compound statement.
  **L2539 CN**: 结束当前词法作用域或复合语句块。
- **L2540 EN**: Blank line separating nearby declarations or logic blocks.
  **L2540 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2541 EN**: Separator comment used for visual grouping.
  **L2541 CN**: 用于视觉分组的分隔注释。
- **L2542 EN**: Comment explains nearby logic, invariants, or intent: `ACTIVEMASK`.
  **L2542 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ACTIVEMASK`。
- **L2543 EN**: Separator comment used for visual grouping.
  **L2543 CN**: 用于视觉分组的分隔注释。
- **L2544 EN**: Declares TableGen def `int_nvvm_activemask`.
  **L2544 CN**: 声明 TableGen def `int_nvvm_activemask`。
- **L2545 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[llvm_i32_ty], [],`.
  **L2545 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[llvm_i32_ty], [],`。
- **L2546 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, IntrConvergent, IntrNoCallback, IntrHasSideEffects]>;`.
  **L2546 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, IntrConvergent, IntrNoCallback, IntrHasSideEffects]>;`。
- **L2547 EN**: Blank line separating nearby declarations or logic blocks.
  **L2547 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2548 EN**: Separator comment used for visual grouping.
  **L2548 CN**: 用于视觉分组的分隔注释。
- **L2549 EN**: Comment explains nearby logic, invariants, or intent: `MATCH.SYNC`.
  **L2549 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MATCH.SYNC`。
- **L2550 EN**: Separator comment used for visual grouping.
  **L2550 CN**: 用于视觉分组的分隔注释。
- **L2551 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2551 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2552 EN**: Comment explains nearby logic, invariants, or intent: `match.any.sync.b32 mask, value`.
  **L2552 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`match.any.sync.b32 mask, value`。
- **L2553 EN**: Declares TableGen def `int_nvvm_match_any_sync_i32`.
  **L2553 CN**: 声明 TableGen def `int_nvvm_match_any_sync_i32`。
- **L2554 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L2554 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L2555 EN**: Comment explains nearby logic, invariants, or intent: `match.any.sync.b64 mask, value`.
  **L2555 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`match.any.sync.b64 mask, value`。
- **L2556 EN**: Declares TableGen def `int_nvvm_match_any_sync_i64`.
  **L2556 CN**: 声明 TableGen def `int_nvvm_match_any_sync_i64`。
- **L2557 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i64_ty]>;`.
  **L2557 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i64_ty]>;`。
- **L2558 EN**: Blank line separating nearby declarations or logic blocks.
  **L2558 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2559 EN**: Comment explains nearby logic, invariants, or intent: `match.all instruction have two variants -- one returns a single value, another`.
  **L2559 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`match.all instruction have two variants -- one returns a single value, another`。
- **L2560 EN**: Comment explains nearby logic, invariants, or intent: `returns a pair {value, predicate}. We currently only implement the latter as`.
  **L2560 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`returns a pair {value, predicate}. We currently only implement the latter as`。

### Lines 2561-2592

````tablegen
  // that's the variant exposed by CUDA API.

  // match.all.sync.b32p mask, value
  def int_nvvm_match_all_sync_i32p :
    Intrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty, llvm_i32_ty]>;
  // match.all.sync.b64p mask, value
  def int_nvvm_match_all_sync_i64p :
    Intrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty, llvm_i64_ty]>;
}
//
// ELECT.SYNC
//
// elect.sync dst|pred, membermask
def int_nvvm_elect_sync :
  DefaultAttrsIntrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty],
                        [IntrInaccessibleMemOnly, IntrConvergent]>;

//
// REDUX.SYNC
//
// redux.sync.op.u32 dst, src, membermask;

let IntrProperties = [IntrConvergent, IntrInaccessibleMemOnly, IntrNoCallback] in {
  foreach op = ["umin", "umax", "add", "min", "max", "and", "xor", "or"] in
    def int_nvvm_redux_sync_ # op : NVVMBuiltin,
      Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;

  // redux.sync.op.{abs}.{NaN}.f32 dst, src, membermask;
  foreach binOp = ["min", "max"] in
    foreach abs = ["", "_abs"] in
      foreach NaN = ["", "_NaN"] in
        def int_nvvm_redux_sync_f # binOp # abs # NaN : NVVMBuiltin,
````
- **L2561 EN**: Comment explains nearby logic, invariants, or intent: `that's the variant exposed by CUDA API.`.
  **L2561 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`that's the variant exposed by CUDA API.`。
- **L2562 EN**: Blank line separating nearby declarations or logic blocks.
  **L2562 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2563 EN**: Comment explains nearby logic, invariants, or intent: `match.all.sync.b32p mask, value`.
  **L2563 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`match.all.sync.b32p mask, value`。
- **L2564 EN**: Declares TableGen def `int_nvvm_match_all_sync_i32p`.
  **L2564 CN**: 声明 TableGen def `int_nvvm_match_all_sync_i32p`。
- **L2565 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L2565 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L2566 EN**: Comment explains nearby logic, invariants, or intent: `match.all.sync.b64p mask, value`.
  **L2566 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`match.all.sync.b64p mask, value`。
- **L2567 EN**: Declares TableGen def `int_nvvm_match_all_sync_i64p`.
  **L2567 CN**: 声明 TableGen def `int_nvvm_match_all_sync_i64p`。
- **L2568 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty, llvm_i64_ty]>;`.
  **L2568 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty, llvm_i64_ty]>;`。
- **L2569 EN**: Closes the current lexical scope or compound statement.
  **L2569 CN**: 结束当前词法作用域或复合语句块。
- **L2570 EN**: Separator comment used for visual grouping.
  **L2570 CN**: 用于视觉分组的分隔注释。
- **L2571 EN**: Comment explains nearby logic, invariants, or intent: `ELECT.SYNC`.
  **L2571 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`ELECT.SYNC`。
- **L2572 EN**: Separator comment used for visual grouping.
  **L2572 CN**: 用于视觉分组的分隔注释。
- **L2573 EN**: Comment explains nearby logic, invariants, or intent: `elect.sync dst|pred, membermask`.
  **L2573 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`elect.sync dst|pred, membermask`。
- **L2574 EN**: Declares TableGen def `int_nvvm_elect_sync`.
  **L2574 CN**: 声明 TableGen def `int_nvvm_elect_sync`。
- **L2575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty],`.
  **L2575 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[llvm_i32_ty, llvm_i1_ty], [llvm_i32_ty],`。
- **L2576 EN**: Executes a standalone statement or declaration: `[IntrInaccessibleMemOnly, IntrConvergent]>;`.
  **L2576 CN**: 执行一条独立语句或声明：`[IntrInaccessibleMemOnly, IntrConvergent]>;`。
- **L2577 EN**: Blank line separating nearby declarations or logic blocks.
  **L2577 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2578 EN**: Separator comment used for visual grouping.
  **L2578 CN**: 用于视觉分组的分隔注释。
- **L2579 EN**: Comment explains nearby logic, invariants, or intent: `REDUX.SYNC`.
  **L2579 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`REDUX.SYNC`。
- **L2580 EN**: Separator comment used for visual grouping.
  **L2580 CN**: 用于视觉分组的分隔注释。
- **L2581 EN**: Comment explains nearby logic, invariants, or intent: `redux.sync.op.u32 dst, src, membermask;`.
  **L2581 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`redux.sync.op.u32 dst, src, membermask;`。
- **L2582 EN**: Blank line separating nearby declarations or logic blocks.
  **L2582 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2583 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2583 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2584 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2584 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2585 EN**: Declares TableGen def `int_nvvm_redux_sync_`.
  **L2585 CN**: 声明 TableGen def `int_nvvm_redux_sync_`。
- **L2586 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`.
  **L2586 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_i32_ty], [llvm_i32_ty, llvm_i32_ty]>;`。
- **L2587 EN**: Blank line separating nearby declarations or logic blocks.
  **L2587 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2588 EN**: Comment explains nearby logic, invariants, or intent: `redux.sync.op.{abs}.{NaN}.f32 dst, src, membermask;`.
  **L2588 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`redux.sync.op.{abs}.{NaN}.f32 dst, src, membermask;`。
- **L2589 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2589 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2590 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2590 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2591 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2591 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2592 EN**: Declares TableGen def `int_nvvm_redux_sync_f`.
  **L2592 CN**: 声明 TableGen def `int_nvvm_redux_sync_f`。

### Lines 2593-2624

````tablegen
          Intrinsic<[llvm_float_ty], [llvm_float_ty, llvm_i32_ty]>;
}

//
// WGMMA fence instructions
//
// wgmma.fence.sync.aligned;
def int_nvvm_wgmma_fence_sync_aligned : Intrinsic<[], [], [IntrConvergent]>;

// wgmma.commit_group.sync.aligned;
def int_nvvm_wgmma_commit_group_sync_aligned
  : Intrinsic<[], [], [IntrConvergent], "llvm.nvvm.wgmma.commit_group.sync.aligned">;

// wgmma.wait_group.sync.aligned N;
def int_nvvm_wgmma_wait_group_sync_aligned
  : Intrinsic<[], [llvm_i64_ty], [IntrConvergent, ImmArg<ArgIndex<0>>], "llvm.nvvm.wgmma.wait_group.sync.aligned">;

//
// WMMA instructions
//
// WMMA.LOAD
class NVVM_WMMA_LD<WMMA_REGS Frag, string Layout, int WithStride>
  : Intrinsic<Frag.regs,
              !if(WithStride, [llvm_anyptr_ty, llvm_i32_ty], [llvm_anyptr_ty]),
              [IntrWillReturn, IntrReadMem, IntrArgMemOnly, IntrNoCallback, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],
              WMMA_NAME_LDST<"load", Frag, Layout, WithStride>.intr_name>;

// WMMA.STORE.D
class NVVM_WMMA_ST<WMMA_REGS Frag, string Layout, int WithStride>
  : Intrinsic<[],
              !listconcat(
                [llvm_anyptr_ty],
````
- **L2593 EN**: Executes a standalone statement or declaration: `Intrinsic<[llvm_float_ty], [llvm_float_ty, llvm_i32_ty]>;`.
  **L2593 CN**: 执行一条独立语句或声明：`Intrinsic<[llvm_float_ty], [llvm_float_ty, llvm_i32_ty]>;`。
- **L2594 EN**: Closes the current lexical scope or compound statement.
  **L2594 CN**: 结束当前词法作用域或复合语句块。
- **L2595 EN**: Blank line separating nearby declarations or logic blocks.
  **L2595 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2596 EN**: Separator comment used for visual grouping.
  **L2596 CN**: 用于视觉分组的分隔注释。
- **L2597 EN**: Comment explains nearby logic, invariants, or intent: `WGMMA fence instructions`.
  **L2597 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WGMMA fence instructions`。
- **L2598 EN**: Separator comment used for visual grouping.
  **L2598 CN**: 用于视觉分组的分隔注释。
- **L2599 EN**: Comment explains nearby logic, invariants, or intent: `wgmma.fence.sync.aligned;`.
  **L2599 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wgmma.fence.sync.aligned;`。
- **L2600 EN**: Declares TableGen def `int_nvvm_wgmma_fence_sync_aligned`.
  **L2600 CN**: 声明 TableGen def `int_nvvm_wgmma_fence_sync_aligned`。
- **L2601 EN**: Blank line separating nearby declarations or logic blocks.
  **L2601 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2602 EN**: Comment explains nearby logic, invariants, or intent: `wgmma.commit_group.sync.aligned;`.
  **L2602 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wgmma.commit_group.sync.aligned;`。
- **L2603 EN**: Declares TableGen def `int_nvvm_wgmma_commit_group_sync_aligned`.
  **L2603 CN**: 声明 TableGen def `int_nvvm_wgmma_commit_group_sync_aligned`。
- **L2604 EN**: Executes a standalone statement or declaration: `: Intrinsic<[], [], [IntrConvergent], "llvm.nvvm.wgmma.commit_group.sync.aligned">;`.
  **L2604 CN**: 执行一条独立语句或声明：`: Intrinsic<[], [], [IntrConvergent], "llvm.nvvm.wgmma.commit_group.sync.aligned">;`。
- **L2605 EN**: Blank line separating nearby declarations or logic blocks.
  **L2605 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2606 EN**: Comment explains nearby logic, invariants, or intent: `wgmma.wait_group.sync.aligned N;`.
  **L2606 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`wgmma.wait_group.sync.aligned N;`。
- **L2607 EN**: Declares TableGen def `int_nvvm_wgmma_wait_group_sync_aligned`.
  **L2607 CN**: 声明 TableGen def `int_nvvm_wgmma_wait_group_sync_aligned`。
- **L2608 EN**: Executes a standalone statement or declaration: `: Intrinsic<[], [llvm_i64_ty], [IntrConvergent, ImmArg<ArgIndex<0>>], "llvm.nvvm.wgmma.wait_group.sync.aligned">;`.
  **L2608 CN**: 执行一条独立语句或声明：`: Intrinsic<[], [llvm_i64_ty], [IntrConvergent, ImmArg<ArgIndex<0>>], "llvm.nvvm.wgmma.wait_group.sync.aligned">;`。
- **L2609 EN**: Blank line separating nearby declarations or logic blocks.
  **L2609 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2610 EN**: Separator comment used for visual grouping.
  **L2610 CN**: 用于视觉分组的分隔注释。
- **L2611 EN**: Comment explains nearby logic, invariants, or intent: `WMMA instructions`.
  **L2611 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA instructions`。
- **L2612 EN**: Separator comment used for visual grouping.
  **L2612 CN**: 用于视觉分组的分隔注释。
- **L2613 EN**: Comment explains nearby logic, invariants, or intent: `WMMA.LOAD`.
  **L2613 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA.LOAD`。
- **L2614 EN**: Declares class `NVVM_WMMA_LD<WMMA_REGS`.
  **L2614 CN**: 声明 class `NVVM_WMMA_LD<WMMA_REGS`。
- **L2615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<Frag.regs,`.
  **L2615 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<Frag.regs,`。
- **L2616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(WithStride, [llvm_anyptr_ty, llvm_i32_ty], [llvm_anyptr_ty]),`.
  **L2616 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(WithStride, [llvm_anyptr_ty, llvm_i32_ty], [llvm_anyptr_ty]),`。
- **L2617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWillReturn, IntrReadMem, IntrArgMemOnly, IntrNoCallback, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`.
  **L2617 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWillReturn, IntrReadMem, IntrArgMemOnly, IntrNoCallback, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`。
- **L2618 EN**: Executes a standalone statement or declaration: `WMMA_NAME_LDST<"load", Frag, Layout, WithStride>.intr_name>;`.
  **L2618 CN**: 执行一条独立语句或声明：`WMMA_NAME_LDST<"load", Frag, Layout, WithStride>.intr_name>;`。
- **L2619 EN**: Blank line separating nearby declarations or logic blocks.
  **L2619 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2620 EN**: Comment explains nearby logic, invariants, or intent: `WMMA.STORE.D`.
  **L2620 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA.STORE.D`。
- **L2621 EN**: Declares class `NVVM_WMMA_ST<WMMA_REGS`.
  **L2621 CN**: 声明 class `NVVM_WMMA_ST<WMMA_REGS`。
- **L2622 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[],`.
  **L2622 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[],`。
- **L2623 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2623 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2624 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_anyptr_ty],`.
  **L2624 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_anyptr_ty],`。

### Lines 2625-2656

````tablegen
                Frag.regs,
                !if(WithStride, [llvm_i32_ty], [])),
              [IntrWriteMem, IntrArgMemOnly, IntrNoCallback, WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],
              WMMA_NAME_LDST<"store", Frag, Layout, WithStride>.intr_name>;

// Create all load/store variants
foreach layout = ["row", "col"] in {
  foreach stride = [0, 1] in {
    foreach frag = NVVM_MMA_OPS.all_ld_ops in
      if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then
        def WMMA_NAME_LDST<"load", frag, layout, stride>.record_name
             : NVVM_WMMA_LD<frag, layout, stride>;
    foreach frag = NVVM_MMA_OPS.all_st_ops in
      if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then
        def WMMA_NAME_LDST<"store", frag, layout, stride>.record_name
             : NVVM_WMMA_ST<frag, layout, stride>;
  }
}

// WMMA.MMA
class NVVM_MMA<WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D>
  : Intrinsic<D.regs,
              !listconcat(A.regs, B.regs, C.regs),
              [IntrNoMem, IntrNoCallback]>;

foreach layout_a = ["row", "col"] in {
  foreach layout_b = ["row", "col"] in {
    foreach satf = [0, 1] in {
      foreach rnd = ["", "rn", "rz", "rm", "rp"] in {
        foreach op = NVVM_MMA_OPS.all_wmma_ops in {
          foreach b1op = NVVM_MMA_B1OPS<op>.ret in {
            if NVVM_WMMA_SUPPORTED<op, layout_a, layout_b, satf, rnd>.ret then {
````
- **L2625 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Frag.regs,`.
  **L2625 CN**: 继续一个多行参数列表、初始化器或聚合项：`Frag.regs,`。
- **L2626 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(WithStride, [llvm_i32_ty], [])),`.
  **L2626 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(WithStride, [llvm_i32_ty], [])),`。
- **L2627 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, IntrArgMemOnly, IntrNoCallback, WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`.
  **L2627 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, IntrArgMemOnly, IntrNoCallback, WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`。
- **L2628 EN**: Executes a standalone statement or declaration: `WMMA_NAME_LDST<"store", Frag, Layout, WithStride>.intr_name>;`.
  **L2628 CN**: 执行一条独立语句或声明：`WMMA_NAME_LDST<"store", Frag, Layout, WithStride>.intr_name>;`。
- **L2629 EN**: Blank line separating nearby declarations or logic blocks.
  **L2629 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2630 EN**: Comment explains nearby logic, invariants, or intent: `Create all load/store variants`.
  **L2630 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Create all load/store variants`。
- **L2631 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2631 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2632 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2632 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2633 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2633 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2634 EN**: Continues the surrounding expression or declaration: `if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then`.
  **L2634 CN**: 继续构造周围的表达式或声明：`if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then`。
- **L2635 EN**: Declares TableGen def `WMMA_NAME_LDST`.
  **L2635 CN**: 声明 TableGen def `WMMA_NAME_LDST`。
- **L2636 EN**: Executes a standalone statement or declaration: `: NVVM_WMMA_LD<frag, layout, stride>;`.
  **L2636 CN**: 执行一条独立语句或声明：`: NVVM_WMMA_LD<frag, layout, stride>;`。
- **L2637 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2637 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2638 EN**: Continues the surrounding expression or declaration: `if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then`.
  **L2638 CN**: 继续构造周围的表达式或声明：`if NVVM_WMMA_LDST_SUPPORTED<frag, layout>.ret then`。
- **L2639 EN**: Declares TableGen def `WMMA_NAME_LDST`.
  **L2639 CN**: 声明 TableGen def `WMMA_NAME_LDST`。
- **L2640 EN**: Executes a standalone statement or declaration: `: NVVM_WMMA_ST<frag, layout, stride>;`.
  **L2640 CN**: 执行一条独立语句或声明：`: NVVM_WMMA_ST<frag, layout, stride>;`。
- **L2641 EN**: Closes the current lexical scope or compound statement.
  **L2641 CN**: 结束当前词法作用域或复合语句块。
- **L2642 EN**: Closes the current lexical scope or compound statement.
  **L2642 CN**: 结束当前词法作用域或复合语句块。
- **L2643 EN**: Blank line separating nearby declarations or logic blocks.
  **L2643 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2644 EN**: Comment explains nearby logic, invariants, or intent: `WMMA.MMA`.
  **L2644 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`WMMA.MMA`。
- **L2645 EN**: Declares class `NVVM_MMA<WMMA_REGS`.
  **L2645 CN**: 声明 class `NVVM_MMA<WMMA_REGS`。
- **L2646 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<D.regs,`.
  **L2646 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<D.regs,`。
- **L2647 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(A.regs, B.regs, C.regs),`.
  **L2647 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(A.regs, B.regs, C.regs),`。
- **L2648 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrNoCallback]>;`.
  **L2648 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrNoCallback]>;`。
- **L2649 EN**: Blank line separating nearby declarations or logic blocks.
  **L2649 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2650 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2650 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2651 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2651 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2652 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2652 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2653 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2653 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2654 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2654 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2655 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2655 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2656 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2656 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2657-2688

````tablegen
              def WMMA_NAME<layout_a, layout_b, satf, rnd, b1op,
                                op[0], op[1], op[2], op[3]>.record_name
                : NVVM_MMA<op[0], op[1], op[2], op[3]>;
            }
          } // b1op
        } // op
      } // rnd
    } // satf
  } // layout_b
} // layout_a

foreach layout_a = ["row", "col"] in {
  foreach layout_b = ["row", "col"] in {
    foreach satf = [0, 1] in {
      foreach op = NVVM_MMA_OPS.all_mma_ops in {
        foreach b1op = NVVM_MMA_B1OPS<op>.ret in {
          foreach kind = ["", "kind::f8f6f4"] in {
            if NVVM_MMA_SUPPORTED<op, layout_a, layout_b, kind, satf>.ret then {
                def MMA_NAME<layout_a, layout_b, satf, b1op, kind, op[0], op[1], op[2], op[3]>.record_name
                : NVVM_MMA<op[0], op[1], op[2], op[3]>;
            }
          } // kind
        } // b1op
      } // op
    } // satf
  } // layout_b
} // layout_a

class NVVM_MMA_BLOCK_SCALE<WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D>
  : Intrinsic<D.regs,
              !listconcat(A.regs, B.regs, C.regs,
              [
````
- **L2657 EN**: Declares TableGen def `WMMA_NAME`.
  **L2657 CN**: 声明 TableGen def `WMMA_NAME`。
- **L2658 EN**: Continues the surrounding expression or declaration: `op[0], op[1], op[2], op[3]>.record_name`.
  **L2658 CN**: 继续构造周围的表达式或声明：`op[0], op[1], op[2], op[3]>.record_name`。
- **L2659 EN**: Executes a standalone statement or declaration: `: NVVM_MMA<op[0], op[1], op[2], op[3]>;`.
  **L2659 CN**: 执行一条独立语句或声明：`: NVVM_MMA<op[0], op[1], op[2], op[3]>;`。
- **L2660 EN**: Closes the current lexical scope or compound statement.
  **L2660 CN**: 结束当前词法作用域或复合语句块。
- **L2661 EN**: Continues the surrounding expression or declaration: `} // b1op`.
  **L2661 CN**: 继续构造周围的表达式或声明：`} // b1op`。
- **L2662 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2662 CN**: 继续构造周围的表达式或声明：`} // op`。
- **L2663 EN**: Continues the surrounding expression or declaration: `} // rnd`.
  **L2663 CN**: 继续构造周围的表达式或声明：`} // rnd`。
- **L2664 EN**: Continues the surrounding expression or declaration: `} // satf`.
  **L2664 CN**: 继续构造周围的表达式或声明：`} // satf`。
- **L2665 EN**: Continues the surrounding expression or declaration: `} // layout_b`.
  **L2665 CN**: 继续构造周围的表达式或声明：`} // layout_b`。
- **L2666 EN**: Continues the surrounding expression or declaration: `} // layout_a`.
  **L2666 CN**: 继续构造周围的表达式或声明：`} // layout_a`。
- **L2667 EN**: Blank line separating nearby declarations or logic blocks.
  **L2667 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2668 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2668 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2669 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2669 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2670 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2670 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2671 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2671 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2672 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2672 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2673 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2673 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2674 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2674 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2675 EN**: Declares TableGen def `MMA_NAME`.
  **L2675 CN**: 声明 TableGen def `MMA_NAME`。
- **L2676 EN**: Executes a standalone statement or declaration: `: NVVM_MMA<op[0], op[1], op[2], op[3]>;`.
  **L2676 CN**: 执行一条独立语句或声明：`: NVVM_MMA<op[0], op[1], op[2], op[3]>;`。
- **L2677 EN**: Closes the current lexical scope or compound statement.
  **L2677 CN**: 结束当前词法作用域或复合语句块。
- **L2678 EN**: Continues the surrounding expression or declaration: `} // kind`.
  **L2678 CN**: 继续构造周围的表达式或声明：`} // kind`。
- **L2679 EN**: Continues the surrounding expression or declaration: `} // b1op`.
  **L2679 CN**: 继续构造周围的表达式或声明：`} // b1op`。
- **L2680 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2680 CN**: 继续构造周围的表达式或声明：`} // op`。
- **L2681 EN**: Continues the surrounding expression or declaration: `} // satf`.
  **L2681 CN**: 继续构造周围的表达式或声明：`} // satf`。
- **L2682 EN**: Continues the surrounding expression or declaration: `} // layout_b`.
  **L2682 CN**: 继续构造周围的表达式或声明：`} // layout_b`。
- **L2683 EN**: Continues the surrounding expression or declaration: `} // layout_a`.
  **L2683 CN**: 继续构造周围的表达式或声明：`} // layout_a`。
- **L2684 EN**: Blank line separating nearby declarations or logic blocks.
  **L2684 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2685 EN**: Declares class `NVVM_MMA_BLOCK_SCALE<WMMA_REGS`.
  **L2685 CN**: 声明 class `NVVM_MMA_BLOCK_SCALE<WMMA_REGS`。
- **L2686 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<D.regs,`.
  **L2686 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<D.regs,`。
- **L2687 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(A.regs, B.regs, C.regs,`.
  **L2687 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(A.regs, B.regs, C.regs,`。
- **L2688 EN**: Continues the surrounding expression or declaration: `[`.
  **L2688 CN**: 继续构造周围的表达式或声明：`[`。

### Lines 2689-2720

````tablegen
                llvm_i32_ty,              // scale-a-data
                llvm_i16_ty, llvm_i16_ty, // byte-id-a, thread-id-a
                llvm_i32_ty,              // scale-b-data,
                llvm_i16_ty, llvm_i16_ty, // byte-id-b, thread-id-b
              ]),
              [IntrNoMem, IntrNoCallback]>;

foreach kind = ["mxf4", "mxf4nvf4", "mxf8f6f4"] in {
  foreach scale_vec_size = ["", ".scale_1x", ".scale_2x", ".scale_4x"] in {
    foreach stype = ["ue8m0", "ue4m3"] in {
      foreach op = NVVM_MMA_OPS.all_mma_block_scale_ops in {
        if NVVM_MMA_BLOCK_SCALE_SUPPORTED<op, kind, stype, scale_vec_size>.ret then {
          def MMA_BLOCK_SCALE_NAME<kind, stype, scale_vec_size,
                                   op[0], op[1], op[2], op[3]>.record_name
            : NVVM_MMA_BLOCK_SCALE<op[0], op[1], op[2], op[3]>;
        }
      } // op
    } // stype
  } // scale_vec_size
} // kind

// MMA.SP
class NVVM_MMA_SP<WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D>
  : Intrinsic<D.regs,
              !listconcat(A.regs, B.regs, C.regs, [llvm_i32_ty], [llvm_i32_ty])> {
    int pos = !size(!listconcat(A.regs, B.regs, C.regs, [llvm_i32_ty]));

    // The range [0;num_threads) is for the sparsity selector that indicates the threads
    // which contribute metadata.
    int num_threads = !if(!or(!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "bf16")),
                              !and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "f16")),
                              !and(!eq(A.geom, "m16n8k16"), !eq(A.ptx_elt_type, "tf32")),
````
- **L2689 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,              // scale-a-data`.
  **L2689 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,              // scale-a-data`。
- **L2690 EN**: Continues the surrounding expression or declaration: `llvm_i16_ty, llvm_i16_ty, // byte-id-a, thread-id-a`.
  **L2690 CN**: 继续构造周围的表达式或声明：`llvm_i16_ty, llvm_i16_ty, // byte-id-a, thread-id-a`。
- **L2691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `llvm_i32_ty,              // scale-b-data,`.
  **L2691 CN**: 继续一个多行参数列表、初始化器或聚合项：`llvm_i32_ty,              // scale-b-data,`。
- **L2692 EN**: Continues the surrounding expression or declaration: `llvm_i16_ty, llvm_i16_ty, // byte-id-b, thread-id-b`.
  **L2692 CN**: 继续构造周围的表达式或声明：`llvm_i16_ty, llvm_i16_ty, // byte-id-b, thread-id-b`。
- **L2693 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `]),`.
  **L2693 CN**: 继续一个多行参数列表、初始化器或聚合项：`]),`。
- **L2694 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrNoCallback]>;`.
  **L2694 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrNoCallback]>;`。
- **L2695 EN**: Blank line separating nearby declarations or logic blocks.
  **L2695 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2696 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2696 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2697 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2697 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2698 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2698 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2699 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2699 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2700 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2700 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2701 EN**: Declares TableGen def `MMA_BLOCK_SCALE_NAME`.
  **L2701 CN**: 声明 TableGen def `MMA_BLOCK_SCALE_NAME`。
- **L2702 EN**: Continues the surrounding expression or declaration: `op[0], op[1], op[2], op[3]>.record_name`.
  **L2702 CN**: 继续构造周围的表达式或声明：`op[0], op[1], op[2], op[3]>.record_name`。
- **L2703 EN**: Executes a standalone statement or declaration: `: NVVM_MMA_BLOCK_SCALE<op[0], op[1], op[2], op[3]>;`.
  **L2703 CN**: 执行一条独立语句或声明：`: NVVM_MMA_BLOCK_SCALE<op[0], op[1], op[2], op[3]>;`。
- **L2704 EN**: Closes the current lexical scope or compound statement.
  **L2704 CN**: 结束当前词法作用域或复合语句块。
- **L2705 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2705 CN**: 继续构造周围的表达式或声明：`} // op`。
- **L2706 EN**: Continues the surrounding expression or declaration: `} // stype`.
  **L2706 CN**: 继续构造周围的表达式或声明：`} // stype`。
- **L2707 EN**: Continues the surrounding expression or declaration: `} // scale_vec_size`.
  **L2707 CN**: 继续构造周围的表达式或声明：`} // scale_vec_size`。
- **L2708 EN**: Continues the surrounding expression or declaration: `} // kind`.
  **L2708 CN**: 继续构造周围的表达式或声明：`} // kind`。
- **L2709 EN**: Blank line separating nearby declarations or logic blocks.
  **L2709 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2710 EN**: Comment explains nearby logic, invariants, or intent: `MMA.SP`.
  **L2710 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA.SP`。
- **L2711 EN**: Declares class `NVVM_MMA_SP<WMMA_REGS`.
  **L2711 CN**: 声明 class `NVVM_MMA_SP<WMMA_REGS`。
- **L2712 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<D.regs,`.
  **L2712 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<D.regs,`。
- **L2713 EN**: Starts a function, method, lambda, or structured scope: `!listconcat(A.regs, B.regs, C.regs, [llvm_i32_ty], [llvm_i32_ty])> {`.
  **L2713 CN**: 开始一个函数、方法、lambda 或结构化作用域：`!listconcat(A.regs, B.regs, C.regs, [llvm_i32_ty], [llvm_i32_ty])> {`。
- **L2714 EN**: Initializes variable `pos` from the right-hand expression.
  **L2714 CN**: 使用右侧表达式初始化变量 `pos`。
- **L2715 EN**: Blank line separating nearby declarations or logic blocks.
  **L2715 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2716 EN**: Comment explains nearby logic, invariants, or intent: `The range [0;num_threads) is for the sparsity selector that indicates the threads`.
  **L2716 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The range [0;num_threads) is for the sparsity selector that indicates the threads`。
- **L2717 EN**: Comment explains nearby logic, invariants, or intent: `which contribute metadata.`.
  **L2717 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`which contribute metadata.`。
- **L2718 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `int num_threads = !if(!or(!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "bf16")),`.
  **L2718 CN**: 继续一个多行参数列表、初始化器或聚合项：`int num_threads = !if(!or(!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "bf16")),`。
- **L2719 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "f16")),`.
  **L2719 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "f16")),`。
- **L2720 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(A.geom, "m16n8k16"), !eq(A.ptx_elt_type, "tf32")),`.
  **L2720 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(A.geom, "m16n8k16"), !eq(A.ptx_elt_type, "tf32")),`。

### Lines 2721-2752

````tablegen
                              !and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "u8")),
                              !and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "s8")),
                              !and(!eq(A.geom, "m16n8k64"), !eq(A.ptx_elt_type, "u4")),
                              !and(!eq(A.geom, "m16n8k64"), !eq(A.ptx_elt_type, "s4"))),
                          2,
                          !if(!and(!eq(A.geom, "m16n8k64"),
                                  !or(!eq(A.ptx_elt_type, "u8"),
                                      !eq(A.ptx_elt_type, "s8"),
                                      !eq(A.ptx_elt_type, "e4m3"),
                                      !eq(A.ptx_elt_type, "e5m2"),
                                      !eq(A.ptx_elt_type, "e3m2"),
                                      !eq(A.ptx_elt_type, "e2m3"),
                                      !eq(A.ptx_elt_type, "e2m1"))),
                              1,
                              !if(!and(!eq(A.geom, "m16n8k128"),
                                       !or(!eq(A.ptx_elt_type, "s4"),
                                           !eq(A.ptx_elt_type, "u4"))),
                                  1, 4)));
    let IntrProperties = [IntrNoMem, IntrNoCallback, ImmArg<ArgIndex<pos>>,
                          Range<ArgIndex<pos>, 0, num_threads>];
}

foreach metadata = ["sp", "sp::ordered_metadata"] in {
  foreach kind = ["", "kind::f8f6f4"] in {
    foreach satf = [0, 1] in {
      foreach op = NVVM_MMA_OPS.all_mma_sp_ops in {
        if NVVM_MMA_SP_SUPPORTED<op, metadata, kind, satf>.ret then {
          def MMA_SP_NAME<metadata, kind, satf,
                          op[0], op[1], op[2], op[3]>.record_name
            : NVVM_MMA_SP<op[0], op[1], op[2], op[3]>;
        }
      } // op
````
- **L2721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "u8")),`.
  **L2721 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "u8")),`。
- **L2722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "s8")),`.
  **L2722 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(A.geom, "m16n8k32"), !eq(A.ptx_elt_type, "s8")),`。
- **L2723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(A.geom, "m16n8k64"), !eq(A.ptx_elt_type, "u4")),`.
  **L2723 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(A.geom, "m16n8k64"), !eq(A.ptx_elt_type, "u4")),`。
- **L2724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!and(!eq(A.geom, "m16n8k64"), !eq(A.ptx_elt_type, "s4"))),`.
  **L2724 CN**: 继续一个多行参数列表、初始化器或聚合项：`!and(!eq(A.geom, "m16n8k64"), !eq(A.ptx_elt_type, "s4"))),`。
- **L2725 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `2,`.
  **L2725 CN**: 继续一个多行参数列表、初始化器或聚合项：`2,`。
- **L2726 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!and(!eq(A.geom, "m16n8k64"),`.
  **L2726 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!and(!eq(A.geom, "m16n8k64"),`。
- **L2727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(A.ptx_elt_type, "u8"),`.
  **L2727 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(A.ptx_elt_type, "u8"),`。
- **L2728 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "s8"),`.
  **L2728 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "s8"),`。
- **L2729 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e4m3"),`.
  **L2729 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e4m3"),`。
- **L2730 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e5m2"),`.
  **L2730 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e5m2"),`。
- **L2731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e3m2"),`.
  **L2731 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e3m2"),`。
- **L2732 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e2m3"),`.
  **L2732 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e2m3"),`。
- **L2733 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "e2m1"))),`.
  **L2733 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "e2m1"))),`。
- **L2734 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `1,`.
  **L2734 CN**: 继续一个多行参数列表、初始化器或聚合项：`1,`。
- **L2735 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!and(!eq(A.geom, "m16n8k128"),`.
  **L2735 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!and(!eq(A.geom, "m16n8k128"),`。
- **L2736 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!or(!eq(A.ptx_elt_type, "s4"),`.
  **L2736 CN**: 继续一个多行参数列表、初始化器或聚合项：`!or(!eq(A.ptx_elt_type, "s4"),`。
- **L2737 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(A.ptx_elt_type, "u4"))),`.
  **L2737 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(A.ptx_elt_type, "u4"))),`。
- **L2738 EN**: Executes a standalone statement or declaration: `1, 4)));`.
  **L2738 CN**: 执行一条独立语句或声明：`1, 4)));`。
- **L2739 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2739 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2740 EN**: Executes a standalone statement or declaration: `Range<ArgIndex<pos>, 0, num_threads>];`.
  **L2740 CN**: 执行一条独立语句或声明：`Range<ArgIndex<pos>, 0, num_threads>];`。
- **L2741 EN**: Closes the current lexical scope or compound statement.
  **L2741 CN**: 结束当前词法作用域或复合语句块。
- **L2742 EN**: Blank line separating nearby declarations or logic blocks.
  **L2742 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2743 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2743 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2744 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2744 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2745 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2745 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2746 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2746 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2747 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2747 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2748 EN**: Declares TableGen def `MMA_SP_NAME`.
  **L2748 CN**: 声明 TableGen def `MMA_SP_NAME`。
- **L2749 EN**: Continues the surrounding expression or declaration: `op[0], op[1], op[2], op[3]>.record_name`.
  **L2749 CN**: 继续构造周围的表达式或声明：`op[0], op[1], op[2], op[3]>.record_name`。
- **L2750 EN**: Executes a standalone statement or declaration: `: NVVM_MMA_SP<op[0], op[1], op[2], op[3]>;`.
  **L2750 CN**: 执行一条独立语句或声明：`: NVVM_MMA_SP<op[0], op[1], op[2], op[3]>;`。
- **L2751 EN**: Closes the current lexical scope or compound statement.
  **L2751 CN**: 结束当前词法作用域或复合语句块。
- **L2752 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2752 CN**: 继续构造周围的表达式或声明：`} // op`。

### Lines 2753-2784

````tablegen
    } // satf
  } // kind
} // metadata

// MMA.SP BLOCK SCALE
class NVVM_MMA_SP_BLOCK_SCALE<WMMA_REGS A, WMMA_REGS B, WMMA_REGS C, WMMA_REGS D>
  : Intrinsic<D.regs,
              !listconcat(A.regs, B.regs, C.regs,
              [
                llvm_i32_ty, // metadata
                llvm_i32_ty, // sparsity selector
                llvm_i32_ty, // scale-a-data
                llvm_i16_ty, llvm_i16_ty, // byte-id-a, thread-id-a
                llvm_i32_ty, // scale-b-data
                llvm_i16_ty, llvm_i16_ty, // byte-id-b, thread-id-b
              ])> {
    int pos = !size(!listconcat(A.regs, B.regs, C.regs, [llvm_i32_ty]));

    // The range [0;num_threads) is for the sparsity selector that indicates the threads
    // which contribute metadata.
    // According to PTX ISA 9.0, the sparsity selector is always 0
    // for sparse MMA block scale instructions
    int num_threads = 1;
    let IntrProperties = [IntrNoMem, IntrNoCallback, ImmArg<ArgIndex<pos>>,
                          Range<ArgIndex<pos>, 0, num_threads>];
}

// According to PTX ISA 9.0
// a_layout = ["row"], b_layout = ["col"], spvariant = ["sp::ordered_metadata"]
foreach kind = ["mxf4", "mxf4nvf4", "mxf8f6f4"] in {
  foreach scale_vec_size = ["", ".scale_1x", ".scale_2x", ".scale_4x"] in {
    foreach stype = ["ue8m0", "ue4m3"] in {
````
- **L2753 EN**: Continues the surrounding expression or declaration: `} // satf`.
  **L2753 CN**: 继续构造周围的表达式或声明：`} // satf`。
- **L2754 EN**: Continues the surrounding expression or declaration: `} // kind`.
  **L2754 CN**: 继续构造周围的表达式或声明：`} // kind`。
- **L2755 EN**: Continues the surrounding expression or declaration: `} // metadata`.
  **L2755 CN**: 继续构造周围的表达式或声明：`} // metadata`。
- **L2756 EN**: Blank line separating nearby declarations or logic blocks.
  **L2756 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2757 EN**: Comment explains nearby logic, invariants, or intent: `MMA.SP BLOCK SCALE`.
  **L2757 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MMA.SP BLOCK SCALE`。
- **L2758 EN**: Declares class `NVVM_MMA_SP_BLOCK_SCALE<WMMA_REGS`.
  **L2758 CN**: 声明 class `NVVM_MMA_SP_BLOCK_SCALE<WMMA_REGS`。
- **L2759 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<D.regs,`.
  **L2759 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<D.regs,`。
- **L2760 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(A.regs, B.regs, C.regs,`.
  **L2760 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(A.regs, B.regs, C.regs,`。
- **L2761 EN**: Continues the surrounding expression or declaration: `[`.
  **L2761 CN**: 继续构造周围的表达式或声明：`[`。
- **L2762 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // metadata`.
  **L2762 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // metadata`。
- **L2763 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // sparsity selector`.
  **L2763 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // sparsity selector`。
- **L2764 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // scale-a-data`.
  **L2764 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // scale-a-data`。
- **L2765 EN**: Continues the surrounding expression or declaration: `llvm_i16_ty, llvm_i16_ty, // byte-id-a, thread-id-a`.
  **L2765 CN**: 继续构造周围的表达式或声明：`llvm_i16_ty, llvm_i16_ty, // byte-id-a, thread-id-a`。
- **L2766 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty, // scale-b-data`.
  **L2766 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty, // scale-b-data`。
- **L2767 EN**: Continues the surrounding expression or declaration: `llvm_i16_ty, llvm_i16_ty, // byte-id-b, thread-id-b`.
  **L2767 CN**: 继续构造周围的表达式或声明：`llvm_i16_ty, llvm_i16_ty, // byte-id-b, thread-id-b`。
- **L2768 EN**: Continues the surrounding expression or declaration: `])> {`.
  **L2768 CN**: 继续构造周围的表达式或声明：`])> {`。
- **L2769 EN**: Initializes variable `pos` from the right-hand expression.
  **L2769 CN**: 使用右侧表达式初始化变量 `pos`。
- **L2770 EN**: Blank line separating nearby declarations or logic blocks.
  **L2770 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2771 EN**: Comment explains nearby logic, invariants, or intent: `The range [0;num_threads) is for the sparsity selector that indicates the threads`.
  **L2771 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The range [0;num_threads) is for the sparsity selector that indicates the threads`。
- **L2772 EN**: Comment explains nearby logic, invariants, or intent: `which contribute metadata.`.
  **L2772 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`which contribute metadata.`。
- **L2773 EN**: Comment explains nearby logic, invariants, or intent: `According to PTX ISA 9.0, the sparsity selector is always 0`.
  **L2773 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`According to PTX ISA 9.0, the sparsity selector is always 0`。
- **L2774 EN**: Comment explains nearby logic, invariants, or intent: `for sparse MMA block scale instructions`.
  **L2774 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`for sparse MMA block scale instructions`。
- **L2775 EN**: Initializes variable `num_threads` from the right-hand expression.
  **L2775 CN**: 使用右侧表达式初始化变量 `num_threads`。
- **L2776 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2776 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2777 EN**: Executes a standalone statement or declaration: `Range<ArgIndex<pos>, 0, num_threads>];`.
  **L2777 CN**: 执行一条独立语句或声明：`Range<ArgIndex<pos>, 0, num_threads>];`。
- **L2778 EN**: Closes the current lexical scope or compound statement.
  **L2778 CN**: 结束当前词法作用域或复合语句块。
- **L2779 EN**: Blank line separating nearby declarations or logic blocks.
  **L2779 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2780 EN**: Comment explains nearby logic, invariants, or intent: `According to PTX ISA 9.0`.
  **L2780 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`According to PTX ISA 9.0`。
- **L2781 EN**: Comment explains nearby logic, invariants, or intent: `a_layout = ["row"], b_layout = ["col"], spvariant = ["sp::ordered_metadata"]`.
  **L2781 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`a_layout = ["row"], b_layout = ["col"], spvariant = ["sp::ordered_metadata"]`。
- **L2782 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2782 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2783 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2783 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2784 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2784 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 2785-2816

````tablegen
      foreach op = NVVM_MMA_OPS.all_mma_sp_block_scale_ops in {
        if NVVM_MMA_SP_BLOCK_SCALE_SUPPORTED<op, kind, stype, scale_vec_size>.ret then {
          def MMA_SP_BLOCK_SCALE_NAME<kind, stype, scale_vec_size,
                                      op[0], op[1], op[2], op[3]>.record_name
            : NVVM_MMA_SP_BLOCK_SCALE<op[0], op[1], op[2], op[3]>;
        }
      } // op
    } // stype
  } // scale_vec_size
} // kind

// LDMATRIX
class NVVM_LDMATRIX<WMMA_REGS Frag, int Transposed>
  : Intrinsic<Frag.regs, [llvm_anyptr_ty],
              [IntrReadMem, IntrArgMemOnly, IntrNoCallback, IntrConvergent,
               ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],
              LDMATRIX_NAME<Frag, Transposed>.intr_name>;

foreach transposed = [0, 1] in {
  foreach frag = NVVM_MMA_OPS.all_ldmatrix_ops in {
    if NVVM_LDMATRIX_SUPPORTED<frag, transposed>.ret then {
      def LDMATRIX_NAME<frag, transposed>.record_name
        : NVVM_LDMATRIX<frag, transposed>;
    }
  }
}

// STMATRIX
class NVVM_STMATRIX<WMMA_REGS Frag, int Transposed>
  : Intrinsic<[],
          !listconcat([llvm_anyptr_ty], Frag.regs),
          [IntrWriteMem, IntrArgMemOnly, IntrNoCallback, IntrConvergent,
````
- **L2785 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2785 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2786 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2786 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2787 EN**: Declares TableGen def `MMA_SP_BLOCK_SCALE_NAME`.
  **L2787 CN**: 声明 TableGen def `MMA_SP_BLOCK_SCALE_NAME`。
- **L2788 EN**: Continues the surrounding expression or declaration: `op[0], op[1], op[2], op[3]>.record_name`.
  **L2788 CN**: 继续构造周围的表达式或声明：`op[0], op[1], op[2], op[3]>.record_name`。
- **L2789 EN**: Executes a standalone statement or declaration: `: NVVM_MMA_SP_BLOCK_SCALE<op[0], op[1], op[2], op[3]>;`.
  **L2789 CN**: 执行一条独立语句或声明：`: NVVM_MMA_SP_BLOCK_SCALE<op[0], op[1], op[2], op[3]>;`。
- **L2790 EN**: Closes the current lexical scope or compound statement.
  **L2790 CN**: 结束当前词法作用域或复合语句块。
- **L2791 EN**: Continues the surrounding expression or declaration: `} // op`.
  **L2791 CN**: 继续构造周围的表达式或声明：`} // op`。
- **L2792 EN**: Continues the surrounding expression or declaration: `} // stype`.
  **L2792 CN**: 继续构造周围的表达式或声明：`} // stype`。
- **L2793 EN**: Continues the surrounding expression or declaration: `} // scale_vec_size`.
  **L2793 CN**: 继续构造周围的表达式或声明：`} // scale_vec_size`。
- **L2794 EN**: Continues the surrounding expression or declaration: `} // kind`.
  **L2794 CN**: 继续构造周围的表达式或声明：`} // kind`。
- **L2795 EN**: Blank line separating nearby declarations or logic blocks.
  **L2795 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2796 EN**: Comment explains nearby logic, invariants, or intent: `LDMATRIX`.
  **L2796 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`LDMATRIX`。
- **L2797 EN**: Declares class `NVVM_LDMATRIX<WMMA_REGS`.
  **L2797 CN**: 声明 class `NVVM_LDMATRIX<WMMA_REGS`。
- **L2798 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<Frag.regs, [llvm_anyptr_ty],`.
  **L2798 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<Frag.regs, [llvm_anyptr_ty],`。
- **L2799 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrReadMem, IntrArgMemOnly, IntrNoCallback, IntrConvergent,`.
  **L2799 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrReadMem, IntrArgMemOnly, IntrNoCallback, IntrConvergent,`。
- **L2800 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`.
  **L2800 CN**: 继续一个多行参数列表、初始化器或聚合项：`ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`。
- **L2801 EN**: Executes a standalone statement or declaration: `LDMATRIX_NAME<Frag, Transposed>.intr_name>;`.
  **L2801 CN**: 执行一条独立语句或声明：`LDMATRIX_NAME<Frag, Transposed>.intr_name>;`。
- **L2802 EN**: Blank line separating nearby declarations or logic blocks.
  **L2802 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2803 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2803 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2804 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2804 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2805 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2805 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2806 EN**: Declares TableGen def `LDMATRIX_NAME`.
  **L2806 CN**: 声明 TableGen def `LDMATRIX_NAME`。
- **L2807 EN**: Executes a standalone statement or declaration: `: NVVM_LDMATRIX<frag, transposed>;`.
  **L2807 CN**: 执行一条独立语句或声明：`: NVVM_LDMATRIX<frag, transposed>;`。
- **L2808 EN**: Closes the current lexical scope or compound statement.
  **L2808 CN**: 结束当前词法作用域或复合语句块。
- **L2809 EN**: Closes the current lexical scope or compound statement.
  **L2809 CN**: 结束当前词法作用域或复合语句块。
- **L2810 EN**: Closes the current lexical scope or compound statement.
  **L2810 CN**: 结束当前词法作用域或复合语句块。
- **L2811 EN**: Blank line separating nearby declarations or logic blocks.
  **L2811 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2812 EN**: Comment explains nearby logic, invariants, or intent: `STMATRIX`.
  **L2812 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`STMATRIX`。
- **L2813 EN**: Declares class `NVVM_STMATRIX<WMMA_REGS`.
  **L2813 CN**: 声明 class `NVVM_STMATRIX<WMMA_REGS`。
- **L2814 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[],`.
  **L2814 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[],`。
- **L2815 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_anyptr_ty], Frag.regs),`.
  **L2815 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_anyptr_ty], Frag.regs),`。
- **L2816 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrWriteMem, IntrArgMemOnly, IntrNoCallback, IntrConvergent,`.
  **L2816 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrWriteMem, IntrArgMemOnly, IntrNoCallback, IntrConvergent,`。

### Lines 2817-2848

````tablegen
           WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],
          STMATRIX_NAME<Frag, Transposed>.intr_name>;

foreach transposed = [0, 1] in {
  foreach frag = NVVM_MMA_OPS.all_stmatrix_ops in {
    if NVVM_STMATRIX_SUPPORTED<frag, transposed>.ret then {
      def STMATRIX_NAME<frag, transposed>.record_name
        : NVVM_STMATRIX<frag, transposed>;
    }
  }
}

// movmatrix
def int_nvvm_movmatrix_sync_aligned_m8n8_trans_b16
  : Intrinsic<[llvm_i32_ty], [llvm_i32_ty],
              [IntrInaccessibleMemOnly, IntrConvergent,
               IntrNoCallback, IntrNoFree]>;

// MAPA
let IntrProperties = [IntrNoMem, IntrSpeculatable, NoCapture<ArgIndex<0>>] in {
  def int_nvvm_mapa
    : DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty]>;
  def int_nvvm_mapa_shared_cluster
    : DefaultAttrsIntrinsic<[llvm_shared_cluster_ptr_ty], [llvm_shared_ptr_ty, llvm_i32_ty]>;
}

// GETCTARANK
let IntrProperties = [IntrNoMem, IntrSpeculatable, NoCapture<ArgIndex<0>>] in {
  def int_nvvm_getctarank
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty]>;
  def int_nvvm_getctarank_shared_cluster
    : DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_shared_ptr_ty]>;
````
- **L2817 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`.
  **L2817 CN**: 继续一个多行参数列表、初始化器或聚合项：`WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>],`。
- **L2818 EN**: Executes a standalone statement or declaration: `STMATRIX_NAME<Frag, Transposed>.intr_name>;`.
  **L2818 CN**: 执行一条独立语句或声明：`STMATRIX_NAME<Frag, Transposed>.intr_name>;`。
- **L2819 EN**: Blank line separating nearby declarations or logic blocks.
  **L2819 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2820 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2820 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2821 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2821 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2822 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L2822 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2823 EN**: Declares TableGen def `STMATRIX_NAME`.
  **L2823 CN**: 声明 TableGen def `STMATRIX_NAME`。
- **L2824 EN**: Executes a standalone statement or declaration: `: NVVM_STMATRIX<frag, transposed>;`.
  **L2824 CN**: 执行一条独立语句或声明：`: NVVM_STMATRIX<frag, transposed>;`。
- **L2825 EN**: Closes the current lexical scope or compound statement.
  **L2825 CN**: 结束当前词法作用域或复合语句块。
- **L2826 EN**: Closes the current lexical scope or compound statement.
  **L2826 CN**: 结束当前词法作用域或复合语句块。
- **L2827 EN**: Closes the current lexical scope or compound statement.
  **L2827 CN**: 结束当前词法作用域或复合语句块。
- **L2828 EN**: Blank line separating nearby declarations or logic blocks.
  **L2828 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2829 EN**: Comment explains nearby logic, invariants, or intent: `movmatrix`.
  **L2829 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`movmatrix`。
- **L2830 EN**: Declares TableGen def `int_nvvm_movmatrix_sync_aligned_m8n8_trans_b16`.
  **L2830 CN**: 声明 TableGen def `int_nvvm_movmatrix_sync_aligned_m8n8_trans_b16`。
- **L2831 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Intrinsic<[llvm_i32_ty], [llvm_i32_ty],`.
  **L2831 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Intrinsic<[llvm_i32_ty], [llvm_i32_ty],`。
- **L2832 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrInaccessibleMemOnly, IntrConvergent,`.
  **L2832 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrInaccessibleMemOnly, IntrConvergent,`。
- **L2833 EN**: Executes a standalone statement or declaration: `IntrNoCallback, IntrNoFree]>;`.
  **L2833 CN**: 执行一条独立语句或声明：`IntrNoCallback, IntrNoFree]>;`。
- **L2834 EN**: Blank line separating nearby declarations or logic blocks.
  **L2834 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2835 EN**: Comment explains nearby logic, invariants, or intent: `MAPA`.
  **L2835 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`MAPA`。
- **L2836 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2836 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2837 EN**: Declares TableGen def `int_nvvm_mapa`.
  **L2837 CN**: 声明 TableGen def `int_nvvm_mapa`。
- **L2838 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty]>;`.
  **L2838 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_ptr_ty], [llvm_ptr_ty, llvm_i32_ty]>;`。
- **L2839 EN**: Declares TableGen def `int_nvvm_mapa_shared_cluster`.
  **L2839 CN**: 声明 TableGen def `int_nvvm_mapa_shared_cluster`。
- **L2840 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_shared_cluster_ptr_ty], [llvm_shared_ptr_ty, llvm_i32_ty]>;`.
  **L2840 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_shared_cluster_ptr_ty], [llvm_shared_ptr_ty, llvm_i32_ty]>;`。
- **L2841 EN**: Closes the current lexical scope or compound statement.
  **L2841 CN**: 结束当前词法作用域或复合语句块。
- **L2842 EN**: Blank line separating nearby declarations or logic blocks.
  **L2842 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2843 EN**: Comment explains nearby logic, invariants, or intent: `GETCTARANK`.
  **L2843 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`GETCTARANK`。
- **L2844 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L2844 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2845 EN**: Declares TableGen def `int_nvvm_getctarank`.
  **L2845 CN**: 声明 TableGen def `int_nvvm_getctarank`。
- **L2846 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty]>;`.
  **L2846 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_ptr_ty]>;`。
- **L2847 EN**: Declares TableGen def `int_nvvm_getctarank_shared_cluster`.
  **L2847 CN**: 声明 TableGen def `int_nvvm_getctarank_shared_cluster`。
- **L2848 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_shared_ptr_ty]>;`.
  **L2848 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[llvm_i32_ty], [llvm_shared_ptr_ty]>;`。

### Lines 2849-2880

````tablegen
}

def int_nvvm_is_explicit_cluster
  : NVVMPureIntrinsic<[llvm_i1_ty], [], [NoUndef<RetIndex>],
              "llvm.nvvm.is_explicit_cluster">;

// Setmaxnreg inc/dec intrinsics
// The imm-arg should be in the range: 24 <= val <= 256
foreach op = ["dec", "inc"] in
  def int_nvvm_setmaxnreg_ # op # _sync_aligned_u32
    : DefaultAttrsIntrinsic<[], [llvm_i32_ty],
              [IntrConvergent, IntrNoMem, IntrHasSideEffects,
               ImmArg<ArgIndex<0>>, Range<ArgIndex<0>, 24, 257>]>;

// Exit
def int_nvvm_exit : NVVMBuiltin,
    Intrinsic<[], [], [IntrConvergent, IntrInaccessibleMemOnly, IntrNoReturn]>;

class DefaultAttrsIntrinsicFlags<list<LLVMType> ret_types,
                list<LLVMType> param_types,
                list<LLVMType> flags,
                list<IntrinsicProperty> intr_properties,
                string name = "">
  : DefaultAttrsIntrinsic<
        ret_types,
        !listconcat(param_types, flags),
        !listconcat(intr_properties,
                    !foreach(i, !range(flags),
                        ImmArg<ArgIndex<!add(i, !size(param_types))>>)),
        name>;

// TMA Tensor Copy Intrinsics: S2G -> From Shared to Global memory variants
````
- **L2849 EN**: Closes the current lexical scope or compound statement.
  **L2849 CN**: 结束当前词法作用域或复合语句块。
- **L2850 EN**: Blank line separating nearby declarations or logic blocks.
  **L2850 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2851 EN**: Declares TableGen def `int_nvvm_is_explicit_cluster`.
  **L2851 CN**: 声明 TableGen def `int_nvvm_is_explicit_cluster`。
- **L2852 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: NVVMPureIntrinsic<[llvm_i1_ty], [], [NoUndef<RetIndex>],`.
  **L2852 CN**: 继续一个多行参数列表、初始化器或聚合项：`: NVVMPureIntrinsic<[llvm_i1_ty], [], [NoUndef<RetIndex>],`。
- **L2853 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.is_explicit_cluster">;`.
  **L2853 CN**: 执行一条独立语句或声明：`"llvm.nvvm.is_explicit_cluster">;`。
- **L2854 EN**: Blank line separating nearby declarations or logic blocks.
  **L2854 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2855 EN**: Comment explains nearby logic, invariants, or intent: `Setmaxnreg inc/dec intrinsics`.
  **L2855 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Setmaxnreg inc/dec intrinsics`。
- **L2856 EN**: Comment explains nearby logic, invariants, or intent: `The imm-arg should be in the range: 24 <= val <= 256`.
  **L2856 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`The imm-arg should be in the range: 24 <= val <= 256`。
- **L2857 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2857 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2858 EN**: Declares TableGen def `int_nvvm_setmaxnreg_`.
  **L2858 CN**: 声明 TableGen def `int_nvvm_setmaxnreg_`。
- **L2859 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_i32_ty],`.
  **L2859 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_i32_ty],`。
- **L2860 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrNoMem, IntrHasSideEffects,`.
  **L2860 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrNoMem, IntrHasSideEffects,`。
- **L2861 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<0>>, Range<ArgIndex<0>, 24, 257>]>;`.
  **L2861 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<0>>, Range<ArgIndex<0>, 24, 257>]>;`。
- **L2862 EN**: Blank line separating nearby declarations or logic blocks.
  **L2862 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2863 EN**: Comment explains nearby logic, invariants, or intent: `Exit`.
  **L2863 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Exit`。
- **L2864 EN**: Declares TableGen def `int_nvvm_exit`.
  **L2864 CN**: 声明 TableGen def `int_nvvm_exit`。
- **L2865 EN**: Executes a standalone statement or declaration: `Intrinsic<[], [], [IntrConvergent, IntrInaccessibleMemOnly, IntrNoReturn]>;`.
  **L2865 CN**: 执行一条独立语句或声明：`Intrinsic<[], [], [IntrConvergent, IntrInaccessibleMemOnly, IntrNoReturn]>;`。
- **L2866 EN**: Blank line separating nearby declarations or logic blocks.
  **L2866 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2867 EN**: Declares class `DefaultAttrsIntrinsicFlags<list<LLVMType>`.
  **L2867 CN**: 声明 class `DefaultAttrsIntrinsicFlags<list<LLVMType>`。
- **L2868 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> param_types,`.
  **L2868 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> param_types,`。
- **L2869 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<LLVMType> flags,`.
  **L2869 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<LLVMType> flags,`。
- **L2870 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `list<IntrinsicProperty> intr_properties,`.
  **L2870 CN**: 继续一个多行参数列表、初始化器或聚合项：`list<IntrinsicProperty> intr_properties,`。
- **L2871 EN**: Continues the surrounding expression or declaration: `string name = "">`.
  **L2871 CN**: 继续构造周围的表达式或声明：`string name = "">`。
- **L2872 EN**: Continues the surrounding expression or declaration: `: DefaultAttrsIntrinsic<`.
  **L2872 CN**: 继续构造周围的表达式或声明：`: DefaultAttrsIntrinsic<`。
- **L2873 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ret_types,`.
  **L2873 CN**: 继续一个多行参数列表、初始化器或聚合项：`ret_types,`。
- **L2874 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(param_types, flags),`.
  **L2874 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(param_types, flags),`。
- **L2875 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(intr_properties,`.
  **L2875 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(intr_properties,`。
- **L2876 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!foreach(i, !range(flags),`.
  **L2876 CN**: 继续一个多行参数列表、初始化器或聚合项：`!foreach(i, !range(flags),`。
- **L2877 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ImmArg<ArgIndex<!add(i, !size(param_types))>>)),`.
  **L2877 CN**: 继续一个多行参数列表、初始化器或聚合项：`ImmArg<ArgIndex<!add(i, !size(param_types))>>)),`。
- **L2878 EN**: Executes a standalone statement or declaration: `name>;`.
  **L2878 CN**: 执行一条独立语句或声明：`name>;`。
- **L2879 EN**: Blank line separating nearby declarations or logic blocks.
  **L2879 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2880 EN**: Comment explains nearby logic, invariants, or intent: `TMA Tensor Copy Intrinsics: S2G -> From Shared to Global memory variants`.
  **L2880 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TMA Tensor Copy Intrinsics: S2G -> From Shared to Global memory variants`。

### Lines 2881-2912

````tablegen
foreach dim = 1...5 in {
  defvar tensor_dim_args = !listsplat(llvm_i32_ty, dim);
  foreach mode = !if(!ge(dim, 3), ["tile", "im2col"], ["tile"]) in {
    def int_nvvm_cp_async_bulk_tensor_s2g_ # mode # _ # dim # d :
      DefaultAttrsIntrinsicFlags<[],
          !listconcat([llvm_shared_ptr_ty,  // src_smem_ptr
                       llvm_ptr_ty],        // tensormap_ptr
                      tensor_dim_args,      // actual tensor dims
                      [llvm_i64_ty]),       // cache_hint
          [llvm_i1_ty],                     // Flag for cache_hint
          [IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;

    // Intrinsics for TMA Copy with reduction
    foreach red_op = ["add", "min", "max", "inc", "dec", "and", "or", "xor"] in
      def int_nvvm_cp_async_bulk_tensor_reduce_ # red_op # _ # mode # _ # dim # d :
        DefaultAttrsIntrinsicFlags<[],
            !listconcat([llvm_shared_ptr_ty,  // src_smem_ptr
                         llvm_ptr_ty],        // tensormap_ptr
                         tensor_dim_args,     // actual tensor dims
                        [llvm_i64_ty]),       // cache_hint
          [llvm_i1_ty],                       // Flag for cache_hint
          [IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;
  }
}

// TMA S2G tile::scatter4
def int_nvvm_cp_async_bulk_tensor_s2g_tile_scatter4_2d
  : DefaultAttrsIntrinsicFlags<[],
      !listconcat([llvm_shared_ptr_ty,        // src_smem_ptr
                   llvm_ptr_ty],              // tensormap_ptr
                  !listsplat(llvm_i32_ty, 5), // dims
                  [llvm_i64_ty]),             // cache_hint
````
- **L2881 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2881 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2882 EN**: Initializes variable `tensor_dim_args` from the right-hand expression.
  **L2882 CN**: 使用右侧表达式初始化变量 `tensor_dim_args`。
- **L2883 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2883 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2884 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_s2g_`.
  **L2884 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_s2g_`。
- **L2885 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsicFlags<[],`.
  **L2885 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsicFlags<[],`。
- **L2886 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2886 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2887 EN**: Continues the surrounding expression or declaration: `llvm_ptr_ty],        // tensormap_ptr`.
  **L2887 CN**: 继续构造周围的表达式或声明：`llvm_ptr_ty],        // tensormap_ptr`。
- **L2888 EN**: Continues the surrounding expression or declaration: `tensor_dim_args,      // actual tensor dims`.
  **L2888 CN**: 继续构造周围的表达式或声明：`tensor_dim_args,      // actual tensor dims`。
- **L2889 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty]),       // cache_hint`.
  **L2889 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty]),       // cache_hint`。
- **L2890 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],                     // Flag for cache_hint`.
  **L2890 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],                     // Flag for cache_hint`。
- **L2891 EN**: Executes a standalone statement or declaration: `[IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`.
  **L2891 CN**: 执行一条独立语句或声明：`[IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`。
- **L2892 EN**: Blank line separating nearby declarations or logic blocks.
  **L2892 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2893 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsics for TMA Copy with reduction`.
  **L2893 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsics for TMA Copy with reduction`。
- **L2894 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2894 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2895 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_reduce_`.
  **L2895 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_reduce_`。
- **L2896 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsicFlags<[],`.
  **L2896 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsicFlags<[],`。
- **L2897 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2897 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2898 EN**: Continues the surrounding expression or declaration: `llvm_ptr_ty],        // tensormap_ptr`.
  **L2898 CN**: 继续构造周围的表达式或声明：`llvm_ptr_ty],        // tensormap_ptr`。
- **L2899 EN**: Continues the surrounding expression or declaration: `tensor_dim_args,     // actual tensor dims`.
  **L2899 CN**: 继续构造周围的表达式或声明：`tensor_dim_args,     // actual tensor dims`。
- **L2900 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty]),       // cache_hint`.
  **L2900 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty]),       // cache_hint`。
- **L2901 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],                       // Flag for cache_hint`.
  **L2901 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],                       // Flag for cache_hint`。
- **L2902 EN**: Executes a standalone statement or declaration: `[IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`.
  **L2902 CN**: 执行一条独立语句或声明：`[IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`。
- **L2903 EN**: Closes the current lexical scope or compound statement.
  **L2903 CN**: 结束当前词法作用域或复合语句块。
- **L2904 EN**: Closes the current lexical scope or compound statement.
  **L2904 CN**: 结束当前词法作用域或复合语句块。
- **L2905 EN**: Blank line separating nearby declarations or logic blocks.
  **L2905 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2906 EN**: Comment explains nearby logic, invariants, or intent: `TMA S2G tile::scatter4`.
  **L2906 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TMA S2G tile::scatter4`。
- **L2907 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_s2g_tile_scatter4_2d`.
  **L2907 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_s2g_tile_scatter4_2d`。
- **L2908 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L2908 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L2909 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2909 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2910 EN**: Continues the surrounding expression or declaration: `llvm_ptr_ty],              // tensormap_ptr`.
  **L2910 CN**: 继续构造周围的表达式或声明：`llvm_ptr_ty],              // tensormap_ptr`。
- **L2911 EN**: Continues logic associated with callable symbol `listsplat`.
  **L2911 CN**: 继续与可调用符号 `listsplat` 相关的逻辑。
- **L2912 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty]),             // cache_hint`.
  **L2912 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty]),             // cache_hint`。

### Lines 2913-2944

````tablegen
      [llvm_i1_ty],                           // Flag for cache_hint
      [IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;

// TMA Tensor Copy Intrinsics: G2S -> From Global to Shared memory variants
foreach dim = 1...5 in {
  defvar tensor_dim_args = !listsplat(llvm_i32_ty, dim);

  foreach mode = !if(!ge(dim, 3), ["tile", "im2col", "im2col_w", "im2col_w_128"], ["tile"]) in {
    defvar is_im2col = !eq(mode, "im2col");
    defvar is_im2colw = !or(!eq(mode, "im2col_w"), !eq(mode, "im2col_w_128"));

    // For im2col_w/w128 modes, the num_offsets is always 2.
    // For im2col mode, the num_offsets is (dim - 2).
    defvar num_im2col_offsets = !if(is_im2colw, 2, !if(is_im2col, !add(dim, -2), 0));
    defvar im2col_offsets_args = !listsplat(llvm_i16_ty, num_im2col_offsets);

    defvar g2s_params = !listconcat(
                          [llvm_shared_cluster_ptr_ty, // dst_ptr
                           llvm_shared_ptr_ty,  // mbarrier_ptr
                           llvm_ptr_ty],        // tensormap_ptr
                          tensor_dim_args,      // actual tensor dims
                          im2col_offsets_args,  // im2col offsets
                          [llvm_i16_ty,         // cta_mask
                           llvm_i64_ty]);       // cache_hint
    defvar g2s_flags = [llvm_i1_ty,             // Flag for cta_mask
                        llvm_i1_ty,             // Flag for cache_hint
                        llvm_i32_ty];           // Flag for cta_group
    defvar cta_group_idx = !add(
                             !size(g2s_params),
                             !sub(!size(g2s_flags), 1));
    defvar g2s_props = [IntrConvergent,
                        WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>,
````
- **L2913 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],                           // Flag for cache_hint`.
  **L2913 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],                           // Flag for cache_hint`。
- **L2914 EN**: Executes a standalone statement or declaration: `[IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`.
  **L2914 CN**: 执行一条独立语句或声明：`[IntrConvergent, ReadOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`。
- **L2915 EN**: Blank line separating nearby declarations or logic blocks.
  **L2915 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2916 EN**: Comment explains nearby logic, invariants, or intent: `TMA Tensor Copy Intrinsics: G2S -> From Global to Shared memory variants`.
  **L2916 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TMA Tensor Copy Intrinsics: G2S -> From Global to Shared memory variants`。
- **L2917 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2917 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2918 EN**: Initializes variable `tensor_dim_args` from the right-hand expression.
  **L2918 CN**: 使用右侧表达式初始化变量 `tensor_dim_args`。
- **L2919 EN**: Blank line separating nearby declarations or logic blocks.
  **L2919 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2920 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L2920 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L2921 EN**: Initializes variable `is_im2col` from the right-hand expression.
  **L2921 CN**: 使用右侧表达式初始化变量 `is_im2col`。
- **L2922 EN**: Initializes variable `is_im2colw` from the right-hand expression.
  **L2922 CN**: 使用右侧表达式初始化变量 `is_im2colw`。
- **L2923 EN**: Blank line separating nearby declarations or logic blocks.
  **L2923 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2924 EN**: Comment explains nearby logic, invariants, or intent: `For im2col_w/w128 modes, the num_offsets is always 2.`.
  **L2924 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For im2col_w/w128 modes, the num_offsets is always 2.`。
- **L2925 EN**: Comment explains nearby logic, invariants, or intent: `For im2col mode, the num_offsets is (dim - 2).`.
  **L2925 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`For im2col mode, the num_offsets is (dim - 2).`。
- **L2926 EN**: Initializes variable `num_im2col_offsets` from the right-hand expression.
  **L2926 CN**: 使用右侧表达式初始化变量 `num_im2col_offsets`。
- **L2927 EN**: Initializes variable `im2col_offsets_args` from the right-hand expression.
  **L2927 CN**: 使用右侧表达式初始化变量 `im2col_offsets_args`。
- **L2928 EN**: Blank line separating nearby declarations or logic blocks.
  **L2928 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2929 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2929 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2930 EN**: Continues the surrounding expression or declaration: `[llvm_shared_cluster_ptr_ty, // dst_ptr`.
  **L2930 CN**: 继续构造周围的表达式或声明：`[llvm_shared_cluster_ptr_ty, // dst_ptr`。
- **L2931 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty,  // mbarrier_ptr`.
  **L2931 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty,  // mbarrier_ptr`。
- **L2932 EN**: Continues the surrounding expression or declaration: `llvm_ptr_ty],        // tensormap_ptr`.
  **L2932 CN**: 继续构造周围的表达式或声明：`llvm_ptr_ty],        // tensormap_ptr`。
- **L2933 EN**: Continues the surrounding expression or declaration: `tensor_dim_args,      // actual tensor dims`.
  **L2933 CN**: 继续构造周围的表达式或声明：`tensor_dim_args,      // actual tensor dims`。
- **L2934 EN**: Continues the surrounding expression or declaration: `im2col_offsets_args,  // im2col offsets`.
  **L2934 CN**: 继续构造周围的表达式或声明：`im2col_offsets_args,  // im2col offsets`。
- **L2935 EN**: Continues the surrounding expression or declaration: `[llvm_i16_ty,         // cta_mask`.
  **L2935 CN**: 继续构造周围的表达式或声明：`[llvm_i16_ty,         // cta_mask`。
- **L2936 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty]);       // cache_hint`.
  **L2936 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty]);       // cache_hint`。
- **L2937 EN**: Continues the surrounding expression or declaration: `defvar g2s_flags = [llvm_i1_ty,             // Flag for cta_mask`.
  **L2937 CN**: 继续构造周围的表达式或声明：`defvar g2s_flags = [llvm_i1_ty,             // Flag for cta_mask`。
- **L2938 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,             // Flag for cache_hint`.
  **L2938 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,             // Flag for cache_hint`。
- **L2939 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty];           // Flag for cta_group`.
  **L2939 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty];           // Flag for cta_group`。
- **L2940 EN**: Continues logic associated with callable symbol `add`.
  **L2940 CN**: 继续与可调用符号 `add` 相关的逻辑。
- **L2941 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!size(g2s_params),`.
  **L2941 CN**: 继续一个多行参数列表、初始化器或聚合项：`!size(g2s_params),`。
- **L2942 EN**: Executes a call or declaration centered on `!sub`.
  **L2942 CN**: 执行以 `!sub` 为核心的调用或声明。
- **L2943 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar g2s_props = [IntrConvergent,`.
  **L2943 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar g2s_props = [IntrConvergent,`。
- **L2944 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>,`.
  **L2944 CN**: 继续一个多行参数列表、初始化器或聚合项：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>,`。

### Lines 2945-2976

````tablegen
                        // Allowed values for cta_group are {0,1,2} i.e [0, 3).
                        Range<ArgIndex<cta_group_idx>, 0, 3>];
    def int_nvvm_cp_async_bulk_tensor_g2s_ # mode # _ # dim # d :
      DefaultAttrsIntrinsicFlags<[], g2s_params, g2s_flags, g2s_props>;

    def int_nvvm_cp_async_bulk_tensor_prefetch_ # mode # _ # dim # d :
      DefaultAttrsIntrinsicFlags<[],
          !listconcat([llvm_ptr_ty],        // tensormap_ptr
                       tensor_dim_args,     // actual tensor dims
                       im2col_offsets_args, // im2col offsets
                      [llvm_i64_ty]),       // cache_hint
          [llvm_i1_ty],                     // Flag for cache_hint
          [IntrConvergent, ReadOnly<ArgIndex<0>>]>;

    def int_nvvm_cp_async_bulk_tensor_g2s_cta_ # mode # _ # dim # d :
      DefaultAttrsIntrinsicFlags<[],
          !listconcat([llvm_shared_ptr_ty,  // dst_ptr
                       llvm_shared_ptr_ty,  // mbarrier_ptr
                       llvm_ptr_ty],        // tensormap_ptr
                       tensor_dim_args,     // actual tensor dims
                       im2col_offsets_args, // im2col offsets
                       [llvm_i64_ty]),      // cache_hint
          [llvm_i1_ty],                     // Flag for cache_hint
          [IntrConvergent, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;
  }
}

// TMA copy for tile::gather4
def int_nvvm_cp_async_bulk_tensor_g2s_tile_gather4_2d
  : DefaultAttrsIntrinsicFlags<[],
      !listconcat(
      [llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr
````
- **L2945 EN**: Comment explains nearby logic, invariants, or intent: `Allowed values for cta_group are {0,1,2} i.e [0, 3).`.
  **L2945 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allowed values for cta_group are {0,1,2} i.e [0, 3).`。
- **L2946 EN**: Executes a standalone statement or declaration: `Range<ArgIndex<cta_group_idx>, 0, 3>];`.
  **L2946 CN**: 执行一条独立语句或声明：`Range<ArgIndex<cta_group_idx>, 0, 3>];`。
- **L2947 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_`.
  **L2947 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_`。
- **L2948 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsicFlags<[], g2s_params, g2s_flags, g2s_props>;`.
  **L2948 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsicFlags<[], g2s_params, g2s_flags, g2s_props>;`。
- **L2949 EN**: Blank line separating nearby declarations or logic blocks.
  **L2949 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2950 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_prefetch_`.
  **L2950 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_prefetch_`。
- **L2951 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsicFlags<[],`.
  **L2951 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsicFlags<[],`。
- **L2952 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2952 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2953 EN**: Continues the surrounding expression or declaration: `tensor_dim_args,     // actual tensor dims`.
  **L2953 CN**: 继续构造周围的表达式或声明：`tensor_dim_args,     // actual tensor dims`。
- **L2954 EN**: Continues the surrounding expression or declaration: `im2col_offsets_args, // im2col offsets`.
  **L2954 CN**: 继续构造周围的表达式或声明：`im2col_offsets_args, // im2col offsets`。
- **L2955 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty]),       // cache_hint`.
  **L2955 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty]),       // cache_hint`。
- **L2956 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],                     // Flag for cache_hint`.
  **L2956 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],                     // Flag for cache_hint`。
- **L2957 EN**: Executes a standalone statement or declaration: `[IntrConvergent, ReadOnly<ArgIndex<0>>]>;`.
  **L2957 CN**: 执行一条独立语句或声明：`[IntrConvergent, ReadOnly<ArgIndex<0>>]>;`。
- **L2958 EN**: Blank line separating nearby declarations or logic blocks.
  **L2958 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2959 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_cta_`.
  **L2959 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_cta_`。
- **L2960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsicFlags<[],`.
  **L2960 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsicFlags<[],`。
- **L2961 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2961 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2962 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty,  // mbarrier_ptr`.
  **L2962 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty,  // mbarrier_ptr`。
- **L2963 EN**: Continues the surrounding expression or declaration: `llvm_ptr_ty],        // tensormap_ptr`.
  **L2963 CN**: 继续构造周围的表达式或声明：`llvm_ptr_ty],        // tensormap_ptr`。
- **L2964 EN**: Continues the surrounding expression or declaration: `tensor_dim_args,     // actual tensor dims`.
  **L2964 CN**: 继续构造周围的表达式或声明：`tensor_dim_args,     // actual tensor dims`。
- **L2965 EN**: Continues the surrounding expression or declaration: `im2col_offsets_args, // im2col offsets`.
  **L2965 CN**: 继续构造周围的表达式或声明：`im2col_offsets_args, // im2col offsets`。
- **L2966 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty]),      // cache_hint`.
  **L2966 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty]),      // cache_hint`。
- **L2967 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],                     // Flag for cache_hint`.
  **L2967 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],                     // Flag for cache_hint`。
- **L2968 EN**: Executes a standalone statement or declaration: `[IntrConvergent, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`.
  **L2968 CN**: 执行一条独立语句或声明：`[IntrConvergent, WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`。
- **L2969 EN**: Closes the current lexical scope or compound statement.
  **L2969 CN**: 结束当前词法作用域或复合语句块。
- **L2970 EN**: Closes the current lexical scope or compound statement.
  **L2970 CN**: 结束当前词法作用域或复合语句块。
- **L2971 EN**: Blank line separating nearby declarations or logic blocks.
  **L2971 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2972 EN**: Comment explains nearby logic, invariants, or intent: `TMA copy for tile::gather4`.
  **L2972 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TMA copy for tile::gather4`。
- **L2973 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_tile_gather4_2d`.
  **L2973 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_tile_gather4_2d`。
- **L2974 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L2974 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L2975 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2975 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2976 EN**: Continues the surrounding expression or declaration: `[llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr`.
  **L2976 CN**: 继续构造周围的表达式或声明：`[llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr`。

### Lines 2977-3008

````tablegen
       llvm_shared_ptr_ty,         // mbarrier_ptr
       llvm_ptr_ty],               // tensormap_ptr
       !listsplat(llvm_i32_ty, 5), // co-ordinates
      [llvm_i16_ty,                // cta_mask
       llvm_i64_ty]),              // cache_hint
      [llvm_i1_ty,                 // Flag for cta_mask
       llvm_i1_ty,                 // Flag for cache_hint
       llvm_i32_ty],               // Flag for cta_group
      [IntrConvergent,
       WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>,
       // Allowed values for cta_group are {0,1,2} i.e [0, 3).
       Range<ArgIndex<12>, 0, 3>]>;

def int_nvvm_cp_async_bulk_tensor_g2s_cta_tile_gather4_2d
  : DefaultAttrsIntrinsicFlags<[],
      !listconcat(
      [llvm_shared_ptr_ty,         // dst_shared_ptr
       llvm_shared_ptr_ty,         // mbarrier_ptr
       llvm_ptr_ty],               // tensormap_ptr
       !listsplat(llvm_i32_ty, 5), // co-ordinates
      [llvm_i64_ty]),              // cache_hint
      [llvm_i1_ty],                // Flag for cache_hint
      [IntrConvergent,
       WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;

// TMA prefetch for tile::gather4
def int_nvvm_cp_async_bulk_tensor_prefetch_tile_gather4_2d
  : DefaultAttrsIntrinsicFlags<[],
      !listconcat([llvm_ptr_ty],               // tensormap_ptr
                  !listsplat(llvm_i32_ty, 5),  // co-ordinates
                  [llvm_i64_ty]),              // cache_hint
      [llvm_i1_ty],                            // Flag for cache_hint
````
- **L2977 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty,         // mbarrier_ptr`.
  **L2977 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty,         // mbarrier_ptr`。
- **L2978 EN**: Continues the surrounding expression or declaration: `llvm_ptr_ty],               // tensormap_ptr`.
  **L2978 CN**: 继续构造周围的表达式或声明：`llvm_ptr_ty],               // tensormap_ptr`。
- **L2979 EN**: Continues logic associated with callable symbol `listsplat`.
  **L2979 CN**: 继续与可调用符号 `listsplat` 相关的逻辑。
- **L2980 EN**: Continues the surrounding expression or declaration: `[llvm_i16_ty,                // cta_mask`.
  **L2980 CN**: 继续构造周围的表达式或声明：`[llvm_i16_ty,                // cta_mask`。
- **L2981 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty]),              // cache_hint`.
  **L2981 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty]),              // cache_hint`。
- **L2982 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty,                 // Flag for cta_mask`.
  **L2982 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty,                 // Flag for cta_mask`。
- **L2983 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,                 // Flag for cache_hint`.
  **L2983 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,                 // Flag for cache_hint`。
- **L2984 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],               // Flag for cta_group`.
  **L2984 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],               // Flag for cta_group`。
- **L2985 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent,`.
  **L2985 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent,`。
- **L2986 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>,`.
  **L2986 CN**: 继续一个多行参数列表、初始化器或聚合项：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>,`。
- **L2987 EN**: Comment explains nearby logic, invariants, or intent: `Allowed values for cta_group are {0,1,2} i.e [0, 3).`.
  **L2987 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Allowed values for cta_group are {0,1,2} i.e [0, 3).`。
- **L2988 EN**: Executes a standalone statement or declaration: `Range<ArgIndex<12>, 0, 3>]>;`.
  **L2988 CN**: 执行一条独立语句或声明：`Range<ArgIndex<12>, 0, 3>]>;`。
- **L2989 EN**: Blank line separating nearby declarations or logic blocks.
  **L2989 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L2990 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_cta_tile_gather4_2d`.
  **L2990 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_g2s_cta_tile_gather4_2d`。
- **L2991 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L2991 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L2992 EN**: Continues logic associated with callable symbol `listconcat`.
  **L2992 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L2993 EN**: Continues the surrounding expression or declaration: `[llvm_shared_ptr_ty,         // dst_shared_ptr`.
  **L2993 CN**: 继续构造周围的表达式或声明：`[llvm_shared_ptr_ty,         // dst_shared_ptr`。
- **L2994 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty,         // mbarrier_ptr`.
  **L2994 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty,         // mbarrier_ptr`。
- **L2995 EN**: Continues the surrounding expression or declaration: `llvm_ptr_ty],               // tensormap_ptr`.
  **L2995 CN**: 继续构造周围的表达式或声明：`llvm_ptr_ty],               // tensormap_ptr`。
- **L2996 EN**: Continues logic associated with callable symbol `listsplat`.
  **L2996 CN**: 继续与可调用符号 `listsplat` 相关的逻辑。
- **L2997 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty]),              // cache_hint`.
  **L2997 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty]),              // cache_hint`。
- **L2998 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],                // Flag for cache_hint`.
  **L2998 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],                // Flag for cache_hint`。
- **L2999 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent,`.
  **L2999 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent,`。
- **L3000 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`.
  **L3000 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`。
- **L3001 EN**: Blank line separating nearby declarations or logic blocks.
  **L3001 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3002 EN**: Comment explains nearby logic, invariants, or intent: `TMA prefetch for tile::gather4`.
  **L3002 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`TMA prefetch for tile::gather4`。
- **L3003 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_tensor_prefetch_tile_gather4_2d`.
  **L3003 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_tensor_prefetch_tile_gather4_2d`。
- **L3004 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L3004 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L3005 EN**: Continues logic associated with callable symbol `listconcat`.
  **L3005 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L3006 EN**: Continues logic associated with callable symbol `listsplat`.
  **L3006 CN**: 继续与可调用符号 `listsplat` 相关的逻辑。
- **L3007 EN**: Continues the surrounding expression or declaration: `[llvm_i64_ty]),              // cache_hint`.
  **L3007 CN**: 继续构造周围的表达式或声明：`[llvm_i64_ty]),              // cache_hint`。
- **L3008 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],                            // Flag for cache_hint`.
  **L3008 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],                            // Flag for cache_hint`。

### Lines 3009-3040

````tablegen
      [IntrConvergent, ReadOnly<ArgIndex<0>>]>;

// Intrinsics for Prefetch and Prefetchu
let IntrProperties = [IntrArgMemOnly, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>] in {
  foreach level = ["L1", "L2"] in {
    def int_nvvm_prefetch_ # level : DefaultAttrsIntrinsic<[], [llvm_ptr_ty]>;
    def int_nvvm_prefetch_global_ # level : DefaultAttrsIntrinsic<[], [llvm_global_ptr_ty]>;
    def int_nvvm_prefetch_local_ # level : DefaultAttrsIntrinsic<[], [llvm_local_ptr_ty]>;
  }

  def int_nvvm_prefetch_tensormap : DefaultAttrsIntrinsic<[], [llvm_anyptr_ty]>;

  foreach eviction_priority = ["evict_normal", "evict_last"] in
    def int_nvvm_prefetch_global_L2_ # eviction_priority : DefaultAttrsIntrinsic<[], [llvm_global_ptr_ty]>;

  def int_nvvm_prefetchu_L1 : DefaultAttrsIntrinsic<[], [llvm_ptr_ty]>;
}

// applypriority
let IntrProperties = [IntrArgMemOnly, ReadOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>,
                      ImmArg<ArgIndex<1>>] in {
  def int_nvvm_applypriority_global_L2_evict_normal
    : DefaultAttrsIntrinsic<[], [llvm_global_ptr_ty, llvm_i64_ty]>;

  def int_nvvm_applypriority_L2_evict_normal
    : DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty]>;
}

// discard
let IntrProperties = [NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<1>>, IntrHasSideEffects] in {
  def int_nvvm_discard_global_L2 : DefaultAttrsIntrinsic<[], [llvm_global_ptr_ty, llvm_i64_ty]>;
  def int_nvvm_discard_L2 : DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty]>;
````
- **L3009 EN**: Executes a standalone statement or declaration: `[IntrConvergent, ReadOnly<ArgIndex<0>>]>;`.
  **L3009 CN**: 执行一条独立语句或声明：`[IntrConvergent, ReadOnly<ArgIndex<0>>]>;`。
- **L3010 EN**: Blank line separating nearby declarations or logic blocks.
  **L3010 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3011 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsics for Prefetch and Prefetchu`.
  **L3011 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsics for Prefetch and Prefetchu`。
- **L3012 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3012 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3013 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3013 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3014 EN**: Declares TableGen def `int_nvvm_prefetch_`.
  **L3014 CN**: 声明 TableGen def `int_nvvm_prefetch_`。
- **L3015 EN**: Declares TableGen def `int_nvvm_prefetch_global_`.
  **L3015 CN**: 声明 TableGen def `int_nvvm_prefetch_global_`。
- **L3016 EN**: Declares TableGen def `int_nvvm_prefetch_local_`.
  **L3016 CN**: 声明 TableGen def `int_nvvm_prefetch_local_`。
- **L3017 EN**: Closes the current lexical scope or compound statement.
  **L3017 CN**: 结束当前词法作用域或复合语句块。
- **L3018 EN**: Blank line separating nearby declarations or logic blocks.
  **L3018 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3019 EN**: Declares TableGen def `int_nvvm_prefetch_tensormap`.
  **L3019 CN**: 声明 TableGen def `int_nvvm_prefetch_tensormap`。
- **L3020 EN**: Blank line separating nearby declarations or logic blocks.
  **L3020 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3021 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3021 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3022 EN**: Declares TableGen def `int_nvvm_prefetch_global_L2_`.
  **L3022 CN**: 声明 TableGen def `int_nvvm_prefetch_global_L2_`。
- **L3023 EN**: Blank line separating nearby declarations or logic blocks.
  **L3023 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3024 EN**: Declares TableGen def `int_nvvm_prefetchu_L1`.
  **L3024 CN**: 声明 TableGen def `int_nvvm_prefetchu_L1`。
- **L3025 EN**: Closes the current lexical scope or compound statement.
  **L3025 CN**: 结束当前词法作用域或复合语句块。
- **L3026 EN**: Blank line separating nearby declarations or logic blocks.
  **L3026 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3027 EN**: Comment explains nearby logic, invariants, or intent: `applypriority`.
  **L3027 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`applypriority`。
- **L3028 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3028 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3029 EN**: Continues the surrounding expression or declaration: `ImmArg<ArgIndex<1>>] in {`.
  **L3029 CN**: 继续构造周围的表达式或声明：`ImmArg<ArgIndex<1>>] in {`。
- **L3030 EN**: Declares TableGen def `int_nvvm_applypriority_global_L2_evict_normal`.
  **L3030 CN**: 声明 TableGen def `int_nvvm_applypriority_global_L2_evict_normal`。
- **L3031 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [llvm_global_ptr_ty, llvm_i64_ty]>;`.
  **L3031 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [llvm_global_ptr_ty, llvm_i64_ty]>;`。
- **L3032 EN**: Blank line separating nearby declarations or logic blocks.
  **L3032 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3033 EN**: Declares TableGen def `int_nvvm_applypriority_L2_evict_normal`.
  **L3033 CN**: 声明 TableGen def `int_nvvm_applypriority_L2_evict_normal`。
- **L3034 EN**: Executes a standalone statement or declaration: `: DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty]>;`.
  **L3034 CN**: 执行一条独立语句或声明：`: DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty]>;`。
- **L3035 EN**: Closes the current lexical scope or compound statement.
  **L3035 CN**: 结束当前词法作用域或复合语句块。
- **L3036 EN**: Blank line separating nearby declarations or logic blocks.
  **L3036 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3037 EN**: Comment explains nearby logic, invariants, or intent: `discard`.
  **L3037 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`discard`。
- **L3038 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3038 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3039 EN**: Declares TableGen def `int_nvvm_discard_global_L2`.
  **L3039 CN**: 声明 TableGen def `int_nvvm_discard_global_L2`。
- **L3040 EN**: Declares TableGen def `int_nvvm_discard_L2`.
  **L3040 CN**: 声明 TableGen def `int_nvvm_discard_L2`。

### Lines 3041-3072

````tablegen
}

// Intrinsics for Bulk Copy using TMA (non-tensor)
// From Global to Shared Cluster
def int_nvvm_cp_async_bulk_global_to_shared_cluster
  : DefaultAttrsIntrinsicFlags<[],
      [llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr
       llvm_shared_ptr_ty,         // mbarrier_ptr
       llvm_global_ptr_ty,         // src_gmem_ptr
       llvm_i32_ty,                // copy_size
       llvm_i16_ty,                // cta_mask
       llvm_i64_ty],               // cache_hint
      [llvm_i1_ty,                 // Flag for cta_mask
       llvm_i1_ty],                // Flag for cache_hint
      [IntrConvergent, IntrArgMemOnly,
       WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;

// From Global to Shared CTA
def int_nvvm_cp_async_bulk_global_to_shared_cta
  : DefaultAttrsIntrinsicFlags<[],
      [llvm_shared_ptr_ty, // dst_shared_cta_ptr
       llvm_shared_ptr_ty, // mbarrier_ptr
       llvm_global_ptr_ty, // src_gmem_ptr
       llvm_i32_ty,        // copy_size
       llvm_i64_ty],       // cache_hint
      [llvm_i1_ty],        // Flag for cache_hint
      [IntrConvergent, IntrArgMemOnly,
       WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;

// From Shared CTA to Shared Cluster
def int_nvvm_cp_async_bulk_shared_cta_to_cluster
  : DefaultAttrsIntrinsic<[],
````
- **L3041 EN**: Closes the current lexical scope or compound statement.
  **L3041 CN**: 结束当前词法作用域或复合语句块。
- **L3042 EN**: Blank line separating nearby declarations or logic blocks.
  **L3042 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3043 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsics for Bulk Copy using TMA (non-tensor)`.
  **L3043 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsics for Bulk Copy using TMA (non-tensor)`。
- **L3044 EN**: Comment explains nearby logic, invariants, or intent: `From Global to Shared Cluster`.
  **L3044 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`From Global to Shared Cluster`。
- **L3045 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_global_to_shared_cluster`.
  **L3045 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_global_to_shared_cluster`。
- **L3046 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L3046 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L3047 EN**: Continues the surrounding expression or declaration: `[llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr`.
  **L3047 CN**: 继续构造周围的表达式或声明：`[llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr`。
- **L3048 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty,         // mbarrier_ptr`.
  **L3048 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty,         // mbarrier_ptr`。
- **L3049 EN**: Continues the surrounding expression or declaration: `llvm_global_ptr_ty,         // src_gmem_ptr`.
  **L3049 CN**: 继续构造周围的表达式或声明：`llvm_global_ptr_ty,         // src_gmem_ptr`。
- **L3050 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,                // copy_size`.
  **L3050 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,                // copy_size`。
- **L3051 EN**: Continues the surrounding expression or declaration: `llvm_i16_ty,                // cta_mask`.
  **L3051 CN**: 继续构造周围的表达式或声明：`llvm_i16_ty,                // cta_mask`。
- **L3052 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty],               // cache_hint`.
  **L3052 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty],               // cache_hint`。
- **L3053 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty,                 // Flag for cta_mask`.
  **L3053 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty,                 // Flag for cta_mask`。
- **L3054 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty],                // Flag for cache_hint`.
  **L3054 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty],                // Flag for cache_hint`。
- **L3055 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3055 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3056 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`.
  **L3056 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`。
- **L3057 EN**: Blank line separating nearby declarations or logic blocks.
  **L3057 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3058 EN**: Comment explains nearby logic, invariants, or intent: `From Global to Shared CTA`.
  **L3058 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`From Global to Shared CTA`。
- **L3059 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_global_to_shared_cta`.
  **L3059 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_global_to_shared_cta`。
- **L3060 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L3060 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L3061 EN**: Continues the surrounding expression or declaration: `[llvm_shared_ptr_ty, // dst_shared_cta_ptr`.
  **L3061 CN**: 继续构造周围的表达式或声明：`[llvm_shared_ptr_ty, // dst_shared_cta_ptr`。
- **L3062 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty, // mbarrier_ptr`.
  **L3062 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty, // mbarrier_ptr`。
- **L3063 EN**: Continues the surrounding expression or declaration: `llvm_global_ptr_ty, // src_gmem_ptr`.
  **L3063 CN**: 继续构造周围的表达式或声明：`llvm_global_ptr_ty, // src_gmem_ptr`。
- **L3064 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,        // copy_size`.
  **L3064 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,        // copy_size`。
- **L3065 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty],       // cache_hint`.
  **L3065 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty],       // cache_hint`。
- **L3066 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],        // Flag for cache_hint`.
  **L3066 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],        // Flag for cache_hint`。
- **L3067 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3067 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3068 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`.
  **L3068 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`。
- **L3069 EN**: Blank line separating nearby declarations or logic blocks.
  **L3069 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3070 EN**: Comment explains nearby logic, invariants, or intent: `From Shared CTA to Shared Cluster`.
  **L3070 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`From Shared CTA to Shared Cluster`。
- **L3071 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_shared_cta_to_cluster`.
  **L3071 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_shared_cta_to_cluster`。
- **L3072 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3072 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。

### Lines 3073-3104

````tablegen
      [llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr
       llvm_shared_ptr_ty,         // mbarrier_ptr
       llvm_shared_ptr_ty,         // src_smem_ptr
       llvm_i32_ty],               // copy_size
      [IntrConvergent, IntrArgMemOnly,
       WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;

// From Shared CTA to Global memory
def int_nvvm_cp_async_bulk_shared_cta_to_global
  : DefaultAttrsIntrinsicFlags<[],
      [llvm_global_ptr_ty, // dst_gmem_ptr
       llvm_shared_ptr_ty, // src_smem_ptr
       llvm_i32_ty,        // copy_size
       llvm_i64_ty],       // cache_hint
      [llvm_i1_ty],        // Flag for cache_hint
      [IntrConvergent, IntrArgMemOnly,
       WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;

// From Shared CTA to Global memory with bytemask
def int_nvvm_cp_async_bulk_shared_cta_to_global_bytemask
  : DefaultAttrsIntrinsic<[],
      [llvm_global_ptr_ty, // dst_gmem_ptr
       llvm_shared_ptr_ty, // src_smem_ptr
       llvm_i32_ty,        // copy_size
       llvm_i64_ty,        // cache_hint
       llvm_i1_ty,         // Flag for cache_hint
       llvm_i16_ty],       // byte_mask
      [IntrConvergent, IntrArgMemOnly,
       WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>,
       ImmArg<ArgIndex<4>>]>;

// Intrinsics for Bulk Copy Prefetch L2
````
- **L3073 EN**: Continues the surrounding expression or declaration: `[llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr`.
  **L3073 CN**: 继续构造周围的表达式或声明：`[llvm_shared_cluster_ptr_ty, // dst_shared_cluster_ptr`。
- **L3074 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty,         // mbarrier_ptr`.
  **L3074 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty,         // mbarrier_ptr`。
- **L3075 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty,         // src_smem_ptr`.
  **L3075 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty,         // src_smem_ptr`。
- **L3076 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],               // copy_size`.
  **L3076 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],               // copy_size`。
- **L3077 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3077 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3078 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`.
  **L3078 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<2>>]>;`。
- **L3079 EN**: Blank line separating nearby declarations or logic blocks.
  **L3079 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3080 EN**: Comment explains nearby logic, invariants, or intent: `From Shared CTA to Global memory`.
  **L3080 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`From Shared CTA to Global memory`。
- **L3081 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_shared_cta_to_global`.
  **L3081 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_shared_cta_to_global`。
- **L3082 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L3082 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L3083 EN**: Continues the surrounding expression or declaration: `[llvm_global_ptr_ty, // dst_gmem_ptr`.
  **L3083 CN**: 继续构造周围的表达式或声明：`[llvm_global_ptr_ty, // dst_gmem_ptr`。
- **L3084 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty, // src_smem_ptr`.
  **L3084 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty, // src_smem_ptr`。
- **L3085 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,        // copy_size`.
  **L3085 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,        // copy_size`。
- **L3086 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty],       // cache_hint`.
  **L3086 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty],       // cache_hint`。
- **L3087 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],        // Flag for cache_hint`.
  **L3087 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],        // Flag for cache_hint`。
- **L3088 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3088 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3089 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`.
  **L3089 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>]>;`。
- **L3090 EN**: Blank line separating nearby declarations or logic blocks.
  **L3090 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3091 EN**: Comment explains nearby logic, invariants, or intent: `From Shared CTA to Global memory with bytemask`.
  **L3091 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`From Shared CTA to Global memory with bytemask`。
- **L3092 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_shared_cta_to_global_bytemask`.
  **L3092 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_shared_cta_to_global_bytemask`。
- **L3093 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[],`.
  **L3093 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[],`。
- **L3094 EN**: Continues the surrounding expression or declaration: `[llvm_global_ptr_ty, // dst_gmem_ptr`.
  **L3094 CN**: 继续构造周围的表达式或声明：`[llvm_global_ptr_ty, // dst_gmem_ptr`。
- **L3095 EN**: Continues the surrounding expression or declaration: `llvm_shared_ptr_ty, // src_smem_ptr`.
  **L3095 CN**: 继续构造周围的表达式或声明：`llvm_shared_ptr_ty, // src_smem_ptr`。
- **L3096 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,        // copy_size`.
  **L3096 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,        // copy_size`。
- **L3097 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty,        // cache_hint`.
  **L3097 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty,        // cache_hint`。
- **L3098 EN**: Continues the surrounding expression or declaration: `llvm_i1_ty,         // Flag for cache_hint`.
  **L3098 CN**: 继续构造周围的表达式或声明：`llvm_i1_ty,         // Flag for cache_hint`。
- **L3099 EN**: Continues the surrounding expression or declaration: `llvm_i16_ty],       // byte_mask`.
  **L3099 CN**: 继续构造周围的表达式或声明：`llvm_i16_ty],       // byte_mask`。
- **L3100 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3100 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3101 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>,`.
  **L3101 CN**: 继续一个多行参数列表、初始化器或聚合项：`WriteOnly<ArgIndex<0>>, ReadOnly<ArgIndex<1>>,`。
- **L3102 EN**: Executes a standalone statement or declaration: `ImmArg<ArgIndex<4>>]>;`.
  **L3102 CN**: 执行一条独立语句或声明：`ImmArg<ArgIndex<4>>]>;`。
- **L3103 EN**: Blank line separating nearby declarations or logic blocks.
  **L3103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3104 EN**: Comment explains nearby logic, invariants, or intent: `Intrinsics for Bulk Copy Prefetch L2`.
  **L3104 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Intrinsics for Bulk Copy Prefetch L2`。

### Lines 3105-3136

````tablegen
def int_nvvm_cp_async_bulk_prefetch_L2
  : DefaultAttrsIntrinsicFlags<[],
      [llvm_global_ptr_ty, // src_gmem_ptr
       llvm_i32_ty,        // copy_size
       llvm_i64_ty],       // cache_hint
      [llvm_i1_ty],        // Flag for cache_hint
      [IntrConvergent, IntrArgMemOnly,
       NoCapture<ArgIndex<0>>, ReadOnly<ArgIndex<0>>]>;

def int_nvvm_griddepcontrol_launch_dependents : Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;
def int_nvvm_griddepcontrol_wait : Intrinsic<[], [], [IntrNoMem, IntrHasSideEffects]>;

//
// Tcgen05 family of Intrinsics
//

// Tcgen05 alloc/dealloc related intrinsics

foreach cta_group = ["cg1", "cg2"] in {
  def int_nvvm_tcgen05_alloc_ # cta_group : Intrinsic<[],
    [llvm_ptr_ty,        // dst_ptr
     llvm_i32_ty] ,      // num_columns
    [IntrConvergent, IntrInaccessibleMemOrArgMemOnly,
     WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;

  def int_nvvm_tcgen05_alloc_shared_ # cta_group : Intrinsic<[],
    [llvm_shared_ptr_ty, // dst_ptr
     llvm_i32_ty],       // num_columns
    [IntrConvergent, IntrInaccessibleMemOrArgMemOnly,
     WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;

  def int_nvvm_tcgen05_dealloc_ # cta_group : Intrinsic<[],
````
- **L3105 EN**: Declares TableGen def `int_nvvm_cp_async_bulk_prefetch_L2`.
  **L3105 CN**: 声明 TableGen def `int_nvvm_cp_async_bulk_prefetch_L2`。
- **L3106 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsicFlags<[],`.
  **L3106 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsicFlags<[],`。
- **L3107 EN**: Continues the surrounding expression or declaration: `[llvm_global_ptr_ty, // src_gmem_ptr`.
  **L3107 CN**: 继续构造周围的表达式或声明：`[llvm_global_ptr_ty, // src_gmem_ptr`。
- **L3108 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,        // copy_size`.
  **L3108 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,        // copy_size`。
- **L3109 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty],       // cache_hint`.
  **L3109 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty],       // cache_hint`。
- **L3110 EN**: Continues the surrounding expression or declaration: `[llvm_i1_ty],        // Flag for cache_hint`.
  **L3110 CN**: 继续构造周围的表达式或声明：`[llvm_i1_ty],        // Flag for cache_hint`。
- **L3111 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3111 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3112 EN**: Executes a standalone statement or declaration: `NoCapture<ArgIndex<0>>, ReadOnly<ArgIndex<0>>]>;`.
  **L3112 CN**: 执行一条独立语句或声明：`NoCapture<ArgIndex<0>>, ReadOnly<ArgIndex<0>>]>;`。
- **L3113 EN**: Blank line separating nearby declarations or logic blocks.
  **L3113 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3114 EN**: Declares TableGen def `int_nvvm_griddepcontrol_launch_dependents`.
  **L3114 CN**: 声明 TableGen def `int_nvvm_griddepcontrol_launch_dependents`。
- **L3115 EN**: Declares TableGen def `int_nvvm_griddepcontrol_wait`.
  **L3115 CN**: 声明 TableGen def `int_nvvm_griddepcontrol_wait`。
- **L3116 EN**: Blank line separating nearby declarations or logic blocks.
  **L3116 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3117 EN**: Separator comment used for visual grouping.
  **L3117 CN**: 用于视觉分组的分隔注释。
- **L3118 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 family of Intrinsics`.
  **L3118 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 family of Intrinsics`。
- **L3119 EN**: Separator comment used for visual grouping.
  **L3119 CN**: 用于视觉分组的分隔注释。
- **L3120 EN**: Blank line separating nearby declarations or logic blocks.
  **L3120 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3121 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 alloc/dealloc related intrinsics`.
  **L3121 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 alloc/dealloc related intrinsics`。
- **L3122 EN**: Blank line separating nearby declarations or logic blocks.
  **L3122 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3123 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3123 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3124 EN**: Declares TableGen def `int_nvvm_tcgen05_alloc_`.
  **L3124 CN**: 声明 TableGen def `int_nvvm_tcgen05_alloc_`。
- **L3125 EN**: Continues the surrounding expression or declaration: `[llvm_ptr_ty,        // dst_ptr`.
  **L3125 CN**: 继续构造周围的表达式或声明：`[llvm_ptr_ty,        // dst_ptr`。
- **L3126 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty] ,      // num_columns`.
  **L3126 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty] ,      // num_columns`。
- **L3127 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`.
  **L3127 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`。
- **L3128 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;`.
  **L3128 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;`。
- **L3129 EN**: Blank line separating nearby declarations or logic blocks.
  **L3129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3130 EN**: Declares TableGen def `int_nvvm_tcgen05_alloc_shared_`.
  **L3130 CN**: 声明 TableGen def `int_nvvm_tcgen05_alloc_shared_`。
- **L3131 EN**: Continues the surrounding expression or declaration: `[llvm_shared_ptr_ty, // dst_ptr`.
  **L3131 CN**: 继续构造周围的表达式或声明：`[llvm_shared_ptr_ty, // dst_ptr`。
- **L3132 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],       // num_columns`.
  **L3132 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],       // num_columns`。
- **L3133 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`.
  **L3133 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`。
- **L3134 EN**: Executes a standalone statement or declaration: `WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;`.
  **L3134 CN**: 执行一条独立语句或声明：`WriteOnly<ArgIndex<0>>, NoCapture<ArgIndex<0>>]>;`。
- **L3135 EN**: Blank line separating nearby declarations or logic blocks.
  **L3135 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3136 EN**: Declares TableGen def `int_nvvm_tcgen05_dealloc_`.
  **L3136 CN**: 声明 TableGen def `int_nvvm_tcgen05_dealloc_`。

### Lines 3137-3168

````tablegen
    [llvm_tmem_ptr_ty,   // tmem_addr
     llvm_i32_ty],       // num_columns
    [IntrConvergent, IntrArgMemOnly,
     NoCapture<ArgIndex<0>>]>;

  def int_nvvm_tcgen05_relinq_alloc_permit_ # cta_group : Intrinsic<[], [],
    [IntrConvergent, IntrInaccessibleMemOnly]>;

  def int_nvvm_tcgen05_commit_ # cta_group : Intrinsic<[],
    [llvm_ptr_ty],        // mbar_ptr
    [IntrConvergent, IntrInaccessibleMemOrArgMemOnly,
     NoCapture<ArgIndex<0>>]>;

  def int_nvvm_tcgen05_commit_shared_ # cta_group : Intrinsic<[],
    [llvm_shared_ptr_ty], // mbar_ptr
    [IntrConvergent, IntrInaccessibleMemOrArgMemOnly,
     NoCapture<ArgIndex<0>>]>;

  def int_nvvm_tcgen05_commit_mc_ # cta_group : Intrinsic<[],
    [llvm_ptr_ty, llvm_i16_ty], // mbar_ptr, cta_mask
    [IntrConvergent, IntrInaccessibleMemOrArgMemOnly,
     NoCapture<ArgIndex<0>>]>;

  def int_nvvm_tcgen05_commit_mc_shared_ # cta_group : Intrinsic<[],
    [llvm_shared_ptr_ty, llvm_i16_ty], // mbar_ptr, cta_mask
    [IntrConvergent, IntrInaccessibleMemOrArgMemOnly,
     NoCapture<ArgIndex<0>>]>;

  def int_nvvm_tcgen05_shift_down_ # cta_group : Intrinsic<[],
    [llvm_tmem_ptr_ty],   // tmem_addr
    [IntrConvergent, IntrArgMemOnly,
     NoCapture<ArgIndex<0>>]>;
````
- **L3137 EN**: Continues the surrounding expression or declaration: `[llvm_tmem_ptr_ty,   // tmem_addr`.
  **L3137 CN**: 继续构造周围的表达式或声明：`[llvm_tmem_ptr_ty,   // tmem_addr`。
- **L3138 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty],       // num_columns`.
  **L3138 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty],       // num_columns`。
- **L3139 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3139 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3140 EN**: Executes a standalone statement or declaration: `NoCapture<ArgIndex<0>>]>;`.
  **L3140 CN**: 执行一条独立语句或声明：`NoCapture<ArgIndex<0>>]>;`。
- **L3141 EN**: Blank line separating nearby declarations or logic blocks.
  **L3141 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3142 EN**: Declares TableGen def `int_nvvm_tcgen05_relinq_alloc_permit_`.
  **L3142 CN**: 声明 TableGen def `int_nvvm_tcgen05_relinq_alloc_permit_`。
- **L3143 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrInaccessibleMemOnly]>;`.
  **L3143 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrInaccessibleMemOnly]>;`。
- **L3144 EN**: Blank line separating nearby declarations or logic blocks.
  **L3144 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3145 EN**: Declares TableGen def `int_nvvm_tcgen05_commit_`.
  **L3145 CN**: 声明 TableGen def `int_nvvm_tcgen05_commit_`。
- **L3146 EN**: Continues the surrounding expression or declaration: `[llvm_ptr_ty],        // mbar_ptr`.
  **L3146 CN**: 继续构造周围的表达式或声明：`[llvm_ptr_ty],        // mbar_ptr`。
- **L3147 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`.
  **L3147 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`。
- **L3148 EN**: Executes a standalone statement or declaration: `NoCapture<ArgIndex<0>>]>;`.
  **L3148 CN**: 执行一条独立语句或声明：`NoCapture<ArgIndex<0>>]>;`。
- **L3149 EN**: Blank line separating nearby declarations or logic blocks.
  **L3149 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3150 EN**: Declares TableGen def `int_nvvm_tcgen05_commit_shared_`.
  **L3150 CN**: 声明 TableGen def `int_nvvm_tcgen05_commit_shared_`。
- **L3151 EN**: Continues the surrounding expression or declaration: `[llvm_shared_ptr_ty], // mbar_ptr`.
  **L3151 CN**: 继续构造周围的表达式或声明：`[llvm_shared_ptr_ty], // mbar_ptr`。
- **L3152 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`.
  **L3152 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`。
- **L3153 EN**: Executes a standalone statement or declaration: `NoCapture<ArgIndex<0>>]>;`.
  **L3153 CN**: 执行一条独立语句或声明：`NoCapture<ArgIndex<0>>]>;`。
- **L3154 EN**: Blank line separating nearby declarations or logic blocks.
  **L3154 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3155 EN**: Declares TableGen def `int_nvvm_tcgen05_commit_mc_`.
  **L3155 CN**: 声明 TableGen def `int_nvvm_tcgen05_commit_mc_`。
- **L3156 EN**: Continues the surrounding expression or declaration: `[llvm_ptr_ty, llvm_i16_ty], // mbar_ptr, cta_mask`.
  **L3156 CN**: 继续构造周围的表达式或声明：`[llvm_ptr_ty, llvm_i16_ty], // mbar_ptr, cta_mask`。
- **L3157 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`.
  **L3157 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`。
- **L3158 EN**: Executes a standalone statement or declaration: `NoCapture<ArgIndex<0>>]>;`.
  **L3158 CN**: 执行一条独立语句或声明：`NoCapture<ArgIndex<0>>]>;`。
- **L3159 EN**: Blank line separating nearby declarations or logic blocks.
  **L3159 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3160 EN**: Declares TableGen def `int_nvvm_tcgen05_commit_mc_shared_`.
  **L3160 CN**: 声明 TableGen def `int_nvvm_tcgen05_commit_mc_shared_`。
- **L3161 EN**: Continues the surrounding expression or declaration: `[llvm_shared_ptr_ty, llvm_i16_ty], // mbar_ptr, cta_mask`.
  **L3161 CN**: 继续构造周围的表达式或声明：`[llvm_shared_ptr_ty, llvm_i16_ty], // mbar_ptr, cta_mask`。
- **L3162 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`.
  **L3162 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOrArgMemOnly,`。
- **L3163 EN**: Executes a standalone statement or declaration: `NoCapture<ArgIndex<0>>]>;`.
  **L3163 CN**: 执行一条独立语句或声明：`NoCapture<ArgIndex<0>>]>;`。
- **L3164 EN**: Blank line separating nearby declarations or logic blocks.
  **L3164 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3165 EN**: Declares TableGen def `int_nvvm_tcgen05_shift_down_`.
  **L3165 CN**: 声明 TableGen def `int_nvvm_tcgen05_shift_down_`。
- **L3166 EN**: Continues the surrounding expression or declaration: `[llvm_tmem_ptr_ty],   // tmem_addr`.
  **L3166 CN**: 继续构造周围的表达式或声明：`[llvm_tmem_ptr_ty],   // tmem_addr`。
- **L3167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrArgMemOnly,`.
  **L3167 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrArgMemOnly,`。
- **L3168 EN**: Executes a standalone statement or declaration: `NoCapture<ArgIndex<0>>]>;`.
  **L3168 CN**: 执行一条独立语句或声明：`NoCapture<ArgIndex<0>>]>;`。

### Lines 3169-3200

````tablegen
}

// Tcgen05 wait_ld/st intrinsics
def int_nvvm_tcgen05_wait_ld : Intrinsic<[], [],
  [IntrConvergent, IntrInaccessibleMemOnly]>;
def int_nvvm_tcgen05_wait_st : Intrinsic<[], [],
  [IntrConvergent, IntrInaccessibleMemOnly]>;

// Tcgen05 Fence intrinsics
def int_nvvm_tcgen05_fence_before_thread_sync : Intrinsic<[], [],
  [IntrNoMem, IntrHasSideEffects]>;
def int_nvvm_tcgen05_fence_after_thread_sync : Intrinsic<[], [],
  [IntrNoMem, IntrHasSideEffects]>;

// Tcgen05 cp intrinsics
foreach cta_group = ["cg1", "cg2"] in {
  foreach src_fmt = ["", "b6x16_p32", "b4x16_p64"] in {
    foreach shape = ["128x256b", "4x256b", "128x128b",
                     "64x128b_warpx2_02_13",
                     "64x128b_warpx2_01_23",
                     "32x128b_warpx4"] in {
      defvar name = "llvm.nvvm.tcgen05.cp."  #
                    StrJoin<".", [shape, src_fmt, cta_group]>.ret;

      defvar intrinsic_name = IntrinsicName<name>;
      def intrinsic_name.record_name : Intrinsic<[],
        [llvm_tmem_ptr_ty,   // tmem_addr
         llvm_i64_ty],       // smem descriptor
        [IntrConvergent, IntrInaccessibleMemOrArgMemOnly, NoCapture<ArgIndex<0>>],
        intrinsic_name.intr_name>;
    }
  }
````
- **L3169 EN**: Closes the current lexical scope or compound statement.
  **L3169 CN**: 结束当前词法作用域或复合语句块。
- **L3170 EN**: Blank line separating nearby declarations or logic blocks.
  **L3170 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3171 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 wait_ld/st intrinsics`.
  **L3171 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 wait_ld/st intrinsics`。
- **L3172 EN**: Declares TableGen def `int_nvvm_tcgen05_wait_ld`.
  **L3172 CN**: 声明 TableGen def `int_nvvm_tcgen05_wait_ld`。
- **L3173 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrInaccessibleMemOnly]>;`.
  **L3173 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrInaccessibleMemOnly]>;`。
- **L3174 EN**: Declares TableGen def `int_nvvm_tcgen05_wait_st`.
  **L3174 CN**: 声明 TableGen def `int_nvvm_tcgen05_wait_st`。
- **L3175 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrInaccessibleMemOnly]>;`.
  **L3175 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrInaccessibleMemOnly]>;`。
- **L3176 EN**: Blank line separating nearby declarations or logic blocks.
  **L3176 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3177 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 Fence intrinsics`.
  **L3177 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 Fence intrinsics`。
- **L3178 EN**: Declares TableGen def `int_nvvm_tcgen05_fence_before_thread_sync`.
  **L3178 CN**: 声明 TableGen def `int_nvvm_tcgen05_fence_before_thread_sync`。
- **L3179 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L3179 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L3180 EN**: Declares TableGen def `int_nvvm_tcgen05_fence_after_thread_sync`.
  **L3180 CN**: 声明 TableGen def `int_nvvm_tcgen05_fence_after_thread_sync`。
- **L3181 EN**: Executes a standalone statement or declaration: `[IntrNoMem, IntrHasSideEffects]>;`.
  **L3181 CN**: 执行一条独立语句或声明：`[IntrNoMem, IntrHasSideEffects]>;`。
- **L3182 EN**: Blank line separating nearby declarations or logic blocks.
  **L3182 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3183 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 cp intrinsics`.
  **L3183 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 cp intrinsics`。
- **L3184 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3184 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3185 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3185 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3186 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3186 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3187 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"64x128b_warpx2_02_13",`.
  **L3187 CN**: 继续一个多行参数列表、初始化器或聚合项：`"64x128b_warpx2_02_13",`。
- **L3188 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"64x128b_warpx2_01_23",`.
  **L3188 CN**: 继续一个多行参数列表、初始化器或聚合项：`"64x128b_warpx2_01_23",`。
- **L3189 EN**: Continues the surrounding expression or declaration: `"32x128b_warpx4"] in {`.
  **L3189 CN**: 继续构造周围的表达式或声明：`"32x128b_warpx4"] in {`。
- **L3190 EN**: Continues the surrounding expression or declaration: `defvar name = "llvm.nvvm.tcgen05.cp."  #`.
  **L3190 CN**: 继续构造周围的表达式或声明：`defvar name = "llvm.nvvm.tcgen05.cp."  #`。
- **L3191 EN**: Executes a standalone statement or declaration: `StrJoin<".", [shape, src_fmt, cta_group]>.ret;`.
  **L3191 CN**: 执行一条独立语句或声明：`StrJoin<".", [shape, src_fmt, cta_group]>.ret;`。
- **L3192 EN**: Blank line separating nearby declarations or logic blocks.
  **L3192 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3193 EN**: Initializes variable `intrinsic_name` from the right-hand expression.
  **L3193 CN**: 使用右侧表达式初始化变量 `intrinsic_name`。
- **L3194 EN**: Declares TableGen def `intrinsic_name.record_name`.
  **L3194 CN**: 声明 TableGen def `intrinsic_name.record_name`。
- **L3195 EN**: Continues the surrounding expression or declaration: `[llvm_tmem_ptr_ty,   // tmem_addr`.
  **L3195 CN**: 继续构造周围的表达式或声明：`[llvm_tmem_ptr_ty,   // tmem_addr`。
- **L3196 EN**: Continues the surrounding expression or declaration: `llvm_i64_ty],       // smem descriptor`.
  **L3196 CN**: 继续构造周围的表达式或声明：`llvm_i64_ty],       // smem descriptor`。
- **L3197 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrConvergent, IntrInaccessibleMemOrArgMemOnly, NoCapture<ArgIndex<0>>],`.
  **L3197 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrConvergent, IntrInaccessibleMemOrArgMemOnly, NoCapture<ArgIndex<0>>],`。
- **L3198 EN**: Executes a standalone statement or declaration: `intrinsic_name.intr_name>;`.
  **L3198 CN**: 执行一条独立语句或声明：`intrinsic_name.intr_name>;`。
- **L3199 EN**: Closes the current lexical scope or compound statement.
  **L3199 CN**: 结束当前词法作用域或复合语句块。
- **L3200 EN**: Closes the current lexical scope or compound statement.
  **L3200 CN**: 结束当前词法作用域或复合语句块。

### Lines 3201-3232

````tablegen
}

// Tcgen05 ld intrinsics
class NVVM_TCGEN05_LD<string Shape, int Num> :
        Intrinsic<[NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.type],
                  !listconcat([llvm_tmem_ptr_ty],
                              !if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),
                              [llvm_i1_ty]),
                  !listconcat([IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>],
                              !if(!eq(Shape, "16x32bx2"),
                                [ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>],
                                [ImmArg<ArgIndex<1>>]))>;

// Tcgen05 ld.red intrinsics
class NVVM_TCGEN05_LD_RED<string Shape, int Num, string RedValTy> :
        DefaultAttrsIntrinsicFlags<
          [NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num, RedValTy>.type, LLVMType<!cast<ValueType>(RedValTy)>],
                  [llvm_tmem_ptr_ty],
                  !listconcat(!if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),
                              [llvm_i32_ty], // RedOp
                              !if(!eq(RedValTy, "f32"), [llvm_i1_ty, llvm_i1_ty], [])), // abs, nan
                  [IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;

// Tcgen05 st intrinsics
class NVVM_TCGEN05_ST<string Shape, int Num> :
        Intrinsic<[],
                  !listconcat([llvm_tmem_ptr_ty],
                              !if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),
                              [NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.type],
                              [llvm_i1_ty]),
                  !listconcat([IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>],
                              !if(!eq(Shape, "16x32bx2"),
````
- **L3201 EN**: Closes the current lexical scope or compound statement.
  **L3201 CN**: 结束当前词法作用域或复合语句块。
- **L3202 EN**: Blank line separating nearby declarations or logic blocks.
  **L3202 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3203 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 ld intrinsics`.
  **L3203 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 ld intrinsics`。
- **L3204 EN**: Declares class `NVVM_TCGEN05_LD<string`.
  **L3204 CN**: 声明 class `NVVM_TCGEN05_LD<string`。
- **L3205 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.type],`.
  **L3205 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.type],`。
- **L3206 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_tmem_ptr_ty],`.
  **L3206 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_tmem_ptr_ty],`。
- **L3207 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),`.
  **L3207 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),`。
- **L3208 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty]),`.
  **L3208 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty]),`。
- **L3209 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>],`.
  **L3209 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>],`。
- **L3210 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(Shape, "16x32bx2"),`.
  **L3210 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(Shape, "16x32bx2"),`。
- **L3211 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>],`.
  **L3211 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<2>>],`。
- **L3212 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<1>>]))>;`.
  **L3212 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<1>>]))>;`。
- **L3213 EN**: Blank line separating nearby declarations or logic blocks.
  **L3213 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3214 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 ld.red intrinsics`.
  **L3214 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 ld.red intrinsics`。
- **L3215 EN**: Declares class `NVVM_TCGEN05_LD_RED<string`.
  **L3215 CN**: 声明 class `NVVM_TCGEN05_LD_RED<string`。
- **L3216 EN**: Continues the surrounding expression or declaration: `DefaultAttrsIntrinsicFlags<`.
  **L3216 CN**: 继续构造周围的表达式或声明：`DefaultAttrsIntrinsicFlags<`。
- **L3217 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num, RedValTy>.type, LLVMType<!cast<ValueType>(RedValTy)>],`.
  **L3217 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num, RedValTy>.type, LLVMType<!cast<ValueType>(RedValTy)>],`。
- **L3218 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_tmem_ptr_ty],`.
  **L3218 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_tmem_ptr_ty],`。
- **L3219 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(!if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),`.
  **L3219 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(!if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),`。
- **L3220 EN**: Continues the surrounding expression or declaration: `[llvm_i32_ty], // RedOp`.
  **L3220 CN**: 继续构造周围的表达式或声明：`[llvm_i32_ty], // RedOp`。
- **L3221 EN**: Continues the surrounding expression or declaration: `!if(!eq(RedValTy, "f32"), [llvm_i1_ty, llvm_i1_ty], [])), // abs, nan`.
  **L3221 CN**: 继续构造周围的表达式或声明：`!if(!eq(RedValTy, "f32"), [llvm_i1_ty, llvm_i1_ty], [])), // abs, nan`。
- **L3222 EN**: Executes a standalone statement or declaration: `[IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`.
  **L3222 CN**: 执行一条独立语句或声明：`[IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>]>;`。
- **L3223 EN**: Blank line separating nearby declarations or logic blocks.
  **L3223 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3224 EN**: Comment explains nearby logic, invariants, or intent: `Tcgen05 st intrinsics`.
  **L3224 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Tcgen05 st intrinsics`。
- **L3225 EN**: Declares class `NVVM_TCGEN05_ST<string`.
  **L3225 CN**: 声明 class `NVVM_TCGEN05_ST<string`。
- **L3226 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Intrinsic<[],`.
  **L3226 CN**: 继续一个多行参数列表、初始化器或聚合项：`Intrinsic<[],`。
- **L3227 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([llvm_tmem_ptr_ty],`.
  **L3227 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([llvm_tmem_ptr_ty],`。
- **L3228 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),`.
  **L3228 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(Shape, "16x32bx2"), [llvm_i64_ty], []),`。
- **L3229 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.type],`.
  **L3229 CN**: 继续一个多行参数列表、初始化器或聚合项：`[NVVM_TCGEN05_LDST_ACCESS_SIZE<Shape, Num>.type],`。
- **L3230 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[llvm_i1_ty]),`.
  **L3230 CN**: 继续一个多行参数列表、初始化器或聚合项：`[llvm_i1_ty]),`。
- **L3231 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat([IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>],`.
  **L3231 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat([IntrConvergent, IntrArgMemOnly, NoCapture<ArgIndex<0>>],`。
- **L3232 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(Shape, "16x32bx2"),`.
  **L3232 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(Shape, "16x32bx2"),`。

### Lines 3233-3264

````tablegen
                                [ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>],
                                [ImmArg<ArgIndex<2>>]))>;

// tcgen05 ld/st intrinsics
foreach shape = ["16x64b", "16x128b", "16x256b", "32x32b", "16x32bx2"] in {
  foreach num = 0...8 in {
    if NVVM_TCGEN05_LDST_ACCESS_SIZE<shape, num>.valid then {
      def int_nvvm_tcgen05_ld_ # shape # _x # !shl(1, num) :
            NVVM_TCGEN05_LD<shape, num>;
      def int_nvvm_tcgen05_st_ # shape # _x # !shl(1, num) :
            NVVM_TCGEN05_ST<shape, num>;
    }
  }
}

// tcgen05.ld.red intrinsics
foreach shape = ["32x32b", "16x32bx2"] in {
  // num starts with x2 (1 << 1) as tcgen05.ld.red does not support x1
  foreach num = 1...8 in {
    foreach ty = ["f32", "i32"] in {
      if NVVM_TCGEN05_LDST_ACCESS_SIZE<shape, num>.valid then {

        def IntrinsicName<"llvm.nvvm.tcgen05.ld.red." # shape
                          # ".x" # !shl(1, num) # "." # ty>.record_name:
              NVVM_TCGEN05_LD_RED<shape, num, ty>;
      } // valid
    } // ty
  } // num
} // shape

//
// Bulk store intrinsics
````
- **L3233 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>],`.
  **L3233 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ImmArg<ArgIndex<1>>, ImmArg<ArgIndex<3>>],`。
- **L3234 EN**: Executes a standalone statement or declaration: `[ImmArg<ArgIndex<2>>]))>;`.
  **L3234 CN**: 执行一条独立语句或声明：`[ImmArg<ArgIndex<2>>]))>;`。
- **L3235 EN**: Blank line separating nearby declarations or logic blocks.
  **L3235 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3236 EN**: Comment explains nearby logic, invariants, or intent: `tcgen05 ld/st intrinsics`.
  **L3236 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tcgen05 ld/st intrinsics`。
- **L3237 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3237 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3238 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3238 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3239 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L3239 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3240 EN**: Declares TableGen def `int_nvvm_tcgen05_ld_`.
  **L3240 CN**: 声明 TableGen def `int_nvvm_tcgen05_ld_`。
- **L3241 EN**: Executes a standalone statement or declaration: `NVVM_TCGEN05_LD<shape, num>;`.
  **L3241 CN**: 执行一条独立语句或声明：`NVVM_TCGEN05_LD<shape, num>;`。
- **L3242 EN**: Declares TableGen def `int_nvvm_tcgen05_st_`.
  **L3242 CN**: 声明 TableGen def `int_nvvm_tcgen05_st_`。
- **L3243 EN**: Executes a standalone statement or declaration: `NVVM_TCGEN05_ST<shape, num>;`.
  **L3243 CN**: 执行一条独立语句或声明：`NVVM_TCGEN05_ST<shape, num>;`。
- **L3244 EN**: Closes the current lexical scope or compound statement.
  **L3244 CN**: 结束当前词法作用域或复合语句块。
- **L3245 EN**: Closes the current lexical scope or compound statement.
  **L3245 CN**: 结束当前词法作用域或复合语句块。
- **L3246 EN**: Closes the current lexical scope or compound statement.
  **L3246 CN**: 结束当前词法作用域或复合语句块。
- **L3247 EN**: Blank line separating nearby declarations or logic blocks.
  **L3247 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3248 EN**: Comment explains nearby logic, invariants, or intent: `tcgen05.ld.red intrinsics`.
  **L3248 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tcgen05.ld.red intrinsics`。
- **L3249 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3249 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3250 EN**: Comment explains nearby logic, invariants, or intent: `num starts with x2 (1 << 1) as tcgen05.ld.red does not support x1`.
  **L3250 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`num starts with x2 (1 << 1) as tcgen05.ld.red does not support x1`。
- **L3251 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3251 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3252 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3252 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3253 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L3253 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3254 EN**: Blank line separating nearby declarations or logic blocks.
  **L3254 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3255 EN**: Declares TableGen def `IntrinsicName`.
  **L3255 CN**: 声明 TableGen def `IntrinsicName`。
- **L3256 EN**: Continues logic associated with callable symbol `shl`.
  **L3256 CN**: 继续与可调用符号 `shl` 相关的逻辑。
- **L3257 EN**: Executes a standalone statement or declaration: `NVVM_TCGEN05_LD_RED<shape, num, ty>;`.
  **L3257 CN**: 执行一条独立语句或声明：`NVVM_TCGEN05_LD_RED<shape, num, ty>;`。
- **L3258 EN**: Continues the surrounding expression or declaration: `} // valid`.
  **L3258 CN**: 继续构造周围的表达式或声明：`} // valid`。
- **L3259 EN**: Continues the surrounding expression or declaration: `} // ty`.
  **L3259 CN**: 继续构造周围的表达式或声明：`} // ty`。
- **L3260 EN**: Continues the surrounding expression or declaration: `} // num`.
  **L3260 CN**: 继续构造周围的表达式或声明：`} // num`。
- **L3261 EN**: Continues the surrounding expression or declaration: `} // shape`.
  **L3261 CN**: 继续构造周围的表达式或声明：`} // shape`。
- **L3262 EN**: Blank line separating nearby declarations or logic blocks.
  **L3262 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3263 EN**: Separator comment used for visual grouping.
  **L3263 CN**: 用于视觉分组的分隔注释。
- **L3264 EN**: Comment explains nearby logic, invariants, or intent: `Bulk store intrinsics`.
  **L3264 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Bulk store intrinsics`。

### Lines 3265-3296

````tablegen
//
let IntrProperties = [IntrArgMemOnly, IntrWriteMem, WriteOnly<ArgIndex<0>>,
                      NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<2>>] in {
  def int_nvvm_st_bulk :
      DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty, llvm_i64_ty]>;

  def int_nvvm_st_bulk_shared_cta :
      DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_i64_ty, llvm_i64_ty]>;
}

//
// clusterlaunchcontorl Intrinsics
//

// clusterlaunchcontrol.try_cancel

def int_nvvm_clusterlaunchcontrol_try_cancel_async_shared
    : DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_shared_ptr_ty],
                [IntrHasSideEffects, IntrArgMemOnly],
                "llvm.nvvm.clusterlaunchcontrol.try_cancel.async.shared">;

def int_nvvm_clusterlaunchcontrol_try_cancel_async_multicast_shared
    : DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_shared_ptr_ty],
                [IntrHasSideEffects, IntrArgMemOnly],
                "llvm.nvvm.clusterlaunchcontrol.try_cancel.async.multicast.shared">;

// clusterlaunchcontrol.query_cancel.is_canceled

def int_nvvm_clusterlaunchcontrol_query_cancel_is_canceled
    : NVVMPureIntrinsic<[llvm_i1_ty], [llvm_i128_ty], [],
                    "llvm.nvvm.clusterlaunchcontrol.query_cancel.is_canceled">;

````
- **L3265 EN**: Separator comment used for visual grouping.
  **L3265 CN**: 用于视觉分组的分隔注释。
- **L3266 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3266 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3267 EN**: Continues the surrounding expression or declaration: `NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<2>>] in {`.
  **L3267 CN**: 继续构造周围的表达式或声明：`NoCapture<ArgIndex<0>>, ImmArg<ArgIndex<2>>] in {`。
- **L3268 EN**: Declares TableGen def `int_nvvm_st_bulk`.
  **L3268 CN**: 声明 TableGen def `int_nvvm_st_bulk`。
- **L3269 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty, llvm_i64_ty]>;`.
  **L3269 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [llvm_ptr_ty, llvm_i64_ty, llvm_i64_ty]>;`。
- **L3270 EN**: Blank line separating nearby declarations or logic blocks.
  **L3270 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3271 EN**: Declares TableGen def `int_nvvm_st_bulk_shared_cta`.
  **L3271 CN**: 声明 TableGen def `int_nvvm_st_bulk_shared_cta`。
- **L3272 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_i64_ty, llvm_i64_ty]>;`.
  **L3272 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_i64_ty, llvm_i64_ty]>;`。
- **L3273 EN**: Closes the current lexical scope or compound statement.
  **L3273 CN**: 结束当前词法作用域或复合语句块。
- **L3274 EN**: Blank line separating nearby declarations or logic blocks.
  **L3274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3275 EN**: Separator comment used for visual grouping.
  **L3275 CN**: 用于视觉分组的分隔注释。
- **L3276 EN**: Comment explains nearby logic, invariants, or intent: `clusterlaunchcontorl Intrinsics`.
  **L3276 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`clusterlaunchcontorl Intrinsics`。
- **L3277 EN**: Separator comment used for visual grouping.
  **L3277 CN**: 用于视觉分组的分隔注释。
- **L3278 EN**: Blank line separating nearby declarations or logic blocks.
  **L3278 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3279 EN**: Comment explains nearby logic, invariants, or intent: `clusterlaunchcontrol.try_cancel`.
  **L3279 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`clusterlaunchcontrol.try_cancel`。
- **L3280 EN**: Blank line separating nearby declarations or logic blocks.
  **L3280 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3281 EN**: Declares TableGen def `int_nvvm_clusterlaunchcontrol_try_cancel_async_shared`.
  **L3281 CN**: 声明 TableGen def `int_nvvm_clusterlaunchcontrol_try_cancel_async_shared`。
- **L3282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_shared_ptr_ty],`.
  **L3282 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_shared_ptr_ty],`。
- **L3283 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrHasSideEffects, IntrArgMemOnly],`.
  **L3283 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrHasSideEffects, IntrArgMemOnly],`。
- **L3284 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.clusterlaunchcontrol.try_cancel.async.shared">;`.
  **L3284 CN**: 执行一条独立语句或声明：`"llvm.nvvm.clusterlaunchcontrol.try_cancel.async.shared">;`。
- **L3285 EN**: Blank line separating nearby declarations or logic blocks.
  **L3285 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3286 EN**: Declares TableGen def `int_nvvm_clusterlaunchcontrol_try_cancel_async_multicast_shared`.
  **L3286 CN**: 声明 TableGen def `int_nvvm_clusterlaunchcontrol_try_cancel_async_multicast_shared`。
- **L3287 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_shared_ptr_ty],`.
  **L3287 CN**: 继续一个多行参数列表、初始化器或聚合项：`: DefaultAttrsIntrinsic<[], [llvm_shared_ptr_ty, llvm_shared_ptr_ty],`。
- **L3288 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrHasSideEffects, IntrArgMemOnly],`.
  **L3288 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrHasSideEffects, IntrArgMemOnly],`。
- **L3289 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.clusterlaunchcontrol.try_cancel.async.multicast.shared">;`.
  **L3289 CN**: 执行一条独立语句或声明：`"llvm.nvvm.clusterlaunchcontrol.try_cancel.async.multicast.shared">;`。
- **L3290 EN**: Blank line separating nearby declarations or logic blocks.
  **L3290 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3291 EN**: Comment explains nearby logic, invariants, or intent: `clusterlaunchcontrol.query_cancel.is_canceled`.
  **L3291 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`clusterlaunchcontrol.query_cancel.is_canceled`。
- **L3292 EN**: Blank line separating nearby declarations or logic blocks.
  **L3292 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3293 EN**: Declares TableGen def `int_nvvm_clusterlaunchcontrol_query_cancel_is_canceled`.
  **L3293 CN**: 声明 TableGen def `int_nvvm_clusterlaunchcontrol_query_cancel_is_canceled`。
- **L3294 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: NVVMPureIntrinsic<[llvm_i1_ty], [llvm_i128_ty], [],`.
  **L3294 CN**: 继续一个多行参数列表、初始化器或聚合项：`: NVVMPureIntrinsic<[llvm_i1_ty], [llvm_i128_ty], [],`。
- **L3295 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.clusterlaunchcontrol.query_cancel.is_canceled">;`.
  **L3295 CN**: 执行一条独立语句或声明：`"llvm.nvvm.clusterlaunchcontrol.query_cancel.is_canceled">;`。
- **L3296 EN**: Blank line separating nearby declarations or logic blocks.
  **L3296 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 3297-3328

````tablegen
foreach dim = ["x", "y", "z"] in
  def int_nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_ # dim
    : NVVMPureIntrinsic<[llvm_i32_ty], [llvm_i128_ty], [],
                    "llvm.nvvm.clusterlaunchcontrol.query_cancel.get_first_ctaid." # dim>;

//
// tcgen05.mma intrinsics
//

foreach sparse = [0, 1] in {
  foreach space = ["tensor", "shared"] in {
    foreach scale_d = [0, 1] in {
      foreach ashift = !if(!eq(space, "tensor"), [0, 1], [0]) in {
        defvar mma = NVVM_TCGEN05_MMA<sparse, space, ashift, scale_d>;
        defvar args = !listconcat(
          mma.common_args,
          !if(!eq(scale_d, 1), [llvm_i64_ty], []) // scale_d_imm
        );
        defvar flags = [llvm_i32_ty,   // kind
                        llvm_i32_ty,   // cta_group
                        llvm_i32_ty];  // collector_usage_a
        defvar nargs = !size(args);
        defvar scale_d_imm = ArgIndex<!sub(nargs, 1)>;
        defvar scale_d_imm_range = [ImmArg<scale_d_imm>, Range<scale_d_imm, 0, 16>];

        // Check if this is the specific llvm.nvvm.tcgen05.mma.tensor intrinsic.
        defvar is_target_intrinsic = !and(!eq(sparse, 0),
                                          !eq(space, "tensor"),
                                          !eq(scale_d, 0),
                                          !eq(ashift, 0));

        defvar base_properties = !listconcat(
````
- **L3297 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3297 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3298 EN**: Declares TableGen def `int_nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_`.
  **L3298 CN**: 声明 TableGen def `int_nvvm_clusterlaunchcontrol_query_cancel_get_first_ctaid_`。
- **L3299 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: NVVMPureIntrinsic<[llvm_i32_ty], [llvm_i128_ty], [],`.
  **L3299 CN**: 继续一个多行参数列表、初始化器或聚合项：`: NVVMPureIntrinsic<[llvm_i32_ty], [llvm_i128_ty], [],`。
- **L3300 EN**: Executes a standalone statement or declaration: `"llvm.nvvm.clusterlaunchcontrol.query_cancel.get_first_ctaid." # dim>;`.
  **L3300 CN**: 执行一条独立语句或声明：`"llvm.nvvm.clusterlaunchcontrol.query_cancel.get_first_ctaid." # dim>;`。
- **L3301 EN**: Blank line separating nearby declarations or logic blocks.
  **L3301 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3302 EN**: Separator comment used for visual grouping.
  **L3302 CN**: 用于视觉分组的分隔注释。
- **L3303 EN**: Comment explains nearby logic, invariants, or intent: `tcgen05.mma intrinsics`.
  **L3303 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tcgen05.mma intrinsics`。
- **L3304 EN**: Separator comment used for visual grouping.
  **L3304 CN**: 用于视觉分组的分隔注释。
- **L3305 EN**: Blank line separating nearby declarations or logic blocks.
  **L3305 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3306 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3306 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3307 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3307 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3308 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3308 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3309 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3309 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3310 EN**: Initializes variable `mma` from the right-hand expression.
  **L3310 CN**: 使用右侧表达式初始化变量 `mma`。
- **L3311 EN**: Continues logic associated with callable symbol `listconcat`.
  **L3311 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L3312 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mma.common_args,`.
  **L3312 CN**: 继续一个多行参数列表、初始化器或聚合项：`mma.common_args,`。
- **L3313 EN**: Continues the surrounding expression or declaration: `!if(!eq(scale_d, 1), [llvm_i64_ty], []) // scale_d_imm`.
  **L3313 CN**: 继续构造周围的表达式或声明：`!if(!eq(scale_d, 1), [llvm_i64_ty], []) // scale_d_imm`。
- **L3314 EN**: Executes a standalone statement or declaration: `);`.
  **L3314 CN**: 执行一条独立语句或声明：`);`。
- **L3315 EN**: Continues the surrounding expression or declaration: `defvar flags = [llvm_i32_ty,   // kind`.
  **L3315 CN**: 继续构造周围的表达式或声明：`defvar flags = [llvm_i32_ty,   // kind`。
- **L3316 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,   // cta_group`.
  **L3316 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,   // cta_group`。
- **L3317 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty];  // collector_usage_a`.
  **L3317 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty];  // collector_usage_a`。
- **L3318 EN**: Initializes variable `nargs` from the right-hand expression.
  **L3318 CN**: 使用右侧表达式初始化变量 `nargs`。
- **L3319 EN**: Initializes variable `scale_d_imm` from the right-hand expression.
  **L3319 CN**: 使用右侧表达式初始化变量 `scale_d_imm`。
- **L3320 EN**: Initializes variable `scale_d_imm_range` from the right-hand expression.
  **L3320 CN**: 使用右侧表达式初始化变量 `scale_d_imm_range`。
- **L3321 EN**: Blank line separating nearby declarations or logic blocks.
  **L3321 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3322 EN**: Comment explains nearby logic, invariants, or intent: `Check if this is the specific llvm.nvvm.tcgen05.mma.tensor intrinsic.`.
  **L3322 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`Check if this is the specific llvm.nvvm.tcgen05.mma.tensor intrinsic.`。
- **L3323 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar is_target_intrinsic = !and(!eq(sparse, 0),`.
  **L3323 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar is_target_intrinsic = !and(!eq(sparse, 0),`。
- **L3324 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(space, "tensor"),`.
  **L3324 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(space, "tensor"),`。
- **L3325 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!eq(scale_d, 0),`.
  **L3325 CN**: 继续一个多行参数列表、初始化器或聚合项：`!eq(scale_d, 0),`。
- **L3326 EN**: Executes a call or declaration centered on `!eq`.
  **L3326 CN**: 执行以 `!eq` 为核心的调用或声明。
- **L3327 EN**: Blank line separating nearby declarations or logic blocks.
  **L3327 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3328 EN**: Continues logic associated with callable symbol `listconcat`.
  **L3328 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。

### Lines 3329-3360

````tablegen
          mma.common_intr_props,
          !if(!eq(scale_d, 1), scale_d_imm_range, []),
          [Range<ArgIndex<nargs>, 0, !if(!eq(scale_d, 1), 2, 4)>, // kind
           Range<ArgIndex<!add(nargs, 1)>, 1, 3>,                 // cta_group
           Range<ArgIndex<!add(nargs, 2)>, 0,
                          !if(!eq(ashift, 1), 2, 4)>              // collector_usage
          ]
        );

        defvar intrinsic_properties = !if(is_target_intrinsic,
          !listconcat(base_properties,
            [ArgInfo<ArgIndex<nargs>, [ArgName<"kind">, ImmArgPrinter<"printTcgen05MMAKind">]>,
             ArgInfo<ArgIndex<!add(nargs, 1)>, [ArgName<"cta_group">]>,
             ArgInfo<ArgIndex<!add(nargs, 2)>, [ArgName<"collector">, ImmArgPrinter<"printTcgen05CollectorUsageOp">]>]),
          base_properties);

        def mma.record_name:
              DefaultAttrsIntrinsicFlags<[], args, flags, intrinsic_properties,
                mma.intr_name>;
      } // ashift
    } // scale_d
  } // space
} // sparse

//
// tcgen05.mma disable_output_lane intrinsics
//
foreach sparse = [0, 1] in {
  foreach space = ["tensor", "shared"] in {
    foreach cta_group = [1, 2] in {
      foreach scale_d = [0, 1] in {
        foreach ashift = !if(!eq(space, "tensor"), [0, 1], [0]) in {
````
- **L3329 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mma.common_intr_props,`.
  **L3329 CN**: 继续一个多行参数列表、初始化器或聚合项：`mma.common_intr_props,`。
- **L3330 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(scale_d, 1), scale_d_imm_range, []),`.
  **L3330 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(scale_d, 1), scale_d_imm_range, []),`。
- **L3331 EN**: Continues the surrounding expression or declaration: `[Range<ArgIndex<nargs>, 0, !if(!eq(scale_d, 1), 2, 4)>, // kind`.
  **L3331 CN**: 继续构造周围的表达式或声明：`[Range<ArgIndex<nargs>, 0, !if(!eq(scale_d, 1), 2, 4)>, // kind`。
- **L3332 EN**: Continues logic associated with callable symbol `add`.
  **L3332 CN**: 继续与可调用符号 `add` 相关的逻辑。
- **L3333 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<ArgIndex<!add(nargs, 2)>, 0,`.
  **L3333 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<ArgIndex<!add(nargs, 2)>, 0,`。
- **L3334 EN**: Continues the surrounding expression or declaration: `!if(!eq(ashift, 1), 2, 4)>              // collector_usage`.
  **L3334 CN**: 继续构造周围的表达式或声明：`!if(!eq(ashift, 1), 2, 4)>              // collector_usage`。
- **L3335 EN**: Continues the surrounding expression or declaration: `]`.
  **L3335 CN**: 继续构造周围的表达式或声明：`]`。
- **L3336 EN**: Executes a standalone statement or declaration: `);`.
  **L3336 CN**: 执行一条独立语句或声明：`);`。
- **L3337 EN**: Blank line separating nearby declarations or logic blocks.
  **L3337 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3338 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar intrinsic_properties = !if(is_target_intrinsic,`.
  **L3338 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar intrinsic_properties = !if(is_target_intrinsic,`。
- **L3339 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(base_properties,`.
  **L3339 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(base_properties,`。
- **L3340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[ArgInfo<ArgIndex<nargs>, [ArgName<"kind">, ImmArgPrinter<"printTcgen05MMAKind">]>,`.
  **L3340 CN**: 继续一个多行参数列表、初始化器或聚合项：`[ArgInfo<ArgIndex<nargs>, [ArgName<"kind">, ImmArgPrinter<"printTcgen05MMAKind">]>,`。
- **L3341 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArgInfo<ArgIndex<!add(nargs, 1)>, [ArgName<"cta_group">]>,`.
  **L3341 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArgInfo<ArgIndex<!add(nargs, 1)>, [ArgName<"cta_group">]>,`。
- **L3342 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArgInfo<ArgIndex<!add(nargs, 2)>, [ArgName<"collector">, ImmArgPrinter<"printTcgen05CollectorUsageOp">]>]),`.
  **L3342 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArgInfo<ArgIndex<!add(nargs, 2)>, [ArgName<"collector">, ImmArgPrinter<"printTcgen05CollectorUsageOp">]>]),`。
- **L3343 EN**: Executes a standalone statement or declaration: `base_properties);`.
  **L3343 CN**: 执行一条独立语句或声明：`base_properties);`。
- **L3344 EN**: Blank line separating nearby declarations or logic blocks.
  **L3344 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3345 EN**: Declares TableGen def `mma.record_name`.
  **L3345 CN**: 声明 TableGen def `mma.record_name`。
- **L3346 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsicFlags<[], args, flags, intrinsic_properties,`.
  **L3346 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsicFlags<[], args, flags, intrinsic_properties,`。
- **L3347 EN**: Executes a standalone statement or declaration: `mma.intr_name>;`.
  **L3347 CN**: 执行一条独立语句或声明：`mma.intr_name>;`。
- **L3348 EN**: Continues the surrounding expression or declaration: `} // ashift`.
  **L3348 CN**: 继续构造周围的表达式或声明：`} // ashift`。
- **L3349 EN**: Continues the surrounding expression or declaration: `} // scale_d`.
  **L3349 CN**: 继续构造周围的表达式或声明：`} // scale_d`。
- **L3350 EN**: Continues the surrounding expression or declaration: `} // space`.
  **L3350 CN**: 继续构造周围的表达式或声明：`} // space`。
- **L3351 EN**: Continues the surrounding expression or declaration: `} // sparse`.
  **L3351 CN**: 继续构造周围的表达式或声明：`} // sparse`。
- **L3352 EN**: Blank line separating nearby declarations or logic blocks.
  **L3352 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3353 EN**: Separator comment used for visual grouping.
  **L3353 CN**: 用于视觉分组的分隔注释。
- **L3354 EN**: Comment explains nearby logic, invariants, or intent: `tcgen05.mma disable_output_lane intrinsics`.
  **L3354 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tcgen05.mma disable_output_lane intrinsics`。
- **L3355 EN**: Separator comment used for visual grouping.
  **L3355 CN**: 用于视觉分组的分隔注释。
- **L3356 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3356 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3357 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3357 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3358 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3358 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3359 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3359 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3360 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3360 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 3361-3392

````tablegen
          defvar mma = NVVM_TCGEN05_MMA_DISABLE_OUTPUT_LANE<
                         sparse, space, cta_group, ashift, scale_d>;
          defvar disable_output_lane_type =
                  !if(!eq(cta_group, 1), llvm_v4i32_ty, llvm_v8i32_ty);
          defvar args = !listconcat(
                          mma.common_args,
                          !if(!eq(scale_d, 1), [llvm_i64_ty], []),
                          [disable_output_lane_type]
                        );
          defvar flags = [llvm_i32_ty,  // kind_flag
                          llvm_i32_ty]; // collector_usage_a_flag
          defvar nargs = !size(args);
          defvar scale_d_imm = ArgIndex<!sub(nargs, 2)>;
          defvar scale_d_imm_range = [ImmArg<scale_d_imm>, Range<scale_d_imm, 0, 16>];
          defvar intrinsic_properties = !listconcat(
                    mma.common_intr_props,
                    !if(!eq(scale_d, 1), scale_d_imm_range, []),
                    [Range<ArgIndex<nargs>, 0, !if(!eq(scale_d, 1), 2, 4)>,
                     Range<ArgIndex<!add(nargs, 1)>, 0, !if(!eq(ashift, 1), 2, 4)>]
                  );

          def mma.record_name : DefaultAttrsIntrinsicFlags<[], args, flags,
                                  intrinsic_properties, mma.intr_name>;
        } // ashift
      } // scale_d
    } // cta_group
  } // space
} // sparse

//
// tcgen05.mma block_scale intrinsics
//
````
- **L3361 EN**: Continues the surrounding expression or declaration: `defvar mma = NVVM_TCGEN05_MMA_DISABLE_OUTPUT_LANE<`.
  **L3361 CN**: 继续构造周围的表达式或声明：`defvar mma = NVVM_TCGEN05_MMA_DISABLE_OUTPUT_LANE<`。
- **L3362 EN**: Executes a standalone statement or declaration: `sparse, space, cta_group, ashift, scale_d>;`.
  **L3362 CN**: 执行一条独立语句或声明：`sparse, space, cta_group, ashift, scale_d>;`。
- **L3363 EN**: Continues the surrounding expression or declaration: `defvar disable_output_lane_type =`.
  **L3363 CN**: 继续构造周围的表达式或声明：`defvar disable_output_lane_type =`。
- **L3364 EN**: Executes a call or declaration centered on `!if`.
  **L3364 CN**: 执行以 `!if` 为核心的调用或声明。
- **L3365 EN**: Continues logic associated with callable symbol `listconcat`.
  **L3365 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L3366 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mma.common_args,`.
  **L3366 CN**: 继续一个多行参数列表、初始化器或聚合项：`mma.common_args,`。
- **L3367 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(scale_d, 1), [llvm_i64_ty], []),`.
  **L3367 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(scale_d, 1), [llvm_i64_ty], []),`。
- **L3368 EN**: Continues the surrounding expression or declaration: `[disable_output_lane_type]`.
  **L3368 CN**: 继续构造周围的表达式或声明：`[disable_output_lane_type]`。
- **L3369 EN**: Executes a standalone statement or declaration: `);`.
  **L3369 CN**: 执行一条独立语句或声明：`);`。
- **L3370 EN**: Continues the surrounding expression or declaration: `defvar flags = [llvm_i32_ty,  // kind_flag`.
  **L3370 CN**: 继续构造周围的表达式或声明：`defvar flags = [llvm_i32_ty,  // kind_flag`。
- **L3371 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty]; // collector_usage_a_flag`.
  **L3371 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty]; // collector_usage_a_flag`。
- **L3372 EN**: Initializes variable `nargs` from the right-hand expression.
  **L3372 CN**: 使用右侧表达式初始化变量 `nargs`。
- **L3373 EN**: Initializes variable `scale_d_imm` from the right-hand expression.
  **L3373 CN**: 使用右侧表达式初始化变量 `scale_d_imm`。
- **L3374 EN**: Initializes variable `scale_d_imm_range` from the right-hand expression.
  **L3374 CN**: 使用右侧表达式初始化变量 `scale_d_imm_range`。
- **L3375 EN**: Continues logic associated with callable symbol `listconcat`.
  **L3375 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L3376 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mma.common_intr_props,`.
  **L3376 CN**: 继续一个多行参数列表、初始化器或聚合项：`mma.common_intr_props,`。
- **L3377 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!if(!eq(scale_d, 1), scale_d_imm_range, []),`.
  **L3377 CN**: 继续一个多行参数列表、初始化器或聚合项：`!if(!eq(scale_d, 1), scale_d_imm_range, []),`。
- **L3378 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<ArgIndex<nargs>, 0, !if(!eq(scale_d, 1), 2, 4)>,`.
  **L3378 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<ArgIndex<nargs>, 0, !if(!eq(scale_d, 1), 2, 4)>,`。
- **L3379 EN**: Continues logic associated with callable symbol `add`.
  **L3379 CN**: 继续与可调用符号 `add` 相关的逻辑。
- **L3380 EN**: Executes a standalone statement or declaration: `);`.
  **L3380 CN**: 执行一条独立语句或声明：`);`。
- **L3381 EN**: Blank line separating nearby declarations or logic blocks.
  **L3381 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3382 EN**: Declares TableGen def `mma.record_name`.
  **L3382 CN**: 声明 TableGen def `mma.record_name`。
- **L3383 EN**: Executes a standalone statement or declaration: `intrinsic_properties, mma.intr_name>;`.
  **L3383 CN**: 执行一条独立语句或声明：`intrinsic_properties, mma.intr_name>;`。
- **L3384 EN**: Continues the surrounding expression or declaration: `} // ashift`.
  **L3384 CN**: 继续构造周围的表达式或声明：`} // ashift`。
- **L3385 EN**: Continues the surrounding expression or declaration: `} // scale_d`.
  **L3385 CN**: 继续构造周围的表达式或声明：`} // scale_d`。
- **L3386 EN**: Continues the surrounding expression or declaration: `} // cta_group`.
  **L3386 CN**: 继续构造周围的表达式或声明：`} // cta_group`。
- **L3387 EN**: Continues the surrounding expression or declaration: `} // space`.
  **L3387 CN**: 继续构造周围的表达式或声明：`} // space`。
- **L3388 EN**: Continues the surrounding expression or declaration: `} // sparse`.
  **L3388 CN**: 继续构造周围的表达式或声明：`} // sparse`。
- **L3389 EN**: Blank line separating nearby declarations or logic blocks.
  **L3389 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3390 EN**: Separator comment used for visual grouping.
  **L3390 CN**: 用于视觉分组的分隔注释。
- **L3391 EN**: Comment explains nearby logic, invariants, or intent: `tcgen05.mma block_scale intrinsics`.
  **L3391 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tcgen05.mma block_scale intrinsics`。
- **L3392 EN**: Separator comment used for visual grouping.
  **L3392 CN**: 用于视觉分组的分隔注释。

### Lines 3393-3424

````tablegen
foreach sparse = [0, 1] in {
  foreach space = ["tensor", "shared"] in {
    foreach kind = ["mxf8f6f4", "mxf4", "mxf4nvf4"] in {
      foreach scale_vec_size = ["", ".block16", ".block32"] in {
        defvar mma = NVVM_TCGEN05_MMA_BLOCKSCALE<sparse, space, kind, scale_vec_size>;
        defvar args = !listconcat(mma.common_args,
                                  [llvm_tmem_ptr_ty,   // scale_a
                                   llvm_tmem_ptr_ty]); // scale_b
        defvar flags = [llvm_i32_ty,  // cta_group
                        llvm_i32_ty]; // collector_usage_a
        defvar nargs = !size(args);
        defvar cta_group = ArgIndex<nargs>;
        defvar collector_usage = ArgIndex<!add(nargs, 1)>;

        if NVVM_TCGEN05_MMA_BLOCKSCALE_SUPPORTED<kind, scale_vec_size>.ret then {
          def mma.record_name : DefaultAttrsIntrinsicFlags<[], args, flags,
            !listconcat(mma.common_intr_props,
                        [Range<cta_group, 1, 3>,
                         Range<collector_usage, 0, 4>]),
            mma.intr_name>;
        }
      } // scale_vec_size
    } // kind
  } // space
} // sparse

//
// tcgen05.mma ws intrinsics
//
foreach sparse = [0, 1] in {
  foreach space = ["tensor", "shared"] in {
    foreach zero_col_mask = [0, 1] in {
````
- **L3393 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3393 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3394 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3394 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3395 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3395 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3396 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3396 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3397 EN**: Initializes variable `mma` from the right-hand expression.
  **L3397 CN**: 使用右侧表达式初始化变量 `mma`。
- **L3398 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `defvar args = !listconcat(mma.common_args,`.
  **L3398 CN**: 继续一个多行参数列表、初始化器或聚合项：`defvar args = !listconcat(mma.common_args,`。
- **L3399 EN**: Continues the surrounding expression or declaration: `[llvm_tmem_ptr_ty,   // scale_a`.
  **L3399 CN**: 继续构造周围的表达式或声明：`[llvm_tmem_ptr_ty,   // scale_a`。
- **L3400 EN**: Continues the surrounding expression or declaration: `llvm_tmem_ptr_ty]); // scale_b`.
  **L3400 CN**: 继续构造周围的表达式或声明：`llvm_tmem_ptr_ty]); // scale_b`。
- **L3401 EN**: Continues the surrounding expression or declaration: `defvar flags = [llvm_i32_ty,  // cta_group`.
  **L3401 CN**: 继续构造周围的表达式或声明：`defvar flags = [llvm_i32_ty,  // cta_group`。
- **L3402 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty]; // collector_usage_a`.
  **L3402 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty]; // collector_usage_a`。
- **L3403 EN**: Initializes variable `nargs` from the right-hand expression.
  **L3403 CN**: 使用右侧表达式初始化变量 `nargs`。
- **L3404 EN**: Initializes variable `cta_group` from the right-hand expression.
  **L3404 CN**: 使用右侧表达式初始化变量 `cta_group`。
- **L3405 EN**: Initializes variable `collector_usage` from the right-hand expression.
  **L3405 CN**: 使用右侧表达式初始化变量 `collector_usage`。
- **L3406 EN**: Blank line separating nearby declarations or logic blocks.
  **L3406 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3407 EN**: Begins a declarative `if` clause controlling subsequent TableGen or C++ entries.
  **L3407 CN**: 开始一个声明式 `if` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3408 EN**: Declares TableGen def `mma.record_name`.
  **L3408 CN**: 声明 TableGen def `mma.record_name`。
- **L3409 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `!listconcat(mma.common_intr_props,`.
  **L3409 CN**: 继续一个多行参数列表、初始化器或聚合项：`!listconcat(mma.common_intr_props,`。
- **L3410 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<cta_group, 1, 3>,`.
  **L3410 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<cta_group, 1, 3>,`。
- **L3411 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<collector_usage, 0, 4>]),`.
  **L3411 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<collector_usage, 0, 4>]),`。
- **L3412 EN**: Executes a standalone statement or declaration: `mma.intr_name>;`.
  **L3412 CN**: 执行一条独立语句或声明：`mma.intr_name>;`。
- **L3413 EN**: Closes the current lexical scope or compound statement.
  **L3413 CN**: 结束当前词法作用域或复合语句块。
- **L3414 EN**: Continues the surrounding expression or declaration: `} // scale_vec_size`.
  **L3414 CN**: 继续构造周围的表达式或声明：`} // scale_vec_size`。
- **L3415 EN**: Continues the surrounding expression or declaration: `} // kind`.
  **L3415 CN**: 继续构造周围的表达式或声明：`} // kind`。
- **L3416 EN**: Continues the surrounding expression or declaration: `} // space`.
  **L3416 CN**: 继续构造周围的表达式或声明：`} // space`。
- **L3417 EN**: Continues the surrounding expression or declaration: `} // sparse`.
  **L3417 CN**: 继续构造周围的表达式或声明：`} // sparse`。
- **L3418 EN**: Blank line separating nearby declarations or logic blocks.
  **L3418 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3419 EN**: Separator comment used for visual grouping.
  **L3419 CN**: 用于视觉分组的分隔注释。
- **L3420 EN**: Comment explains nearby logic, invariants, or intent: `tcgen05.mma ws intrinsics`.
  **L3420 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tcgen05.mma ws intrinsics`。
- **L3421 EN**: Separator comment used for visual grouping.
  **L3421 CN**: 用于视觉分组的分隔注释。
- **L3422 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3422 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3423 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3423 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3424 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3424 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。

### Lines 3425-3456

````tablegen
      defvar mma = NVVM_TCGEN05_MMA_WS<sparse, space, zero_col_mask>;
      defvar args = !listconcat(
        mma.common_args,
        !if(!eq(zero_col_mask, 1), [llvm_i64_ty], [])
      );
      defvar flags = [llvm_i32_ty,        // kind
                      llvm_i32_ty,        // collector_buffer_b
                      llvm_i32_ty];       // collector_usage_b_op
      defvar nargs = !size(args);
      defvar intrinsic_properties = !listconcat(
        mma.common_intr_props,
        [Range<ArgIndex<nargs>, 0, 4>,
         Range<ArgIndex<!add(nargs, 1)>, 0, 4>,
         Range<ArgIndex<!add(nargs, 2)>, 0, 4>]
      );

      def mma.record_name:
            DefaultAttrsIntrinsicFlags<[], args, flags, intrinsic_properties,
              mma.intr_name>;
    } // zero_col_mask
  } // space
} // sparse

//
// tensormap.replace intrinsics
//

let IntrProperties = [IntrArgMemOnly, IntrWriteMem, NoCapture<ArgIndex<0>>] in {
  def int_nvvm_tensormap_replace_global_address :
    DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i64_ty], []>;
  def int_nvvm_tensormap_replace_rank :
    DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], []>;
````
- **L3425 EN**: Initializes variable `mma` from the right-hand expression.
  **L3425 CN**: 使用右侧表达式初始化变量 `mma`。
- **L3426 EN**: Continues logic associated with callable symbol `listconcat`.
  **L3426 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L3427 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mma.common_args,`.
  **L3427 CN**: 继续一个多行参数列表、初始化器或聚合项：`mma.common_args,`。
- **L3428 EN**: Continues the surrounding expression or declaration: `!if(!eq(zero_col_mask, 1), [llvm_i64_ty], [])`.
  **L3428 CN**: 继续构造周围的表达式或声明：`!if(!eq(zero_col_mask, 1), [llvm_i64_ty], [])`。
- **L3429 EN**: Executes a standalone statement or declaration: `);`.
  **L3429 CN**: 执行一条独立语句或声明：`);`。
- **L3430 EN**: Continues the surrounding expression or declaration: `defvar flags = [llvm_i32_ty,        // kind`.
  **L3430 CN**: 继续构造周围的表达式或声明：`defvar flags = [llvm_i32_ty,        // kind`。
- **L3431 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty,        // collector_buffer_b`.
  **L3431 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty,        // collector_buffer_b`。
- **L3432 EN**: Continues the surrounding expression or declaration: `llvm_i32_ty];       // collector_usage_b_op`.
  **L3432 CN**: 继续构造周围的表达式或声明：`llvm_i32_ty];       // collector_usage_b_op`。
- **L3433 EN**: Initializes variable `nargs` from the right-hand expression.
  **L3433 CN**: 使用右侧表达式初始化变量 `nargs`。
- **L3434 EN**: Continues logic associated with callable symbol `listconcat`.
  **L3434 CN**: 继续与可调用符号 `listconcat` 相关的逻辑。
- **L3435 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `mma.common_intr_props,`.
  **L3435 CN**: 继续一个多行参数列表、初始化器或聚合项：`mma.common_intr_props,`。
- **L3436 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[Range<ArgIndex<nargs>, 0, 4>,`.
  **L3436 CN**: 继续一个多行参数列表、初始化器或聚合项：`[Range<ArgIndex<nargs>, 0, 4>,`。
- **L3437 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<ArgIndex<!add(nargs, 1)>, 0, 4>,`.
  **L3437 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<ArgIndex<!add(nargs, 1)>, 0, 4>,`。
- **L3438 EN**: Continues logic associated with callable symbol `add`.
  **L3438 CN**: 继续与可调用符号 `add` 相关的逻辑。
- **L3439 EN**: Executes a standalone statement or declaration: `);`.
  **L3439 CN**: 执行一条独立语句或声明：`);`。
- **L3440 EN**: Blank line separating nearby declarations or logic blocks.
  **L3440 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3441 EN**: Declares TableGen def `mma.record_name`.
  **L3441 CN**: 声明 TableGen def `mma.record_name`。
- **L3442 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsicFlags<[], args, flags, intrinsic_properties,`.
  **L3442 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsicFlags<[], args, flags, intrinsic_properties,`。
- **L3443 EN**: Executes a standalone statement or declaration: `mma.intr_name>;`.
  **L3443 CN**: 执行一条独立语句或声明：`mma.intr_name>;`。
- **L3444 EN**: Continues the surrounding expression or declaration: `} // zero_col_mask`.
  **L3444 CN**: 继续构造周围的表达式或声明：`} // zero_col_mask`。
- **L3445 EN**: Continues the surrounding expression or declaration: `} // space`.
  **L3445 CN**: 继续构造周围的表达式或声明：`} // space`。
- **L3446 EN**: Continues the surrounding expression or declaration: `} // sparse`.
  **L3446 CN**: 继续构造周围的表达式或声明：`} // sparse`。
- **L3447 EN**: Blank line separating nearby declarations or logic blocks.
  **L3447 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3448 EN**: Separator comment used for visual grouping.
  **L3448 CN**: 用于视觉分组的分隔注释。
- **L3449 EN**: Comment explains nearby logic, invariants, or intent: `tensormap.replace intrinsics`.
  **L3449 CN**: 注释说明了附近代码的逻辑、不变式或设计意图：`tensormap.replace intrinsics`。
- **L3450 EN**: Separator comment used for visual grouping.
  **L3450 CN**: 用于视觉分组的分隔注释。
- **L3451 EN**: Blank line separating nearby declarations or logic blocks.
  **L3451 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3452 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3452 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3453 EN**: Declares TableGen def `int_nvvm_tensormap_replace_global_address`.
  **L3453 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_global_address`。
- **L3454 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i64_ty], []>;`.
  **L3454 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i64_ty], []>;`。
- **L3455 EN**: Declares TableGen def `int_nvvm_tensormap_replace_rank`.
  **L3455 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_rank`。
- **L3456 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], []>;`.
  **L3456 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], []>;`。

### Lines 3457-3488

````tablegen
}

let IntrProperties = [IntrArgMemOnly, ImmArg<ArgIndex<1>>, IntrWriteMem,
                      NoCapture<ArgIndex<0>>] in {
  def int_nvvm_tensormap_replace_global_stride :
    DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty, llvm_i64_ty], []>;
  foreach tmap_field = ["box_dim", "global_dim", "element_stride"] in {
    def int_nvvm_tensormap_replace_ # tmap_field :
      DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty, llvm_i32_ty], []>;
  }
}

def int_nvvm_tensormap_replace_elemtype : 
  DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], 
    [IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,
     Range<ArgIndex<1>, 0, 16>,
     ArgInfo<ArgIndex<1>, [ArgName<"elemtype">, 
                           ImmArgPrinter<"printTensormapElemType">]>]>;
def int_nvvm_tensormap_replace_interleave_layout :
  DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], 
    [IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,
     Range<ArgIndex<1>, 0, 3>,
     ArgInfo<ArgIndex<1>, [ArgName<"interleave_layout">, 
                           ImmArgPrinter<"printTensormapInterleaveLayout">]>]>;
def int_nvvm_tensormap_replace_swizzle_mode :
  DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], 
    [IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,
     Range<ArgIndex<1>, 0, 5>,
     ArgInfo<ArgIndex<1>, [ArgName<"swizzle_mode">, 
                           ImmArgPrinter<"printTensormapSwizzleMode">]>]>;
def int_nvvm_tensormap_replace_swizzle_atomicity :
  DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], 
````
- **L3457 EN**: Closes the current lexical scope or compound statement.
  **L3457 CN**: 结束当前词法作用域或复合语句块。
- **L3458 EN**: Blank line separating nearby declarations or logic blocks.
  **L3458 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3459 EN**: Begins a declarative `let` clause controlling subsequent TableGen or C++ entries.
  **L3459 CN**: 开始一个声明式 `let` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3460 EN**: Continues the surrounding expression or declaration: `NoCapture<ArgIndex<0>>] in {`.
  **L3460 CN**: 继续构造周围的表达式或声明：`NoCapture<ArgIndex<0>>] in {`。
- **L3461 EN**: Declares TableGen def `int_nvvm_tensormap_replace_global_stride`.
  **L3461 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_global_stride`。
- **L3462 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty, llvm_i64_ty], []>;`.
  **L3462 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty, llvm_i64_ty], []>;`。
- **L3463 EN**: Begins a declarative `foreach` clause controlling subsequent TableGen or C++ entries.
  **L3463 CN**: 开始一个声明式 `foreach` 子句，用于控制后续 TableGen 或 C++ 条目。
- **L3464 EN**: Declares TableGen def `int_nvvm_tensormap_replace_`.
  **L3464 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_`。
- **L3465 EN**: Executes a standalone statement or declaration: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty, llvm_i32_ty], []>;`.
  **L3465 CN**: 执行一条独立语句或声明：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty, llvm_i32_ty], []>;`。
- **L3466 EN**: Closes the current lexical scope or compound statement.
  **L3466 CN**: 结束当前词法作用域或复合语句块。
- **L3467 EN**: Closes the current lexical scope or compound statement.
  **L3467 CN**: 结束当前词法作用域或复合语句块。
- **L3468 EN**: Blank line separating nearby declarations or logic blocks.
  **L3468 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3469 EN**: Declares TableGen def `int_nvvm_tensormap_replace_elemtype`.
  **L3469 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_elemtype`。
- **L3470 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L3470 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`。
- **L3471 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`.
  **L3471 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`。
- **L3472 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<ArgIndex<1>, 0, 16>,`.
  **L3472 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<ArgIndex<1>, 0, 16>,`。
- **L3473 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArgInfo<ArgIndex<1>, [ArgName<"elemtype">,`.
  **L3473 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArgInfo<ArgIndex<1>, [ArgName<"elemtype">,`。
- **L3474 EN**: Executes a standalone statement or declaration: `ImmArgPrinter<"printTensormapElemType">]>]>;`.
  **L3474 CN**: 执行一条独立语句或声明：`ImmArgPrinter<"printTensormapElemType">]>]>;`。
- **L3475 EN**: Declares TableGen def `int_nvvm_tensormap_replace_interleave_layout`.
  **L3475 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_interleave_layout`。
- **L3476 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L3476 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`。
- **L3477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`.
  **L3477 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`。
- **L3478 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<ArgIndex<1>, 0, 3>,`.
  **L3478 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<ArgIndex<1>, 0, 3>,`。
- **L3479 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArgInfo<ArgIndex<1>, [ArgName<"interleave_layout">,`.
  **L3479 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArgInfo<ArgIndex<1>, [ArgName<"interleave_layout">,`。
- **L3480 EN**: Executes a standalone statement or declaration: `ImmArgPrinter<"printTensormapInterleaveLayout">]>]>;`.
  **L3480 CN**: 执行一条独立语句或声明：`ImmArgPrinter<"printTensormapInterleaveLayout">]>]>;`。
- **L3481 EN**: Declares TableGen def `int_nvvm_tensormap_replace_swizzle_mode`.
  **L3481 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_swizzle_mode`。
- **L3482 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L3482 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`。
- **L3483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`.
  **L3483 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`。
- **L3484 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<ArgIndex<1>, 0, 5>,`.
  **L3484 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<ArgIndex<1>, 0, 5>,`。
- **L3485 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArgInfo<ArgIndex<1>, [ArgName<"swizzle_mode">,`.
  **L3485 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArgInfo<ArgIndex<1>, [ArgName<"swizzle_mode">,`。
- **L3486 EN**: Executes a standalone statement or declaration: `ImmArgPrinter<"printTensormapSwizzleMode">]>]>;`.
  **L3486 CN**: 执行一条独立语句或声明：`ImmArgPrinter<"printTensormapSwizzleMode">]>]>;`。
- **L3487 EN**: Declares TableGen def `int_nvvm_tensormap_replace_swizzle_atomicity`.
  **L3487 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_swizzle_atomicity`。
- **L3488 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L3488 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`。

### Lines 3489-3500

````tablegen
    [IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,
     Range<ArgIndex<1>, 0, 4>,
     ArgInfo<ArgIndex<1>, [ArgName<"swizzle_atomicity">, 
                           ImmArgPrinter<"printTensormapSwizzleAtomicity">]>]>;
def int_nvvm_tensormap_replace_fill_mode :
  DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty], 
    [IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,
     Range<ArgIndex<1>, 0, 2>,
     ArgInfo<ArgIndex<1>, [ArgName<"fill_mode">, 
                           ImmArgPrinter<"printTensormapFillMode">]>]>;

} // let TargetPrefix = "nvvm"
````
- **L3489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`.
  **L3489 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`。
- **L3490 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<ArgIndex<1>, 0, 4>,`.
  **L3490 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<ArgIndex<1>, 0, 4>,`。
- **L3491 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArgInfo<ArgIndex<1>, [ArgName<"swizzle_atomicity">,`.
  **L3491 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArgInfo<ArgIndex<1>, [ArgName<"swizzle_atomicity">,`。
- **L3492 EN**: Executes a standalone statement or declaration: `ImmArgPrinter<"printTensormapSwizzleAtomicity">]>]>;`.
  **L3492 CN**: 执行一条独立语句或声明：`ImmArgPrinter<"printTensormapSwizzleAtomicity">]>]>;`。
- **L3493 EN**: Declares TableGen def `int_nvvm_tensormap_replace_fill_mode`.
  **L3493 CN**: 声明 TableGen def `int_nvvm_tensormap_replace_fill_mode`。
- **L3494 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`.
  **L3494 CN**: 继续一个多行参数列表、初始化器或聚合项：`DefaultAttrsIntrinsic<[], [llvm_anyptr_ty, llvm_i32_ty],`。
- **L3495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`.
  **L3495 CN**: 继续一个多行参数列表、初始化器或聚合项：`[IntrArgMemOnly, IntrWriteMem, ImmArg<ArgIndex<1>>, NoCapture<ArgIndex<0>>,`。
- **L3496 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Range<ArgIndex<1>, 0, 2>,`.
  **L3496 CN**: 继续一个多行参数列表、初始化器或聚合项：`Range<ArgIndex<1>, 0, 2>,`。
- **L3497 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `ArgInfo<ArgIndex<1>, [ArgName<"fill_mode">,`.
  **L3497 CN**: 继续一个多行参数列表、初始化器或聚合项：`ArgInfo<ArgIndex<1>, [ArgName<"fill_mode">,`。
- **L3498 EN**: Executes a standalone statement or declaration: `ImmArgPrinter<"printTensormapFillMode">]>]>;`.
  **L3498 CN**: 执行一条独立语句或声明：`ImmArgPrinter<"printTensormapFillMode">]>]>;`。
- **L3499 EN**: Blank line separating nearby declarations or logic blocks.
  **L3499 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3500 EN**: Continues the surrounding expression or declaration: `} // let TargetPrefix = "nvvm"`.
  **L3500 CN**: 继续构造周围的表达式或声明：`} // let TargetPrefix = "nvvm"`。

## Key Concepts / 关键概念

- **TableGen record modeling / TableGen 记录建模**
- **Function-level IR management / 函数级 IR 管理**
- **Instruction semantics / 指令语义**
- **Intrinsic declaration and lowering hooks / Intrinsic 声明与降级钩子**
- **Metadata representation / 元数据表示**
- **Typed error propagation / 类型化错误传播**
- **Target intrinsic catalogs / 目标 Intrinsic 目录**

## Dependencies / 依赖关系

- No direct include-style dependencies appear in this file. / 该文件中没有直接出现 include 风格依赖。
