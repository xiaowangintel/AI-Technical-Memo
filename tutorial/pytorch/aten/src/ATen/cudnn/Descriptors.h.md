# Descriptors.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/cudnn/Descriptors.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Bridges ATen operators and helpers to cuDNN descriptors, handles, and execution paths. This specific file centers on `Descriptors.h`. Descriptor/handle lifecycle management is important here. Random-number generation or reproducibility semantics are explicitly encoded. Transform-aware tensor semantics used by functorch are part of the file scope.
- **Purpose (CN)**: 把 ATen 算子与辅助逻辑连接到 cuDNN 的描述符、句柄和执行路径。 该文件具体围绕 `Descriptors.h` 展开。 描述符/句柄的生命周期管理是这里的重要内容。 该文件显式编码了随机数生成或可复现性语义。 该文件范围内还包含 functorch 所需的变换感知张量语义。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行

```cpp
0001: #pragma once
0002: 
0003: #include <string>
0004: 
0005: #include <ATen/cuda/CUDAContext.h>
0006: #include <ATen/cuda/Exceptions.h>
0007: 
0008: #include <ATen/cudnn/cudnn-wrapper.h>
0009: #include <ATen/cudnn/Utils.h>
0010: #include <ATen/core/Tensor.h>
0011: #include <ATen/TensorUtils.h>
0012: #include <ATen/cuda/ATenCUDAGeneral.h>
0013: #include <cuda.h>
0014: 
0015: #ifndef AT_PER_OPERATOR_HEADERS
0016: #include <ATen/Functions.h>
0017: #else
0018: #include <ATen/ops/empty.h>
0019: #endif
0020: 
```

- **EN:** Configures compile-time behavior with preprocessor directives and feature gates. Key symbols: no prominent local symbols.
- **CN:** 通过预处理指令和特性开关配置编译期行为。关键符号：无明显局部符号。

### Lines 21-40 / 第 21-40 行

```cpp
0021: #if defined(CUDNN_VERSION) && CUDNN_VERSION >= 8907
0022: #define USE_CUDNN_RNN_V8_API
0023: #endif
0024: 
0025: namespace at::native {
0026: 
0027: std::string cudnnTypeToString(cudnnDataType_t dtype);
0028: 
0029: // TODO: Add constructors for all of the descriptors
0030: 
0031: inline int dataSize(cudnnDataType_t dataType)
0032: {
0033:   switch (dataType) {
0034:     case CUDNN_DATA_BFLOAT16:
0035:     case CUDNN_DATA_HALF: return 2;
0036:     case CUDNN_DATA_FLOAT: return 4;
0037:     default: return 8;
0038:   }
0039: }
0040: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; produces a result or forwards a computed value; manipulates tensor metadata, layout assumptions, or values; connects ATen logic to backend library/resource abstractions. Key symbols: `cudnnTypeToString`, `dataSize`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；返回结果或转发已计算的值；操作张量元数据、布局假设或实际数值；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`cudnnTypeToString`, `dataSize`。

### Lines 41-61 / 第 41-61 行

```cpp
0041: // NOTE [ cudnn fixSizeOneDimStride ]
0042: // The stride for a size-1 dimensions is not uniquely determined; in
0043: // fact, it can be anything you want, because the fact that the
0044: // tensor is size 1 at this dimension means that you will never actually
0045: // try advancing your pointer by this stride.
0046: //
0047: // However, CuDNN has a much more stringent requirement on strides:
0048: // if you are passing a contiguous input, it better be the case
0049: // that the stride for dim i is the product of the sizes of dims
0050: // i+1 to the end.  This stride is indeed uniquely determined.  This
0051: // function modifies 'stride' in place so this invariant holds.
0052: template <typename T>
0053: static inline void fixSizeOneDimStride(int dim, const T *size, T *stride, bool nhwc) {
0054:   int64_t z = 1;
0055:   int index = 0;
0056:   std::vector<int> permutation(dim);
0057: 
0058:   if (nhwc) {
0059:     permutation[index++] = 1;
0060:   }
0061:   for (int d = dim-1; d > 1; d--) {
```

- **EN:** Builds a reusable template/helper layer around `Descriptors`. Key symbols: `fixSizeOneDimStride`, `permutation`.
- **CN:** 围绕 `Descriptors` 构建可复用的模板或辅助层。关键符号：`fixSizeOneDimStride`, `permutation`。

### Lines 62-84 / 第 62-84 行

```cpp
0062:     permutation[index++] = d;
0063:   }
0064:   if (!nhwc) {
0065:     permutation[index++] = 1;
0066:   }
0067:   permutation[index++] = 0;
0068:   for (int d : permutation) {
0069:     if (size[d] == 1) {
0070:       stride[d] = z;
0071:     } else {
0072:       z *= size[d];
0073:     }
0074:   }
0075: }
0076: 
0077: template <typename T, cudnnStatus_t (*dtor)(T*)>
0078: struct DescriptorDeleter {
0079:   void operator()(T* x) {
0080:     if (x != nullptr) {
0081:       AT_CUDNN_CHECK(dtor(x));
0082:     }
0083:   }
0084: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `DescriptorDeleter`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`DescriptorDeleter`。

### Lines 85-107 / 第 85-107 行

```cpp
0085: 
0086: // A generic class for wrapping cuDNN descriptor types.  All you need
0087: // is to give the underlying type the Descriptor_t points to (usually,
0088: // if it's cudnnTensorDescriptor_t it points to cudnnTensorStruct),
0089: // the constructor and the destructor.  Subclasses are responsible
0090: // for defining a set() function to actually set the descriptor.
0091: //
0092: // Descriptors default construct to a nullptr, and have a descriptor
0093: // initialized the first time you call set() or any other initializing
0094: // function.
0095: template <typename T, cudnnStatus_t (*ctor)(T**), cudnnStatus_t (*dtor)(T*)>
0096: // NOLINTNEXTLINE(bugprone-exception-escape)
0097: class TORCH_CUDA_CPP_API Descriptor {
0098:  public:
0099:   // TODO: Figure out why const-correctness doesn't work here
0100: 
0101:   // Use desc() to access the underlying descriptor pointer in
0102:   // a read-only fashion.  Most client code should use this.
0103:   // If the descriptor was never initialized, this will return
0104:   // nullptr.
0105:   T* desc() const { return desc_.get(); }
0106:   T* desc() { return desc_.get(); }
0107: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `Descriptor`, `desc`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`Descriptor`, `desc`。

### Lines 108-135 / 第 108-135 行

```cpp
0108:   // Use mut_desc() to access the underlying descriptor pointer
0109:   // if you intend to modify what it points to (e.g., using
0110:   // cudnnSetFooDescriptor).  This will ensure that the descriptor
0111:   // is initialized.  Code in this file will use this function.
0112:   T* mut_desc() { init(); return desc_.get(); }
0113: protected:
0114:   void init() {
0115:     if (desc_ == nullptr) {
0116:       T* raw_desc = nullptr;
0117:       AT_CUDNN_CHECK(ctor(&raw_desc));
0118:       desc_.reset(raw_desc);
0119:     }
0120:   }
0121: private:
0122:   std::unique_ptr<T, DescriptorDeleter<T, dtor>> desc_;
0123: };
0124: 
0125: class TORCH_CUDA_CPP_API RNNDataDescriptor : public Descriptor<
0126:                                        cudnnRNNDataStruct,
0127:                                        &cudnnCreateRNNDataDescriptor,
0128:                                        &cudnnDestroyRNNDataDescriptor> {
0129: public:
0130:   void set(const at::Tensor &t, cudnnRNNDataLayout_t layout, int maxSeqLength, int batchSize, int vectorSize, const int* seqLengthArray);
0131: private:
0132:   void set(cudnnDataType_t dataType, cudnnRNNDataLayout_t layout, int maxSeqLength, int batchSize, int vectorSize, const int* seqLengthArray) {
0133:     AT_CUDNN_CHECK(cudnnSetRNNDataDescriptor(mut_desc(), dataType, layout, maxSeqLength, batchSize, vectorSize, seqLengthArray, nullptr));
0134:   }
0135: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RNNDataDescriptor`, `mut_desc`, `init`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RNNDataDescriptor`, `mut_desc`, `init`, `set`。

### Lines 136-159 / 第 136-159 行

```cpp
0136: 
0137: class TORCH_CUDA_CPP_API TensorDescriptor : public Descriptor<
0138:                                                cudnnTensorStruct,
0139:                                                &cudnnCreateTensorDescriptor,
0140:                                                &cudnnDestroyTensorDescriptor> {
0141:  public:
0142:   TensorDescriptor() = default;
0143:   explicit TensorDescriptor(const at::Tensor &t, size_t pad = 0) {
0144:     set(t, pad);
0145:   }
0146: 
0147:   // Note [CuDNN broadcast padding]
0148:   // ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
0149:   // pad specifies the minimum dimensionality of the tensor descriptor
0150:   // we produce (it doesn't have anything to do with, e.g., convolution
0151:   // padding).  If 't' is lower-dimensional than 'pad', the remaining
0152:   // dimensions (on the right) are padded with ones.  This doesn't
0153:   // affect the underlying data layout.  This is particularly useful for
0154:   // dealing with a peculiarity of the CuDNN API, which is that broadcasting in CuDNN is
0155:   // done in two steps: first, the client code is expected to pad out
0156:   // (the dimensions) input tensors to be the same dimension as the
0157:   // target broadcast, and then second, CuDNN takes of actually
0158:   // broadcasting size 1 dimensions.
0159: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `TensorDescriptor`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`TensorDescriptor`, `set`。

### Lines 160-186 / 第 160-186 行

```cpp
0160:   void set(const at::Tensor &t, size_t pad = 0);
0161:   void set(const at::Tensor &t, at::MemoryFormat memory_format, size_t pad = 0);
0162:   void set(cudnnDataType_t dataType, IntArrayRef sizes, IntArrayRef strides, size_t pad = 0);
0163: 
0164:   void print();
0165: 
0166: private:
0167:   void set(cudnnDataType_t dataType, IntArrayRef sizes, IntArrayRef strides, size_t pad, bool nhwc);
0168: 
0169:   void set(cudnnDataType_t dataType, int dim, int* size, int* stride, bool nhwc) {
0170:     std::vector<int> strides_copy(stride, stride + dim);
0171:     fixSizeOneDimStride<int>(dim, size, strides_copy.data(), nhwc);
0172:     AT_CUDNN_CHECK(cudnnSetTensorNdDescriptor(mut_desc(), dataType, dim, size, strides_copy.data()));
0173:   }
0174: };
0175: 
0176: std::ostream& operator<<(std::ostream & out, const TensorDescriptor& d);
0177: 
0178: class TORCH_CUDA_CPP_API FilterDescriptor : public Descriptor<
0179:                                                cudnnFilterStruct,
0180:                                                &cudnnCreateFilterDescriptor,
0181:                                                &cudnnDestroyFilterDescriptor> {
0182:  public:
0183:   void set(const at::Tensor &t, int64_t pad = 0) {
0184:     set(t, at::MemoryFormat::Contiguous, pad);
0185:   }
0186: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `FilterDescriptor`, `set`, `print`, `strides_copy`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`FilterDescriptor`, `set`, `print`, `strides_copy`。

### Lines 187-211 / 第 187-211 行

```cpp
0187:   void set(const at::Tensor &t, const at::MemoryFormat memory_format, int64_t pad = 0);
0188: 
0189:   void print();
0190: private:
0191:   void set(cudnnDataType_t dataType, int dim, int* size, cudnnTensorFormat_t filter_format) {
0192:     AT_CUDNN_CHECK(cudnnSetFilterNdDescriptor(mut_desc(), dataType, filter_format, dim, size));
0193:   }
0194: };
0195: 
0196: std::ostream& operator<<(std::ostream & out, const FilterDescriptor& d);
0197: 
0198: struct TORCH_CUDA_CPP_API ConvolutionDescriptor
0199:     : public Descriptor<
0200:           cudnnConvolutionStruct,
0201:           &cudnnCreateConvolutionDescriptor,
0202:           &cudnnDestroyConvolutionDescriptor> {
0203:   void set(cudnnDataType_t dataType, int dim, int* pad, int* stride, int * upscale /* aka dilation */, int groups, bool allow_tf32) {
0204:     cudnnDataType_t mathType = dataType;
0205:     if (dataType == CUDNN_DATA_HALF) mathType = CUDNN_DATA_FLOAT;
0206:     AT_CUDNN_CHECK(cudnnSetConvolutionNdDescriptor(mut_desc(), dim, pad, stride, upscale,
0207:                                           CUDNN_CROSS_CORRELATION, mathType));
0208:     AT_CUDNN_CHECK(cudnnSetConvolutionGroupCount(mut_desc(), groups));
0209:     // See Note [behavior of cudnnFind and cudnnGet]
0210:     AT_CUDNN_CHECK(cudnnSetConvolutionMathType(mut_desc(), CUDNN_DEFAULT_MATH));
0211:     if(dataType == CUDNN_DATA_HALF) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ConvolutionDescriptor`, `set`, `print`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ConvolutionDescriptor`, `set`, `print`。

### Lines 212-236 / 第 212-236 行

```cpp
0212:       AT_CUDNN_CHECK(cudnnSetConvolutionMathType(mut_desc(), CUDNN_TENSOR_OP_MATH));
0213:     } else if (dataType == CUDNN_DATA_FLOAT && !allow_tf32) {
0214:       AT_CUDNN_CHECK(cudnnSetConvolutionMathType(mut_desc(), CUDNN_FMA_MATH));
0215:     }
0216:   }
0217: };
0218: 
0219: struct TORCH_CUDA_CPP_API SpatialTransformerDescriptor
0220:     : public Descriptor<
0221:           cudnnSpatialTransformerStruct,
0222:           &cudnnCreateSpatialTransformerDescriptor,
0223:           &cudnnDestroySpatialTransformerDescriptor> {
0224:   void set(cudnnDataType_t dataType, int dim, int* size) {
0225:     AT_CUDNN_CHECK(cudnnSetSpatialTransformerNdDescriptor(mut_desc(), CUDNN_SAMPLER_BILINEAR, dataType, dim, size));
0226:   }
0227: };
0228: 
0229: // NOLINTNEXTLINE(bugprone-exception-escape)
0230: struct TORCH_CUDA_CPP_API DropoutDescriptor
0231:     : public Descriptor<
0232:           cudnnDropoutStruct,
0233:           &cudnnCreateDropoutDescriptor,
0234:           &cudnnDestroyDropoutDescriptor> {
0235:   at::Tensor state;
0236: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `SpatialTransformerDescriptor`, `DropoutDescriptor`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`SpatialTransformerDescriptor`, `DropoutDescriptor`, `set`。

### Lines 237-257 / 第 237-257 行

```cpp
0237:   // Initialize a dropout descriptor's RNG state.
0238:   // WARNING: This function is very expensive, avoid calling this function!
0239:   void initialize_rng(cudnnHandle_t handle, float dropout, long long int seed, const TensorOptions& options) {
0240:     TORCH_INTERNAL_ASSERT(dropout > 0, "dropout must be nonzero; otherwise call set_no_dropout");
0241:     size_t state_size = 0;
0242:     AT_CUDNN_CHECK(cudnnDropoutGetStatesSize(handle, &state_size));
0243:     AT_ASSERT(options.device().type() == kCUDA);
0244:     AT_ASSERT(options.dtype() == kByte);
0245:     state = at::empty({static_cast<int64_t>(state_size)}, options);
0246:     AT_CUDNN_CHECK(cudnnSetDropoutDescriptor(mut_desc(), handle, dropout, state.data_ptr(), state_size, seed));
0247:   }
0248: 
0249:   // Restore a dropout descriptor given a dropout probability and existing RNG state.
0250:   void set(cudnnHandle_t handle, float dropout, const at::Tensor& state) {
0251:     TORCH_INTERNAL_ASSERT(dropout > 0, "dropout must be nonzero; otherwise call set_no_dropout");
0252:     void *state_ptr = state.data_ptr();
0253:     size_t state_size = state.size(0);
0254:     // NB: The seed doesn't actually matter, so we give a dummy value
0255:     AT_CUDNN_CHECK(cudnnRestoreDropoutDescriptor(mut_desc(), handle, dropout, state_ptr, state_size, 0 /* seed */));
0256:   }
0257: 
```

- **EN:** This block manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions; validates runtime invariants before continuing; encodes random-number generation or reproducibility semantics. Key symbols: `initialize_rng`, `set`.
- **CN:** 该代码块操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象；在继续执行前校验运行时不变量；编码随机数生成或可复现性语义。关键符号：`initialize_rng`, `set`。

### Lines 258-277 / 第 258-277 行

```cpp
0258:   // Restore a dropout descriptor corresponding to no dropout
0259:   void set_no_dropout(cudnnHandle_t handle) {
0260:     // NB: seed doesn't matter when dropout = 0, because no random number
0261:     // initialization actually takes place when there is no dropout.
0262:     // NB: Empirically, cudnnSetDropoutDescriptor is cheap when
0263:     // dropout == 0
0264:     AT_CUDNN_CHECK(cudnnSetDropoutDescriptor(mut_desc(), handle, 0 /* dropout */, nullptr, 0 /* state_size */, 0 /* seed */));
0265:   }
0266: };
0267: 
0268: struct TORCH_CUDA_CPP_API RNNDescriptor : public Descriptor<
0269:                                              cudnnRNNStruct,
0270:                                              &cudnnCreateRNNDescriptor,
0271:                                              &cudnnDestroyRNNDescriptor> {
0272:   DropoutDescriptor dropout_desc_;
0273:   void set(cudnnHandle_t handle,
0274: #ifdef USE_CUDNN_RNN_V8_API
0275:        int input_size,
0276:        bool packed,
0277: #endif
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `RNNDescriptor`, `set_no_dropout`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`RNNDescriptor`, `set_no_dropout`。

### Lines 278-302 / 第 278-302 行

```cpp
0278:        int hidden_size, int proj_size, int num_layers, DropoutDescriptor&& dropout_desc,
0279:            cudnnRNNInputMode_t input_mode, cudnnDirectionMode_t bidirectional,
0280:            cudnnRNNMode_t mode, cudnnDataType_t datatype, cudnnDataType_t input_type, cudnnRNNAlgo_t algo, bool allow_tf32) {
0281:     dropout_desc_ = std::move(dropout_desc);
0282: #ifndef USE_CUDNN_RNN_V8_API
0283:     AT_CUDNN_CHECK(cudnnSetRNNDescriptor_v6(
0284:           handle,
0285:           mut_desc(),
0286:           hidden_size,
0287:           num_layers,
0288:           dropout_desc_.desc(),
0289:           input_mode,
0290:           bidirectional,
0291:           mode,
0292:           algo,
0293:           datatype));
0294:     if (proj_size != 0) {
0295:       AT_CUDNN_CHECK(cudnnSetRNNProjectionLayers(
0296:             handle,
0297:             /*rnnDesc=*/mut_desc(),
0298:             /*recProjSize=*/proj_size,
0299:             /*outProjSize=*/0));
0300:     }
0301:     cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
0302:     if (prop->major >= 7) {
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：无明显局部符号。

### Lines 303-341 / 第 303-341 行

```cpp
0303:       if (input_type == CUDNN_DATA_HALF) {
0304:         cudnnSetRNNMatrixMathType(mut_desc(), CUDNN_TENSOR_OP_MATH);
0305:       }
0306:       else if (input_type == CUDNN_DATA_FLOAT && !allow_tf32) {
0307:         cudnnSetRNNMatrixMathType(mut_desc(), CUDNN_FMA_MATH);
0308:       }
0309:       else {
0310:         // Technically, as the default it's not necessary to explicitly
0311:         // set this.
0312:         cudnnSetRNNMatrixMathType(mut_desc(), CUDNN_DEFAULT_MATH);
0313:       }
0314:     }
0315: #else
0316:     cudaDeviceProp* prop = at::cuda::getCurrentDeviceProperties();
0317:     auto math_type = CUDNN_DEFAULT_MATH;
0318:     if (prop->major >= 7) {
0319:       if (input_type == CUDNN_DATA_HALF) {
0320:         math_type = CUDNN_TENSOR_OP_MATH;
0321:       } else if (!allow_tf32) {
0322:         math_type = CUDNN_FMA_MATH;
0323:       }
0324:     }
0325:     AT_CUDNN_CHECK(cudnnSetRNNDescriptor_v8(
0326:           mut_desc(),
0327:           algo,
0328:           mode,
0329:           CUDNN_RNN_DOUBLE_BIAS,
0330:           bidirectional,
0331:           input_mode,
0332:           input_type,
0333:           datatype,
0334:           math_type,
0335:           input_size,
0336:           hidden_size,
0337:           proj_size ? proj_size : hidden_size,
0338:           num_layers,
0339:           dropout_desc_.desc(),
0340:           packed ? CUDNN_RNN_PADDED_IO_DISABLED : CUDNN_RNN_PADDED_IO_ENABLED));
0341: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; manipulates tensor metadata, layout assumptions, or values; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: `cudnnSetRNNMatrixMathType`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；操作张量元数据、布局假设或实际数值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`cudnnSetRNNMatrixMathType`。

### Lines 342-365 / 第 342-365 行

```cpp
0342:   }
0343: };
0344: 
0345: struct TORCH_CUDA_CPP_API CTCLossDescriptor
0346:     : public Descriptor<
0347:           cudnnCTCLossStruct,
0348:           &cudnnCreateCTCLossDescriptor,
0349:           &cudnnDestroyCTCLossDescriptor> {
0350:   void set(cudnnDataType_t datatype) {
0351:     AT_CUDNN_CHECK(cudnnSetCTCLossDescriptor(mut_desc(), datatype));
0352:   }
0353:   void setEx(
0354:       cudnnDataType_t datatype,
0355:       cudnnLossNormalizationMode_t normMode,
0356:       cudnnNanPropagation_t gradMode) {
0357:     AT_CUDNN_CHECK(
0358:         cudnnSetCTCLossDescriptorEx(mut_desc(), datatype, normMode, gradMode));
0359:   }
0360:   void set_v8_v9(
0361:       cudnnDataType_t datatype,
0362:       cudnnLossNormalizationMode_t normMode,
0363:       cudnnNanPropagation_t gradMode,
0364:       int maxLabelLength) {
0365: #if defined(CUDNN_VERSION) && CUDNN_VERSION >= 90000
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `CTCLossDescriptor`, `set`, `setEx`, `set_v8_v9`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`CTCLossDescriptor`, `set`, `setEx`, `set_v8_v9`。

### Lines 366-395 / 第 366-395 行

```cpp
0366:     auto gradModev9 = CUDNN_CTC_ZERO_OOB_GRADIENTS;
0367:     if (gradMode == cudnnNanPropagation_t::CUDNN_PROPAGATE_NAN) {
0368:       gradModev9 = CUDNN_CTC_SKIP_OOB_GRADIENTS;
0369:     }
0370:     AT_CUDNN_CHECK(
0371:         cudnnSetCTCLossDescriptor_v9(mut_desc(), datatype, normMode, gradModev9, maxLabelLength));
0372: #else
0373:     AT_CUDNN_CHECK(
0374:         cudnnSetCTCLossDescriptor_v8(mut_desc(), datatype, normMode, gradMode, maxLabelLength));
0375: #endif
0376:   }
0377: 
0378: };
0379: 
0380: struct TORCH_CUDA_CPP_API ActivationDescriptor
0381:     : public Descriptor<
0382:           cudnnActivationStruct,
0383:           &cudnnCreateActivationDescriptor,
0384:           &cudnnDestroyActivationDescriptor> {
0385:   void set(cudnnActivationMode_t mode) {
0386:     AT_ASSERT(
0387:         mode == CUDNN_ACTIVATION_RELU,
0388:         "TODO: support more cuDNN activation modes");
0389:     AT_CUDNN_CHECK(cudnnSetActivationDescriptor(
0390:         mut_desc(),
0391:         mode,
0392:         cudnnNanPropagation_t::CUDNN_NOT_PROPAGATE_NAN,
0393:         std::numeric_limits<double>::max()));
0394:   }
0395: };
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ActivationDescriptor`, `set`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ActivationDescriptor`, `set`。

### Lines 396-410 / 第 396-410 行

```cpp
0396: 
0397: union Constant
0398: {
0399:   float f;
0400:   double d;
0401:   Constant(cudnnDataType_t dataType, double value) {
0402:     if (dataType == CUDNN_DATA_HALF || dataType == CUDNN_DATA_FLOAT) {
0403:       f = static_cast<float>(value);
0404:     } else {
0405:       d = value;
0406:     }
0407:   }
0408: };
0409: 
0410: } // namespace
```

- **EN:** This block handles conditional branches and special cases; connects ATen logic to backend library/resource abstractions. Key symbols: `Constant`.
- **CN:** 该代码块处理条件分支与特殊情况；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`Constant`。


## Key Concepts / 关键概念
- **cuDNN integration** — cuDNN 集成
- **Tensor metadata and value flow** — 张量元数据与数值流
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Device and stream coordination** — 设备与流协调
- **Random-number generation** — 随机数生成
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **functorch transform support** — functorch 变换支持
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: DescriptorDeleter, Descriptor, RNNDataDescriptor, TensorDescriptor, FilterDescriptor, ConvolutionDescriptor, SpatialTransformerDescriptor, DropoutDescriptor** — 核心符号：DescriptorDeleter、Descriptor、RNNDataDescriptor、TensorDescriptor、FilterDescriptor、ConvolutionDescriptor、SpatialTransformerDescriptor、DropoutDescriptor

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/cuda/CUDAContext.h`, `ATen/cuda/Exceptions.h`, `ATen/cudnn/cudnn-wrapper.h`, `ATen/cudnn/Utils.h`, `ATen/core/Tensor.h`, `ATen/TensorUtils.h`, `ATen/cuda/ATenCUDAGeneral.h`, `ATen/Functions.h`, `ATen/ops/empty.h`
- **External includes / 外部头文件**: `string`, `cuda.h`
- **Namespaces / 命名空间**: `at::native`
- **Representative symbols / 代表性符号**: `DescriptorDeleter`, `Descriptor`, `RNNDataDescriptor`, `TensorDescriptor`, `FilterDescriptor`, `ConvolutionDescriptor`, `SpatialTransformerDescriptor`, `DropoutDescriptor`, `RNNDescriptor`, `CTCLossDescriptor`, `ActivationDescriptor`, `cudnnTypeToString`, `...`
