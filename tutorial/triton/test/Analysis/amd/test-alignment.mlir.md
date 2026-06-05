# test-alignment.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Analysis/amd/test-alignment.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises alignment and axis information in Triton's analysis coverage. **CN:** 该MLIR 测试用于覆盖 Triton 分析中的对齐与轴信息相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -test-print-amd-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: none; diagnostics annotations: remark×5. **CN:** FileCheck 标记为 none；诊断标注为 remark×5。
- **Validation / 验证内容:** **EN:** The file mainly validates that alignment and axis information emits the expected analysis remarks/notes. **CN:** 该文件主要验证 对齐与轴信息 会发出预期的分析备注/说明。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -test-print-amd-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -test-print-amd-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -test-print-amd-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-6
```mlir
// Test: Contiguity is limited by output shape when offset is zero.
// Input has contiguity [256, 64], but output shape is [128, 32], so
// contiguity is capped to [128, 32]. Divisibility remains unchanged at [8, 8].
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 8-12
```mlir
tt.func public @kernel_contiguity_limiting_size(%arg0: tensor<256x64xf16, #mma> {tt.contiguity = dense<[256, 64]> : tensor<2xi32>, tt.divisibility = dense<[8, 8]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
  // expected-remark @below {{contiguity = [128, 32], divisibility = [8, 8], constancy = [1, 1], constant_value = <none>}}
  %0 = amdg.extract_slice %arg0 [0, 0] : tensor<256x64xf16, #mma> to tensor<128x32xf16, #mma>
  tt.return
}
```
**EN:** This block defines `kernel_contiguity_limiting_size` and exercises operations such as `tt.func`, `tt.contiguity`, `tt.divisibility`, `tt.constancy`, `amdg.extract_slice`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `kernel_contiguity_limiting_size`，并覆盖 如 `tt.func`、`tt.contiguity`、`tt.divisibility`、`tt.constancy`、`amdg.extract_slice` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 14
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 16-19
```mlir
// Test: Contiguity is limited by non-zero offset breaking alignment.
// Offset [64, 16] breaks contiguity groups: gcd(64, 256)=64, gcd(16, 64)=16.
// New contiguity is [64, 16]. Divisibility remains [8, 8] since gcd(8, 64)=8 and gcd(8, 16)=8.
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 1], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 21-25
```mlir
tt.func public @kernel_contiguity_limiting_offset(%arg0: tensor<256x64xf16, #mma> {tt.contiguity = dense<[256, 64]> : tensor<2xi32>, tt.divisibility = dense<[8, 8]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
  // expected-remark @below {{contiguity = [64, 16], divisibility = [8, 8], constancy = [1, 1], constant_value = <none>}}
  %0 = amdg.extract_slice %arg0 [64, 16] : tensor<256x64xf16, #mma> to tensor<128x32xf16, #mma>
  tt.return
}
```
**EN:** This block defines `kernel_contiguity_limiting_offset` and exercises operations such as `tt.func`, `tt.contiguity`, `tt.divisibility`, `tt.constancy`, `amdg.extract_slice`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `kernel_contiguity_limiting_offset`，并覆盖 如 `tt.func`、`tt.contiguity`、`tt.divisibility`、`tt.constancy`、`amdg.extract_slice` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 27
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 29-32
```mlir
// Test: Divisibility is reduced by offset.
// Offset [128, 32] reduces divisibility: gcd(1024, 128)=128, gcd(1024, 32)=32.
// Contiguity is [32, 32] (limited by output shape).
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 1], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 34-38
```mlir
tt.func public @kernel_divisibility(%arg0: tensor<256x64xf16, #mma> {tt.contiguity = dense<[256, 64]> : tensor<2xi32>, tt.divisibility = dense<[1024, 1024]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
  // expected-remark @below {{contiguity = [32, 32], divisibility = [128, 32], constancy = [1, 1], constant_value = <none>}}
  %0 = amdg.extract_slice %arg0 [128, 32] : tensor<256x64xf16, #mma> to tensor<32x32xf16, #mma>
  tt.return
}
```
**EN:** This block defines `kernel_divisibility` and exercises operations such as `tt.func`, `tt.contiguity`, `tt.divisibility`, `tt.constancy`, `amdg.extract_slice`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `kernel_divisibility`，并覆盖 如 `tt.func`、`tt.contiguity`、`tt.divisibility`、`tt.constancy`、`amdg.extract_slice` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 40
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 42-45
```mlir
// Test: Constancy is limited by output shape when offset is zero.
// Input has constancy [256, 64], but output shape is [128, 32], so
// constancy is capped to [128, 32]. Divisibility remains unchanged.
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 1], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 47-51
```mlir
tt.func public @kernel_constancy_limiting_size(%arg0: tensor<256x64xf16, #mma> {tt.contiguity = dense<[1, 1]> : tensor<2xi32>, tt.divisibility = dense<[1024, 1024]> : tensor<2xi32>, tt.constancy = dense<[256, 64]> : tensor<2xi32>}) {
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1024, 1024], constancy = [128, 32], constant_value = <none>}}
  %0 = amdg.extract_slice %arg0 [0, 0] : tensor<256x64xf16, #mma> to tensor<128x32xf16, #mma>
  tt.return
}
```
**EN:** This block defines `kernel_constancy_limiting_size` and exercises operations such as `tt.func`, `tt.contiguity`, `tt.divisibility`, `tt.constancy`, `amdg.extract_slice`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `kernel_constancy_limiting_size`，并覆盖 如 `tt.func`、`tt.contiguity`、`tt.divisibility`、`tt.constancy`、`amdg.extract_slice` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 53
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 55-58
```mlir
// Test: Constancy is limited by non-zero offset breaking alignment.
// Offset [64, 16] breaks constancy groups: gcd(64, 128)=64, gcd(16, 32)=16.
// New constancy is [64, 16]. Divisibility remains unchanged.
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 1], instrShape = [32, 32, 8], isTransposed = true}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 60-64
```mlir
tt.func public @kernel_constancy_limiting_offset(%arg0: tensor<256x64xf16, #mma> {tt.contiguity = dense<[1, 1]> : tensor<2xi32>, tt.divisibility = dense<[1024, 1024]> : tensor<2xi32>, tt.constancy = dense<[128, 32]> : tensor<2xi32>}) {
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1024, 1024], constancy = [64, 16], constant_value = <none>}}
  %0 = amdg.extract_slice %arg0 [64, 16] : tensor<256x64xf16, #mma> to tensor<128x32xf16, #mma>
  tt.return
}
```
**EN:** This block defines `kernel_constancy_limiting_offset` and exercises operations such as `tt.func`, `tt.contiguity`, `tt.divisibility`, `tt.constancy`, `amdg.extract_slice`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `kernel_constancy_limiting_offset`，并覆盖 如 `tt.func`、`tt.contiguity`、`tt.divisibility`、`tt.constancy`、`amdg.extract_slice` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's analysis coverage and focuses on alignment and axis information.  
  **CN:** 该文件属于 Triton 的 分析 测试覆盖，关注点是 对齐与轴信息。
- **EN:** The `RUN` pipeline drives tools/passes such as `-test-print-amd-alignment`, `-verify-diagnostics=only-expected`.  
  **CN:** `RUN` 流水线会驱动 `-test-print-amd-alignment`, `-verify-diagnostics=only-expected` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）。
- **EN:** Expected remarks/notes are used to pin down the exact analysis output produced by the pass.  
  **CN:** 这里的期望备注/说明用于固定 pass 产生的精确分析输出。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`.  
  **CN:** 外部测试工具：`triton-opt`。
- **EN:** IR dialect dependencies: `tt`.  
  **CN:** IR 方言依赖：`tt`。
