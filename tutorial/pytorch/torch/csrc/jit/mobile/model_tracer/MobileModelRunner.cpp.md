# MobileModelRunner.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/mobile/model_tracer/MobileModelRunner.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements loading, execution, optimization, or serialization support for the mobile / lite interpreter runtime. This specific file centers on `MobileModelRunner.cpp`. It is tailored for lite interpreter or mobile deployment constraints.
- **Purpose (CN)**: 实现移动端 / 轻量解释器运行时的加载、执行、优化或序列化支持。 该文件具体围绕 `MobileModelRunner.cpp` 展开。 其设计面向轻量解释器或移动端部署约束。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/mobile/model_tracer/MobileModelRunner.h>
#include <torch/csrc/jit/mobile/model_tracer/TensorUtils.h>

namespace torch::jit::mobile {

std::vector<std::vector<at::IValue>> MobileModelRunner::
    ivalue_to_bundled_inputs(const c10::IValue& bundled_inputs) {
  CAFFE_ENFORCE(
      bundled_inputs.isList(),
      "Expected get_all_bundled_inputs to ",
      "return a list but got a ",
      bundled_inputs.tagKind(),
      " instead");

  c10::List<at::IValue> all_inputs = bundled_inputs.toList();
  CAFFE_ENFORCE(
      !all_inputs.empty(),
      "Expected at least 1 bundled input, ",
      "but found none. Please use ",
      "torch.utils.bundled_inputs.augment_model_with_bundled_inputs to add.");

  std::vector<std::vector<at::IValue>> ret;
  for (at::IValue input : all_inputs) {
    CAFFE_ENFORCE(
        input.isTuple(),
        "Expected list element to be a tuple ",
        "but got a ",
        input.tagKind(),
```

- **EN:** It enters or references namespace scopes such as torch::jit::mobile, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::mobile 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include ivalue_to_bundled_inputs, CAFFE_ENFORCE.
- **CN:** 这一段的重要可调用入口包括 ivalue_to_bundled_inputs, CAFFE_ENFORCE。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。

### Lines 29-56 / 第 29-56 行

```cpp
        " instead");
    ret.push_back(input.toTupleRef().elements());
  }

  return ret;
}

std::unordered_map<std::string, std::string> MobileModelRunner::
    ivalue_to_bundled_inputs_map(const c10::IValue& bundled_inputs) {
  CAFFE_ENFORCE(
      bundled_inputs.isGenericDict(),
      "Expected get_bundled_inputs_functions_and_info to ",
      "return a dict but got a ",
      bundled_inputs.tagKind(),
      " instead");

  c10::Dict<at::IValue, at::IValue> all_inputs = bundled_inputs.toGenericDict();
  CAFFE_ENFORCE(
      !all_inputs.empty(),
      "Expected at least 1 function with bundled inputs, ",
      "but found none. Please use ",
      "torch.utils.bundled_inputs.augment_model_with_bundled_inputs to add.");

  std::unordered_map<std::string, std::string> ret;
  for (auto& input : all_inputs) {
    const at::IValue& function_name = input.key();
    const at::IValue& nested_dict = input.value();
    CAFFE_ENFORCE(
```

- **EN:** Important callable entry points in this range include ivalue_to_bundled_inputs_map, CAFFE_ENFORCE.
- **CN:** 这一段的重要可调用入口包括 ivalue_to_bundled_inputs_map, CAFFE_ENFORCE。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 57-84 / 第 57-84 行

```cpp
        function_name.isString(),
        "Expected function with inputs to be a string ",
        "but got a ",
        function_name.tagKind(),
        " instead");
    CAFFE_ENFORCE(
        nested_dict.isGenericDict(),
        "Expected function name to map to dictionary ",
        "but got a ",
        nested_dict.tagKind(),
        " instead");

    // Got the nested dict now need to convert that into std types
    c10::Dict<at::IValue, at::IValue> function_and_info_ival_dict =
        nested_dict.toGenericDict();
    std::unordered_map<std::string, std::vector<std::string>>
        function_and_info_dict;
    for (auto& entry : function_and_info_ival_dict) {
      const at::IValue& key = entry.key();
      const at::IValue& value = entry.value();
      CAFFE_ENFORCE(
          key.isString(),
          "Expected extra information key to be a string ",
          "but got a ",
          value.tagKind(),
          " instead");
      CAFFE_ENFORCE(
          value.isList(),
```

- **EN:** Important callable entry points in this range include CAFFE_ENFORCE.
- **CN:** 这一段的重要可调用入口包括 CAFFE_ENFORCE。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Declared symbols / 声明的符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Declared symbols / 声明的符号, Iteration / 迭代处理。

### Lines 85-112 / 第 85-112 行

```cpp
          "Expected extra information values to be a list ",
          "but got a ",
          value.tagKind(),
          " instead");

      // Got the value of the nested dict entry now need to convert it to std
      // types
      std::vector<std::string> data_list;
      c10::List<at::IValue> ival_data = value.toList();
      for (at::IValue data : ival_data) {
        CAFFE_ENFORCE(
            data.isString(),
            "Expected list element of nested dict entries to be a string ",
            "but got a ",
            data.tagKind(),
            " instead");
        data_list.push_back(data.toStringRef());
      }

      // Add entry into std type mapping
      function_and_info_dict[key.toStringRef()] = data_list;
    }

    // Could store the full mapping of std types, but the 'info' section isn't
    // needed here
    std::string input_function =
        function_and_info_dict["get_inputs_function_name"][0];
    ret[function_name.toStringRef()] = input_function;
```

- **EN:** Important callable entry points in this range include CAFFE_ENFORCE.
- **CN:** 这一段的重要可调用入口包括 CAFFE_ENFORCE。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Declared symbols / 声明的符号, Iteration / 迭代处理.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Declared symbols / 声明的符号, Iteration / 迭代处理。

### Lines 113-140 / 第 113-140 行

```cpp
  }

  return ret;
}

std::vector<std::vector<at::IValue>> MobileModelRunner::
    get_all_bundled_inputs() {
  auto has_bundled_input = module_->find_method("get_all_bundled_inputs");
  CAFFE_ENFORCE(
      has_bundled_input,
      "Model does not have bundled inputs. ",
      "Use torch.utils.bundled_inputs.augment_model_with_bundled_inputs to add.");

  c10::IValue bundled_inputs = module_->run_method("get_all_bundled_inputs");
  return ivalue_to_bundled_inputs(bundled_inputs);
}

std::unordered_map<std::string, std::vector<std::vector<at::IValue>>>
MobileModelRunner::get_many_functions_bundled_inputs() {
  auto has_bundled_input =
      module_->find_method("get_bundled_inputs_functions_and_info");
  CAFFE_ENFORCE(
      has_bundled_input,
      "Model does not have bundled inputs. ",
      "Use torch.utils.bundled_inputs.augment_many_model_functions_with_bundled_inputs to add.");

  auto ival_bundled_inputs_mapping =
      module_->run_method("get_bundled_inputs_functions_and_info");
```

- **EN:** Important callable entry points in this range include get_all_bundled_inputs, CAFFE_ENFORCE, ivalue_to_bundled_inputs, get_many_functions_bundled_inputs.
- **CN:** 这一段的重要可调用入口包括 get_all_bundled_inputs, CAFFE_ENFORCE, ivalue_to_bundled_inputs, get_many_functions_bundled_inputs。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 141-168 / 第 141-168 行

```cpp
  auto bundled_inputs_mapping =
      ivalue_to_bundled_inputs_map(ival_bundled_inputs_mapping);

  std::unordered_map<std::string, std::vector<std::vector<at::IValue>>> ret;

  for (auto& entry : bundled_inputs_mapping) {
    std::string function_name = entry.first;
    std::string function_to_call = entry.second;

    auto has_func_to_call = module_->find_method(function_to_call);
    CAFFE_ENFORCE(
        has_func_to_call,
        "Model does not have ",
        function_to_call,
        "Use torch.utils.bundled_inputs.augment_many_model_functions_with_bundled_inputs to add.");

    c10::IValue bundled_inputs = module_->run_method(function_to_call);
    ret[function_name] = ivalue_to_bundled_inputs(bundled_inputs);
  }
  return ret;
}

std::vector<at::IValue> MobileModelRunner::run_with_inputs(
    std::vector<std::vector<at::IValue>> const& bundled_inputs) {
  std::vector<at::IValue> ret;
  ret.reserve(bundled_inputs.size());
  for (std::vector<at::IValue> const& input : bundled_inputs) {
    ret.emplace_back(module_->forward(input));
```

- **EN:** Important callable entry points in this range include ivalue_to_bundled_inputs_map, CAFFE_ENFORCE, run_with_inputs.
- **CN:** 这一段的重要可调用入口包括 ivalue_to_bundled_inputs_map, CAFFE_ENFORCE, run_with_inputs。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 169-196 / 第 169-196 行

```cpp
  }
  return ret;
}

std::vector<at::IValue> MobileModelRunner::run_with_inputs(
    const std::string& function_name,
    std::vector<std::vector<at::IValue>> const& bundled_inputs) const {
  std::vector<at::IValue> ret;
  ret.reserve(bundled_inputs.size());
  auto has_bundled_input = module_->find_method(function_name);
  CAFFE_ENFORCE(
      has_bundled_input,
      "Model does not have the method named ",
      function_name,
      "Please ensure that it was exported correctly");
  for (std::vector<at::IValue> const& input : bundled_inputs) {
    auto func = module_->get_method(function_name);
    ret.emplace_back(func(input));
  }
  return ret;
}

void MobileModelRunner::run_argless_functions(
    const std::vector<std::string>& functions) {
  for (auto& function_name : functions) {
    if (module_->find_method(function_name)) {
      module_->run_method(function_name);
    }
```

- **EN:** Important callable entry points in this range include run_with_inputs, CAFFE_ENFORCE, run_argless_functions.
- **CN:** 这一段的重要可调用入口包括 run_with_inputs, CAFFE_ENFORCE, run_argless_functions。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 197-224 / 第 197-224 行

```cpp
  }
}

bool MobileModelRunner::set_has_metal_gpu_operators(
    std::set<std::string> const& op_list) {
  for (std::string const& op : op_list) {
    if (op.find("metal::") == 0 || op.find("metal_prepack::") == 0 ||
        op.find("metal_prepack_unet::") == 0) {
      return true;
    }
  }
  return false;
}

void MobileModelRunner::for_each_tensor_in_bundled_inputs(
    std::function<void(const ::at::Tensor&)> const& func) {
  if (has_new_style_bundled_inputs()) {
    // Get the bundled inputs and access the arg level ivalues stored within
    auto bundled_inputs_mapping = this->get_many_functions_bundled_inputs();

    // Loop over functions
    for (auto& entry : bundled_inputs_mapping) {
      std::vector<std::vector<at::IValue>> bundled_inputs = entry.second;
      // Loop through inputs
      for (const std::vector<at::IValue>& input : bundled_inputs) {
        // Loop through values in an input
        for (const at::IValue& iv : input) {
          for_each_tensor_in_ivalue(iv, func);
```

- **EN:** Important callable entry points in this range include set_has_metal_gpu_operators, for_each_tensor_in_bundled_inputs, for_each_tensor_in_ivalue.
- **CN:** 这一段的重要可调用入口包括 set_has_metal_gpu_operators, for_each_tensor_in_bundled_inputs, for_each_tensor_in_ivalue。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。

### Lines 225-233 / 第 225-233 行

```cpp
        }
      }
    }
  } else {
    c10::IValue iv = module_->run_method("get_all_bundled_inputs");
    for_each_tensor_in_ivalue(iv, func);
  }
}
} // namespace torch::jit::mobile
```

- **EN:** Important callable entry points in this range include for_each_tensor_in_ivalue.
- **CN:** 这一段的重要可调用入口包括 for_each_tensor_in_ivalue。
- **EN:** The logic is oriented toward compact model packaging, bytecode handling, or mobile-runtime execution paths.
- **CN:** 该逻辑面向紧凑模型封装、字节码处理或移动端运行时执行路径。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Module API / 模块 API, Mobile runtime / 移动端运行时, Namespace scoping / 命名空间作用域.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Module API / 模块 API, Mobile runtime / 移动端运行时, Namespace scoping / 命名空间作用域。

## Key Concepts / 关键概念

- **Lite interpreter runtime** — 轻量解释器运行时
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Module API** — 模块 API
- **Mobile runtime** — 移动端运行时
- **Registration** — 注册机制
- **Core symbols: ivalue_to_bundled_inputs, CAFFE_ENFORCE, ivalue_to_bundled_inputs_map, get_all_bundled_inputs, get_many_functions_bundled_inputs, run_with_inputs, run_argless_functions, set_has_metal_gpu_operators** — 核心符号：ivalue_to_bundled_inputs、CAFFE_ENFORCE、ivalue_to_bundled_inputs_map、get_all_bundled_inputs、get_many_functions_bundled_inputs、run_with_inputs、run_argless_functions、set_has_metal_gpu_operators

## Dependencies / 依赖关系

- `torch/csrc/jit/mobile/model_tracer/MobileModelRunner.h`
- `torch/csrc/jit/mobile/model_tracer/TensorUtils.h`
