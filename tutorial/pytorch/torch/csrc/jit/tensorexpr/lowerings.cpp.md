# lowerings.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/csrc/jit/tensorexpr/lowerings.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines operator wrappers, schemas, or dispatch helpers that connect TorchScript nodes to runtime kernels.
- **Purpose (CN)**: 定义算子包装、schema 或分发辅助逻辑，将 TorchScript 节点连接到运行时内核。
## Line-by-Line Analysis / 逐行分析

### Lines 1-31
```cpp
#include <torch/csrc/jit/frontend/function_schema_parser.h>
#include <torch/csrc/jit/tensorexpr/ir_simplifier.h>
#include <torch/csrc/jit/tensorexpr/lowerings.h>
#include <torch/csrc/jit/tensorexpr/operators/operators.h>

#include <ATen/native/Activation.h>
#include <ATen/native/mkldnn/Common.h>

namespace torch::jit::tensorexpr {

FunctionSchemaMap<NNCLoweringFunction>& getNNCLoweringRegistry() {
  static FunctionSchemaMap<NNCLoweringFunction> lowering_registry_;
  return lowering_registry_;
}

RegisterNNCLoweringsFunction::RegisterNNCLoweringsFunction(
    const std::vector<std::string>& schemas,
    const NNCLoweringFunction& fn) {
  for (const auto& schema_str : schemas) {
    getNNCLoweringRegistry().insert(parseSchema(schema_str), fn);
  }
}

namespace {
int nnc_lowerings_lazy_registration() {
  RegisterNNCLoweringsFunction aten_dropout(
      {"aten::dropout(Tensor input, float p, bool train) -> (Tensor)"},
      computeNoop);
  RegisterNNCLoweringsFunction aten_contiguous(
      {"aten::contiguous(Tensor(a) self, *, MemoryFormat memory_format=contiguous_format) -> (Tensor(a))"},
      computeNoop);
```
- **EN**: This block assembles the compilation dependencies, pulling in local JIT headers such as torch/csrc/jit/frontend/function_schema_parser.h, torch/csrc/jit/tensorexpr/ir_simplifier.h, torch/csrc/jit/tensorexpr/lowerings.h, and 1 more; ATen/c10 facilities such as ATen/native/Activation.h, ATen/native/mkldnn/Common.h. The namespace declarations place the code inside torch::jit::tensorexpr, matching the surrounding JIT subsystem. This chunk defines `nnc_lowerings_lazy_registration`, which lowers higher-level IR into a backend-specific executable form. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段组织编译依赖，引入了本地 JIT 头文件，如 torch/csrc/jit/frontend/function_schema_parser.h、torch/csrc/jit/tensorexpr/ir_simplifier.h、torch/csrc/jit/tensorexpr/lowerings.h 等共 4 项；ATen/c10 基础设施，如 ATen/native/Activation.h、ATen/native/mkldnn/Common.h。 命名空间声明把代码放入 torch::jit::tensorexpr 中，与周边 JIT 子系统保持一致。 这一段定义了 `nnc_lowerings_lazy_registration`，其作用是把高层 IR 降级为后端特定的可执行形式。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 33-68
```cpp
#ifdef USE_XNNPACK
  // TODO: add a test
  RegisterNNCLoweringsFunction prepacked_conv2d_clamp_run(
      {"prepacked::conv2d_clamp_run(Tensor X, __torch__.torch.classes.xnnpack.Conv2dOpContext W_prepack) -> (Tensor Y)"},
      computePrepackedConv2dClampRun);

  // TODO: add a test
  RegisterNNCLoweringsFunction prepacked_linear_clamp_run(
      {"prepacked::linear_clamp_run(Tensor X, __torch__.torch.classes.xnnpack.LinearOpContext W_prepack) -> (Tensor Y)"},
      computePrepackedLinearClampRun);
#endif

#if AT_MKLDNN_ENABLED()
  RegisterNNCLoweringsFunction mkldnn_prepacked_conv2d_run(
      {"mkldnn_prepacked::conv2d_run(Tensor X, __torch__.torch.classes.mkldnn.ConvOpContext W_prepack) -> (Tensor Y)"},
      computeMkldnnPrepackedConvRun);
#endif // AT_MKLDNN_ENABLED()

  RegisterNNCLoweringsFunction aten_sub(
      {"aten::sub.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> (Tensor)",
       "aten::sub.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        auto sub_lambda = [](const ExprHandle& lhs, const ExprHandle& rhs) {
          // NB: sub isn't supported on boolean, no need to promote to integer.
          return lhs - rhs;
        };
        TORCH_INTERNAL_ASSERT(
            inputs.size() == 2 || inputs.size() == 3,
            buildErrorMessage("Invalid number of input operands"));
        return (inputs.size() > 2) ? computeTwoOperandWithAlpha(
                                         "aten_sub",
                                         inputs,
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 69-99
```cpp
                                         outputShape,
                                         outputStrides,
                                         outputType,
                                         sub_lambda)
                                   : computeTwoOperand(
                                         "aten_sub",
                                         inputs,
                                         outputShape,
                                         outputStrides,
                                         outputType,
                                         sub_lambda);
      });

  RegisterNNCLoweringsFunction aten_mul(
      {"aten::mul.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::mul.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_mul",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return boolToInteger(lhs) * boolToInteger(rhs);
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 101-136
```cpp
#define DEFINE_BINARY_SCALAR_OP_LOWERING(op_name, op)                     \
  RegisterNNCLoweringsFunction aten_##op_name##_scalar(                   \
      {"aten::" #op_name ".int(int a, int b) -> (int)",                   \
       "aten::" #op_name ".int_float(int a, float b) -> (float)",         \
       "aten::" #op_name ".float_int(float a, int b) -> (float)",         \
       "aten::" #op_name ".float(float a, float b) -> (float)"},          \
      [](const std::vector<ArgValue>& inputs,                             \
         const std::vector<ExprHandle>& outputShape,                      \
         const std::vector<ExprHandle>& outputStrides,                    \
         const std::optional<ScalarType>& outputType,                     \
         at::Device device) {                                             \
        return computeScalar(                                             \
            "aten_#op_name",                                              \
            inputs,                                                       \
            outputShape,                                                  \
            outputStrides,                                                \
            outputType,                                                   \
            [](const ExprHandle& a, const ExprHandle& b) { return op; }); \
      });
  DEFINE_BINARY_SCALAR_OP_LOWERING(mul, a * b)
  DEFINE_BINARY_SCALAR_OP_LOWERING(add, a + b)
  DEFINE_BINARY_SCALAR_OP_LOWERING(sub, a - b)
#undef DEFINE_BINARY_SCALAR_OP_LOWERING
  RegisterNNCLoweringsFunction aten_div_scalar(
      {"aten::div(Scalar a, Scalar b) -> (float)",
       "aten::div.int(int a, int b) -> (float)",
       "aten::div.int_float(int a, float b) -> (float)",
       "aten::div.float_int(float a, int b) -> (float)",
       "aten::div.float(float a, float b) -> (float)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeScalar(
            "aten_div",
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 137-172
```cpp
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a, const ExprHandle& b) {
              return promoteIntegerToDefaultType(a) /
                  promoteIntegerToDefaultType(b);
            });
      });

#define DEFINE_COMPARISON_SCALAR_OP_LOWERING(op_name, op)                 \
  RegisterNNCLoweringsFunction aten_##op_name##_scalar(                   \
      {"aten::" #op_name ".bool(bool a, bool b) -> (bool)",               \
       "aten::" #op_name ".int(int a, int b) -> (bool)",                  \
       "aten::" #op_name ".int_float(int a, float b) -> (bool)",          \
       "aten::" #op_name ".float_int(float a, int b) -> (bool)",          \
       "aten::" #op_name ".float(float a, float b) -> (bool)"},           \
      [](const std::vector<ArgValue>& inputs,                             \
         const std::vector<ExprHandle>& outputShape,                      \
         const std::vector<ExprHandle>& outputStrides,                    \
         const std::optional<ScalarType>& outputType,                     \
         at::Device device) {                                             \
        return computeScalar(                                             \
            "aten_#op_name",                                              \
            inputs,                                                       \
            outputShape,                                                  \
            outputStrides,                                                \
            outputType,                                                   \
            [](const ExprHandle& a, const ExprHandle& b) { return op; }); \
      });
  DEFINE_COMPARISON_SCALAR_OP_LOWERING(lt, cast<bool>(a < b))
  DEFINE_COMPARISON_SCALAR_OP_LOWERING(le, cast<bool>(a <= b))
  DEFINE_COMPARISON_SCALAR_OP_LOWERING(eq, cast<bool>(a == b))
  DEFINE_COMPARISON_SCALAR_OP_LOWERING(ne, cast<bool>(a != b))
  DEFINE_COMPARISON_SCALAR_OP_LOWERING(gt, cast<bool>(a > b))
  DEFINE_COMPARISON_SCALAR_OP_LOWERING(ge, cast<bool>(a >= b))
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 173-198
```cpp
#undef DEFINE_COMPARISON_SCALAR_OP_LOWERING

#define DEFINE_BITWISE_SCALAR_OP_LOWERING(op_name, op)                    \
  RegisterNNCLoweringsFunction aten_##op_name##_int_scalar(               \
      {"aten::" #op_name ".int(int a, int b) -> (int)"},                  \
      [](const std::vector<ArgValue>& inputs,                             \
         const std::vector<ExprHandle>& outputShape,                      \
         const std::vector<ExprHandle>& outputStrides,                    \
         const std::optional<ScalarType>& outputType,                     \
         at::Device device) {                                             \
        return computeScalar(                                             \
            "aten_#op_name",                                              \
            inputs,                                                       \
            outputShape,                                                  \
            outputStrides,                                                \
            outputType,                                                   \
            [](const ExprHandle& a, const ExprHandle& b) { return op; }); \
      });
  DEFINE_BITWISE_SCALAR_OP_LOWERING(
      __and__, boolToInteger(a) & boolToInteger(b))
  DEFINE_BITWISE_SCALAR_OP_LOWERING(__or__, boolToInteger(a) | boolToInteger(b))
  DEFINE_BITWISE_SCALAR_OP_LOWERING(
      __xor__, boolToInteger(a) ^ boolToInteger(b))
  DEFINE_BITWISE_SCALAR_OP_LOWERING(__lshift__, a << b)
  DEFINE_BITWISE_SCALAR_OP_LOWERING(__rshift__, a >> b)
#undef DEFINE_BITWISE_SCALAR_OP_LOWERING
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 200-235
```cpp
#define DEFINE_LOGICAL_SCALAR_OP_LOWERING(op_name, op)                    \
  RegisterNNCLoweringsFunction aten_##op_name##_bool_scalar(              \
      {"aten::" #op_name ".bool(bool a, bool b) -> (bool)"},              \
      [](const std::vector<ArgValue>& inputs,                             \
         const std::vector<ExprHandle>& outputShape,                      \
         const std::vector<ExprHandle>& outputStrides,                    \
         const std::optional<ScalarType>& outputType,                     \
         at::Device device) {                                             \
        return computeScalar(                                             \
            "aten_#op_name",                                              \
            inputs,                                                       \
            outputShape,                                                  \
            outputStrides,                                                \
            outputType,                                                   \
            [](const ExprHandle& a, const ExprHandle& b) { return op; }); \
      });
  DEFINE_LOGICAL_SCALAR_OP_LOWERING(__and__, a && b)
  DEFINE_LOGICAL_SCALAR_OP_LOWERING(__or__, a || b)
  DEFINE_LOGICAL_SCALAR_OP_LOWERING(__xor__, a != b)
#undef DEFINE_LOGICAL_SCALAR_OP_LOWERING

  RegisterNNCLoweringsFunction aten_div(
      {"aten::div.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::div.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_div",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 236-258
```cpp
              return promoteIntegerToDefaultType(lhs) /
                  promoteIntegerToDefaultType(rhs);
            });
      });

  RegisterNNCLoweringsFunction aten___and__(
      {"aten::__and__.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::__and__.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_and",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return boolToInteger(lhs) & boolToInteger(rhs);
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 260-295
```cpp
  RegisterNNCLoweringsFunction aten___or__(
      {"aten::__or__.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::__or__.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_or",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return boolToInteger(lhs) | boolToInteger(rhs);
            });
      });

  RegisterNNCLoweringsFunction aten___xor__(
      {"aten::__xor__.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::__xor__.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_xor",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return boolToInteger(lhs) ^ boolToInteger(rhs);
            });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 296-331
```cpp
      });

  RegisterNNCLoweringsFunction aten___lshift__(
      {"aten::__lshift__.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::__lshift__.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_lshift",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return lhs << rhs;
            });
      });

  RegisterNNCLoweringsFunction aten___rshift__(
      {"aten::__rshift__.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::__rshift__.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_rshift",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 332-367
```cpp
              return lhs >> rhs;
            });
      });

  RegisterNNCLoweringsFunction aten_eq(
      {"aten::eq.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::eq.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_eq",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return cast<bool>(lhs == rhs);
            });
      });

  RegisterNNCLoweringsFunction aten_ne(
      {"aten::ne.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::ne.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_ne",
            inputs,
            outputShape,
            outputStrides,
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 368-391
```cpp
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return cast<bool>(lhs != rhs);
            });
      });

  RegisterNNCLoweringsFunction aten_ge(
      {"aten::ge.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::ge.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_ge",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return cast<bool>(lhs >= rhs);
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 393-428
```cpp
  RegisterNNCLoweringsFunction aten_gt(
      {"aten::gt.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::gt.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_gt",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return cast<bool>(lhs > rhs);
            });
      });

  RegisterNNCLoweringsFunction aten_le(
      {"aten::le.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::le.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_le",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return cast<bool>(lhs <= rhs);
            });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 429-464
```cpp
      });

  RegisterNNCLoweringsFunction aten_lt(
      {"aten::lt.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::lt.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_lt",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return cast<bool>(lhs < rhs);
            });
      });

  RegisterNNCLoweringsFunction aten_min_pointwise(
      {"aten::min.other(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_min",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return Min::make(boolToInteger(lhs), boolToInteger(rhs), false);
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 465-500
```cpp
            });
      });

  RegisterNNCLoweringsFunction aten_max_pointwise(
      {"aten::max.other(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_max",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return Max::make(boolToInteger(lhs), boolToInteger(rhs), false);
            });
      });

  RegisterNNCLoweringsFunction aten_masked_fill(
      {"aten::masked_fill.Scalar(Tensor self, Tensor mask, Scalar value) -> (Tensor)",
       "aten::masked_fill.Tensor(Tensor self, Tensor mask, Tensor value) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeThreeOperand(
            "aten_masked_fill",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& input,
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 501-525
```cpp
               const ExprHandle& mask,
               const ExprHandle& value) {
              // value needs to promote to input, not vice versa
              auto val = promoteToDtype(value, input.dtype().scalar_type());
              return ifThenElse(mask, val, input);
            },
            /*promote_inputs*/ false);
      });
  RegisterNNCLoweringsFunction aten_clamp(
      {"aten::clamp(Tensor self, Scalar? min=None, Scalar? max=None) -> (Tensor)",
       "aten::clamp.Tensor(Tensor self, Tensor? min=None, Tensor? max=None) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        bool noMin = false;
        bool noMax = false;
        if (std::get_if<ArgNone>(&inputs[1])) {
          noMin = true;
        }

        if (std::get_if<ArgNone>(&inputs[2])) {
          noMax = true;
        }
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 527-556
```cpp
        return computeThreeOperand(
            "aten_clamp",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [noMin, noMax](
                const ExprHandle& in,
                const ExprHandle& min,
                const ExprHandle& max) {
              auto cast = [&](const ExprHandle& e) {
                return Cast::make(in.dtype(), e);
              };

              if (noMin && noMax) {
                return in;
              } else if (noMin) {
                auto cmax = cast(max);
                return CompareSelect::make(in, cmax, cmax, in, kGT);
              } else if (noMax) {
                auto cmin = cast(min);
                return CompareSelect::make(in, cmin, cmin, in, kLT);
              } else {
                auto cmax = cast(max);
                auto cmin = cast(min);
                return clamp(cmin, cmax, in);
              }
            },
            false /* promote_inputs */);
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 558-593
```cpp
  RegisterNNCLoweringsFunction aten_addcmul(
      {"aten::addcmul(Tensor self, Tensor tensor1, Tensor tensor2, *, Scalar value=1) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeFourOperand(
            "aten_addcmul",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a0,
               const ExprHandle& a1,
               const ExprHandle& a2,
               const ExprHandle& a3) { return a0 + a3 * a1 * a2; });
      });

  RegisterNNCLoweringsFunction aten_sigmoid(
      {"aten::sigmoid(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        // check if the activation is quantized
        const BufHandle& x = std::get<BufHandle>(inputs[0]);
        if (x.node()->qscale()) {
          return computeQuantizedSigmoidExternalCall(
              inputs, outputShape, outputStrides, outputType, device);
        }
        return computeOneOperand(
            "aten_sigmoid",
            inputs,
            outputShape,
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 594-629
```cpp
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return sigmoid(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_silu(
      {"aten::silu(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_silu",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) { return a * sigmoid(a); });
      });

  RegisterNNCLoweringsFunction aten_reciprocal(
      {"aten::reciprocal(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_reciprocal",
            inputs,
            outputShape,
            outputStrides,
            outputType,
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 630-665
```cpp
            [](const ExprHandle& a) { return ExprHandle(1.0f) / a; });
      });

  RegisterNNCLoweringsFunction aten_neg(
      {"aten::neg(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_neg",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) { return ExprHandle(-0) - a; });
      });

  RegisterNNCLoweringsFunction aten_isnan(
      {"aten::isnan(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_isnan",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              if (!a.dtype().is_floating_point()) {
                return IntImm::make(0);
              }
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 666-692
```cpp
              return isnan(a);
            });
      });

  RegisterNNCLoweringsFunction aten_relu(
      {"aten::relu(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        auto A = std::get<BufHandle>(inputs[0]);
        if (A.node()->qscale()) {
          return computeQuantizedRelu(
              inputs, outputShape, outputStrides, outputType, device);
        }
        return computeOneOperand(
            "aten_relu",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              auto zero = Cast::make(a.dtype(), 0);
              return CompareSelect::make(a, zero, zero, a, kLT);
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 694-729
```cpp
  RegisterNNCLoweringsFunction aten_leaky_relu(
      {"aten::leaky_relu(Tensor self, Scalar negative_slope=0.01) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_leaky_relu",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a, const ExprHandle& negative_slope) {
              auto neg_slope = Cast::make(a.dtype(), negative_slope);
              auto zero = Cast::make(a.dtype(), 0);
              auto one = Cast::make(a.dtype(), 1);
              auto cs = CompareSelect::make(a, zero, one, neg_slope, kGT);
              return a * cs;
            });
      });

  RegisterNNCLoweringsFunction aten_relu6(
      {"aten::relu6(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_relu6",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 730-765
```cpp
              auto zero = Cast::make(a.dtype(), 0);
              auto six = Cast::make(a.dtype(), 6.);
              return clamp(zero, six, a);
            });
      });

  RegisterNNCLoweringsFunction aten_gelu(
      {"aten::gelu(Tensor self, *, str approximate='none') -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        const auto& kApproximate = std::get<std::string>(inputs[1]);
        std::vector<ArgValue> operands = {inputs.front()};
        if (at::native::get_gelutype_enum(kApproximate) ==
            at::native::GeluType::Tanh) {
          // approximate == 'tanh'
          return computeOneOperand(
              "aten_tanh_gelu",
              operands,
              outputShape,
              outputStrides,
              outputType,
              [](const ExprHandle& a) {
                auto one = Cast::make(a.dtype(), 1.);
                auto point_five = Cast::make(a.dtype(), .5);
                auto beta = Cast::make(a.dtype(), M_SQRT2 * M_2_SQRTPI * 0.5);
                auto kappa = Cast::make(a.dtype(), 0.044715);
                auto a_cube = a * a * a;
                auto inner = beta * (a + kappa * a_cube);
                return point_five * a * (one + tanh(inner));
              });
        } else {
          // approximate == 'none'
          return computeOneOperand(
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 766-800
```cpp
              "aten_gelu",
              operands,
              outputShape,
              outputStrides,
              outputType,
              [](const ExprHandle& a) {
                auto m_sqrt1_2 = Cast::make(a.dtype(), M_SQRT1_2);
                auto one = Cast::make(a.dtype(), 1.);
                auto point_five = Cast::make(a.dtype(), .5);
                return a * point_five * (one + erf(a * m_sqrt1_2));
              });
        }
      });

  RegisterNNCLoweringsFunction aten_batch_norm(
      {"aten::batch_norm(Tensor input, Tensor? weight, Tensor? bias, Tensor? running_mean, Tensor? running_var, bool training, float momentum, float eps, bool cudnn_enabled) -> (Tensor)"},
      computeBatchNorm);

  RegisterNNCLoweringsFunction aten_log(
      {"aten::log(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_log",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return log(promoteIntegerToDefaultType(a));
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 802-836
```cpp
  RegisterNNCLoweringsFunction aten_log10(
      {"aten::log10(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_log10",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return log10(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_log1p(
      {"aten::log1p(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_log1p",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return log1p(promoteIntegerToDefaultType(a));
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 838-872
```cpp
  RegisterNNCLoweringsFunction aten_log2(
      {"aten::log2(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_log2",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return log2(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_exp(
      {"aten::exp(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_exp",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return exp(promoteIntegerToDefaultType(a));
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 874-908
```cpp
  RegisterNNCLoweringsFunction aten_expm1(
      {"aten::expm1(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_expm1",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return expm1(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_erf(
      {"aten::erf(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_erf",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return erf(promoteIntegerToDefaultType(a));
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 910-944
```cpp
  RegisterNNCLoweringsFunction aten_erfc(
      {"aten::erfc(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_erfc",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return erfc(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_cos(
      {"aten::cos(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_cos",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return cos(promoteIntegerToDefaultType(a));
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 946-980
```cpp
  RegisterNNCLoweringsFunction aten_sin(
      {"aten::sin(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_sin",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return sin(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_tan(
      {"aten::tan(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_tan",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return tan(promoteIntegerToDefaultType(a));
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 982-1017
```cpp
  RegisterNNCLoweringsFunction aten_type_as(
      {"aten::type_as(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        const BufHandle& rhs = std::get<BufHandle>(inputs[1]);
        auto dtype = rhs.dtype();
        return computeOneOperand(
            "aten_type_as",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [dtype](const ExprHandle& lhs) { return Cast::make(dtype, lhs); });
      });

  RegisterNNCLoweringsFunction aten_pow(
      {"aten::pow.Tensor_Scalar(Tensor self, Scalar exponent) -> (Tensor)",
       "aten::pow.Tensor_Tensor(Tensor self, Tensor exponent) -> (Tensor)",
       "aten::pow.Scalar(Scalar self, Tensor exponent) -> Tensor"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_pow",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              if (!rhs.node()->isConstant()) {
                return pow(lhs, rhs);
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1018-1044
```cpp
              }
              double val =
                  immediateAs<double>(IRSimplifier::simplify(rhs.node()));

              if (val == 1.0f) {
                return lhs;
              } else if (val == 2.0f) { // NOLINT
                return lhs * lhs;
              } else if (val == 3.0f) { // NOLINT
                return (lhs * lhs) * lhs;
              } else if (val == 4.0f) { // NOLINT
                ExprHandle tmp = lhs * lhs;
                return tmp * tmp;
              } else if (val == 0.5f) { // NOLINT
                return sqrt(lhs);
              } else if (val == 0.0f) {
                return ExprHandle(1.0f);
              } else if (val == -0.5f) { // NOLINT
                return rsqrt(lhs);
              } else if (val == -1.0f) {
                return ExprHandle(1.0f) / lhs;
              } else if (val == -2.0f) { // NOLINT
                return ExprHandle(1.0f) / (lhs * lhs);
              }
              return pow(lhs, rhs);
            });
      });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1046-1081
```cpp
  RegisterNNCLoweringsFunction aten_fmod(
      {"aten::fmod.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::fmod.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_fmod",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return fmod(promoteHalfToFloat(lhs), promoteHalfToFloat(rhs));
            });
      });

  RegisterNNCLoweringsFunction aten_lerp(
      {"aten::lerp.Scalar(Tensor self, Tensor end, Scalar weight) -> (Tensor)",
       "aten::lerp.Tensor(Tensor self, Tensor end, Tensor weight) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeThreeOperand(
            "aten_lerp",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a,
               const ExprHandle& end,
               const ExprHandle& weight) { return a + weight * (end - a); });
```
- **EN**: This chunk continues `nnc_lowerings_lazy_registration` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `nnc_lowerings_lazy_registration`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1082-1110
```cpp
      });

  RegisterNNCLoweringsFunction aten_remainder(
      {"aten::remainder.Scalar(Tensor self, Scalar other) -> (Tensor)",
       "aten::remainder.Scalar_Tensor(Scalar self, Tensor other) -> (Tensor)",
       "aten::remainder.Tensor(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        auto imodImpl = [](const ExprHandle& lhs, const ExprHandle& rhs) {
          return Mod::make(lhs, rhs);
        };
        auto fmodImpl = [](const ExprHandle& lhs, const ExprHandle& rhs) {
          auto lhs_t = promoteHalfToFloat(lhs);
          auto rhs_t = promoteHalfToFloat(rhs);
          return fmod((rhs_t + fmod(lhs_t, rhs_t)), rhs_t);
        };
        {
          auto const& shape =
              broadcastShapes(valueShape(inputs[0]), valueShape(inputs[1]));
          return Compute(
              "aten_remainder", shape, [&](const std::vector<VarHandle>& axes) {
                std::vector<ExprHandle> indices(axes.begin(), axes.end());
                std::vector<ExprHandle> exprInputs = {
                    tensorOrConstant(inputs[0], indices),
                    tensorOrConstant(inputs[1], indices),
                };
```
- **EN**: This chunk defines `indices`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `indices`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1112-1147
```cpp
                promoteInputs(exprInputs);
                bool allInt = true;
                for (auto& e : exprInputs) {
                  if (e.dtype().is_floating_point()) {
                    allInt = false;
                    break;
                  }
                }
                if (allInt) {
                  return demoteOutput(
                      imodImpl(exprInputs[0], exprInputs[1]), outputType);
                } else {
                  return demoteOutput(
                      fmodImpl(exprInputs[0], exprInputs[1]), outputType);
                }
              });
        }
      });

  RegisterNNCLoweringsFunction prim_ConstantChunk(
      {"prim::ConstantChunk(...) -> (...)"}, computeChunk);

  RegisterNNCLoweringsFunction aten_acos(
      {"aten::acos(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_acos",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1148-1183
```cpp
              return acos(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_asin(
      {"aten::asin(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_asin",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return asin(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_cosh(
      {"aten::cosh(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_cosh",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1184-1219
```cpp
              return cosh(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_sinh(
      {"aten::sinh(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_sinh",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return sinh(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_atan(
      {"aten::atan(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_atan",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1220-1242
```cpp
              return atan(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_atan2(
      {"aten::atan2(Tensor self, Tensor other) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_atan2",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& lhs, const ExprHandle& rhs) {
              return atan2(
                  promoteIntegerToDefaultType(lhs),
                  promoteIntegerToDefaultType(rhs));
            });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1244-1279
```cpp
  RegisterNNCLoweringsFunction aten_tanh(
      {"aten::tanh(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_tanh",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return tanh(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_hardtanh(
      {"aten::hardtanh(Tensor self, Scalar min_val=-1, Scalar max_val=1) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeThreeOperand(
            "aten_hardtanh",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a,
               const ExprHandle& min_val,
               const ExprHandle& max_val) {
              auto mm = CompareSelect::make(a, min_val, min_val, a, kLT);
              return CompareSelect::make(mm, max_val, max_val, mm, kGT);
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1280-1309
```cpp
            });
      });

  RegisterNNCLoweringsFunction aten_softplus(
      {"aten::softplus(Tensor self, Scalar beta=1, Scalar threshold=20) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeThreeOperand(
            "aten_softplus",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a,
               const ExprHandle& beta,
               const ExprHandle& threshold) {
              auto beta_promoted = Cast::make(a.dtype(), beta);
              auto threshold_promoted = Cast::make(a.dtype(), threshold);
              auto beta_a = beta_promoted * a;
              return CompareSelect::make(
                  beta_a,
                  threshold_promoted,
                  a,
                  log1p(exp(beta_a)) / beta_promoted,
                  kGT);
            });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1311-1346
```cpp
  RegisterNNCLoweringsFunction aten_mish(
      {"aten::mish(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_mish",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              auto default_type_a = promoteIntegerToDefaultType(a);
              return default_type_a * tanh(log1p(exp(default_type_a)));
            });
      });

  RegisterNNCLoweringsFunction aten_elu(
      {"aten::elu(Tensor self, Scalar alpha=1, Scalar scale=1, Scalar input_scale=1) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeFourOperand(
            "aten_elu",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a,
               const ExprHandle& alpha,
               const ExprHandle& scale,
               const ExprHandle& input_scale) {
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1347-1382
```cpp
              auto zero = Cast::make(a.dtype(), 0);
              auto one = Cast::make(a.dtype(), 1);

              auto poscoef = Cast::make(a.dtype(), scale);
              auto negiptcoef = Cast::make(a.dtype(), input_scale);
              auto negcoef = Cast::make(a.dtype(), alpha) * poscoef;

              return CompareSelect::make(
                  a,
                  zero,
                  a * poscoef,
                  (exp(a * negiptcoef) - one) * negcoef,
                  kGT);
            });
      });

  RegisterNNCLoweringsFunction aten_hardsigmoid(
      {"aten::hardsigmoid(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_hardsigmoid",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              auto zero = Cast::make(a.dtype(), 0.0);
              auto three = Cast::make(a.dtype(), 3.0);
              auto six = Cast::make(a.dtype(), 6.0);
              return clamp(zero, six, a + three) / six;
            });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1384-1419
```cpp
  RegisterNNCLoweringsFunction aten_hardswish(
      {"aten::hardswish(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_hardswish",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              //  x * torch.clamp(x + 3.0, 0.0, 6.0) / 6.0
              auto zero = Cast::make(a.dtype(), 0.);
              auto three = Cast::make(a.dtype(), 3.);
              auto six = Cast::make(a.dtype(), 6.);

              return a * clamp(zero, six, a + three) / six;
            });
      });

  RegisterNNCLoweringsFunction aten_hardshrink(
      {"aten::hardshrink(Tensor self, Scalar lambd=0.5) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTwoOperand(
            "aten_hardshrink",
            inputs,
            outputShape,
            outputStrides,
            outputType,
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1420-1446
```cpp
            [](const ExprHandle& a, const ExprHandle& lambd) {
              auto pos_clambd = Cast::make(a.dtype(), lambd);
              auto neg_clambd =
                  Cast::make(a.dtype(), ExprHandle(-0)) - pos_clambd;
              auto zero = Cast::make(a.dtype(), 0);
              auto mm = CompareSelect::make(a, neg_clambd, a, zero, kLT);
              return CompareSelect::make(a, pos_clambd, a, mm, kGT);
            });
      });

  RegisterNNCLoweringsFunction aten_sqrt(
      {"aten::sqrt(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_sqrt",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return tensorexpr::sqrt(promoteIntegerToDefaultType(a));
            });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1448-1483
```cpp
  RegisterNNCLoweringsFunction aten_rsqrt(
      {"aten::rsqrt(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_rsqrt",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return rsqrt(promoteIntegerToDefaultType(a));
            });
      });

  RegisterNNCLoweringsFunction aten_abs(
      {"aten::abs(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_abs",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return tensorexpr::abs(promoteHalfToFloat(a));
            },
            kIntegralTypes | kFloatingPointTypes | kBoolType);
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1485-1507
```cpp
  RegisterNNCLoweringsFunction aten_sign(
      {"aten::sign(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) { return computeSign(inputs, outputShape); });

  RegisterNNCLoweringsFunction aten_ceil(
      {"aten::ceil(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_ceil",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) { return ceil(a); });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1509-1539
```cpp
  RegisterNNCLoweringsFunction aten_floor(
      {"aten::floor(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_floor",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) { return floor(a); });
      });

  RegisterNNCLoweringsFunction aten_round(
      {"aten::round(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_round",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) { return round(a); });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1541-1571
```cpp
  RegisterNNCLoweringsFunction aten_trunc(
      {"aten::trunc(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_trunc",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) { return trunc(a); });
      });

  RegisterNNCLoweringsFunction aten__cast_Float(
      {"aten::_cast_Float(Tensor self, bool non_blocking=False) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_cast_float",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) { return cast<float>(a); });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1573-1599
```cpp
  RegisterNNCLoweringsFunction aten_to(
      {"aten::to.dtype(Tensor(a) self, int dtype, bool non_blocking=False, bool copy=False, int? memory_format=None) -> (Tensor(a))",
       "aten::to.dtype_layout(Tensor(a) self, *, int? dtype=None, int? layout=None, Device? device=None, bool? pin_memory=None, bool non_blocking=False, bool copy=False, int? memory_format=None) -> (Tensor(a))",
       "aten::to.device(Tensor(a) self, Device device, int dtype, bool non_blocking=False, bool copy=False, int? memory_format=None) -> (Tensor(a))",
       "aten::to.prim_Device(Tensor(a) self, Device? device, int? dtype=None, bool non_blocking=False, bool copy=False) -> Tensor(a|b)",
       "aten::to.prim_dtype(Tensor(a) self, int? dtype=None, bool non_blocking=False, bool copy=False) -> Tensor(a|b)",
       "aten::_autocast_to_reduced_precision(Tensor(a) self, bool cuda_enabled, bool cpu_enabled, ScalarType cuda_dtype, ScalarType cpu_dtype) -> Tensor(a)",
       "aten::_autocast_to_full_precision(Tensor(a) self, bool cuda_enabled, bool cpu_enabled) -> Tensor(a)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        // see handling of aten::to in tensorexpr_fuser.cpp for why we only
        // need to handle the first input
        return computeOneOperand(
            "aten_to",
            {inputs[0]},
            outputShape,
            outputStrides,
            outputType,
            [outputType](const ExprHandle& a) {
              TORCH_INTERNAL_ASSERT(
                  outputType, buildErrorMessage("Output type is null."));
              return Cast::make(ToDtype(*outputType), a);
            });
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1601-1636
```cpp
  RegisterNNCLoweringsFunction aten_threshold(
      {"aten::threshold(Tensor self, Scalar threshold, Scalar value) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeThreeOperand(
            "aten_threshold",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a,
               const ExprHandle& threshold,
               const ExprHandle& value) {
              return ifThenElse(
                  CompareSelect::make(a, threshold, kLE), value, a);
            });
      });

  RegisterNNCLoweringsFunction aten_where(
      {"aten::where.ScalarOther(Tensor condition, Tensor self, Scalar other) -> (Tensor)",
       "aten::where.ScalarSelf(Tensor condition, Scalar self, Tensor other) -> (Tensor)",
       "aten::where.self(Tensor condition, Tensor self, Tensor other) -> (Tensor)",
       "aten::where.Scalar(Tensor condition, Scalar self, Scalar other) -> Tensor"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeConditionWithTwoOperand(
            "aten_where",
            inputs,
            outputShape,
            outputStrides,
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1637-1661
```cpp
            outputType,
            [](const ExprHandle& a0,
               const ExprHandle& a1,
               const ExprHandle& a2) { return ifThenElse(a0, a1, a2); });
      });

  RegisterNNCLoweringsFunction aten_frac(
      {"aten::frac(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_frac",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              auto aa = promoteHalfToFloat(a);
              return aa - floor(aa);
            },
            kFloatingPointTypes);
      });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1663-1696
```cpp
  RegisterNNCLoweringsFunction aten_lgamma(
      {"aten::lgamma(Tensor self) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeOneOperand(
            "aten_lgamma",
            inputs,
            outputShape,
            outputStrides,
            outputType,
            [](const ExprHandle& a) {
              return lgamma(promoteIntegerToDefaultType(a));
            });
      });

  // TODO: convert to schema, add a test
  // RegisterNNCLoweringsFunction aten_rand_like(
  //     {"aten::rand_like"},
  //     [](const std::vector<ArgValue>& inputs,
  //        const std::vector<ExprHandle>& outputShape,
  //        const std::optional<ScalarType>& outputType,
  //        at::Device device) {
  //       return computeOneOperand(
  //           "aten_rand_like",
  //           inputs,
  //           outputShape,
  //           outputType,
  //           [](const ExprHandle& a) {
  //             return Intrinsics::make(IntrinsicsOp::kRand, a.dtype());
  //           });
  //     });
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1698-1733
```cpp
  // TODO: convert to schema, add a test
  // RegisterNNCLoweringsFunction aten_slice(
  //     {"aten::slice"},
  //     [](const std::vector<ArgValue>& inputs,
  //        const std::vector<ExprHandle>& outputShape,
  //        const std::optional<ScalarType>& outputType,
  //        at::Device device) {
  //       return Compute(
  //           "aten_slice",
  //           outputShape,
  //           [&](const std::vector<VarHandle>& axes) {
  //             int64_t dim =
  //                 at::maybe_wrap_dim(std::get<int64_t>(inputs[1]),
  //                 axes.size());
  //             ExprHandle start = constant(inputs[2]);
  //             ExprHandle stride = constant(inputs[4]);

  //             std::vector<ExprHandle> newAxes(axes.begin(), axes.end());
  //             newAxes[dim] = stride * newAxes[dim] + start;
  //             return tensorOrConstant(inputs[0], newAxes);
  //           });
  //     });
  RegisterNNCLoweringsFunction aten_unsqueeze(
      {"aten::unsqueeze(Tensor(a) self, int dim) -> (Tensor(a))"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return Compute(
            "aten_unsqueeze",
            outputShape,
            outputStrides,
            [&](const std::vector<VarHandle>& axes) {
              int64_t dim = std::get<int64_t>(inputs[1]);
              if (dim < 0) {
```
- **EN**: This chunk continues `indices` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `indices`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1734-1769
```cpp
                if (axes.empty()) {
                  throw malformed_input("axes are zero handling unsqueeze");
                }
                dim += axes.size();
              }
              // To construct an expression for an 'unsqueezed' tensor we need
              // to drop the DIM-th axis, i.e.
              //    unsqueezed_v[i,j,k,l] = v[i,j,l] # dim = 2 - drop index 'k'
              //                 0 1 2 3
              std::vector<ExprHandle> indices;
              int64_t i = 0;
              for (const auto& a : axes) {
                if (i++ != dim) {
                  indices.emplace_back(a.node());
                }
              }

              return broadcast(std::get<BufHandle>(inputs[0]), indices);
            });
      });
  RegisterNNCLoweringsFunction aten_t(
      {"aten::t(Tensor(a) self) -> (Tensor(a))"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeTranspose(
            {inputs[0], (int64_t)1, (int64_t)0},
            outputShape,
            outputStrides,
            outputType,
            device);
      });
  RegisterNNCLoweringsFunction aten_transpose(
      {"aten::transpose.int(Tensor(a) self, int dim0, int dim1) -> (Tensor(a))"},
```
- **EN**: This chunk defines `malformed_input`, which implements a focused step in Tensor Expression analysis, lowering, or code generation. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `malformed_input`，其作用是实现 Tensor Expression 分析、降级或代码生成中的一个关键步骤。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1770-1805
```cpp
      computeTranspose);
  RegisterNNCLoweringsFunction aten_permute(
      {"aten::permute(Tensor(a) self, int[] dims) -> (Tensor(a))"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        auto A = std::get<BufHandle>(inputs[0]);
        // Trivial case of 0-dim tensors: just a copy of the input
        if (A.ndim() == 0) {
          auto tensor = Compute(
              "aten_permute",
              outputShape,
              outputStrides,
              [&](const std::vector<VarHandle>& axes) {
                std::vector<ExprHandle> empty_indices;
                return A.load(empty_indices);
              });
          if (A.node()->qscale()) {
            tensor.buf()->set_qscale(A.node()->qscale());
            tensor.buf()->set_qzero(A.node()->qzero());
          }
          return tensor;
        }
        auto permute_dims = std::get<IntList>(inputs[1]);
        auto tensor = Compute(
            "aten_permute",
            outputShape,
            [&](const std::vector<VarHandle>& axes) {
              std::vector<VarHandle> new_axes;
              new_axes.resize(axes.size());
              assert(permute_dims.size() == axes.size());
              for (unsigned i = 0; i < axes.size(); i++) {
                auto new_dim = at::maybe_wrap_dim(permute_dims[i], A.ndim());
                new_axes[new_dim] = axes[i];
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Looping logic walks collections, IR nodes, or table entries so the same rule can be applied systematically. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 循环逻辑遍历集合、IR 节点或表项，从而把同一规则系统地应用到每个元素。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1806-1838
```cpp
              }
              return A.load(new_axes);
            });
        if (A.node()->qscale()) {
          tensor.buf()->set_qscale(A.node()->qscale());
          tensor.buf()->set_qzero(A.node()->qzero());
        }
        return tensor;
      });
  RegisterNNCLoweringsFunction aten_expand(
      {"aten::expand(Tensor(a) self, int[] size, *, bool implicit=False) -> (Tensor(a))",
       "aten::expand_as(Tensor(a) self, Tensor other) -> (Tensor(a))"},
      computeExpand);

  // TODO: add a test
  RegisterNNCLoweringsFunction aten_flatten(
      {"aten::flatten.using_ints(Tensor(a) self, int start_dim=0, int end_dim=-1) -> (Tensor(a))"},
      computeFlatten);
  RegisterNNCLoweringsFunction aten_view(
      {"aten::reshape(Tensor(a) self, int[] shape) -> (Tensor(a))",
       "aten::reshape_as(Tensor(a) self, Tensor other) -> (Tensor(a))",
       "aten::view(Tensor(a) self, int[] size) -> (Tensor(a))",
       "aten::view_as(Tensor(a) self, Tensor other) -> (Tensor(a))"},
      computeReshape);

  // aten::mm is a subset of aten::matmul where both inputs are rank 2
  RegisterNNCLoweringsFunction aten_matmul(
      {"aten::mm(Tensor self, Tensor mat2) -> (Tensor)",
       "aten::matmul(Tensor self, Tensor other) -> (Tensor)"},
      computeMatmul);

  RegisterNNCLoweringsFunction aten_cat(
      {"aten::cat(Tensor[] tensors, int dim=0) -> (Tensor)"}, computeCat);
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1840-1874
```cpp
  RegisterNNCLoweringsFunction aten_sum(
      {"aten::sum(Tensor self, *, int? dtype=None) -> (Tensor)",
       "aten::sum.dim_IntList(Tensor self, int[1]? dim, bool keepdim=False, *, int? dtype=None) -> (Tensor)"},
      computeSum);

  RegisterNNCLoweringsFunction aten_softmax(
      {"aten::softmax.int(Tensor self, int dim, int? dtype=None) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeSoftmax(inputs, outputShape, outputStrides, false);
      });

  RegisterNNCLoweringsFunction aten_log_softmax(
      {"aten::log_softmax.int(Tensor self, int dim, int? dtype=None) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        return computeSoftmax(inputs, outputShape, outputStrides, true);
      });

  RegisterNNCLoweringsFunction aten_conv1d(
      {"aten::conv1d(Tensor input, Tensor weight, Tensor? bias=None, int[1] stride=1, int[1] padding=0, int[1] dilation=1, int groups=1) -> (Tensor)"},
      computeConv1d);
  RegisterNNCLoweringsFunction aten_conv2d(
      {"aten::conv2d(Tensor input, Tensor weight, Tensor? bias=None, int[2] stride=[1, 1], int[2] padding=[0, 0], int[2] dilation=[1, 1], int groups=1) -> (Tensor)"},
      computeConv2d);

  RegisterNNCLoweringsFunction aten_addmm(
      {"aten::addmm(Tensor self, Tensor mat1, Tensor mat2, *, Scalar beta=1, Scalar alpha=1) -> (Tensor)"},
      computeAddMM);
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1876-1911
```cpp
  RegisterNNCLoweringsFunction aten_mean(
      {"aten::mean(Tensor self, *, int? dtype=None) -> (Tensor)",
       "aten::mean.dim(Tensor self, int[1]? dim, bool keepdim=False, *, int? dtype=None) -> (Tensor)"},
      computeMean);
  RegisterNNCLoweringsFunction aten_max_reduction(
      {"aten::max.dim(Tensor self, int dim, bool keepdim=False) -> (Tensor values, Tensor indices)"},
      computeMax);

  RegisterNNCLoweringsFunction aten_adaptive_avg_pool2d(
      {"aten::adaptive_avg_pool2d(Tensor self, int[2] output_size) -> (Tensor)"},
      computeAdaptiveAvgPool2d);

  RegisterNNCLoweringsFunction aten_add(
      {"aten::add.Scalar(Tensor self, Scalar other, Scalar alpha=1) -> (Tensor)",
       "aten::add.Tensor(Tensor self, Tensor other, *, Scalar alpha=1) -> (Tensor)"},
      [](const std::vector<ArgValue>& inputs,
         const std::vector<ExprHandle>& outputShape,
         const std::vector<ExprHandle>& outputStrides,
         const std::optional<ScalarType>& outputType,
         at::Device device) {
        auto add_lambda = [](const ExprHandle& lhs, const ExprHandle& rhs) {
          return boolToInteger(lhs) + boolToInteger(rhs);
        };
        TORCH_INTERNAL_ASSERT(
            inputs.size() == 2 || inputs.size() == 3,
            buildErrorMessage("Invalid number of input operands"));
        return (inputs.size() > 2) ? computeTwoOperandWithAlpha(
                                         "aten_add",
                                         inputs,
                                         outputShape,
                                         outputStrides,
                                         outputType,
                                         add_lambda)
                                   : computeTwoOperand(
                                         "aten_add",
                                         inputs,
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement. Assertions and error paths make invalid states fail early instead of silently corrupting later stages. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 断言与错误路径让非法状态尽早失败，避免悄悄破坏后续阶段。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 1912-1945
```cpp
                                         outputShape,
                                         outputStrides,
                                         outputType,
                                         add_lambda);
      });
  RegisterNNCLoweringsFunction aten_embedding(
      {"aten::embedding(Tensor weight, Tensor indices, int padding_idx=-1, bool scale_grad_by_freq=False, bool sparse=False) -> Tensor"},
      computeEmbedding);

#define NNC_QUANTIZATION_EXPR_QUANT 1
#define NNC_QUANTIZATION_EXPR_DEQUANT 1

  RegisterNNCLoweringsFunction aten_quantize_per_tensor(
      {"aten::quantize_per_tensor(Tensor self, float scale, int zero_point, int dtype) -> (Tensor)",
       "aten::quantize_per_tensor.tensor_qparams(Tensor self, Tensor scale, Tensor zero_point, int dtype) -> (Tensor)",
       "aten::quantize_per_tensor.tensors(Tensor[] tensors, Tensor scales, Tensor zero_points, int dtype) -> (Tensor[])"},
#if NNC_QUANTIZATION_EXPR_QUANT == 1
      computeQuantizePerTensor
#else
      computeQuantizePerTensorExternalCall
#endif
  );

  RegisterNNCLoweringsFunction aten_dequantize(
      {"aten::dequantize.self(Tensor self) -> (Tensor)"},
#if NNC_QUANTIZATION_EXPR_DEQUANT == 1
      computeDequantize
#else
      computeDequantizeExternalCall
#endif
  );
  RegisterNNCLoweringsFunction quantized_conv1d(
      {"quantized::conv1d(Tensor qx, __torch__.torch.classes.quantized.Conv2dPackedParamsBase packed_weight, float output_scale, int output_zero_point) -> (Tensor)"},
      computeQuantizedConv1d);
```
- **EN**: The preprocessor guard keeps declarations single-instanced when the header is included transitively. This chunk continues `malformed_input` and expands its internal control flow or data movement. Conditional branches guard special cases and preserve type, shape, or serialization invariants.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。

### Lines 1947-1981
```cpp
  RegisterNNCLoweringsFunction quantized_conv2d(
      {"quantized::conv2d.new(Tensor qx, __torch__.torch.classes.quantized.Conv2dPackedParamsBase packed_weight, float output_scale, int output_zero_point) -> (Tensor)"},
      computeQuantizedConv2d);

  RegisterNNCLoweringsFunction quantized_conv2d_relu(
      {"quantized::conv2d_relu.new(Tensor qx, __torch__.torch.classes.quantized.Conv2dPackedParamsBase packed_weight, float output_scale, int output_zero_point) -> (Tensor)"},
      computeQuantizedConv2dRelu);

  RegisterNNCLoweringsFunction quantized_linear(
      {"quantized::linear(Tensor X, __torch__.torch.classes.quantized.LinearPackedParamsBase W_prepack, float Y_scale_i, int Y_zero_point_i) -> (Tensor Y)"},
      computeQuantizedLinear);

  RegisterNNCLoweringsFunction quantized_linear_relu(
      {"quantized::linear_relu(Tensor X, __torch__.torch.classes.quantized.LinearPackedParamsBase W_prepack, float Y_scale_i, int Y_zero_point_i) -> (Tensor Y)"},
      computeQuantizedLinear);

  RegisterNNCLoweringsFunction quantized_add(
      {"quantized::add(Tensor qa, Tensor qb, float scale, int zero_point) -> (Tensor qc)"},
      computeQuantizedAdd);

  RegisterNNCLoweringsFunction quantized_mul(
      {"quantized::mul(Tensor qa, Tensor qb, float scale, int zero_point) -> (Tensor qc)"},
      computeQuantizedMul);

  RegisterNNCLoweringsFunction quantized_mul_scalar(
      {"quantized::mul.Scalar(Tensor qa, Scalar b) -> (Tensor qc)"},
      computeQuantizedMulScalar);

  RegisterNNCLoweringsFunction quantized_conv2d_prepack(
      {"quantized::conv2d_prepack(Tensor weight, Tensor? bias, int[] stride, int[] padding, int[] dilation, int groups) -> (__torch__.torch.classes.quantized.Conv2dPackedParamsBase)"},
      computeQuantizedConv2dPrepack);

  RegisterNNCLoweringsFunction quantized_cat(
      {"quantized::cat(Tensor[] qx, int dim, float? scale, int? zero_point) -> (Tensor)"},
      computeQuantizedCat);
```
- **EN**: This chunk continues `malformed_input` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `malformed_input`，进一步展开其内部控制流或数据流转。

### Lines 1983-2000
```cpp
  RegisterNNCLoweringsFunction aten_upsample_nearest2d(
      {"aten::upsample_nearest2d.vec(Tensor input, int[]? output_size, float[]? scale_factors) -> (Tensor)"},
      computeUpsampleNearest2dExternalCall);

  return 0;
}
} // namespace

NNCLoweringFunction getStandardLoweringFor(const std::string& schema_str) {
  [[maybe_unused]] static const int once = nnc_lowerings_lazy_registration();
  const auto& lowerings = getNNCLoweringRegistry();
  if (auto l = lowerings.find(parseSchema(schema_str))) {
    return *l;
  }
  return nullptr;
}

} // namespace torch::jit::tensorexpr
```
- **EN**: This chunk defines `getStandardLoweringFor`, which lowers higher-level IR into a backend-specific executable form. Conditional branches guard special cases and preserve type, shape, or serialization invariants. The tail returns the accumulated result or hands the updated state to the caller.
- **CN**: 这一段定义了 `getStandardLoweringFor`，其作用是把高层 IR 降级为后端特定的可执行形式。 条件分支用于处理特殊情况，并保持类型、形状或序列化不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

## Key Concepts / 关键概念

- **Tensor Expression IR**
  - EN: Implements Tensor Expression IR, analyses, scheduling, lowering, and backend code generation.
  - CN: 实现 Tensor Expression IR、分析、调度、降级以及后端代码生成。
- **getNNCLoweringRegistry**
  - EN: `getNNCLoweringRegistry` is a central symbol declared or implemented in this file.
  - CN: `getNNCLoweringRegistry` 是本文件声明或实现的核心符号。
- **nnc_lowerings_lazy_registration**
  - EN: `nnc_lowerings_lazy_registration` is a central symbol declared or implemented in this file.
  - CN: `nnc_lowerings_lazy_registration` 是本文件声明或实现的核心符号。
- **Operator dispatch**
  - EN: Bridges symbolic operator names or schemas to concrete C++ implementations.
  - CN: 把符号化算子名或 schema 连接到具体的 C++ 实现。
- **IR simplification**
  - EN: Rewrites expressions and statements into simpler but equivalent forms.
  - CN: 把表达式和语句重写为更简单但等价的形式。
## Dependencies / 依赖关系

- **Internal PyTorch headers / 内部 PyTorch 头文件**: `torch/csrc/jit/frontend/function_schema_parser.h`, `torch/csrc/jit/tensorexpr/ir_simplifier.h`, `torch/csrc/jit/tensorexpr/lowerings.h`, `torch/csrc/jit/tensorexpr/operators/operators.h`
- **ATen/c10 foundations / ATen/c10 基础设施**: `ATen/native/Activation.h`, `ATen/native/mkldnn/Common.h`
- **Primary symbols in this file / 本文件核心符号**: `getNNCLoweringRegistry`, `nnc_lowerings_lazy_registration`
