# DiagnosticDriverKinds.td — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `clang/include/clang/Basic/DiagnosticDriverKinds.td`
- **Repository / 仓库**: `/root/xw/llvm-project`
- **Purpose (EN)**: Declares foundational compiler metadata, diagnostics, target descriptions, or builtin-related definitions for `DiagnosticDriverKinds`.
- **Purpose (CN)**: 声明与 `DiagnosticDriverKinds` 相关的编译器基础元数据、诊断信息、目标描述或 builtin 定义。
- **Line Count / 行数**: 970

## Line-by-Line Analysis / 逐行分析

### Lines 1-24

````tablegen
//==--- DiagnosticDriverKinds.td - libdriver diagnostics ------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

let Component = "Driver" in {

def err_drv_no_such_file : Error<"no such file or directory: '%0'">;
def err_drv_no_such_file_with_suggestion : Error<
  "no such file or directory: '%0'; did you mean '%1'?">;
def err_drv_unsupported_opt : Error<"unsupported option '%0'">;
def err_drv_unsupported_opt_with_suggestion : Error<
  "unsupported option '%0'; did you mean '%1'?">;
def err_drv_unsupported_opt_for_target : Error<
  "unsupported option '%0' for target '%1'">;
def err_drv_unsupported_opt_for_language_mode : Error<
  "unsupported option '%0' for language mode '%1'">;
def err_drv_unsupported_option_argument : Error<
  "unsupported argument '%1' to option '%0'">;
def err_drv_unsupported_option_argument_for_target : Error<
  "unsupported argument '%1' to option '%0' for target '%2'">;
````
- **L1 EN**: Comment explains nearby logic, constraints, or intent: `DiagnosticDriverKinds.td - libdriver diagnostics`.
  **L1 CN**: 注释解释附近代码的逻辑、约束或设计意图：`DiagnosticDriverKinds.td - libdriver diagnostics`。
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
- **L8 EN**: Blank line separating nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L9 EN**: Assigns a TableGen property that affects following records or inherited fields: `let Component = "Driver" in {`.
  **L9 CN**: 设置一个 TableGen 属性，影响后续记录或继承字段：`let Component = "Driver" in {`。
- **L10 EN**: Blank line separating nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L11 EN**: Declares TableGen def record `err_drv_no_such_file`.
  **L11 CN**: 声明 TableGen def 记录 `err_drv_no_such_file`。
- **L12 EN**: Declares TableGen def record `err_drv_no_such_file_with_suggestion`.
  **L12 CN**: 声明 TableGen def 记录 `err_drv_no_such_file_with_suggestion`。
- **L13 EN**: Adds a standalone statement or declaration: `"no such file or directory: '%0'; did you mean '%1'?">;`.
  **L13 CN**: 添加一条独立语句或声明：`"no such file or directory: '%0'; did you mean '%1'?">;`。
- **L14 EN**: Declares TableGen def record `err_drv_unsupported_opt`.
  **L14 CN**: 声明 TableGen def 记录 `err_drv_unsupported_opt`。
- **L15 EN**: Declares TableGen def record `err_drv_unsupported_opt_with_suggestion`.
  **L15 CN**: 声明 TableGen def 记录 `err_drv_unsupported_opt_with_suggestion`。
- **L16 EN**: Adds a standalone statement or declaration: `"unsupported option '%0'; did you mean '%1'?">;`.
  **L16 CN**: 添加一条独立语句或声明：`"unsupported option '%0'; did you mean '%1'?">;`。
- **L17 EN**: Declares TableGen def record `err_drv_unsupported_opt_for_target`.
  **L17 CN**: 声明 TableGen def 记录 `err_drv_unsupported_opt_for_target`。
- **L18 EN**: Adds a standalone statement or declaration: `"unsupported option '%0' for target '%1'">;`.
  **L18 CN**: 添加一条独立语句或声明：`"unsupported option '%0' for target '%1'">;`。
- **L19 EN**: Declares TableGen def record `err_drv_unsupported_opt_for_language_mode`.
  **L19 CN**: 声明 TableGen def 记录 `err_drv_unsupported_opt_for_language_mode`。
- **L20 EN**: Adds a standalone statement or declaration: `"unsupported option '%0' for language mode '%1'">;`.
  **L20 CN**: 添加一条独立语句或声明：`"unsupported option '%0' for language mode '%1'">;`。
- **L21 EN**: Declares TableGen def record `err_drv_unsupported_option_argument`.
  **L21 CN**: 声明 TableGen def 记录 `err_drv_unsupported_option_argument`。
- **L22 EN**: Adds a standalone statement or declaration: `"unsupported argument '%1' to option '%0'">;`.
  **L22 CN**: 添加一条独立语句或声明：`"unsupported argument '%1' to option '%0'">;`。
- **L23 EN**: Declares TableGen def record `err_drv_unsupported_option_argument_for_target`.
  **L23 CN**: 声明 TableGen def 记录 `err_drv_unsupported_option_argument_for_target`。
- **L24 EN**: Adds a standalone statement or declaration: `"unsupported argument '%1' to option '%0' for target '%2'">;`.
  **L24 CN**: 添加一条独立语句或声明：`"unsupported argument '%1' to option '%0' for target '%2'">;`。

### Lines 25-48

````tablegen
def err_drv_unknown_stdin_type : Error<
  "-E or -x required when input is from standard input">;
def err_drv_unknown_stdin_type_clang_cl : Error<
  "use /Tc or /Tp to set input type for standard input">;
def err_drv_unknown_language : Error<"language not recognized: '%0'">;
def err_drv_invalid_arch_name : Error<
  "invalid arch name '%0'">;
def err_drv_invalid_riscv_arch_name : Error<
  "invalid arch name '%0', %1">;
def err_drv_invalid_riscv_cpu_name_for_target : Error<
  "cpu '%0' does not support rv%select{32|64}1">;
def warn_drv_invalid_arch_name_with_suggestion : Warning<
  "ignoring invalid /arch: argument '%0'; for %select{64|32}1-bit expected one of %2">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_avr_mcu_not_specified : Warning<
  "no target microcontroller specified, please pass -mmcu=<mcu name>">,
  InGroup<AVRRtlibLinkingQuirks>;
def warn_drv_avr_libc_not_found: Warning<
  "no avr-libc installation can be found on the system, "
  "cannot link standard libraries">,
  InGroup<AVRRtlibLinkingQuirks>;
def warn_drv_avr_family_linking_stdlibs_not_implemented: Warning<
  "support for linking stdlibs for microcontroller '%0' is not implemented">,
  InGroup<AVRRtlibLinkingQuirks>;
````
- **L25 EN**: Declares TableGen def record `err_drv_unknown_stdin_type`.
  **L25 CN**: 声明 TableGen def 记录 `err_drv_unknown_stdin_type`。
- **L26 EN**: Adds a standalone statement or declaration: `"-E or -x required when input is from standard input">;`.
  **L26 CN**: 添加一条独立语句或声明：`"-E or -x required when input is from standard input">;`。
- **L27 EN**: Declares TableGen def record `err_drv_unknown_stdin_type_clang_cl`.
  **L27 CN**: 声明 TableGen def 记录 `err_drv_unknown_stdin_type_clang_cl`。
- **L28 EN**: Adds a standalone statement or declaration: `"use /Tc or /Tp to set input type for standard input">;`.
  **L28 CN**: 添加一条独立语句或声明：`"use /Tc or /Tp to set input type for standard input">;`。
- **L29 EN**: Declares TableGen def record `err_drv_unknown_language`.
  **L29 CN**: 声明 TableGen def 记录 `err_drv_unknown_language`。
- **L30 EN**: Declares TableGen def record `err_drv_invalid_arch_name`.
  **L30 CN**: 声明 TableGen def 记录 `err_drv_invalid_arch_name`。
- **L31 EN**: Adds a standalone statement or declaration: `"invalid arch name '%0'">;`.
  **L31 CN**: 添加一条独立语句或声明：`"invalid arch name '%0'">;`。
- **L32 EN**: Declares TableGen def record `err_drv_invalid_riscv_arch_name`.
  **L32 CN**: 声明 TableGen def 记录 `err_drv_invalid_riscv_arch_name`。
- **L33 EN**: Adds a standalone statement or declaration: `"invalid arch name '%0', %1">;`.
  **L33 CN**: 添加一条独立语句或声明：`"invalid arch name '%0', %1">;`。
- **L34 EN**: Declares TableGen def record `err_drv_invalid_riscv_cpu_name_for_target`.
  **L34 CN**: 声明 TableGen def 记录 `err_drv_invalid_riscv_cpu_name_for_target`。
- **L35 EN**: Adds a standalone statement or declaration: `"cpu '%0' does not support rv%select{32|64}1">;`.
  **L35 CN**: 添加一条独立语句或声明：`"cpu '%0' does not support rv%select{32|64}1">;`。
- **L36 EN**: Declares TableGen def record `warn_drv_invalid_arch_name_with_suggestion`.
  **L36 CN**: 声明 TableGen def 记录 `warn_drv_invalid_arch_name_with_suggestion`。
- **L37 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring invalid /arch: argument '%0'; for %select{64|32}1-bit expected one of %2">,`.
  **L37 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring invalid /arch: argument '%0'; for %select{64|32}1-bit expected one of %2">,`。
- **L38 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L38 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L39 EN**: Declares TableGen def record `warn_drv_avr_mcu_not_specified`.
  **L39 CN**: 声明 TableGen def 记录 `warn_drv_avr_mcu_not_specified`。
- **L40 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"no target microcontroller specified, please pass -mmcu=<mcu name>">,`.
  **L40 CN**: 继续一个多行参数列表、初始化器或聚合项：`"no target microcontroller specified, please pass -mmcu=<mcu name>">,`。
- **L41 EN**: Adds a standalone statement or declaration: `InGroup<AVRRtlibLinkingQuirks>;`.
  **L41 CN**: 添加一条独立语句或声明：`InGroup<AVRRtlibLinkingQuirks>;`。
- **L42 EN**: Declares TableGen def record `warn_drv_avr_libc_not_found`.
  **L42 CN**: 声明 TableGen def 记录 `warn_drv_avr_libc_not_found`。
- **L43 EN**: Continues the surrounding expression or declaration: `"no avr-libc installation can be found on the system, "`.
  **L43 CN**: 继续构造周围的表达式或声明：`"no avr-libc installation can be found on the system, "`。
- **L44 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"cannot link standard libraries">,`.
  **L44 CN**: 继续一个多行参数列表、初始化器或聚合项：`"cannot link standard libraries">,`。
- **L45 EN**: Adds a standalone statement or declaration: `InGroup<AVRRtlibLinkingQuirks>;`.
  **L45 CN**: 添加一条独立语句或声明：`InGroup<AVRRtlibLinkingQuirks>;`。
- **L46 EN**: Declares TableGen def record `warn_drv_avr_family_linking_stdlibs_not_implemented`.
  **L46 CN**: 声明 TableGen def 记录 `warn_drv_avr_family_linking_stdlibs_not_implemented`。
- **L47 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"support for linking stdlibs for microcontroller '%0' is not implemented">,`.
  **L47 CN**: 继续一个多行参数列表、初始化器或聚合项：`"support for linking stdlibs for microcontroller '%0' is not implemented">,`。
- **L48 EN**: Adds a standalone statement or declaration: `InGroup<AVRRtlibLinkingQuirks>;`.
  **L48 CN**: 添加一条独立语句或声明：`InGroup<AVRRtlibLinkingQuirks>;`。

### Lines 49-72

````tablegen
def warn_drv_avr_linker_section_addresses_not_implemented: Warning<
  "support for passing the data section address to the linker for "
  "microcontroller '%0' is not implemented">,
  InGroup<AVRRtlibLinkingQuirks>;
def warn_drv_avr_stdlib_not_linked: Warning<
  "standard library not linked and so no interrupt vector table or "
  "compiler runtime routines will be linked">,
  InGroup<AVRRtlibLinkingQuirks>;
def err_drv_cuda_bad_gpu_arch : Error<"unsupported CUDA gpu architecture: %0">;
def err_drv_offload_bad_gpu_arch : Error<"unsupported %0 gpu architecture: %1">;
def err_drv_offload_missing_gpu_arch : Error<
  "must pass in an explicit %0 gpu architecture to '%1'">;
def err_drv_no_cuda_installation : Error<
  "cannot find CUDA installation; provide its path via '--cuda-path', or pass "
  "'-nocudainc' to build without CUDA includes">;
def err_drv_no_cuda_libdevice : Error<
  "cannot find libdevice for %0; provide path to different CUDA installation "
  "via '--cuda-path', or pass '-nocudalib' to build without linking with "
  "libdevice">;

def err_drv_no_rocm_device_lib : Error<
  "cannot find ROCm device library%select{| for %1| for ABI version %1"
  "%select{|, which requires ROCm %3 or higher}2}0; provide its path via "
  "'--rocm-path' or '--rocm-device-lib-path', or pass '-nogpulib' to build "
````
- **L49 EN**: Declares TableGen def record `warn_drv_avr_linker_section_addresses_not_implemented`.
  **L49 CN**: 声明 TableGen def 记录 `warn_drv_avr_linker_section_addresses_not_implemented`。
- **L50 EN**: Continues the surrounding expression or declaration: `"support for passing the data section address to the linker for "`.
  **L50 CN**: 继续构造周围的表达式或声明：`"support for passing the data section address to the linker for "`。
- **L51 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"microcontroller '%0' is not implemented">,`.
  **L51 CN**: 继续一个多行参数列表、初始化器或聚合项：`"microcontroller '%0' is not implemented">,`。
- **L52 EN**: Adds a standalone statement or declaration: `InGroup<AVRRtlibLinkingQuirks>;`.
  **L52 CN**: 添加一条独立语句或声明：`InGroup<AVRRtlibLinkingQuirks>;`。
- **L53 EN**: Declares TableGen def record `warn_drv_avr_stdlib_not_linked`.
  **L53 CN**: 声明 TableGen def 记录 `warn_drv_avr_stdlib_not_linked`。
- **L54 EN**: Continues the surrounding expression or declaration: `"standard library not linked and so no interrupt vector table or "`.
  **L54 CN**: 继续构造周围的表达式或声明：`"standard library not linked and so no interrupt vector table or "`。
- **L55 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"compiler runtime routines will be linked">,`.
  **L55 CN**: 继续一个多行参数列表、初始化器或聚合项：`"compiler runtime routines will be linked">,`。
- **L56 EN**: Adds a standalone statement or declaration: `InGroup<AVRRtlibLinkingQuirks>;`.
  **L56 CN**: 添加一条独立语句或声明：`InGroup<AVRRtlibLinkingQuirks>;`。
- **L57 EN**: Declares TableGen def record `err_drv_cuda_bad_gpu_arch`.
  **L57 CN**: 声明 TableGen def 记录 `err_drv_cuda_bad_gpu_arch`。
- **L58 EN**: Declares TableGen def record `err_drv_offload_bad_gpu_arch`.
  **L58 CN**: 声明 TableGen def 记录 `err_drv_offload_bad_gpu_arch`。
- **L59 EN**: Declares TableGen def record `err_drv_offload_missing_gpu_arch`.
  **L59 CN**: 声明 TableGen def 记录 `err_drv_offload_missing_gpu_arch`。
- **L60 EN**: Adds a standalone statement or declaration: `"must pass in an explicit %0 gpu architecture to '%1'">;`.
  **L60 CN**: 添加一条独立语句或声明：`"must pass in an explicit %0 gpu architecture to '%1'">;`。
- **L61 EN**: Declares TableGen def record `err_drv_no_cuda_installation`.
  **L61 CN**: 声明 TableGen def 记录 `err_drv_no_cuda_installation`。
- **L62 EN**: Continues the surrounding expression or declaration: `"cannot find CUDA installation; provide its path via '--cuda-path', or pass "`.
  **L62 CN**: 继续构造周围的表达式或声明：`"cannot find CUDA installation; provide its path via '--cuda-path', or pass "`。
- **L63 EN**: Adds a standalone statement or declaration: `"'-nocudainc' to build without CUDA includes">;`.
  **L63 CN**: 添加一条独立语句或声明：`"'-nocudainc' to build without CUDA includes">;`。
- **L64 EN**: Declares TableGen def record `err_drv_no_cuda_libdevice`.
  **L64 CN**: 声明 TableGen def 记录 `err_drv_no_cuda_libdevice`。
- **L65 EN**: Continues the surrounding expression or declaration: `"cannot find libdevice for %0; provide path to different CUDA installation "`.
  **L65 CN**: 继续构造周围的表达式或声明：`"cannot find libdevice for %0; provide path to different CUDA installation "`。
- **L66 EN**: Continues the surrounding expression or declaration: `"via '--cuda-path', or pass '-nocudalib' to build without linking with "`.
  **L66 CN**: 继续构造周围的表达式或声明：`"via '--cuda-path', or pass '-nocudalib' to build without linking with "`。
- **L67 EN**: Adds a standalone statement or declaration: `"libdevice">;`.
  **L67 CN**: 添加一条独立语句或声明：`"libdevice">;`。
- **L68 EN**: Blank line separating nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L69 EN**: Declares TableGen def record `err_drv_no_rocm_device_lib`.
  **L69 CN**: 声明 TableGen def 记录 `err_drv_no_rocm_device_lib`。
- **L70 EN**: Continues the surrounding expression or declaration: `"cannot find ROCm device library%select{| for %1| for ABI version %1"`.
  **L70 CN**: 继续构造周围的表达式或声明：`"cannot find ROCm device library%select{| for %1| for ABI version %1"`。
- **L71 EN**: Continues the surrounding expression or declaration: `"%select{|, which requires ROCm %3 or higher}2}0; provide its path via "`.
  **L71 CN**: 继续构造周围的表达式或声明：`"%select{|, which requires ROCm %3 or higher}2}0; provide its path via "`。
- **L72 EN**: Continues the surrounding expression or declaration: `"'--rocm-path' or '--rocm-device-lib-path', or pass '-nogpulib' to build "`.
  **L72 CN**: 继续构造周围的表达式或声明：`"'--rocm-path' or '--rocm-device-lib-path', or pass '-nogpulib' to build "`。

### Lines 73-96

````tablegen
  "without ROCm device library">;
def err_drv_no_hip_runtime : Error<
  "cannot find HIP runtime; provide its path via '--rocm-path', or pass "
  "'-nogpuinc' to build without HIP runtime">;
def err_drv_no_hipstdpar_lib : Error<
  "cannot find HIP Standard Parallelism Acceleration library; provide it via "
  "'--hipstdpar-path'">;
def err_drv_no_hipstdpar_thrust_lib : Error<
  "cannot find rocThrust, which is required by the HIP Standard Parallelism "
  "Acceleration library; provide it via "
  "'--hipstdpar-thrust-path'">;
def err_drv_no_hipstdpar_prim_lib : Error<
  "cannot find rocPrim, which is required by the HIP Standard Parallelism "
  "Acceleration library; provide it via '--hipstdpar-prim-path'">;

def err_drv_no_hipspv_device_lib : Error<
  "cannot find HIP device library%select{| for %1}0; provide its path via "
  "'--hip-path' or '--hip-device-lib-path', or pass '-nogpulib' to build "
  "without HIP device library">;
def err_drv_hipspv_no_hip_path : Error<
  "'--hip-path' must be specified when offloading to SPIR-V unless '-nogpuinc' "
  "is given">;

// SYCL Windows-specific diagnostics
````
- **L73 EN**: Adds a standalone statement or declaration: `"without ROCm device library">;`.
  **L73 CN**: 添加一条独立语句或声明：`"without ROCm device library">;`。
- **L74 EN**: Declares TableGen def record `err_drv_no_hip_runtime`.
  **L74 CN**: 声明 TableGen def 记录 `err_drv_no_hip_runtime`。
- **L75 EN**: Continues the surrounding expression or declaration: `"cannot find HIP runtime; provide its path via '--rocm-path', or pass "`.
  **L75 CN**: 继续构造周围的表达式或声明：`"cannot find HIP runtime; provide its path via '--rocm-path', or pass "`。
- **L76 EN**: Adds a standalone statement or declaration: `"'-nogpuinc' to build without HIP runtime">;`.
  **L76 CN**: 添加一条独立语句或声明：`"'-nogpuinc' to build without HIP runtime">;`。
- **L77 EN**: Declares TableGen def record `err_drv_no_hipstdpar_lib`.
  **L77 CN**: 声明 TableGen def 记录 `err_drv_no_hipstdpar_lib`。
- **L78 EN**: Continues the surrounding expression or declaration: `"cannot find HIP Standard Parallelism Acceleration library; provide it via "`.
  **L78 CN**: 继续构造周围的表达式或声明：`"cannot find HIP Standard Parallelism Acceleration library; provide it via "`。
- **L79 EN**: Adds a standalone statement or declaration: `"'--hipstdpar-path'">;`.
  **L79 CN**: 添加一条独立语句或声明：`"'--hipstdpar-path'">;`。
- **L80 EN**: Declares TableGen def record `err_drv_no_hipstdpar_thrust_lib`.
  **L80 CN**: 声明 TableGen def 记录 `err_drv_no_hipstdpar_thrust_lib`。
- **L81 EN**: Continues the surrounding expression or declaration: `"cannot find rocThrust, which is required by the HIP Standard Parallelism "`.
  **L81 CN**: 继续构造周围的表达式或声明：`"cannot find rocThrust, which is required by the HIP Standard Parallelism "`。
- **L82 EN**: Continues the surrounding expression or declaration: `"Acceleration library; provide it via "`.
  **L82 CN**: 继续构造周围的表达式或声明：`"Acceleration library; provide it via "`。
- **L83 EN**: Adds a standalone statement or declaration: `"'--hipstdpar-thrust-path'">;`.
  **L83 CN**: 添加一条独立语句或声明：`"'--hipstdpar-thrust-path'">;`。
- **L84 EN**: Declares TableGen def record `err_drv_no_hipstdpar_prim_lib`.
  **L84 CN**: 声明 TableGen def 记录 `err_drv_no_hipstdpar_prim_lib`。
- **L85 EN**: Continues the surrounding expression or declaration: `"cannot find rocPrim, which is required by the HIP Standard Parallelism "`.
  **L85 CN**: 继续构造周围的表达式或声明：`"cannot find rocPrim, which is required by the HIP Standard Parallelism "`。
- **L86 EN**: Adds a standalone statement or declaration: `"Acceleration library; provide it via '--hipstdpar-prim-path'">;`.
  **L86 CN**: 添加一条独立语句或声明：`"Acceleration library; provide it via '--hipstdpar-prim-path'">;`。
- **L87 EN**: Blank line separating nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L88 EN**: Declares TableGen def record `err_drv_no_hipspv_device_lib`.
  **L88 CN**: 声明 TableGen def 记录 `err_drv_no_hipspv_device_lib`。
- **L89 EN**: Continues the surrounding expression or declaration: `"cannot find HIP device library%select{| for %1}0; provide its path via "`.
  **L89 CN**: 继续构造周围的表达式或声明：`"cannot find HIP device library%select{| for %1}0; provide its path via "`。
- **L90 EN**: Continues the surrounding expression or declaration: `"'--hip-path' or '--hip-device-lib-path', or pass '-nogpulib' to build "`.
  **L90 CN**: 继续构造周围的表达式或声明：`"'--hip-path' or '--hip-device-lib-path', or pass '-nogpulib' to build "`。
- **L91 EN**: Adds a standalone statement or declaration: `"without HIP device library">;`.
  **L91 CN**: 添加一条独立语句或声明：`"without HIP device library">;`。
- **L92 EN**: Declares TableGen def record `err_drv_hipspv_no_hip_path`.
  **L92 CN**: 声明 TableGen def 记录 `err_drv_hipspv_no_hip_path`。
- **L93 EN**: Continues the surrounding expression or declaration: `"'--hip-path' must be specified when offloading to SPIR-V unless '-nogpuinc' "`.
  **L93 CN**: 继续构造周围的表达式或声明：`"'--hip-path' must be specified when offloading to SPIR-V unless '-nogpuinc' "`。
- **L94 EN**: Adds a standalone statement or declaration: `"is given">;`.
  **L94 CN**: 添加一条独立语句或声明：`"is given">;`。
- **L95 EN**: Blank line separating nearby declarations or logic blocks.
  **L95 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L96 EN**: Comment explains nearby logic, constraints, or intent: `SYCL Windows-specific diagnostics`.
  **L96 CN**: 注释解释附近代码的逻辑、约束或设计意图：`SYCL Windows-specific diagnostics`。

### Lines 97-120

````tablegen
def err_drv_sycl_requires_dynamic_crt : Error<
  "SYCL requires dynamic C++ runtime (/MD or /MDd); "
  "static C++ runtime (/MT or /MTd) is not supported">;

def err_drv_no_spv_tools : Error<"cannot find SPIR-V Tools binary '%0', which "
                                 "is required for SPIR-V compilation. "
                                 "It can be obtained from your system package "
                                 "manager or from KhronosGroup/SPIRV-Tools "
                                 "on GitHub">;

def err_drv_undetermined_gpu_arch : Error<
  "cannot determine %0 architecture: %1; consider passing it via '%2'; "
  "environment variable CLANG_TOOLCHAIN_PROGRAM_TIMEOUT specifies the tool "
  "timeout (integer secs, <=0 is infinite)">;

def warn_drv_multi_gpu_arch : Warning<
  "multiple %0 architectures are detected: %1; only the first one is used for "
  "'%2'">, InGroup<MultiGPU>;
def err_drv_cuda_version_unsupported : Error<
  "GPU arch %0 is supported by CUDA versions between %1 and %2 (inclusive), "
  "but installation at %3 is %4; use '--cuda-path' to specify a different CUDA "
  "install, pass a different GPU arch with '--cuda-gpu-arch', or pass "
  "'--no-cuda-version-check'">;
def warn_drv_new_cuda_version: Warning<
````
- **L97 EN**: Declares TableGen def record `err_drv_sycl_requires_dynamic_crt`.
  **L97 CN**: 声明 TableGen def 记录 `err_drv_sycl_requires_dynamic_crt`。
- **L98 EN**: Continues logic associated with callable symbol `runtime`.
  **L98 CN**: 继续与可调用符号 `runtime` 相关的逻辑。
- **L99 EN**: Executes a call or declaration centered on `runtime`.
  **L99 CN**: 执行以 `runtime` 为核心的调用或声明。
- **L100 EN**: Blank line separating nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L101 EN**: Declares TableGen def record `err_drv_no_spv_tools`.
  **L101 CN**: 声明 TableGen def 记录 `err_drv_no_spv_tools`。
- **L102 EN**: Continues the surrounding expression or declaration: `"is required for SPIR-V compilation. "`.
  **L102 CN**: 继续构造周围的表达式或声明：`"is required for SPIR-V compilation. "`。
- **L103 EN**: Continues the surrounding expression or declaration: `"It can be obtained from your system package "`.
  **L103 CN**: 继续构造周围的表达式或声明：`"It can be obtained from your system package "`。
- **L104 EN**: Continues the surrounding expression or declaration: `"manager or from KhronosGroup/SPIRV-Tools "`.
  **L104 CN**: 继续构造周围的表达式或声明：`"manager or from KhronosGroup/SPIRV-Tools "`。
- **L105 EN**: Adds a standalone statement or declaration: `"on GitHub">;`.
  **L105 CN**: 添加一条独立语句或声明：`"on GitHub">;`。
- **L106 EN**: Blank line separating nearby declarations or logic blocks.
  **L106 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L107 EN**: Declares TableGen def record `err_drv_undetermined_gpu_arch`.
  **L107 CN**: 声明 TableGen def 记录 `err_drv_undetermined_gpu_arch`。
- **L108 EN**: Continues the surrounding expression or declaration: `"cannot determine %0 architecture: %1; consider passing it via '%2'; "`.
  **L108 CN**: 继续构造周围的表达式或声明：`"cannot determine %0 architecture: %1; consider passing it via '%2'; "`。
- **L109 EN**: Continues the surrounding expression or declaration: `"environment variable CLANG_TOOLCHAIN_PROGRAM_TIMEOUT specifies the tool "`.
  **L109 CN**: 继续构造周围的表达式或声明：`"environment variable CLANG_TOOLCHAIN_PROGRAM_TIMEOUT specifies the tool "`。
- **L110 EN**: Executes a call or declaration centered on `"timeout`.
  **L110 CN**: 执行以 `"timeout` 为核心的调用或声明。
- **L111 EN**: Blank line separating nearby declarations or logic blocks.
  **L111 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L112 EN**: Declares TableGen def record `warn_drv_multi_gpu_arch`.
  **L112 CN**: 声明 TableGen def 记录 `warn_drv_multi_gpu_arch`。
- **L113 EN**: Continues the surrounding expression or declaration: `"multiple %0 architectures are detected: %1; only the first one is used for "`.
  **L113 CN**: 继续构造周围的表达式或声明：`"multiple %0 architectures are detected: %1; only the first one is used for "`。
- **L114 EN**: Adds a standalone statement or declaration: `"'%2'">, InGroup<MultiGPU>;`.
  **L114 CN**: 添加一条独立语句或声明：`"'%2'">, InGroup<MultiGPU>;`。
- **L115 EN**: Declares TableGen def record `err_drv_cuda_version_unsupported`.
  **L115 CN**: 声明 TableGen def 记录 `err_drv_cuda_version_unsupported`。
- **L116 EN**: Continues the surrounding expression or declaration: `"GPU arch %0 is supported by CUDA versions between %1 and %2 (inclusive), "`.
  **L116 CN**: 继续构造周围的表达式或声明：`"GPU arch %0 is supported by CUDA versions between %1 and %2 (inclusive), "`。
- **L117 EN**: Continues the surrounding expression or declaration: `"but installation at %3 is %4; use '--cuda-path' to specify a different CUDA "`.
  **L117 CN**: 继续构造周围的表达式或声明：`"but installation at %3 is %4; use '--cuda-path' to specify a different CUDA "`。
- **L118 EN**: Continues the surrounding expression or declaration: `"install, pass a different GPU arch with '--cuda-gpu-arch', or pass "`.
  **L118 CN**: 继续构造周围的表达式或声明：`"install, pass a different GPU arch with '--cuda-gpu-arch', or pass "`。
- **L119 EN**: Adds a standalone statement or declaration: `"'--no-cuda-version-check'">;`.
  **L119 CN**: 添加一条独立语句或声明：`"'--no-cuda-version-check'">;`。
- **L120 EN**: Declares TableGen def record `warn_drv_new_cuda_version`.
  **L120 CN**: 声明 TableGen def 记录 `warn_drv_new_cuda_version`。

### Lines 121-144

````tablegen
  "CUDA version%0 is newer than the latest%select{| partially}1 supported version %2">,
  InGroup<CudaUnknownVersion>;
def warn_drv_partially_supported_cuda_version: Warning<
  "CUDA version %0 is only partially supported">,
  InGroup<CudaUnknownVersion>;
def err_drv_cuda_host_arch : Error<
  "unsupported architecture '%0' for host compilation">;
def err_drv_mix_cuda_hip : Error<
  "mixed CUDA and HIP compilation is not supported">;
def err_drv_mix_offload : Error<
  "mixed %0 and %1 offloading compilation is not supported">;
def err_drv_bad_target_id : Error<
  "invalid target ID '%0'; format is a processor name followed by an optional "
  "colon-delimited list of features followed by an enable/disable sign (e.g., "
  "'gfx908:sramecc+:xnack-')">;
def err_drv_bad_offload_arch_combo : Error<
  "invalid offload arch combinations: '%0' and '%1' (for a specific processor, "
  "a feature should either exist in all offload archs, or not exist in any "
  "offload archs)">;
def err_drv_unsupported_option_for_offload_arch_req_feature : Error<
  "'%0' option for offload arch '%1' is not currently supported "
  "there. Use it with an offload arch containing '%2' instead">;
def warn_drv_unsupported_option_for_offload_arch_req_feature : Warning<
  "ignoring '%0' option for offload arch '%1' as it is not currently supported "
````
- **L121 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CUDA version%0 is newer than the latest%select{| partially}1 supported version %2">,`.
  **L121 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CUDA version%0 is newer than the latest%select{| partially}1 supported version %2">,`。
- **L122 EN**: Adds a standalone statement or declaration: `InGroup<CudaUnknownVersion>;`.
  **L122 CN**: 添加一条独立语句或声明：`InGroup<CudaUnknownVersion>;`。
- **L123 EN**: Declares TableGen def record `warn_drv_partially_supported_cuda_version`.
  **L123 CN**: 声明 TableGen def 记录 `warn_drv_partially_supported_cuda_version`。
- **L124 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"CUDA version %0 is only partially supported">,`.
  **L124 CN**: 继续一个多行参数列表、初始化器或聚合项：`"CUDA version %0 is only partially supported">,`。
- **L125 EN**: Adds a standalone statement or declaration: `InGroup<CudaUnknownVersion>;`.
  **L125 CN**: 添加一条独立语句或声明：`InGroup<CudaUnknownVersion>;`。
- **L126 EN**: Declares TableGen def record `err_drv_cuda_host_arch`.
  **L126 CN**: 声明 TableGen def 记录 `err_drv_cuda_host_arch`。
- **L127 EN**: Adds a standalone statement or declaration: `"unsupported architecture '%0' for host compilation">;`.
  **L127 CN**: 添加一条独立语句或声明：`"unsupported architecture '%0' for host compilation">;`。
- **L128 EN**: Declares TableGen def record `err_drv_mix_cuda_hip`.
  **L128 CN**: 声明 TableGen def 记录 `err_drv_mix_cuda_hip`。
- **L129 EN**: Adds a standalone statement or declaration: `"mixed CUDA and HIP compilation is not supported">;`.
  **L129 CN**: 添加一条独立语句或声明：`"mixed CUDA and HIP compilation is not supported">;`。
- **L130 EN**: Declares TableGen def record `err_drv_mix_offload`.
  **L130 CN**: 声明 TableGen def 记录 `err_drv_mix_offload`。
- **L131 EN**: Adds a standalone statement or declaration: `"mixed %0 and %1 offloading compilation is not supported">;`.
  **L131 CN**: 添加一条独立语句或声明：`"mixed %0 and %1 offloading compilation is not supported">;`。
- **L132 EN**: Declares TableGen def record `err_drv_bad_target_id`.
  **L132 CN**: 声明 TableGen def 记录 `err_drv_bad_target_id`。
- **L133 EN**: Continues the surrounding expression or declaration: `"invalid target ID '%0'; format is a processor name followed by an optional "`.
  **L133 CN**: 继续构造周围的表达式或声明：`"invalid target ID '%0'; format is a processor name followed by an optional "`。
- **L134 EN**: Continues logic associated with callable symbol `sign`.
  **L134 CN**: 继续与可调用符号 `sign` 相关的逻辑。
- **L135 EN**: Adds a standalone statement or declaration: `"'gfx908:sramecc+:xnack-')">;`.
  **L135 CN**: 添加一条独立语句或声明：`"'gfx908:sramecc+:xnack-')">;`。
- **L136 EN**: Declares TableGen def record `err_drv_bad_offload_arch_combo`.
  **L136 CN**: 声明 TableGen def 记录 `err_drv_bad_offload_arch_combo`。
- **L137 EN**: Continues the surrounding expression or declaration: `"invalid offload arch combinations: '%0' and '%1' (for a specific processor, "`.
  **L137 CN**: 继续构造周围的表达式或声明：`"invalid offload arch combinations: '%0' and '%1' (for a specific processor, "`。
- **L138 EN**: Continues the surrounding expression or declaration: `"a feature should either exist in all offload archs, or not exist in any "`.
  **L138 CN**: 继续构造周围的表达式或声明：`"a feature should either exist in all offload archs, or not exist in any "`。
- **L139 EN**: Adds a standalone statement or declaration: `"offload archs)">;`.
  **L139 CN**: 添加一条独立语句或声明：`"offload archs)">;`。
- **L140 EN**: Declares TableGen def record `err_drv_unsupported_option_for_offload_arch_req_feature`.
  **L140 CN**: 声明 TableGen def 记录 `err_drv_unsupported_option_for_offload_arch_req_feature`。
- **L141 EN**: Continues the surrounding expression or declaration: `"'%0' option for offload arch '%1' is not currently supported "`.
  **L141 CN**: 继续构造周围的表达式或声明：`"'%0' option for offload arch '%1' is not currently supported "`。
- **L142 EN**: Adds a standalone statement or declaration: `"there. Use it with an offload arch containing '%2' instead">;`.
  **L142 CN**: 添加一条独立语句或声明：`"there. Use it with an offload arch containing '%2' instead">;`。
- **L143 EN**: Declares TableGen def record `warn_drv_unsupported_option_for_offload_arch_req_feature`.
  **L143 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_option_for_offload_arch_req_feature`。
- **L144 EN**: Continues the surrounding expression or declaration: `"ignoring '%0' option for offload arch '%1' as it is not currently supported "`.
  **L144 CN**: 继续构造周围的表达式或声明：`"ignoring '%0' option for offload arch '%1' as it is not currently supported "`。

### Lines 145-168

````tablegen
  "there. Use it with an offload arch containing '%2' instead">,
  InGroup<OptionIgnored>;
def warn_drv_unsupported_option_for_target : Warning<
  "ignoring '%0' option as it is not currently supported for target '%1'">,
  InGroup<OptionIgnored>;
def err_drv_unsupported_option_for_target : Error<
  "'%0' option is not currently supported for target '%1'">;
def warn_drv_unsupported_option_part_for_target : Warning<
  "ignoring '%0' in '%1' option as it is not currently supported for target '%2'">,
  InGroup<OptionIgnored>;
def err_drv_unsupported_option_part_for_target : Error<
  "'%0' in '%1' option is not currently supported for target '%2'">;
def warn_drv_invalid_argument_for_flang : Warning<
  "'%0' is not valid for Fortran">,
  InGroup<OptionIgnored>;
def warn_drv_unsupported_option_for_flang : Warning<
  "the argument '%0' is not supported for option '%1'. Mapping to '%1%2'">,
  InGroup<OptionIgnored>;
def warn_drv_unsupported_diag_option_for_flang : Warning<
  "the warning option '-%0' is not supported">,
  InGroup<OptionIgnored>;
def warn_drv_unsupported_option_for_processor : Warning<
  "ignoring '%0' option as it is not currently supported for processor '%1'">,
  InGroup<OptionIgnored>;
````
- **L145 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"there. Use it with an offload arch containing '%2' instead">,`.
  **L145 CN**: 继续一个多行参数列表、初始化器或聚合项：`"there. Use it with an offload arch containing '%2' instead">,`。
- **L146 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L146 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L147 EN**: Declares TableGen def record `warn_drv_unsupported_option_for_target`.
  **L147 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_option_for_target`。
- **L148 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '%0' option as it is not currently supported for target '%1'">,`.
  **L148 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '%0' option as it is not currently supported for target '%1'">,`。
- **L149 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L149 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L150 EN**: Declares TableGen def record `err_drv_unsupported_option_for_target`.
  **L150 CN**: 声明 TableGen def 记录 `err_drv_unsupported_option_for_target`。
- **L151 EN**: Adds a standalone statement or declaration: `"'%0' option is not currently supported for target '%1'">;`.
  **L151 CN**: 添加一条独立语句或声明：`"'%0' option is not currently supported for target '%1'">;`。
- **L152 EN**: Declares TableGen def record `warn_drv_unsupported_option_part_for_target`.
  **L152 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_option_part_for_target`。
- **L153 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '%0' in '%1' option as it is not currently supported for target '%2'">,`.
  **L153 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '%0' in '%1' option as it is not currently supported for target '%2'">,`。
- **L154 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L154 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L155 EN**: Declares TableGen def record `err_drv_unsupported_option_part_for_target`.
  **L155 CN**: 声明 TableGen def 记录 `err_drv_unsupported_option_part_for_target`。
- **L156 EN**: Adds a standalone statement or declaration: `"'%0' in '%1' option is not currently supported for target '%2'">;`.
  **L156 CN**: 添加一条独立语句或声明：`"'%0' in '%1' option is not currently supported for target '%2'">;`。
- **L157 EN**: Declares TableGen def record `warn_drv_invalid_argument_for_flang`.
  **L157 CN**: 声明 TableGen def 记录 `warn_drv_invalid_argument_for_flang`。
- **L158 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' is not valid for Fortran">,`.
  **L158 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' is not valid for Fortran">,`。
- **L159 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L159 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L160 EN**: Declares TableGen def record `warn_drv_unsupported_option_for_flang`.
  **L160 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_option_for_flang`。
- **L161 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the argument '%0' is not supported for option '%1'. Mapping to '%1%2'">,`.
  **L161 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the argument '%0' is not supported for option '%1'. Mapping to '%1%2'">,`。
- **L162 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L162 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L163 EN**: Declares TableGen def record `warn_drv_unsupported_diag_option_for_flang`.
  **L163 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_diag_option_for_flang`。
- **L164 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the warning option '-%0' is not supported">,`.
  **L164 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the warning option '-%0' is not supported">,`。
- **L165 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L165 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L166 EN**: Declares TableGen def record `warn_drv_unsupported_option_for_processor`.
  **L166 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_option_for_processor`。
- **L167 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '%0' option as it is not currently supported for processor '%1'">,`.
  **L167 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '%0' option as it is not currently supported for processor '%1'">,`。
- **L168 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L168 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。

### Lines 169-192

````tablegen
def warn_drv_unsupported_option_for_runtime : Warning<
  "ignoring '%0' option as it is not currently supported for runtime '%1'">,
  InGroup<OptionIgnored>;
def warn_drv_unsupported_openmp_library : Warning<
  "the library '%0=%1' is not supported, OpenMP will not be enabled">,
  InGroup<OptionIgnored>;
def warn_openmp_impl_incomplete : Warning<
  "OpenMP support for version %0 in flang is still incomplete">,
  InGroup<ExperimentalOption>;
def warn_openmp_spec_incomplete : Warning<
  "the specification for OpenMP version %0 is still under development; "
  "the syntax and semantics of new features may be subject to change">,
  InGroup<ExperimentalOption>;
def err_drv_invalid_thread_model_for_target : Error<
  "invalid thread model '%0' in '%1' for this target">;
def err_drv_invalid_linker_name : Error<
  "invalid linker name in argument '%0'">;
def err_drv_invalid_rtlib_name : Error<
  "invalid runtime library name in argument '%0'">;
def err_drv_unsupported_rtlib_for_platform : Error<
  "unsupported runtime library '%0' for platform '%1'">;
def err_drv_invalid_unwindlib_name : Error<
  "invalid unwind library name in argument '%0'">;
def err_drv_unsupported_unwind_for_platform : Error<
````
- **L169 EN**: Declares TableGen def record `warn_drv_unsupported_option_for_runtime`.
  **L169 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_option_for_runtime`。
- **L170 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '%0' option as it is not currently supported for runtime '%1'">,`.
  **L170 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '%0' option as it is not currently supported for runtime '%1'">,`。
- **L171 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L171 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L172 EN**: Declares TableGen def record `warn_drv_unsupported_openmp_library`.
  **L172 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_openmp_library`。
- **L173 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the library '%0=%1' is not supported, OpenMP will not be enabled">,`.
  **L173 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the library '%0=%1' is not supported, OpenMP will not be enabled">,`。
- **L174 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L174 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L175 EN**: Declares TableGen def record `warn_openmp_impl_incomplete`.
  **L175 CN**: 声明 TableGen def 记录 `warn_openmp_impl_incomplete`。
- **L176 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"OpenMP support for version %0 in flang is still incomplete">,`.
  **L176 CN**: 继续一个多行参数列表、初始化器或聚合项：`"OpenMP support for version %0 in flang is still incomplete">,`。
- **L177 EN**: Adds a standalone statement or declaration: `InGroup<ExperimentalOption>;`.
  **L177 CN**: 添加一条独立语句或声明：`InGroup<ExperimentalOption>;`。
- **L178 EN**: Declares TableGen def record `warn_openmp_spec_incomplete`.
  **L178 CN**: 声明 TableGen def 记录 `warn_openmp_spec_incomplete`。
- **L179 EN**: Continues the surrounding expression or declaration: `"the specification for OpenMP version %0 is still under development; "`.
  **L179 CN**: 继续构造周围的表达式或声明：`"the specification for OpenMP version %0 is still under development; "`。
- **L180 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the syntax and semantics of new features may be subject to change">,`.
  **L180 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the syntax and semantics of new features may be subject to change">,`。
- **L181 EN**: Adds a standalone statement or declaration: `InGroup<ExperimentalOption>;`.
  **L181 CN**: 添加一条独立语句或声明：`InGroup<ExperimentalOption>;`。
- **L182 EN**: Declares TableGen def record `err_drv_invalid_thread_model_for_target`.
  **L182 CN**: 声明 TableGen def 记录 `err_drv_invalid_thread_model_for_target`。
- **L183 EN**: Adds a standalone statement or declaration: `"invalid thread model '%0' in '%1' for this target">;`.
  **L183 CN**: 添加一条独立语句或声明：`"invalid thread model '%0' in '%1' for this target">;`。
- **L184 EN**: Declares TableGen def record `err_drv_invalid_linker_name`.
  **L184 CN**: 声明 TableGen def 记录 `err_drv_invalid_linker_name`。
- **L185 EN**: Adds a standalone statement or declaration: `"invalid linker name in argument '%0'">;`.
  **L185 CN**: 添加一条独立语句或声明：`"invalid linker name in argument '%0'">;`。
- **L186 EN**: Declares TableGen def record `err_drv_invalid_rtlib_name`.
  **L186 CN**: 声明 TableGen def 记录 `err_drv_invalid_rtlib_name`。
- **L187 EN**: Adds a standalone statement or declaration: `"invalid runtime library name in argument '%0'">;`.
  **L187 CN**: 添加一条独立语句或声明：`"invalid runtime library name in argument '%0'">;`。
- **L188 EN**: Declares TableGen def record `err_drv_unsupported_rtlib_for_platform`.
  **L188 CN**: 声明 TableGen def 记录 `err_drv_unsupported_rtlib_for_platform`。
- **L189 EN**: Adds a standalone statement or declaration: `"unsupported runtime library '%0' for platform '%1'">;`.
  **L189 CN**: 添加一条独立语句或声明：`"unsupported runtime library '%0' for platform '%1'">;`。
- **L190 EN**: Declares TableGen def record `err_drv_invalid_unwindlib_name`.
  **L190 CN**: 声明 TableGen def 记录 `err_drv_invalid_unwindlib_name`。
- **L191 EN**: Adds a standalone statement or declaration: `"invalid unwind library name in argument '%0'">;`.
  **L191 CN**: 添加一条独立语句或声明：`"invalid unwind library name in argument '%0'">;`。
- **L192 EN**: Declares TableGen def record `err_drv_unsupported_unwind_for_platform`.
  **L192 CN**: 声明 TableGen def 记录 `err_drv_unsupported_unwind_for_platform`。

### Lines 193-216

````tablegen
  "unsupported unwind library '%0' for platform '%1'">;
def err_drv_incompatible_unwindlib : Error<
  "--rtlib=libgcc requires --unwindlib=libgcc">;
def err_drv_incompatible_options : Error<
  "the combination of '%0' and '%1' is incompatible">;
def err_drv_invalid_cstdlib_name : Error<
  "invalid C library name in argument '%0'">;
def err_drv_invalid_stdlib_name : Error<
  "invalid library name in argument '%0'">;
def err_drv_invalid_output_with_multiple_archs : Error<
  "cannot use '%0' output with multiple -arch options">;
def err_drv_no_input_files : Error<"no input files">;
def err_drv_output_argument_with_multiple_files : Error<
  "cannot specify -o when generating multiple output files">;
def err_drv_out_file_argument_with_multiple_sources : Error<
  "cannot specify '%0%1' when compiling multiple source files">;
def err_no_external_assembler : Error<
  "there is no external assembler that can be used on this platform">;
def err_drv_unable_to_remove_file : Error<
  "unable to remove file: %0">;
def err_drv_unable_to_set_working_directory : Error <
  "unable to set working directory: %0">;
def err_drv_command_failure : Error<
  "unable to execute command: %0">;
````
- **L193 EN**: Adds a standalone statement or declaration: `"unsupported unwind library '%0' for platform '%1'">;`.
  **L193 CN**: 添加一条独立语句或声明：`"unsupported unwind library '%0' for platform '%1'">;`。
- **L194 EN**: Declares TableGen def record `err_drv_incompatible_unwindlib`.
  **L194 CN**: 声明 TableGen def 记录 `err_drv_incompatible_unwindlib`。
- **L195 EN**: Adds a standalone statement or declaration: `"--rtlib=libgcc requires --unwindlib=libgcc">;`.
  **L195 CN**: 添加一条独立语句或声明：`"--rtlib=libgcc requires --unwindlib=libgcc">;`。
- **L196 EN**: Declares TableGen def record `err_drv_incompatible_options`.
  **L196 CN**: 声明 TableGen def 记录 `err_drv_incompatible_options`。
- **L197 EN**: Adds a standalone statement or declaration: `"the combination of '%0' and '%1' is incompatible">;`.
  **L197 CN**: 添加一条独立语句或声明：`"the combination of '%0' and '%1' is incompatible">;`。
- **L198 EN**: Declares TableGen def record `err_drv_invalid_cstdlib_name`.
  **L198 CN**: 声明 TableGen def 记录 `err_drv_invalid_cstdlib_name`。
- **L199 EN**: Adds a standalone statement or declaration: `"invalid C library name in argument '%0'">;`.
  **L199 CN**: 添加一条独立语句或声明：`"invalid C library name in argument '%0'">;`。
- **L200 EN**: Declares TableGen def record `err_drv_invalid_stdlib_name`.
  **L200 CN**: 声明 TableGen def 记录 `err_drv_invalid_stdlib_name`。
- **L201 EN**: Adds a standalone statement or declaration: `"invalid library name in argument '%0'">;`.
  **L201 CN**: 添加一条独立语句或声明：`"invalid library name in argument '%0'">;`。
- **L202 EN**: Declares TableGen def record `err_drv_invalid_output_with_multiple_archs`.
  **L202 CN**: 声明 TableGen def 记录 `err_drv_invalid_output_with_multiple_archs`。
- **L203 EN**: Adds a standalone statement or declaration: `"cannot use '%0' output with multiple -arch options">;`.
  **L203 CN**: 添加一条独立语句或声明：`"cannot use '%0' output with multiple -arch options">;`。
- **L204 EN**: Declares TableGen def record `err_drv_no_input_files`.
  **L204 CN**: 声明 TableGen def 记录 `err_drv_no_input_files`。
- **L205 EN**: Declares TableGen def record `err_drv_output_argument_with_multiple_files`.
  **L205 CN**: 声明 TableGen def 记录 `err_drv_output_argument_with_multiple_files`。
- **L206 EN**: Adds a standalone statement or declaration: `"cannot specify -o when generating multiple output files">;`.
  **L206 CN**: 添加一条独立语句或声明：`"cannot specify -o when generating multiple output files">;`。
- **L207 EN**: Declares TableGen def record `err_drv_out_file_argument_with_multiple_sources`.
  **L207 CN**: 声明 TableGen def 记录 `err_drv_out_file_argument_with_multiple_sources`。
- **L208 EN**: Adds a standalone statement or declaration: `"cannot specify '%0%1' when compiling multiple source files">;`.
  **L208 CN**: 添加一条独立语句或声明：`"cannot specify '%0%1' when compiling multiple source files">;`。
- **L209 EN**: Declares TableGen def record `err_no_external_assembler`.
  **L209 CN**: 声明 TableGen def 记录 `err_no_external_assembler`。
- **L210 EN**: Adds a standalone statement or declaration: `"there is no external assembler that can be used on this platform">;`.
  **L210 CN**: 添加一条独立语句或声明：`"there is no external assembler that can be used on this platform">;`。
- **L211 EN**: Declares TableGen def record `err_drv_unable_to_remove_file`.
  **L211 CN**: 声明 TableGen def 记录 `err_drv_unable_to_remove_file`。
- **L212 EN**: Adds a standalone statement or declaration: `"unable to remove file: %0">;`.
  **L212 CN**: 添加一条独立语句或声明：`"unable to remove file: %0">;`。
- **L213 EN**: Declares TableGen def record `err_drv_unable_to_set_working_directory`.
  **L213 CN**: 声明 TableGen def 记录 `err_drv_unable_to_set_working_directory`。
- **L214 EN**: Adds a standalone statement or declaration: `"unable to set working directory: %0">;`.
  **L214 CN**: 添加一条独立语句或声明：`"unable to set working directory: %0">;`。
- **L215 EN**: Declares TableGen def record `err_drv_command_failure`.
  **L215 CN**: 声明 TableGen def 记录 `err_drv_command_failure`。
- **L216 EN**: Adds a standalone statement or declaration: `"unable to execute command: %0">;`.
  **L216 CN**: 添加一条独立语句或声明：`"unable to execute command: %0">;`。

### Lines 217-240

````tablegen
def err_drv_invalid_darwin_version : Error<
  "invalid Darwin version number: %0">;
def err_drv_invalid_diagnotics_hotness_threshold : Error<
  "invalid argument in '%0', only integer or 'auto' is supported">;
def err_drv_invalid_diagnotics_misexpect_tolerance : Error<
  "invalid argument in '%0', only integers are supported">;
def err_drv_missing_argument : Error<
  "argument to '%0' is missing (expected %1 value%s1)">;
def err_drv_invalid_Xarch_argument_with_args : Error<
  "invalid Xarch argument: '%0', options requiring arguments are unsupported">;
def err_drv_Xopenmp_target_missing_triple : Error<
  "cannot deduce implicit triple value for -Xopenmp-target, specify triple using -Xopenmp-target=<triple>">;
def err_drv_invalid_Xopenmp_target_with_args : Error<
  "invalid -Xopenmp-target argument: '%0', options requiring arguments are unsupported">;
def err_drv_argument_only_allowed_with : Error<
  "invalid argument '%0' only allowed with '%1'">;
def err_drv_opt_unsupported_input_type : Error<
  "'%0' invalid for input of type %1">;
def err_drv_amdgpu_ieee_without_no_honor_nans : Error<
  "invalid argument '-mno-amdgpu-ieee' only allowed with relaxed NaN handling">;
def err_drv_argument_not_allowed_with : Error<
  "invalid argument '%0' not allowed with '%1'">;
def warn_drv_argument_not_allowed_with : Warning<
  "invalid argument '%0' not allowed with '%1'">,
````
- **L217 EN**: Declares TableGen def record `err_drv_invalid_darwin_version`.
  **L217 CN**: 声明 TableGen def 记录 `err_drv_invalid_darwin_version`。
- **L218 EN**: Adds a standalone statement or declaration: `"invalid Darwin version number: %0">;`.
  **L218 CN**: 添加一条独立语句或声明：`"invalid Darwin version number: %0">;`。
- **L219 EN**: Declares TableGen def record `err_drv_invalid_diagnotics_hotness_threshold`.
  **L219 CN**: 声明 TableGen def 记录 `err_drv_invalid_diagnotics_hotness_threshold`。
- **L220 EN**: Adds a standalone statement or declaration: `"invalid argument in '%0', only integer or 'auto' is supported">;`.
  **L220 CN**: 添加一条独立语句或声明：`"invalid argument in '%0', only integer or 'auto' is supported">;`。
- **L221 EN**: Declares TableGen def record `err_drv_invalid_diagnotics_misexpect_tolerance`.
  **L221 CN**: 声明 TableGen def 记录 `err_drv_invalid_diagnotics_misexpect_tolerance`。
- **L222 EN**: Adds a standalone statement or declaration: `"invalid argument in '%0', only integers are supported">;`.
  **L222 CN**: 添加一条独立语句或声明：`"invalid argument in '%0', only integers are supported">;`。
- **L223 EN**: Declares TableGen def record `err_drv_missing_argument`.
  **L223 CN**: 声明 TableGen def 记录 `err_drv_missing_argument`。
- **L224 EN**: Executes a call or declaration centered on `missing`.
  **L224 CN**: 执行以 `missing` 为核心的调用或声明。
- **L225 EN**: Declares TableGen def record `err_drv_invalid_Xarch_argument_with_args`.
  **L225 CN**: 声明 TableGen def 记录 `err_drv_invalid_Xarch_argument_with_args`。
- **L226 EN**: Adds a standalone statement or declaration: `"invalid Xarch argument: '%0', options requiring arguments are unsupported">;`.
  **L226 CN**: 添加一条独立语句或声明：`"invalid Xarch argument: '%0', options requiring arguments are unsupported">;`。
- **L227 EN**: Declares TableGen def record `err_drv_Xopenmp_target_missing_triple`.
  **L227 CN**: 声明 TableGen def 记录 `err_drv_Xopenmp_target_missing_triple`。
- **L228 EN**: Adds a standalone statement or declaration: `"cannot deduce implicit triple value for -Xopenmp-target, specify triple using -Xopenmp-target=<triple>">;`.
  **L228 CN**: 添加一条独立语句或声明：`"cannot deduce implicit triple value for -Xopenmp-target, specify triple using -Xopenmp-target=<triple>">;`。
- **L229 EN**: Declares TableGen def record `err_drv_invalid_Xopenmp_target_with_args`.
  **L229 CN**: 声明 TableGen def 记录 `err_drv_invalid_Xopenmp_target_with_args`。
- **L230 EN**: Adds a standalone statement or declaration: `"invalid -Xopenmp-target argument: '%0', options requiring arguments are unsupported">;`.
  **L230 CN**: 添加一条独立语句或声明：`"invalid -Xopenmp-target argument: '%0', options requiring arguments are unsupported">;`。
- **L231 EN**: Declares TableGen def record `err_drv_argument_only_allowed_with`.
  **L231 CN**: 声明 TableGen def 记录 `err_drv_argument_only_allowed_with`。
- **L232 EN**: Adds a standalone statement or declaration: `"invalid argument '%0' only allowed with '%1'">;`.
  **L232 CN**: 添加一条独立语句或声明：`"invalid argument '%0' only allowed with '%1'">;`。
- **L233 EN**: Declares TableGen def record `err_drv_opt_unsupported_input_type`.
  **L233 CN**: 声明 TableGen def 记录 `err_drv_opt_unsupported_input_type`。
- **L234 EN**: Adds a standalone statement or declaration: `"'%0' invalid for input of type %1">;`.
  **L234 CN**: 添加一条独立语句或声明：`"'%0' invalid for input of type %1">;`。
- **L235 EN**: Declares TableGen def record `err_drv_amdgpu_ieee_without_no_honor_nans`.
  **L235 CN**: 声明 TableGen def 记录 `err_drv_amdgpu_ieee_without_no_honor_nans`。
- **L236 EN**: Adds a standalone statement or declaration: `"invalid argument '-mno-amdgpu-ieee' only allowed with relaxed NaN handling">;`.
  **L236 CN**: 添加一条独立语句或声明：`"invalid argument '-mno-amdgpu-ieee' only allowed with relaxed NaN handling">;`。
- **L237 EN**: Declares TableGen def record `err_drv_argument_not_allowed_with`.
  **L237 CN**: 声明 TableGen def 记录 `err_drv_argument_not_allowed_with`。
- **L238 EN**: Adds a standalone statement or declaration: `"invalid argument '%0' not allowed with '%1'">;`.
  **L238 CN**: 添加一条独立语句或声明：`"invalid argument '%0' not allowed with '%1'">;`。
- **L239 EN**: Declares TableGen def record `warn_drv_argument_not_allowed_with`.
  **L239 CN**: 声明 TableGen def 记录 `warn_drv_argument_not_allowed_with`。
- **L240 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"invalid argument '%0' not allowed with '%1'">,`.
  **L240 CN**: 继续一个多行参数列表、初始化器或聚合项：`"invalid argument '%0' not allowed with '%1'">,`。

### Lines 241-264

````tablegen
  InGroup<OptionIgnored>;
def err_drv_sycl_requires_cxx17 : Error<
  "SYCL requires C++17 or later; '%0' is not supported">;
def err_drv_cannot_open_randomize_layout_seed_file : Error<
  "cannot read randomize layout seed file '%0'">;
def err_drv_invalid_version_number : Error<
  "invalid version number in '%0'">;
def err_drv_invalid_version_number_inferred
    : Error<"invalid version number '%0' inferred from '%1'">;
def err_drv_missing_version_number : Error<"missing version number in '%0'">;
def err_drv_kcfi_arity_unsupported_target : Error<
  "target '%0' is unsupported by -fsanitize-kcfi-arity">;
def err_drv_no_linker_llvm_support : Error<
  "'%0': unable to pass LLVM bit-code files to linker">;
def err_drv_no_ast_support : Error<
  "'%0': unable to use AST files with this tool">;
def err_drv_no_module_support : Error<
  "'%0': unable to use module files with this tool">;
def err_drv_clang_unsupported : Error<
  "the clang compiler does not support '%0'">;
def err_drv_clang_unsupported_opt_cxx_darwin_i386 : Error<
  "the clang compiler does not support '%0' for C++ on Darwin/i386">;
def err_drv_clang_unsupported_opt_pg_darwin: Error<
  "the clang compiler does not support -pg option on %select{Darwin|versions of OS X 10.9 and later}0">;
````
- **L241 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L241 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L242 EN**: Declares TableGen def record `err_drv_sycl_requires_cxx17`.
  **L242 CN**: 声明 TableGen def 记录 `err_drv_sycl_requires_cxx17`。
- **L243 EN**: Adds a standalone statement or declaration: `"SYCL requires C++17 or later; '%0' is not supported">;`.
  **L243 CN**: 添加一条独立语句或声明：`"SYCL requires C++17 or later; '%0' is not supported">;`。
- **L244 EN**: Declares TableGen def record `err_drv_cannot_open_randomize_layout_seed_file`.
  **L244 CN**: 声明 TableGen def 记录 `err_drv_cannot_open_randomize_layout_seed_file`。
- **L245 EN**: Adds a standalone statement or declaration: `"cannot read randomize layout seed file '%0'">;`.
  **L245 CN**: 添加一条独立语句或声明：`"cannot read randomize layout seed file '%0'">;`。
- **L246 EN**: Declares TableGen def record `err_drv_invalid_version_number`.
  **L246 CN**: 声明 TableGen def 记录 `err_drv_invalid_version_number`。
- **L247 EN**: Adds a standalone statement or declaration: `"invalid version number in '%0'">;`.
  **L247 CN**: 添加一条独立语句或声明：`"invalid version number in '%0'">;`。
- **L248 EN**: Declares TableGen def record `err_drv_invalid_version_number_inferred`.
  **L248 CN**: 声明 TableGen def 记录 `err_drv_invalid_version_number_inferred`。
- **L249 EN**: Adds a standalone statement or declaration: `: Error<"invalid version number '%0' inferred from '%1'">;`.
  **L249 CN**: 添加一条独立语句或声明：`: Error<"invalid version number '%0' inferred from '%1'">;`。
- **L250 EN**: Declares TableGen def record `err_drv_missing_version_number`.
  **L250 CN**: 声明 TableGen def 记录 `err_drv_missing_version_number`。
- **L251 EN**: Declares TableGen def record `err_drv_kcfi_arity_unsupported_target`.
  **L251 CN**: 声明 TableGen def 记录 `err_drv_kcfi_arity_unsupported_target`。
- **L252 EN**: Adds a standalone statement or declaration: `"target '%0' is unsupported by -fsanitize-kcfi-arity">;`.
  **L252 CN**: 添加一条独立语句或声明：`"target '%0' is unsupported by -fsanitize-kcfi-arity">;`。
- **L253 EN**: Declares TableGen def record `err_drv_no_linker_llvm_support`.
  **L253 CN**: 声明 TableGen def 记录 `err_drv_no_linker_llvm_support`。
- **L254 EN**: Adds a standalone statement or declaration: `"'%0': unable to pass LLVM bit-code files to linker">;`.
  **L254 CN**: 添加一条独立语句或声明：`"'%0': unable to pass LLVM bit-code files to linker">;`。
- **L255 EN**: Declares TableGen def record `err_drv_no_ast_support`.
  **L255 CN**: 声明 TableGen def 记录 `err_drv_no_ast_support`。
- **L256 EN**: Adds a standalone statement or declaration: `"'%0': unable to use AST files with this tool">;`.
  **L256 CN**: 添加一条独立语句或声明：`"'%0': unable to use AST files with this tool">;`。
- **L257 EN**: Declares TableGen def record `err_drv_no_module_support`.
  **L257 CN**: 声明 TableGen def 记录 `err_drv_no_module_support`。
- **L258 EN**: Adds a standalone statement or declaration: `"'%0': unable to use module files with this tool">;`.
  **L258 CN**: 添加一条独立语句或声明：`"'%0': unable to use module files with this tool">;`。
- **L259 EN**: Declares TableGen def record `err_drv_clang_unsupported`.
  **L259 CN**: 声明 TableGen def 记录 `err_drv_clang_unsupported`。
- **L260 EN**: Adds a standalone statement or declaration: `"the clang compiler does not support '%0'">;`.
  **L260 CN**: 添加一条独立语句或声明：`"the clang compiler does not support '%0'">;`。
- **L261 EN**: Declares TableGen def record `err_drv_clang_unsupported_opt_cxx_darwin_i386`.
  **L261 CN**: 声明 TableGen def 记录 `err_drv_clang_unsupported_opt_cxx_darwin_i386`。
- **L262 EN**: Adds a standalone statement or declaration: `"the clang compiler does not support '%0' for C++ on Darwin/i386">;`.
  **L262 CN**: 添加一条独立语句或声明：`"the clang compiler does not support '%0' for C++ on Darwin/i386">;`。
- **L263 EN**: Declares TableGen def record `err_drv_clang_unsupported_opt_pg_darwin`.
  **L263 CN**: 声明 TableGen def 记录 `err_drv_clang_unsupported_opt_pg_darwin`。
- **L264 EN**: Adds a standalone statement or declaration: `"the clang compiler does not support -pg option on %select{Darwin|versions of OS X 10.9 and later}0">;`.
  **L264 CN**: 添加一条独立语句或声明：`"the clang compiler does not support -pg option on %select{Darwin|versions of OS X 10.9 and later}0">;`。

### Lines 265-288

````tablegen
def err_drv_clang_unsupported_opt_faltivec : Error<
  "the clang compiler does not support '%0', %1">;
def err_drv_command_failed : Error<
  "%0 command failed with exit code %1 (use -v to see invocation)">;
def err_drv_compilationdatabase : Error<
  "compilation database '%0' could not be opened: %1">;
def err_drv_command_signalled : Error<
  "%0 command failed due to signal (use -v to see invocation)">;
def err_drv_force_crash : Error<
  "failing because %select{environment variable 'FORCE_CLANG_DIAGNOSTICS_CRASH' is set|'-gen-reproducer' is used}0">;
def err_drv_invalid_mfloat_abi : Error<
  "invalid float ABI '%0'">;
def err_drv_invalid_mtp : Error<
  "invalid thread pointer reading mode '%0'">;
def err_drv_missing_arg_mtp : Error<
  "missing argument to '%0'">;
def warn_drv_missing_plugin_name : Warning<
  "missing plugin name in %0">,
  InGroup<InvalidCommandLineArgument>;
def warn_drv_missing_plugin_arg : Warning<
  "missing plugin argument for plugin %0 in %1">,
  InGroup<InvalidCommandLineArgument>;
def err_drv_invalid_argument_to_option : Error<
  "invalid argument '%0' to -%1">;
````
- **L265 EN**: Declares TableGen def record `err_drv_clang_unsupported_opt_faltivec`.
  **L265 CN**: 声明 TableGen def 记录 `err_drv_clang_unsupported_opt_faltivec`。
- **L266 EN**: Adds a standalone statement or declaration: `"the clang compiler does not support '%0', %1">;`.
  **L266 CN**: 添加一条独立语句或声明：`"the clang compiler does not support '%0', %1">;`。
- **L267 EN**: Declares TableGen def record `err_drv_command_failed`.
  **L267 CN**: 声明 TableGen def 记录 `err_drv_command_failed`。
- **L268 EN**: Executes a call or declaration centered on `%1`.
  **L268 CN**: 执行以 `%1` 为核心的调用或声明。
- **L269 EN**: Declares TableGen def record `err_drv_compilationdatabase`.
  **L269 CN**: 声明 TableGen def 记录 `err_drv_compilationdatabase`。
- **L270 EN**: Adds a standalone statement or declaration: `"compilation database '%0' could not be opened: %1">;`.
  **L270 CN**: 添加一条独立语句或声明：`"compilation database '%0' could not be opened: %1">;`。
- **L271 EN**: Declares TableGen def record `err_drv_command_signalled`.
  **L271 CN**: 声明 TableGen def 记录 `err_drv_command_signalled`。
- **L272 EN**: Executes a call or declaration centered on `signal`.
  **L272 CN**: 执行以 `signal` 为核心的调用或声明。
- **L273 EN**: Declares TableGen def record `err_drv_force_crash`.
  **L273 CN**: 声明 TableGen def 记录 `err_drv_force_crash`。
- **L274 EN**: Adds a standalone statement or declaration: `"failing because %select{environment variable 'FORCE_CLANG_DIAGNOSTICS_CRASH' is set|'-gen-reproducer' is used}0">;`.
  **L274 CN**: 添加一条独立语句或声明：`"failing because %select{environment variable 'FORCE_CLANG_DIAGNOSTICS_CRASH' is set|'-gen-reproducer' is used}0">;`。
- **L275 EN**: Declares TableGen def record `err_drv_invalid_mfloat_abi`.
  **L275 CN**: 声明 TableGen def 记录 `err_drv_invalid_mfloat_abi`。
- **L276 EN**: Adds a standalone statement or declaration: `"invalid float ABI '%0'">;`.
  **L276 CN**: 添加一条独立语句或声明：`"invalid float ABI '%0'">;`。
- **L277 EN**: Declares TableGen def record `err_drv_invalid_mtp`.
  **L277 CN**: 声明 TableGen def 记录 `err_drv_invalid_mtp`。
- **L278 EN**: Adds a standalone statement or declaration: `"invalid thread pointer reading mode '%0'">;`.
  **L278 CN**: 添加一条独立语句或声明：`"invalid thread pointer reading mode '%0'">;`。
- **L279 EN**: Declares TableGen def record `err_drv_missing_arg_mtp`.
  **L279 CN**: 声明 TableGen def 记录 `err_drv_missing_arg_mtp`。
- **L280 EN**: Adds a standalone statement or declaration: `"missing argument to '%0'">;`.
  **L280 CN**: 添加一条独立语句或声明：`"missing argument to '%0'">;`。
- **L281 EN**: Declares TableGen def record `warn_drv_missing_plugin_name`.
  **L281 CN**: 声明 TableGen def 记录 `warn_drv_missing_plugin_name`。
- **L282 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"missing plugin name in %0">,`.
  **L282 CN**: 继续一个多行参数列表、初始化器或聚合项：`"missing plugin name in %0">,`。
- **L283 EN**: Adds a standalone statement or declaration: `InGroup<InvalidCommandLineArgument>;`.
  **L283 CN**: 添加一条独立语句或声明：`InGroup<InvalidCommandLineArgument>;`。
- **L284 EN**: Declares TableGen def record `warn_drv_missing_plugin_arg`.
  **L284 CN**: 声明 TableGen def 记录 `warn_drv_missing_plugin_arg`。
- **L285 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"missing plugin argument for plugin %0 in %1">,`.
  **L285 CN**: 继续一个多行参数列表、初始化器或聚合项：`"missing plugin argument for plugin %0 in %1">,`。
- **L286 EN**: Adds a standalone statement or declaration: `InGroup<InvalidCommandLineArgument>;`.
  **L286 CN**: 添加一条独立语句或声明：`InGroup<InvalidCommandLineArgument>;`。
- **L287 EN**: Declares TableGen def record `err_drv_invalid_argument_to_option`.
  **L287 CN**: 声明 TableGen def 记录 `err_drv_invalid_argument_to_option`。
- **L288 EN**: Adds a standalone statement or declaration: `"invalid argument '%0' to -%1">;`.
  **L288 CN**: 添加一条独立语句或声明：`"invalid argument '%0' to -%1">;`。

### Lines 289-312

````tablegen
def err_drv_missing_sanitizer_ignorelist : Error<
  "missing sanitizer ignorelist: '%0'">;
def err_drv_malformed_sanitizer_ignorelist : Error<
  "malformed sanitizer ignorelist: '%0'">;
def err_drv_malformed_sanitizer_coverage_allowlist : Error<
  "malformed sanitizer coverage allowlist: '%0'">;
def err_drv_malformed_sanitizer_coverage_ignorelist : Error<
  "malformed sanitizer coverage ignorelist: '%0'">;
def err_drv_malformed_sanitizer_metadata_ignorelist : Error<
  "malformed sanitizer metadata ignorelist: '%0'">;
def err_drv_unsupported_static_sanitizer_darwin : Error<
  "static %0 runtime is not supported on darwin">;
def err_drv_duplicate_config : Error<
  "no more than one option '--config' is allowed">;
def err_drv_cannot_open_config_file : Error<
  "configuration file '%0' cannot be opened: %1">;
def err_drv_config_file_not_found : Error<
  "configuration file '%0' cannot be found">;
def note_drv_config_file_searched_in : Note<
  "was searched for in the directory: %0">;
def err_drv_cannot_read_config_file : Error<
  "cannot read configuration file '%0': %1">;
def err_drv_arg_requires_bitcode_input: Error<
  "option '%0' requires input to be LLVM bitcode">;
````
- **L289 EN**: Declares TableGen def record `err_drv_missing_sanitizer_ignorelist`.
  **L289 CN**: 声明 TableGen def 记录 `err_drv_missing_sanitizer_ignorelist`。
- **L290 EN**: Adds a standalone statement or declaration: `"missing sanitizer ignorelist: '%0'">;`.
  **L290 CN**: 添加一条独立语句或声明：`"missing sanitizer ignorelist: '%0'">;`。
- **L291 EN**: Declares TableGen def record `err_drv_malformed_sanitizer_ignorelist`.
  **L291 CN**: 声明 TableGen def 记录 `err_drv_malformed_sanitizer_ignorelist`。
- **L292 EN**: Adds a standalone statement or declaration: `"malformed sanitizer ignorelist: '%0'">;`.
  **L292 CN**: 添加一条独立语句或声明：`"malformed sanitizer ignorelist: '%0'">;`。
- **L293 EN**: Declares TableGen def record `err_drv_malformed_sanitizer_coverage_allowlist`.
  **L293 CN**: 声明 TableGen def 记录 `err_drv_malformed_sanitizer_coverage_allowlist`。
- **L294 EN**: Adds a standalone statement or declaration: `"malformed sanitizer coverage allowlist: '%0'">;`.
  **L294 CN**: 添加一条独立语句或声明：`"malformed sanitizer coverage allowlist: '%0'">;`。
- **L295 EN**: Declares TableGen def record `err_drv_malformed_sanitizer_coverage_ignorelist`.
  **L295 CN**: 声明 TableGen def 记录 `err_drv_malformed_sanitizer_coverage_ignorelist`。
- **L296 EN**: Adds a standalone statement or declaration: `"malformed sanitizer coverage ignorelist: '%0'">;`.
  **L296 CN**: 添加一条独立语句或声明：`"malformed sanitizer coverage ignorelist: '%0'">;`。
- **L297 EN**: Declares TableGen def record `err_drv_malformed_sanitizer_metadata_ignorelist`.
  **L297 CN**: 声明 TableGen def 记录 `err_drv_malformed_sanitizer_metadata_ignorelist`。
- **L298 EN**: Adds a standalone statement or declaration: `"malformed sanitizer metadata ignorelist: '%0'">;`.
  **L298 CN**: 添加一条独立语句或声明：`"malformed sanitizer metadata ignorelist: '%0'">;`。
- **L299 EN**: Declares TableGen def record `err_drv_unsupported_static_sanitizer_darwin`.
  **L299 CN**: 声明 TableGen def 记录 `err_drv_unsupported_static_sanitizer_darwin`。
- **L300 EN**: Adds a standalone statement or declaration: `"static %0 runtime is not supported on darwin">;`.
  **L300 CN**: 添加一条独立语句或声明：`"static %0 runtime is not supported on darwin">;`。
- **L301 EN**: Declares TableGen def record `err_drv_duplicate_config`.
  **L301 CN**: 声明 TableGen def 记录 `err_drv_duplicate_config`。
- **L302 EN**: Adds a standalone statement or declaration: `"no more than one option '--config' is allowed">;`.
  **L302 CN**: 添加一条独立语句或声明：`"no more than one option '--config' is allowed">;`。
- **L303 EN**: Declares TableGen def record `err_drv_cannot_open_config_file`.
  **L303 CN**: 声明 TableGen def 记录 `err_drv_cannot_open_config_file`。
- **L304 EN**: Adds a standalone statement or declaration: `"configuration file '%0' cannot be opened: %1">;`.
  **L304 CN**: 添加一条独立语句或声明：`"configuration file '%0' cannot be opened: %1">;`。
- **L305 EN**: Declares TableGen def record `err_drv_config_file_not_found`.
  **L305 CN**: 声明 TableGen def 记录 `err_drv_config_file_not_found`。
- **L306 EN**: Adds a standalone statement or declaration: `"configuration file '%0' cannot be found">;`.
  **L306 CN**: 添加一条独立语句或声明：`"configuration file '%0' cannot be found">;`。
- **L307 EN**: Declares TableGen def record `note_drv_config_file_searched_in`.
  **L307 CN**: 声明 TableGen def 记录 `note_drv_config_file_searched_in`。
- **L308 EN**: Adds a standalone statement or declaration: `"was searched for in the directory: %0">;`.
  **L308 CN**: 添加一条独立语句或声明：`"was searched for in the directory: %0">;`。
- **L309 EN**: Declares TableGen def record `err_drv_cannot_read_config_file`.
  **L309 CN**: 声明 TableGen def 记录 `err_drv_cannot_read_config_file`。
- **L310 EN**: Adds a standalone statement or declaration: `"cannot read configuration file '%0': %1">;`.
  **L310 CN**: 添加一条独立语句或声明：`"cannot read configuration file '%0': %1">;`。
- **L311 EN**: Declares TableGen def record `err_drv_arg_requires_bitcode_input`.
  **L311 CN**: 声明 TableGen def 记录 `err_drv_arg_requires_bitcode_input`。
- **L312 EN**: Adds a standalone statement or declaration: `"option '%0' requires input to be LLVM bitcode">;`.
  **L312 CN**: 添加一条独立语句或声明：`"option '%0' requires input to be LLVM bitcode">;`。

### Lines 313-336

````tablegen

def err_target_unsupported_arch
  : Error<"the target architecture '%0' is not supported by the target '%1'">;
def err_cpu_unsupported_isa
  : Error<"CPU '%0' does not support '%1' execution mode">;
def err_arch_unsupported_isa
  : Error<"architecture '%0' does not support '%1' execution mode">;

def err_zos_target_release_discontinued
  : Error<"z/OS target level \"%0\" is discontinued">;
def err_zos_target_unrecognized_release
  : Error<"z/OS target level \"%0\" is invalid">;

def err_drv_I_dash_not_supported : Error<
  "'%0' not supported, please use -iquote instead">;
def err_drv_unknown_argument : Error<"unknown argument: '%0'">;
def err_drv_unknown_argument_with_suggestion : Error<
  "unknown argument '%0'; did you mean '%1'?">;
def warn_drv_unknown_argument_clang_cl : Warning<
  "unknown argument ignored in clang-cl: '%0'">,
  InGroup<UnknownArgument>;
def warn_drv_unknown_argument_clang_cl_with_suggestion : Warning<
  "unknown argument ignored in clang-cl '%0'; did you mean '%1'?">,
  InGroup<UnknownArgument>;
````
- **L313 EN**: Blank line separating nearby declarations or logic blocks.
  **L313 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L314 EN**: Declares TableGen def record `err_target_unsupported_arch`.
  **L314 CN**: 声明 TableGen def 记录 `err_target_unsupported_arch`。
- **L315 EN**: Adds a standalone statement or declaration: `: Error<"the target architecture '%0' is not supported by the target '%1'">;`.
  **L315 CN**: 添加一条独立语句或声明：`: Error<"the target architecture '%0' is not supported by the target '%1'">;`。
- **L316 EN**: Declares TableGen def record `err_cpu_unsupported_isa`.
  **L316 CN**: 声明 TableGen def 记录 `err_cpu_unsupported_isa`。
- **L317 EN**: Adds a standalone statement or declaration: `: Error<"CPU '%0' does not support '%1' execution mode">;`.
  **L317 CN**: 添加一条独立语句或声明：`: Error<"CPU '%0' does not support '%1' execution mode">;`。
- **L318 EN**: Declares TableGen def record `err_arch_unsupported_isa`.
  **L318 CN**: 声明 TableGen def 记录 `err_arch_unsupported_isa`。
- **L319 EN**: Adds a standalone statement or declaration: `: Error<"architecture '%0' does not support '%1' execution mode">;`.
  **L319 CN**: 添加一条独立语句或声明：`: Error<"architecture '%0' does not support '%1' execution mode">;`。
- **L320 EN**: Blank line separating nearby declarations or logic blocks.
  **L320 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L321 EN**: Declares TableGen def record `err_zos_target_release_discontinued`.
  **L321 CN**: 声明 TableGen def 记录 `err_zos_target_release_discontinued`。
- **L322 EN**: Adds a standalone statement or declaration: `: Error<"z/OS target level \"%0\" is discontinued">;`.
  **L322 CN**: 添加一条独立语句或声明：`: Error<"z/OS target level \"%0\" is discontinued">;`。
- **L323 EN**: Declares TableGen def record `err_zos_target_unrecognized_release`.
  **L323 CN**: 声明 TableGen def 记录 `err_zos_target_unrecognized_release`。
- **L324 EN**: Adds a standalone statement or declaration: `: Error<"z/OS target level \"%0\" is invalid">;`.
  **L324 CN**: 添加一条独立语句或声明：`: Error<"z/OS target level \"%0\" is invalid">;`。
- **L325 EN**: Blank line separating nearby declarations or logic blocks.
  **L325 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L326 EN**: Declares TableGen def record `err_drv_I_dash_not_supported`.
  **L326 CN**: 声明 TableGen def 记录 `err_drv_I_dash_not_supported`。
- **L327 EN**: Adds a standalone statement or declaration: `"'%0' not supported, please use -iquote instead">;`.
  **L327 CN**: 添加一条独立语句或声明：`"'%0' not supported, please use -iquote instead">;`。
- **L328 EN**: Declares TableGen def record `err_drv_unknown_argument`.
  **L328 CN**: 声明 TableGen def 记录 `err_drv_unknown_argument`。
- **L329 EN**: Declares TableGen def record `err_drv_unknown_argument_with_suggestion`.
  **L329 CN**: 声明 TableGen def 记录 `err_drv_unknown_argument_with_suggestion`。
- **L330 EN**: Adds a standalone statement or declaration: `"unknown argument '%0'; did you mean '%1'?">;`.
  **L330 CN**: 添加一条独立语句或声明：`"unknown argument '%0'; did you mean '%1'?">;`。
- **L331 EN**: Declares TableGen def record `warn_drv_unknown_argument_clang_cl`.
  **L331 CN**: 声明 TableGen def 记录 `warn_drv_unknown_argument_clang_cl`。
- **L332 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"unknown argument ignored in clang-cl: '%0'">,`.
  **L332 CN**: 继续一个多行参数列表、初始化器或聚合项：`"unknown argument ignored in clang-cl: '%0'">,`。
- **L333 EN**: Adds a standalone statement or declaration: `InGroup<UnknownArgument>;`.
  **L333 CN**: 添加一条独立语句或声明：`InGroup<UnknownArgument>;`。
- **L334 EN**: Declares TableGen def record `warn_drv_unknown_argument_clang_cl_with_suggestion`.
  **L334 CN**: 声明 TableGen def 记录 `warn_drv_unknown_argument_clang_cl_with_suggestion`。
- **L335 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"unknown argument ignored in clang-cl '%0'; did you mean '%1'?">,`.
  **L335 CN**: 继续一个多行参数列表、初始化器或聚合项：`"unknown argument ignored in clang-cl '%0'; did you mean '%1'?">,`。
- **L336 EN**: Adds a standalone statement or declaration: `InGroup<UnknownArgument>;`.
  **L336 CN**: 添加一条独立语句或声明：`InGroup<UnknownArgument>;`。

### Lines 337-360

````tablegen
def err_drv_unknown_target_triple : Error<"unknown target triple '%0'">;

def warn_drv_ycyu_different_arg_clang_cl : Warning<
  "support for '/Yc' and '/Yu' with different filenames not implemented yet; flags ignored">,
  InGroup<ClangClPch>;
def warn_drv_yc_multiple_inputs_clang_cl : Warning<
  "support for '/Yc' with more than one source file not implemented yet; flag ignored">,
  InGroup<ClangClPch>;

def warn_drv_potentially_misspelled_joined_argument : Warning<
  "joined argument treated as '%0'; did you mean '%1'?">, InGroup<UnknownArgument>;

def err_drv_too_many_actions: Error<
    "only one action option is allowed. Got %0">;
def err_drv_invalid_value : Error<"invalid value '%1' in '%0'">;
def err_drv_invalid_int_value : Error<"invalid integral value '%1' in '%0'">;
def err_drv_invalid_value_with_suggestion : Error<
    "invalid value '%1' in '%0', expected one of: %2">;
def err_drv_alignment_not_power_of_two : Error<"alignment is not a power of 2 in '%0'">;
def err_drv_invalid_remap_file : Error<
    "invalid option '%0' not of the form <from-file>;<to-file>">;
def err_drv_invalid_gcc_install_dir : Error<"'%0' does not contain a GCC installation">;
def err_drv_invalid_gcc_output_type : Error<
    "invalid output type '%0' for use with gcc tool">;
````
- **L337 EN**: Declares TableGen def record `err_drv_unknown_target_triple`.
  **L337 CN**: 声明 TableGen def 记录 `err_drv_unknown_target_triple`。
- **L338 EN**: Blank line separating nearby declarations or logic blocks.
  **L338 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L339 EN**: Declares TableGen def record `warn_drv_ycyu_different_arg_clang_cl`.
  **L339 CN**: 声明 TableGen def 记录 `warn_drv_ycyu_different_arg_clang_cl`。
- **L340 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"support for '/Yc' and '/Yu' with different filenames not implemented yet; flags ignored">,`.
  **L340 CN**: 继续一个多行参数列表、初始化器或聚合项：`"support for '/Yc' and '/Yu' with different filenames not implemented yet; flags ignored">,`。
- **L341 EN**: Adds a standalone statement or declaration: `InGroup<ClangClPch>;`.
  **L341 CN**: 添加一条独立语句或声明：`InGroup<ClangClPch>;`。
- **L342 EN**: Declares TableGen def record `warn_drv_yc_multiple_inputs_clang_cl`.
  **L342 CN**: 声明 TableGen def 记录 `warn_drv_yc_multiple_inputs_clang_cl`。
- **L343 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"support for '/Yc' with more than one source file not implemented yet; flag ignored">,`.
  **L343 CN**: 继续一个多行参数列表、初始化器或聚合项：`"support for '/Yc' with more than one source file not implemented yet; flag ignored">,`。
- **L344 EN**: Adds a standalone statement or declaration: `InGroup<ClangClPch>;`.
  **L344 CN**: 添加一条独立语句或声明：`InGroup<ClangClPch>;`。
- **L345 EN**: Blank line separating nearby declarations or logic blocks.
  **L345 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L346 EN**: Declares TableGen def record `warn_drv_potentially_misspelled_joined_argument`.
  **L346 CN**: 声明 TableGen def 记录 `warn_drv_potentially_misspelled_joined_argument`。
- **L347 EN**: Adds a standalone statement or declaration: `"joined argument treated as '%0'; did you mean '%1'?">, InGroup<UnknownArgument>;`.
  **L347 CN**: 添加一条独立语句或声明：`"joined argument treated as '%0'; did you mean '%1'?">, InGroup<UnknownArgument>;`。
- **L348 EN**: Blank line separating nearby declarations or logic blocks.
  **L348 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L349 EN**: Declares TableGen def record `err_drv_too_many_actions`.
  **L349 CN**: 声明 TableGen def 记录 `err_drv_too_many_actions`。
- **L350 EN**: Adds a standalone statement or declaration: `"only one action option is allowed. Got %0">;`.
  **L350 CN**: 添加一条独立语句或声明：`"only one action option is allowed. Got %0">;`。
- **L351 EN**: Declares TableGen def record `err_drv_invalid_value`.
  **L351 CN**: 声明 TableGen def 记录 `err_drv_invalid_value`。
- **L352 EN**: Declares TableGen def record `err_drv_invalid_int_value`.
  **L352 CN**: 声明 TableGen def 记录 `err_drv_invalid_int_value`。
- **L353 EN**: Declares TableGen def record `err_drv_invalid_value_with_suggestion`.
  **L353 CN**: 声明 TableGen def 记录 `err_drv_invalid_value_with_suggestion`。
- **L354 EN**: Adds a standalone statement or declaration: `"invalid value '%1' in '%0', expected one of: %2">;`.
  **L354 CN**: 添加一条独立语句或声明：`"invalid value '%1' in '%0', expected one of: %2">;`。
- **L355 EN**: Declares TableGen def record `err_drv_alignment_not_power_of_two`.
  **L355 CN**: 声明 TableGen def 记录 `err_drv_alignment_not_power_of_two`。
- **L356 EN**: Declares TableGen def record `err_drv_invalid_remap_file`.
  **L356 CN**: 声明 TableGen def 记录 `err_drv_invalid_remap_file`。
- **L357 EN**: Adds a standalone statement or declaration: `"invalid option '%0' not of the form <from-file>;<to-file>">;`.
  **L357 CN**: 添加一条独立语句或声明：`"invalid option '%0' not of the form <from-file>;<to-file>">;`。
- **L358 EN**: Declares TableGen def record `err_drv_invalid_gcc_install_dir`.
  **L358 CN**: 声明 TableGen def 记录 `err_drv_invalid_gcc_install_dir`。
- **L359 EN**: Declares TableGen def record `err_drv_invalid_gcc_output_type`.
  **L359 CN**: 声明 TableGen def 记录 `err_drv_invalid_gcc_output_type`。
- **L360 EN**: Adds a standalone statement or declaration: `"invalid output type '%0' for use with gcc tool">;`.
  **L360 CN**: 添加一条独立语句或声明：`"invalid output type '%0' for use with gcc tool">;`。

### Lines 361-384

````tablegen
def err_drv_cc_print_options_failure : Error<
    "unable to open CC_PRINT_OPTIONS file: %0">;
def err_drv_lto_without_lld : Error<"LTO requires -fuse-ld=lld">;
def err_drv_preamble_format : Error<
    "incorrect format for -preamble-bytes=N,END">;
def err_drv_header_unit_extra_inputs : Error<
    "multiple inputs are not valid for header units (first extra '%0')">;
def warn_invalid_ios_deployment_target : Warning<
  "invalid iOS deployment version '%0', iOS 10 is the maximum deployment "
  "target for 32-bit targets">, InGroup<InvalidIOSDeploymentTarget>,
  DefaultError;
def err_invalid_macos_32bit_deployment_target : Error<
  "32-bit targets are not supported when building for Mac Catalyst">;
def err_drv_invalid_os_in_arg : Error<"invalid OS value '%0' in '%1'">;
def err_drv_conflicting_deployment_targets : Error<
  "conflicting deployment targets, both '%0' and '%1' are present in environment">;
def err_arc_unsupported_on_runtime : Error<
  "-fobjc-arc is not supported on platforms using the legacy runtime">;
def err_arc_unsupported_on_toolchain : Error< // feel free to generalize this
  "-fobjc-arc is not supported on versions of OS X prior to 10.6">;
def err_objc_weak_with_gc : Error<
  "-fobjc-weak is not supported in Objective-C garbage collection">;
def err_objc_weak_unsupported : Error<
  "-fobjc-weak is not supported on the current deployment target">;
````
- **L361 EN**: Declares TableGen def record `err_drv_cc_print_options_failure`.
  **L361 CN**: 声明 TableGen def 记录 `err_drv_cc_print_options_failure`。
- **L362 EN**: Adds a standalone statement or declaration: `"unable to open CC_PRINT_OPTIONS file: %0">;`.
  **L362 CN**: 添加一条独立语句或声明：`"unable to open CC_PRINT_OPTIONS file: %0">;`。
- **L363 EN**: Declares TableGen def record `err_drv_lto_without_lld`.
  **L363 CN**: 声明 TableGen def 记录 `err_drv_lto_without_lld`。
- **L364 EN**: Declares TableGen def record `err_drv_preamble_format`.
  **L364 CN**: 声明 TableGen def 记录 `err_drv_preamble_format`。
- **L365 EN**: Adds a standalone statement or declaration: `"incorrect format for -preamble-bytes=N,END">;`.
  **L365 CN**: 添加一条独立语句或声明：`"incorrect format for -preamble-bytes=N,END">;`。
- **L366 EN**: Declares TableGen def record `err_drv_header_unit_extra_inputs`.
  **L366 CN**: 声明 TableGen def 记录 `err_drv_header_unit_extra_inputs`。
- **L367 EN**: Executes a call or declaration centered on `units`.
  **L367 CN**: 执行以 `units` 为核心的调用或声明。
- **L368 EN**: Declares TableGen def record `warn_invalid_ios_deployment_target`.
  **L368 CN**: 声明 TableGen def 记录 `warn_invalid_ios_deployment_target`。
- **L369 EN**: Continues the surrounding expression or declaration: `"invalid iOS deployment version '%0', iOS 10 is the maximum deployment "`.
  **L369 CN**: 继续构造周围的表达式或声明：`"invalid iOS deployment version '%0', iOS 10 is the maximum deployment "`。
- **L370 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"target for 32-bit targets">, InGroup<InvalidIOSDeploymentTarget>,`.
  **L370 CN**: 继续一个多行参数列表、初始化器或聚合项：`"target for 32-bit targets">, InGroup<InvalidIOSDeploymentTarget>,`。
- **L371 EN**: Adds a standalone statement or declaration: `DefaultError;`.
  **L371 CN**: 添加一条独立语句或声明：`DefaultError;`。
- **L372 EN**: Declares TableGen def record `err_invalid_macos_32bit_deployment_target`.
  **L372 CN**: 声明 TableGen def 记录 `err_invalid_macos_32bit_deployment_target`。
- **L373 EN**: Adds a standalone statement or declaration: `"32-bit targets are not supported when building for Mac Catalyst">;`.
  **L373 CN**: 添加一条独立语句或声明：`"32-bit targets are not supported when building for Mac Catalyst">;`。
- **L374 EN**: Declares TableGen def record `err_drv_invalid_os_in_arg`.
  **L374 CN**: 声明 TableGen def 记录 `err_drv_invalid_os_in_arg`。
- **L375 EN**: Declares TableGen def record `err_drv_conflicting_deployment_targets`.
  **L375 CN**: 声明 TableGen def 记录 `err_drv_conflicting_deployment_targets`。
- **L376 EN**: Adds a standalone statement or declaration: `"conflicting deployment targets, both '%0' and '%1' are present in environment">;`.
  **L376 CN**: 添加一条独立语句或声明：`"conflicting deployment targets, both '%0' and '%1' are present in environment">;`。
- **L377 EN**: Declares TableGen def record `err_arc_unsupported_on_runtime`.
  **L377 CN**: 声明 TableGen def 记录 `err_arc_unsupported_on_runtime`。
- **L378 EN**: Adds a standalone statement or declaration: `"-fobjc-arc is not supported on platforms using the legacy runtime">;`.
  **L378 CN**: 添加一条独立语句或声明：`"-fobjc-arc is not supported on platforms using the legacy runtime">;`。
- **L379 EN**: Declares TableGen def record `err_arc_unsupported_on_toolchain`.
  **L379 CN**: 声明 TableGen def 记录 `err_arc_unsupported_on_toolchain`。
- **L380 EN**: Adds a standalone statement or declaration: `"-fobjc-arc is not supported on versions of OS X prior to 10.6">;`.
  **L380 CN**: 添加一条独立语句或声明：`"-fobjc-arc is not supported on versions of OS X prior to 10.6">;`。
- **L381 EN**: Declares TableGen def record `err_objc_weak_with_gc`.
  **L381 CN**: 声明 TableGen def 记录 `err_objc_weak_with_gc`。
- **L382 EN**: Adds a standalone statement or declaration: `"-fobjc-weak is not supported in Objective-C garbage collection">;`.
  **L382 CN**: 添加一条独立语句或声明：`"-fobjc-weak is not supported in Objective-C garbage collection">;`。
- **L383 EN**: Declares TableGen def record `err_objc_weak_unsupported`.
  **L383 CN**: 声明 TableGen def 记录 `err_objc_weak_unsupported`。
- **L384 EN**: Adds a standalone statement or declaration: `"-fobjc-weak is not supported on the current deployment target">;`.
  **L384 CN**: 添加一条独立语句或声明：`"-fobjc-weak is not supported on the current deployment target">;`。

### Lines 385-408

````tablegen
def err_drv_mg_requires_m_or_mm : Error<
  "option '-MG' requires '-M' or '-MM'">;
def err_drv_unknown_objc_runtime : Error<
  "unknown or ill-formed Objective-C runtime '%0'">;
def err_drv_invalid_cf_runtime_abi
  : Error<"invalid CoreFoundation Runtime ABI '%0'; must be one of "
          "'objc', 'standalone', 'swift', 'swift-5.0', 'swift-4.2', 'swift-4.1'">;
def err_drv_gnustep_objc_runtime_incompatible_binary : Error<
  "GNUstep Objective-C runtime version %0 incompatible with target binary format">;
def err_drv_emit_llvm_link : Error<
   "-emit-llvm cannot be used when linking">;
def err_drv_optimization_remark_pattern : Error<
  "in pattern '%1': %0">;
def err_drv_optimization_remark_format : Error<
  "unknown remark serializer format: '%0'">;
def err_drv_no_neon_modifier : Error<"[no]neon is not accepted as modifier, please use [no]simd instead">;
def err_drv_invalid_omp_target : Error<"OpenMP target is invalid: '%0'">;
def err_drv_incompatible_omp_arch : Error<"OpenMP target architecture '%0' pointer size is incompatible with host '%1'">;
def err_drv_omp_host_target_not_supported : Error<
  "target '%0' is not a supported OpenMP host target">;
def err_drv_expecting_fopenmp_with_fopenmp_targets : Error<
  "'-fopenmp-targets' must be used in conjunction with a '-fopenmp' option "
  "compatible with offloading; e.g., '-fopenmp=libomp' or '-fopenmp=libiomp5'">;
def err_drv_failed_to_deduce_target_from_arch : Error<
````
- **L385 EN**: Declares TableGen def record `err_drv_mg_requires_m_or_mm`.
  **L385 CN**: 声明 TableGen def 记录 `err_drv_mg_requires_m_or_mm`。
- **L386 EN**: Adds a standalone statement or declaration: `"option '-MG' requires '-M' or '-MM'">;`.
  **L386 CN**: 添加一条独立语句或声明：`"option '-MG' requires '-M' or '-MM'">;`。
- **L387 EN**: Declares TableGen def record `err_drv_unknown_objc_runtime`.
  **L387 CN**: 声明 TableGen def 记录 `err_drv_unknown_objc_runtime`。
- **L388 EN**: Adds a standalone statement or declaration: `"unknown or ill-formed Objective-C runtime '%0'">;`.
  **L388 CN**: 添加一条独立语句或声明：`"unknown or ill-formed Objective-C runtime '%0'">;`。
- **L389 EN**: Declares TableGen def record `err_drv_invalid_cf_runtime_abi`.
  **L389 CN**: 声明 TableGen def 记录 `err_drv_invalid_cf_runtime_abi`。
- **L390 EN**: Continues the surrounding expression or declaration: `: Error<"invalid CoreFoundation Runtime ABI '%0'; must be one of "`.
  **L390 CN**: 继续构造周围的表达式或声明：`: Error<"invalid CoreFoundation Runtime ABI '%0'; must be one of "`。
- **L391 EN**: Adds a standalone statement or declaration: `"'objc', 'standalone', 'swift', 'swift-5.0', 'swift-4.2', 'swift-4.1'">;`.
  **L391 CN**: 添加一条独立语句或声明：`"'objc', 'standalone', 'swift', 'swift-5.0', 'swift-4.2', 'swift-4.1'">;`。
- **L392 EN**: Declares TableGen def record `err_drv_gnustep_objc_runtime_incompatible_binary`.
  **L392 CN**: 声明 TableGen def 记录 `err_drv_gnustep_objc_runtime_incompatible_binary`。
- **L393 EN**: Adds a standalone statement or declaration: `"GNUstep Objective-C runtime version %0 incompatible with target binary format">;`.
  **L393 CN**: 添加一条独立语句或声明：`"GNUstep Objective-C runtime version %0 incompatible with target binary format">;`。
- **L394 EN**: Declares TableGen def record `err_drv_emit_llvm_link`.
  **L394 CN**: 声明 TableGen def 记录 `err_drv_emit_llvm_link`。
- **L395 EN**: Adds a standalone statement or declaration: `"-emit-llvm cannot be used when linking">;`.
  **L395 CN**: 添加一条独立语句或声明：`"-emit-llvm cannot be used when linking">;`。
- **L396 EN**: Declares TableGen def record `err_drv_optimization_remark_pattern`.
  **L396 CN**: 声明 TableGen def 记录 `err_drv_optimization_remark_pattern`。
- **L397 EN**: Adds a standalone statement or declaration: `"in pattern '%1': %0">;`.
  **L397 CN**: 添加一条独立语句或声明：`"in pattern '%1': %0">;`。
- **L398 EN**: Declares TableGen def record `err_drv_optimization_remark_format`.
  **L398 CN**: 声明 TableGen def 记录 `err_drv_optimization_remark_format`。
- **L399 EN**: Adds a standalone statement or declaration: `"unknown remark serializer format: '%0'">;`.
  **L399 CN**: 添加一条独立语句或声明：`"unknown remark serializer format: '%0'">;`。
- **L400 EN**: Declares TableGen def record `err_drv_no_neon_modifier`.
  **L400 CN**: 声明 TableGen def 记录 `err_drv_no_neon_modifier`。
- **L401 EN**: Declares TableGen def record `err_drv_invalid_omp_target`.
  **L401 CN**: 声明 TableGen def 记录 `err_drv_invalid_omp_target`。
- **L402 EN**: Declares TableGen def record `err_drv_incompatible_omp_arch`.
  **L402 CN**: 声明 TableGen def 记录 `err_drv_incompatible_omp_arch`。
- **L403 EN**: Declares TableGen def record `err_drv_omp_host_target_not_supported`.
  **L403 CN**: 声明 TableGen def 记录 `err_drv_omp_host_target_not_supported`。
- **L404 EN**: Adds a standalone statement or declaration: `"target '%0' is not a supported OpenMP host target">;`.
  **L404 CN**: 添加一条独立语句或声明：`"target '%0' is not a supported OpenMP host target">;`。
- **L405 EN**: Declares TableGen def record `err_drv_expecting_fopenmp_with_fopenmp_targets`.
  **L405 CN**: 声明 TableGen def 记录 `err_drv_expecting_fopenmp_with_fopenmp_targets`。
- **L406 EN**: Continues the surrounding expression or declaration: `"'-fopenmp-targets' must be used in conjunction with a '-fopenmp' option "`.
  **L406 CN**: 继续构造周围的表达式或声明：`"'-fopenmp-targets' must be used in conjunction with a '-fopenmp' option "`。
- **L407 EN**: Adds a standalone statement or declaration: `"compatible with offloading; e.g., '-fopenmp=libomp' or '-fopenmp=libiomp5'">;`.
  **L407 CN**: 添加一条独立语句或声明：`"compatible with offloading; e.g., '-fopenmp=libomp' or '-fopenmp=libiomp5'">;`。
- **L408 EN**: Declares TableGen def record `err_drv_failed_to_deduce_target_from_arch`.
  **L408 CN**: 声明 TableGen def 记录 `err_drv_failed_to_deduce_target_from_arch`。

### Lines 409-432

````tablegen
  "failed to deduce triple for target architecture '%0'; specify the triple "
  "using '-fopenmp-targets' and '-Xopenmp-target' instead">;
def err_drv_omp_offload_target_missingbcruntime : Error<
  "no library '%0' found in the default clang lib directory or in LIBRARY_PATH"
  "; use '--libomptarget-%1-bc-path' to specify %1 bitcode library">;
def err_drv_omp_offload_target_bcruntime_not_found : Error<
  "bitcode library '%0' does not exist">;
def err_drv_omp_offload_target_cuda_version_not_support : Error<
  "NVPTX target requires CUDA 9.2 or above; CUDA %0 detected">;
def warn_drv_omp_offload_target_duplicate : Warning<
  "OpenMP offloading target '%0' is similar to target '%1' already specified; "
  "will be ignored">, InGroup<OpenMPTarget>;
def err_drv_unsupported_embed_bitcode
    : Error<"%0 is not supported with -fembed-bitcode">;
def err_drv_bitcode_unsupported_on_toolchain : Error<
  "-fembed-bitcode is not supported on versions of iOS prior to 6.0">;
def err_drv_negative_columns : Error<
  "invalid value '%1' in '%0', value must be 'none' or a positive integer">;
def err_drv_small_columns : Error<
  "invalid value '%1' in '%0', value must be '%2' or greater">;
def warn_drv_fraw_string_literals_in_cxx11 : Warning<
  "ignoring '-f%select{no-|}0raw-string-literals', which is only valid for C and C++ standards before C++11">,
  InGroup<UnusedCommandLineArgument>;

````
- **L409 EN**: Continues the surrounding expression or declaration: `"failed to deduce triple for target architecture '%0'; specify the triple "`.
  **L409 CN**: 继续构造周围的表达式或声明：`"failed to deduce triple for target architecture '%0'; specify the triple "`。
- **L410 EN**: Adds a standalone statement or declaration: `"using '-fopenmp-targets' and '-Xopenmp-target' instead">;`.
  **L410 CN**: 添加一条独立语句或声明：`"using '-fopenmp-targets' and '-Xopenmp-target' instead">;`。
- **L411 EN**: Declares TableGen def record `err_drv_omp_offload_target_missingbcruntime`.
  **L411 CN**: 声明 TableGen def 记录 `err_drv_omp_offload_target_missingbcruntime`。
- **L412 EN**: Continues the surrounding expression or declaration: `"no library '%0' found in the default clang lib directory or in LIBRARY_PATH"`.
  **L412 CN**: 继续构造周围的表达式或声明：`"no library '%0' found in the default clang lib directory or in LIBRARY_PATH"`。
- **L413 EN**: Adds a standalone statement or declaration: `"; use '--libomptarget-%1-bc-path' to specify %1 bitcode library">;`.
  **L413 CN**: 添加一条独立语句或声明：`"; use '--libomptarget-%1-bc-path' to specify %1 bitcode library">;`。
- **L414 EN**: Declares TableGen def record `err_drv_omp_offload_target_bcruntime_not_found`.
  **L414 CN**: 声明 TableGen def 记录 `err_drv_omp_offload_target_bcruntime_not_found`。
- **L415 EN**: Adds a standalone statement or declaration: `"bitcode library '%0' does not exist">;`.
  **L415 CN**: 添加一条独立语句或声明：`"bitcode library '%0' does not exist">;`。
- **L416 EN**: Declares TableGen def record `err_drv_omp_offload_target_cuda_version_not_support`.
  **L416 CN**: 声明 TableGen def 记录 `err_drv_omp_offload_target_cuda_version_not_support`。
- **L417 EN**: Adds a standalone statement or declaration: `"NVPTX target requires CUDA 9.2 or above; CUDA %0 detected">;`.
  **L417 CN**: 添加一条独立语句或声明：`"NVPTX target requires CUDA 9.2 or above; CUDA %0 detected">;`。
- **L418 EN**: Declares TableGen def record `warn_drv_omp_offload_target_duplicate`.
  **L418 CN**: 声明 TableGen def 记录 `warn_drv_omp_offload_target_duplicate`。
- **L419 EN**: Continues the surrounding expression or declaration: `"OpenMP offloading target '%0' is similar to target '%1' already specified; "`.
  **L419 CN**: 继续构造周围的表达式或声明：`"OpenMP offloading target '%0' is similar to target '%1' already specified; "`。
- **L420 EN**: Adds a standalone statement or declaration: `"will be ignored">, InGroup<OpenMPTarget>;`.
  **L420 CN**: 添加一条独立语句或声明：`"will be ignored">, InGroup<OpenMPTarget>;`。
- **L421 EN**: Declares TableGen def record `err_drv_unsupported_embed_bitcode`.
  **L421 CN**: 声明 TableGen def 记录 `err_drv_unsupported_embed_bitcode`。
- **L422 EN**: Adds a standalone statement or declaration: `: Error<"%0 is not supported with -fembed-bitcode">;`.
  **L422 CN**: 添加一条独立语句或声明：`: Error<"%0 is not supported with -fembed-bitcode">;`。
- **L423 EN**: Declares TableGen def record `err_drv_bitcode_unsupported_on_toolchain`.
  **L423 CN**: 声明 TableGen def 记录 `err_drv_bitcode_unsupported_on_toolchain`。
- **L424 EN**: Adds a standalone statement or declaration: `"-fembed-bitcode is not supported on versions of iOS prior to 6.0">;`.
  **L424 CN**: 添加一条独立语句或声明：`"-fembed-bitcode is not supported on versions of iOS prior to 6.0">;`。
- **L425 EN**: Declares TableGen def record `err_drv_negative_columns`.
  **L425 CN**: 声明 TableGen def 记录 `err_drv_negative_columns`。
- **L426 EN**: Adds a standalone statement or declaration: `"invalid value '%1' in '%0', value must be 'none' or a positive integer">;`.
  **L426 CN**: 添加一条独立语句或声明：`"invalid value '%1' in '%0', value must be 'none' or a positive integer">;`。
- **L427 EN**: Declares TableGen def record `err_drv_small_columns`.
  **L427 CN**: 声明 TableGen def 记录 `err_drv_small_columns`。
- **L428 EN**: Adds a standalone statement or declaration: `"invalid value '%1' in '%0', value must be '%2' or greater">;`.
  **L428 CN**: 添加一条独立语句或声明：`"invalid value '%1' in '%0', value must be '%2' or greater">;`。
- **L429 EN**: Declares TableGen def record `warn_drv_fraw_string_literals_in_cxx11`.
  **L429 CN**: 声明 TableGen def 记录 `warn_drv_fraw_string_literals_in_cxx11`。
- **L430 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '-f%select{no-|}0raw-string-literals', which is only valid for C and C++ standards before C++11">,`.
  **L430 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '-f%select{no-|}0raw-string-literals', which is only valid for C and C++ standards before C++11">,`。
- **L431 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L431 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L432 EN**: Blank line separating nearby declarations or logic blocks.
  **L432 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 433-456

````tablegen
def err_drv_libclc_not_found : Error<"no libclc library '%0' found in the clang resource directory">;

def err_drv_invalid_malign_branch_EQ : Error<
  "invalid argument '%0' to -malign-branch=; each element must be one of: %1">;

def err_drv_print_header_env_var : Error<
  "environment variable CC_PRINT_HEADERS_%select{FORMAT|FILTERING}0 has invalid value %1">;
def err_drv_print_header_env_var_combination : Error<
  "unsupported combination: CC_PRINT_HEADERS_FORMAT=%0 and CC_PRINT_HEADERS_FILTERING=%1">;
def err_drv_print_header_env_var_invalid_format : Error<
  "environment variable CC_PRINT_HEADERS_FORMAT=%0 requires a compatible value for CC_PRINT_HEADERS_FILTERING">;
def err_drv_print_header_cc1_invalid_combination : Error<
  "unsupported combination: -header-include-format=%0 and -header-include-filtering=%1">;
def err_drv_print_header_cc1_invalid_filtering : Error<
  "-header-include-filtering=%0 requires a compatible value for -header-include-format">;
def err_drv_print_header_cc1_invalid_format : Error<
  "-header-include-format=%0 requires a compatible value for -header-include-filtering">;

def warn_O4_is_O3 : Warning<"-O4 is equivalent to -O3">, InGroup<Deprecated>;
def warn_drv_optimization_value : Warning<"optimization level '%0' is not supported; using '%1%2' instead">,
  InGroup<InvalidCommandLineArgument>;
def warn_ignored_gcc_optimization : Warning<"optimization flag '%0' is not supported">,
  InGroup<IgnoredOptimizationArgument>;
def warn_ignored_clang_option : Warning<"the flag '%0' has been deprecated and will be ignored">,
````
- **L433 EN**: Declares TableGen def record `err_drv_libclc_not_found`.
  **L433 CN**: 声明 TableGen def 记录 `err_drv_libclc_not_found`。
- **L434 EN**: Blank line separating nearby declarations or logic blocks.
  **L434 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L435 EN**: Declares TableGen def record `err_drv_invalid_malign_branch_EQ`.
  **L435 CN**: 声明 TableGen def 记录 `err_drv_invalid_malign_branch_EQ`。
- **L436 EN**: Adds a standalone statement or declaration: `"invalid argument '%0' to -malign-branch=; each element must be one of: %1">;`.
  **L436 CN**: 添加一条独立语句或声明：`"invalid argument '%0' to -malign-branch=; each element must be one of: %1">;`。
- **L437 EN**: Blank line separating nearby declarations or logic blocks.
  **L437 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L438 EN**: Declares TableGen def record `err_drv_print_header_env_var`.
  **L438 CN**: 声明 TableGen def 记录 `err_drv_print_header_env_var`。
- **L439 EN**: Adds a standalone statement or declaration: `"environment variable CC_PRINT_HEADERS_%select{FORMAT|FILTERING}0 has invalid value %1">;`.
  **L439 CN**: 添加一条独立语句或声明：`"environment variable CC_PRINT_HEADERS_%select{FORMAT|FILTERING}0 has invalid value %1">;`。
- **L440 EN**: Declares TableGen def record `err_drv_print_header_env_var_combination`.
  **L440 CN**: 声明 TableGen def 记录 `err_drv_print_header_env_var_combination`。
- **L441 EN**: Adds a standalone statement or declaration: `"unsupported combination: CC_PRINT_HEADERS_FORMAT=%0 and CC_PRINT_HEADERS_FILTERING=%1">;`.
  **L441 CN**: 添加一条独立语句或声明：`"unsupported combination: CC_PRINT_HEADERS_FORMAT=%0 and CC_PRINT_HEADERS_FILTERING=%1">;`。
- **L442 EN**: Declares TableGen def record `err_drv_print_header_env_var_invalid_format`.
  **L442 CN**: 声明 TableGen def 记录 `err_drv_print_header_env_var_invalid_format`。
- **L443 EN**: Adds a standalone statement or declaration: `"environment variable CC_PRINT_HEADERS_FORMAT=%0 requires a compatible value for CC_PRINT_HEADERS_FILTERING">;`.
  **L443 CN**: 添加一条独立语句或声明：`"environment variable CC_PRINT_HEADERS_FORMAT=%0 requires a compatible value for CC_PRINT_HEADERS_FILTERING">;`。
- **L444 EN**: Declares TableGen def record `err_drv_print_header_cc1_invalid_combination`.
  **L444 CN**: 声明 TableGen def 记录 `err_drv_print_header_cc1_invalid_combination`。
- **L445 EN**: Adds a standalone statement or declaration: `"unsupported combination: -header-include-format=%0 and -header-include-filtering=%1">;`.
  **L445 CN**: 添加一条独立语句或声明：`"unsupported combination: -header-include-format=%0 and -header-include-filtering=%1">;`。
- **L446 EN**: Declares TableGen def record `err_drv_print_header_cc1_invalid_filtering`.
  **L446 CN**: 声明 TableGen def 记录 `err_drv_print_header_cc1_invalid_filtering`。
- **L447 EN**: Adds a standalone statement or declaration: `"-header-include-filtering=%0 requires a compatible value for -header-include-format">;`.
  **L447 CN**: 添加一条独立语句或声明：`"-header-include-filtering=%0 requires a compatible value for -header-include-format">;`。
- **L448 EN**: Declares TableGen def record `err_drv_print_header_cc1_invalid_format`.
  **L448 CN**: 声明 TableGen def 记录 `err_drv_print_header_cc1_invalid_format`。
- **L449 EN**: Adds a standalone statement or declaration: `"-header-include-format=%0 requires a compatible value for -header-include-filtering">;`.
  **L449 CN**: 添加一条独立语句或声明：`"-header-include-format=%0 requires a compatible value for -header-include-filtering">;`。
- **L450 EN**: Blank line separating nearby declarations or logic blocks.
  **L450 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L451 EN**: Declares TableGen def record `warn_O4_is_O3`.
  **L451 CN**: 声明 TableGen def 记录 `warn_O4_is_O3`。
- **L452 EN**: Declares TableGen def record `warn_drv_optimization_value`.
  **L452 CN**: 声明 TableGen def 记录 `warn_drv_optimization_value`。
- **L453 EN**: Adds a standalone statement or declaration: `InGroup<InvalidCommandLineArgument>;`.
  **L453 CN**: 添加一条独立语句或声明：`InGroup<InvalidCommandLineArgument>;`。
- **L454 EN**: Declares TableGen def record `warn_ignored_gcc_optimization`.
  **L454 CN**: 声明 TableGen def 记录 `warn_ignored_gcc_optimization`。
- **L455 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredOptimizationArgument>;`.
  **L455 CN**: 添加一条独立语句或声明：`InGroup<IgnoredOptimizationArgument>;`。
- **L456 EN**: Declares TableGen def record `warn_ignored_clang_option`.
  **L456 CN**: 声明 TableGen def 记录 `warn_ignored_clang_option`。

### Lines 457-480

````tablegen
  InGroup<UnusedCommandLineArgument>;
def warn_drv_unsupported_opt_for_target : Warning<
  "optimization flag '%0' is not supported for target '%1'">,
  InGroup<IgnoredOptimizationArgument>;
def warn_drv_riscv_be_experimental : Warning<
  "big-endian RISC-V target support is experimental">,
  InGroup<RISCVBEExperimental>;
def warn_drv_unsupported_debug_info_opt_for_target : Warning<
  "debug information option '%0' is not supported for target '%1'">,
  InGroup<UnsupportedTargetOpt>;
def warn_drv_dwarf_version_limited_by_target : Warning<
  "debug information option '%0' is not supported; requires DWARF-%2 but "
  "target '%1' only provides DWARF-%3">,
  InGroup<UnsupportedTargetOpt>;
def warn_c_kext : Warning<
  "ignoring -fapple-kext which is valid for C++ and Objective-C++ only">;
def warn_ignoring_fdiscard_for_bitcode : Warning<
  "ignoring -fdiscard-value-names for LLVM Bitcode">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_input_file_unused : Warning<
  "%0: '%1' input unused%select{ when '%3' is present|}2">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_input_file_unused_by_cpp : Warning<
  "%0: '%1' input unused in cpp mode">,
````
- **L457 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L457 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L458 EN**: Declares TableGen def record `warn_drv_unsupported_opt_for_target`.
  **L458 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_opt_for_target`。
- **L459 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"optimization flag '%0' is not supported for target '%1'">,`.
  **L459 CN**: 继续一个多行参数列表、初始化器或聚合项：`"optimization flag '%0' is not supported for target '%1'">,`。
- **L460 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredOptimizationArgument>;`.
  **L460 CN**: 添加一条独立语句或声明：`InGroup<IgnoredOptimizationArgument>;`。
- **L461 EN**: Declares TableGen def record `warn_drv_riscv_be_experimental`.
  **L461 CN**: 声明 TableGen def 记录 `warn_drv_riscv_be_experimental`。
- **L462 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"big-endian RISC-V target support is experimental">,`.
  **L462 CN**: 继续一个多行参数列表、初始化器或聚合项：`"big-endian RISC-V target support is experimental">,`。
- **L463 EN**: Adds a standalone statement or declaration: `InGroup<RISCVBEExperimental>;`.
  **L463 CN**: 添加一条独立语句或声明：`InGroup<RISCVBEExperimental>;`。
- **L464 EN**: Declares TableGen def record `warn_drv_unsupported_debug_info_opt_for_target`.
  **L464 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_debug_info_opt_for_target`。
- **L465 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"debug information option '%0' is not supported for target '%1'">,`.
  **L465 CN**: 继续一个多行参数列表、初始化器或聚合项：`"debug information option '%0' is not supported for target '%1'">,`。
- **L466 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedTargetOpt>;`.
  **L466 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedTargetOpt>;`。
- **L467 EN**: Declares TableGen def record `warn_drv_dwarf_version_limited_by_target`.
  **L467 CN**: 声明 TableGen def 记录 `warn_drv_dwarf_version_limited_by_target`。
- **L468 EN**: Continues the surrounding expression or declaration: `"debug information option '%0' is not supported; requires DWARF-%2 but "`.
  **L468 CN**: 继续构造周围的表达式或声明：`"debug information option '%0' is not supported; requires DWARF-%2 but "`。
- **L469 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"target '%1' only provides DWARF-%3">,`.
  **L469 CN**: 继续一个多行参数列表、初始化器或聚合项：`"target '%1' only provides DWARF-%3">,`。
- **L470 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedTargetOpt>;`.
  **L470 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedTargetOpt>;`。
- **L471 EN**: Declares TableGen def record `warn_c_kext`.
  **L471 CN**: 声明 TableGen def 记录 `warn_c_kext`。
- **L472 EN**: Adds a standalone statement or declaration: `"ignoring -fapple-kext which is valid for C++ and Objective-C++ only">;`.
  **L472 CN**: 添加一条独立语句或声明：`"ignoring -fapple-kext which is valid for C++ and Objective-C++ only">;`。
- **L473 EN**: Declares TableGen def record `warn_ignoring_fdiscard_for_bitcode`.
  **L473 CN**: 声明 TableGen def 记录 `warn_ignoring_fdiscard_for_bitcode`。
- **L474 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring -fdiscard-value-names for LLVM Bitcode">,`.
  **L474 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring -fdiscard-value-names for LLVM Bitcode">,`。
- **L475 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L475 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L476 EN**: Declares TableGen def record `warn_drv_input_file_unused`.
  **L476 CN**: 声明 TableGen def 记录 `warn_drv_input_file_unused`。
- **L477 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%0: '%1' input unused%select{ when '%3' is present|}2">,`.
  **L477 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%0: '%1' input unused%select{ when '%3' is present|}2">,`。
- **L478 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L478 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L479 EN**: Declares TableGen def record `warn_drv_input_file_unused_by_cpp`.
  **L479 CN**: 声明 TableGen def 记录 `warn_drv_input_file_unused_by_cpp`。
- **L480 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%0: '%1' input unused in cpp mode">,`.
  **L480 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%0: '%1' input unused in cpp mode">,`。

### Lines 481-504

````tablegen
  InGroup<UnusedCommandLineArgument>;
def warn_drv_preprocessed_input_file_unused : Warning<
  "%0: previously preprocessed input%select{ unused when '%2' is present|}1">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_unused_argument : Warning<
  "argument unused during compilation: '%0'">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_unused_x : Warning<
  "'-x %0' after last input file has no effect">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_empty_joined_argument : Warning<
  "joined argument expects additional value: '%0'">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_diagnostics_hotness_requires_pgo : Warning<
  "argument '%0' requires profile-guided optimization information">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_diagnostics_misexpect_requires_pgo : Warning<
  "argument '%0' requires profile-guided optimization information">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_clang_unsupported : Warning<
  "the clang compiler does not support '%0'">;
def warn_drv_deprecated_arg : Warning<
  "argument '%0' is deprecated%select{|, use '%2' instead}1">, InGroup<Deprecated>;
def warn_drv_deprecated_arg_ofast : Warning<
````
- **L481 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L481 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L482 EN**: Declares TableGen def record `warn_drv_preprocessed_input_file_unused`.
  **L482 CN**: 声明 TableGen def 记录 `warn_drv_preprocessed_input_file_unused`。
- **L483 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%0: previously preprocessed input%select{ unused when '%2' is present|}1">,`.
  **L483 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%0: previously preprocessed input%select{ unused when '%2' is present|}1">,`。
- **L484 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L484 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L485 EN**: Declares TableGen def record `warn_drv_unused_argument`.
  **L485 CN**: 声明 TableGen def 记录 `warn_drv_unused_argument`。
- **L486 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"argument unused during compilation: '%0'">,`.
  **L486 CN**: 继续一个多行参数列表、初始化器或聚合项：`"argument unused during compilation: '%0'">,`。
- **L487 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L487 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L488 EN**: Declares TableGen def record `warn_drv_unused_x`.
  **L488 CN**: 声明 TableGen def 记录 `warn_drv_unused_x`。
- **L489 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'-x %0' after last input file has no effect">,`.
  **L489 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'-x %0' after last input file has no effect">,`。
- **L490 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L490 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L491 EN**: Declares TableGen def record `warn_drv_empty_joined_argument`.
  **L491 CN**: 声明 TableGen def 记录 `warn_drv_empty_joined_argument`。
- **L492 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"joined argument expects additional value: '%0'">,`.
  **L492 CN**: 继续一个多行参数列表、初始化器或聚合项：`"joined argument expects additional value: '%0'">,`。
- **L493 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L493 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L494 EN**: Declares TableGen def record `warn_drv_diagnostics_hotness_requires_pgo`.
  **L494 CN**: 声明 TableGen def 记录 `warn_drv_diagnostics_hotness_requires_pgo`。
- **L495 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"argument '%0' requires profile-guided optimization information">,`.
  **L495 CN**: 继续一个多行参数列表、初始化器或聚合项：`"argument '%0' requires profile-guided optimization information">,`。
- **L496 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L496 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L497 EN**: Declares TableGen def record `warn_drv_diagnostics_misexpect_requires_pgo`.
  **L497 CN**: 声明 TableGen def 记录 `warn_drv_diagnostics_misexpect_requires_pgo`。
- **L498 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"argument '%0' requires profile-guided optimization information">,`.
  **L498 CN**: 继续一个多行参数列表、初始化器或聚合项：`"argument '%0' requires profile-guided optimization information">,`。
- **L499 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L499 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L500 EN**: Declares TableGen def record `warn_drv_clang_unsupported`.
  **L500 CN**: 声明 TableGen def 记录 `warn_drv_clang_unsupported`。
- **L501 EN**: Adds a standalone statement or declaration: `"the clang compiler does not support '%0'">;`.
  **L501 CN**: 添加一条独立语句或声明：`"the clang compiler does not support '%0'">;`。
- **L502 EN**: Declares TableGen def record `warn_drv_deprecated_arg`.
  **L502 CN**: 声明 TableGen def 记录 `warn_drv_deprecated_arg`。
- **L503 EN**: Adds a standalone statement or declaration: `"argument '%0' is deprecated%select{|, use '%2' instead}1">, InGroup<Deprecated>;`.
  **L503 CN**: 添加一条独立语句或声明：`"argument '%0' is deprecated%select{|, use '%2' instead}1">, InGroup<Deprecated>;`。
- **L504 EN**: Declares TableGen def record `warn_drv_deprecated_arg_ofast`.
  **L504 CN**: 声明 TableGen def 记录 `warn_drv_deprecated_arg_ofast`。

### Lines 505-528

````tablegen
  "argument '-Ofast' is deprecated; use '-O3' to enable only conforming optimizations, "
  "or consult the documentation for the same behavior">,
  InGroup<DeprecatedOFast>;
def warn_drv_deprecated_arg_ofast_for_flang : Warning<
  "argument '-Ofast' is deprecated; use '-O3 -ffast-math -fstack-arrays -fno-protect-parens' for the same behavior,"
  " or '-O3 -fstack-arrays' to enable only conforming optimizations">,
  InGroup<DeprecatedOFast>;
def warn_drv_deprecated_custom : Warning<
  "argument '%0' is deprecated, %1">, InGroup<Deprecated>;
def warn_drv_assuming_mfloat_abi_is : Warning<
  "unknown platform, assuming -mfloat-abi=%0">;
def warn_drv_unsupported_float_abi_by_lib : Warning<
  "float ABI '%0' is not supported by current library">,
  InGroup<UnsupportedABI>;
def warn_drv_no_floating_point_registers: Warning<
  "'%0': selected processor lacks floating point registers">,
  InGroup<UnsupportedABI>;
def warn_ignoring_ftabstop_value : Warning<
  "ignoring invalid -ftabstop value '%0', using default value %1">;
def warn_drv_overriding_option : Warning<
  "overriding '%0' option with '%1'">,
  InGroup<DiagGroup<"overriding-option">>;
def warn_drv_overriding_deployment_version
    : Warning<"overriding deployment version from '%0' to '%1'">,
````
- **L505 EN**: Continues the surrounding expression or declaration: `"argument '-Ofast' is deprecated; use '-O3' to enable only conforming optimizations, "`.
  **L505 CN**: 继续构造周围的表达式或声明：`"argument '-Ofast' is deprecated; use '-O3' to enable only conforming optimizations, "`。
- **L506 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"or consult the documentation for the same behavior">,`.
  **L506 CN**: 继续一个多行参数列表、初始化器或聚合项：`"or consult the documentation for the same behavior">,`。
- **L507 EN**: Adds a standalone statement or declaration: `InGroup<DeprecatedOFast>;`.
  **L507 CN**: 添加一条独立语句或声明：`InGroup<DeprecatedOFast>;`。
- **L508 EN**: Declares TableGen def record `warn_drv_deprecated_arg_ofast_for_flang`.
  **L508 CN**: 声明 TableGen def 记录 `warn_drv_deprecated_arg_ofast_for_flang`。
- **L509 EN**: Continues the surrounding expression or declaration: `"argument '-Ofast' is deprecated; use '-O3 -ffast-math -fstack-arrays -fno-protect-parens' for the same behavior,"`.
  **L509 CN**: 继续构造周围的表达式或声明：`"argument '-Ofast' is deprecated; use '-O3 -ffast-math -fstack-arrays -fno-protect-parens' for the same behavior,"`。
- **L510 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" or '-O3 -fstack-arrays' to enable only conforming optimizations">,`.
  **L510 CN**: 继续一个多行参数列表、初始化器或聚合项：`" or '-O3 -fstack-arrays' to enable only conforming optimizations">,`。
- **L511 EN**: Adds a standalone statement or declaration: `InGroup<DeprecatedOFast>;`.
  **L511 CN**: 添加一条独立语句或声明：`InGroup<DeprecatedOFast>;`。
- **L512 EN**: Declares TableGen def record `warn_drv_deprecated_custom`.
  **L512 CN**: 声明 TableGen def 记录 `warn_drv_deprecated_custom`。
- **L513 EN**: Adds a standalone statement or declaration: `"argument '%0' is deprecated, %1">, InGroup<Deprecated>;`.
  **L513 CN**: 添加一条独立语句或声明：`"argument '%0' is deprecated, %1">, InGroup<Deprecated>;`。
- **L514 EN**: Declares TableGen def record `warn_drv_assuming_mfloat_abi_is`.
  **L514 CN**: 声明 TableGen def 记录 `warn_drv_assuming_mfloat_abi_is`。
- **L515 EN**: Adds a standalone statement or declaration: `"unknown platform, assuming -mfloat-abi=%0">;`.
  **L515 CN**: 添加一条独立语句或声明：`"unknown platform, assuming -mfloat-abi=%0">;`。
- **L516 EN**: Declares TableGen def record `warn_drv_unsupported_float_abi_by_lib`.
  **L516 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_float_abi_by_lib`。
- **L517 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"float ABI '%0' is not supported by current library">,`.
  **L517 CN**: 继续一个多行参数列表、初始化器或聚合项：`"float ABI '%0' is not supported by current library">,`。
- **L518 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedABI>;`.
  **L518 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedABI>;`。
- **L519 EN**: Declares TableGen def record `warn_drv_no_floating_point_registers`.
  **L519 CN**: 声明 TableGen def 记录 `warn_drv_no_floating_point_registers`。
- **L520 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0': selected processor lacks floating point registers">,`.
  **L520 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0': selected processor lacks floating point registers">,`。
- **L521 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedABI>;`.
  **L521 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedABI>;`。
- **L522 EN**: Declares TableGen def record `warn_ignoring_ftabstop_value`.
  **L522 CN**: 声明 TableGen def 记录 `warn_ignoring_ftabstop_value`。
- **L523 EN**: Adds a standalone statement or declaration: `"ignoring invalid -ftabstop value '%0', using default value %1">;`.
  **L523 CN**: 添加一条独立语句或声明：`"ignoring invalid -ftabstop value '%0', using default value %1">;`。
- **L524 EN**: Declares TableGen def record `warn_drv_overriding_option`.
  **L524 CN**: 声明 TableGen def 记录 `warn_drv_overriding_option`。
- **L525 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"overriding '%0' option with '%1'">,`.
  **L525 CN**: 继续一个多行参数列表、初始化器或聚合项：`"overriding '%0' option with '%1'">,`。
- **L526 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"overriding-option">>;`.
  **L526 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"overriding-option">>;`。
- **L527 EN**: Declares TableGen def record `warn_drv_overriding_deployment_version`.
  **L527 CN**: 声明 TableGen def 记录 `warn_drv_overriding_deployment_version`。
- **L528 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `: Warning<"overriding deployment version from '%0' to '%1'">,`.
  **L528 CN**: 继续一个多行参数列表、初始化器或聚合项：`: Warning<"overriding deployment version from '%0' to '%1'">,`。

### Lines 529-552

````tablegen
      InGroup<DiagGroup<"overriding-deployment-version">>;
def warn_drv_overriding_complex_range : Warning<
  "'%1' sets complex range to \"%3\" overriding the setting of \"%2\" that was implied by '%0'">,
  InGroup<DiagGroup<"overriding-complex-range">>;
def warn_drv_treating_input_as_cxx : Warning<
  "treating '%0' input as '%1' when in C++ mode, this behavior is deprecated">,
  InGroup<Deprecated>;
def warn_drv_pch_not_first_include : Warning<
  "precompiled header '%0' was ignored because '%1' is not first '-include'">;
def warn_drv_pch_ignoring_gch_file : Warning<
  "precompiled header '%0' was ignored because it is not a clang PCH file">,
  InGroup<IgnoredGCH>;
def warn_drv_pch_ignoring_gch_dir : Warning<
  "precompiled header directory '%0' was ignored because it contains no clang PCH files">,
  InGroup<IgnoredGCH>;
def warn_missing_sysroot : Warning<"no such sysroot directory: '%0'">,
  InGroup<DiagGroup<"missing-sysroot">>;
def warn_incompatible_sysroot : Warning<"using sysroot for '%0' but targeting '%1'">,
  InGroup<DiagGroup<"incompatible-sysroot">>;
def warn_debug_compression_unavailable : Warning<"cannot compress debug sections (%0 not enabled)">,
  InGroup<DiagGroup<"debug-compression-unavailable">>;
def warn_drv_disabling_vptr_no_rtti_default : Warning<
  "implicitly disabling vptr sanitizer because rtti wasn't enabled">,
  InGroup<AutoDisableVptrSanitizer>;
````
- **L529 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"overriding-deployment-version">>;`.
  **L529 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"overriding-deployment-version">>;`。
- **L530 EN**: Declares TableGen def record `warn_drv_overriding_complex_range`.
  **L530 CN**: 声明 TableGen def 记录 `warn_drv_overriding_complex_range`。
- **L531 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%1' sets complex range to \"%3\" overriding the setting of \"%2\" that was implied by '%0'">,`.
  **L531 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%1' sets complex range to \"%3\" overriding the setting of \"%2\" that was implied by '%0'">,`。
- **L532 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"overriding-complex-range">>;`.
  **L532 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"overriding-complex-range">>;`。
- **L533 EN**: Declares TableGen def record `warn_drv_treating_input_as_cxx`.
  **L533 CN**: 声明 TableGen def 记录 `warn_drv_treating_input_as_cxx`。
- **L534 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"treating '%0' input as '%1' when in C++ mode, this behavior is deprecated">,`.
  **L534 CN**: 继续一个多行参数列表、初始化器或聚合项：`"treating '%0' input as '%1' when in C++ mode, this behavior is deprecated">,`。
- **L535 EN**: Adds a standalone statement or declaration: `InGroup<Deprecated>;`.
  **L535 CN**: 添加一条独立语句或声明：`InGroup<Deprecated>;`。
- **L536 EN**: Declares TableGen def record `warn_drv_pch_not_first_include`.
  **L536 CN**: 声明 TableGen def 记录 `warn_drv_pch_not_first_include`。
- **L537 EN**: Adds a standalone statement or declaration: `"precompiled header '%0' was ignored because '%1' is not first '-include'">;`.
  **L537 CN**: 添加一条独立语句或声明：`"precompiled header '%0' was ignored because '%1' is not first '-include'">;`。
- **L538 EN**: Declares TableGen def record `warn_drv_pch_ignoring_gch_file`.
  **L538 CN**: 声明 TableGen def 记录 `warn_drv_pch_ignoring_gch_file`。
- **L539 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"precompiled header '%0' was ignored because it is not a clang PCH file">,`.
  **L539 CN**: 继续一个多行参数列表、初始化器或聚合项：`"precompiled header '%0' was ignored because it is not a clang PCH file">,`。
- **L540 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredGCH>;`.
  **L540 CN**: 添加一条独立语句或声明：`InGroup<IgnoredGCH>;`。
- **L541 EN**: Declares TableGen def record `warn_drv_pch_ignoring_gch_dir`.
  **L541 CN**: 声明 TableGen def 记录 `warn_drv_pch_ignoring_gch_dir`。
- **L542 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"precompiled header directory '%0' was ignored because it contains no clang PCH files">,`.
  **L542 CN**: 继续一个多行参数列表、初始化器或聚合项：`"precompiled header directory '%0' was ignored because it contains no clang PCH files">,`。
- **L543 EN**: Adds a standalone statement or declaration: `InGroup<IgnoredGCH>;`.
  **L543 CN**: 添加一条独立语句或声明：`InGroup<IgnoredGCH>;`。
- **L544 EN**: Declares TableGen def record `warn_missing_sysroot`.
  **L544 CN**: 声明 TableGen def 记录 `warn_missing_sysroot`。
- **L545 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"missing-sysroot">>;`.
  **L545 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"missing-sysroot">>;`。
- **L546 EN**: Declares TableGen def record `warn_incompatible_sysroot`.
  **L546 CN**: 声明 TableGen def 记录 `warn_incompatible_sysroot`。
- **L547 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"incompatible-sysroot">>;`.
  **L547 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"incompatible-sysroot">>;`。
- **L548 EN**: Declares TableGen def record `warn_debug_compression_unavailable`.
  **L548 CN**: 声明 TableGen def 记录 `warn_debug_compression_unavailable`。
- **L549 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"debug-compression-unavailable">>;`.
  **L549 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"debug-compression-unavailable">>;`。
- **L550 EN**: Declares TableGen def record `warn_drv_disabling_vptr_no_rtti_default`.
  **L550 CN**: 声明 TableGen def 记录 `warn_drv_disabling_vptr_no_rtti_default`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"implicitly disabling vptr sanitizer because rtti wasn't enabled">,`.
  **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`"implicitly disabling vptr sanitizer because rtti wasn't enabled">,`。
- **L552 EN**: Adds a standalone statement or declaration: `InGroup<AutoDisableVptrSanitizer>;`.
  **L552 CN**: 添加一条独立语句或声明：`InGroup<AutoDisableVptrSanitizer>;`。

### Lines 553-576

````tablegen
def warn_drv_object_size_disabled_O0 : Warning<
  "the object size sanitizer has no effect at -O0, but is explicitly enabled: %0">,
  InGroup<InvalidCommandLineArgument>, DefaultWarnNoWerror;
def warn_ignoring_verify_debuginfo_preserve_export : Warning<
  "ignoring -fverify-debuginfo-preserve-export=%0 because "
  "-fverify-debuginfo-preserve wasn't enabled">,
  InGroup<UnusedCommandLineArgument>;
def warn_unsupported_branch_protection: Warning <
  "invalid branch protection option '%0' in '%1'">, InGroup<BranchProtection>;
def err_sls_hardening_arm_not_supported : Error<
  "-mharden-sls is only supported on armv7-a or later">;
def warn_drv_large_data_threshold_invalid_code_model: Warning<
  "'%0' only applies to medium and large code models">,
  InGroup<UnusedCommandLineArgument>;
def warn_drv_math_errno_enabled_after_veclib: Warning<
  "math errno enabled by '%0' after it was implicitly disabled by '%1',"
  " this may limit the utilization of the vector library">,
  InGroup<MathErrnoEnabledWithVecLib>;

def note_drv_command_failed_diag_msg : Note<
  "diagnostic msg: %0">;
def note_drv_t_option_is_global : Note<
  "the last '/TC' or '/TP' option takes precedence over earlier instances">;
def note_drv_address_sanitizer_debug_runtime : Note<
````
- **L553 EN**: Declares TableGen def record `warn_drv_object_size_disabled_O0`.
  **L553 CN**: 声明 TableGen def 记录 `warn_drv_object_size_disabled_O0`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the object size sanitizer has no effect at -O0, but is explicitly enabled: %0">,`.
  **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the object size sanitizer has no effect at -O0, but is explicitly enabled: %0">,`。
- **L555 EN**: Adds a standalone statement or declaration: `InGroup<InvalidCommandLineArgument>, DefaultWarnNoWerror;`.
  **L555 CN**: 添加一条独立语句或声明：`InGroup<InvalidCommandLineArgument>, DefaultWarnNoWerror;`。
- **L556 EN**: Declares TableGen def record `warn_ignoring_verify_debuginfo_preserve_export`.
  **L556 CN**: 声明 TableGen def 记录 `warn_ignoring_verify_debuginfo_preserve_export`。
- **L557 EN**: Continues the surrounding expression or declaration: `"ignoring -fverify-debuginfo-preserve-export=%0 because "`.
  **L557 CN**: 继续构造周围的表达式或声明：`"ignoring -fverify-debuginfo-preserve-export=%0 because "`。
- **L558 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"-fverify-debuginfo-preserve wasn't enabled">,`.
  **L558 CN**: 继续一个多行参数列表、初始化器或聚合项：`"-fverify-debuginfo-preserve wasn't enabled">,`。
- **L559 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L559 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L560 EN**: Declares TableGen def record `warn_unsupported_branch_protection`.
  **L560 CN**: 声明 TableGen def 记录 `warn_unsupported_branch_protection`。
- **L561 EN**: Adds a standalone statement or declaration: `"invalid branch protection option '%0' in '%1'">, InGroup<BranchProtection>;`.
  **L561 CN**: 添加一条独立语句或声明：`"invalid branch protection option '%0' in '%1'">, InGroup<BranchProtection>;`。
- **L562 EN**: Declares TableGen def record `err_sls_hardening_arm_not_supported`.
  **L562 CN**: 声明 TableGen def 记录 `err_sls_hardening_arm_not_supported`。
- **L563 EN**: Adds a standalone statement or declaration: `"-mharden-sls is only supported on armv7-a or later">;`.
  **L563 CN**: 添加一条独立语句或声明：`"-mharden-sls is only supported on armv7-a or later">;`。
- **L564 EN**: Declares TableGen def record `warn_drv_large_data_threshold_invalid_code_model`.
  **L564 CN**: 声明 TableGen def 记录 `warn_drv_large_data_threshold_invalid_code_model`。
- **L565 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' only applies to medium and large code models">,`.
  **L565 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' only applies to medium and large code models">,`。
- **L566 EN**: Adds a standalone statement or declaration: `InGroup<UnusedCommandLineArgument>;`.
  **L566 CN**: 添加一条独立语句或声明：`InGroup<UnusedCommandLineArgument>;`。
- **L567 EN**: Declares TableGen def record `warn_drv_math_errno_enabled_after_veclib`.
  **L567 CN**: 声明 TableGen def 记录 `warn_drv_math_errno_enabled_after_veclib`。
- **L568 EN**: Continues the surrounding expression or declaration: `"math errno enabled by '%0' after it was implicitly disabled by '%1',"`.
  **L568 CN**: 继续构造周围的表达式或声明：`"math errno enabled by '%0' after it was implicitly disabled by '%1',"`。
- **L569 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" this may limit the utilization of the vector library">,`.
  **L569 CN**: 继续一个多行参数列表、初始化器或聚合项：`" this may limit the utilization of the vector library">,`。
- **L570 EN**: Adds a standalone statement or declaration: `InGroup<MathErrnoEnabledWithVecLib>;`.
  **L570 CN**: 添加一条独立语句或声明：`InGroup<MathErrnoEnabledWithVecLib>;`。
- **L571 EN**: Blank line separating nearby declarations or logic blocks.
  **L571 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L572 EN**: Declares TableGen def record `note_drv_command_failed_diag_msg`.
  **L572 CN**: 声明 TableGen def 记录 `note_drv_command_failed_diag_msg`。
- **L573 EN**: Adds a standalone statement or declaration: `"diagnostic msg: %0">;`.
  **L573 CN**: 添加一条独立语句或声明：`"diagnostic msg: %0">;`。
- **L574 EN**: Declares TableGen def record `note_drv_t_option_is_global`.
  **L574 CN**: 声明 TableGen def 记录 `note_drv_t_option_is_global`。
- **L575 EN**: Adds a standalone statement or declaration: `"the last '/TC' or '/TP' option takes precedence over earlier instances">;`.
  **L575 CN**: 添加一条独立语句或声明：`"the last '/TC' or '/TP' option takes precedence over earlier instances">;`。
- **L576 EN**: Declares TableGen def record `note_drv_address_sanitizer_debug_runtime`.
  **L576 CN**: 声明 TableGen def 记录 `note_drv_address_sanitizer_debug_runtime`。

### Lines 577-600

````tablegen
  "AddressSanitizer doesn't support linking with debug runtime libraries yet">;
def note_drv_use_standard : Note<"use '%0'"
  "%select{| or '%3'|, '%3', or '%4'|, '%3', '%4', or '%5'}2 "
  "for '%1' standard">;

def err_analyzer_config_no_value : Error<
  "analyzer-config option '%0' has a key but no value">;
def err_analyzer_config_multiple_values : Error<
  "analyzer-config option '%0' should contain only one '='">;
def err_analyzer_config_invalid_input : Error<
  "invalid input for analyzer-config option '%0', that expects %1 value">;
def err_analyzer_config_unknown : Error<"unknown analyzer-config '%0'">;
def err_analyzer_checker_option_unknown : Error<
  "checker '%0' has no option called '%1'">;
def err_analyzer_checker_option_invalid_input : Error<
  "invalid input for checker option '%0', that expects %1">;
def err_analyzer_checker_incompatible_analyzer_option : Error<
  "checker cannot be enabled with analyzer option '%0' == %1">;
def err_analyzer_not_built_with_z3 : Error<
  "analyzer constraint manager 'z3' is only available if LLVM was built with "
  "-DLLVM_ENABLE_Z3_SOLVER=ON">;

def warn_drv_needs_hvx : Warning<
  "%0 requires HVX, use -mhvx/-mhvx= to enable it">,
````
- **L577 EN**: Adds a standalone statement or declaration: `"AddressSanitizer doesn't support linking with debug runtime libraries yet">;`.
  **L577 CN**: 添加一条独立语句或声明：`"AddressSanitizer doesn't support linking with debug runtime libraries yet">;`。
- **L578 EN**: Declares TableGen def record `note_drv_use_standard`.
  **L578 CN**: 声明 TableGen def 记录 `note_drv_use_standard`。
- **L579 EN**: Continues the surrounding expression or declaration: `"%select{| or '%3'|, '%3', or '%4'|, '%3', '%4', or '%5'}2 "`.
  **L579 CN**: 继续构造周围的表达式或声明：`"%select{| or '%3'|, '%3', or '%4'|, '%3', '%4', or '%5'}2 "`。
- **L580 EN**: Adds a standalone statement or declaration: `"for '%1' standard">;`.
  **L580 CN**: 添加一条独立语句或声明：`"for '%1' standard">;`。
- **L581 EN**: Blank line separating nearby declarations or logic blocks.
  **L581 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L582 EN**: Declares TableGen def record `err_analyzer_config_no_value`.
  **L582 CN**: 声明 TableGen def 记录 `err_analyzer_config_no_value`。
- **L583 EN**: Adds a standalone statement or declaration: `"analyzer-config option '%0' has a key but no value">;`.
  **L583 CN**: 添加一条独立语句或声明：`"analyzer-config option '%0' has a key but no value">;`。
- **L584 EN**: Declares TableGen def record `err_analyzer_config_multiple_values`.
  **L584 CN**: 声明 TableGen def 记录 `err_analyzer_config_multiple_values`。
- **L585 EN**: Adds a standalone statement or declaration: `"analyzer-config option '%0' should contain only one '='">;`.
  **L585 CN**: 添加一条独立语句或声明：`"analyzer-config option '%0' should contain only one '='">;`。
- **L586 EN**: Declares TableGen def record `err_analyzer_config_invalid_input`.
  **L586 CN**: 声明 TableGen def 记录 `err_analyzer_config_invalid_input`。
- **L587 EN**: Adds a standalone statement or declaration: `"invalid input for analyzer-config option '%0', that expects %1 value">;`.
  **L587 CN**: 添加一条独立语句或声明：`"invalid input for analyzer-config option '%0', that expects %1 value">;`。
- **L588 EN**: Declares TableGen def record `err_analyzer_config_unknown`.
  **L588 CN**: 声明 TableGen def 记录 `err_analyzer_config_unknown`。
- **L589 EN**: Declares TableGen def record `err_analyzer_checker_option_unknown`.
  **L589 CN**: 声明 TableGen def 记录 `err_analyzer_checker_option_unknown`。
- **L590 EN**: Adds a standalone statement or declaration: `"checker '%0' has no option called '%1'">;`.
  **L590 CN**: 添加一条独立语句或声明：`"checker '%0' has no option called '%1'">;`。
- **L591 EN**: Declares TableGen def record `err_analyzer_checker_option_invalid_input`.
  **L591 CN**: 声明 TableGen def 记录 `err_analyzer_checker_option_invalid_input`。
- **L592 EN**: Adds a standalone statement or declaration: `"invalid input for checker option '%0', that expects %1">;`.
  **L592 CN**: 添加一条独立语句或声明：`"invalid input for checker option '%0', that expects %1">;`。
- **L593 EN**: Declares TableGen def record `err_analyzer_checker_incompatible_analyzer_option`.
  **L593 CN**: 声明 TableGen def 记录 `err_analyzer_checker_incompatible_analyzer_option`。
- **L594 EN**: Adds a standalone statement or declaration: `"checker cannot be enabled with analyzer option '%0' == %1">;`.
  **L594 CN**: 添加一条独立语句或声明：`"checker cannot be enabled with analyzer option '%0' == %1">;`。
- **L595 EN**: Declares TableGen def record `err_analyzer_not_built_with_z3`.
  **L595 CN**: 声明 TableGen def 记录 `err_analyzer_not_built_with_z3`。
- **L596 EN**: Continues the surrounding expression or declaration: `"analyzer constraint manager 'z3' is only available if LLVM was built with "`.
  **L596 CN**: 继续构造周围的表达式或声明：`"analyzer constraint manager 'z3' is only available if LLVM was built with "`。
- **L597 EN**: Adds a standalone statement or declaration: `"-DLLVM_ENABLE_Z3_SOLVER=ON">;`.
  **L597 CN**: 添加一条独立语句或声明：`"-DLLVM_ENABLE_Z3_SOLVER=ON">;`。
- **L598 EN**: Blank line separating nearby declarations or logic blocks.
  **L598 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L599 EN**: Declares TableGen def record `warn_drv_needs_hvx`.
  **L599 CN**: 声明 TableGen def 记录 `warn_drv_needs_hvx`。
- **L600 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%0 requires HVX, use -mhvx/-mhvx= to enable it">,`.
  **L600 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%0 requires HVX, use -mhvx/-mhvx= to enable it">,`。

### Lines 601-624

````tablegen
  InGroup<OptionIgnored>;
def err_drv_needs_hvx : Error<
  "%0 requires HVX, use -mhvx/-mhvx= to enable it">;
def err_drv_needs_hvx_version : Error<
  "%0 is not supported on HVX %1">;

def err_drv_module_header_wrong_kind : Error<
  "header file '%0' input type '%1' does not match type of prior input "
  "in module compilation; use '-x %2' to override">;
def err_drv_modules_validate_once_requires_timestamp : Error<
  "option '-fmodules-validate-once-per-build-session' requires "
  "'-fbuild-session-timestamp=<seconds since Epoch>' or '-fbuild-session-file=<file>'">;

def err_test_module_file_extension_format : Error<
  "-ftest-module-file-extension argument '%0' is not of the required form "
  "'blockname:major:minor:hashed:user info'">;

def err_drv_module_output_with_multiple_arch : Error<
  "option '-fmodule-output' cannot be used with multiple arch options">;

def err_drv_reduced_module_output_overrided : Warning<
  "the implicit output of reduced BMI may be overrided by the output file specified by '--precompile'. "
  "please consider use '-fmodule-output=' to specify the output file for reduced BMI explicitly">,
  InGroup<DiagGroup<"reduced-bmi-output-overrided">>;
````
- **L601 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L601 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L602 EN**: Declares TableGen def record `err_drv_needs_hvx`.
  **L602 CN**: 声明 TableGen def 记录 `err_drv_needs_hvx`。
- **L603 EN**: Adds a standalone statement or declaration: `"%0 requires HVX, use -mhvx/-mhvx= to enable it">;`.
  **L603 CN**: 添加一条独立语句或声明：`"%0 requires HVX, use -mhvx/-mhvx= to enable it">;`。
- **L604 EN**: Declares TableGen def record `err_drv_needs_hvx_version`.
  **L604 CN**: 声明 TableGen def 记录 `err_drv_needs_hvx_version`。
- **L605 EN**: Adds a standalone statement or declaration: `"%0 is not supported on HVX %1">;`.
  **L605 CN**: 添加一条独立语句或声明：`"%0 is not supported on HVX %1">;`。
- **L606 EN**: Blank line separating nearby declarations or logic blocks.
  **L606 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L607 EN**: Declares TableGen def record `err_drv_module_header_wrong_kind`.
  **L607 CN**: 声明 TableGen def 记录 `err_drv_module_header_wrong_kind`。
- **L608 EN**: Continues the surrounding expression or declaration: `"header file '%0' input type '%1' does not match type of prior input "`.
  **L608 CN**: 继续构造周围的表达式或声明：`"header file '%0' input type '%1' does not match type of prior input "`。
- **L609 EN**: Adds a standalone statement or declaration: `"in module compilation; use '-x %2' to override">;`.
  **L609 CN**: 添加一条独立语句或声明：`"in module compilation; use '-x %2' to override">;`。
- **L610 EN**: Declares TableGen def record `err_drv_modules_validate_once_requires_timestamp`.
  **L610 CN**: 声明 TableGen def 记录 `err_drv_modules_validate_once_requires_timestamp`。
- **L611 EN**: Continues the surrounding expression or declaration: `"option '-fmodules-validate-once-per-build-session' requires "`.
  **L611 CN**: 继续构造周围的表达式或声明：`"option '-fmodules-validate-once-per-build-session' requires "`。
- **L612 EN**: Adds a standalone statement or declaration: `"'-fbuild-session-timestamp=<seconds since Epoch>' or '-fbuild-session-file=<file>'">;`.
  **L612 CN**: 添加一条独立语句或声明：`"'-fbuild-session-timestamp=<seconds since Epoch>' or '-fbuild-session-file=<file>'">;`。
- **L613 EN**: Blank line separating nearby declarations or logic blocks.
  **L613 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L614 EN**: Declares TableGen def record `err_test_module_file_extension_format`.
  **L614 CN**: 声明 TableGen def 记录 `err_test_module_file_extension_format`。
- **L615 EN**: Continues the surrounding expression or declaration: `"-ftest-module-file-extension argument '%0' is not of the required form "`.
  **L615 CN**: 继续构造周围的表达式或声明：`"-ftest-module-file-extension argument '%0' is not of the required form "`。
- **L616 EN**: Adds a standalone statement or declaration: `"'blockname:major:minor:hashed:user info'">;`.
  **L616 CN**: 添加一条独立语句或声明：`"'blockname:major:minor:hashed:user info'">;`。
- **L617 EN**: Blank line separating nearby declarations or logic blocks.
  **L617 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L618 EN**: Declares TableGen def record `err_drv_module_output_with_multiple_arch`.
  **L618 CN**: 声明 TableGen def 记录 `err_drv_module_output_with_multiple_arch`。
- **L619 EN**: Adds a standalone statement or declaration: `"option '-fmodule-output' cannot be used with multiple arch options">;`.
  **L619 CN**: 添加一条独立语句或声明：`"option '-fmodule-output' cannot be used with multiple arch options">;`。
- **L620 EN**: Blank line separating nearby declarations or logic blocks.
  **L620 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L621 EN**: Declares TableGen def record `err_drv_reduced_module_output_overrided`.
  **L621 CN**: 声明 TableGen def 记录 `err_drv_reduced_module_output_overrided`。
- **L622 EN**: Continues the surrounding expression or declaration: `"the implicit output of reduced BMI may be overrided by the output file specified by '--precompile'. "`.
  **L622 CN**: 继续构造周围的表达式或声明：`"the implicit output of reduced BMI may be overrided by the output file specified by '--precompile'. "`。
- **L623 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"please consider use '-fmodule-output=' to specify the output file for reduced BMI explicitly">,`.
  **L623 CN**: 继续一个多行参数列表、初始化器或聚合项：`"please consider use '-fmodule-output=' to specify the output file for reduced BMI explicitly">,`。
- **L624 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"reduced-bmi-output-overrided">>;`.
  **L624 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"reduced-bmi-output-overrided">>;`。

### Lines 625-648

````tablegen

def err_drv_modules_driver_requires_reduced_bmi : Error<
  "'-fmodules-driver' is currently incompatible with '-fno-modules-reduced-bmi'">;
def remark_performing_driver_managed_module_build : Remark<
  "performing driver managed module build">, InGroup<ModulesDriver>;
def remark_modules_manifest_not_found : Remark<
  "standard modules manifest file not found; import of standard library "
  "modules not supported">,
  InGroup<ModulesDriver>;
def remark_using_modules_manifest : Remark<
  "using standard modules manifest file '%0'">,
  InGroup<ModulesDriver>;
def err_modules_manifest_failed_parse : Error<
  "failure while parsing standard modules manifest: '%0'">;
def err_default_modules_cache_not_available : Error<
  "unable to determine the default -fmodules-cache-path. Please specify it "
  "explicitly using -fmodules-cache-path or the CLANG_MODULE_CACHE_PATH "
  "environment variable">;
def err_dependency_scan_failed : Error<
  "failed to perform dependency scan">;
def err_modules_driver_named_module_redefinition : Error<
  "duplicate definitions of C++20 named module '%0' in '%1' and '%2'">;
def remark_printing_module_graph : Remark<
  "printing module dependency graph">,
````
- **L625 EN**: Blank line separating nearby declarations or logic blocks.
  **L625 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L626 EN**: Declares TableGen def record `err_drv_modules_driver_requires_reduced_bmi`.
  **L626 CN**: 声明 TableGen def 记录 `err_drv_modules_driver_requires_reduced_bmi`。
- **L627 EN**: Adds a standalone statement or declaration: `"'-fmodules-driver' is currently incompatible with '-fno-modules-reduced-bmi'">;`.
  **L627 CN**: 添加一条独立语句或声明：`"'-fmodules-driver' is currently incompatible with '-fno-modules-reduced-bmi'">;`。
- **L628 EN**: Declares TableGen def record `remark_performing_driver_managed_module_build`.
  **L628 CN**: 声明 TableGen def 记录 `remark_performing_driver_managed_module_build`。
- **L629 EN**: Adds a standalone statement or declaration: `"performing driver managed module build">, InGroup<ModulesDriver>;`.
  **L629 CN**: 添加一条独立语句或声明：`"performing driver managed module build">, InGroup<ModulesDriver>;`。
- **L630 EN**: Declares TableGen def record `remark_modules_manifest_not_found`.
  **L630 CN**: 声明 TableGen def 记录 `remark_modules_manifest_not_found`。
- **L631 EN**: Continues the surrounding expression or declaration: `"standard modules manifest file not found; import of standard library "`.
  **L631 CN**: 继续构造周围的表达式或声明：`"standard modules manifest file not found; import of standard library "`。
- **L632 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"modules not supported">,`.
  **L632 CN**: 继续一个多行参数列表、初始化器或聚合项：`"modules not supported">,`。
- **L633 EN**: Adds a standalone statement or declaration: `InGroup<ModulesDriver>;`.
  **L633 CN**: 添加一条独立语句或声明：`InGroup<ModulesDriver>;`。
- **L634 EN**: Declares TableGen def record `remark_using_modules_manifest`.
  **L634 CN**: 声明 TableGen def 记录 `remark_using_modules_manifest`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"using standard modules manifest file '%0'">,`.
  **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`"using standard modules manifest file '%0'">,`。
- **L636 EN**: Adds a standalone statement or declaration: `InGroup<ModulesDriver>;`.
  **L636 CN**: 添加一条独立语句或声明：`InGroup<ModulesDriver>;`。
- **L637 EN**: Declares TableGen def record `err_modules_manifest_failed_parse`.
  **L637 CN**: 声明 TableGen def 记录 `err_modules_manifest_failed_parse`。
- **L638 EN**: Adds a standalone statement or declaration: `"failure while parsing standard modules manifest: '%0'">;`.
  **L638 CN**: 添加一条独立语句或声明：`"failure while parsing standard modules manifest: '%0'">;`。
- **L639 EN**: Declares TableGen def record `err_default_modules_cache_not_available`.
  **L639 CN**: 声明 TableGen def 记录 `err_default_modules_cache_not_available`。
- **L640 EN**: Continues the surrounding expression or declaration: `"unable to determine the default -fmodules-cache-path. Please specify it "`.
  **L640 CN**: 继续构造周围的表达式或声明：`"unable to determine the default -fmodules-cache-path. Please specify it "`。
- **L641 EN**: Continues the surrounding expression or declaration: `"explicitly using -fmodules-cache-path or the CLANG_MODULE_CACHE_PATH "`.
  **L641 CN**: 继续构造周围的表达式或声明：`"explicitly using -fmodules-cache-path or the CLANG_MODULE_CACHE_PATH "`。
- **L642 EN**: Adds a standalone statement or declaration: `"environment variable">;`.
  **L642 CN**: 添加一条独立语句或声明：`"environment variable">;`。
- **L643 EN**: Declares TableGen def record `err_dependency_scan_failed`.
  **L643 CN**: 声明 TableGen def 记录 `err_dependency_scan_failed`。
- **L644 EN**: Adds a standalone statement or declaration: `"failed to perform dependency scan">;`.
  **L644 CN**: 添加一条独立语句或声明：`"failed to perform dependency scan">;`。
- **L645 EN**: Declares TableGen def record `err_modules_driver_named_module_redefinition`.
  **L645 CN**: 声明 TableGen def 记录 `err_modules_driver_named_module_redefinition`。
- **L646 EN**: Adds a standalone statement or declaration: `"duplicate definitions of C++20 named module '%0' in '%1' and '%2'">;`.
  **L646 CN**: 添加一条独立语句或声明：`"duplicate definitions of C++20 named module '%0' in '%1' and '%2'">;`。
- **L647 EN**: Declares TableGen def record `remark_printing_module_graph`.
  **L647 CN**: 声明 TableGen def 记录 `remark_printing_module_graph`。
- **L648 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"printing module dependency graph">,`.
  **L648 CN**: 继续一个多行参数列表、初始化器或聚合项：`"printing module dependency graph">,`。

### Lines 649-672

````tablegen
  InGroup<ModulesDriver>;
def err_module_defined_outside_of_module_source : Error<
  "module '%0' is defined in file '%1', but module declarations are only "
  "allowed in C++ module inputs; use the '.cppm' extension or '-x c++module'">;

def warn_drv_delayed_template_parsing_after_cxx20 : Warning<
  "-fdelayed-template-parsing is deprecated after C++20">,
  InGroup<DiagGroup<"delayed-template-parsing-in-cxx20">>;

def err_drv_extract_api_wrong_kind : Error<
  "header file '%0' input '%1' does not match the type of prior input "
  "in api extraction; use '-x %2' to override">;

def err_drv_missing_symbol_graph_dir: Error<
  "must provide a symbol graph output directory using "
  "'--symbol-graph-dir=<directory>'">;

def err_drv_unexpected_symbol_graph_output : Error<
  "unexpected output symbol graph '%1'; please provide "
  "'--symbol-graph-dir=<directory>' instead">;

def warn_slash_u_filename : Warning<"'/U%0' treated as the '/U' option">,
  InGroup<DiagGroup<"slash-u-filename">>;
def note_use_dashdash : Note<
````
- **L649 EN**: Adds a standalone statement or declaration: `InGroup<ModulesDriver>;`.
  **L649 CN**: 添加一条独立语句或声明：`InGroup<ModulesDriver>;`。
- **L650 EN**: Declares TableGen def record `err_module_defined_outside_of_module_source`.
  **L650 CN**: 声明 TableGen def 记录 `err_module_defined_outside_of_module_source`。
- **L651 EN**: Continues the surrounding expression or declaration: `"module '%0' is defined in file '%1', but module declarations are only "`.
  **L651 CN**: 继续构造周围的表达式或声明：`"module '%0' is defined in file '%1', but module declarations are only "`。
- **L652 EN**: Adds a standalone statement or declaration: `"allowed in C++ module inputs; use the '.cppm' extension or '-x c++module'">;`.
  **L652 CN**: 添加一条独立语句或声明：`"allowed in C++ module inputs; use the '.cppm' extension or '-x c++module'">;`。
- **L653 EN**: Blank line separating nearby declarations or logic blocks.
  **L653 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L654 EN**: Declares TableGen def record `warn_drv_delayed_template_parsing_after_cxx20`.
  **L654 CN**: 声明 TableGen def 记录 `warn_drv_delayed_template_parsing_after_cxx20`。
- **L655 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"-fdelayed-template-parsing is deprecated after C++20">,`.
  **L655 CN**: 继续一个多行参数列表、初始化器或聚合项：`"-fdelayed-template-parsing is deprecated after C++20">,`。
- **L656 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"delayed-template-parsing-in-cxx20">>;`.
  **L656 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"delayed-template-parsing-in-cxx20">>;`。
- **L657 EN**: Blank line separating nearby declarations or logic blocks.
  **L657 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L658 EN**: Declares TableGen def record `err_drv_extract_api_wrong_kind`.
  **L658 CN**: 声明 TableGen def 记录 `err_drv_extract_api_wrong_kind`。
- **L659 EN**: Continues the surrounding expression or declaration: `"header file '%0' input '%1' does not match the type of prior input "`.
  **L659 CN**: 继续构造周围的表达式或声明：`"header file '%0' input '%1' does not match the type of prior input "`。
- **L660 EN**: Adds a standalone statement or declaration: `"in api extraction; use '-x %2' to override">;`.
  **L660 CN**: 添加一条独立语句或声明：`"in api extraction; use '-x %2' to override">;`。
- **L661 EN**: Blank line separating nearby declarations or logic blocks.
  **L661 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L662 EN**: Declares TableGen def record `err_drv_missing_symbol_graph_dir`.
  **L662 CN**: 声明 TableGen def 记录 `err_drv_missing_symbol_graph_dir`。
- **L663 EN**: Continues the surrounding expression or declaration: `"must provide a symbol graph output directory using "`.
  **L663 CN**: 继续构造周围的表达式或声明：`"must provide a symbol graph output directory using "`。
- **L664 EN**: Adds a standalone statement or declaration: `"'--symbol-graph-dir=<directory>'">;`.
  **L664 CN**: 添加一条独立语句或声明：`"'--symbol-graph-dir=<directory>'">;`。
- **L665 EN**: Blank line separating nearby declarations or logic blocks.
  **L665 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L666 EN**: Declares TableGen def record `err_drv_unexpected_symbol_graph_output`.
  **L666 CN**: 声明 TableGen def 记录 `err_drv_unexpected_symbol_graph_output`。
- **L667 EN**: Continues the surrounding expression or declaration: `"unexpected output symbol graph '%1'; please provide "`.
  **L667 CN**: 继续构造周围的表达式或声明：`"unexpected output symbol graph '%1'; please provide "`。
- **L668 EN**: Adds a standalone statement or declaration: `"'--symbol-graph-dir=<directory>' instead">;`.
  **L668 CN**: 添加一条独立语句或声明：`"'--symbol-graph-dir=<directory>' instead">;`。
- **L669 EN**: Blank line separating nearby declarations or logic blocks.
  **L669 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L670 EN**: Declares TableGen def record `warn_slash_u_filename`.
  **L670 CN**: 声明 TableGen def 记录 `warn_slash_u_filename`。
- **L671 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"slash-u-filename">>;`.
  **L671 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"slash-u-filename">>;`。
- **L672 EN**: Declares TableGen def record `note_use_dashdash`.
  **L672 CN**: 声明 TableGen def 记录 `note_use_dashdash`。

### Lines 673-696

````tablegen
  "use '--' to treat subsequent arguments as filenames">;

def err_drv_ropi_rwpi_incompatible_with_pic : Error<
  "embedded and GOT-based position independence are incompatible">;
def err_drv_ropi_incompatible_with_cxx : Error<
  "ROPI is not compatible with c++">;

def err_stack_tagging_requires_hardware_feature : Error<
  "'-fsanitize=memtag-stack' requires hardware support (+memtag). For Armv8 or "
  "Armv9, try compiling with -march=armv8a+memtag or -march=armv9a+memtag">;

def err_cmse_pi_are_incompatible : Error<
  "cmse is not compatible with %select{RWPI|ROPI}0">;

def warn_target_unsupported_nan2008 : Warning<
  "ignoring '-mnan=2008' option because the '%0' architecture does not support it">,
  InGroup<UnsupportedNan>;
def warn_target_unsupported_nanlegacy : Warning<
  "ignoring '-mnan=legacy' option because the '%0' architecture does not support it">,
  InGroup<UnsupportedNan>;
def warn_target_unsupported_abslegacy : Warning<
  "ignoring '-mabs=legacy' option because the '%0' architecture does not support it">,
  InGroup<UnsupportedAbs>;
def warn_target_unsupported_abs2008 : Warning<
````
- **L673 EN**: Adds a standalone statement or declaration: `"use '--' to treat subsequent arguments as filenames">;`.
  **L673 CN**: 添加一条独立语句或声明：`"use '--' to treat subsequent arguments as filenames">;`。
- **L674 EN**: Blank line separating nearby declarations or logic blocks.
  **L674 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L675 EN**: Declares TableGen def record `err_drv_ropi_rwpi_incompatible_with_pic`.
  **L675 CN**: 声明 TableGen def 记录 `err_drv_ropi_rwpi_incompatible_with_pic`。
- **L676 EN**: Adds a standalone statement or declaration: `"embedded and GOT-based position independence are incompatible">;`.
  **L676 CN**: 添加一条独立语句或声明：`"embedded and GOT-based position independence are incompatible">;`。
- **L677 EN**: Declares TableGen def record `err_drv_ropi_incompatible_with_cxx`.
  **L677 CN**: 声明 TableGen def 记录 `err_drv_ropi_incompatible_with_cxx`。
- **L678 EN**: Adds a standalone statement or declaration: `"ROPI is not compatible with c++">;`.
  **L678 CN**: 添加一条独立语句或声明：`"ROPI is not compatible with c++">;`。
- **L679 EN**: Blank line separating nearby declarations or logic blocks.
  **L679 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L680 EN**: Declares TableGen def record `err_stack_tagging_requires_hardware_feature`.
  **L680 CN**: 声明 TableGen def 记录 `err_stack_tagging_requires_hardware_feature`。
- **L681 EN**: Continues logic associated with callable symbol `support`.
  **L681 CN**: 继续与可调用符号 `support` 相关的逻辑。
- **L682 EN**: Adds a standalone statement or declaration: `"Armv9, try compiling with -march=armv8a+memtag or -march=armv9a+memtag">;`.
  **L682 CN**: 添加一条独立语句或声明：`"Armv9, try compiling with -march=armv8a+memtag or -march=armv9a+memtag">;`。
- **L683 EN**: Blank line separating nearby declarations or logic blocks.
  **L683 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L684 EN**: Declares TableGen def record `err_cmse_pi_are_incompatible`.
  **L684 CN**: 声明 TableGen def 记录 `err_cmse_pi_are_incompatible`。
- **L685 EN**: Adds a standalone statement or declaration: `"cmse is not compatible with %select{RWPI|ROPI}0">;`.
  **L685 CN**: 添加一条独立语句或声明：`"cmse is not compatible with %select{RWPI|ROPI}0">;`。
- **L686 EN**: Blank line separating nearby declarations or logic blocks.
  **L686 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L687 EN**: Declares TableGen def record `warn_target_unsupported_nan2008`.
  **L687 CN**: 声明 TableGen def 记录 `warn_target_unsupported_nan2008`。
- **L688 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '-mnan=2008' option because the '%0' architecture does not support it">,`.
  **L688 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '-mnan=2008' option because the '%0' architecture does not support it">,`。
- **L689 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedNan>;`.
  **L689 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedNan>;`。
- **L690 EN**: Declares TableGen def record `warn_target_unsupported_nanlegacy`.
  **L690 CN**: 声明 TableGen def 记录 `warn_target_unsupported_nanlegacy`。
- **L691 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '-mnan=legacy' option because the '%0' architecture does not support it">,`.
  **L691 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '-mnan=legacy' option because the '%0' architecture does not support it">,`。
- **L692 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedNan>;`.
  **L692 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedNan>;`。
- **L693 EN**: Declares TableGen def record `warn_target_unsupported_abslegacy`.
  **L693 CN**: 声明 TableGen def 记录 `warn_target_unsupported_abslegacy`。
- **L694 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '-mabs=legacy' option because the '%0' architecture does not support it">,`.
  **L694 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '-mabs=legacy' option because the '%0' architecture does not support it">,`。
- **L695 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedAbs>;`.
  **L695 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedAbs>;`。
- **L696 EN**: Declares TableGen def record `warn_target_unsupported_abs2008`.
  **L696 CN**: 声明 TableGen def 记录 `warn_target_unsupported_abs2008`。

### Lines 697-720

````tablegen
  "ignoring '-mabs=2008' option because the '%0' architecture does not support it">,
  InGroup<UnsupportedAbs>;
def warn_target_unsupported_compact_branches : Warning<
  "ignoring '-mcompact-branches=' option because the '%0' architecture does not"
  " support it">, InGroup<UnsupportedCB>;
def warn_target_unsupported_extension : Warning<
  "ignoring extension '%0' because the '%1' architecture does not support it">,
   InGroup<InvalidCommandLineArgument>;
def warn_drv_unsupported_gpopt : Warning<
  "ignoring '-mgpopt' option as it cannot be used with %select{|the implicit"
  " usage of }0-mabicalls">,
  InGroup<UnsupportedGPOpt>;
def warn_drv_unsupported_longcalls : Warning<
  "ignoring '-mlong-calls' option as it is not currently supported with "
  "%select{|the implicit usage of }0-mabicalls">,
  InGroup<OptionIgnored>;
def warn_drv_unsupported_pic_with_mabicalls : Warning<
  "ignoring '%0' option as it cannot be used with "
  "%select{implicit usage of|}1 -mabicalls and the N64 ABI">,
  InGroup<OptionIgnored>;
def err_drv_unsupported_noabicalls_pic : Error<
  "position-independent code requires '-mabicalls'">;
def err_drv_unsupported_indirect_jump_opt : Error<
  "'-mindirect-jump=%0' is unsupported with the '%1' architecture">;
````
- **L697 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '-mabs=2008' option because the '%0' architecture does not support it">,`.
  **L697 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '-mabs=2008' option because the '%0' architecture does not support it">,`。
- **L698 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedAbs>;`.
  **L698 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedAbs>;`。
- **L699 EN**: Declares TableGen def record `warn_target_unsupported_compact_branches`.
  **L699 CN**: 声明 TableGen def 记录 `warn_target_unsupported_compact_branches`。
- **L700 EN**: Continues the surrounding expression or declaration: `"ignoring '-mcompact-branches=' option because the '%0' architecture does not"`.
  **L700 CN**: 继续构造周围的表达式或声明：`"ignoring '-mcompact-branches=' option because the '%0' architecture does not"`。
- **L701 EN**: Adds a standalone statement or declaration: `" support it">, InGroup<UnsupportedCB>;`.
  **L701 CN**: 添加一条独立语句或声明：`" support it">, InGroup<UnsupportedCB>;`。
- **L702 EN**: Declares TableGen def record `warn_target_unsupported_extension`.
  **L702 CN**: 声明 TableGen def 记录 `warn_target_unsupported_extension`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring extension '%0' because the '%1' architecture does not support it">,`.
  **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring extension '%0' because the '%1' architecture does not support it">,`。
- **L704 EN**: Adds a standalone statement or declaration: `InGroup<InvalidCommandLineArgument>;`.
  **L704 CN**: 添加一条独立语句或声明：`InGroup<InvalidCommandLineArgument>;`。
- **L705 EN**: Declares TableGen def record `warn_drv_unsupported_gpopt`.
  **L705 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_gpopt`。
- **L706 EN**: Continues the surrounding expression or declaration: `"ignoring '-mgpopt' option as it cannot be used with %select{|the implicit"`.
  **L706 CN**: 继续构造周围的表达式或声明：`"ignoring '-mgpopt' option as it cannot be used with %select{|the implicit"`。
- **L707 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `" usage of }0-mabicalls">,`.
  **L707 CN**: 继续一个多行参数列表、初始化器或聚合项：`" usage of }0-mabicalls">,`。
- **L708 EN**: Adds a standalone statement or declaration: `InGroup<UnsupportedGPOpt>;`.
  **L708 CN**: 添加一条独立语句或声明：`InGroup<UnsupportedGPOpt>;`。
- **L709 EN**: Declares TableGen def record `warn_drv_unsupported_longcalls`.
  **L709 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_longcalls`。
- **L710 EN**: Continues the surrounding expression or declaration: `"ignoring '-mlong-calls' option as it is not currently supported with "`.
  **L710 CN**: 继续构造周围的表达式或声明：`"ignoring '-mlong-calls' option as it is not currently supported with "`。
- **L711 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{|the implicit usage of }0-mabicalls">,`.
  **L711 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{|the implicit usage of }0-mabicalls">,`。
- **L712 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L712 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L713 EN**: Declares TableGen def record `warn_drv_unsupported_pic_with_mabicalls`.
  **L713 CN**: 声明 TableGen def 记录 `warn_drv_unsupported_pic_with_mabicalls`。
- **L714 EN**: Continues the surrounding expression or declaration: `"ignoring '%0' option as it cannot be used with "`.
  **L714 CN**: 继续构造周围的表达式或声明：`"ignoring '%0' option as it cannot be used with "`。
- **L715 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"%select{implicit usage of|}1 -mabicalls and the N64 ABI">,`.
  **L715 CN**: 继续一个多行参数列表、初始化器或聚合项：`"%select{implicit usage of|}1 -mabicalls and the N64 ABI">,`。
- **L716 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L716 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L717 EN**: Declares TableGen def record `err_drv_unsupported_noabicalls_pic`.
  **L717 CN**: 声明 TableGen def 记录 `err_drv_unsupported_noabicalls_pic`。
- **L718 EN**: Adds a standalone statement or declaration: `"position-independent code requires '-mabicalls'">;`.
  **L718 CN**: 添加一条独立语句或声明：`"position-independent code requires '-mabicalls'">;`。
- **L719 EN**: Declares TableGen def record `err_drv_unsupported_indirect_jump_opt`.
  **L719 CN**: 声明 TableGen def 记录 `err_drv_unsupported_indirect_jump_opt`。
- **L720 EN**: Adds a standalone statement or declaration: `"'-mindirect-jump=%0' is unsupported with the '%1' architecture">;`.
  **L720 CN**: 添加一条独立语句或声明：`"'-mindirect-jump=%0' is unsupported with the '%1' architecture">;`。

### Lines 721-744

````tablegen
def err_drv_unknown_indirect_jump_opt : Error<
  "unknown '-mindirect-jump=' option '%0'">;
def err_drv_unsupported_fpatchable_function_entry_argument : Error<
  "the second argument of '-fpatchable-function-entry' must be smaller than the first argument">;

def warn_drv_unable_to_find_directory_expected : Warning<
  "unable to find %0 directory, expected to be in '%1' found via %2">,
  InGroup<InvalidOrNonExistentDirectory>, DefaultIgnore;

def warn_drv_ps_force_pic : Warning<
  "option '%0' was ignored by the %1 toolchain, using '-fPIC'">,
  InGroup<OptionIgnored>;

def err_drv_defsym_invalid_format : Error<"defsym must be of the form: sym=value: %0">;
def err_drv_defsym_invalid_symval : Error<"value is not an integer: %0">;
def warn_drv_msvc_not_found : Warning<
  "unable to find a Visual Studio installation; "
  "try running Clang from a developer command prompt">,
  InGroup<DiagGroup<"msvc-not-found">>;

def warn_drv_fuse_ld_path : Warning<
  "'-fuse-ld=' taking a path is deprecated; use '--ld-path=' instead">,
  InGroup<FUseLdPath>, DefaultIgnore;

````
- **L721 EN**: Declares TableGen def record `err_drv_unknown_indirect_jump_opt`.
  **L721 CN**: 声明 TableGen def 记录 `err_drv_unknown_indirect_jump_opt`。
- **L722 EN**: Adds a standalone statement or declaration: `"unknown '-mindirect-jump=' option '%0'">;`.
  **L722 CN**: 添加一条独立语句或声明：`"unknown '-mindirect-jump=' option '%0'">;`。
- **L723 EN**: Declares TableGen def record `err_drv_unsupported_fpatchable_function_entry_argument`.
  **L723 CN**: 声明 TableGen def 记录 `err_drv_unsupported_fpatchable_function_entry_argument`。
- **L724 EN**: Adds a standalone statement or declaration: `"the second argument of '-fpatchable-function-entry' must be smaller than the first argument">;`.
  **L724 CN**: 添加一条独立语句或声明：`"the second argument of '-fpatchable-function-entry' must be smaller than the first argument">;`。
- **L725 EN**: Blank line separating nearby declarations or logic blocks.
  **L725 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L726 EN**: Declares TableGen def record `warn_drv_unable_to_find_directory_expected`.
  **L726 CN**: 声明 TableGen def 记录 `warn_drv_unable_to_find_directory_expected`。
- **L727 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"unable to find %0 directory, expected to be in '%1' found via %2">,`.
  **L727 CN**: 继续一个多行参数列表、初始化器或聚合项：`"unable to find %0 directory, expected to be in '%1' found via %2">,`。
- **L728 EN**: Adds a standalone statement or declaration: `InGroup<InvalidOrNonExistentDirectory>, DefaultIgnore;`.
  **L728 CN**: 添加一条独立语句或声明：`InGroup<InvalidOrNonExistentDirectory>, DefaultIgnore;`。
- **L729 EN**: Blank line separating nearby declarations or logic blocks.
  **L729 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L730 EN**: Declares TableGen def record `warn_drv_ps_force_pic`.
  **L730 CN**: 声明 TableGen def 记录 `warn_drv_ps_force_pic`。
- **L731 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"option '%0' was ignored by the %1 toolchain, using '-fPIC'">,`.
  **L731 CN**: 继续一个多行参数列表、初始化器或聚合项：`"option '%0' was ignored by the %1 toolchain, using '-fPIC'">,`。
- **L732 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L732 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L733 EN**: Blank line separating nearby declarations or logic blocks.
  **L733 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L734 EN**: Declares TableGen def record `err_drv_defsym_invalid_format`.
  **L734 CN**: 声明 TableGen def 记录 `err_drv_defsym_invalid_format`。
- **L735 EN**: Declares TableGen def record `err_drv_defsym_invalid_symval`.
  **L735 CN**: 声明 TableGen def 记录 `err_drv_defsym_invalid_symval`。
- **L736 EN**: Declares TableGen def record `warn_drv_msvc_not_found`.
  **L736 CN**: 声明 TableGen def 记录 `warn_drv_msvc_not_found`。
- **L737 EN**: Continues the surrounding expression or declaration: `"unable to find a Visual Studio installation; "`.
  **L737 CN**: 继续构造周围的表达式或声明：`"unable to find a Visual Studio installation; "`。
- **L738 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"try running Clang from a developer command prompt">,`.
  **L738 CN**: 继续一个多行参数列表、初始化器或聚合项：`"try running Clang from a developer command prompt">,`。
- **L739 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"msvc-not-found">>;`.
  **L739 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"msvc-not-found">>;`。
- **L740 EN**: Blank line separating nearby declarations or logic blocks.
  **L740 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L741 EN**: Declares TableGen def record `warn_drv_fuse_ld_path`.
  **L741 CN**: 声明 TableGen def 记录 `warn_drv_fuse_ld_path`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'-fuse-ld=' taking a path is deprecated; use '--ld-path=' instead">,`.
  **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'-fuse-ld=' taking a path is deprecated; use '--ld-path=' instead">,`。
- **L743 EN**: Adds a standalone statement or declaration: `InGroup<FUseLdPath>, DefaultIgnore;`.
  **L743 CN**: 添加一条独立语句或声明：`InGroup<FUseLdPath>, DefaultIgnore;`。
- **L744 EN**: Blank line separating nearby declarations or logic blocks.
  **L744 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 745-768

````tablegen
def warn_drv_fine_grained_bitfield_accesses_ignored : Warning<
  "option '-ffine-grained-bitfield-accesses' cannot be enabled together with a sanitizer; flag ignored">,
  InGroup<OptionIgnored>;

def err_drv_profile_instrument_use_path_with_no_kind : Error<
  "option '-fprofile-instrument-use-path=' requires -fprofile-instrument-use=<kind>">;

def note_drv_verify_prefix_spelling : Note<
  "-verify prefixes must start with a letter and contain only alphanumeric"
  " characters, hyphens, and underscores">;

def note_command_line_code_loc_requirement
    : Note<"-code-completion-at=<file>:<line>:<column> requires <line> and "
           "<column> to be integers greater than zero">;

def warn_drv_global_isel_incomplete : Warning<
  "-fglobal-isel support for the '%0' architecture is incomplete">,
  InGroup<GlobalISel>;

def warn_drv_global_isel_incomplete_opt : Warning<
  "-fglobal-isel support is incomplete for this architecture at the current optimization level">,
  InGroup<GlobalISel>;

def warn_drv_moutline_unsupported_opt : Warning<
````
- **L745 EN**: Declares TableGen def record `warn_drv_fine_grained_bitfield_accesses_ignored`.
  **L745 CN**: 声明 TableGen def 记录 `warn_drv_fine_grained_bitfield_accesses_ignored`。
- **L746 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"option '-ffine-grained-bitfield-accesses' cannot be enabled together with a sanitizer; flag ignored">,`.
  **L746 CN**: 继续一个多行参数列表、初始化器或聚合项：`"option '-ffine-grained-bitfield-accesses' cannot be enabled together with a sanitizer; flag ignored">,`。
- **L747 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L747 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L748 EN**: Blank line separating nearby declarations or logic blocks.
  **L748 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L749 EN**: Declares TableGen def record `err_drv_profile_instrument_use_path_with_no_kind`.
  **L749 CN**: 声明 TableGen def 记录 `err_drv_profile_instrument_use_path_with_no_kind`。
- **L750 EN**: Adds a standalone statement or declaration: `"option '-fprofile-instrument-use-path=' requires -fprofile-instrument-use=<kind>">;`.
  **L750 CN**: 添加一条独立语句或声明：`"option '-fprofile-instrument-use-path=' requires -fprofile-instrument-use=<kind>">;`。
- **L751 EN**: Blank line separating nearby declarations or logic blocks.
  **L751 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L752 EN**: Declares TableGen def record `note_drv_verify_prefix_spelling`.
  **L752 CN**: 声明 TableGen def 记录 `note_drv_verify_prefix_spelling`。
- **L753 EN**: Continues the surrounding expression or declaration: `"-verify prefixes must start with a letter and contain only alphanumeric"`.
  **L753 CN**: 继续构造周围的表达式或声明：`"-verify prefixes must start with a letter and contain only alphanumeric"`。
- **L754 EN**: Adds a standalone statement or declaration: `" characters, hyphens, and underscores">;`.
  **L754 CN**: 添加一条独立语句或声明：`" characters, hyphens, and underscores">;`。
- **L755 EN**: Blank line separating nearby declarations or logic blocks.
  **L755 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L756 EN**: Declares TableGen def record `note_command_line_code_loc_requirement`.
  **L756 CN**: 声明 TableGen def 记录 `note_command_line_code_loc_requirement`。
- **L757 EN**: Continues the surrounding expression or declaration: `: Note<"-code-completion-at=<file>:<line>:<column> requires <line> and "`.
  **L757 CN**: 继续构造周围的表达式或声明：`: Note<"-code-completion-at=<file>:<line>:<column> requires <line> and "`。
- **L758 EN**: Adds a standalone statement or declaration: `"<column> to be integers greater than zero">;`.
  **L758 CN**: 添加一条独立语句或声明：`"<column> to be integers greater than zero">;`。
- **L759 EN**: Blank line separating nearby declarations or logic blocks.
  **L759 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L760 EN**: Declares TableGen def record `warn_drv_global_isel_incomplete`.
  **L760 CN**: 声明 TableGen def 记录 `warn_drv_global_isel_incomplete`。
- **L761 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"-fglobal-isel support for the '%0' architecture is incomplete">,`.
  **L761 CN**: 继续一个多行参数列表、初始化器或聚合项：`"-fglobal-isel support for the '%0' architecture is incomplete">,`。
- **L762 EN**: Adds a standalone statement or declaration: `InGroup<GlobalISel>;`.
  **L762 CN**: 添加一条独立语句或声明：`InGroup<GlobalISel>;`。
- **L763 EN**: Blank line separating nearby declarations or logic blocks.
  **L763 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L764 EN**: Declares TableGen def record `warn_drv_global_isel_incomplete_opt`.
  **L764 CN**: 声明 TableGen def 记录 `warn_drv_global_isel_incomplete_opt`。
- **L765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"-fglobal-isel support is incomplete for this architecture at the current optimization level">,`.
  **L765 CN**: 继续一个多行参数列表、初始化器或聚合项：`"-fglobal-isel support is incomplete for this architecture at the current optimization level">,`。
- **L766 EN**: Adds a standalone statement or declaration: `InGroup<GlobalISel>;`.
  **L766 CN**: 添加一条独立语句或声明：`InGroup<GlobalISel>;`。
- **L767 EN**: Blank line separating nearby declarations or logic blocks.
  **L767 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L768 EN**: Declares TableGen def record `warn_drv_moutline_unsupported_opt`.
  **L768 CN**: 声明 TableGen def 记录 `warn_drv_moutline_unsupported_opt`。

### Lines 769-792

````tablegen
  "'%0' does not support '-moutline'; flag ignored">,
  InGroup<OptionIgnored>;

def warn_drv_moutline_atomics_unsupported_opt : Warning<
  "'%0' does not support '-%1'; flag ignored">,
  InGroup<OptionIgnored>;

def warn_drv_darwin_sdk_invalid_settings : Warning<
  "SDK settings were ignored as 'SDKSettings.json' could not be parsed">,
  InGroup<DiagGroup<"darwin-sdk-settings">>;

def err_missing_sysroot : Error<"no such sysroot directory: '%0'">;
def err_drv_darwin_sdk_missing_arclite : Error<
  "SDK does not contain 'libarclite' at the path '%0'; try increasing the minimum deployment target">;

def err_drv_trivial_auto_var_init_stop_after_missing_dependency : Error<
  "'-ftrivial-auto-var-init-stop-after=*' is used without "
  "'-ftrivial-auto-var-init=zero' or '-ftrivial-auto-var-init=pattern'">;

def err_drv_trivial_auto_var_init_stop_after_invalid_value : Error<
  "'-ftrivial-auto-var-init-stop-after=*' only accepts positive integers">;

def err_drv_trivial_auto_var_init_max_size_missing_dependency : Error<
  "'-ftrivial-auto-var-init-max-size=*' is used without "
````
- **L769 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' does not support '-moutline'; flag ignored">,`.
  **L769 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' does not support '-moutline'; flag ignored">,`。
- **L770 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L770 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L771 EN**: Blank line separating nearby declarations or logic blocks.
  **L771 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L772 EN**: Declares TableGen def record `warn_drv_moutline_atomics_unsupported_opt`.
  **L772 CN**: 声明 TableGen def 记录 `warn_drv_moutline_atomics_unsupported_opt`。
- **L773 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"'%0' does not support '-%1'; flag ignored">,`.
  **L773 CN**: 继续一个多行参数列表、初始化器或聚合项：`"'%0' does not support '-%1'; flag ignored">,`。
- **L774 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L774 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L775 EN**: Blank line separating nearby declarations or logic blocks.
  **L775 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L776 EN**: Declares TableGen def record `warn_drv_darwin_sdk_invalid_settings`.
  **L776 CN**: 声明 TableGen def 记录 `warn_drv_darwin_sdk_invalid_settings`。
- **L777 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"SDK settings were ignored as 'SDKSettings.json' could not be parsed">,`.
  **L777 CN**: 继续一个多行参数列表、初始化器或聚合项：`"SDK settings were ignored as 'SDKSettings.json' could not be parsed">,`。
- **L778 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"darwin-sdk-settings">>;`.
  **L778 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"darwin-sdk-settings">>;`。
- **L779 EN**: Blank line separating nearby declarations or logic blocks.
  **L779 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L780 EN**: Declares TableGen def record `err_missing_sysroot`.
  **L780 CN**: 声明 TableGen def 记录 `err_missing_sysroot`。
- **L781 EN**: Declares TableGen def record `err_drv_darwin_sdk_missing_arclite`.
  **L781 CN**: 声明 TableGen def 记录 `err_drv_darwin_sdk_missing_arclite`。
- **L782 EN**: Adds a standalone statement or declaration: `"SDK does not contain 'libarclite' at the path '%0'; try increasing the minimum deployment target">;`.
  **L782 CN**: 添加一条独立语句或声明：`"SDK does not contain 'libarclite' at the path '%0'; try increasing the minimum deployment target">;`。
- **L783 EN**: Blank line separating nearby declarations or logic blocks.
  **L783 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L784 EN**: Declares TableGen def record `err_drv_trivial_auto_var_init_stop_after_missing_dependency`.
  **L784 CN**: 声明 TableGen def 记录 `err_drv_trivial_auto_var_init_stop_after_missing_dependency`。
- **L785 EN**: Continues the surrounding expression or declaration: `"'-ftrivial-auto-var-init-stop-after=*' is used without "`.
  **L785 CN**: 继续构造周围的表达式或声明：`"'-ftrivial-auto-var-init-stop-after=*' is used without "`。
- **L786 EN**: Adds a standalone statement or declaration: `"'-ftrivial-auto-var-init=zero' or '-ftrivial-auto-var-init=pattern'">;`.
  **L786 CN**: 添加一条独立语句或声明：`"'-ftrivial-auto-var-init=zero' or '-ftrivial-auto-var-init=pattern'">;`。
- **L787 EN**: Blank line separating nearby declarations or logic blocks.
  **L787 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L788 EN**: Declares TableGen def record `err_drv_trivial_auto_var_init_stop_after_invalid_value`.
  **L788 CN**: 声明 TableGen def 记录 `err_drv_trivial_auto_var_init_stop_after_invalid_value`。
- **L789 EN**: Adds a standalone statement or declaration: `"'-ftrivial-auto-var-init-stop-after=*' only accepts positive integers">;`.
  **L789 CN**: 添加一条独立语句或声明：`"'-ftrivial-auto-var-init-stop-after=*' only accepts positive integers">;`。
- **L790 EN**: Blank line separating nearby declarations or logic blocks.
  **L790 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L791 EN**: Declares TableGen def record `err_drv_trivial_auto_var_init_max_size_missing_dependency`.
  **L791 CN**: 声明 TableGen def 记录 `err_drv_trivial_auto_var_init_max_size_missing_dependency`。
- **L792 EN**: Continues the surrounding expression or declaration: `"'-ftrivial-auto-var-init-max-size=*' is used without "`.
  **L792 CN**: 继续构造周围的表达式或声明：`"'-ftrivial-auto-var-init-max-size=*' is used without "`。

### Lines 793-816

````tablegen
  "'-ftrivial-auto-var-init=zero' or '-ftrivial-auto-var-init=pattern'">;

def err_drv_trivial_auto_var_init_max_size_invalid_value : Error<
  "'-ftrivial-auto-var-init-max-size=*' only accepts positive integers (in bytes)">;

def warn_drv_msp430_hwmult_unsupported : Warning<
  "the given MCU does not support hardware multiply, but '-mhwmult' is set to "
  "%0">, InGroup<InvalidCommandLineArgument>;
def warn_drv_msp430_hwmult_mismatch : Warning<
  "the given MCU supports %0 hardware multiply, but '-mhwmult' is set to %1">,
   InGroup<InvalidCommandLineArgument>;
def warn_drv_msp430_hwmult_no_device : Warning<
  "no MCU device specified, but '-mhwmult' is set to 'auto', assuming no "
  "hardware multiply; use '-mmcu' to specify an MSP430 device, or '-mhwmult' "
  "to set the hardware multiply type explicitly">,
  InGroup<InvalidCommandLineArgument>;

def warn_drv_libstdcxx_not_found : Warning<
  "include path for libstdc++ headers not found; pass '-stdlib=libc++' on the "
  "command line to use the libc++ standard library instead">,
  InGroup<DiagGroup<"stdlibcxx-not-found">>;

def err_drv_cannot_mix_options : Error<"cannot specify '%1' along with '%0'">;

````
- **L793 EN**: Adds a standalone statement or declaration: `"'-ftrivial-auto-var-init=zero' or '-ftrivial-auto-var-init=pattern'">;`.
  **L793 CN**: 添加一条独立语句或声明：`"'-ftrivial-auto-var-init=zero' or '-ftrivial-auto-var-init=pattern'">;`。
- **L794 EN**: Blank line separating nearby declarations or logic blocks.
  **L794 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L795 EN**: Declares TableGen def record `err_drv_trivial_auto_var_init_max_size_invalid_value`.
  **L795 CN**: 声明 TableGen def 记录 `err_drv_trivial_auto_var_init_max_size_invalid_value`。
- **L796 EN**: Executes a call or declaration centered on `integers`.
  **L796 CN**: 执行以 `integers` 为核心的调用或声明。
- **L797 EN**: Blank line separating nearby declarations or logic blocks.
  **L797 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L798 EN**: Declares TableGen def record `warn_drv_msp430_hwmult_unsupported`.
  **L798 CN**: 声明 TableGen def 记录 `warn_drv_msp430_hwmult_unsupported`。
- **L799 EN**: Continues the surrounding expression or declaration: `"the given MCU does not support hardware multiply, but '-mhwmult' is set to "`.
  **L799 CN**: 继续构造周围的表达式或声明：`"the given MCU does not support hardware multiply, but '-mhwmult' is set to "`。
- **L800 EN**: Adds a standalone statement or declaration: `"%0">, InGroup<InvalidCommandLineArgument>;`.
  **L800 CN**: 添加一条独立语句或声明：`"%0">, InGroup<InvalidCommandLineArgument>;`。
- **L801 EN**: Declares TableGen def record `warn_drv_msp430_hwmult_mismatch`.
  **L801 CN**: 声明 TableGen def 记录 `warn_drv_msp430_hwmult_mismatch`。
- **L802 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"the given MCU supports %0 hardware multiply, but '-mhwmult' is set to %1">,`.
  **L802 CN**: 继续一个多行参数列表、初始化器或聚合项：`"the given MCU supports %0 hardware multiply, but '-mhwmult' is set to %1">,`。
- **L803 EN**: Adds a standalone statement or declaration: `InGroup<InvalidCommandLineArgument>;`.
  **L803 CN**: 添加一条独立语句或声明：`InGroup<InvalidCommandLineArgument>;`。
- **L804 EN**: Declares TableGen def record `warn_drv_msp430_hwmult_no_device`.
  **L804 CN**: 声明 TableGen def 记录 `warn_drv_msp430_hwmult_no_device`。
- **L805 EN**: Continues the surrounding expression or declaration: `"no MCU device specified, but '-mhwmult' is set to 'auto', assuming no "`.
  **L805 CN**: 继续构造周围的表达式或声明：`"no MCU device specified, but '-mhwmult' is set to 'auto', assuming no "`。
- **L806 EN**: Continues the surrounding expression or declaration: `"hardware multiply; use '-mmcu' to specify an MSP430 device, or '-mhwmult' "`.
  **L806 CN**: 继续构造周围的表达式或声明：`"hardware multiply; use '-mmcu' to specify an MSP430 device, or '-mhwmult' "`。
- **L807 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"to set the hardware multiply type explicitly">,`.
  **L807 CN**: 继续一个多行参数列表、初始化器或聚合项：`"to set the hardware multiply type explicitly">,`。
- **L808 EN**: Adds a standalone statement or declaration: `InGroup<InvalidCommandLineArgument>;`.
  **L808 CN**: 添加一条独立语句或声明：`InGroup<InvalidCommandLineArgument>;`。
- **L809 EN**: Blank line separating nearby declarations or logic blocks.
  **L809 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L810 EN**: Declares TableGen def record `warn_drv_libstdcxx_not_found`.
  **L810 CN**: 声明 TableGen def 记录 `warn_drv_libstdcxx_not_found`。
- **L811 EN**: Continues the surrounding expression or declaration: `"include path for libstdc++ headers not found; pass '-stdlib=libc++' on the "`.
  **L811 CN**: 继续构造周围的表达式或声明：`"include path for libstdc++ headers not found; pass '-stdlib=libc++' on the "`。
- **L812 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"command line to use the libc++ standard library instead">,`.
  **L812 CN**: 继续一个多行参数列表、初始化器或聚合项：`"command line to use the libc++ standard library instead">,`。
- **L813 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"stdlibcxx-not-found">>;`.
  **L813 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"stdlibcxx-not-found">>;`。
- **L814 EN**: Blank line separating nearby declarations or logic blocks.
  **L814 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L815 EN**: Declares TableGen def record `err_drv_cannot_mix_options`.
  **L815 CN**: 声明 TableGen def 记录 `err_drv_cannot_mix_options`。
- **L816 EN**: Blank line separating nearby declarations or logic blocks.
  **L816 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 817-840

````tablegen
def err_drv_invalid_object_mode : Error<
  "OBJECT_MODE setting %0 is not recognized and is not a valid setting">;

def err_roptr_requires_data_sections: Error<"-mxcoff-roptr is supported only with -fdata-sections">;
def err_roptr_cannot_build_shared: Error<"-mxcoff-roptr is not supported with -shared">;

def err_invalid_cxx_abi : Error<"invalid C++ ABI name '%0'">;
def err_unsupported_cxx_abi : Error<"C++ ABI '%0' is not supported on target triple '%1'">;

def note_cc1_round_trip_original : Note<"original arguments in round-trip: %0">;
def note_cc1_round_trip_generated : Note<
  "generated arguments #%0 in round-trip: %1">;
def remark_cc1_round_trip_generated : Remark<
  "generated arguments #%0 in round-trip: %1">, InGroup<RoundTripCC1Args>;
def err_cc1_round_trip_fail_then_ok : Error<
  "original arguments parse failed, then succeeded in round-trip">;
def err_cc1_round_trip_ok_then_fail : Error<
  "generated arguments parse failed in round-trip">;
def err_cc1_round_trip_mismatch : Error<
  "generated arguments do not match in round-trip">;
def err_cc1_unbounded_vscale_min : Error<
  "minimum vscale must be an unsigned integer greater than 0">;

def err_conflicting_matrix_layout_flags: Error<
````
- **L817 EN**: Declares TableGen def record `err_drv_invalid_object_mode`.
  **L817 CN**: 声明 TableGen def 记录 `err_drv_invalid_object_mode`。
- **L818 EN**: Adds a standalone statement or declaration: `"OBJECT_MODE setting %0 is not recognized and is not a valid setting">;`.
  **L818 CN**: 添加一条独立语句或声明：`"OBJECT_MODE setting %0 is not recognized and is not a valid setting">;`。
- **L819 EN**: Blank line separating nearby declarations or logic blocks.
  **L819 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L820 EN**: Declares TableGen def record `err_roptr_requires_data_sections`.
  **L820 CN**: 声明 TableGen def 记录 `err_roptr_requires_data_sections`。
- **L821 EN**: Declares TableGen def record `err_roptr_cannot_build_shared`.
  **L821 CN**: 声明 TableGen def 记录 `err_roptr_cannot_build_shared`。
- **L822 EN**: Blank line separating nearby declarations or logic blocks.
  **L822 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L823 EN**: Declares TableGen def record `err_invalid_cxx_abi`.
  **L823 CN**: 声明 TableGen def 记录 `err_invalid_cxx_abi`。
- **L824 EN**: Declares TableGen def record `err_unsupported_cxx_abi`.
  **L824 CN**: 声明 TableGen def 记录 `err_unsupported_cxx_abi`。
- **L825 EN**: Blank line separating nearby declarations or logic blocks.
  **L825 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L826 EN**: Declares TableGen def record `note_cc1_round_trip_original`.
  **L826 CN**: 声明 TableGen def 记录 `note_cc1_round_trip_original`。
- **L827 EN**: Declares TableGen def record `note_cc1_round_trip_generated`.
  **L827 CN**: 声明 TableGen def 记录 `note_cc1_round_trip_generated`。
- **L828 EN**: Adds a standalone statement or declaration: `"generated arguments #%0 in round-trip: %1">;`.
  **L828 CN**: 添加一条独立语句或声明：`"generated arguments #%0 in round-trip: %1">;`。
- **L829 EN**: Declares TableGen def record `remark_cc1_round_trip_generated`.
  **L829 CN**: 声明 TableGen def 记录 `remark_cc1_round_trip_generated`。
- **L830 EN**: Adds a standalone statement or declaration: `"generated arguments #%0 in round-trip: %1">, InGroup<RoundTripCC1Args>;`.
  **L830 CN**: 添加一条独立语句或声明：`"generated arguments #%0 in round-trip: %1">, InGroup<RoundTripCC1Args>;`。
- **L831 EN**: Declares TableGen def record `err_cc1_round_trip_fail_then_ok`.
  **L831 CN**: 声明 TableGen def 记录 `err_cc1_round_trip_fail_then_ok`。
- **L832 EN**: Adds a standalone statement or declaration: `"original arguments parse failed, then succeeded in round-trip">;`.
  **L832 CN**: 添加一条独立语句或声明：`"original arguments parse failed, then succeeded in round-trip">;`。
- **L833 EN**: Declares TableGen def record `err_cc1_round_trip_ok_then_fail`.
  **L833 CN**: 声明 TableGen def 记录 `err_cc1_round_trip_ok_then_fail`。
- **L834 EN**: Adds a standalone statement or declaration: `"generated arguments parse failed in round-trip">;`.
  **L834 CN**: 添加一条独立语句或声明：`"generated arguments parse failed in round-trip">;`。
- **L835 EN**: Declares TableGen def record `err_cc1_round_trip_mismatch`.
  **L835 CN**: 声明 TableGen def 记录 `err_cc1_round_trip_mismatch`。
- **L836 EN**: Adds a standalone statement or declaration: `"generated arguments do not match in round-trip">;`.
  **L836 CN**: 添加一条独立语句或声明：`"generated arguments do not match in round-trip">;`。
- **L837 EN**: Declares TableGen def record `err_cc1_unbounded_vscale_min`.
  **L837 CN**: 声明 TableGen def 记录 `err_cc1_unbounded_vscale_min`。
- **L838 EN**: Adds a standalone statement or declaration: `"minimum vscale must be an unsigned integer greater than 0">;`.
  **L838 CN**: 添加一条独立语句或声明：`"minimum vscale must be an unsigned integer greater than 0">;`。
- **L839 EN**: Blank line separating nearby declarations or logic blocks.
  **L839 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L840 EN**: Declares TableGen def record `err_conflicting_matrix_layout_flags`.
  **L840 CN**: 声明 TableGen def 记录 `err_conflicting_matrix_layout_flags`。

### Lines 841-864

````tablegen
  "-fmatrix-memory-layout=%0 conflicts with -mllvm -matrix-default-layout=%1">;

def err_drv_using_omit_rtti_component_without_no_rtti : Error<
  "-fexperimental-omit-vtable-rtti call only be used with -fno-rtti">;

def err_drv_ssp_missing_offset_argument : Error<
  "'%0' is used without '-mstack-protector-guard-offset', and there is no default">;

def err_drv_only_one_offload_target_supported : Error<
  "only one offload target is supported">;
def err_drv_invalid_or_unsupported_offload_target : Error<
  "invalid or unsupported offload target: '%0'">;
def err_drv_cuda_offload_only_emit_bc : Error<
  "CUDA offload target is supported only along with --emit-llvm">;

def warn_drv_jmc_requires_debuginfo : Warning<
  "%0 requires debug info. Use %1 or debug options that enable debugger's "
  "stepping function; option ignored">,
  InGroup<OptionIgnored>;

def warn_drv_fjmc_for_elf_only : Warning<
  "-fjmc works only for ELF; option ignored">,
  InGroup<OptionIgnored>;

````
- **L841 EN**: Adds a standalone statement or declaration: `"-fmatrix-memory-layout=%0 conflicts with -mllvm -matrix-default-layout=%1">;`.
  **L841 CN**: 添加一条独立语句或声明：`"-fmatrix-memory-layout=%0 conflicts with -mllvm -matrix-default-layout=%1">;`。
- **L842 EN**: Blank line separating nearby declarations or logic blocks.
  **L842 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L843 EN**: Declares TableGen def record `err_drv_using_omit_rtti_component_without_no_rtti`.
  **L843 CN**: 声明 TableGen def 记录 `err_drv_using_omit_rtti_component_without_no_rtti`。
- **L844 EN**: Adds a standalone statement or declaration: `"-fexperimental-omit-vtable-rtti call only be used with -fno-rtti">;`.
  **L844 CN**: 添加一条独立语句或声明：`"-fexperimental-omit-vtable-rtti call only be used with -fno-rtti">;`。
- **L845 EN**: Blank line separating nearby declarations or logic blocks.
  **L845 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L846 EN**: Declares TableGen def record `err_drv_ssp_missing_offset_argument`.
  **L846 CN**: 声明 TableGen def 记录 `err_drv_ssp_missing_offset_argument`。
- **L847 EN**: Adds a standalone statement or declaration: `"'%0' is used without '-mstack-protector-guard-offset', and there is no default">;`.
  **L847 CN**: 添加一条独立语句或声明：`"'%0' is used without '-mstack-protector-guard-offset', and there is no default">;`。
- **L848 EN**: Blank line separating nearby declarations or logic blocks.
  **L848 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L849 EN**: Declares TableGen def record `err_drv_only_one_offload_target_supported`.
  **L849 CN**: 声明 TableGen def 记录 `err_drv_only_one_offload_target_supported`。
- **L850 EN**: Adds a standalone statement or declaration: `"only one offload target is supported">;`.
  **L850 CN**: 添加一条独立语句或声明：`"only one offload target is supported">;`。
- **L851 EN**: Declares TableGen def record `err_drv_invalid_or_unsupported_offload_target`.
  **L851 CN**: 声明 TableGen def 记录 `err_drv_invalid_or_unsupported_offload_target`。
- **L852 EN**: Adds a standalone statement or declaration: `"invalid or unsupported offload target: '%0'">;`.
  **L852 CN**: 添加一条独立语句或声明：`"invalid or unsupported offload target: '%0'">;`。
- **L853 EN**: Declares TableGen def record `err_drv_cuda_offload_only_emit_bc`.
  **L853 CN**: 声明 TableGen def 记录 `err_drv_cuda_offload_only_emit_bc`。
- **L854 EN**: Adds a standalone statement or declaration: `"CUDA offload target is supported only along with --emit-llvm">;`.
  **L854 CN**: 添加一条独立语句或声明：`"CUDA offload target is supported only along with --emit-llvm">;`。
- **L855 EN**: Blank line separating nearby declarations or logic blocks.
  **L855 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L856 EN**: Declares TableGen def record `warn_drv_jmc_requires_debuginfo`.
  **L856 CN**: 声明 TableGen def 记录 `warn_drv_jmc_requires_debuginfo`。
- **L857 EN**: Continues the surrounding expression or declaration: `"%0 requires debug info. Use %1 or debug options that enable debugger's "`.
  **L857 CN**: 继续构造周围的表达式或声明：`"%0 requires debug info. Use %1 or debug options that enable debugger's "`。
- **L858 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"stepping function; option ignored">,`.
  **L858 CN**: 继续一个多行参数列表、初始化器或聚合项：`"stepping function; option ignored">,`。
- **L859 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L859 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L860 EN**: Blank line separating nearby declarations or logic blocks.
  **L860 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L861 EN**: Declares TableGen def record `warn_drv_fjmc_for_elf_only`.
  **L861 CN**: 声明 TableGen def 记录 `warn_drv_fjmc_for_elf_only`。
- **L862 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"-fjmc works only for ELF; option ignored">,`.
  **L862 CN**: 继续一个多行参数列表、初始化器或聚合项：`"-fjmc works only for ELF; option ignored">,`。
- **L863 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L863 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L864 EN**: Blank line separating nearby declarations or logic blocks.
  **L864 CN**: 空行，用于分隔附近的声明或逻辑块。

### Lines 865-888

````tablegen
def warn_target_override_arm64ec : Warning<
  "/arm64EC has been overridden by specified target: %0; option ignored">,
  InGroup<OptionIgnored>;

def err_drv_target_variant_invalid : Error<
  "unsupported '%0' value '%1'; use 'ios-macabi' instead">;

def err_drv_invalid_directx_shader_module : Error<
  "invalid profile : %0">;
def err_drv_dxc_invalid_matrix_layout : Error<
  "cannot specify /Zpr and /Zpc together">;
def err_drv_dxc_missing_target_profile : Error<
  "target profile option (-T) is missing">;
def err_drv_dxc_Fre_requires_Fo_metal
    : Error<"-Fre option requires -Fo option when targeting Metal">;
def err_drv_hlsl_unsupported_target : Error<
  "HLSL code generation is unsupported for target '%0'">;
def err_drv_hlsl_bad_shader_required_in_target : Error<
  "%select{shader model|Vulkan environment|shader stage}0 is required as %select{OS|environment}1 in target '%2' for HLSL code generation">;
def err_drv_hlsl_16bit_types_unsupported: Error<
  "'%0' option requires target HLSL Version >= 2018%select{| and shader model >= 6.2}1, but HLSL Version is '%2'%select{| and shader model is '%3'}1">;
def err_drv_hlsl_bad_shader_unsupported : Error<
  "%select{shader model|Vulkan environment|shader stage}0 '%1' in target '%2' is invalid for HLSL code generation">;
def warn_drv_dxc_missing_dxv : Warning<
````
- **L865 EN**: Declares TableGen def record `warn_target_override_arm64ec`.
  **L865 CN**: 声明 TableGen def 记录 `warn_target_override_arm64ec`。
- **L866 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"/arm64EC has been overridden by specified target: %0; option ignored">,`.
  **L866 CN**: 继续一个多行参数列表、初始化器或聚合项：`"/arm64EC has been overridden by specified target: %0; option ignored">,`。
- **L867 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L867 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L868 EN**: Blank line separating nearby declarations or logic blocks.
  **L868 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L869 EN**: Declares TableGen def record `err_drv_target_variant_invalid`.
  **L869 CN**: 声明 TableGen def 记录 `err_drv_target_variant_invalid`。
- **L870 EN**: Adds a standalone statement or declaration: `"unsupported '%0' value '%1'; use 'ios-macabi' instead">;`.
  **L870 CN**: 添加一条独立语句或声明：`"unsupported '%0' value '%1'; use 'ios-macabi' instead">;`。
- **L871 EN**: Blank line separating nearby declarations or logic blocks.
  **L871 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L872 EN**: Declares TableGen def record `err_drv_invalid_directx_shader_module`.
  **L872 CN**: 声明 TableGen def 记录 `err_drv_invalid_directx_shader_module`。
- **L873 EN**: Adds a standalone statement or declaration: `"invalid profile : %0">;`.
  **L873 CN**: 添加一条独立语句或声明：`"invalid profile : %0">;`。
- **L874 EN**: Declares TableGen def record `err_drv_dxc_invalid_matrix_layout`.
  **L874 CN**: 声明 TableGen def 记录 `err_drv_dxc_invalid_matrix_layout`。
- **L875 EN**: Adds a standalone statement or declaration: `"cannot specify /Zpr and /Zpc together">;`.
  **L875 CN**: 添加一条独立语句或声明：`"cannot specify /Zpr and /Zpc together">;`。
- **L876 EN**: Declares TableGen def record `err_drv_dxc_missing_target_profile`.
  **L876 CN**: 声明 TableGen def 记录 `err_drv_dxc_missing_target_profile`。
- **L877 EN**: Executes a call or declaration centered on `option`.
  **L877 CN**: 执行以 `option` 为核心的调用或声明。
- **L878 EN**: Declares TableGen def record `err_drv_dxc_Fre_requires_Fo_metal`.
  **L878 CN**: 声明 TableGen def 记录 `err_drv_dxc_Fre_requires_Fo_metal`。
- **L879 EN**: Adds a standalone statement or declaration: `: Error<"-Fre option requires -Fo option when targeting Metal">;`.
  **L879 CN**: 添加一条独立语句或声明：`: Error<"-Fre option requires -Fo option when targeting Metal">;`。
- **L880 EN**: Declares TableGen def record `err_drv_hlsl_unsupported_target`.
  **L880 CN**: 声明 TableGen def 记录 `err_drv_hlsl_unsupported_target`。
- **L881 EN**: Adds a standalone statement or declaration: `"HLSL code generation is unsupported for target '%0'">;`.
  **L881 CN**: 添加一条独立语句或声明：`"HLSL code generation is unsupported for target '%0'">;`。
- **L882 EN**: Declares TableGen def record `err_drv_hlsl_bad_shader_required_in_target`.
  **L882 CN**: 声明 TableGen def 记录 `err_drv_hlsl_bad_shader_required_in_target`。
- **L883 EN**: Adds a standalone statement or declaration: `"%select{shader model|Vulkan environment|shader stage}0 is required as %select{OS|environment}1 in target '%2' for HLSL code generation">;`.
  **L883 CN**: 添加一条独立语句或声明：`"%select{shader model|Vulkan environment|shader stage}0 is required as %select{OS|environment}1 in target '%2' for HLSL code generation">;`。
- **L884 EN**: Declares TableGen def record `err_drv_hlsl_16bit_types_unsupported`.
  **L884 CN**: 声明 TableGen def 记录 `err_drv_hlsl_16bit_types_unsupported`。
- **L885 EN**: Adds a standalone statement or declaration: `"'%0' option requires target HLSL Version >= 2018%select{| and shader model >= 6.2}1, but HLSL Version is '%2'%select{| and shader model is '%3'}1">;`.
  **L885 CN**: 添加一条独立语句或声明：`"'%0' option requires target HLSL Version >= 2018%select{| and shader model >= 6.2}1, but HLSL Version is '%2'%select{| and shader model is '%3'}1">;`。
- **L886 EN**: Declares TableGen def record `err_drv_hlsl_bad_shader_unsupported`.
  **L886 CN**: 声明 TableGen def 记录 `err_drv_hlsl_bad_shader_unsupported`。
- **L887 EN**: Adds a standalone statement or declaration: `"%select{shader model|Vulkan environment|shader stage}0 '%1' in target '%2' is invalid for HLSL code generation">;`.
  **L887 CN**: 添加一条独立语句或声明：`"%select{shader model|Vulkan environment|shader stage}0 '%1' in target '%2' is invalid for HLSL code generation">;`。
- **L888 EN**: Declares TableGen def record `warn_drv_dxc_missing_dxv`.
  **L888 CN**: 声明 TableGen def 记录 `warn_drv_dxc_missing_dxv`。

### Lines 889-912

````tablegen
  "dxv not found; resulting DXIL will not be validated or signed for use in "
  "release environment">, InGroup<DXILValidation>;
def warn_drv_dxc_missing_spirv_val : Warning<
  "spirv-val not found; resulting SPIR-V will not be validated">,
  InGroup<SPIRVValidation>;

def err_drv_invalid_range_dxil_validator_version : Error<
  "invalid validator version : %0; validator version must be less than or "
  "equal to current internal version">;
def err_drv_invalid_format_dxil_validator_version : Error<
  "invalid validator version : %0; format of validator version is "
  "\"<major>.<minor>\" (ex:\"1.4\")">;
def err_drv_invalid_empty_dxil_validator_version : Error<
  "invalid validator version : %0; if validator major version is 0, minor "
  "version must also be 0">;

def warn_drv_sarif_format_unstable : Warning<
  "diagnostic formatting in SARIF mode is currently unstable">,
  InGroup<DiagGroup<"sarif-format-unstable">>;

def warn_drv_loongarch_conflicting_implied_val : Warning<
  "ignoring '%0' as it conflicts with that implied by '%1' (%2)">,
  InGroup<OptionIgnored>;
def err_drv_loongarch_invalid_mfpu_EQ : Error<
````
- **L889 EN**: Continues the surrounding expression or declaration: `"dxv not found; resulting DXIL will not be validated or signed for use in "`.
  **L889 CN**: 继续构造周围的表达式或声明：`"dxv not found; resulting DXIL will not be validated or signed for use in "`。
- **L890 EN**: Adds a standalone statement or declaration: `"release environment">, InGroup<DXILValidation>;`.
  **L890 CN**: 添加一条独立语句或声明：`"release environment">, InGroup<DXILValidation>;`。
- **L891 EN**: Declares TableGen def record `warn_drv_dxc_missing_spirv_val`.
  **L891 CN**: 声明 TableGen def 记录 `warn_drv_dxc_missing_spirv_val`。
- **L892 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"spirv-val not found; resulting SPIR-V will not be validated">,`.
  **L892 CN**: 继续一个多行参数列表、初始化器或聚合项：`"spirv-val not found; resulting SPIR-V will not be validated">,`。
- **L893 EN**: Adds a standalone statement or declaration: `InGroup<SPIRVValidation>;`.
  **L893 CN**: 添加一条独立语句或声明：`InGroup<SPIRVValidation>;`。
- **L894 EN**: Blank line separating nearby declarations or logic blocks.
  **L894 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L895 EN**: Declares TableGen def record `err_drv_invalid_range_dxil_validator_version`.
  **L895 CN**: 声明 TableGen def 记录 `err_drv_invalid_range_dxil_validator_version`。
- **L896 EN**: Continues the surrounding expression or declaration: `"invalid validator version : %0; validator version must be less than or "`.
  **L896 CN**: 继续构造周围的表达式或声明：`"invalid validator version : %0; validator version must be less than or "`。
- **L897 EN**: Adds a standalone statement or declaration: `"equal to current internal version">;`.
  **L897 CN**: 添加一条独立语句或声明：`"equal to current internal version">;`。
- **L898 EN**: Declares TableGen def record `err_drv_invalid_format_dxil_validator_version`.
  **L898 CN**: 声明 TableGen def 记录 `err_drv_invalid_format_dxil_validator_version`。
- **L899 EN**: Continues the surrounding expression or declaration: `"invalid validator version : %0; format of validator version is "`.
  **L899 CN**: 继续构造周围的表达式或声明：`"invalid validator version : %0; format of validator version is "`。
- **L900 EN**: Executes a call or declaration centered on `"\"<major>.<minor>\"`.
  **L900 CN**: 执行以 `"\"<major>.<minor>\"` 为核心的调用或声明。
- **L901 EN**: Declares TableGen def record `err_drv_invalid_empty_dxil_validator_version`.
  **L901 CN**: 声明 TableGen def 记录 `err_drv_invalid_empty_dxil_validator_version`。
- **L902 EN**: Continues the surrounding expression or declaration: `"invalid validator version : %0; if validator major version is 0, minor "`.
  **L902 CN**: 继续构造周围的表达式或声明：`"invalid validator version : %0; if validator major version is 0, minor "`。
- **L903 EN**: Adds a standalone statement or declaration: `"version must also be 0">;`.
  **L903 CN**: 添加一条独立语句或声明：`"version must also be 0">;`。
- **L904 EN**: Blank line separating nearby declarations or logic blocks.
  **L904 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L905 EN**: Declares TableGen def record `warn_drv_sarif_format_unstable`.
  **L905 CN**: 声明 TableGen def 记录 `warn_drv_sarif_format_unstable`。
- **L906 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"diagnostic formatting in SARIF mode is currently unstable">,`.
  **L906 CN**: 继续一个多行参数列表、初始化器或聚合项：`"diagnostic formatting in SARIF mode is currently unstable">,`。
- **L907 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"sarif-format-unstable">>;`.
  **L907 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"sarif-format-unstable">>;`。
- **L908 EN**: Blank line separating nearby declarations or logic blocks.
  **L908 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L909 EN**: Declares TableGen def record `warn_drv_loongarch_conflicting_implied_val`.
  **L909 CN**: 声明 TableGen def 记录 `warn_drv_loongarch_conflicting_implied_val`。
- **L910 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"ignoring '%0' as it conflicts with that implied by '%1' (%2)">,`.
  **L910 CN**: 继续一个多行参数列表、初始化器或聚合项：`"ignoring '%0' as it conflicts with that implied by '%1' (%2)">,`。
- **L911 EN**: Adds a standalone statement or declaration: `InGroup<OptionIgnored>;`.
  **L911 CN**: 添加一条独立语句或声明：`InGroup<OptionIgnored>;`。
- **L912 EN**: Declares TableGen def record `err_drv_loongarch_invalid_mfpu_EQ`.
  **L912 CN**: 声明 TableGen def 记录 `err_drv_loongarch_invalid_mfpu_EQ`。

### Lines 913-936

````tablegen
  "invalid argument '%0' to -mfpu=; must be one of: 64, 32, none, 0 (alias for none)">;
def err_drv_loongarch_wrong_fpu_width : Error<
  "wrong fpu width; %select{LSX|LASX}0 depends on 64-bit FPU">;
def err_drv_loongarch_invalid_simd_option_combination : Error<
  "invalid option combination; LASX depends on LSX">;
def err_drv_loongarch_invalid_msimd_EQ : Error<
  "invalid argument '%0' to -msimd=; must be one of: none, lsx, lasx">;

def err_drv_expand_response_file : Error<
  "failed to expand response file: %0">;

def warn_drv_missing_multilib : Warning<
  "no multilib found matching flags: %0">,
  InGroup<DiagGroup<"missing-multilib">>;
def note_drv_available_multilibs : Note<
  "available multilibs are:%0">;
def err_drv_multilib_custom_error : Error<
  "multilib configuration error: %0">;
def warn_drv_multilib_not_available_for_target: Warning<
  "no multilib structure encoded for Arm, Aarch64 and PPC targets">,
  InGroup<DiagGroup<"multilib-not-found">>;

def err_drv_experimental_crel : Error<
  "-Wa,--allow-experimental-crel must be specified to use -Wa,--crel. "
````
- **L913 EN**: Executes a call or declaration centered on `0`.
  **L913 CN**: 执行以 `0` 为核心的调用或声明。
- **L914 EN**: Declares TableGen def record `err_drv_loongarch_wrong_fpu_width`.
  **L914 CN**: 声明 TableGen def 记录 `err_drv_loongarch_wrong_fpu_width`。
- **L915 EN**: Adds a standalone statement or declaration: `"wrong fpu width; %select{LSX|LASX}0 depends on 64-bit FPU">;`.
  **L915 CN**: 添加一条独立语句或声明：`"wrong fpu width; %select{LSX|LASX}0 depends on 64-bit FPU">;`。
- **L916 EN**: Declares TableGen def record `err_drv_loongarch_invalid_simd_option_combination`.
  **L916 CN**: 声明 TableGen def 记录 `err_drv_loongarch_invalid_simd_option_combination`。
- **L917 EN**: Adds a standalone statement or declaration: `"invalid option combination; LASX depends on LSX">;`.
  **L917 CN**: 添加一条独立语句或声明：`"invalid option combination; LASX depends on LSX">;`。
- **L918 EN**: Declares TableGen def record `err_drv_loongarch_invalid_msimd_EQ`.
  **L918 CN**: 声明 TableGen def 记录 `err_drv_loongarch_invalid_msimd_EQ`。
- **L919 EN**: Adds a standalone statement or declaration: `"invalid argument '%0' to -msimd=; must be one of: none, lsx, lasx">;`.
  **L919 CN**: 添加一条独立语句或声明：`"invalid argument '%0' to -msimd=; must be one of: none, lsx, lasx">;`。
- **L920 EN**: Blank line separating nearby declarations or logic blocks.
  **L920 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L921 EN**: Declares TableGen def record `err_drv_expand_response_file`.
  **L921 CN**: 声明 TableGen def 记录 `err_drv_expand_response_file`。
- **L922 EN**: Adds a standalone statement or declaration: `"failed to expand response file: %0">;`.
  **L922 CN**: 添加一条独立语句或声明：`"failed to expand response file: %0">;`。
- **L923 EN**: Blank line separating nearby declarations or logic blocks.
  **L923 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L924 EN**: Declares TableGen def record `warn_drv_missing_multilib`.
  **L924 CN**: 声明 TableGen def 记录 `warn_drv_missing_multilib`。
- **L925 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"no multilib found matching flags: %0">,`.
  **L925 CN**: 继续一个多行参数列表、初始化器或聚合项：`"no multilib found matching flags: %0">,`。
- **L926 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"missing-multilib">>;`.
  **L926 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"missing-multilib">>;`。
- **L927 EN**: Declares TableGen def record `note_drv_available_multilibs`.
  **L927 CN**: 声明 TableGen def 记录 `note_drv_available_multilibs`。
- **L928 EN**: Adds a standalone statement or declaration: `"available multilibs are:%0">;`.
  **L928 CN**: 添加一条独立语句或声明：`"available multilibs are:%0">;`。
- **L929 EN**: Declares TableGen def record `err_drv_multilib_custom_error`.
  **L929 CN**: 声明 TableGen def 记录 `err_drv_multilib_custom_error`。
- **L930 EN**: Adds a standalone statement or declaration: `"multilib configuration error: %0">;`.
  **L930 CN**: 添加一条独立语句或声明：`"multilib configuration error: %0">;`。
- **L931 EN**: Declares TableGen def record `warn_drv_multilib_not_available_for_target`.
  **L931 CN**: 声明 TableGen def 记录 `warn_drv_multilib_not_available_for_target`。
- **L932 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"no multilib structure encoded for Arm, Aarch64 and PPC targets">,`.
  **L932 CN**: 继续一个多行参数列表、初始化器或聚合项：`"no multilib structure encoded for Arm, Aarch64 and PPC targets">,`。
- **L933 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"multilib-not-found">>;`.
  **L933 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"multilib-not-found">>;`。
- **L934 EN**: Blank line separating nearby declarations or logic blocks.
  **L934 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L935 EN**: Declares TableGen def record `err_drv_experimental_crel`.
  **L935 CN**: 声明 TableGen def 记录 `err_drv_experimental_crel`。
- **L936 EN**: Continues the surrounding expression or declaration: `"-Wa,--allow-experimental-crel must be specified to use -Wa,--crel. "`.
  **L936 CN**: 继续构造周围的表达式或声明：`"-Wa,--allow-experimental-crel must be specified to use -Wa,--crel. "`。

### Lines 937-960

````tablegen
  "CREL is experimental and uses a non-standard section type code">;

def err_drv_experimental_sframe : Error<
  "-Wa,--allow-experimental-sframe must be specified to use -Wa,--gsframe. "
  "SFrames are experimental and may be removed at any time without warning">;

def warn_android_unversioned_fallback : Warning<
  "using unversioned Android target directory %0 for target %1; unversioned "
  "directories will not be used in Clang 19 -- provide a versioned directory "
  "for the target version or lower instead">,
  InGroup<DiagGroup<"android-unversioned-fallback">>;

def err_drv_triple_version_invalid : Error<
  "version '%0' in target triple '%1' is invalid">;

def warn_missing_include_dirs : Warning<
  "no such include directory: '%0'">, InGroup<MissingIncludeDirs>, DefaultIgnore;

def err_drv_malformed_warning_suppression_mapping : Error<
  "failed to process suppression mapping file '%0': %1">;

def warn_drv_openacc_without_cir
    : Warning<"OpenACC directives will result in no runtime behavior; use "
              "-fclangir to enable runtime effect">,
````
- **L937 EN**: Adds a standalone statement or declaration: `"CREL is experimental and uses a non-standard section type code">;`.
  **L937 CN**: 添加一条独立语句或声明：`"CREL is experimental and uses a non-standard section type code">;`。
- **L938 EN**: Blank line separating nearby declarations or logic blocks.
  **L938 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L939 EN**: Declares TableGen def record `err_drv_experimental_sframe`.
  **L939 CN**: 声明 TableGen def 记录 `err_drv_experimental_sframe`。
- **L940 EN**: Continues the surrounding expression or declaration: `"-Wa,--allow-experimental-sframe must be specified to use -Wa,--gsframe. "`.
  **L940 CN**: 继续构造周围的表达式或声明：`"-Wa,--allow-experimental-sframe must be specified to use -Wa,--gsframe. "`。
- **L941 EN**: Adds a standalone statement or declaration: `"SFrames are experimental and may be removed at any time without warning">;`.
  **L941 CN**: 添加一条独立语句或声明：`"SFrames are experimental and may be removed at any time without warning">;`。
- **L942 EN**: Blank line separating nearby declarations or logic blocks.
  **L942 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L943 EN**: Declares TableGen def record `warn_android_unversioned_fallback`.
  **L943 CN**: 声明 TableGen def 记录 `warn_android_unversioned_fallback`。
- **L944 EN**: Continues the surrounding expression or declaration: `"using unversioned Android target directory %0 for target %1; unversioned "`.
  **L944 CN**: 继续构造周围的表达式或声明：`"using unversioned Android target directory %0 for target %1; unversioned "`。
- **L945 EN**: Continues the surrounding expression or declaration: `"directories will not be used in Clang 19 -- provide a versioned directory "`.
  **L945 CN**: 继续构造周围的表达式或声明：`"directories will not be used in Clang 19 -- provide a versioned directory "`。
- **L946 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"for the target version or lower instead">,`.
  **L946 CN**: 继续一个多行参数列表、初始化器或聚合项：`"for the target version or lower instead">,`。
- **L947 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"android-unversioned-fallback">>;`.
  **L947 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"android-unversioned-fallback">>;`。
- **L948 EN**: Blank line separating nearby declarations or logic blocks.
  **L948 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L949 EN**: Declares TableGen def record `err_drv_triple_version_invalid`.
  **L949 CN**: 声明 TableGen def 记录 `err_drv_triple_version_invalid`。
- **L950 EN**: Adds a standalone statement or declaration: `"version '%0' in target triple '%1' is invalid">;`.
  **L950 CN**: 添加一条独立语句或声明：`"version '%0' in target triple '%1' is invalid">;`。
- **L951 EN**: Blank line separating nearby declarations or logic blocks.
  **L951 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L952 EN**: Declares TableGen def record `warn_missing_include_dirs`.
  **L952 CN**: 声明 TableGen def 记录 `warn_missing_include_dirs`。
- **L953 EN**: Adds a standalone statement or declaration: `"no such include directory: '%0'">, InGroup<MissingIncludeDirs>, DefaultIgnore;`.
  **L953 CN**: 添加一条独立语句或声明：`"no such include directory: '%0'">, InGroup<MissingIncludeDirs>, DefaultIgnore;`。
- **L954 EN**: Blank line separating nearby declarations or logic blocks.
  **L954 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L955 EN**: Declares TableGen def record `err_drv_malformed_warning_suppression_mapping`.
  **L955 CN**: 声明 TableGen def 记录 `err_drv_malformed_warning_suppression_mapping`。
- **L956 EN**: Adds a standalone statement or declaration: `"failed to process suppression mapping file '%0': %1">;`.
  **L956 CN**: 添加一条独立语句或声明：`"failed to process suppression mapping file '%0': %1">;`。
- **L957 EN**: Blank line separating nearby declarations or logic blocks.
  **L957 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L958 EN**: Declares TableGen def record `warn_drv_openacc_without_cir`.
  **L958 CN**: 声明 TableGen def 记录 `warn_drv_openacc_without_cir`。
- **L959 EN**: Continues the surrounding expression or declaration: `: Warning<"OpenACC directives will result in no runtime behavior; use "`.
  **L959 CN**: 继续构造周围的表达式或声明：`: Warning<"OpenACC directives will result in no runtime behavior; use "`。
- **L960 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"-fclangir to enable runtime effect">,`.
  **L960 CN**: 继续一个多行参数列表、初始化器或聚合项：`"-fclangir to enable runtime effect">,`。

### Lines 961-970

````tablegen
      InGroup<SourceUsesOpenACC>;

def warn_drv_gcc_install_dir_libstdcxx : Warning<
    "future releases of the clang compiler will prefer GCC installations "
    "containing libstdc++ include directories; '%0' would be chosen over '%1'">,
    InGroup<DiagGroup<"gcc-install-dir-libstdcxx">>;

def err_drv_reflection_requires_cxx26 : Error<
  "option '%0' is only supported when compiling in C++26 mode">;
}
````
- **L961 EN**: Adds a standalone statement or declaration: `InGroup<SourceUsesOpenACC>;`.
  **L961 CN**: 添加一条独立语句或声明：`InGroup<SourceUsesOpenACC>;`。
- **L962 EN**: Blank line separating nearby declarations or logic blocks.
  **L962 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L963 EN**: Declares TableGen def record `warn_drv_gcc_install_dir_libstdcxx`.
  **L963 CN**: 声明 TableGen def 记录 `warn_drv_gcc_install_dir_libstdcxx`。
- **L964 EN**: Continues the surrounding expression or declaration: `"future releases of the clang compiler will prefer GCC installations "`.
  **L964 CN**: 继续构造周围的表达式或声明：`"future releases of the clang compiler will prefer GCC installations "`。
- **L965 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `"containing libstdc++ include directories; '%0' would be chosen over '%1'">,`.
  **L965 CN**: 继续一个多行参数列表、初始化器或聚合项：`"containing libstdc++ include directories; '%0' would be chosen over '%1'">,`。
- **L966 EN**: Adds a standalone statement or declaration: `InGroup<DiagGroup<"gcc-install-dir-libstdcxx">>;`.
  **L966 CN**: 添加一条独立语句或声明：`InGroup<DiagGroup<"gcc-install-dir-libstdcxx">>;`。
- **L967 EN**: Blank line separating nearby declarations or logic blocks.
  **L967 CN**: 空行，用于分隔附近的声明或逻辑块。
- **L968 EN**: Declares TableGen def record `err_drv_reflection_requires_cxx26`.
  **L968 CN**: 声明 TableGen def 记录 `err_drv_reflection_requires_cxx26`。
- **L969 EN**: Adds a standalone statement or declaration: `"option '%0' is only supported when compiling in C++26 mode">;`.
  **L969 CN**: 添加一条独立语句或声明：`"option '%0' is only supported when compiling in C++26 mode">;`。
- **L970 EN**: Closes the current lexical scope or compound statement.
  **L970 CN**: 结束当前词法作用域或复合语句块。

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
- **Diagnostics tables / 诊断表**
  - **EN**: Models diagnostic identifiers, categories, and message metadata consumed by Clang diagnostics.
  - **CN**: 建模 Clang 诊断系统使用的诊断标识、类别与消息元数据。
- **ABI contracts / ABI 契约**
  - **EN**: Records ABI-relevant enums and conventions used to represent constructors, destructors, or calling details.
  - **CN**: 记录用于表示构造函数、析构函数或调用细节的 ABI 相关枚举与约定。
- **Type-system support / 类型系统支撑**
  - **EN**: Provides metadata or helpers used to model Clang types and related dispatch logic.
  - **CN**: 提供建模 Clang 类型及相关分发逻辑所需的元数据或辅助工具。
- **Virtual dispatch layout / 虚派发布局**
  - **EN**: Encodes vtable structure, method slots, and base-adjustment information for C++.
  - **CN**: 编码 C++ 的虚表结构、方法槽位与基类调整信息。
- **Source-location handling / 源码位置处理**
  - **EN**: Tracks files, offsets, and source ranges needed by diagnostics and tooling.
  - **CN**: 跟踪诊断与工具所需的文件、偏移与源码范围。
- **AArch64 target support / AArch64 目标支持**
  - **EN**: Encodes AArch64-specific compiler metadata or builtin descriptions.
  - **CN**: 编码 AArch64 专用的编译器元数据或 builtin 描述。

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: None detected. / 未检测到。
- **Macros / 宏**: No prominent macros detected. / 未检测到明显宏。
- **Types / 类型**: No obvious type declarations detected by the generator. / 生成器未检测到明显类型声明。
- **Functions or callables / 函数或可调用对象**: `runtime`, `timeout`, `missing`, `signal`, `units`, `sections`, `support`, `integers`, `option`
- **TableGen records / TableGen 记录**: `err_drv_no_such_file`, `err_drv_no_such_file_with_suggestion`, `err_drv_unsupported_opt`, `err_drv_unsupported_opt_with_suggestion`, `err_drv_unsupported_opt_for_target`, `err_drv_unsupported_opt_for_language_mode`, `err_drv_unsupported_option_argument`, `err_drv_unsupported_option_argument_for_target`, `err_drv_unknown_stdin_type`, `err_drv_unknown_stdin_type_clang_cl`, `err_drv_unknown_language`, `err_drv_invalid_arch_name`, `err_drv_invalid_riscv_arch_name`, `err_drv_invalid_riscv_cpu_name_for_target`, `warn_drv_invalid_arch_name_with_suggestion`, `warn_drv_avr_mcu_not_specified`
- **Namespaces / 命名空间**: No explicit namespaces detected. / 未检测到显式命名空间。
