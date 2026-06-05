# BuiltinsLoongArchLASX.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/BuiltinsLoongArchLASX.def`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: LoongArch Builtin function database C++.
- **Purpose (CN)**: 声明与 `BuiltinsLoongArchLASX` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 1007

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````cpp
//=BuiltinsLoongArchLASX.def - LoongArch Builtin function database -- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file defines the LoongArch-specific LASX builtin function database.
// Users of this file must define the BUILTIN macro to make use of this
// information.
//
//===----------------------------------------------------------------------===//

TARGET_BUILTIN(__builtin_lasx_xvadd_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvadd_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvadd_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvadd_d, "V4LLiV4LLiV4LLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvadd_q, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsub_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsub_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsub_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsub_d, "V4LLiV4LLiV4LLi", "nc", "lasx")
````
- **L1 EN**: Comment explains nearby logic, constraints, or intent: `BuiltinsLoongArchLASX.def - LoongArch Builtin function database C++`.
  **L1 CN**: 注释解释附近代码的逻辑、约束或设计意图：`BuiltinsLoongArchLASX.def - LoongArch Builtin function database C++`。
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
- **L9 EN**: Comment explains nearby logic, constraints, or intent: `This file defines the LoongArch-specific LASX builtin function database.`.
  **L9 CN**: 注释解释附近代码的逻辑、约束或设计意图：`This file defines the LoongArch-specific LASX builtin function database.`。
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
TARGET_BUILTIN(__builtin_lasx_xvsub_q, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvaddi_bu, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddi_hu, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddi_wu, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddi_du, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsubi_bu, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubi_hu, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubi_wu, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubi_du, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvneg_b, "V32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvneg_h, "V16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvneg_w, "V8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvneg_d, "V4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsadd_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsadd_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsadd_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsadd_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsadd_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsadd_hu, "V16UsV16UsV16Us", "nc", "lasx")
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
TARGET_BUILTIN(__builtin_lasx_xvsadd_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsadd_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssub_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssub_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssub_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssub_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssub_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssub_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssub_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssub_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvhaddw_h_b, "V16SsV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhaddw_w_h, "V8SiV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhaddw_d_w, "V4SLLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhaddw_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvhaddw_hu_bu, "V16UsV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhaddw_wu_hu, "V8UiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhaddw_du_wu, "V4ULLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhaddw_qu_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvhsubw_h_b, "V16SsV32ScV32Sc", "nc", "lasx")
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
TARGET_BUILTIN(__builtin_lasx_xvhsubw_w_h, "V8SiV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhsubw_d_w, "V4SLLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhsubw_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvhsubw_hu_bu, "V16UsV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhsubw_wu_hu, "V8UiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhsubw_du_wu, "V4ULLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvhsubw_qu_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvaddwev_h_b, "V16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_w_h, "V8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_d_w, "V4LLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvaddwod_h_b, "V16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_w_h, "V8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_d_w, "V4LLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsubwev_h_b, "V16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwev_w_h, "V8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwev_d_w, "V4LLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwev_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

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
TARGET_BUILTIN(__builtin_lasx_xvsubwod_h_b, "V16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwod_w_h, "V8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwod_d_w, "V4LLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwod_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvaddwev_h_bu, "V16sV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_w_hu, "V8SiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_d_wu, "V4LLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_q_du, "V4LLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvaddwod_h_bu, "V16sV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_w_hu, "V8SiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_d_wu, "V4LLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_q_du, "V4LLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsubwev_h_bu, "V16sV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwev_w_hu, "V8SiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwev_d_wu, "V4LLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwev_q_du, "V4LLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsubwod_h_bu, "V16sV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwod_w_hu, "V8SiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwod_d_wu, "V4LLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsubwod_q_du, "V4LLiV4ULLiV4ULLi", "nc", "lasx")
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

TARGET_BUILTIN(__builtin_lasx_xvaddwev_h_bu_b, "V16sV32UcV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_w_hu_h, "V8SiV16UsV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_d_wu_w, "V4LLiV8UiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwev_q_du_d, "V4LLiV4ULLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvaddwod_h_bu_b, "V16sV32UcV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_w_hu_h, "V8SiV16UsV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_d_wu_w, "V4LLiV8UiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvaddwod_q_du_d, "V4LLiV4ULLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvavg_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavg_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavg_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavg_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvavg_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavg_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavg_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavg_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvavgr_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavgr_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavgr_w, "V8SiV8SiV8Si", "nc", "lasx")
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
TARGET_BUILTIN(__builtin_lasx_xvavgr_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvavgr_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavgr_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavgr_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvavgr_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvabsd_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvabsd_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvabsd_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvabsd_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvabsd_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvabsd_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvabsd_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvabsd_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvadda_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvadda_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvadda_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvadda_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmax_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmax_h, "V16SsV16SsV16Ss", "nc", "lasx")
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
TARGET_BUILTIN(__builtin_lasx_xvmax_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmax_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaxi_b, "V32ScV32ScIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaxi_h, "V16SsV16SsIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaxi_w, "V8SiV8SiIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaxi_d, "V4SLLiV4SLLiIi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmax_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmax_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmax_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmax_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaxi_bu, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaxi_hu, "V16UsV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaxi_wu, "V8UiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaxi_du, "V4ULLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmin_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmin_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmin_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmin_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmini_b, "V32ScV32ScIi", "nc", "lasx")
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
TARGET_BUILTIN(__builtin_lasx_xvmini_h, "V16SsV16SsIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmini_w, "V8SiV8SiIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmini_d, "V4SLLiV4SLLiIi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmin_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmin_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmin_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmin_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmini_bu, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmini_hu, "V16UsV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmini_wu, "V8UiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmini_du, "V4ULLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmul_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmul_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmul_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmul_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmuh_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmuh_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmuh_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmuh_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

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
TARGET_BUILTIN(__builtin_lasx_xvmuh_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmuh_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmuh_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmuh_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmulwev_h_b, "V16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_w_h, "V8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_d_w, "V4LLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmulwod_h_b, "V16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_w_h, "V8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_d_w, "V4LLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_q_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmulwev_h_bu, "V16sV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_w_hu, "V8SiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_d_wu, "V4LLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_q_du, "V4LLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmulwod_h_bu, "V16sV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_w_hu, "V8SiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_d_wu, "V4LLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_q_du, "V4LLiV4ULLiV4ULLi", "nc", "lasx")
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

TARGET_BUILTIN(__builtin_lasx_xvmulwev_h_bu_b, "V16sV32UcV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_w_hu_h, "V8SiV16UsV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_d_wu_w, "V4LLiV8UiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwev_q_du_d, "V4LLiV4ULLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmulwod_h_bu_b, "V16sV32UcV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_w_hu_h, "V8SiV16UsV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_d_wu_w, "V4LLiV8UiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmulwod_q_du_d, "V4LLiV4ULLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmadd_b, "V32ScV32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmadd_h, "V16SsV16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmadd_w, "V8SiV8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmadd_d, "V4SLLiV4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmsub_b, "V32ScV32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmsub_h, "V16SsV16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmsub_w, "V8SiV8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmsub_d, "V4SLLiV4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaddwev_h_b, "V16sV16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_w_h, "V8SiV8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_d_w, "V4LLiV4LLiV8SiV8Si", "nc", "lasx")
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
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_q_d, "V4LLiV4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaddwod_h_b, "V16sV16sV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_w_h, "V8SiV8SiV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_d_w, "V4LLiV4LLiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_q_d, "V4LLiV4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaddwev_h_bu, "V16UsV16UsV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_w_hu, "V8UiV8UiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_d_wu, "V4ULLiV4ULLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_q_du, "V4ULLiV4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaddwod_h_bu, "V16UsV16UsV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_w_hu, "V8UiV8UiV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_d_wu, "V4ULLiV4ULLiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_q_du, "V4ULLiV4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaddwev_h_bu_b, "V16sV16sV32UcV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_w_hu_h, "V8SiV8SiV16UsV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_d_wu_w, "V4LLiV4LLiV8UiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwev_q_du_d, "V4LLiV4LLiV4ULLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmaddwod_h_bu_b, "V16sV16sV32UcV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_w_hu_h, "V8SiV8SiV16UsV16s", "nc", "lasx")
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
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_d_wu_w, "V4LLiV4LLiV8UiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmaddwod_q_du_d, "V4LLiV4LLiV4ULLiV4LLi", "nc", "lasx")


TARGET_BUILTIN(__builtin_lasx_xvdiv_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvdiv_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvdiv_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvdiv_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvdiv_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvdiv_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvdiv_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvdiv_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmod_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmod_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmod_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmod_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmod_bu, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmod_hu, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmod_wu, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmod_du, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

````
- **L289 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L289 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L290 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L290 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L291 EN**: Blank line separating nearby declarations or logic blocks.
  **L291 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L292 EN**: Blank line separating nearby declarations or logic blocks.
  **L292 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L293 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L293 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L294 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L294 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L295 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L295 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L296 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L296 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L297 EN**: Blank line separating nearby declarations or logic blocks.
  **L297 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L298 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L298 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L299 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L299 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L300 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L300 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L301 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L301 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L302 EN**: Blank line separating nearby declarations or logic blocks.
  **L302 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L303 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L303 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L304 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L304 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L305 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L305 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L306 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L306 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
TARGET_BUILTIN(__builtin_lasx_xvsat_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsat_h, "V16SsV16SsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsat_w, "V8SiV8SiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsat_d, "V4SLLiV4SLLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsat_bu, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsat_hu, "V16UsV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsat_wu, "V8UiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsat_du, "V4ULLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvexth_h_b, "V16sV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvexth_w_h, "V8SiV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvexth_d_w, "V4LLiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvexth_q_d, "V4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvexth_hu_bu, "V16UsV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvexth_wu_hu, "V8UiV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvexth_du_wu, "V4ULLiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvexth_qu_du, "V4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_vext2xv_h_b, "V16sV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_w_b, "V8SiV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_d_b, "V4LLiV32Sc", "nc", "lasx")

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
- **L336 EN**: Blank line separating nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 337-360

````cpp
TARGET_BUILTIN(__builtin_lasx_vext2xv_w_h, "V8SiV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_d_h, "V4LLiV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_d_w, "V4LLiV8Si", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_vext2xv_hu_bu, "V16sV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_wu_bu, "V8SiV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_du_bu, "V4LLiV32Sc", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_vext2xv_wu_hu, "V8SiV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_du_hu, "V4LLiV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_vext2xv_du_wu, "V4LLiV8Si", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsigncov_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsigncov_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsigncov_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsigncov_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmskltz_b, "V32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmskltz_h, "V16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmskltz_w, "V8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmskltz_d, "V4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvmskgez_b, "V32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvmsknz_b, "V32ScV32Sc", "nc", "lasx")
````
- **L337 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L337 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L338 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L338 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L339 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L339 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L340 EN**: Blank line separating nearby declarations or logic blocks.
  **L340 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L341 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L341 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L342 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L342 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L343 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L343 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L344 EN**: Blank line separating nearby declarations or logic blocks.
  **L344 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L345 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L345 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L346 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L346 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L347 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L347 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L349 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L350 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L350 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L351 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L351 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L352 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L352 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L353 EN**: Blank line separating nearby declarations or logic blocks.
  **L353 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L354 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L354 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L355 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L355 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L356 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L356 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L357 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L357 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L358 EN**: Blank line separating nearby declarations or logic blocks.
  **L358 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L359 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L359 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L360 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L360 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 361-384

````cpp

TARGET_BUILTIN(__builtin_lasx_xvldi, "V4LLiIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrepli_b, "V32ScIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrepli_h, "V16sIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrepli_w, "V8iIi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrepli_d, "V4LLiIi", "nc", "lasx")


TARGET_BUILTIN(__builtin_lasx_xvand_v, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvor_v, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvxor_v, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvnor_v, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvandn_v, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvorn_v, "V32UcV32UcV32Uc", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvandi_b, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvori_b, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvxori_b, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvnori_b, "V32UcV32UcIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsll_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsll_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsll_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsll_d, "V4LLiV4LLiV4LLi", "nc", "lasx")
````
- **L361 EN**: Blank line separating nearby declarations or logic blocks.
  **L361 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L362 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L362 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L363 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L363 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L364 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L364 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L365 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L365 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L366 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L366 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L367 EN**: Blank line separating nearby declarations or logic blocks.
  **L367 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L368 EN**: Blank line separating nearby declarations or logic blocks.
  **L368 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L369 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L369 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L370 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L370 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L371 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L371 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L372 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L372 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L373 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L373 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L374 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L374 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L375 EN**: Blank line separating nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L376 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L376 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L377 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L377 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L378 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L378 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L379 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L379 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L380 EN**: Blank line separating nearby declarations or logic blocks.
  **L380 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L381 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L381 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L382 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L382 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L383 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L383 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L384 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L384 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 385-408

````cpp

TARGET_BUILTIN(__builtin_lasx_xvslli_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslli_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslli_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslli_d, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrl_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrl_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrl_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrl_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrli_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrli_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrli_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrli_d, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsra_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsra_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsra_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsra_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrai_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrai_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrai_w, "V8iV8iIUi", "nc", "lasx")
````
- **L385 EN**: Blank line separating nearby declarations or logic blocks.
  **L385 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L386 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L386 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L387 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L387 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L388 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L388 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L389 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L389 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L390 EN**: Blank line separating nearby declarations or logic blocks.
  **L390 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L391 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L391 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L392 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L392 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L393 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L393 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L394 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L394 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L395 EN**: Blank line separating nearby declarations or logic blocks.
  **L395 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L396 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L396 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L397 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L397 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L398 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L398 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L399 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L399 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L400 EN**: Blank line separating nearby declarations or logic blocks.
  **L400 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L401 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L401 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L402 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L402 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L403 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L403 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L404 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L404 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L405 EN**: Blank line separating nearby declarations or logic blocks.
  **L405 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L406 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L406 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L407 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L407 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L408 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L408 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 409-432

````cpp
TARGET_BUILTIN(__builtin_lasx_xvsrai_d, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvrotr_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrotr_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrotr_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrotr_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvrotri_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrotri_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrotri_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrotri_d, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsllwil_h_b, "V16sV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsllwil_w_h, "V8SiV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsllwil_d_w, "V4LLiV8SiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvextl_q_d, "V4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsllwil_hu_bu, "V16UsV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsllwil_wu_hu, "V8UiV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsllwil_du_wu, "V4ULLiV8UiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvextl_qu_du, "V4LLiV4ULLi", "nc", "lasx")

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
- **L414 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L414 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L415 EN**: Blank line separating nearby declarations or logic blocks.
  **L415 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L416 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L416 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L417 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L417 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L418 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L418 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L419 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L419 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L420 EN**: Blank line separating nearby declarations or logic blocks.
  **L420 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L421 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L421 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L422 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L422 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L423 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L423 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L424 EN**: Blank line separating nearby declarations or logic blocks.
  **L424 CN**: 空行，用于分隔附近的声明或逻辑块。
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
- **L430 EN**: Blank line separating nearby declarations or logic blocks.
  **L430 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L431 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L431 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````cpp
TARGET_BUILTIN(__builtin_lasx_xvsrlr_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlr_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlr_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlr_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrlri_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlri_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlri_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlri_d, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrar_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrar_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrar_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrar_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrari_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrari_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrari_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrari_d, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrln_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrln_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrln_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

````
- **L433 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L433 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L434 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L434 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L435 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L435 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L436 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L436 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L438 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L439 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L439 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L440 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L440 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L441 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L441 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L442 EN**: Blank line separating nearby declarations or logic blocks.
  **L442 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L443 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L443 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L444 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L444 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L445 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L445 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L446 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L446 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L447 EN**: Blank line separating nearby declarations or logic blocks.
  **L447 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L448 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L448 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L449 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L449 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L450 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L450 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L451 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L451 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L452 EN**: Blank line separating nearby declarations or logic blocks.
  **L452 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L453 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L453 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L454 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L454 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L455 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L455 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L456 EN**: Blank line separating nearby declarations or logic blocks.
  **L456 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 457-480

````cpp
TARGET_BUILTIN(__builtin_lasx_xvsran_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsran_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsran_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrlni_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlni_h_w, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlni_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlni_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrani_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrani_h_w, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrani_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrani_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrlrn_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlrn_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlrn_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrarn_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrarn_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrarn_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrlrni_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlrni_h_w, "V16sV16sV16sIUi", "nc", "lasx")
````
- **L457 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L457 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L458 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L458 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L459 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L459 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L460 EN**: Blank line separating nearby declarations or logic blocks.
  **L460 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L461 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L461 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L462 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L462 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L463 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L463 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L464 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L464 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L465 EN**: Blank line separating nearby declarations or logic blocks.
  **L465 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L466 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L466 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L467 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L467 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L468 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L468 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L469 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L469 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L470 EN**: Blank line separating nearby declarations or logic blocks.
  **L470 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L471 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L471 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L472 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L472 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L473 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L473 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L474 EN**: Blank line separating nearby declarations or logic blocks.
  **L474 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L475 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L475 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L476 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L476 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L477 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L477 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L478 EN**: Blank line separating nearby declarations or logic blocks.
  **L478 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L479 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L479 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L480 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L480 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 481-504

````cpp
TARGET_BUILTIN(__builtin_lasx_xvsrlrni_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrlrni_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsrarni_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrarni_h_w, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrarni_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsrarni_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrln_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrln_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrln_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssran_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssran_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssran_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrln_bu_h, "V32UcV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrln_hu_w, "V16UsV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrln_wu_d, "V8UiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssran_bu_h, "V32UcV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssran_hu_w, "V16UsV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssran_wu_d, "V8UiV4ULLiV4ULLi", "nc", "lasx")

````
- **L481 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L481 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L482 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L482 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L483 EN**: Blank line separating nearby declarations or logic blocks.
  **L483 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L484 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L484 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L492 EN**: Blank line separating nearby declarations or logic blocks.
  **L492 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L493 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L493 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L494 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L494 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L495 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L495 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L496 EN**: Blank line separating nearby declarations or logic blocks.
  **L496 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L497 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L497 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L498 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L498 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L499 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L499 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L500 EN**: Blank line separating nearby declarations or logic blocks.
  **L500 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L501 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L501 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L502 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L502 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L503 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L503 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L504 EN**: Blank line separating nearby declarations or logic blocks.
  **L504 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 505-528

````cpp
TARGET_BUILTIN(__builtin_lasx_xvssrlni_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlni_h_w, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlni_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlni_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrani_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrani_h_w, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrani_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrani_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrlrni_bu_h, "V32ScV32UcV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrni_hu_w, "V16sV16UsV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrni_wu_d, "V8iV8UiV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrni_du_q, "V4LLiV4ULLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrani_bu_h, "V32ScV32UcV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrani_hu_w, "V16sV16UsV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrani_wu_d, "V8iV8UiV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrani_du_q, "V4LLiV4ULLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrlrn_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrn_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrn_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

````
- **L505 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L505 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L506 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L506 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L507 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L507 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L508 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L508 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L509 EN**: Blank line separating nearby declarations or logic blocks.
  **L509 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L510 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L510 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L511 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L511 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L512 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L512 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L513 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L513 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L514 EN**: Blank line separating nearby declarations or logic blocks.
  **L514 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L515 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L515 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L516 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L516 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L517 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L517 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L518 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L518 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L519 EN**: Blank line separating nearby declarations or logic blocks.
  **L519 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L520 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L520 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L528 EN**: Blank line separating nearby declarations or logic blocks.
  **L528 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 529-552

````cpp
TARGET_BUILTIN(__builtin_lasx_xvssrarn_b_h, "V32ScV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarn_h_w, "V16sV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarn_w_d, "V8SiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrlrn_bu_h, "V32UcV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrn_hu_w, "V16UsV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrn_wu_d, "V8UiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrarn_bu_h, "V32UcV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarn_hu_w, "V16UsV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarn_wu_d, "V8UiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrlrni_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrni_h_w, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrni_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlrni_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrarni_b_h, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarni_h_w, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarni_w_d, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarni_d_q, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrlni_bu_h, "V32ScV32UcV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlni_hu_w, "V16sV16UsV16sIUi", "nc", "lasx")
````
- **L529 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L529 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L530 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L530 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L531 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L531 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L532 EN**: Blank line separating nearby declarations or logic blocks.
  **L532 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L533 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L533 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L534 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L534 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L535 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L535 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L536 EN**: Blank line separating nearby declarations or logic blocks.
  **L536 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L537 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L537 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L538 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L538 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L539 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L539 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L540 EN**: Blank line separating nearby declarations or logic blocks.
  **L540 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L541 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L541 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L542 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L542 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L543 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L543 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L544 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L544 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L545 EN**: Blank line separating nearby declarations or logic blocks.
  **L545 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L546 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L546 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L547 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L547 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L548 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L548 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L549 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L549 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L550 EN**: Blank line separating nearby declarations or logic blocks.
  **L550 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L551 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L551 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L552 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L552 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 553-576

````cpp
TARGET_BUILTIN(__builtin_lasx_xvssrlni_wu_d, "V8iV8UiV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrlni_du_q, "V4LLiV4ULLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvssrarni_bu_h, "V32UcV32UcV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarni_hu_w, "V16sV16UsV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarni_wu_d, "V8iV8UiV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvssrarni_du_q, "V4LLiV4ULLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvclo_b, "V32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvclo_h, "V16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvclo_w, "V8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvclo_d, "V4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvclz_b, "V32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvclz_h, "V16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvclz_w, "V8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvclz_d, "V4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpcnt_b, "V32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpcnt_h, "V16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpcnt_w, "V8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpcnt_d, "V4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbitclr_b, "V32UcV32UcV32Uc", "nc", "lasx")
````
- **L553 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L553 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L554 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L554 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L555 EN**: Blank line separating nearby declarations or logic blocks.
  **L555 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L556 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L556 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L557 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L557 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L558 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L558 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L559 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L559 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L560 EN**: Blank line separating nearby declarations or logic blocks.
  **L560 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L561 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L561 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L562 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L562 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L563 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L563 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L564 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L564 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L565 EN**: Blank line separating nearby declarations or logic blocks.
  **L565 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L566 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L566 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L567 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L567 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L568 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L568 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L569 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L569 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L570 EN**: Blank line separating nearby declarations or logic blocks.
  **L570 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L571 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L571 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L572 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L572 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L573 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L573 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L574 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L574 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L575 EN**: Blank line separating nearby declarations or logic blocks.
  **L575 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L576 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L576 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 577-600

````cpp
TARGET_BUILTIN(__builtin_lasx_xvbitclr_h, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitclr_w, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitclr_d, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbitclri_b, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitclri_h, "V16UsV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitclri_w, "V8UiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitclri_d, "V4ULLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbitset_b, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitset_h, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitset_w, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitset_d, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbitseti_b, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitseti_h, "V16UsV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitseti_w, "V8UiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitseti_d, "V4ULLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbitrev_b, "V32UcV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitrev_h, "V16UsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitrev_w, "V8UiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitrev_d, "V4ULLiV4ULLiV4ULLi", "nc", "lasx")

````
- **L577 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L577 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L578 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L578 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L579 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L579 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L580 EN**: Blank line separating nearby declarations or logic blocks.
  **L580 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L581 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L581 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L582 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L582 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L583 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L583 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L584 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L584 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L585 EN**: Blank line separating nearby declarations or logic blocks.
  **L585 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L586 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L586 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L587 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L587 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L588 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L588 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L589 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L589 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L590 EN**: Blank line separating nearby declarations or logic blocks.
  **L590 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L591 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L591 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L592 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L592 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L598 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L598 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L599 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L599 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L600 EN**: Blank line separating nearby declarations or logic blocks.
  **L600 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 601-624

````cpp
TARGET_BUILTIN(__builtin_lasx_xvbitrevi_b, "V32UcV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitrevi_h, "V16UsV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitrevi_w, "V8UiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbitrevi_d, "V4ULLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrstp_b, "V32ScV32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrstp_h, "V16SsV16SsV16SsV16Ss", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrstpi_b, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrstpi_h, "V16sV16sV16sIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfadd_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfadd_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfsub_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfsub_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfmul_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfmul_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfdiv_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfdiv_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfmadd_s, "V8fV8fV8fV8f", "nc", "lasx")
````
- **L601 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L601 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L602 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L602 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L603 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L603 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L604 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L604 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L605 EN**: Blank line separating nearby declarations or logic blocks.
  **L605 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L606 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L606 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L607 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L607 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L608 EN**: Blank line separating nearby declarations or logic blocks.
  **L608 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L609 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L609 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L610 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L610 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L611 EN**: Blank line separating nearby declarations or logic blocks.
  **L611 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L612 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L612 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L613 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L613 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L614 EN**: Blank line separating nearby declarations or logic blocks.
  **L614 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L615 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L615 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L616 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L616 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L618 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L619 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L619 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L621 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L622 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L622 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L623 EN**: Blank line separating nearby declarations or logic blocks.
  **L623 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L624 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L624 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 625-648

````cpp
TARGET_BUILTIN(__builtin_lasx_xvfmadd_d, "V4dV4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfmsub_s, "V8fV8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfmsub_d, "V4dV4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfnmadd_s, "V8fV8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfnmadd_d, "V4dV4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfnmsub_s, "V8fV8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfnmsub_d, "V4dV4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfmax_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfmax_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfmin_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfmin_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfmaxa_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfmaxa_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfmina_s, "V8fV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfmina_d, "V4dV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvflogb_s, "V8fV8f", "nc", "lasx")
````
- **L625 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L625 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L626 EN**: Blank line separating nearby declarations or logic blocks.
  **L626 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L627 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L627 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L628 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L628 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L629 EN**: Blank line separating nearby declarations or logic blocks.
  **L629 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L630 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L630 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L631 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L631 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L632 EN**: Blank line separating nearby declarations or logic blocks.
  **L632 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L633 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L633 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L634 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L634 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L635 EN**: Blank line separating nearby declarations or logic blocks.
  **L635 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L636 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L636 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L637 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L637 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L638 EN**: Blank line separating nearby declarations or logic blocks.
  **L638 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L639 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L639 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L640 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L640 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L641 EN**: Blank line separating nearby declarations or logic blocks.
  **L641 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L642 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L642 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L643 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L643 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L644 EN**: Blank line separating nearby declarations or logic blocks.
  **L644 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L645 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L645 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L646 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L646 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L647 EN**: Blank line separating nearby declarations or logic blocks.
  **L647 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L648 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L648 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 649-672

````cpp
TARGET_BUILTIN(__builtin_lasx_xvflogb_d, "V4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfclass_s, "V8iV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfclass_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfsqrt_s, "V8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfsqrt_d, "V4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrecip_s, "V8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrecip_d, "V4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrecipe_s, "V8fV8f", "nc", "lasx,frecipe")
TARGET_BUILTIN(__builtin_lasx_xvfrecipe_d, "V4dV4d", "nc", "lasx,frecipe")

TARGET_BUILTIN(__builtin_lasx_xvfrsqrt_s, "V8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrsqrt_d, "V4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrsqrte_s, "V8fV8f", "nc", "lasx,frecipe")
TARGET_BUILTIN(__builtin_lasx_xvfrsqrte_d, "V4dV4d", "nc", "lasx,frecipe")

TARGET_BUILTIN(__builtin_lasx_xvfcvtl_s_h, "V8fV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcvth_s_h, "V8fV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcvtl_d_s, "V4dV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcvth_d_s, "V4dV8f", "nc", "lasx")
````
- **L649 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L649 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L650 EN**: Blank line separating nearby declarations or logic blocks.
  **L650 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L651 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L651 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L652 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L652 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L654 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L655 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L655 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L656 EN**: Blank line separating nearby declarations or logic blocks.
  **L656 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L657 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L657 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L658 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L658 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L659 EN**: Blank line separating nearby declarations or logic blocks.
  **L659 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L660 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L660 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L661 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L661 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L662 EN**: Blank line separating nearby declarations or logic blocks.
  **L662 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L663 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L663 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L664 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L664 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L666 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L666 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L667 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L667 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L668 EN**: Blank line separating nearby declarations or logic blocks.
  **L668 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L669 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L669 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L670 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L670 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L671 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L671 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L672 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L672 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 673-696

````cpp

TARGET_BUILTIN(__builtin_lasx_xvfcvt_h_s, "V16sV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcvt_s_d, "V8fV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrintrne_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrintrne_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrintrz_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrintrz_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrintrp_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrintrp_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrintrm_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrintrm_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfrint_s, "V8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfrint_d, "V4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrne_w_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrne_l_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrz_w_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrz_l_d, "V4LLiV4d", "nc", "lasx")
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

TARGET_BUILTIN(__builtin_lasx_xvftintrp_w_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrp_l_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrm_w_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrm_l_d, "V4LLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftint_w_s, "V8SiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftint_l_d, "V4SLLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrz_wu_s, "V8UiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrz_lu_d, "V4ULLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftint_wu_s, "V8UiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftint_lu_d, "V4ULLiV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrne_w_d, "V8SiV4dV4d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrz_w_d, "V8SiV4dV4d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrp_w_d, "V8SiV4dV4d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrm_w_d, "V8SiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftint_w_d, "V8SiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrnel_l_s, "V4LLiV8f", "nc", "lasx")
````
- **L697 EN**: Blank line separating nearby declarations or logic blocks.
  **L697 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L698 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L698 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L699 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L699 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L700 EN**: Blank line separating nearby declarations or logic blocks.
  **L700 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L701 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L701 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L702 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L702 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L703 EN**: Blank line separating nearby declarations or logic blocks.
  **L703 CN**: 空行，用于分隔附近的声明或逻辑块。
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
- **L715 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L715 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L716 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L716 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L717 EN**: Blank line separating nearby declarations or logic blocks.
  **L717 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L718 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L718 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L719 EN**: Blank line separating nearby declarations or logic blocks.
  **L719 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L720 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L720 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 721-744

````cpp
TARGET_BUILTIN(__builtin_lasx_xvftintrneh_l_s, "V4LLiV8f", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrzl_l_s, "V4LLiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrzh_l_s, "V4LLiV8f", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrpl_l_s, "V4LLiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrph_l_s, "V4LLiV8f", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintrml_l_s, "V4LLiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftintrmh_l_s, "V4LLiV8f", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvftintl_l_s, "V4LLiV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvftinth_l_s, "V4LLiV8f", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvffint_s_w, "V8fV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvffint_d_l, "V4dV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvffint_s_wu, "V8fV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvffint_d_lu, "V4dV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvffintl_d_w, "V4dV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvffinth_d_w, "V4dV8Si", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvffint_s_l, "V8fV4LLiV4LLi", "nc", "lasx")
````
- **L721 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L721 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L722 EN**: Blank line separating nearby declarations or logic blocks.
  **L722 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L723 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L723 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L724 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L724 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L726 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L726 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L727 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L727 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L728 EN**: Blank line separating nearby declarations or logic blocks.
  **L728 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L729 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L729 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L730 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L730 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L731 EN**: Blank line separating nearby declarations or logic blocks.
  **L731 CN**: 空行，用于分隔附近的声明或逻辑块。
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
- **L737 EN**: Blank line separating nearby declarations or logic blocks.
  **L737 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L738 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L738 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L739 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L739 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L741 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L742 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L742 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L743 EN**: Blank line separating nearby declarations or logic blocks.
  **L743 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L744 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L744 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 745-768

````cpp

TARGET_BUILTIN(__builtin_lasx_xvseq_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvseq_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvseq_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvseq_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvseqi_b, "V32ScV32ScISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvseqi_h, "V16SsV16SsISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvseqi_w, "V8SiV8SiISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvseqi_d, "V4SLLiV4SLLiISi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsle_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsle_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsle_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsle_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvslei_b, "V32ScV32ScISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslei_h, "V16SsV16SsISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslei_w, "V8SiV8SiISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslei_d, "V4SLLiV4SLLiISi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvsle_bu, "V32ScV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsle_hu, "V16SsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvsle_wu, "V8SiV8UiV8Ui", "nc", "lasx")
````
- **L745 EN**: Blank line separating nearby declarations or logic blocks.
  **L745 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L746 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L746 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L747 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L747 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L748 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L748 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L749 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L749 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L750 EN**: Blank line separating nearby declarations or logic blocks.
  **L750 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L751 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L751 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L752 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L752 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L753 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L753 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L754 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L754 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L755 EN**: Blank line separating nearby declarations or logic blocks.
  **L755 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L756 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L756 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L757 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L757 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L758 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L758 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L759 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L759 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L760 EN**: Blank line separating nearby declarations or logic blocks.
  **L760 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L761 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L761 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L762 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L762 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L763 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L763 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L764 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L764 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L765 EN**: Blank line separating nearby declarations or logic blocks.
  **L765 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L766 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L766 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L767 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L767 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L768 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L768 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 769-792

````cpp
TARGET_BUILTIN(__builtin_lasx_xvsle_du, "V4SLLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvslei_bu, "V32ScV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslei_hu, "V16SsV16UsIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslei_wu, "V8SiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslei_du, "V4SLLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvslt_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslt_h, "V16SsV16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslt_w, "V8SiV8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslt_d, "V4SLLiV4SLLiV4SLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvslti_b, "V32ScV32ScISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslti_h, "V16SsV16SsISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslti_w, "V8SiV8SiISi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslti_d, "V4SLLiV4SLLiISi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvslt_bu, "V32ScV32UcV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslt_hu, "V16SsV16UsV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslt_wu, "V8SiV8UiV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslt_du, "V4SLLiV4ULLiV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvslti_bu, "V32ScV32UcIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslti_hu, "V16SsV16UsIUi", "nc", "lasx")
````
- **L769 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L769 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L770 EN**: Blank line separating nearby declarations or logic blocks.
  **L770 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L771 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L771 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L772 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L772 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L773 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L773 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L774 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L774 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L775 EN**: Blank line separating nearby declarations or logic blocks.
  **L775 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L776 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L776 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L777 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L777 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L778 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L778 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L779 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L779 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L780 EN**: Blank line separating nearby declarations or logic blocks.
  **L780 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L781 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L781 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L782 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L782 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L788 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L788 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L789 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L789 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L790 EN**: Blank line separating nearby declarations or logic blocks.
  **L790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L791 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L791 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L792 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L792 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 793-816

````cpp
TARGET_BUILTIN(__builtin_lasx_xvslti_wu, "V8SiV8UiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvslti_du, "V4SLLiV4ULLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_caf_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_caf_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_cun_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cun_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_ceq_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_ceq_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_cueq_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cueq_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_clt_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_clt_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_cult_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cult_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_cle_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cle_d, "V4SLLiV4dV4d", "nc", "lasx")

````
- **L793 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L793 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L794 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L794 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L795 EN**: Blank line separating nearby declarations or logic blocks.
  **L795 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L796 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L796 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L797 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L797 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L798 EN**: Blank line separating nearby declarations or logic blocks.
  **L798 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L799 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L799 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L800 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L800 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L801 EN**: Blank line separating nearby declarations or logic blocks.
  **L801 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L802 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L802 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L803 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L803 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L804 EN**: Blank line separating nearby declarations or logic blocks.
  **L804 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L805 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L805 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L806 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L806 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L807 EN**: Blank line separating nearby declarations or logic blocks.
  **L807 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L808 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L808 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L809 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L809 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L810 EN**: Blank line separating nearby declarations or logic blocks.
  **L810 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L811 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L811 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L812 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L812 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L813 EN**: Blank line separating nearby declarations or logic blocks.
  **L813 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L814 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L814 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L815 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L815 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 817-840

````cpp
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cule_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cule_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_cne_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cne_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_cor_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cor_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_cune_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_cune_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_saf_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_saf_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sun_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sun_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_seq_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_seq_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sueq_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sueq_d, "V4SLLiV4dV4d", "nc", "lasx")

````
- **L817 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L817 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L818 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L818 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L820 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L820 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L821 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L821 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L823 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L824 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L824 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L826 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L827 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L827 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L828 EN**: Blank line separating nearby declarations or logic blocks.
  **L828 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L829 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L829 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L830 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L830 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L831 EN**: Blank line separating nearby declarations or logic blocks.
  **L831 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L832 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L832 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L833 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L833 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L834 EN**: Blank line separating nearby declarations or logic blocks.
  **L834 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L835 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L835 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L836 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L836 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L837 EN**: Blank line separating nearby declarations or logic blocks.
  **L837 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L838 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L838 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L839 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L839 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L840 EN**: Blank line separating nearby declarations or logic blocks.
  **L840 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 841-864

````cpp
TARGET_BUILTIN(__builtin_lasx_xvfcmp_slt_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_slt_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sult_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sult_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sle_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sle_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sule_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sule_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sne_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sne_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sor_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sor_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvfcmp_sune_s, "V8SiV8fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvfcmp_sune_d, "V4SLLiV4dV4d", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbitsel_v, "V32UcV32UcV32UcV32Uc", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbitseli_b, "V32UcV32UcV32UcIUi", "nc", "lasx")
````
- **L841 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L841 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L842 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L842 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L843 EN**: Blank line separating nearby declarations or logic blocks.
  **L843 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L844 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L844 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L845 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L845 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L846 EN**: Blank line separating nearby declarations or logic blocks.
  **L846 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L847 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L847 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L848 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L848 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L849 EN**: Blank line separating nearby declarations or logic blocks.
  **L849 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L850 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L850 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L851 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L851 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L852 EN**: Blank line separating nearby declarations or logic blocks.
  **L852 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L853 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L853 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L854 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L854 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L855 EN**: Blank line separating nearby declarations or logic blocks.
  **L855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L856 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L856 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L857 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L857 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L858 EN**: Blank line separating nearby declarations or logic blocks.
  **L858 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L859 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L859 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L860 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L860 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L861 EN**: Blank line separating nearby declarations or logic blocks.
  **L861 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L862 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L862 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L863 EN**: Blank line separating nearby declarations or logic blocks.
  **L863 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L864 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L864 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 865-888

````cpp

TARGET_BUILTIN(__builtin_lasx_xvreplgr2vr_b, "V32Sci", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplgr2vr_h, "V16Ssi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplgr2vr_w, "V8Sii", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplgr2vr_d, "V4SLLiLLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvinsgr2vr_w, "V8SiV8SiiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvinsgr2vr_d, "V4SLLiV4SLLiLLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpickve2gr_w, "iV8SiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickve2gr_d, "LLiV4SLLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpickve2gr_wu, "iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickve2gr_du, "LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvreplve_b, "V32ScV32ScUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplve_h, "V16sV16sUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplve_w, "V8iV8iUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplve_d, "V4LLiV4LLiUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvrepl128vei_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrepl128vei_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrepl128vei_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvrepl128vei_d, "V4LLiV4LLiIUi", "nc", "lasx")
````
- **L865 EN**: Blank line separating nearby declarations or logic blocks.
  **L865 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L866 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L866 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L867 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L867 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L868 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L868 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L869 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L869 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L870 EN**: Blank line separating nearby declarations or logic blocks.
  **L870 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L871 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L871 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L872 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L872 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L873 EN**: Blank line separating nearby declarations or logic blocks.
  **L873 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L874 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L874 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L875 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L875 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L876 EN**: Blank line separating nearby declarations or logic blocks.
  **L876 CN**: 空行，用于分隔附近的声明或逻辑块。
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
- **L882 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L882 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L883 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L883 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L884 EN**: Blank line separating nearby declarations or logic blocks.
  **L884 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L885 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L885 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L886 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L886 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L887 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L887 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L888 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L888 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 889-912

````cpp

TARGET_BUILTIN(__builtin_lasx_xvreplve0_b, "V32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplve0_h, "V16SsV16Ss", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplve0_w, "V8SiV8Si", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplve0_d, "V4SLLiV4SLLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvreplve0_q, "V32ScV32Sc", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvinsve0_w, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvinsve0_d, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpickve_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickve_d, "V4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpickve_w_f, "V8fV8fIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickve_d_f, "V4dV4dIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvbsll_v, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvbsrl_v, "V32ScV32ScIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpackev_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpackev_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpackev_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpackev_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

````
- **L889 EN**: Blank line separating nearby declarations or logic blocks.
  **L889 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L890 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L890 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L891 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L891 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L892 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L892 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L893 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L893 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L894 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L894 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L895 EN**: Blank line separating nearby declarations or logic blocks.
  **L895 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L896 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L896 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L897 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L897 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L898 EN**: Blank line separating nearby declarations or logic blocks.
  **L898 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L899 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L899 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L900 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L900 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L901 EN**: Blank line separating nearby declarations or logic blocks.
  **L901 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L902 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L902 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L903 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L903 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L904 EN**: Blank line separating nearby declarations or logic blocks.
  **L904 CN**: 空行，用于分隔附近的声明或逻辑块。
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
TARGET_BUILTIN(__builtin_lasx_xvpackod_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpackod_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpackod_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpackod_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpickev_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickev_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickev_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickev_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpickod_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickod_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickod_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpickod_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvilvl_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvilvl_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvilvl_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvilvl_d, "V4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvilvh_b, "V32ScV32ScV32Sc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvilvh_h, "V16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvilvh_w, "V8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvilvh_d, "V4LLiV4LLiV4LLi", "nc", "lasx")
````
- **L913 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L913 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L914 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L914 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L915 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L915 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L916 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L916 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L917 EN**: Blank line separating nearby declarations or logic blocks.
  **L917 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L918 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L918 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L919 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L919 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L920 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L920 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L921 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L921 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L922 EN**: Blank line separating nearby declarations or logic blocks.
  **L922 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L923 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L923 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L924 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L924 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L925 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L925 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L926 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L926 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L927 EN**: Blank line separating nearby declarations or logic blocks.
  **L927 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L928 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L928 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L929 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L929 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L930 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L930 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L931 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L931 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L932 EN**: Blank line separating nearby declarations or logic blocks.
  **L932 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L933 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L933 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L934 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L934 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L935 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L935 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L936 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L936 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 937-960

````cpp

TARGET_BUILTIN(__builtin_lasx_xvshuf_b, "V32ScV32ScV32ScV32Sc", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvshuf_h, "V16sV16sV16sV16s", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvshuf_w, "V8iV8iV8iV8i", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvshuf_d, "V4LLiV4LLiV4LLiV4LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvperm_w, "V8iV8iV8i", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvshuf4i_b, "V32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvshuf4i_h, "V16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvshuf4i_w, "V8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvshuf4i_d, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvpermi_w, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpermi_d, "V4LLiV4LLiIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvpermi_q, "V32ScV32ScV32ScIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvextrins_b, "V32ScV32ScV32ScIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvextrins_h, "V16sV16sV16sIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvextrins_w, "V8iV8iV8iIUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvextrins_d, "V4LLiV4LLiV4LLiIUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvld, "V32ScvC*Ii", "nc", "lasx")
````
- **L937 EN**: Blank line separating nearby declarations or logic blocks.
  **L937 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L938 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L938 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L939 EN**: Blank line separating nearby declarations or logic blocks.
  **L939 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L940 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L940 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L941 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L941 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L942 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L942 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L943 EN**: Blank line separating nearby declarations or logic blocks.
  **L943 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L944 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L944 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L945 EN**: Blank line separating nearby declarations or logic blocks.
  **L945 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L946 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L946 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L947 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L947 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L948 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L948 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
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
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L955 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L955 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L956 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L956 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L957 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L957 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L958 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L958 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L959 EN**: Blank line separating nearby declarations or logic blocks.
  **L959 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L960 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L960 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

### Lines 961-984

````cpp
TARGET_BUILTIN(__builtin_lasx_xvst, "vV32Scv*Ii", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvldx, "V32ScvC*LLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvstx, "vV32Scv*LLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvldrepl_b, "V32ScvC*Ii", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvldrepl_h, "V16svC*Ii", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvldrepl_w, "V8ivC*Ii", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvldrepl_d, "V4LLivC*Ii", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xvstelm_b, "vV32Scv*IiUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvstelm_h, "vV16Ssv*IiUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvstelm_w, "vV8Siv*IiUi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xvstelm_d, "vV4SLLiv*IiUi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xbz_v, "iV32Uc", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xbnz_v, "iV32Uc", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_xbz_b, "iV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xbz_h, "iV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xbz_w, "iV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xbz_d, "iV4ULLi", "nc", "lasx")

````
- **L961 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L961 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L963 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L964 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L964 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L965 EN**: Blank line separating nearby declarations or logic blocks.
  **L965 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L966 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L966 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L967 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L967 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L968 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L968 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L969 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L969 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L970 EN**: Blank line separating nearby declarations or logic blocks.
  **L970 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L971 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L971 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L972 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L972 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L973 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L973 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L974 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L974 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L975 EN**: Blank line separating nearby declarations or logic blocks.
  **L975 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L976 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L976 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L977 EN**: Blank line separating nearby declarations or logic blocks.
  **L977 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L978 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L978 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L979 EN**: Blank line separating nearby declarations or logic blocks.
  **L979 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L980 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L980 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L981 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L981 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L982 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L982 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L983 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L983 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L984 EN**: Blank line separating nearby declarations or logic blocks.
  **L984 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 985-1007

````cpp
TARGET_BUILTIN(__builtin_lasx_xbnz_b, "iV32Uc", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xbnz_h, "iV16Us", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xbnz_w, "iV8Ui", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_xbnz_d, "iV4ULLi", "nc", "lasx")

TARGET_BUILTIN(__builtin_lasx_cast_128_s, "V8fV4f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_cast_128_d, "V4dV2d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_cast_128, "V4LLiV2LLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_concat_128_s, "V8fV4fV4f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_concat_128_d, "V4dV2dV2d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_concat_128, "V4LLiV2LLiV2LLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_extract_128_lo_s, "V4fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_extract_128_lo_d, "V2dV4d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_extract_128_lo, "V2LLiV4LLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_extract_128_hi_s, "V4fV8f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_extract_128_hi_d, "V2dV4d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_extract_128_hi, "V2LLiV4LLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_insert_128_lo_s, "V8fV8fV4f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_insert_128_lo_d, "V4dV4dV2d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_insert_128_lo, "V4LLiV4LLiV2LLi", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_insert_128_hi_s, "V8fV8fV4f", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_insert_128_hi_d, "V4dV4dV2d", "nc", "lasx")
TARGET_BUILTIN(__builtin_lasx_insert_128_hi, "V4LLiV4LLiV2LLi", "nc", "lasx")
````
- **L985 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L985 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L986 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L986 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L987 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L987 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L988 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L988 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L989 EN**: Blank line separating nearby declarations or logic blocks.
  **L989 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L990 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L990 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L991 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L991 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L992 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L992 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L993 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L993 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L994 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L994 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L995 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L995 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L996 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L996 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L997 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L997 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L998 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L998 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L999 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L999 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1000 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1000 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1001 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1001 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1002 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1002 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1003 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1003 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1004 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1004 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1005 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1005 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1006 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1006 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。
- **L1007 EN**: Invokes macro `TARGET_BUILTIN` to contribute one entry to a table-driven definition list.
  **L1007 CN**: 调用宏 `TARGET_BUILTIN`，向表驱动定义列表贡献一个条目。

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
