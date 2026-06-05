# Descriptors.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/miopen/Descriptors.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Bridges ATen operators and helpers to MIOpen descriptors, handles, and execution paths. This specific file centers on `Descriptors.h`. Descriptor/handle lifecycle management is important here. Random-number generation or reproducibility semantics are explicitly encoded.
- **Purpose (CN)**: 把 ATen 算子与辅助逻辑连接到 MIOpen 的描述符、句柄和执行路径。 该文件具体围绕 `Descriptors.h` 展开。 描述符/句柄的生命周期管理是这里的重要内容。 该文件显式编码了随机数生成或可复现性语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14 / 第 1-14 行

```cpp
0001: #pragma once
0002: 
0003: #include <ATen/miopen/Exceptions.h>
0004: 
0005: #include <ATen/miopen/miopen-wrapper.h>
0006: #include <ATen/core/Tensor.h>
0007: #include <ATen/TensorUtils.h>
0008: #include <c10/macros/Export.h>
0009: 
0010: namespace at { namespace native {
0011: 
0012: std::string miopenTypeToString(miopenDataType_t dtype);
0013: 
0014: inline int dataSize(miopenDataType_t dataType)
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions. Key symbols: `miopenTypeToString`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`miopenTypeToString`。

### Lines 15-30 / 第 15-30 行

```cpp
0015: {
0016:   switch (dataType) {
0017:     case miopenHalf: return 2;
0018:     case miopenFloat: return 4;
0019:     case miopenBFloat16: return 2;
0020:     default: return 8;
0021:   }
0022: }
0023: 
0024: // See NOTE [ cudnn fixSizeOneDimStride ] in aten/src/ATen/cudnn/Descriptors.h
0025: template <typename T>
0026: static inline void fixSizeOneDimStride(int dim, const T *size, T *stride, bool nhwc) {
0027:   int64_t z = 1;
0028:   int index = 0;
0029:   std::vector<int> permutation(dim);
0030: 
```

- **EN:** Builds a reusable template/helper layer around `Descriptors`. Key symbols: `fixSizeOneDimStride`, `permutation`.
- **CN:** 围绕 `Descriptors` 构建可复用的模板或辅助层。关键符号：`fixSizeOneDimStride`, `permutation`。

### Lines 31-49 / 第 31-49 行

```cpp
0031:   if (nhwc) {
0032:     permutation[index++] = 1;
0033:   }
0034:   for (int d = dim-1; d > 1; d--) {
0035:     permutation[index++] = d;
0036:   }
0037:   if (!nhwc) {
0038:     permutation[index++] = 1;
0039:   }
0040:   permutation[index++] = 0;
0041:   for (int d : permutation) {
0042:     if (size[d] == 1) {
0043:       stride[d] = z;
0044:     } else {
0045:       z *= size[d];
0046:     }
0047:   }
0048: }
0049: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: no prominent local symbols.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：无明显局部符号。

### Lines 50-68 / 第 50-68 行

```cpp
0050: template <typename T, miopenStatus_t (*dtor)(T*)>
0051: struct DescriptorDeleter {
0052:   void operator()(T* x) {
0053:     if (x != nullptr) {
0054:       MIOPEN_CHECK(dtor(x));
0055:     }
0056:   }
0057: };
0058: 
0059: // A generic class for wrapping MIOpen descriptor types.  All you need
0060: // is to give the underlying type the Descriptor_t points to (usually,
0061: // if it's miopenTensorDescriptor_t it points to miopenTensorStruct),
0062: // the constructor and the destructor.  Subclasses are responsible
0063: // for defining a set() function to actually set the descriptor.
0064: //
0065: // Descriptors default construct to a nullptr, and have a descriptor
0066: // initialized the first time you call set() or any other initializing
0067: // function.
0068: template <typename T, miopenStatus_t (*ctor)(T**), miopenStatus_t (*dtor)(T*)>
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `DescriptorDeleter`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`DescriptorDeleter`。

### Lines 69-86 / 第 69-86 行

```cpp
0069: // NOLINTNEXTLINE(bugprone-exception-escape)
0070: class TORCH_CUDA_CPP_API Descriptor {
0071:  public:
0072:   // Use desc() to access the underlying descriptor pointer in
0073:   // a read-only fashion.  Most client code should use this.
0074:   // If the descriptor was never initialized, this will return
0075:   // nullptr.
0076:   T* desc() const { return desc_.get(); }
0077:   T* desc() { return desc_.get(); }
0078: 
0079:   // Use mut_desc() to access the underlying descriptor pointer
0080:   // if you intend to modify what it points to (e.g., using
0081:   // miopenSetFooDescriptor).  This will ensure that the descriptor
0082:   // is initialized.  Code in this file will use this function.
0083:   T* mut_desc() { init(); return desc_.get(); }
0084: protected:
0085:   void init() {
0086:     if (desc_ == nullptr) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `Descriptor`, `desc`, `mut_desc`, `init`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`Descriptor`, `desc`, `mut_desc`, `init`。

### Lines 87-105 / 第 87-105 行

```cpp
0087:       T* raw_desc = nullptr;
0088:       MIOPEN_CHECK(ctor(&raw_desc));
0089:       desc_.reset(raw_desc);
0090:     }
0091:   }
0092: private:
0093:   std::unique_ptr<T, DescriptorDeleter<T, dtor>> desc_;
0094: };
0095: 
0096: class TORCH_CUDA_CPP_API TensorDescriptor : public Descriptor<
0097:                                                miopenTensorDescriptor,
0098:                                                &miopenCreateTensorDescriptor,
0099:                                                &miopenDestroyTensorDescriptor> {
0100:  public:
0101:   TensorDescriptor() = default;
0102:   explicit TensorDescriptor(const at::Tensor &t, size_t pad = 0) {
0103:     set(t, pad);
0104:   }
0105: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `TensorDescriptor`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`TensorDescriptor`, `set`。

### Lines 106-122 / 第 106-122 行

```cpp
0106:   // See Note [CuDNN broadcast padding]
0107:   void set(const at::Tensor &t, size_t pad = 0);
0108:   void set(const at::Tensor &t, at::MemoryFormat memory_format, size_t pad = 0);
0109:   void set(miopenDataType_t dataType, IntArrayRef sizes, IntArrayRef strides, size_t pad = 0);
0110: 
0111:   void print();
0112: 
0113: private:
0114:   void set(miopenDataType_t dataType, IntArrayRef sizes, IntArrayRef strides, size_t pad, bool nhwc);
0115: 
0116:   void set(miopenDataType_t dataType, int dim, size_t* size, size_t* stride, bool nhwc) {
0117:     std::vector<size_t> strides_copy(stride, stride + dim);
0118:     fixSizeOneDimStride<size_t>(dim, size, strides_copy.data(), nhwc);
0119:     // Use V2 API which supports 64-bit size/stride for large tensors
0120:     MIOPEN_CHECK(miopenSetTensorDescriptorV2(mut_desc(), dataType, dim, size, strides_copy.data()));
0121:   }
0122: };
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions. Key symbols: `set`, `print`, `strides_copy`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`set`, `print`, `strides_copy`。

### Lines 123-136 / 第 123-136 行

```cpp
0123: 
0124: std::ostream& operator<<(std::ostream & out, const TensorDescriptor& d);
0125: 
0126: class TORCH_CUDA_CPP_API FilterDescriptor : public Descriptor<
0127:                                                miopenTensorDescriptor,
0128:                                                &miopenCreateTensorDescriptor,
0129:                                                &miopenDestroyTensorDescriptor> {
0130:  public:
0131:   void set(const at::Tensor &t, int64_t pad = 0) {
0132:     set(t, at::MemoryFormat::Contiguous, pad);
0133:   }
0134: 
0135:   void set(const at::Tensor &t, const at::MemoryFormat memory_format, int64_t pad = 0);
0136: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `FilterDescriptor`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`FilterDescriptor`, `set`。

### Lines 137-155 / 第 137-155 行

```cpp
0137: private:
0138:   void set(miopenDataType_t dataType, int dim, size_t* size, size_t* stride, bool nhwc) {
0139:     std::vector<size_t> strides_copy(stride, stride + dim);
0140:     fixSizeOneDimStride<size_t>(dim, size, strides_copy.data(), nhwc);
0141:     // Use V2 API which supports 64-bit size/stride for large tensors
0142:     MIOPEN_CHECK(miopenSetTensorDescriptorV2(mut_desc(), dataType, dim, size, strides_copy.data()));
0143:   }
0144: };
0145: 
0146: struct TORCH_CUDA_CPP_API ConvolutionDescriptor
0147:     : public Descriptor<
0148:           miopenConvolutionDescriptor,
0149:           &miopenCreateConvolutionDescriptor,
0150:           &miopenDestroyConvolutionDescriptor> {
0151:   void set(miopenDataType_t dataType, miopenConvolutionMode_t c_mode,  int dim, int* pad, int* stride, int * upscale /* aka dilation */, int groups, bool benchmark, bool deterministic) {
0152:     MIOPEN_CHECK(miopenInitConvolutionNdDescriptor(mut_desc(), dim, pad, stride, upscale, c_mode));
0153:     MIOPEN_CHECK(miopenSetConvolutionGroupCount(mut_desc(), groups));
0154:     MIOPEN_CHECK(miopenSetConvolutionAttribute(mut_desc(), MIOPEN_CONVOLUTION_ATTRIB_DETERMINISTIC, deterministic ? 1 : 0));
0155:     if (benchmark) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ConvolutionDescriptor`, `set`, `strides_copy`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ConvolutionDescriptor`, `set`, `strides_copy`。

### Lines 156-171 / 第 156-171 行

```cpp
0156:       MIOPEN_CHECK(miopenSetConvolutionFindMode(mut_desc(), miopenConvolutionFindModeNormal));
0157:     }
0158:   }
0159: };
0160: 
0161: // NOLINTNEXTLINE(bugprone-exception-escape)
0162: struct TORCH_CUDA_CPP_API DropoutDescriptor
0163:     : public Descriptor<
0164:           miopenDropoutDescriptor,
0165:           &miopenCreateDropoutDescriptor,
0166:           &miopenDestroyDropoutDescriptor> {
0167:     void set(miopenHandle_t handle, float dropout, void* states, size_t stateSizeInBytes,
0168:              unsigned long long seed, bool use_mask, bool state_evo, miopenRNGType_t rng_mode) {
0169:       MIOPEN_CHECK(miopenSetDropoutDescriptor(mut_desc(), handle, dropout, states, stateSizeInBytes, seed, use_mask, state_evo, rng_mode));
0170:     }
0171: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `DropoutDescriptor`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`DropoutDescriptor`, `set`。

### Lines 172-187 / 第 172-187 行

```cpp
0172:     void restore(miopenHandle_t handle, float dropout, void* states, size_t stateSizeInBytes,
0173:       unsigned long long seed, bool use_mask, bool state_evo, miopenRNGType_t rng_mode) {
0174:       MIOPEN_CHECK(miopenRestoreDropoutDescriptor(mut_desc(), handle, dropout, states, stateSizeInBytes, seed, use_mask, state_evo, rng_mode));
0175:     }
0176: };
0177: 
0178: struct TORCH_CUDA_CPP_API RNNDescriptor
0179:   : public Descriptor<miopenRNNDescriptor,
0180:                       &miopenCreateRNNDescriptor,
0181:                       &miopenDestroyRNNDescriptor>
0182: {
0183:     void set(int64_t hidden_size, int64_t num_layers, miopenRNNInputMode_t input_mode, miopenRNNDirectionMode_t direction, miopenRNNMode_t rnn_mode,
0184:              miopenRNNBiasMode_t bias_mode, miopenRNNAlgo_t algorithm, miopenDataType_t datatype) {
0185:       MIOPEN_CHECK(miopenSetRNNDescriptor(mut_desc(), hidden_size, num_layers, input_mode, direction, rnn_mode, bias_mode, algorithm, datatype));
0186:     }
0187: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RNNDescriptor`, `restore`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RNNDescriptor`, `restore`, `set`。

### Lines 188-205 / 第 188-205 行

```cpp
0188:     void setWithDropout(DropoutDescriptor& dropout_desc, int64_t hidden_size, int64_t num_layers, miopenRNNInputMode_t input_mode, miopenRNNDirectionMode_t direction,
0189:                         miopenRNNMode_t rnn_mode, miopenRNNBiasMode_t bias_mode, miopenRNNAlgo_t algorithm, miopenDataType_t datatype) {
0190:       MIOPEN_CHECK(miopenSetRNNDescriptor_V2(mut_desc(), hidden_size, num_layers, dropout_desc.mut_desc(), input_mode, direction, rnn_mode, bias_mode, algorithm, datatype));
0191:     }
0192: };
0193: 
0194: union Constant
0195: {
0196:   float f;
0197:   double d;
0198:   Constant(miopenDataType_t dataType, double value) {
0199:     if (dataType == miopenHalf || dataType == miopenFloat || dataType == miopenBFloat16) {
0200:       f = static_cast<float>(value);
0201:     } else {
0202:       d = value;
0203:     }
0204:   }
0205: };
```

- **EN:** This block handles conditional branches and special cases; connects ATen logic to backend library/resource abstractions. Key symbols: `setWithDropout`, `Constant`.
- **CN:** 该代码块处理条件分支与特殊情况；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`setWithDropout`, `Constant`。

### Lines 206-207 / 第 206-207 行

```cpp
0206: 
0207: }} // namespace
```

- **EN:** This block implements local helper logic for `Descriptors`. Key symbols: no prominent local symbols.
- **CN:** 该代码块实现与 `Descriptors` 相关的局部辅助逻辑。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **MIOpen integration** — MIOpen 集成
- **Tensor metadata and value flow** — 张量元数据与数值流
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Device and stream coordination** — 设备与流协调
- **Random-number generation** — 随机数生成
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: DescriptorDeleter, Descriptor, TensorDescriptor, FilterDescriptor, ConvolutionDescriptor, DropoutDescriptor, RNNDescriptor, miopenTypeToString** — 核心符号：DescriptorDeleter、Descriptor、TensorDescriptor、FilterDescriptor、ConvolutionDescriptor、DropoutDescriptor、RNNDescriptor、miopenTypeToString

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/miopen/Exceptions.h`, `ATen/miopen/miopen-wrapper.h`, `ATen/core/Tensor.h`, `ATen/TensorUtils.h`, `c10/macros/Export.h`
- **External includes / 外部头文件**: 无
- **Namespaces / 命名空间**: `at`, `native`
- **Representative symbols / 代表性符号**: `DescriptorDeleter`, `Descriptor`, `TensorDescriptor`, `FilterDescriptor`, `ConvolutionDescriptor`, `DropoutDescriptor`, `RNNDescriptor`, `miopenTypeToString`, `dataSize`, `fixSizeOneDimStride`, `permutation`, `desc`, `...`
