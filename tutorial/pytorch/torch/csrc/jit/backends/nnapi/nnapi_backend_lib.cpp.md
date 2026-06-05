# nnapi_backend_lib.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/backends/nnapi/nnapi_backend_lib.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements the extension points, compilation flow, and runtime helpers used by custom JIT backends. This specific file centers on `nnapi_backend_lib.cpp`.
- **Purpose (CN)**: 实现自定义 JIT 后端所需的扩展点、编译流程和运行时辅助逻辑。 该文件具体围绕 `nnapi_backend_lib.cpp` 展开。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

```cpp
#include <memory>

#include <ATen/nnapi/nnapi_bind.h>
#include <torch/csrc/jit/backends/backend.h>
#include <torch/csrc/jit/mobile/import.h>
#include <torch/csrc/jit/mobile/module.h>

namespace torch::jit {

// Implementation of Android NNAPI Backend delegate

// The Android Neural Networks API (NNAPI) is an Android C API designed
// for running computationally intensive operations for machine learning on
// Android devices. The API is available on all Android devices running
// Android 8.1 (API level 27) or higher.

// Implementation is reflective of caffe2/torch/backends/_nnapi/prepare.py
// NnapiModule.forward()
class NnapiBackend : public PyTorchBackendInterface {
 public:
```

- **EN:** It enters or references namespace scopes such as torch::jit, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** The block declares or refines core types including NnapiBackend.
- **CN:** 该代码块声明或细化了 NnapiBackend 等核心类型。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Concepts touched here: Module API / 模块 API, Mobile runtime / 移动端运行时, Backend integration / 后端集成, Alias analysis / 别名分析, Declared symbols / 声明的符号, Header composition / 头文件组织.
- **CN:** 这里涉及的概念包括：Module API / 模块 API, Mobile runtime / 移动端运行时, Backend integration / 后端集成, Alias analysis / 别名分析, Declared symbols / 声明的符号, Header composition / 头文件组织。

### Lines 21-40 / 第 21-40 行

```cpp
  // Constructor.
  explicit NnapiBackend() = default;
  ~NnapiBackend() override = default;

  bool is_available() override {
    return true;
  }

  c10::impl::GenericDict compile(
      c10::IValue processed,
      c10::impl::GenericDict method_compile_spec) override {
    // Wrap processed in dictionary: {"forward": processed}
    auto dict = processed.toGenericDict();
    c10::Dict<c10::IValue, c10::IValue> handles(
        c10::StringType::get(), c10::AnyType::get());
    handles.insert("forward", dict);
    return c10::impl::toGenericDict(handles);
  }

  c10::impl::GenericList execute(
```

- **EN:** Important callable entry points in this range include is_available, compile, handles, toGenericDict.
- **CN:** 这一段的重要可调用入口包括 is_available, compile, handles, toGenericDict。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Module API / 模块 API, Backend integration / 后端集成, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Module API / 模块 API, Backend integration / 后端集成, Result propagation / 结果传递。

### Lines 41-60 / 第 41-60 行

```cpp
      c10::IValue handle,
      c10::impl::GenericList inputs) override {
    // Convert inputs to Tensors
    c10::List<at::Tensor> tensorInp;
    for (c10::IValue element : inputs) {
      tensorInp.push_back(element.toTensor());
    }

    // Lazily call init()
    if (comp_ == nullptr) {
      init(handle, tensorInp);
    }
    TORCH_CHECK(comp_ != nullptr)

    c10::List<at::Tensor> outputs;
    for (at::Tensor out : out_templates_) {
      outputs.push_back(at::empty_like(out));
    }

    // Adjust input memory formats
```

- **EN:** Important callable entry points in this range include init.
- **CN:** 这一段的重要可调用入口包括 init。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 61-80 / 第 61-80 行

```cpp
    auto dict = handle.toGenericDict();
    auto inp_mem_fmts = dict.at("inp_mem_fmts").toIntList();
    TORCH_CHECK(tensorInp.size() == inp_mem_fmts.size());
    std::vector<at::Tensor> fixed_inputs;
    for (auto i = 0U; i < tensorInp.size(); i++) {
      int fmt = inp_mem_fmts[i];
      // These constants match the values in DimOrder in serializer.py
      // 0: NCHW, 1: NHWC
      // TODO: See if it's possible to use those directly.
      if (fmt == 0) {
        fixed_inputs.push_back(tensorInp.get(i).contiguous());
      } else if (fmt == 1) {
        fixed_inputs.push_back(
            tensorInp.get(i).permute({0, 2, 3, 1}).contiguous());
      } else {
        TORCH_CHECK(false, "Invalid mem_fmt");
      }
    }

    comp_->run(fixed_inputs, outputs.vec());
```

- **EN:** Important callable entry points in this range include TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 81-100 / 第 81-100 行

```cpp

    // Adjust output memory formats
    auto out_mem_fmts = dict.at("out_mem_fmts").toIntList();
    TORCH_CHECK(outputs.size() == out_mem_fmts.size());
    for (auto i = 0U; i < outputs.size(); i++) {
      int fmt = out_mem_fmts[i];
      // These constants match the values in DimOrder in serializer.py
      // 0: NCHW, 1: NHWC
      // TODO: See if it's possible to use those directly.
      if (fmt == 1) {
        outputs.set(i, outputs.get(i).permute({0, 3, 1, 2}));
      } else {
        TORCH_CHECK(fmt == 0, "Invalid mem_fmt");
      }
    }

    return c10::impl::toList(outputs);
  }

 private:
```

- **EN:** Important callable entry points in this range include TORCH_CHECK, toList.
- **CN:** 这一段的重要可调用入口包括 TORCH_CHECK, toList。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 101-120 / 第 101-120 行

```cpp
  // The following variables are modified by init() during execution,
  // and cannot be passed through the handles dictionary
  std::unique_ptr<torch::nnapi::bind::NnapiCompilation> comp_;
  c10::List<at::Tensor> out_templates_;

  // Runs once per model initialization
  // Cannot be moved to compile(), because init() requires actual inputs
  void init(const c10::IValue& handle, const c10::List<at::Tensor>& inputs) {
    TORCH_CHECK(comp_ == nullptr);
    auto dict = handle.toGenericDict();

    // Get ser_model
    auto ser_model = dict.at("ser_model").toTensor();
    // Load shape computation module
    std::stringstream ss;
    auto shape_ptr = dict.at("shape_compute_module").toString();
    ss.str(*shape_ptr);
    auto shape_compute_module = _load_for_mobile(ss);
    out_templates_ =
        shape_compute_module.run_method("prepare", ser_model, inputs)
```

- **EN:** Important callable entry points in this range include init, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 init, TORCH_CHECK。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Module API / 模块 API, Mobile runtime / 移动端运行时, Backend integration / 后端集成, Optimization pass / 优化 pass, Shape/resource guard / 形状或资源保护.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Module API / 模块 API, Mobile runtime / 移动端运行时, Backend integration / 后端集成, Optimization pass / 优化 pass, Shape/resource guard / 形状或资源保护。

### Lines 121-135 / 第 121-135 行

```cpp
            .toTensorList();

    // Create and initialize NnapiCompilation object
    comp_ = std::make_unique<torch::nnapi::bind::NnapiCompilation>();
    auto weights = dict.at("weights").toTensorVector();
    comp_->init(ser_model, weights);
  }
};

namespace {
constexpr auto backend_name = "nnapi";
static auto cls = torch::jit::backend<NnapiBackend>(backend_name);
} // namespace

} // namespace torch::jit
```

- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Concepts touched here: Module API / 模块 API, Backend integration / 后端集成, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Module API / 模块 API, Backend integration / 后端集成, Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **Custom backend pipeline** — 自定义后端流水线
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Serialization** — 序列化
- **Mobile runtime** — 移动端运行时
- **Backend integration** — 后端集成
- **Optimization pass** — 优化 pass
- **Alias analysis** — 别名分析

## Dependencies / 依赖关系

- `ATen/nnapi/nnapi_bind.h`
- `torch/csrc/jit/backends/backend.h`
- `torch/csrc/jit/mobile/import.h`
- `torch/csrc/jit/mobile/module.h`
