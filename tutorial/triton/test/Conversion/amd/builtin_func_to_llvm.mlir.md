# builtin_func_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/builtin_func_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ`<br>`triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ`<br>`triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ`<br>`triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK-LABEL×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK-LABEL×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that AMD-specific lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 AMD 专用降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
// RUN: triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ
// RUN: triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ
// RUN: triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ
// RUN: triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ; triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ; triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ; triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ; triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=True" --convert-builtin-func-to-llvm="ftz=True" | FileCheck %s --check-prefix=LLVM_FTZ; triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx942 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ; triton-opt %s --convert-triton-amdgpu-to-llvm="gfx-arch=gfx950 ftz=False" --convert-builtin-func-to-llvm="ftz=False" | FileCheck %s --check-prefix=LLVM_NO_FTZ`，从而以预期的 pass 顺序和校验模式执行测试。

### Line 6
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 8-16
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @test_fast_expf(%arg0: tensor<64xf32, #blocked>) -> tensor<64xf32, #blocked> {
    // CHECK-LABEL: test_fast_expf
    // LLVM_FTZ: rocdl.exp2
    // LLVM_NO_FTZ: llvm.intr.exp2
    %0 = tt.extern_elementwise %arg0 {libname = "libdevice", libpath = "", pure = true, symbol = "__triton_hip_fast_expf"} : (tensor<64xf32, #blocked>) -> tensor<64xf32, #blocked>
    tt.return %0 : tensor<64xf32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_fast_expf`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_fast_expf`。

### Lines 18-26
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @test_fast_tanhf(%arg0: tensor<64xf32, #blocked>) -> tensor<64xf32, #blocked> {
    // CHECK-LABEL: test_fast_tanhf
    // LLVM_FTZ: rocdl.exp2
    // LLVM_NO_FTZ: llvm.intr.exp2
    %0 = tt.extern_elementwise %arg0 {libname = "libdevice", libpath = "", pure = true, symbol = "__triton_hip_fast_tanhf"} : (tensor<64xf32, #blocked>) -> tensor<64xf32, #blocked>
    tt.return %0 : tensor<64xf32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_fast_tanhf`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_fast_tanhf`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx942`, `--convert-builtin-func-to-llvm="ftz=True"`, `--check-prefix=LLVM_FTZ`, `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx950`, `--convert-builtin-func-to-llvm="ftz=False"`, `--check-prefix=LLVM_NO_FTZ`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx942`, `--convert-builtin-func-to-llvm="ftz=True"`, `--check-prefix=LLVM_FTZ`, `--convert-triton-amdgpu-to-llvm="gfx-arch=gfx950`, `--convert-builtin-func-to-llvm="ftz=False"`, `--check-prefix=LLVM_NO_FTZ` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`。
