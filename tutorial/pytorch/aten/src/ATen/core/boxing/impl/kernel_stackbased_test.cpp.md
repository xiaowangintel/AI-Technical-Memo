# kernel_stackbased_test.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/boxing/impl/kernel_stackbased_test.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `errorKernel`, `EXPECT_TRUE`, `incrementKernel`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `errorKernel`, `EXPECT_TRUE`, `incrementKernel`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp

#include <gtest/gtest.h>
#include <ATen/core/boxing/impl/test_helpers.h>

#include <ATen/core/op_registration/op_registration.h>
#include <ATen/core/Tensor.h>
#include <torch/csrc/jit/frontend/function_schema_parser.h>
#include <torch/library.h>

#include <ATen/core/LegacyTypeDispatch.h>

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 12-24
```cpp
using c10::RegisterOperators;
using c10::DispatchKey;
using c10::Stack;
using std::make_unique;
using c10::OperatorHandle;
using std::unique_ptr;

namespace {

void errorKernel(const OperatorHandle&, Stack* stack) {
  EXPECT_TRUE(false); // this kernel should never be called
}

```
- EN: Focus symbols: `errorKernel`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`errorKernel`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 25-36
```cpp
void incrementKernel(const OperatorHandle&, Stack* stack) {
  int input = torch::jit::pop(*stack).toInt();
  torch::jit::pop(*stack); // pop the dummy tensor
  torch::jit::push(*stack, input + 1);
}

void decrementKernel(const OperatorHandle&, Stack* stack) {
  int input = torch::jit::pop(*stack).toInt();
  torch::jit::pop(*stack); // pop the dummy tensor
  torch::jit::push(*stack, input - 1);
}

```
- EN: Focus symbols: `incrementKernel`, `pop`, `toInt`, `push`, `decrementKernel`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`incrementKernel`, `pop`, `toInt`, `push`, `decrementKernel`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 37-47
```cpp
bool called_redispatching_kernel = false;
void redispatchingKernel_with_DispatchKeySet(const OperatorHandle& op, c10::DispatchKeySet ks, Stack* stack) {
  // this kernel is a no-op- it just redispatches to the lower-priority kernel
  called_redispatching_kernel = true;
  auto updated_ks = ks & c10::DispatchKeySet(c10::DispatchKeySet::FULL_AFTER, c10::DispatchKey::TESTING_ONLY_GenericWrapper);
  op.redispatchBoxed(updated_ks, stack);
}

void expectCallsIncrement(c10::DispatchKeySet ks) {
  at::AutoDispatchBelowAutograd mode;

```
- EN: Focus symbols: `redispatchingKernel_with_DispatchKeySet`, `DispatchKeySet`, `redispatchBoxed`, `expectCallsIncrement`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`redispatchingKernel_with_DispatchKeySet`, `DispatchKeySet`, `redispatchBoxed`, `expectCallsIncrement`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 48-59
```cpp
  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  auto result = callOp(*op, dummyTensor(ks), 5);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(6, result[0].toInt());
}

void expectCallsIncrement(DispatchKey dispatch_key) {
  expectCallsIncrement(c10::DispatchKeySet(dispatch_key));
}

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 60-72
```cpp
void expectCallsIncrementUnboxed(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  int64_t result = callOpUnboxed<int64_t, at::Tensor, int64_t>(*op, dummyTensor(dispatch_key), 5);
  EXPECT_EQ(6, result);
}

void expectCallsDecrement(DispatchKey dispatch_key) {
  at::AutoDispatchBelowAutograd mode;

```
- EN: Focus symbols: `expectCallsIncrementUnboxed`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`expectCallsIncrementUnboxed`, `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 73-85
```cpp
  // assert that schema and cpu kernel are present
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());
  auto result = callOp(*op, dummyTensor(dispatch_key), 5);
  EXPECT_EQ(1, result.size());
  EXPECT_EQ(4, result[0].toInt());
}

TEST(OperatorRegistrationTestStackBasedKernel, givenKernel_whenRegistered_thenCanBeCalled) {
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<&incrementKernel>(DispatchKey::CPU));
  expectCallsIncrement(DispatchKey::CPU);
}

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 86-95
```cpp
TEST(OperatorRegistrationTestStackBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInOneRegistrar_thenCallsRightKernel) {
  auto registrar = RegisterOperators()
      .op("_test::my_op(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<&incrementKernel>(DispatchKey::CPU)
                                                                                      .kernel<&errorKernel>(DispatchKey::CUDA))
      .op("_test::error(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<&errorKernel>(DispatchKey::CPU)
                                                                                      .kernel<&errorKernel>(DispatchKey::CUDA));
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestStackBasedKernel, givenMultipleOperatorsAndKernels_whenRegisteredInMultipleRegistrars_thenCallsRightKernel) {
```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `my_op`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `my_op`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 96-105
```cpp
  auto registrar1 = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<&incrementKernel>(DispatchKey::CPU)
                                                                                                                       .kernel<&errorKernel>(DispatchKey::CUDA));
  auto registrar2 = RegisterOperators().op("_test::error(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<&errorKernel>(DispatchKey::CPU)
                                                                                                                       .kernel<&errorKernel>(DispatchKey::CUDA));
  expectCallsIncrement(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestStackBasedKernel, givenKernel_whenRegistrationRunsOutOfScope_thenCannotBeCalledAnymore) {
  {
    auto m = MAKE_TORCH_LIBRARY(_test);
```
- EN: Focus symbols: `RegisterOperators`, `op`, `my_op`, `options`, `error`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`RegisterOperators`, `op`, `my_op`, `options`, `error`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 106-117
```cpp
    m.def("_test::my_op(Tensor dummy, int input) -> int");
    auto m_cpu = MAKE_TORCH_LIBRARY_IMPL(_test, CPU);
    m_cpu.impl("my_op", DispatchKey::CPU, torch::CppFunction::makeFromBoxedFunction<incrementKernel>());
    {
      auto m_cuda = MAKE_TORCH_LIBRARY_IMPL(_test, CUDA);
      m_cuda.impl("my_op", DispatchKey::CUDA, torch::CppFunction::makeFromBoxedFunction<decrementKernel>());

      // assert that schema and cpu kernel are present
      expectCallsIncrement(DispatchKey::CPU);
      expectCallsDecrement(DispatchKey::CUDA);
    }

```
- EN: Focus symbols: `def`, `my_op`, `MAKE_TORCH_LIBRARY_IMPL`, `impl`, `expectCallsIncrement`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`def`, `my_op`, `MAKE_TORCH_LIBRARY_IMPL`, `impl`, `expectCallsIncrement`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 118-128
```cpp
    // now registrar2 is destructed. Assert that schema is still present but cpu kernel is not
    expectCallsIncrement(DispatchKey::CPU);
    expectDoesntFindKernel("_test::my_op", DispatchKey::CUDA);
  }

  // now both registrars are destructed. Assert that the whole schema is gone
  expectDoesntFindOperator("_test::my_op");
}

bool called = false;

```
- EN: Focus symbols: `expectCallsIncrement`, `expectDoesntFindKernel`, `expectDoesntFindOperator`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`expectCallsIncrement`, `expectDoesntFindKernel`, `expectDoesntFindOperator`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 129-139
```cpp
void kernelWithoutInputs(const OperatorHandle&, Stack*) {
  called = true;
}

TEST(OperatorRegistrationTestStackBasedKernel, givenFallbackKernelWithoutAnyArguments_whenRegistered_thenCanBeCalled) {
  // note: non-fallback kernels without tensor arguments don't work because there
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args() -> ()", RegisterOperators::options().catchAllKernel<&kernelWithoutInputs>());

```
- EN: Focus symbols: `kernelWithoutInputs`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernelWithoutInputs`, `TEST`, `RegisterOperators`, `op`, `no_tensor_args`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 140-151
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

  called = false;
  auto outputs = callOp(*op);
  EXPECT_TRUE(called);
}

void kernelWithoutTensorInputs(const OperatorHandle&, Stack* stack) {
  stack->back() = stack->back().toInt() + 1;
}

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `callOp`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 152-161
```cpp
TEST(OperatorRegistrationTestStackBasedKernel, givenFallbackKernelWithoutTensorArguments_whenRegistered_thenCanBeCalled) {
  // note: non-fallback kernels without tensor arguments don't work because there
  // is no way to get the dispatch key. For operators that only have a fallback
  // kernel, this must work for backwards compatibility.
  auto registrar = RegisterOperators()
      .op("_test::no_tensor_args(int arg) -> int", RegisterOperators::options().catchAllKernel<&kernelWithoutTensorInputs>());

  auto op = c10::Dispatcher::singleton().findSchema({"_test::no_tensor_args", ""});
  ASSERT_TRUE(op.has_value());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `no_tensor_args`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `no_tensor_args`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 162-175
```cpp
  auto outputs = callOp(*op, 3);
  EXPECT_EQ(1, outputs.size());
  EXPECT_EQ(4, outputs[0].toInt());
}

void kernelForSchemaInference(const OperatorHandle&, Stack* stack) {
}

TEST(OperatorRegistrationTestStackBasedKernel, givenKernel_whenRegisteredWithoutSpecifyingSchema_thenFailsBecauseItCannotInferFromStackBasedKernel) {
  expectThrows<c10::Error>([] {
      RegisterOperators().op("_test::no_schema_specified", RegisterOperators::options().catchAllKernel<&kernelForSchemaInference>());
  }, "Cannot infer operator schema for this kind of kernel in registration of operator _test::no_schema_specified");
}

```
- EN: Focus symbols: `callOp`, `EXPECT_EQ`, `size`, `toInt`, `kernelForSchemaInference`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callOp`, `EXPECT_EQ`, `size`, `toInt`, `kernelForSchemaInference`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 176-188
```cpp
TEST(OperatorRegistrationTestStackBasedKernel, givenKernel_whenRegistered_thenCanAlsoBeCalledUnboxed) {
  auto registrar = RegisterOperators().op("_test::my_op(Tensor dummy, int input) -> int", RegisterOperators::options().kernel<&incrementKernel>(DispatchKey::CPU));
  expectCallsIncrementUnboxed(DispatchKey::CPU);
}

TEST(OperatorRegistrationTestStackBasedKernel, callKernelsWithDispatchKeySetConvention_redispatchesToLowerPriorityKernels) {
  auto m = MAKE_TORCH_LIBRARY(_test);
  m.def("my_op(Tensor dummy, int input) -> int");
  auto m_cpu = MAKE_TORCH_LIBRARY_IMPL(_, CPU);
  m_cpu.fallback(torch::CppFunction::makeFromBoxedFunction<&incrementKernel>());
  auto m_testing = MAKE_TORCH_LIBRARY_IMPL(_, TESTING_ONLY_GenericWrapper);
  m_testing.fallback(torch::CppFunction::makeFromBoxedFunction<&redispatchingKernel_with_DispatchKeySet>());

```
- EN: Focus symbols: `TEST`, `RegisterOperators`, `op`, `my_op`, `options`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `RegisterOperators`, `op`, `my_op`, `options`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 189-200
```cpp
  auto op = c10::Dispatcher::singleton().findSchema({"_test::my_op", ""});
  ASSERT_TRUE(op.has_value());

  auto testing_cpu_set = c10::DispatchKeySet()
                                    .add(c10::DispatchKey::TESTING_ONLY_GenericWrapper)
                                    .add(c10::DispatchKey::CPU);
  called_redispatching_kernel = false;

  // call CPU (and not TESTING_ONLY_GenericWrapper)
  expectCallsIncrement(DispatchKey::CPU);
  ASSERT_FALSE(called_redispatching_kernel);

```
- EN: Focus symbols: `singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `DispatchKeySet`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`singleton`, `findSchema`, `ASSERT_TRUE`, `has_value`, `DispatchKeySet`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 201-206
```cpp
  // call TESTING_ONLY_GenericWrapper -> call CPU
  expectCallsIncrement(testing_cpu_set);
  ASSERT_TRUE(called_redispatching_kernel);
}

}
```
- EN: Focus symbols: `expectCallsIncrement`, `ASSERT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`expectCallsIncrement`, `ASSERT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- Backend/device dispatch / 后端/设备分发
- Behavioral regression tests / 行为回归测试
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/core/boxing/impl/test_helpers.h`, `ATen/core/op_registration/op_registration.h`, `ATen/core/Tensor.h`, `torch/csrc/jit/frontend/function_schema_parser.h`, `torch/library.h`, `ATen/core/LegacyTypeDispatch.h`
- External/system includes / 外部或系统头: `gtest/gtest.h`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; operator registration / 算子注册; namespace scoping / 命名空间作用域
