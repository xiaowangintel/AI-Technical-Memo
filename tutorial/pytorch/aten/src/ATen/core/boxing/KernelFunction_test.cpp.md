# KernelFunction_test.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/core/boxing/KernelFunction_test.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements core ATen/C10 abstractions, with primary focus on `unboxed_functor_with_return`, `unboxed_functor_without_return`, `unboxed_functor_with_return_factory`.
- 用途（中文）: 该文件实现ATen/C10 核心抽象，核心关注对象是 `unboxed_functor_with_return`, `unboxed_functor_without_return`, `unboxed_functor_with_return_factory`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
#include <gtest/gtest.h>
#include <ATen/ATen.h>
#include <ATen/core/boxing/KernelFunction.h>
#include <ATen/core/boxing/impl/test_helpers.h>
#include <ATen/core/op_registration/op_registration.h>

using std::vector;
using std::tuple;
using std::optional;
using c10::IValue;
using c10::OperatorKernel;
using c10::OperatorHandle;
using c10::Stack;
using c10::KernelFunction;

namespace {

```
- EN: This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 18-35
```cpp
namespace kernels {
// This namespace contains several fake kernels.
// Some of these kernels expect to be called with two int64_t arguments
// and store these arguments in called_with_args.
// Kernels may return a single value, or multiple values, or no value.
// The kernels with a single return value return int value 5,
// The expectXXX() functions further below use these invariants
// to check that calling a specific kernels works correctly.

optional<tuple<int64_t, int64_t>> called_with_args;


// The calling convention in the dispatcher requires that calls to KernelFunction::call()/callBoxed()
// take in a DispatchKeySet.
// The value itself is meaningless for all of the tests that use kernels without a DispatchKeySet argument.
// See Note [Plumbing Keys Through The Dispatcher] for details.
c10::DispatchKeySet CPU_TEST_SET = c10::DispatchKeySet(c10::DispatchKey::CPU);

```
- EN: Focus symbols: `kernels`, `contains`, `DispatchKeySet`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`kernels`, `contains`, `DispatchKeySet`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 36-54
```cpp
void boxed_func_with_return(const OperatorHandle& /*opHandle*/, Stack* stack) {
  EXPECT_EQ(2, stack->size());
  EXPECT_TRUE(stack->at(0).isInt());
  EXPECT_TRUE(stack->at(1).isInt());
  called_with_args = tuple<int64_t, int64_t>(stack->at(0).toInt(), stack->at(1).toInt());

  stack->clear();
  stack->push_back(5);
}

void boxed_func_without_return(const OperatorHandle& /*opHandle*/, Stack* stack) {
  EXPECT_EQ(2, stack->size());
  EXPECT_TRUE(stack->at(0).isInt());
  EXPECT_TRUE(stack->at(1).isInt());
  called_with_args = tuple<int64_t, int64_t>(stack->at(0).toInt(), stack->at(1).toInt());

  stack->clear();
}

```
- EN: Focus symbols: `boxed_func_with_return`, `EXPECT_EQ`, `size`, `EXPECT_TRUE`, `at`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`boxed_func_with_return`, `EXPECT_EQ`, `size`, `EXPECT_TRUE`, `at`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 55-74
```cpp
void boxed_func_with_multi_return(const OperatorHandle& /*opHandle*/, Stack* stack) {
  EXPECT_EQ(2, stack->size());
  EXPECT_TRUE(stack->at(0).isInt());
  int64_t a = stack->at(0).toInt();
  EXPECT_TRUE(stack->at(1).isInt());
  int64_t b = stack->at(1).toInt();
  called_with_args = tuple<int64_t, int64_t>(a, b);

  stack->clear();
  torch::jit::push(stack, a + b);
  torch::jit::push(stack, a * b);
}

struct unboxed_functor_with_return final : OperatorKernel {
  int64_t operator()(int64_t a, int64_t b) {
    called_with_args = tuple<int64_t, int64_t>(a, b);
    return 5;
  }
};

```
- EN: Focus symbols: `unboxed_functor_with_return`, `boxed_func_with_multi_return`, `EXPECT_EQ`, `size`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`unboxed_functor_with_return`, `boxed_func_with_multi_return`, `EXPECT_EQ`, `size`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 75-92
```cpp
struct unboxed_functor_without_return final : OperatorKernel {
  void operator()(int64_t a, int64_t b) {
    called_with_args = tuple<int64_t, int64_t>(a, b);
  }
};

struct unboxed_functor_with_return_factory final {
  std::unique_ptr<OperatorKernel> operator()() {
    return std::make_unique<unboxed_functor_with_return>();
  }
};

struct unboxed_functor_without_return_factory final {
  std::unique_ptr<OperatorKernel> operator()() {
    return std::make_unique<unboxed_functor_without_return>();
  }
};

```
- EN: Focus symbols: `unboxed_functor_without_return`, `unboxed_functor_with_return_factory`, `unboxed_functor_without_return_factory`, `operator`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`unboxed_functor_without_return`, `unboxed_functor_with_return_factory`, `unboxed_functor_without_return_factory`, `operator`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 93-110
```cpp
int64_t unboxed_function_with_return(int64_t a, int64_t b) {
  called_with_args = tuple<int64_t, int64_t>(a, b);
  return 5;
}

void unboxed_function_without_return(int64_t a, int64_t b) {
  called_with_args = tuple<int64_t, int64_t>(a, b);
}

auto unboxed_lambda_with_return = [] (int64_t a, int64_t b) -> int64_t {
  called_with_args = tuple<int64_t, int64_t>(a, b);
  return 5;
};

auto unboxed_lambda_without_return = [] (int64_t a, int64_t b) -> void{
  called_with_args = tuple<int64_t, int64_t>(a, b);
};

```
- EN: Focus symbols: `unboxed_function_with_return`, `unboxed_function_without_return`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`unboxed_function_with_return`, `unboxed_function_without_return`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 111-129
```cpp
OperatorHandle makeDummyOperatorHandle() {
  static auto registry = torch::RegisterOperators().op("my::dummy() -> ()");
  return c10::Dispatcher::singleton().findSchema({"my::dummy", ""}).value();
}

//
// boxed kernels that return refs to tensor arguments, a la inplace/outplace kernels
//

void boxed_func_for_inplace_op(const OperatorHandle& /*opHandle*/, Stack* stack) {
  // (Tensor(a!), Scalar) -> Tensor(a!)
  EXPECT_EQ(2, stack->size());

  ASSERT_TRUE(stack->at(0).isTensor());
  auto t = stack->at(0).toTensor();

  ASSERT_TRUE(stack->at(1).isScalar());
  auto s = stack->at(1).toScalar();

```
- EN: Focus symbols: `makeDummyOperatorHandle`, `RegisterOperators`, `op`, `dummy`, `singleton`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`makeDummyOperatorHandle`, `RegisterOperators`, `op`, `dummy`, `singleton`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 130-147
```cpp
  t.add_(s);

  stack->clear();
  torch::jit::push(stack, t);
}

void boxed_func_for_outofplace_op(const OperatorHandle& /*opHandle*/, Stack* stack) {
  // (Scalar, Tensor(a!)) -> Tensor(a!)
  EXPECT_EQ(2, stack->size());

  ASSERT_TRUE(stack->at(0).isScalar());
  auto s = stack->at(0).toScalar();

  ASSERT_TRUE(stack->at(1).isTensor());
  auto t = stack->at(1).toTensor();

  t.add_(s);

```
- EN: Focus symbols: `add_`, `clear`, `push`, `boxed_func_for_outofplace_op`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`add_`, `clear`, `push`, `boxed_func_for_outofplace_op`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 148-164
```cpp
  stack->clear();
  torch::jit::push(stack, t);
}

void boxed_func_for_outofplace_multi_op(const OperatorHandle& /*opHandle*/, Stack* stack) {
  // (Scalar, Scalar, Tensor(a!), Tensor(b!)) -> (Tensor(a!), Tensor(b!))
  EXPECT_EQ(4, stack->size());

  ASSERT_TRUE(stack->at(0).isScalar());
  auto s1 = stack->at(0).toScalar();

  ASSERT_TRUE(stack->at(1).isScalar());
  auto s2 = stack->at(1).toScalar();

  ASSERT_TRUE(stack->at(2).isTensor());
  auto t1 = stack->at(2).toTensor();

```
- EN: Focus symbols: `clear`, `push`, `boxed_func_for_outofplace_multi_op`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`clear`, `push`, `boxed_func_for_outofplace_multi_op`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 165-181
```cpp
  ASSERT_TRUE(stack->at(3).isTensor());
  auto t2 = stack->at(3).toTensor();

  t1.add_(s1);
  t2.add_(s2);

  stack->clear();
  torch::jit::push(stack, t1);
  torch::jit::push(stack, t2);
}

//
// boxed calling tests:
//

// functional

```
- EN: Focus symbols: `ASSERT_TRUE`, `at`, `isTensor`, `toTensor`, `add_`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`ASSERT_TRUE`, `at`, `isTensor`, `toTensor`, `add_`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 182-200
```cpp
void expectBoxedCallingWithReturnWorks(const KernelFunction& func) {
  called_with_args = std::nullopt;
  vector<IValue> stack {3, 4};
  OperatorHandle dummy = makeDummyOperatorHandle();

  func.callBoxed(dummy, CPU_TEST_SET, &stack);

  EXPECT_TRUE(called_with_args.has_value());
  EXPECT_EQ((tuple<int64_t, int64_t>(3, 4)), *called_with_args);
  EXPECT_EQ(1, stack.size());
  EXPECT_TRUE(stack[0].isInt());
  EXPECT_EQ(5, stack[0].toInt());
}

void expectBoxedCallingWithoutReturnWorks(const KernelFunction& func) {
  called_with_args = std::nullopt;
  vector<IValue> stack {3, 4};
  OperatorHandle dummy = makeDummyOperatorHandle();

```
- EN: Focus symbols: `expectBoxedCallingWithReturnWorks`, `makeDummyOperatorHandle`, `callBoxed`, `EXPECT_TRUE`, `has_value`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`expectBoxedCallingWithReturnWorks`, `makeDummyOperatorHandle`, `callBoxed`, `EXPECT_TRUE`, `has_value`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 201-218
```cpp
  func.callBoxed(dummy, CPU_TEST_SET, &stack);

  EXPECT_TRUE(called_with_args.has_value());
  EXPECT_EQ((tuple<int64_t, int64_t>(3, 4)), *called_with_args);
  EXPECT_EQ(0, stack.size());
}

void expectBoxedCallingWithMultiReturnWorks(const KernelFunction& func) {
  called_with_args = std::nullopt;
  vector<IValue> stack {3, 4};
  OperatorHandle dummy = makeDummyOperatorHandle();

  func.callBoxed(dummy, CPU_TEST_SET, &stack);

  EXPECT_TRUE(called_with_args.has_value());
  EXPECT_EQ((tuple<int64_t, int64_t>(3, 4)), *called_with_args);
  EXPECT_EQ(2, stack.size());

```
- EN: Focus symbols: `callBoxed`, `EXPECT_TRUE`, `has_value`, `EXPECT_EQ`, `size`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`callBoxed`, `EXPECT_TRUE`, `has_value`, `EXPECT_EQ`, `size`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 219-235
```cpp
  EXPECT_TRUE(stack[0].isInt());
  EXPECT_EQ(7, stack[0].toInt());

  EXPECT_TRUE(stack[1].isInt());
  EXPECT_EQ(12, stack[1].toInt());
}

// in/out

void expectInPlaceBoxedCallingWorks(const KernelFunction& func) {
  OperatorHandle dummy = makeDummyOperatorHandle();

  auto t = at::zeros({1});
  auto s = 1.0f;
  vector<IValue> stack {t, s};
  func.callBoxed(dummy, CPU_TEST_SET, &stack);

```
- EN: Focus symbols: `EXPECT_TRUE`, `isInt`, `EXPECT_EQ`, `toInt`, `expectInPlaceBoxedCallingWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_TRUE`, `isInt`, `EXPECT_EQ`, `toInt`, `expectInPlaceBoxedCallingWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 236-251
```cpp
  // kernel should have updated out arg and returned it
  EXPECT_EQ(t.item().toFloat(), 1.0f);
  EXPECT_EQ(1, stack.size());
  EXPECT_TRUE(stack[0].isTensor());
  EXPECT_TRUE(stack[0].toTensor().is_same(t));
}

void expectOutOfPlaceBoxedCallingWorks(const KernelFunction& func) {
  OperatorHandle dummy = makeDummyOperatorHandle();

  auto s = 1.0f;
  auto t = at::zeros({1});
  vector<IValue> stack {s, t};
  func.callBoxed(dummy, CPU_TEST_SET, &stack);

  // kernel should have updated out arg and returned it on the stack
```
- EN: Focus symbols: `EXPECT_EQ`, `item`, `toFloat`, `size`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `item`, `toFloat`, `size`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 252-267
```cpp
  EXPECT_EQ(t.item().toFloat(), 1.0f);
  EXPECT_EQ(1, stack.size());
  EXPECT_TRUE(stack[0].isTensor());
  EXPECT_TRUE(stack[0].toTensor().is_same(t));
}

void expectOutOfPlaceMultiBoxedCallingWorks(const KernelFunction& func) {
  OperatorHandle dummy = makeDummyOperatorHandle();

  auto s1 = 1.0f;
  auto s2 = 2.0f;
  auto t1 = at::zeros({1});
  auto t2 = at::zeros({1});
  vector<IValue> stack {s1, s2, t1, t2};
  func.callBoxed(dummy, CPU_TEST_SET, &stack);

```
- EN: Focus symbols: `EXPECT_EQ`, `item`, `toFloat`, `size`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `item`, `toFloat`, `size`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 268-283
```cpp
  // kernel should have updated output args and returned them on the stack
  EXPECT_EQ(t1.item().toFloat(), 1.0f);
  EXPECT_EQ(t2.item().toFloat(), 2.0f);
  EXPECT_EQ(2, stack.size());
  EXPECT_TRUE(stack[0].isTensor());
  EXPECT_TRUE(stack[0].toTensor().is_same(t1));
  EXPECT_TRUE(stack[1].isTensor());
  EXPECT_TRUE(stack[1].toTensor().is_same(t2));
}

//
// unboxed calling tests:
//

// functional

```
- EN: Focus symbols: `EXPECT_EQ`, `item`, `toFloat`, `size`, `EXPECT_TRUE`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `item`, `toFloat`, `size`, `EXPECT_TRUE`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 284-302
```cpp
// make an unboxed call to a kernel that returns a single value.
//
void expectUnboxedCallingWithReturnWorks(const KernelFunction& func) {
  called_with_args = std::nullopt;
  OperatorHandle dummy = makeDummyOperatorHandle();

  int64_t result = func.call<int64_t, int64_t, int64_t>(dummy, CPU_TEST_SET, 3, 4);

  EXPECT_TRUE(called_with_args.has_value());
  EXPECT_EQ((tuple<int64_t, int64_t>(3, 4)), *called_with_args);
  EXPECT_EQ(5, result);
}

// make an unboxed call to a kernel that returns nothing.
//
void expectUnboxedCallingWithoutReturnWorks(const KernelFunction& func) {
  called_with_args = std::nullopt;
  OperatorHandle dummy = makeDummyOperatorHandle();

```
- EN: Focus symbols: `expectUnboxedCallingWithReturnWorks`, `makeDummyOperatorHandle`, `EXPECT_TRUE`, `has_value`, `EXPECT_EQ`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`expectUnboxedCallingWithReturnWorks`, `makeDummyOperatorHandle`, `EXPECT_TRUE`, `has_value`, `EXPECT_EQ`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 303-320
```cpp
  func.call<void, int64_t, int64_t>(dummy, CPU_TEST_SET, 3, 4);

  EXPECT_TRUE(called_with_args.has_value());
  EXPECT_EQ((tuple<int64_t, int64_t>(3, 4)), *called_with_args);
}

// make an unboxed call to a kernel that returns two values.
// When calling unboxed, multiple values are returned as a tuple.
//
void expectUnboxedCallingWithMultiReturnWorks(const KernelFunction& func) {
  called_with_args = std::nullopt;
  OperatorHandle dummy = makeDummyOperatorHandle();

  auto result = func.call<std::tuple<int64_t, int64_t>, int64_t, int64_t>(dummy, CPU_TEST_SET, 3, 4);

  EXPECT_TRUE(called_with_args.has_value());
  EXPECT_EQ((tuple<int64_t, int64_t>(3, 4)), *called_with_args);

```
- EN: Focus symbols: `EXPECT_TRUE`, `has_value`, `EXPECT_EQ`, `expectUnboxedCallingWithMultiReturnWorks`, `makeDummyOperatorHandle`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_TRUE`, `has_value`, `EXPECT_EQ`, `expectUnboxedCallingWithMultiReturnWorks`, `makeDummyOperatorHandle`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 321-339
```cpp
  EXPECT_EQ((tuple<int64_t, int64_t>(7, 12)), result);
}

// in/out

void expectInPlaceUnboxedCallingWorks(const KernelFunction& func) {
  OperatorHandle dummy = makeDummyOperatorHandle();

  auto t = at::zeros({1});
  at::Tensor& t_out = func.call<at::Tensor&, at::Tensor&, at::Scalar>(dummy, CPU_TEST_SET, t, 1.0f);

  // should have updated first arg and returned it
  EXPECT_EQ(t.item().toFloat(), 1.0f);
  EXPECT_EQ(&t, &t_out);
}

void expectOutOfPlaceUnboxedCallingWorks(const KernelFunction& func) {
  OperatorHandle dummy = makeDummyOperatorHandle();

```
- EN: Focus symbols: `EXPECT_EQ`, `expectInPlaceUnboxedCallingWorks`, `makeDummyOperatorHandle`, `zeros`, `item`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `expectInPlaceUnboxedCallingWorks`, `makeDummyOperatorHandle`, `zeros`, `item`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 340-359
```cpp
  auto t = at::zeros({1});
  at::Tensor& t_out = func.call<at::Tensor&, at::Scalar, at::Tensor&>(dummy, CPU_TEST_SET, 1.0f, t);

  // should have updated out arg and returned it
  EXPECT_EQ(t.item().toFloat(), 1.0f);
  EXPECT_EQ(&t, &t_out);
}

void expectOutOfPlaceMultiUnboxedCallingWorks(const KernelFunction& func) {
  OperatorHandle dummy = makeDummyOperatorHandle();

  auto s1 = 1.0f;
  auto s2 = 2.0f;
  auto t1 = at::zeros({1});
  auto t2 = at::zeros({1});

  const auto [t1_out, t2_out] = func.call<
    std::tuple<at::Tensor&, at::Tensor&>, at::Scalar, at::Scalar, at::Tensor&, at::Tensor&
  >(dummy, CPU_TEST_SET, s1, s2, t1, t2);

```
- EN: Focus symbols: `zeros`, `EXPECT_EQ`, `item`, `toFloat`, `expectOutOfPlaceMultiUnboxedCallingWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`zeros`, `EXPECT_EQ`, `item`, `toFloat`, `expectOutOfPlaceMultiUnboxedCallingWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 360-379
```cpp
  // kernel should have updated out args and returned them in a tuple
  EXPECT_EQ(t1.item().toFloat(), 1.0f);
  EXPECT_EQ(t2.item().toFloat(), 2.0f);

  EXPECT_EQ(t1_out.item().toFloat(), 1.0f);
  EXPECT_TRUE(t1_out.is_same(t1));

  EXPECT_EQ(t2_out.item().toFloat(), 2.0f);
  EXPECT_TRUE(t2_out.is_same(t2));
}

}

// functional, boxed calling

TEST(KernelFunctionTest, givenBoxedFunction_withReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_with_return>();
  kernels::expectBoxedCallingWithReturnWorks(func);
}

```
- EN: Focus symbols: `EXPECT_EQ`, `item`, `toFloat`, `EXPECT_TRUE`, `is_same`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`EXPECT_EQ`, `item`, `toFloat`, `EXPECT_TRUE`, `is_same`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 380-396
```cpp
TEST(KernelFunctionTest, givenBoxedFunction_withoutReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_without_return>();
  kernels::expectBoxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenBoxedFunction_withMultiReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_with_multi_return>();
  kernels::expectBoxedCallingWithMultiReturnWorks(func);
}

// in/out, boxed calling

TEST(KernelFunctionTest, givenBoxedFunction_withInPlaceSignature_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_for_inplace_op>();
  kernels::expectInPlaceBoxedCallingWorks(func);
}

```
- EN: Focus symbols: `TEST`, `expectBoxedCallingWithoutReturnWorks`, `expectBoxedCallingWithMultiReturnWorks`, `expectInPlaceBoxedCallingWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `expectBoxedCallingWithoutReturnWorks`, `expectBoxedCallingWithMultiReturnWorks`, `expectInPlaceBoxedCallingWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 397-413
```cpp
TEST(KernelFunctionTest, givenBoxedFunction_withOutOfPlaceSignature_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_for_outofplace_op>();
  kernels::expectOutOfPlaceBoxedCallingWorks(func);
}

TEST(KernelFunctionTest, givenBoxedFunction_withOutOfPlaceMultiSignature_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_for_outofplace_multi_op>();
  kernels::expectOutOfPlaceMultiBoxedCallingWorks(func);
}

// functional, unboxed calling

TEST(KernelFunctionTest, givenBoxedFunction_withReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_with_return>();
  kernels::expectUnboxedCallingWithReturnWorks(func);
}

```
- EN: Focus symbols: `TEST`, `expectOutOfPlaceBoxedCallingWorks`, `expectOutOfPlaceMultiBoxedCallingWorks`, `expectUnboxedCallingWithReturnWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `expectOutOfPlaceBoxedCallingWorks`, `expectOutOfPlaceMultiBoxedCallingWorks`, `expectUnboxedCallingWithReturnWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 414-430
```cpp
TEST(KernelFunctionTest, givenBoxedFunction_withoutReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_without_return>();
  kernels::expectUnboxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenBoxedFunction_withMultiReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_with_multi_return>();
  kernels::expectUnboxedCallingWithMultiReturnWorks(func);
}

// in/out, unboxed calling

TEST(KernelFunctionTest, givenBoxedFunction_withInPlaceSignature_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_for_inplace_op>();
  kernels::expectInPlaceUnboxedCallingWorks(func);
}

```
- EN: Focus symbols: `TEST`, `expectUnboxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithMultiReturnWorks`, `expectInPlaceUnboxedCallingWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `expectUnboxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithMultiReturnWorks`, `expectInPlaceUnboxedCallingWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 431-447
```cpp
TEST(KernelFunctionTest, givenBoxedFunction_withOutOfPlaceSignature_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_for_outofplace_op>();
  kernels::expectOutOfPlaceUnboxedCallingWorks(func);
}

TEST(KernelFunctionTest, givenBoxedFunction_withOutOfPlaceMultiSignature_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromBoxedFunction<&kernels::boxed_func_for_outofplace_multi_op>();
  kernels::expectOutOfPlaceMultiUnboxedCallingWorks(func);
}

// functors etc.

TEST(KernelFunctionTest, givenUnboxedFunctor_withReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunctor<false, kernels::unboxed_functor_with_return>(std::unique_ptr<OperatorKernel>(std::make_unique<kernels::unboxed_functor_with_return>()));
  kernels::expectBoxedCallingWithReturnWorks(func);
}

```
- EN: Focus symbols: `TEST`, `expectOutOfPlaceUnboxedCallingWorks`, `expectOutOfPlaceMultiUnboxedCallingWorks`, `expectBoxedCallingWithReturnWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `expectOutOfPlaceUnboxedCallingWorks`, `expectOutOfPlaceMultiUnboxedCallingWorks`, `expectBoxedCallingWithReturnWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 448-467
```cpp
TEST(KernelFunctionTest, givenUnboxedFunctor_withoutReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunctor<false, kernels::unboxed_functor_without_return>(std::unique_ptr<OperatorKernel>(std::make_unique<kernels::unboxed_functor_without_return>()));
  kernels::expectBoxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedFunctor_withReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunctor<false, kernels::unboxed_functor_with_return>(std::unique_ptr<OperatorKernel>(std::make_unique<kernels::unboxed_functor_with_return>()));
  kernels::expectUnboxedCallingWithReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedFunctor_withoutReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunctor<false, kernels::unboxed_functor_without_return>(std::unique_ptr<OperatorKernel>(std::make_unique<kernels::unboxed_functor_without_return>()));
  kernels::expectUnboxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedFunction_withReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunction(TORCH_FN(kernels::unboxed_function_with_return));
  kernels::expectBoxedCallingWithReturnWorks(func);
}

```
- EN: Focus symbols: `TEST`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`, `expectUnboxedCallingWithoutReturnWorks`, `makeFromUnboxedFunction`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`, `expectUnboxedCallingWithoutReturnWorks`, `makeFromUnboxedFunction`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 468-487
```cpp
TEST(KernelFunctionTest, givenUnboxedFunction_withoutReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunction(TORCH_FN(kernels::unboxed_function_without_return));
  kernels::expectBoxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedFunction_withReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunction(TORCH_FN(kernels::unboxed_function_with_return));
  kernels::expectUnboxedCallingWithReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedFunction_withoutReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedFunction(TORCH_FN(kernels::unboxed_function_without_return));
  kernels::expectUnboxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedRuntimeFunction_withReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedRuntimeFunction(&kernels::unboxed_function_with_return);
  kernels::expectBoxedCallingWithReturnWorks(func);
}

```
- EN: Focus symbols: `TEST`, `makeFromUnboxedFunction`, `TORCH_FN`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `makeFromUnboxedFunction`, `TORCH_FN`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 488-507
```cpp
TEST(KernelFunctionTest, givenUnboxedRuntimeFunction_withoutReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedRuntimeFunction(&kernels::unboxed_function_without_return);
  kernels::expectBoxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedRuntimeFunction_withReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedRuntimeFunction(&kernels::unboxed_function_with_return);
  kernels::expectUnboxedCallingWithReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedRuntimeFunction_withoutReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedRuntimeFunction(&kernels::unboxed_function_without_return);
  kernels::expectUnboxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedLambda_withReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedLambda(kernels::unboxed_lambda_with_return);
  kernels::expectBoxedCallingWithReturnWorks(func);
}

```
- EN: Focus symbols: `TEST`, `makeFromUnboxedRuntimeFunction`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`, `expectUnboxedCallingWithoutReturnWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `makeFromUnboxedRuntimeFunction`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`, `expectUnboxedCallingWithoutReturnWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 508-524
```cpp
TEST(KernelFunctionTest, givenUnboxedLambda_withoutReturn_whenCallingBoxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedLambda(kernels::unboxed_lambda_without_return);
  kernels::expectBoxedCallingWithoutReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedLambda_withReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedLambda(kernels::unboxed_lambda_with_return);
  kernels::expectUnboxedCallingWithReturnWorks(func);
}

TEST(KernelFunctionTest, givenUnboxedLambda_withoutReturn_whenCallingUnboxed_thenWorks) {
  KernelFunction func = KernelFunction::makeFromUnboxedLambda(kernels::unboxed_lambda_without_return);
  kernels::expectUnboxedCallingWithoutReturnWorks(func);
}

}

```
- EN: Focus symbols: `TEST`, `makeFromUnboxedLambda`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`, `expectUnboxedCallingWithoutReturnWorks`. This block encodes test setup, expectations, or regression coverage. The block checks observable behavior so future changes do not silently break the contract.
- CN: 关注符号：`TEST`, `makeFromUnboxedLambda`, `expectBoxedCallingWithoutReturnWorks`, `expectUnboxedCallingWithReturnWorks`, `expectUnboxedCallingWithoutReturnWorks`。该代码块编码测试准备、预期结果或回归覆盖。该代码块检查可观察行为，防止后续修改悄悄破坏契约。

### Lines 525-525
```cpp
// TODO Also test different variants of calling unboxed with wrong signatures
```
- EN: This block documents the design intent, assumptions, or usage notes. The comments give maintainers the context needed to interpret the following code correctly.
- CN: 该代码块说明设计意图、前提假设或使用说明。这些注释为维护者提供理解后续代码所需的上下文。

## Key Concepts / 关键概念
- core ATen/C10 abstractions / ATen/C10 核心抽象
- Type-system design / 类型系统设计
- Backend/device dispatch / 后端/设备分发
- Behavioral regression tests / 行为回归测试
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/ATen.h`, `ATen/core/boxing/KernelFunction.h`, `ATen/core/boxing/impl/test_helpers.h`, `ATen/core/op_registration/op_registration.h`
- External/system includes / 外部或系统头: `gtest/gtest.h`
- Inferred semantic dependencies / 推断出的语义依赖: tensor abstractions / 张量抽象; namespace scoping / 命名空间作用域
