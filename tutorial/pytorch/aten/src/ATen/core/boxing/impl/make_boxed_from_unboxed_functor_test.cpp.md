# make_boxed_from_unboxed_functor_test.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/boxing/impl/make_boxed_from_unboxed_functor_test.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `ErrorKernel`, `IncrementKernel`, `DecrementKernel`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `ErrorKernel`, `IncrementKernel`, `DecrementKernel`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-16
```cpp
#include <gtest/gtest.h>
#include <ATen/core/boxing/impl/test_helpers.h>

#include <ATen/core/op_registration/op_registration.h>
#include <ATen/core/Tensor.h>
#include <torch/csrc/jit/frontend/function_schema_parser.h>
#include <torch/library.h>

#include <ATen/core/LegacyTypeDispatch.h>

using c10::RegisterOperators;
using c10::OperatorKernel;
using c10::DispatchKey;
using c10::Stack;
using std::make_unique;
using c10::intrusive_ptr;
```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 17-36
```cpp
using c10::Dict;
using at::Tensor;
using std::unique_ptr;
using std::string;

namespace {

struct ErrorKernel final : public OperatorKernel {
  int64_t operator()(const Tensor&, int64_t) {
    EXPECT_TRUE(false); // this kernel should never be called
    return 0;
  }
};

struct IncrementKernel final : OperatorKernel {
  int64_t operator()(const Tensor& tensor, int64_t input) {
    return input + 1;
  }
};

```
- EN: Focus symbols: `ErrorKernel`, `IncrementKernel`, `operator`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`ErrorKernel`, `IncrementKernel`, `operator`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 37-53
```cpp
struct DecrementKernel final : OperatorKernel {
  int64_t operator()(const Tensor& tensor, int64_t input) {
    return input - 1;
  }
};

void expectCallsIncrement(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  auto result = callOp(*op, dummyTensor(dispatch_key), 5);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(6, result[0].toInt());
}

```
- EN: Focus symbols: `DecrementKernel`, `operator`, `expectCallsIncrement`, `singleton`, `findSchema`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`DecrementKernel`, `operator`, `expectCallsIncrement`, `singleton`, `findSchema`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 54-69
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernel_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<IncrementKernel>(DispatchKey::CPU));
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestFunctorBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInOneRegistrar_thenCallsRightKernel) {
  auto registrar = RegisterOperators()
      .op("_test::my_op(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<IncrementKernel>(DispatchKey::CPU)
                                                                                      .kernel<ErrorKernel>(DispatchKey::CUDA))
      .op("_test::error(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<ErrorKernel>(DispatchKey::CPU)
                                                                                      .kernel<ErrorKernel>(DispatchKey::CUDA));
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestFunctorBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInMultipleRegistrars_thenCallsRightKernel) {
  auto registrar1 = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<IncrementKernel>(DispatchKey::CPU)
```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `my_op`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `my_op`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 70-86
```cpp
                                                                                                                       .kernel<ErrorKernel>(DispatchKey::CUDA));
  auto registrar2 = RegisterOperators().op("_test::error(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<ErrorKernel>(DispatchKey::CPU)
                                                                                                                       .kernel<ErrorKernel>(DispatchKey::CUDA));
  expectCallsIncrement(DispatchKey::CPU);
}

bool was_called = false;

struct KernelWithoutOutput final : OperatorKernel {
  void operator()(const Tensor&) {
    was_called = true;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::no_return(Tensor dummy) -> ()", RegisterOperators::options().kernel<KernelWithoutOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithoutOutput`, `RegisterOperators`, `op`, `error`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithoutOutput`, `RegisterOperators`, `op`, `error`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 87-104
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_return", ""});
  ASSERT_TRUE(op.has_value());
  was_called = false;
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_TRUE(was_called);
  EXPECT_EQ(0, result.size());
}

struct KernelWithZeroOutputs final : OperatorKernel {
  std::tuple<> operator()(const Tensor&) {
    was_called = true;
    return std::make_tuple();
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithZeroOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::zero_outputs(Tensor dummy) -> ()", RegisterOperators::options().kernel<KernelWithZeroOutputs>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithZeroOutputs`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithZeroOutputs`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 105-122
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::zero_outputs", ""});
  ASSERT_TRUE(op.has_value());
  was_called = false;
  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_TRUE(was_called);
  EXPECT_EQ(0, result.size());
}

struct KernelWithIntOutput final : OperatorKernel {
  int64_t operator()(Tensor, int64_t a, int64_t b) {
    return a + b;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithIntOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_output(Tensor dummy, int a, int b) -> int", RegisterOperators::options().kernel<KernelWithIntOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithIntOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithIntOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 123-141
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_output", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), 3, 6);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(9, result[0].toInt());
}

struct KernelWithTensorOutput final : OperatorKernel {
  Tensor operator()(const Tensor& input) {
    return input;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::returning_tensor(Tensor input) -> Tensor", RegisterOperators::options().kernel<KernelWithTensorOutput>(DispatchKey::CPU)
                                                                                         .kernel<KernelWithTensorOutput>(DispatchKey::CUDA));

```
- EN: Focus symbols: `KernelWithTensorOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTensorOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 142-159
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::returning_tensor", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensor()));

  result = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensor()));
}

struct KernelWithTensorListOutput final : OperatorKernel {
  c10::List<Tensor> operator()(const Tensor& input1, const Tensor& input2, const Tensor& input3) {
    return c10::List<Tensor>({input1, input2, input3});
  }
};

```
- EN: Focus symbols: `KernelWithTensorListOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTensorListOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 160-175
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Tensor input1, Tensor input2, Tensor input3) -> Tensor[]", RegisterOperators::options().kernel<KernelWithTensorListOutput>(DispatchKey::CUDA));

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA), dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(3, result[0].toTensorVector().size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensorVector()[0]));
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(result[0].toTensorVector()[1]));
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(result[0].toTensorVector()[2]));
}

struct KernelWithIntListOutput final : OperatorKernel {
```
- EN: Focus symbols: `KernelWithIntListOutput`, `TEST`, `RegisterOperators`, `op`, `list_output`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithIntListOutput`, `TEST`, `RegisterOperators`, `op`, `list_output`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 176-195
```cpp
  c10::List<int64_t> operator()(const Tensor&, int64_t input1, int64_t input2, int64_t input3) {
    return c10::List<int64_t>({input1, input2, input3});
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithIntListOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::list_output(Tensor dummy, int input1, int input2, int input3) -> int[]", RegisterOperators::options().kernel<KernelWithIntListOutput>(DispatchKey::CPU));

  auto op = c10::Dispatcher::singleton().findSchema({"_test::list_output", ""});
  ASSERT_TRUE(op.has_value());

  auto result = callOp(*op, dummyTensor(DispatchKey::CPU), 2, 4, 6);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(3, result[0].toIntVector().size());
  EXPECT_EQ(2, result[0].toIntVector()[0]);
  EXPECT_EQ(4, result[0].toIntVector()[1]);
  EXPECT_EQ(6, result[0].toIntVector()[2]);
}

```
- EN: Focus symbols: `operator`, `TEST`, `RegisterOperators`, `op`, `list_output`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`operator`, `TEST`, `RegisterOperators`, `op`, `list_output`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 196-214
```cpp
struct KernelWithMultipleOutputs final : OperatorKernel {
  std::tuple<Tensor, int64_t, c10::List<Tensor>, std::optional<int64_t>, Dict<string, Tensor>> operator()(Tensor) {
    Dict<string, Tensor> dict;
    dict.insert("first", dummyTensor(DispatchKey::CPU));
    dict.insert("second", dummyTensor(DispatchKey::CUDA));
    return std::tuple<Tensor, int64_t, c10::List<Tensor>, std::optional<int64_t>, Dict<string, Tensor>>(
      dummyTensor(DispatchKey::CUDA),
      5,
      c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CUDA)}),
      std::optional<int64_t>(std::in_place, 0),
      dict
    );
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithMultipleOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
     .op("_test::multiple_outputs(Tensor dummy) -> (Tensor, int, Tensor[], int?, Dict(str, Tensor))", RegisterOperators::options().kernel<KernelWithMultipleOutputs>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithMultipleOutputs`, `operator`, `insert`, `dummyTensor`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithMultipleOutputs`, `operator`, `insert`, `dummyTensor`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 215-231
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

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 232-248
```cpp
struct KernelWithTensorInputByReferenceWithOutput final : OperatorKernel {
  Tensor operator()(const Tensor& input1) {
    return input1;
  }
};

struct KernelWithTensorInputByValueWithOutput final : OperatorKernel {
  Tensor operator()(Tensor input1) {
    return input1;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorInputByReference_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> Tensor", RegisterOperators::options().kernel<KernelWithTensorInputByReferenceWithOutput>(DispatchKey::CPU)
                                                                                     .kernel<KernelWithTensorInputByReferenceWithOutput>(DispatchKey::CUDA));

```
- EN: Focus symbols: `KernelWithTensorInputByReferenceWithOutput`, `KernelWithTensorInputByValueWithOutput`, `operator`, `TEST`, `RegisterOperators`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTensorInputByReferenceWithOutput`, `KernelWithTensorInputByValueWithOutput`, `operator`, `TEST`, `RegisterOperators`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 249-265
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

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorInputByValue_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> Tensor", RegisterOperators::options().kernel<KernelWithTensorInputByValueWithOutput>(DispatchKey::CPU)
                                                                                     .kernel<KernelWithTensorInputByValueWithOutput>(DispatchKey::CUDA));

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 266-285
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

Tensor captured_input;

struct KernelWithTensorInputByReferenceWithoutOutput final : OperatorKernel {
  void operator()(const Tensor& input1) {
    captured_input = input1;
  }
};

```
- EN: Focus symbols: `KernelWithTensorInputByReferenceWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTensorInputByReferenceWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 286-303
```cpp
struct KernelWithTensorInputByValueWithoutOutput final : OperatorKernel {
  void operator()(Tensor input1) {
    captured_input = input1;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorInputByReference_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> ()", RegisterOperators::options().kernel<KernelWithTensorInputByReferenceWithoutOutput>(DispatchKey::CPU)
                                                                                 .kernel<KernelWithTensorInputByReferenceWithoutOutput>(DispatchKey::CUDA));

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(captured_input));

```
- EN: Focus symbols: `KernelWithTensorInputByValueWithoutOutput`, `operator`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTensorInputByValueWithoutOutput`, `operator`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 304-320
```cpp
  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(captured_input));
}

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorInputByValue_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_input(Tensor input) -> ()", RegisterOperators::options().kernel<KernelWithTensorInputByValueWithoutOutput>(DispatchKey::CPU)
                                                                                 .kernel<KernelWithTensorInputByValueWithoutOutput>(DispatchKey::CUDA));

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(captured_input));

```
- EN: Focus symbols: `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `EXPECT_EQ`, `size`, `extractDispatchKey`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 321-337
```cpp
  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(DispatchKey::CUDA, extractDispatchKey(captured_input));
}

int64_t captured_int_input = 0;

struct KernelWithIntInputWithoutOutput final : OperatorKernel {
  void operator()(Tensor, int64_t input1) {
    captured_int_input = input1;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithIntInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_input(Tensor dummy, int input) -> ()", RegisterOperators::options().kernel<KernelWithIntInputWithoutOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithIntInputWithoutOutput`, `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithIntInputWithoutOutput`, `callOp`, `dummyTensor`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 338-356
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_int_input = 0;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 3);
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(3, captured_int_input);
}

struct KernelWithIntInputWithOutput final : OperatorKernel {
  int64_t operator()(Tensor, int64_t input1) {
    return input1 + 1;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithIntInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_input(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<KernelWithIntInputWithOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithIntInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithIntInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 357-376
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 3);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(4, outputs[0].toInt());
}

int64_t captured_input_list_size = 0;

struct KernelWithIntListInputWithoutOutput final : OperatorKernel {
  void operator()(Tensor, const c10::List<int64_t>& input1) {
    captured_input_list_size = input1.size();
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithIntListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_list_input(Tensor dummy, int[] input) -> ()", RegisterOperators::options().kernel<KernelWithIntListInputWithoutOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithIntListInputWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithIntListInputWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 377-395
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::List<int64_t>({2, 4, 6}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(3, captured_input_list_size);
}

struct KernelWithIntListInputWithOutput final : OperatorKernel {
  int64_t operator()(Tensor, const c10::List<int64_t>& input1) {
    return input1.size();
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithIntListInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::int_list_input(Tensor dummy, int[] input) -> int", RegisterOperators::options().kernel<KernelWithIntListInputWithOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithIntListInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithIntListInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 396-413
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::int_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::List<int64_t>({2, 4, 6}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(3, outputs[0].toInt());
}

struct KernelWithTensorListInputWithoutOutput final : OperatorKernel {
  void operator()(const c10::List<Tensor>& input1) {
    captured_input_list_size = input1.size();
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorListInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> ()", RegisterOperators::options().kernel<KernelWithTensorListInputWithoutOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithTensorListInputWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTensorListInputWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 414-432
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  captured_input_list_size = 0;
  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(0, outputs.size());
  EXPECT_EQ(2, captured_input_list_size);
}

struct KernelWithTensorListInputWithOutput final : OperatorKernel {
  int64_t operator()(const c10::List<Tensor>& input1) {
    return input1.size();
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTensorListInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tensor_list_input(Tensor[] input) -> int", RegisterOperators::options().kernel<KernelWithTensorListInputWithOutput>(DispatchKey::CPU));

```
- EN: Focus symbols: `KernelWithTensorListInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTensorListInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 433-452
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::tensor_list_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, c10::List<Tensor>({dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU)}));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(2, outputs[0].toInt());
}

int captured_dict_size = 0;

struct KernelWithDictInputWithoutOutput final : OperatorKernel {
  void operator()(Dict<string, Tensor> input1) {
    captured_dict_size = input1.size();
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithDictInput_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, Tensor) input) -> ()", RegisterOperators::options().catchAllKernel<KernelWithDictInputWithoutOutput>());

```
- EN: Focus symbols: `KernelWithDictInputWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithDictInputWithoutOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 453-470
```cpp
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

struct KernelWithDictInputWithOutput final : OperatorKernel {
std::string operator()(Dict<string, std::string> input1) {
    return input1.at("key2");
  }
};

```
- EN: Focus symbols: `KernelWithDictInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithDictInputWithOutput`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 471-486
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithDictInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_input(Dict(str, str) input) -> str", RegisterOperators::options().catchAllKernel<KernelWithDictInputWithOutput>());

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_input", ""});
  ASSERT_TRUE(op.has_value());

  Dict<string, std::string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ("value2", outputs[0].toStringRef());
}

struct KernelWithDictOutput final : OperatorKernel {
```
- EN: Focus symbols: `KernelWithDictOutput`, `TEST`, `RegisterOperators`, `op`, `dict_input`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithDictOutput`, `TEST`, `RegisterOperators`, `op`, `dict_input`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 487-505
```cpp
  Dict<string, std::string> operator()(Dict<string, std::string> input) {
    return input;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithDictOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::dict_output(Dict(str, str) input) -> Dict(str, str)", RegisterOperators::options().catchAllKernel<KernelWithDictOutput>());

  auto op = c10::Dispatcher::singleton().findSchema({"_test::dict_output", ""});
  ASSERT_TRUE(op.has_value());

  Dict<string, std::string> dict;
  dict.insert("key1", "value1");
  dict.insert("key2", "value2");
  auto outputs = callOp(*op, dict);
  EXPECT_EQ(1, outputs.size());
  auto output = c10::impl::toTypedDict<string, std::string>(outputs[0].toGenericDict());

```
- EN: Focus symbols: `operator`, `TEST`, `RegisterOperators`, `op`, `dict_output`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`operator`, `TEST`, `RegisterOperators`, `op`, `dict_output`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 506-521
```cpp
  EXPECT_EQ(2, output.size());
  EXPECT_EQ("value1", output.at("key1"));
  EXPECT_EQ("value2", output.at("key2"));
}

class KernelWithCache final : public OperatorKernel {
public:
  KernelWithCache(): counter(3) {}

  int64_t operator()(Tensor) {
    return ++counter;
  }
private:
  int64_t counter;
};

```
- EN: Focus symbols: `KernelWithCache`, `EXPECT_EQ`, `size`, `at`, `counter`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithCache`, `EXPECT_EQ`, `size`, `at`, `counter`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 522-540
```cpp
struct KernelWithTupleInput final : OperatorKernel {
  std::string operator()(std::tuple<string, int64_t, double> input1) {
    return std::get<0>(input1);
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithTupleInput_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::tuple_input((str, int, float) input) -> str", RegisterOperators::options().catchAllKernel<KernelWithTupleInput>());

  auto op = c10::Dispatcher::singleton().findSchema({"_test::tuple_input", ""});
  ASSERT_TRUE(op.has_value());

  std::tuple<string, int64_t, float> tup{"foobar", 123, 420.1337};
  auto outputs = callOp(*op, tup);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ("foobar", outputs[0].toStringRef());
}

```
- EN: Focus symbols: `KernelWithTupleInput`, `operator`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithTupleInput`, `operator`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 541-559
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithCache_thenCacheIsKeptCorrectly) {
  auto registrar = RegisterOperators()
      .op("_test::cache_op(Tensor input) -> int", RegisterOperators::options().kernel<KernelWithCache>(DispatchKey::CPU));

  auto op = c10::Dispatcher::singleton().findSchema({"_test::cache_op", ""});
  ASSERT_TRUE(op.has_value());

  // expect first time calling returns a 4 (4 is the initial value in the cache)
  auto stack = makeStack(dummyTensor(DispatchKey::CPU));
  op->callBoxed(&stack);
  EXPECT_EQ(1, stack.size());
  EXPECT_EQ(4, stack[0].toInt());

  // expect second time calling returns a 5
  stack = makeStack(dummyTensor(DispatchKey::CPU));
  op->callBoxed(&stack);
  EXPECT_EQ(1, stack.size());
  EXPECT_EQ(5, stack[0].toInt());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `cache_op`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `cache_op`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 560-579
```cpp
  // expect third time calling returns a 6
  stack = makeStack(dummyTensor(DispatchKey::CPU));
  op->callBoxed(&stack);
  EXPECT_EQ(1, stack.size());
  EXPECT_EQ(6, stack[0].toInt());
}

class KernelWithConstructorArg final : public OperatorKernel {
public:
  explicit KernelWithConstructorArg(int64_t offset)
  : offset_(offset) {}

  int64_t operator()(const Tensor&, int64_t input) {
    return input + offset_;
  }

private:
  int64_t offset_;
};

```
- EN: Focus symbols: `KernelWithConstructorArg`, `makeStack`, `dummyTensor`, `callBoxed`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithConstructorArg`, `makeStack`, `dummyTensor`, `callBoxed`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 580-596
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithConstructorArg_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::offset_op(Tensor tensor, int input) -> int", RegisterOperators::options().kernel<KernelWithConstructorArg>(DispatchKey::CPU, 2)
                                                                                           .kernel<KernelWithConstructorArg>(DispatchKey::CUDA, 4));

  auto op = c10::Dispatcher::singleton().findSchema({"_test::offset_op", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 4);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(6, outputs[0].toInt());

  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA), 4);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(8, outputs[0].toInt());
}

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `offset_op`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `offset_op`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 597-614
```cpp
class KernelWithMultipleConstructorArgs final : public OperatorKernel {
public:
  explicit KernelWithMultipleConstructorArgs(int64_t offset1, int64_t offset2)
  : offset_(offset1 + offset2) {}

  int64_t operator()(const Tensor&, int64_t input) {
    return input + offset_;
  }

private:
  int64_t offset_;
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithMultipleConstructorArgs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators()
      .op("_test::offset_op(Tensor tensor, int input) -> int", RegisterOperators::options().kernel<KernelWithMultipleConstructorArgs>(DispatchKey::CPU, 2, 3)
                                                                                           .kernel<KernelWithMultipleConstructorArgs>(DispatchKey::CUDA, 4, 5));

```
- EN: Focus symbols: `KernelWithMultipleConstructorArgs`, `offset_`, `operator`, `TEST`, `RegisterOperators`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithMultipleConstructorArgs`, `offset_`, `operator`, `TEST`, `RegisterOperators`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 615-634
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::offset_op", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), 4);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(9, outputs[0].toInt());

  outputs = callOp(*op, dummyTensor(DispatchKey::CUDA), 4);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(13, outputs[0].toInt());
}

bool called = false;

struct KernelWithoutInputs final : OperatorKernel {
  void operator()() {
    called = true;
  }
};

```
- EN: Focus symbols: `KernelWithoutInputs`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithoutInputs`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 635-650
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenFallbackKernelWithoutAnyArguments_whenRegistered_thenCanBeCalled) {
  // note: non-fallback kernels without tensor arguments don't work because there
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args() -> ()", RegisterOperators::options().catchAllKernel<KernelWithoutInputs>());

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op);
  EXPECT_TRUE(called);
}

struct KernelWithoutTensorInputs final : OperatorKernel {
```
- EN: Focus symbols: `KernelWithoutTensorInputs`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithoutTensorInputs`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 651-670
```cpp
  int64_t operator()(int64_t arg) {
    return arg + 1;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenFallbackKernelWithoutTensorArguments_whenRegistered_thenCanBeCalled) {
  // note: non-fallback kernels without tensor arguments don't work because there
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args(int arg) -> int", RegisterOperators::options().catchAllKernel<KernelWithoutTensorInputs>());

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, 3);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(4, outputs[0].toInt());
}

```
- EN: Focus symbols: `operator`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`operator`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 671-688
```cpp
std::optional<Tensor> called_arg2 = std::nullopt;
std::optional<int64_t> called_arg3 = std::nullopt;
std::optional<std::string> called_arg4 = std::nullopt;

struct KernelWithOptInputWithoutOutput final : OperatorKernel {
  void operator()(Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
    called = true;
    called_arg2 = arg2;
    called_arg3 = arg3;
    called_arg4 = arg4;
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithOptionalInputs_withoutOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> ()", RegisterOperators::options().kernel<KernelWithOptInputWithoutOutput>(DispatchKey::CPU));
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `KernelWithOptInputWithoutOutput`, `operator`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithOptInputWithoutOutput`, `operator`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 689-704
```cpp
  called = false;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU), c10::IValue(), std::string("text"));
  EXPECT_EQ(0, outputs.size());

  EXPECT_TRUE(called);
  EXPECT_TRUE(called_arg2.has_value());
  EXPECT_EQ(extractDispatchKey(*called_arg2), DispatchKey::CPU);
  EXPECT_FALSE(called_arg3.has_value());
  EXPECT_TRUE(called_arg4.has_value());
  EXPECT_EQ(*called_arg4, "text");

  called = false;
  outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::IValue(), 4, c10::IValue());
  EXPECT_EQ(0, outputs.size());

  EXPECT_TRUE(called);
```
- EN: Focus symbols: `callOp`, `dummyTensor`, `IValue`, `string`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `dummyTensor`, `IValue`, `string`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 705-720
```cpp
  EXPECT_FALSE(called_arg2.has_value());
  EXPECT_TRUE(called_arg3.has_value());
  EXPECT_EQ(*called_arg3, 4);
  EXPECT_FALSE(called_arg4.has_value());
}

struct KernelWithOptInputWithOutput final : OperatorKernel {
  std::optional<Tensor> operator()(Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
    called = true;
    called_arg2 = arg2;
    called_arg3 = arg3;
    called_arg4 = arg4;
    return arg2;
  }
};

```
- EN: Focus symbols: `KernelWithOptInputWithOutput`, `EXPECT_FALSE`, `has_value`, `EXPECT_TRUE`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithOptInputWithOutput`, `EXPECT_FALSE`, `has_value`, `EXPECT_TRUE`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 721-737
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithOptionalInputs_withOutput_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> Tensor?", RegisterOperators::options().kernel<KernelWithOptInputWithOutput>(DispatchKey::CPU));
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU), c10::IValue(), std::string("text"));
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(outputs[0].toTensor()));

  EXPECT_TRUE(called);
  EXPECT_TRUE(called_arg2.has_value());
  EXPECT_EQ(extractDispatchKey(*called_arg2), DispatchKey::CPU);
  EXPECT_FALSE(called_arg3.has_value());
  EXPECT_TRUE(called_arg4.has_value());
  EXPECT_EQ(*called_arg4, "text");

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `opt_input`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `opt_input`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 738-756
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

struct KernelWithOptInputWithMultipleOutputs final : OperatorKernel {
  std::tuple<std::optional<Tensor>, std::optional<int64_t>, std::optional<std::string>>
  operator()(Tensor arg1, const std::optional<Tensor>& arg2, std::optional<int64_t> arg3, std::optional<std::string> arg4) {
    return std::make_tuple(arg2, arg3, arg4);
  }
};

```
- EN: Focus symbols: `KernelWithOptInputWithMultipleOutputs`, `callOp`, `dummyTensor`, `IValue`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelWithOptInputWithMultipleOutputs`, `callOp`, `dummyTensor`, `IValue`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 757-774
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernelWithOptionalInputs_withMultipleOutputs_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::opt_input(Tensor arg1, Tensor? arg2, int? arg3, str? arg4) -> (Tensor?, int?, str?)", RegisterOperators::options().kernel<KernelWithOptInputWithMultipleOutputs>(DispatchKey::CPU));
  auto op = c10::Dispatcher::singleton().findSchema({"_test::opt_input", ""});
  ASSERT_TRUE(op.has_value());

  auto outputs = callOp(*op, dummyTensor(DispatchKey::CPU), dummyTensor(DispatchKey::CPU), c10::IValue(), std::string("text"));
  EXPECT_EQ(3, outputs.size());
  EXPECT_EQ(DispatchKey::CPU, extractDispatchKey(outputs[0].toTensor()));
  EXPECT_TRUE(outputs[1].isNone());
  EXPECT_EQ("text", outputs[2].toStringRef());

  outputs = callOp(*op, dummyTensor(DispatchKey::CPU), c10::IValue(), 4, c10::IValue());
  EXPECT_EQ(3, outputs.size());
  EXPECT_TRUE(outputs[0].isNone());
  EXPECT_EQ(4, outputs[1].toInt());
  EXPECT_TRUE(outputs[2].isNone());
}

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `opt_input`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `opt_input`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 775-794
```cpp
struct ConcatKernel final : OperatorKernel {
  explicit ConcatKernel(std::string prefix): prefix_(std::move(prefix)) {}

  std::string operator()(const Tensor& tensor1, std::string a, const std::string& b, int64_t c) {
    return prefix_ + a + b + std::to_string(c);
  }

  std::string prefix_;
};

void expectCallsConcatUnboxed(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  std::string result = callOpUnboxed<std::string, const Tensor&, std::string, const std::string&, int64_t>(*op, dummyTensor(dispatch_key), "1", "2", 3);
  EXPECT_EQ("prefix123", result);
}

```
- EN: Focus symbols: `ConcatKernel`, `prefix_`, `move`, `operator`, `to_string`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`ConcatKernel`, `prefix_`, `move`, `operator`, `to_string`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 795-812
```cpp
TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernel_whenRegistered_thenCanBeCalledUnboxed) {
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, str a, str b, int c) -> str", RegisterOperators::options().kernel<ConcatKernel>(DispatchKey::CPU, "prefix"));
  expectCallsConcatUnboxed(DispatchKey::CPU);
}

struct KernelForSchemaInference final : OperatorKernel {
  std::tuple<int64_t, Tensor> operator()(Tensor arg1, int64_t arg2, const c10::List<Tensor>& arg3) {
    return {};
  }
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernel_whenRegisteredWithoutSpecifyingSchema_thenInfersSchema) {
  auto registrar = RegisterOperators()
      .op("_test::no_schema_specified", RegisterOperators::options().kernel<KernelForSchemaInference>(DispatchKey::CPU));

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_schema_specified", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `KernelForSchemaInference`, `TEST`, `RegisterOperators`, `op`, `my_op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelForSchemaInference`, `TEST`, `RegisterOperators`, `op`, `my_op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 813-828
```cpp
  std::optional<std::string> differences = c10::findSchemaDifferences(torch::jit::parseSchema("_test::no_schema_specified(Tensor arg1, int arg2, Tensor[] arg3) -> (int, Tensor)"), op->schema());
  EXPECT_FALSE(differences.has_value());
}

TEST(OperatorRegistrationTestFunctorBasedKernel, givenKernel_whenRegisteredCatchAllWithoutSpecifyingSchema_thenInfersSchema) {
  auto registrar = RegisterOperators()
      .op("_test::no_schema_specified", RegisterOperators::options().catchAllKernel<KernelForSchemaInference>());

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_schema_specified", ""});
  ASSERT_TRUE(op.has_value());

  std::optional<std::string> differences = c10::findSchemaDifferences(torch::jit::parseSchema("_test::no_schema_specified(Tensor arg1, int arg2, Tensor[] arg3) -> (int, Tensor)"), op->schema());
  EXPECT_FALSE(differences.has_value());
}

template<class Return, class... Args> struct KernelFunc final : OperatorKernel{
```
- EN: Focus symbols: `Return`, `KernelFunc`, `findSchemaDifferences`, `parseSchema`, `no_schema_specified`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`Return`, `KernelFunc`, `findSchemaDifferences`, `parseSchema`, `no_schema_specified`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 829-846
```cpp
  Return operator()(Args...) { return {}; }
};
template<class... Args> struct KernelFunc<void, Args...> final : OperatorKernel {
  void operator()(Args...) {}
};

TEST(OperatorRegistrationTestFunctorBasedKernel, givenMismatchedKernel_withDifferentNumArguments_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg, Tensor arg2) -> int", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));
    }, "The number of arguments is different. 2 vs 1"
  );

```
- EN: Focus symbols: `KernelFunc`, `operator`, `TEST`, `RegisterOperators`, `op`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`KernelFunc`, `operator`, `TEST`, `RegisterOperators`, `op`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 847-863
```cpp
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg, Tensor arg2) -> ()", RegisterOperators::options().kernel<KernelFunc<void, Tensor, Tensor>>(DispatchKey::CPU));

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch() -> ()", RegisterOperators::options().kernel<KernelFunc<void, Tensor, Tensor>>(DispatchKey::CPU));
    }, "The number of arguments is different. 0 vs 2"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", RegisterOperators::options().kernel<KernelFunc<void, Tensor, Tensor>>(DispatchKey::CPU));
    }, "The number of arguments is different. 1 vs 2"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 864-882
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg, Tensor arg2, Tensor arg3) -> ()", RegisterOperators::options().kernel<KernelFunc<void, Tensor, Tensor>>(DispatchKey::CPU));
    }, "The number of arguments is different. 3 vs 2"
  );
}

TEST(OperatorRegistrationTestFunctorBasedKernel, givenMismatchedKernel_withDifferentArgumentType_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg1, int arg2) -> int", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor, int64_t>>(DispatchKey::CPU));

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg1, float arg2) -> int", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor, int64_t>>(DispatchKey::CPU));
    }, "Type mismatch in argument 2: float vs int"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 883-901
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(int arg1, int arg2) -> int", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor, int64_t>>(DispatchKey::CPU));
    }, "Type mismatch in argument 1: int vs Tensor"
  );
}

TEST(OperatorRegistrationTestFunctorBasedKernel, givenMismatchedKernel_withDifferentNumReturns_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));
    }, "The number of returns is different. 0 vs 1"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 902-918
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (int, int)", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));
    }, "The number of returns is different. 2 vs 1"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> ()", RegisterOperators::options().kernel<KernelFunc<void, Tensor>>(DispatchKey::CPU));

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", RegisterOperators::options().kernel<KernelFunc<void, Tensor>>(DispatchKey::CPU));
    }, "The number of returns is different. 1 vs 0"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 919-935
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor)", RegisterOperators::options().kernel<KernelFunc<void, Tensor>>(DispatchKey::CPU));
    }, "The number of returns is different. 2 vs 0"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor)", RegisterOperators::options().kernel<KernelFunc<std::tuple<Tensor, Tensor>, Tensor>>(DispatchKey::CPU));

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> ()", RegisterOperators::options().kernel<KernelFunc<std::tuple<Tensor, Tensor>, Tensor>>(DispatchKey::CPU));
    }, "The number of returns is different. 0 vs 2"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 936-953
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", RegisterOperators::options().kernel<KernelFunc<std::tuple<Tensor, Tensor>, Tensor>>(DispatchKey::CPU));
    }, "The number of returns is different. 1 vs 2"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, Tensor, Tensor)", RegisterOperators::options().kernel<KernelFunc<std::tuple<Tensor, Tensor>, Tensor>>(DispatchKey::CPU));
    }, "The number of returns is different. 3 vs 2"
  );
}

TEST(OperatorRegistrationTestFunctorBasedKernel, givenMismatchedKernel_withDifferentReturnTypes_whenRegistering_thenFails) {
  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> int", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`, `TEST`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`, `TEST`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 954-970
```cpp
  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> Tensor", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));
    }, "Type mismatch in return 1: Tensor vs int"
  );

  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> float", RegisterOperators::options().kernel<KernelFunc<int64_t, Tensor>>(DispatchKey::CPU));
    }, "Type mismatch in return 1: float vs int"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> Tensor", RegisterOperators::options().kernel<KernelFunc<Tensor, Tensor>>(DispatchKey::CPU));

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 971-988
```cpp
  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> float", RegisterOperators::options().kernel<KernelFunc<Tensor, Tensor>>(DispatchKey::CPU));
    }, "Type mismatch in return 1: float vs Tensor"
  );

  // assert this does not fail because it matches
  RegisterOperators()
      .op("_test::mismatch(Tensor arg) -> (Tensor, int)", RegisterOperators::options().kernel<KernelFunc<std::tuple<Tensor, int64_t>, Tensor>>(DispatchKey::CPU));

  // and now a set of mismatching schemas
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (Tensor, float)", RegisterOperators::options().kernel<KernelFunc<std::tuple<Tensor, int64_t>, Tensor>>(DispatchKey::CPU));
    }, "Type mismatch in return 2: float vs int"
  );

```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 989-996
```cpp
  expectThrows<c10::Error>([] {
    RegisterOperators()
        .op("_test::mismatch(Tensor arg) -> (int, int)", RegisterOperators::options().kernel<KernelFunc<std::tuple<Tensor, int64_t>, Tensor>>(DispatchKey::CPU));
    }, "Type mismatch in return 1: int vs Tensor"
  );
}

}
```
- EN: Focus symbols: `RegisterOperators`, `op`, `mismatch`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `mismatch`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- Type-system design / 类型系统设计
- Backend/device dispatch / 后端/设备分发
- Behavioral regression tests / 行为回归测试
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/boxing/impl/test_helpers.h`, `ATen/core/op_registration/op_registration.h`, `ATen/core/Tensor.h`, `torch/csrc/jit/frontend/function_schema_parser.h`, `torch/library.h`, `ATen/core/LegacyTypeDispatch.h`
- External/system includes / 外部或系统头: `gtest/gtest.h`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; namespace scoping / 命名空间作用域
