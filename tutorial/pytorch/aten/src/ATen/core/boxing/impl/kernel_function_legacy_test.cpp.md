# kernel_function_legacy_test.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/boxing/impl/kernel_function_legacy_test.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `Return`, `kernel_func`, `kernel`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `Return`, `kernel_func`, `kernel`。

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
 * This file tests the legacy function-based API for registering kernels.
 *
 * > namespace { Tensor kernel(Tensor a) {...} }
 * > static auto registry = c10::RegisterOperators()
 * >   .op("func(Tensor a) -> Tensor", &kernel);
 */

using c10::RegisterOperators;
using c10::DispatchKey;
using c10::Stack;
```
- EN: Focus symbols: `kernel`, `RegisterOperators`, `op`, `func`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`kernel`, `RegisterOperators`, `op`, `func`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 25-48
```cpp
using std::make_unique;
using c10::intrusive_ptr;
using c10::Dict;
using at::Tensor;
using std::string;
using std::unique_ptr;

namespace {

int64_t errorKernel(const Tensor& tensor, int64_t input) {
  EXPECT_TRUE(false); // this kernel should never be called
  return 0;
}

int64_t incrementKernel(const Tensor& tensor, int64_t input) {
  return input + 1;
}

void expectCallsIncrement(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
```
- EN: Focus symbols: `errorKernel`, `EXPECT_TRUE`, `incrementKernel`, `expectCallsIncrement`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`errorKernel`, `EXPECT_TRUE`, `incrementKernel`, `expectCallsIncrement`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 49-76
```cpp
  auto result = callOp(*op, dummyTensor(dispatch_key), 5);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(6, result[0].toInt());
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernel_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", &incrementKernel);
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernel_whenRegisteredInConstructor_thenCanBeCalled) {
  auto registrar = RegisterOperators("_test::my_op(Tensor dummy, int input) -> int", &incrementKernel);
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInOneRegistrar_thenCallsRightKernel) {
  auto registrar = RegisterOperators()
      .op("_test::my_op(Tensor dummy, int input) -> int", &incrementKernel)
      .op("_test::error(Tensor dummy, int input) -> int", &errorKernel);
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInMultipleRegistrars_thenCallsRightKernel) {
  auto registrar1 = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", &incrementKernel);
  auto registrar2 = RegisterOperators().op("_test::error(Tensor dummy, int input) -> int", &errorKernel);
  expectCallsIncrement(DispatchKey::CPU);
}

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 77-104
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernel_whenRegistrationRunsOutOfScope_thenCannotBeCalledAnymore) {
  {
    auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", &incrementKernel);

    expectCallsIncrement(DispatchKey::CPU);
  }

  // now the registrar is destructed. Assert that the schema is gone.
  expectDoesntFindOperator("_test::my_op");
}

bool was_called = false;

void kernelWithoutOutput(const Tensor&) {
  was_called = true;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::no_return(Tensor dummy) -> ()", &kernelWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_return", ""});
  ASSERT_TRUE(op.has_value());
  was_called = false;
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_TRUE(was_called);
  EXPECT_EQ(0, result.size());
}

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `my_op`, `expectCallsIncrement`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `my_op`, `expectCallsIncrement`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 105-131
```cpp
std::tuple<> kernelWithZeroOutputs(const Tensor&) {
  was_called = true;
  return std::make_tuple();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithZeroOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::zero_outputs(Tensor dummy) -> ()", &kernelWithZeroOutputs);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::zero_outputs", ""});
  ASSERT_TRUE(op.has_value());
  was_called = false;
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_TRUE(was_called);
  EXPECT_EQ(0, result.size());
}

int64_t kernelWithIntOutput(Tensor, int64_t a, int64_t b) {
  return a + b;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithIntOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_output(Tensor dummy, int a, int b) -> int", &kernelWithIntOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_output", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `kernelWithZeroOutputs`, `make_tuple`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithZeroOutputs`, `make_tuple`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 132-156
```cpp
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), 3, 6);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(9, result[0].toInt());
}

Tensor kernelWithTensorOutput(const Tensor& input) {
  return input;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::returning_tensor(Tensor input) -> Tensor", &kernelWithTensorOutput);

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
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 157-183
```cpp
std::vector<Tensor> kernelWithTensorListOutput(const Tensor& input1, const Tensor& input2, const Tensor& input3) {
  return {input1, input2, input3};
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Tensor input1, Tensor input2, Tensor input3) -> Tensor[]", &kernelWithTensorListOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(3, result[0].toTensorVector().size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensorVector()[0]));
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensorVector()[1]));
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensorVector()[2]));
}

std::vector<int64_t> kernelWithIntListOutput(const Tensor&, int64_t input1, int64_t input2, int64_t input3) {
  return {input1, input2, input3};
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithIntListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Tensor dummy, int input1, int input2, int input3) -> int[]", &kernelWithIntListOutput);

```
- EN: Focus symbols: `kernelWithTensorListOutput`, `TEST`, `RegisterOperators`, `op`, `list_output`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithTensorListOutput`, `TEST`, `RegisterOperators`, `op`, `list_output`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 184-211
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), 2, 4, 6);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(3, result[0].toIntVector().size());
  EXPECT_EQ(2, result[0].toIntVector()[0]);
  EXPECT_EQ(4, result[0].toIntVector()[1]);
  EXPECT_EQ(6, result[0].toIntVector()[2]);
}

std::tuple<Tensor, int64_t, std::vector<Tensor>, std::optional<int64_t>, Dict<string, Tensor>> kernelWithMultipleOutputs(Tensor) {
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
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithMultipleOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
     .op("_test::multiple_outputs(Tensor dummy) -> (Tensor, int, Tensor[], int?, Dict(str, Tensor))", &kernelWithMultipleOutputs);

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 212-239
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::multiple_outputs", ""});
  ASSERT_TRUE(op.has_value());

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

Tensor kernelWithTensorInputByReferenceWithOutput(const Tensor& input1) {
  return input1;
}

Tensor kernelWithTensorInputByValueWithOutput(Tensor input1) {
  return input1;
}
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorInputByReference_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> Tensor", &kernelWithTensorInputByReferenceWithOutput);

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 240-267
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensor()));

  result = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensor()));
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorInputByValue_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> Tensor", &kernelWithTensorInputByValueWithOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
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

### Lines 268-293
```cpp
Tensor captured_input;

void kernelWithTensorInputByReferenceWithoutOutput(const Tensor& input1) {
  captured_input = input1;
}

void kernelWithTensorInputByValueWithoutOutput(Tensor input1) {
  captured_input = input1;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorInputByReference_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> ()", &kernelWithTensorInputByReferenceWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(captured_input));

  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(captured_input));
}

```
- EN: Focus symbols: `kernelWithTensorInputByReferenceWithoutOutput`, `kernelWithTensorInputByValueWithoutOutput`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithTensorInputByReferenceWithoutOutput`, `kernelWithTensorInputByValueWithoutOutput`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 294-319
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorInputByValue_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> ()", &kernelWithTensorInputByValueWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(captured_input));

  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(captured_input));
}

int64_t captured_int_input = 0;

void kernelWithIntInputWithoutOutput(Tensor, int64_t input1) {
  captured_int_input = input1;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithIntInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_input(Tensor dummy, int input) -> ()", &kernelWithIntInputWithoutOutput);

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `tensor_input`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `tensor_input`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 320-344
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_int_input = 0;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 3);
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(3, captured_int_input);
}

int64_t kernelWithIntInputWithOutput(Tensor, int64_t input1) {
  return input1 + 1;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithIntInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_input(Tensor dummy, int input) -> int", &kernelWithIntInputWithOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 3);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(4, outputs[0].toInt());
}

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 345-371
```cpp
int64_t captured_input_list_size = 0;

void kernelWithIntListInputWithoutOutput(Tensor, const std::vector<int64_t>& input1) {
  captured_input_list_size = input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithIntListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_list_input(Tensor dummy, int[] input) -> ()", &kernelWithIntListInputWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::List<int64_t>({2, 4, 6}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(3, captured_input_list_size);
}

int64_t kernelWithIntListInputWithOutput(Tensor, const std::vector<int64_t>& input1) {
  return input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithIntListInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_list_input(Tensor dummy, int[] input) -> int", &kernelWithIntListInputWithOutput);

```
- EN: Focus symbols: `kernelWithIntListInputWithoutOutput`, `size`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithIntListInputWithoutOutput`, `size`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 372-396
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::List<int64_t>({2, 4, 6}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(3, outputs[0].toInt());
}

void kernelWithTensorListInputWithoutOutput(const std::vector<Tensor>& input1) {
  captured_input_list_size = input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> ()", &kernelWithTensorListInputWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_input_list_size);
}

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 397-423
```cpp
int64_t kernelWithTensorListInputWithOutput(const std::vector<Tensor>& input1) {
  return input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithTensorListInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> int", &kernelWithTensorListInputWithOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(2, outputs[0].toInt());
}

void kernelWithLegacyTensorVectorInputWithoutOutput(const std::vector<Tensor>& input1) {
  captured_input_list_size = input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithLegacyTensorVectorInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> ()", &kernelWithLegacyTensorVectorInputWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `kernelWithTensorListInputWithOutput`, `size`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithTensorListInputWithOutput`, `size`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 424-449
```cpp
  captured_input_list_size = 0;
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_input_list_size);
}

int64_t kernelWithLegacyTensorVectorInputWithOutput(const std::vector<Tensor>& input1) {
  return input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithLegacyTensorVectorInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> int", &kernelWithLegacyTensorVectorInputWithOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(2, outputs[0].toInt());
}

void kernelWithLegacyTensorListInputWithoutOutput(std::vector<Tensor> input1) {
  captured_input_list_size = input1.size();
}

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `kernelWithLegacyTensorVectorInputWithOutput`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `kernelWithLegacyTensorVectorInputWithOutput`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 450-473
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithLegacyTensorListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> ()", &kernelWithLegacyTensorListInputWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_input_list_size);
}

int64_t kernelWithLegacyTensorListInputWithOutput(std::vector<Tensor> input1) {
  return input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithLegacyTensorListInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> int", &kernelWithLegacyTensorListInputWithOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `tensor_list_input`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `tensor_list_input`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 474-499
```cpp
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(2, outputs[0].toInt());
}

std::vector<std::string> kernelWithStringListOutput(std::vector<std::string> input) {
  return input;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithStringListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::stringlist_output(str[] input) -> str[]", &kernelWithStringListOutput);

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

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `toInt`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 500-525
```cpp
int captured_dict_size = 0;

void kernelWithDictInputWithoutOutput(Dict<string, Tensor> input1) {
  captured_dict_size = input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithDictInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, Tensor) input) -> ()", &kernelWithDictInputWithoutOutput);

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

std::string kernelWithDictInputWithOutput(Dict<string, string> input1) {
  return input1.at("key2");
}

```
- EN: Focus symbols: `kernelWithDictInputWithoutOutput`, `size`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithDictInputWithoutOutput`, `size`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 526-551
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithDictInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, str) input) -> str", &kernelWithDictInputWithOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

  Dict<string, string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ("value2", outputs[0].toStringRef());
}

Dict<string, string> kernelWithDictOutput(Dict<string, string> input) {
  return input;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithDictOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_output(Dict(str, str) input) -> Dict(str, str)", &kernelWithDictOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `dict_input`, `Dict`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `dict_input`, `Dict`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 552-575
```cpp
  Dict<string, string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, string>(outputs[0].toGenericDict());

  EXPECT_EQ(2, output.size());
  EXPECT_EQ("value1", output.at("key1"));
  EXPECT_EQ("value2", output.at("key2"));
}

void kernelWithUnorderedMapInputWithoutOutput(std::unordered_map<string, Tensor> input1) {
  captured_dict_size = input1.size();
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithUnorderedMapInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, Tensor) input) -> ()", &kernelWithUnorderedMapInputWithoutOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_dict_size = 0;
```
- EN: Focus symbols: `insert`, `callOp`, `EXPECT_EQ`, `size`, `toGenericDict`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `callOp`, `EXPECT_EQ`, `size`, `toGenericDict`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 576-602
```cpp
  c10::Dict<string, Tensor> dict;
  dict.insert("key1", dummyTensor(DispatchKey::CPU));
  dict.insert("key2", dummyTensor(DispatchKey::CUDA));
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_dict_size);
}

std::string kernelWithUnorderedMapInputWithOutput(std::unordered_map<string, string> input1) {
  return input1.at("key2");
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithUnorderedMapInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, str) input) -> str", &kernelWithUnorderedMapInputWithOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ("value2", outputs[0].toStringRef());
}

```
- EN: Focus symbols: `insert`, `dummyTensor`, `callOp`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `dummyTensor`, `callOp`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 603-629
```cpp
std::unordered_map<string, string> kernelWithUnorderedMapOutput(std::unordered_map<string, string> input) {
  return input;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithUnorderedMapOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_output(Dict(str, str) input) -> Dict(str, str)", &kernelWithUnorderedMapOutput);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

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

std::unordered_map<string, std::vector<int64_t>> kernelWithMapOfIntList(std::unordered_map<string, std::vector<int64_t>> input) {
  return input;
}

```
- EN: Focus symbols: `kernelWithUnorderedMapOutput`, `TEST`, `RegisterOperators`, `op`, `dict_output`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithUnorderedMapOutput`, `TEST`, `RegisterOperators`, `op`, `dict_output`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 630-656
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithMapOfList_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_output(Dict(str, int[]) input) -> Dict(str, int[])", &kernelWithMapOfIntList);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, c10::List<int64_t>> dict;
  dict.insert("key1", c10::List<int64_t>({10, 20}));
  dict.insert("key2", c10::List<int64_t>({30, 40}));
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, c10::List<int64_t>>(outputs[0].toGenericDict());

  EXPECT_EQ(2, output.size());
  EXPECT_EQ(2, output.at("key1").size());
  EXPECT_EQ(10, output.at("key1").get(0));
  EXPECT_EQ(20, output.at("key1").get(1));
  EXPECT_EQ(2, output.at("key2").size());
  EXPECT_EQ(30, output.at("key2").get(0));
  EXPECT_EQ(40, output.at("key2").get(1));
}

std::unordered_map<string, std::vector<std::unordered_map<int64_t, string>>> kernelWithMapOfListOfMap(std::unordered_map<string, std::vector<std::unordered_map<int64_t, string>>> input) {
  return input;
}

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `dict_output`, `Dict`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `dict_output`, `Dict`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 657-680
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithMapOfListOfMap_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_output(Dict(str, Dict(int,str)[]) input) -> Dict(str, Dict(int,str)[])", &kernelWithMapOfListOfMap);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, c10::List<c10::Dict<int64_t, string>>> dict;
  c10::Dict<int64_t, string> dict1;
  dict1.insert(10, "10");
  dict1.insert(20, "20");
  dict.insert("key1", c10::List<c10::Dict<int64_t, string>>({dict1}));
  c10::Dict<int64_t, string> dict2;
  dict2.insert(30, "30");
  dict2.insert(40, "40");
  dict.insert("key2", c10::List<c10::Dict<int64_t, string>>({dict2}));
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, c10::List<c10::Dict<int64_t, string>>>(outputs[0].toGenericDict());

  EXPECT_EQ(2, output.size());
  EXPECT_EQ(1, output.at("key1").size());
  EXPECT_EQ(2, output.at("key1").get(0).size());
  EXPECT_EQ("10", output.at("key1").get(0).at(10));
```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `dict_output`, `Dict`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `dict_output`, `Dict`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 681-708
```cpp
  EXPECT_EQ("20", output.at("key1").get(0).at(20));
  EXPECT_EQ(2, output.at("key2").get(0).size());
  EXPECT_EQ("30", output.at("key2").get(0).at(30));
  EXPECT_EQ("40", output.at("key2").get(0).at(40));
}

std::vector<std::unordered_map<string, int64_t>> kernelWithListOfMap(std::vector<std::unordered_map<string, int64_t>> input) {
  return input;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithListOfMap_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Dict(str, int)[] input) -> Dict(str, int)[]", &kernelWithListOfMap);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

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

```
- EN: Focus symbols: `EXPECT_EQ`, `at`, `get`, `size`, `kernelWithListOfMap`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `at`, `get`, `size`, `kernelWithListOfMap`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 709-732
```cpp
  EXPECT_EQ(2, output.size());
  EXPECT_EQ(2, output.get(0).toGenericDict().size());
  EXPECT_EQ(1, output.get(0).toGenericDict().at("1").toInt());
  EXPECT_EQ(2, output.get(0).toGenericDict().at("2").toInt());
  EXPECT_EQ(2, output.get(1).toGenericDict().size());
  EXPECT_EQ(3, output.get(1).toGenericDict().at("3").toInt());
  EXPECT_EQ(4, output.get(1).toGenericDict().at("4").toInt());
}

std::vector<std::unordered_map<string, std::vector<int64_t>>> kernelWithListOfMapOfIntList(std::vector<std::unordered_map<string, std::vector<int64_t>>> input) {
  return input;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithListOfMapOfIntList_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Dict(str, int[])[] input) -> Dict(str, int[])[]", &kernelWithListOfMapOfIntList);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

  c10::Dict<string, c10::List<int64_t>> dict1;
  dict1.insert("1", c10::List<int64_t>({1, 2}));
  dict1.insert("3", c10::List<int64_t>({3, 4}));
  c10::Dict<string, c10::List<int64_t>> dict2;
```
- EN: Focus symbols: `EXPECT_EQ`, `size`, `get`, `toGenericDict`, `at`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `size`, `get`, `toGenericDict`, `at`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 733-757
```cpp
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
  EXPECT_EQ(6, output.get(1).toGenericDict().at("5").toIntVector()[1]);
  EXPECT_EQ(2, output.get(1).toGenericDict().at("7").toIntVector().size());
  EXPECT_EQ(7, output.get(1).toGenericDict().at("7").toIntVector()[0]);
  EXPECT_EQ(8, output.get(1).toGenericDict().at("7").toIntVector()[1]);
}

bool called = false;

```
- EN: Focus symbols: `insert`, `list`, `callOp`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`insert`, `list`, `callOp`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 758-781
```cpp
void kernelWithoutInputs() {
  called = true;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenFallbackKernelWithoutAnyArguments_whenRegistered_thenCanBeCalled) {
  // note: non-fallback kernels without tensor arguments don't work because there
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args() -> ()", &kernelWithoutInputs);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op);
  EXPECT_TRUE(called);
}

int64_t kernelWithoutTensorInputs(int64_t arg) {
  return arg + 1;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenFallbackKernelWithoutTensorArguments_whenRegistered_thenCanBeCalled) {
```
- EN: Focus symbols: `kernelWithoutInputs`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithoutInputs`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 782-806
```cpp
  // note: non-fallback kernels without tensor arguments don't work because there
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args(int arg) -> int", &kernelWithoutTensorInputs);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, 3);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(4, outputs[0].toInt());
}

std::optional<Tensor> called_arg2 = std::nullopt;
std::optional<int64_t> called_arg3 = std::nullopt;
std::optional<std::string> called_arg4 = std::nullopt;

void kernelWithOptInputWithoutOutput(Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
  called = true;
  called_arg2 = arg2;
  called_arg3 = arg3;
  called_arg4 = arg4;
}

```
- EN: Focus symbols: `RegisterOperators`, `op`, `no_tensor_args`, `singleton`, `findSchema`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `no_tensor_args`, `singleton`, `findSchema`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 807-833
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithOptionalInputs_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> ()", &kernelWithOptInputWithoutOutput);
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
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `opt_input`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `opt_input`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 834-858
```cpp
std::optional<Tensor> kernelWithOptInputWithOutput(Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
  called = true;
  called_arg2 = arg2;
  called_arg3 = arg3;
  called_arg4 = arg4;
  return arg2;
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithOptionalInputs_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> Tensor?", &kernelWithOptInputWithOutput);
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), c10::IValue(), std::string("text"));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(outputs[0].toTensor()));

  EXPECT_TRUE(called);
  EXPECT_TRUE(called_arg2.has_value());
  EXPECT_EQ(extractDispatchKey(*called_arg2), DispatchKey::CUDA);
  EXPECT_FALSE(called_arg3.has_value());
  EXPECT_TRUE(called_arg4.has_value());
  EXPECT_EQ(*called_arg4, "text");

```
- EN: Focus symbols: `kernelWithOptInputWithOutput`, `TEST`, `RegisterOperators`, `op`, `opt_input`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithOptInputWithOutput`, `TEST`, `RegisterOperators`, `op`, `opt_input`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 859-886
```cpp
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

std::tuple<std::optional<Tensor>, std::optional<int64_t>, std::optional<std::string>>
kernelWithOptInputWithMultipleOutputs(Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
  return std::make_tuple(arg2, arg3, arg4);
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernelWithOptionalInputs_withMultipleOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> (Tensor?, int?, str?)", &kernelWithOptInputWithMultipleOutputs);
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), c10::IValue(), std::string("text"));
  EXPECT_EQ(3, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(outputs[0].toTensor()));
  EXPECT_TRUE(outputs[1].isNone());
  EXPECT_EQ("text", outputs[2].toStringRef());

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `IValue`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `IValue`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 887-912
```cpp
  outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::IValue(), 4, c10::IValue());
  EXPECT_EQ(3, outputs.size());
  EXPECT_TRUE(outputs[0].isNone());
  EXPECT_EQ(4, outputs[1].toInt());
  EXPECT_TRUE(outputs[2].isNone());
}

std::string concatKernel(const Tensor& tensor1, std::string a, const std::string& b, int64_t c) {
  return a + b + std::to_string(c);
}

void expectCallsConcatUnboxed(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  std::string result = callOpUnboxed<std::string, const Tensor&, std::string, const std::string&, int64_t>(*op, dummyTensor(dispatch_key), "1", "2", 3);
  EXPECT_EQ("123", result);
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernel_whenRegistered_thenCanBeCalledUnboxed) {
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, str a, str b, int c) -> str", &concatKernel);
  expectCallsConcatUnboxed(DispatchKey::CPU);
}

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `IValue`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `IValue`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 913-939
```cpp
std::tuple<int64_t, Tensor> kernelForSchemaInference(Tensor arg1, int64_t arg2, const std::vector<Tensor>& arg3) {
  return {};
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenKernel_whenRegisteredWithoutSpecifyingSchema_thenInfersSchema) {
  auto registrar = RegisterOperators()
      .op("_test::no_schema_specified", &kernelForSchemaInference);

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_schema_specified", ""});
  ASSERT_TRUE(op.has_value());

  std::optional<std::string> differences = c10::findSchemaDifferences(torch::jit::parseSchema("_test::no_schema_specified(Tensor arg1, int arg2, Tensor[] arg3) -> (int, Tensor)"), op->schema());
  EXPECT_FALSE(differences.has_value());
}

template<class Return, class... Args> struct kernel_func final {
  static Return func(Args...) { return {}; }
};
template<class... Args> struct kernel_func<void, Args...> final {
  static void func(Args...) {}
};

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenMismatchedKernel_withDifferentNumArguments_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", &kernel_func<int64_t, Tensor>::func);

```
- EN: Focus symbols: `Return`, `kernel_func`, `kernelForSchemaInference`, `TEST`, `RegisterOperators`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`Return`, `kernel_func`, `kernelForSchemaInference`, `TEST`, `RegisterOperators`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 940-963
```cpp
  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg, Tensor arg2) -> int", &kernel_func<int64_t, Tensor>::func);
    }, "The number of arguments is different. 2 vs 1"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg, Tensor arg2) -> ()", &kernel_func<void, Tensor, Tensor>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch() -> ()", &kernel_func<void, Tensor, Tensor>::func);
    }, "The number of arguments is different. 0 vs 2"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", &kernel_func<void, Tensor, Tensor>::func);
    }, "The number of arguments is different. 1 vs 2"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 964-989
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg, Tensor arg2, Tensor arg3) -> ()", &kernel_func<void, Tensor, Tensor>::func);
    }, "The number of arguments is different. 3 vs 2"
  );
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenMismatchedKernel_withDifferentArgumentType_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg1, int arg2) -> int", &kernel_func<int64_t, Tensor, int64_t>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg1, float arg2) -> int", &kernel_func<int64_t, Tensor, int64_t>::func);
    }, "Type mismatch in argument 2: float vs int"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(int arg1, int arg2) -> int", &kernel_func<int64_t, Tensor, int64_t>::func);
    }, "Type mismatch in argument 1: int vs Tensor"
  );
}

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 990-1013
```cpp
TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenMismatchedKernel_withDifferentNumReturns_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", &kernel_func<int64_t, Tensor>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", &kernel_func<int64_t, Tensor>::func);
    }, "The number of returns is different. 0 vs 1"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (int, int)", &kernel_func<int64_t, Tensor>::func);
    }, "The number of returns is different. 2 vs 1"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> ()", &kernel_func<void, Tensor>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1014-1041
```cpp
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", &kernel_func<void, Tensor>::func);
    }, "The number of returns is different. 1 vs 0"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor)", &kernel_func<void, Tensor>::func);
    }, "The number of returns is different. 2 vs 0"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor)", &kernel_func<std::tuple<Tensor, Tensor>, Tensor>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", &kernel_func<std::tuple<Tensor, Tensor>, Tensor>::func);
    }, "The number of returns is different. 0 vs 2"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", &kernel_func<std::tuple<Tensor, Tensor>, Tensor>::func);
    }, "The number of returns is different. 1 vs 2"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1042-1066
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor, Tensor)", &kernel_func<std::tuple<Tensor, Tensor>, Tensor>::func);
    }, "The number of returns is different. 3 vs 2"
  );
}

TEST(OperatorRegistrationTestLegacyFunctionBasedKernel, givenMismatchedKernel_withDifferentReturnTypes_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", &kernel_func<int64_t, Tensor>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", &kernel_func<int64_t, Tensor>::func);
    }, "Type mismatch in return 1: Tensor vs int"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> float", &kernel_func<int64_t, Tensor>::func);
    }, "Type mismatch in return 1: float vs int"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1067-1090
```cpp
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> Tensor", &kernel_func<Tensor, Tensor>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> float", &kernel_func<Tensor, Tensor>::func);
    }, "Type mismatch in return 1: float vs Tensor"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> (Tensor, int)", &kernel_func<std::tuple<Tensor, int64_t>, Tensor>::func);

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, float)", &kernel_func<std::tuple<Tensor, int64_t>, Tensor>::func);
    }, "Type mismatch in return 2: float vs int"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 1091-1098
```cpp
        .op("_test::mismatch(Tensor arg) -> (int, int)", &kernel_func<std::tuple<Tensor, int64_t>, Tensor>::func);
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
- Type-system design / 类型系统设计
- Backend/device dispatch / 后端/设备分发
- Behavioral regression tests / 行为回归测试
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/boxing/impl/test_helpers.h`, `ATen/core/op_registration/op_registration.h`, `ATen/core/Tensor.h`, `torch/csrc/jit/frontend/function_schema_parser.h`, `ATen/core/LegacyTypeDispatch.h`
- External/system includes / 外部或系统头: `gtest/gtest.h`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; namespace scoping / 命名空间作用域
