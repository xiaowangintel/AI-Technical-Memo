# CuFFTPlanCache.h — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/CuFFTPlanCache.h`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Declares or defines CUDA helpers/templates associated with `CuFFTParams`, `cufft_complex_input`, `cufft_complex_output`, `GetCuFFTTransformType`.
- 用途（中文）: 声明或定义与 `CuFFTParams`, `cufft_complex_input`, `cufft_complex_output`, `GetCuFFTTransformType` 相关的 CUDA 辅助函数/模板。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```cpp
   1: #include <ATen/Config.h>
   2: #include <ATen/core/DimVector.h>
   3: #include <ATen/cuda/CUDAContext.h>
   4: #include <ATen/native/cuda/CuFFTUtils.h>
   5: #include <ATen/native/utils/ParamsHash.h>
   6: #include <c10/util/accumulate.h>
   7: #include <c10/util/irange.h>
   8: 
   9: #include <cufft.h>
  10: #include <cufftXt.h>
  11: 
  12: #include <limits>
  13: #include <list>
  14: #include <sstream>
  15: #include <stdexcept>
  16: #include <string>
  17: #include <unordered_map>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Config.h>`, `<ATen/core/DimVector.h>`, `<ATen/cuda/CUDAContext.h>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Config.h>`, `<ATen/core/DimVector.h>`, `<ATen/cuda/CUDAContext.h>`。

### Lines 19-39
```cpp
  19: namespace at::native::detail {
  20: 
  21: // Enum representing the FFT type
  22: enum class CuFFTTransformType : int8_t {
  23:   C2C,  // Complex-to-complex
  24:   R2C,  // Real-to-complex
  25:   C2R,  // Complex-to-real
  26: };
  27: 
  28: // This struct is used to let us easily compute hashes of the
  29: // parameters.
  30: // It will be the **key** to the plan cache.
  31: struct CuFFTParams
  32: {
  33:   int64_t signal_ndim_; // between 1 and max_rank, i.e., 1 <= signal_ndim <= 3
  34:   // These include additional batch dimension as well.
  35:   int64_t sizes_[max_rank + 1];
  36:   int64_t input_strides_[max_rank + 1];
  37:   int64_t output_strides_[max_rank + 1];
  38:   CuFFTTransformType fft_type_;
  39:   ScalarType value_type_;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 41-41
```cpp
  41:   CuFFTParams() = default;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 43-58
```cpp
  43:   CuFFTParams(IntArrayRef in_strides, IntArrayRef out_strides,
  44:       IntArrayRef signal_sizes, CuFFTTransformType fft_type, ScalarType value_type) {
  45:     // Padding bits must be zeroed for hashing
  46:     memset(this, 0, sizeof(*this));
  47:     signal_ndim_ = signal_sizes.size() - 1;
  48:     fft_type_ = fft_type;
  49:     value_type_ = value_type;
  50: 
  51:     TORCH_INTERNAL_ASSERT(in_strides.size() == signal_sizes.size());
  52:     TORCH_INTERNAL_ASSERT(out_strides.size() == signal_sizes.size());
  53:     TORCH_INTERNAL_ASSERT(1 <= signal_ndim_ && signal_ndim_ <= max_rank);
  54: 
  55:     std::copy(signal_sizes.cbegin(), signal_sizes.cend(), sizes_);
  56:     std::copy(in_strides.cbegin(), in_strides.cend(), input_strides_);
  57:     std::copy(out_strides.cbegin(), out_strides.cend(), output_strides_);
  58:   }
```
- EN: This block defines or continues the implementation of `CuFFTParams`.
- CN: 该代码块定义或继续实现 `CuFFTParams`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 59-59
```cpp
  59: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 61-61
```cpp
  61: static_assert(std::is_trivial_v<CuFFTParams> );
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 63-63
```cpp
  63: // Returns true if the transform type has complex input
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 64-74
```cpp
  64: inline bool cufft_complex_input(CuFFTTransformType type) {
  65:   switch (type) {
  66:     case CuFFTTransformType::C2C:
  67:     case CuFFTTransformType::C2R:
  68:       return true;
  69: 
  70:     case CuFFTTransformType::R2C:
  71:       return false;
  72:   }
  73:   TORCH_INTERNAL_ASSERT(false);
  74: }
```
- EN: This block defines or continues the implementation of `cufft_complex_input`.
- CN: 该代码块定义或继续实现 `cufft_complex_input`。

### Lines 76-76
```cpp
  76: // Returns true if the transform type has complex output
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 77-87
```cpp
  77: inline bool cufft_complex_output(CuFFTTransformType type) {
  78:   switch (type) {
  79:     case CuFFTTransformType::C2C:
  80:     case CuFFTTransformType::R2C:
  81:       return true;
  82: 
  83:     case CuFFTTransformType::C2R:
  84:       return false;
  85:   }
  86:   TORCH_INTERNAL_ASSERT(false);
  87: }
```
- EN: This block defines or continues the implementation of `cufft_complex_output`.
- CN: 该代码块定义或继续实现 `cufft_complex_output`。

### Lines 89-89
```cpp
  89: // Create transform type enum from bools representing if input and output are complex
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 90-99
```cpp
  90: inline CuFFTTransformType GetCuFFTTransformType(bool complex_input, bool complex_output) {
  91:   if (complex_input && complex_output) {
  92:     return CuFFTTransformType::C2C;
  93:   } else if (complex_input && !complex_output) {
  94:     return CuFFTTransformType::C2R;
  95:   } else if (!complex_input && complex_output) {
  96:     return CuFFTTransformType::R2C;
  97:   }
  98:   TORCH_INTERNAL_ASSERT(false, "Real to real FFTs are not supported");
  99: }
```
- EN: This block defines or continues the implementation of `GetCuFFTTransformType`.
- CN: 该代码块定义或继续实现 `GetCuFFTTransformType`。

### Lines 102-119
```cpp
 102: class CuFFTHandle {
 103:   ::cufftHandle handle_;
 104: public:
 105: 
 106:   CuFFTHandle() {
 107:     CUFFT_CHECK(cufftCreate(&handle_));
 108:   }
 109: 
 110:   ::cufftHandle & get() { return handle_; }
 111:   const ::cufftHandle & get() const { return handle_; }
 112: 
 113:   ~CuFFTHandle() {
 114: // Not using fftDestroy() for rocFFT to work around double freeing of handles
 115: #if !defined(USE_ROCM)
 116:     cufftDestroy(handle_);
 117: #endif
 118:   }
 119: };
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。
- EN: This block defines or continues the implementation of `CuFFTHandle`, `get`.
- CN: 该代码块定义或继续实现 `CuFFTHandle`, `get`。

### Lines 121-124
```cpp
 121: __forceinline__
 122: static bool is_pow_of_two(int64_t x) {
 123:   return (x & (x - 1)) == 0;
 124: }
```
- EN: This block defines or continues the implementation of `is_pow_of_two`.
- CN: 该代码块定义或继续实现 `is_pow_of_two`。

### Lines 126-126
```cpp
 126: using cufft_size_type = long long int;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 128-128
```cpp
 128: using CuFFTDimVector = c10::SmallVector<cufft_size_type, at::kDimVectorStaticSize>;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 130-131
```cpp
 130: // Struct representing a tensor in CuFFT's data layout for planning transforms
 131: // See NOTE [ cuFFT Embedded Strides ].
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 132-136
```cpp
 132: struct CuFFTDataLayout {
 133:   CuFFTDimVector embed;
 134:   cufft_size_type stride, dist;
 135:   bool must_clone, simple;
 136: };
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 138-140
```cpp
 138: // Returns a cufft embedding for a contiguous signal of the given size.
 139: // e.g. if the input is cloned, this will be the resulting data layout
 140: // See NOTE [ cuFFT Embedded Strides ].
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 141-155
```cpp
 141: inline CuFFTDataLayout cufft_simple_embed(IntArrayRef sizes, bool onesided) {
 142:   CuFFTDataLayout layout;
 143:   layout.simple = true;
 144:   layout.must_clone = false;
 145:   layout.embed.assign(sizes.cbegin() + 1, sizes.cend());
 146:   if (onesided) {
 147:     layout.embed.back() = sizes.back() / 2 + 1;
 148:   }
 149:   layout.stride = 1;
 150:   layout.dist = 1;
 151:   for (const auto& len : layout.embed) {
 152:     layout.dist *= len;
 153:   }
 154:   return layout;
 155: }
```
- EN: This block defines or continues the implementation of `cufft_simple_embed`.
- CN: 该代码块定义或继续实现 `cufft_simple_embed`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 157-159
```cpp
 157: // Convert strides to a CuFFT embedded representation.
 158: // If strides cannot be embedded, returns a simple layout and sets must_clone flag
 159: // See NOTE [ cuFFT Embedded Strides ].
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 160-181
```cpp
 160: inline CuFFTDataLayout as_cufft_embed(IntArrayRef strides, IntArrayRef sizes, bool onesided) {
 161:   const auto signal_ndim = strides.size() - 1;
 162:   CuFFTDataLayout layout;
 163:   auto last_stride = strides[signal_ndim];
 164:   layout.must_clone = (last_stride <= 0);
 165: 
 166:   const auto last_dim_size = onesided ?
 167:       sizes[signal_ndim] / 2 + 1 : sizes[signal_ndim];
 168:   const auto signal_numel = c10::multiply_integers(sizes.slice(1, sizes.size() - 2)) * last_dim_size;
 169: 
 170:   // Zero stides are not allowed, even if the batch size is one.
 171:   // If that happens just set a dummy case
 172:   if (sizes[0] == 1) {
 173:     layout.dist = signal_numel;
 174:   } else if (strides[0] == 0) {
 175:     layout.must_clone = true;
 176:   } else {
 177:     layout.dist = strides[0];
 178:   }
 179: 
 180:   // Calculate the embedding shape, or set must_clone if the strides cannot be embedded
 181:   layout.embed.resize(signal_ndim);
```
- EN: This block defines or continues the implementation of `as_cufft_embed`.
- CN: 该代码块定义或继续实现 `as_cufft_embed`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 182-192
```cpp
 182:   for (auto i = signal_ndim - 1; !layout.must_clone && i > 0; i--) {
 183:     auto stride = strides[i];
 184:     if (sizes[i] == 1) {
 185:       layout.embed[i] = 1;
 186:     } else if (stride > 0 && stride % last_stride == 0) {
 187:       layout.embed[i] = stride / last_stride;
 188:       last_stride = stride;
 189:     } else {
 190:       layout.must_clone = true;
 191:     }
 192:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 194-212
```cpp
 194:   if (layout.must_clone) {
 195:     // If the input needs to be cloned, assume it will be contiguous
 196:     layout = cufft_simple_embed(sizes, onesided);
 197:     layout.must_clone = true;
 198:   } else {
 199:     layout.embed[0] = sizes[1];
 200:     layout.stride = strides[signal_ndim];
 201:     // Determine if layout represents a simple embedding (contiguous data)
 202:     layout.simple = [&] {
 203:       for (const auto i : c10::irange(1, signal_ndim - 1)) {
 204:         if (layout.embed[i] != sizes[i + 1]) {
 205:           return false;
 206:         }
 207:       }
 208: 
 209:       return (layout.stride == 1 && layout.dist == signal_numel &&
 210:           layout.embed.back() == last_dim_size);
 211:     }();
 212:   }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 213-214
```cpp
 213:   return layout;
 214: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 216-222
```cpp
 216: // This class contains all the information needed to execute a cuFFT plan:
 217: //   1. the plan
 218: //   2. whether to clone input before executing the plan
 219: //   3. the workspace size needed
 220: //
 221: // This class will be the **value** in the plan cache.
 222: // It **owns** the raw plan via a unique_ptr.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 223-244
```cpp
 223: class CuFFTConfig {
 224: public:
 225: 
 226:   // Only move semantics is enough for this class. Although we already use
 227:   // unique_ptr for the plan, still remove copy constructor and assignment op so
 228:   // we don't accidentally copy and take perf hit.
 229:   CuFFTConfig(const CuFFTConfig&) = delete;
 230:   CuFFTConfig& operator=(CuFFTConfig const&) = delete;
 231: 
 232:   explicit CuFFTConfig(const CuFFTParams& params):
 233:       CuFFTConfig(
 234:           IntArrayRef(params.input_strides_, params.signal_ndim_ + 1),
 235:           IntArrayRef(params.output_strides_, params.signal_ndim_ + 1),
 236:           IntArrayRef(params.sizes_, params.signal_ndim_ + 1),
 237:           params.fft_type_,
 238:           params.value_type_) {}
 239: 
 240:   // For complex types, strides are in units of 2 * element_size(dtype)
 241:   // sizes are for the full signal, including batch size and always two-sided
 242:   CuFFTConfig(IntArrayRef in_strides, IntArrayRef out_strides,
 243:       IntArrayRef sizes, CuFFTTransformType fft_type, ScalarType dtype):
 244:         fft_type_(fft_type), value_type_(dtype) {
```
- EN: This block defines or continues the implementation of `CuFFTConfig`, `element_size`.
- CN: 该代码块定义或继续实现 `CuFFTConfig`, `element_size`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 246-246
```cpp
 246:     // signal sizes (excluding batch dim)
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 247-247
```cpp
 247:     CuFFTDimVector signal_sizes(sizes.begin() + 1, sizes.end());
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 249-249
```cpp
 249:     // input batch size
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 250-251
```cpp
 250:     const int64_t batch = sizes[0];
 251:     const int64_t signal_ndim = sizes.size() - 1;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 253-255
```cpp
 253:     // Since cuFFT has limited non-unit stride support and various constraints, we
 254:     // use a flag to keep track throughout this function to see if we need to
 255:     // input = input.clone();
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 257-257
```cpp
 257: #if defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 258-258
```cpp
 258:     // clone input to avoid issues with hipfft clobering the input and failing tests
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 259-262
```cpp
 259:     clone_input = true;
 260: #else
 261:     clone_input = false;
 262: #endif
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 264-266
```cpp
 264:     // For half, base strides on the real part of real-to-complex and
 265:     // complex-to-real transforms are not supported. Since our output is always
 266:     // contiguous, only need to check real-to-complex case.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 267-281
```cpp
 267:     if (dtype == ScalarType::Half) {
 268:       // cuFFT on half requires compute capability of at least SM_53
 269:       auto dev_prop = at::cuda::getCurrentDeviceProperties();
 270:       TORCH_CHECK(dev_prop->major >= 5 && !(dev_prop->major == 5 && dev_prop->minor < 3),
 271:                "cuFFT doesn't support signals of half type with compute "
 272:                "capability less than SM_53, but the device containing input half "
 273:                "tensor only has SM_", dev_prop->major, dev_prop->minor);
 274:       for (const auto i : c10::irange(signal_ndim)) {
 275:         TORCH_CHECK(is_pow_of_two(sizes[i + 1]),
 276:             "cuFFT only supports dimensions whose sizes are powers of two when"
 277:             " computing in half precision, but got a signal size of",
 278:             sizes.slice(1));
 279:       }
 280:       clone_input |= in_strides.back() != 1;
 281:     }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 283-291
```cpp
 283:     CuFFTDataLayout in_layout;
 284:     if (clone_input) {
 285:       in_layout = cufft_simple_embed(sizes, fft_type == CuFFTTransformType::C2R);
 286:     } else {
 287:       in_layout = as_cufft_embed(in_strides, sizes, fft_type == CuFFTTransformType::C2R);
 288:     }
 289:     auto out_layout = as_cufft_embed(out_strides, sizes, fft_type == CuFFTTransformType::R2C);
 290:     TORCH_INTERNAL_ASSERT(!out_layout.must_clone, "Out strides cannot be represented as CuFFT embedding");
 291:     clone_input |= in_layout.must_clone;
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 293-295
```cpp
 293:     // Check if we can take advantage of simple data layout.
 294:     //
 295:     // See NOTE [ cuFFT Embedded Strides ] in native/cuda/SpectralOps.cu.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 297-315
```cpp
 297:     const bool simple_layout = in_layout.simple && out_layout.simple;
 298:     cudaDataType itype, otype, exec_type;
 299:     const auto complex_input = cufft_complex_input(fft_type);
 300:     const auto complex_output = cufft_complex_output(fft_type);
 301:     if (dtype == ScalarType::Float) {
 302:       itype = complex_input ? CUDA_C_32F : CUDA_R_32F;
 303:       otype = complex_output ? CUDA_C_32F : CUDA_R_32F;
 304:       exec_type = CUDA_C_32F;
 305:     } else if (dtype == ScalarType::Double) {
 306:       itype = complex_input ? CUDA_C_64F : CUDA_R_64F;
 307:       otype = complex_output ? CUDA_C_64F : CUDA_R_64F;
 308:       exec_type = CUDA_C_64F;
 309:     } else if (dtype == ScalarType::Half) {
 310:       itype = complex_input ? CUDA_C_16F : CUDA_R_16F;
 311:       otype = complex_output ? CUDA_C_16F : CUDA_R_16F;
 312:       exec_type = CUDA_C_16F;
 313:     } else {
 314:       TORCH_CHECK(false, "cuFFT doesn't support tensor of type: ", dtype);
 315:     }
```
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。

### Lines 317-317
```cpp
 317:     // disable auto allocation of workspace to use THC allocator
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 318-318
```cpp
 318:     CUFFT_CHECK(cufftSetAutoAllocation(plan(), /* autoAllocate */ 0));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 320-320
```cpp
 320:     size_t ws_size_t;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 322-322
```cpp
 322:     // make plan
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 323-338
```cpp
 323:     if (simple_layout) {
 324:       // If with unit-stride, we tell cuFFT by setting inembed == onembed == NULL.
 325:       // In such case, cuFFT ignores istride, ostride, idist, and odist
 326:       // by assuming istride = ostride = 1.
 327:       //
 328:       // See NOTE [ cuFFT Embedded Strides ] in native/cuda/SpectralOps.cu.
 329:       CUFFT_CHECK(cufftXtMakePlanMany(plan(), signal_ndim, signal_sizes.data(),
 330:         /* inembed */ nullptr, /* base_istride */ 1, /* idist */ 1, itype,
 331:         /* onembed */ nullptr, /* base_ostride */ 1, /* odist */ 1, otype,
 332:         batch, &ws_size_t, exec_type));
 333:     } else {
 334:       CUFFT_CHECK(cufftXtMakePlanMany(plan(), signal_ndim, signal_sizes.data(),
 335:             in_layout.embed.data(), in_layout.stride, in_layout.dist, itype,
 336:             out_layout.embed.data(), out_layout.stride, out_layout.dist, otype,
 337:             batch, &ws_size_t, exec_type));
 338:     }
```
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 339-340
```cpp
 339:     ws_size = static_cast<int64_t>(ws_size_t);
 340:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 342-342
```cpp
 342:   const cufftHandle &plan() const { return plan_ptr.get(); }
```
- EN: This block defines or continues the implementation of `plan`.
- CN: 该代码块定义或继续实现 `plan`。

### Lines 344-347
```cpp
 344:   CuFFTTransformType transform_type() const { return fft_type_; }
 345:   ScalarType data_type() const { return value_type_; }
 346:   bool should_clone_input() const { return clone_input; }
 347:   int64_t workspace_size() const { return ws_size; }
```
- EN: This block defines or continues the implementation of `transform_type`, `data_type`, `should_clone_input`.
- CN: 该代码块定义或继续实现 `transform_type`, `data_type`, `should_clone_input`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 349-355
```cpp
 349: private:
 350:   CuFFTHandle plan_ptr;
 351:   bool clone_input;
 352:   int64_t ws_size;
 353:   CuFFTTransformType fft_type_;
 354:   ScalarType value_type_;
 355: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 357-357
```cpp
 357: #if defined(USE_ROCM)
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 358-359
```cpp
 358:   // Note that the max plan number for CUDA version < 10 has to be 1023
 359:   // due to a bug that fails on the 1024th plan
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 360-372
```cpp
 360:   constexpr int64_t CUFFT_MAX_PLAN_NUM = 1023;
 361:   constexpr int64_t CUFFT_DEFAULT_CACHE_SIZE = CUFFT_MAX_PLAN_NUM;
 362: #else
 363:   constexpr int64_t CUFFT_MAX_PLAN_NUM = std::numeric_limits<int64_t>::max();
 364:   // The default max cache size chosen for CUDA version > 10 is arbitrary.
 365:   // This number puts a limit on how big of a plan cache should we maintain by
 366:   // default. Users can always configure it via cufft_set_plan_cache_max_size.
 367:   constexpr int64_t CUFFT_DEFAULT_CACHE_SIZE = 4096;
 368: #endif
 369: static_assert(0 <= CUFFT_MAX_PLAN_NUM && CUFFT_MAX_PLAN_NUM <= std::numeric_limits<int64_t>::max(),
 370:               "CUFFT_MAX_PLAN_NUM not in size_t range");
 371: static_assert(CUFFT_DEFAULT_CACHE_SIZE >= 0 && CUFFT_DEFAULT_CACHE_SIZE <= CUFFT_MAX_PLAN_NUM,
 372:               "CUFFT_DEFAULT_CACHE_SIZE not in [0, CUFFT_MAX_PLAN_NUM] range");
```
- EN: The preprocessor directives choose compile-time branches so the source can adapt to optional features or alternate CUDA paths.
- CN: 这些预处理指令在编译期选择不同分支，使源码能够适配可选特性或备用 CUDA 路径。

### Lines 374-378
```cpp
 374: // This cache assumes that the mapping from key to value never changes.
 375: // This is **NOT** thread-safe. Please use a mutex when using it **AND** the
 376: // value returned from try_emplace_value.
 377: // The contract of using this cache is that try_emplace_value should only be
 378: // used when the max_size is positive.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 379-400
```cpp
 379: class CuFFTParamsLRUCache {
 380: public:
 381:   using kv_t = typename std::pair<CuFFTParams, CuFFTConfig>;
 382:   using map_t = typename std::unordered_map<std::reference_wrapper<CuFFTParams>,
 383:                                             typename std::list<kv_t>::iterator,
 384:                                             ParamsHash<CuFFTParams>,
 385:                                             ParamsEqual<CuFFTParams>>;
 386:   using map_kkv_iter_t = typename map_t::iterator;
 387: 
 388: 
 389:   CuFFTParamsLRUCache() : CuFFTParamsLRUCache(CUFFT_DEFAULT_CACHE_SIZE) {}
 390: 
 391:   CuFFTParamsLRUCache(int64_t max_size) {
 392:     _set_max_size(max_size);
 393:   }
 394: 
 395:   CuFFTParamsLRUCache(CuFFTParamsLRUCache&& other) noexcept :
 396:     _usage_list(std::move(other._usage_list)),
 397:     _cache_map(std::move(other._cache_map)),
 398:     _max_size(other._max_size) {}
 399: 
 400:   CuFFTParamsLRUCache& operator=(CuFFTParamsLRUCache&& other) noexcept {
```
- EN: This block defines or continues the implementation of `CuFFTParamsLRUCache`.
- CN: 该代码块定义或继续实现 `CuFFTParamsLRUCache`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 401-405
```cpp
 401:     _usage_list = std::move(other._usage_list);
 402:     _cache_map = std::move(other._cache_map);
 403:     _max_size = other._max_size;
 404:     return *this;
 405:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 407-410
```cpp
 407:   // If key is in this cache, return the cached config. Otherwise, emplace the
 408:   // config in this cache and return it.
 409:   // Return const reference because CuFFTConfig shouldn't be tampered with once
 410:   // created.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 411-432
```cpp
 411:   const CuFFTConfig &lookup(CuFFTParams params) {
 412:     AT_ASSERT(_max_size > 0);
 413: 
 414:     map_kkv_iter_t map_it = _cache_map.find(params);
 415:     // Hit, put to list front
 416:     if (map_it != _cache_map.end()) {
 417:       _usage_list.splice(_usage_list.begin(), _usage_list, map_it->second);
 418:       return map_it->second->second;
 419:     }
 420: 
 421:     // Miss
 422:     // remove if needed
 423:     if (_usage_list.size() >= _max_size) {
 424:       auto last = _usage_list.end();
 425:       last--;
 426:       _cache_map.erase(last->first);
 427:       _usage_list.pop_back();
 428:     }
 429: 
 430:     // construct new plan at list front, then insert into _cache_map
 431:     _usage_list.emplace_front(std::piecewise_construct,
 432:                        std::forward_as_tuple(params),
```
- EN: This block defines or continues the implementation of `lookup`.
- CN: 该代码块定义或继续实现 `lookup`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 433-439
```cpp
 433:                        std::forward_as_tuple(params));
 434:     auto kv_it = _usage_list.begin();
 435:     _cache_map.emplace(std::piecewise_construct,
 436:                 std::forward_as_tuple(kv_it->first),
 437:                 std::forward_as_tuple(kv_it));
 438:     return kv_it->second;
 439:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 441-444
```cpp
 441:   void clear() {
 442:     _cache_map.clear();
 443:     _usage_list.clear();
 444:   }
```
- EN: This block defines or continues the implementation of `clear`.
- CN: 该代码块定义或继续实现 `clear`。

### Lines 446-457
```cpp
 446:   void resize(int64_t new_size) {
 447:     _set_max_size(new_size);
 448:     auto cur_size = _usage_list.size();
 449:     if (cur_size > _max_size) {
 450:       auto delete_it = _usage_list.end();
 451:       for (size_t i = 0; i < cur_size - _max_size; i++) {
 452:         delete_it--;
 453:         _cache_map.erase(delete_it->first);
 454:       }
 455:       _usage_list.erase(delete_it, _usage_list.end());
 456:     }
 457:   }
```
- EN: This block defines or continues the implementation of `resize`.
- CN: 该代码块定义或继续实现 `resize`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 459-459
```cpp
 459:   size_t size() const { return _cache_map.size(); }
```
- EN: This block defines or continues the implementation of `size`.
- CN: 该代码块定义或继续实现 `size`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 461-461
```cpp
 461:   size_t max_size() const noexcept { return _max_size; }
```
- EN: This block defines or continues the implementation of `max_size`.
- CN: 该代码块定义或继续实现 `max_size`。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 463-463
```cpp
 463:   std::mutex mutex;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 465-476
```cpp
 465: private:
 466:   // Only sets size and does value check. Does not resize the data structures.
 467:   void _set_max_size(int64_t new_size) {
 468:     // We check that 0 <= new_size <= CUFFT_MAX_PLAN_NUM here. Since
 469:     // CUFFT_MAX_PLAN_NUM is of type size_t, we need to do non-negativity check
 470:     // first.
 471:     TORCH_CHECK(new_size >= 0,
 472:              "cuFFT plan cache size must be non-negative, but got ", new_size);
 473:     TORCH_CHECK(new_size <= CUFFT_MAX_PLAN_NUM,
 474:              "cuFFT plan cache size can not be larger than ", CUFFT_MAX_PLAN_NUM, ", but got ", new_size);
 475:     _max_size = static_cast<size_t>(new_size);
 476:   }
```
- EN: This block defines or continues the implementation of `_set_max_size`.
- CN: 该代码块定义或继续实现 `_set_max_size`。
- EN: `TORCH_CHECK` guards runtime assumptions before device-side work begins.
- CN: `TORCH_CHECK` 会在设备端计算开始前保护关键运行时假设。
- EN: Shape, stride, or offset handling here maps logical tensor coordinates onto concrete memory locations.
- CN: 这里对 shape、stride 或 offset 的处理负责把逻辑张量坐标映射到实际内存位置。

### Lines 478-481
```cpp
 478:   std::list<kv_t> _usage_list;
 479:   map_t _cache_map;
 480:   size_t _max_size;
 481: };
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 483-488
```cpp
 483: // Since ATen is separated into CPU build and CUDA build, we need a way to call
 484: // these functions only when CUDA is loaded. We use CUDA hooks for this purpose
 485: // (at cuda/detail/CUDAHooks.cpp), and call the hooked functions from the actual
 486: // native function counterparts (at native/SpectralOps.cpp), i.e.,
 487: // _cufft_get_plan_cache_max_size, _cufft_set_plan_cache_max_size
 488: // _cufft_get_plan_cache_size, and _cufft_clear_plan_cache.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 489-492
```cpp
 489: int64_t cufft_get_plan_cache_max_size_impl(DeviceIndex device_index);
 490: void cufft_set_plan_cache_max_size_impl(DeviceIndex device_index, int64_t max_size);
 491: int64_t cufft_get_plan_cache_size_impl(DeviceIndex device_index);
 492: void cufft_clear_plan_cache_impl(DeviceIndex device_index);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 494-494
```cpp
 494: } // namespace at::native::detail
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。

## Key Concepts / 关键概念

- `TORCH_CHECK` validates runtime assumptions before launching device work. / `TORCH_CHECK` 在启动设备端计算前校验运行时条件。
- `at::cuda` helpers expose streams, launch configuration, and low-level CUDA runtime glue. / `at::cuda` 辅助工具提供流、启动配置和底层 CUDA 运行时胶水代码。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Config.h>`
  - `<ATen/core/DimVector.h>`
  - `<ATen/cuda/CUDAContext.h>`
  - `<ATen/native/cuda/CuFFTUtils.h>`
  - `<ATen/native/utils/ParamsHash.h>`
  - `<c10/util/accumulate.h>`
  - `<c10/util/irange.h>`
  - `<cufft.h>`
  - `<cufftXt.h>`
  - `<limits>`
  - `<list>`
  - `<sstream>`
- Runtime symbols / 运行时符号:
  - `at::cuda::getCurrentDeviceProperties`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
