# promoted_prim_ops.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/mobile/promoted_prim_ops.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements loading, execution, optimization, or serialization support for the mobile / lite interpreter runtime. This specific file centers on `promoted_prim_ops.cpp`. It is tailored for lite interpreter or mobile deployment constraints.
- **Purpose (CN)**: 实现移动端 / 轻量解释器运行时的加载、执行、优化或序列化支持。 该文件具体围绕 `promoted_prim_ops.cpp` 展开。 其设计面向轻量解释器或移动端部署约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <ATen/ScalarOps.h>
#include <fmt/format.h>
#include <torch/csrc/jit/mobile/promoted_prim_ops.h>

namespace torch::jit {

void tupleIndex(Stack& stack) {
  int64_t index = pop(stack).toInt();
  auto tuple = pop(stack).toTuple();
  auto norm_index =
      normalizeIndex(index, static_cast<int64_t>(tuple->elements().size()));
  if (norm_index < 0 ||
      norm_index >= static_cast<int64_t>(tuple->elements().size())) {
    throw std::out_of_range("Tuple list index out of range");
  }
  stack.emplace_back(tuple->elements()[norm_index]);
}

void raiseException(Stack& stack) {
  // this kernel supports RaiseException with only one argument: the error
  // DEPRECATED from bytecode_version 8;
  // Please do not make any changes to this to support BC
  throw JITException(pop(stack).toStringRef());
}

void raiseExceptionWithMessage(Stack& stack) {
  // this kernel supports RaiseException with only two arguments: the error and
  // the message Please make changes only to this kernel
```

- **EN:** It enters or references namespace scopes such as torch::jit, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include tupleIndex, normalizeIndex, out_of_range, raiseException, JITException, raiseExceptionWithMessage.
- **CN:** 这一段的重要可调用入口包括 tupleIndex, normalizeIndex, out_of_range, raiseException, JITException, raiseExceptionWithMessage。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 29-56 / 第 29-56 行

```cpp
  std::optional<std::string> qualified_class_name =
      pop(stack).toOptional<std::string>();
  std::string message;
  pop(stack, message);

  throw JITException(message, qualified_class_name);
}

void is(Stack& stack) {
  IValue self, obj;
  pop(stack, self, obj);
  push(stack, self.is(obj));
}

void unInitialized(Stack& stack) {
  push(stack, IValue::uninitialized());
}

void isNot(Stack& stack) {
  IValue self, obj;
  pop(stack, self, obj);
  push(stack, !self.is(obj));
}

void aten_format(Stack& stack) {
  size_t num_inputs = pop(stack).toInt();
  format(stack, num_inputs);
}
```

- **EN:** Important callable entry points in this range include pop, JITException, is, push, unInitialized, isNot.
- **CN:** 这一段的重要可调用入口包括 pop, JITException, is, push, unInitialized, isNot。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Declared symbols / 声明的符号.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Declared symbols / 声明的符号。

### Lines 57-84 / 第 57-84 行

```cpp

void size(Stack& stack) {
  auto t = std::move(pop(stack)).toTensor();
  pack(stack, t.sizes().vec());
}

void sym_size(Stack& stack) {
  auto t = std::move(pop(stack)).toTensor();
  pack(stack, t.sym_sizes().vec());
}
void sym_size_int(Stack& stack) {
  auto dim = pop(stack).toInt();
  auto t = pop(stack).toTensor();
  push(stack, t.sym_sizes()[dim]);
}
void sym_stride_int(Stack& stack) {
  auto dim = pop(stack).toInt();
  auto t = pop(stack).toTensor();
  push(stack, t.sym_strides()[dim]);
}

void sym_numel(Stack& stack) {
  auto t = std::move(pop(stack)).toTensor();
  push(stack, t.sym_numel());
}

void sym_storage_offset(Stack& stack) {
  auto t = std::move(pop(stack)).toTensor();
```

- **EN:** Important callable entry points in this range include size, pack, sym_size, sym_size_int, push, sym_stride_int.
- **CN:** 这一段的重要可调用入口包括 size, pack, sym_size, sym_size_int, push, sym_stride_int。

### Lines 85-112 / 第 85-112 行

```cpp
  push(stack, t.sym_storage_offset());
}

void sym_stride(Stack& stack) {
  auto t = std::move(pop(stack)).toTensor();
  pack(stack, t.sym_strides().vec());
}

void device(Stack& stack) {
  push(stack, pop(stack).toTensor().device());
}

void device_with_index(Stack& stack) {
  std::string type = pop(stack).toStringRef();
  auto index = pop(stack).toInt();
  std::string device_str = fmt::format("{}:{}", type, index);
  auto device = c10::Device(device_str);
  push(stack, device);
}

void dtype(Stack& stack) {
  at::Tensor a;
  pop(stack, a);
  push(stack, static_cast<int64_t>(a.scalar_type()));
}

void layout(Stack& stack) {
  push(stack, pop(stack).toTensor().layout());
```

- **EN:** Important callable entry points in this range include push, sym_stride, pack, device, device_with_index, dtype.
- **CN:** 这一段的重要可调用入口包括 push, sym_stride, pack, device, device_with_index, dtype。
- **EN:** Concepts touched here: Type system / 类型系统.
- **CN:** 这里涉及的概念包括：Type system / 类型系统。

### Lines 113-140 / 第 113-140 行

```cpp
}

void toPrimDType(Stack& stack) {
  bool non_blocking = false;
  bool copy = false;
  pop(stack, non_blocking, copy);
  std::optional<at::ScalarType> scalarType =
      pop(stack).toOptional<at::ScalarType>();
  std::optional<c10::Device> device = std::nullopt;
  at::Tensor self = pop(stack).toTensor();
  push(stack, to_dispatch(self, device, scalarType, non_blocking, copy));
}

void dim(Stack& stack) {
  at::Tensor arg = pop(stack).toTensor();
  push(stack, arg.dim());
}

void _not(Stack& stack) {
  push(stack, !pop(stack).toBool());
}

void boolTensor(Stack& stack) {
  at::Tensor a;
  pop(stack, a);
  push(stack, at::native::is_nonzero(a));
}

```

- **EN:** Important callable entry points in this range include toPrimDType, pop, push, dim, _not, boolTensor.
- **CN:** 这一段的重要可调用入口包括 toPrimDType, pop, push, dim, _not, boolTensor。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Control-flow blocks / 控制流块, Type system / 类型系统.
- **CN:** 这里涉及的概念包括：Control-flow blocks / 控制流块, Type system / 类型系统。

### Lines 141-168 / 第 141-168 行

```cpp
void toList(Stack& stack) {
  int elem_ty_val = 0;
  int dim_val = 0;
  at::Tensor t;

  pop(stack, elem_ty_val);
  pop(stack, dim_val);
  pop(stack, t);

  // If the Tensor is not on the CPU, transfer it.
  if (!t.device().is_cpu()) {
    t = t.cpu();
  }

  // Rebuild the output type using elem_ty_val and dim_val. Start
  // with the element type corresponding to elem_ty_val.
  at::TypePtr out_ty;
  if (elem_ty_val == 0) {
    out_ty = at::IntType::get();
  } else if (elem_ty_val == 1) {
    out_ty = at::FloatType::get();
  } else if (elem_ty_val == 2) {
    out_ty = at::BoolType::get();
  } else if (elem_ty_val == 3) {
    out_ty = at::ComplexType::get();
  } else {
    TORCH_CHECK(
        false,
```

- **EN:** Important callable entry points in this range include toList, pop.
- **CN:** 这一段的重要可调用入口包括 toList, pop。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Type system / 类型系统, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Type system / 类型系统, Branching logic / 分支逻辑。

### Lines 169-196 / 第 169-196 行

```cpp
        "Unsupported element type for tolist; only int, float, complex and bool are supported");
  }

  // Check that type of the Tensor matches that of the annotation.
  // Make an exception for the case in which the annotated type is
  // float/complex and the Tensor data type is also float/complex;
  // the elements will be casted to double/c10::complex<double>
  // later.
  TORCH_CHECK(
      (out_ty == at::FloatType::get() && t.is_floating_point()) ||
          (out_ty == at::ComplexType::get() && t.is_complex()) ||
          tryScalarTypeFromJitType(*out_ty) == t.scalar_type(),
      "Output annotation element type and runtime tensor element type must match for tolist(): ",
      *tryScalarTypeFromJitType(*out_ty),
      " vs ",
      t.scalar_type());

  // Check that the dimension of the Tensor matches that of the
  // annotation.
  TORCH_CHECK(
      dim_val == t.dim(),
      "Output annotation list dimension and runtime tensor dimension must match for tolist()");

  // Wrap out_ty in a ListType dim times.
  for ([[maybe_unused]] const auto i : c10::irange(dim_val)) {
    out_ty = at::ListType::create(out_ty);
  }

```

- **EN:** Important callable entry points in this range include TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Declared symbols / 声明的符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Declared symbols / 声明的符号, Iteration / 迭代处理。

### Lines 197-224 / 第 197-224 行

```cpp
  int64_t dim = t.dim();
  auto sizes = t.sizes();
  auto strides = t.strides();
  size_t element_size = t.element_size();
  char* data = static_cast<char*>(t.data_ptr());
  auto result = tensorToListRecursive(
      data, 0, dim, out_ty, t.scalar_type(), sizes, strides, element_size);
  push(stack, std::move(result));
}

void numToTensorScalar(Stack& stack) {
  at::Scalar s;
  pop(stack, s);
  push(stack, c10::scalar_to_tensor(s));
}

void isCuda(Stack& stack) {
  at::Tensor a;
  pop(stack, a);
  push(stack, a.is_cuda());
}

void numToTensorBool(Stack& stack) {
  bool b = false;
  pop(stack, b);
  push(stack, c10::scalar_to_tensor(b));
}

```

- **EN:** Important callable entry points in this range include push, numToTensorScalar, pop, isCuda, numToTensorBool.
- **CN:** 这一段的重要可调用入口包括 push, numToTensorScalar, pop, isCuda, numToTensorBool。
- **EN:** Concepts touched here: Type system / 类型系统.
- **CN:** 这里涉及的概念包括：Type system / 类型系统。

### Lines 225-252 / 第 225-252 行

```cpp
void dictIndex(Stack& stack) {
  auto key = pop(stack);
  auto dict = pop(stack).toGenericDict();
  auto value = dict.find(key);
  if (value == dict.end()) {
    TORCH_CHECK(false, "KeyError: ", key);
  }
  push(stack, value->value());
}

[[maybe_unused]] static const std::array<mobile::prim_op_fn_register, 16>
    op_reg = {
        mobile::prim_op_fn_register("prim::TupleIndex", tupleIndex),
        mobile::prim_op_fn_register("aten::Bool.Tensor", boolTensor),
        mobile::prim_op_fn_register("aten::format", aten_format),
        mobile::prim_op_fn_register(
            "prim::NumToTensor.Scalar",
            numToTensorScalar),
        mobile::prim_op_fn_register(
            "prim::RaiseException",
            raiseExceptionWithMessage),
        mobile::prim_op_fn_register("prim::device", device),
        mobile::prim_op_fn_register("prim::dtype", dtype),
        mobile::prim_op_fn_register("prim::layout", layout),
        mobile::prim_op_fn_register("aten::__not__", _not),
        mobile::prim_op_fn_register("aten::__is__", is),
        mobile::prim_op_fn_register("aten::__isnot__", isNot),
        mobile::prim_op_fn_register("aten::dim", dim),
```

- **EN:** Important callable entry points in this range include dictIndex, TORCH_CHECK, push.
- **CN:** 这一段的重要可调用入口包括 dictIndex, TORCH_CHECK, push。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。

### Lines 253-262 / 第 253-262 行

```cpp
        mobile::prim_op_fn_register("prim::Uninitialized", unInitialized),
        mobile::prim_op_fn_register("prim::is_cuda", isCuda),
        mobile::prim_op_fn_register("aten::__getitem__.Dict_str", dictIndex),
        mobile::prim_op_fn_register("prim::unchecked_cast", noop),
        // TODO: (@pavithran) size is overloaded with int[] and Tensor
        // so this throws error expecting int not Tensor
        // mobile::prim_op_fn_register("aten::size", size)
};

} // namespace torch::jit
```

- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Concepts touched here: Mobile runtime / 移动端运行时, Registration / 注册机制, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Mobile runtime / 移动端运行时, Registration / 注册机制, Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **Lite interpreter runtime** — 轻量解释器运行时
- **Nodes and values** — 节点与值
- **Control-flow blocks** — 控制流块
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Mobile runtime** — 移动端运行时
- **Registration** — 注册机制
- **Core symbols: tupleIndex, normalizeIndex, out_of_range, raiseException, JITException, raiseExceptionWithMessage, pop, is** — 核心符号：tupleIndex、normalizeIndex、out_of_range、raiseException、JITException、raiseExceptionWithMessage、pop、is

## Dependencies / 依赖关系

- `ATen/ScalarOps.h`
- `torch/csrc/jit/mobile/promoted_prim_ops.h`
