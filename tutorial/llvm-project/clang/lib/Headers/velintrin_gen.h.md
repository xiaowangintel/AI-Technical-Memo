# velintrin_gen.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/lib/Headers/velintrin_gen.h`
- **Repository / 仓库**: `llvm-project`
- **Purpose (EN)**: This header supplies ve vector interfaces for `velintrin_gen` inside Clang's public header set.
- **Purpose (CN)**: 该头文件在 Clang 公共头文件集合中为 `velintrin_gen` 提供VE 向量接口。
- **Line Count / 行数**: 1257

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````c
#define _vel_vld_vssl __builtin_ve_vl_vld_vssl
#define _vel_vld_vssvl __builtin_ve_vl_vld_vssvl
#define _vel_vldnc_vssl __builtin_ve_vl_vldnc_vssl
#define _vel_vldnc_vssvl __builtin_ve_vl_vldnc_vssvl
#define _vel_vldu_vssl __builtin_ve_vl_vldu_vssl
#define _vel_vldu_vssvl __builtin_ve_vl_vldu_vssvl
#define _vel_vldunc_vssl __builtin_ve_vl_vldunc_vssl
#define _vel_vldunc_vssvl __builtin_ve_vl_vldunc_vssvl
#define _vel_vldlsx_vssl __builtin_ve_vl_vldlsx_vssl
#define _vel_vldlsx_vssvl __builtin_ve_vl_vldlsx_vssvl
#define _vel_vldlsxnc_vssl __builtin_ve_vl_vldlsxnc_vssl
#define _vel_vldlsxnc_vssvl __builtin_ve_vl_vldlsxnc_vssvl
#define _vel_vldlzx_vssl __builtin_ve_vl_vldlzx_vssl
#define _vel_vldlzx_vssvl __builtin_ve_vl_vldlzx_vssvl
#define _vel_vldlzxnc_vssl __builtin_ve_vl_vldlzxnc_vssl
#define _vel_vldlzxnc_vssvl __builtin_ve_vl_vldlzxnc_vssvl
#define _vel_vld2d_vssl __builtin_ve_vl_vld2d_vssl
#define _vel_vld2d_vssvl __builtin_ve_vl_vld2d_vssvl
#define _vel_vld2dnc_vssl __builtin_ve_vl_vld2dnc_vssl
#define _vel_vld2dnc_vssvl __builtin_ve_vl_vld2dnc_vssvl
#define _vel_vldu2d_vssl __builtin_ve_vl_vldu2d_vssl
#define _vel_vldu2d_vssvl __builtin_ve_vl_vldu2d_vssvl
#define _vel_vldu2dnc_vssl __builtin_ve_vl_vldu2dnc_vssl
#define _vel_vldu2dnc_vssvl __builtin_ve_vl_vldu2dnc_vssvl
````
- **L1 EN**: Defines macro `_vel_vld_vssl` for conditional compilation, shorthand, or API generation.
  **L1 CN**: 定义宏 `_vel_vld_vssl`，用于条件编译、简写或 API 生成。
- **L2 EN**: Defines macro `_vel_vld_vssvl` for conditional compilation, shorthand, or API generation.
  **L2 CN**: 定义宏 `_vel_vld_vssvl`，用于条件编译、简写或 API 生成。
- **L3 EN**: Defines macro `_vel_vldnc_vssl` for conditional compilation, shorthand, or API generation.
  **L3 CN**: 定义宏 `_vel_vldnc_vssl`，用于条件编译、简写或 API 生成。
- **L4 EN**: Defines macro `_vel_vldnc_vssvl` for conditional compilation, shorthand, or API generation.
  **L4 CN**: 定义宏 `_vel_vldnc_vssvl`，用于条件编译、简写或 API 生成。
- **L5 EN**: Defines macro `_vel_vldu_vssl` for conditional compilation, shorthand, or API generation.
  **L5 CN**: 定义宏 `_vel_vldu_vssl`，用于条件编译、简写或 API 生成。
- **L6 EN**: Defines macro `_vel_vldu_vssvl` for conditional compilation, shorthand, or API generation.
  **L6 CN**: 定义宏 `_vel_vldu_vssvl`，用于条件编译、简写或 API 生成。
- **L7 EN**: Defines macro `_vel_vldunc_vssl` for conditional compilation, shorthand, or API generation.
  **L7 CN**: 定义宏 `_vel_vldunc_vssl`，用于条件编译、简写或 API 生成。
- **L8 EN**: Defines macro `_vel_vldunc_vssvl` for conditional compilation, shorthand, or API generation.
  **L8 CN**: 定义宏 `_vel_vldunc_vssvl`，用于条件编译、简写或 API 生成。
- **L9 EN**: Defines macro `_vel_vldlsx_vssl` for conditional compilation, shorthand, or API generation.
  **L9 CN**: 定义宏 `_vel_vldlsx_vssl`，用于条件编译、简写或 API 生成。
- **L10 EN**: Defines macro `_vel_vldlsx_vssvl` for conditional compilation, shorthand, or API generation.
  **L10 CN**: 定义宏 `_vel_vldlsx_vssvl`，用于条件编译、简写或 API 生成。
- **L11 EN**: Defines macro `_vel_vldlsxnc_vssl` for conditional compilation, shorthand, or API generation.
  **L11 CN**: 定义宏 `_vel_vldlsxnc_vssl`，用于条件编译、简写或 API 生成。
- **L12 EN**: Defines macro `_vel_vldlsxnc_vssvl` for conditional compilation, shorthand, or API generation.
  **L12 CN**: 定义宏 `_vel_vldlsxnc_vssvl`，用于条件编译、简写或 API 生成。
- **L13 EN**: Defines macro `_vel_vldlzx_vssl` for conditional compilation, shorthand, or API generation.
  **L13 CN**: 定义宏 `_vel_vldlzx_vssl`，用于条件编译、简写或 API 生成。
- **L14 EN**: Defines macro `_vel_vldlzx_vssvl` for conditional compilation, shorthand, or API generation.
  **L14 CN**: 定义宏 `_vel_vldlzx_vssvl`，用于条件编译、简写或 API 生成。
- **L15 EN**: Defines macro `_vel_vldlzxnc_vssl` for conditional compilation, shorthand, or API generation.
  **L15 CN**: 定义宏 `_vel_vldlzxnc_vssl`，用于条件编译、简写或 API 生成。
- **L16 EN**: Defines macro `_vel_vldlzxnc_vssvl` for conditional compilation, shorthand, or API generation.
  **L16 CN**: 定义宏 `_vel_vldlzxnc_vssvl`，用于条件编译、简写或 API 生成。
- **L17 EN**: Defines macro `_vel_vld2d_vssl` for conditional compilation, shorthand, or API generation.
  **L17 CN**: 定义宏 `_vel_vld2d_vssl`，用于条件编译、简写或 API 生成。
- **L18 EN**: Defines macro `_vel_vld2d_vssvl` for conditional compilation, shorthand, or API generation.
  **L18 CN**: 定义宏 `_vel_vld2d_vssvl`，用于条件编译、简写或 API 生成。
- **L19 EN**: Defines macro `_vel_vld2dnc_vssl` for conditional compilation, shorthand, or API generation.
  **L19 CN**: 定义宏 `_vel_vld2dnc_vssl`，用于条件编译、简写或 API 生成。
- **L20 EN**: Defines macro `_vel_vld2dnc_vssvl` for conditional compilation, shorthand, or API generation.
  **L20 CN**: 定义宏 `_vel_vld2dnc_vssvl`，用于条件编译、简写或 API 生成。
- **L21 EN**: Defines macro `_vel_vldu2d_vssl` for conditional compilation, shorthand, or API generation.
  **L21 CN**: 定义宏 `_vel_vldu2d_vssl`，用于条件编译、简写或 API 生成。
- **L22 EN**: Defines macro `_vel_vldu2d_vssvl` for conditional compilation, shorthand, or API generation.
  **L22 CN**: 定义宏 `_vel_vldu2d_vssvl`，用于条件编译、简写或 API 生成。
- **L23 EN**: Defines macro `_vel_vldu2dnc_vssl` for conditional compilation, shorthand, or API generation.
  **L23 CN**: 定义宏 `_vel_vldu2dnc_vssl`，用于条件编译、简写或 API 生成。
- **L24 EN**: Defines macro `_vel_vldu2dnc_vssvl` for conditional compilation, shorthand, or API generation.
  **L24 CN**: 定义宏 `_vel_vldu2dnc_vssvl`，用于条件编译、简写或 API 生成。

### Lines 25-48

````c
#define _vel_vldl2dsx_vssl __builtin_ve_vl_vldl2dsx_vssl
#define _vel_vldl2dsx_vssvl __builtin_ve_vl_vldl2dsx_vssvl
#define _vel_vldl2dsxnc_vssl __builtin_ve_vl_vldl2dsxnc_vssl
#define _vel_vldl2dsxnc_vssvl __builtin_ve_vl_vldl2dsxnc_vssvl
#define _vel_vldl2dzx_vssl __builtin_ve_vl_vldl2dzx_vssl
#define _vel_vldl2dzx_vssvl __builtin_ve_vl_vldl2dzx_vssvl
#define _vel_vldl2dzxnc_vssl __builtin_ve_vl_vldl2dzxnc_vssl
#define _vel_vldl2dzxnc_vssvl __builtin_ve_vl_vldl2dzxnc_vssvl
#define _vel_vst_vssl __builtin_ve_vl_vst_vssl
#define _vel_vst_vssml __builtin_ve_vl_vst_vssml
#define _vel_vstnc_vssl __builtin_ve_vl_vstnc_vssl
#define _vel_vstnc_vssml __builtin_ve_vl_vstnc_vssml
#define _vel_vstot_vssl __builtin_ve_vl_vstot_vssl
#define _vel_vstot_vssml __builtin_ve_vl_vstot_vssml
#define _vel_vstncot_vssl __builtin_ve_vl_vstncot_vssl
#define _vel_vstncot_vssml __builtin_ve_vl_vstncot_vssml
#define _vel_vstu_vssl __builtin_ve_vl_vstu_vssl
#define _vel_vstu_vssml __builtin_ve_vl_vstu_vssml
#define _vel_vstunc_vssl __builtin_ve_vl_vstunc_vssl
#define _vel_vstunc_vssml __builtin_ve_vl_vstunc_vssml
#define _vel_vstuot_vssl __builtin_ve_vl_vstuot_vssl
#define _vel_vstuot_vssml __builtin_ve_vl_vstuot_vssml
#define _vel_vstuncot_vssl __builtin_ve_vl_vstuncot_vssl
#define _vel_vstuncot_vssml __builtin_ve_vl_vstuncot_vssml
````
- **L25 EN**: Defines macro `_vel_vldl2dsx_vssl` for conditional compilation, shorthand, or API generation.
  **L25 CN**: 定义宏 `_vel_vldl2dsx_vssl`，用于条件编译、简写或 API 生成。
- **L26 EN**: Defines macro `_vel_vldl2dsx_vssvl` for conditional compilation, shorthand, or API generation.
  **L26 CN**: 定义宏 `_vel_vldl2dsx_vssvl`，用于条件编译、简写或 API 生成。
- **L27 EN**: Defines macro `_vel_vldl2dsxnc_vssl` for conditional compilation, shorthand, or API generation.
  **L27 CN**: 定义宏 `_vel_vldl2dsxnc_vssl`，用于条件编译、简写或 API 生成。
- **L28 EN**: Defines macro `_vel_vldl2dsxnc_vssvl` for conditional compilation, shorthand, or API generation.
  **L28 CN**: 定义宏 `_vel_vldl2dsxnc_vssvl`，用于条件编译、简写或 API 生成。
- **L29 EN**: Defines macro `_vel_vldl2dzx_vssl` for conditional compilation, shorthand, or API generation.
  **L29 CN**: 定义宏 `_vel_vldl2dzx_vssl`，用于条件编译、简写或 API 生成。
- **L30 EN**: Defines macro `_vel_vldl2dzx_vssvl` for conditional compilation, shorthand, or API generation.
  **L30 CN**: 定义宏 `_vel_vldl2dzx_vssvl`，用于条件编译、简写或 API 生成。
- **L31 EN**: Defines macro `_vel_vldl2dzxnc_vssl` for conditional compilation, shorthand, or API generation.
  **L31 CN**: 定义宏 `_vel_vldl2dzxnc_vssl`，用于条件编译、简写或 API 生成。
- **L32 EN**: Defines macro `_vel_vldl2dzxnc_vssvl` for conditional compilation, shorthand, or API generation.
  **L32 CN**: 定义宏 `_vel_vldl2dzxnc_vssvl`，用于条件编译、简写或 API 生成。
- **L33 EN**: Defines macro `_vel_vst_vssl` for conditional compilation, shorthand, or API generation.
  **L33 CN**: 定义宏 `_vel_vst_vssl`，用于条件编译、简写或 API 生成。
- **L34 EN**: Defines macro `_vel_vst_vssml` for conditional compilation, shorthand, or API generation.
  **L34 CN**: 定义宏 `_vel_vst_vssml`，用于条件编译、简写或 API 生成。
- **L35 EN**: Defines macro `_vel_vstnc_vssl` for conditional compilation, shorthand, or API generation.
  **L35 CN**: 定义宏 `_vel_vstnc_vssl`，用于条件编译、简写或 API 生成。
- **L36 EN**: Defines macro `_vel_vstnc_vssml` for conditional compilation, shorthand, or API generation.
  **L36 CN**: 定义宏 `_vel_vstnc_vssml`，用于条件编译、简写或 API 生成。
- **L37 EN**: Defines macro `_vel_vstot_vssl` for conditional compilation, shorthand, or API generation.
  **L37 CN**: 定义宏 `_vel_vstot_vssl`，用于条件编译、简写或 API 生成。
- **L38 EN**: Defines macro `_vel_vstot_vssml` for conditional compilation, shorthand, or API generation.
  **L38 CN**: 定义宏 `_vel_vstot_vssml`，用于条件编译、简写或 API 生成。
- **L39 EN**: Defines macro `_vel_vstncot_vssl` for conditional compilation, shorthand, or API generation.
  **L39 CN**: 定义宏 `_vel_vstncot_vssl`，用于条件编译、简写或 API 生成。
- **L40 EN**: Defines macro `_vel_vstncot_vssml` for conditional compilation, shorthand, or API generation.
  **L40 CN**: 定义宏 `_vel_vstncot_vssml`，用于条件编译、简写或 API 生成。
- **L41 EN**: Defines macro `_vel_vstu_vssl` for conditional compilation, shorthand, or API generation.
  **L41 CN**: 定义宏 `_vel_vstu_vssl`，用于条件编译、简写或 API 生成。
- **L42 EN**: Defines macro `_vel_vstu_vssml` for conditional compilation, shorthand, or API generation.
  **L42 CN**: 定义宏 `_vel_vstu_vssml`，用于条件编译、简写或 API 生成。
- **L43 EN**: Defines macro `_vel_vstunc_vssl` for conditional compilation, shorthand, or API generation.
  **L43 CN**: 定义宏 `_vel_vstunc_vssl`，用于条件编译、简写或 API 生成。
- **L44 EN**: Defines macro `_vel_vstunc_vssml` for conditional compilation, shorthand, or API generation.
  **L44 CN**: 定义宏 `_vel_vstunc_vssml`，用于条件编译、简写或 API 生成。
- **L45 EN**: Defines macro `_vel_vstuot_vssl` for conditional compilation, shorthand, or API generation.
  **L45 CN**: 定义宏 `_vel_vstuot_vssl`，用于条件编译、简写或 API 生成。
- **L46 EN**: Defines macro `_vel_vstuot_vssml` for conditional compilation, shorthand, or API generation.
  **L46 CN**: 定义宏 `_vel_vstuot_vssml`，用于条件编译、简写或 API 生成。
- **L47 EN**: Defines macro `_vel_vstuncot_vssl` for conditional compilation, shorthand, or API generation.
  **L47 CN**: 定义宏 `_vel_vstuncot_vssl`，用于条件编译、简写或 API 生成。
- **L48 EN**: Defines macro `_vel_vstuncot_vssml` for conditional compilation, shorthand, or API generation.
  **L48 CN**: 定义宏 `_vel_vstuncot_vssml`，用于条件编译、简写或 API 生成。

### Lines 49-72

````c
#define _vel_vstl_vssl __builtin_ve_vl_vstl_vssl
#define _vel_vstl_vssml __builtin_ve_vl_vstl_vssml
#define _vel_vstlnc_vssl __builtin_ve_vl_vstlnc_vssl
#define _vel_vstlnc_vssml __builtin_ve_vl_vstlnc_vssml
#define _vel_vstlot_vssl __builtin_ve_vl_vstlot_vssl
#define _vel_vstlot_vssml __builtin_ve_vl_vstlot_vssml
#define _vel_vstlncot_vssl __builtin_ve_vl_vstlncot_vssl
#define _vel_vstlncot_vssml __builtin_ve_vl_vstlncot_vssml
#define _vel_vst2d_vssl __builtin_ve_vl_vst2d_vssl
#define _vel_vst2d_vssml __builtin_ve_vl_vst2d_vssml
#define _vel_vst2dnc_vssl __builtin_ve_vl_vst2dnc_vssl
#define _vel_vst2dnc_vssml __builtin_ve_vl_vst2dnc_vssml
#define _vel_vst2dot_vssl __builtin_ve_vl_vst2dot_vssl
#define _vel_vst2dot_vssml __builtin_ve_vl_vst2dot_vssml
#define _vel_vst2dncot_vssl __builtin_ve_vl_vst2dncot_vssl
#define _vel_vst2dncot_vssml __builtin_ve_vl_vst2dncot_vssml
#define _vel_vstu2d_vssl __builtin_ve_vl_vstu2d_vssl
#define _vel_vstu2d_vssml __builtin_ve_vl_vstu2d_vssml
#define _vel_vstu2dnc_vssl __builtin_ve_vl_vstu2dnc_vssl
#define _vel_vstu2dnc_vssml __builtin_ve_vl_vstu2dnc_vssml
#define _vel_vstu2dot_vssl __builtin_ve_vl_vstu2dot_vssl
#define _vel_vstu2dot_vssml __builtin_ve_vl_vstu2dot_vssml
#define _vel_vstu2dncot_vssl __builtin_ve_vl_vstu2dncot_vssl
#define _vel_vstu2dncot_vssml __builtin_ve_vl_vstu2dncot_vssml
````
- **L49 EN**: Defines macro `_vel_vstl_vssl` for conditional compilation, shorthand, or API generation.
  **L49 CN**: 定义宏 `_vel_vstl_vssl`，用于条件编译、简写或 API 生成。
- **L50 EN**: Defines macro `_vel_vstl_vssml` for conditional compilation, shorthand, or API generation.
  **L50 CN**: 定义宏 `_vel_vstl_vssml`，用于条件编译、简写或 API 生成。
- **L51 EN**: Defines macro `_vel_vstlnc_vssl` for conditional compilation, shorthand, or API generation.
  **L51 CN**: 定义宏 `_vel_vstlnc_vssl`，用于条件编译、简写或 API 生成。
- **L52 EN**: Defines macro `_vel_vstlnc_vssml` for conditional compilation, shorthand, or API generation.
  **L52 CN**: 定义宏 `_vel_vstlnc_vssml`，用于条件编译、简写或 API 生成。
- **L53 EN**: Defines macro `_vel_vstlot_vssl` for conditional compilation, shorthand, or API generation.
  **L53 CN**: 定义宏 `_vel_vstlot_vssl`，用于条件编译、简写或 API 生成。
- **L54 EN**: Defines macro `_vel_vstlot_vssml` for conditional compilation, shorthand, or API generation.
  **L54 CN**: 定义宏 `_vel_vstlot_vssml`，用于条件编译、简写或 API 生成。
- **L55 EN**: Defines macro `_vel_vstlncot_vssl` for conditional compilation, shorthand, or API generation.
  **L55 CN**: 定义宏 `_vel_vstlncot_vssl`，用于条件编译、简写或 API 生成。
- **L56 EN**: Defines macro `_vel_vstlncot_vssml` for conditional compilation, shorthand, or API generation.
  **L56 CN**: 定义宏 `_vel_vstlncot_vssml`，用于条件编译、简写或 API 生成。
- **L57 EN**: Defines macro `_vel_vst2d_vssl` for conditional compilation, shorthand, or API generation.
  **L57 CN**: 定义宏 `_vel_vst2d_vssl`，用于条件编译、简写或 API 生成。
- **L58 EN**: Defines macro `_vel_vst2d_vssml` for conditional compilation, shorthand, or API generation.
  **L58 CN**: 定义宏 `_vel_vst2d_vssml`，用于条件编译、简写或 API 生成。
- **L59 EN**: Defines macro `_vel_vst2dnc_vssl` for conditional compilation, shorthand, or API generation.
  **L59 CN**: 定义宏 `_vel_vst2dnc_vssl`，用于条件编译、简写或 API 生成。
- **L60 EN**: Defines macro `_vel_vst2dnc_vssml` for conditional compilation, shorthand, or API generation.
  **L60 CN**: 定义宏 `_vel_vst2dnc_vssml`，用于条件编译、简写或 API 生成。
- **L61 EN**: Defines macro `_vel_vst2dot_vssl` for conditional compilation, shorthand, or API generation.
  **L61 CN**: 定义宏 `_vel_vst2dot_vssl`，用于条件编译、简写或 API 生成。
- **L62 EN**: Defines macro `_vel_vst2dot_vssml` for conditional compilation, shorthand, or API generation.
  **L62 CN**: 定义宏 `_vel_vst2dot_vssml`，用于条件编译、简写或 API 生成。
- **L63 EN**: Defines macro `_vel_vst2dncot_vssl` for conditional compilation, shorthand, or API generation.
  **L63 CN**: 定义宏 `_vel_vst2dncot_vssl`，用于条件编译、简写或 API 生成。
- **L64 EN**: Defines macro `_vel_vst2dncot_vssml` for conditional compilation, shorthand, or API generation.
  **L64 CN**: 定义宏 `_vel_vst2dncot_vssml`，用于条件编译、简写或 API 生成。
- **L65 EN**: Defines macro `_vel_vstu2d_vssl` for conditional compilation, shorthand, or API generation.
  **L65 CN**: 定义宏 `_vel_vstu2d_vssl`，用于条件编译、简写或 API 生成。
- **L66 EN**: Defines macro `_vel_vstu2d_vssml` for conditional compilation, shorthand, or API generation.
  **L66 CN**: 定义宏 `_vel_vstu2d_vssml`，用于条件编译、简写或 API 生成。
- **L67 EN**: Defines macro `_vel_vstu2dnc_vssl` for conditional compilation, shorthand, or API generation.
  **L67 CN**: 定义宏 `_vel_vstu2dnc_vssl`，用于条件编译、简写或 API 生成。
- **L68 EN**: Defines macro `_vel_vstu2dnc_vssml` for conditional compilation, shorthand, or API generation.
  **L68 CN**: 定义宏 `_vel_vstu2dnc_vssml`，用于条件编译、简写或 API 生成。
- **L69 EN**: Defines macro `_vel_vstu2dot_vssl` for conditional compilation, shorthand, or API generation.
  **L69 CN**: 定义宏 `_vel_vstu2dot_vssl`，用于条件编译、简写或 API 生成。
- **L70 EN**: Defines macro `_vel_vstu2dot_vssml` for conditional compilation, shorthand, or API generation.
  **L70 CN**: 定义宏 `_vel_vstu2dot_vssml`，用于条件编译、简写或 API 生成。
- **L71 EN**: Defines macro `_vel_vstu2dncot_vssl` for conditional compilation, shorthand, or API generation.
  **L71 CN**: 定义宏 `_vel_vstu2dncot_vssl`，用于条件编译、简写或 API 生成。
- **L72 EN**: Defines macro `_vel_vstu2dncot_vssml` for conditional compilation, shorthand, or API generation.
  **L72 CN**: 定义宏 `_vel_vstu2dncot_vssml`，用于条件编译、简写或 API 生成。

### Lines 73-96

````c
#define _vel_vstl2d_vssl __builtin_ve_vl_vstl2d_vssl
#define _vel_vstl2d_vssml __builtin_ve_vl_vstl2d_vssml
#define _vel_vstl2dnc_vssl __builtin_ve_vl_vstl2dnc_vssl
#define _vel_vstl2dnc_vssml __builtin_ve_vl_vstl2dnc_vssml
#define _vel_vstl2dot_vssl __builtin_ve_vl_vstl2dot_vssl
#define _vel_vstl2dot_vssml __builtin_ve_vl_vstl2dot_vssml
#define _vel_vstl2dncot_vssl __builtin_ve_vl_vstl2dncot_vssl
#define _vel_vstl2dncot_vssml __builtin_ve_vl_vstl2dncot_vssml
#define _vel_pfchv_ssl __builtin_ve_vl_pfchv_ssl
#define _vel_pfchvnc_ssl __builtin_ve_vl_pfchvnc_ssl
#define _vel_lsv_vvss __builtin_ve_vl_lsv_vvss
#define _vel_lvsl_svs __builtin_ve_vl_lvsl_svs
#define _vel_lvsd_svs __builtin_ve_vl_lvsd_svs
#define _vel_lvss_svs __builtin_ve_vl_lvss_svs
#define _vel_lvm_mmss __builtin_ve_vl_lvm_mmss
#define _vel_lvm_MMss __builtin_ve_vl_lvm_MMss
#define _vel_svm_sms __builtin_ve_vl_svm_sms
#define _vel_svm_sMs __builtin_ve_vl_svm_sMs
#define _vel_vbrdd_vsl __builtin_ve_vl_vbrdd_vsl
#define _vel_vbrdd_vsvl __builtin_ve_vl_vbrdd_vsvl
#define _vel_vbrdd_vsmvl __builtin_ve_vl_vbrdd_vsmvl
#define _vel_vbrdl_vsl __builtin_ve_vl_vbrdl_vsl
#define _vel_vbrdl_vsvl __builtin_ve_vl_vbrdl_vsvl
#define _vel_vbrdl_vsmvl __builtin_ve_vl_vbrdl_vsmvl
````
- **L73 EN**: Defines macro `_vel_vstl2d_vssl` for conditional compilation, shorthand, or API generation.
  **L73 CN**: 定义宏 `_vel_vstl2d_vssl`，用于条件编译、简写或 API 生成。
- **L74 EN**: Defines macro `_vel_vstl2d_vssml` for conditional compilation, shorthand, or API generation.
  **L74 CN**: 定义宏 `_vel_vstl2d_vssml`，用于条件编译、简写或 API 生成。
- **L75 EN**: Defines macro `_vel_vstl2dnc_vssl` for conditional compilation, shorthand, or API generation.
  **L75 CN**: 定义宏 `_vel_vstl2dnc_vssl`，用于条件编译、简写或 API 生成。
- **L76 EN**: Defines macro `_vel_vstl2dnc_vssml` for conditional compilation, shorthand, or API generation.
  **L76 CN**: 定义宏 `_vel_vstl2dnc_vssml`，用于条件编译、简写或 API 生成。
- **L77 EN**: Defines macro `_vel_vstl2dot_vssl` for conditional compilation, shorthand, or API generation.
  **L77 CN**: 定义宏 `_vel_vstl2dot_vssl`，用于条件编译、简写或 API 生成。
- **L78 EN**: Defines macro `_vel_vstl2dot_vssml` for conditional compilation, shorthand, or API generation.
  **L78 CN**: 定义宏 `_vel_vstl2dot_vssml`，用于条件编译、简写或 API 生成。
- **L79 EN**: Defines macro `_vel_vstl2dncot_vssl` for conditional compilation, shorthand, or API generation.
  **L79 CN**: 定义宏 `_vel_vstl2dncot_vssl`，用于条件编译、简写或 API 生成。
- **L80 EN**: Defines macro `_vel_vstl2dncot_vssml` for conditional compilation, shorthand, or API generation.
  **L80 CN**: 定义宏 `_vel_vstl2dncot_vssml`，用于条件编译、简写或 API 生成。
- **L81 EN**: Defines macro `_vel_pfchv_ssl` for conditional compilation, shorthand, or API generation.
  **L81 CN**: 定义宏 `_vel_pfchv_ssl`，用于条件编译、简写或 API 生成。
- **L82 EN**: Defines macro `_vel_pfchvnc_ssl` for conditional compilation, shorthand, or API generation.
  **L82 CN**: 定义宏 `_vel_pfchvnc_ssl`，用于条件编译、简写或 API 生成。
- **L83 EN**: Defines macro `_vel_lsv_vvss` for conditional compilation, shorthand, or API generation.
  **L83 CN**: 定义宏 `_vel_lsv_vvss`，用于条件编译、简写或 API 生成。
- **L84 EN**: Defines macro `_vel_lvsl_svs` for conditional compilation, shorthand, or API generation.
  **L84 CN**: 定义宏 `_vel_lvsl_svs`，用于条件编译、简写或 API 生成。
- **L85 EN**: Defines macro `_vel_lvsd_svs` for conditional compilation, shorthand, or API generation.
  **L85 CN**: 定义宏 `_vel_lvsd_svs`，用于条件编译、简写或 API 生成。
- **L86 EN**: Defines macro `_vel_lvss_svs` for conditional compilation, shorthand, or API generation.
  **L86 CN**: 定义宏 `_vel_lvss_svs`，用于条件编译、简写或 API 生成。
- **L87 EN**: Defines macro `_vel_lvm_mmss` for conditional compilation, shorthand, or API generation.
  **L87 CN**: 定义宏 `_vel_lvm_mmss`，用于条件编译、简写或 API 生成。
- **L88 EN**: Defines macro `_vel_lvm_MMss` for conditional compilation, shorthand, or API generation.
  **L88 CN**: 定义宏 `_vel_lvm_MMss`，用于条件编译、简写或 API 生成。
- **L89 EN**: Defines macro `_vel_svm_sms` for conditional compilation, shorthand, or API generation.
  **L89 CN**: 定义宏 `_vel_svm_sms`，用于条件编译、简写或 API 生成。
- **L90 EN**: Defines macro `_vel_svm_sMs` for conditional compilation, shorthand, or API generation.
  **L90 CN**: 定义宏 `_vel_svm_sMs`，用于条件编译、简写或 API 生成。
- **L91 EN**: Defines macro `_vel_vbrdd_vsl` for conditional compilation, shorthand, or API generation.
  **L91 CN**: 定义宏 `_vel_vbrdd_vsl`，用于条件编译、简写或 API 生成。
- **L92 EN**: Defines macro `_vel_vbrdd_vsvl` for conditional compilation, shorthand, or API generation.
  **L92 CN**: 定义宏 `_vel_vbrdd_vsvl`，用于条件编译、简写或 API 生成。
- **L93 EN**: Defines macro `_vel_vbrdd_vsmvl` for conditional compilation, shorthand, or API generation.
  **L93 CN**: 定义宏 `_vel_vbrdd_vsmvl`，用于条件编译、简写或 API 生成。
- **L94 EN**: Defines macro `_vel_vbrdl_vsl` for conditional compilation, shorthand, or API generation.
  **L94 CN**: 定义宏 `_vel_vbrdl_vsl`，用于条件编译、简写或 API 生成。
- **L95 EN**: Defines macro `_vel_vbrdl_vsvl` for conditional compilation, shorthand, or API generation.
  **L95 CN**: 定义宏 `_vel_vbrdl_vsvl`，用于条件编译、简写或 API 生成。
- **L96 EN**: Defines macro `_vel_vbrdl_vsmvl` for conditional compilation, shorthand, or API generation.
  **L96 CN**: 定义宏 `_vel_vbrdl_vsmvl`，用于条件编译、简写或 API 生成。

### Lines 97-120

````c
#define _vel_vbrds_vsl __builtin_ve_vl_vbrds_vsl
#define _vel_vbrds_vsvl __builtin_ve_vl_vbrds_vsvl
#define _vel_vbrds_vsmvl __builtin_ve_vl_vbrds_vsmvl
#define _vel_vbrdw_vsl __builtin_ve_vl_vbrdw_vsl
#define _vel_vbrdw_vsvl __builtin_ve_vl_vbrdw_vsvl
#define _vel_vbrdw_vsmvl __builtin_ve_vl_vbrdw_vsmvl
#define _vel_pvbrd_vsl __builtin_ve_vl_pvbrd_vsl
#define _vel_pvbrd_vsvl __builtin_ve_vl_pvbrd_vsvl
#define _vel_pvbrd_vsMvl __builtin_ve_vl_pvbrd_vsMvl
#define _vel_vmv_vsvl __builtin_ve_vl_vmv_vsvl
#define _vel_vmv_vsvvl __builtin_ve_vl_vmv_vsvvl
#define _vel_vmv_vsvmvl __builtin_ve_vl_vmv_vsvmvl
#define _vel_vaddul_vvvl __builtin_ve_vl_vaddul_vvvl
#define _vel_vaddul_vvvvl __builtin_ve_vl_vaddul_vvvvl
#define _vel_vaddul_vsvl __builtin_ve_vl_vaddul_vsvl
#define _vel_vaddul_vsvvl __builtin_ve_vl_vaddul_vsvvl
#define _vel_vaddul_vvvmvl __builtin_ve_vl_vaddul_vvvmvl
#define _vel_vaddul_vsvmvl __builtin_ve_vl_vaddul_vsvmvl
#define _vel_vadduw_vvvl __builtin_ve_vl_vadduw_vvvl
#define _vel_vadduw_vvvvl __builtin_ve_vl_vadduw_vvvvl
#define _vel_vadduw_vsvl __builtin_ve_vl_vadduw_vsvl
#define _vel_vadduw_vsvvl __builtin_ve_vl_vadduw_vsvvl
#define _vel_vadduw_vvvmvl __builtin_ve_vl_vadduw_vvvmvl
#define _vel_vadduw_vsvmvl __builtin_ve_vl_vadduw_vsvmvl
````
- **L97 EN**: Defines macro `_vel_vbrds_vsl` for conditional compilation, shorthand, or API generation.
  **L97 CN**: 定义宏 `_vel_vbrds_vsl`，用于条件编译、简写或 API 生成。
- **L98 EN**: Defines macro `_vel_vbrds_vsvl` for conditional compilation, shorthand, or API generation.
  **L98 CN**: 定义宏 `_vel_vbrds_vsvl`，用于条件编译、简写或 API 生成。
- **L99 EN**: Defines macro `_vel_vbrds_vsmvl` for conditional compilation, shorthand, or API generation.
  **L99 CN**: 定义宏 `_vel_vbrds_vsmvl`，用于条件编译、简写或 API 生成。
- **L100 EN**: Defines macro `_vel_vbrdw_vsl` for conditional compilation, shorthand, or API generation.
  **L100 CN**: 定义宏 `_vel_vbrdw_vsl`，用于条件编译、简写或 API 生成。
- **L101 EN**: Defines macro `_vel_vbrdw_vsvl` for conditional compilation, shorthand, or API generation.
  **L101 CN**: 定义宏 `_vel_vbrdw_vsvl`，用于条件编译、简写或 API 生成。
- **L102 EN**: Defines macro `_vel_vbrdw_vsmvl` for conditional compilation, shorthand, or API generation.
  **L102 CN**: 定义宏 `_vel_vbrdw_vsmvl`，用于条件编译、简写或 API 生成。
- **L103 EN**: Defines macro `_vel_pvbrd_vsl` for conditional compilation, shorthand, or API generation.
  **L103 CN**: 定义宏 `_vel_pvbrd_vsl`，用于条件编译、简写或 API 生成。
- **L104 EN**: Defines macro `_vel_pvbrd_vsvl` for conditional compilation, shorthand, or API generation.
  **L104 CN**: 定义宏 `_vel_pvbrd_vsvl`，用于条件编译、简写或 API 生成。
- **L105 EN**: Defines macro `_vel_pvbrd_vsMvl` for conditional compilation, shorthand, or API generation.
  **L105 CN**: 定义宏 `_vel_pvbrd_vsMvl`，用于条件编译、简写或 API 生成。
- **L106 EN**: Defines macro `_vel_vmv_vsvl` for conditional compilation, shorthand, or API generation.
  **L106 CN**: 定义宏 `_vel_vmv_vsvl`，用于条件编译、简写或 API 生成。
- **L107 EN**: Defines macro `_vel_vmv_vsvvl` for conditional compilation, shorthand, or API generation.
  **L107 CN**: 定义宏 `_vel_vmv_vsvvl`，用于条件编译、简写或 API 生成。
- **L108 EN**: Defines macro `_vel_vmv_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L108 CN**: 定义宏 `_vel_vmv_vsvmvl`，用于条件编译、简写或 API 生成。
- **L109 EN**: Defines macro `_vel_vaddul_vvvl` for conditional compilation, shorthand, or API generation.
  **L109 CN**: 定义宏 `_vel_vaddul_vvvl`，用于条件编译、简写或 API 生成。
- **L110 EN**: Defines macro `_vel_vaddul_vvvvl` for conditional compilation, shorthand, or API generation.
  **L110 CN**: 定义宏 `_vel_vaddul_vvvvl`，用于条件编译、简写或 API 生成。
- **L111 EN**: Defines macro `_vel_vaddul_vsvl` for conditional compilation, shorthand, or API generation.
  **L111 CN**: 定义宏 `_vel_vaddul_vsvl`，用于条件编译、简写或 API 生成。
- **L112 EN**: Defines macro `_vel_vaddul_vsvvl` for conditional compilation, shorthand, or API generation.
  **L112 CN**: 定义宏 `_vel_vaddul_vsvvl`，用于条件编译、简写或 API 生成。
- **L113 EN**: Defines macro `_vel_vaddul_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L113 CN**: 定义宏 `_vel_vaddul_vvvmvl`，用于条件编译、简写或 API 生成。
- **L114 EN**: Defines macro `_vel_vaddul_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L114 CN**: 定义宏 `_vel_vaddul_vsvmvl`，用于条件编译、简写或 API 生成。
- **L115 EN**: Defines macro `_vel_vadduw_vvvl` for conditional compilation, shorthand, or API generation.
  **L115 CN**: 定义宏 `_vel_vadduw_vvvl`，用于条件编译、简写或 API 生成。
- **L116 EN**: Defines macro `_vel_vadduw_vvvvl` for conditional compilation, shorthand, or API generation.
  **L116 CN**: 定义宏 `_vel_vadduw_vvvvl`，用于条件编译、简写或 API 生成。
- **L117 EN**: Defines macro `_vel_vadduw_vsvl` for conditional compilation, shorthand, or API generation.
  **L117 CN**: 定义宏 `_vel_vadduw_vsvl`，用于条件编译、简写或 API 生成。
- **L118 EN**: Defines macro `_vel_vadduw_vsvvl` for conditional compilation, shorthand, or API generation.
  **L118 CN**: 定义宏 `_vel_vadduw_vsvvl`，用于条件编译、简写或 API 生成。
- **L119 EN**: Defines macro `_vel_vadduw_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L119 CN**: 定义宏 `_vel_vadduw_vvvmvl`，用于条件编译、简写或 API 生成。
- **L120 EN**: Defines macro `_vel_vadduw_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L120 CN**: 定义宏 `_vel_vadduw_vsvmvl`，用于条件编译、简写或 API 生成。

### Lines 121-144

````c
#define _vel_pvaddu_vvvl __builtin_ve_vl_pvaddu_vvvl
#define _vel_pvaddu_vvvvl __builtin_ve_vl_pvaddu_vvvvl
#define _vel_pvaddu_vsvl __builtin_ve_vl_pvaddu_vsvl
#define _vel_pvaddu_vsvvl __builtin_ve_vl_pvaddu_vsvvl
#define _vel_pvaddu_vvvMvl __builtin_ve_vl_pvaddu_vvvMvl
#define _vel_pvaddu_vsvMvl __builtin_ve_vl_pvaddu_vsvMvl
#define _vel_vaddswsx_vvvl __builtin_ve_vl_vaddswsx_vvvl
#define _vel_vaddswsx_vvvvl __builtin_ve_vl_vaddswsx_vvvvl
#define _vel_vaddswsx_vsvl __builtin_ve_vl_vaddswsx_vsvl
#define _vel_vaddswsx_vsvvl __builtin_ve_vl_vaddswsx_vsvvl
#define _vel_vaddswsx_vvvmvl __builtin_ve_vl_vaddswsx_vvvmvl
#define _vel_vaddswsx_vsvmvl __builtin_ve_vl_vaddswsx_vsvmvl
#define _vel_vaddswzx_vvvl __builtin_ve_vl_vaddswzx_vvvl
#define _vel_vaddswzx_vvvvl __builtin_ve_vl_vaddswzx_vvvvl
#define _vel_vaddswzx_vsvl __builtin_ve_vl_vaddswzx_vsvl
#define _vel_vaddswzx_vsvvl __builtin_ve_vl_vaddswzx_vsvvl
#define _vel_vaddswzx_vvvmvl __builtin_ve_vl_vaddswzx_vvvmvl
#define _vel_vaddswzx_vsvmvl __builtin_ve_vl_vaddswzx_vsvmvl
#define _vel_pvadds_vvvl __builtin_ve_vl_pvadds_vvvl
#define _vel_pvadds_vvvvl __builtin_ve_vl_pvadds_vvvvl
#define _vel_pvadds_vsvl __builtin_ve_vl_pvadds_vsvl
#define _vel_pvadds_vsvvl __builtin_ve_vl_pvadds_vsvvl
#define _vel_pvadds_vvvMvl __builtin_ve_vl_pvadds_vvvMvl
#define _vel_pvadds_vsvMvl __builtin_ve_vl_pvadds_vsvMvl
````
- **L121 EN**: Defines macro `_vel_pvaddu_vvvl` for conditional compilation, shorthand, or API generation.
  **L121 CN**: 定义宏 `_vel_pvaddu_vvvl`，用于条件编译、简写或 API 生成。
- **L122 EN**: Defines macro `_vel_pvaddu_vvvvl` for conditional compilation, shorthand, or API generation.
  **L122 CN**: 定义宏 `_vel_pvaddu_vvvvl`，用于条件编译、简写或 API 生成。
- **L123 EN**: Defines macro `_vel_pvaddu_vsvl` for conditional compilation, shorthand, or API generation.
  **L123 CN**: 定义宏 `_vel_pvaddu_vsvl`，用于条件编译、简写或 API 生成。
- **L124 EN**: Defines macro `_vel_pvaddu_vsvvl` for conditional compilation, shorthand, or API generation.
  **L124 CN**: 定义宏 `_vel_pvaddu_vsvvl`，用于条件编译、简写或 API 生成。
- **L125 EN**: Defines macro `_vel_pvaddu_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L125 CN**: 定义宏 `_vel_pvaddu_vvvMvl`，用于条件编译、简写或 API 生成。
- **L126 EN**: Defines macro `_vel_pvaddu_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L126 CN**: 定义宏 `_vel_pvaddu_vsvMvl`，用于条件编译、简写或 API 生成。
- **L127 EN**: Defines macro `_vel_vaddswsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L127 CN**: 定义宏 `_vel_vaddswsx_vvvl`，用于条件编译、简写或 API 生成。
- **L128 EN**: Defines macro `_vel_vaddswsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L128 CN**: 定义宏 `_vel_vaddswsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L129 EN**: Defines macro `_vel_vaddswsx_vsvl` for conditional compilation, shorthand, or API generation.
  **L129 CN**: 定义宏 `_vel_vaddswsx_vsvl`，用于条件编译、简写或 API 生成。
- **L130 EN**: Defines macro `_vel_vaddswsx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L130 CN**: 定义宏 `_vel_vaddswsx_vsvvl`，用于条件编译、简写或 API 生成。
- **L131 EN**: Defines macro `_vel_vaddswsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L131 CN**: 定义宏 `_vel_vaddswsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L132 EN**: Defines macro `_vel_vaddswsx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L132 CN**: 定义宏 `_vel_vaddswsx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L133 EN**: Defines macro `_vel_vaddswzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L133 CN**: 定义宏 `_vel_vaddswzx_vvvl`，用于条件编译、简写或 API 生成。
- **L134 EN**: Defines macro `_vel_vaddswzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L134 CN**: 定义宏 `_vel_vaddswzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L135 EN**: Defines macro `_vel_vaddswzx_vsvl` for conditional compilation, shorthand, or API generation.
  **L135 CN**: 定义宏 `_vel_vaddswzx_vsvl`，用于条件编译、简写或 API 生成。
- **L136 EN**: Defines macro `_vel_vaddswzx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L136 CN**: 定义宏 `_vel_vaddswzx_vsvvl`，用于条件编译、简写或 API 生成。
- **L137 EN**: Defines macro `_vel_vaddswzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L137 CN**: 定义宏 `_vel_vaddswzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L138 EN**: Defines macro `_vel_vaddswzx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L138 CN**: 定义宏 `_vel_vaddswzx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L139 EN**: Defines macro `_vel_pvadds_vvvl` for conditional compilation, shorthand, or API generation.
  **L139 CN**: 定义宏 `_vel_pvadds_vvvl`，用于条件编译、简写或 API 生成。
- **L140 EN**: Defines macro `_vel_pvadds_vvvvl` for conditional compilation, shorthand, or API generation.
  **L140 CN**: 定义宏 `_vel_pvadds_vvvvl`，用于条件编译、简写或 API 生成。
- **L141 EN**: Defines macro `_vel_pvadds_vsvl` for conditional compilation, shorthand, or API generation.
  **L141 CN**: 定义宏 `_vel_pvadds_vsvl`，用于条件编译、简写或 API 生成。
- **L142 EN**: Defines macro `_vel_pvadds_vsvvl` for conditional compilation, shorthand, or API generation.
  **L142 CN**: 定义宏 `_vel_pvadds_vsvvl`，用于条件编译、简写或 API 生成。
- **L143 EN**: Defines macro `_vel_pvadds_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L143 CN**: 定义宏 `_vel_pvadds_vvvMvl`，用于条件编译、简写或 API 生成。
- **L144 EN**: Defines macro `_vel_pvadds_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L144 CN**: 定义宏 `_vel_pvadds_vsvMvl`，用于条件编译、简写或 API 生成。

### Lines 145-168

````c
#define _vel_vaddsl_vvvl __builtin_ve_vl_vaddsl_vvvl
#define _vel_vaddsl_vvvvl __builtin_ve_vl_vaddsl_vvvvl
#define _vel_vaddsl_vsvl __builtin_ve_vl_vaddsl_vsvl
#define _vel_vaddsl_vsvvl __builtin_ve_vl_vaddsl_vsvvl
#define _vel_vaddsl_vvvmvl __builtin_ve_vl_vaddsl_vvvmvl
#define _vel_vaddsl_vsvmvl __builtin_ve_vl_vaddsl_vsvmvl
#define _vel_vsubul_vvvl __builtin_ve_vl_vsubul_vvvl
#define _vel_vsubul_vvvvl __builtin_ve_vl_vsubul_vvvvl
#define _vel_vsubul_vsvl __builtin_ve_vl_vsubul_vsvl
#define _vel_vsubul_vsvvl __builtin_ve_vl_vsubul_vsvvl
#define _vel_vsubul_vvvmvl __builtin_ve_vl_vsubul_vvvmvl
#define _vel_vsubul_vsvmvl __builtin_ve_vl_vsubul_vsvmvl
#define _vel_vsubuw_vvvl __builtin_ve_vl_vsubuw_vvvl
#define _vel_vsubuw_vvvvl __builtin_ve_vl_vsubuw_vvvvl
#define _vel_vsubuw_vsvl __builtin_ve_vl_vsubuw_vsvl
#define _vel_vsubuw_vsvvl __builtin_ve_vl_vsubuw_vsvvl
#define _vel_vsubuw_vvvmvl __builtin_ve_vl_vsubuw_vvvmvl
#define _vel_vsubuw_vsvmvl __builtin_ve_vl_vsubuw_vsvmvl
#define _vel_pvsubu_vvvl __builtin_ve_vl_pvsubu_vvvl
#define _vel_pvsubu_vvvvl __builtin_ve_vl_pvsubu_vvvvl
#define _vel_pvsubu_vsvl __builtin_ve_vl_pvsubu_vsvl
#define _vel_pvsubu_vsvvl __builtin_ve_vl_pvsubu_vsvvl
#define _vel_pvsubu_vvvMvl __builtin_ve_vl_pvsubu_vvvMvl
#define _vel_pvsubu_vsvMvl __builtin_ve_vl_pvsubu_vsvMvl
````
- **L145 EN**: Defines macro `_vel_vaddsl_vvvl` for conditional compilation, shorthand, or API generation.
  **L145 CN**: 定义宏 `_vel_vaddsl_vvvl`，用于条件编译、简写或 API 生成。
- **L146 EN**: Defines macro `_vel_vaddsl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L146 CN**: 定义宏 `_vel_vaddsl_vvvvl`，用于条件编译、简写或 API 生成。
- **L147 EN**: Defines macro `_vel_vaddsl_vsvl` for conditional compilation, shorthand, or API generation.
  **L147 CN**: 定义宏 `_vel_vaddsl_vsvl`，用于条件编译、简写或 API 生成。
- **L148 EN**: Defines macro `_vel_vaddsl_vsvvl` for conditional compilation, shorthand, or API generation.
  **L148 CN**: 定义宏 `_vel_vaddsl_vsvvl`，用于条件编译、简写或 API 生成。
- **L149 EN**: Defines macro `_vel_vaddsl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L149 CN**: 定义宏 `_vel_vaddsl_vvvmvl`，用于条件编译、简写或 API 生成。
- **L150 EN**: Defines macro `_vel_vaddsl_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L150 CN**: 定义宏 `_vel_vaddsl_vsvmvl`，用于条件编译、简写或 API 生成。
- **L151 EN**: Defines macro `_vel_vsubul_vvvl` for conditional compilation, shorthand, or API generation.
  **L151 CN**: 定义宏 `_vel_vsubul_vvvl`，用于条件编译、简写或 API 生成。
- **L152 EN**: Defines macro `_vel_vsubul_vvvvl` for conditional compilation, shorthand, or API generation.
  **L152 CN**: 定义宏 `_vel_vsubul_vvvvl`，用于条件编译、简写或 API 生成。
- **L153 EN**: Defines macro `_vel_vsubul_vsvl` for conditional compilation, shorthand, or API generation.
  **L153 CN**: 定义宏 `_vel_vsubul_vsvl`，用于条件编译、简写或 API 生成。
- **L154 EN**: Defines macro `_vel_vsubul_vsvvl` for conditional compilation, shorthand, or API generation.
  **L154 CN**: 定义宏 `_vel_vsubul_vsvvl`，用于条件编译、简写或 API 生成。
- **L155 EN**: Defines macro `_vel_vsubul_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L155 CN**: 定义宏 `_vel_vsubul_vvvmvl`，用于条件编译、简写或 API 生成。
- **L156 EN**: Defines macro `_vel_vsubul_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L156 CN**: 定义宏 `_vel_vsubul_vsvmvl`，用于条件编译、简写或 API 生成。
- **L157 EN**: Defines macro `_vel_vsubuw_vvvl` for conditional compilation, shorthand, or API generation.
  **L157 CN**: 定义宏 `_vel_vsubuw_vvvl`，用于条件编译、简写或 API 生成。
- **L158 EN**: Defines macro `_vel_vsubuw_vvvvl` for conditional compilation, shorthand, or API generation.
  **L158 CN**: 定义宏 `_vel_vsubuw_vvvvl`，用于条件编译、简写或 API 生成。
- **L159 EN**: Defines macro `_vel_vsubuw_vsvl` for conditional compilation, shorthand, or API generation.
  **L159 CN**: 定义宏 `_vel_vsubuw_vsvl`，用于条件编译、简写或 API 生成。
- **L160 EN**: Defines macro `_vel_vsubuw_vsvvl` for conditional compilation, shorthand, or API generation.
  **L160 CN**: 定义宏 `_vel_vsubuw_vsvvl`，用于条件编译、简写或 API 生成。
- **L161 EN**: Defines macro `_vel_vsubuw_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L161 CN**: 定义宏 `_vel_vsubuw_vvvmvl`，用于条件编译、简写或 API 生成。
- **L162 EN**: Defines macro `_vel_vsubuw_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L162 CN**: 定义宏 `_vel_vsubuw_vsvmvl`，用于条件编译、简写或 API 生成。
- **L163 EN**: Defines macro `_vel_pvsubu_vvvl` for conditional compilation, shorthand, or API generation.
  **L163 CN**: 定义宏 `_vel_pvsubu_vvvl`，用于条件编译、简写或 API 生成。
- **L164 EN**: Defines macro `_vel_pvsubu_vvvvl` for conditional compilation, shorthand, or API generation.
  **L164 CN**: 定义宏 `_vel_pvsubu_vvvvl`，用于条件编译、简写或 API 生成。
- **L165 EN**: Defines macro `_vel_pvsubu_vsvl` for conditional compilation, shorthand, or API generation.
  **L165 CN**: 定义宏 `_vel_pvsubu_vsvl`，用于条件编译、简写或 API 生成。
- **L166 EN**: Defines macro `_vel_pvsubu_vsvvl` for conditional compilation, shorthand, or API generation.
  **L166 CN**: 定义宏 `_vel_pvsubu_vsvvl`，用于条件编译、简写或 API 生成。
- **L167 EN**: Defines macro `_vel_pvsubu_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L167 CN**: 定义宏 `_vel_pvsubu_vvvMvl`，用于条件编译、简写或 API 生成。
- **L168 EN**: Defines macro `_vel_pvsubu_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L168 CN**: 定义宏 `_vel_pvsubu_vsvMvl`，用于条件编译、简写或 API 生成。

### Lines 169-192

````c
#define _vel_vsubswsx_vvvl __builtin_ve_vl_vsubswsx_vvvl
#define _vel_vsubswsx_vvvvl __builtin_ve_vl_vsubswsx_vvvvl
#define _vel_vsubswsx_vsvl __builtin_ve_vl_vsubswsx_vsvl
#define _vel_vsubswsx_vsvvl __builtin_ve_vl_vsubswsx_vsvvl
#define _vel_vsubswsx_vvvmvl __builtin_ve_vl_vsubswsx_vvvmvl
#define _vel_vsubswsx_vsvmvl __builtin_ve_vl_vsubswsx_vsvmvl
#define _vel_vsubswzx_vvvl __builtin_ve_vl_vsubswzx_vvvl
#define _vel_vsubswzx_vvvvl __builtin_ve_vl_vsubswzx_vvvvl
#define _vel_vsubswzx_vsvl __builtin_ve_vl_vsubswzx_vsvl
#define _vel_vsubswzx_vsvvl __builtin_ve_vl_vsubswzx_vsvvl
#define _vel_vsubswzx_vvvmvl __builtin_ve_vl_vsubswzx_vvvmvl
#define _vel_vsubswzx_vsvmvl __builtin_ve_vl_vsubswzx_vsvmvl
#define _vel_pvsubs_vvvl __builtin_ve_vl_pvsubs_vvvl
#define _vel_pvsubs_vvvvl __builtin_ve_vl_pvsubs_vvvvl
#define _vel_pvsubs_vsvl __builtin_ve_vl_pvsubs_vsvl
#define _vel_pvsubs_vsvvl __builtin_ve_vl_pvsubs_vsvvl
#define _vel_pvsubs_vvvMvl __builtin_ve_vl_pvsubs_vvvMvl
#define _vel_pvsubs_vsvMvl __builtin_ve_vl_pvsubs_vsvMvl
#define _vel_vsubsl_vvvl __builtin_ve_vl_vsubsl_vvvl
#define _vel_vsubsl_vvvvl __builtin_ve_vl_vsubsl_vvvvl
#define _vel_vsubsl_vsvl __builtin_ve_vl_vsubsl_vsvl
#define _vel_vsubsl_vsvvl __builtin_ve_vl_vsubsl_vsvvl
#define _vel_vsubsl_vvvmvl __builtin_ve_vl_vsubsl_vvvmvl
#define _vel_vsubsl_vsvmvl __builtin_ve_vl_vsubsl_vsvmvl
````
- **L169 EN**: Defines macro `_vel_vsubswsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L169 CN**: 定义宏 `_vel_vsubswsx_vvvl`，用于条件编译、简写或 API 生成。
- **L170 EN**: Defines macro `_vel_vsubswsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L170 CN**: 定义宏 `_vel_vsubswsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L171 EN**: Defines macro `_vel_vsubswsx_vsvl` for conditional compilation, shorthand, or API generation.
  **L171 CN**: 定义宏 `_vel_vsubswsx_vsvl`，用于条件编译、简写或 API 生成。
- **L172 EN**: Defines macro `_vel_vsubswsx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L172 CN**: 定义宏 `_vel_vsubswsx_vsvvl`，用于条件编译、简写或 API 生成。
- **L173 EN**: Defines macro `_vel_vsubswsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L173 CN**: 定义宏 `_vel_vsubswsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L174 EN**: Defines macro `_vel_vsubswsx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L174 CN**: 定义宏 `_vel_vsubswsx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L175 EN**: Defines macro `_vel_vsubswzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L175 CN**: 定义宏 `_vel_vsubswzx_vvvl`，用于条件编译、简写或 API 生成。
- **L176 EN**: Defines macro `_vel_vsubswzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L176 CN**: 定义宏 `_vel_vsubswzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L177 EN**: Defines macro `_vel_vsubswzx_vsvl` for conditional compilation, shorthand, or API generation.
  **L177 CN**: 定义宏 `_vel_vsubswzx_vsvl`，用于条件编译、简写或 API 生成。
- **L178 EN**: Defines macro `_vel_vsubswzx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L178 CN**: 定义宏 `_vel_vsubswzx_vsvvl`，用于条件编译、简写或 API 生成。
- **L179 EN**: Defines macro `_vel_vsubswzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L179 CN**: 定义宏 `_vel_vsubswzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L180 EN**: Defines macro `_vel_vsubswzx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L180 CN**: 定义宏 `_vel_vsubswzx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L181 EN**: Defines macro `_vel_pvsubs_vvvl` for conditional compilation, shorthand, or API generation.
  **L181 CN**: 定义宏 `_vel_pvsubs_vvvl`，用于条件编译、简写或 API 生成。
- **L182 EN**: Defines macro `_vel_pvsubs_vvvvl` for conditional compilation, shorthand, or API generation.
  **L182 CN**: 定义宏 `_vel_pvsubs_vvvvl`，用于条件编译、简写或 API 生成。
- **L183 EN**: Defines macro `_vel_pvsubs_vsvl` for conditional compilation, shorthand, or API generation.
  **L183 CN**: 定义宏 `_vel_pvsubs_vsvl`，用于条件编译、简写或 API 生成。
- **L184 EN**: Defines macro `_vel_pvsubs_vsvvl` for conditional compilation, shorthand, or API generation.
  **L184 CN**: 定义宏 `_vel_pvsubs_vsvvl`，用于条件编译、简写或 API 生成。
- **L185 EN**: Defines macro `_vel_pvsubs_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L185 CN**: 定义宏 `_vel_pvsubs_vvvMvl`，用于条件编译、简写或 API 生成。
- **L186 EN**: Defines macro `_vel_pvsubs_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L186 CN**: 定义宏 `_vel_pvsubs_vsvMvl`，用于条件编译、简写或 API 生成。
- **L187 EN**: Defines macro `_vel_vsubsl_vvvl` for conditional compilation, shorthand, or API generation.
  **L187 CN**: 定义宏 `_vel_vsubsl_vvvl`，用于条件编译、简写或 API 生成。
- **L188 EN**: Defines macro `_vel_vsubsl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L188 CN**: 定义宏 `_vel_vsubsl_vvvvl`，用于条件编译、简写或 API 生成。
- **L189 EN**: Defines macro `_vel_vsubsl_vsvl` for conditional compilation, shorthand, or API generation.
  **L189 CN**: 定义宏 `_vel_vsubsl_vsvl`，用于条件编译、简写或 API 生成。
- **L190 EN**: Defines macro `_vel_vsubsl_vsvvl` for conditional compilation, shorthand, or API generation.
  **L190 CN**: 定义宏 `_vel_vsubsl_vsvvl`，用于条件编译、简写或 API 生成。
- **L191 EN**: Defines macro `_vel_vsubsl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L191 CN**: 定义宏 `_vel_vsubsl_vvvmvl`，用于条件编译、简写或 API 生成。
- **L192 EN**: Defines macro `_vel_vsubsl_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L192 CN**: 定义宏 `_vel_vsubsl_vsvmvl`，用于条件编译、简写或 API 生成。

### Lines 193-216

````c
#define _vel_vmulul_vvvl __builtin_ve_vl_vmulul_vvvl
#define _vel_vmulul_vvvvl __builtin_ve_vl_vmulul_vvvvl
#define _vel_vmulul_vsvl __builtin_ve_vl_vmulul_vsvl
#define _vel_vmulul_vsvvl __builtin_ve_vl_vmulul_vsvvl
#define _vel_vmulul_vvvmvl __builtin_ve_vl_vmulul_vvvmvl
#define _vel_vmulul_vsvmvl __builtin_ve_vl_vmulul_vsvmvl
#define _vel_vmuluw_vvvl __builtin_ve_vl_vmuluw_vvvl
#define _vel_vmuluw_vvvvl __builtin_ve_vl_vmuluw_vvvvl
#define _vel_vmuluw_vsvl __builtin_ve_vl_vmuluw_vsvl
#define _vel_vmuluw_vsvvl __builtin_ve_vl_vmuluw_vsvvl
#define _vel_vmuluw_vvvmvl __builtin_ve_vl_vmuluw_vvvmvl
#define _vel_vmuluw_vsvmvl __builtin_ve_vl_vmuluw_vsvmvl
#define _vel_vmulswsx_vvvl __builtin_ve_vl_vmulswsx_vvvl
#define _vel_vmulswsx_vvvvl __builtin_ve_vl_vmulswsx_vvvvl
#define _vel_vmulswsx_vsvl __builtin_ve_vl_vmulswsx_vsvl
#define _vel_vmulswsx_vsvvl __builtin_ve_vl_vmulswsx_vsvvl
#define _vel_vmulswsx_vvvmvl __builtin_ve_vl_vmulswsx_vvvmvl
#define _vel_vmulswsx_vsvmvl __builtin_ve_vl_vmulswsx_vsvmvl
#define _vel_vmulswzx_vvvl __builtin_ve_vl_vmulswzx_vvvl
#define _vel_vmulswzx_vvvvl __builtin_ve_vl_vmulswzx_vvvvl
#define _vel_vmulswzx_vsvl __builtin_ve_vl_vmulswzx_vsvl
#define _vel_vmulswzx_vsvvl __builtin_ve_vl_vmulswzx_vsvvl
#define _vel_vmulswzx_vvvmvl __builtin_ve_vl_vmulswzx_vvvmvl
#define _vel_vmulswzx_vsvmvl __builtin_ve_vl_vmulswzx_vsvmvl
````
- **L193 EN**: Defines macro `_vel_vmulul_vvvl` for conditional compilation, shorthand, or API generation.
  **L193 CN**: 定义宏 `_vel_vmulul_vvvl`，用于条件编译、简写或 API 生成。
- **L194 EN**: Defines macro `_vel_vmulul_vvvvl` for conditional compilation, shorthand, or API generation.
  **L194 CN**: 定义宏 `_vel_vmulul_vvvvl`，用于条件编译、简写或 API 生成。
- **L195 EN**: Defines macro `_vel_vmulul_vsvl` for conditional compilation, shorthand, or API generation.
  **L195 CN**: 定义宏 `_vel_vmulul_vsvl`，用于条件编译、简写或 API 生成。
- **L196 EN**: Defines macro `_vel_vmulul_vsvvl` for conditional compilation, shorthand, or API generation.
  **L196 CN**: 定义宏 `_vel_vmulul_vsvvl`，用于条件编译、简写或 API 生成。
- **L197 EN**: Defines macro `_vel_vmulul_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L197 CN**: 定义宏 `_vel_vmulul_vvvmvl`，用于条件编译、简写或 API 生成。
- **L198 EN**: Defines macro `_vel_vmulul_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L198 CN**: 定义宏 `_vel_vmulul_vsvmvl`，用于条件编译、简写或 API 生成。
- **L199 EN**: Defines macro `_vel_vmuluw_vvvl` for conditional compilation, shorthand, or API generation.
  **L199 CN**: 定义宏 `_vel_vmuluw_vvvl`，用于条件编译、简写或 API 生成。
- **L200 EN**: Defines macro `_vel_vmuluw_vvvvl` for conditional compilation, shorthand, or API generation.
  **L200 CN**: 定义宏 `_vel_vmuluw_vvvvl`，用于条件编译、简写或 API 生成。
- **L201 EN**: Defines macro `_vel_vmuluw_vsvl` for conditional compilation, shorthand, or API generation.
  **L201 CN**: 定义宏 `_vel_vmuluw_vsvl`，用于条件编译、简写或 API 生成。
- **L202 EN**: Defines macro `_vel_vmuluw_vsvvl` for conditional compilation, shorthand, or API generation.
  **L202 CN**: 定义宏 `_vel_vmuluw_vsvvl`，用于条件编译、简写或 API 生成。
- **L203 EN**: Defines macro `_vel_vmuluw_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L203 CN**: 定义宏 `_vel_vmuluw_vvvmvl`，用于条件编译、简写或 API 生成。
- **L204 EN**: Defines macro `_vel_vmuluw_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L204 CN**: 定义宏 `_vel_vmuluw_vsvmvl`，用于条件编译、简写或 API 生成。
- **L205 EN**: Defines macro `_vel_vmulswsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L205 CN**: 定义宏 `_vel_vmulswsx_vvvl`，用于条件编译、简写或 API 生成。
- **L206 EN**: Defines macro `_vel_vmulswsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L206 CN**: 定义宏 `_vel_vmulswsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L207 EN**: Defines macro `_vel_vmulswsx_vsvl` for conditional compilation, shorthand, or API generation.
  **L207 CN**: 定义宏 `_vel_vmulswsx_vsvl`，用于条件编译、简写或 API 生成。
- **L208 EN**: Defines macro `_vel_vmulswsx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L208 CN**: 定义宏 `_vel_vmulswsx_vsvvl`，用于条件编译、简写或 API 生成。
- **L209 EN**: Defines macro `_vel_vmulswsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L209 CN**: 定义宏 `_vel_vmulswsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L210 EN**: Defines macro `_vel_vmulswsx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L210 CN**: 定义宏 `_vel_vmulswsx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L211 EN**: Defines macro `_vel_vmulswzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L211 CN**: 定义宏 `_vel_vmulswzx_vvvl`，用于条件编译、简写或 API 生成。
- **L212 EN**: Defines macro `_vel_vmulswzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L212 CN**: 定义宏 `_vel_vmulswzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L213 EN**: Defines macro `_vel_vmulswzx_vsvl` for conditional compilation, shorthand, or API generation.
  **L213 CN**: 定义宏 `_vel_vmulswzx_vsvl`，用于条件编译、简写或 API 生成。
- **L214 EN**: Defines macro `_vel_vmulswzx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L214 CN**: 定义宏 `_vel_vmulswzx_vsvvl`，用于条件编译、简写或 API 生成。
- **L215 EN**: Defines macro `_vel_vmulswzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L215 CN**: 定义宏 `_vel_vmulswzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L216 EN**: Defines macro `_vel_vmulswzx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L216 CN**: 定义宏 `_vel_vmulswzx_vsvmvl`，用于条件编译、简写或 API 生成。

### Lines 217-240

````c
#define _vel_vmulsl_vvvl __builtin_ve_vl_vmulsl_vvvl
#define _vel_vmulsl_vvvvl __builtin_ve_vl_vmulsl_vvvvl
#define _vel_vmulsl_vsvl __builtin_ve_vl_vmulsl_vsvl
#define _vel_vmulsl_vsvvl __builtin_ve_vl_vmulsl_vsvvl
#define _vel_vmulsl_vvvmvl __builtin_ve_vl_vmulsl_vvvmvl
#define _vel_vmulsl_vsvmvl __builtin_ve_vl_vmulsl_vsvmvl
#define _vel_vmulslw_vvvl __builtin_ve_vl_vmulslw_vvvl
#define _vel_vmulslw_vvvvl __builtin_ve_vl_vmulslw_vvvvl
#define _vel_vmulslw_vsvl __builtin_ve_vl_vmulslw_vsvl
#define _vel_vmulslw_vsvvl __builtin_ve_vl_vmulslw_vsvvl
#define _vel_vdivul_vvvl __builtin_ve_vl_vdivul_vvvl
#define _vel_vdivul_vvvvl __builtin_ve_vl_vdivul_vvvvl
#define _vel_vdivul_vsvl __builtin_ve_vl_vdivul_vsvl
#define _vel_vdivul_vsvvl __builtin_ve_vl_vdivul_vsvvl
#define _vel_vdivul_vvvmvl __builtin_ve_vl_vdivul_vvvmvl
#define _vel_vdivul_vsvmvl __builtin_ve_vl_vdivul_vsvmvl
#define _vel_vdivuw_vvvl __builtin_ve_vl_vdivuw_vvvl
#define _vel_vdivuw_vvvvl __builtin_ve_vl_vdivuw_vvvvl
#define _vel_vdivuw_vsvl __builtin_ve_vl_vdivuw_vsvl
#define _vel_vdivuw_vsvvl __builtin_ve_vl_vdivuw_vsvvl
#define _vel_vdivuw_vvvmvl __builtin_ve_vl_vdivuw_vvvmvl
#define _vel_vdivuw_vsvmvl __builtin_ve_vl_vdivuw_vsvmvl
#define _vel_vdivul_vvsl __builtin_ve_vl_vdivul_vvsl
#define _vel_vdivul_vvsvl __builtin_ve_vl_vdivul_vvsvl
````
- **L217 EN**: Defines macro `_vel_vmulsl_vvvl` for conditional compilation, shorthand, or API generation.
  **L217 CN**: 定义宏 `_vel_vmulsl_vvvl`，用于条件编译、简写或 API 生成。
- **L218 EN**: Defines macro `_vel_vmulsl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L218 CN**: 定义宏 `_vel_vmulsl_vvvvl`，用于条件编译、简写或 API 生成。
- **L219 EN**: Defines macro `_vel_vmulsl_vsvl` for conditional compilation, shorthand, or API generation.
  **L219 CN**: 定义宏 `_vel_vmulsl_vsvl`，用于条件编译、简写或 API 生成。
- **L220 EN**: Defines macro `_vel_vmulsl_vsvvl` for conditional compilation, shorthand, or API generation.
  **L220 CN**: 定义宏 `_vel_vmulsl_vsvvl`，用于条件编译、简写或 API 生成。
- **L221 EN**: Defines macro `_vel_vmulsl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L221 CN**: 定义宏 `_vel_vmulsl_vvvmvl`，用于条件编译、简写或 API 生成。
- **L222 EN**: Defines macro `_vel_vmulsl_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L222 CN**: 定义宏 `_vel_vmulsl_vsvmvl`，用于条件编译、简写或 API 生成。
- **L223 EN**: Defines macro `_vel_vmulslw_vvvl` for conditional compilation, shorthand, or API generation.
  **L223 CN**: 定义宏 `_vel_vmulslw_vvvl`，用于条件编译、简写或 API 生成。
- **L224 EN**: Defines macro `_vel_vmulslw_vvvvl` for conditional compilation, shorthand, or API generation.
  **L224 CN**: 定义宏 `_vel_vmulslw_vvvvl`，用于条件编译、简写或 API 生成。
- **L225 EN**: Defines macro `_vel_vmulslw_vsvl` for conditional compilation, shorthand, or API generation.
  **L225 CN**: 定义宏 `_vel_vmulslw_vsvl`，用于条件编译、简写或 API 生成。
- **L226 EN**: Defines macro `_vel_vmulslw_vsvvl` for conditional compilation, shorthand, or API generation.
  **L226 CN**: 定义宏 `_vel_vmulslw_vsvvl`，用于条件编译、简写或 API 生成。
- **L227 EN**: Defines macro `_vel_vdivul_vvvl` for conditional compilation, shorthand, or API generation.
  **L227 CN**: 定义宏 `_vel_vdivul_vvvl`，用于条件编译、简写或 API 生成。
- **L228 EN**: Defines macro `_vel_vdivul_vvvvl` for conditional compilation, shorthand, or API generation.
  **L228 CN**: 定义宏 `_vel_vdivul_vvvvl`，用于条件编译、简写或 API 生成。
- **L229 EN**: Defines macro `_vel_vdivul_vsvl` for conditional compilation, shorthand, or API generation.
  **L229 CN**: 定义宏 `_vel_vdivul_vsvl`，用于条件编译、简写或 API 生成。
- **L230 EN**: Defines macro `_vel_vdivul_vsvvl` for conditional compilation, shorthand, or API generation.
  **L230 CN**: 定义宏 `_vel_vdivul_vsvvl`，用于条件编译、简写或 API 生成。
- **L231 EN**: Defines macro `_vel_vdivul_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L231 CN**: 定义宏 `_vel_vdivul_vvvmvl`，用于条件编译、简写或 API 生成。
- **L232 EN**: Defines macro `_vel_vdivul_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L232 CN**: 定义宏 `_vel_vdivul_vsvmvl`，用于条件编译、简写或 API 生成。
- **L233 EN**: Defines macro `_vel_vdivuw_vvvl` for conditional compilation, shorthand, or API generation.
  **L233 CN**: 定义宏 `_vel_vdivuw_vvvl`，用于条件编译、简写或 API 生成。
- **L234 EN**: Defines macro `_vel_vdivuw_vvvvl` for conditional compilation, shorthand, or API generation.
  **L234 CN**: 定义宏 `_vel_vdivuw_vvvvl`，用于条件编译、简写或 API 生成。
- **L235 EN**: Defines macro `_vel_vdivuw_vsvl` for conditional compilation, shorthand, or API generation.
  **L235 CN**: 定义宏 `_vel_vdivuw_vsvl`，用于条件编译、简写或 API 生成。
- **L236 EN**: Defines macro `_vel_vdivuw_vsvvl` for conditional compilation, shorthand, or API generation.
  **L236 CN**: 定义宏 `_vel_vdivuw_vsvvl`，用于条件编译、简写或 API 生成。
- **L237 EN**: Defines macro `_vel_vdivuw_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L237 CN**: 定义宏 `_vel_vdivuw_vvvmvl`，用于条件编译、简写或 API 生成。
- **L238 EN**: Defines macro `_vel_vdivuw_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L238 CN**: 定义宏 `_vel_vdivuw_vsvmvl`，用于条件编译、简写或 API 生成。
- **L239 EN**: Defines macro `_vel_vdivul_vvsl` for conditional compilation, shorthand, or API generation.
  **L239 CN**: 定义宏 `_vel_vdivul_vvsl`，用于条件编译、简写或 API 生成。
- **L240 EN**: Defines macro `_vel_vdivul_vvsvl` for conditional compilation, shorthand, or API generation.
  **L240 CN**: 定义宏 `_vel_vdivul_vvsvl`，用于条件编译、简写或 API 生成。

### Lines 241-264

````c
#define _vel_vdivul_vvsmvl __builtin_ve_vl_vdivul_vvsmvl
#define _vel_vdivuw_vvsl __builtin_ve_vl_vdivuw_vvsl
#define _vel_vdivuw_vvsvl __builtin_ve_vl_vdivuw_vvsvl
#define _vel_vdivuw_vvsmvl __builtin_ve_vl_vdivuw_vvsmvl
#define _vel_vdivswsx_vvvl __builtin_ve_vl_vdivswsx_vvvl
#define _vel_vdivswsx_vvvvl __builtin_ve_vl_vdivswsx_vvvvl
#define _vel_vdivswsx_vsvl __builtin_ve_vl_vdivswsx_vsvl
#define _vel_vdivswsx_vsvvl __builtin_ve_vl_vdivswsx_vsvvl
#define _vel_vdivswsx_vvvmvl __builtin_ve_vl_vdivswsx_vvvmvl
#define _vel_vdivswsx_vsvmvl __builtin_ve_vl_vdivswsx_vsvmvl
#define _vel_vdivswzx_vvvl __builtin_ve_vl_vdivswzx_vvvl
#define _vel_vdivswzx_vvvvl __builtin_ve_vl_vdivswzx_vvvvl
#define _vel_vdivswzx_vsvl __builtin_ve_vl_vdivswzx_vsvl
#define _vel_vdivswzx_vsvvl __builtin_ve_vl_vdivswzx_vsvvl
#define _vel_vdivswzx_vvvmvl __builtin_ve_vl_vdivswzx_vvvmvl
#define _vel_vdivswzx_vsvmvl __builtin_ve_vl_vdivswzx_vsvmvl
#define _vel_vdivswsx_vvsl __builtin_ve_vl_vdivswsx_vvsl
#define _vel_vdivswsx_vvsvl __builtin_ve_vl_vdivswsx_vvsvl
#define _vel_vdivswsx_vvsmvl __builtin_ve_vl_vdivswsx_vvsmvl
#define _vel_vdivswzx_vvsl __builtin_ve_vl_vdivswzx_vvsl
#define _vel_vdivswzx_vvsvl __builtin_ve_vl_vdivswzx_vvsvl
#define _vel_vdivswzx_vvsmvl __builtin_ve_vl_vdivswzx_vvsmvl
#define _vel_vdivsl_vvvl __builtin_ve_vl_vdivsl_vvvl
#define _vel_vdivsl_vvvvl __builtin_ve_vl_vdivsl_vvvvl
````
- **L241 EN**: Defines macro `_vel_vdivul_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L241 CN**: 定义宏 `_vel_vdivul_vvsmvl`，用于条件编译、简写或 API 生成。
- **L242 EN**: Defines macro `_vel_vdivuw_vvsl` for conditional compilation, shorthand, or API generation.
  **L242 CN**: 定义宏 `_vel_vdivuw_vvsl`，用于条件编译、简写或 API 生成。
- **L243 EN**: Defines macro `_vel_vdivuw_vvsvl` for conditional compilation, shorthand, or API generation.
  **L243 CN**: 定义宏 `_vel_vdivuw_vvsvl`，用于条件编译、简写或 API 生成。
- **L244 EN**: Defines macro `_vel_vdivuw_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L244 CN**: 定义宏 `_vel_vdivuw_vvsmvl`，用于条件编译、简写或 API 生成。
- **L245 EN**: Defines macro `_vel_vdivswsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L245 CN**: 定义宏 `_vel_vdivswsx_vvvl`，用于条件编译、简写或 API 生成。
- **L246 EN**: Defines macro `_vel_vdivswsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L246 CN**: 定义宏 `_vel_vdivswsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L247 EN**: Defines macro `_vel_vdivswsx_vsvl` for conditional compilation, shorthand, or API generation.
  **L247 CN**: 定义宏 `_vel_vdivswsx_vsvl`，用于条件编译、简写或 API 生成。
- **L248 EN**: Defines macro `_vel_vdivswsx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L248 CN**: 定义宏 `_vel_vdivswsx_vsvvl`，用于条件编译、简写或 API 生成。
- **L249 EN**: Defines macro `_vel_vdivswsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L249 CN**: 定义宏 `_vel_vdivswsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L250 EN**: Defines macro `_vel_vdivswsx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L250 CN**: 定义宏 `_vel_vdivswsx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L251 EN**: Defines macro `_vel_vdivswzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L251 CN**: 定义宏 `_vel_vdivswzx_vvvl`，用于条件编译、简写或 API 生成。
- **L252 EN**: Defines macro `_vel_vdivswzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L252 CN**: 定义宏 `_vel_vdivswzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L253 EN**: Defines macro `_vel_vdivswzx_vsvl` for conditional compilation, shorthand, or API generation.
  **L253 CN**: 定义宏 `_vel_vdivswzx_vsvl`，用于条件编译、简写或 API 生成。
- **L254 EN**: Defines macro `_vel_vdivswzx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L254 CN**: 定义宏 `_vel_vdivswzx_vsvvl`，用于条件编译、简写或 API 生成。
- **L255 EN**: Defines macro `_vel_vdivswzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L255 CN**: 定义宏 `_vel_vdivswzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L256 EN**: Defines macro `_vel_vdivswzx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L256 CN**: 定义宏 `_vel_vdivswzx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L257 EN**: Defines macro `_vel_vdivswsx_vvsl` for conditional compilation, shorthand, or API generation.
  **L257 CN**: 定义宏 `_vel_vdivswsx_vvsl`，用于条件编译、简写或 API 生成。
- **L258 EN**: Defines macro `_vel_vdivswsx_vvsvl` for conditional compilation, shorthand, or API generation.
  **L258 CN**: 定义宏 `_vel_vdivswsx_vvsvl`，用于条件编译、简写或 API 生成。
- **L259 EN**: Defines macro `_vel_vdivswsx_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L259 CN**: 定义宏 `_vel_vdivswsx_vvsmvl`，用于条件编译、简写或 API 生成。
- **L260 EN**: Defines macro `_vel_vdivswzx_vvsl` for conditional compilation, shorthand, or API generation.
  **L260 CN**: 定义宏 `_vel_vdivswzx_vvsl`，用于条件编译、简写或 API 生成。
- **L261 EN**: Defines macro `_vel_vdivswzx_vvsvl` for conditional compilation, shorthand, or API generation.
  **L261 CN**: 定义宏 `_vel_vdivswzx_vvsvl`，用于条件编译、简写或 API 生成。
- **L262 EN**: Defines macro `_vel_vdivswzx_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L262 CN**: 定义宏 `_vel_vdivswzx_vvsmvl`，用于条件编译、简写或 API 生成。
- **L263 EN**: Defines macro `_vel_vdivsl_vvvl` for conditional compilation, shorthand, or API generation.
  **L263 CN**: 定义宏 `_vel_vdivsl_vvvl`，用于条件编译、简写或 API 生成。
- **L264 EN**: Defines macro `_vel_vdivsl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L264 CN**: 定义宏 `_vel_vdivsl_vvvvl`，用于条件编译、简写或 API 生成。

### Lines 265-288

````c
#define _vel_vdivsl_vsvl __builtin_ve_vl_vdivsl_vsvl
#define _vel_vdivsl_vsvvl __builtin_ve_vl_vdivsl_vsvvl
#define _vel_vdivsl_vvvmvl __builtin_ve_vl_vdivsl_vvvmvl
#define _vel_vdivsl_vsvmvl __builtin_ve_vl_vdivsl_vsvmvl
#define _vel_vdivsl_vvsl __builtin_ve_vl_vdivsl_vvsl
#define _vel_vdivsl_vvsvl __builtin_ve_vl_vdivsl_vvsvl
#define _vel_vdivsl_vvsmvl __builtin_ve_vl_vdivsl_vvsmvl
#define _vel_vcmpul_vvvl __builtin_ve_vl_vcmpul_vvvl
#define _vel_vcmpul_vvvvl __builtin_ve_vl_vcmpul_vvvvl
#define _vel_vcmpul_vsvl __builtin_ve_vl_vcmpul_vsvl
#define _vel_vcmpul_vsvvl __builtin_ve_vl_vcmpul_vsvvl
#define _vel_vcmpul_vvvmvl __builtin_ve_vl_vcmpul_vvvmvl
#define _vel_vcmpul_vsvmvl __builtin_ve_vl_vcmpul_vsvmvl
#define _vel_vcmpuw_vvvl __builtin_ve_vl_vcmpuw_vvvl
#define _vel_vcmpuw_vvvvl __builtin_ve_vl_vcmpuw_vvvvl
#define _vel_vcmpuw_vsvl __builtin_ve_vl_vcmpuw_vsvl
#define _vel_vcmpuw_vsvvl __builtin_ve_vl_vcmpuw_vsvvl
#define _vel_vcmpuw_vvvmvl __builtin_ve_vl_vcmpuw_vvvmvl
#define _vel_vcmpuw_vsvmvl __builtin_ve_vl_vcmpuw_vsvmvl
#define _vel_pvcmpu_vvvl __builtin_ve_vl_pvcmpu_vvvl
#define _vel_pvcmpu_vvvvl __builtin_ve_vl_pvcmpu_vvvvl
#define _vel_pvcmpu_vsvl __builtin_ve_vl_pvcmpu_vsvl
#define _vel_pvcmpu_vsvvl __builtin_ve_vl_pvcmpu_vsvvl
#define _vel_pvcmpu_vvvMvl __builtin_ve_vl_pvcmpu_vvvMvl
````
- **L265 EN**: Defines macro `_vel_vdivsl_vsvl` for conditional compilation, shorthand, or API generation.
  **L265 CN**: 定义宏 `_vel_vdivsl_vsvl`，用于条件编译、简写或 API 生成。
- **L266 EN**: Defines macro `_vel_vdivsl_vsvvl` for conditional compilation, shorthand, or API generation.
  **L266 CN**: 定义宏 `_vel_vdivsl_vsvvl`，用于条件编译、简写或 API 生成。
- **L267 EN**: Defines macro `_vel_vdivsl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L267 CN**: 定义宏 `_vel_vdivsl_vvvmvl`，用于条件编译、简写或 API 生成。
- **L268 EN**: Defines macro `_vel_vdivsl_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L268 CN**: 定义宏 `_vel_vdivsl_vsvmvl`，用于条件编译、简写或 API 生成。
- **L269 EN**: Defines macro `_vel_vdivsl_vvsl` for conditional compilation, shorthand, or API generation.
  **L269 CN**: 定义宏 `_vel_vdivsl_vvsl`，用于条件编译、简写或 API 生成。
- **L270 EN**: Defines macro `_vel_vdivsl_vvsvl` for conditional compilation, shorthand, or API generation.
  **L270 CN**: 定义宏 `_vel_vdivsl_vvsvl`，用于条件编译、简写或 API 生成。
- **L271 EN**: Defines macro `_vel_vdivsl_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L271 CN**: 定义宏 `_vel_vdivsl_vvsmvl`，用于条件编译、简写或 API 生成。
- **L272 EN**: Defines macro `_vel_vcmpul_vvvl` for conditional compilation, shorthand, or API generation.
  **L272 CN**: 定义宏 `_vel_vcmpul_vvvl`，用于条件编译、简写或 API 生成。
- **L273 EN**: Defines macro `_vel_vcmpul_vvvvl` for conditional compilation, shorthand, or API generation.
  **L273 CN**: 定义宏 `_vel_vcmpul_vvvvl`，用于条件编译、简写或 API 生成。
- **L274 EN**: Defines macro `_vel_vcmpul_vsvl` for conditional compilation, shorthand, or API generation.
  **L274 CN**: 定义宏 `_vel_vcmpul_vsvl`，用于条件编译、简写或 API 生成。
- **L275 EN**: Defines macro `_vel_vcmpul_vsvvl` for conditional compilation, shorthand, or API generation.
  **L275 CN**: 定义宏 `_vel_vcmpul_vsvvl`，用于条件编译、简写或 API 生成。
- **L276 EN**: Defines macro `_vel_vcmpul_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L276 CN**: 定义宏 `_vel_vcmpul_vvvmvl`，用于条件编译、简写或 API 生成。
- **L277 EN**: Defines macro `_vel_vcmpul_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L277 CN**: 定义宏 `_vel_vcmpul_vsvmvl`，用于条件编译、简写或 API 生成。
- **L278 EN**: Defines macro `_vel_vcmpuw_vvvl` for conditional compilation, shorthand, or API generation.
  **L278 CN**: 定义宏 `_vel_vcmpuw_vvvl`，用于条件编译、简写或 API 生成。
- **L279 EN**: Defines macro `_vel_vcmpuw_vvvvl` for conditional compilation, shorthand, or API generation.
  **L279 CN**: 定义宏 `_vel_vcmpuw_vvvvl`，用于条件编译、简写或 API 生成。
- **L280 EN**: Defines macro `_vel_vcmpuw_vsvl` for conditional compilation, shorthand, or API generation.
  **L280 CN**: 定义宏 `_vel_vcmpuw_vsvl`，用于条件编译、简写或 API 生成。
- **L281 EN**: Defines macro `_vel_vcmpuw_vsvvl` for conditional compilation, shorthand, or API generation.
  **L281 CN**: 定义宏 `_vel_vcmpuw_vsvvl`，用于条件编译、简写或 API 生成。
- **L282 EN**: Defines macro `_vel_vcmpuw_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L282 CN**: 定义宏 `_vel_vcmpuw_vvvmvl`，用于条件编译、简写或 API 生成。
- **L283 EN**: Defines macro `_vel_vcmpuw_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L283 CN**: 定义宏 `_vel_vcmpuw_vsvmvl`，用于条件编译、简写或 API 生成。
- **L284 EN**: Defines macro `_vel_pvcmpu_vvvl` for conditional compilation, shorthand, or API generation.
  **L284 CN**: 定义宏 `_vel_pvcmpu_vvvl`，用于条件编译、简写或 API 生成。
- **L285 EN**: Defines macro `_vel_pvcmpu_vvvvl` for conditional compilation, shorthand, or API generation.
  **L285 CN**: 定义宏 `_vel_pvcmpu_vvvvl`，用于条件编译、简写或 API 生成。
- **L286 EN**: Defines macro `_vel_pvcmpu_vsvl` for conditional compilation, shorthand, or API generation.
  **L286 CN**: 定义宏 `_vel_pvcmpu_vsvl`，用于条件编译、简写或 API 生成。
- **L287 EN**: Defines macro `_vel_pvcmpu_vsvvl` for conditional compilation, shorthand, or API generation.
  **L287 CN**: 定义宏 `_vel_pvcmpu_vsvvl`，用于条件编译、简写或 API 生成。
- **L288 EN**: Defines macro `_vel_pvcmpu_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L288 CN**: 定义宏 `_vel_pvcmpu_vvvMvl`，用于条件编译、简写或 API 生成。

### Lines 289-312

````c
#define _vel_pvcmpu_vsvMvl __builtin_ve_vl_pvcmpu_vsvMvl
#define _vel_vcmpswsx_vvvl __builtin_ve_vl_vcmpswsx_vvvl
#define _vel_vcmpswsx_vvvvl __builtin_ve_vl_vcmpswsx_vvvvl
#define _vel_vcmpswsx_vsvl __builtin_ve_vl_vcmpswsx_vsvl
#define _vel_vcmpswsx_vsvvl __builtin_ve_vl_vcmpswsx_vsvvl
#define _vel_vcmpswsx_vvvmvl __builtin_ve_vl_vcmpswsx_vvvmvl
#define _vel_vcmpswsx_vsvmvl __builtin_ve_vl_vcmpswsx_vsvmvl
#define _vel_vcmpswzx_vvvl __builtin_ve_vl_vcmpswzx_vvvl
#define _vel_vcmpswzx_vvvvl __builtin_ve_vl_vcmpswzx_vvvvl
#define _vel_vcmpswzx_vsvl __builtin_ve_vl_vcmpswzx_vsvl
#define _vel_vcmpswzx_vsvvl __builtin_ve_vl_vcmpswzx_vsvvl
#define _vel_vcmpswzx_vvvmvl __builtin_ve_vl_vcmpswzx_vvvmvl
#define _vel_vcmpswzx_vsvmvl __builtin_ve_vl_vcmpswzx_vsvmvl
#define _vel_pvcmps_vvvl __builtin_ve_vl_pvcmps_vvvl
#define _vel_pvcmps_vvvvl __builtin_ve_vl_pvcmps_vvvvl
#define _vel_pvcmps_vsvl __builtin_ve_vl_pvcmps_vsvl
#define _vel_pvcmps_vsvvl __builtin_ve_vl_pvcmps_vsvvl
#define _vel_pvcmps_vvvMvl __builtin_ve_vl_pvcmps_vvvMvl
#define _vel_pvcmps_vsvMvl __builtin_ve_vl_pvcmps_vsvMvl
#define _vel_vcmpsl_vvvl __builtin_ve_vl_vcmpsl_vvvl
#define _vel_vcmpsl_vvvvl __builtin_ve_vl_vcmpsl_vvvvl
#define _vel_vcmpsl_vsvl __builtin_ve_vl_vcmpsl_vsvl
#define _vel_vcmpsl_vsvvl __builtin_ve_vl_vcmpsl_vsvvl
#define _vel_vcmpsl_vvvmvl __builtin_ve_vl_vcmpsl_vvvmvl
````
- **L289 EN**: Defines macro `_vel_pvcmpu_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L289 CN**: 定义宏 `_vel_pvcmpu_vsvMvl`，用于条件编译、简写或 API 生成。
- **L290 EN**: Defines macro `_vel_vcmpswsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L290 CN**: 定义宏 `_vel_vcmpswsx_vvvl`，用于条件编译、简写或 API 生成。
- **L291 EN**: Defines macro `_vel_vcmpswsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L291 CN**: 定义宏 `_vel_vcmpswsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L292 EN**: Defines macro `_vel_vcmpswsx_vsvl` for conditional compilation, shorthand, or API generation.
  **L292 CN**: 定义宏 `_vel_vcmpswsx_vsvl`，用于条件编译、简写或 API 生成。
- **L293 EN**: Defines macro `_vel_vcmpswsx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L293 CN**: 定义宏 `_vel_vcmpswsx_vsvvl`，用于条件编译、简写或 API 生成。
- **L294 EN**: Defines macro `_vel_vcmpswsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L294 CN**: 定义宏 `_vel_vcmpswsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L295 EN**: Defines macro `_vel_vcmpswsx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L295 CN**: 定义宏 `_vel_vcmpswsx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L296 EN**: Defines macro `_vel_vcmpswzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L296 CN**: 定义宏 `_vel_vcmpswzx_vvvl`，用于条件编译、简写或 API 生成。
- **L297 EN**: Defines macro `_vel_vcmpswzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L297 CN**: 定义宏 `_vel_vcmpswzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L298 EN**: Defines macro `_vel_vcmpswzx_vsvl` for conditional compilation, shorthand, or API generation.
  **L298 CN**: 定义宏 `_vel_vcmpswzx_vsvl`，用于条件编译、简写或 API 生成。
- **L299 EN**: Defines macro `_vel_vcmpswzx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L299 CN**: 定义宏 `_vel_vcmpswzx_vsvvl`，用于条件编译、简写或 API 生成。
- **L300 EN**: Defines macro `_vel_vcmpswzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L300 CN**: 定义宏 `_vel_vcmpswzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L301 EN**: Defines macro `_vel_vcmpswzx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L301 CN**: 定义宏 `_vel_vcmpswzx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L302 EN**: Defines macro `_vel_pvcmps_vvvl` for conditional compilation, shorthand, or API generation.
  **L302 CN**: 定义宏 `_vel_pvcmps_vvvl`，用于条件编译、简写或 API 生成。
- **L303 EN**: Defines macro `_vel_pvcmps_vvvvl` for conditional compilation, shorthand, or API generation.
  **L303 CN**: 定义宏 `_vel_pvcmps_vvvvl`，用于条件编译、简写或 API 生成。
- **L304 EN**: Defines macro `_vel_pvcmps_vsvl` for conditional compilation, shorthand, or API generation.
  **L304 CN**: 定义宏 `_vel_pvcmps_vsvl`，用于条件编译、简写或 API 生成。
- **L305 EN**: Defines macro `_vel_pvcmps_vsvvl` for conditional compilation, shorthand, or API generation.
  **L305 CN**: 定义宏 `_vel_pvcmps_vsvvl`，用于条件编译、简写或 API 生成。
- **L306 EN**: Defines macro `_vel_pvcmps_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L306 CN**: 定义宏 `_vel_pvcmps_vvvMvl`，用于条件编译、简写或 API 生成。
- **L307 EN**: Defines macro `_vel_pvcmps_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L307 CN**: 定义宏 `_vel_pvcmps_vsvMvl`，用于条件编译、简写或 API 生成。
- **L308 EN**: Defines macro `_vel_vcmpsl_vvvl` for conditional compilation, shorthand, or API generation.
  **L308 CN**: 定义宏 `_vel_vcmpsl_vvvl`，用于条件编译、简写或 API 生成。
- **L309 EN**: Defines macro `_vel_vcmpsl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L309 CN**: 定义宏 `_vel_vcmpsl_vvvvl`，用于条件编译、简写或 API 生成。
- **L310 EN**: Defines macro `_vel_vcmpsl_vsvl` for conditional compilation, shorthand, or API generation.
  **L310 CN**: 定义宏 `_vel_vcmpsl_vsvl`，用于条件编译、简写或 API 生成。
- **L311 EN**: Defines macro `_vel_vcmpsl_vsvvl` for conditional compilation, shorthand, or API generation.
  **L311 CN**: 定义宏 `_vel_vcmpsl_vsvvl`，用于条件编译、简写或 API 生成。
- **L312 EN**: Defines macro `_vel_vcmpsl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L312 CN**: 定义宏 `_vel_vcmpsl_vvvmvl`，用于条件编译、简写或 API 生成。

### Lines 313-336

````c
#define _vel_vcmpsl_vsvmvl __builtin_ve_vl_vcmpsl_vsvmvl
#define _vel_vmaxswsx_vvvl __builtin_ve_vl_vmaxswsx_vvvl
#define _vel_vmaxswsx_vvvvl __builtin_ve_vl_vmaxswsx_vvvvl
#define _vel_vmaxswsx_vsvl __builtin_ve_vl_vmaxswsx_vsvl
#define _vel_vmaxswsx_vsvvl __builtin_ve_vl_vmaxswsx_vsvvl
#define _vel_vmaxswsx_vvvmvl __builtin_ve_vl_vmaxswsx_vvvmvl
#define _vel_vmaxswsx_vsvmvl __builtin_ve_vl_vmaxswsx_vsvmvl
#define _vel_vmaxswzx_vvvl __builtin_ve_vl_vmaxswzx_vvvl
#define _vel_vmaxswzx_vvvvl __builtin_ve_vl_vmaxswzx_vvvvl
#define _vel_vmaxswzx_vsvl __builtin_ve_vl_vmaxswzx_vsvl
#define _vel_vmaxswzx_vsvvl __builtin_ve_vl_vmaxswzx_vsvvl
#define _vel_vmaxswzx_vvvmvl __builtin_ve_vl_vmaxswzx_vvvmvl
#define _vel_vmaxswzx_vsvmvl __builtin_ve_vl_vmaxswzx_vsvmvl
#define _vel_pvmaxs_vvvl __builtin_ve_vl_pvmaxs_vvvl
#define _vel_pvmaxs_vvvvl __builtin_ve_vl_pvmaxs_vvvvl
#define _vel_pvmaxs_vsvl __builtin_ve_vl_pvmaxs_vsvl
#define _vel_pvmaxs_vsvvl __builtin_ve_vl_pvmaxs_vsvvl
#define _vel_pvmaxs_vvvMvl __builtin_ve_vl_pvmaxs_vvvMvl
#define _vel_pvmaxs_vsvMvl __builtin_ve_vl_pvmaxs_vsvMvl
#define _vel_vminswsx_vvvl __builtin_ve_vl_vminswsx_vvvl
#define _vel_vminswsx_vvvvl __builtin_ve_vl_vminswsx_vvvvl
#define _vel_vminswsx_vsvl __builtin_ve_vl_vminswsx_vsvl
#define _vel_vminswsx_vsvvl __builtin_ve_vl_vminswsx_vsvvl
#define _vel_vminswsx_vvvmvl __builtin_ve_vl_vminswsx_vvvmvl
````
- **L313 EN**: Defines macro `_vel_vcmpsl_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L313 CN**: 定义宏 `_vel_vcmpsl_vsvmvl`，用于条件编译、简写或 API 生成。
- **L314 EN**: Defines macro `_vel_vmaxswsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L314 CN**: 定义宏 `_vel_vmaxswsx_vvvl`，用于条件编译、简写或 API 生成。
- **L315 EN**: Defines macro `_vel_vmaxswsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L315 CN**: 定义宏 `_vel_vmaxswsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L316 EN**: Defines macro `_vel_vmaxswsx_vsvl` for conditional compilation, shorthand, or API generation.
  **L316 CN**: 定义宏 `_vel_vmaxswsx_vsvl`，用于条件编译、简写或 API 生成。
- **L317 EN**: Defines macro `_vel_vmaxswsx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L317 CN**: 定义宏 `_vel_vmaxswsx_vsvvl`，用于条件编译、简写或 API 生成。
- **L318 EN**: Defines macro `_vel_vmaxswsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L318 CN**: 定义宏 `_vel_vmaxswsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L319 EN**: Defines macro `_vel_vmaxswsx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L319 CN**: 定义宏 `_vel_vmaxswsx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L320 EN**: Defines macro `_vel_vmaxswzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L320 CN**: 定义宏 `_vel_vmaxswzx_vvvl`，用于条件编译、简写或 API 生成。
- **L321 EN**: Defines macro `_vel_vmaxswzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L321 CN**: 定义宏 `_vel_vmaxswzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L322 EN**: Defines macro `_vel_vmaxswzx_vsvl` for conditional compilation, shorthand, or API generation.
  **L322 CN**: 定义宏 `_vel_vmaxswzx_vsvl`，用于条件编译、简写或 API 生成。
- **L323 EN**: Defines macro `_vel_vmaxswzx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L323 CN**: 定义宏 `_vel_vmaxswzx_vsvvl`，用于条件编译、简写或 API 生成。
- **L324 EN**: Defines macro `_vel_vmaxswzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L324 CN**: 定义宏 `_vel_vmaxswzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L325 EN**: Defines macro `_vel_vmaxswzx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L325 CN**: 定义宏 `_vel_vmaxswzx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L326 EN**: Defines macro `_vel_pvmaxs_vvvl` for conditional compilation, shorthand, or API generation.
  **L326 CN**: 定义宏 `_vel_pvmaxs_vvvl`，用于条件编译、简写或 API 生成。
- **L327 EN**: Defines macro `_vel_pvmaxs_vvvvl` for conditional compilation, shorthand, or API generation.
  **L327 CN**: 定义宏 `_vel_pvmaxs_vvvvl`，用于条件编译、简写或 API 生成。
- **L328 EN**: Defines macro `_vel_pvmaxs_vsvl` for conditional compilation, shorthand, or API generation.
  **L328 CN**: 定义宏 `_vel_pvmaxs_vsvl`，用于条件编译、简写或 API 生成。
- **L329 EN**: Defines macro `_vel_pvmaxs_vsvvl` for conditional compilation, shorthand, or API generation.
  **L329 CN**: 定义宏 `_vel_pvmaxs_vsvvl`，用于条件编译、简写或 API 生成。
- **L330 EN**: Defines macro `_vel_pvmaxs_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L330 CN**: 定义宏 `_vel_pvmaxs_vvvMvl`，用于条件编译、简写或 API 生成。
- **L331 EN**: Defines macro `_vel_pvmaxs_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L331 CN**: 定义宏 `_vel_pvmaxs_vsvMvl`，用于条件编译、简写或 API 生成。
- **L332 EN**: Defines macro `_vel_vminswsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L332 CN**: 定义宏 `_vel_vminswsx_vvvl`，用于条件编译、简写或 API 生成。
- **L333 EN**: Defines macro `_vel_vminswsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L333 CN**: 定义宏 `_vel_vminswsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L334 EN**: Defines macro `_vel_vminswsx_vsvl` for conditional compilation, shorthand, or API generation.
  **L334 CN**: 定义宏 `_vel_vminswsx_vsvl`，用于条件编译、简写或 API 生成。
- **L335 EN**: Defines macro `_vel_vminswsx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L335 CN**: 定义宏 `_vel_vminswsx_vsvvl`，用于条件编译、简写或 API 生成。
- **L336 EN**: Defines macro `_vel_vminswsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L336 CN**: 定义宏 `_vel_vminswsx_vvvmvl`，用于条件编译、简写或 API 生成。

### Lines 337-360

````c
#define _vel_vminswsx_vsvmvl __builtin_ve_vl_vminswsx_vsvmvl
#define _vel_vminswzx_vvvl __builtin_ve_vl_vminswzx_vvvl
#define _vel_vminswzx_vvvvl __builtin_ve_vl_vminswzx_vvvvl
#define _vel_vminswzx_vsvl __builtin_ve_vl_vminswzx_vsvl
#define _vel_vminswzx_vsvvl __builtin_ve_vl_vminswzx_vsvvl
#define _vel_vminswzx_vvvmvl __builtin_ve_vl_vminswzx_vvvmvl
#define _vel_vminswzx_vsvmvl __builtin_ve_vl_vminswzx_vsvmvl
#define _vel_pvmins_vvvl __builtin_ve_vl_pvmins_vvvl
#define _vel_pvmins_vvvvl __builtin_ve_vl_pvmins_vvvvl
#define _vel_pvmins_vsvl __builtin_ve_vl_pvmins_vsvl
#define _vel_pvmins_vsvvl __builtin_ve_vl_pvmins_vsvvl
#define _vel_pvmins_vvvMvl __builtin_ve_vl_pvmins_vvvMvl
#define _vel_pvmins_vsvMvl __builtin_ve_vl_pvmins_vsvMvl
#define _vel_vmaxsl_vvvl __builtin_ve_vl_vmaxsl_vvvl
#define _vel_vmaxsl_vvvvl __builtin_ve_vl_vmaxsl_vvvvl
#define _vel_vmaxsl_vsvl __builtin_ve_vl_vmaxsl_vsvl
#define _vel_vmaxsl_vsvvl __builtin_ve_vl_vmaxsl_vsvvl
#define _vel_vmaxsl_vvvmvl __builtin_ve_vl_vmaxsl_vvvmvl
#define _vel_vmaxsl_vsvmvl __builtin_ve_vl_vmaxsl_vsvmvl
#define _vel_vminsl_vvvl __builtin_ve_vl_vminsl_vvvl
#define _vel_vminsl_vvvvl __builtin_ve_vl_vminsl_vvvvl
#define _vel_vminsl_vsvl __builtin_ve_vl_vminsl_vsvl
#define _vel_vminsl_vsvvl __builtin_ve_vl_vminsl_vsvvl
#define _vel_vminsl_vvvmvl __builtin_ve_vl_vminsl_vvvmvl
````
- **L337 EN**: Defines macro `_vel_vminswsx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L337 CN**: 定义宏 `_vel_vminswsx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L338 EN**: Defines macro `_vel_vminswzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L338 CN**: 定义宏 `_vel_vminswzx_vvvl`，用于条件编译、简写或 API 生成。
- **L339 EN**: Defines macro `_vel_vminswzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L339 CN**: 定义宏 `_vel_vminswzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L340 EN**: Defines macro `_vel_vminswzx_vsvl` for conditional compilation, shorthand, or API generation.
  **L340 CN**: 定义宏 `_vel_vminswzx_vsvl`，用于条件编译、简写或 API 生成。
- **L341 EN**: Defines macro `_vel_vminswzx_vsvvl` for conditional compilation, shorthand, or API generation.
  **L341 CN**: 定义宏 `_vel_vminswzx_vsvvl`，用于条件编译、简写或 API 生成。
- **L342 EN**: Defines macro `_vel_vminswzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L342 CN**: 定义宏 `_vel_vminswzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L343 EN**: Defines macro `_vel_vminswzx_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L343 CN**: 定义宏 `_vel_vminswzx_vsvmvl`，用于条件编译、简写或 API 生成。
- **L344 EN**: Defines macro `_vel_pvmins_vvvl` for conditional compilation, shorthand, or API generation.
  **L344 CN**: 定义宏 `_vel_pvmins_vvvl`，用于条件编译、简写或 API 生成。
- **L345 EN**: Defines macro `_vel_pvmins_vvvvl` for conditional compilation, shorthand, or API generation.
  **L345 CN**: 定义宏 `_vel_pvmins_vvvvl`，用于条件编译、简写或 API 生成。
- **L346 EN**: Defines macro `_vel_pvmins_vsvl` for conditional compilation, shorthand, or API generation.
  **L346 CN**: 定义宏 `_vel_pvmins_vsvl`，用于条件编译、简写或 API 生成。
- **L347 EN**: Defines macro `_vel_pvmins_vsvvl` for conditional compilation, shorthand, or API generation.
  **L347 CN**: 定义宏 `_vel_pvmins_vsvvl`，用于条件编译、简写或 API 生成。
- **L348 EN**: Defines macro `_vel_pvmins_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L348 CN**: 定义宏 `_vel_pvmins_vvvMvl`，用于条件编译、简写或 API 生成。
- **L349 EN**: Defines macro `_vel_pvmins_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L349 CN**: 定义宏 `_vel_pvmins_vsvMvl`，用于条件编译、简写或 API 生成。
- **L350 EN**: Defines macro `_vel_vmaxsl_vvvl` for conditional compilation, shorthand, or API generation.
  **L350 CN**: 定义宏 `_vel_vmaxsl_vvvl`，用于条件编译、简写或 API 生成。
- **L351 EN**: Defines macro `_vel_vmaxsl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L351 CN**: 定义宏 `_vel_vmaxsl_vvvvl`，用于条件编译、简写或 API 生成。
- **L352 EN**: Defines macro `_vel_vmaxsl_vsvl` for conditional compilation, shorthand, or API generation.
  **L352 CN**: 定义宏 `_vel_vmaxsl_vsvl`，用于条件编译、简写或 API 生成。
- **L353 EN**: Defines macro `_vel_vmaxsl_vsvvl` for conditional compilation, shorthand, or API generation.
  **L353 CN**: 定义宏 `_vel_vmaxsl_vsvvl`，用于条件编译、简写或 API 生成。
- **L354 EN**: Defines macro `_vel_vmaxsl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L354 CN**: 定义宏 `_vel_vmaxsl_vvvmvl`，用于条件编译、简写或 API 生成。
- **L355 EN**: Defines macro `_vel_vmaxsl_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L355 CN**: 定义宏 `_vel_vmaxsl_vsvmvl`，用于条件编译、简写或 API 生成。
- **L356 EN**: Defines macro `_vel_vminsl_vvvl` for conditional compilation, shorthand, or API generation.
  **L356 CN**: 定义宏 `_vel_vminsl_vvvl`，用于条件编译、简写或 API 生成。
- **L357 EN**: Defines macro `_vel_vminsl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L357 CN**: 定义宏 `_vel_vminsl_vvvvl`，用于条件编译、简写或 API 生成。
- **L358 EN**: Defines macro `_vel_vminsl_vsvl` for conditional compilation, shorthand, or API generation.
  **L358 CN**: 定义宏 `_vel_vminsl_vsvl`，用于条件编译、简写或 API 生成。
- **L359 EN**: Defines macro `_vel_vminsl_vsvvl` for conditional compilation, shorthand, or API generation.
  **L359 CN**: 定义宏 `_vel_vminsl_vsvvl`，用于条件编译、简写或 API 生成。
- **L360 EN**: Defines macro `_vel_vminsl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L360 CN**: 定义宏 `_vel_vminsl_vvvmvl`，用于条件编译、简写或 API 生成。

### Lines 361-384

````c
#define _vel_vminsl_vsvmvl __builtin_ve_vl_vminsl_vsvmvl
#define _vel_vand_vvvl __builtin_ve_vl_vand_vvvl
#define _vel_vand_vvvvl __builtin_ve_vl_vand_vvvvl
#define _vel_vand_vsvl __builtin_ve_vl_vand_vsvl
#define _vel_vand_vsvvl __builtin_ve_vl_vand_vsvvl
#define _vel_vand_vvvmvl __builtin_ve_vl_vand_vvvmvl
#define _vel_vand_vsvmvl __builtin_ve_vl_vand_vsvmvl
#define _vel_pvand_vvvl __builtin_ve_vl_pvand_vvvl
#define _vel_pvand_vvvvl __builtin_ve_vl_pvand_vvvvl
#define _vel_pvand_vsvl __builtin_ve_vl_pvand_vsvl
#define _vel_pvand_vsvvl __builtin_ve_vl_pvand_vsvvl
#define _vel_pvand_vvvMvl __builtin_ve_vl_pvand_vvvMvl
#define _vel_pvand_vsvMvl __builtin_ve_vl_pvand_vsvMvl
#define _vel_vor_vvvl __builtin_ve_vl_vor_vvvl
#define _vel_vor_vvvvl __builtin_ve_vl_vor_vvvvl
#define _vel_vor_vsvl __builtin_ve_vl_vor_vsvl
#define _vel_vor_vsvvl __builtin_ve_vl_vor_vsvvl
#define _vel_vor_vvvmvl __builtin_ve_vl_vor_vvvmvl
#define _vel_vor_vsvmvl __builtin_ve_vl_vor_vsvmvl
#define _vel_pvor_vvvl __builtin_ve_vl_pvor_vvvl
#define _vel_pvor_vvvvl __builtin_ve_vl_pvor_vvvvl
#define _vel_pvor_vsvl __builtin_ve_vl_pvor_vsvl
#define _vel_pvor_vsvvl __builtin_ve_vl_pvor_vsvvl
#define _vel_pvor_vvvMvl __builtin_ve_vl_pvor_vvvMvl
````
- **L361 EN**: Defines macro `_vel_vminsl_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L361 CN**: 定义宏 `_vel_vminsl_vsvmvl`，用于条件编译、简写或 API 生成。
- **L362 EN**: Defines macro `_vel_vand_vvvl` for conditional compilation, shorthand, or API generation.
  **L362 CN**: 定义宏 `_vel_vand_vvvl`，用于条件编译、简写或 API 生成。
- **L363 EN**: Defines macro `_vel_vand_vvvvl` for conditional compilation, shorthand, or API generation.
  **L363 CN**: 定义宏 `_vel_vand_vvvvl`，用于条件编译、简写或 API 生成。
- **L364 EN**: Defines macro `_vel_vand_vsvl` for conditional compilation, shorthand, or API generation.
  **L364 CN**: 定义宏 `_vel_vand_vsvl`，用于条件编译、简写或 API 生成。
- **L365 EN**: Defines macro `_vel_vand_vsvvl` for conditional compilation, shorthand, or API generation.
  **L365 CN**: 定义宏 `_vel_vand_vsvvl`，用于条件编译、简写或 API 生成。
- **L366 EN**: Defines macro `_vel_vand_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L366 CN**: 定义宏 `_vel_vand_vvvmvl`，用于条件编译、简写或 API 生成。
- **L367 EN**: Defines macro `_vel_vand_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L367 CN**: 定义宏 `_vel_vand_vsvmvl`，用于条件编译、简写或 API 生成。
- **L368 EN**: Defines macro `_vel_pvand_vvvl` for conditional compilation, shorthand, or API generation.
  **L368 CN**: 定义宏 `_vel_pvand_vvvl`，用于条件编译、简写或 API 生成。
- **L369 EN**: Defines macro `_vel_pvand_vvvvl` for conditional compilation, shorthand, or API generation.
  **L369 CN**: 定义宏 `_vel_pvand_vvvvl`，用于条件编译、简写或 API 生成。
- **L370 EN**: Defines macro `_vel_pvand_vsvl` for conditional compilation, shorthand, or API generation.
  **L370 CN**: 定义宏 `_vel_pvand_vsvl`，用于条件编译、简写或 API 生成。
- **L371 EN**: Defines macro `_vel_pvand_vsvvl` for conditional compilation, shorthand, or API generation.
  **L371 CN**: 定义宏 `_vel_pvand_vsvvl`，用于条件编译、简写或 API 生成。
- **L372 EN**: Defines macro `_vel_pvand_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L372 CN**: 定义宏 `_vel_pvand_vvvMvl`，用于条件编译、简写或 API 生成。
- **L373 EN**: Defines macro `_vel_pvand_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L373 CN**: 定义宏 `_vel_pvand_vsvMvl`，用于条件编译、简写或 API 生成。
- **L374 EN**: Defines macro `_vel_vor_vvvl` for conditional compilation, shorthand, or API generation.
  **L374 CN**: 定义宏 `_vel_vor_vvvl`，用于条件编译、简写或 API 生成。
- **L375 EN**: Defines macro `_vel_vor_vvvvl` for conditional compilation, shorthand, or API generation.
  **L375 CN**: 定义宏 `_vel_vor_vvvvl`，用于条件编译、简写或 API 生成。
- **L376 EN**: Defines macro `_vel_vor_vsvl` for conditional compilation, shorthand, or API generation.
  **L376 CN**: 定义宏 `_vel_vor_vsvl`，用于条件编译、简写或 API 生成。
- **L377 EN**: Defines macro `_vel_vor_vsvvl` for conditional compilation, shorthand, or API generation.
  **L377 CN**: 定义宏 `_vel_vor_vsvvl`，用于条件编译、简写或 API 生成。
- **L378 EN**: Defines macro `_vel_vor_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L378 CN**: 定义宏 `_vel_vor_vvvmvl`，用于条件编译、简写或 API 生成。
- **L379 EN**: Defines macro `_vel_vor_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L379 CN**: 定义宏 `_vel_vor_vsvmvl`，用于条件编译、简写或 API 生成。
- **L380 EN**: Defines macro `_vel_pvor_vvvl` for conditional compilation, shorthand, or API generation.
  **L380 CN**: 定义宏 `_vel_pvor_vvvl`，用于条件编译、简写或 API 生成。
- **L381 EN**: Defines macro `_vel_pvor_vvvvl` for conditional compilation, shorthand, or API generation.
  **L381 CN**: 定义宏 `_vel_pvor_vvvvl`，用于条件编译、简写或 API 生成。
- **L382 EN**: Defines macro `_vel_pvor_vsvl` for conditional compilation, shorthand, or API generation.
  **L382 CN**: 定义宏 `_vel_pvor_vsvl`，用于条件编译、简写或 API 生成。
- **L383 EN**: Defines macro `_vel_pvor_vsvvl` for conditional compilation, shorthand, or API generation.
  **L383 CN**: 定义宏 `_vel_pvor_vsvvl`，用于条件编译、简写或 API 生成。
- **L384 EN**: Defines macro `_vel_pvor_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L384 CN**: 定义宏 `_vel_pvor_vvvMvl`，用于条件编译、简写或 API 生成。

### Lines 385-408

````c
#define _vel_pvor_vsvMvl __builtin_ve_vl_pvor_vsvMvl
#define _vel_vxor_vvvl __builtin_ve_vl_vxor_vvvl
#define _vel_vxor_vvvvl __builtin_ve_vl_vxor_vvvvl
#define _vel_vxor_vsvl __builtin_ve_vl_vxor_vsvl
#define _vel_vxor_vsvvl __builtin_ve_vl_vxor_vsvvl
#define _vel_vxor_vvvmvl __builtin_ve_vl_vxor_vvvmvl
#define _vel_vxor_vsvmvl __builtin_ve_vl_vxor_vsvmvl
#define _vel_pvxor_vvvl __builtin_ve_vl_pvxor_vvvl
#define _vel_pvxor_vvvvl __builtin_ve_vl_pvxor_vvvvl
#define _vel_pvxor_vsvl __builtin_ve_vl_pvxor_vsvl
#define _vel_pvxor_vsvvl __builtin_ve_vl_pvxor_vsvvl
#define _vel_pvxor_vvvMvl __builtin_ve_vl_pvxor_vvvMvl
#define _vel_pvxor_vsvMvl __builtin_ve_vl_pvxor_vsvMvl
#define _vel_veqv_vvvl __builtin_ve_vl_veqv_vvvl
#define _vel_veqv_vvvvl __builtin_ve_vl_veqv_vvvvl
#define _vel_veqv_vsvl __builtin_ve_vl_veqv_vsvl
#define _vel_veqv_vsvvl __builtin_ve_vl_veqv_vsvvl
#define _vel_veqv_vvvmvl __builtin_ve_vl_veqv_vvvmvl
#define _vel_veqv_vsvmvl __builtin_ve_vl_veqv_vsvmvl
#define _vel_pveqv_vvvl __builtin_ve_vl_pveqv_vvvl
#define _vel_pveqv_vvvvl __builtin_ve_vl_pveqv_vvvvl
#define _vel_pveqv_vsvl __builtin_ve_vl_pveqv_vsvl
#define _vel_pveqv_vsvvl __builtin_ve_vl_pveqv_vsvvl
#define _vel_pveqv_vvvMvl __builtin_ve_vl_pveqv_vvvMvl
````
- **L385 EN**: Defines macro `_vel_pvor_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L385 CN**: 定义宏 `_vel_pvor_vsvMvl`，用于条件编译、简写或 API 生成。
- **L386 EN**: Defines macro `_vel_vxor_vvvl` for conditional compilation, shorthand, or API generation.
  **L386 CN**: 定义宏 `_vel_vxor_vvvl`，用于条件编译、简写或 API 生成。
- **L387 EN**: Defines macro `_vel_vxor_vvvvl` for conditional compilation, shorthand, or API generation.
  **L387 CN**: 定义宏 `_vel_vxor_vvvvl`，用于条件编译、简写或 API 生成。
- **L388 EN**: Defines macro `_vel_vxor_vsvl` for conditional compilation, shorthand, or API generation.
  **L388 CN**: 定义宏 `_vel_vxor_vsvl`，用于条件编译、简写或 API 生成。
- **L389 EN**: Defines macro `_vel_vxor_vsvvl` for conditional compilation, shorthand, or API generation.
  **L389 CN**: 定义宏 `_vel_vxor_vsvvl`，用于条件编译、简写或 API 生成。
- **L390 EN**: Defines macro `_vel_vxor_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L390 CN**: 定义宏 `_vel_vxor_vvvmvl`，用于条件编译、简写或 API 生成。
- **L391 EN**: Defines macro `_vel_vxor_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L391 CN**: 定义宏 `_vel_vxor_vsvmvl`，用于条件编译、简写或 API 生成。
- **L392 EN**: Defines macro `_vel_pvxor_vvvl` for conditional compilation, shorthand, or API generation.
  **L392 CN**: 定义宏 `_vel_pvxor_vvvl`，用于条件编译、简写或 API 生成。
- **L393 EN**: Defines macro `_vel_pvxor_vvvvl` for conditional compilation, shorthand, or API generation.
  **L393 CN**: 定义宏 `_vel_pvxor_vvvvl`，用于条件编译、简写或 API 生成。
- **L394 EN**: Defines macro `_vel_pvxor_vsvl` for conditional compilation, shorthand, or API generation.
  **L394 CN**: 定义宏 `_vel_pvxor_vsvl`，用于条件编译、简写或 API 生成。
- **L395 EN**: Defines macro `_vel_pvxor_vsvvl` for conditional compilation, shorthand, or API generation.
  **L395 CN**: 定义宏 `_vel_pvxor_vsvvl`，用于条件编译、简写或 API 生成。
- **L396 EN**: Defines macro `_vel_pvxor_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L396 CN**: 定义宏 `_vel_pvxor_vvvMvl`，用于条件编译、简写或 API 生成。
- **L397 EN**: Defines macro `_vel_pvxor_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L397 CN**: 定义宏 `_vel_pvxor_vsvMvl`，用于条件编译、简写或 API 生成。
- **L398 EN**: Defines macro `_vel_veqv_vvvl` for conditional compilation, shorthand, or API generation.
  **L398 CN**: 定义宏 `_vel_veqv_vvvl`，用于条件编译、简写或 API 生成。
- **L399 EN**: Defines macro `_vel_veqv_vvvvl` for conditional compilation, shorthand, or API generation.
  **L399 CN**: 定义宏 `_vel_veqv_vvvvl`，用于条件编译、简写或 API 生成。
- **L400 EN**: Defines macro `_vel_veqv_vsvl` for conditional compilation, shorthand, or API generation.
  **L400 CN**: 定义宏 `_vel_veqv_vsvl`，用于条件编译、简写或 API 生成。
- **L401 EN**: Defines macro `_vel_veqv_vsvvl` for conditional compilation, shorthand, or API generation.
  **L401 CN**: 定义宏 `_vel_veqv_vsvvl`，用于条件编译、简写或 API 生成。
- **L402 EN**: Defines macro `_vel_veqv_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L402 CN**: 定义宏 `_vel_veqv_vvvmvl`，用于条件编译、简写或 API 生成。
- **L403 EN**: Defines macro `_vel_veqv_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L403 CN**: 定义宏 `_vel_veqv_vsvmvl`，用于条件编译、简写或 API 生成。
- **L404 EN**: Defines macro `_vel_pveqv_vvvl` for conditional compilation, shorthand, or API generation.
  **L404 CN**: 定义宏 `_vel_pveqv_vvvl`，用于条件编译、简写或 API 生成。
- **L405 EN**: Defines macro `_vel_pveqv_vvvvl` for conditional compilation, shorthand, or API generation.
  **L405 CN**: 定义宏 `_vel_pveqv_vvvvl`，用于条件编译、简写或 API 生成。
- **L406 EN**: Defines macro `_vel_pveqv_vsvl` for conditional compilation, shorthand, or API generation.
  **L406 CN**: 定义宏 `_vel_pveqv_vsvl`，用于条件编译、简写或 API 生成。
- **L407 EN**: Defines macro `_vel_pveqv_vsvvl` for conditional compilation, shorthand, or API generation.
  **L407 CN**: 定义宏 `_vel_pveqv_vsvvl`，用于条件编译、简写或 API 生成。
- **L408 EN**: Defines macro `_vel_pveqv_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L408 CN**: 定义宏 `_vel_pveqv_vvvMvl`，用于条件编译、简写或 API 生成。

### Lines 409-432

````c
#define _vel_pveqv_vsvMvl __builtin_ve_vl_pveqv_vsvMvl
#define _vel_vldz_vvl __builtin_ve_vl_vldz_vvl
#define _vel_vldz_vvvl __builtin_ve_vl_vldz_vvvl
#define _vel_vldz_vvmvl __builtin_ve_vl_vldz_vvmvl
#define _vel_pvldzlo_vvl __builtin_ve_vl_pvldzlo_vvl
#define _vel_pvldzlo_vvvl __builtin_ve_vl_pvldzlo_vvvl
#define _vel_pvldzlo_vvmvl __builtin_ve_vl_pvldzlo_vvmvl
#define _vel_pvldzup_vvl __builtin_ve_vl_pvldzup_vvl
#define _vel_pvldzup_vvvl __builtin_ve_vl_pvldzup_vvvl
#define _vel_pvldzup_vvmvl __builtin_ve_vl_pvldzup_vvmvl
#define _vel_pvldz_vvl __builtin_ve_vl_pvldz_vvl
#define _vel_pvldz_vvvl __builtin_ve_vl_pvldz_vvvl
#define _vel_pvldz_vvMvl __builtin_ve_vl_pvldz_vvMvl
#define _vel_vpcnt_vvl __builtin_ve_vl_vpcnt_vvl
#define _vel_vpcnt_vvvl __builtin_ve_vl_vpcnt_vvvl
#define _vel_vpcnt_vvmvl __builtin_ve_vl_vpcnt_vvmvl
#define _vel_pvpcntlo_vvl __builtin_ve_vl_pvpcntlo_vvl
#define _vel_pvpcntlo_vvvl __builtin_ve_vl_pvpcntlo_vvvl
#define _vel_pvpcntlo_vvmvl __builtin_ve_vl_pvpcntlo_vvmvl
#define _vel_pvpcntup_vvl __builtin_ve_vl_pvpcntup_vvl
#define _vel_pvpcntup_vvvl __builtin_ve_vl_pvpcntup_vvvl
#define _vel_pvpcntup_vvmvl __builtin_ve_vl_pvpcntup_vvmvl
#define _vel_pvpcnt_vvl __builtin_ve_vl_pvpcnt_vvl
#define _vel_pvpcnt_vvvl __builtin_ve_vl_pvpcnt_vvvl
````
- **L409 EN**: Defines macro `_vel_pveqv_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L409 CN**: 定义宏 `_vel_pveqv_vsvMvl`，用于条件编译、简写或 API 生成。
- **L410 EN**: Defines macro `_vel_vldz_vvl` for conditional compilation, shorthand, or API generation.
  **L410 CN**: 定义宏 `_vel_vldz_vvl`，用于条件编译、简写或 API 生成。
- **L411 EN**: Defines macro `_vel_vldz_vvvl` for conditional compilation, shorthand, or API generation.
  **L411 CN**: 定义宏 `_vel_vldz_vvvl`，用于条件编译、简写或 API 生成。
- **L412 EN**: Defines macro `_vel_vldz_vvmvl` for conditional compilation, shorthand, or API generation.
  **L412 CN**: 定义宏 `_vel_vldz_vvmvl`，用于条件编译、简写或 API 生成。
- **L413 EN**: Defines macro `_vel_pvldzlo_vvl` for conditional compilation, shorthand, or API generation.
  **L413 CN**: 定义宏 `_vel_pvldzlo_vvl`，用于条件编译、简写或 API 生成。
- **L414 EN**: Defines macro `_vel_pvldzlo_vvvl` for conditional compilation, shorthand, or API generation.
  **L414 CN**: 定义宏 `_vel_pvldzlo_vvvl`，用于条件编译、简写或 API 生成。
- **L415 EN**: Defines macro `_vel_pvldzlo_vvmvl` for conditional compilation, shorthand, or API generation.
  **L415 CN**: 定义宏 `_vel_pvldzlo_vvmvl`，用于条件编译、简写或 API 生成。
- **L416 EN**: Defines macro `_vel_pvldzup_vvl` for conditional compilation, shorthand, or API generation.
  **L416 CN**: 定义宏 `_vel_pvldzup_vvl`，用于条件编译、简写或 API 生成。
- **L417 EN**: Defines macro `_vel_pvldzup_vvvl` for conditional compilation, shorthand, or API generation.
  **L417 CN**: 定义宏 `_vel_pvldzup_vvvl`，用于条件编译、简写或 API 生成。
- **L418 EN**: Defines macro `_vel_pvldzup_vvmvl` for conditional compilation, shorthand, or API generation.
  **L418 CN**: 定义宏 `_vel_pvldzup_vvmvl`，用于条件编译、简写或 API 生成。
- **L419 EN**: Defines macro `_vel_pvldz_vvl` for conditional compilation, shorthand, or API generation.
  **L419 CN**: 定义宏 `_vel_pvldz_vvl`，用于条件编译、简写或 API 生成。
- **L420 EN**: Defines macro `_vel_pvldz_vvvl` for conditional compilation, shorthand, or API generation.
  **L420 CN**: 定义宏 `_vel_pvldz_vvvl`，用于条件编译、简写或 API 生成。
- **L421 EN**: Defines macro `_vel_pvldz_vvMvl` for conditional compilation, shorthand, or API generation.
  **L421 CN**: 定义宏 `_vel_pvldz_vvMvl`，用于条件编译、简写或 API 生成。
- **L422 EN**: Defines macro `_vel_vpcnt_vvl` for conditional compilation, shorthand, or API generation.
  **L422 CN**: 定义宏 `_vel_vpcnt_vvl`，用于条件编译、简写或 API 生成。
- **L423 EN**: Defines macro `_vel_vpcnt_vvvl` for conditional compilation, shorthand, or API generation.
  **L423 CN**: 定义宏 `_vel_vpcnt_vvvl`，用于条件编译、简写或 API 生成。
- **L424 EN**: Defines macro `_vel_vpcnt_vvmvl` for conditional compilation, shorthand, or API generation.
  **L424 CN**: 定义宏 `_vel_vpcnt_vvmvl`，用于条件编译、简写或 API 生成。
- **L425 EN**: Defines macro `_vel_pvpcntlo_vvl` for conditional compilation, shorthand, or API generation.
  **L425 CN**: 定义宏 `_vel_pvpcntlo_vvl`，用于条件编译、简写或 API 生成。
- **L426 EN**: Defines macro `_vel_pvpcntlo_vvvl` for conditional compilation, shorthand, or API generation.
  **L426 CN**: 定义宏 `_vel_pvpcntlo_vvvl`，用于条件编译、简写或 API 生成。
- **L427 EN**: Defines macro `_vel_pvpcntlo_vvmvl` for conditional compilation, shorthand, or API generation.
  **L427 CN**: 定义宏 `_vel_pvpcntlo_vvmvl`，用于条件编译、简写或 API 生成。
- **L428 EN**: Defines macro `_vel_pvpcntup_vvl` for conditional compilation, shorthand, or API generation.
  **L428 CN**: 定义宏 `_vel_pvpcntup_vvl`，用于条件编译、简写或 API 生成。
- **L429 EN**: Defines macro `_vel_pvpcntup_vvvl` for conditional compilation, shorthand, or API generation.
  **L429 CN**: 定义宏 `_vel_pvpcntup_vvvl`，用于条件编译、简写或 API 生成。
- **L430 EN**: Defines macro `_vel_pvpcntup_vvmvl` for conditional compilation, shorthand, or API generation.
  **L430 CN**: 定义宏 `_vel_pvpcntup_vvmvl`，用于条件编译、简写或 API 生成。
- **L431 EN**: Defines macro `_vel_pvpcnt_vvl` for conditional compilation, shorthand, or API generation.
  **L431 CN**: 定义宏 `_vel_pvpcnt_vvl`，用于条件编译、简写或 API 生成。
- **L432 EN**: Defines macro `_vel_pvpcnt_vvvl` for conditional compilation, shorthand, or API generation.
  **L432 CN**: 定义宏 `_vel_pvpcnt_vvvl`，用于条件编译、简写或 API 生成。

### Lines 433-456

````c
#define _vel_pvpcnt_vvMvl __builtin_ve_vl_pvpcnt_vvMvl
#define _vel_vbrv_vvl __builtin_ve_vl_vbrv_vvl
#define _vel_vbrv_vvvl __builtin_ve_vl_vbrv_vvvl
#define _vel_vbrv_vvmvl __builtin_ve_vl_vbrv_vvmvl
#define _vel_pvbrvlo_vvl __builtin_ve_vl_pvbrvlo_vvl
#define _vel_pvbrvlo_vvvl __builtin_ve_vl_pvbrvlo_vvvl
#define _vel_pvbrvlo_vvmvl __builtin_ve_vl_pvbrvlo_vvmvl
#define _vel_pvbrvup_vvl __builtin_ve_vl_pvbrvup_vvl
#define _vel_pvbrvup_vvvl __builtin_ve_vl_pvbrvup_vvvl
#define _vel_pvbrvup_vvmvl __builtin_ve_vl_pvbrvup_vvmvl
#define _vel_pvbrv_vvl __builtin_ve_vl_pvbrv_vvl
#define _vel_pvbrv_vvvl __builtin_ve_vl_pvbrv_vvvl
#define _vel_pvbrv_vvMvl __builtin_ve_vl_pvbrv_vvMvl
#define _vel_vseq_vl __builtin_ve_vl_vseq_vl
#define _vel_vseq_vvl __builtin_ve_vl_vseq_vvl
#define _vel_pvseqlo_vl __builtin_ve_vl_pvseqlo_vl
#define _vel_pvseqlo_vvl __builtin_ve_vl_pvseqlo_vvl
#define _vel_pvsequp_vl __builtin_ve_vl_pvsequp_vl
#define _vel_pvsequp_vvl __builtin_ve_vl_pvsequp_vvl
#define _vel_pvseq_vl __builtin_ve_vl_pvseq_vl
#define _vel_pvseq_vvl __builtin_ve_vl_pvseq_vvl
#define _vel_vsll_vvvl __builtin_ve_vl_vsll_vvvl
#define _vel_vsll_vvvvl __builtin_ve_vl_vsll_vvvvl
#define _vel_vsll_vvsl __builtin_ve_vl_vsll_vvsl
````
- **L433 EN**: Defines macro `_vel_pvpcnt_vvMvl` for conditional compilation, shorthand, or API generation.
  **L433 CN**: 定义宏 `_vel_pvpcnt_vvMvl`，用于条件编译、简写或 API 生成。
- **L434 EN**: Defines macro `_vel_vbrv_vvl` for conditional compilation, shorthand, or API generation.
  **L434 CN**: 定义宏 `_vel_vbrv_vvl`，用于条件编译、简写或 API 生成。
- **L435 EN**: Defines macro `_vel_vbrv_vvvl` for conditional compilation, shorthand, or API generation.
  **L435 CN**: 定义宏 `_vel_vbrv_vvvl`，用于条件编译、简写或 API 生成。
- **L436 EN**: Defines macro `_vel_vbrv_vvmvl` for conditional compilation, shorthand, or API generation.
  **L436 CN**: 定义宏 `_vel_vbrv_vvmvl`，用于条件编译、简写或 API 生成。
- **L437 EN**: Defines macro `_vel_pvbrvlo_vvl` for conditional compilation, shorthand, or API generation.
  **L437 CN**: 定义宏 `_vel_pvbrvlo_vvl`，用于条件编译、简写或 API 生成。
- **L438 EN**: Defines macro `_vel_pvbrvlo_vvvl` for conditional compilation, shorthand, or API generation.
  **L438 CN**: 定义宏 `_vel_pvbrvlo_vvvl`，用于条件编译、简写或 API 生成。
- **L439 EN**: Defines macro `_vel_pvbrvlo_vvmvl` for conditional compilation, shorthand, or API generation.
  **L439 CN**: 定义宏 `_vel_pvbrvlo_vvmvl`，用于条件编译、简写或 API 生成。
- **L440 EN**: Defines macro `_vel_pvbrvup_vvl` for conditional compilation, shorthand, or API generation.
  **L440 CN**: 定义宏 `_vel_pvbrvup_vvl`，用于条件编译、简写或 API 生成。
- **L441 EN**: Defines macro `_vel_pvbrvup_vvvl` for conditional compilation, shorthand, or API generation.
  **L441 CN**: 定义宏 `_vel_pvbrvup_vvvl`，用于条件编译、简写或 API 生成。
- **L442 EN**: Defines macro `_vel_pvbrvup_vvmvl` for conditional compilation, shorthand, or API generation.
  **L442 CN**: 定义宏 `_vel_pvbrvup_vvmvl`，用于条件编译、简写或 API 生成。
- **L443 EN**: Defines macro `_vel_pvbrv_vvl` for conditional compilation, shorthand, or API generation.
  **L443 CN**: 定义宏 `_vel_pvbrv_vvl`，用于条件编译、简写或 API 生成。
- **L444 EN**: Defines macro `_vel_pvbrv_vvvl` for conditional compilation, shorthand, or API generation.
  **L444 CN**: 定义宏 `_vel_pvbrv_vvvl`，用于条件编译、简写或 API 生成。
- **L445 EN**: Defines macro `_vel_pvbrv_vvMvl` for conditional compilation, shorthand, or API generation.
  **L445 CN**: 定义宏 `_vel_pvbrv_vvMvl`，用于条件编译、简写或 API 生成。
- **L446 EN**: Defines macro `_vel_vseq_vl` for conditional compilation, shorthand, or API generation.
  **L446 CN**: 定义宏 `_vel_vseq_vl`，用于条件编译、简写或 API 生成。
- **L447 EN**: Defines macro `_vel_vseq_vvl` for conditional compilation, shorthand, or API generation.
  **L447 CN**: 定义宏 `_vel_vseq_vvl`，用于条件编译、简写或 API 生成。
- **L448 EN**: Defines macro `_vel_pvseqlo_vl` for conditional compilation, shorthand, or API generation.
  **L448 CN**: 定义宏 `_vel_pvseqlo_vl`，用于条件编译、简写或 API 生成。
- **L449 EN**: Defines macro `_vel_pvseqlo_vvl` for conditional compilation, shorthand, or API generation.
  **L449 CN**: 定义宏 `_vel_pvseqlo_vvl`，用于条件编译、简写或 API 生成。
- **L450 EN**: Defines macro `_vel_pvsequp_vl` for conditional compilation, shorthand, or API generation.
  **L450 CN**: 定义宏 `_vel_pvsequp_vl`，用于条件编译、简写或 API 生成。
- **L451 EN**: Defines macro `_vel_pvsequp_vvl` for conditional compilation, shorthand, or API generation.
  **L451 CN**: 定义宏 `_vel_pvsequp_vvl`，用于条件编译、简写或 API 生成。
- **L452 EN**: Defines macro `_vel_pvseq_vl` for conditional compilation, shorthand, or API generation.
  **L452 CN**: 定义宏 `_vel_pvseq_vl`，用于条件编译、简写或 API 生成。
- **L453 EN**: Defines macro `_vel_pvseq_vvl` for conditional compilation, shorthand, or API generation.
  **L453 CN**: 定义宏 `_vel_pvseq_vvl`，用于条件编译、简写或 API 生成。
- **L454 EN**: Defines macro `_vel_vsll_vvvl` for conditional compilation, shorthand, or API generation.
  **L454 CN**: 定义宏 `_vel_vsll_vvvl`，用于条件编译、简写或 API 生成。
- **L455 EN**: Defines macro `_vel_vsll_vvvvl` for conditional compilation, shorthand, or API generation.
  **L455 CN**: 定义宏 `_vel_vsll_vvvvl`，用于条件编译、简写或 API 生成。
- **L456 EN**: Defines macro `_vel_vsll_vvsl` for conditional compilation, shorthand, or API generation.
  **L456 CN**: 定义宏 `_vel_vsll_vvsl`，用于条件编译、简写或 API 生成。

### Lines 457-480

````c
#define _vel_vsll_vvsvl __builtin_ve_vl_vsll_vvsvl
#define _vel_vsll_vvvmvl __builtin_ve_vl_vsll_vvvmvl
#define _vel_vsll_vvsmvl __builtin_ve_vl_vsll_vvsmvl
#define _vel_pvsll_vvvl __builtin_ve_vl_pvsll_vvvl
#define _vel_pvsll_vvvvl __builtin_ve_vl_pvsll_vvvvl
#define _vel_pvsll_vvsl __builtin_ve_vl_pvsll_vvsl
#define _vel_pvsll_vvsvl __builtin_ve_vl_pvsll_vvsvl
#define _vel_pvsll_vvvMvl __builtin_ve_vl_pvsll_vvvMvl
#define _vel_pvsll_vvsMvl __builtin_ve_vl_pvsll_vvsMvl
#define _vel_vsrl_vvvl __builtin_ve_vl_vsrl_vvvl
#define _vel_vsrl_vvvvl __builtin_ve_vl_vsrl_vvvvl
#define _vel_vsrl_vvsl __builtin_ve_vl_vsrl_vvsl
#define _vel_vsrl_vvsvl __builtin_ve_vl_vsrl_vvsvl
#define _vel_vsrl_vvvmvl __builtin_ve_vl_vsrl_vvvmvl
#define _vel_vsrl_vvsmvl __builtin_ve_vl_vsrl_vvsmvl
#define _vel_pvsrl_vvvl __builtin_ve_vl_pvsrl_vvvl
#define _vel_pvsrl_vvvvl __builtin_ve_vl_pvsrl_vvvvl
#define _vel_pvsrl_vvsl __builtin_ve_vl_pvsrl_vvsl
#define _vel_pvsrl_vvsvl __builtin_ve_vl_pvsrl_vvsvl
#define _vel_pvsrl_vvvMvl __builtin_ve_vl_pvsrl_vvvMvl
#define _vel_pvsrl_vvsMvl __builtin_ve_vl_pvsrl_vvsMvl
#define _vel_vslawsx_vvvl __builtin_ve_vl_vslawsx_vvvl
#define _vel_vslawsx_vvvvl __builtin_ve_vl_vslawsx_vvvvl
#define _vel_vslawsx_vvsl __builtin_ve_vl_vslawsx_vvsl
````
- **L457 EN**: Defines macro `_vel_vsll_vvsvl` for conditional compilation, shorthand, or API generation.
  **L457 CN**: 定义宏 `_vel_vsll_vvsvl`，用于条件编译、简写或 API 生成。
- **L458 EN**: Defines macro `_vel_vsll_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L458 CN**: 定义宏 `_vel_vsll_vvvmvl`，用于条件编译、简写或 API 生成。
- **L459 EN**: Defines macro `_vel_vsll_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L459 CN**: 定义宏 `_vel_vsll_vvsmvl`，用于条件编译、简写或 API 生成。
- **L460 EN**: Defines macro `_vel_pvsll_vvvl` for conditional compilation, shorthand, or API generation.
  **L460 CN**: 定义宏 `_vel_pvsll_vvvl`，用于条件编译、简写或 API 生成。
- **L461 EN**: Defines macro `_vel_pvsll_vvvvl` for conditional compilation, shorthand, or API generation.
  **L461 CN**: 定义宏 `_vel_pvsll_vvvvl`，用于条件编译、简写或 API 生成。
- **L462 EN**: Defines macro `_vel_pvsll_vvsl` for conditional compilation, shorthand, or API generation.
  **L462 CN**: 定义宏 `_vel_pvsll_vvsl`，用于条件编译、简写或 API 生成。
- **L463 EN**: Defines macro `_vel_pvsll_vvsvl` for conditional compilation, shorthand, or API generation.
  **L463 CN**: 定义宏 `_vel_pvsll_vvsvl`，用于条件编译、简写或 API 生成。
- **L464 EN**: Defines macro `_vel_pvsll_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L464 CN**: 定义宏 `_vel_pvsll_vvvMvl`，用于条件编译、简写或 API 生成。
- **L465 EN**: Defines macro `_vel_pvsll_vvsMvl` for conditional compilation, shorthand, or API generation.
  **L465 CN**: 定义宏 `_vel_pvsll_vvsMvl`，用于条件编译、简写或 API 生成。
- **L466 EN**: Defines macro `_vel_vsrl_vvvl` for conditional compilation, shorthand, or API generation.
  **L466 CN**: 定义宏 `_vel_vsrl_vvvl`，用于条件编译、简写或 API 生成。
- **L467 EN**: Defines macro `_vel_vsrl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L467 CN**: 定义宏 `_vel_vsrl_vvvvl`，用于条件编译、简写或 API 生成。
- **L468 EN**: Defines macro `_vel_vsrl_vvsl` for conditional compilation, shorthand, or API generation.
  **L468 CN**: 定义宏 `_vel_vsrl_vvsl`，用于条件编译、简写或 API 生成。
- **L469 EN**: Defines macro `_vel_vsrl_vvsvl` for conditional compilation, shorthand, or API generation.
  **L469 CN**: 定义宏 `_vel_vsrl_vvsvl`，用于条件编译、简写或 API 生成。
- **L470 EN**: Defines macro `_vel_vsrl_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L470 CN**: 定义宏 `_vel_vsrl_vvvmvl`，用于条件编译、简写或 API 生成。
- **L471 EN**: Defines macro `_vel_vsrl_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L471 CN**: 定义宏 `_vel_vsrl_vvsmvl`，用于条件编译、简写或 API 生成。
- **L472 EN**: Defines macro `_vel_pvsrl_vvvl` for conditional compilation, shorthand, or API generation.
  **L472 CN**: 定义宏 `_vel_pvsrl_vvvl`，用于条件编译、简写或 API 生成。
- **L473 EN**: Defines macro `_vel_pvsrl_vvvvl` for conditional compilation, shorthand, or API generation.
  **L473 CN**: 定义宏 `_vel_pvsrl_vvvvl`，用于条件编译、简写或 API 生成。
- **L474 EN**: Defines macro `_vel_pvsrl_vvsl` for conditional compilation, shorthand, or API generation.
  **L474 CN**: 定义宏 `_vel_pvsrl_vvsl`，用于条件编译、简写或 API 生成。
- **L475 EN**: Defines macro `_vel_pvsrl_vvsvl` for conditional compilation, shorthand, or API generation.
  **L475 CN**: 定义宏 `_vel_pvsrl_vvsvl`，用于条件编译、简写或 API 生成。
- **L476 EN**: Defines macro `_vel_pvsrl_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L476 CN**: 定义宏 `_vel_pvsrl_vvvMvl`，用于条件编译、简写或 API 生成。
- **L477 EN**: Defines macro `_vel_pvsrl_vvsMvl` for conditional compilation, shorthand, or API generation.
  **L477 CN**: 定义宏 `_vel_pvsrl_vvsMvl`，用于条件编译、简写或 API 生成。
- **L478 EN**: Defines macro `_vel_vslawsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L478 CN**: 定义宏 `_vel_vslawsx_vvvl`，用于条件编译、简写或 API 生成。
- **L479 EN**: Defines macro `_vel_vslawsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L479 CN**: 定义宏 `_vel_vslawsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L480 EN**: Defines macro `_vel_vslawsx_vvsl` for conditional compilation, shorthand, or API generation.
  **L480 CN**: 定义宏 `_vel_vslawsx_vvsl`，用于条件编译、简写或 API 生成。

### Lines 481-504

````c
#define _vel_vslawsx_vvsvl __builtin_ve_vl_vslawsx_vvsvl
#define _vel_vslawsx_vvvmvl __builtin_ve_vl_vslawsx_vvvmvl
#define _vel_vslawsx_vvsmvl __builtin_ve_vl_vslawsx_vvsmvl
#define _vel_vslawzx_vvvl __builtin_ve_vl_vslawzx_vvvl
#define _vel_vslawzx_vvvvl __builtin_ve_vl_vslawzx_vvvvl
#define _vel_vslawzx_vvsl __builtin_ve_vl_vslawzx_vvsl
#define _vel_vslawzx_vvsvl __builtin_ve_vl_vslawzx_vvsvl
#define _vel_vslawzx_vvvmvl __builtin_ve_vl_vslawzx_vvvmvl
#define _vel_vslawzx_vvsmvl __builtin_ve_vl_vslawzx_vvsmvl
#define _vel_pvsla_vvvl __builtin_ve_vl_pvsla_vvvl
#define _vel_pvsla_vvvvl __builtin_ve_vl_pvsla_vvvvl
#define _vel_pvsla_vvsl __builtin_ve_vl_pvsla_vvsl
#define _vel_pvsla_vvsvl __builtin_ve_vl_pvsla_vvsvl
#define _vel_pvsla_vvvMvl __builtin_ve_vl_pvsla_vvvMvl
#define _vel_pvsla_vvsMvl __builtin_ve_vl_pvsla_vvsMvl
#define _vel_vslal_vvvl __builtin_ve_vl_vslal_vvvl
#define _vel_vslal_vvvvl __builtin_ve_vl_vslal_vvvvl
#define _vel_vslal_vvsl __builtin_ve_vl_vslal_vvsl
#define _vel_vslal_vvsvl __builtin_ve_vl_vslal_vvsvl
#define _vel_vslal_vvvmvl __builtin_ve_vl_vslal_vvvmvl
#define _vel_vslal_vvsmvl __builtin_ve_vl_vslal_vvsmvl
#define _vel_vsrawsx_vvvl __builtin_ve_vl_vsrawsx_vvvl
#define _vel_vsrawsx_vvvvl __builtin_ve_vl_vsrawsx_vvvvl
#define _vel_vsrawsx_vvsl __builtin_ve_vl_vsrawsx_vvsl
````
- **L481 EN**: Defines macro `_vel_vslawsx_vvsvl` for conditional compilation, shorthand, or API generation.
  **L481 CN**: 定义宏 `_vel_vslawsx_vvsvl`，用于条件编译、简写或 API 生成。
- **L482 EN**: Defines macro `_vel_vslawsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L482 CN**: 定义宏 `_vel_vslawsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L483 EN**: Defines macro `_vel_vslawsx_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L483 CN**: 定义宏 `_vel_vslawsx_vvsmvl`，用于条件编译、简写或 API 生成。
- **L484 EN**: Defines macro `_vel_vslawzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L484 CN**: 定义宏 `_vel_vslawzx_vvvl`，用于条件编译、简写或 API 生成。
- **L485 EN**: Defines macro `_vel_vslawzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L485 CN**: 定义宏 `_vel_vslawzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L486 EN**: Defines macro `_vel_vslawzx_vvsl` for conditional compilation, shorthand, or API generation.
  **L486 CN**: 定义宏 `_vel_vslawzx_vvsl`，用于条件编译、简写或 API 生成。
- **L487 EN**: Defines macro `_vel_vslawzx_vvsvl` for conditional compilation, shorthand, or API generation.
  **L487 CN**: 定义宏 `_vel_vslawzx_vvsvl`，用于条件编译、简写或 API 生成。
- **L488 EN**: Defines macro `_vel_vslawzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L488 CN**: 定义宏 `_vel_vslawzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L489 EN**: Defines macro `_vel_vslawzx_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L489 CN**: 定义宏 `_vel_vslawzx_vvsmvl`，用于条件编译、简写或 API 生成。
- **L490 EN**: Defines macro `_vel_pvsla_vvvl` for conditional compilation, shorthand, or API generation.
  **L490 CN**: 定义宏 `_vel_pvsla_vvvl`，用于条件编译、简写或 API 生成。
- **L491 EN**: Defines macro `_vel_pvsla_vvvvl` for conditional compilation, shorthand, or API generation.
  **L491 CN**: 定义宏 `_vel_pvsla_vvvvl`，用于条件编译、简写或 API 生成。
- **L492 EN**: Defines macro `_vel_pvsla_vvsl` for conditional compilation, shorthand, or API generation.
  **L492 CN**: 定义宏 `_vel_pvsla_vvsl`，用于条件编译、简写或 API 生成。
- **L493 EN**: Defines macro `_vel_pvsla_vvsvl` for conditional compilation, shorthand, or API generation.
  **L493 CN**: 定义宏 `_vel_pvsla_vvsvl`，用于条件编译、简写或 API 生成。
- **L494 EN**: Defines macro `_vel_pvsla_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L494 CN**: 定义宏 `_vel_pvsla_vvvMvl`，用于条件编译、简写或 API 生成。
- **L495 EN**: Defines macro `_vel_pvsla_vvsMvl` for conditional compilation, shorthand, or API generation.
  **L495 CN**: 定义宏 `_vel_pvsla_vvsMvl`，用于条件编译、简写或 API 生成。
- **L496 EN**: Defines macro `_vel_vslal_vvvl` for conditional compilation, shorthand, or API generation.
  **L496 CN**: 定义宏 `_vel_vslal_vvvl`，用于条件编译、简写或 API 生成。
- **L497 EN**: Defines macro `_vel_vslal_vvvvl` for conditional compilation, shorthand, or API generation.
  **L497 CN**: 定义宏 `_vel_vslal_vvvvl`，用于条件编译、简写或 API 生成。
- **L498 EN**: Defines macro `_vel_vslal_vvsl` for conditional compilation, shorthand, or API generation.
  **L498 CN**: 定义宏 `_vel_vslal_vvsl`，用于条件编译、简写或 API 生成。
- **L499 EN**: Defines macro `_vel_vslal_vvsvl` for conditional compilation, shorthand, or API generation.
  **L499 CN**: 定义宏 `_vel_vslal_vvsvl`，用于条件编译、简写或 API 生成。
- **L500 EN**: Defines macro `_vel_vslal_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L500 CN**: 定义宏 `_vel_vslal_vvvmvl`，用于条件编译、简写或 API 生成。
- **L501 EN**: Defines macro `_vel_vslal_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L501 CN**: 定义宏 `_vel_vslal_vvsmvl`，用于条件编译、简写或 API 生成。
- **L502 EN**: Defines macro `_vel_vsrawsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L502 CN**: 定义宏 `_vel_vsrawsx_vvvl`，用于条件编译、简写或 API 生成。
- **L503 EN**: Defines macro `_vel_vsrawsx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L503 CN**: 定义宏 `_vel_vsrawsx_vvvvl`，用于条件编译、简写或 API 生成。
- **L504 EN**: Defines macro `_vel_vsrawsx_vvsl` for conditional compilation, shorthand, or API generation.
  **L504 CN**: 定义宏 `_vel_vsrawsx_vvsl`，用于条件编译、简写或 API 生成。

### Lines 505-528

````c
#define _vel_vsrawsx_vvsvl __builtin_ve_vl_vsrawsx_vvsvl
#define _vel_vsrawsx_vvvmvl __builtin_ve_vl_vsrawsx_vvvmvl
#define _vel_vsrawsx_vvsmvl __builtin_ve_vl_vsrawsx_vvsmvl
#define _vel_vsrawzx_vvvl __builtin_ve_vl_vsrawzx_vvvl
#define _vel_vsrawzx_vvvvl __builtin_ve_vl_vsrawzx_vvvvl
#define _vel_vsrawzx_vvsl __builtin_ve_vl_vsrawzx_vvsl
#define _vel_vsrawzx_vvsvl __builtin_ve_vl_vsrawzx_vvsvl
#define _vel_vsrawzx_vvvmvl __builtin_ve_vl_vsrawzx_vvvmvl
#define _vel_vsrawzx_vvsmvl __builtin_ve_vl_vsrawzx_vvsmvl
#define _vel_pvsra_vvvl __builtin_ve_vl_pvsra_vvvl
#define _vel_pvsra_vvvvl __builtin_ve_vl_pvsra_vvvvl
#define _vel_pvsra_vvsl __builtin_ve_vl_pvsra_vvsl
#define _vel_pvsra_vvsvl __builtin_ve_vl_pvsra_vvsvl
#define _vel_pvsra_vvvMvl __builtin_ve_vl_pvsra_vvvMvl
#define _vel_pvsra_vvsMvl __builtin_ve_vl_pvsra_vvsMvl
#define _vel_vsral_vvvl __builtin_ve_vl_vsral_vvvl
#define _vel_vsral_vvvvl __builtin_ve_vl_vsral_vvvvl
#define _vel_vsral_vvsl __builtin_ve_vl_vsral_vvsl
#define _vel_vsral_vvsvl __builtin_ve_vl_vsral_vvsvl
#define _vel_vsral_vvvmvl __builtin_ve_vl_vsral_vvvmvl
#define _vel_vsral_vvsmvl __builtin_ve_vl_vsral_vvsmvl
#define _vel_vsfa_vvssl __builtin_ve_vl_vsfa_vvssl
#define _vel_vsfa_vvssvl __builtin_ve_vl_vsfa_vvssvl
#define _vel_vsfa_vvssmvl __builtin_ve_vl_vsfa_vvssmvl
````
- **L505 EN**: Defines macro `_vel_vsrawsx_vvsvl` for conditional compilation, shorthand, or API generation.
  **L505 CN**: 定义宏 `_vel_vsrawsx_vvsvl`，用于条件编译、简写或 API 生成。
- **L506 EN**: Defines macro `_vel_vsrawsx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L506 CN**: 定义宏 `_vel_vsrawsx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L507 EN**: Defines macro `_vel_vsrawsx_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L507 CN**: 定义宏 `_vel_vsrawsx_vvsmvl`，用于条件编译、简写或 API 生成。
- **L508 EN**: Defines macro `_vel_vsrawzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L508 CN**: 定义宏 `_vel_vsrawzx_vvvl`，用于条件编译、简写或 API 生成。
- **L509 EN**: Defines macro `_vel_vsrawzx_vvvvl` for conditional compilation, shorthand, or API generation.
  **L509 CN**: 定义宏 `_vel_vsrawzx_vvvvl`，用于条件编译、简写或 API 生成。
- **L510 EN**: Defines macro `_vel_vsrawzx_vvsl` for conditional compilation, shorthand, or API generation.
  **L510 CN**: 定义宏 `_vel_vsrawzx_vvsl`，用于条件编译、简写或 API 生成。
- **L511 EN**: Defines macro `_vel_vsrawzx_vvsvl` for conditional compilation, shorthand, or API generation.
  **L511 CN**: 定义宏 `_vel_vsrawzx_vvsvl`，用于条件编译、简写或 API 生成。
- **L512 EN**: Defines macro `_vel_vsrawzx_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L512 CN**: 定义宏 `_vel_vsrawzx_vvvmvl`，用于条件编译、简写或 API 生成。
- **L513 EN**: Defines macro `_vel_vsrawzx_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L513 CN**: 定义宏 `_vel_vsrawzx_vvsmvl`，用于条件编译、简写或 API 生成。
- **L514 EN**: Defines macro `_vel_pvsra_vvvl` for conditional compilation, shorthand, or API generation.
  **L514 CN**: 定义宏 `_vel_pvsra_vvvl`，用于条件编译、简写或 API 生成。
- **L515 EN**: Defines macro `_vel_pvsra_vvvvl` for conditional compilation, shorthand, or API generation.
  **L515 CN**: 定义宏 `_vel_pvsra_vvvvl`，用于条件编译、简写或 API 生成。
- **L516 EN**: Defines macro `_vel_pvsra_vvsl` for conditional compilation, shorthand, or API generation.
  **L516 CN**: 定义宏 `_vel_pvsra_vvsl`，用于条件编译、简写或 API 生成。
- **L517 EN**: Defines macro `_vel_pvsra_vvsvl` for conditional compilation, shorthand, or API generation.
  **L517 CN**: 定义宏 `_vel_pvsra_vvsvl`，用于条件编译、简写或 API 生成。
- **L518 EN**: Defines macro `_vel_pvsra_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L518 CN**: 定义宏 `_vel_pvsra_vvvMvl`，用于条件编译、简写或 API 生成。
- **L519 EN**: Defines macro `_vel_pvsra_vvsMvl` for conditional compilation, shorthand, or API generation.
  **L519 CN**: 定义宏 `_vel_pvsra_vvsMvl`，用于条件编译、简写或 API 生成。
- **L520 EN**: Defines macro `_vel_vsral_vvvl` for conditional compilation, shorthand, or API generation.
  **L520 CN**: 定义宏 `_vel_vsral_vvvl`，用于条件编译、简写或 API 生成。
- **L521 EN**: Defines macro `_vel_vsral_vvvvl` for conditional compilation, shorthand, or API generation.
  **L521 CN**: 定义宏 `_vel_vsral_vvvvl`，用于条件编译、简写或 API 生成。
- **L522 EN**: Defines macro `_vel_vsral_vvsl` for conditional compilation, shorthand, or API generation.
  **L522 CN**: 定义宏 `_vel_vsral_vvsl`，用于条件编译、简写或 API 生成。
- **L523 EN**: Defines macro `_vel_vsral_vvsvl` for conditional compilation, shorthand, or API generation.
  **L523 CN**: 定义宏 `_vel_vsral_vvsvl`，用于条件编译、简写或 API 生成。
- **L524 EN**: Defines macro `_vel_vsral_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L524 CN**: 定义宏 `_vel_vsral_vvvmvl`，用于条件编译、简写或 API 生成。
- **L525 EN**: Defines macro `_vel_vsral_vvsmvl` for conditional compilation, shorthand, or API generation.
  **L525 CN**: 定义宏 `_vel_vsral_vvsmvl`，用于条件编译、简写或 API 生成。
- **L526 EN**: Defines macro `_vel_vsfa_vvssl` for conditional compilation, shorthand, or API generation.
  **L526 CN**: 定义宏 `_vel_vsfa_vvssl`，用于条件编译、简写或 API 生成。
- **L527 EN**: Defines macro `_vel_vsfa_vvssvl` for conditional compilation, shorthand, or API generation.
  **L527 CN**: 定义宏 `_vel_vsfa_vvssvl`，用于条件编译、简写或 API 生成。
- **L528 EN**: Defines macro `_vel_vsfa_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L528 CN**: 定义宏 `_vel_vsfa_vvssmvl`，用于条件编译、简写或 API 生成。

### Lines 529-552

````c
#define _vel_vfaddd_vvvl __builtin_ve_vl_vfaddd_vvvl
#define _vel_vfaddd_vvvvl __builtin_ve_vl_vfaddd_vvvvl
#define _vel_vfaddd_vsvl __builtin_ve_vl_vfaddd_vsvl
#define _vel_vfaddd_vsvvl __builtin_ve_vl_vfaddd_vsvvl
#define _vel_vfaddd_vvvmvl __builtin_ve_vl_vfaddd_vvvmvl
#define _vel_vfaddd_vsvmvl __builtin_ve_vl_vfaddd_vsvmvl
#define _vel_vfadds_vvvl __builtin_ve_vl_vfadds_vvvl
#define _vel_vfadds_vvvvl __builtin_ve_vl_vfadds_vvvvl
#define _vel_vfadds_vsvl __builtin_ve_vl_vfadds_vsvl
#define _vel_vfadds_vsvvl __builtin_ve_vl_vfadds_vsvvl
#define _vel_vfadds_vvvmvl __builtin_ve_vl_vfadds_vvvmvl
#define _vel_vfadds_vsvmvl __builtin_ve_vl_vfadds_vsvmvl
#define _vel_pvfadd_vvvl __builtin_ve_vl_pvfadd_vvvl
#define _vel_pvfadd_vvvvl __builtin_ve_vl_pvfadd_vvvvl
#define _vel_pvfadd_vsvl __builtin_ve_vl_pvfadd_vsvl
#define _vel_pvfadd_vsvvl __builtin_ve_vl_pvfadd_vsvvl
#define _vel_pvfadd_vvvMvl __builtin_ve_vl_pvfadd_vvvMvl
#define _vel_pvfadd_vsvMvl __builtin_ve_vl_pvfadd_vsvMvl
#define _vel_vfsubd_vvvl __builtin_ve_vl_vfsubd_vvvl
#define _vel_vfsubd_vvvvl __builtin_ve_vl_vfsubd_vvvvl
#define _vel_vfsubd_vsvl __builtin_ve_vl_vfsubd_vsvl
#define _vel_vfsubd_vsvvl __builtin_ve_vl_vfsubd_vsvvl
#define _vel_vfsubd_vvvmvl __builtin_ve_vl_vfsubd_vvvmvl
#define _vel_vfsubd_vsvmvl __builtin_ve_vl_vfsubd_vsvmvl
````
- **L529 EN**: Defines macro `_vel_vfaddd_vvvl` for conditional compilation, shorthand, or API generation.
  **L529 CN**: 定义宏 `_vel_vfaddd_vvvl`，用于条件编译、简写或 API 生成。
- **L530 EN**: Defines macro `_vel_vfaddd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L530 CN**: 定义宏 `_vel_vfaddd_vvvvl`，用于条件编译、简写或 API 生成。
- **L531 EN**: Defines macro `_vel_vfaddd_vsvl` for conditional compilation, shorthand, or API generation.
  **L531 CN**: 定义宏 `_vel_vfaddd_vsvl`，用于条件编译、简写或 API 生成。
- **L532 EN**: Defines macro `_vel_vfaddd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L532 CN**: 定义宏 `_vel_vfaddd_vsvvl`，用于条件编译、简写或 API 生成。
- **L533 EN**: Defines macro `_vel_vfaddd_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L533 CN**: 定义宏 `_vel_vfaddd_vvvmvl`，用于条件编译、简写或 API 生成。
- **L534 EN**: Defines macro `_vel_vfaddd_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L534 CN**: 定义宏 `_vel_vfaddd_vsvmvl`，用于条件编译、简写或 API 生成。
- **L535 EN**: Defines macro `_vel_vfadds_vvvl` for conditional compilation, shorthand, or API generation.
  **L535 CN**: 定义宏 `_vel_vfadds_vvvl`，用于条件编译、简写或 API 生成。
- **L536 EN**: Defines macro `_vel_vfadds_vvvvl` for conditional compilation, shorthand, or API generation.
  **L536 CN**: 定义宏 `_vel_vfadds_vvvvl`，用于条件编译、简写或 API 生成。
- **L537 EN**: Defines macro `_vel_vfadds_vsvl` for conditional compilation, shorthand, or API generation.
  **L537 CN**: 定义宏 `_vel_vfadds_vsvl`，用于条件编译、简写或 API 生成。
- **L538 EN**: Defines macro `_vel_vfadds_vsvvl` for conditional compilation, shorthand, or API generation.
  **L538 CN**: 定义宏 `_vel_vfadds_vsvvl`，用于条件编译、简写或 API 生成。
- **L539 EN**: Defines macro `_vel_vfadds_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L539 CN**: 定义宏 `_vel_vfadds_vvvmvl`，用于条件编译、简写或 API 生成。
- **L540 EN**: Defines macro `_vel_vfadds_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L540 CN**: 定义宏 `_vel_vfadds_vsvmvl`，用于条件编译、简写或 API 生成。
- **L541 EN**: Defines macro `_vel_pvfadd_vvvl` for conditional compilation, shorthand, or API generation.
  **L541 CN**: 定义宏 `_vel_pvfadd_vvvl`，用于条件编译、简写或 API 生成。
- **L542 EN**: Defines macro `_vel_pvfadd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L542 CN**: 定义宏 `_vel_pvfadd_vvvvl`，用于条件编译、简写或 API 生成。
- **L543 EN**: Defines macro `_vel_pvfadd_vsvl` for conditional compilation, shorthand, or API generation.
  **L543 CN**: 定义宏 `_vel_pvfadd_vsvl`，用于条件编译、简写或 API 生成。
- **L544 EN**: Defines macro `_vel_pvfadd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L544 CN**: 定义宏 `_vel_pvfadd_vsvvl`，用于条件编译、简写或 API 生成。
- **L545 EN**: Defines macro `_vel_pvfadd_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L545 CN**: 定义宏 `_vel_pvfadd_vvvMvl`，用于条件编译、简写或 API 生成。
- **L546 EN**: Defines macro `_vel_pvfadd_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L546 CN**: 定义宏 `_vel_pvfadd_vsvMvl`，用于条件编译、简写或 API 生成。
- **L547 EN**: Defines macro `_vel_vfsubd_vvvl` for conditional compilation, shorthand, or API generation.
  **L547 CN**: 定义宏 `_vel_vfsubd_vvvl`，用于条件编译、简写或 API 生成。
- **L548 EN**: Defines macro `_vel_vfsubd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L548 CN**: 定义宏 `_vel_vfsubd_vvvvl`，用于条件编译、简写或 API 生成。
- **L549 EN**: Defines macro `_vel_vfsubd_vsvl` for conditional compilation, shorthand, or API generation.
  **L549 CN**: 定义宏 `_vel_vfsubd_vsvl`，用于条件编译、简写或 API 生成。
- **L550 EN**: Defines macro `_vel_vfsubd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L550 CN**: 定义宏 `_vel_vfsubd_vsvvl`，用于条件编译、简写或 API 生成。
- **L551 EN**: Defines macro `_vel_vfsubd_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L551 CN**: 定义宏 `_vel_vfsubd_vvvmvl`，用于条件编译、简写或 API 生成。
- **L552 EN**: Defines macro `_vel_vfsubd_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L552 CN**: 定义宏 `_vel_vfsubd_vsvmvl`，用于条件编译、简写或 API 生成。

### Lines 553-576

````c
#define _vel_vfsubs_vvvl __builtin_ve_vl_vfsubs_vvvl
#define _vel_vfsubs_vvvvl __builtin_ve_vl_vfsubs_vvvvl
#define _vel_vfsubs_vsvl __builtin_ve_vl_vfsubs_vsvl
#define _vel_vfsubs_vsvvl __builtin_ve_vl_vfsubs_vsvvl
#define _vel_vfsubs_vvvmvl __builtin_ve_vl_vfsubs_vvvmvl
#define _vel_vfsubs_vsvmvl __builtin_ve_vl_vfsubs_vsvmvl
#define _vel_pvfsub_vvvl __builtin_ve_vl_pvfsub_vvvl
#define _vel_pvfsub_vvvvl __builtin_ve_vl_pvfsub_vvvvl
#define _vel_pvfsub_vsvl __builtin_ve_vl_pvfsub_vsvl
#define _vel_pvfsub_vsvvl __builtin_ve_vl_pvfsub_vsvvl
#define _vel_pvfsub_vvvMvl __builtin_ve_vl_pvfsub_vvvMvl
#define _vel_pvfsub_vsvMvl __builtin_ve_vl_pvfsub_vsvMvl
#define _vel_vfmuld_vvvl __builtin_ve_vl_vfmuld_vvvl
#define _vel_vfmuld_vvvvl __builtin_ve_vl_vfmuld_vvvvl
#define _vel_vfmuld_vsvl __builtin_ve_vl_vfmuld_vsvl
#define _vel_vfmuld_vsvvl __builtin_ve_vl_vfmuld_vsvvl
#define _vel_vfmuld_vvvmvl __builtin_ve_vl_vfmuld_vvvmvl
#define _vel_vfmuld_vsvmvl __builtin_ve_vl_vfmuld_vsvmvl
#define _vel_vfmuls_vvvl __builtin_ve_vl_vfmuls_vvvl
#define _vel_vfmuls_vvvvl __builtin_ve_vl_vfmuls_vvvvl
#define _vel_vfmuls_vsvl __builtin_ve_vl_vfmuls_vsvl
#define _vel_vfmuls_vsvvl __builtin_ve_vl_vfmuls_vsvvl
#define _vel_vfmuls_vvvmvl __builtin_ve_vl_vfmuls_vvvmvl
#define _vel_vfmuls_vsvmvl __builtin_ve_vl_vfmuls_vsvmvl
````
- **L553 EN**: Defines macro `_vel_vfsubs_vvvl` for conditional compilation, shorthand, or API generation.
  **L553 CN**: 定义宏 `_vel_vfsubs_vvvl`，用于条件编译、简写或 API 生成。
- **L554 EN**: Defines macro `_vel_vfsubs_vvvvl` for conditional compilation, shorthand, or API generation.
  **L554 CN**: 定义宏 `_vel_vfsubs_vvvvl`，用于条件编译、简写或 API 生成。
- **L555 EN**: Defines macro `_vel_vfsubs_vsvl` for conditional compilation, shorthand, or API generation.
  **L555 CN**: 定义宏 `_vel_vfsubs_vsvl`，用于条件编译、简写或 API 生成。
- **L556 EN**: Defines macro `_vel_vfsubs_vsvvl` for conditional compilation, shorthand, or API generation.
  **L556 CN**: 定义宏 `_vel_vfsubs_vsvvl`，用于条件编译、简写或 API 生成。
- **L557 EN**: Defines macro `_vel_vfsubs_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L557 CN**: 定义宏 `_vel_vfsubs_vvvmvl`，用于条件编译、简写或 API 生成。
- **L558 EN**: Defines macro `_vel_vfsubs_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L558 CN**: 定义宏 `_vel_vfsubs_vsvmvl`，用于条件编译、简写或 API 生成。
- **L559 EN**: Defines macro `_vel_pvfsub_vvvl` for conditional compilation, shorthand, or API generation.
  **L559 CN**: 定义宏 `_vel_pvfsub_vvvl`，用于条件编译、简写或 API 生成。
- **L560 EN**: Defines macro `_vel_pvfsub_vvvvl` for conditional compilation, shorthand, or API generation.
  **L560 CN**: 定义宏 `_vel_pvfsub_vvvvl`，用于条件编译、简写或 API 生成。
- **L561 EN**: Defines macro `_vel_pvfsub_vsvl` for conditional compilation, shorthand, or API generation.
  **L561 CN**: 定义宏 `_vel_pvfsub_vsvl`，用于条件编译、简写或 API 生成。
- **L562 EN**: Defines macro `_vel_pvfsub_vsvvl` for conditional compilation, shorthand, or API generation.
  **L562 CN**: 定义宏 `_vel_pvfsub_vsvvl`，用于条件编译、简写或 API 生成。
- **L563 EN**: Defines macro `_vel_pvfsub_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L563 CN**: 定义宏 `_vel_pvfsub_vvvMvl`，用于条件编译、简写或 API 生成。
- **L564 EN**: Defines macro `_vel_pvfsub_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L564 CN**: 定义宏 `_vel_pvfsub_vsvMvl`，用于条件编译、简写或 API 生成。
- **L565 EN**: Defines macro `_vel_vfmuld_vvvl` for conditional compilation, shorthand, or API generation.
  **L565 CN**: 定义宏 `_vel_vfmuld_vvvl`，用于条件编译、简写或 API 生成。
- **L566 EN**: Defines macro `_vel_vfmuld_vvvvl` for conditional compilation, shorthand, or API generation.
  **L566 CN**: 定义宏 `_vel_vfmuld_vvvvl`，用于条件编译、简写或 API 生成。
- **L567 EN**: Defines macro `_vel_vfmuld_vsvl` for conditional compilation, shorthand, or API generation.
  **L567 CN**: 定义宏 `_vel_vfmuld_vsvl`，用于条件编译、简写或 API 生成。
- **L568 EN**: Defines macro `_vel_vfmuld_vsvvl` for conditional compilation, shorthand, or API generation.
  **L568 CN**: 定义宏 `_vel_vfmuld_vsvvl`，用于条件编译、简写或 API 生成。
- **L569 EN**: Defines macro `_vel_vfmuld_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L569 CN**: 定义宏 `_vel_vfmuld_vvvmvl`，用于条件编译、简写或 API 生成。
- **L570 EN**: Defines macro `_vel_vfmuld_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L570 CN**: 定义宏 `_vel_vfmuld_vsvmvl`，用于条件编译、简写或 API 生成。
- **L571 EN**: Defines macro `_vel_vfmuls_vvvl` for conditional compilation, shorthand, or API generation.
  **L571 CN**: 定义宏 `_vel_vfmuls_vvvl`，用于条件编译、简写或 API 生成。
- **L572 EN**: Defines macro `_vel_vfmuls_vvvvl` for conditional compilation, shorthand, or API generation.
  **L572 CN**: 定义宏 `_vel_vfmuls_vvvvl`，用于条件编译、简写或 API 生成。
- **L573 EN**: Defines macro `_vel_vfmuls_vsvl` for conditional compilation, shorthand, or API generation.
  **L573 CN**: 定义宏 `_vel_vfmuls_vsvl`，用于条件编译、简写或 API 生成。
- **L574 EN**: Defines macro `_vel_vfmuls_vsvvl` for conditional compilation, shorthand, or API generation.
  **L574 CN**: 定义宏 `_vel_vfmuls_vsvvl`，用于条件编译、简写或 API 生成。
- **L575 EN**: Defines macro `_vel_vfmuls_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L575 CN**: 定义宏 `_vel_vfmuls_vvvmvl`，用于条件编译、简写或 API 生成。
- **L576 EN**: Defines macro `_vel_vfmuls_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L576 CN**: 定义宏 `_vel_vfmuls_vsvmvl`，用于条件编译、简写或 API 生成。

### Lines 577-600

````c
#define _vel_pvfmul_vvvl __builtin_ve_vl_pvfmul_vvvl
#define _vel_pvfmul_vvvvl __builtin_ve_vl_pvfmul_vvvvl
#define _vel_pvfmul_vsvl __builtin_ve_vl_pvfmul_vsvl
#define _vel_pvfmul_vsvvl __builtin_ve_vl_pvfmul_vsvvl
#define _vel_pvfmul_vvvMvl __builtin_ve_vl_pvfmul_vvvMvl
#define _vel_pvfmul_vsvMvl __builtin_ve_vl_pvfmul_vsvMvl
#define _vel_vfdivd_vvvl __builtin_ve_vl_vfdivd_vvvl
#define _vel_vfdivd_vvvvl __builtin_ve_vl_vfdivd_vvvvl
#define _vel_vfdivd_vsvl __builtin_ve_vl_vfdivd_vsvl
#define _vel_vfdivd_vsvvl __builtin_ve_vl_vfdivd_vsvvl
#define _vel_vfdivd_vvvmvl __builtin_ve_vl_vfdivd_vvvmvl
#define _vel_vfdivd_vsvmvl __builtin_ve_vl_vfdivd_vsvmvl
#define _vel_vfdivs_vvvl __builtin_ve_vl_vfdivs_vvvl
#define _vel_vfdivs_vvvvl __builtin_ve_vl_vfdivs_vvvvl
#define _vel_vfdivs_vsvl __builtin_ve_vl_vfdivs_vsvl
#define _vel_vfdivs_vsvvl __builtin_ve_vl_vfdivs_vsvvl
#define _vel_vfdivs_vvvmvl __builtin_ve_vl_vfdivs_vvvmvl
#define _vel_vfdivs_vsvmvl __builtin_ve_vl_vfdivs_vsvmvl
#define _vel_vfsqrtd_vvl __builtin_ve_vl_vfsqrtd_vvl
#define _vel_vfsqrtd_vvvl __builtin_ve_vl_vfsqrtd_vvvl
#define _vel_vfsqrts_vvl __builtin_ve_vl_vfsqrts_vvl
#define _vel_vfsqrts_vvvl __builtin_ve_vl_vfsqrts_vvvl
#define _vel_vfcmpd_vvvl __builtin_ve_vl_vfcmpd_vvvl
#define _vel_vfcmpd_vvvvl __builtin_ve_vl_vfcmpd_vvvvl
````
- **L577 EN**: Defines macro `_vel_pvfmul_vvvl` for conditional compilation, shorthand, or API generation.
  **L577 CN**: 定义宏 `_vel_pvfmul_vvvl`，用于条件编译、简写或 API 生成。
- **L578 EN**: Defines macro `_vel_pvfmul_vvvvl` for conditional compilation, shorthand, or API generation.
  **L578 CN**: 定义宏 `_vel_pvfmul_vvvvl`，用于条件编译、简写或 API 生成。
- **L579 EN**: Defines macro `_vel_pvfmul_vsvl` for conditional compilation, shorthand, or API generation.
  **L579 CN**: 定义宏 `_vel_pvfmul_vsvl`，用于条件编译、简写或 API 生成。
- **L580 EN**: Defines macro `_vel_pvfmul_vsvvl` for conditional compilation, shorthand, or API generation.
  **L580 CN**: 定义宏 `_vel_pvfmul_vsvvl`，用于条件编译、简写或 API 生成。
- **L581 EN**: Defines macro `_vel_pvfmul_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L581 CN**: 定义宏 `_vel_pvfmul_vvvMvl`，用于条件编译、简写或 API 生成。
- **L582 EN**: Defines macro `_vel_pvfmul_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L582 CN**: 定义宏 `_vel_pvfmul_vsvMvl`，用于条件编译、简写或 API 生成。
- **L583 EN**: Defines macro `_vel_vfdivd_vvvl` for conditional compilation, shorthand, or API generation.
  **L583 CN**: 定义宏 `_vel_vfdivd_vvvl`，用于条件编译、简写或 API 生成。
- **L584 EN**: Defines macro `_vel_vfdivd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L584 CN**: 定义宏 `_vel_vfdivd_vvvvl`，用于条件编译、简写或 API 生成。
- **L585 EN**: Defines macro `_vel_vfdivd_vsvl` for conditional compilation, shorthand, or API generation.
  **L585 CN**: 定义宏 `_vel_vfdivd_vsvl`，用于条件编译、简写或 API 生成。
- **L586 EN**: Defines macro `_vel_vfdivd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L586 CN**: 定义宏 `_vel_vfdivd_vsvvl`，用于条件编译、简写或 API 生成。
- **L587 EN**: Defines macro `_vel_vfdivd_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L587 CN**: 定义宏 `_vel_vfdivd_vvvmvl`，用于条件编译、简写或 API 生成。
- **L588 EN**: Defines macro `_vel_vfdivd_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L588 CN**: 定义宏 `_vel_vfdivd_vsvmvl`，用于条件编译、简写或 API 生成。
- **L589 EN**: Defines macro `_vel_vfdivs_vvvl` for conditional compilation, shorthand, or API generation.
  **L589 CN**: 定义宏 `_vel_vfdivs_vvvl`，用于条件编译、简写或 API 生成。
- **L590 EN**: Defines macro `_vel_vfdivs_vvvvl` for conditional compilation, shorthand, or API generation.
  **L590 CN**: 定义宏 `_vel_vfdivs_vvvvl`，用于条件编译、简写或 API 生成。
- **L591 EN**: Defines macro `_vel_vfdivs_vsvl` for conditional compilation, shorthand, or API generation.
  **L591 CN**: 定义宏 `_vel_vfdivs_vsvl`，用于条件编译、简写或 API 生成。
- **L592 EN**: Defines macro `_vel_vfdivs_vsvvl` for conditional compilation, shorthand, or API generation.
  **L592 CN**: 定义宏 `_vel_vfdivs_vsvvl`，用于条件编译、简写或 API 生成。
- **L593 EN**: Defines macro `_vel_vfdivs_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L593 CN**: 定义宏 `_vel_vfdivs_vvvmvl`，用于条件编译、简写或 API 生成。
- **L594 EN**: Defines macro `_vel_vfdivs_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L594 CN**: 定义宏 `_vel_vfdivs_vsvmvl`，用于条件编译、简写或 API 生成。
- **L595 EN**: Defines macro `_vel_vfsqrtd_vvl` for conditional compilation, shorthand, or API generation.
  **L595 CN**: 定义宏 `_vel_vfsqrtd_vvl`，用于条件编译、简写或 API 生成。
- **L596 EN**: Defines macro `_vel_vfsqrtd_vvvl` for conditional compilation, shorthand, or API generation.
  **L596 CN**: 定义宏 `_vel_vfsqrtd_vvvl`，用于条件编译、简写或 API 生成。
- **L597 EN**: Defines macro `_vel_vfsqrts_vvl` for conditional compilation, shorthand, or API generation.
  **L597 CN**: 定义宏 `_vel_vfsqrts_vvl`，用于条件编译、简写或 API 生成。
- **L598 EN**: Defines macro `_vel_vfsqrts_vvvl` for conditional compilation, shorthand, or API generation.
  **L598 CN**: 定义宏 `_vel_vfsqrts_vvvl`，用于条件编译、简写或 API 生成。
- **L599 EN**: Defines macro `_vel_vfcmpd_vvvl` for conditional compilation, shorthand, or API generation.
  **L599 CN**: 定义宏 `_vel_vfcmpd_vvvl`，用于条件编译、简写或 API 生成。
- **L600 EN**: Defines macro `_vel_vfcmpd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L600 CN**: 定义宏 `_vel_vfcmpd_vvvvl`，用于条件编译、简写或 API 生成。

### Lines 601-624

````c
#define _vel_vfcmpd_vsvl __builtin_ve_vl_vfcmpd_vsvl
#define _vel_vfcmpd_vsvvl __builtin_ve_vl_vfcmpd_vsvvl
#define _vel_vfcmpd_vvvmvl __builtin_ve_vl_vfcmpd_vvvmvl
#define _vel_vfcmpd_vsvmvl __builtin_ve_vl_vfcmpd_vsvmvl
#define _vel_vfcmps_vvvl __builtin_ve_vl_vfcmps_vvvl
#define _vel_vfcmps_vvvvl __builtin_ve_vl_vfcmps_vvvvl
#define _vel_vfcmps_vsvl __builtin_ve_vl_vfcmps_vsvl
#define _vel_vfcmps_vsvvl __builtin_ve_vl_vfcmps_vsvvl
#define _vel_vfcmps_vvvmvl __builtin_ve_vl_vfcmps_vvvmvl
#define _vel_vfcmps_vsvmvl __builtin_ve_vl_vfcmps_vsvmvl
#define _vel_pvfcmp_vvvl __builtin_ve_vl_pvfcmp_vvvl
#define _vel_pvfcmp_vvvvl __builtin_ve_vl_pvfcmp_vvvvl
#define _vel_pvfcmp_vsvl __builtin_ve_vl_pvfcmp_vsvl
#define _vel_pvfcmp_vsvvl __builtin_ve_vl_pvfcmp_vsvvl
#define _vel_pvfcmp_vvvMvl __builtin_ve_vl_pvfcmp_vvvMvl
#define _vel_pvfcmp_vsvMvl __builtin_ve_vl_pvfcmp_vsvMvl
#define _vel_vfmaxd_vvvl __builtin_ve_vl_vfmaxd_vvvl
#define _vel_vfmaxd_vvvvl __builtin_ve_vl_vfmaxd_vvvvl
#define _vel_vfmaxd_vsvl __builtin_ve_vl_vfmaxd_vsvl
#define _vel_vfmaxd_vsvvl __builtin_ve_vl_vfmaxd_vsvvl
#define _vel_vfmaxd_vvvmvl __builtin_ve_vl_vfmaxd_vvvmvl
#define _vel_vfmaxd_vsvmvl __builtin_ve_vl_vfmaxd_vsvmvl
#define _vel_vfmaxs_vvvl __builtin_ve_vl_vfmaxs_vvvl
#define _vel_vfmaxs_vvvvl __builtin_ve_vl_vfmaxs_vvvvl
````
- **L601 EN**: Defines macro `_vel_vfcmpd_vsvl` for conditional compilation, shorthand, or API generation.
  **L601 CN**: 定义宏 `_vel_vfcmpd_vsvl`，用于条件编译、简写或 API 生成。
- **L602 EN**: Defines macro `_vel_vfcmpd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L602 CN**: 定义宏 `_vel_vfcmpd_vsvvl`，用于条件编译、简写或 API 生成。
- **L603 EN**: Defines macro `_vel_vfcmpd_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L603 CN**: 定义宏 `_vel_vfcmpd_vvvmvl`，用于条件编译、简写或 API 生成。
- **L604 EN**: Defines macro `_vel_vfcmpd_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L604 CN**: 定义宏 `_vel_vfcmpd_vsvmvl`，用于条件编译、简写或 API 生成。
- **L605 EN**: Defines macro `_vel_vfcmps_vvvl` for conditional compilation, shorthand, or API generation.
  **L605 CN**: 定义宏 `_vel_vfcmps_vvvl`，用于条件编译、简写或 API 生成。
- **L606 EN**: Defines macro `_vel_vfcmps_vvvvl` for conditional compilation, shorthand, or API generation.
  **L606 CN**: 定义宏 `_vel_vfcmps_vvvvl`，用于条件编译、简写或 API 生成。
- **L607 EN**: Defines macro `_vel_vfcmps_vsvl` for conditional compilation, shorthand, or API generation.
  **L607 CN**: 定义宏 `_vel_vfcmps_vsvl`，用于条件编译、简写或 API 生成。
- **L608 EN**: Defines macro `_vel_vfcmps_vsvvl` for conditional compilation, shorthand, or API generation.
  **L608 CN**: 定义宏 `_vel_vfcmps_vsvvl`，用于条件编译、简写或 API 生成。
- **L609 EN**: Defines macro `_vel_vfcmps_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L609 CN**: 定义宏 `_vel_vfcmps_vvvmvl`，用于条件编译、简写或 API 生成。
- **L610 EN**: Defines macro `_vel_vfcmps_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L610 CN**: 定义宏 `_vel_vfcmps_vsvmvl`，用于条件编译、简写或 API 生成。
- **L611 EN**: Defines macro `_vel_pvfcmp_vvvl` for conditional compilation, shorthand, or API generation.
  **L611 CN**: 定义宏 `_vel_pvfcmp_vvvl`，用于条件编译、简写或 API 生成。
- **L612 EN**: Defines macro `_vel_pvfcmp_vvvvl` for conditional compilation, shorthand, or API generation.
  **L612 CN**: 定义宏 `_vel_pvfcmp_vvvvl`，用于条件编译、简写或 API 生成。
- **L613 EN**: Defines macro `_vel_pvfcmp_vsvl` for conditional compilation, shorthand, or API generation.
  **L613 CN**: 定义宏 `_vel_pvfcmp_vsvl`，用于条件编译、简写或 API 生成。
- **L614 EN**: Defines macro `_vel_pvfcmp_vsvvl` for conditional compilation, shorthand, or API generation.
  **L614 CN**: 定义宏 `_vel_pvfcmp_vsvvl`，用于条件编译、简写或 API 生成。
- **L615 EN**: Defines macro `_vel_pvfcmp_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L615 CN**: 定义宏 `_vel_pvfcmp_vvvMvl`，用于条件编译、简写或 API 生成。
- **L616 EN**: Defines macro `_vel_pvfcmp_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L616 CN**: 定义宏 `_vel_pvfcmp_vsvMvl`，用于条件编译、简写或 API 生成。
- **L617 EN**: Defines macro `_vel_vfmaxd_vvvl` for conditional compilation, shorthand, or API generation.
  **L617 CN**: 定义宏 `_vel_vfmaxd_vvvl`，用于条件编译、简写或 API 生成。
- **L618 EN**: Defines macro `_vel_vfmaxd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L618 CN**: 定义宏 `_vel_vfmaxd_vvvvl`，用于条件编译、简写或 API 生成。
- **L619 EN**: Defines macro `_vel_vfmaxd_vsvl` for conditional compilation, shorthand, or API generation.
  **L619 CN**: 定义宏 `_vel_vfmaxd_vsvl`，用于条件编译、简写或 API 生成。
- **L620 EN**: Defines macro `_vel_vfmaxd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L620 CN**: 定义宏 `_vel_vfmaxd_vsvvl`，用于条件编译、简写或 API 生成。
- **L621 EN**: Defines macro `_vel_vfmaxd_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L621 CN**: 定义宏 `_vel_vfmaxd_vvvmvl`，用于条件编译、简写或 API 生成。
- **L622 EN**: Defines macro `_vel_vfmaxd_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L622 CN**: 定义宏 `_vel_vfmaxd_vsvmvl`，用于条件编译、简写或 API 生成。
- **L623 EN**: Defines macro `_vel_vfmaxs_vvvl` for conditional compilation, shorthand, or API generation.
  **L623 CN**: 定义宏 `_vel_vfmaxs_vvvl`，用于条件编译、简写或 API 生成。
- **L624 EN**: Defines macro `_vel_vfmaxs_vvvvl` for conditional compilation, shorthand, or API generation.
  **L624 CN**: 定义宏 `_vel_vfmaxs_vvvvl`，用于条件编译、简写或 API 生成。

### Lines 625-648

````c
#define _vel_vfmaxs_vsvl __builtin_ve_vl_vfmaxs_vsvl
#define _vel_vfmaxs_vsvvl __builtin_ve_vl_vfmaxs_vsvvl
#define _vel_vfmaxs_vvvmvl __builtin_ve_vl_vfmaxs_vvvmvl
#define _vel_vfmaxs_vsvmvl __builtin_ve_vl_vfmaxs_vsvmvl
#define _vel_pvfmax_vvvl __builtin_ve_vl_pvfmax_vvvl
#define _vel_pvfmax_vvvvl __builtin_ve_vl_pvfmax_vvvvl
#define _vel_pvfmax_vsvl __builtin_ve_vl_pvfmax_vsvl
#define _vel_pvfmax_vsvvl __builtin_ve_vl_pvfmax_vsvvl
#define _vel_pvfmax_vvvMvl __builtin_ve_vl_pvfmax_vvvMvl
#define _vel_pvfmax_vsvMvl __builtin_ve_vl_pvfmax_vsvMvl
#define _vel_vfmind_vvvl __builtin_ve_vl_vfmind_vvvl
#define _vel_vfmind_vvvvl __builtin_ve_vl_vfmind_vvvvl
#define _vel_vfmind_vsvl __builtin_ve_vl_vfmind_vsvl
#define _vel_vfmind_vsvvl __builtin_ve_vl_vfmind_vsvvl
#define _vel_vfmind_vvvmvl __builtin_ve_vl_vfmind_vvvmvl
#define _vel_vfmind_vsvmvl __builtin_ve_vl_vfmind_vsvmvl
#define _vel_vfmins_vvvl __builtin_ve_vl_vfmins_vvvl
#define _vel_vfmins_vvvvl __builtin_ve_vl_vfmins_vvvvl
#define _vel_vfmins_vsvl __builtin_ve_vl_vfmins_vsvl
#define _vel_vfmins_vsvvl __builtin_ve_vl_vfmins_vsvvl
#define _vel_vfmins_vvvmvl __builtin_ve_vl_vfmins_vvvmvl
#define _vel_vfmins_vsvmvl __builtin_ve_vl_vfmins_vsvmvl
#define _vel_pvfmin_vvvl __builtin_ve_vl_pvfmin_vvvl
#define _vel_pvfmin_vvvvl __builtin_ve_vl_pvfmin_vvvvl
````
- **L625 EN**: Defines macro `_vel_vfmaxs_vsvl` for conditional compilation, shorthand, or API generation.
  **L625 CN**: 定义宏 `_vel_vfmaxs_vsvl`，用于条件编译、简写或 API 生成。
- **L626 EN**: Defines macro `_vel_vfmaxs_vsvvl` for conditional compilation, shorthand, or API generation.
  **L626 CN**: 定义宏 `_vel_vfmaxs_vsvvl`，用于条件编译、简写或 API 生成。
- **L627 EN**: Defines macro `_vel_vfmaxs_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L627 CN**: 定义宏 `_vel_vfmaxs_vvvmvl`，用于条件编译、简写或 API 生成。
- **L628 EN**: Defines macro `_vel_vfmaxs_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L628 CN**: 定义宏 `_vel_vfmaxs_vsvmvl`，用于条件编译、简写或 API 生成。
- **L629 EN**: Defines macro `_vel_pvfmax_vvvl` for conditional compilation, shorthand, or API generation.
  **L629 CN**: 定义宏 `_vel_pvfmax_vvvl`，用于条件编译、简写或 API 生成。
- **L630 EN**: Defines macro `_vel_pvfmax_vvvvl` for conditional compilation, shorthand, or API generation.
  **L630 CN**: 定义宏 `_vel_pvfmax_vvvvl`，用于条件编译、简写或 API 生成。
- **L631 EN**: Defines macro `_vel_pvfmax_vsvl` for conditional compilation, shorthand, or API generation.
  **L631 CN**: 定义宏 `_vel_pvfmax_vsvl`，用于条件编译、简写或 API 生成。
- **L632 EN**: Defines macro `_vel_pvfmax_vsvvl` for conditional compilation, shorthand, or API generation.
  **L632 CN**: 定义宏 `_vel_pvfmax_vsvvl`，用于条件编译、简写或 API 生成。
- **L633 EN**: Defines macro `_vel_pvfmax_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L633 CN**: 定义宏 `_vel_pvfmax_vvvMvl`，用于条件编译、简写或 API 生成。
- **L634 EN**: Defines macro `_vel_pvfmax_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L634 CN**: 定义宏 `_vel_pvfmax_vsvMvl`，用于条件编译、简写或 API 生成。
- **L635 EN**: Defines macro `_vel_vfmind_vvvl` for conditional compilation, shorthand, or API generation.
  **L635 CN**: 定义宏 `_vel_vfmind_vvvl`，用于条件编译、简写或 API 生成。
- **L636 EN**: Defines macro `_vel_vfmind_vvvvl` for conditional compilation, shorthand, or API generation.
  **L636 CN**: 定义宏 `_vel_vfmind_vvvvl`，用于条件编译、简写或 API 生成。
- **L637 EN**: Defines macro `_vel_vfmind_vsvl` for conditional compilation, shorthand, or API generation.
  **L637 CN**: 定义宏 `_vel_vfmind_vsvl`，用于条件编译、简写或 API 生成。
- **L638 EN**: Defines macro `_vel_vfmind_vsvvl` for conditional compilation, shorthand, or API generation.
  **L638 CN**: 定义宏 `_vel_vfmind_vsvvl`，用于条件编译、简写或 API 生成。
- **L639 EN**: Defines macro `_vel_vfmind_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L639 CN**: 定义宏 `_vel_vfmind_vvvmvl`，用于条件编译、简写或 API 生成。
- **L640 EN**: Defines macro `_vel_vfmind_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L640 CN**: 定义宏 `_vel_vfmind_vsvmvl`，用于条件编译、简写或 API 生成。
- **L641 EN**: Defines macro `_vel_vfmins_vvvl` for conditional compilation, shorthand, or API generation.
  **L641 CN**: 定义宏 `_vel_vfmins_vvvl`，用于条件编译、简写或 API 生成。
- **L642 EN**: Defines macro `_vel_vfmins_vvvvl` for conditional compilation, shorthand, or API generation.
  **L642 CN**: 定义宏 `_vel_vfmins_vvvvl`，用于条件编译、简写或 API 生成。
- **L643 EN**: Defines macro `_vel_vfmins_vsvl` for conditional compilation, shorthand, or API generation.
  **L643 CN**: 定义宏 `_vel_vfmins_vsvl`，用于条件编译、简写或 API 生成。
- **L644 EN**: Defines macro `_vel_vfmins_vsvvl` for conditional compilation, shorthand, or API generation.
  **L644 CN**: 定义宏 `_vel_vfmins_vsvvl`，用于条件编译、简写或 API 生成。
- **L645 EN**: Defines macro `_vel_vfmins_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L645 CN**: 定义宏 `_vel_vfmins_vvvmvl`，用于条件编译、简写或 API 生成。
- **L646 EN**: Defines macro `_vel_vfmins_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L646 CN**: 定义宏 `_vel_vfmins_vsvmvl`，用于条件编译、简写或 API 生成。
- **L647 EN**: Defines macro `_vel_pvfmin_vvvl` for conditional compilation, shorthand, or API generation.
  **L647 CN**: 定义宏 `_vel_pvfmin_vvvl`，用于条件编译、简写或 API 生成。
- **L648 EN**: Defines macro `_vel_pvfmin_vvvvl` for conditional compilation, shorthand, or API generation.
  **L648 CN**: 定义宏 `_vel_pvfmin_vvvvl`，用于条件编译、简写或 API 生成。

### Lines 649-672

````c
#define _vel_pvfmin_vsvl __builtin_ve_vl_pvfmin_vsvl
#define _vel_pvfmin_vsvvl __builtin_ve_vl_pvfmin_vsvvl
#define _vel_pvfmin_vvvMvl __builtin_ve_vl_pvfmin_vvvMvl
#define _vel_pvfmin_vsvMvl __builtin_ve_vl_pvfmin_vsvMvl
#define _vel_vfmadd_vvvvl __builtin_ve_vl_vfmadd_vvvvl
#define _vel_vfmadd_vvvvvl __builtin_ve_vl_vfmadd_vvvvvl
#define _vel_vfmadd_vsvvl __builtin_ve_vl_vfmadd_vsvvl
#define _vel_vfmadd_vsvvvl __builtin_ve_vl_vfmadd_vsvvvl
#define _vel_vfmadd_vvsvl __builtin_ve_vl_vfmadd_vvsvl
#define _vel_vfmadd_vvsvvl __builtin_ve_vl_vfmadd_vvsvvl
#define _vel_vfmadd_vvvvmvl __builtin_ve_vl_vfmadd_vvvvmvl
#define _vel_vfmadd_vsvvmvl __builtin_ve_vl_vfmadd_vsvvmvl
#define _vel_vfmadd_vvsvmvl __builtin_ve_vl_vfmadd_vvsvmvl
#define _vel_vfmads_vvvvl __builtin_ve_vl_vfmads_vvvvl
#define _vel_vfmads_vvvvvl __builtin_ve_vl_vfmads_vvvvvl
#define _vel_vfmads_vsvvl __builtin_ve_vl_vfmads_vsvvl
#define _vel_vfmads_vsvvvl __builtin_ve_vl_vfmads_vsvvvl
#define _vel_vfmads_vvsvl __builtin_ve_vl_vfmads_vvsvl
#define _vel_vfmads_vvsvvl __builtin_ve_vl_vfmads_vvsvvl
#define _vel_vfmads_vvvvmvl __builtin_ve_vl_vfmads_vvvvmvl
#define _vel_vfmads_vsvvmvl __builtin_ve_vl_vfmads_vsvvmvl
#define _vel_vfmads_vvsvmvl __builtin_ve_vl_vfmads_vvsvmvl
#define _vel_pvfmad_vvvvl __builtin_ve_vl_pvfmad_vvvvl
#define _vel_pvfmad_vvvvvl __builtin_ve_vl_pvfmad_vvvvvl
````
- **L649 EN**: Defines macro `_vel_pvfmin_vsvl` for conditional compilation, shorthand, or API generation.
  **L649 CN**: 定义宏 `_vel_pvfmin_vsvl`，用于条件编译、简写或 API 生成。
- **L650 EN**: Defines macro `_vel_pvfmin_vsvvl` for conditional compilation, shorthand, or API generation.
  **L650 CN**: 定义宏 `_vel_pvfmin_vsvvl`，用于条件编译、简写或 API 生成。
- **L651 EN**: Defines macro `_vel_pvfmin_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L651 CN**: 定义宏 `_vel_pvfmin_vvvMvl`，用于条件编译、简写或 API 生成。
- **L652 EN**: Defines macro `_vel_pvfmin_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L652 CN**: 定义宏 `_vel_pvfmin_vsvMvl`，用于条件编译、简写或 API 生成。
- **L653 EN**: Defines macro `_vel_vfmadd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L653 CN**: 定义宏 `_vel_vfmadd_vvvvl`，用于条件编译、简写或 API 生成。
- **L654 EN**: Defines macro `_vel_vfmadd_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L654 CN**: 定义宏 `_vel_vfmadd_vvvvvl`，用于条件编译、简写或 API 生成。
- **L655 EN**: Defines macro `_vel_vfmadd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L655 CN**: 定义宏 `_vel_vfmadd_vsvvl`，用于条件编译、简写或 API 生成。
- **L656 EN**: Defines macro `_vel_vfmadd_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L656 CN**: 定义宏 `_vel_vfmadd_vsvvvl`，用于条件编译、简写或 API 生成。
- **L657 EN**: Defines macro `_vel_vfmadd_vvsvl` for conditional compilation, shorthand, or API generation.
  **L657 CN**: 定义宏 `_vel_vfmadd_vvsvl`，用于条件编译、简写或 API 生成。
- **L658 EN**: Defines macro `_vel_vfmadd_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L658 CN**: 定义宏 `_vel_vfmadd_vvsvvl`，用于条件编译、简写或 API 生成。
- **L659 EN**: Defines macro `_vel_vfmadd_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L659 CN**: 定义宏 `_vel_vfmadd_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L660 EN**: Defines macro `_vel_vfmadd_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L660 CN**: 定义宏 `_vel_vfmadd_vsvvmvl`，用于条件编译、简写或 API 生成。
- **L661 EN**: Defines macro `_vel_vfmadd_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L661 CN**: 定义宏 `_vel_vfmadd_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L662 EN**: Defines macro `_vel_vfmads_vvvvl` for conditional compilation, shorthand, or API generation.
  **L662 CN**: 定义宏 `_vel_vfmads_vvvvl`，用于条件编译、简写或 API 生成。
- **L663 EN**: Defines macro `_vel_vfmads_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L663 CN**: 定义宏 `_vel_vfmads_vvvvvl`，用于条件编译、简写或 API 生成。
- **L664 EN**: Defines macro `_vel_vfmads_vsvvl` for conditional compilation, shorthand, or API generation.
  **L664 CN**: 定义宏 `_vel_vfmads_vsvvl`，用于条件编译、简写或 API 生成。
- **L665 EN**: Defines macro `_vel_vfmads_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L665 CN**: 定义宏 `_vel_vfmads_vsvvvl`，用于条件编译、简写或 API 生成。
- **L666 EN**: Defines macro `_vel_vfmads_vvsvl` for conditional compilation, shorthand, or API generation.
  **L666 CN**: 定义宏 `_vel_vfmads_vvsvl`，用于条件编译、简写或 API 生成。
- **L667 EN**: Defines macro `_vel_vfmads_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L667 CN**: 定义宏 `_vel_vfmads_vvsvvl`，用于条件编译、简写或 API 生成。
- **L668 EN**: Defines macro `_vel_vfmads_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L668 CN**: 定义宏 `_vel_vfmads_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L669 EN**: Defines macro `_vel_vfmads_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L669 CN**: 定义宏 `_vel_vfmads_vsvvmvl`，用于条件编译、简写或 API 生成。
- **L670 EN**: Defines macro `_vel_vfmads_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L670 CN**: 定义宏 `_vel_vfmads_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L671 EN**: Defines macro `_vel_pvfmad_vvvvl` for conditional compilation, shorthand, or API generation.
  **L671 CN**: 定义宏 `_vel_pvfmad_vvvvl`，用于条件编译、简写或 API 生成。
- **L672 EN**: Defines macro `_vel_pvfmad_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L672 CN**: 定义宏 `_vel_pvfmad_vvvvvl`，用于条件编译、简写或 API 生成。

### Lines 673-696

````c
#define _vel_pvfmad_vsvvl __builtin_ve_vl_pvfmad_vsvvl
#define _vel_pvfmad_vsvvvl __builtin_ve_vl_pvfmad_vsvvvl
#define _vel_pvfmad_vvsvl __builtin_ve_vl_pvfmad_vvsvl
#define _vel_pvfmad_vvsvvl __builtin_ve_vl_pvfmad_vvsvvl
#define _vel_pvfmad_vvvvMvl __builtin_ve_vl_pvfmad_vvvvMvl
#define _vel_pvfmad_vsvvMvl __builtin_ve_vl_pvfmad_vsvvMvl
#define _vel_pvfmad_vvsvMvl __builtin_ve_vl_pvfmad_vvsvMvl
#define _vel_vfmsbd_vvvvl __builtin_ve_vl_vfmsbd_vvvvl
#define _vel_vfmsbd_vvvvvl __builtin_ve_vl_vfmsbd_vvvvvl
#define _vel_vfmsbd_vsvvl __builtin_ve_vl_vfmsbd_vsvvl
#define _vel_vfmsbd_vsvvvl __builtin_ve_vl_vfmsbd_vsvvvl
#define _vel_vfmsbd_vvsvl __builtin_ve_vl_vfmsbd_vvsvl
#define _vel_vfmsbd_vvsvvl __builtin_ve_vl_vfmsbd_vvsvvl
#define _vel_vfmsbd_vvvvmvl __builtin_ve_vl_vfmsbd_vvvvmvl
#define _vel_vfmsbd_vsvvmvl __builtin_ve_vl_vfmsbd_vsvvmvl
#define _vel_vfmsbd_vvsvmvl __builtin_ve_vl_vfmsbd_vvsvmvl
#define _vel_vfmsbs_vvvvl __builtin_ve_vl_vfmsbs_vvvvl
#define _vel_vfmsbs_vvvvvl __builtin_ve_vl_vfmsbs_vvvvvl
#define _vel_vfmsbs_vsvvl __builtin_ve_vl_vfmsbs_vsvvl
#define _vel_vfmsbs_vsvvvl __builtin_ve_vl_vfmsbs_vsvvvl
#define _vel_vfmsbs_vvsvl __builtin_ve_vl_vfmsbs_vvsvl
#define _vel_vfmsbs_vvsvvl __builtin_ve_vl_vfmsbs_vvsvvl
#define _vel_vfmsbs_vvvvmvl __builtin_ve_vl_vfmsbs_vvvvmvl
#define _vel_vfmsbs_vsvvmvl __builtin_ve_vl_vfmsbs_vsvvmvl
````
- **L673 EN**: Defines macro `_vel_pvfmad_vsvvl` for conditional compilation, shorthand, or API generation.
  **L673 CN**: 定义宏 `_vel_pvfmad_vsvvl`，用于条件编译、简写或 API 生成。
- **L674 EN**: Defines macro `_vel_pvfmad_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L674 CN**: 定义宏 `_vel_pvfmad_vsvvvl`，用于条件编译、简写或 API 生成。
- **L675 EN**: Defines macro `_vel_pvfmad_vvsvl` for conditional compilation, shorthand, or API generation.
  **L675 CN**: 定义宏 `_vel_pvfmad_vvsvl`，用于条件编译、简写或 API 生成。
- **L676 EN**: Defines macro `_vel_pvfmad_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L676 CN**: 定义宏 `_vel_pvfmad_vvsvvl`，用于条件编译、简写或 API 生成。
- **L677 EN**: Defines macro `_vel_pvfmad_vvvvMvl` for conditional compilation, shorthand, or API generation.
  **L677 CN**: 定义宏 `_vel_pvfmad_vvvvMvl`，用于条件编译、简写或 API 生成。
- **L678 EN**: Defines macro `_vel_pvfmad_vsvvMvl` for conditional compilation, shorthand, or API generation.
  **L678 CN**: 定义宏 `_vel_pvfmad_vsvvMvl`，用于条件编译、简写或 API 生成。
- **L679 EN**: Defines macro `_vel_pvfmad_vvsvMvl` for conditional compilation, shorthand, or API generation.
  **L679 CN**: 定义宏 `_vel_pvfmad_vvsvMvl`，用于条件编译、简写或 API 生成。
- **L680 EN**: Defines macro `_vel_vfmsbd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L680 CN**: 定义宏 `_vel_vfmsbd_vvvvl`，用于条件编译、简写或 API 生成。
- **L681 EN**: Defines macro `_vel_vfmsbd_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L681 CN**: 定义宏 `_vel_vfmsbd_vvvvvl`，用于条件编译、简写或 API 生成。
- **L682 EN**: Defines macro `_vel_vfmsbd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L682 CN**: 定义宏 `_vel_vfmsbd_vsvvl`，用于条件编译、简写或 API 生成。
- **L683 EN**: Defines macro `_vel_vfmsbd_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L683 CN**: 定义宏 `_vel_vfmsbd_vsvvvl`，用于条件编译、简写或 API 生成。
- **L684 EN**: Defines macro `_vel_vfmsbd_vvsvl` for conditional compilation, shorthand, or API generation.
  **L684 CN**: 定义宏 `_vel_vfmsbd_vvsvl`，用于条件编译、简写或 API 生成。
- **L685 EN**: Defines macro `_vel_vfmsbd_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L685 CN**: 定义宏 `_vel_vfmsbd_vvsvvl`，用于条件编译、简写或 API 生成。
- **L686 EN**: Defines macro `_vel_vfmsbd_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L686 CN**: 定义宏 `_vel_vfmsbd_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L687 EN**: Defines macro `_vel_vfmsbd_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L687 CN**: 定义宏 `_vel_vfmsbd_vsvvmvl`，用于条件编译、简写或 API 生成。
- **L688 EN**: Defines macro `_vel_vfmsbd_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L688 CN**: 定义宏 `_vel_vfmsbd_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L689 EN**: Defines macro `_vel_vfmsbs_vvvvl` for conditional compilation, shorthand, or API generation.
  **L689 CN**: 定义宏 `_vel_vfmsbs_vvvvl`，用于条件编译、简写或 API 生成。
- **L690 EN**: Defines macro `_vel_vfmsbs_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L690 CN**: 定义宏 `_vel_vfmsbs_vvvvvl`，用于条件编译、简写或 API 生成。
- **L691 EN**: Defines macro `_vel_vfmsbs_vsvvl` for conditional compilation, shorthand, or API generation.
  **L691 CN**: 定义宏 `_vel_vfmsbs_vsvvl`，用于条件编译、简写或 API 生成。
- **L692 EN**: Defines macro `_vel_vfmsbs_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L692 CN**: 定义宏 `_vel_vfmsbs_vsvvvl`，用于条件编译、简写或 API 生成。
- **L693 EN**: Defines macro `_vel_vfmsbs_vvsvl` for conditional compilation, shorthand, or API generation.
  **L693 CN**: 定义宏 `_vel_vfmsbs_vvsvl`，用于条件编译、简写或 API 生成。
- **L694 EN**: Defines macro `_vel_vfmsbs_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L694 CN**: 定义宏 `_vel_vfmsbs_vvsvvl`，用于条件编译、简写或 API 生成。
- **L695 EN**: Defines macro `_vel_vfmsbs_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L695 CN**: 定义宏 `_vel_vfmsbs_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L696 EN**: Defines macro `_vel_vfmsbs_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L696 CN**: 定义宏 `_vel_vfmsbs_vsvvmvl`，用于条件编译、简写或 API 生成。

### Lines 697-720

````c
#define _vel_vfmsbs_vvsvmvl __builtin_ve_vl_vfmsbs_vvsvmvl
#define _vel_pvfmsb_vvvvl __builtin_ve_vl_pvfmsb_vvvvl
#define _vel_pvfmsb_vvvvvl __builtin_ve_vl_pvfmsb_vvvvvl
#define _vel_pvfmsb_vsvvl __builtin_ve_vl_pvfmsb_vsvvl
#define _vel_pvfmsb_vsvvvl __builtin_ve_vl_pvfmsb_vsvvvl
#define _vel_pvfmsb_vvsvl __builtin_ve_vl_pvfmsb_vvsvl
#define _vel_pvfmsb_vvsvvl __builtin_ve_vl_pvfmsb_vvsvvl
#define _vel_pvfmsb_vvvvMvl __builtin_ve_vl_pvfmsb_vvvvMvl
#define _vel_pvfmsb_vsvvMvl __builtin_ve_vl_pvfmsb_vsvvMvl
#define _vel_pvfmsb_vvsvMvl __builtin_ve_vl_pvfmsb_vvsvMvl
#define _vel_vfnmadd_vvvvl __builtin_ve_vl_vfnmadd_vvvvl
#define _vel_vfnmadd_vvvvvl __builtin_ve_vl_vfnmadd_vvvvvl
#define _vel_vfnmadd_vsvvl __builtin_ve_vl_vfnmadd_vsvvl
#define _vel_vfnmadd_vsvvvl __builtin_ve_vl_vfnmadd_vsvvvl
#define _vel_vfnmadd_vvsvl __builtin_ve_vl_vfnmadd_vvsvl
#define _vel_vfnmadd_vvsvvl __builtin_ve_vl_vfnmadd_vvsvvl
#define _vel_vfnmadd_vvvvmvl __builtin_ve_vl_vfnmadd_vvvvmvl
#define _vel_vfnmadd_vsvvmvl __builtin_ve_vl_vfnmadd_vsvvmvl
#define _vel_vfnmadd_vvsvmvl __builtin_ve_vl_vfnmadd_vvsvmvl
#define _vel_vfnmads_vvvvl __builtin_ve_vl_vfnmads_vvvvl
#define _vel_vfnmads_vvvvvl __builtin_ve_vl_vfnmads_vvvvvl
#define _vel_vfnmads_vsvvl __builtin_ve_vl_vfnmads_vsvvl
#define _vel_vfnmads_vsvvvl __builtin_ve_vl_vfnmads_vsvvvl
#define _vel_vfnmads_vvsvl __builtin_ve_vl_vfnmads_vvsvl
````
- **L697 EN**: Defines macro `_vel_vfmsbs_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L697 CN**: 定义宏 `_vel_vfmsbs_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L698 EN**: Defines macro `_vel_pvfmsb_vvvvl` for conditional compilation, shorthand, or API generation.
  **L698 CN**: 定义宏 `_vel_pvfmsb_vvvvl`，用于条件编译、简写或 API 生成。
- **L699 EN**: Defines macro `_vel_pvfmsb_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L699 CN**: 定义宏 `_vel_pvfmsb_vvvvvl`，用于条件编译、简写或 API 生成。
- **L700 EN**: Defines macro `_vel_pvfmsb_vsvvl` for conditional compilation, shorthand, or API generation.
  **L700 CN**: 定义宏 `_vel_pvfmsb_vsvvl`，用于条件编译、简写或 API 生成。
- **L701 EN**: Defines macro `_vel_pvfmsb_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L701 CN**: 定义宏 `_vel_pvfmsb_vsvvvl`，用于条件编译、简写或 API 生成。
- **L702 EN**: Defines macro `_vel_pvfmsb_vvsvl` for conditional compilation, shorthand, or API generation.
  **L702 CN**: 定义宏 `_vel_pvfmsb_vvsvl`，用于条件编译、简写或 API 生成。
- **L703 EN**: Defines macro `_vel_pvfmsb_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L703 CN**: 定义宏 `_vel_pvfmsb_vvsvvl`，用于条件编译、简写或 API 生成。
- **L704 EN**: Defines macro `_vel_pvfmsb_vvvvMvl` for conditional compilation, shorthand, or API generation.
  **L704 CN**: 定义宏 `_vel_pvfmsb_vvvvMvl`，用于条件编译、简写或 API 生成。
- **L705 EN**: Defines macro `_vel_pvfmsb_vsvvMvl` for conditional compilation, shorthand, or API generation.
  **L705 CN**: 定义宏 `_vel_pvfmsb_vsvvMvl`，用于条件编译、简写或 API 生成。
- **L706 EN**: Defines macro `_vel_pvfmsb_vvsvMvl` for conditional compilation, shorthand, or API generation.
  **L706 CN**: 定义宏 `_vel_pvfmsb_vvsvMvl`，用于条件编译、简写或 API 生成。
- **L707 EN**: Defines macro `_vel_vfnmadd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L707 CN**: 定义宏 `_vel_vfnmadd_vvvvl`，用于条件编译、简写或 API 生成。
- **L708 EN**: Defines macro `_vel_vfnmadd_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L708 CN**: 定义宏 `_vel_vfnmadd_vvvvvl`，用于条件编译、简写或 API 生成。
- **L709 EN**: Defines macro `_vel_vfnmadd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L709 CN**: 定义宏 `_vel_vfnmadd_vsvvl`，用于条件编译、简写或 API 生成。
- **L710 EN**: Defines macro `_vel_vfnmadd_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L710 CN**: 定义宏 `_vel_vfnmadd_vsvvvl`，用于条件编译、简写或 API 生成。
- **L711 EN**: Defines macro `_vel_vfnmadd_vvsvl` for conditional compilation, shorthand, or API generation.
  **L711 CN**: 定义宏 `_vel_vfnmadd_vvsvl`，用于条件编译、简写或 API 生成。
- **L712 EN**: Defines macro `_vel_vfnmadd_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L712 CN**: 定义宏 `_vel_vfnmadd_vvsvvl`，用于条件编译、简写或 API 生成。
- **L713 EN**: Defines macro `_vel_vfnmadd_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L713 CN**: 定义宏 `_vel_vfnmadd_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L714 EN**: Defines macro `_vel_vfnmadd_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L714 CN**: 定义宏 `_vel_vfnmadd_vsvvmvl`，用于条件编译、简写或 API 生成。
- **L715 EN**: Defines macro `_vel_vfnmadd_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L715 CN**: 定义宏 `_vel_vfnmadd_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L716 EN**: Defines macro `_vel_vfnmads_vvvvl` for conditional compilation, shorthand, or API generation.
  **L716 CN**: 定义宏 `_vel_vfnmads_vvvvl`，用于条件编译、简写或 API 生成。
- **L717 EN**: Defines macro `_vel_vfnmads_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L717 CN**: 定义宏 `_vel_vfnmads_vvvvvl`，用于条件编译、简写或 API 生成。
- **L718 EN**: Defines macro `_vel_vfnmads_vsvvl` for conditional compilation, shorthand, or API generation.
  **L718 CN**: 定义宏 `_vel_vfnmads_vsvvl`，用于条件编译、简写或 API 生成。
- **L719 EN**: Defines macro `_vel_vfnmads_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L719 CN**: 定义宏 `_vel_vfnmads_vsvvvl`，用于条件编译、简写或 API 生成。
- **L720 EN**: Defines macro `_vel_vfnmads_vvsvl` for conditional compilation, shorthand, or API generation.
  **L720 CN**: 定义宏 `_vel_vfnmads_vvsvl`，用于条件编译、简写或 API 生成。

### Lines 721-744

````c
#define _vel_vfnmads_vvsvvl __builtin_ve_vl_vfnmads_vvsvvl
#define _vel_vfnmads_vvvvmvl __builtin_ve_vl_vfnmads_vvvvmvl
#define _vel_vfnmads_vsvvmvl __builtin_ve_vl_vfnmads_vsvvmvl
#define _vel_vfnmads_vvsvmvl __builtin_ve_vl_vfnmads_vvsvmvl
#define _vel_pvfnmad_vvvvl __builtin_ve_vl_pvfnmad_vvvvl
#define _vel_pvfnmad_vvvvvl __builtin_ve_vl_pvfnmad_vvvvvl
#define _vel_pvfnmad_vsvvl __builtin_ve_vl_pvfnmad_vsvvl
#define _vel_pvfnmad_vsvvvl __builtin_ve_vl_pvfnmad_vsvvvl
#define _vel_pvfnmad_vvsvl __builtin_ve_vl_pvfnmad_vvsvl
#define _vel_pvfnmad_vvsvvl __builtin_ve_vl_pvfnmad_vvsvvl
#define _vel_pvfnmad_vvvvMvl __builtin_ve_vl_pvfnmad_vvvvMvl
#define _vel_pvfnmad_vsvvMvl __builtin_ve_vl_pvfnmad_vsvvMvl
#define _vel_pvfnmad_vvsvMvl __builtin_ve_vl_pvfnmad_vvsvMvl
#define _vel_vfnmsbd_vvvvl __builtin_ve_vl_vfnmsbd_vvvvl
#define _vel_vfnmsbd_vvvvvl __builtin_ve_vl_vfnmsbd_vvvvvl
#define _vel_vfnmsbd_vsvvl __builtin_ve_vl_vfnmsbd_vsvvl
#define _vel_vfnmsbd_vsvvvl __builtin_ve_vl_vfnmsbd_vsvvvl
#define _vel_vfnmsbd_vvsvl __builtin_ve_vl_vfnmsbd_vvsvl
#define _vel_vfnmsbd_vvsvvl __builtin_ve_vl_vfnmsbd_vvsvvl
#define _vel_vfnmsbd_vvvvmvl __builtin_ve_vl_vfnmsbd_vvvvmvl
#define _vel_vfnmsbd_vsvvmvl __builtin_ve_vl_vfnmsbd_vsvvmvl
#define _vel_vfnmsbd_vvsvmvl __builtin_ve_vl_vfnmsbd_vvsvmvl
#define _vel_vfnmsbs_vvvvl __builtin_ve_vl_vfnmsbs_vvvvl
#define _vel_vfnmsbs_vvvvvl __builtin_ve_vl_vfnmsbs_vvvvvl
````
- **L721 EN**: Defines macro `_vel_vfnmads_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L721 CN**: 定义宏 `_vel_vfnmads_vvsvvl`，用于条件编译、简写或 API 生成。
- **L722 EN**: Defines macro `_vel_vfnmads_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L722 CN**: 定义宏 `_vel_vfnmads_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L723 EN**: Defines macro `_vel_vfnmads_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L723 CN**: 定义宏 `_vel_vfnmads_vsvvmvl`，用于条件编译、简写或 API 生成。
- **L724 EN**: Defines macro `_vel_vfnmads_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L724 CN**: 定义宏 `_vel_vfnmads_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L725 EN**: Defines macro `_vel_pvfnmad_vvvvl` for conditional compilation, shorthand, or API generation.
  **L725 CN**: 定义宏 `_vel_pvfnmad_vvvvl`，用于条件编译、简写或 API 生成。
- **L726 EN**: Defines macro `_vel_pvfnmad_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L726 CN**: 定义宏 `_vel_pvfnmad_vvvvvl`，用于条件编译、简写或 API 生成。
- **L727 EN**: Defines macro `_vel_pvfnmad_vsvvl` for conditional compilation, shorthand, or API generation.
  **L727 CN**: 定义宏 `_vel_pvfnmad_vsvvl`，用于条件编译、简写或 API 生成。
- **L728 EN**: Defines macro `_vel_pvfnmad_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L728 CN**: 定义宏 `_vel_pvfnmad_vsvvvl`，用于条件编译、简写或 API 生成。
- **L729 EN**: Defines macro `_vel_pvfnmad_vvsvl` for conditional compilation, shorthand, or API generation.
  **L729 CN**: 定义宏 `_vel_pvfnmad_vvsvl`，用于条件编译、简写或 API 生成。
- **L730 EN**: Defines macro `_vel_pvfnmad_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L730 CN**: 定义宏 `_vel_pvfnmad_vvsvvl`，用于条件编译、简写或 API 生成。
- **L731 EN**: Defines macro `_vel_pvfnmad_vvvvMvl` for conditional compilation, shorthand, or API generation.
  **L731 CN**: 定义宏 `_vel_pvfnmad_vvvvMvl`，用于条件编译、简写或 API 生成。
- **L732 EN**: Defines macro `_vel_pvfnmad_vsvvMvl` for conditional compilation, shorthand, or API generation.
  **L732 CN**: 定义宏 `_vel_pvfnmad_vsvvMvl`，用于条件编译、简写或 API 生成。
- **L733 EN**: Defines macro `_vel_pvfnmad_vvsvMvl` for conditional compilation, shorthand, or API generation.
  **L733 CN**: 定义宏 `_vel_pvfnmad_vvsvMvl`，用于条件编译、简写或 API 生成。
- **L734 EN**: Defines macro `_vel_vfnmsbd_vvvvl` for conditional compilation, shorthand, or API generation.
  **L734 CN**: 定义宏 `_vel_vfnmsbd_vvvvl`，用于条件编译、简写或 API 生成。
- **L735 EN**: Defines macro `_vel_vfnmsbd_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L735 CN**: 定义宏 `_vel_vfnmsbd_vvvvvl`，用于条件编译、简写或 API 生成。
- **L736 EN**: Defines macro `_vel_vfnmsbd_vsvvl` for conditional compilation, shorthand, or API generation.
  **L736 CN**: 定义宏 `_vel_vfnmsbd_vsvvl`，用于条件编译、简写或 API 生成。
- **L737 EN**: Defines macro `_vel_vfnmsbd_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L737 CN**: 定义宏 `_vel_vfnmsbd_vsvvvl`，用于条件编译、简写或 API 生成。
- **L738 EN**: Defines macro `_vel_vfnmsbd_vvsvl` for conditional compilation, shorthand, or API generation.
  **L738 CN**: 定义宏 `_vel_vfnmsbd_vvsvl`，用于条件编译、简写或 API 生成。
- **L739 EN**: Defines macro `_vel_vfnmsbd_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L739 CN**: 定义宏 `_vel_vfnmsbd_vvsvvl`，用于条件编译、简写或 API 生成。
- **L740 EN**: Defines macro `_vel_vfnmsbd_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L740 CN**: 定义宏 `_vel_vfnmsbd_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L741 EN**: Defines macro `_vel_vfnmsbd_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L741 CN**: 定义宏 `_vel_vfnmsbd_vsvvmvl`，用于条件编译、简写或 API 生成。
- **L742 EN**: Defines macro `_vel_vfnmsbd_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L742 CN**: 定义宏 `_vel_vfnmsbd_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L743 EN**: Defines macro `_vel_vfnmsbs_vvvvl` for conditional compilation, shorthand, or API generation.
  **L743 CN**: 定义宏 `_vel_vfnmsbs_vvvvl`，用于条件编译、简写或 API 生成。
- **L744 EN**: Defines macro `_vel_vfnmsbs_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L744 CN**: 定义宏 `_vel_vfnmsbs_vvvvvl`，用于条件编译、简写或 API 生成。

### Lines 745-768

````c
#define _vel_vfnmsbs_vsvvl __builtin_ve_vl_vfnmsbs_vsvvl
#define _vel_vfnmsbs_vsvvvl __builtin_ve_vl_vfnmsbs_vsvvvl
#define _vel_vfnmsbs_vvsvl __builtin_ve_vl_vfnmsbs_vvsvl
#define _vel_vfnmsbs_vvsvvl __builtin_ve_vl_vfnmsbs_vvsvvl
#define _vel_vfnmsbs_vvvvmvl __builtin_ve_vl_vfnmsbs_vvvvmvl
#define _vel_vfnmsbs_vsvvmvl __builtin_ve_vl_vfnmsbs_vsvvmvl
#define _vel_vfnmsbs_vvsvmvl __builtin_ve_vl_vfnmsbs_vvsvmvl
#define _vel_pvfnmsb_vvvvl __builtin_ve_vl_pvfnmsb_vvvvl
#define _vel_pvfnmsb_vvvvvl __builtin_ve_vl_pvfnmsb_vvvvvl
#define _vel_pvfnmsb_vsvvl __builtin_ve_vl_pvfnmsb_vsvvl
#define _vel_pvfnmsb_vsvvvl __builtin_ve_vl_pvfnmsb_vsvvvl
#define _vel_pvfnmsb_vvsvl __builtin_ve_vl_pvfnmsb_vvsvl
#define _vel_pvfnmsb_vvsvvl __builtin_ve_vl_pvfnmsb_vvsvvl
#define _vel_pvfnmsb_vvvvMvl __builtin_ve_vl_pvfnmsb_vvvvMvl
#define _vel_pvfnmsb_vsvvMvl __builtin_ve_vl_pvfnmsb_vsvvMvl
#define _vel_pvfnmsb_vvsvMvl __builtin_ve_vl_pvfnmsb_vvsvMvl
#define _vel_vrcpd_vvl __builtin_ve_vl_vrcpd_vvl
#define _vel_vrcpd_vvvl __builtin_ve_vl_vrcpd_vvvl
#define _vel_vrcps_vvl __builtin_ve_vl_vrcps_vvl
#define _vel_vrcps_vvvl __builtin_ve_vl_vrcps_vvvl
#define _vel_pvrcp_vvl __builtin_ve_vl_pvrcp_vvl
#define _vel_pvrcp_vvvl __builtin_ve_vl_pvrcp_vvvl
#define _vel_vrsqrtd_vvl __builtin_ve_vl_vrsqrtd_vvl
#define _vel_vrsqrtd_vvvl __builtin_ve_vl_vrsqrtd_vvvl
````
- **L745 EN**: Defines macro `_vel_vfnmsbs_vsvvl` for conditional compilation, shorthand, or API generation.
  **L745 CN**: 定义宏 `_vel_vfnmsbs_vsvvl`，用于条件编译、简写或 API 生成。
- **L746 EN**: Defines macro `_vel_vfnmsbs_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L746 CN**: 定义宏 `_vel_vfnmsbs_vsvvvl`，用于条件编译、简写或 API 生成。
- **L747 EN**: Defines macro `_vel_vfnmsbs_vvsvl` for conditional compilation, shorthand, or API generation.
  **L747 CN**: 定义宏 `_vel_vfnmsbs_vvsvl`，用于条件编译、简写或 API 生成。
- **L748 EN**: Defines macro `_vel_vfnmsbs_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L748 CN**: 定义宏 `_vel_vfnmsbs_vvsvvl`，用于条件编译、简写或 API 生成。
- **L749 EN**: Defines macro `_vel_vfnmsbs_vvvvmvl` for conditional compilation, shorthand, or API generation.
  **L749 CN**: 定义宏 `_vel_vfnmsbs_vvvvmvl`，用于条件编译、简写或 API 生成。
- **L750 EN**: Defines macro `_vel_vfnmsbs_vsvvmvl` for conditional compilation, shorthand, or API generation.
  **L750 CN**: 定义宏 `_vel_vfnmsbs_vsvvmvl`，用于条件编译、简写或 API 生成。
- **L751 EN**: Defines macro `_vel_vfnmsbs_vvsvmvl` for conditional compilation, shorthand, or API generation.
  **L751 CN**: 定义宏 `_vel_vfnmsbs_vvsvmvl`，用于条件编译、简写或 API 生成。
- **L752 EN**: Defines macro `_vel_pvfnmsb_vvvvl` for conditional compilation, shorthand, or API generation.
  **L752 CN**: 定义宏 `_vel_pvfnmsb_vvvvl`，用于条件编译、简写或 API 生成。
- **L753 EN**: Defines macro `_vel_pvfnmsb_vvvvvl` for conditional compilation, shorthand, or API generation.
  **L753 CN**: 定义宏 `_vel_pvfnmsb_vvvvvl`，用于条件编译、简写或 API 生成。
- **L754 EN**: Defines macro `_vel_pvfnmsb_vsvvl` for conditional compilation, shorthand, or API generation.
  **L754 CN**: 定义宏 `_vel_pvfnmsb_vsvvl`，用于条件编译、简写或 API 生成。
- **L755 EN**: Defines macro `_vel_pvfnmsb_vsvvvl` for conditional compilation, shorthand, or API generation.
  **L755 CN**: 定义宏 `_vel_pvfnmsb_vsvvvl`，用于条件编译、简写或 API 生成。
- **L756 EN**: Defines macro `_vel_pvfnmsb_vvsvl` for conditional compilation, shorthand, or API generation.
  **L756 CN**: 定义宏 `_vel_pvfnmsb_vvsvl`，用于条件编译、简写或 API 生成。
- **L757 EN**: Defines macro `_vel_pvfnmsb_vvsvvl` for conditional compilation, shorthand, or API generation.
  **L757 CN**: 定义宏 `_vel_pvfnmsb_vvsvvl`，用于条件编译、简写或 API 生成。
- **L758 EN**: Defines macro `_vel_pvfnmsb_vvvvMvl` for conditional compilation, shorthand, or API generation.
  **L758 CN**: 定义宏 `_vel_pvfnmsb_vvvvMvl`，用于条件编译、简写或 API 生成。
- **L759 EN**: Defines macro `_vel_pvfnmsb_vsvvMvl` for conditional compilation, shorthand, or API generation.
  **L759 CN**: 定义宏 `_vel_pvfnmsb_vsvvMvl`，用于条件编译、简写或 API 生成。
- **L760 EN**: Defines macro `_vel_pvfnmsb_vvsvMvl` for conditional compilation, shorthand, or API generation.
  **L760 CN**: 定义宏 `_vel_pvfnmsb_vvsvMvl`，用于条件编译、简写或 API 生成。
- **L761 EN**: Defines macro `_vel_vrcpd_vvl` for conditional compilation, shorthand, or API generation.
  **L761 CN**: 定义宏 `_vel_vrcpd_vvl`，用于条件编译、简写或 API 生成。
- **L762 EN**: Defines macro `_vel_vrcpd_vvvl` for conditional compilation, shorthand, or API generation.
  **L762 CN**: 定义宏 `_vel_vrcpd_vvvl`，用于条件编译、简写或 API 生成。
- **L763 EN**: Defines macro `_vel_vrcps_vvl` for conditional compilation, shorthand, or API generation.
  **L763 CN**: 定义宏 `_vel_vrcps_vvl`，用于条件编译、简写或 API 生成。
- **L764 EN**: Defines macro `_vel_vrcps_vvvl` for conditional compilation, shorthand, or API generation.
  **L764 CN**: 定义宏 `_vel_vrcps_vvvl`，用于条件编译、简写或 API 生成。
- **L765 EN**: Defines macro `_vel_pvrcp_vvl` for conditional compilation, shorthand, or API generation.
  **L765 CN**: 定义宏 `_vel_pvrcp_vvl`，用于条件编译、简写或 API 生成。
- **L766 EN**: Defines macro `_vel_pvrcp_vvvl` for conditional compilation, shorthand, or API generation.
  **L766 CN**: 定义宏 `_vel_pvrcp_vvvl`，用于条件编译、简写或 API 生成。
- **L767 EN**: Defines macro `_vel_vrsqrtd_vvl` for conditional compilation, shorthand, or API generation.
  **L767 CN**: 定义宏 `_vel_vrsqrtd_vvl`，用于条件编译、简写或 API 生成。
- **L768 EN**: Defines macro `_vel_vrsqrtd_vvvl` for conditional compilation, shorthand, or API generation.
  **L768 CN**: 定义宏 `_vel_vrsqrtd_vvvl`，用于条件编译、简写或 API 生成。

### Lines 769-792

````c
#define _vel_vrsqrts_vvl __builtin_ve_vl_vrsqrts_vvl
#define _vel_vrsqrts_vvvl __builtin_ve_vl_vrsqrts_vvvl
#define _vel_pvrsqrt_vvl __builtin_ve_vl_pvrsqrt_vvl
#define _vel_pvrsqrt_vvvl __builtin_ve_vl_pvrsqrt_vvvl
#define _vel_vrsqrtdnex_vvl __builtin_ve_vl_vrsqrtdnex_vvl
#define _vel_vrsqrtdnex_vvvl __builtin_ve_vl_vrsqrtdnex_vvvl
#define _vel_vrsqrtsnex_vvl __builtin_ve_vl_vrsqrtsnex_vvl
#define _vel_vrsqrtsnex_vvvl __builtin_ve_vl_vrsqrtsnex_vvvl
#define _vel_pvrsqrtnex_vvl __builtin_ve_vl_pvrsqrtnex_vvl
#define _vel_pvrsqrtnex_vvvl __builtin_ve_vl_pvrsqrtnex_vvvl
#define _vel_vcvtwdsx_vvl __builtin_ve_vl_vcvtwdsx_vvl
#define _vel_vcvtwdsx_vvvl __builtin_ve_vl_vcvtwdsx_vvvl
#define _vel_vcvtwdsx_vvmvl __builtin_ve_vl_vcvtwdsx_vvmvl
#define _vel_vcvtwdsxrz_vvl __builtin_ve_vl_vcvtwdsxrz_vvl
#define _vel_vcvtwdsxrz_vvvl __builtin_ve_vl_vcvtwdsxrz_vvvl
#define _vel_vcvtwdsxrz_vvmvl __builtin_ve_vl_vcvtwdsxrz_vvmvl
#define _vel_vcvtwdzx_vvl __builtin_ve_vl_vcvtwdzx_vvl
#define _vel_vcvtwdzx_vvvl __builtin_ve_vl_vcvtwdzx_vvvl
#define _vel_vcvtwdzx_vvmvl __builtin_ve_vl_vcvtwdzx_vvmvl
#define _vel_vcvtwdzxrz_vvl __builtin_ve_vl_vcvtwdzxrz_vvl
#define _vel_vcvtwdzxrz_vvvl __builtin_ve_vl_vcvtwdzxrz_vvvl
#define _vel_vcvtwdzxrz_vvmvl __builtin_ve_vl_vcvtwdzxrz_vvmvl
#define _vel_vcvtwssx_vvl __builtin_ve_vl_vcvtwssx_vvl
#define _vel_vcvtwssx_vvvl __builtin_ve_vl_vcvtwssx_vvvl
````
- **L769 EN**: Defines macro `_vel_vrsqrts_vvl` for conditional compilation, shorthand, or API generation.
  **L769 CN**: 定义宏 `_vel_vrsqrts_vvl`，用于条件编译、简写或 API 生成。
- **L770 EN**: Defines macro `_vel_vrsqrts_vvvl` for conditional compilation, shorthand, or API generation.
  **L770 CN**: 定义宏 `_vel_vrsqrts_vvvl`，用于条件编译、简写或 API 生成。
- **L771 EN**: Defines macro `_vel_pvrsqrt_vvl` for conditional compilation, shorthand, or API generation.
  **L771 CN**: 定义宏 `_vel_pvrsqrt_vvl`，用于条件编译、简写或 API 生成。
- **L772 EN**: Defines macro `_vel_pvrsqrt_vvvl` for conditional compilation, shorthand, or API generation.
  **L772 CN**: 定义宏 `_vel_pvrsqrt_vvvl`，用于条件编译、简写或 API 生成。
- **L773 EN**: Defines macro `_vel_vrsqrtdnex_vvl` for conditional compilation, shorthand, or API generation.
  **L773 CN**: 定义宏 `_vel_vrsqrtdnex_vvl`，用于条件编译、简写或 API 生成。
- **L774 EN**: Defines macro `_vel_vrsqrtdnex_vvvl` for conditional compilation, shorthand, or API generation.
  **L774 CN**: 定义宏 `_vel_vrsqrtdnex_vvvl`，用于条件编译、简写或 API 生成。
- **L775 EN**: Defines macro `_vel_vrsqrtsnex_vvl` for conditional compilation, shorthand, or API generation.
  **L775 CN**: 定义宏 `_vel_vrsqrtsnex_vvl`，用于条件编译、简写或 API 生成。
- **L776 EN**: Defines macro `_vel_vrsqrtsnex_vvvl` for conditional compilation, shorthand, or API generation.
  **L776 CN**: 定义宏 `_vel_vrsqrtsnex_vvvl`，用于条件编译、简写或 API 生成。
- **L777 EN**: Defines macro `_vel_pvrsqrtnex_vvl` for conditional compilation, shorthand, or API generation.
  **L777 CN**: 定义宏 `_vel_pvrsqrtnex_vvl`，用于条件编译、简写或 API 生成。
- **L778 EN**: Defines macro `_vel_pvrsqrtnex_vvvl` for conditional compilation, shorthand, or API generation.
  **L778 CN**: 定义宏 `_vel_pvrsqrtnex_vvvl`，用于条件编译、简写或 API 生成。
- **L779 EN**: Defines macro `_vel_vcvtwdsx_vvl` for conditional compilation, shorthand, or API generation.
  **L779 CN**: 定义宏 `_vel_vcvtwdsx_vvl`，用于条件编译、简写或 API 生成。
- **L780 EN**: Defines macro `_vel_vcvtwdsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L780 CN**: 定义宏 `_vel_vcvtwdsx_vvvl`，用于条件编译、简写或 API 生成。
- **L781 EN**: Defines macro `_vel_vcvtwdsx_vvmvl` for conditional compilation, shorthand, or API generation.
  **L781 CN**: 定义宏 `_vel_vcvtwdsx_vvmvl`，用于条件编译、简写或 API 生成。
- **L782 EN**: Defines macro `_vel_vcvtwdsxrz_vvl` for conditional compilation, shorthand, or API generation.
  **L782 CN**: 定义宏 `_vel_vcvtwdsxrz_vvl`，用于条件编译、简写或 API 生成。
- **L783 EN**: Defines macro `_vel_vcvtwdsxrz_vvvl` for conditional compilation, shorthand, or API generation.
  **L783 CN**: 定义宏 `_vel_vcvtwdsxrz_vvvl`，用于条件编译、简写或 API 生成。
- **L784 EN**: Defines macro `_vel_vcvtwdsxrz_vvmvl` for conditional compilation, shorthand, or API generation.
  **L784 CN**: 定义宏 `_vel_vcvtwdsxrz_vvmvl`，用于条件编译、简写或 API 生成。
- **L785 EN**: Defines macro `_vel_vcvtwdzx_vvl` for conditional compilation, shorthand, or API generation.
  **L785 CN**: 定义宏 `_vel_vcvtwdzx_vvl`，用于条件编译、简写或 API 生成。
- **L786 EN**: Defines macro `_vel_vcvtwdzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L786 CN**: 定义宏 `_vel_vcvtwdzx_vvvl`，用于条件编译、简写或 API 生成。
- **L787 EN**: Defines macro `_vel_vcvtwdzx_vvmvl` for conditional compilation, shorthand, or API generation.
  **L787 CN**: 定义宏 `_vel_vcvtwdzx_vvmvl`，用于条件编译、简写或 API 生成。
- **L788 EN**: Defines macro `_vel_vcvtwdzxrz_vvl` for conditional compilation, shorthand, or API generation.
  **L788 CN**: 定义宏 `_vel_vcvtwdzxrz_vvl`，用于条件编译、简写或 API 生成。
- **L789 EN**: Defines macro `_vel_vcvtwdzxrz_vvvl` for conditional compilation, shorthand, or API generation.
  **L789 CN**: 定义宏 `_vel_vcvtwdzxrz_vvvl`，用于条件编译、简写或 API 生成。
- **L790 EN**: Defines macro `_vel_vcvtwdzxrz_vvmvl` for conditional compilation, shorthand, or API generation.
  **L790 CN**: 定义宏 `_vel_vcvtwdzxrz_vvmvl`，用于条件编译、简写或 API 生成。
- **L791 EN**: Defines macro `_vel_vcvtwssx_vvl` for conditional compilation, shorthand, or API generation.
  **L791 CN**: 定义宏 `_vel_vcvtwssx_vvl`，用于条件编译、简写或 API 生成。
- **L792 EN**: Defines macro `_vel_vcvtwssx_vvvl` for conditional compilation, shorthand, or API generation.
  **L792 CN**: 定义宏 `_vel_vcvtwssx_vvvl`，用于条件编译、简写或 API 生成。

### Lines 793-816

````c
#define _vel_vcvtwssx_vvmvl __builtin_ve_vl_vcvtwssx_vvmvl
#define _vel_vcvtwssxrz_vvl __builtin_ve_vl_vcvtwssxrz_vvl
#define _vel_vcvtwssxrz_vvvl __builtin_ve_vl_vcvtwssxrz_vvvl
#define _vel_vcvtwssxrz_vvmvl __builtin_ve_vl_vcvtwssxrz_vvmvl
#define _vel_vcvtwszx_vvl __builtin_ve_vl_vcvtwszx_vvl
#define _vel_vcvtwszx_vvvl __builtin_ve_vl_vcvtwszx_vvvl
#define _vel_vcvtwszx_vvmvl __builtin_ve_vl_vcvtwszx_vvmvl
#define _vel_vcvtwszxrz_vvl __builtin_ve_vl_vcvtwszxrz_vvl
#define _vel_vcvtwszxrz_vvvl __builtin_ve_vl_vcvtwszxrz_vvvl
#define _vel_vcvtwszxrz_vvmvl __builtin_ve_vl_vcvtwszxrz_vvmvl
#define _vel_pvcvtws_vvl __builtin_ve_vl_pvcvtws_vvl
#define _vel_pvcvtws_vvvl __builtin_ve_vl_pvcvtws_vvvl
#define _vel_pvcvtws_vvMvl __builtin_ve_vl_pvcvtws_vvMvl
#define _vel_pvcvtwsrz_vvl __builtin_ve_vl_pvcvtwsrz_vvl
#define _vel_pvcvtwsrz_vvvl __builtin_ve_vl_pvcvtwsrz_vvvl
#define _vel_pvcvtwsrz_vvMvl __builtin_ve_vl_pvcvtwsrz_vvMvl
#define _vel_vcvtld_vvl __builtin_ve_vl_vcvtld_vvl
#define _vel_vcvtld_vvvl __builtin_ve_vl_vcvtld_vvvl
#define _vel_vcvtld_vvmvl __builtin_ve_vl_vcvtld_vvmvl
#define _vel_vcvtldrz_vvl __builtin_ve_vl_vcvtldrz_vvl
#define _vel_vcvtldrz_vvvl __builtin_ve_vl_vcvtldrz_vvvl
#define _vel_vcvtldrz_vvmvl __builtin_ve_vl_vcvtldrz_vvmvl
#define _vel_vcvtdw_vvl __builtin_ve_vl_vcvtdw_vvl
#define _vel_vcvtdw_vvvl __builtin_ve_vl_vcvtdw_vvvl
````
- **L793 EN**: Defines macro `_vel_vcvtwssx_vvmvl` for conditional compilation, shorthand, or API generation.
  **L793 CN**: 定义宏 `_vel_vcvtwssx_vvmvl`，用于条件编译、简写或 API 生成。
- **L794 EN**: Defines macro `_vel_vcvtwssxrz_vvl` for conditional compilation, shorthand, or API generation.
  **L794 CN**: 定义宏 `_vel_vcvtwssxrz_vvl`，用于条件编译、简写或 API 生成。
- **L795 EN**: Defines macro `_vel_vcvtwssxrz_vvvl` for conditional compilation, shorthand, or API generation.
  **L795 CN**: 定义宏 `_vel_vcvtwssxrz_vvvl`，用于条件编译、简写或 API 生成。
- **L796 EN**: Defines macro `_vel_vcvtwssxrz_vvmvl` for conditional compilation, shorthand, or API generation.
  **L796 CN**: 定义宏 `_vel_vcvtwssxrz_vvmvl`，用于条件编译、简写或 API 生成。
- **L797 EN**: Defines macro `_vel_vcvtwszx_vvl` for conditional compilation, shorthand, or API generation.
  **L797 CN**: 定义宏 `_vel_vcvtwszx_vvl`，用于条件编译、简写或 API 生成。
- **L798 EN**: Defines macro `_vel_vcvtwszx_vvvl` for conditional compilation, shorthand, or API generation.
  **L798 CN**: 定义宏 `_vel_vcvtwszx_vvvl`，用于条件编译、简写或 API 生成。
- **L799 EN**: Defines macro `_vel_vcvtwszx_vvmvl` for conditional compilation, shorthand, or API generation.
  **L799 CN**: 定义宏 `_vel_vcvtwszx_vvmvl`，用于条件编译、简写或 API 生成。
- **L800 EN**: Defines macro `_vel_vcvtwszxrz_vvl` for conditional compilation, shorthand, or API generation.
  **L800 CN**: 定义宏 `_vel_vcvtwszxrz_vvl`，用于条件编译、简写或 API 生成。
- **L801 EN**: Defines macro `_vel_vcvtwszxrz_vvvl` for conditional compilation, shorthand, or API generation.
  **L801 CN**: 定义宏 `_vel_vcvtwszxrz_vvvl`，用于条件编译、简写或 API 生成。
- **L802 EN**: Defines macro `_vel_vcvtwszxrz_vvmvl` for conditional compilation, shorthand, or API generation.
  **L802 CN**: 定义宏 `_vel_vcvtwszxrz_vvmvl`，用于条件编译、简写或 API 生成。
- **L803 EN**: Defines macro `_vel_pvcvtws_vvl` for conditional compilation, shorthand, or API generation.
  **L803 CN**: 定义宏 `_vel_pvcvtws_vvl`，用于条件编译、简写或 API 生成。
- **L804 EN**: Defines macro `_vel_pvcvtws_vvvl` for conditional compilation, shorthand, or API generation.
  **L804 CN**: 定义宏 `_vel_pvcvtws_vvvl`，用于条件编译、简写或 API 生成。
- **L805 EN**: Defines macro `_vel_pvcvtws_vvMvl` for conditional compilation, shorthand, or API generation.
  **L805 CN**: 定义宏 `_vel_pvcvtws_vvMvl`，用于条件编译、简写或 API 生成。
- **L806 EN**: Defines macro `_vel_pvcvtwsrz_vvl` for conditional compilation, shorthand, or API generation.
  **L806 CN**: 定义宏 `_vel_pvcvtwsrz_vvl`，用于条件编译、简写或 API 生成。
- **L807 EN**: Defines macro `_vel_pvcvtwsrz_vvvl` for conditional compilation, shorthand, or API generation.
  **L807 CN**: 定义宏 `_vel_pvcvtwsrz_vvvl`，用于条件编译、简写或 API 生成。
- **L808 EN**: Defines macro `_vel_pvcvtwsrz_vvMvl` for conditional compilation, shorthand, or API generation.
  **L808 CN**: 定义宏 `_vel_pvcvtwsrz_vvMvl`，用于条件编译、简写或 API 生成。
- **L809 EN**: Defines macro `_vel_vcvtld_vvl` for conditional compilation, shorthand, or API generation.
  **L809 CN**: 定义宏 `_vel_vcvtld_vvl`，用于条件编译、简写或 API 生成。
- **L810 EN**: Defines macro `_vel_vcvtld_vvvl` for conditional compilation, shorthand, or API generation.
  **L810 CN**: 定义宏 `_vel_vcvtld_vvvl`，用于条件编译、简写或 API 生成。
- **L811 EN**: Defines macro `_vel_vcvtld_vvmvl` for conditional compilation, shorthand, or API generation.
  **L811 CN**: 定义宏 `_vel_vcvtld_vvmvl`，用于条件编译、简写或 API 生成。
- **L812 EN**: Defines macro `_vel_vcvtldrz_vvl` for conditional compilation, shorthand, or API generation.
  **L812 CN**: 定义宏 `_vel_vcvtldrz_vvl`，用于条件编译、简写或 API 生成。
- **L813 EN**: Defines macro `_vel_vcvtldrz_vvvl` for conditional compilation, shorthand, or API generation.
  **L813 CN**: 定义宏 `_vel_vcvtldrz_vvvl`，用于条件编译、简写或 API 生成。
- **L814 EN**: Defines macro `_vel_vcvtldrz_vvmvl` for conditional compilation, shorthand, or API generation.
  **L814 CN**: 定义宏 `_vel_vcvtldrz_vvmvl`，用于条件编译、简写或 API 生成。
- **L815 EN**: Defines macro `_vel_vcvtdw_vvl` for conditional compilation, shorthand, or API generation.
  **L815 CN**: 定义宏 `_vel_vcvtdw_vvl`，用于条件编译、简写或 API 生成。
- **L816 EN**: Defines macro `_vel_vcvtdw_vvvl` for conditional compilation, shorthand, or API generation.
  **L816 CN**: 定义宏 `_vel_vcvtdw_vvvl`，用于条件编译、简写或 API 生成。

### Lines 817-840

````c
#define _vel_vcvtsw_vvl __builtin_ve_vl_vcvtsw_vvl
#define _vel_vcvtsw_vvvl __builtin_ve_vl_vcvtsw_vvvl
#define _vel_pvcvtsw_vvl __builtin_ve_vl_pvcvtsw_vvl
#define _vel_pvcvtsw_vvvl __builtin_ve_vl_pvcvtsw_vvvl
#define _vel_vcvtdl_vvl __builtin_ve_vl_vcvtdl_vvl
#define _vel_vcvtdl_vvvl __builtin_ve_vl_vcvtdl_vvvl
#define _vel_vcvtds_vvl __builtin_ve_vl_vcvtds_vvl
#define _vel_vcvtds_vvvl __builtin_ve_vl_vcvtds_vvvl
#define _vel_vcvtsd_vvl __builtin_ve_vl_vcvtsd_vvl
#define _vel_vcvtsd_vvvl __builtin_ve_vl_vcvtsd_vvvl
#define _vel_vmrg_vvvml __builtin_ve_vl_vmrg_vvvml
#define _vel_vmrg_vvvmvl __builtin_ve_vl_vmrg_vvvmvl
#define _vel_vmrg_vsvml __builtin_ve_vl_vmrg_vsvml
#define _vel_vmrg_vsvmvl __builtin_ve_vl_vmrg_vsvmvl
#define _vel_vmrgw_vvvMl __builtin_ve_vl_vmrgw_vvvMl
#define _vel_vmrgw_vvvMvl __builtin_ve_vl_vmrgw_vvvMvl
#define _vel_vmrgw_vsvMl __builtin_ve_vl_vmrgw_vsvMl
#define _vel_vmrgw_vsvMvl __builtin_ve_vl_vmrgw_vsvMvl
#define _vel_vshf_vvvsl __builtin_ve_vl_vshf_vvvsl
#define _vel_vshf_vvvsvl __builtin_ve_vl_vshf_vvvsvl
#define _vel_vcp_vvmvl __builtin_ve_vl_vcp_vvmvl
#define _vel_vex_vvmvl __builtin_ve_vl_vex_vvmvl
#define _vel_vfmklat_ml __builtin_ve_vl_vfmklat_ml
#define _vel_vfmklaf_ml __builtin_ve_vl_vfmklaf_ml
````
- **L817 EN**: Defines macro `_vel_vcvtsw_vvl` for conditional compilation, shorthand, or API generation.
  **L817 CN**: 定义宏 `_vel_vcvtsw_vvl`，用于条件编译、简写或 API 生成。
- **L818 EN**: Defines macro `_vel_vcvtsw_vvvl` for conditional compilation, shorthand, or API generation.
  **L818 CN**: 定义宏 `_vel_vcvtsw_vvvl`，用于条件编译、简写或 API 生成。
- **L819 EN**: Defines macro `_vel_pvcvtsw_vvl` for conditional compilation, shorthand, or API generation.
  **L819 CN**: 定义宏 `_vel_pvcvtsw_vvl`，用于条件编译、简写或 API 生成。
- **L820 EN**: Defines macro `_vel_pvcvtsw_vvvl` for conditional compilation, shorthand, or API generation.
  **L820 CN**: 定义宏 `_vel_pvcvtsw_vvvl`，用于条件编译、简写或 API 生成。
- **L821 EN**: Defines macro `_vel_vcvtdl_vvl` for conditional compilation, shorthand, or API generation.
  **L821 CN**: 定义宏 `_vel_vcvtdl_vvl`，用于条件编译、简写或 API 生成。
- **L822 EN**: Defines macro `_vel_vcvtdl_vvvl` for conditional compilation, shorthand, or API generation.
  **L822 CN**: 定义宏 `_vel_vcvtdl_vvvl`，用于条件编译、简写或 API 生成。
- **L823 EN**: Defines macro `_vel_vcvtds_vvl` for conditional compilation, shorthand, or API generation.
  **L823 CN**: 定义宏 `_vel_vcvtds_vvl`，用于条件编译、简写或 API 生成。
- **L824 EN**: Defines macro `_vel_vcvtds_vvvl` for conditional compilation, shorthand, or API generation.
  **L824 CN**: 定义宏 `_vel_vcvtds_vvvl`，用于条件编译、简写或 API 生成。
- **L825 EN**: Defines macro `_vel_vcvtsd_vvl` for conditional compilation, shorthand, or API generation.
  **L825 CN**: 定义宏 `_vel_vcvtsd_vvl`，用于条件编译、简写或 API 生成。
- **L826 EN**: Defines macro `_vel_vcvtsd_vvvl` for conditional compilation, shorthand, or API generation.
  **L826 CN**: 定义宏 `_vel_vcvtsd_vvvl`，用于条件编译、简写或 API 生成。
- **L827 EN**: Defines macro `_vel_vmrg_vvvml` for conditional compilation, shorthand, or API generation.
  **L827 CN**: 定义宏 `_vel_vmrg_vvvml`，用于条件编译、简写或 API 生成。
- **L828 EN**: Defines macro `_vel_vmrg_vvvmvl` for conditional compilation, shorthand, or API generation.
  **L828 CN**: 定义宏 `_vel_vmrg_vvvmvl`，用于条件编译、简写或 API 生成。
- **L829 EN**: Defines macro `_vel_vmrg_vsvml` for conditional compilation, shorthand, or API generation.
  **L829 CN**: 定义宏 `_vel_vmrg_vsvml`，用于条件编译、简写或 API 生成。
- **L830 EN**: Defines macro `_vel_vmrg_vsvmvl` for conditional compilation, shorthand, or API generation.
  **L830 CN**: 定义宏 `_vel_vmrg_vsvmvl`，用于条件编译、简写或 API 生成。
- **L831 EN**: Defines macro `_vel_vmrgw_vvvMl` for conditional compilation, shorthand, or API generation.
  **L831 CN**: 定义宏 `_vel_vmrgw_vvvMl`，用于条件编译、简写或 API 生成。
- **L832 EN**: Defines macro `_vel_vmrgw_vvvMvl` for conditional compilation, shorthand, or API generation.
  **L832 CN**: 定义宏 `_vel_vmrgw_vvvMvl`，用于条件编译、简写或 API 生成。
- **L833 EN**: Defines macro `_vel_vmrgw_vsvMl` for conditional compilation, shorthand, or API generation.
  **L833 CN**: 定义宏 `_vel_vmrgw_vsvMl`，用于条件编译、简写或 API 生成。
- **L834 EN**: Defines macro `_vel_vmrgw_vsvMvl` for conditional compilation, shorthand, or API generation.
  **L834 CN**: 定义宏 `_vel_vmrgw_vsvMvl`，用于条件编译、简写或 API 生成。
- **L835 EN**: Defines macro `_vel_vshf_vvvsl` for conditional compilation, shorthand, or API generation.
  **L835 CN**: 定义宏 `_vel_vshf_vvvsl`，用于条件编译、简写或 API 生成。
- **L836 EN**: Defines macro `_vel_vshf_vvvsvl` for conditional compilation, shorthand, or API generation.
  **L836 CN**: 定义宏 `_vel_vshf_vvvsvl`，用于条件编译、简写或 API 生成。
- **L837 EN**: Defines macro `_vel_vcp_vvmvl` for conditional compilation, shorthand, or API generation.
  **L837 CN**: 定义宏 `_vel_vcp_vvmvl`，用于条件编译、简写或 API 生成。
- **L838 EN**: Defines macro `_vel_vex_vvmvl` for conditional compilation, shorthand, or API generation.
  **L838 CN**: 定义宏 `_vel_vex_vvmvl`，用于条件编译、简写或 API 生成。
- **L839 EN**: Defines macro `_vel_vfmklat_ml` for conditional compilation, shorthand, or API generation.
  **L839 CN**: 定义宏 `_vel_vfmklat_ml`，用于条件编译、简写或 API 生成。
- **L840 EN**: Defines macro `_vel_vfmklaf_ml` for conditional compilation, shorthand, or API generation.
  **L840 CN**: 定义宏 `_vel_vfmklaf_ml`，用于条件编译、简写或 API 生成。

### Lines 841-864

````c
#define _vel_pvfmkat_Ml __builtin_ve_vl_pvfmkat_Ml
#define _vel_pvfmkaf_Ml __builtin_ve_vl_pvfmkaf_Ml
#define _vel_vfmklgt_mvl __builtin_ve_vl_vfmklgt_mvl
#define _vel_vfmklgt_mvml __builtin_ve_vl_vfmklgt_mvml
#define _vel_vfmkllt_mvl __builtin_ve_vl_vfmkllt_mvl
#define _vel_vfmkllt_mvml __builtin_ve_vl_vfmkllt_mvml
#define _vel_vfmklne_mvl __builtin_ve_vl_vfmklne_mvl
#define _vel_vfmklne_mvml __builtin_ve_vl_vfmklne_mvml
#define _vel_vfmkleq_mvl __builtin_ve_vl_vfmkleq_mvl
#define _vel_vfmkleq_mvml __builtin_ve_vl_vfmkleq_mvml
#define _vel_vfmklge_mvl __builtin_ve_vl_vfmklge_mvl
#define _vel_vfmklge_mvml __builtin_ve_vl_vfmklge_mvml
#define _vel_vfmklle_mvl __builtin_ve_vl_vfmklle_mvl
#define _vel_vfmklle_mvml __builtin_ve_vl_vfmklle_mvml
#define _vel_vfmklnum_mvl __builtin_ve_vl_vfmklnum_mvl
#define _vel_vfmklnum_mvml __builtin_ve_vl_vfmklnum_mvml
#define _vel_vfmklnan_mvl __builtin_ve_vl_vfmklnan_mvl
#define _vel_vfmklnan_mvml __builtin_ve_vl_vfmklnan_mvml
#define _vel_vfmklgtnan_mvl __builtin_ve_vl_vfmklgtnan_mvl
#define _vel_vfmklgtnan_mvml __builtin_ve_vl_vfmklgtnan_mvml
#define _vel_vfmklltnan_mvl __builtin_ve_vl_vfmklltnan_mvl
#define _vel_vfmklltnan_mvml __builtin_ve_vl_vfmklltnan_mvml
#define _vel_vfmklnenan_mvl __builtin_ve_vl_vfmklnenan_mvl
#define _vel_vfmklnenan_mvml __builtin_ve_vl_vfmklnenan_mvml
````
- **L841 EN**: Defines macro `_vel_pvfmkat_Ml` for conditional compilation, shorthand, or API generation.
  **L841 CN**: 定义宏 `_vel_pvfmkat_Ml`，用于条件编译、简写或 API 生成。
- **L842 EN**: Defines macro `_vel_pvfmkaf_Ml` for conditional compilation, shorthand, or API generation.
  **L842 CN**: 定义宏 `_vel_pvfmkaf_Ml`，用于条件编译、简写或 API 生成。
- **L843 EN**: Defines macro `_vel_vfmklgt_mvl` for conditional compilation, shorthand, or API generation.
  **L843 CN**: 定义宏 `_vel_vfmklgt_mvl`，用于条件编译、简写或 API 生成。
- **L844 EN**: Defines macro `_vel_vfmklgt_mvml` for conditional compilation, shorthand, or API generation.
  **L844 CN**: 定义宏 `_vel_vfmklgt_mvml`，用于条件编译、简写或 API 生成。
- **L845 EN**: Defines macro `_vel_vfmkllt_mvl` for conditional compilation, shorthand, or API generation.
  **L845 CN**: 定义宏 `_vel_vfmkllt_mvl`，用于条件编译、简写或 API 生成。
- **L846 EN**: Defines macro `_vel_vfmkllt_mvml` for conditional compilation, shorthand, or API generation.
  **L846 CN**: 定义宏 `_vel_vfmkllt_mvml`，用于条件编译、简写或 API 生成。
- **L847 EN**: Defines macro `_vel_vfmklne_mvl` for conditional compilation, shorthand, or API generation.
  **L847 CN**: 定义宏 `_vel_vfmklne_mvl`，用于条件编译、简写或 API 生成。
- **L848 EN**: Defines macro `_vel_vfmklne_mvml` for conditional compilation, shorthand, or API generation.
  **L848 CN**: 定义宏 `_vel_vfmklne_mvml`，用于条件编译、简写或 API 生成。
- **L849 EN**: Defines macro `_vel_vfmkleq_mvl` for conditional compilation, shorthand, or API generation.
  **L849 CN**: 定义宏 `_vel_vfmkleq_mvl`，用于条件编译、简写或 API 生成。
- **L850 EN**: Defines macro `_vel_vfmkleq_mvml` for conditional compilation, shorthand, or API generation.
  **L850 CN**: 定义宏 `_vel_vfmkleq_mvml`，用于条件编译、简写或 API 生成。
- **L851 EN**: Defines macro `_vel_vfmklge_mvl` for conditional compilation, shorthand, or API generation.
  **L851 CN**: 定义宏 `_vel_vfmklge_mvl`，用于条件编译、简写或 API 生成。
- **L852 EN**: Defines macro `_vel_vfmklge_mvml` for conditional compilation, shorthand, or API generation.
  **L852 CN**: 定义宏 `_vel_vfmklge_mvml`，用于条件编译、简写或 API 生成。
- **L853 EN**: Defines macro `_vel_vfmklle_mvl` for conditional compilation, shorthand, or API generation.
  **L853 CN**: 定义宏 `_vel_vfmklle_mvl`，用于条件编译、简写或 API 生成。
- **L854 EN**: Defines macro `_vel_vfmklle_mvml` for conditional compilation, shorthand, or API generation.
  **L854 CN**: 定义宏 `_vel_vfmklle_mvml`，用于条件编译、简写或 API 生成。
- **L855 EN**: Defines macro `_vel_vfmklnum_mvl` for conditional compilation, shorthand, or API generation.
  **L855 CN**: 定义宏 `_vel_vfmklnum_mvl`，用于条件编译、简写或 API 生成。
- **L856 EN**: Defines macro `_vel_vfmklnum_mvml` for conditional compilation, shorthand, or API generation.
  **L856 CN**: 定义宏 `_vel_vfmklnum_mvml`，用于条件编译、简写或 API 生成。
- **L857 EN**: Defines macro `_vel_vfmklnan_mvl` for conditional compilation, shorthand, or API generation.
  **L857 CN**: 定义宏 `_vel_vfmklnan_mvl`，用于条件编译、简写或 API 生成。
- **L858 EN**: Defines macro `_vel_vfmklnan_mvml` for conditional compilation, shorthand, or API generation.
  **L858 CN**: 定义宏 `_vel_vfmklnan_mvml`，用于条件编译、简写或 API 生成。
- **L859 EN**: Defines macro `_vel_vfmklgtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L859 CN**: 定义宏 `_vel_vfmklgtnan_mvl`，用于条件编译、简写或 API 生成。
- **L860 EN**: Defines macro `_vel_vfmklgtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L860 CN**: 定义宏 `_vel_vfmklgtnan_mvml`，用于条件编译、简写或 API 生成。
- **L861 EN**: Defines macro `_vel_vfmklltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L861 CN**: 定义宏 `_vel_vfmklltnan_mvl`，用于条件编译、简写或 API 生成。
- **L862 EN**: Defines macro `_vel_vfmklltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L862 CN**: 定义宏 `_vel_vfmklltnan_mvml`，用于条件编译、简写或 API 生成。
- **L863 EN**: Defines macro `_vel_vfmklnenan_mvl` for conditional compilation, shorthand, or API generation.
  **L863 CN**: 定义宏 `_vel_vfmklnenan_mvl`，用于条件编译、简写或 API 生成。
- **L864 EN**: Defines macro `_vel_vfmklnenan_mvml` for conditional compilation, shorthand, or API generation.
  **L864 CN**: 定义宏 `_vel_vfmklnenan_mvml`，用于条件编译、简写或 API 生成。

### Lines 865-888

````c
#define _vel_vfmkleqnan_mvl __builtin_ve_vl_vfmkleqnan_mvl
#define _vel_vfmkleqnan_mvml __builtin_ve_vl_vfmkleqnan_mvml
#define _vel_vfmklgenan_mvl __builtin_ve_vl_vfmklgenan_mvl
#define _vel_vfmklgenan_mvml __builtin_ve_vl_vfmklgenan_mvml
#define _vel_vfmkllenan_mvl __builtin_ve_vl_vfmkllenan_mvl
#define _vel_vfmkllenan_mvml __builtin_ve_vl_vfmkllenan_mvml
#define _vel_vfmkwgt_mvl __builtin_ve_vl_vfmkwgt_mvl
#define _vel_vfmkwgt_mvml __builtin_ve_vl_vfmkwgt_mvml
#define _vel_vfmkwlt_mvl __builtin_ve_vl_vfmkwlt_mvl
#define _vel_vfmkwlt_mvml __builtin_ve_vl_vfmkwlt_mvml
#define _vel_vfmkwne_mvl __builtin_ve_vl_vfmkwne_mvl
#define _vel_vfmkwne_mvml __builtin_ve_vl_vfmkwne_mvml
#define _vel_vfmkweq_mvl __builtin_ve_vl_vfmkweq_mvl
#define _vel_vfmkweq_mvml __builtin_ve_vl_vfmkweq_mvml
#define _vel_vfmkwge_mvl __builtin_ve_vl_vfmkwge_mvl
#define _vel_vfmkwge_mvml __builtin_ve_vl_vfmkwge_mvml
#define _vel_vfmkwle_mvl __builtin_ve_vl_vfmkwle_mvl
#define _vel_vfmkwle_mvml __builtin_ve_vl_vfmkwle_mvml
#define _vel_vfmkwnum_mvl __builtin_ve_vl_vfmkwnum_mvl
#define _vel_vfmkwnum_mvml __builtin_ve_vl_vfmkwnum_mvml
#define _vel_vfmkwnan_mvl __builtin_ve_vl_vfmkwnan_mvl
#define _vel_vfmkwnan_mvml __builtin_ve_vl_vfmkwnan_mvml
#define _vel_vfmkwgtnan_mvl __builtin_ve_vl_vfmkwgtnan_mvl
#define _vel_vfmkwgtnan_mvml __builtin_ve_vl_vfmkwgtnan_mvml
````
- **L865 EN**: Defines macro `_vel_vfmkleqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L865 CN**: 定义宏 `_vel_vfmkleqnan_mvl`，用于条件编译、简写或 API 生成。
- **L866 EN**: Defines macro `_vel_vfmkleqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L866 CN**: 定义宏 `_vel_vfmkleqnan_mvml`，用于条件编译、简写或 API 生成。
- **L867 EN**: Defines macro `_vel_vfmklgenan_mvl` for conditional compilation, shorthand, or API generation.
  **L867 CN**: 定义宏 `_vel_vfmklgenan_mvl`，用于条件编译、简写或 API 生成。
- **L868 EN**: Defines macro `_vel_vfmklgenan_mvml` for conditional compilation, shorthand, or API generation.
  **L868 CN**: 定义宏 `_vel_vfmklgenan_mvml`，用于条件编译、简写或 API 生成。
- **L869 EN**: Defines macro `_vel_vfmkllenan_mvl` for conditional compilation, shorthand, or API generation.
  **L869 CN**: 定义宏 `_vel_vfmkllenan_mvl`，用于条件编译、简写或 API 生成。
- **L870 EN**: Defines macro `_vel_vfmkllenan_mvml` for conditional compilation, shorthand, or API generation.
  **L870 CN**: 定义宏 `_vel_vfmkllenan_mvml`，用于条件编译、简写或 API 生成。
- **L871 EN**: Defines macro `_vel_vfmkwgt_mvl` for conditional compilation, shorthand, or API generation.
  **L871 CN**: 定义宏 `_vel_vfmkwgt_mvl`，用于条件编译、简写或 API 生成。
- **L872 EN**: Defines macro `_vel_vfmkwgt_mvml` for conditional compilation, shorthand, or API generation.
  **L872 CN**: 定义宏 `_vel_vfmkwgt_mvml`，用于条件编译、简写或 API 生成。
- **L873 EN**: Defines macro `_vel_vfmkwlt_mvl` for conditional compilation, shorthand, or API generation.
  **L873 CN**: 定义宏 `_vel_vfmkwlt_mvl`，用于条件编译、简写或 API 生成。
- **L874 EN**: Defines macro `_vel_vfmkwlt_mvml` for conditional compilation, shorthand, or API generation.
  **L874 CN**: 定义宏 `_vel_vfmkwlt_mvml`，用于条件编译、简写或 API 生成。
- **L875 EN**: Defines macro `_vel_vfmkwne_mvl` for conditional compilation, shorthand, or API generation.
  **L875 CN**: 定义宏 `_vel_vfmkwne_mvl`，用于条件编译、简写或 API 生成。
- **L876 EN**: Defines macro `_vel_vfmkwne_mvml` for conditional compilation, shorthand, or API generation.
  **L876 CN**: 定义宏 `_vel_vfmkwne_mvml`，用于条件编译、简写或 API 生成。
- **L877 EN**: Defines macro `_vel_vfmkweq_mvl` for conditional compilation, shorthand, or API generation.
  **L877 CN**: 定义宏 `_vel_vfmkweq_mvl`，用于条件编译、简写或 API 生成。
- **L878 EN**: Defines macro `_vel_vfmkweq_mvml` for conditional compilation, shorthand, or API generation.
  **L878 CN**: 定义宏 `_vel_vfmkweq_mvml`，用于条件编译、简写或 API 生成。
- **L879 EN**: Defines macro `_vel_vfmkwge_mvl` for conditional compilation, shorthand, or API generation.
  **L879 CN**: 定义宏 `_vel_vfmkwge_mvl`，用于条件编译、简写或 API 生成。
- **L880 EN**: Defines macro `_vel_vfmkwge_mvml` for conditional compilation, shorthand, or API generation.
  **L880 CN**: 定义宏 `_vel_vfmkwge_mvml`，用于条件编译、简写或 API 生成。
- **L881 EN**: Defines macro `_vel_vfmkwle_mvl` for conditional compilation, shorthand, or API generation.
  **L881 CN**: 定义宏 `_vel_vfmkwle_mvl`，用于条件编译、简写或 API 生成。
- **L882 EN**: Defines macro `_vel_vfmkwle_mvml` for conditional compilation, shorthand, or API generation.
  **L882 CN**: 定义宏 `_vel_vfmkwle_mvml`，用于条件编译、简写或 API 生成。
- **L883 EN**: Defines macro `_vel_vfmkwnum_mvl` for conditional compilation, shorthand, or API generation.
  **L883 CN**: 定义宏 `_vel_vfmkwnum_mvl`，用于条件编译、简写或 API 生成。
- **L884 EN**: Defines macro `_vel_vfmkwnum_mvml` for conditional compilation, shorthand, or API generation.
  **L884 CN**: 定义宏 `_vel_vfmkwnum_mvml`，用于条件编译、简写或 API 生成。
- **L885 EN**: Defines macro `_vel_vfmkwnan_mvl` for conditional compilation, shorthand, or API generation.
  **L885 CN**: 定义宏 `_vel_vfmkwnan_mvl`，用于条件编译、简写或 API 生成。
- **L886 EN**: Defines macro `_vel_vfmkwnan_mvml` for conditional compilation, shorthand, or API generation.
  **L886 CN**: 定义宏 `_vel_vfmkwnan_mvml`，用于条件编译、简写或 API 生成。
- **L887 EN**: Defines macro `_vel_vfmkwgtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L887 CN**: 定义宏 `_vel_vfmkwgtnan_mvl`，用于条件编译、简写或 API 生成。
- **L888 EN**: Defines macro `_vel_vfmkwgtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L888 CN**: 定义宏 `_vel_vfmkwgtnan_mvml`，用于条件编译、简写或 API 生成。

### Lines 889-912

````c
#define _vel_vfmkwltnan_mvl __builtin_ve_vl_vfmkwltnan_mvl
#define _vel_vfmkwltnan_mvml __builtin_ve_vl_vfmkwltnan_mvml
#define _vel_vfmkwnenan_mvl __builtin_ve_vl_vfmkwnenan_mvl
#define _vel_vfmkwnenan_mvml __builtin_ve_vl_vfmkwnenan_mvml
#define _vel_vfmkweqnan_mvl __builtin_ve_vl_vfmkweqnan_mvl
#define _vel_vfmkweqnan_mvml __builtin_ve_vl_vfmkweqnan_mvml
#define _vel_vfmkwgenan_mvl __builtin_ve_vl_vfmkwgenan_mvl
#define _vel_vfmkwgenan_mvml __builtin_ve_vl_vfmkwgenan_mvml
#define _vel_vfmkwlenan_mvl __builtin_ve_vl_vfmkwlenan_mvl
#define _vel_vfmkwlenan_mvml __builtin_ve_vl_vfmkwlenan_mvml
#define _vel_pvfmkwlogt_mvl __builtin_ve_vl_pvfmkwlogt_mvl
#define _vel_pvfmkwupgt_mvl __builtin_ve_vl_pvfmkwupgt_mvl
#define _vel_pvfmkwlogt_mvml __builtin_ve_vl_pvfmkwlogt_mvml
#define _vel_pvfmkwupgt_mvml __builtin_ve_vl_pvfmkwupgt_mvml
#define _vel_pvfmkwlolt_mvl __builtin_ve_vl_pvfmkwlolt_mvl
#define _vel_pvfmkwuplt_mvl __builtin_ve_vl_pvfmkwuplt_mvl
#define _vel_pvfmkwlolt_mvml __builtin_ve_vl_pvfmkwlolt_mvml
#define _vel_pvfmkwuplt_mvml __builtin_ve_vl_pvfmkwuplt_mvml
#define _vel_pvfmkwlone_mvl __builtin_ve_vl_pvfmkwlone_mvl
#define _vel_pvfmkwupne_mvl __builtin_ve_vl_pvfmkwupne_mvl
#define _vel_pvfmkwlone_mvml __builtin_ve_vl_pvfmkwlone_mvml
#define _vel_pvfmkwupne_mvml __builtin_ve_vl_pvfmkwupne_mvml
#define _vel_pvfmkwloeq_mvl __builtin_ve_vl_pvfmkwloeq_mvl
#define _vel_pvfmkwupeq_mvl __builtin_ve_vl_pvfmkwupeq_mvl
````
- **L889 EN**: Defines macro `_vel_vfmkwltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L889 CN**: 定义宏 `_vel_vfmkwltnan_mvl`，用于条件编译、简写或 API 生成。
- **L890 EN**: Defines macro `_vel_vfmkwltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L890 CN**: 定义宏 `_vel_vfmkwltnan_mvml`，用于条件编译、简写或 API 生成。
- **L891 EN**: Defines macro `_vel_vfmkwnenan_mvl` for conditional compilation, shorthand, or API generation.
  **L891 CN**: 定义宏 `_vel_vfmkwnenan_mvl`，用于条件编译、简写或 API 生成。
- **L892 EN**: Defines macro `_vel_vfmkwnenan_mvml` for conditional compilation, shorthand, or API generation.
  **L892 CN**: 定义宏 `_vel_vfmkwnenan_mvml`，用于条件编译、简写或 API 生成。
- **L893 EN**: Defines macro `_vel_vfmkweqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L893 CN**: 定义宏 `_vel_vfmkweqnan_mvl`，用于条件编译、简写或 API 生成。
- **L894 EN**: Defines macro `_vel_vfmkweqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L894 CN**: 定义宏 `_vel_vfmkweqnan_mvml`，用于条件编译、简写或 API 生成。
- **L895 EN**: Defines macro `_vel_vfmkwgenan_mvl` for conditional compilation, shorthand, or API generation.
  **L895 CN**: 定义宏 `_vel_vfmkwgenan_mvl`，用于条件编译、简写或 API 生成。
- **L896 EN**: Defines macro `_vel_vfmkwgenan_mvml` for conditional compilation, shorthand, or API generation.
  **L896 CN**: 定义宏 `_vel_vfmkwgenan_mvml`，用于条件编译、简写或 API 生成。
- **L897 EN**: Defines macro `_vel_vfmkwlenan_mvl` for conditional compilation, shorthand, or API generation.
  **L897 CN**: 定义宏 `_vel_vfmkwlenan_mvl`，用于条件编译、简写或 API 生成。
- **L898 EN**: Defines macro `_vel_vfmkwlenan_mvml` for conditional compilation, shorthand, or API generation.
  **L898 CN**: 定义宏 `_vel_vfmkwlenan_mvml`，用于条件编译、简写或 API 生成。
- **L899 EN**: Defines macro `_vel_pvfmkwlogt_mvl` for conditional compilation, shorthand, or API generation.
  **L899 CN**: 定义宏 `_vel_pvfmkwlogt_mvl`，用于条件编译、简写或 API 生成。
- **L900 EN**: Defines macro `_vel_pvfmkwupgt_mvl` for conditional compilation, shorthand, or API generation.
  **L900 CN**: 定义宏 `_vel_pvfmkwupgt_mvl`，用于条件编译、简写或 API 生成。
- **L901 EN**: Defines macro `_vel_pvfmkwlogt_mvml` for conditional compilation, shorthand, or API generation.
  **L901 CN**: 定义宏 `_vel_pvfmkwlogt_mvml`，用于条件编译、简写或 API 生成。
- **L902 EN**: Defines macro `_vel_pvfmkwupgt_mvml` for conditional compilation, shorthand, or API generation.
  **L902 CN**: 定义宏 `_vel_pvfmkwupgt_mvml`，用于条件编译、简写或 API 生成。
- **L903 EN**: Defines macro `_vel_pvfmkwlolt_mvl` for conditional compilation, shorthand, or API generation.
  **L903 CN**: 定义宏 `_vel_pvfmkwlolt_mvl`，用于条件编译、简写或 API 生成。
- **L904 EN**: Defines macro `_vel_pvfmkwuplt_mvl` for conditional compilation, shorthand, or API generation.
  **L904 CN**: 定义宏 `_vel_pvfmkwuplt_mvl`，用于条件编译、简写或 API 生成。
- **L905 EN**: Defines macro `_vel_pvfmkwlolt_mvml` for conditional compilation, shorthand, or API generation.
  **L905 CN**: 定义宏 `_vel_pvfmkwlolt_mvml`，用于条件编译、简写或 API 生成。
- **L906 EN**: Defines macro `_vel_pvfmkwuplt_mvml` for conditional compilation, shorthand, or API generation.
  **L906 CN**: 定义宏 `_vel_pvfmkwuplt_mvml`，用于条件编译、简写或 API 生成。
- **L907 EN**: Defines macro `_vel_pvfmkwlone_mvl` for conditional compilation, shorthand, or API generation.
  **L907 CN**: 定义宏 `_vel_pvfmkwlone_mvl`，用于条件编译、简写或 API 生成。
- **L908 EN**: Defines macro `_vel_pvfmkwupne_mvl` for conditional compilation, shorthand, or API generation.
  **L908 CN**: 定义宏 `_vel_pvfmkwupne_mvl`，用于条件编译、简写或 API 生成。
- **L909 EN**: Defines macro `_vel_pvfmkwlone_mvml` for conditional compilation, shorthand, or API generation.
  **L909 CN**: 定义宏 `_vel_pvfmkwlone_mvml`，用于条件编译、简写或 API 生成。
- **L910 EN**: Defines macro `_vel_pvfmkwupne_mvml` for conditional compilation, shorthand, or API generation.
  **L910 CN**: 定义宏 `_vel_pvfmkwupne_mvml`，用于条件编译、简写或 API 生成。
- **L911 EN**: Defines macro `_vel_pvfmkwloeq_mvl` for conditional compilation, shorthand, or API generation.
  **L911 CN**: 定义宏 `_vel_pvfmkwloeq_mvl`，用于条件编译、简写或 API 生成。
- **L912 EN**: Defines macro `_vel_pvfmkwupeq_mvl` for conditional compilation, shorthand, or API generation.
  **L912 CN**: 定义宏 `_vel_pvfmkwupeq_mvl`，用于条件编译、简写或 API 生成。

### Lines 913-936

````c
#define _vel_pvfmkwloeq_mvml __builtin_ve_vl_pvfmkwloeq_mvml
#define _vel_pvfmkwupeq_mvml __builtin_ve_vl_pvfmkwupeq_mvml
#define _vel_pvfmkwloge_mvl __builtin_ve_vl_pvfmkwloge_mvl
#define _vel_pvfmkwupge_mvl __builtin_ve_vl_pvfmkwupge_mvl
#define _vel_pvfmkwloge_mvml __builtin_ve_vl_pvfmkwloge_mvml
#define _vel_pvfmkwupge_mvml __builtin_ve_vl_pvfmkwupge_mvml
#define _vel_pvfmkwlole_mvl __builtin_ve_vl_pvfmkwlole_mvl
#define _vel_pvfmkwuple_mvl __builtin_ve_vl_pvfmkwuple_mvl
#define _vel_pvfmkwlole_mvml __builtin_ve_vl_pvfmkwlole_mvml
#define _vel_pvfmkwuple_mvml __builtin_ve_vl_pvfmkwuple_mvml
#define _vel_pvfmkwlonum_mvl __builtin_ve_vl_pvfmkwlonum_mvl
#define _vel_pvfmkwupnum_mvl __builtin_ve_vl_pvfmkwupnum_mvl
#define _vel_pvfmkwlonum_mvml __builtin_ve_vl_pvfmkwlonum_mvml
#define _vel_pvfmkwupnum_mvml __builtin_ve_vl_pvfmkwupnum_mvml
#define _vel_pvfmkwlonan_mvl __builtin_ve_vl_pvfmkwlonan_mvl
#define _vel_pvfmkwupnan_mvl __builtin_ve_vl_pvfmkwupnan_mvl
#define _vel_pvfmkwlonan_mvml __builtin_ve_vl_pvfmkwlonan_mvml
#define _vel_pvfmkwupnan_mvml __builtin_ve_vl_pvfmkwupnan_mvml
#define _vel_pvfmkwlogtnan_mvl __builtin_ve_vl_pvfmkwlogtnan_mvl
#define _vel_pvfmkwupgtnan_mvl __builtin_ve_vl_pvfmkwupgtnan_mvl
#define _vel_pvfmkwlogtnan_mvml __builtin_ve_vl_pvfmkwlogtnan_mvml
#define _vel_pvfmkwupgtnan_mvml __builtin_ve_vl_pvfmkwupgtnan_mvml
#define _vel_pvfmkwloltnan_mvl __builtin_ve_vl_pvfmkwloltnan_mvl
#define _vel_pvfmkwupltnan_mvl __builtin_ve_vl_pvfmkwupltnan_mvl
````
- **L913 EN**: Defines macro `_vel_pvfmkwloeq_mvml` for conditional compilation, shorthand, or API generation.
  **L913 CN**: 定义宏 `_vel_pvfmkwloeq_mvml`，用于条件编译、简写或 API 生成。
- **L914 EN**: Defines macro `_vel_pvfmkwupeq_mvml` for conditional compilation, shorthand, or API generation.
  **L914 CN**: 定义宏 `_vel_pvfmkwupeq_mvml`，用于条件编译、简写或 API 生成。
- **L915 EN**: Defines macro `_vel_pvfmkwloge_mvl` for conditional compilation, shorthand, or API generation.
  **L915 CN**: 定义宏 `_vel_pvfmkwloge_mvl`，用于条件编译、简写或 API 生成。
- **L916 EN**: Defines macro `_vel_pvfmkwupge_mvl` for conditional compilation, shorthand, or API generation.
  **L916 CN**: 定义宏 `_vel_pvfmkwupge_mvl`，用于条件编译、简写或 API 生成。
- **L917 EN**: Defines macro `_vel_pvfmkwloge_mvml` for conditional compilation, shorthand, or API generation.
  **L917 CN**: 定义宏 `_vel_pvfmkwloge_mvml`，用于条件编译、简写或 API 生成。
- **L918 EN**: Defines macro `_vel_pvfmkwupge_mvml` for conditional compilation, shorthand, or API generation.
  **L918 CN**: 定义宏 `_vel_pvfmkwupge_mvml`，用于条件编译、简写或 API 生成。
- **L919 EN**: Defines macro `_vel_pvfmkwlole_mvl` for conditional compilation, shorthand, or API generation.
  **L919 CN**: 定义宏 `_vel_pvfmkwlole_mvl`，用于条件编译、简写或 API 生成。
- **L920 EN**: Defines macro `_vel_pvfmkwuple_mvl` for conditional compilation, shorthand, or API generation.
  **L920 CN**: 定义宏 `_vel_pvfmkwuple_mvl`，用于条件编译、简写或 API 生成。
- **L921 EN**: Defines macro `_vel_pvfmkwlole_mvml` for conditional compilation, shorthand, or API generation.
  **L921 CN**: 定义宏 `_vel_pvfmkwlole_mvml`，用于条件编译、简写或 API 生成。
- **L922 EN**: Defines macro `_vel_pvfmkwuple_mvml` for conditional compilation, shorthand, or API generation.
  **L922 CN**: 定义宏 `_vel_pvfmkwuple_mvml`，用于条件编译、简写或 API 生成。
- **L923 EN**: Defines macro `_vel_pvfmkwlonum_mvl` for conditional compilation, shorthand, or API generation.
  **L923 CN**: 定义宏 `_vel_pvfmkwlonum_mvl`，用于条件编译、简写或 API 生成。
- **L924 EN**: Defines macro `_vel_pvfmkwupnum_mvl` for conditional compilation, shorthand, or API generation.
  **L924 CN**: 定义宏 `_vel_pvfmkwupnum_mvl`，用于条件编译、简写或 API 生成。
- **L925 EN**: Defines macro `_vel_pvfmkwlonum_mvml` for conditional compilation, shorthand, or API generation.
  **L925 CN**: 定义宏 `_vel_pvfmkwlonum_mvml`，用于条件编译、简写或 API 生成。
- **L926 EN**: Defines macro `_vel_pvfmkwupnum_mvml` for conditional compilation, shorthand, or API generation.
  **L926 CN**: 定义宏 `_vel_pvfmkwupnum_mvml`，用于条件编译、简写或 API 生成。
- **L927 EN**: Defines macro `_vel_pvfmkwlonan_mvl` for conditional compilation, shorthand, or API generation.
  **L927 CN**: 定义宏 `_vel_pvfmkwlonan_mvl`，用于条件编译、简写或 API 生成。
- **L928 EN**: Defines macro `_vel_pvfmkwupnan_mvl` for conditional compilation, shorthand, or API generation.
  **L928 CN**: 定义宏 `_vel_pvfmkwupnan_mvl`，用于条件编译、简写或 API 生成。
- **L929 EN**: Defines macro `_vel_pvfmkwlonan_mvml` for conditional compilation, shorthand, or API generation.
  **L929 CN**: 定义宏 `_vel_pvfmkwlonan_mvml`，用于条件编译、简写或 API 生成。
- **L930 EN**: Defines macro `_vel_pvfmkwupnan_mvml` for conditional compilation, shorthand, or API generation.
  **L930 CN**: 定义宏 `_vel_pvfmkwupnan_mvml`，用于条件编译、简写或 API 生成。
- **L931 EN**: Defines macro `_vel_pvfmkwlogtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L931 CN**: 定义宏 `_vel_pvfmkwlogtnan_mvl`，用于条件编译、简写或 API 生成。
- **L932 EN**: Defines macro `_vel_pvfmkwupgtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L932 CN**: 定义宏 `_vel_pvfmkwupgtnan_mvl`，用于条件编译、简写或 API 生成。
- **L933 EN**: Defines macro `_vel_pvfmkwlogtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L933 CN**: 定义宏 `_vel_pvfmkwlogtnan_mvml`，用于条件编译、简写或 API 生成。
- **L934 EN**: Defines macro `_vel_pvfmkwupgtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L934 CN**: 定义宏 `_vel_pvfmkwupgtnan_mvml`，用于条件编译、简写或 API 生成。
- **L935 EN**: Defines macro `_vel_pvfmkwloltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L935 CN**: 定义宏 `_vel_pvfmkwloltnan_mvl`，用于条件编译、简写或 API 生成。
- **L936 EN**: Defines macro `_vel_pvfmkwupltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L936 CN**: 定义宏 `_vel_pvfmkwupltnan_mvl`，用于条件编译、简写或 API 生成。

### Lines 937-960

````c
#define _vel_pvfmkwloltnan_mvml __builtin_ve_vl_pvfmkwloltnan_mvml
#define _vel_pvfmkwupltnan_mvml __builtin_ve_vl_pvfmkwupltnan_mvml
#define _vel_pvfmkwlonenan_mvl __builtin_ve_vl_pvfmkwlonenan_mvl
#define _vel_pvfmkwupnenan_mvl __builtin_ve_vl_pvfmkwupnenan_mvl
#define _vel_pvfmkwlonenan_mvml __builtin_ve_vl_pvfmkwlonenan_mvml
#define _vel_pvfmkwupnenan_mvml __builtin_ve_vl_pvfmkwupnenan_mvml
#define _vel_pvfmkwloeqnan_mvl __builtin_ve_vl_pvfmkwloeqnan_mvl
#define _vel_pvfmkwupeqnan_mvl __builtin_ve_vl_pvfmkwupeqnan_mvl
#define _vel_pvfmkwloeqnan_mvml __builtin_ve_vl_pvfmkwloeqnan_mvml
#define _vel_pvfmkwupeqnan_mvml __builtin_ve_vl_pvfmkwupeqnan_mvml
#define _vel_pvfmkwlogenan_mvl __builtin_ve_vl_pvfmkwlogenan_mvl
#define _vel_pvfmkwupgenan_mvl __builtin_ve_vl_pvfmkwupgenan_mvl
#define _vel_pvfmkwlogenan_mvml __builtin_ve_vl_pvfmkwlogenan_mvml
#define _vel_pvfmkwupgenan_mvml __builtin_ve_vl_pvfmkwupgenan_mvml
#define _vel_pvfmkwlolenan_mvl __builtin_ve_vl_pvfmkwlolenan_mvl
#define _vel_pvfmkwuplenan_mvl __builtin_ve_vl_pvfmkwuplenan_mvl
#define _vel_pvfmkwlolenan_mvml __builtin_ve_vl_pvfmkwlolenan_mvml
#define _vel_pvfmkwuplenan_mvml __builtin_ve_vl_pvfmkwuplenan_mvml
#define _vel_pvfmkwgt_Mvl __builtin_ve_vl_pvfmkwgt_Mvl
#define _vel_pvfmkwgt_MvMl __builtin_ve_vl_pvfmkwgt_MvMl
#define _vel_pvfmkwlt_Mvl __builtin_ve_vl_pvfmkwlt_Mvl
#define _vel_pvfmkwlt_MvMl __builtin_ve_vl_pvfmkwlt_MvMl
#define _vel_pvfmkwne_Mvl __builtin_ve_vl_pvfmkwne_Mvl
#define _vel_pvfmkwne_MvMl __builtin_ve_vl_pvfmkwne_MvMl
````
- **L937 EN**: Defines macro `_vel_pvfmkwloltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L937 CN**: 定义宏 `_vel_pvfmkwloltnan_mvml`，用于条件编译、简写或 API 生成。
- **L938 EN**: Defines macro `_vel_pvfmkwupltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L938 CN**: 定义宏 `_vel_pvfmkwupltnan_mvml`，用于条件编译、简写或 API 生成。
- **L939 EN**: Defines macro `_vel_pvfmkwlonenan_mvl` for conditional compilation, shorthand, or API generation.
  **L939 CN**: 定义宏 `_vel_pvfmkwlonenan_mvl`，用于条件编译、简写或 API 生成。
- **L940 EN**: Defines macro `_vel_pvfmkwupnenan_mvl` for conditional compilation, shorthand, or API generation.
  **L940 CN**: 定义宏 `_vel_pvfmkwupnenan_mvl`，用于条件编译、简写或 API 生成。
- **L941 EN**: Defines macro `_vel_pvfmkwlonenan_mvml` for conditional compilation, shorthand, or API generation.
  **L941 CN**: 定义宏 `_vel_pvfmkwlonenan_mvml`，用于条件编译、简写或 API 生成。
- **L942 EN**: Defines macro `_vel_pvfmkwupnenan_mvml` for conditional compilation, shorthand, or API generation.
  **L942 CN**: 定义宏 `_vel_pvfmkwupnenan_mvml`，用于条件编译、简写或 API 生成。
- **L943 EN**: Defines macro `_vel_pvfmkwloeqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L943 CN**: 定义宏 `_vel_pvfmkwloeqnan_mvl`，用于条件编译、简写或 API 生成。
- **L944 EN**: Defines macro `_vel_pvfmkwupeqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L944 CN**: 定义宏 `_vel_pvfmkwupeqnan_mvl`，用于条件编译、简写或 API 生成。
- **L945 EN**: Defines macro `_vel_pvfmkwloeqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L945 CN**: 定义宏 `_vel_pvfmkwloeqnan_mvml`，用于条件编译、简写或 API 生成。
- **L946 EN**: Defines macro `_vel_pvfmkwupeqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L946 CN**: 定义宏 `_vel_pvfmkwupeqnan_mvml`，用于条件编译、简写或 API 生成。
- **L947 EN**: Defines macro `_vel_pvfmkwlogenan_mvl` for conditional compilation, shorthand, or API generation.
  **L947 CN**: 定义宏 `_vel_pvfmkwlogenan_mvl`，用于条件编译、简写或 API 生成。
- **L948 EN**: Defines macro `_vel_pvfmkwupgenan_mvl` for conditional compilation, shorthand, or API generation.
  **L948 CN**: 定义宏 `_vel_pvfmkwupgenan_mvl`，用于条件编译、简写或 API 生成。
- **L949 EN**: Defines macro `_vel_pvfmkwlogenan_mvml` for conditional compilation, shorthand, or API generation.
  **L949 CN**: 定义宏 `_vel_pvfmkwlogenan_mvml`，用于条件编译、简写或 API 生成。
- **L950 EN**: Defines macro `_vel_pvfmkwupgenan_mvml` for conditional compilation, shorthand, or API generation.
  **L950 CN**: 定义宏 `_vel_pvfmkwupgenan_mvml`，用于条件编译、简写或 API 生成。
- **L951 EN**: Defines macro `_vel_pvfmkwlolenan_mvl` for conditional compilation, shorthand, or API generation.
  **L951 CN**: 定义宏 `_vel_pvfmkwlolenan_mvl`，用于条件编译、简写或 API 生成。
- **L952 EN**: Defines macro `_vel_pvfmkwuplenan_mvl` for conditional compilation, shorthand, or API generation.
  **L952 CN**: 定义宏 `_vel_pvfmkwuplenan_mvl`，用于条件编译、简写或 API 生成。
- **L953 EN**: Defines macro `_vel_pvfmkwlolenan_mvml` for conditional compilation, shorthand, or API generation.
  **L953 CN**: 定义宏 `_vel_pvfmkwlolenan_mvml`，用于条件编译、简写或 API 生成。
- **L954 EN**: Defines macro `_vel_pvfmkwuplenan_mvml` for conditional compilation, shorthand, or API generation.
  **L954 CN**: 定义宏 `_vel_pvfmkwuplenan_mvml`，用于条件编译、简写或 API 生成。
- **L955 EN**: Defines macro `_vel_pvfmkwgt_Mvl` for conditional compilation, shorthand, or API generation.
  **L955 CN**: 定义宏 `_vel_pvfmkwgt_Mvl`，用于条件编译、简写或 API 生成。
- **L956 EN**: Defines macro `_vel_pvfmkwgt_MvMl` for conditional compilation, shorthand, or API generation.
  **L956 CN**: 定义宏 `_vel_pvfmkwgt_MvMl`，用于条件编译、简写或 API 生成。
- **L957 EN**: Defines macro `_vel_pvfmkwlt_Mvl` for conditional compilation, shorthand, or API generation.
  **L957 CN**: 定义宏 `_vel_pvfmkwlt_Mvl`，用于条件编译、简写或 API 生成。
- **L958 EN**: Defines macro `_vel_pvfmkwlt_MvMl` for conditional compilation, shorthand, or API generation.
  **L958 CN**: 定义宏 `_vel_pvfmkwlt_MvMl`，用于条件编译、简写或 API 生成。
- **L959 EN**: Defines macro `_vel_pvfmkwne_Mvl` for conditional compilation, shorthand, or API generation.
  **L959 CN**: 定义宏 `_vel_pvfmkwne_Mvl`，用于条件编译、简写或 API 生成。
- **L960 EN**: Defines macro `_vel_pvfmkwne_MvMl` for conditional compilation, shorthand, or API generation.
  **L960 CN**: 定义宏 `_vel_pvfmkwne_MvMl`，用于条件编译、简写或 API 生成。

### Lines 961-984

````c
#define _vel_pvfmkweq_Mvl __builtin_ve_vl_pvfmkweq_Mvl
#define _vel_pvfmkweq_MvMl __builtin_ve_vl_pvfmkweq_MvMl
#define _vel_pvfmkwge_Mvl __builtin_ve_vl_pvfmkwge_Mvl
#define _vel_pvfmkwge_MvMl __builtin_ve_vl_pvfmkwge_MvMl
#define _vel_pvfmkwle_Mvl __builtin_ve_vl_pvfmkwle_Mvl
#define _vel_pvfmkwle_MvMl __builtin_ve_vl_pvfmkwle_MvMl
#define _vel_pvfmkwnum_Mvl __builtin_ve_vl_pvfmkwnum_Mvl
#define _vel_pvfmkwnum_MvMl __builtin_ve_vl_pvfmkwnum_MvMl
#define _vel_pvfmkwnan_Mvl __builtin_ve_vl_pvfmkwnan_Mvl
#define _vel_pvfmkwnan_MvMl __builtin_ve_vl_pvfmkwnan_MvMl
#define _vel_pvfmkwgtnan_Mvl __builtin_ve_vl_pvfmkwgtnan_Mvl
#define _vel_pvfmkwgtnan_MvMl __builtin_ve_vl_pvfmkwgtnan_MvMl
#define _vel_pvfmkwltnan_Mvl __builtin_ve_vl_pvfmkwltnan_Mvl
#define _vel_pvfmkwltnan_MvMl __builtin_ve_vl_pvfmkwltnan_MvMl
#define _vel_pvfmkwnenan_Mvl __builtin_ve_vl_pvfmkwnenan_Mvl
#define _vel_pvfmkwnenan_MvMl __builtin_ve_vl_pvfmkwnenan_MvMl
#define _vel_pvfmkweqnan_Mvl __builtin_ve_vl_pvfmkweqnan_Mvl
#define _vel_pvfmkweqnan_MvMl __builtin_ve_vl_pvfmkweqnan_MvMl
#define _vel_pvfmkwgenan_Mvl __builtin_ve_vl_pvfmkwgenan_Mvl
#define _vel_pvfmkwgenan_MvMl __builtin_ve_vl_pvfmkwgenan_MvMl
#define _vel_pvfmkwlenan_Mvl __builtin_ve_vl_pvfmkwlenan_Mvl
#define _vel_pvfmkwlenan_MvMl __builtin_ve_vl_pvfmkwlenan_MvMl
#define _vel_vfmkdgt_mvl __builtin_ve_vl_vfmkdgt_mvl
#define _vel_vfmkdgt_mvml __builtin_ve_vl_vfmkdgt_mvml
````
- **L961 EN**: Defines macro `_vel_pvfmkweq_Mvl` for conditional compilation, shorthand, or API generation.
  **L961 CN**: 定义宏 `_vel_pvfmkweq_Mvl`，用于条件编译、简写或 API 生成。
- **L962 EN**: Defines macro `_vel_pvfmkweq_MvMl` for conditional compilation, shorthand, or API generation.
  **L962 CN**: 定义宏 `_vel_pvfmkweq_MvMl`，用于条件编译、简写或 API 生成。
- **L963 EN**: Defines macro `_vel_pvfmkwge_Mvl` for conditional compilation, shorthand, or API generation.
  **L963 CN**: 定义宏 `_vel_pvfmkwge_Mvl`，用于条件编译、简写或 API 生成。
- **L964 EN**: Defines macro `_vel_pvfmkwge_MvMl` for conditional compilation, shorthand, or API generation.
  **L964 CN**: 定义宏 `_vel_pvfmkwge_MvMl`，用于条件编译、简写或 API 生成。
- **L965 EN**: Defines macro `_vel_pvfmkwle_Mvl` for conditional compilation, shorthand, or API generation.
  **L965 CN**: 定义宏 `_vel_pvfmkwle_Mvl`，用于条件编译、简写或 API 生成。
- **L966 EN**: Defines macro `_vel_pvfmkwle_MvMl` for conditional compilation, shorthand, or API generation.
  **L966 CN**: 定义宏 `_vel_pvfmkwle_MvMl`，用于条件编译、简写或 API 生成。
- **L967 EN**: Defines macro `_vel_pvfmkwnum_Mvl` for conditional compilation, shorthand, or API generation.
  **L967 CN**: 定义宏 `_vel_pvfmkwnum_Mvl`，用于条件编译、简写或 API 生成。
- **L968 EN**: Defines macro `_vel_pvfmkwnum_MvMl` for conditional compilation, shorthand, or API generation.
  **L968 CN**: 定义宏 `_vel_pvfmkwnum_MvMl`，用于条件编译、简写或 API 生成。
- **L969 EN**: Defines macro `_vel_pvfmkwnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L969 CN**: 定义宏 `_vel_pvfmkwnan_Mvl`，用于条件编译、简写或 API 生成。
- **L970 EN**: Defines macro `_vel_pvfmkwnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L970 CN**: 定义宏 `_vel_pvfmkwnan_MvMl`，用于条件编译、简写或 API 生成。
- **L971 EN**: Defines macro `_vel_pvfmkwgtnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L971 CN**: 定义宏 `_vel_pvfmkwgtnan_Mvl`，用于条件编译、简写或 API 生成。
- **L972 EN**: Defines macro `_vel_pvfmkwgtnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L972 CN**: 定义宏 `_vel_pvfmkwgtnan_MvMl`，用于条件编译、简写或 API 生成。
- **L973 EN**: Defines macro `_vel_pvfmkwltnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L973 CN**: 定义宏 `_vel_pvfmkwltnan_Mvl`，用于条件编译、简写或 API 生成。
- **L974 EN**: Defines macro `_vel_pvfmkwltnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L974 CN**: 定义宏 `_vel_pvfmkwltnan_MvMl`，用于条件编译、简写或 API 生成。
- **L975 EN**: Defines macro `_vel_pvfmkwnenan_Mvl` for conditional compilation, shorthand, or API generation.
  **L975 CN**: 定义宏 `_vel_pvfmkwnenan_Mvl`，用于条件编译、简写或 API 生成。
- **L976 EN**: Defines macro `_vel_pvfmkwnenan_MvMl` for conditional compilation, shorthand, or API generation.
  **L976 CN**: 定义宏 `_vel_pvfmkwnenan_MvMl`，用于条件编译、简写或 API 生成。
- **L977 EN**: Defines macro `_vel_pvfmkweqnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L977 CN**: 定义宏 `_vel_pvfmkweqnan_Mvl`，用于条件编译、简写或 API 生成。
- **L978 EN**: Defines macro `_vel_pvfmkweqnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L978 CN**: 定义宏 `_vel_pvfmkweqnan_MvMl`，用于条件编译、简写或 API 生成。
- **L979 EN**: Defines macro `_vel_pvfmkwgenan_Mvl` for conditional compilation, shorthand, or API generation.
  **L979 CN**: 定义宏 `_vel_pvfmkwgenan_Mvl`，用于条件编译、简写或 API 生成。
- **L980 EN**: Defines macro `_vel_pvfmkwgenan_MvMl` for conditional compilation, shorthand, or API generation.
  **L980 CN**: 定义宏 `_vel_pvfmkwgenan_MvMl`，用于条件编译、简写或 API 生成。
- **L981 EN**: Defines macro `_vel_pvfmkwlenan_Mvl` for conditional compilation, shorthand, or API generation.
  **L981 CN**: 定义宏 `_vel_pvfmkwlenan_Mvl`，用于条件编译、简写或 API 生成。
- **L982 EN**: Defines macro `_vel_pvfmkwlenan_MvMl` for conditional compilation, shorthand, or API generation.
  **L982 CN**: 定义宏 `_vel_pvfmkwlenan_MvMl`，用于条件编译、简写或 API 生成。
- **L983 EN**: Defines macro `_vel_vfmkdgt_mvl` for conditional compilation, shorthand, or API generation.
  **L983 CN**: 定义宏 `_vel_vfmkdgt_mvl`，用于条件编译、简写或 API 生成。
- **L984 EN**: Defines macro `_vel_vfmkdgt_mvml` for conditional compilation, shorthand, or API generation.
  **L984 CN**: 定义宏 `_vel_vfmkdgt_mvml`，用于条件编译、简写或 API 生成。

### Lines 985-1008

````c
#define _vel_vfmkdlt_mvl __builtin_ve_vl_vfmkdlt_mvl
#define _vel_vfmkdlt_mvml __builtin_ve_vl_vfmkdlt_mvml
#define _vel_vfmkdne_mvl __builtin_ve_vl_vfmkdne_mvl
#define _vel_vfmkdne_mvml __builtin_ve_vl_vfmkdne_mvml
#define _vel_vfmkdeq_mvl __builtin_ve_vl_vfmkdeq_mvl
#define _vel_vfmkdeq_mvml __builtin_ve_vl_vfmkdeq_mvml
#define _vel_vfmkdge_mvl __builtin_ve_vl_vfmkdge_mvl
#define _vel_vfmkdge_mvml __builtin_ve_vl_vfmkdge_mvml
#define _vel_vfmkdle_mvl __builtin_ve_vl_vfmkdle_mvl
#define _vel_vfmkdle_mvml __builtin_ve_vl_vfmkdle_mvml
#define _vel_vfmkdnum_mvl __builtin_ve_vl_vfmkdnum_mvl
#define _vel_vfmkdnum_mvml __builtin_ve_vl_vfmkdnum_mvml
#define _vel_vfmkdnan_mvl __builtin_ve_vl_vfmkdnan_mvl
#define _vel_vfmkdnan_mvml __builtin_ve_vl_vfmkdnan_mvml
#define _vel_vfmkdgtnan_mvl __builtin_ve_vl_vfmkdgtnan_mvl
#define _vel_vfmkdgtnan_mvml __builtin_ve_vl_vfmkdgtnan_mvml
#define _vel_vfmkdltnan_mvl __builtin_ve_vl_vfmkdltnan_mvl
#define _vel_vfmkdltnan_mvml __builtin_ve_vl_vfmkdltnan_mvml
#define _vel_vfmkdnenan_mvl __builtin_ve_vl_vfmkdnenan_mvl
#define _vel_vfmkdnenan_mvml __builtin_ve_vl_vfmkdnenan_mvml
#define _vel_vfmkdeqnan_mvl __builtin_ve_vl_vfmkdeqnan_mvl
#define _vel_vfmkdeqnan_mvml __builtin_ve_vl_vfmkdeqnan_mvml
#define _vel_vfmkdgenan_mvl __builtin_ve_vl_vfmkdgenan_mvl
#define _vel_vfmkdgenan_mvml __builtin_ve_vl_vfmkdgenan_mvml
````
- **L985 EN**: Defines macro `_vel_vfmkdlt_mvl` for conditional compilation, shorthand, or API generation.
  **L985 CN**: 定义宏 `_vel_vfmkdlt_mvl`，用于条件编译、简写或 API 生成。
- **L986 EN**: Defines macro `_vel_vfmkdlt_mvml` for conditional compilation, shorthand, or API generation.
  **L986 CN**: 定义宏 `_vel_vfmkdlt_mvml`，用于条件编译、简写或 API 生成。
- **L987 EN**: Defines macro `_vel_vfmkdne_mvl` for conditional compilation, shorthand, or API generation.
  **L987 CN**: 定义宏 `_vel_vfmkdne_mvl`，用于条件编译、简写或 API 生成。
- **L988 EN**: Defines macro `_vel_vfmkdne_mvml` for conditional compilation, shorthand, or API generation.
  **L988 CN**: 定义宏 `_vel_vfmkdne_mvml`，用于条件编译、简写或 API 生成。
- **L989 EN**: Defines macro `_vel_vfmkdeq_mvl` for conditional compilation, shorthand, or API generation.
  **L989 CN**: 定义宏 `_vel_vfmkdeq_mvl`，用于条件编译、简写或 API 生成。
- **L990 EN**: Defines macro `_vel_vfmkdeq_mvml` for conditional compilation, shorthand, or API generation.
  **L990 CN**: 定义宏 `_vel_vfmkdeq_mvml`，用于条件编译、简写或 API 生成。
- **L991 EN**: Defines macro `_vel_vfmkdge_mvl` for conditional compilation, shorthand, or API generation.
  **L991 CN**: 定义宏 `_vel_vfmkdge_mvl`，用于条件编译、简写或 API 生成。
- **L992 EN**: Defines macro `_vel_vfmkdge_mvml` for conditional compilation, shorthand, or API generation.
  **L992 CN**: 定义宏 `_vel_vfmkdge_mvml`，用于条件编译、简写或 API 生成。
- **L993 EN**: Defines macro `_vel_vfmkdle_mvl` for conditional compilation, shorthand, or API generation.
  **L993 CN**: 定义宏 `_vel_vfmkdle_mvl`，用于条件编译、简写或 API 生成。
- **L994 EN**: Defines macro `_vel_vfmkdle_mvml` for conditional compilation, shorthand, or API generation.
  **L994 CN**: 定义宏 `_vel_vfmkdle_mvml`，用于条件编译、简写或 API 生成。
- **L995 EN**: Defines macro `_vel_vfmkdnum_mvl` for conditional compilation, shorthand, or API generation.
  **L995 CN**: 定义宏 `_vel_vfmkdnum_mvl`，用于条件编译、简写或 API 生成。
- **L996 EN**: Defines macro `_vel_vfmkdnum_mvml` for conditional compilation, shorthand, or API generation.
  **L996 CN**: 定义宏 `_vel_vfmkdnum_mvml`，用于条件编译、简写或 API 生成。
- **L997 EN**: Defines macro `_vel_vfmkdnan_mvl` for conditional compilation, shorthand, or API generation.
  **L997 CN**: 定义宏 `_vel_vfmkdnan_mvl`，用于条件编译、简写或 API 生成。
- **L998 EN**: Defines macro `_vel_vfmkdnan_mvml` for conditional compilation, shorthand, or API generation.
  **L998 CN**: 定义宏 `_vel_vfmkdnan_mvml`，用于条件编译、简写或 API 生成。
- **L999 EN**: Defines macro `_vel_vfmkdgtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L999 CN**: 定义宏 `_vel_vfmkdgtnan_mvl`，用于条件编译、简写或 API 生成。
- **L1000 EN**: Defines macro `_vel_vfmkdgtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1000 CN**: 定义宏 `_vel_vfmkdgtnan_mvml`，用于条件编译、简写或 API 生成。
- **L1001 EN**: Defines macro `_vel_vfmkdltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1001 CN**: 定义宏 `_vel_vfmkdltnan_mvl`，用于条件编译、简写或 API 生成。
- **L1002 EN**: Defines macro `_vel_vfmkdltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1002 CN**: 定义宏 `_vel_vfmkdltnan_mvml`，用于条件编译、简写或 API 生成。
- **L1003 EN**: Defines macro `_vel_vfmkdnenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1003 CN**: 定义宏 `_vel_vfmkdnenan_mvl`，用于条件编译、简写或 API 生成。
- **L1004 EN**: Defines macro `_vel_vfmkdnenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1004 CN**: 定义宏 `_vel_vfmkdnenan_mvml`，用于条件编译、简写或 API 生成。
- **L1005 EN**: Defines macro `_vel_vfmkdeqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1005 CN**: 定义宏 `_vel_vfmkdeqnan_mvl`，用于条件编译、简写或 API 生成。
- **L1006 EN**: Defines macro `_vel_vfmkdeqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1006 CN**: 定义宏 `_vel_vfmkdeqnan_mvml`，用于条件编译、简写或 API 生成。
- **L1007 EN**: Defines macro `_vel_vfmkdgenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1007 CN**: 定义宏 `_vel_vfmkdgenan_mvl`，用于条件编译、简写或 API 生成。
- **L1008 EN**: Defines macro `_vel_vfmkdgenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1008 CN**: 定义宏 `_vel_vfmkdgenan_mvml`，用于条件编译、简写或 API 生成。

### Lines 1009-1032

````c
#define _vel_vfmkdlenan_mvl __builtin_ve_vl_vfmkdlenan_mvl
#define _vel_vfmkdlenan_mvml __builtin_ve_vl_vfmkdlenan_mvml
#define _vel_vfmksgt_mvl __builtin_ve_vl_vfmksgt_mvl
#define _vel_vfmksgt_mvml __builtin_ve_vl_vfmksgt_mvml
#define _vel_vfmkslt_mvl __builtin_ve_vl_vfmkslt_mvl
#define _vel_vfmkslt_mvml __builtin_ve_vl_vfmkslt_mvml
#define _vel_vfmksne_mvl __builtin_ve_vl_vfmksne_mvl
#define _vel_vfmksne_mvml __builtin_ve_vl_vfmksne_mvml
#define _vel_vfmkseq_mvl __builtin_ve_vl_vfmkseq_mvl
#define _vel_vfmkseq_mvml __builtin_ve_vl_vfmkseq_mvml
#define _vel_vfmksge_mvl __builtin_ve_vl_vfmksge_mvl
#define _vel_vfmksge_mvml __builtin_ve_vl_vfmksge_mvml
#define _vel_vfmksle_mvl __builtin_ve_vl_vfmksle_mvl
#define _vel_vfmksle_mvml __builtin_ve_vl_vfmksle_mvml
#define _vel_vfmksnum_mvl __builtin_ve_vl_vfmksnum_mvl
#define _vel_vfmksnum_mvml __builtin_ve_vl_vfmksnum_mvml
#define _vel_vfmksnan_mvl __builtin_ve_vl_vfmksnan_mvl
#define _vel_vfmksnan_mvml __builtin_ve_vl_vfmksnan_mvml
#define _vel_vfmksgtnan_mvl __builtin_ve_vl_vfmksgtnan_mvl
#define _vel_vfmksgtnan_mvml __builtin_ve_vl_vfmksgtnan_mvml
#define _vel_vfmksltnan_mvl __builtin_ve_vl_vfmksltnan_mvl
#define _vel_vfmksltnan_mvml __builtin_ve_vl_vfmksltnan_mvml
#define _vel_vfmksnenan_mvl __builtin_ve_vl_vfmksnenan_mvl
#define _vel_vfmksnenan_mvml __builtin_ve_vl_vfmksnenan_mvml
````
- **L1009 EN**: Defines macro `_vel_vfmkdlenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1009 CN**: 定义宏 `_vel_vfmkdlenan_mvl`，用于条件编译、简写或 API 生成。
- **L1010 EN**: Defines macro `_vel_vfmkdlenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1010 CN**: 定义宏 `_vel_vfmkdlenan_mvml`，用于条件编译、简写或 API 生成。
- **L1011 EN**: Defines macro `_vel_vfmksgt_mvl` for conditional compilation, shorthand, or API generation.
  **L1011 CN**: 定义宏 `_vel_vfmksgt_mvl`，用于条件编译、简写或 API 生成。
- **L1012 EN**: Defines macro `_vel_vfmksgt_mvml` for conditional compilation, shorthand, or API generation.
  **L1012 CN**: 定义宏 `_vel_vfmksgt_mvml`，用于条件编译、简写或 API 生成。
- **L1013 EN**: Defines macro `_vel_vfmkslt_mvl` for conditional compilation, shorthand, or API generation.
  **L1013 CN**: 定义宏 `_vel_vfmkslt_mvl`，用于条件编译、简写或 API 生成。
- **L1014 EN**: Defines macro `_vel_vfmkslt_mvml` for conditional compilation, shorthand, or API generation.
  **L1014 CN**: 定义宏 `_vel_vfmkslt_mvml`，用于条件编译、简写或 API 生成。
- **L1015 EN**: Defines macro `_vel_vfmksne_mvl` for conditional compilation, shorthand, or API generation.
  **L1015 CN**: 定义宏 `_vel_vfmksne_mvl`，用于条件编译、简写或 API 生成。
- **L1016 EN**: Defines macro `_vel_vfmksne_mvml` for conditional compilation, shorthand, or API generation.
  **L1016 CN**: 定义宏 `_vel_vfmksne_mvml`，用于条件编译、简写或 API 生成。
- **L1017 EN**: Defines macro `_vel_vfmkseq_mvl` for conditional compilation, shorthand, or API generation.
  **L1017 CN**: 定义宏 `_vel_vfmkseq_mvl`，用于条件编译、简写或 API 生成。
- **L1018 EN**: Defines macro `_vel_vfmkseq_mvml` for conditional compilation, shorthand, or API generation.
  **L1018 CN**: 定义宏 `_vel_vfmkseq_mvml`，用于条件编译、简写或 API 生成。
- **L1019 EN**: Defines macro `_vel_vfmksge_mvl` for conditional compilation, shorthand, or API generation.
  **L1019 CN**: 定义宏 `_vel_vfmksge_mvl`，用于条件编译、简写或 API 生成。
- **L1020 EN**: Defines macro `_vel_vfmksge_mvml` for conditional compilation, shorthand, or API generation.
  **L1020 CN**: 定义宏 `_vel_vfmksge_mvml`，用于条件编译、简写或 API 生成。
- **L1021 EN**: Defines macro `_vel_vfmksle_mvl` for conditional compilation, shorthand, or API generation.
  **L1021 CN**: 定义宏 `_vel_vfmksle_mvl`，用于条件编译、简写或 API 生成。
- **L1022 EN**: Defines macro `_vel_vfmksle_mvml` for conditional compilation, shorthand, or API generation.
  **L1022 CN**: 定义宏 `_vel_vfmksle_mvml`，用于条件编译、简写或 API 生成。
- **L1023 EN**: Defines macro `_vel_vfmksnum_mvl` for conditional compilation, shorthand, or API generation.
  **L1023 CN**: 定义宏 `_vel_vfmksnum_mvl`，用于条件编译、简写或 API 生成。
- **L1024 EN**: Defines macro `_vel_vfmksnum_mvml` for conditional compilation, shorthand, or API generation.
  **L1024 CN**: 定义宏 `_vel_vfmksnum_mvml`，用于条件编译、简写或 API 生成。
- **L1025 EN**: Defines macro `_vel_vfmksnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1025 CN**: 定义宏 `_vel_vfmksnan_mvl`，用于条件编译、简写或 API 生成。
- **L1026 EN**: Defines macro `_vel_vfmksnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1026 CN**: 定义宏 `_vel_vfmksnan_mvml`，用于条件编译、简写或 API 生成。
- **L1027 EN**: Defines macro `_vel_vfmksgtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1027 CN**: 定义宏 `_vel_vfmksgtnan_mvl`，用于条件编译、简写或 API 生成。
- **L1028 EN**: Defines macro `_vel_vfmksgtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1028 CN**: 定义宏 `_vel_vfmksgtnan_mvml`，用于条件编译、简写或 API 生成。
- **L1029 EN**: Defines macro `_vel_vfmksltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1029 CN**: 定义宏 `_vel_vfmksltnan_mvl`，用于条件编译、简写或 API 生成。
- **L1030 EN**: Defines macro `_vel_vfmksltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1030 CN**: 定义宏 `_vel_vfmksltnan_mvml`，用于条件编译、简写或 API 生成。
- **L1031 EN**: Defines macro `_vel_vfmksnenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1031 CN**: 定义宏 `_vel_vfmksnenan_mvl`，用于条件编译、简写或 API 生成。
- **L1032 EN**: Defines macro `_vel_vfmksnenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1032 CN**: 定义宏 `_vel_vfmksnenan_mvml`，用于条件编译、简写或 API 生成。

### Lines 1033-1056

````c
#define _vel_vfmkseqnan_mvl __builtin_ve_vl_vfmkseqnan_mvl
#define _vel_vfmkseqnan_mvml __builtin_ve_vl_vfmkseqnan_mvml
#define _vel_vfmksgenan_mvl __builtin_ve_vl_vfmksgenan_mvl
#define _vel_vfmksgenan_mvml __builtin_ve_vl_vfmksgenan_mvml
#define _vel_vfmkslenan_mvl __builtin_ve_vl_vfmkslenan_mvl
#define _vel_vfmkslenan_mvml __builtin_ve_vl_vfmkslenan_mvml
#define _vel_pvfmkslogt_mvl __builtin_ve_vl_pvfmkslogt_mvl
#define _vel_pvfmksupgt_mvl __builtin_ve_vl_pvfmksupgt_mvl
#define _vel_pvfmkslogt_mvml __builtin_ve_vl_pvfmkslogt_mvml
#define _vel_pvfmksupgt_mvml __builtin_ve_vl_pvfmksupgt_mvml
#define _vel_pvfmkslolt_mvl __builtin_ve_vl_pvfmkslolt_mvl
#define _vel_pvfmksuplt_mvl __builtin_ve_vl_pvfmksuplt_mvl
#define _vel_pvfmkslolt_mvml __builtin_ve_vl_pvfmkslolt_mvml
#define _vel_pvfmksuplt_mvml __builtin_ve_vl_pvfmksuplt_mvml
#define _vel_pvfmkslone_mvl __builtin_ve_vl_pvfmkslone_mvl
#define _vel_pvfmksupne_mvl __builtin_ve_vl_pvfmksupne_mvl
#define _vel_pvfmkslone_mvml __builtin_ve_vl_pvfmkslone_mvml
#define _vel_pvfmksupne_mvml __builtin_ve_vl_pvfmksupne_mvml
#define _vel_pvfmksloeq_mvl __builtin_ve_vl_pvfmksloeq_mvl
#define _vel_pvfmksupeq_mvl __builtin_ve_vl_pvfmksupeq_mvl
#define _vel_pvfmksloeq_mvml __builtin_ve_vl_pvfmksloeq_mvml
#define _vel_pvfmksupeq_mvml __builtin_ve_vl_pvfmksupeq_mvml
#define _vel_pvfmksloge_mvl __builtin_ve_vl_pvfmksloge_mvl
#define _vel_pvfmksupge_mvl __builtin_ve_vl_pvfmksupge_mvl
````
- **L1033 EN**: Defines macro `_vel_vfmkseqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1033 CN**: 定义宏 `_vel_vfmkseqnan_mvl`，用于条件编译、简写或 API 生成。
- **L1034 EN**: Defines macro `_vel_vfmkseqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1034 CN**: 定义宏 `_vel_vfmkseqnan_mvml`，用于条件编译、简写或 API 生成。
- **L1035 EN**: Defines macro `_vel_vfmksgenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1035 CN**: 定义宏 `_vel_vfmksgenan_mvl`，用于条件编译、简写或 API 生成。
- **L1036 EN**: Defines macro `_vel_vfmksgenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1036 CN**: 定义宏 `_vel_vfmksgenan_mvml`，用于条件编译、简写或 API 生成。
- **L1037 EN**: Defines macro `_vel_vfmkslenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1037 CN**: 定义宏 `_vel_vfmkslenan_mvl`，用于条件编译、简写或 API 生成。
- **L1038 EN**: Defines macro `_vel_vfmkslenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1038 CN**: 定义宏 `_vel_vfmkslenan_mvml`，用于条件编译、简写或 API 生成。
- **L1039 EN**: Defines macro `_vel_pvfmkslogt_mvl` for conditional compilation, shorthand, or API generation.
  **L1039 CN**: 定义宏 `_vel_pvfmkslogt_mvl`，用于条件编译、简写或 API 生成。
- **L1040 EN**: Defines macro `_vel_pvfmksupgt_mvl` for conditional compilation, shorthand, or API generation.
  **L1040 CN**: 定义宏 `_vel_pvfmksupgt_mvl`，用于条件编译、简写或 API 生成。
- **L1041 EN**: Defines macro `_vel_pvfmkslogt_mvml` for conditional compilation, shorthand, or API generation.
  **L1041 CN**: 定义宏 `_vel_pvfmkslogt_mvml`，用于条件编译、简写或 API 生成。
- **L1042 EN**: Defines macro `_vel_pvfmksupgt_mvml` for conditional compilation, shorthand, or API generation.
  **L1042 CN**: 定义宏 `_vel_pvfmksupgt_mvml`，用于条件编译、简写或 API 生成。
- **L1043 EN**: Defines macro `_vel_pvfmkslolt_mvl` for conditional compilation, shorthand, or API generation.
  **L1043 CN**: 定义宏 `_vel_pvfmkslolt_mvl`，用于条件编译、简写或 API 生成。
- **L1044 EN**: Defines macro `_vel_pvfmksuplt_mvl` for conditional compilation, shorthand, or API generation.
  **L1044 CN**: 定义宏 `_vel_pvfmksuplt_mvl`，用于条件编译、简写或 API 生成。
- **L1045 EN**: Defines macro `_vel_pvfmkslolt_mvml` for conditional compilation, shorthand, or API generation.
  **L1045 CN**: 定义宏 `_vel_pvfmkslolt_mvml`，用于条件编译、简写或 API 生成。
- **L1046 EN**: Defines macro `_vel_pvfmksuplt_mvml` for conditional compilation, shorthand, or API generation.
  **L1046 CN**: 定义宏 `_vel_pvfmksuplt_mvml`，用于条件编译、简写或 API 生成。
- **L1047 EN**: Defines macro `_vel_pvfmkslone_mvl` for conditional compilation, shorthand, or API generation.
  **L1047 CN**: 定义宏 `_vel_pvfmkslone_mvl`，用于条件编译、简写或 API 生成。
- **L1048 EN**: Defines macro `_vel_pvfmksupne_mvl` for conditional compilation, shorthand, or API generation.
  **L1048 CN**: 定义宏 `_vel_pvfmksupne_mvl`，用于条件编译、简写或 API 生成。
- **L1049 EN**: Defines macro `_vel_pvfmkslone_mvml` for conditional compilation, shorthand, or API generation.
  **L1049 CN**: 定义宏 `_vel_pvfmkslone_mvml`，用于条件编译、简写或 API 生成。
- **L1050 EN**: Defines macro `_vel_pvfmksupne_mvml` for conditional compilation, shorthand, or API generation.
  **L1050 CN**: 定义宏 `_vel_pvfmksupne_mvml`，用于条件编译、简写或 API 生成。
- **L1051 EN**: Defines macro `_vel_pvfmksloeq_mvl` for conditional compilation, shorthand, or API generation.
  **L1051 CN**: 定义宏 `_vel_pvfmksloeq_mvl`，用于条件编译、简写或 API 生成。
- **L1052 EN**: Defines macro `_vel_pvfmksupeq_mvl` for conditional compilation, shorthand, or API generation.
  **L1052 CN**: 定义宏 `_vel_pvfmksupeq_mvl`，用于条件编译、简写或 API 生成。
- **L1053 EN**: Defines macro `_vel_pvfmksloeq_mvml` for conditional compilation, shorthand, or API generation.
  **L1053 CN**: 定义宏 `_vel_pvfmksloeq_mvml`，用于条件编译、简写或 API 生成。
- **L1054 EN**: Defines macro `_vel_pvfmksupeq_mvml` for conditional compilation, shorthand, or API generation.
  **L1054 CN**: 定义宏 `_vel_pvfmksupeq_mvml`，用于条件编译、简写或 API 生成。
- **L1055 EN**: Defines macro `_vel_pvfmksloge_mvl` for conditional compilation, shorthand, or API generation.
  **L1055 CN**: 定义宏 `_vel_pvfmksloge_mvl`，用于条件编译、简写或 API 生成。
- **L1056 EN**: Defines macro `_vel_pvfmksupge_mvl` for conditional compilation, shorthand, or API generation.
  **L1056 CN**: 定义宏 `_vel_pvfmksupge_mvl`，用于条件编译、简写或 API 生成。

### Lines 1057-1080

````c
#define _vel_pvfmksloge_mvml __builtin_ve_vl_pvfmksloge_mvml
#define _vel_pvfmksupge_mvml __builtin_ve_vl_pvfmksupge_mvml
#define _vel_pvfmkslole_mvl __builtin_ve_vl_pvfmkslole_mvl
#define _vel_pvfmksuple_mvl __builtin_ve_vl_pvfmksuple_mvl
#define _vel_pvfmkslole_mvml __builtin_ve_vl_pvfmkslole_mvml
#define _vel_pvfmksuple_mvml __builtin_ve_vl_pvfmksuple_mvml
#define _vel_pvfmkslonum_mvl __builtin_ve_vl_pvfmkslonum_mvl
#define _vel_pvfmksupnum_mvl __builtin_ve_vl_pvfmksupnum_mvl
#define _vel_pvfmkslonum_mvml __builtin_ve_vl_pvfmkslonum_mvml
#define _vel_pvfmksupnum_mvml __builtin_ve_vl_pvfmksupnum_mvml
#define _vel_pvfmkslonan_mvl __builtin_ve_vl_pvfmkslonan_mvl
#define _vel_pvfmksupnan_mvl __builtin_ve_vl_pvfmksupnan_mvl
#define _vel_pvfmkslonan_mvml __builtin_ve_vl_pvfmkslonan_mvml
#define _vel_pvfmksupnan_mvml __builtin_ve_vl_pvfmksupnan_mvml
#define _vel_pvfmkslogtnan_mvl __builtin_ve_vl_pvfmkslogtnan_mvl
#define _vel_pvfmksupgtnan_mvl __builtin_ve_vl_pvfmksupgtnan_mvl
#define _vel_pvfmkslogtnan_mvml __builtin_ve_vl_pvfmkslogtnan_mvml
#define _vel_pvfmksupgtnan_mvml __builtin_ve_vl_pvfmksupgtnan_mvml
#define _vel_pvfmksloltnan_mvl __builtin_ve_vl_pvfmksloltnan_mvl
#define _vel_pvfmksupltnan_mvl __builtin_ve_vl_pvfmksupltnan_mvl
#define _vel_pvfmksloltnan_mvml __builtin_ve_vl_pvfmksloltnan_mvml
#define _vel_pvfmksupltnan_mvml __builtin_ve_vl_pvfmksupltnan_mvml
#define _vel_pvfmkslonenan_mvl __builtin_ve_vl_pvfmkslonenan_mvl
#define _vel_pvfmksupnenan_mvl __builtin_ve_vl_pvfmksupnenan_mvl
````
- **L1057 EN**: Defines macro `_vel_pvfmksloge_mvml` for conditional compilation, shorthand, or API generation.
  **L1057 CN**: 定义宏 `_vel_pvfmksloge_mvml`，用于条件编译、简写或 API 生成。
- **L1058 EN**: Defines macro `_vel_pvfmksupge_mvml` for conditional compilation, shorthand, or API generation.
  **L1058 CN**: 定义宏 `_vel_pvfmksupge_mvml`，用于条件编译、简写或 API 生成。
- **L1059 EN**: Defines macro `_vel_pvfmkslole_mvl` for conditional compilation, shorthand, or API generation.
  **L1059 CN**: 定义宏 `_vel_pvfmkslole_mvl`，用于条件编译、简写或 API 生成。
- **L1060 EN**: Defines macro `_vel_pvfmksuple_mvl` for conditional compilation, shorthand, or API generation.
  **L1060 CN**: 定义宏 `_vel_pvfmksuple_mvl`，用于条件编译、简写或 API 生成。
- **L1061 EN**: Defines macro `_vel_pvfmkslole_mvml` for conditional compilation, shorthand, or API generation.
  **L1061 CN**: 定义宏 `_vel_pvfmkslole_mvml`，用于条件编译、简写或 API 生成。
- **L1062 EN**: Defines macro `_vel_pvfmksuple_mvml` for conditional compilation, shorthand, or API generation.
  **L1062 CN**: 定义宏 `_vel_pvfmksuple_mvml`，用于条件编译、简写或 API 生成。
- **L1063 EN**: Defines macro `_vel_pvfmkslonum_mvl` for conditional compilation, shorthand, or API generation.
  **L1063 CN**: 定义宏 `_vel_pvfmkslonum_mvl`，用于条件编译、简写或 API 生成。
- **L1064 EN**: Defines macro `_vel_pvfmksupnum_mvl` for conditional compilation, shorthand, or API generation.
  **L1064 CN**: 定义宏 `_vel_pvfmksupnum_mvl`，用于条件编译、简写或 API 生成。
- **L1065 EN**: Defines macro `_vel_pvfmkslonum_mvml` for conditional compilation, shorthand, or API generation.
  **L1065 CN**: 定义宏 `_vel_pvfmkslonum_mvml`，用于条件编译、简写或 API 生成。
- **L1066 EN**: Defines macro `_vel_pvfmksupnum_mvml` for conditional compilation, shorthand, or API generation.
  **L1066 CN**: 定义宏 `_vel_pvfmksupnum_mvml`，用于条件编译、简写或 API 生成。
- **L1067 EN**: Defines macro `_vel_pvfmkslonan_mvl` for conditional compilation, shorthand, or API generation.
  **L1067 CN**: 定义宏 `_vel_pvfmkslonan_mvl`，用于条件编译、简写或 API 生成。
- **L1068 EN**: Defines macro `_vel_pvfmksupnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1068 CN**: 定义宏 `_vel_pvfmksupnan_mvl`，用于条件编译、简写或 API 生成。
- **L1069 EN**: Defines macro `_vel_pvfmkslonan_mvml` for conditional compilation, shorthand, or API generation.
  **L1069 CN**: 定义宏 `_vel_pvfmkslonan_mvml`，用于条件编译、简写或 API 生成。
- **L1070 EN**: Defines macro `_vel_pvfmksupnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1070 CN**: 定义宏 `_vel_pvfmksupnan_mvml`，用于条件编译、简写或 API 生成。
- **L1071 EN**: Defines macro `_vel_pvfmkslogtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1071 CN**: 定义宏 `_vel_pvfmkslogtnan_mvl`，用于条件编译、简写或 API 生成。
- **L1072 EN**: Defines macro `_vel_pvfmksupgtnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1072 CN**: 定义宏 `_vel_pvfmksupgtnan_mvl`，用于条件编译、简写或 API 生成。
- **L1073 EN**: Defines macro `_vel_pvfmkslogtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1073 CN**: 定义宏 `_vel_pvfmkslogtnan_mvml`，用于条件编译、简写或 API 生成。
- **L1074 EN**: Defines macro `_vel_pvfmksupgtnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1074 CN**: 定义宏 `_vel_pvfmksupgtnan_mvml`，用于条件编译、简写或 API 生成。
- **L1075 EN**: Defines macro `_vel_pvfmksloltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1075 CN**: 定义宏 `_vel_pvfmksloltnan_mvl`，用于条件编译、简写或 API 生成。
- **L1076 EN**: Defines macro `_vel_pvfmksupltnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1076 CN**: 定义宏 `_vel_pvfmksupltnan_mvl`，用于条件编译、简写或 API 生成。
- **L1077 EN**: Defines macro `_vel_pvfmksloltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1077 CN**: 定义宏 `_vel_pvfmksloltnan_mvml`，用于条件编译、简写或 API 生成。
- **L1078 EN**: Defines macro `_vel_pvfmksupltnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1078 CN**: 定义宏 `_vel_pvfmksupltnan_mvml`，用于条件编译、简写或 API 生成。
- **L1079 EN**: Defines macro `_vel_pvfmkslonenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1079 CN**: 定义宏 `_vel_pvfmkslonenan_mvl`，用于条件编译、简写或 API 生成。
- **L1080 EN**: Defines macro `_vel_pvfmksupnenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1080 CN**: 定义宏 `_vel_pvfmksupnenan_mvl`，用于条件编译、简写或 API 生成。

### Lines 1081-1104

````c
#define _vel_pvfmkslonenan_mvml __builtin_ve_vl_pvfmkslonenan_mvml
#define _vel_pvfmksupnenan_mvml __builtin_ve_vl_pvfmksupnenan_mvml
#define _vel_pvfmksloeqnan_mvl __builtin_ve_vl_pvfmksloeqnan_mvl
#define _vel_pvfmksupeqnan_mvl __builtin_ve_vl_pvfmksupeqnan_mvl
#define _vel_pvfmksloeqnan_mvml __builtin_ve_vl_pvfmksloeqnan_mvml
#define _vel_pvfmksupeqnan_mvml __builtin_ve_vl_pvfmksupeqnan_mvml
#define _vel_pvfmkslogenan_mvl __builtin_ve_vl_pvfmkslogenan_mvl
#define _vel_pvfmksupgenan_mvl __builtin_ve_vl_pvfmksupgenan_mvl
#define _vel_pvfmkslogenan_mvml __builtin_ve_vl_pvfmkslogenan_mvml
#define _vel_pvfmksupgenan_mvml __builtin_ve_vl_pvfmksupgenan_mvml
#define _vel_pvfmkslolenan_mvl __builtin_ve_vl_pvfmkslolenan_mvl
#define _vel_pvfmksuplenan_mvl __builtin_ve_vl_pvfmksuplenan_mvl
#define _vel_pvfmkslolenan_mvml __builtin_ve_vl_pvfmkslolenan_mvml
#define _vel_pvfmksuplenan_mvml __builtin_ve_vl_pvfmksuplenan_mvml
#define _vel_pvfmksgt_Mvl __builtin_ve_vl_pvfmksgt_Mvl
#define _vel_pvfmksgt_MvMl __builtin_ve_vl_pvfmksgt_MvMl
#define _vel_pvfmkslt_Mvl __builtin_ve_vl_pvfmkslt_Mvl
#define _vel_pvfmkslt_MvMl __builtin_ve_vl_pvfmkslt_MvMl
#define _vel_pvfmksne_Mvl __builtin_ve_vl_pvfmksne_Mvl
#define _vel_pvfmksne_MvMl __builtin_ve_vl_pvfmksne_MvMl
#define _vel_pvfmkseq_Mvl __builtin_ve_vl_pvfmkseq_Mvl
#define _vel_pvfmkseq_MvMl __builtin_ve_vl_pvfmkseq_MvMl
#define _vel_pvfmksge_Mvl __builtin_ve_vl_pvfmksge_Mvl
#define _vel_pvfmksge_MvMl __builtin_ve_vl_pvfmksge_MvMl
````
- **L1081 EN**: Defines macro `_vel_pvfmkslonenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1081 CN**: 定义宏 `_vel_pvfmkslonenan_mvml`，用于条件编译、简写或 API 生成。
- **L1082 EN**: Defines macro `_vel_pvfmksupnenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1082 CN**: 定义宏 `_vel_pvfmksupnenan_mvml`，用于条件编译、简写或 API 生成。
- **L1083 EN**: Defines macro `_vel_pvfmksloeqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1083 CN**: 定义宏 `_vel_pvfmksloeqnan_mvl`，用于条件编译、简写或 API 生成。
- **L1084 EN**: Defines macro `_vel_pvfmksupeqnan_mvl` for conditional compilation, shorthand, or API generation.
  **L1084 CN**: 定义宏 `_vel_pvfmksupeqnan_mvl`，用于条件编译、简写或 API 生成。
- **L1085 EN**: Defines macro `_vel_pvfmksloeqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1085 CN**: 定义宏 `_vel_pvfmksloeqnan_mvml`，用于条件编译、简写或 API 生成。
- **L1086 EN**: Defines macro `_vel_pvfmksupeqnan_mvml` for conditional compilation, shorthand, or API generation.
  **L1086 CN**: 定义宏 `_vel_pvfmksupeqnan_mvml`，用于条件编译、简写或 API 生成。
- **L1087 EN**: Defines macro `_vel_pvfmkslogenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1087 CN**: 定义宏 `_vel_pvfmkslogenan_mvl`，用于条件编译、简写或 API 生成。
- **L1088 EN**: Defines macro `_vel_pvfmksupgenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1088 CN**: 定义宏 `_vel_pvfmksupgenan_mvl`，用于条件编译、简写或 API 生成。
- **L1089 EN**: Defines macro `_vel_pvfmkslogenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1089 CN**: 定义宏 `_vel_pvfmkslogenan_mvml`，用于条件编译、简写或 API 生成。
- **L1090 EN**: Defines macro `_vel_pvfmksupgenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1090 CN**: 定义宏 `_vel_pvfmksupgenan_mvml`，用于条件编译、简写或 API 生成。
- **L1091 EN**: Defines macro `_vel_pvfmkslolenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1091 CN**: 定义宏 `_vel_pvfmkslolenan_mvl`，用于条件编译、简写或 API 生成。
- **L1092 EN**: Defines macro `_vel_pvfmksuplenan_mvl` for conditional compilation, shorthand, or API generation.
  **L1092 CN**: 定义宏 `_vel_pvfmksuplenan_mvl`，用于条件编译、简写或 API 生成。
- **L1093 EN**: Defines macro `_vel_pvfmkslolenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1093 CN**: 定义宏 `_vel_pvfmkslolenan_mvml`，用于条件编译、简写或 API 生成。
- **L1094 EN**: Defines macro `_vel_pvfmksuplenan_mvml` for conditional compilation, shorthand, or API generation.
  **L1094 CN**: 定义宏 `_vel_pvfmksuplenan_mvml`，用于条件编译、简写或 API 生成。
- **L1095 EN**: Defines macro `_vel_pvfmksgt_Mvl` for conditional compilation, shorthand, or API generation.
  **L1095 CN**: 定义宏 `_vel_pvfmksgt_Mvl`，用于条件编译、简写或 API 生成。
- **L1096 EN**: Defines macro `_vel_pvfmksgt_MvMl` for conditional compilation, shorthand, or API generation.
  **L1096 CN**: 定义宏 `_vel_pvfmksgt_MvMl`，用于条件编译、简写或 API 生成。
- **L1097 EN**: Defines macro `_vel_pvfmkslt_Mvl` for conditional compilation, shorthand, or API generation.
  **L1097 CN**: 定义宏 `_vel_pvfmkslt_Mvl`，用于条件编译、简写或 API 生成。
- **L1098 EN**: Defines macro `_vel_pvfmkslt_MvMl` for conditional compilation, shorthand, or API generation.
  **L1098 CN**: 定义宏 `_vel_pvfmkslt_MvMl`，用于条件编译、简写或 API 生成。
- **L1099 EN**: Defines macro `_vel_pvfmksne_Mvl` for conditional compilation, shorthand, or API generation.
  **L1099 CN**: 定义宏 `_vel_pvfmksne_Mvl`，用于条件编译、简写或 API 生成。
- **L1100 EN**: Defines macro `_vel_pvfmksne_MvMl` for conditional compilation, shorthand, or API generation.
  **L1100 CN**: 定义宏 `_vel_pvfmksne_MvMl`，用于条件编译、简写或 API 生成。
- **L1101 EN**: Defines macro `_vel_pvfmkseq_Mvl` for conditional compilation, shorthand, or API generation.
  **L1101 CN**: 定义宏 `_vel_pvfmkseq_Mvl`，用于条件编译、简写或 API 生成。
- **L1102 EN**: Defines macro `_vel_pvfmkseq_MvMl` for conditional compilation, shorthand, or API generation.
  **L1102 CN**: 定义宏 `_vel_pvfmkseq_MvMl`，用于条件编译、简写或 API 生成。
- **L1103 EN**: Defines macro `_vel_pvfmksge_Mvl` for conditional compilation, shorthand, or API generation.
  **L1103 CN**: 定义宏 `_vel_pvfmksge_Mvl`，用于条件编译、简写或 API 生成。
- **L1104 EN**: Defines macro `_vel_pvfmksge_MvMl` for conditional compilation, shorthand, or API generation.
  **L1104 CN**: 定义宏 `_vel_pvfmksge_MvMl`，用于条件编译、简写或 API 生成。

### Lines 1105-1128

````c
#define _vel_pvfmksle_Mvl __builtin_ve_vl_pvfmksle_Mvl
#define _vel_pvfmksle_MvMl __builtin_ve_vl_pvfmksle_MvMl
#define _vel_pvfmksnum_Mvl __builtin_ve_vl_pvfmksnum_Mvl
#define _vel_pvfmksnum_MvMl __builtin_ve_vl_pvfmksnum_MvMl
#define _vel_pvfmksnan_Mvl __builtin_ve_vl_pvfmksnan_Mvl
#define _vel_pvfmksnan_MvMl __builtin_ve_vl_pvfmksnan_MvMl
#define _vel_pvfmksgtnan_Mvl __builtin_ve_vl_pvfmksgtnan_Mvl
#define _vel_pvfmksgtnan_MvMl __builtin_ve_vl_pvfmksgtnan_MvMl
#define _vel_pvfmksltnan_Mvl __builtin_ve_vl_pvfmksltnan_Mvl
#define _vel_pvfmksltnan_MvMl __builtin_ve_vl_pvfmksltnan_MvMl
#define _vel_pvfmksnenan_Mvl __builtin_ve_vl_pvfmksnenan_Mvl
#define _vel_pvfmksnenan_MvMl __builtin_ve_vl_pvfmksnenan_MvMl
#define _vel_pvfmkseqnan_Mvl __builtin_ve_vl_pvfmkseqnan_Mvl
#define _vel_pvfmkseqnan_MvMl __builtin_ve_vl_pvfmkseqnan_MvMl
#define _vel_pvfmksgenan_Mvl __builtin_ve_vl_pvfmksgenan_Mvl
#define _vel_pvfmksgenan_MvMl __builtin_ve_vl_pvfmksgenan_MvMl
#define _vel_pvfmkslenan_Mvl __builtin_ve_vl_pvfmkslenan_Mvl
#define _vel_pvfmkslenan_MvMl __builtin_ve_vl_pvfmkslenan_MvMl
#define _vel_vsumwsx_vvl __builtin_ve_vl_vsumwsx_vvl
#define _vel_vsumwsx_vvml __builtin_ve_vl_vsumwsx_vvml
#define _vel_vsumwzx_vvl __builtin_ve_vl_vsumwzx_vvl
#define _vel_vsumwzx_vvml __builtin_ve_vl_vsumwzx_vvml
#define _vel_vsuml_vvl __builtin_ve_vl_vsuml_vvl
#define _vel_vsuml_vvml __builtin_ve_vl_vsuml_vvml
````
- **L1105 EN**: Defines macro `_vel_pvfmksle_Mvl` for conditional compilation, shorthand, or API generation.
  **L1105 CN**: 定义宏 `_vel_pvfmksle_Mvl`，用于条件编译、简写或 API 生成。
- **L1106 EN**: Defines macro `_vel_pvfmksle_MvMl` for conditional compilation, shorthand, or API generation.
  **L1106 CN**: 定义宏 `_vel_pvfmksle_MvMl`，用于条件编译、简写或 API 生成。
- **L1107 EN**: Defines macro `_vel_pvfmksnum_Mvl` for conditional compilation, shorthand, or API generation.
  **L1107 CN**: 定义宏 `_vel_pvfmksnum_Mvl`，用于条件编译、简写或 API 生成。
- **L1108 EN**: Defines macro `_vel_pvfmksnum_MvMl` for conditional compilation, shorthand, or API generation.
  **L1108 CN**: 定义宏 `_vel_pvfmksnum_MvMl`，用于条件编译、简写或 API 生成。
- **L1109 EN**: Defines macro `_vel_pvfmksnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L1109 CN**: 定义宏 `_vel_pvfmksnan_Mvl`，用于条件编译、简写或 API 生成。
- **L1110 EN**: Defines macro `_vel_pvfmksnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L1110 CN**: 定义宏 `_vel_pvfmksnan_MvMl`，用于条件编译、简写或 API 生成。
- **L1111 EN**: Defines macro `_vel_pvfmksgtnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L1111 CN**: 定义宏 `_vel_pvfmksgtnan_Mvl`，用于条件编译、简写或 API 生成。
- **L1112 EN**: Defines macro `_vel_pvfmksgtnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L1112 CN**: 定义宏 `_vel_pvfmksgtnan_MvMl`，用于条件编译、简写或 API 生成。
- **L1113 EN**: Defines macro `_vel_pvfmksltnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L1113 CN**: 定义宏 `_vel_pvfmksltnan_Mvl`，用于条件编译、简写或 API 生成。
- **L1114 EN**: Defines macro `_vel_pvfmksltnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L1114 CN**: 定义宏 `_vel_pvfmksltnan_MvMl`，用于条件编译、简写或 API 生成。
- **L1115 EN**: Defines macro `_vel_pvfmksnenan_Mvl` for conditional compilation, shorthand, or API generation.
  **L1115 CN**: 定义宏 `_vel_pvfmksnenan_Mvl`，用于条件编译、简写或 API 生成。
- **L1116 EN**: Defines macro `_vel_pvfmksnenan_MvMl` for conditional compilation, shorthand, or API generation.
  **L1116 CN**: 定义宏 `_vel_pvfmksnenan_MvMl`，用于条件编译、简写或 API 生成。
- **L1117 EN**: Defines macro `_vel_pvfmkseqnan_Mvl` for conditional compilation, shorthand, or API generation.
  **L1117 CN**: 定义宏 `_vel_pvfmkseqnan_Mvl`，用于条件编译、简写或 API 生成。
- **L1118 EN**: Defines macro `_vel_pvfmkseqnan_MvMl` for conditional compilation, shorthand, or API generation.
  **L1118 CN**: 定义宏 `_vel_pvfmkseqnan_MvMl`，用于条件编译、简写或 API 生成。
- **L1119 EN**: Defines macro `_vel_pvfmksgenan_Mvl` for conditional compilation, shorthand, or API generation.
  **L1119 CN**: 定义宏 `_vel_pvfmksgenan_Mvl`，用于条件编译、简写或 API 生成。
- **L1120 EN**: Defines macro `_vel_pvfmksgenan_MvMl` for conditional compilation, shorthand, or API generation.
  **L1120 CN**: 定义宏 `_vel_pvfmksgenan_MvMl`，用于条件编译、简写或 API 生成。
- **L1121 EN**: Defines macro `_vel_pvfmkslenan_Mvl` for conditional compilation, shorthand, or API generation.
  **L1121 CN**: 定义宏 `_vel_pvfmkslenan_Mvl`，用于条件编译、简写或 API 生成。
- **L1122 EN**: Defines macro `_vel_pvfmkslenan_MvMl` for conditional compilation, shorthand, or API generation.
  **L1122 CN**: 定义宏 `_vel_pvfmkslenan_MvMl`，用于条件编译、简写或 API 生成。
- **L1123 EN**: Defines macro `_vel_vsumwsx_vvl` for conditional compilation, shorthand, or API generation.
  **L1123 CN**: 定义宏 `_vel_vsumwsx_vvl`，用于条件编译、简写或 API 生成。
- **L1124 EN**: Defines macro `_vel_vsumwsx_vvml` for conditional compilation, shorthand, or API generation.
  **L1124 CN**: 定义宏 `_vel_vsumwsx_vvml`，用于条件编译、简写或 API 生成。
- **L1125 EN**: Defines macro `_vel_vsumwzx_vvl` for conditional compilation, shorthand, or API generation.
  **L1125 CN**: 定义宏 `_vel_vsumwzx_vvl`，用于条件编译、简写或 API 生成。
- **L1126 EN**: Defines macro `_vel_vsumwzx_vvml` for conditional compilation, shorthand, or API generation.
  **L1126 CN**: 定义宏 `_vel_vsumwzx_vvml`，用于条件编译、简写或 API 生成。
- **L1127 EN**: Defines macro `_vel_vsuml_vvl` for conditional compilation, shorthand, or API generation.
  **L1127 CN**: 定义宏 `_vel_vsuml_vvl`，用于条件编译、简写或 API 生成。
- **L1128 EN**: Defines macro `_vel_vsuml_vvml` for conditional compilation, shorthand, or API generation.
  **L1128 CN**: 定义宏 `_vel_vsuml_vvml`，用于条件编译、简写或 API 生成。

### Lines 1129-1152

````c
#define _vel_vfsumd_vvl __builtin_ve_vl_vfsumd_vvl
#define _vel_vfsumd_vvml __builtin_ve_vl_vfsumd_vvml
#define _vel_vfsums_vvl __builtin_ve_vl_vfsums_vvl
#define _vel_vfsums_vvml __builtin_ve_vl_vfsums_vvml
#define _vel_vrmaxswfstsx_vvl __builtin_ve_vl_vrmaxswfstsx_vvl
#define _vel_vrmaxswfstsx_vvvl __builtin_ve_vl_vrmaxswfstsx_vvvl
#define _vel_vrmaxswlstsx_vvl __builtin_ve_vl_vrmaxswlstsx_vvl
#define _vel_vrmaxswlstsx_vvvl __builtin_ve_vl_vrmaxswlstsx_vvvl
#define _vel_vrmaxswfstzx_vvl __builtin_ve_vl_vrmaxswfstzx_vvl
#define _vel_vrmaxswfstzx_vvvl __builtin_ve_vl_vrmaxswfstzx_vvvl
#define _vel_vrmaxswlstzx_vvl __builtin_ve_vl_vrmaxswlstzx_vvl
#define _vel_vrmaxswlstzx_vvvl __builtin_ve_vl_vrmaxswlstzx_vvvl
#define _vel_vrminswfstsx_vvl __builtin_ve_vl_vrminswfstsx_vvl
#define _vel_vrminswfstsx_vvvl __builtin_ve_vl_vrminswfstsx_vvvl
#define _vel_vrminswlstsx_vvl __builtin_ve_vl_vrminswlstsx_vvl
#define _vel_vrminswlstsx_vvvl __builtin_ve_vl_vrminswlstsx_vvvl
#define _vel_vrminswfstzx_vvl __builtin_ve_vl_vrminswfstzx_vvl
#define _vel_vrminswfstzx_vvvl __builtin_ve_vl_vrminswfstzx_vvvl
#define _vel_vrminswlstzx_vvl __builtin_ve_vl_vrminswlstzx_vvl
#define _vel_vrminswlstzx_vvvl __builtin_ve_vl_vrminswlstzx_vvvl
#define _vel_vrmaxslfst_vvl __builtin_ve_vl_vrmaxslfst_vvl
#define _vel_vrmaxslfst_vvvl __builtin_ve_vl_vrmaxslfst_vvvl
#define _vel_vrmaxsllst_vvl __builtin_ve_vl_vrmaxsllst_vvl
#define _vel_vrmaxsllst_vvvl __builtin_ve_vl_vrmaxsllst_vvvl
````
- **L1129 EN**: Defines macro `_vel_vfsumd_vvl` for conditional compilation, shorthand, or API generation.
  **L1129 CN**: 定义宏 `_vel_vfsumd_vvl`，用于条件编译、简写或 API 生成。
- **L1130 EN**: Defines macro `_vel_vfsumd_vvml` for conditional compilation, shorthand, or API generation.
  **L1130 CN**: 定义宏 `_vel_vfsumd_vvml`，用于条件编译、简写或 API 生成。
- **L1131 EN**: Defines macro `_vel_vfsums_vvl` for conditional compilation, shorthand, or API generation.
  **L1131 CN**: 定义宏 `_vel_vfsums_vvl`，用于条件编译、简写或 API 生成。
- **L1132 EN**: Defines macro `_vel_vfsums_vvml` for conditional compilation, shorthand, or API generation.
  **L1132 CN**: 定义宏 `_vel_vfsums_vvml`，用于条件编译、简写或 API 生成。
- **L1133 EN**: Defines macro `_vel_vrmaxswfstsx_vvl` for conditional compilation, shorthand, or API generation.
  **L1133 CN**: 定义宏 `_vel_vrmaxswfstsx_vvl`，用于条件编译、简写或 API 生成。
- **L1134 EN**: Defines macro `_vel_vrmaxswfstsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1134 CN**: 定义宏 `_vel_vrmaxswfstsx_vvvl`，用于条件编译、简写或 API 生成。
- **L1135 EN**: Defines macro `_vel_vrmaxswlstsx_vvl` for conditional compilation, shorthand, or API generation.
  **L1135 CN**: 定义宏 `_vel_vrmaxswlstsx_vvl`，用于条件编译、简写或 API 生成。
- **L1136 EN**: Defines macro `_vel_vrmaxswlstsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1136 CN**: 定义宏 `_vel_vrmaxswlstsx_vvvl`，用于条件编译、简写或 API 生成。
- **L1137 EN**: Defines macro `_vel_vrmaxswfstzx_vvl` for conditional compilation, shorthand, or API generation.
  **L1137 CN**: 定义宏 `_vel_vrmaxswfstzx_vvl`，用于条件编译、简写或 API 生成。
- **L1138 EN**: Defines macro `_vel_vrmaxswfstzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1138 CN**: 定义宏 `_vel_vrmaxswfstzx_vvvl`，用于条件编译、简写或 API 生成。
- **L1139 EN**: Defines macro `_vel_vrmaxswlstzx_vvl` for conditional compilation, shorthand, or API generation.
  **L1139 CN**: 定义宏 `_vel_vrmaxswlstzx_vvl`，用于条件编译、简写或 API 生成。
- **L1140 EN**: Defines macro `_vel_vrmaxswlstzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1140 CN**: 定义宏 `_vel_vrmaxswlstzx_vvvl`，用于条件编译、简写或 API 生成。
- **L1141 EN**: Defines macro `_vel_vrminswfstsx_vvl` for conditional compilation, shorthand, or API generation.
  **L1141 CN**: 定义宏 `_vel_vrminswfstsx_vvl`，用于条件编译、简写或 API 生成。
- **L1142 EN**: Defines macro `_vel_vrminswfstsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1142 CN**: 定义宏 `_vel_vrminswfstsx_vvvl`，用于条件编译、简写或 API 生成。
- **L1143 EN**: Defines macro `_vel_vrminswlstsx_vvl` for conditional compilation, shorthand, or API generation.
  **L1143 CN**: 定义宏 `_vel_vrminswlstsx_vvl`，用于条件编译、简写或 API 生成。
- **L1144 EN**: Defines macro `_vel_vrminswlstsx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1144 CN**: 定义宏 `_vel_vrminswlstsx_vvvl`，用于条件编译、简写或 API 生成。
- **L1145 EN**: Defines macro `_vel_vrminswfstzx_vvl` for conditional compilation, shorthand, or API generation.
  **L1145 CN**: 定义宏 `_vel_vrminswfstzx_vvl`，用于条件编译、简写或 API 生成。
- **L1146 EN**: Defines macro `_vel_vrminswfstzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1146 CN**: 定义宏 `_vel_vrminswfstzx_vvvl`，用于条件编译、简写或 API 生成。
- **L1147 EN**: Defines macro `_vel_vrminswlstzx_vvl` for conditional compilation, shorthand, or API generation.
  **L1147 CN**: 定义宏 `_vel_vrminswlstzx_vvl`，用于条件编译、简写或 API 生成。
- **L1148 EN**: Defines macro `_vel_vrminswlstzx_vvvl` for conditional compilation, shorthand, or API generation.
  **L1148 CN**: 定义宏 `_vel_vrminswlstzx_vvvl`，用于条件编译、简写或 API 生成。
- **L1149 EN**: Defines macro `_vel_vrmaxslfst_vvl` for conditional compilation, shorthand, or API generation.
  **L1149 CN**: 定义宏 `_vel_vrmaxslfst_vvl`，用于条件编译、简写或 API 生成。
- **L1150 EN**: Defines macro `_vel_vrmaxslfst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1150 CN**: 定义宏 `_vel_vrmaxslfst_vvvl`，用于条件编译、简写或 API 生成。
- **L1151 EN**: Defines macro `_vel_vrmaxsllst_vvl` for conditional compilation, shorthand, or API generation.
  **L1151 CN**: 定义宏 `_vel_vrmaxsllst_vvl`，用于条件编译、简写或 API 生成。
- **L1152 EN**: Defines macro `_vel_vrmaxsllst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1152 CN**: 定义宏 `_vel_vrmaxsllst_vvvl`，用于条件编译、简写或 API 生成。

### Lines 1153-1176

````c
#define _vel_vrminslfst_vvl __builtin_ve_vl_vrminslfst_vvl
#define _vel_vrminslfst_vvvl __builtin_ve_vl_vrminslfst_vvvl
#define _vel_vrminsllst_vvl __builtin_ve_vl_vrminsllst_vvl
#define _vel_vrminsllst_vvvl __builtin_ve_vl_vrminsllst_vvvl
#define _vel_vfrmaxdfst_vvl __builtin_ve_vl_vfrmaxdfst_vvl
#define _vel_vfrmaxdfst_vvvl __builtin_ve_vl_vfrmaxdfst_vvvl
#define _vel_vfrmaxdlst_vvl __builtin_ve_vl_vfrmaxdlst_vvl
#define _vel_vfrmaxdlst_vvvl __builtin_ve_vl_vfrmaxdlst_vvvl
#define _vel_vfrmaxsfst_vvl __builtin_ve_vl_vfrmaxsfst_vvl
#define _vel_vfrmaxsfst_vvvl __builtin_ve_vl_vfrmaxsfst_vvvl
#define _vel_vfrmaxslst_vvl __builtin_ve_vl_vfrmaxslst_vvl
#define _vel_vfrmaxslst_vvvl __builtin_ve_vl_vfrmaxslst_vvvl
#define _vel_vfrmindfst_vvl __builtin_ve_vl_vfrmindfst_vvl
#define _vel_vfrmindfst_vvvl __builtin_ve_vl_vfrmindfst_vvvl
#define _vel_vfrmindlst_vvl __builtin_ve_vl_vfrmindlst_vvl
#define _vel_vfrmindlst_vvvl __builtin_ve_vl_vfrmindlst_vvvl
#define _vel_vfrminsfst_vvl __builtin_ve_vl_vfrminsfst_vvl
#define _vel_vfrminsfst_vvvl __builtin_ve_vl_vfrminsfst_vvvl
#define _vel_vfrminslst_vvl __builtin_ve_vl_vfrminslst_vvl
#define _vel_vfrminslst_vvvl __builtin_ve_vl_vfrminslst_vvvl
#define _vel_vrand_vvl __builtin_ve_vl_vrand_vvl
#define _vel_vrand_vvml __builtin_ve_vl_vrand_vvml
#define _vel_vror_vvl __builtin_ve_vl_vror_vvl
#define _vel_vror_vvml __builtin_ve_vl_vror_vvml
````
- **L1153 EN**: Defines macro `_vel_vrminslfst_vvl` for conditional compilation, shorthand, or API generation.
  **L1153 CN**: 定义宏 `_vel_vrminslfst_vvl`，用于条件编译、简写或 API 生成。
- **L1154 EN**: Defines macro `_vel_vrminslfst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1154 CN**: 定义宏 `_vel_vrminslfst_vvvl`，用于条件编译、简写或 API 生成。
- **L1155 EN**: Defines macro `_vel_vrminsllst_vvl` for conditional compilation, shorthand, or API generation.
  **L1155 CN**: 定义宏 `_vel_vrminsllst_vvl`，用于条件编译、简写或 API 生成。
- **L1156 EN**: Defines macro `_vel_vrminsllst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1156 CN**: 定义宏 `_vel_vrminsllst_vvvl`，用于条件编译、简写或 API 生成。
- **L1157 EN**: Defines macro `_vel_vfrmaxdfst_vvl` for conditional compilation, shorthand, or API generation.
  **L1157 CN**: 定义宏 `_vel_vfrmaxdfst_vvl`，用于条件编译、简写或 API 生成。
- **L1158 EN**: Defines macro `_vel_vfrmaxdfst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1158 CN**: 定义宏 `_vel_vfrmaxdfst_vvvl`，用于条件编译、简写或 API 生成。
- **L1159 EN**: Defines macro `_vel_vfrmaxdlst_vvl` for conditional compilation, shorthand, or API generation.
  **L1159 CN**: 定义宏 `_vel_vfrmaxdlst_vvl`，用于条件编译、简写或 API 生成。
- **L1160 EN**: Defines macro `_vel_vfrmaxdlst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1160 CN**: 定义宏 `_vel_vfrmaxdlst_vvvl`，用于条件编译、简写或 API 生成。
- **L1161 EN**: Defines macro `_vel_vfrmaxsfst_vvl` for conditional compilation, shorthand, or API generation.
  **L1161 CN**: 定义宏 `_vel_vfrmaxsfst_vvl`，用于条件编译、简写或 API 生成。
- **L1162 EN**: Defines macro `_vel_vfrmaxsfst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1162 CN**: 定义宏 `_vel_vfrmaxsfst_vvvl`，用于条件编译、简写或 API 生成。
- **L1163 EN**: Defines macro `_vel_vfrmaxslst_vvl` for conditional compilation, shorthand, or API generation.
  **L1163 CN**: 定义宏 `_vel_vfrmaxslst_vvl`，用于条件编译、简写或 API 生成。
- **L1164 EN**: Defines macro `_vel_vfrmaxslst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1164 CN**: 定义宏 `_vel_vfrmaxslst_vvvl`，用于条件编译、简写或 API 生成。
- **L1165 EN**: Defines macro `_vel_vfrmindfst_vvl` for conditional compilation, shorthand, or API generation.
  **L1165 CN**: 定义宏 `_vel_vfrmindfst_vvl`，用于条件编译、简写或 API 生成。
- **L1166 EN**: Defines macro `_vel_vfrmindfst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1166 CN**: 定义宏 `_vel_vfrmindfst_vvvl`，用于条件编译、简写或 API 生成。
- **L1167 EN**: Defines macro `_vel_vfrmindlst_vvl` for conditional compilation, shorthand, or API generation.
  **L1167 CN**: 定义宏 `_vel_vfrmindlst_vvl`，用于条件编译、简写或 API 生成。
- **L1168 EN**: Defines macro `_vel_vfrmindlst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1168 CN**: 定义宏 `_vel_vfrmindlst_vvvl`，用于条件编译、简写或 API 生成。
- **L1169 EN**: Defines macro `_vel_vfrminsfst_vvl` for conditional compilation, shorthand, or API generation.
  **L1169 CN**: 定义宏 `_vel_vfrminsfst_vvl`，用于条件编译、简写或 API 生成。
- **L1170 EN**: Defines macro `_vel_vfrminsfst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1170 CN**: 定义宏 `_vel_vfrminsfst_vvvl`，用于条件编译、简写或 API 生成。
- **L1171 EN**: Defines macro `_vel_vfrminslst_vvl` for conditional compilation, shorthand, or API generation.
  **L1171 CN**: 定义宏 `_vel_vfrminslst_vvl`，用于条件编译、简写或 API 生成。
- **L1172 EN**: Defines macro `_vel_vfrminslst_vvvl` for conditional compilation, shorthand, or API generation.
  **L1172 CN**: 定义宏 `_vel_vfrminslst_vvvl`，用于条件编译、简写或 API 生成。
- **L1173 EN**: Defines macro `_vel_vrand_vvl` for conditional compilation, shorthand, or API generation.
  **L1173 CN**: 定义宏 `_vel_vrand_vvl`，用于条件编译、简写或 API 生成。
- **L1174 EN**: Defines macro `_vel_vrand_vvml` for conditional compilation, shorthand, or API generation.
  **L1174 CN**: 定义宏 `_vel_vrand_vvml`，用于条件编译、简写或 API 生成。
- **L1175 EN**: Defines macro `_vel_vror_vvl` for conditional compilation, shorthand, or API generation.
  **L1175 CN**: 定义宏 `_vel_vror_vvl`，用于条件编译、简写或 API 生成。
- **L1176 EN**: Defines macro `_vel_vror_vvml` for conditional compilation, shorthand, or API generation.
  **L1176 CN**: 定义宏 `_vel_vror_vvml`，用于条件编译、简写或 API 生成。

### Lines 1177-1200

````c
#define _vel_vrxor_vvl __builtin_ve_vl_vrxor_vvl
#define _vel_vrxor_vvml __builtin_ve_vl_vrxor_vvml
#define _vel_vgt_vvssl __builtin_ve_vl_vgt_vvssl
#define _vel_vgt_vvssvl __builtin_ve_vl_vgt_vvssvl
#define _vel_vgt_vvssml __builtin_ve_vl_vgt_vvssml
#define _vel_vgt_vvssmvl __builtin_ve_vl_vgt_vvssmvl
#define _vel_vgtnc_vvssl __builtin_ve_vl_vgtnc_vvssl
#define _vel_vgtnc_vvssvl __builtin_ve_vl_vgtnc_vvssvl
#define _vel_vgtnc_vvssml __builtin_ve_vl_vgtnc_vvssml
#define _vel_vgtnc_vvssmvl __builtin_ve_vl_vgtnc_vvssmvl
#define _vel_vgtu_vvssl __builtin_ve_vl_vgtu_vvssl
#define _vel_vgtu_vvssvl __builtin_ve_vl_vgtu_vvssvl
#define _vel_vgtu_vvssml __builtin_ve_vl_vgtu_vvssml
#define _vel_vgtu_vvssmvl __builtin_ve_vl_vgtu_vvssmvl
#define _vel_vgtunc_vvssl __builtin_ve_vl_vgtunc_vvssl
#define _vel_vgtunc_vvssvl __builtin_ve_vl_vgtunc_vvssvl
#define _vel_vgtunc_vvssml __builtin_ve_vl_vgtunc_vvssml
#define _vel_vgtunc_vvssmvl __builtin_ve_vl_vgtunc_vvssmvl
#define _vel_vgtlsx_vvssl __builtin_ve_vl_vgtlsx_vvssl
#define _vel_vgtlsx_vvssvl __builtin_ve_vl_vgtlsx_vvssvl
#define _vel_vgtlsx_vvssml __builtin_ve_vl_vgtlsx_vvssml
#define _vel_vgtlsx_vvssmvl __builtin_ve_vl_vgtlsx_vvssmvl
#define _vel_vgtlsxnc_vvssl __builtin_ve_vl_vgtlsxnc_vvssl
#define _vel_vgtlsxnc_vvssvl __builtin_ve_vl_vgtlsxnc_vvssvl
````
- **L1177 EN**: Defines macro `_vel_vrxor_vvl` for conditional compilation, shorthand, or API generation.
  **L1177 CN**: 定义宏 `_vel_vrxor_vvl`，用于条件编译、简写或 API 生成。
- **L1178 EN**: Defines macro `_vel_vrxor_vvml` for conditional compilation, shorthand, or API generation.
  **L1178 CN**: 定义宏 `_vel_vrxor_vvml`，用于条件编译、简写或 API 生成。
- **L1179 EN**: Defines macro `_vel_vgt_vvssl` for conditional compilation, shorthand, or API generation.
  **L1179 CN**: 定义宏 `_vel_vgt_vvssl`，用于条件编译、简写或 API 生成。
- **L1180 EN**: Defines macro `_vel_vgt_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1180 CN**: 定义宏 `_vel_vgt_vvssvl`，用于条件编译、简写或 API 生成。
- **L1181 EN**: Defines macro `_vel_vgt_vvssml` for conditional compilation, shorthand, or API generation.
  **L1181 CN**: 定义宏 `_vel_vgt_vvssml`，用于条件编译、简写或 API 生成。
- **L1182 EN**: Defines macro `_vel_vgt_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1182 CN**: 定义宏 `_vel_vgt_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1183 EN**: Defines macro `_vel_vgtnc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1183 CN**: 定义宏 `_vel_vgtnc_vvssl`，用于条件编译、简写或 API 生成。
- **L1184 EN**: Defines macro `_vel_vgtnc_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1184 CN**: 定义宏 `_vel_vgtnc_vvssvl`，用于条件编译、简写或 API 生成。
- **L1185 EN**: Defines macro `_vel_vgtnc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1185 CN**: 定义宏 `_vel_vgtnc_vvssml`，用于条件编译、简写或 API 生成。
- **L1186 EN**: Defines macro `_vel_vgtnc_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1186 CN**: 定义宏 `_vel_vgtnc_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1187 EN**: Defines macro `_vel_vgtu_vvssl` for conditional compilation, shorthand, or API generation.
  **L1187 CN**: 定义宏 `_vel_vgtu_vvssl`，用于条件编译、简写或 API 生成。
- **L1188 EN**: Defines macro `_vel_vgtu_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1188 CN**: 定义宏 `_vel_vgtu_vvssvl`，用于条件编译、简写或 API 生成。
- **L1189 EN**: Defines macro `_vel_vgtu_vvssml` for conditional compilation, shorthand, or API generation.
  **L1189 CN**: 定义宏 `_vel_vgtu_vvssml`，用于条件编译、简写或 API 生成。
- **L1190 EN**: Defines macro `_vel_vgtu_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1190 CN**: 定义宏 `_vel_vgtu_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1191 EN**: Defines macro `_vel_vgtunc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1191 CN**: 定义宏 `_vel_vgtunc_vvssl`，用于条件编译、简写或 API 生成。
- **L1192 EN**: Defines macro `_vel_vgtunc_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1192 CN**: 定义宏 `_vel_vgtunc_vvssvl`，用于条件编译、简写或 API 生成。
- **L1193 EN**: Defines macro `_vel_vgtunc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1193 CN**: 定义宏 `_vel_vgtunc_vvssml`，用于条件编译、简写或 API 生成。
- **L1194 EN**: Defines macro `_vel_vgtunc_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1194 CN**: 定义宏 `_vel_vgtunc_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1195 EN**: Defines macro `_vel_vgtlsx_vvssl` for conditional compilation, shorthand, or API generation.
  **L1195 CN**: 定义宏 `_vel_vgtlsx_vvssl`，用于条件编译、简写或 API 生成。
- **L1196 EN**: Defines macro `_vel_vgtlsx_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1196 CN**: 定义宏 `_vel_vgtlsx_vvssvl`，用于条件编译、简写或 API 生成。
- **L1197 EN**: Defines macro `_vel_vgtlsx_vvssml` for conditional compilation, shorthand, or API generation.
  **L1197 CN**: 定义宏 `_vel_vgtlsx_vvssml`，用于条件编译、简写或 API 生成。
- **L1198 EN**: Defines macro `_vel_vgtlsx_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1198 CN**: 定义宏 `_vel_vgtlsx_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1199 EN**: Defines macro `_vel_vgtlsxnc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1199 CN**: 定义宏 `_vel_vgtlsxnc_vvssl`，用于条件编译、简写或 API 生成。
- **L1200 EN**: Defines macro `_vel_vgtlsxnc_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1200 CN**: 定义宏 `_vel_vgtlsxnc_vvssvl`，用于条件编译、简写或 API 生成。

### Lines 1201-1224

````c
#define _vel_vgtlsxnc_vvssml __builtin_ve_vl_vgtlsxnc_vvssml
#define _vel_vgtlsxnc_vvssmvl __builtin_ve_vl_vgtlsxnc_vvssmvl
#define _vel_vgtlzx_vvssl __builtin_ve_vl_vgtlzx_vvssl
#define _vel_vgtlzx_vvssvl __builtin_ve_vl_vgtlzx_vvssvl
#define _vel_vgtlzx_vvssml __builtin_ve_vl_vgtlzx_vvssml
#define _vel_vgtlzx_vvssmvl __builtin_ve_vl_vgtlzx_vvssmvl
#define _vel_vgtlzxnc_vvssl __builtin_ve_vl_vgtlzxnc_vvssl
#define _vel_vgtlzxnc_vvssvl __builtin_ve_vl_vgtlzxnc_vvssvl
#define _vel_vgtlzxnc_vvssml __builtin_ve_vl_vgtlzxnc_vvssml
#define _vel_vgtlzxnc_vvssmvl __builtin_ve_vl_vgtlzxnc_vvssmvl
#define _vel_vsc_vvssl __builtin_ve_vl_vsc_vvssl
#define _vel_vsc_vvssml __builtin_ve_vl_vsc_vvssml
#define _vel_vscnc_vvssl __builtin_ve_vl_vscnc_vvssl
#define _vel_vscnc_vvssml __builtin_ve_vl_vscnc_vvssml
#define _vel_vscot_vvssl __builtin_ve_vl_vscot_vvssl
#define _vel_vscot_vvssml __builtin_ve_vl_vscot_vvssml
#define _vel_vscncot_vvssl __builtin_ve_vl_vscncot_vvssl
#define _vel_vscncot_vvssml __builtin_ve_vl_vscncot_vvssml
#define _vel_vscu_vvssl __builtin_ve_vl_vscu_vvssl
#define _vel_vscu_vvssml __builtin_ve_vl_vscu_vvssml
#define _vel_vscunc_vvssl __builtin_ve_vl_vscunc_vvssl
#define _vel_vscunc_vvssml __builtin_ve_vl_vscunc_vvssml
#define _vel_vscuot_vvssl __builtin_ve_vl_vscuot_vvssl
#define _vel_vscuot_vvssml __builtin_ve_vl_vscuot_vvssml
````
- **L1201 EN**: Defines macro `_vel_vgtlsxnc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1201 CN**: 定义宏 `_vel_vgtlsxnc_vvssml`，用于条件编译、简写或 API 生成。
- **L1202 EN**: Defines macro `_vel_vgtlsxnc_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1202 CN**: 定义宏 `_vel_vgtlsxnc_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1203 EN**: Defines macro `_vel_vgtlzx_vvssl` for conditional compilation, shorthand, or API generation.
  **L1203 CN**: 定义宏 `_vel_vgtlzx_vvssl`，用于条件编译、简写或 API 生成。
- **L1204 EN**: Defines macro `_vel_vgtlzx_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1204 CN**: 定义宏 `_vel_vgtlzx_vvssvl`，用于条件编译、简写或 API 生成。
- **L1205 EN**: Defines macro `_vel_vgtlzx_vvssml` for conditional compilation, shorthand, or API generation.
  **L1205 CN**: 定义宏 `_vel_vgtlzx_vvssml`，用于条件编译、简写或 API 生成。
- **L1206 EN**: Defines macro `_vel_vgtlzx_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1206 CN**: 定义宏 `_vel_vgtlzx_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1207 EN**: Defines macro `_vel_vgtlzxnc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1207 CN**: 定义宏 `_vel_vgtlzxnc_vvssl`，用于条件编译、简写或 API 生成。
- **L1208 EN**: Defines macro `_vel_vgtlzxnc_vvssvl` for conditional compilation, shorthand, or API generation.
  **L1208 CN**: 定义宏 `_vel_vgtlzxnc_vvssvl`，用于条件编译、简写或 API 生成。
- **L1209 EN**: Defines macro `_vel_vgtlzxnc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1209 CN**: 定义宏 `_vel_vgtlzxnc_vvssml`，用于条件编译、简写或 API 生成。
- **L1210 EN**: Defines macro `_vel_vgtlzxnc_vvssmvl` for conditional compilation, shorthand, or API generation.
  **L1210 CN**: 定义宏 `_vel_vgtlzxnc_vvssmvl`，用于条件编译、简写或 API 生成。
- **L1211 EN**: Defines macro `_vel_vsc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1211 CN**: 定义宏 `_vel_vsc_vvssl`，用于条件编译、简写或 API 生成。
- **L1212 EN**: Defines macro `_vel_vsc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1212 CN**: 定义宏 `_vel_vsc_vvssml`，用于条件编译、简写或 API 生成。
- **L1213 EN**: Defines macro `_vel_vscnc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1213 CN**: 定义宏 `_vel_vscnc_vvssl`，用于条件编译、简写或 API 生成。
- **L1214 EN**: Defines macro `_vel_vscnc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1214 CN**: 定义宏 `_vel_vscnc_vvssml`，用于条件编译、简写或 API 生成。
- **L1215 EN**: Defines macro `_vel_vscot_vvssl` for conditional compilation, shorthand, or API generation.
  **L1215 CN**: 定义宏 `_vel_vscot_vvssl`，用于条件编译、简写或 API 生成。
- **L1216 EN**: Defines macro `_vel_vscot_vvssml` for conditional compilation, shorthand, or API generation.
  **L1216 CN**: 定义宏 `_vel_vscot_vvssml`，用于条件编译、简写或 API 生成。
- **L1217 EN**: Defines macro `_vel_vscncot_vvssl` for conditional compilation, shorthand, or API generation.
  **L1217 CN**: 定义宏 `_vel_vscncot_vvssl`，用于条件编译、简写或 API 生成。
- **L1218 EN**: Defines macro `_vel_vscncot_vvssml` for conditional compilation, shorthand, or API generation.
  **L1218 CN**: 定义宏 `_vel_vscncot_vvssml`，用于条件编译、简写或 API 生成。
- **L1219 EN**: Defines macro `_vel_vscu_vvssl` for conditional compilation, shorthand, or API generation.
  **L1219 CN**: 定义宏 `_vel_vscu_vvssl`，用于条件编译、简写或 API 生成。
- **L1220 EN**: Defines macro `_vel_vscu_vvssml` for conditional compilation, shorthand, or API generation.
  **L1220 CN**: 定义宏 `_vel_vscu_vvssml`，用于条件编译、简写或 API 生成。
- **L1221 EN**: Defines macro `_vel_vscunc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1221 CN**: 定义宏 `_vel_vscunc_vvssl`，用于条件编译、简写或 API 生成。
- **L1222 EN**: Defines macro `_vel_vscunc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1222 CN**: 定义宏 `_vel_vscunc_vvssml`，用于条件编译、简写或 API 生成。
- **L1223 EN**: Defines macro `_vel_vscuot_vvssl` for conditional compilation, shorthand, or API generation.
  **L1223 CN**: 定义宏 `_vel_vscuot_vvssl`，用于条件编译、简写或 API 生成。
- **L1224 EN**: Defines macro `_vel_vscuot_vvssml` for conditional compilation, shorthand, or API generation.
  **L1224 CN**: 定义宏 `_vel_vscuot_vvssml`，用于条件编译、简写或 API 生成。

### Lines 1225-1248

````c
#define _vel_vscuncot_vvssl __builtin_ve_vl_vscuncot_vvssl
#define _vel_vscuncot_vvssml __builtin_ve_vl_vscuncot_vvssml
#define _vel_vscl_vvssl __builtin_ve_vl_vscl_vvssl
#define _vel_vscl_vvssml __builtin_ve_vl_vscl_vvssml
#define _vel_vsclnc_vvssl __builtin_ve_vl_vsclnc_vvssl
#define _vel_vsclnc_vvssml __builtin_ve_vl_vsclnc_vvssml
#define _vel_vsclot_vvssl __builtin_ve_vl_vsclot_vvssl
#define _vel_vsclot_vvssml __builtin_ve_vl_vsclot_vvssml
#define _vel_vsclncot_vvssl __builtin_ve_vl_vsclncot_vvssl
#define _vel_vsclncot_vvssml __builtin_ve_vl_vsclncot_vvssml
#define _vel_andm_mmm __builtin_ve_vl_andm_mmm
#define _vel_andm_MMM __builtin_ve_vl_andm_MMM
#define _vel_orm_mmm __builtin_ve_vl_orm_mmm
#define _vel_orm_MMM __builtin_ve_vl_orm_MMM
#define _vel_xorm_mmm __builtin_ve_vl_xorm_mmm
#define _vel_xorm_MMM __builtin_ve_vl_xorm_MMM
#define _vel_eqvm_mmm __builtin_ve_vl_eqvm_mmm
#define _vel_eqvm_MMM __builtin_ve_vl_eqvm_MMM
#define _vel_nndm_mmm __builtin_ve_vl_nndm_mmm
#define _vel_nndm_MMM __builtin_ve_vl_nndm_MMM
#define _vel_negm_mm __builtin_ve_vl_negm_mm
#define _vel_negm_MM __builtin_ve_vl_negm_MM
#define _vel_pcvm_sml __builtin_ve_vl_pcvm_sml
#define _vel_lzvm_sml __builtin_ve_vl_lzvm_sml
````
- **L1225 EN**: Defines macro `_vel_vscuncot_vvssl` for conditional compilation, shorthand, or API generation.
  **L1225 CN**: 定义宏 `_vel_vscuncot_vvssl`，用于条件编译、简写或 API 生成。
- **L1226 EN**: Defines macro `_vel_vscuncot_vvssml` for conditional compilation, shorthand, or API generation.
  **L1226 CN**: 定义宏 `_vel_vscuncot_vvssml`，用于条件编译、简写或 API 生成。
- **L1227 EN**: Defines macro `_vel_vscl_vvssl` for conditional compilation, shorthand, or API generation.
  **L1227 CN**: 定义宏 `_vel_vscl_vvssl`，用于条件编译、简写或 API 生成。
- **L1228 EN**: Defines macro `_vel_vscl_vvssml` for conditional compilation, shorthand, or API generation.
  **L1228 CN**: 定义宏 `_vel_vscl_vvssml`，用于条件编译、简写或 API 生成。
- **L1229 EN**: Defines macro `_vel_vsclnc_vvssl` for conditional compilation, shorthand, or API generation.
  **L1229 CN**: 定义宏 `_vel_vsclnc_vvssl`，用于条件编译、简写或 API 生成。
- **L1230 EN**: Defines macro `_vel_vsclnc_vvssml` for conditional compilation, shorthand, or API generation.
  **L1230 CN**: 定义宏 `_vel_vsclnc_vvssml`，用于条件编译、简写或 API 生成。
- **L1231 EN**: Defines macro `_vel_vsclot_vvssl` for conditional compilation, shorthand, or API generation.
  **L1231 CN**: 定义宏 `_vel_vsclot_vvssl`，用于条件编译、简写或 API 生成。
- **L1232 EN**: Defines macro `_vel_vsclot_vvssml` for conditional compilation, shorthand, or API generation.
  **L1232 CN**: 定义宏 `_vel_vsclot_vvssml`，用于条件编译、简写或 API 生成。
- **L1233 EN**: Defines macro `_vel_vsclncot_vvssl` for conditional compilation, shorthand, or API generation.
  **L1233 CN**: 定义宏 `_vel_vsclncot_vvssl`，用于条件编译、简写或 API 生成。
- **L1234 EN**: Defines macro `_vel_vsclncot_vvssml` for conditional compilation, shorthand, or API generation.
  **L1234 CN**: 定义宏 `_vel_vsclncot_vvssml`，用于条件编译、简写或 API 生成。
- **L1235 EN**: Defines macro `_vel_andm_mmm` for conditional compilation, shorthand, or API generation.
  **L1235 CN**: 定义宏 `_vel_andm_mmm`，用于条件编译、简写或 API 生成。
- **L1236 EN**: Defines macro `_vel_andm_MMM` for conditional compilation, shorthand, or API generation.
  **L1236 CN**: 定义宏 `_vel_andm_MMM`，用于条件编译、简写或 API 生成。
- **L1237 EN**: Defines macro `_vel_orm_mmm` for conditional compilation, shorthand, or API generation.
  **L1237 CN**: 定义宏 `_vel_orm_mmm`，用于条件编译、简写或 API 生成。
- **L1238 EN**: Defines macro `_vel_orm_MMM` for conditional compilation, shorthand, or API generation.
  **L1238 CN**: 定义宏 `_vel_orm_MMM`，用于条件编译、简写或 API 生成。
- **L1239 EN**: Defines macro `_vel_xorm_mmm` for conditional compilation, shorthand, or API generation.
  **L1239 CN**: 定义宏 `_vel_xorm_mmm`，用于条件编译、简写或 API 生成。
- **L1240 EN**: Defines macro `_vel_xorm_MMM` for conditional compilation, shorthand, or API generation.
  **L1240 CN**: 定义宏 `_vel_xorm_MMM`，用于条件编译、简写或 API 生成。
- **L1241 EN**: Defines macro `_vel_eqvm_mmm` for conditional compilation, shorthand, or API generation.
  **L1241 CN**: 定义宏 `_vel_eqvm_mmm`，用于条件编译、简写或 API 生成。
- **L1242 EN**: Defines macro `_vel_eqvm_MMM` for conditional compilation, shorthand, or API generation.
  **L1242 CN**: 定义宏 `_vel_eqvm_MMM`，用于条件编译、简写或 API 生成。
- **L1243 EN**: Defines macro `_vel_nndm_mmm` for conditional compilation, shorthand, or API generation.
  **L1243 CN**: 定义宏 `_vel_nndm_mmm`，用于条件编译、简写或 API 生成。
- **L1244 EN**: Defines macro `_vel_nndm_MMM` for conditional compilation, shorthand, or API generation.
  **L1244 CN**: 定义宏 `_vel_nndm_MMM`，用于条件编译、简写或 API 生成。
- **L1245 EN**: Defines macro `_vel_negm_mm` for conditional compilation, shorthand, or API generation.
  **L1245 CN**: 定义宏 `_vel_negm_mm`，用于条件编译、简写或 API 生成。
- **L1246 EN**: Defines macro `_vel_negm_MM` for conditional compilation, shorthand, or API generation.
  **L1246 CN**: 定义宏 `_vel_negm_MM`，用于条件编译、简写或 API 生成。
- **L1247 EN**: Defines macro `_vel_pcvm_sml` for conditional compilation, shorthand, or API generation.
  **L1247 CN**: 定义宏 `_vel_pcvm_sml`，用于条件编译、简写或 API 生成。
- **L1248 EN**: Defines macro `_vel_lzvm_sml` for conditional compilation, shorthand, or API generation.
  **L1248 CN**: 定义宏 `_vel_lzvm_sml`，用于条件编译、简写或 API 生成。

### Lines 1249-1257

````c
#define _vel_tovm_sml __builtin_ve_vl_tovm_sml
#define _vel_lcr_sss __builtin_ve_vl_lcr_sss
#define _vel_scr_sss __builtin_ve_vl_scr_sss
#define _vel_tscr_ssss __builtin_ve_vl_tscr_ssss
#define _vel_fidcr_sss __builtin_ve_vl_fidcr_sss
#define _vel_fencei __builtin_ve_vl_fencei
#define _vel_fencem_s __builtin_ve_vl_fencem_s
#define _vel_fencec_s __builtin_ve_vl_fencec_s
#define _vel_svob __builtin_ve_vl_svob
````
- **L1249 EN**: Defines macro `_vel_tovm_sml` for conditional compilation, shorthand, or API generation.
  **L1249 CN**: 定义宏 `_vel_tovm_sml`，用于条件编译、简写或 API 生成。
- **L1250 EN**: Defines macro `_vel_lcr_sss` for conditional compilation, shorthand, or API generation.
  **L1250 CN**: 定义宏 `_vel_lcr_sss`，用于条件编译、简写或 API 生成。
- **L1251 EN**: Defines macro `_vel_scr_sss` for conditional compilation, shorthand, or API generation.
  **L1251 CN**: 定义宏 `_vel_scr_sss`，用于条件编译、简写或 API 生成。
- **L1252 EN**: Defines macro `_vel_tscr_ssss` for conditional compilation, shorthand, or API generation.
  **L1252 CN**: 定义宏 `_vel_tscr_ssss`，用于条件编译、简写或 API 生成。
- **L1253 EN**: Defines macro `_vel_fidcr_sss` for conditional compilation, shorthand, or API generation.
  **L1253 CN**: 定义宏 `_vel_fidcr_sss`，用于条件编译、简写或 API 生成。
- **L1254 EN**: Defines macro `_vel_fencei` for conditional compilation, shorthand, or API generation.
  **L1254 CN**: 定义宏 `_vel_fencei`，用于条件编译、简写或 API 生成。
- **L1255 EN**: Defines macro `_vel_fencem_s` for conditional compilation, shorthand, or API generation.
  **L1255 CN**: 定义宏 `_vel_fencem_s`，用于条件编译、简写或 API 生成。
- **L1256 EN**: Defines macro `_vel_fencec_s` for conditional compilation, shorthand, or API generation.
  **L1256 CN**: 定义宏 `_vel_fencec_s`，用于条件编译、简写或 API 生成。
- **L1257 EN**: Defines macro `_vel_svob` for conditional compilation, shorthand, or API generation.
  **L1257 CN**: 定义宏 `_vel_svob`，用于条件编译、简写或 API 生成。

## Key Concepts / 关键概念

- **Clang public headers / Clang 公共头文件**
- **VE vector interfaces / VE 向量接口**
- **Compiler builtin forwarding / 编译器 builtin 转发**
- **Macro-based API construction / 基于宏的 API 构造**

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None. / 无。
- **Conditional macros / 条件宏**: No prominent feature guards detected. / 未检测到明显的特性保护宏。
- **External builtins / 外部 builtin**: `__builtin_ve_vl_vld_vssl`, `__builtin_ve_vl_vld_vssvl`, `__builtin_ve_vl_vldnc_vssl`, `__builtin_ve_vl_vldnc_vssvl`, `__builtin_ve_vl_vldu_vssl`, `__builtin_ve_vl_vldu_vssvl`, `__builtin_ve_vl_vldunc_vssl`, `__builtin_ve_vl_vldunc_vssvl`, `__builtin_ve_vl_vldlsx_vssl`, `__builtin_ve_vl_vldlsx_vssvl`, `__builtin_ve_vl_vldlsxnc_vssl`, `__builtin_ve_vl_vldlsxnc_vssvl`
