# kernel_lambda_legacy_test.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/boxing/impl/kernel_lambda_legacy_test.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `RegisterOperators`, `op`, `myfunc`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `RegisterOperators`, `op`, `myfunc`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-24
```cpp
#include <gtest/gtest.h>

// This intentionally tests a deprecated API
#pragma GCC diagnostic push
#pragma GCC diagnostic ignored "-Wdeprecated-declarations"

#include <ATen/core/boxing/impl/test_helpers.h>
#include <ATen/core/op_registration/op_registration.h>
#include <ATen/core/Tensor.h>
#include <torch/csrc/jit/frontend/function_schema_parser.h>

#include <ATen/core/LegacyTypeDispatch.h>

/**
 * This file tests the legacy lambda-based API for registering kernels:
 *
 * > auto registry = c10::RegisterOperators()
 * >    .op("myfunc(Tensor a) -> Tensor", [] (Tensor a) -> Tensor {...});
 */

using c10::RegisterOperators;
using c10::DispatchKey;
using c10::Stack;
using std::make_unique;
```
- EN: Focus symbols: `RegisterOperators`, `op`, `myfunc`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`RegisterOperators`, `op`, `myfunc`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 25-50
```cpp
using c10::intrusive_ptr;
using c10::Dict;
using at::Tensor;
using std::string;
using std::unique_ptr;

namespace {

void expectCallsIncrement(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  auto result = callOp(*op, dummyTensor(dispatch_key), 5);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(6, result[0].toInt());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernel_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", [] (const Tensor& tensor, int64_t input) -> int64_t {
      return input + 1;
    });
  expectCallsIncrement(DispatchKey::CPU);
}

```
- EN: Focus symbols: `expectCallsIncrement`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`expectCallsIncrement`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 51-74
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernel_whenRegisteredInConstructor_thenCanBeCalled) {
  auto registrar = RegisterOperators("_test::my_op(Tensor dummy, int input) -> int", [] (const Tensor& tensor, int64_t input) -> int64_t {
      return input + 1;
    });
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInOneRegistrar_thenCallsRightKernel) {
  auto registrar = RegisterOperators()
      .op("_test::my_op(Tensor dummy, int input) -> int", [] (const Tensor& tensor, int64_t input) -> int64_t {
          return input + 1;
        })
      .op("_test::error(Tensor dummy, int input) -> int", [] (const Tensor& tensor, int64_t input) -> int64_t {
          EXPECT_TRUE(false); // this kernel should never be called
          return 0;
        });
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInMultipleRegistrars_thenCallsRightKernel) {
  auto registrar1 = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", [] (const Tensor& tensor, int64_t input) -> int64_t {
      return input + 1;
    });
  auto registrar2 = RegisterOperators().op("_test::error(Tensor dummy, int input) -> int", [] (const Tensor& tensor, int64_t input) -> int64_t {
```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `my_op`, `expectCallsIncrement`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `my_op`, `expectCallsIncrement`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 75-100
```cpp
      EXPECT_TRUE(false); // this kernel should never be called
      return 0;
    });
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernel_whenRegistrationRunsOutOfScope_thenCannotBeCalledAnymore) {
  {
    auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", [] (const Tensor& tensor, int64_t input) -> int64_t {
        return input + 1;
      });

    expectCallsIncrement(DispatchKey::CPU);
  }

  // now the registrar is destructed. Assert that the schema is gone.
  expectDoesntFindOperator("_test::my_op");
}

bool was_called = false;

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::no_return(Tensor dummy) -> ()", [] (const Tensor&) -> void {
    was_called = true;
  });

```
- EN: Focus symbols: `EXPECT_TRUE`, `expectCallsIncrement`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_TRUE`, `expectCallsIncrement`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 101-128
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_return", ""});
  ASSERT_TRUE(op.has_value());
  was_called = false;
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_TRUE(was_called);
  EXPECT_EQ(0, result.size());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithZeroOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::zero_outputs(Tensor dummy) -> ()", [] (const Tensor&) -> std::tuple<> {
    was_called = true;
    return std::make_tuple();
  });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::zero_outputs", ""});
  ASSERT_TRUE(op.has_value());
  was_called = false;
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_TRUE(was_called);
  EXPECT_EQ(0, result.size());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithIntOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_output(Tensor dummy, int a, int b) -> int", [] (Tensor, int64_t a, int64_t b) -> int64_t {
        return a + b;
      });

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 129-154
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_output", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), 3, 6);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(9, result[0].toInt());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::returning_tensor(Tensor input) -> Tensor", [] (const Tensor& input) -> Tensor {
        return input;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::returning_tensor", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensor()));

  result = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensor()));
}

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 155-180
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Tensor input1, Tensor input2, Tensor input3) -> Tensor[]", [] (const Tensor& input1, const Tensor& input2, const Tensor& input3) -> std::vector<Tensor> {
        return {input1, input2, input3};
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(3, result[0].toTensorVector().size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensorVector()[0]));
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensorVector()[1]));
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensorVector()[2]));
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithIntListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Tensor dummy, int input1, int input2, int input3) -> int[]", [](const Tensor&, int64_t input1, int64_t input2, int64_t input3) -> std::vector<int64_t> {
        return {input1, input2, input3};
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `list_output`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `list_output`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 181-206
```cpp
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), 2, 4, 6);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(3, result[0].toIntVector().size());
  EXPECT_EQ(2, result[0].toIntVector()[0]);
  EXPECT_EQ(4, result[0].toIntVector()[1]);
  EXPECT_EQ(6, result[0].toIntVector()[2]);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithMultipleOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
     .op("_test::multiple_outputs(Tensor dummy) -> (Tensor, int, Tensor[], int?, Dict(str, Tensor))", [] (Tensor) -> std::tuple<Tensor, int64_t, std::vector<Tensor>, std::optional<int64_t>, Dict<string, Tensor>> {
       Dict<string, Tensor> dict;
       dict.insert("first", dummyTensor(DispatchKey::CPU));
       dict.insert("second", dummyTensor(DispatchKey::CUDA));
       return std::tuple<Tensor, int64_t, std::vector<Tensor>, std::optional<int64_t>, Dict<string, Tensor>>(
         dummyTensor(DispatchKey::CUDA),
         5,
         {dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA)},
         std::optional<int64_t>(std::in_place, 0),
         dict
       );
     });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::multiple_outputs", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toIntVector`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toIntVector`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 207-233
```cpp
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(5, result.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensor()));
  EXPECT_EQ(5, result[1].toInt());
  EXPECT_EQ(2, result[2].toTensorVector().size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[2].toTensorVector()[0]));
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[2].toTensorVector()[1]));
  EXPECT_EQ(0, result[3].toInt());
  auto result_dict = c10::impl::toTypedDict<string, Tensor>(result[4].toGenericDict());
  EXPECT_EQ(2, result_dict.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result_dict.at("first")));
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result_dict.at("second")));
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorInputByReference_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> Tensor", [] (const Tensor& input1) -> Tensor {
        return input1;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensor()));

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 234-258
```cpp
  result = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensor()));
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorInputByValue_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> Tensor", [](Tensor input1) -> Tensor {
        return input1;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensor()));

  result = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensor()));
}

Tensor captured_input;

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 259-285
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorInputByReference_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> ()", [] (const Tensor& input1) -> void {
        captured_input = input1;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(captured_input));

  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(captured_input));
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorInputByValue_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> ()", [] (Tensor input1) -> void {
        captured_input = input1;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `tensor_input`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `tensor_input`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 286-311
```cpp
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(captured_input));

  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(captured_input));
}

int64_t captured_int_input = 0;

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithIntInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_input(Tensor dummy, int input) -> ()", [](Tensor, int64_t input1) -> void {
        captured_int_input = input1;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_int_input = 0;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 3);
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(3, captured_int_input);
}

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 312-336
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithIntInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_input(Tensor dummy, int input) -> int", [] (Tensor, int64_t input1) -> int64_t {
        return input1 + 1;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 3);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(4, outputs[0].toInt());
}

int64_t captured_input_list_size = 0;

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithIntListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_list_input(Tensor dummy, int[] input) -> ()", [] (Tensor, const std::vector<int64_t>& input1) -> void {
        captured_input_list_size = input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_list_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `int_input`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `int_input`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 337-362
```cpp
  captured_input_list_size = 0;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::List<int64_t>({2, 4, 6}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(3, captured_input_list_size);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithIntListInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_list_input(Tensor dummy, int[] input) -> int", [](Tensor, const std::vector<int64_t>& input1) -> int64_t {
        return input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::List<int64_t>({2, 4, 6}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(3, outputs[0].toInt());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> ()", [] (const std::vector<Tensor>& input1) -> void {
        captured_input_list_size = input1.size();
      });

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 363-386
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_input_list_size);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithTensorVectorInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> int", [] (const std::vector<Tensor>& input1) -> int64_t {
        return input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(2, outputs[0].toInt());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithLegacyTensorVectorInput_withoutOutput_whenRegistered_thenCanBeCalled) {
```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 387-414
```cpp
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> ()", [] (const std::vector<Tensor>& input1) -> void {
        captured_input_list_size = input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_input_list_size);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithLegacyTensorVectorInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> int", [] (const std::vector<Tensor>& input1) -> int64_t {
        return input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(2, outputs[0].toInt());
}

```
- EN: Focus symbols: `RegisterOperators`, `op`, `tensor_list_input`, `size`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `tensor_list_input`, `size`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 415-438
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithLegacyTensorListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> ()", [] (std::vector<Tensor> input1) -> void {
        captured_input_list_size = input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_input_list_size);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithLegacyTensorListInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> int", [] (std::vector<Tensor> input1) -> int64_t {
        return input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `tensor_list_input`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `tensor_list_input`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 439-465
```cpp
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(2, outputs[0].toInt());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithStringListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::stringlist_output(str[] input) -> str[]", [](std::vector<std::string> input) {
        return input;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::stringlist_output", ""});
  ASSERT_TRUE(op.has_value());

  c10::List<std::string> list({"value1", "value2"});
  auto outputs = callOp(*op, list);
  EXPECT_EQ(1, outputs.size());
  auto output = std::move(outputs[0]).toList();

  EXPECT_EQ(2, output.size());
  EXPECT_EQ("value1", output.get(0).toStringRef());
  EXPECT_EQ("value2", output.get(1).toStringRef());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithDictInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  int captured_dict_size = 0;

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 466-491
```cpp
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, Tensor) input) -> ()", [&] (Dict<string, Tensor> input1) {
        captured_dict_size = input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_dict_size = 0;
  Dict<string, Tensor> dict;
  dict.insert("key1", dummyTensor(DispatchKey::CPU));
  dict.insert("key2", dummyTensor(DispatchKey::CUDA));
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_dict_size);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithDictInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, str) input) -> str", [&] (Dict<string, string> input1) {
        return input1.at("key2");
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `RegisterOperators`, `op`, `dict_input`, `Dict`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `dict_input`, `Dict`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 492-515
```cpp
  Dict<string, string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ("value2", outputs[0].toStringRef());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithDictOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
    .op("_test::dict_output(Dict(str, str) input) -> Dict(str, str)", [] (Dict<string, string> input) {
      return input;
    });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

  Dict<string, string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, string>(outputs[0].toGenericDict());

```
- EN: Focus symbols: `insert`, `callOp`, `EXPECT_EQ`, `size`, `toStringRef`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `callOp`, `EXPECT_EQ`, `size`, `toStringRef`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 516-540
```cpp
  EXPECT_EQ(2, output.size());
  EXPECT_EQ("value1", output.at("key1"));
  EXPECT_EQ("value2", output.at("key2"));
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithUnorderedMapInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  int captured_dict_size = 0;

  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, Tensor) input) -> ()", [&] (std::unordered_map<string, Tensor> input1) {
        captured_dict_size = input1.size();
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_dict_size = 0;
  c10::Dict<string, Tensor> dict;
  dict.insert("key1", dummyTensor(DispatchKey::CPU));
  dict.insert("key2", dummyTensor(DispatchKey::CUDA));
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_dict_size);
}

```
- EN: Focus symbols: `EXPECT_EQ`, `size`, `at`, `TEST`, `RegisterOperators`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `size`, `at`, `TEST`, `RegisterOperators`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 541-566
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithUnorderedMapInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, str) input) -> str", [&] (std::unordered_map<string, string> input1) {
        return input1.at("key2");
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ("value2", outputs[0].toStringRef());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithUnorderedMapOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
    .op("_test::dict_output(Dict(str, str) input) -> Dict(str, str)", [] (std::unordered_map<string, string> input) {
      return input;
    });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `dict_input`, `Dict`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `dict_input`, `Dict`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 567-594
```cpp
  c10::Dict<string, string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, string>(outputs[0].toGenericDict());

  EXPECT_EQ(2, output.size());
  EXPECT_EQ("value1", output.at("key1"));
  EXPECT_EQ("value2", output.at("key2"));
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithMapOfList_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_output(Dict(str, int[]) input) -> Dict(str, int[])", [](std::unordered_map<string, std::vector<int64_t>> input) {
        return input;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, c10::List<int64_t>> dict;
  dict.insert("key1", c10::List<int64_t>({10, 20}));
  dict.insert("key2", c10::List<int64_t>({30, 40}));
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, c10::List<int64_t>>(outputs[0].toGenericDict());

```
- EN: Focus symbols: `insert`, `callOp`, `EXPECT_EQ`, `size`, `toGenericDict`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `callOp`, `EXPECT_EQ`, `size`, `toGenericDict`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 595-618
```cpp
  EXPECT_EQ(2, output.size());
  EXPECT_EQ(2, output.at("key1").size());
  EXPECT_EQ(10, output.at("key1").get(0));
  EXPECT_EQ(20, output.at("key1").get(1));
  EXPECT_EQ(2, output.at("key2").size());
  EXPECT_EQ(30, output.at("key2").get(0));
  EXPECT_EQ(40, output.at("key2").get(1));
}


TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithMapOfListOfMap_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_output(Dict(str, Dict(int,str)[]) input) -> Dict(str, Dict(int,str)[])", [](std::unordered_map<string, std::vector<std::unordered_map<int64_t, string>>> input) {
        return input;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, c10::List<c10::Dict<int64_t, string>>> dict;
  c10::Dict<int64_t, string> dict1;
  dict1.insert(10, "10");
  dict1.insert(20, "20");
  dict.insert("key1", c10::List<c10::Dict<int64_t, string>>({dict1}));
```
- EN: Focus symbols: `EXPECT_EQ`, `size`, `at`, `get`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `size`, `at`, `get`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 619-645
```cpp
  c10::Dict<int64_t, string> dict2;
  dict2.insert(30, "30");
  dict2.insert(40, "40");
  dict.insert("key2", c10::List<c10::Dict<int64_t, string>>({dict2}));
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, c10::List<std::unordered_map<int64_t, string>>>(outputs[0].toGenericDict());

  EXPECT_EQ(2, output.size());
  EXPECT_EQ(1, output.at("key1").size());
  EXPECT_EQ(2, output.at("key1").get(0).size());
  EXPECT_EQ("10", output.at("key1").get(0).at(10));
  EXPECT_EQ("20", output.at("key1").get(0).at(20));
  EXPECT_EQ(2, output.at("key2").get(0).size());
  EXPECT_EQ("30", output.at("key2").get(0).at(30));
  EXPECT_EQ("40", output.at("key2").get(0).at(40));
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithListOfMap_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Dict(str, int)[] input) -> Dict(str, int)[]", [](std::vector<std::unordered_map<string, int64_t>> input) {
        return input;
      });

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `insert`, `callOp`, `EXPECT_EQ`, `size`, `toGenericDict`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `callOp`, `EXPECT_EQ`, `size`, `toGenericDict`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 646-671
```cpp
  c10::Dict<string, int64_t> dict1;
  dict1.insert("1", 1);
  dict1.insert("2", 2);
  c10::Dict<string, int64_t> dict2;
  dict2.insert("3", 3);
  dict2.insert("4", 4);
  c10::List<c10::Dict<string, int64_t>> list({dict1, dict2});
  auto outputs = callOp(*op, list);
  EXPECT_EQ(1, outputs.size());
  c10::impl::GenericList output = std::move(outputs[0]).toList();

  EXPECT_EQ(2, output.size());
  EXPECT_EQ(2, output.get(0).toGenericDict().size());
  EXPECT_EQ(1, output.get(0).toGenericDict().at("1").toInt());
  EXPECT_EQ(2, output.get(0).toGenericDict().at("2").toInt());
  EXPECT_EQ(2, output.get(1).toGenericDict().size());
  EXPECT_EQ(3, output.get(1).toGenericDict().at("3").toInt());
  EXPECT_EQ(4, output.get(1).toGenericDict().at("4").toInt());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithListOfMapOfIntList_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Dict(str, int[])[] input) -> Dict(str, int[])[]", [](std::vector<std::unordered_map<string, std::vector<int64_t>>> input) {
        return input;
      });

```
- EN: Focus symbols: `insert`, `list`, `callOp`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `list`, `callOp`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 672-695
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, c10::List<int64_t>> dict1;
  dict1.insert("1", c10::List<int64_t>({1, 2}));
  dict1.insert("3", c10::List<int64_t>({3, 4}));
  c10::Dict<string, c10::List<int64_t>> dict2;
  dict2.insert("5", c10::List<int64_t>({5, 6}));
  dict2.insert("7", c10::List<int64_t>({7, 8}));
  c10::List<c10::Dict<string, c10::List<int64_t>>> list({ dict1, dict2 });
  auto outputs = callOp(*op, list);
  EXPECT_EQ(1, outputs.size());
  c10::impl::GenericList output = std::move(outputs[0]).toList();

  EXPECT_EQ(2, output.size());
  EXPECT_EQ(2, output.get(0).toGenericDict().size());
  EXPECT_EQ(2, output.get(0).toGenericDict().at("1").toIntVector().size());
  EXPECT_EQ(1, output.get(0).toGenericDict().at("1").toIntVector()[0]);
  EXPECT_EQ(2, output.get(0).toGenericDict().at("1").toIntVector()[1]);
  EXPECT_EQ(2, output.get(0).toGenericDict().at("3").toIntVector().size());
  EXPECT_EQ(3, output.get(0).toGenericDict().at("3").toIntVector()[0]);
  EXPECT_EQ(4, output.get(0).toGenericDict().at("3").toIntVector()[1]);
  EXPECT_EQ(2, output.get(1).toGenericDict().at("5").toIntVector().size());
  EXPECT_EQ(5, output.get(1).toGenericDict().at("5").toIntVector()[0]);
```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `insert`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `insert`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 696-719
```cpp
  EXPECT_EQ(6, output.get(1).toGenericDict().at("5").toIntVector()[1]);
  EXPECT_EQ(2, output.get(1).toGenericDict().at("7").toIntVector().size());
  EXPECT_EQ(7, output.get(1).toGenericDict().at("7").toIntVector()[0]);
  EXPECT_EQ(8, output.get(1).toGenericDict().at("7").toIntVector()[1]);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenFallbackKernelWithoutAnyArguments_whenRegistered_thenCanBeCalled) {
  // note: non-fallback kernels without tensor arguments don't work because there
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  bool called = false;
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args() -> ()", [&] () {called = true;});

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op);
  EXPECT_TRUE(called);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenFallbackKernelWithoutTensorArguments_whenRegistered_thenCanBeCalled) {
  // note: non-fallback kernels without tensor arguments don't work because there
```
- EN: Focus symbols: `EXPECT_EQ`, `get`, `toGenericDict`, `at`, `toIntVector`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `get`, `toGenericDict`, `at`, `toIntVector`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 720-743
```cpp
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args(int arg) -> int", [] (int64_t arg) {return arg + 1;});

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, 3);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(4, outputs[0].toInt());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithOptionalInputs_withoutOutput_whenRegistered_thenCanBeCalled) {
  bool called = false;
  std::optional<Tensor> called_arg2 = std::nullopt;
  std::optional<int64_t> called_arg3 = std::nullopt;
  std::optional<std::string> called_arg4 = std::nullopt;

  auto registrar = RegisterOperators().op(
    "_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> ()",
    [&] (Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
      called = true;
      called_arg2 = arg2;
```
- EN: Focus symbols: `RegisterOperators`, `op`, `no_tensor_args`, `singleton`, `findSchema`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `no_tensor_args`, `singleton`, `findSchema`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 744-771
```cpp
      called_arg3 = arg3;
      called_arg4 = arg4;
    });
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), c10::IValue(), std::string("text"));
  EXPECT_EQ(0, outputs.size());

  EXPECT_TRUE(called);
  EXPECT_TRUE(called_arg2.has_value());
  EXPECT_EQ(extractDispatchKey(*called_arg2), DispatchKey::CUDA);
  EXPECT_FALSE(called_arg3.has_value());
  EXPECT_TRUE(called_arg4.has_value());
  EXPECT_EQ(*called_arg4, "text");

  called = false;
  outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::IValue(), 4, c10::IValue());
  EXPECT_EQ(0, outputs.size());

  EXPECT_TRUE(called);
  EXPECT_FALSE(called_arg2.has_value());
  EXPECT_TRUE(called_arg3.has_value());
  EXPECT_EQ(*called_arg3, 4);
  EXPECT_FALSE(called_arg4.has_value());
}

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 772-795
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithOptionalInputs_withOutput_whenRegistered_thenCanBeCalled) {
  bool called = false;
  std::optional<Tensor> called_arg2 = std::nullopt;
  std::optional<int64_t> called_arg3 = std::nullopt;
  std::optional<std::string> called_arg4 = std::nullopt;

  auto registrar = RegisterOperators().op(
    "_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> Tensor?",
    [&] (Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
      called = true;
      called_arg2 = arg2;
      called_arg3 = arg3;
      called_arg4 = arg4;
      return arg2;
    });
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), c10::IValue(), std::string("text"));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(outputs[0].toTensor()));

  EXPECT_TRUE(called);
```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `opt_input`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `opt_input`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 796-822
```cpp
  EXPECT_TRUE(called_arg2.has_value());
  EXPECT_EQ(extractDispatchKey(*called_arg2), DispatchKey::CUDA);
  EXPECT_FALSE(called_arg3.has_value());
  EXPECT_TRUE(called_arg4.has_value());
  EXPECT_EQ(*called_arg4, "text");

  called = false;
  outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::IValue(), 4, c10::IValue());
  EXPECT_EQ(1, outputs.size());
  EXPECT_TRUE(outputs[0].isNone());

  EXPECT_TRUE(called);
  EXPECT_FALSE(called_arg2.has_value());
  EXPECT_TRUE(called_arg3.has_value());
  EXPECT_EQ(*called_arg3, 4);
  EXPECT_FALSE(called_arg4.has_value());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernelWithOptionalInputs_withMultipleOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op(
    "_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> (Tensor?, int?, str?)",
    [] (Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
      return std::make_tuple(arg2, arg3, arg4);
    });
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `EXPECT_TRUE`, `has_value`, `EXPECT_EQ`, `extractDispatchKey`, `EXPECT_FALSE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_TRUE`, `has_value`, `EXPECT_EQ`, `extractDispatchKey`, `EXPECT_FALSE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 823-846
```cpp
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), c10::IValue(), std::string("text"));
  EXPECT_EQ(3, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(outputs[0].toTensor()));
  EXPECT_TRUE(outputs[1].isNone());
  EXPECT_EQ("text", outputs[2].toStringRef());

  outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::IValue(), 4, c10::IValue());
  EXPECT_EQ(3, outputs.size());
  EXPECT_TRUE(outputs[0].isNone());
  EXPECT_EQ(4, outputs[1].toInt());
  EXPECT_TRUE(outputs[2].isNone());
}

void expectCallsConcatUnboxed(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  std::string result = callOpUnboxed<std::string, const Tensor&, std::string, const std::string&, int64_t>(*op, dummyTensor(dispatch_key), "1", "2", 3);
  EXPECT_EQ("prefix123", result);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernel_whenRegistered_thenCanBeCalledUnboxed) {
```
- EN: Focus symbols: `callOp`, `dummyTensor`, `IValue`, `string`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `IValue`, `string`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 847-870
```cpp
  std::string prefix = "prefix";
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, str a, str b, int c) -> str", [&] (const Tensor& tensor1, std::string a, const std::string& b, int64_t c) {
    return prefix + a + b + std::to_string(c);
  });
  expectCallsConcatUnboxed(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenKernel_whenRegisteredWithoutSpecifyingSchema_thenInfersSchema) {
  auto registrar = RegisterOperators()
      .op("_test::no_schema_specified", [] (Tensor arg1, int64_t arg2, const std::vector<Tensor>& arg3) -> std::tuple<int64_t, Tensor> {return {};});

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_schema_specified", ""});
  ASSERT_TRUE(op.has_value());

  std::optional<std::string> differences = c10::findSchemaDifferences(torch::jit::parseSchema("_test::no_schema_specified(Tensor arg1, int arg2, Tensor[] arg3) -> (int, Tensor)"), op->schema());
  EXPECT_FALSE(differences.has_value());
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenMismatchedKernel_withDifferentNumArguments_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", [] (Tensor) -> int64_t {return 0;});

  // and now a set of mismatching schemas
```
- EN: Focus symbols: `RegisterOperators`, `op`, `my_op`, `to_string`, `expectCallsConcatUnboxed`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `my_op`, `to_string`, `expectCallsConcatUnboxed`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 871-894
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg, Tensor arg2) -> int", [] (Tensor) -> int64_t {return 0;});
    }, "The number of arguments is different. 2 vs 1"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg, Tensor arg2) -> ()", [] (Tensor, Tensor) -> void {});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch() -> ()", [] (Tensor, Tensor) -> void {});
    }, "The number of arguments is different. 0 vs 2"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", [] (Tensor, Tensor) -> void {});
    }, "The number of arguments is different. 1 vs 2"
  );

  expectThrows<c10::Error>([] {
```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 895-919
```cpp
    RegisterOperators()
        .op("_test::mismatch(Tensor arg, Tensor arg2, Tensor arg3) -> ()", [] (Tensor, Tensor) -> void {});
    }, "The number of arguments is different. 3 vs 2"
  );
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenMismatchedKernel_withDifferentArgumentType_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg1, int arg2) -> int", [] (Tensor, int64_t) -> int64_t {return 0;});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg1, float arg2) -> int", [] (Tensor, int64_t) -> int64_t {return 0;});
    }, "Type mismatch in argument 2: float vs int"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(int arg1, int arg2) -> int", [] (Tensor, int64_t) -> int64_t {return 0;});
    }, "Type mismatch in argument 1: int vs Tensor"
  );
}

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 920-943
```cpp
TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenMismatchedKernel_withDifferentNumReturns_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", [] (Tensor) -> int64_t {return 0;});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", [] (Tensor) -> int64_t {return 0;});
    }, "The number of returns is different. 0 vs 1"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (int, int)", [] (Tensor) -> int64_t {return 0;});
    }, "The number of returns is different. 2 vs 1"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> ()", [] (Tensor) -> void {});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 944-971
```cpp
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", [] (Tensor) -> void {});
    }, "The number of returns is different. 1 vs 0"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor)", [] (Tensor) -> void {});
    }, "The number of returns is different. 2 vs 0"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor)", [] (Tensor) -> std::tuple<Tensor, Tensor> {return {};});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", [] (Tensor) -> std::tuple<Tensor, Tensor> {return {};});
    }, "The number of returns is different. 0 vs 2"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", [] (Tensor) -> std::tuple<Tensor, Tensor> {return {};});
    }, "The number of returns is different. 1 vs 2"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 972-996
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor, Tensor)", [] (Tensor) -> std::tuple<Tensor, Tensor> {return {};});
    }, "The number of returns is different. 3 vs 2"
  );
}

TEST(OperatorRegistrationTestLegacyLambdaBasedKernel, givenMismatchedKernel_withDifferentReturnTypes_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", [] (Tensor) -> int64_t {return 0;});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", [] (Tensor) -> int64_t {return 0;});
    }, "Type mismatch in return 1: Tensor vs int"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> float", [] (Tensor) -> int64_t {return 0;});
    }, "Type mismatch in return 1: float vs int"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 997-1020
```cpp
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> Tensor", [] (Tensor) -> Tensor {return {};});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> float", [] (Tensor) -> Tensor {return {};});
    }, "Type mismatch in return 1: float vs Tensor"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> (Tensor, int)", [] (Tensor) -> std::tuple<Tensor, int64_t> {return {};});

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, float)", [] (Tensor) -> std::tuple<Tensor, int64_t> {return {};});
    }, "Type mismatch in return 2: float vs int"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1021-1028
```cpp
        .op("_test::mismatch(Tensor arg) -> (int, int)", [] (Tensor) -> std::tuple<Tensor, int64_t> {return {};});
    }, "Type mismatch in return 1: int vs Tensor"
  );
}

}

#pragma GCC diagnostic pop
```
- EN: Focus symbols: `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- Backend/device dispatch / 后端/设备分发
- Behavioral regression tests / 行为回归测试
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/boxing/impl/test_helpers.h`, `ATen/core/op_registration/op_registration.h`, `ATen/core/Tensor.h`, `torch/csrc/jit/frontend/function_schema_parser.h`, `ATen/core/LegacyTypeDispatch.h`
- External/system includes / 外部或系统头: `gtest/gtest.h`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; namespace scoping / 命名空间作用域
