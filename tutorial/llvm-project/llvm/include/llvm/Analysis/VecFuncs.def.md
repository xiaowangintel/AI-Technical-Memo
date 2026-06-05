# VecFuncs.def — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `llvm/include/llvm/Analysis/VecFuncs.def`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 目的**: This definition file lists reusable Library information entries for X-macro style expansion in LLVM's analysis interfaces and cached program facts layer. / 该定义文件在 LLVM 的分析接口与缓存的程序事实层中列出可复用条目，供 X-macro 风格展开生成 VecFuncs 相关逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28

```cpp
//===-- VecFuncs.def - Library information -------------*- C++ -*-----------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

// This .def file will create mappings from scalar math functions to vector
// functions along with their vectorization factor. The current support includes
// such mappings for Accelerate framework, MASS vector library, and SVML library.
// This .def file also allows creating an array of vector functions supported in
// the specified framework or library.

#define FIXED(NL) ElementCount::getFixed(NL)
#define SCALABLE(NL) ElementCount::getScalable(NL)
#define NOMASK false
#define MASKED true
#define NOCC std::nullopt

#if !(defined(TLI_DEFINE_VECFUNC))
#define TLI_DEFINE_VECFUNC(SCAL, VEC, VF, VABI_PREFIX) {SCAL, VEC, VF, NOMASK, VABI_PREFIX, NOCC},
#endif

#if defined(TLI_DEFINE_ACCELERATE_VECFUNCS)
// Accelerate framework's Vector Functions

// Floating-Point Arithmetic and Auxiliary Functions
```

- **L1**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L2**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L3**: Comment documents the nearby API, invariant, or algorithmic intent: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`. / 这行注释说明了附近 API、不变量或算法意图：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L4**: Comment documents the nearby API, invariant, or algorithmic intent: `See https://llvm.org/LICENSE.txt for license information.`. / 这行注释说明了附近 API、不变量或算法意图：`See https://llvm.org/LICENSE.txt for license information.`。
- **L5**: Comment documents the nearby API, invariant, or algorithmic intent: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`. / 这行注释说明了附近 API、不变量或算法意图：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L6**: Separator comment used to visually divide nearby sections. / 分隔性注释，用于在视觉上划分附近的小节。
- **L7**: Banner comment that marks a major file section boundary. / 横幅注释，用于标记文件中的主要分节边界。
- **L8**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L9**: Comment documents the nearby API, invariant, or algorithmic intent: `This .def file will create mappings from scalar math functions to vector`. / 这行注释说明了附近 API、不变量或算法意图：`This .def file will create mappings from scalar math functions to vector`。
- **L10**: Comment documents the nearby API, invariant, or algorithmic intent: `functions along with their vectorization factor. The current support includes`. / 这行注释说明了附近 API、不变量或算法意图：`functions along with their vectorization factor. The current support includes`。
- **L11**: Comment documents the nearby API, invariant, or algorithmic intent: `such mappings for Accelerate framework, MASS vector library, and SVML library.`. / 这行注释说明了附近 API、不变量或算法意图：`such mappings for Accelerate framework, MASS vector library, and SVML library.`。
- **L12**: Comment documents the nearby API, invariant, or algorithmic intent: `This .def file also allows creating an array of vector functions supported in`. / 这行注释说明了附近 API、不变量或算法意图：`This .def file also allows creating an array of vector functions supported in`。
- **L13**: Comment documents the nearby API, invariant, or algorithmic intent: `the specified framework or library.`. / 这行注释说明了附近 API、不变量或算法意图：`the specified framework or library.`。
- **L14**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L15**: Defines macro `FIXED` for later conditional compilation, generated entries, or annotations. / 定义宏 `FIXED`，供后续条件编译、生成条目或注解使用。
- **L16**: Defines macro `SCALABLE` for later conditional compilation, generated entries, or annotations. / 定义宏 `SCALABLE`，供后续条件编译、生成条目或注解使用。
- **L17**: Defines macro `NOMASK` for later conditional compilation, generated entries, or annotations. / 定义宏 `NOMASK`，供后续条件编译、生成条目或注解使用。
- **L18**: Defines macro `MASKED` for later conditional compilation, generated entries, or annotations. / 定义宏 `MASKED`，供后续条件编译、生成条目或注解使用。
- **L19**: Defines macro `NOCC` for later conditional compilation, generated entries, or annotations. / 定义宏 `NOCC`，供后续条件编译、生成条目或注解使用。
- **L20**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L21**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L22**: Defines macro `TLI_DEFINE_VECFUNC` for later conditional compilation, generated entries, or annotations. / 定义宏 `TLI_DEFINE_VECFUNC`，供后续条件编译、生成条目或注解使用。
- **L23**: Closes the current preprocessor conditional or header guard. / 结束当前的预处理条件块或头文件保护。
- **L24**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L25**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L26**: Comment documents the nearby API, invariant, or algorithmic intent: `Accelerate framework's Vector Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Accelerate framework's Vector Functions`。
- **L27**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L28**: Comment documents the nearby API, invariant, or algorithmic intent: `Floating-Point Arithmetic and Auxiliary Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Floating-Point Arithmetic and Auxiliary Functions`。

### Lines 29-56

```cpp
TLI_DEFINE_VECFUNC("ceilf", "vceilf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("fabsf", "vfabsf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.fabs.f32", "vfabsf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("floorf", "vfloorf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sqrtf", "vsqrtf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sqrt.f32", "vsqrtf", FIXED(4), "_ZGV_LLVM_N4v")

// Exponential and Logarithmic Functions
TLI_DEFINE_VECFUNC("expf", "vexpf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "vexpf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("expm1f", "vexpm1f", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("logf", "vlogf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "vlogf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log1pf", "vlog1pf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log10f", "vlog10f", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "vlog10f", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("logbf", "vlogbf", FIXED(4), "_ZGV_LLVM_N4v")

// Trigonometric Functions
TLI_DEFINE_VECFUNC("sinf", "vsinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "vsinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cosf", "vcosf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "vcosf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanf", "vtanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "vtanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("asinf", "vasinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "vasinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("acosf", "vacosf", FIXED(4), "_ZGV_LLVM_N4v")
```

- **L29**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L30**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L31**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L32**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L33**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L34**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L35**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L36**: Comment documents the nearby API, invariant, or algorithmic intent: `Exponential and Logarithmic Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Exponential and Logarithmic Functions`。
- **L37**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L38**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L39**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L40**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L41**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L42**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L43**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L44**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L45**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L46**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L47**: Comment documents the nearby API, invariant, or algorithmic intent: `Trigonometric Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Trigonometric Functions`。
- **L48**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L49**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L50**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L51**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L52**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L53**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L54**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L55**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L56**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 57-84

```cpp
TLI_DEFINE_VECFUNC("llvm.acos.f32", "vacosf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atanf", "vatanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "vatanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atan2f", "vatan2f", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "vatan2f", FIXED(4), "_ZGV_LLVM_N4vv")

// Hyperbolic Functions
TLI_DEFINE_VECFUNC("sinhf", "vsinhf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "vsinhf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("coshf", "vcoshf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "vcoshf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanhf", "vtanhf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "vtanhf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("asinhf", "vasinhf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("acoshf", "vacoshf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atanhf", "vatanhf", FIXED(4), "_ZGV_LLVM_N4v")

#elif defined(TLI_DEFINE_DARWIN_LIBSYSTEM_M_VECFUNCS)
// Darwin libsystem_m vector functions.

// Exponential and Logarithmic Functions
TLI_DEFINE_VECFUNC("exp", "_simd_exp_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "_simd_exp_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("expf", "_simd_exp_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "_simd_exp_f4", FIXED(4), "_ZGV_LLVM_N4v")

// Trigonometric Functions
TLI_DEFINE_VECFUNC("acos", "_simd_acos_d2", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L57**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L58**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L59**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L60**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L61**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L62**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L63**: Comment documents the nearby API, invariant, or algorithmic intent: `Hyperbolic Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Hyperbolic Functions`。
- **L64**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L65**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L66**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L67**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L68**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L69**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L70**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L71**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L72**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L73**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L74**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L75**: Comment documents the nearby API, invariant, or algorithmic intent: `Darwin libsystem_m vector functions.`. / 这行注释说明了附近 API、不变量或算法意图：`Darwin libsystem_m vector functions.`。
- **L76**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L77**: Comment documents the nearby API, invariant, or algorithmic intent: `Exponential and Logarithmic Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Exponential and Logarithmic Functions`。
- **L78**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L79**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L80**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L81**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L82**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L83**: Comment documents the nearby API, invariant, or algorithmic intent: `Trigonometric Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Trigonometric Functions`。
- **L84**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 85-112

```cpp
TLI_DEFINE_VECFUNC("llvm.acos.f64", "_simd_acos_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("acosf", "_simd_acos_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.acos.f32", "_simd_acos_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("asin", "_simd_asin_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.asin.f64", "_simd_asin_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("asinf", "_simd_asin_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "_simd_asin_f4", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("atan", "_simd_atan_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.atan.f64", "_simd_atan_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("atanf", "_simd_atan_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "_simd_atan_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atan2", "_simd_atan2_d2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.atan2.f64", "_simd_atan2_d2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("atan2f", "_simd_atan2_f4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "_simd_atan2_f4", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("cos", "_simd_cos_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "_simd_cos_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cosf", "_simd_cos_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "_simd_cos_f4", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("sin", "_simd_sin_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "_simd_sin_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("sinf", "_simd_sin_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "_simd_sin_f4", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("tan", "_simd_tan_d2", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L85**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L86**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L87**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L88**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L89**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L90**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L91**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L92**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L93**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L94**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L95**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L96**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L97**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L98**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L99**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L100**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L101**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L102**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L103**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L104**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L105**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L106**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L107**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L108**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L109**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L110**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L111**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L112**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 113-140

```cpp
TLI_DEFINE_VECFUNC("llvm.tan.f64", "_simd_tan_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("tanf", "_simd_tan_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "_simd_tan_f4", FIXED(4), "_ZGV_LLVM_N4v")

// Floating-Point Arithmetic and Auxiliary Functions
TLI_DEFINE_VECFUNC("cbrt", "_simd_cbrt_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cbrtf", "_simd_cbrt_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("erf", "_simd_erf_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("erff", "_simd_erf_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("pow", "_simd_pow_d2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "_simd_pow_d2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("powf", "_simd_pow_f4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "_simd_pow_f4", FIXED(4), "_ZGV_LLVM_N4vv")

// Hyperbolic Functions
TLI_DEFINE_VECFUNC("sinh", "_simd_sinh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.sinh.f64", "_simd_sinh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("sinhf", "_simd_sinh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "_simd_sinh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cosh", "_simd_cosh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.cosh.f64", "_simd_cosh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("coshf", "_simd_cosh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "_simd_cosh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanh", "_simd_tanh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.tanh.f64", "_simd_tanh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("tanhf", "_simd_tanh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "_simd_tanh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("asinh", "_simd_asinh_d2", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L113**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L114**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L115**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L116**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L117**: Comment documents the nearby API, invariant, or algorithmic intent: `Floating-Point Arithmetic and Auxiliary Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Floating-Point Arithmetic and Auxiliary Functions`。
- **L118**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L119**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L120**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L121**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L122**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L123**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L124**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L125**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L126**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L127**: Comment documents the nearby API, invariant, or algorithmic intent: `Hyperbolic Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Hyperbolic Functions`。
- **L128**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L129**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L130**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L131**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L132**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L133**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L134**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L135**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L136**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L137**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L138**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L139**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L140**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 141-168

```cpp
TLI_DEFINE_VECFUNC("asinhf", "_simd_asinh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("acosh", "_simd_acosh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("acoshf", "_simd_acosh_f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atanh", "_simd_atanh_d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("atanhf", "_simd_atanh_f4", FIXED(4), "_ZGV_LLVM_N4v")

#elif defined(TLI_DEFINE_LIBMVEC_X86_VECFUNCS)
// GLIBC Vector math Functions

TLI_DEFINE_VECFUNC("sin", "_ZGVbN2v_sin", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("sin", "_ZGVdN4v_sin", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("sinf", "_ZGVbN4v_sinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sinf", "_ZGVdN8v_sinf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.sin.f64", "_ZGVbN2v_sin", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "_ZGVdN4v_sin", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.sin.f32", "_ZGVbN4v_sinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "_ZGVdN8v_sinf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("cos", "_ZGVbN2v_cos", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cos", "_ZGVdN4v_cos", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("cosf", "_ZGVbN4v_cosf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cosf", "_ZGVdN8v_cosf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.cos.f64", "_ZGVbN2v_cos", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L141**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L142**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L143**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L144**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L145**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L146**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L147**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L148**: Comment documents the nearby API, invariant, or algorithmic intent: `GLIBC Vector math Functions`. / 这行注释说明了附近 API、不变量或算法意图：`GLIBC Vector math Functions`。
- **L149**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L150**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L151**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L152**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L153**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L154**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L155**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L156**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L157**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L158**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L159**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L160**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L161**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L162**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L163**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L164**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L165**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L166**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L167**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L168**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 169-196

```cpp
TLI_DEFINE_VECFUNC("llvm.cos.f64", "_ZGVdN4v_cos", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.cos.f32", "_ZGVbN4v_cosf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "_ZGVdN8v_cosf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("tan", "_ZGVbN2v_tan", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("tan", "_ZGVdN4v_tan", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("tanf", "_ZGVbN4v_tanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanf", "_ZGVdN8v_tanf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.tan.f64", "_ZGVbN2v_tan", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.tan.f64", "_ZGVdN4v_tan", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.tan.f32", "_ZGVbN4v_tanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "_ZGVdN8v_tanf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("pow", "_ZGVbN2vv_pow", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("pow", "_ZGVdN4vv_pow", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("powf", "_ZGVbN4vv_powf", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("powf", "_ZGVdN8vv_powf", FIXED(8), "_ZGV_LLVM_N8vv")

TLI_DEFINE_VECFUNC("__pow_finite", "_ZGVbN2vv___pow_finite", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("__pow_finite", "_ZGVdN4vv___pow_finite", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("__powf_finite", "_ZGVbN4vv___powf_finite", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("__powf_finite", "_ZGVdN8vv___powf_finite", FIXED(8), "_ZGV_LLVM_N8vv")
```

- **L169**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L170**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L171**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L172**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L173**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L174**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L175**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L176**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L177**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L178**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L179**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L180**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L181**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L182**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L183**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L184**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L185**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L186**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L187**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L188**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L189**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L190**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L191**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L192**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L193**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L194**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L195**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L196**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 197-224

```cpp

TLI_DEFINE_VECFUNC("llvm.pow.f64", "_ZGVbN2vv_pow", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "_ZGVdN4vv_pow", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("llvm.pow.f32", "_ZGVbN4vv_powf", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "_ZGVdN8vv_powf", FIXED(8), "_ZGV_LLVM_N8vv")

TLI_DEFINE_VECFUNC("exp", "_ZGVbN2v_exp", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("exp", "_ZGVdN4v_exp", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("expf", "_ZGVbN4v_expf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("expf", "_ZGVdN8v_expf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__exp_finite", "_ZGVbN2v___exp_finite", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__exp_finite", "_ZGVdN4v___exp_finite", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("__expf_finite", "_ZGVbN4v___expf_finite", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__expf_finite", "_ZGVdN8v___expf_finite", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.exp.f64", "_ZGVbN2v_exp", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "_ZGVdN4v_exp", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.exp.f32", "_ZGVbN4v_expf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "_ZGVdN8v_expf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("log", "_ZGVbN2v_log", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log", "_ZGVdN4v_log", FIXED(4), "_ZGV_LLVM_N4v")

```

- **L197**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L198**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L199**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L200**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L201**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L202**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L203**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L204**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L205**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L206**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L207**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L208**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L209**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L210**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L211**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L212**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L213**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L214**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L215**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L216**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L217**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L218**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L219**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L220**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L221**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L222**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L223**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L224**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 225-252

```cpp
TLI_DEFINE_VECFUNC("logf", "_ZGVbN4v_logf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("logf", "_ZGVdN8v_logf", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__log_finite", "_ZGVbN2v___log_finite", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__log_finite", "_ZGVdN4v___log_finite", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("__logf_finite", "_ZGVbN4v___logf_finite", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__logf_finite", "_ZGVdN8v___logf_finite", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.log.f64", "_ZGVbN2v_log", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "_ZGVdN4v_log", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.log.f32", "_ZGVbN4v_logf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "_ZGVdN8v_logf", FIXED(8), "_ZGV_LLVM_N8v")

#elif defined(TLI_DEFINE_LIBMVEC_AARCH64_VECFUNCS)

TLI_DEFINE_VECFUNC("acos", "_ZGVnN2v_acos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acosf", "_ZGVnN2v_acosf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acosf", "_ZGVnN4v_acosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acos", "_ZGVsMxv_acos", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("acosf", "_ZGVsMxv_acosf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.acos.f64", "_ZGVnN2v_acos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.acos.f32", "_ZGVnN2v_acosf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.acos.f32", "_ZGVnN4v_acosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.acos.f64", "_ZGVsMxv_acos", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.acos.f32", "_ZGVsMxv_acosf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)
```

- **L225**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L226**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L227**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L228**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L229**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L230**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L231**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L232**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L233**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L234**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L235**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L236**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L237**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L238**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L239**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L240**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L241**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L242**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L243**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L244**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L245**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L246**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L247**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L248**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L249**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L250**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L251**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L252**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 253-280

```cpp

TLI_DEFINE_VECFUNC("acosh", "_ZGVnN2v_acosh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acoshf", "_ZGVnN2v_acoshf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acoshf", "_ZGVnN4v_acoshf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acosh", "_ZGVsMxv_acosh",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("acoshf", "_ZGVsMxv_acoshf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("asin", "_ZGVnN2v_asin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinf", "_ZGVnN2v_asinf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinf", "_ZGVnN4v_asinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asin", "_ZGVsMxv_asin", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("asinf", "_ZGVsMxv_asinf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.asin.f64", "_ZGVnN2v_asin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.asin.f32", "_ZGVnN2v_asinf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.asin.f32", "_ZGVnN4v_asinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.asin.f64", "_ZGVsMxv_asin", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.asin.f32", "_ZGVsMxv_asinf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("asinh", "_ZGVnN2v_asinh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinhf", "_ZGVnN2v_asinhf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinhf", "_ZGVnN4v_asinhf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinh", "_ZGVsMxv_asinh",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("asinhf", "_ZGVsMxv_asinhf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("atan", "_ZGVnN2v_atan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanf", "_ZGVnN2v_atanf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanf", "_ZGVnN4v_atanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
```

- **L253**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L254**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L255**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L256**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L257**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L258**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L259**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L260**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L261**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L262**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L263**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L264**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L265**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L266**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L267**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L268**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L269**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L270**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L271**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L272**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L273**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L274**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L275**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L276**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L277**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L278**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L279**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L280**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 281-308

```cpp
TLI_DEFINE_VECFUNC("atan", "_ZGVsMxv_atan", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("atanf", "_ZGVsMxv_atanf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.atan.f64", "_ZGVnN2v_atan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan.f32", "_ZGVnN2v_atanf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan.f32", "_ZGVnN4v_atanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan.f64", "_ZGVsMxv_atan", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.atan.f32", "_ZGVsMxv_atanf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("atan2", "_ZGVnN2vv_atan2", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atan2f", "_ZGVnN2vv_atan2f", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atan2f", "_ZGVnN4vv_atan2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atan2", "_ZGVsMxvv_atan2", SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("atan2f", "_ZGVsMxvv_atan2f", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("llvm.atan2.f64", "_ZGVnN2vv_atan2", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "_ZGVnN2vv_atan2f", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "_ZGVnN4vv_atan2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan2.f64", "_ZGVsMxvv_atan2", SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "_ZGVsMxvv_atan2f", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("atanh", "_ZGVnN2v_atanh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanhf", "_ZGVnN2v_atanhf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanhf", "_ZGVnN4v_atanhf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanh", "_ZGVsMxv_atanh",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("atanhf", "_ZGVsMxv_atanhf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("cbrt", "_ZGVnN2v_cbrt", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
```

- **L281**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L282**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L283**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L284**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L285**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L286**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L287**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L288**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L289**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L290**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L291**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L292**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L293**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L294**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L295**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L296**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L297**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L298**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L299**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L300**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L301**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L302**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L303**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L304**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L305**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L306**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L307**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L308**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 309-336

```cpp
TLI_DEFINE_VECFUNC("cbrtf", "_ZGVnN2v_cbrtf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cbrtf", "_ZGVnN4v_cbrtf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cbrt", "_ZGVsMxv_cbrt",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("cbrtf", "_ZGVsMxv_cbrtf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("cos", "_ZGVnN2v_cos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cosf", "_ZGVnN2v_cosf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cosf", "_ZGVnN4v_cosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cos", "_ZGVsMxv_cos",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("cosf", "_ZGVsMxv_cosf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.cos.f64", "_ZGVnN2v_cos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cos.f32", "_ZGVnN2v_cosf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cos.f32", "_ZGVnN4v_cosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cos.f64", "_ZGVsMxv_cos", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.cos.f32", "_ZGVsMxv_cosf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("cosh", "_ZGVnN2v_cosh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("coshf", "_ZGVnN2v_coshf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("coshf", "_ZGVnN4v_coshf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cosh", "_ZGVsMxv_cosh",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("coshf", "_ZGVsMxv_coshf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.cosh.f64", "_ZGVnN2v_cosh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "_ZGVnN2v_coshf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "_ZGVnN4v_coshf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cosh.f64", "_ZGVsMxv_cosh", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "_ZGVsMxv_coshf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)
```

- **L309**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L310**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L311**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L312**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L313**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L314**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L315**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L316**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L317**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L318**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L319**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L320**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L321**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L322**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L323**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L324**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L325**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L326**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L327**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L328**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L329**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L330**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L331**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L332**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L333**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L334**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L335**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L336**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 337-364

```cpp

TLI_DEFINE_VECFUNC("erf", "_ZGVnN2v_erf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erff", "_ZGVnN2v_erff", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erff", "_ZGVnN4v_erff", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erf", "_ZGVsMxv_erf",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("erff", "_ZGVsMxv_erff", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("erfc", "_ZGVnN2v_erfc", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erfcf", "_ZGVnN2v_erfcf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erfcf", "_ZGVnN4v_erfcf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erfc", "_ZGVsMxv_erfc",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("erfcf", "_ZGVsMxv_erfcf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("exp", "_ZGVnN2v_exp", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expf", "_ZGVnN2v_expf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expf", "_ZGVnN4v_expf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp", "_ZGVsMxv_exp",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("expf", "_ZGVsMxv_expf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.exp.f64", "_ZGVnN2v_exp", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp.f32", "_ZGVnN2v_expf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp.f32", "_ZGVnN4v_expf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp.f64", "_ZGVsMxv_exp", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.exp.f32", "_ZGVsMxv_expf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("exp10", "_ZGVnN2v_exp10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp10f", "_ZGVnN2v_exp10f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp10f", "_ZGVnN4v_exp10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
```

- **L337**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L338**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L339**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L340**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L341**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L342**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L343**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L344**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L345**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L346**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L347**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L348**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L349**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L350**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L351**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L352**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L353**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L354**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L355**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L356**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L357**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L358**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L359**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L360**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L361**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L362**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L363**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L364**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 365-392

```cpp
TLI_DEFINE_VECFUNC("exp10", "_ZGVsMxv_exp10",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("exp10f", "_ZGVsMxv_exp10f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.exp10.f64", "_ZGVnN2v_exp10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "_ZGVnN2v_exp10f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "_ZGVnN4v_exp10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp10.f64", "_ZGVsMxv_exp10", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "_ZGVsMxv_exp10f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("exp2", "_ZGVnN2v_exp2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp2f", "_ZGVnN2v_exp2f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp2f", "_ZGVnN4v_exp2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp2", "_ZGVsMxv_exp2",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("exp2f", "_ZGVsMxv_exp2f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.exp2.f64", "_ZGVnN2v_exp2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "_ZGVnN2v_exp2f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "_ZGVnN4v_exp2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "_ZGVsMxv_exp2", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "_ZGVsMxv_exp2f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("expm1", "_ZGVnN2v_expm1", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expm1f", "_ZGVnN2v_expm1f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expm1f", "_ZGVnN4v_expm1f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expm1", "_ZGVsMxv_expm1",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("expm1f", "_ZGVsMxv_expm1f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("hypot", "_ZGVnN2vv_hypot", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
```

- **L365**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L366**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L367**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L368**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L369**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L370**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L371**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L372**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L373**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L374**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L375**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L376**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L377**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L378**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L379**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L380**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L381**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L382**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L383**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L384**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L385**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L386**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L387**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L388**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L389**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L390**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L391**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L392**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 393-420

```cpp
TLI_DEFINE_VECFUNC("hypotf", "_ZGVnN2vv_hypotf", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("hypotf", "_ZGVnN4vv_hypotf", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("hypot", "_ZGVsMxvv_hypot", SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("hypotf", "_ZGVsMxvv_hypotf", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("log", "_ZGVnN2v_log", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("logf", "_ZGVnN2v_logf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("logf", "_ZGVnN4v_logf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log", "_ZGVsMxv_log",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("logf", "_ZGVsMxv_logf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.log.f64", "_ZGVnN2v_log", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log.f32", "_ZGVnN2v_logf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log.f32", "_ZGVnN4v_logf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log.f64", "_ZGVsMxv_log", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.log.f32", "_ZGVsMxv_logf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("log10", "_ZGVnN2v_log10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log10f", "_ZGVnN2v_log10f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log10f", "_ZGVnN4v_log10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log10", "_ZGVsMxv_log10",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("log10f", "_ZGVsMxv_log10f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.log10.f64", "_ZGVnN2v_log10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log10.f32", "_ZGVnN2v_log10f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log10.f32", "_ZGVnN4v_log10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log10.f64", "_ZGVsMxv_log10", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.log10.f32", "_ZGVsMxv_log10f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)
```

- **L393**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L394**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L395**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L396**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L397**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L398**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L399**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L400**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L401**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L402**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L403**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L404**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L405**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L406**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L407**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L408**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L409**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L410**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L411**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L412**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L413**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L414**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L415**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L416**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L417**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L418**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L419**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L420**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 421-448

```cpp

TLI_DEFINE_VECFUNC("log1p", "_ZGVnN2v_log1p", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log1pf", "_ZGVnN2v_log1pf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log1pf", "_ZGVnN4v_log1pf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log1p", "_ZGVsMxv_log1p",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("log1pf", "_ZGVsMxv_log1pf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("log2", "_ZGVnN2v_log2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log2f", "_ZGVnN2v_log2f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log2f", "_ZGVnN4v_log2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log2", "_ZGVsMxv_log2",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("log2f", "_ZGVsMxv_log2f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.log2.f64", "_ZGVnN2v_log2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log2.f32", "_ZGVnN2v_log2f", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log2.f32", "_ZGVnN4v_log2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log2.f64", "_ZGVsMxv_log2", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.log2.f32", "_ZGVsMxv_log2f", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("pow", "_ZGVnN2vv_pow", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("powf", "_ZGVnN2vv_powf", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("powf", "_ZGVnN4vv_powf", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("pow", "_ZGVsMxvv_pow", SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("powf", "_ZGVsMxvv_powf", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("llvm.pow.f64", "_ZGVnN2vv_pow", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.pow.f32", "_ZGVnN2vv_powf", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.pow.f32", "_ZGVnN4vv_powf", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
```

- **L421**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L422**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L423**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L424**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L425**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L426**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L427**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L428**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L429**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L430**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L431**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L432**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L433**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L434**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L435**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L436**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L437**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L438**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L439**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L440**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L441**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L442**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L443**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L444**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L445**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L446**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L447**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L448**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 449-476

```cpp
TLI_DEFINE_VECFUNC("llvm.pow.f64", "_ZGVsMxvv_pow", SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("llvm.pow.f32", "_ZGVsMxvv_powf", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("sin", "_ZGVnN2v_sin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinf", "_ZGVnN2v_sinf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinf", "_ZGVnN4v_sinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sin", "_ZGVsMxv_sin", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("sinf", "_ZGVsMxv_sinf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.sin.f64", "_ZGVnN2v_sin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sin.f32", "_ZGVnN2v_sinf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sin.f32", "_ZGVnN4v_sinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sin.f64", "_ZGVsMxv_sin", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.sin.f32", "_ZGVsMxv_sinf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("sinh", "_ZGVnN2v_sinh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinhf", "_ZGVnN2v_sinhf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinhf", "_ZGVnN4v_sinhf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinh", "_ZGVsMxv_sinh",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("sinhf", "_ZGVsMxv_sinhf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.sinh.f64", "_ZGVnN2v_sinh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "_ZGVnN2v_sinhf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "_ZGVnN4v_sinhf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sinh.f64", "_ZGVsMxv_sinh", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "_ZGVsMxv_sinhf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("tan", "_ZGVnN2v_tan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
```

- **L449**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L450**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L451**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L452**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L453**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L454**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L455**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L456**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L457**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L458**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L459**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L460**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L461**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L462**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L463**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L464**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L465**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L466**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L467**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L468**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L469**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L470**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L471**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L472**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L473**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L474**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L475**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L476**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 477-504

```cpp
TLI_DEFINE_VECFUNC("tanf", "_ZGVnN2v_tanf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tanf", "_ZGVnN4v_tanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tan", "_ZGVsMxv_tan",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("tanf", "_ZGVsMxv_tanf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.tan.f64", "_ZGVnN2v_tan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tan.f32", "_ZGVnN2v_tanf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tan.f32", "_ZGVnN4v_tanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tan.f64", "_ZGVsMxv_tan", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.tan.f32", "_ZGVsMxv_tanf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("tanh", "_ZGVnN2v_tanh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tanhf", "_ZGVnN2v_tanhf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tanhf", "_ZGVnN4v_tanhf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tanh", "_ZGVsMxv_tanh",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("tanhf", "_ZGVsMxv_tanhf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.tanh.f64", "_ZGVnN2v_tanh", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "_ZGVnN2v_tanhf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "_ZGVnN4v_tanhf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tanh.f64", "_ZGVsMxv_tanh", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "_ZGVsMxv_tanhf", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

#elif defined(TLI_DEFINE_MASSV_VECFUNCS)
// IBM MASS library's vector Functions

// Floating-Point Arithmetic and Auxiliary Functions
TLI_DEFINE_VECFUNC("cbrt", "__cbrtd2", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L477**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L478**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L479**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L480**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L481**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L482**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L483**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L484**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L485**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L486**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L487**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L488**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L489**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L490**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L491**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L492**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L493**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L494**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L495**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L496**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L497**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L498**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L499**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L500**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L501**: Comment documents the nearby API, invariant, or algorithmic intent: `IBM MASS library's vector Functions`. / 这行注释说明了附近 API、不变量或算法意图：`IBM MASS library's vector Functions`。
- **L502**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L503**: Comment documents the nearby API, invariant, or algorithmic intent: `Floating-Point Arithmetic and Auxiliary Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Floating-Point Arithmetic and Auxiliary Functions`。
- **L504**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 505-532

```cpp
TLI_DEFINE_VECFUNC("cbrtf", "__cbrtf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("pow", "__powd2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "__powd2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("powf", "__powf4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "__powf4", FIXED(4), "_ZGV_LLVM_N4vv")

// Exponential and Logarithmic Functions
TLI_DEFINE_VECFUNC("exp", "__expd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "__expd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("expf", "__expf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "__expf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp2", "__exp2d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "__exp2d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("exp2f", "__exp2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "__exp2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("expm1", "__expm1d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("expm1f", "__expm1f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log", "__logd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "__logd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("logf", "__logf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "__logf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log1p", "__log1pd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log1pf", "__log1pf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log10", "__log10d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log10.f64", "__log10d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log10f", "__log10f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "__log10f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log2", "__log2d2", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L505**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L506**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L507**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L508**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L509**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L510**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L511**: Comment documents the nearby API, invariant, or algorithmic intent: `Exponential and Logarithmic Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Exponential and Logarithmic Functions`。
- **L512**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L513**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L514**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L515**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L516**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L517**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L518**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L519**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L520**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L521**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L522**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L523**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L524**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L525**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L526**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L527**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L528**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L529**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L530**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L531**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L532**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 533-560

```cpp
TLI_DEFINE_VECFUNC("llvm.log2.f64", "__log2d2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log2f", "__log2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "__log2f4", FIXED(4), "_ZGV_LLVM_N4v")

// Trigonometric Functions
TLI_DEFINE_VECFUNC("sin", "__sind2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "__sind2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("sinf", "__sinf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "__sinf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cos", "__cosd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "__cosd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cosf", "__cosf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "__cosf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tan", "__tand2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("tanf", "__tanf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("asin", "__asind2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("asinf", "__asinf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("acos", "__acosd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("acosf", "__acosf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atan", "__atand2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("atanf", "__atanf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atan2", "__atan2d2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.atan2.f64", "__atan2d2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("atan2f", "__atan2f4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "__atan2f4", FIXED(4), "_ZGV_LLVM_N4vv")

// Hyperbolic Functions
TLI_DEFINE_VECFUNC("sinh", "__sinhd2", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L533**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L534**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L535**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L536**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L537**: Comment documents the nearby API, invariant, or algorithmic intent: `Trigonometric Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Trigonometric Functions`。
- **L538**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L539**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L540**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L541**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L542**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L543**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L544**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L545**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L546**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L547**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L548**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L549**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L550**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L551**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L552**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L553**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L554**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L555**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L556**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L557**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L558**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L559**: Comment documents the nearby API, invariant, or algorithmic intent: `Hyperbolic Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Hyperbolic Functions`。
- **L560**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 561-588

```cpp
TLI_DEFINE_VECFUNC("sinhf", "__sinhf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cosh", "__coshd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("coshf", "__coshf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanh", "__tanhd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("tanhf", "__tanhf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("asinh", "__asinhd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("asinhf", "__asinhf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("acosh", "__acoshd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("acoshf", "__acoshf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atanh", "__atanhd2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("atanhf", "__atanhf4", FIXED(4), "_ZGV_LLVM_N4v")


#elif defined(TLI_DEFINE_SVML_VECFUNCS)
// Intel SVM library's Vector Functions

TLI_DEFINE_VECFUNC("sin", "__svml_sin2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("sin", "__svml_sin4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sin", "__svml_sin8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("sinf", "__svml_sinf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sinf", "__svml_sinf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("sinf", "__svml_sinf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.sin.f64", "__svml_sin2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "__svml_sin4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "__svml_sin8", FIXED(8), "_ZGV_LLVM_N8v")

```

- **L561**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L562**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L563**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L564**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L565**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L566**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L567**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L568**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L569**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L570**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L571**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L572**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L573**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L574**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L575**: Comment documents the nearby API, invariant, or algorithmic intent: `Intel SVM library's Vector Functions`. / 这行注释说明了附近 API、不变量或算法意图：`Intel SVM library's Vector Functions`。
- **L576**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L577**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L578**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L579**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L580**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L581**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L582**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L583**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L584**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L585**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L586**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L587**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L588**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 589-616

```cpp
TLI_DEFINE_VECFUNC("llvm.sin.f32", "__svml_sinf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "__svml_sinf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "__svml_sinf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("cos", "__svml_cos2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cos", "__svml_cos4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cos", "__svml_cos8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("cosf", "__svml_cosf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cosf", "__svml_cosf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("cosf", "__svml_cosf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.cos.f64", "__svml_cos2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "__svml_cos4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "__svml_cos8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.cos.f32", "__svml_cosf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "__svml_cosf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "__svml_cosf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("tan", "__svml_tan2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("tan", "__svml_tan4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tan", "__svml_tan8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("tanf", "__svml_tanf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanf", "__svml_tanf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("tanf", "__svml_tanf16", FIXED(16), "_ZGV_LLVM_N16v")

```

- **L589**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L590**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L591**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L592**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L593**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L594**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L595**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L596**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L597**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L598**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L599**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L600**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L601**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L602**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L603**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L604**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L605**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L606**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L607**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L608**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L609**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L610**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L611**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L612**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L613**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L614**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L615**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L616**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 617-644

```cpp
TLI_DEFINE_VECFUNC("llvm.tan.f64", "__svml_tan2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.tan.f64", "__svml_tan4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tan.f64", "__svml_tan8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.tan.f32", "__svml_tanf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "__svml_tanf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "__svml_tanf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("pow", "__svml_pow2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("pow", "__svml_pow4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("pow", "__svml_pow8", FIXED(8), "_ZGV_LLVM_N8vv")

TLI_DEFINE_VECFUNC("powf", "__svml_powf4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("powf", "__svml_powf8", FIXED(8), "_ZGV_LLVM_N8vv")
TLI_DEFINE_VECFUNC("powf", "__svml_powf16", FIXED(16), "_ZGV_LLVM_N16vv")

TLI_DEFINE_VECFUNC("__pow_finite", "__svml_pow2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("__pow_finite", "__svml_pow4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("__pow_finite", "__svml_pow8", FIXED(8), "_ZGV_LLVM_N8vv")

TLI_DEFINE_VECFUNC("__powf_finite", "__svml_powf4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("__powf_finite", "__svml_powf8", FIXED(8), "_ZGV_LLVM_N8vv")
TLI_DEFINE_VECFUNC("__powf_finite", "__svml_powf16", FIXED(16), "_ZGV_LLVM_N16vv")

TLI_DEFINE_VECFUNC("llvm.pow.f64", "__svml_pow2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "__svml_pow4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "__svml_pow8", FIXED(8), "_ZGV_LLVM_N8vv")

```

- **L617**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L618**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L619**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L620**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L621**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L622**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L623**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L624**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L625**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L626**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L627**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L628**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L629**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L630**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L631**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L632**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L633**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L634**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L635**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L636**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L637**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L638**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L639**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L640**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L641**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L642**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L643**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L644**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 645-672

```cpp
TLI_DEFINE_VECFUNC("llvm.pow.f32", "__svml_powf4", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "__svml_powf8", FIXED(8), "_ZGV_LLVM_N8vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "__svml_powf16", FIXED(16), "_ZGV_LLVM_N16vv")

TLI_DEFINE_VECFUNC("exp", "__svml_exp2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("exp", "__svml_exp4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp", "__svml_exp8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("expf", "__svml_expf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("expf", "__svml_expf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("expf", "__svml_expf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("__exp_finite", "__svml_exp2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__exp_finite", "__svml_exp4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__exp_finite", "__svml_exp8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__expf_finite", "__svml_expf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__expf_finite", "__svml_expf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__expf_finite", "__svml_expf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.exp.f64", "__svml_exp2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "__svml_exp4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "__svml_exp8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.exp.f32", "__svml_expf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "__svml_expf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "__svml_expf16", FIXED(16), "_ZGV_LLVM_N16v")

```

- **L645**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L646**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L647**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L648**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L649**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L650**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L651**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L652**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L653**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L654**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L655**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L656**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L657**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L658**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L659**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L660**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L661**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L662**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L663**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L664**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L665**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L666**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L667**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L668**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L669**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L670**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L671**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L672**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 673-700

```cpp
TLI_DEFINE_VECFUNC("log", "__svml_log2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log", "__svml_log4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log", "__svml_log8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("logf", "__svml_logf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("logf", "__svml_logf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("logf", "__svml_logf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("__log_finite", "__svml_log2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__log_finite", "__svml_log4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log_finite", "__svml_log8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__logf_finite", "__svml_logf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__logf_finite", "__svml_logf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__logf_finite", "__svml_logf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.log.f64", "__svml_log2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "__svml_log4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "__svml_log8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.log.f32", "__svml_logf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "__svml_logf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "__svml_logf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("log2", "__svml_log22", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log2", "__svml_log24", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log2", "__svml_log28", FIXED(8), "_ZGV_LLVM_N8v")

```

- **L673**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L674**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L675**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L676**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L677**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L678**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L679**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L680**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L681**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L682**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L683**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L684**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L685**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L686**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L687**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L688**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L689**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L690**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L691**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L692**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L693**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L694**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L695**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L696**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L697**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L698**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L699**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L700**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 701-728

```cpp
TLI_DEFINE_VECFUNC("log2f", "__svml_log2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log2f", "__svml_log2f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("log2f", "__svml_log2f16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("__log2_finite", "__svml_log22", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__log2_finite", "__svml_log24", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log2_finite", "__svml_log28", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__log2f_finite", "__svml_log2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log2f_finite", "__svml_log2f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__log2f_finite", "__svml_log2f16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.log2.f64", "__svml_log22", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "__svml_log24", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "__svml_log28", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.log2.f32", "__svml_log2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "__svml_log2f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "__svml_log2f16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("log10", "__svml_log102", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log10", "__svml_log104", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log10", "__svml_log108", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("log10f", "__svml_log10f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log10f", "__svml_log10f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("log10f", "__svml_log10f16", FIXED(16), "_ZGV_LLVM_N16v")

```

- **L701**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L702**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L703**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L704**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L705**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L706**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L707**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L708**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L709**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L710**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L711**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L712**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L713**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L714**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L715**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L716**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L717**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L718**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L719**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L720**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L721**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L722**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L723**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L724**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L725**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L726**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L727**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L728**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 729-756

```cpp
TLI_DEFINE_VECFUNC("__log10_finite", "__svml_log102", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__log10_finite", "__svml_log104", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log10_finite", "__svml_log108", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__log10f_finite", "__svml_log10f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log10f_finite", "__svml_log10f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__log10f_finite", "__svml_log10f16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.log10.f64", "__svml_log102", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log10.f64", "__svml_log104", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log10.f64", "__svml_log108", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.log10.f32", "__svml_log10f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "__svml_log10f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "__svml_log10f16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("sqrt", "__svml_sqrt2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("sqrt", "__svml_sqrt4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sqrt", "__svml_sqrt8", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("sqrtf", "__svml_sqrtf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sqrtf", "__svml_sqrtf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("sqrtf", "__svml_sqrtf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("__sqrt_finite", "__svml_sqrt2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__sqrt_finite", "__svml_sqrt4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__sqrt_finite", "__svml_sqrt8", FIXED(8), "_ZGV_LLVM_N8v")

```

- **L729**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L730**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L731**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L732**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L733**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L734**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L735**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L736**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L737**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L738**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L739**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L740**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L741**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L742**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L743**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L744**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L745**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L746**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L747**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L748**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L749**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L750**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L751**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L752**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L753**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L754**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L755**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L756**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 757-784

```cpp
TLI_DEFINE_VECFUNC("__sqrtf_finite", "__svml_sqrtf4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__sqrtf_finite", "__svml_sqrtf8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__sqrtf_finite", "__svml_sqrtf16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("exp2", "__svml_exp22", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("exp2", "__svml_exp24", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp2", "__svml_exp28", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("exp2f", "__svml_exp2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp2f", "__svml_exp2f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("exp2f", "__svml_exp2f16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.exp2.f64", "__svml_exp22", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "__svml_exp24", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "__svml_exp28", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.exp2.f32", "__svml_exp2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "__svml_exp2f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "__svml_exp2f16", FIXED(16), "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("__exp2_finite", "__svml_exp22", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__exp2_finite", "__svml_exp24", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__exp2_finite", "__svml_exp28", FIXED(8), "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__exp2f_finite", "__svml_exp2f4", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__exp2f_finite", "__svml_exp2f8", FIXED(8), "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__exp2f_finite", "__svml_exp2f16", FIXED(16), "_ZGV_LLVM_N16v")

```

- **L757**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L758**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L759**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L760**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L761**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L762**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L763**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L764**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L765**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L766**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L767**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L768**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L769**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L770**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L771**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L772**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L773**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L774**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L775**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L776**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L777**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L778**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L779**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L780**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L781**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L782**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L783**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L784**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 785-812

```cpp
#elif defined(TLI_DEFINE_SLEEFGNUABI_VF2_VECFUNCS)

TLI_DEFINE_VECFUNC("acos", "_ZGVnN2v_acos", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.acos.f64", "_ZGVnN2v_acos", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("acosh", "_ZGVnN2v_acosh", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("asin", "_ZGVnN2v_asin", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.asin.f64", "_ZGVnN2v_asin", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("asinh", "_ZGVnN2v_asinh", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("atan", "_ZGVnN2v_atan", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.atan.f64", "_ZGVnN2v_atan", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("atan2", "_ZGVnN2vv_atan2", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.atan2.f64", "_ZGVnN2vv_atan2", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("atanh", "_ZGVnN2v_atanh", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("cbrt", "_ZGVnN2v_cbrt", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("copysign", "_ZGVnN2vv_copysign", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("cos", "_ZGVnN2v_cos", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "_ZGVnN2v_cos", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("cosh", "_ZGVnN2v_cosh", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L785**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L786**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L787**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L788**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L789**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L790**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L791**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L792**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L793**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L794**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L795**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L796**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L797**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L798**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L799**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L800**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L801**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L802**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L803**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L804**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L805**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L806**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L807**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L808**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L809**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L810**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L811**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L812**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 813-840

```cpp
TLI_DEFINE_VECFUNC("llvm.cosh.f64", "_ZGVnN2v_cosh", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("cospi", "_ZGVnN2v_cospi", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("erf", "_ZGVnN2v_erf", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("erfc", "_ZGVnN2v_erfc", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("exp", "_ZGVnN2v_exp", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "_ZGVnN2v_exp", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("exp10", "_ZGVnN2v_exp10", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp10.f64", "_ZGVnN2v_exp10", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("exp2", "_ZGVnN2v_exp2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "_ZGVnN2v_exp2", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("expm1", "_ZGVnN2v_expm1", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("fdim", "_ZGVnN2vv_fdim", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("fma", "_ZGVnN2vvv_fma", FIXED(2), "_ZGV_LLVM_N2vvv")

TLI_DEFINE_VECFUNC("fmax", "_ZGVnN2vv_fmax", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("fmin", "_ZGVnN2vv_fmin", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("fmod", "_ZGVnN2vv_fmod", FIXED(2), "_ZGV_LLVM_N2vv")
```

- **L813**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L814**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L815**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L816**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L817**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L818**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L819**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L820**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L821**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L822**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L823**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L824**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L825**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L826**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L827**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L828**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L829**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L830**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L831**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L832**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L833**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L834**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L835**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L836**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L837**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L838**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L839**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L840**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 841-868

```cpp

TLI_DEFINE_VECFUNC("hypot", "_ZGVnN2vv_hypot", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("ilogb", "_ZGVnN2v_ilogb", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("ldexp", "_ZGVnN2vv_ldexp", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("lgamma", "_ZGVnN2v_lgamma", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("log", "_ZGVnN2v_log", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "_ZGVnN2v_log", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("log10", "_ZGVnN2v_log10", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log10.f64", "_ZGVnN2v_log10", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("log1p", "_ZGVnN2v_log1p", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("log2", "_ZGVnN2v_log2", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "_ZGVnN2v_log2", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("modf", "_ZGVnN2vl8_modf", FIXED(2), "_ZGV_LLVM_N2vl8")

TLI_DEFINE_VECFUNC("nextafter", "_ZGVnN2vv_nextafter", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("pow", "_ZGVnN2vv_pow", FIXED(2), "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "_ZGVnN2vv_pow", FIXED(2), "_ZGV_LLVM_N2vv")

TLI_DEFINE_VECFUNC("sin", "_ZGVnN2v_sin", FIXED(2), "_ZGV_LLVM_N2v")
```

- **L841**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L842**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L843**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L844**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L845**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L846**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L847**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L848**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L849**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L850**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L851**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L852**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L853**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L854**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L855**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L856**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L857**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L858**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L859**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L860**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L861**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L862**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L863**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L864**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L865**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L866**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L867**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L868**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 869-896

```cpp
TLI_DEFINE_VECFUNC("llvm.sin.f64", "_ZGVnN2v_sin", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("sincos", "_ZGVnN2vl8l8_sincos", FIXED(2), "_ZGV_LLVM_N2vl8l8")

TLI_DEFINE_VECFUNC("sincospi", "_ZGVnN2vl8l8_sincospi", FIXED(2), "_ZGV_LLVM_N2vl8l8")

TLI_DEFINE_VECFUNC("sinh", "_ZGVnN2v_sinh", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.sinh.f64", "_ZGVnN2v_sinh", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("sinpi", "_ZGVnN2v_sinpi", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("sqrt", "_ZGVnN2v_sqrt", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("tan", "_ZGVnN2v_tan", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.tan.f64", "_ZGVnN2v_tan", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("tanh", "_ZGVnN2v_tanh", FIXED(2), "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.tanh.f64", "_ZGVnN2v_tanh", FIXED(2), "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("tgamma", "_ZGVnN2v_tgamma", FIXED(2), "_ZGV_LLVM_N2v")

#elif defined(TLI_DEFINE_SLEEFGNUABI_VF4_VECFUNCS)

TLI_DEFINE_VECFUNC("acosf", "_ZGVnN4v_acosf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.acos.f32", "_ZGVnN4v_acosf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("acoshf", "_ZGVnN4v_acoshf", FIXED(4), "_ZGV_LLVM_N4v")

```

- **L869**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L870**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L871**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L872**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L873**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L874**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L875**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L876**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L877**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L878**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L879**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L880**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L881**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L882**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L883**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L884**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L885**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L886**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L887**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L888**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L889**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L890**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L891**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L892**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L893**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L894**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L895**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L896**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 897-924

```cpp
TLI_DEFINE_VECFUNC("asinf", "_ZGVnN4v_asinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "_ZGVnN4v_asinf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("asinhf", "_ZGVnN4v_asinhf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("atanf", "_ZGVnN4v_atanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "_ZGVnN4v_atanf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("atan2f", "_ZGVnN4vv_atan2f", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "_ZGVnN4vv_atan2f", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("atanhf", "_ZGVnN4v_atanhf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("cbrtf", "_ZGVnN4v_cbrtf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("copysignf", "_ZGVnN4vv_copysignf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("cosf", "_ZGVnN4v_cosf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "_ZGVnN4v_cosf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("coshf", "_ZGVnN4v_coshf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "_ZGVnN4v_coshf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("cospif", "_ZGVnN4v_cospif", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("erff", "_ZGVnN4v_erff", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("erfcf", "_ZGVnN4v_erfcf", FIXED(4), "_ZGV_LLVM_N4v")
```

- **L897**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L898**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L899**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L900**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L901**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L902**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L903**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L904**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L905**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L906**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L907**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L908**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L909**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L910**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L911**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L912**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L913**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L914**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L915**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L916**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L917**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L918**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L919**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L920**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L921**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L922**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L923**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L924**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 925-952

```cpp

TLI_DEFINE_VECFUNC("expf", "_ZGVnN4v_expf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "_ZGVnN4v_expf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("exp10f", "_ZGVnN4v_exp10f", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "_ZGVnN4v_exp10f", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("exp2f", "_ZGVnN4v_exp2f", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "_ZGVnN4v_exp2f", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("expm1f", "_ZGVnN4v_expm1f", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("fdimf", "_ZGVnN4vv_fdimf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("fmaf", "_ZGVnN4vvv_fmaf", FIXED(4), "_ZGV_LLVM_N4vvv")

TLI_DEFINE_VECFUNC("fmaxf", "_ZGVnN4vv_fmaxf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("fminf", "_ZGVnN4vv_fminf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("fmodf", "_ZGVnN4vv_fmodf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("hypotf", "_ZGVnN4vv_hypotf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("ilogbf", "_ZGVnN4v_ilogbf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("ldexpf", "_ZGVnN4vv_ldexpf", FIXED(4), "_ZGV_LLVM_N4vv")

```

- **L925**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L926**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L927**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L928**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L929**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L930**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L931**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L932**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L933**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L934**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L935**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L936**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L937**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L938**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L939**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L940**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L941**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L942**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L943**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L944**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L945**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L946**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L947**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L948**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L949**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L950**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L951**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L952**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 953-980

```cpp
TLI_DEFINE_VECFUNC("lgammaf", "_ZGVnN4v_lgammaf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("logf", "_ZGVnN4v_logf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "_ZGVnN4v_logf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("log10f", "_ZGVnN4v_log10f", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "_ZGVnN4v_log10f", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("log1pf", "_ZGVnN4v_log1pf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("log2f", "_ZGVnN4v_log2f", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "_ZGVnN4v_log2f", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("modff", "_ZGVnN4vl4_modff", FIXED(4), "_ZGV_LLVM_N4vl4")

TLI_DEFINE_VECFUNC("nextafterf", "_ZGVnN4vv_nextafterf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("powf", "_ZGVnN4vv_powf", FIXED(4), "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "_ZGVnN4vv_powf", FIXED(4), "_ZGV_LLVM_N4vv")

TLI_DEFINE_VECFUNC("sinf", "_ZGVnN4v_sinf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "_ZGVnN4v_sinf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("sincosf", "_ZGVnN4vl4l4_sincosf", FIXED(4), "_ZGV_LLVM_N4vl4l4")

TLI_DEFINE_VECFUNC("sincospif", "_ZGVnN4vl4l4_sincospif", FIXED(4), "_ZGV_LLVM_N4vl4l4")

TLI_DEFINE_VECFUNC("sinhf", "_ZGVnN4v_sinhf", FIXED(4), "_ZGV_LLVM_N4v")
```

- **L953**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L954**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L955**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L956**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L957**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L958**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L959**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L960**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L961**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L962**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L963**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L964**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L965**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L966**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L967**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L968**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L969**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L970**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L971**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L972**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L973**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L974**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L975**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L976**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L977**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L978**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L979**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L980**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 981-1008

```cpp
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "_ZGVnN4v_sinhf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("sinpif", "_ZGVnN4v_sinpif", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("sqrtf", "_ZGVnN4v_sqrtf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("tanf", "_ZGVnN4v_tanf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "_ZGVnN4v_tanf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("tanhf", "_ZGVnN4v_tanhf", FIXED(4), "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "_ZGVnN4v_tanhf", FIXED(4), "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("tgammaf", "_ZGVnN4v_tgammaf", FIXED(4), "_ZGV_LLVM_N4v")

#elif defined(TLI_DEFINE_SLEEFGNUABI_SCALABLE_VECFUNCS)

TLI_DEFINE_VECFUNC("acos", "_ZGVsMxv_acos",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("acosf", "_ZGVsMxv_acosf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.acos.f64", "_ZGVsMxv_acos", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.acos.f32", "_ZGVsMxv_acosf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("acosh", "_ZGVsMxv_acosh",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("acoshf", "_ZGVsMxv_acoshf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("asin", "_ZGVsMxv_asin",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("asinf", "_ZGVsMxv_asinf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.asin.f64", "_ZGVsMxv_asin", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "_ZGVsMxv_asinf", SCALABLE(4), MASKED, "_ZGVsMxv")
```

- **L981**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L982**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L983**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L984**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L985**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L986**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L987**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L988**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L989**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L990**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L991**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L992**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L993**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L994**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L995**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L996**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L997**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L998**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L999**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1000**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1001**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1002**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1003**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1004**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1005**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1006**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1007**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1008**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1009-1036

```cpp

TLI_DEFINE_VECFUNC("asinh", "_ZGVsMxv_asinh",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("asinhf", "_ZGVsMxv_asinhf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("atan", "_ZGVsMxv_atan",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("atanf", "_ZGVsMxv_atanf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.atan.f64", "_ZGVsMxv_atan", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "_ZGVsMxv_atanf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("atan2", "_ZGVsMxvv_atan2",  SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("atan2f", "_ZGVsMxvv_atan2f", SCALABLE(4), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("llvm.atan2.f64", "_ZGVsMxvv_atan2",  SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "_ZGVsMxvv_atan2f", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("atanh", "_ZGVsMxv_atanh",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("atanhf", "_ZGVsMxv_atanhf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("cbrt", "_ZGVsMxv_cbrt",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("cbrtf", "_ZGVsMxv_cbrtf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("copysign", "_ZGVsMxvv_copysign",  SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("copysignf", "_ZGVsMxvv_copysignf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("cos", "_ZGVsMxv_cos",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("cosf", "_ZGVsMxv_cosf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "_ZGVsMxv_cos", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "_ZGVsMxv_cosf", SCALABLE(4), MASKED, "_ZGVsMxv")

```

- **L1009**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1010**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1011**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1012**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1013**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1014**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1015**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1016**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1017**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1018**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1019**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1020**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1021**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1022**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1023**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1024**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1025**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1026**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1027**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1028**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1029**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1030**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1031**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1032**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1033**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1034**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1035**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1036**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1037-1064

```cpp
TLI_DEFINE_VECFUNC("cosh", "_ZGVsMxv_cosh",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("coshf", "_ZGVsMxv_coshf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.cosh.f64", "_ZGVsMxv_cosh", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "_ZGVsMxv_coshf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("cospi", "_ZGVsMxv_cospi",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("cospif", "_ZGVsMxv_cospif", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("erf", "_ZGVsMxv_erf",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("erff", "_ZGVsMxv_erff", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("erfc", "_ZGVsMxv_erfc",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("erfcf", "_ZGVsMxv_erfcf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("exp", "_ZGVsMxv_exp",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("expf", "_ZGVsMxv_expf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "_ZGVsMxv_exp", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "_ZGVsMxv_expf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("exp10", "_ZGVsMxv_exp10",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("exp10f", "_ZGVsMxv_exp10f", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.exp10.f64", "_ZGVsMxv_exp10",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "_ZGVsMxv_exp10f", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("exp2", "_ZGVsMxv_exp2",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("exp2f", "_ZGVsMxv_exp2f", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "_ZGVsMxv_exp2", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "_ZGVsMxv_exp2f", SCALABLE(4), MASKED, "_ZGVsMxv")
```

- **L1037**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1038**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1039**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1040**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1041**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1042**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1043**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1044**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1045**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1046**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1047**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1048**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1049**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1050**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1051**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1052**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1053**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1054**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1055**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1056**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1057**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1058**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1059**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1060**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1061**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1062**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1063**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1064**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1065-1092

```cpp

TLI_DEFINE_VECFUNC("expm1", "_ZGVsMxv_expm1",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("expm1f", "_ZGVsMxv_expm1f", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("fdim", "_ZGVsMxvv_fdim",  SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("fdimf", "_ZGVsMxvv_fdimf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("fma", "_ZGVsMxvvv_fma",  SCALABLE(2), MASKED, "_ZGVsMxvvv")
TLI_DEFINE_VECFUNC("fmaf", "_ZGVsMxvvv_fmaf", SCALABLE(4), MASKED, "_ZGVsMxvvv")

TLI_DEFINE_VECFUNC("fmax", "_ZGVsMxvv_fmax",  SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("fmaxf", "_ZGVsMxvv_fmaxf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("fmin", "_ZGVsMxvv_fmin",  SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("fminf", "_ZGVsMxvv_fminf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("fmod", "_ZGVsMxvv_fmod", SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("fmodf", "_ZGVsMxvv_fmodf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("hypot", "_ZGVsMxvv_hypot", SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("hypotf", "_ZGVsMxvv_hypotf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("ilogb", "_ZGVsMxv_ilogb",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("ilogbf", "_ZGVsMxv_ilogbf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("ldexp", "_ZGVsMxvv_ldexp",  SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("ldexpf", "_ZGVsMxvv_ldexpf", SCALABLE(4), MASKED, "_ZGVsMxvv")

```

- **L1065**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1066**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1067**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1068**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1069**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1070**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1071**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1072**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1073**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1074**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1075**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1076**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1077**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1078**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1079**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1080**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1081**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1082**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1083**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1084**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1085**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1086**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1087**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1088**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1089**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1090**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1091**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1092**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1093-1120

```cpp
TLI_DEFINE_VECFUNC("lgamma", "_ZGVsMxv_lgamma",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("lgammaf", "_ZGVsMxv_lgammaf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("log", "_ZGVsMxv_log",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("logf", "_ZGVsMxv_logf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.log.f64", "_ZGVsMxv_log", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.log.f32", "_ZGVsMxv_logf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("log10", "_ZGVsMxv_log10",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("log10f", "_ZGVsMxv_log10f", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.log10.f64", "_ZGVsMxv_log10", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "_ZGVsMxv_log10f", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("log1p", "_ZGVsMxv_log1p",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("log1pf", "_ZGVsMxv_log1pf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("log2", "_ZGVsMxv_log2", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("log2f", "_ZGVsMxv_log2f", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "_ZGVsMxv_log2", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "_ZGVsMxv_log2f", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("modf", "_ZGVsNxvl8_modf", SCALABLE(2), NOMASK, "_ZGVsNxvl8")
TLI_DEFINE_VECFUNC("modff", "_ZGVsNxvl4_modff", SCALABLE(4), NOMASK, "_ZGVsNxvl4")

TLI_DEFINE_VECFUNC("nextafter", "_ZGVsMxvv_nextafter", SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("nextafterf", "_ZGVsMxvv_nextafterf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("pow", "_ZGVsMxvv_pow", SCALABLE(2), MASKED, "_ZGVsMxvv")
```

- **L1093**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1094**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1095**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1096**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1097**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1098**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1099**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1100**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1101**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1102**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1103**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1104**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1105**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1106**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1107**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1108**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1109**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1110**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1111**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1112**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1113**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1114**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1115**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1116**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1117**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1118**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1119**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1120**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1121-1148

```cpp
TLI_DEFINE_VECFUNC("powf", "_ZGVsMxvv_powf", SCALABLE(4), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "_ZGVsMxvv_pow", SCALABLE(2), MASKED, "_ZGVsMxvv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "_ZGVsMxvv_powf", SCALABLE(4), MASKED, "_ZGVsMxvv")

TLI_DEFINE_VECFUNC("sin", "_ZGVsMxv_sin",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("sinf", "_ZGVsMxv_sinf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "_ZGVsMxv_sin", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "_ZGVsMxv_sinf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("sincos", "_ZGVsNxvl8l8_sincos", SCALABLE(2), NOMASK, "_ZGVsNxvl8l8")
TLI_DEFINE_VECFUNC("sincosf", "_ZGVsNxvl4l4_sincosf", SCALABLE(4), NOMASK, "_ZGVsNxvl4l4")

TLI_DEFINE_VECFUNC("sincospi", "_ZGVsNxvl8l8_sincospi", SCALABLE(2), NOMASK, "_ZGVsNxvl8l8")
TLI_DEFINE_VECFUNC("sincospif", "_ZGVsNxvl4l4_sincospif", SCALABLE(4), NOMASK, "_ZGVsNxvl4l4")

TLI_DEFINE_VECFUNC("sinh", "_ZGVsMxv_sinh",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("sinhf", "_ZGVsMxv_sinhf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.sinh.f64", "_ZGVsMxv_sinh", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "_ZGVsMxv_sinhf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("sinpi", "_ZGVsMxv_sinpi",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("sinpif", "_ZGVsMxv_sinpif", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("sqrt", "_ZGVsMxv_sqrt",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("sqrtf", "_ZGVsMxv_sqrtf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("tan", "_ZGVsMxv_tan",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("tanf", "_ZGVsMxv_tanf", SCALABLE(4), MASKED, "_ZGVsMxv")
```

- **L1121**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1122**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1123**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1124**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1125**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1126**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1127**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1128**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1129**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1130**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1131**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1132**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1133**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1134**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1135**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1136**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1137**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1138**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1139**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1140**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1141**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1142**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1143**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1144**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1145**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1146**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1147**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1148**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1149-1176

```cpp
TLI_DEFINE_VECFUNC("llvm.tan.f64", "_ZGVsMxv_tan", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "_ZGVsMxv_tanf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("tanh", "_ZGVsMxv_tanh",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("tanhf", "_ZGVsMxv_tanhf", SCALABLE(4), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.tanh.f64", "_ZGVsMxv_tanh", SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "_ZGVsMxv_tanhf", SCALABLE(4), MASKED, "_ZGVsMxv")

TLI_DEFINE_VECFUNC("tgamma", "_ZGVsMxv_tgamma",  SCALABLE(2), MASKED, "_ZGVsMxv")
TLI_DEFINE_VECFUNC("tgammaf", "_ZGVsMxv_tgammaf", SCALABLE(4), MASKED, "_ZGVsMxv")

#elif defined(TLI_DEFINE_SLEEFGNUABI_SCALABLE_VECFUNCS_RISCV)

TLI_DEFINE_VECFUNC("acos", "Sleef_acosdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.acos.f64", "Sleef_acosdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("acosf", "Sleef_acosfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.acos.f32", "Sleef_acosfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("acosh", "Sleef_acoshdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("acoshf", "Sleef_acoshfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("asin", "Sleef_asindx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.asin.f64", "Sleef_asindx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("asinf", "Sleef_asinfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "Sleef_asinfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("asinh", "Sleef_asinhdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("asinhf", "Sleef_asinhfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
```

- **L1149**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1150**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1151**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1152**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1153**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1154**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1155**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1156**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1157**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1158**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1159**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1160**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L1161**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1162**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1163**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1164**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1165**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1166**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1167**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1168**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1169**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1170**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1171**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1172**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1173**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1174**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1175**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1176**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1177-1204

```cpp

TLI_DEFINE_VECFUNC("atan", "Sleef_atandx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.atan.f64", "Sleef_atandx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("atanf", "Sleef_atanfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "Sleef_atanfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("atan2", "Sleef_atan2dx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("llvm.atan2.f64", "Sleef_atan2dx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("atan2f", "Sleef_atan2fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "Sleef_atan2fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("atanh", "Sleef_atanhdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("atanhf", "Sleef_atanhfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("cbrt", "Sleef_cbrtdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("cbrtf", "Sleef_cbrtfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("copysign", "Sleef_copysigndx_rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("copysignf", "Sleef_copysignfx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("cos", "Sleef_cosdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "Sleef_cosdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("cosf", "Sleef_cosfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "Sleef_cosfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("cosh", "Sleef_coshdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.cosh.f64", "Sleef_coshdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("coshf", "Sleef_coshfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
```

- **L1177**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1178**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1179**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1180**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1181**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1182**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1183**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1184**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1185**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1186**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1187**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1188**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1189**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1190**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1191**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1192**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1193**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1194**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1195**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1196**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1197**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1198**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1199**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1200**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1201**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1202**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1203**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1204**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1205-1232

```cpp
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "Sleef_coshfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("cospi", "Sleef_cospidx_u05rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("cospif", "Sleef_cospifx_u05rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("erf", "Sleef_erfdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("erff", "Sleef_erffx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("erfc", "Sleef_erfcdx_u15rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("erfcf", "Sleef_erfcfx_u15rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("exp", "Sleef_expdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "Sleef_expdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("expf", "Sleef_expfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "Sleef_expfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("exp10", "Sleef_exp10dx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.exp10.f64", "Sleef_exp10dx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("exp10f", "Sleef_exp10fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "Sleef_exp10fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("exp2", "Sleef_exp2dx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "Sleef_exp2dx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("exp2f", "Sleef_exp2fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "Sleef_exp2fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("expm1", "Sleef_expm1dx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("expm1f", "Sleef_expm1fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
```

- **L1205**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1206**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1207**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1208**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1209**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1210**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1211**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1212**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1213**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1214**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1215**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1216**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1217**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1218**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1219**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1220**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1221**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1222**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1223**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1224**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1225**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1226**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1227**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1228**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1229**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1230**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1231**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1232**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1233-1260

```cpp

TLI_DEFINE_VECFUNC("fdim", "Sleef_fdimdx_rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("fdimf", "Sleef_fdimfx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("fma", "Sleef_fmadx_rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvvv")
TLI_DEFINE_VECFUNC("llvm.fma.f64", "Sleef_fmadx_rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvvv")
TLI_DEFINE_VECFUNC("fmaf", "Sleef_fmafx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvvv")
TLI_DEFINE_VECFUNC("llvm.fma.f32", "Sleef_fmafx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvvv")

TLI_DEFINE_VECFUNC("fmax", "Sleef_fmaxdx_rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("fmaxf", "Sleef_fmaxfx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("fmin", "Sleef_fmindx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("fminf", "Sleef_fminfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("fmod", "Sleef_fmoddx_rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("fmodf", "Sleef_fmodfx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("hypot", "Sleef_hypotdx_u05rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("hypotf", "Sleef_hypotfx_u05rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("ilogb", "Sleef_ilogbdx_rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("ilogbf", "Sleef_ilogbfx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("ldexp", "Sleef_ldexpdx_rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("ldexpf", "Sleef_ldexpfx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("lgamma", "Sleef_lgammadx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
```

- **L1233**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1234**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1235**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1236**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1237**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1238**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1239**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1240**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1241**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1242**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1243**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1244**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1245**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1246**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1247**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1248**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1249**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1250**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1251**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1252**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1253**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1254**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1255**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1256**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1257**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1258**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1259**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1260**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1261-1288

```cpp
TLI_DEFINE_VECFUNC("lgammaf", "Sleef_lgammafx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("log", "Sleef_logdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVsNxv")
TLI_DEFINE_VECFUNC("llvm.log.f64", "Sleef_logdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("logf", "Sleef_logfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.log.f32", "Sleef_logfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("log10", "Sleef_log10dx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.log10.f64", "Sleef_log10dx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("log10f", "Sleef_log10fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "Sleef_log10fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("log1p", "Sleef_log1pdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("log1pf", "Sleef_log1pfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("log2", "Sleef_log2dx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "Sleef_log2dx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("log2f", "Sleef_log2fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "Sleef_log2fx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("modf", "Sleef_modfdx_rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvl8")
TLI_DEFINE_VECFUNC("modff", "Sleef_modffx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvl4")

TLI_DEFINE_VECFUNC("nextafter", "Sleef_nextafterdx_rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("nextafterf", "Sleef_nextafterfx_rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("pow", "Sleef_powdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "Sleef_powdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvv")
```

- **L1261**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1262**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1263**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1264**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1265**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1266**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1267**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1268**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1269**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1270**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1271**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1272**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1273**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1274**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1275**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1276**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1277**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1278**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1279**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1280**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1281**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1282**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1283**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1284**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1285**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1286**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1287**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1288**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1289-1316

```cpp
TLI_DEFINE_VECFUNC("powf", "Sleef_powfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "Sleef_powfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvv")

TLI_DEFINE_VECFUNC("sin", "Sleef_sindx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "Sleef_sindx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("sinf", "Sleef_sinfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "Sleef_sinfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("sincos", "Sleef_sincosdx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvl8l8")
TLI_DEFINE_VECFUNC("sincosf", "Sleef_sincosfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvl4l4")

TLI_DEFINE_VECFUNC("sincospi", "Sleef_sincospidx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxvl8l8")
TLI_DEFINE_VECFUNC("sincospif", "Sleef_sincospifx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxvl4l4")

TLI_DEFINE_VECFUNC("sinh", "Sleef_sinhdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.sinh.f64", "Sleef_sinhdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("sinhf", "Sleef_sinhfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "Sleef_sinhfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("sinpi", "Sleef_sinpidx_u05rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("sinpif", "Sleef_sinpifx_u05rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("sqrt", "Sleef_sqrtdx_u05rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.sqrt.f64", "Sleef_sqrtdx_u05rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("sqrtf", "Sleef_sqrtfx_u05rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.sqrt.f32", "Sleef_sqrtfx_u05rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("tan", "Sleef_tandx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
```

- **L1289**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1290**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1291**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1292**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1293**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1294**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1295**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1296**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1297**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1298**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1299**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1300**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1301**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1302**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1303**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1304**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1305**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1306**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1307**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1308**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1309**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1310**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1311**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1312**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1313**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1314**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1315**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1316**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1317-1344

```cpp
TLI_DEFINE_VECFUNC("llvm.tan.f64", "Sleef_tandx_u10rvvm2", SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("tanf", "Sleef_tanfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "Sleef_tanfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("tanh", "Sleef_tanhdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.tanh.f64", "Sleef_tanhdx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("tanhf", "Sleef_tanhfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "Sleef_tanhfx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

TLI_DEFINE_VECFUNC("tgamma", "Sleef_tgammadx_u10rvvm2",  SCALABLE(2), NOMASK, "_ZGVrNxv")
TLI_DEFINE_VECFUNC("tgammaf", "Sleef_tgammafx_u10rvvm2", SCALABLE(4), NOMASK, "_ZGVrNxv")

#elif defined(TLI_DEFINE_ARMPL_VECFUNCS)

TLI_DEFINE_VECFUNC("acos", "armpl_vacosq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acosf", "armpl_vacosq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acos", "armpl_svacos_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("acosf", "armpl_svacos_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.acos.f64", "armpl_vacosq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.acos.f32", "armpl_vacosq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.acos.f64", "armpl_svacos_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.acos.f32", "armpl_svacos_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("acosh", "armpl_vacoshq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acoshf", "armpl_vacoshq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("acosh", "armpl_svacosh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("acoshf", "armpl_svacosh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)
```

- **L1317**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1318**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1319**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1320**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1321**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1322**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1323**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1324**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1325**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1326**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1327**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1328**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1329**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L1330**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1331**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1332**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1333**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1334**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1335**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1336**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1337**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1338**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1339**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1340**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1341**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1342**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1343**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1344**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1345-1372

```cpp

TLI_DEFINE_VECFUNC("asin", "armpl_vasinq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinf", "armpl_vasinq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asin", "armpl_svasin_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("asinf", "armpl_svasin_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.asin.f64", "armpl_vasinq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.asin.f32", "armpl_vasinq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.asin.f64", "armpl_svasin_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.asin.f32", "armpl_svasin_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("asinh", "armpl_vasinhq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinhf", "armpl_vasinhq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("asinh", "armpl_svasinh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("asinhf", "armpl_svasinh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("atan", "armpl_vatanq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanf", "armpl_vatanq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atan", "armpl_svatan_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("atanf", "armpl_svatan_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.atan.f64", "armpl_vatanq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan.f32", "armpl_vatanq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan.f64", "armpl_svatan_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.atan.f32", "armpl_svatan_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("atan2", "armpl_vatan2q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atan2f", "armpl_vatan2q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
```

- **L1345**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1346**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1347**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1348**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1349**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1350**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1351**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1352**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1353**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1354**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1355**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1356**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1357**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1358**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1359**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1360**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1361**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1362**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1363**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1364**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1365**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1366**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1367**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1368**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1369**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1370**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1371**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1372**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1373-1400

```cpp
TLI_DEFINE_VECFUNC("atan2", "armpl_svatan2_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("atan2f", "armpl_svatan2_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("llvm.atan2.f64", "armpl_vatan2q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "armpl_vatan2q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.atan2.f64", "armpl_svatan2_f64_x", SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("llvm.atan2.f32", "armpl_svatan2_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("atanh", "armpl_vatanhq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanhf", "armpl_vatanhq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("atanh", "armpl_svatanh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("atanhf", "armpl_svatanh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("cbrt", "armpl_vcbrtq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cbrtf", "armpl_vcbrtq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cbrt", "armpl_svcbrt_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("cbrtf", "armpl_svcbrt_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("copysign", "armpl_vcopysignq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("copysignf", "armpl_vcopysignq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("copysign", "armpl_svcopysign_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("copysignf", "armpl_svcopysign_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("cos", "armpl_vcosq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cosf", "armpl_vcosq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cos", "armpl_svcos_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("cosf", "armpl_svcos_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

```

- **L1373**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1374**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1375**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1376**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1377**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1378**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1379**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1380**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1381**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1382**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1383**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1384**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1385**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1386**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1387**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1388**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1389**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1390**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1391**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1392**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1393**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1394**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1395**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1396**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1397**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1398**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1399**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1400**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1401-1428

```cpp
TLI_DEFINE_VECFUNC("llvm.cos.f64", "armpl_vcosq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cos.f32", "armpl_vcosq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cos.f64", "armpl_svcos_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.cos.f32", "armpl_svcos_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("cosh", "armpl_vcoshq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("coshf", "armpl_vcoshq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cosh", "armpl_svcosh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("coshf", "armpl_svcosh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.cosh.f64", "armpl_vcoshq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "armpl_vcoshq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.cosh.f64", "armpl_svcosh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "armpl_svcosh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("cospi", "armpl_vcospiq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cospif", "armpl_vcospiq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("cospi", "armpl_svcospi_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("cospif", "armpl_svcospi_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("erf", "armpl_verfq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erff", "armpl_verfq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erf", "armpl_sverf_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("erff", "armpl_sverf_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("erfc", "armpl_verfcq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erfcf", "armpl_verfcq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("erfc", "armpl_sverfc_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
```

- **L1401**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1402**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1403**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1404**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1405**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1406**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1407**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1408**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1409**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1410**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1411**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1412**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1413**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1414**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1415**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1416**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1417**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1418**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1419**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1420**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1421**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1422**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1423**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1424**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1425**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1426**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1427**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1428**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1429-1456

```cpp
TLI_DEFINE_VECFUNC("erfcf", "armpl_sverfc_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("exp", "armpl_vexpq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expf", "armpl_vexpq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp", "armpl_svexp_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("expf", "armpl_svexp_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.exp.f64", "armpl_vexpq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp.f32", "armpl_vexpq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp.f64", "armpl_svexp_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.exp.f32", "armpl_svexp_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("exp10", "armpl_vexp10q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp10f", "armpl_vexp10q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp10", "armpl_svexp10_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("exp10f", "armpl_svexp10_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.exp10.f64", "armpl_vexp10q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "armpl_vexp10q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp10.f64", "armpl_svexp10_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "armpl_svexp10_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("exp2", "armpl_vexp2q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp2f", "armpl_vexp2q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("exp2", "armpl_svexp2_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("exp2f", "armpl_svexp2_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.exp2.f64", "armpl_vexp2q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
```

- **L1429**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1430**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1431**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1432**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1433**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1434**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1435**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1436**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1437**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1438**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1439**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1440**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1441**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1442**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1443**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1444**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1445**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1446**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1447**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1448**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1449**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1450**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1451**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1452**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1453**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1454**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1455**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1456**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1457-1484

```cpp
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "armpl_vexp2q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "armpl_svexp2_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "armpl_svexp2_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("expm1", "armpl_vexpm1q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expm1f", "armpl_vexpm1q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("expm1", "armpl_svexpm1_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("expm1f", "armpl_svexpm1_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("fdim", "armpl_vfdimq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fdimf", "armpl_vfdimq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fdim", "armpl_svfdim_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("fdimf", "armpl_svfdim_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("fma", "armpl_vfmaq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vvv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fmaf", "armpl_vfmaq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vvv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fma", "armpl_svfma_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvvv", NOCC)
TLI_DEFINE_VECFUNC("fmaf", "armpl_svfma_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvvv", NOCC)

TLI_DEFINE_VECFUNC("fmax", "armpl_vfmaxq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fmaxf", "armpl_vfmaxq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fmax", "armpl_svfmax_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("fmaxf", "armpl_svfmax_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("fmin", "armpl_vfminq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fminf", "armpl_vfminq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fmin", "armpl_svfmin_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("fminf", "armpl_svfmin_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)
```

- **L1457**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1458**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1459**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1460**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1461**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1462**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1463**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1464**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1465**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1466**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1467**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1468**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1469**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1470**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1471**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1472**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1473**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1474**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1475**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1476**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1477**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1478**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1479**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1480**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1481**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1482**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1483**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1484**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1485-1512

```cpp

TLI_DEFINE_VECFUNC("fmod", "armpl_vfmodq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fmodf", "armpl_vfmodq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("fmod", "armpl_svfmod_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("fmodf", "armpl_svfmod_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("hypot", "armpl_vhypotq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("hypotf", "armpl_vhypotq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("hypot", "armpl_svhypot_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("hypotf", "armpl_svhypot_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("ilogb", "armpl_vilogbq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("ilogbf", "armpl_vilogbq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("ilogb", "armpl_svilogb_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("ilogbf", "armpl_svilogb_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("ldexp", "armpl_vldexpq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("ldexpf", "armpl_vldexpq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("ldexp", "armpl_svldexp_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("ldexpf", "armpl_svldexp_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("lgamma", "armpl_vlgammaq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("lgammaf", "armpl_vlgammaq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("lgamma", "armpl_svlgamma_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("lgammaf", "armpl_svlgamma_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("log", "armpl_vlogq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("logf", "armpl_vlogq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
```

- **L1485**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1486**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1487**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1488**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1489**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1490**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1491**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1492**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1493**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1494**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1495**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1496**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1497**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1498**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1499**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1500**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1501**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1502**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1503**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1504**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1505**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1506**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1507**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1508**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1509**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1510**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1511**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1512**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1513-1540

```cpp
TLI_DEFINE_VECFUNC("log", "armpl_svlog_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("logf", "armpl_svlog_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.log.f64", "armpl_vlogq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log.f32", "armpl_vlogq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log.f64", "armpl_svlog_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.log.f32", "armpl_svlog_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("log10", "armpl_vlog10q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log10f", "armpl_vlog10q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log10", "armpl_svlog10_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("log10f", "armpl_svlog10_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.log10.f64", "armpl_vlog10q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log10.f32", "armpl_vlog10q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log10.f64", "armpl_svlog10_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.log10.f32", "armpl_svlog10_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("log1p", "armpl_vlog1pq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log1pf", "armpl_vlog1pq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log1p", "armpl_svlog1p_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("log1pf", "armpl_svlog1p_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("log2", "armpl_vlog2q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log2f", "armpl_vlog2q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("log2", "armpl_svlog2_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("log2f", "armpl_svlog2_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

```

- **L1513**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1514**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1515**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1516**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1517**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1518**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1519**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1520**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1521**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1522**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1523**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1524**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1525**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1526**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1527**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1528**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1529**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1530**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1531**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1532**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1533**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1534**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1535**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1536**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1537**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1538**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1539**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1540**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1541-1568

```cpp
TLI_DEFINE_VECFUNC("llvm.log2.f64", "armpl_vlog2q_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log2.f32", "armpl_vlog2q_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.log2.f64", "armpl_svlog2_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.log2.f32", "armpl_svlog2_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("modf", "armpl_vmodfq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vl8", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("modff", "armpl_vmodfq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vl4", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("modf", "armpl_svmodf_f64_x", SCALABLE(2), MASKED, "_ZGVsMxvl8", NOCC)
TLI_DEFINE_VECFUNC("modff", "armpl_svmodf_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvl4", NOCC)

TLI_DEFINE_VECFUNC("llvm.modf.f64", "armpl_vmodfq_f64_stret", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.modf.f32", "armpl_vmodfq_f32_stret", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.modf.f64", "armpl_svmodf_f64_stret_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.modf.f32", "armpl_svmodf_f32_stret_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("nextafter", "armpl_vnextafterq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("nextafterf", "armpl_vnextafterq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("nextafter", "armpl_svnextafter_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("nextafterf", "armpl_svnextafter_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("pow", "armpl_vpowq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("powf", "armpl_vpowq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("pow", "armpl_svpow_f64_x", SCALABLE(2), MASKED, "_ZGVsMxvv", NOCC)
TLI_DEFINE_VECFUNC("powf", "armpl_svpow_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvv", NOCC)

TLI_DEFINE_VECFUNC("sin", "armpl_vsinq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinf", "armpl_vsinq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sin", "armpl_svsin_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
```

- **L1541**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1542**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1543**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1544**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1545**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1546**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1547**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1548**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1549**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1550**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1551**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1552**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1553**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1554**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1555**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1556**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1557**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1558**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1559**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1560**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1561**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1562**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1563**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1564**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1565**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1566**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1567**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1568**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1569-1596

```cpp
TLI_DEFINE_VECFUNC("sinf", "armpl_svsin_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.sin.f64", "armpl_vsinq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sin.f32", "armpl_vsinq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sin.f64", "armpl_svsin_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.sin.f32", "armpl_svsin_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("sincos", "armpl_vsincosq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vl8l8", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sincosf", "armpl_vsincosq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vl4l4", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sincos", "armpl_svsincos_f64_x", SCALABLE(2), MASKED, "_ZGVsMxvl8l8", NOCC)
TLI_DEFINE_VECFUNC("sincosf", "armpl_svsincos_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvl4l4", NOCC)

TLI_DEFINE_VECFUNC("llvm.sincos.f64", "armpl_vcexpiq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sincos.f32", "armpl_vcexpiq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sincos.f64", "armpl_svcexpi_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.sincos.f32", "armpl_svcexpi_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("sincospi", "armpl_vsincospiq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2vl8l8", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sincospif", "armpl_vsincospiq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4vl4l4", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sincospi", "armpl_svsincospi_f64_x", SCALABLE(2), MASKED, "_ZGVsMxvl8l8", NOCC)
TLI_DEFINE_VECFUNC("sincospif", "armpl_svsincospi_f32_x", SCALABLE(4), MASKED, "_ZGVsMxvl4l4", NOCC)

TLI_DEFINE_VECFUNC("llvm.sincospi.f64", "armpl_vcexpipiq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sincospi.f32", "armpl_vcexpipiq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sincospi.f64", "armpl_svcexpipi_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.sincospi.f32", "armpl_svcexpipi_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("sinh", "armpl_vsinhq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
```

- **L1569**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1570**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1571**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1572**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1573**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1574**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1575**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1576**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1577**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1578**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1579**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1580**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1581**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1582**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1583**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1584**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1585**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1586**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1587**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1588**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1589**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1590**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1591**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1592**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1593**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1594**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1595**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1596**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1597-1624

```cpp
TLI_DEFINE_VECFUNC("sinhf", "armpl_vsinhq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinh", "armpl_svsinh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("sinhf", "armpl_svsinh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.sinh.f64", "armpl_vsinhq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "armpl_vsinhq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.sinh.f64", "armpl_svsinh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.sinh.f32", "armpl_svsinh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("sinpi", "armpl_vsinpiq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinpif", "armpl_vsinpiq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sinpi", "armpl_svsinpi_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("sinpif", "armpl_svsinpi_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("sqrt", "armpl_vsqrtq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sqrtf", "armpl_vsqrtq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("sqrt", "armpl_svsqrt_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("sqrtf", "armpl_svsqrt_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("tan", "armpl_vtanq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tanf", "armpl_vtanq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tan", "armpl_svtan_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("tanf", "armpl_svtan_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.tan.f64", "armpl_vtanq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tan.f32", "armpl_vtanq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tan.f64", "armpl_svtan_f64_x", SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.tan.f32", "armpl_svtan_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)
```

- **L1597**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1598**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1599**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1600**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1601**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1602**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1603**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1604**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1605**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1606**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1607**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1608**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1609**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1610**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1611**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1612**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1613**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1614**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1615**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1616**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1617**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1618**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1619**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1620**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1621**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1622**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1623**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1624**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1625-1652

```cpp

TLI_DEFINE_VECFUNC("tanh", "armpl_vtanhq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tanhf", "armpl_vtanhq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tanh", "armpl_svtanh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("tanhf", "armpl_svtanh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("llvm.tanh.f64", "armpl_vtanhq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "armpl_vtanhq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("llvm.tanh.f64", "armpl_svtanh_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "armpl_svtanh_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

TLI_DEFINE_VECFUNC("tgamma", "armpl_vtgammaq_f64", FIXED(2), NOMASK, "_ZGV_LLVM_N2v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tgammaf", "armpl_vtgammaq_f32", FIXED(4), NOMASK, "_ZGV_LLVM_N4v", CallingConv::AArch64_VectorCall)
TLI_DEFINE_VECFUNC("tgamma", "armpl_svtgamma_f64_x",  SCALABLE(2), MASKED, "_ZGVsMxv", NOCC)
TLI_DEFINE_VECFUNC("tgammaf", "armpl_svtgamma_f32_x", SCALABLE(4), MASKED, "_ZGVsMxv", NOCC)

#elif defined(TLI_DEFINE_AMDLIBM_VECFUNCS)
TLI_DEFINE_VECFUNC("sinf", "amd_vrs16_sinf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("sinf", "amd_vrs8_sinf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("sinf", "amd_vrs4_sinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sin", "amd_vrd8_sin", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("sin", "amd_vrd4_sin", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("sin", "amd_vrd2_sin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("llvm.sin.f32", "amd_vrs16_sinf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "amd_vrs8_sinf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.sin.f32", "amd_vrs4_sinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "amd_vrd8_sin", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
```

- **L1625**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1626**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1627**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1628**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1629**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1630**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1631**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1632**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1633**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1634**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1635**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1636**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1637**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1638**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1639**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1640**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1641**: Introduces a conditional-compilation check that selects declarations for specific build settings. / 引入条件编译检查，以便为特定构建设置选择声明。
- **L1642**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1643**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1644**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1645**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1646**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1647**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1648**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1649**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1650**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1651**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1652**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1653-1680

```cpp
TLI_DEFINE_VECFUNC("llvm.sin.f64", "amd_vrd4_sin", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.sin.f64", "amd_vrd2_sin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("cosf", "amd_vrs16_cosf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("cosf", "amd_vrs8_cosf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("cosf", "amd_vrs4_cosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cos", "amd_vrd8_cos", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("cos", "amd_vrd4_cos", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cos", "amd_vrd2_cos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("llvm.cos.f32", "amd_vrs16_cosf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "amd_vrs8_cosf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.cos.f32", "amd_vrs4_cosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "amd_vrd8_cos", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "amd_vrd4_cos", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cos.f64", "amd_vrd2_cos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("expf", "amd_vrs16_expf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("expf", "amd_vrs8_expf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("expf", "amd_vrs4_expf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp", "amd_vrd2_exp", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("exp", "amd_vrd4_exp", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp", "amd_vrd8_exp", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__expf_finite", "amd_vrs16_expf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("__expf_finite", "amd_vrs8_expf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__expf_finite", "amd_vrs4_expf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__exp_finite", "amd_vrd2_exp", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
```

- **L1653**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1654**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1655**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1656**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1657**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1658**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1659**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1660**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1661**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1662**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1663**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1664**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1665**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1666**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1667**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1668**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1669**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1670**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1671**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1672**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1673**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1674**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1675**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1676**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1677**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1678**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1679**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1680**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1681-1708

```cpp
TLI_DEFINE_VECFUNC("__exp_finite", "amd_vrd4_exp", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__exp_finite", "amd_vrd8_exp", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.exp.f32", "amd_vrs16_expf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "amd_vrs8_expf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.exp.f32", "amd_vrs4_expf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "amd_vrd2_exp", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "amd_vrd4_exp", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp.f64", "amd_vrd8_exp", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("exp2f", "amd_vrs16_exp2f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("exp2f", "amd_vrs8_exp2f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("exp2f", "amd_vrs4_exp2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp2", "amd_vrd2_exp2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("exp2", "amd_vrd4_exp2", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("exp2", "amd_vrd8_exp2", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__exp2f_finite", "amd_vrs16_exp2f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("__exp2f_finite", "amd_vrs8_exp2f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__exp2f_finite", "amd_vrs4_exp2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__exp2_finite", "amd_vrd2_exp2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__exp2_finite", "amd_vrd4_exp2", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__exp2_finite", "amd_vrd8_exp2", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.exp2.f32", "amd_vrs16_exp2f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "amd_vrs8_exp2f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.exp2.f32", "amd_vrs4_exp2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "amd_vrd2_exp2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
```

- **L1681**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1682**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1683**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1684**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1685**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1686**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1687**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1688**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1689**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1690**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1691**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1692**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1693**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1694**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1695**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1696**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1697**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1698**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1699**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1700**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1701**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1702**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1703**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1704**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1705**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1706**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1707**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1708**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1709-1736

```cpp
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "amd_vrd4_exp2", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.exp2.f64", "amd_vrd8_exp2", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("powf", "amd_vrs16_powf", FIXED(16), NOMASK, "_ZGV_LLVM_N16vv")
TLI_DEFINE_VECFUNC("powf", "amd_vrs8_powf", FIXED(8), NOMASK, "_ZGV_LLVM_N8vv")
TLI_DEFINE_VECFUNC("powf", "amd_vrs4_powf", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("pow", "amd_vrd2_pow", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("pow", "amd_vrd4_pow", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("pow", "amd_vrd8_pow", FIXED(8), NOMASK, "_ZGV_LLVM_N8vv")

TLI_DEFINE_VECFUNC("__powf_finite", "amd_vrs16_powf", FIXED(16), NOMASK, "_ZGV_LLVM_N16vv")
TLI_DEFINE_VECFUNC("__powf_finite", "amd_vrs8_powf", FIXED(8), NOMASK, "_ZGV_LLVM_N8vv")
TLI_DEFINE_VECFUNC("__powf_finite", "amd_vrs4_powf", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("__pow_finite", "amd_vrd2_pow", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("__pow_finite", "amd_vrd4_pow", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("__pow_finite", "amd_vrd8_pow", FIXED(8), NOMASK, "_ZGV_LLVM_N8vv")

TLI_DEFINE_VECFUNC("llvm.pow.f32", "amd_vrs16_powf", FIXED(16), NOMASK, "_ZGV_LLVM_N16vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "amd_vrs8_powf", FIXED(8), NOMASK, "_ZGV_LLVM_N8vv")
TLI_DEFINE_VECFUNC("llvm.pow.f32", "amd_vrs4_powf", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "amd_vrd2_pow", FIXED(2), NOMASK, "_ZGV_LLVM_N2vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "amd_vrd4_pow", FIXED(4), NOMASK, "_ZGV_LLVM_N4vv")
TLI_DEFINE_VECFUNC("llvm.pow.f64", "amd_vrd8_pow", FIXED(8), NOMASK, "_ZGV_LLVM_N8vv")

TLI_DEFINE_VECFUNC("logf", "amd_vrs16_logf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("logf", "amd_vrs8_logf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("logf", "amd_vrs4_logf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log", "amd_vrd2_log", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
```

- **L1709**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1710**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1711**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1712**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1713**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1714**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1715**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1716**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1717**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1718**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1719**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1720**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1721**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1722**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1723**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1724**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1725**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1726**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1727**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1728**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1729**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1730**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1731**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1732**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1733**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1734**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1735**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1736**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1737-1764

```cpp
TLI_DEFINE_VECFUNC("log", "amd_vrd4_log", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log", "amd_vrd8_log", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__logf_finite", "amd_vrs16_logf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("__logf_finite", "amd_vrs8_logf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__logf_finite", "amd_vrs4_logf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log_finite", "amd_vrd2_log", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__log_finite", "amd_vrd4_log", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log_finite", "amd_vrd8_log", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.log.f32", "amd_vrs16_logf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "amd_vrs8_logf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.log.f32", "amd_vrs4_logf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "amd_vrd2_log", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "amd_vrd4_log", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log.f64", "amd_vrd8_log", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("log2f", "amd_vrs16_log2f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("log2f", "amd_vrs8_log2f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("log2f", "amd_vrs4_log2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log2", "amd_vrd2_log2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log2", "amd_vrd4_log2", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("log2", "amd_vrd8_log2", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("__log2f_finite", "amd_vrs16_log2f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("__log2f_finite", "amd_vrs8_log2f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__log2f_finite", "amd_vrs4_log2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log2_finite", "amd_vrd2_log2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
```

- **L1737**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1738**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1739**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1740**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1741**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1742**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1743**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1744**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1745**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1746**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1747**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1748**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1749**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1750**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1751**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1752**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1753**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1754**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1755**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1756**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1757**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1758**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1759**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1760**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1761**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1762**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1763**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1764**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1765-1792

```cpp
TLI_DEFINE_VECFUNC("__log2_finite", "amd_vrd4_log2", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__log2_finite", "amd_vrd8_log2", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.log2.f32", "amd_vrs16_log2f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "amd_vrs8_log2f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.log2.f32", "amd_vrs4_log2f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "amd_vrd2_log2", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "amd_vrd4_log2", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.log2.f64", "amd_vrd8_log2", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("log10", "amd_vrd2_log10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log10f", "amd_vrs16_log10f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("log10f", "amd_vrs8_log10f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("log10f", "amd_vrs4_log10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("__log10_finite", "amd_vrd2_log10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__log10f_finite", "amd_vrs16_log10f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("__log10f_finite", "amd_vrs8_log10f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__log10f_finite", "amd_vrs4_log10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.log10.f64", "amd_vrd2_log10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "amd_vrs16_log10f", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "amd_vrs8_log10f", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.log10.f32", "amd_vrs4_log10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("erff", "amd_vrs4_erff", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("erff", "amd_vrs8_erff", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("erff", "amd_vrs16_erff", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
```

- **L1765**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1766**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1767**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1768**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1769**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1770**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1771**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1772**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1773**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1774**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1775**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1776**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1777**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1778**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1779**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1780**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1781**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1782**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1783**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1784**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1785**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1786**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1787**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1788**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1789**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1790**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1791**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1792**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1793-1820

```cpp
TLI_DEFINE_VECFUNC("erf", "amd_vrd2_erf", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("erf", "amd_vrd4_erf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("erf", "amd_vrd8_erf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("exp10", "amd_vrd2_exp10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("exp10f", "amd_vrs4_exp10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("__exp10_finite", "amd_vrd2_exp10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__exp10f_finite", "amd_vrs4_exp10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.exp10.f64", "amd_vrd2_exp10", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.exp10.f32", "amd_vrs4_exp10f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("expm1f", "amd_vrs4_expm1f", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("log1p", "amd_vrd2_log1p", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("log1pf", "amd_vrs4_log1pf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("tan", "amd_vrd2_tan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("tan", "amd_vrd4_tan", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tan", "amd_vrd8_tan", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("tanf", "amd_vrs4_tanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanf", "amd_vrs8_tanf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("tanf", "amd_vrs16_tanf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.tan.f32", "amd_vrs16_tanf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "amd_vrs8_tanf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.tan.f32", "amd_vrs4_tanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
```

- **L1793**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1794**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1795**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1796**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1797**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1798**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1799**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1800**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1801**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1802**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1803**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1804**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1805**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1806**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1807**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1808**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1809**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1810**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1811**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1812**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1813**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1814**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1815**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1816**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1817**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1818**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1819**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1820**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1821-1848

```cpp
TLI_DEFINE_VECFUNC("llvm.tan.f64", "amd_vrd8_tan", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.tan.f64", "amd_vrd4_tan", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tan.f64", "amd_vrd2_tan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("asin", "amd_vrd8_asin", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("asinf", "amd_vrs4_asinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("asinf", "amd_vrs8_asinf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("asinf", "amd_vrs16_asinf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.asin.f64", "amd_vrd8_asin", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "amd_vrs4_asinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "amd_vrs8_asinf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.asin.f32", "amd_vrs16_asinf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("__asin_finite", "amd_vrd8_asin", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__asinf_finite", "amd_vrs4_asinf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__asinf_finite", "amd_vrs8_asinf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__asinf_finite", "amd_vrs16_asinf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("acosf", "amd_vrs4_acosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("acosf", "amd_vrs8_acosf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("acosf", "amd_vrs16_acosf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("__acosf_finite", "amd_vrs4_acosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("__acosf_finite", "amd_vrs8_acosf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("__acosf_finite", "amd_vrs16_acosf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.acos.f32", "amd_vrs16_acosf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
```

- **L1821**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1822**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1823**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1824**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1825**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1826**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1827**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1828**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1829**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1830**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1831**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1832**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1833**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1834**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1835**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1836**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1837**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1838**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1839**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1840**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1841**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1842**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1843**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1844**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1845**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1846**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1847**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1848**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1849-1876

```cpp
TLI_DEFINE_VECFUNC("llvm.acos.f32", "amd_vrs8_acosf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.acos.f32", "amd_vrs4_acosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("atan", "amd_vrd2_atan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("atan", "amd_vrd4_atan", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atan", "amd_vrd8_atan", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("atanf", "amd_vrs4_atanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("atanf", "amd_vrs8_atanf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("atanf", "amd_vrs16_atanf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("llvm.atan.f64", "amd_vrd2_atan", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.atan.f64", "amd_vrd4_atan", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.atan.f64", "amd_vrd8_atan", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "amd_vrs4_atanf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "amd_vrs8_atanf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.atan.f32", "amd_vrs16_atanf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("cosh",  "amd_vrd2_cosh" , FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("coshf", "amd_vrs4_coshf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("coshf", "amd_vrs8_coshf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("llvm.cosh.f64", "amd_vrd2_cosh" , FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "amd_vrs4_coshf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.cosh.f32", "amd_vrs8_coshf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("tanhf", "amd_vrs4_tanhf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("tanhf", "amd_vrs8_tanhf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("tanhf", "amd_vrs16_tanhf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
```

- **L1849**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1850**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1851**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1852**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1853**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1854**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1855**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1856**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1857**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1858**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1859**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1860**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1861**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1862**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1863**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1864**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1865**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1866**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1867**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1868**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1869**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1870**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1871**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1872**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1873**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1874**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1875**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1876**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1877-1904

```cpp

TLI_DEFINE_VECFUNC("llvm.tanh.f32", "amd_vrs4_tanhf",  FIXED(4),  NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "amd_vrs8_tanhf",  FIXED(8),  NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.tanh.f32", "amd_vrs16_tanhf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("cbrt", "amd_vrd2_cbrt", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cbrtf", "amd_vrs4_cbrtf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("sincos", "amd_vrd2_sincos", FIXED(2), NOMASK, "_ZGV_LLVM_N2vl8l8")
TLI_DEFINE_VECFUNC("sincos", "amd_vrd4_sincos", FIXED(4), NOMASK, "_ZGV_LLVM_N4vl8l8")
TLI_DEFINE_VECFUNC("sincos", "amd_vrd8_sincos", FIXED(8), NOMASK, "_ZGV_LLVM_N8vl8l8")

TLI_DEFINE_VECFUNC("sincosf", "amd_vrs4_sincosf", FIXED(4), NOMASK, "_ZGV_LLVM_N4vl4l4")
TLI_DEFINE_VECFUNC("sincosf", "amd_vrs8_sincosf", FIXED(8), NOMASK, "_ZGV_LLVM_N8vl4l4")
TLI_DEFINE_VECFUNC("sincosf", "amd_vrs16_sincosf", FIXED(16), NOMASK, "_ZGV_LLVM_N16vl4l4")

TLI_DEFINE_VECFUNC("asin", "amd_vrd2_asin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("asin", "amd_vrd4_asin", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("llvm.asin.f64", "amd_vrd2_asin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.asin.f64", "amd_vrd4_asin", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("__asin_finite", "amd_vrd2_asin", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__asin_finite", "amd_vrd4_asin", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("acos", "amd_vrd2_acos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("acos", "amd_vrd4_acos", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

```

- **L1877**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1878**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1879**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1880**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1881**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1882**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1883**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1884**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1885**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1886**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1887**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1888**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1889**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1890**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1891**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1892**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1893**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1894**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1895**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1896**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1897**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1898**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1899**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1900**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1901**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1902**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1903**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1904**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。

### Lines 1905-1932

```cpp
TLI_DEFINE_VECFUNC("llvm.acos.f64", "amd_vrd2_acos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("llvm.acos.f64", "amd_vrd4_acos", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("__acos_finite", "amd_vrd2_acos", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("__acos_finite", "amd_vrd4_acos", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")

TLI_DEFINE_VECFUNC("erfc", "amd_vrd2_erfc", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("erfc", "amd_vrd4_erfc", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("erfc", "amd_vrd8_erfc", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("erfinv", "amd_vrd2_erfinv", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("erfinv", "amd_vrd4_erfinv", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("erfinv", "amd_vrd8_erfinv", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("erfcinv", "amd_vrd2_erfcinv", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("erfcinv", "amd_vrd4_erfcinv", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("erfcinv", "amd_vrd8_erfcinv", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("erfcf", "amd_vrs4_erfcf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("erfcf", "amd_vrs8_erfcf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("erfcf", "amd_vrs16_erfcf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")

TLI_DEFINE_VECFUNC("cdfnorm", "amd_vrd2_cdfnorm", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cdfnorm", "amd_vrd4_cdfnorm", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("cdfnorm", "amd_vrd8_cdfnorm", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("cdfnorminv", "amd_vrd2_cdfnorminv", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
TLI_DEFINE_VECFUNC("cdfnorminv", "amd_vrd4_cdfnorminv", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
```

- **L1905**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1906**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1907**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1908**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1909**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1910**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1911**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1912**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1913**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1914**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1915**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1916**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1917**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1918**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1919**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1920**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1921**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1922**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1923**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1924**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1925**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1926**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1927**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1928**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1929**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1930**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1931**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1932**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。

### Lines 1933-1957

```cpp
TLI_DEFINE_VECFUNC("cdfnorminv", "amd_vrd8_cdfnorminv", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")

TLI_DEFINE_VECFUNC("roundf", "amd_vrs16_roundf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("roundf", "amd_vrs8_roundf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("roundf", "amd_vrs4_roundf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("round", "amd_vrd8_round", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("round", "amd_vrd4_round", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("round", "amd_vrd2_round", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")

TLI_DEFINE_VECFUNC("llvm.round.f32", "amd_vrs16_roundf", FIXED(16), NOMASK, "_ZGV_LLVM_N16v")
TLI_DEFINE_VECFUNC("llvm.round.f32", "amd_vrs8_roundf", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.round.f32", "amd_vrs4_roundf", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.round.f64", "amd_vrd8_round", FIXED(8), NOMASK, "_ZGV_LLVM_N8v")
TLI_DEFINE_VECFUNC("llvm.round.f64", "amd_vrd4_round", FIXED(4), NOMASK, "_ZGV_LLVM_N4v")
TLI_DEFINE_VECFUNC("llvm.round.f64", "amd_vrd2_round", FIXED(2), NOMASK, "_ZGV_LLVM_N2v")
#else
#error "Must choose which vector library functions are to be defined."
#endif

#undef MASKED
#undef NOMASK
#undef SCALABLE
#undef FIXED

#undef TLI_DEFINE_VECFUNC
```

- **L1933**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1934**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1935**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1936**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1937**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1938**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1939**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1940**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1941**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1942**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1943**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1944**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1945**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1946**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1947**: Invokes macro `TLI_DEFINE_VECFUNC` to emit generated declarations, attributes, or table entries. / 调用宏 `TLI_DEFINE_VECFUNC` 来生成声明、属性或表项。
- **L1948**: Switches to the alternate branch of the surrounding preprocessor conditional. / 切换到当前预处理条件的另一条分支。
- **L1949**: Continues the surrounding declaration or implementation logic. / 继续展开周围的声明或实现逻辑。
- **L1950**: Closes the current preprocessor conditional or header guard. / 结束当前的预处理条件块或头文件保护。
- **L1951**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1952**: Undefines macro `MASKED` to keep the preprocessor namespace clean after inclusion-based reuse. / 取消定义宏 `MASKED`，以便在基于包含的复用之后清理预处理器命名空间。
- **L1953**: Undefines macro `NOMASK` to keep the preprocessor namespace clean after inclusion-based reuse. / 取消定义宏 `NOMASK`，以便在基于包含的复用之后清理预处理器命名空间。
- **L1954**: Undefines macro `SCALABLE` to keep the preprocessor namespace clean after inclusion-based reuse. / 取消定义宏 `SCALABLE`，以便在基于包含的复用之后清理预处理器命名空间。
- **L1955**: Undefines macro `FIXED` to keep the preprocessor namespace clean after inclusion-based reuse. / 取消定义宏 `FIXED`，以便在基于包含的复用之后清理预处理器命名空间。
- **L1956**: Blank line separating adjacent declarations or logic blocks. / 空行，用于分隔相邻的声明或逻辑块。
- **L1957**: Undefines macro `TLI_DEFINE_VECFUNC` to keep the preprocessor namespace clean after inclusion-based reuse. / 取消定义宏 `TLI_DEFINE_VECFUNC`，以便在基于包含的复用之后清理预处理器命名空间。

## Key Concepts / 关键概念

- EN: Layer: `Analysis` belongs to LLVM's analysis interfaces and cached program facts subsystem.
  - CN: 层次：`Analysis` 属于 LLVM 的分析接口与缓存的程序事实子系统。
- EN: X-macro pattern: the file is intended to be included multiple times under different macro definitions to generate tables or boilerplate.
  - CN: X-macro 模式：该文件预期在不同宏定义下被多次包含，以生成表格或样板代码。

## Dependencies / 依赖关系

- EN: Macro consumers: entries here are activated by macros such as `LLVM`, `LICENSE`, `SPDX`, `WITH`, `MASS`, `SVML`, `FIXED`, `SCALABLE`, which are expected to be defined by including files.
  - CN: 宏消费者：这里的条目通过 `LLVM`, `LICENSE`, `SPDX`, `WITH`, `MASS`, `SVML`, `FIXED`, `SCALABLE` 等宏被激活，而这些宏通常由包含它的文件预先定义。
