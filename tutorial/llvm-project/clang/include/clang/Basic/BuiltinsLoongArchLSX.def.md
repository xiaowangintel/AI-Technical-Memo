# BuiltinsLoongArchLSX.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsLoongArchLSX.def`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: Declares foundational compiler metadata, diagnostics, target descriptions, or builtin-related definitions for `BuiltinsLoongArchLSX`.
- **Purpose (CN)**: 声明与 `BuiltinsLoongArchLSX` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 959

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//=============------------- BuiltinsLoongArchLSX.def --------------- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the LoongArch-specific LSX builtin function database.
// Users of this file must define the BUILTIN macro to make use of this
// information.
//
//===----------------------------------------------------------------------===//

TARGET_BUILTIN(__builtin_lsx_vadd_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vadd_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vadd_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vadd_d, "V2LLiV2LLiV2LLi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vadd_q, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsub_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsub_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsub_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsub_d, "V2LLiV2LLiV2LLi", "nc", "lsx")
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the LoongArch-specific LSX builtin function database.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the LoongArch-specific LSX builtin function database.`。
- **L10 EN**: Comment explains nearby logic, constraints, or intent: `Users of this file must define the BUILTIN macro to make use of this`.
  **L10 CN**: 注释解释附近代码的逻辑、约束或设计意图：`Users of this file must define the BUILTIN macro to make use of this`。
- **L11 EN**: Comment explains nearby logic, constraints, or intent: `information.`.
  **L11 CN**: 注释解释附近代码的逻辑、约束或设计意图：`information.`。
- **L12 EN**: Separator comment used for visual grouping.
  **L12 CN**: 用于视觉分组的分隔注释。
- **L13 EN**: Banner comment marking a file or section boundary.
  **L13 CN**: 横幅注释，用于标记文件或章节边界。
- **L14 EN**: Blank line separating nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L15 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L15 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L16 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L16 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L17 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L17 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L18 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L18 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L19 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L19 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L20 EN**: Blank line separating nearby declarations or logic blocks.
  **L20 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L21 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L21 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L22 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L22 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L23 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L23 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L24 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L24 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 25-48

````cpp
TARGET_BUILTIN(__builtin_lsx_vsub_q, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vaddi_bu, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddi_hu, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddi_wu, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddi_du, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsubi_bu, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubi_hu, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubi_wu, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubi_du, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vneg_b, "V16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vneg_h, "V8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vneg_w, "V4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vneg_d, "V2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsadd_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsadd_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsadd_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsadd_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsadd_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsadd_hu, "V8UsV8UsV8Us", "nc", "lsx")
````
- **L25 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L25 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L26 EN**: Blank line separating nearby declarations or logic blocks.
  **L26 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L27 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L27 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L28 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L28 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L29 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L29 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L30 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L30 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L31 EN**: Blank line separating nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L32 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L32 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L33 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L33 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L34 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L34 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L35 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L35 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L36 EN**: Blank line separating nearby declarations or logic blocks.
  **L36 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L37 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L37 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L38 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L38 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L39 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L39 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L40 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L40 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L41 EN**: Blank line separating nearby declarations or logic blocks.
  **L41 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L42 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L42 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L43 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L43 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L44 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L44 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L45 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L45 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L46 EN**: Blank line separating nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L47 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L47 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L48 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L48 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 49-72

````cpp
TARGET_BUILTIN(__builtin_lsx_vsadd_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsadd_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssub_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssub_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssub_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssub_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssub_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssub_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssub_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssub_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vhaddw_h_b, "V8SsV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhaddw_w_h, "V4SiV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhaddw_d_w, "V2SLLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhaddw_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vhaddw_hu_bu, "V8UsV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhaddw_wu_hu, "V4UiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhaddw_du_wu, "V2ULLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhaddw_qu_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vhsubw_h_b, "V8SsV16ScV16Sc", "nc", "lsx")
````
- **L49 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L49 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L50 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L50 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L51 EN**: Blank line separating nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L52 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L52 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L53 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L53 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L54 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L54 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L55 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L55 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L56 EN**: Blank line separating nearby declarations or logic blocks.
  **L56 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L57 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L57 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L58 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L58 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L59 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L59 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L60 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L60 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L61 EN**: Blank line separating nearby declarations or logic blocks.
  **L61 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L62 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L62 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L63 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L63 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L64 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L64 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L65 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L65 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L66 EN**: Blank line separating nearby declarations or logic blocks.
  **L66 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L67 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L67 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L68 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L68 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L69 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L69 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L70 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L70 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L71 EN**: Blank line separating nearby declarations or logic blocks.
  **L71 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L72 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L72 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 73-96

````cpp
TARGET_BUILTIN(__builtin_lsx_vhsubw_w_h, "V4SiV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhsubw_d_w, "V2SLLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhsubw_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vhsubw_hu_bu, "V8UsV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhsubw_wu_hu, "V4UiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhsubw_du_wu, "V2ULLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vhsubw_qu_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vaddwev_h_b, "V8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_w_h, "V4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_d_w, "V2LLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vaddwod_h_b, "V8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_w_h, "V4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_d_w, "V2LLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsubwev_h_b, "V8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwev_w_h, "V4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwev_d_w, "V2LLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwev_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

````
- **L73 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L73 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L74 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L74 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L75 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L75 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L76 EN**: Blank line separating nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L77 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L77 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L78 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L78 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L79 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L79 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L80 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L80 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L81 EN**: Blank line separating nearby declarations or logic blocks.
  **L81 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L82 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L82 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L83 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L83 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L84 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L84 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L85 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L85 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L86 EN**: Blank line separating nearby declarations or logic blocks.
  **L86 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L87 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L87 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L88 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L88 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L89 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L89 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L90 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L90 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L91 EN**: Blank line separating nearby declarations or logic blocks.
  **L91 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L92 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L92 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L93 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L93 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L94 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L94 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L95 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L95 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L96 EN**: Blank line separating nearby declarations or logic blocks.
  **L96 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 97-120

````cpp
TARGET_BUILTIN(__builtin_lsx_vsubwod_h_b, "V8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwod_w_h, "V4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwod_d_w, "V2LLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwod_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vaddwev_h_bu, "V8sV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_w_hu, "V4SiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_d_wu, "V2LLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_q_du, "V2LLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vaddwod_h_bu, "V8sV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_w_hu, "V4SiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_d_wu, "V2LLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_q_du, "V2LLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsubwev_h_bu, "V8sV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwev_w_hu, "V4SiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwev_d_wu, "V2LLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwev_q_du, "V2LLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsubwod_h_bu, "V8sV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwod_w_hu, "V4SiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwod_d_wu, "V2LLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsubwod_q_du, "V2LLiV2ULLiV2ULLi", "nc", "lsx")
````
- **L97 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L97 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L98 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L98 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L99 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L99 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L100 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L100 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L101 EN**: Blank line separating nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L102 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L102 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L103 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L103 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L104 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L104 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L105 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L105 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L107 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L107 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L108 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L108 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L109 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L109 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L110 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L110 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L112 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L112 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L113 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L113 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L114 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L114 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L115 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L115 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L116 EN**: Blank line separating nearby declarations or logic blocks.
  **L116 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L117 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L117 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L118 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L118 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L119 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L119 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L120 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L120 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 121-144

````cpp

TARGET_BUILTIN(__builtin_lsx_vaddwev_h_bu_b, "V8sV16UcV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_w_hu_h, "V4SiV8UsV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_d_wu_w, "V2LLiV4UiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwev_q_du_d, "V2LLiV2ULLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vaddwod_h_bu_b, "V8sV16UcV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_w_hu_h, "V4SiV8UsV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_d_wu_w, "V2LLiV4UiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vaddwod_q_du_d, "V2LLiV2ULLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vavg_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavg_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavg_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavg_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vavg_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavg_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavg_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavg_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vavgr_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavgr_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavgr_w, "V4SiV4SiV4Si", "nc", "lsx")
````
- **L121 EN**: Blank line separating nearby declarations or logic blocks.
  **L121 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L122 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L122 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L123 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L123 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L124 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L124 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L125 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L125 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L126 EN**: Blank line separating nearby declarations or logic blocks.
  **L126 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L127 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L127 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L128 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L128 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L129 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L129 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L130 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L130 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L131 EN**: Blank line separating nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L132 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L132 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L133 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L133 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L134 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L134 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L135 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L135 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L136 EN**: Blank line separating nearby declarations or logic blocks.
  **L136 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L137 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L137 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L138 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L138 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L139 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L139 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L140 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L140 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L141 EN**: Blank line separating nearby declarations or logic blocks.
  **L141 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L142 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L142 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L143 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L143 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L144 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L144 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 145-168

````cpp
TARGET_BUILTIN(__builtin_lsx_vavgr_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vavgr_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavgr_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavgr_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vavgr_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vabsd_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vabsd_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vabsd_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vabsd_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vabsd_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vabsd_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vabsd_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vabsd_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vadda_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vadda_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vadda_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vadda_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmax_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmax_h, "V8SsV8SsV8Ss", "nc", "lsx")
````
- **L145 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L145 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L146 EN**: Blank line separating nearby declarations or logic blocks.
  **L146 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L147 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L147 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L148 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L148 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L149 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L149 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L150 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L150 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L151 EN**: Blank line separating nearby declarations or logic blocks.
  **L151 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L152 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L152 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L153 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L153 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L154 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L154 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L155 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L155 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L156 EN**: Blank line separating nearby declarations or logic blocks.
  **L156 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L157 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L157 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L158 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L158 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L159 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L159 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L160 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L160 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L161 EN**: Blank line separating nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L162 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L162 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L163 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L163 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L164 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L164 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L165 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L165 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L166 EN**: Blank line separating nearby declarations or logic blocks.
  **L166 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L167 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L167 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L168 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L168 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 169-192

````cpp
TARGET_BUILTIN(__builtin_lsx_vmax_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmax_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaxi_b, "V16ScV16ScIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaxi_h, "V8SsV8SsIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaxi_w, "V4SiV4SiIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaxi_d, "V2SLLiV2SLLiIi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmax_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmax_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmax_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmax_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaxi_bu, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaxi_hu, "V8UsV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaxi_wu, "V4UiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaxi_du, "V2ULLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmin_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmin_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmin_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmin_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmini_b, "V16ScV16ScIi", "nc", "lsx")
````
- **L169 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L169 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L170 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L170 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L171 EN**: Blank line separating nearby declarations or logic blocks.
  **L171 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L172 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L172 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L173 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L173 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L174 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L174 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L175 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L175 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L176 EN**: Blank line separating nearby declarations or logic blocks.
  **L176 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L177 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L177 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L178 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L178 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L179 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L179 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L180 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L180 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L181 EN**: Blank line separating nearby declarations or logic blocks.
  **L181 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L182 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L182 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L183 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L183 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L184 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L184 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L185 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L185 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L186 EN**: Blank line separating nearby declarations or logic blocks.
  **L186 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L187 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L187 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L188 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L188 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L189 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L189 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L190 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L190 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L191 EN**: Blank line separating nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L192 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L192 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 193-216

````cpp
TARGET_BUILTIN(__builtin_lsx_vmini_h, "V8SsV8SsIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmini_w, "V4SiV4SiIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmini_d, "V2SLLiV2SLLiIi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmin_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmin_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmin_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmin_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmini_bu, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmini_hu, "V8UsV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmini_wu, "V4UiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmini_du, "V2ULLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmul_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmul_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmul_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmul_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmuh_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmuh_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmuh_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmuh_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

````
- **L193 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L193 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L194 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L194 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L195 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L195 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L196 EN**: Blank line separating nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L197 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L197 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L198 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L198 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L199 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L199 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L200 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L200 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L201 EN**: Blank line separating nearby declarations or logic blocks.
  **L201 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L202 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L202 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L203 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L203 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L204 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L204 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L205 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L205 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L206 EN**: Blank line separating nearby declarations or logic blocks.
  **L206 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L207 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L207 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L208 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L208 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L209 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L209 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L210 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L210 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L211 EN**: Blank line separating nearby declarations or logic blocks.
  **L211 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L212 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L212 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L213 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L213 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L214 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L214 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L215 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L215 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L216 EN**: Blank line separating nearby declarations or logic blocks.
  **L216 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 217-240

````cpp
TARGET_BUILTIN(__builtin_lsx_vmuh_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmuh_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmuh_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmuh_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmulwev_h_b, "V8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_w_h, "V4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_d_w, "V2LLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmulwod_h_b, "V8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_w_h, "V4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_d_w, "V2LLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_q_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmulwev_h_bu, "V8sV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_w_hu, "V4SiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_d_wu, "V2LLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_q_du, "V2LLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmulwod_h_bu, "V8sV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_w_hu, "V4SiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_d_wu, "V2LLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_q_du, "V2LLiV2ULLiV2ULLi", "nc", "lsx")
````
- **L217 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L217 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L218 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L218 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L219 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L219 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L220 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L220 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L221 EN**: Blank line separating nearby declarations or logic blocks.
  **L221 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L222 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L222 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L223 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L223 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L224 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L224 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L225 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L225 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L226 EN**: Blank line separating nearby declarations or logic blocks.
  **L226 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L227 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L227 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L228 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L228 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L229 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L229 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L230 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L230 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L231 EN**: Blank line separating nearby declarations or logic blocks.
  **L231 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L232 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L232 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L233 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L233 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L234 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L234 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L235 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L235 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L236 EN**: Blank line separating nearby declarations or logic blocks.
  **L236 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L237 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L237 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L238 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L238 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L239 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L239 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L240 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L240 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 241-264

````cpp

TARGET_BUILTIN(__builtin_lsx_vmulwev_h_bu_b, "V8sV16UcV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_w_hu_h, "V4SiV8UsV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_d_wu_w, "V2LLiV4UiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwev_q_du_d, "V2LLiV2ULLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmulwod_h_bu_b, "V8sV16UcV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_w_hu_h, "V4SiV8UsV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_d_wu_w, "V2LLiV4UiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmulwod_q_du_d, "V2LLiV2ULLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmadd_b, "V16ScV16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmadd_h, "V8SsV8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmadd_w, "V4SiV4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmadd_d, "V2SLLiV2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmsub_b, "V16ScV16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmsub_h, "V8SsV8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmsub_w, "V4SiV4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmsub_d, "V2SLLiV2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaddwev_h_b, "V8sV8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_w_h, "V4SiV4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_d_w, "V2LLiV2LLiV4SiV4Si", "nc", "lsx")
````
- **L241 EN**: Blank line separating nearby declarations or logic blocks.
  **L241 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L242 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L242 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L243 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L243 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L244 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L244 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L245 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L245 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L246 EN**: Blank line separating nearby declarations or logic blocks.
  **L246 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L247 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L247 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L248 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L248 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L249 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L249 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L250 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L250 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L251 EN**: Blank line separating nearby declarations or logic blocks.
  **L251 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L252 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L252 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L253 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L253 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L254 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L254 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L255 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L255 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L256 EN**: Blank line separating nearby declarations or logic blocks.
  **L256 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L257 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L257 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L258 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L258 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L259 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L259 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L260 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L260 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L261 EN**: Blank line separating nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L262 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L262 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L263 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L263 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L264 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L264 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 265-288

````cpp
TARGET_BUILTIN(__builtin_lsx_vmaddwev_q_d, "V2LLiV2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaddwod_h_b, "V8sV8sV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_w_h, "V4SiV4SiV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_d_w, "V2LLiV2LLiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_q_d, "V2LLiV2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaddwev_h_bu, "V8UsV8UsV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_w_hu, "V4UiV4UiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_d_wu, "V2ULLiV2ULLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_q_du, "V2ULLiV2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaddwod_h_bu, "V8UsV8UsV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_w_hu, "V4UiV4UiV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_d_wu, "V2ULLiV2ULLiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_q_du, "V2ULLiV2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaddwev_h_bu_b, "V8sV8sV16UcV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_w_hu_h, "V4SiV4SiV8UsV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_d_wu_w, "V2LLiV2LLiV4UiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwev_q_du_d, "V2LLiV2LLiV2ULLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmaddwod_h_bu_b, "V8sV8sV16UcV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_w_hu_h, "V4SiV4SiV8UsV8s", "nc", "lsx")
````
- **L265 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L265 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L266 EN**: Blank line separating nearby declarations or logic blocks.
  **L266 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L267 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L267 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L268 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L268 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L269 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L269 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L270 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L270 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L271 EN**: Blank line separating nearby declarations or logic blocks.
  **L271 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L272 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L272 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L273 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L273 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L274 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L274 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L275 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L275 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L276 EN**: Blank line separating nearby declarations or logic blocks.
  **L276 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L277 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L277 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L278 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L278 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L279 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L279 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L280 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L280 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L281 EN**: Blank line separating nearby declarations or logic blocks.
  **L281 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L282 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L282 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L283 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L283 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L284 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L284 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L285 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L285 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L286 EN**: Blank line separating nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L287 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L287 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L288 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L288 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 289-312

````cpp
TARGET_BUILTIN(__builtin_lsx_vmaddwod_d_wu_w, "V2LLiV2LLiV4UiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmaddwod_q_du_d, "V2LLiV2LLiV2ULLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vdiv_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vdiv_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vdiv_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vdiv_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vdiv_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vdiv_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vdiv_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vdiv_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmod_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmod_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmod_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmod_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")


TARGET_BUILTIN(__builtin_lsx_vmod_bu, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmod_hu, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmod_wu, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmod_du, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

````
- **L289 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L289 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L290 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L290 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L291 EN**: Blank line separating nearby declarations or logic blocks.
  **L291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L292 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L292 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L293 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L293 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L294 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L294 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L295 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L295 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L296 EN**: Blank line separating nearby declarations or logic blocks.
  **L296 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L297 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L297 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L298 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L298 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L299 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L299 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L300 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L300 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L301 EN**: Blank line separating nearby declarations or logic blocks.
  **L301 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L302 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L302 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L303 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L303 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L304 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L304 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L305 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L305 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L306 EN**: Blank line separating nearby declarations or logic blocks.
  **L306 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L307 EN**: Blank line separating nearby declarations or logic blocks.
  **L307 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L308 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L308 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L309 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L309 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L310 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L310 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L311 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L311 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L312 EN**: Blank line separating nearby declarations or logic blocks.
  **L312 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 313-336

````cpp
TARGET_BUILTIN(__builtin_lsx_vsat_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsat_h, "V8SsV8SsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsat_w, "V4SiV4SiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsat_d, "V2SLLiV2SLLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsat_bu, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsat_hu, "V8UsV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsat_wu, "V4UiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsat_du, "V2ULLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vexth_h_b, "V8sV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vexth_w_h, "V4SiV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vexth_d_w, "V2LLiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vexth_q_d, "V2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vexth_hu_bu, "V8UsV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vexth_wu_hu, "V4UiV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vexth_du_wu, "V2ULLiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vexth_qu_du, "V2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsigncov_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsigncov_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsigncov_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsigncov_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")
````
- **L313 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L313 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L314 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L314 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L315 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L315 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L316 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L316 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L317 EN**: Blank line separating nearby declarations or logic blocks.
  **L317 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L318 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L318 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L319 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L319 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L320 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L320 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L321 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L321 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L322 EN**: Blank line separating nearby declarations or logic blocks.
  **L322 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L323 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L323 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L324 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L324 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L325 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L325 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L326 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L326 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L327 EN**: Blank line separating nearby declarations or logic blocks.
  **L327 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L328 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L328 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L329 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L329 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L330 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L330 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L331 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L331 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L332 EN**: Blank line separating nearby declarations or logic blocks.
  **L332 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L333 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L333 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L334 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L334 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L335 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L335 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L336 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L336 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 337-360

````cpp

TARGET_BUILTIN(__builtin_lsx_vmskltz_b, "V16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmskltz_h, "V8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmskltz_w, "V4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmskltz_d, "V2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vmskgez_b, "V16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vmsknz_b, "V16ScV16Sc", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vldi, "V2LLiIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrepli_b, "V16ScIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrepli_h, "V8sIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrepli_w, "V4iIi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrepli_d, "V2LLiIi", "nc", "lsx")


TARGET_BUILTIN(__builtin_lsx_vand_v, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vor_v, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vxor_v, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vnor_v, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vandn_v, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vorn_v, "V16UcV16UcV16Uc", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vandi_b, "V16UcV16UcIUi", "nc", "lsx")
````
- **L337 EN**: Blank line separating nearby declarations or logic blocks.
  **L337 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L338 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L338 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L339 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L339 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L340 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L340 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L341 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L341 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L342 EN**: Blank line separating nearby declarations or logic blocks.
  **L342 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L343 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L343 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L344 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L344 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L346 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L347 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L347 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L348 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L348 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L349 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L349 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L350 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L350 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L351 EN**: Blank line separating nearby declarations or logic blocks.
  **L351 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L352 EN**: Blank line separating nearby declarations or logic blocks.
  **L352 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L353 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L353 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L354 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L354 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L355 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L355 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L356 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L356 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L357 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L357 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L358 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L358 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L359 EN**: Blank line separating nearby declarations or logic blocks.
  **L359 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L360 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L360 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 361-384

````cpp
TARGET_BUILTIN(__builtin_lsx_vori_b, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vxori_b, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vnori_b, "V16UcV16UcIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsll_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsll_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsll_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsll_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vslli_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslli_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslli_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslli_d, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrl_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrl_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrl_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrl_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrli_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrli_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrli_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrli_d, "V2LLiV2LLiIUi", "nc", "lsx")

````
- **L361 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L361 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L362 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L362 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L363 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L363 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L364 EN**: Blank line separating nearby declarations or logic blocks.
  **L364 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L365 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L365 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L366 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L366 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L367 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L367 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L368 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L368 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L369 EN**: Blank line separating nearby declarations or logic blocks.
  **L369 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L370 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L370 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L371 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L371 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L372 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L372 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L373 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L373 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L374 EN**: Blank line separating nearby declarations or logic blocks.
  **L374 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L375 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L375 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L376 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L376 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L377 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L377 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L378 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L378 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L379 EN**: Blank line separating nearby declarations or logic blocks.
  **L379 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L380 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L380 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L381 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L381 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L382 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L382 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L383 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L383 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L384 EN**: Blank line separating nearby declarations or logic blocks.
  **L384 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 385-408

````cpp
TARGET_BUILTIN(__builtin_lsx_vsra_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsra_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsra_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsra_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrai_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrai_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrai_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrai_d, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vrotr_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrotr_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrotr_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrotr_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vrotri_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrotri_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrotri_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vrotri_d, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsllwil_h_b, "V8sV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsllwil_w_h, "V4SiV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsllwil_d_w, "V2LLiV4SiIUi", "nc", "lsx")

````
- **L385 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L385 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L386 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L386 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L387 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L387 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L388 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L388 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L389 EN**: Blank line separating nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L390 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L390 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L391 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L391 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L392 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L392 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L393 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L393 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L394 EN**: Blank line separating nearby declarations or logic blocks.
  **L394 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L395 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L395 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L396 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L396 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L397 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L397 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L398 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L398 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L399 EN**: Blank line separating nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L400 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L400 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L401 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L401 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L402 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L402 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L403 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L403 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L404 EN**: Blank line separating nearby declarations or logic blocks.
  **L404 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L405 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L405 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L406 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L406 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L407 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L407 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L408 EN**: Blank line separating nearby declarations or logic blocks.
  **L408 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 409-432

````cpp
TARGET_BUILTIN(__builtin_lsx_vextl_q_d, "V2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsllwil_hu_bu, "V8UsV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsllwil_wu_hu, "V4UiV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsllwil_du_wu, "V2ULLiV4UiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vextl_qu_du, "V2LLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrlr_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlr_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlr_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlr_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrlri_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlri_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlri_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlri_d, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrar_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrar_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrar_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrar_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrari_b, "V16ScV16ScIUi", "nc", "lsx")
````
- **L409 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L409 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L410 EN**: Blank line separating nearby declarations or logic blocks.
  **L410 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L411 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L411 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L412 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L412 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L413 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L413 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L414 EN**: Blank line separating nearby declarations or logic blocks.
  **L414 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L415 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L415 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L416 EN**: Blank line separating nearby declarations or logic blocks.
  **L416 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L417 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L417 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L418 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L418 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L419 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L419 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L420 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L420 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L421 EN**: Blank line separating nearby declarations or logic blocks.
  **L421 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L422 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L422 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L423 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L423 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L424 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L424 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L425 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L425 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L426 EN**: Blank line separating nearby declarations or logic blocks.
  **L426 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L427 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L427 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L428 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L428 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L429 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L429 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L430 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L430 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L431 EN**: Blank line separating nearby declarations or logic blocks.
  **L431 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L432 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L432 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 433-456

````cpp
TARGET_BUILTIN(__builtin_lsx_vsrari_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrari_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrari_d, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrln_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrln_h_w, "V8sV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrln_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsran_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsran_h_w, "V8sV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsran_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrlni_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlni_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlni_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlni_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrani_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrani_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrani_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrani_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrlrn_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlrn_h_w, "V8sV4SiV4Si", "nc", "lsx")
````
- **L433 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L433 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L434 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L434 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L435 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L435 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L436 EN**: Blank line separating nearby declarations or logic blocks.
  **L436 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L437 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L437 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L438 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L438 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L439 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L439 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L440 EN**: Blank line separating nearby declarations or logic blocks.
  **L440 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L441 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L441 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L442 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L442 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L443 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L443 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L444 EN**: Blank line separating nearby declarations or logic blocks.
  **L444 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L445 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L445 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L446 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L446 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L447 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L447 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L448 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L448 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L449 EN**: Blank line separating nearby declarations or logic blocks.
  **L449 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L450 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L450 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L451 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L451 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L452 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L452 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L453 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L453 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L454 EN**: Blank line separating nearby declarations or logic blocks.
  **L454 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L455 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L455 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L456 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L456 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 457-480

````cpp
TARGET_BUILTIN(__builtin_lsx_vsrlrn_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrarn_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrarn_h_w, "V8sV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrarn_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrlrni_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlrni_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlrni_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrlrni_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsrarni_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrarni_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrarni_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsrarni_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrln_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrln_h_w, "V8sV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrln_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssran_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssran_h_w, "V8sV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssran_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

````
- **L457 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L457 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L458 EN**: Blank line separating nearby declarations or logic blocks.
  **L458 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L459 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L459 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L460 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L460 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L461 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L461 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L462 EN**: Blank line separating nearby declarations or logic blocks.
  **L462 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L463 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L463 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L464 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L464 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L465 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L465 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L466 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L466 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L467 EN**: Blank line separating nearby declarations or logic blocks.
  **L467 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L468 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L468 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L469 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L469 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L470 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L470 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L471 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L471 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L472 EN**: Blank line separating nearby declarations or logic blocks.
  **L472 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L473 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L473 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L474 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L474 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L475 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L475 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L476 EN**: Blank line separating nearby declarations or logic blocks.
  **L476 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L477 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L477 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L478 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L478 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L479 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L479 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L480 EN**: Blank line separating nearby declarations or logic blocks.
  **L480 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 481-504

````cpp
TARGET_BUILTIN(__builtin_lsx_vssrln_bu_h, "V16UcV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrln_hu_w, "V8UsV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrln_wu_d, "V4UiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssran_bu_h, "V16UcV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssran_hu_w, "V8UsV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssran_wu_d, "V4UiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrlni_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlni_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlni_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlni_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrani_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrani_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrani_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrani_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrlrni_bu_h, "V16ScV16UcV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrni_hu_w, "V8sV8UsV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrni_wu_d, "V4iV4UiV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrni_du_q, "V2LLiV2ULLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrani_bu_h, "V16ScV16UcV16ScIUi", "nc", "lsx")
````
- **L481 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L481 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L482 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L482 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L483 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L483 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L484 EN**: Blank line separating nearby declarations or logic blocks.
  **L484 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L485 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L485 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L486 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L486 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L487 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L487 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L488 EN**: Blank line separating nearby declarations or logic blocks.
  **L488 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L489 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L489 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L490 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L490 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L491 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L491 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L492 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L492 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L493 EN**: Blank line separating nearby declarations or logic blocks.
  **L493 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L494 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L494 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L495 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L495 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L496 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L496 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L497 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L497 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L498 EN**: Blank line separating nearby declarations or logic blocks.
  **L498 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L499 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L499 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L500 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L500 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L501 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L501 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L502 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L502 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L503 EN**: Blank line separating nearby declarations or logic blocks.
  **L503 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L504 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L504 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 505-528

````cpp
TARGET_BUILTIN(__builtin_lsx_vssrani_hu_w, "V8sV8UsV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrani_wu_d, "V4iV4UiV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrani_du_q, "V2LLiV2ULLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrlrn_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrn_h_w, "V8sV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrn_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrarn_b_h, "V16ScV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarn_h_w, "V8sV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarn_w_d, "V4SiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrlrn_bu_h, "V16UcV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrn_hu_w, "V8UsV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrn_wu_d, "V4UiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrarn_bu_h, "V16UcV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarn_hu_w, "V8UsV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarn_wu_d, "V4UiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrlrni_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrni_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrni_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlrni_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")
````
- **L505 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L505 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L506 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L506 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L507 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L507 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L508 EN**: Blank line separating nearby declarations or logic blocks.
  **L508 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L509 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L509 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L510 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L510 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L511 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L511 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L512 EN**: Blank line separating nearby declarations or logic blocks.
  **L512 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L513 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L513 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L514 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L514 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L515 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L515 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L516 EN**: Blank line separating nearby declarations or logic blocks.
  **L516 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L517 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L517 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L518 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L518 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L519 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L519 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L520 EN**: Blank line separating nearby declarations or logic blocks.
  **L520 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L521 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L521 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L522 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L522 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L523 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L523 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L524 EN**: Blank line separating nearby declarations or logic blocks.
  **L524 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L525 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L525 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L526 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L526 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L527 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L527 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L528 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L528 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 529-552

````cpp

TARGET_BUILTIN(__builtin_lsx_vssrarni_b_h, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarni_h_w, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarni_w_d, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarni_d_q, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrlni_bu_h, "V16ScV16UcV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlni_hu_w, "V8sV8UsV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlni_wu_d, "V4iV4UiV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrlni_du_q, "V2LLiV2ULLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vssrarni_bu_h, "V16ScV16UcV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarni_hu_w, "V8sV8UsV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarni_wu_d, "V4iV4UiV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vssrarni_du_q, "V2LLiV2ULLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vclo_b, "V16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vclo_h, "V8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vclo_w, "V4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vclo_d, "V2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vclz_b, "V16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vclz_h, "V8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vclz_w, "V4SiV4Si", "nc", "lsx")
````
- **L529 EN**: Blank line separating nearby declarations or logic blocks.
  **L529 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L530 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L530 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L531 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L531 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L532 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L532 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L533 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L533 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L534 EN**: Blank line separating nearby declarations or logic blocks.
  **L534 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L535 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L535 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L536 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L536 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L537 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L537 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L538 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L538 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L539 EN**: Blank line separating nearby declarations or logic blocks.
  **L539 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L540 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L540 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L541 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L541 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L542 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L542 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L543 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L543 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L544 EN**: Blank line separating nearby declarations or logic blocks.
  **L544 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L545 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L545 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L546 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L546 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L547 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L547 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L548 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L548 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L549 EN**: Blank line separating nearby declarations or logic blocks.
  **L549 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L550 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L550 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L551 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L551 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L552 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L552 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 553-576

````cpp
TARGET_BUILTIN(__builtin_lsx_vclz_d, "V2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vpcnt_b, "V16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpcnt_h, "V8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpcnt_w, "V4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpcnt_d, "V2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitclr_b, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitclr_h, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitclr_w, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitclr_d, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitclri_b, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitclri_h, "V8UsV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitclri_w, "V4UiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitclri_d, "V2ULLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitset_b, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitset_h, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitset_w, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitset_d, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitseti_b, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitseti_h, "V8UsV8UsIUi", "nc", "lsx")
````
- **L553 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L553 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L554 EN**: Blank line separating nearby declarations or logic blocks.
  **L554 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L555 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L555 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L556 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L556 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L557 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L557 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L558 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L558 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L559 EN**: Blank line separating nearby declarations or logic blocks.
  **L559 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L560 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L560 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L561 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L561 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L562 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L562 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L563 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L563 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L564 EN**: Blank line separating nearby declarations or logic blocks.
  **L564 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L565 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L565 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L566 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L566 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L567 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L567 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L568 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L568 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L569 EN**: Blank line separating nearby declarations or logic blocks.
  **L569 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L570 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L570 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L571 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L571 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L572 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L572 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L573 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L573 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L574 EN**: Blank line separating nearby declarations or logic blocks.
  **L574 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L575 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L575 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L576 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L576 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 577-600

````cpp
TARGET_BUILTIN(__builtin_lsx_vbitseti_w, "V4UiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitseti_d, "V2ULLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitrev_b, "V16UcV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitrev_h, "V8UsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitrev_w, "V4UiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitrev_d, "V2ULLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitrevi_b, "V16UcV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitrevi_h, "V8UsV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitrevi_w, "V4UiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbitrevi_d, "V2ULLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrstp_b, "V16ScV16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrstp_h, "V8SsV8SsV8SsV8Ss", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrstpi_b, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrstpi_h, "V8sV8sV8sIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfadd_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfadd_d, "V2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfsub_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfsub_d, "V2dV2dV2d", "nc", "lsx")
````
- **L577 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L577 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L578 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L578 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L579 EN**: Blank line separating nearby declarations or logic blocks.
  **L579 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L580 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L580 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L581 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L581 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L582 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L582 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L583 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L583 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L584 EN**: Blank line separating nearby declarations or logic blocks.
  **L584 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L585 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L585 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L586 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L586 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L587 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L587 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L588 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L588 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L589 EN**: Blank line separating nearby declarations or logic blocks.
  **L589 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L590 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L590 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L591 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L591 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L592 EN**: Blank line separating nearby declarations or logic blocks.
  **L592 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L593 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L593 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L594 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L594 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L595 EN**: Blank line separating nearby declarations or logic blocks.
  **L595 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L596 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L596 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L597 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L597 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L599 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L599 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L600 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L600 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 601-624

````cpp

TARGET_BUILTIN(__builtin_lsx_vfmul_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfmul_d, "V2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfdiv_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfdiv_d, "V2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfmadd_s, "V4fV4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfmadd_d, "V2dV2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfmsub_s, "V4fV4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfmsub_d, "V2dV2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfnmadd_s, "V4fV4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfnmadd_d, "V2dV2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfnmsub_s, "V4fV4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfnmsub_d, "V2dV2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfmax_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfmax_d, "V2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfmin_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfmin_d, "V2dV2dV2d", "nc", "lsx")
````
- **L601 EN**: Blank line separating nearby declarations or logic blocks.
  **L601 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L602 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L602 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L603 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L603 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L604 EN**: Blank line separating nearby declarations or logic blocks.
  **L604 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L605 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L605 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L606 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L606 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L607 EN**: Blank line separating nearby declarations or logic blocks.
  **L607 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L608 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L608 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L609 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L609 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L610 EN**: Blank line separating nearby declarations or logic blocks.
  **L610 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L611 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L611 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L612 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L612 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L614 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L615 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L615 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L616 EN**: Blank line separating nearby declarations or logic blocks.
  **L616 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L617 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L617 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L618 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L618 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L619 EN**: Blank line separating nearby declarations or logic blocks.
  **L619 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L620 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L620 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L621 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L621 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L622 EN**: Blank line separating nearby declarations or logic blocks.
  **L622 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L623 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L623 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L624 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L624 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 625-648

````cpp

TARGET_BUILTIN(__builtin_lsx_vfmaxa_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfmaxa_d, "V2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfmina_s, "V4fV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfmina_d, "V2dV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vflogb_s, "V4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vflogb_d, "V2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfclass_s, "V4iV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfclass_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfsqrt_s, "V4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfsqrt_d, "V2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrecip_s, "V4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrecip_d, "V2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrecipe_s, "V4fV4f", "nc", "lsx,frecipe")
TARGET_BUILTIN(__builtin_lsx_vfrecipe_d, "V2dV2d", "nc", "lsx,frecipe")

TARGET_BUILTIN(__builtin_lsx_vfrsqrt_s, "V4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrsqrt_d, "V2dV2d", "nc", "lsx")
````
- **L625 EN**: Blank line separating nearby declarations or logic blocks.
  **L625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L626 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L626 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L627 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L627 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L628 EN**: Blank line separating nearby declarations or logic blocks.
  **L628 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L629 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L629 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L630 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L630 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L631 EN**: Blank line separating nearby declarations or logic blocks.
  **L631 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L632 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L632 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L633 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L633 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L634 EN**: Blank line separating nearby declarations or logic blocks.
  **L634 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L635 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L635 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L636 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L636 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L637 EN**: Blank line separating nearby declarations or logic blocks.
  **L637 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L638 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L638 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L639 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L639 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L640 EN**: Blank line separating nearby declarations or logic blocks.
  **L640 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L641 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L641 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L642 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L642 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L643 EN**: Blank line separating nearby declarations or logic blocks.
  **L643 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L644 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L644 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L645 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L645 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L646 EN**: Blank line separating nearby declarations or logic blocks.
  **L646 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L647 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L647 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L648 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L648 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 649-672

````cpp

TARGET_BUILTIN(__builtin_lsx_vfrsqrte_s, "V4fV4f", "nc", "lsx,frecipe")
TARGET_BUILTIN(__builtin_lsx_vfrsqrte_d, "V2dV2d", "nc", "lsx,frecipe")

TARGET_BUILTIN(__builtin_lsx_vfcvtl_s_h, "V4fV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcvtl_d_s, "V2dV4f", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcvth_s_h, "V4fV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcvth_d_s, "V2dV4f", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcvt_h_s, "V8sV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcvt_s_d, "V4fV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrintrne_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrintrne_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrintrz_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrintrz_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrintrp_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrintrp_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfrintrm_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrintrm_d, "V2LLiV2d", "nc", "lsx")
````
- **L649 EN**: Blank line separating nearby declarations or logic blocks.
  **L649 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L650 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L650 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L651 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L651 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L652 EN**: Blank line separating nearby declarations or logic blocks.
  **L652 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L653 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L653 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L654 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L654 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L655 EN**: Blank line separating nearby declarations or logic blocks.
  **L655 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L656 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L656 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L657 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L657 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L658 EN**: Blank line separating nearby declarations or logic blocks.
  **L658 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L659 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L659 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L660 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L660 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L661 EN**: Blank line separating nearby declarations or logic blocks.
  **L661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L662 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L662 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L663 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L663 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L664 EN**: Blank line separating nearby declarations or logic blocks.
  **L664 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L665 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L665 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L666 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L666 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L667 EN**: Blank line separating nearby declarations or logic blocks.
  **L667 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L668 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L668 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L669 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L669 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L670 EN**: Blank line separating nearby declarations or logic blocks.
  **L670 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L671 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L671 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L672 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L672 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 673-696

````cpp

TARGET_BUILTIN(__builtin_lsx_vfrint_s, "V4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfrint_d, "V2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrne_w_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrne_l_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrz_w_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrz_l_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrp_w_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrp_l_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrm_w_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrm_l_d, "V2LLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftint_w_s, "V4SiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftint_l_d, "V2SLLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrz_wu_s, "V4UiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrz_lu_d, "V2ULLiV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftint_wu_s, "V4UiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftint_lu_d, "V2ULLiV2d", "nc", "lsx")
````
- **L673 EN**: Blank line separating nearby declarations or logic blocks.
  **L673 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L674 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L674 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L675 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L675 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L676 EN**: Blank line separating nearby declarations or logic blocks.
  **L676 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L677 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L677 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L678 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L678 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L680 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L680 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L681 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L681 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L682 EN**: Blank line separating nearby declarations or logic blocks.
  **L682 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L683 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L683 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L684 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L684 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L685 EN**: Blank line separating nearby declarations or logic blocks.
  **L685 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L686 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L686 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L687 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L687 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L688 EN**: Blank line separating nearby declarations or logic blocks.
  **L688 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L689 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L689 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L690 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L690 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L691 EN**: Blank line separating nearby declarations or logic blocks.
  **L691 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L692 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L692 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L693 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L693 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L694 EN**: Blank line separating nearby declarations or logic blocks.
  **L694 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L695 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L695 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L696 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L696 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 697-720

````cpp

TARGET_BUILTIN(__builtin_lsx_vftintrne_w_d, "V4SiV2dV2d", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrz_w_d, "V4SiV2dV2d", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrp_w_d, "V4SiV2dV2d", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrm_w_d, "V4SiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftint_w_d, "V4SiV2dV2d", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrnel_l_s, "V2LLiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrneh_l_s, "V2LLiV4f", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrzl_l_s, "V2LLiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrzh_l_s, "V2LLiV4f", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrpl_l_s, "V2LLiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrph_l_s, "V2LLiV4f", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintrml_l_s, "V2LLiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftintrmh_l_s, "V2LLiV4f", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vftintl_l_s, "V2LLiV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vftinth_l_s, "V2LLiV4f", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vffint_s_w, "V4fV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vffint_d_l, "V2dV2SLLi", "nc", "lsx")
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L698 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L698 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L699 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L699 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L700 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L700 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L701 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L701 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L702 EN**: Blank line separating nearby declarations or logic blocks.
  **L702 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L703 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L703 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L704 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L704 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L705 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L705 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L706 EN**: Blank line separating nearby declarations or logic blocks.
  **L706 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L707 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L707 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L708 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L708 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L709 EN**: Blank line separating nearby declarations or logic blocks.
  **L709 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L710 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L710 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L711 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L711 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L712 EN**: Blank line separating nearby declarations or logic blocks.
  **L712 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L713 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L713 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L714 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L714 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L715 EN**: Blank line separating nearby declarations or logic blocks.
  **L715 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L716 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L716 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L717 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L717 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L718 EN**: Blank line separating nearby declarations or logic blocks.
  **L718 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L719 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L719 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L720 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L720 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 721-744

````cpp

TARGET_BUILTIN(__builtin_lsx_vffint_s_wu, "V4fV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vffint_d_lu, "V2dV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vffintl_d_w, "V2dV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vffinth_d_w, "V2dV4Si", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vffint_s_l, "V4fV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vseq_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vseq_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vseq_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vseq_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vseqi_b, "V16ScV16ScISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vseqi_h, "V8SsV8SsISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vseqi_w, "V4SiV4SiISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vseqi_d, "V2SLLiV2SLLiISi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsle_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsle_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsle_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsle_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

````
- **L721 EN**: Blank line separating nearby declarations or logic blocks.
  **L721 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L722 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L722 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L723 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L723 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L724 EN**: Blank line separating nearby declarations or logic blocks.
  **L724 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L725 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L725 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L726 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L726 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L727 EN**: Blank line separating nearby declarations or logic blocks.
  **L727 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L728 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L728 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L730 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L730 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L731 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L731 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L732 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L732 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L733 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L733 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L734 EN**: Blank line separating nearby declarations or logic blocks.
  **L734 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L735 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L735 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L736 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L736 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L737 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L737 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L738 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L738 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L739 EN**: Blank line separating nearby declarations or logic blocks.
  **L739 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L740 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L740 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L741 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L741 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L742 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L742 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L743 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L743 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L744 EN**: Blank line separating nearby declarations or logic blocks.
  **L744 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 745-768

````cpp
TARGET_BUILTIN(__builtin_lsx_vslei_b, "V16ScV16ScISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslei_h, "V8SsV8SsISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslei_w, "V4SiV4SiISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslei_d, "V2SLLiV2SLLiISi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vsle_bu, "V16ScV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsle_hu, "V8SsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsle_wu, "V4SiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vsle_du, "V2SLLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vslei_bu, "V16ScV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslei_hu, "V8SsV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslei_wu, "V4SiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslei_du, "V2SLLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vslt_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslt_h, "V8SsV8SsV8Ss", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslt_w, "V4SiV4SiV4Si", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslt_d, "V2SLLiV2SLLiV2SLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vslti_b, "V16ScV16ScISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslti_h, "V8SsV8SsISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslti_w, "V4SiV4SiISi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslti_d, "V2SLLiV2SLLiISi", "nc", "lsx")
````
- **L745 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L745 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L746 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L746 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L747 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L747 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L748 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L748 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L749 EN**: Blank line separating nearby declarations or logic blocks.
  **L749 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L750 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L750 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L751 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L751 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L752 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L752 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L753 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L753 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L754 EN**: Blank line separating nearby declarations or logic blocks.
  **L754 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L755 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L755 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L756 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L756 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L757 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L757 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L758 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L758 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L759 EN**: Blank line separating nearby declarations or logic blocks.
  **L759 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L760 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L760 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L761 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L761 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L762 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L762 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L763 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L763 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L764 EN**: Blank line separating nearby declarations or logic blocks.
  **L764 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L765 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L765 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L766 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L766 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L767 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L767 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L768 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L768 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 769-792

````cpp

TARGET_BUILTIN(__builtin_lsx_vslt_bu, "V16ScV16UcV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslt_hu, "V8SsV8UsV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslt_wu, "V4SiV4UiV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslt_du, "V2SLLiV2ULLiV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vslti_bu, "V16ScV16UcIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslti_hu, "V8SsV8UsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslti_wu, "V4SiV4UiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vslti_du, "V2SLLiV2ULLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_caf_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_caf_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cun_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cun_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_ceq_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_ceq_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cueq_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cueq_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_clt_s, "V4SiV4fV4f", "nc", "lsx")
````
- **L769 EN**: Blank line separating nearby declarations or logic blocks.
  **L769 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L770 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L770 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L771 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L771 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L772 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L772 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L773 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L773 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L774 EN**: Blank line separating nearby declarations or logic blocks.
  **L774 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L775 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L775 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L776 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L776 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L777 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L777 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L778 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L778 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L780 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L780 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L781 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L781 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L782 EN**: Blank line separating nearby declarations or logic blocks.
  **L782 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L783 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L783 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L784 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L784 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L785 EN**: Blank line separating nearby declarations or logic blocks.
  **L785 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L786 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L786 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L787 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L787 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L788 EN**: Blank line separating nearby declarations or logic blocks.
  **L788 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L789 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L789 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L790 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L790 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L791 EN**: Blank line separating nearby declarations or logic blocks.
  **L791 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L792 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L792 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 793-816

````cpp
TARGET_BUILTIN(__builtin_lsx_vfcmp_clt_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cult_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cult_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cle_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cle_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cule_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cule_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cne_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cne_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cor_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cor_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_cune_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_cune_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_saf_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_saf_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sun_s, "V4SiV4fV4f", "nc", "lsx")
````
- **L793 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L793 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L795 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L796 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L796 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L798 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L798 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L799 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L799 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L800 EN**: Blank line separating nearby declarations or logic blocks.
  **L800 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L801 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L801 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L802 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L802 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L803 EN**: Blank line separating nearby declarations or logic blocks.
  **L803 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L804 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L804 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L805 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L805 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L806 EN**: Blank line separating nearby declarations or logic blocks.
  **L806 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L807 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L807 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L808 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L808 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L810 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L810 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L811 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L811 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L812 EN**: Blank line separating nearby declarations or logic blocks.
  **L812 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L813 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L813 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L814 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L814 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L815 EN**: Blank line separating nearby declarations or logic blocks.
  **L815 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L816 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L816 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 817-840

````cpp
TARGET_BUILTIN(__builtin_lsx_vfcmp_sun_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_seq_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_seq_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sueq_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_sueq_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_slt_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_slt_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sult_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_sult_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sle_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_sle_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sule_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_sule_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sne_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_sne_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sor_s, "V4SiV4fV4f", "nc", "lsx")
````
- **L817 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L817 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L818 EN**: Blank line separating nearby declarations or logic blocks.
  **L818 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L819 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L819 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L820 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L820 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L821 EN**: Blank line separating nearby declarations or logic blocks.
  **L821 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L822 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L822 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L823 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L823 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L824 EN**: Blank line separating nearby declarations or logic blocks.
  **L824 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L825 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L825 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L826 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L826 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L827 EN**: Blank line separating nearby declarations or logic blocks.
  **L827 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L828 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L828 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L829 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L829 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L830 EN**: Blank line separating nearby declarations or logic blocks.
  **L830 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L831 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L831 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L832 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L832 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L833 EN**: Blank line separating nearby declarations or logic blocks.
  **L833 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L834 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L834 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L835 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L835 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L836 EN**: Blank line separating nearby declarations or logic blocks.
  **L836 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L837 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L837 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L838 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L838 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L839 EN**: Blank line separating nearby declarations or logic blocks.
  **L839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L840 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L840 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 841-864

````cpp
TARGET_BUILTIN(__builtin_lsx_vfcmp_sor_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vfcmp_sune_s, "V4SiV4fV4f", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vfcmp_sune_d, "V2SLLiV2dV2d", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitsel_v, "V16UcV16UcV16UcV16Uc", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbitseli_b, "V16UcV16UcV16UcIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vreplgr2vr_b, "V16Sci", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplgr2vr_h, "V8Ssi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplgr2vr_w, "V4Sii", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplgr2vr_d, "V2SLLiLLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vinsgr2vr_b, "V16ScV16SciIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vinsgr2vr_h, "V8SsV8SsiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vinsgr2vr_w, "V4SiV4SiiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vinsgr2vr_d, "V2SLLiV2SLLiLLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vpickve2gr_b, "iV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickve2gr_h, "iV8SsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickve2gr_w, "iV4SiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickve2gr_d, "LLiV2SLLiIUi", "nc", "lsx")

````
- **L841 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L841 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L843 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L843 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L844 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L844 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L846 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L846 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L847 EN**: Blank line separating nearby declarations or logic blocks.
  **L847 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L848 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L848 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L849 EN**: Blank line separating nearby declarations or logic blocks.
  **L849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L850 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L850 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L851 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L851 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L852 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L852 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L853 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L853 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L854 EN**: Blank line separating nearby declarations or logic blocks.
  **L854 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L855 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L855 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L856 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L856 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L857 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L857 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L858 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L858 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L859 EN**: Blank line separating nearby declarations or logic blocks.
  **L859 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L860 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L860 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L861 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L861 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L862 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L862 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L863 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L863 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 865-888

````cpp
TARGET_BUILTIN(__builtin_lsx_vpickve2gr_bu, "iV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickve2gr_hu, "iV8SsIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickve2gr_wu, "iV4SiIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickve2gr_du, "LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vreplve_b, "V16ScV16ScUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplve_h, "V8sV8sUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplve_w, "V4iV4iUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplve_d, "V2LLiV2LLiUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vreplvei_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplvei_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplvei_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vreplvei_d, "V2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vbsll_v, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vbsrl_v, "V16ScV16ScIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vpackev_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpackev_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpackev_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpackev_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vpackod_b, "V16ScV16ScV16Sc", "nc", "lsx")
````
- **L865 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L865 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L866 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L866 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L867 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L867 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L868 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L868 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L869 EN**: Blank line separating nearby declarations or logic blocks.
  **L869 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L870 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L870 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L871 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L871 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L872 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L872 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L873 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L873 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L874 EN**: Blank line separating nearby declarations or logic blocks.
  **L874 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L875 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L875 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L876 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L876 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L877 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L877 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L878 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L878 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L879 EN**: Blank line separating nearby declarations or logic blocks.
  **L879 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L880 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L880 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L881 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L881 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L882 EN**: Blank line separating nearby declarations or logic blocks.
  **L882 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L883 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L883 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L884 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L884 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L885 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L885 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L886 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L886 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L887 EN**: Blank line separating nearby declarations or logic blocks.
  **L887 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L888 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L888 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 889-912

````cpp
TARGET_BUILTIN(__builtin_lsx_vpackod_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpackod_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpackod_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vpickev_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickev_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickev_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickev_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vpickod_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickod_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickod_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vpickod_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vilvl_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vilvl_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vilvl_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vilvl_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vilvh_b, "V16ScV16ScV16Sc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vilvh_h, "V8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vilvh_w, "V4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vilvh_d, "V2LLiV2LLiV2LLi", "nc", "lsx")

````
- **L889 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L889 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L890 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L890 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L891 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L891 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L892 EN**: Blank line separating nearby declarations or logic blocks.
  **L892 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L893 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L893 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L894 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L894 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L895 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L895 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L896 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L896 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L897 EN**: Blank line separating nearby declarations or logic blocks.
  **L897 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L898 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L898 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L899 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L899 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L900 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L900 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L901 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L901 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L902 EN**: Blank line separating nearby declarations or logic blocks.
  **L902 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L903 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L903 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L904 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L904 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L905 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L905 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L906 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L906 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L907 EN**: Blank line separating nearby declarations or logic blocks.
  **L907 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L908 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L908 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L909 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L909 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L910 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L910 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L911 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L911 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L912 EN**: Blank line separating nearby declarations or logic blocks.
  **L912 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 913-936

````cpp
TARGET_BUILTIN(__builtin_lsx_vshuf_b, "V16ScV16ScV16ScV16Sc", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vshuf_h, "V8sV8sV8sV8s", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vshuf_w, "V4iV4iV4iV4i", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vshuf_d, "V2LLiV2LLiV2LLiV2LLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vshuf4i_b, "V16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vshuf4i_h, "V8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vshuf4i_w, "V4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vshuf4i_d, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vpermi_w, "V4iV4iV4iIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vextrins_b, "V16ScV16ScV16ScIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vextrins_h, "V8sV8sV8sIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vextrins_w, "V4iV4iV4iIUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vextrins_d, "V2LLiV2LLiV2LLiIUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vld, "V16ScvC*Ii", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vst, "vV16Scv*Ii", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vldx, "V16ScvC*LLi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vstx, "vV16Scv*LLi", "nc", "lsx")

````
- **L913 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L913 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L914 EN**: Blank line separating nearby declarations or logic blocks.
  **L914 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L915 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L915 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L916 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L916 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L917 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L917 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L918 EN**: Blank line separating nearby declarations or logic blocks.
  **L918 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L919 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L919 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L920 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L920 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L921 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L921 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L922 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L922 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L923 EN**: Blank line separating nearby declarations or logic blocks.
  **L923 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L924 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L924 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L925 EN**: Blank line separating nearby declarations or logic blocks.
  **L925 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L926 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L926 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L927 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L927 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L928 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L928 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L929 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L929 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L930 EN**: Blank line separating nearby declarations or logic blocks.
  **L930 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L931 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L931 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L932 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L932 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L933 EN**: Blank line separating nearby declarations or logic blocks.
  **L933 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L934 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L934 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L935 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L935 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L936 EN**: Blank line separating nearby declarations or logic blocks.
  **L936 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 937-959

````cpp
TARGET_BUILTIN(__builtin_lsx_vldrepl_b, "V16ScvC*Ii", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vldrepl_h, "V8svC*Ii", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vldrepl_w, "V4ivC*Ii", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vldrepl_d, "V2LLivC*Ii", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_vstelm_b, "vV16Scv*IiUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vstelm_h, "vV8Ssv*IiUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vstelm_w, "vV4Siv*IiUi", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_vstelm_d, "vV2SLLiv*IiUi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_bz_v, "iV16Uc", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_bnz_v, "iV16Uc", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_bz_b, "iV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_bz_h, "iV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_bz_w, "iV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_bz_d, "iV2ULLi", "nc", "lsx")

TARGET_BUILTIN(__builtin_lsx_bnz_b, "iV16Uc", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_bnz_h, "iV8Us", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_bnz_w, "iV4Ui", "nc", "lsx")
TARGET_BUILTIN(__builtin_lsx_bnz_d, "iV2ULLi", "nc", "lsx")
````
- **L937 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L937 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L938 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L938 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L939 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L939 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L940 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L940 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L941 EN**: Blank line separating nearby declarations or logic blocks.
  **L941 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L942 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L942 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L943 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L943 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L944 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L944 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L945 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L945 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L946 EN**: Blank line separating nearby declarations or logic blocks.
  **L946 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L947 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L947 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L949 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L949 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L950 EN**: Blank line separating nearby declarations or logic blocks.
  **L950 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L951 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L951 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L952 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L952 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L953 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L953 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L954 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L954 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L955 EN**: Blank line separating nearby declarations or logic blocks.
  **L955 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L956 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L956 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L957 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L957 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L958 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L958 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L959 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L959 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

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
- **Macro-driven definition lists / 宏驱动定义列表**
  - **EN**: Provides reusable entry lists that different includers expand under different macro definitions.
  - **CN**: 提供可复用的条目列表，不同包含者可在不同宏定义下对其展开。
- **Builtin descriptions / Builtin 描述**
  - **EN**: Describes compiler builtins, their signatures, properties, or target-specific availability.
  - **CN**: 描述编译器 builtin 的签名、属性或目标相关可用性。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `TARGET_BUILTIN`
- **TableGen records / TableGen 记录**: No prominent TableGen record names detected. / 未检测到明显的 TableGen 记录名称。
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
