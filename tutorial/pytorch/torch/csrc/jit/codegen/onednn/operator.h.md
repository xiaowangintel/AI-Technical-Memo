# operator.h — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/onednn/operator.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `operator.h`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `operator.h` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#pragma once

#include <oneapi/dnnl/dnnl_graph.hpp>
#include <torch/csrc/jit/codegen/onednn/LlgaTensorImpl.h>
#include <torch/csrc/jit/ir/ir.h>

namespace torch::jit::fuser::onednn {

class Operator {
 public:
  Operator(const Node* node, dnnl::graph::op::kind kind)
      : n(node), o(getId(node), kind, node->kind().toQualString()), k(kind) {}

  // Returns output index if the Value is a graph output.
  // Otherwise returns -1
  int32_t graphOutputIdx(Value* v) {
    int32_t i = 0;
    for (const Value* output : v->owningGraph()->outputs()) {
      if (v == output) {
        return i;
```

- **EN:** It enters or references namespace scopes such as torch::jit::fuser::onednn, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::onednn 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** The block declares or refines core types including Operator.
- **CN:** 该代码块声明或细化了 Operator 等核心类型。
- **EN:** Important callable entry points in this range include Operator, graphOutputIdx.
- **CN:** 这一段的重要可调用入口包括 Operator, graphOutputIdx。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 21-40 / 第 21-40 行

```cpp
      }
      i++;
    }
    return -1;
  }

  Operator& setInputValue(Value* v) {
    if (v->mustNotBeNone()) {
      if (v->type()->kind() == c10::TensorType::Kind) {
        o.add_input(createLogicalTensor(v));
      }
    }
    return *this;
  }

  Operator& setInput(size_t offset) {
    return setInputValue(n->input(offset));
  }

  template <typename... Ts>
```

- **EN:** Important callable entry points in this range include setInputValue, setInput.
- **CN:** 这一段的重要可调用入口包括 setInputValue, setInput。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 41-60 / 第 41-60 行

```cpp
  Operator& setInput(size_t offset, Ts... other) {
    setInput(offset);
    return setInput(other...);
  }

  Operator& setOutputValue(Value* v) {
    if (v->mustNotBeNone()) {
      o.add_output(createLogicalTensor(v));
    }
    return *this;
  }

  // setOutputValue & setOutput require a pointer to the LLGA graph, as output
  // logical tensors that are graph outputs should be connected to an End LLGA
  // op. A value of NULL can be provided for the graph pointer in order to
  // maintain the legacy functionality of this function.
  Operator& setOutputValue(Value* v, std::unique_ptr<dnnl::graph::graph>& g) {
    if (v->mustNotBeNone()) {
      auto output_tensor = createLogicalTensor(v);
      o.add_output(output_tensor);
```

- **EN:** Important callable entry points in this range include setInput, setOutputValue.
- **CN:** 这一段的重要可调用入口包括 setInput, setOutputValue。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 61-80 / 第 61-80 行

```cpp
      if (g) {
        int32_t outputIndex = graphOutputIdx(v);
        if (outputIndex != -1) {
          dnnl::graph::op newEndNode(
              LONG_MAX - outputIndex,
              dnnl::graph::op::kind::End,
              "EndNodeForGraphOutput");
          newEndNode.add_input(output_tensor);
          g->add_op(newEndNode);
        }
      }
    }
    return *this;
  }

  Operator& setOutput(std::unique_ptr<dnnl::graph::graph>& g, size_t offset) {
    return setOutputValue(n->output(offset), g);
  }

  Operator& setOutput(size_t offset) {
```

- **EN:** Important callable entry points in this range include newEndNode, setOutput, setOutputValue.
- **CN:** 这一段的重要可调用入口包括 newEndNode, setOutput, setOutputValue。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 81-100 / 第 81-100 行

```cpp
    return setOutputValue(n->output(offset));
  }

  template <typename... Ts>
  Operator& setOutput(
      std::unique_ptr<dnnl::graph::graph>& g,
      size_t offset,
      Ts... other) {
    setOutput(g, offset);
    return setOutput(g, other...);
  }

  template <typename Attr>
  Operator& setAttr(dnnl::graph::op::attr name, Attr&& attr) {
    o.set_attr(name, std::forward<Attr>(attr));
    return *this;
  }

  template <typename F>
  Operator& setAttr(dnnl::graph::op::attr name, const F& fn, size_t offset) {
```

- **EN:** Important callable entry points in this range include setOutputValue, setOutput, setAttr.
- **CN:** 这一段的重要可调用入口包括 setOutputValue, setOutput, setAttr。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 101-120 / 第 101-120 行

```cpp
    return setAttr(name, fn(n, offset));
  }

  static float ScalarToFloat(const Node* node, size_t offset) {
    return toIValue(node->input(offset))->toScalar().to<float>();
  }

  static std::vector<int64_t> Ints(const Node* node, size_t offset) {
    return toIValue(node->input(offset))->toIntVector();
  }

  static int64_t Int(const Node* node, size_t offset) {
    return toIValue(node->input(offset))->toInt();
  }

  static float Float(const Node* node, size_t offset) {
    return static_cast<float>(toIValue(node->input(offset))->toDouble());
  }

  static bool Bool(const Node* node, size_t offset) {
```

- **EN:** Important callable entry points in this range include setAttr, ScalarToFloat, toIValue, Ints, Int, Float.
- **CN:** 这一段的重要可调用入口包括 setAttr, ScalarToFloat, toIValue, Ints, Int, Float。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Operator schema / 算子模式, Declared symbols / 声明的符号, Result propagation / 结果传递。

### Lines 121-140 / 第 121-140 行

```cpp
    return toIValue(node->input(offset))->toBool();
  }

  static uint64_t getId(const Node* node) {
    return reinterpret_cast<uint64_t>(node); // cast node address as op id
  }

  dnnl::graph::op::kind kind() const {
    return k;
  }

  dnnl::graph::op llgaOp() const {
    return o;
  }

 private:
  dnnl::graph::logical_tensor createLogicalTensor(Value* value) const {
    return LlgaTensorDesc(value).logical_tensor();
  }

```

- **EN:** Important callable entry points in this range include toIValue, getId, kind, llgaOp, createLogicalTensor, LlgaTensorDesc.
- **CN:** 这一段的重要可调用入口包括 toIValue, getId, kind, llgaOp, createLogicalTensor, LlgaTensorDesc。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Operator schema / 算子模式, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Operator schema / 算子模式, Result propagation / 结果传递。

### Lines 141-146 / 第 141-146 行

```cpp
  const Node* n;
  dnnl::graph::op o;
  dnnl::graph::op::kind k;
};

} // namespace torch::jit::fuser::onednn
```

- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Graph IR / 图中间表示, Nodes and values / 节点与值, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Graph IR / 图中间表示, Nodes and values / 节点与值, Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Graph IR** — 图中间表示
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Code generation** — 代码生成
- **Registration** — 注册机制
- **Core symbols: Operator, graphOutputIdx, setInputValue, setInput, setOutputValue, newEndNode, setOutput, setAttr** — 核心符号：Operator、graphOutputIdx、setInputValue、setInput、setOutputValue、newEndNode、setOutput、setAttr

## Dependencies / 依赖关系

- `torch/csrc/jit/codegen/onednn/LlgaTensorImpl.h`
- `torch/csrc/jit/ir/ir.h`
