# constants.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/ir/constants.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines or manipulates the TorchScript/JIT intermediate representation, including graphs, nodes, blocks, and analysis utilities. This specific file centers on `constants.cpp`.
- **Purpose (CN)**: 定义或操作 TorchScript/JIT 中间表示，包括图、节点、块及其分析工具。 该文件具体围绕 `constants.cpp` 展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <c10/util/Exception.h>
#include <torch/csrc/jit/ir/constants.h>
#include <torch/csrc/jit/ir/ir.h>
#include <torch/csrc/jit/runtime/operator.h>

namespace torch::jit {

static bool insertableTensor(const at::Tensor& ten) {
  // bail if tensor has no storage i.e. opaque tensor used in MKLdnn.
  // or gradients because we have no way of serializing them & are mutable
  return !ten.requires_grad() && ten.has_storage() && !ten.is_nested();
}

static bool insertableIValue(const IValue& ivalue) {
  if (ivalue.isInt() || ivalue.isNone() || ivalue.isBool() ||
      ivalue.isDouble() || ivalue.isComplexDouble() || ivalue.isString() ||
      ivalue.isDevice() || ivalue.isEnum()) {
    return true;
  }
  if (ivalue.isTensor()) {
    return insertableTensor(ivalue.toTensor());
  }
  if (ivalue.isList() || ivalue.isTuple()) {
    c10::ArrayRef<IValue> elems;
    if (ivalue.isTuple()) {
      elems = ivalue.toTupleRef().elements();
    } else {
      elems = ivalue.toListRef();
```

- **EN:** It enters or references namespace scopes such as torch::jit, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include insertableTensor, insertableIValue.
- **CN:** 这一段的重要可调用入口包括 insertableTensor, insertableIValue。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。

### Lines 29-56 / 第 29-56 行

```cpp
    }
    return std::all_of(elems.begin(), elems.end(), [](const IValue& tup_elem) {
      return insertableIValue(tup_elem);
    });
  }
  if (ivalue.isGenericDict()) {
    const auto& dict = ivalue.toGenericDict();
    return std::all_of(dict.begin(), dict.end(), [](const auto& entry) {
      return insertableIValue(entry.key()) && insertableIValue(entry.value());
    });
  }

  return false;
}

Value* insertConstant(
    Graph& g,
    const IValue& val,
    std::optional<SourceRange> loc,
    std::optional<ScopePtr> scope) {
  auto value = tryInsertConstant(g, val, std::move(loc), std::move(scope));
  if (value) {
    return *value;
  }
  TORCH_CHECK(false, "Unsupported value kind: ", val.tagKind());
}

// IValue -> Constant node
```

- **EN:** Important callable entry points in this range include all_of, insertableIValue, insertConstant, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 all_of, insertableIValue, insertConstant, TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 57-84 / 第 57-84 行

```cpp
std::optional<Value*> tryInsertConstant(
    Graph& g,
    const IValue& val,
    std::optional<SourceRange> loc,
    std::optional<ScopePtr> scope) {
  Node* n = g.create(prim::Constant);
  if (val.isTensor()) {
    at::Tensor ref = val.toTensor();
    if (!insertableTensor(val.toTensor())) {
      n->destroy();
      return std::nullopt;
    }
    if (!ref.defined()) {
      n->destroy();
      return g.insertNode(g.createNone())->output();
    }
    TORCH_INTERNAL_ASSERT(!ref.requires_grad());
    n->output()->inferTypeFrom(
        ref); // note: before t_ because of std::move(ref)
    n->t_(attr::value, std::move(ref));
  } else if (val.isInt()) {
    n->i_(attr::value, val.toInt());
    n->output()->setType(IntType::get());
  } else if (val.isDouble()) {
    n->f_(attr::value, val.toDouble());
    n->output()->setType(FloatType::get());
  } else if (val.isComplexDouble()) {
    n->c_(attr::value, val.toComplexDouble());
```

- **EN:** Important callable entry points in this range include tryInsertConstant, TORCH_INTERNAL_ASSERT.
- **CN:** 这一段的重要可调用入口包括 tryInsertConstant, TORCH_INTERNAL_ASSERT。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 85-112 / 第 85-112 行

```cpp
    n->output()->setType(ComplexType::get());
  } else if (val.isBool()) {
    n->i_(attr::value, val.toBool());
    n->output()->setType(BoolType::get());
  } else if (val.isList()) {
    bool fast_path_list =
        val.isBoolList() || val.isIntList() || val.isDoubleList();
    if (fast_path_list || insertableIValue(val)) {
      n->ival_(attr::value, val);
      n->output()->setType(val.type());
    } else {
      n->destroy();
      return std::nullopt;
    }
  } else if (val.isString()) {
    n->s_(attr::value, val.toStringRef());
    n->output()->setType(StringType::get());
  } else if (val.isDevice()) {
    std::stringstream ss;
    ss << val.toDevice();
    n->s_(attr::value, ss.str());
    n->output()->setType(DeviceObjType::get());
  } else if (val.isGenerator()) {
    auto generator = val.toGenerator();
    n->ival_(attr::value, generator);
    n->output()->setType(GeneratorType::get());
  } else if (val.isStream()) {
    // packing into int64_t removed
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 113-140 / 第 113-140 行

```cpp
    n->ival_(attr::value, val);
    n->output()->setType(StreamObjType::get());
  } else if (val.isNone()) {
    n->output()->setType(NoneType::get());
  } else if (val.isTuple()) {
    if (insertableIValue(val)) {
      n->ival_(attr::value, val);
      n->output()->setType(val.type());
    } else {
      n->destroy();
      return std::nullopt;
    };
  } else if (val.isObject()) {
    const auto& ref = val.toObjectRef();
    // see: [Constant Object Weak CompilationUnit Reference]
    if (!ref.type()->is_module() &&
        (ref.is_weak_compilation_ref() ||
         ref.is_empty_strong_compilation_ref())) {
      n->ival_(attr::value, val);
      n->output()->setType(val.type());
    } else {
      n->destroy();
      return std::nullopt;
    }
  } else if ((val.isGenericDict() && insertableIValue(val)) || (val.isEnum())) {
    n->ival_(attr::value, val);
    n->output()->setType(val.type());
  } else {
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 141-168 / 第 141-168 行

```cpp
    n->destroy();
    return std::nullopt;
  }
  if (loc)
    n->setSourceRange(*loc);
  if (scope)
    n->setScope(*scope);
  return g.insertNode(n)->output();
}

std::optional<IValue> toIValue(const Value* v) {
  if (v->node()->kind() != prim::Constant || v->type()->cast<FunctionType>()) {
    return std::nullopt;
  }
  const Node* node = v->node();
  const TypePtr& type = v->type();
  if (type->isSubtypeOf(*TensorType::get())) {
    return node->t(attr::value);
  } else if (type->isSubtypeOf(*BoolType::get())) {
    return (bool)node->i(attr::value);
  } else if (
      type->isSubtypeOf(*NumberType::get()) &&
      node->kindOf(attr::value) == AttributeKind::i) {
    return node->i(attr::value);
  } else if (
      type->isSubtypeOf(*NumberType::get()) &&
      node->kindOf(attr::value) == AttributeKind::f) {
    return node->f(attr::value);
```

- **EN:** Important callable entry points in this range include toIValue.
- **CN:** 这一段的重要可调用入口包括 toIValue。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 169-196 / 第 169-196 行

```cpp
  } else if (
      type->isSubtypeOf(*NumberType::get()) &&
      node->kindOf(attr::value) == AttributeKind::c) {
    return node->c(attr::value);
  } else if (
      type->cast<ListType>() &&
      node->kindOf(attr::value) == AttributeKind::ival) {
    const auto& list = node->ival(attr::value);
    TORCH_INTERNAL_ASSERT(list.isList());
    return list;
  } else if (
      type->cast<DictType>() &&
      node->kindOf(attr::value) == AttributeKind::ival) {
    const auto& dict = node->ival(attr::value);
    TORCH_INTERNAL_ASSERT(dict.isGenericDict());
    return dict;
  } else if (
      type->cast<TupleType>() &&
      node->kindOf(attr::value) == AttributeKind::ival) {
    const auto& tup = node->ival(attr::value);
    TORCH_INTERNAL_ASSERT(tup.isTuple());
    return tup;
  } else if (type == StringType::get()) {
    const auto& s = node->s(attr::value);
    return s;
  } else if (type == DeviceObjType::get()) {
    auto d = c10::Device(node->s(attr::value));
    return d;
```

- **EN:** Important callable entry points in this range include TORCH_INTERNAL_ASSERT.
- **CN:** 这一段的重要可调用入口包括 TORCH_INTERNAL_ASSERT。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 197-217 / 第 197-217 行

```cpp
  } else if (type == GeneratorType::get()) {
    auto generator = node->ival(attr::value).toGenerator();
    return generator;
  } else if (type == StreamObjType::get()) {
    // int64_t packing removed
    auto s = node->ival(attr::value).toStream();
    return s;
  } else if (node->mustBeNone()) {
    return IValue();
  } else if (type->cast<EnumType>()) {
    const auto& enum_val = node->ival(attr::value);
    return enum_val;
  } else if (type->cast<ClassType>() && !type->is_module()) {
    const auto& class_val = node->ival(attr::value);
    return class_val;
  } else {
    TORCH_CHECK(false, "constant literal not supported for: ", type->str());
  }
}

} // namespace torch::jit
```

- **EN:** Important callable entry points in this range include IValue, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 IValue, TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

## Key Concepts / 关键概念

- **TorchScript IR internals** — TorchScript IR 内部结构
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Mobile runtime** — 移动端运行时
- **Registration** — 注册机制
- **Core symbols: insertableTensor, insertableIValue, all_of, insertConstant, TORCH_CHECK, tryInsertConstant, TORCH_INTERNAL_ASSERT, toIValue** — 核心符号：insertableTensor、insertableIValue、all_of、insertConstant、TORCH_CHECK、tryInsertConstant、TORCH_INTERNAL_ASSERT、toIValue

## Dependencies / 依赖关系

- `c10/util/Exception.h`
- `torch/csrc/jit/ir/constants.h`
- `torch/csrc/jit/ir/ir.h`
- `torch/csrc/jit/runtime/operator.h`
