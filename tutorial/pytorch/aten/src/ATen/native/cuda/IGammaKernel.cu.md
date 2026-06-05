# IGammaKernel.cu — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/cuda/IGammaKernel.cu`
- Repository: `pytorch/pytorch` (local path: `/root/xw/pytorch`)
- Purpose (EN): Implements CUDA/native kernel logic related to `ratevl`, `lanczos_sum_expg_scaled`, `_igam_helper_fac`, `_igam_helper_series`.
- 用途（中文）: 实现与 `ratevl`, `lanczos_sum_expg_scaled`, `_igam_helper_fac`, `_igam_helper_series` 相关的 CUDA / 原生内核逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
   1: #define TORCH_ASSERT_NO_OPERATORS
   2: #include <ATen/Dispatch.h>
   3: #include <ATen/native/DispatchStub.h>
   4: #include <ATen/native/cuda/Loops.cuh>
   5: #include <ATen/native/cuda/Math.cuh>
   6: #include <ATen/native/TensorIterator.h>
   7: #include <ATen/native/BinaryOps.h>
```
- EN: This include block pulls in the ATen/CUDA facilities needed later in the file, such as `<ATen/Dispatch.h>`, `<ATen/native/DispatchStub.h>`, `<ATen/native/cuda/Loops.cuh>`.
- CN: 这一组 `#include` 引入了后续实现所需的 ATen/CUDA 组件，例如 `<ATen/Dispatch.h>`, `<ATen/native/DispatchStub.h>`, `<ATen/native/cuda/Loops.cuh>`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

### Lines 9-14
```cpp
   9: // NOTE: CUDA on Windows requires that the enclosing function
  10: // of a __device__ lambda not have internal linkage.
  11: 
  12: // TODO: review jiterating igamma and igammac if/when a persistent (across processes)
  13: //   cache is implemented, because they take a VERY long time to compile
  14: // TODO: it's also odd these ops use gpu_kernel_with_scalars
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 16-36
```cpp
  16: namespace {
  17: 
  18: /*
  19:  * This implementation of the regularized incomplete gamma functions and
  20:  * their helper functions are derived from the implementation of SciPy's
  21:  * gammainc, Cephes's igam and igamc, and Boost's Lanczos approximations.
  22:  * See NOTICE for the licenses.
  23:  */
  24: // regularized lower & upper incomplete gamma
  25: template <typename scalar_t>
  26: __host__ __device__ scalar_t ratevl(scalar_t x, const scalar_t num[], int64_t M,
  27:     const scalar_t denom[], int64_t N) {
  28:   // evaluating rational function, i.e., the ratio of two polynomials
  29:   // the coefficients for numerator are given by `num` while coeffs for
  30:   // denumerator are given by `denom`
  31: 
  32:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
  33:   int64_t i, dir;
  34:   accscalar_t y, num_ans, denom_ans;
  35:   accscalar_t absx = ::fabs(x);
  36:   const accscalar_t *p;
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `ratevl`.
- CN: 该代码块定义或继续实现 `ratevl`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 38-48
```cpp
  38:   if (absx > 1) {
  39:     /* Evaluate as a polynomial in 1/x. */
  40:     dir = -1;
  41:     p = num + M;
  42:     y = 1 / x;
  43:   }
  44:   else {
  45:     dir = 1;
  46:     p = num;
  47:     y = x;
  48:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 50-50
```cpp
  50:   /* Evaluate the numerator */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 51-63
```cpp
  51:   num_ans = *p;
  52:   p += dir;
  53:   for (i = 1; i <= M; i++) {
  54:     num_ans = num_ans * y + *p;
  55:     p += dir;
  56:   }
  57:   /* Evaluate the denominator */
  58:   if (absx > 1) {
  59:     p = denom + N;
  60:   }
  61:   else {
  62:     p = denom;
  63:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 65-78
```cpp
  65:   denom_ans = *p;
  66:   p += dir;
  67:   for (i = 1; i <= N; i++) {
  68:     denom_ans = denom_ans * y + *p;
  69:     p += dir;
  70:   }
  71:   if (absx > 1) {
  72:     i = N - M;
  73:     return ::pow(x, static_cast<accscalar_t>(i)) * num_ans / denom_ans;
  74:   }
  75:   else {
  76:     return num_ans / denom_ans;
  77:   }
  78: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 80-101
```cpp
  80: template <typename scalar_t>
  81: __host__ __device__ scalar_t lanczos_sum_expg_scaled(scalar_t x) {
  82:   // lanczos approximation
  83:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
  84: 
  85:   constexpr accscalar_t lanczos_sum_expg_scaled_num[13] = {
  86:     0.006061842346248906525783753964555936883222,
  87:     0.5098416655656676188125178644804694509993,
  88:     19.51992788247617482847860966235652136208,
  89:     449.9445569063168119446858607650988409623,
  90:     6955.999602515376140356310115515198987526,
  91:     75999.29304014542649875303443598909137092,
  92:     601859.6171681098786670226533699352302507,
  93:     3481712.15498064590882071018964774556468,
  94:     14605578.08768506808414169982791359218571,
  95:     43338889.32467613834773723740590533316085,
  96:     86363131.28813859145546927288977868422342,
  97:     103794043.1163445451906271053616070238554,
  98:     56906521.91347156388090791033559122686859
  99:   };
 100:   constexpr accscalar_t lanczos_sum_expg_scaled_denom[13] = {
 101:     1.,
```
- EN: This block defines or continues the implementation of `lanczos_sum_expg_scaled`.
- CN: 该代码块定义或继续实现 `lanczos_sum_expg_scaled`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 102-115
```cpp
 102:     66.,
 103:     1925.,
 104:     32670.,
 105:     357423.,
 106:     2637558.,
 107:     13339535.,
 108:     45995730.,
 109:     105258076.,
 110:     150917976.,
 111:     120543840.,
 112:     39916800.,
 113:     0
 114:   };
 115:   return ratevl(static_cast<accscalar_t>(x), lanczos_sum_expg_scaled_num,
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 116-119
```cpp
 116:       sizeof(lanczos_sum_expg_scaled_num) / sizeof(lanczos_sum_expg_scaled_num[0]) - 1,
 117:       lanczos_sum_expg_scaled_denom,
 118:       sizeof(lanczos_sum_expg_scaled_denom) / sizeof(lanczos_sum_expg_scaled_denom[0]) - 1);
 119: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 121-142
```cpp
 121: template <typename scalar_t>
 122: __host__ __device__ scalar_t _igam_helper_fac(scalar_t a, scalar_t x) {
 123:   // compute x^a * exp(-a) / gamma(a)
 124:   // corrected from (15) and (16) in [igam2] by replacing exp(x - a) with
 125:   // exp(a - x).
 126: 
 127:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
 128:   accscalar_t ax, fac, res, num, numfac;
 129:   constexpr accscalar_t MAXLOG = std::is_same_v<accscalar_t,double> ?
 130:     7.09782712893383996843E2 : 88.72283905206835;
 131:   constexpr accscalar_t EXP1 = 2.718281828459045;
 132:   constexpr accscalar_t lanczos_g = 6.024680040776729583740234375;
 133: 
 134:   if (::fabs(a - x) > 0.4 * ::fabs(a)) {
 135:     ax = a * ::log(x) - x - ::lgamma(a);
 136:     if (ax < -MAXLOG) {
 137:       return 0.0;
 138:     }
 139:     return ::exp(ax);
 140:   }
 141: 
 142:   fac = a + lanczos_g - 0.5;
```
- EN: This block defines or continues the implementation of `_igam_helper_fac`.
- CN: 该代码块定义或继续实现 `_igam_helper_fac`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 143-143
```cpp
 143:   res = ::sqrt(fac / EXP1) / lanczos_sum_expg_scaled(a);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 145-154
```cpp
 145:   if ((a < 200) && (x < 200)) {
 146:     res *= ::exp(a - x) * ::pow(x / fac, a);
 147:   }
 148:   else {
 149:     num = x - a - lanczos_g + 0.5;
 150:     numfac = num / fac;
 151:     res *= ::exp(a * (::log1p(numfac) - numfac) + x * (0.5 - lanczos_g) / fac);
 152:   }
 153:   return res;
 154: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 156-176
```cpp
 156: template <typename scalar_t>
 157: __host__ __device__ scalar_t _igam_helper_series(scalar_t a, scalar_t x) {
 158:   // Compute igam using DLMF 8.11.4. [igam1]
 159: 
 160:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
 161:   constexpr accscalar_t MACHEP = std::is_same_v<accscalar_t, double> ?
 162:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
 163:   constexpr int MAXITER = 2000;
 164: 
 165:   int i;
 166:   accscalar_t ans, ax, c, r;
 167: 
 168:   ax = _igam_helper_fac(a, x);
 169:   if (ax == 0.0) {
 170:     return 0.0;
 171:   }
 172: 
 173:   /* power series */
 174:   r = a;
 175:   c = 1.0;
 176:   ans = 1.0;
```
- EN: This block defines or continues the implementation of `_igam_helper_series`.
- CN: 该代码块定义或继续实现 `_igam_helper_series`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 178-187
```cpp
 178:   for (i = 0; i < MAXITER; i++) {
 179:     r += 1.0;
 180:     c *= x / r;
 181:     ans += c;
 182:     if (c <= MACHEP * ans) {
 183:       break;
 184:     }
 185:   }
 186:   return (ans * ax / a);
 187: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 189-210
```cpp
 189: template <typename scalar_t>
 190: __host__ __device__ scalar_t _igamc_helper_series(scalar_t a, scalar_t x) {
 191:   // Compute igamc using DLMF 8.7.3 [igam1]. This is related to the series in
 192:   // _igam_helper_series but extra care is taken to avoid cancellation.
 193: 
 194:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
 195:   int n;
 196:   accscalar_t fac = 1;
 197:   accscalar_t sum = 0;
 198:   accscalar_t term, logx;
 199:   constexpr int MAXITER = 2000;
 200:   constexpr accscalar_t MACHEP = std::is_same_v<accscalar_t, double> ?
 201:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
 202: 
 203:   for (n = 1; n < MAXITER; n++) {
 204:     fac *= -x / n;
 205:     term = fac / (a + n);
 206:     sum += term;
 207:     if (::fabs(term) <= MACHEP * ::fabs(sum)) {
 208:         break;
 209:     }
 210:   }
```
- EN: This block defines or continues the implementation of `_igamc_helper_series`.
- CN: 该代码块定义或继续实现 `_igamc_helper_series`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 212-215
```cpp
 212:   logx = ::log(x);
 213:   term = -::expm1(a * logx - ::lgamma(1+a));
 214:   return term - ::exp(a * logx - ::lgamma(a)) * sum;
 215: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 217-238
```cpp
 217: template <typename scalar_t>
 218: __host__ __device__ scalar_t _igam_helper_asymptotic_series(scalar_t a, scalar_t x, bool igam) {
 219:   // Compute igam/igamc using DLMF 8.12.3/8.12.4 [igam1]
 220: 
 221:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
 222:   constexpr accscalar_t d[25][25] =
 223:     {{-3.3333333333333333e-1, 8.3333333333333333e-2, -1.4814814814814815e-2, 1.1574074074074074e-3, 3.527336860670194e-4, -1.7875514403292181e-4, 3.9192631785224378e-5, -2.1854485106799922e-6, -1.85406221071516e-6, 8.296711340953086e-7, -1.7665952736826079e-7, 6.7078535434014986e-9, 1.0261809784240308e-8, -4.3820360184533532e-9, 9.1476995822367902e-10, -2.551419399494625e-11, -5.8307721325504251e-11, 2.4361948020667416e-11, -5.0276692801141756e-12, 1.1004392031956135e-13, 3.3717632624009854e-13, -1.3923887224181621e-13, 2.8534893807047443e-14, -5.1391118342425726e-16, -1.9752288294349443e-15},
 224:     {-1.8518518518518519e-3, -3.4722222222222222e-3, 2.6455026455026455e-3, -9.9022633744855967e-4, 2.0576131687242798e-4, -4.0187757201646091e-7, -1.8098550334489978e-5, 7.6491609160811101e-6, -1.6120900894563446e-6, 4.6471278028074343e-9, 1.378633446915721e-7, -5.752545603517705e-8, 1.1951628599778147e-8, -1.7543241719747648e-11, -1.0091543710600413e-9, 4.1627929918425826e-10, -8.5639070264929806e-11, 6.0672151016047586e-14, 7.1624989648114854e-12, -2.9331866437714371e-12, 5.9966963656836887e-13, -2.1671786527323314e-16, -4.9783399723692616e-14, 2.0291628823713425e-14, -4.13125571381061e-15},
 225:     {4.1335978835978836e-3, -2.6813271604938272e-3, 7.7160493827160494e-4, 2.0093878600823045e-6, -1.0736653226365161e-4, 5.2923448829120125e-5, -1.2760635188618728e-5, 3.4235787340961381e-8, 1.3721957309062933e-6, -6.298992138380055e-7, 1.4280614206064242e-7, -2.0477098421990866e-10, -1.4092529910867521e-8, 6.228974084922022e-9, -1.3670488396617113e-9, 9.4283561590146782e-13, 1.2872252400089318e-10, -5.5645956134363321e-11, 1.1975935546366981e-11, -4.1689782251838635e-15, -1.0940640427884594e-12, 4.6622399463901357e-13, -9.905105763906906e-14, 1.8931876768373515e-17, 8.8592218725911273e-15},
 226:     {6.4943415637860082e-4, 2.2947209362139918e-4, -4.6918949439525571e-4, 2.6772063206283885e-4, -7.5618016718839764e-5, -2.3965051138672967e-7, 1.1082654115347302e-5, -5.6749528269915966e-6, 1.4230900732435884e-6, -2.7861080291528142e-11, -1.6958404091930277e-7, 8.0994649053880824e-8, -1.9111168485973654e-8, 2.3928620439808118e-12, 2.0620131815488798e-9, -9.4604966618551322e-10, 2.1541049775774908e-10, -1.388823336813903e-14, -2.1894761681963939e-11, 9.7909989511716851e-12, -2.1782191880180962e-12, 6.2088195734079014e-17, 2.126978363279737e-13, -9.3446887915174333e-14, 2.0453671226782849e-14},
 227:     {-8.618882909167117e-4, 7.8403922172006663e-4, -2.9907248030319018e-4, -1.4638452578843418e-6, 6.6414982154651222e-5, -3.9683650471794347e-5, 1.1375726970678419e-5, 2.5074972262375328e-10, -1.6954149536558306e-6, 8.9075075322053097e-7, -2.2929348340008049e-7, 2.956794137544049e-11, 2.8865829742708784e-8, -1.4189739437803219e-8, 3.4463580499464897e-9, -2.3024517174528067e-13, -3.9409233028046405e-10, 1.8602338968504502e-10, -4.356323005056618e-11, 1.2786001016296231e-15, 4.6792750266579195e-12, -2.1492464706134829e-12, 4.9088156148096522e-13, -6.3385914848915603e-18, -5.0453320690800944e-14},
 228:     {-3.3679855336635815e-4, -6.9728137583658578e-5, 2.7727532449593921e-4, -1.9932570516188848e-4, 6.7977804779372078e-5, 1.419062920643967e-7, -1.3594048189768693e-5, 8.0184702563342015e-6, -2.2914811765080952e-6, -3.252473551298454e-10, 3.4652846491085265e-7, -1.8447187191171343e-7, 4.8240967037894181e-8, -1.7989466721743515e-14, -6.3061945000135234e-9, 3.1624176287745679e-9, -7.8409242536974293e-10, 5.1926791652540407e-15, 9.3589442423067836e-11, -4.5134262161632782e-11, 1.0799129993116827e-11, -3.661886712685252e-17, -1.210902069055155e-12, 5.6807435849905643e-13, -1.3249659916340829e-13},
 229:     {5.3130793646399222e-4, -5.9216643735369388e-4, 2.7087820967180448e-4, 7.9023532326603279e-7, -8.1539693675619688e-5, 5.6116827531062497e-5, -1.8329116582843376e-5, -3.0796134506033048e-9, 3.4651553688036091e-6, -2.0291327396058604e-6, 5.7887928631490037e-7, 2.338630673826657e-13, -8.8286007463304835e-8, 4.7435958880408128e-8, -1.2545415020710382e-8, 8.6496488580102925e-14, 1.6846058979264063e-9, -8.5754928235775947e-10, 2.1598224929232125e-10, -7.6132305204761539e-16, -2.6639822008536144e-11, 1.3065700536611057e-11, -3.1799163902367977e-12, 4.7109761213674315e-18, 3.6902800842763467e-13},
 230:     {3.4436760689237767e-4, 5.1717909082605922e-5, -3.3493161081142236e-4, 2.812695154763237e-4, -1.0976582244684731e-4, -1.2741009095484485e-7, 2.7744451511563644e-5, -1.8263488805711333e-5, 5.7876949497350524e-6, 4.9387589339362704e-10, -1.0595367014026043e-6, 6.1667143761104075e-7, -1.7562973359060462e-7, -1.2974473287015439e-12, 2.695423606288966e-8, -1.4578352908731271e-8, 3.887645959386175e-9, -3.8810022510194121e-17, -5.3279941738772867e-10, 2.7437977643314845e-10, -6.9957960920705679e-11, 2.5899863874868481e-17, 8.8566890996696381e-12, -4.403168815871311e-12, 1.0865561947091654e-12},
 231:     {-6.5262391859530942e-4, 8.3949872067208728e-4, -4.3829709854172101e-4, -6.969091458420552e-7, 1.6644846642067548e-4, -1.2783517679769219e-4, 4.6299532636913043e-5, 4.5579098679227077e-9, -1.0595271125805195e-5, 6.7833429048651666e-6, -2.1075476666258804e-6, -1.7213731432817145e-11, 3.7735877416110979e-7, -2.1867506700122867e-7, 6.2202288040189269e-8, 6.5977038267330006e-16, -9.5903864974256858e-9, 5.2132144922808078e-9, -1.3991589583935709e-9, 5.382058999060575e-16, 1.9484714275467745e-10, -1.0127287556389682e-10, 2.6077347197254926e-11, -5.0904186999932993e-18, -3.3721464474854592e-12},
 232:     {-5.9676129019274625e-4, -7.2048954160200106e-5, 6.7823088376673284e-4, -6.4014752602627585e-4, 2.7750107634328704e-4, 1.8197008380465151e-7, -8.4795071170685032e-5, 6.105192082501531e-5, -2.1073920183404862e-5, -8.8585890141255994e-10, 4.5284535953805377e-6, -2.8427815022504408e-6, 8.7082341778646412e-7, 3.6886101871706965e-12, -1.5344695190702061e-7, 8.862466778790695e-8, -2.5184812301826817e-8, -1.0225912098215092e-14, 3.8969470758154777e-9, -2.1267304792235635e-9, 5.7370135528051385e-10, -1.887749850169741e-19, -8.0931538694657866e-11, 4.2382723283449199e-11, -1.1002224534207726e-11},
 233:     {1.3324454494800656e-3, -1.9144384985654775e-3, 1.1089369134596637e-3, 9.932404122642299e-7, -5.0874501293093199e-4, 4.2735056665392884e-4, -1.6858853767910799e-4, -8.1301893922784998e-9, 4.5284402370562147e-5, -3.127053674781734e-5, 1.044986828530338e-5, 4.8435226265680926e-11, -2.1482565873456258e-6, 1.329369701097492e-6, -4.0295693092101029e-7, -1.7567877666323291e-13, 7.0145043163668257e-8, -4.040787734999483e-8, 1.1474026743371963e-8, 3.9642746853563325e-18, -1.7804938269892714e-9, 9.7480262548731646e-10, -2.6405338676507616e-10, 5.794875163403742e-18, 3.7647749553543836e-11},
 234:     {1.579727660730835e-3, 1.6251626278391582e-4, -2.0633421035543276e-3, 2.1389686185689098e-3, -1.0108559391263003e-3, -3.9912705529919201e-7, 3.6235025084764691e-4, -2.8143901463712154e-4, 1.0449513336495887e-4, 2.1211418491830297e-9, -2.5779417251947842e-5, 1.7281818956040463e-5, -5.6413773872904282e-6, -1.1024320105776174e-11, 1.1223224418895175e-6, -6.8693396379526735e-7, 2.0653236975414887e-7, 4.6714772409838506e-14, -3.5609886164949055e-8, 2.0470855345905963e-8, -5.8091738633283358e-9, -1.332821287582869e-16, 9.0354604391335133e-10, -4.9598782517330834e-10, 1.3481607129399749e-10},
 235:     {-4.0725121195140166e-3, 6.4033628338080698e-3, -4.0410161081676618e-3, -2.183732802866233e-6, 2.1740441801254639e-3, -1.9700440518418892e-3, 8.3595469747962458e-4, 1.9445447567109655e-8, -2.5779387120421696e-4, 1.9009987368139304e-4, -6.7696499937438965e-5, -1.4440629666426572e-10, 1.5712512518742269e-5, -1.0304008744776893e-5, 3.304517767401387e-6, 7.9829760242325709e-13, -6.4097794149313004e-7, 3.8894624761300056e-7, -1.1618347644948869e-7, -2.816808630596451e-15, 1.9878012911297093e-8, -1.1407719956357511e-8, 3.2355857064185555e-9, 4.1759468293455945e-20, -5.0423112718105824e-10},
 236:     {-5.9475779383993003e-3, -5.4016476789260452e-4, 8.7910413550767898e-3, -9.8576315587856125e-3, 5.0134695031021538e-3, 1.2807521786221875e-6, -2.0626019342754683e-3, 1.7109128573523058e-3, -6.7695312714133799e-4, -6.9011545676562133e-9, 1.8855128143995902e-4, -1.3395215663491969e-4, 4.6263183033528039e-5, 4.0034230613321351e-11, -1.0255652921494033e-5, 6.612086372797651e-6, -2.0913022027253008e-6, -2.0951775649603837e-13, 3.9756029041993247e-7, -2.3956211978815887e-7, 7.1182883382145864e-8, 8.925574873053455e-16, -1.2101547235064676e-8, 6.9350618248334386e-9, -1.9661464453856102e-9},
 237:     {1.7402027787522711e-2, -2.9527880945699121e-2, 2.0045875571402799e-2, 7.0289515966903407e-6, -1.2375421071343148e-2, 1.1976293444235254e-2, -5.4156038466518525e-3, -6.3290893396418616e-8, 1.8855118129005065e-3, -1.473473274825001e-3, 5.5515810097708387e-4, 5.2406834412550662e-10, -1.4357913535784836e-4, 9.9181293224943297e-5, -3.3460834749478311e-5, -3.5755837291098993e-12, 7.1560851960630076e-6, -4.5516802628155526e-6, 1.4236576649271475e-6, 1.8803149082089664e-14, -2.6623403898929211e-7, 1.5950642189595716e-7, -4.7187514673841102e-8, -6.5107872958755177e-17, 7.9795091026746235e-9},
 238:     {3.0249124160905891e-2, 2.4817436002649977e-3, -4.9939134373457022e-2, 5.9915643009307869e-2, -3.2483207601623391e-2, -5.7212968652103441e-6, 1.5085251778569354e-2, -1.3261324005088445e-2, 5.5515262632426148e-3, 3.0263182257030016e-8, -1.7229548406756723e-3, 1.2893570099929637e-3, -4.6845138348319876e-4, -1.830259937893045e-10, 1.1449739014822654e-4, -7.7378565221244477e-5, 2.5625836246985201e-5, 1.0766165333192814e-12, -5.3246809282422621e-6, 3.349634863064464e-6, -1.0381253128684018e-6, -5.608909920621128e-15, 1.9150821930676591e-7, -1.1418365800203486e-7, 3.3654425209171788e-8},
```
- EN: This block defines or continues the implementation of `_igam_helper_asymptotic_series`.
- CN: 该代码块定义或继续实现 `_igam_helper_asymptotic_series`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 239-247
```cpp
 239:     {-9.9051020880159045e-2, 1.7954011706123486e-1, -1.2989606383463778e-1, -3.1478872752284357e-5, 9.0510635276848131e-2, -9.2828824411184397e-2, 4.4412112839877808e-2, 2.7779236316835888e-7, -1.7229543805449697e-2, 1.4182925050891573e-2, -5.6214161633747336e-3, -2.39598509186381e-9, 1.6029634366079908e-3, -1.1606784674435773e-3, 4.1001337768153873e-4, 1.8365800754090661e-11, -9.5844256563655903e-5, 6.3643062337764708e-5, -2.076250624489065e-5, -1.1806020912804483e-13, 4.2131808239120649e-6, -2.6262241337012467e-6, 8.0770620494930662e-7, 6.0125912123632725e-16, -1.4729737374018841e-7},
 240:     {-1.9994542198219728e-1, -1.5056113040026424e-2, 3.6470239469348489e-1, -4.6435192311733545e-1, 2.6640934719197893e-1, 3.4038266027147191e-5, -1.3784338709329624e-1, 1.276467178337056e-1, -5.6213828755200985e-2, -1.753150885483011e-7, 1.9235592956768113e-2, -1.5088821281095315e-2, 5.7401854451350123e-3, 1.0622382710310225e-9, -1.5335082692563998e-3, 1.0819320643228214e-3, -3.7372510193945659e-4, -6.6170909729031985e-12, 8.4263617380909628e-5, -5.5150706827483479e-5, 1.7769536448348069e-5, 3.8827923210205533e-14, -3.53513697488768e-6, 2.1865832130045269e-6, -6.6812849447625594e-7},
 241:     {7.2438608504029431e-1, -1.3918010932653375, 1.0654143352413968, 1.876173868950258e-4, -8.2705501176152696e-1, 8.9352433347828414e-1, -4.4971003995291339e-1, -1.6107401567546652e-6, 1.9235590165271091e-1, -1.6597702160042609e-1, 6.8882222681814333e-2, 1.3910091724608687e-8, -2.146911561508663e-2, 1.6228980898865892e-2, -5.9796016172584256e-3, -1.1287469112826745e-10, 1.5167451119784857e-3, -1.0478634293553899e-3, 3.5539072889126421e-4, 8.1704322111801517e-13, -7.7773013442452395e-5, 5.0291413897007722e-5, -1.6035083867000518e-5, 1.2469354315487605e-14, 3.1369106244517615e-6},
 242:     {1.6668949727276811, 1.165462765994632e-1, -3.3288393225018906, 4.4692325482864037, -2.6977693045875807, -2.600667859891061e-4, 1.5389017615694539, -1.4937962361134612, 6.8881964633233148e-1, 1.3077482004552385e-6, -2.5762963325596288e-1, 2.1097676102125449e-1, -8.3714408359219882e-2, -7.7920428881354753e-9, 2.4267923064833599e-2, -1.7813678334552311e-2, 6.3970330388900056e-3, 4.9430807090480523e-11, -1.5554602758465635e-3, 1.0561196919903214e-3, -3.5277184460472902e-4, 9.3002334645022459e-14, 7.5285855026557172e-5, -4.8186515569156351e-5, 1.5227271505597605e-5},
 243:     {-6.6188298861372935, 1.3397985455142589e+1, -1.0789350606845146e+1, -1.4352254537875018e-3, 9.2333694596189809, -1.0456552819547769e+1, 5.5105526029033471, 1.2024439690716742e-5, -2.5762961164755816, 2.3207442745387179, -1.0045728797216284, -1.0207833290021914e-7, 3.3975092171169466e-1, -2.6720517450757468e-1, 1.0235252851562706e-1, 8.4329730484871625e-10, -2.7998284958442595e-2, 2.0066274144976813e-2, -7.0554368915086242e-3, 1.9402238183698188e-12, 1.6562888105449611e-3, -1.1082898580743683e-3, 3.654545161310169e-4, -5.1290032026971794e-11, -7.6340103696869031e-5},
 244:     {-1.7112706061976095e+1, -1.1208044642899116, 3.7131966511885444e+1, -5.2298271025348962e+1, 3.3058589696624618e+1, 2.4791298976200222e-3, -2.061089403411526e+1, 2.088672775145582e+1, -1.0045703956517752e+1, -1.2238783449063012e-5, 4.0770134274221141, -3.473667358470195, 1.4329352617312006, 7.1359914411879712e-8, -4.4797257159115612e-1, 3.4112666080644461e-1, -1.2699786326594923e-1, -2.8953677269081528e-10, 3.3125776278259863e-2, -2.3274087021036101e-2, 8.0399993503648882e-3, -1.177805216235265e-9, -1.8321624891071668e-3, 1.2108282933588665e-3, -3.9479941246822517e-4},
 245:     {7.389033153567425e+1, -1.5680141270402273e+2, 1.322177542759164e+2, 1.3692876877324546e-2, -1.2366496885920151e+2, 1.4620689391062729e+2, -8.0365587724865346e+1, -1.1259851148881298e-4, 4.0770132196179938e+1, -3.8210340013273034e+1, 1.719522294277362e+1, 9.3519707955168356e-7, -6.2716159907747034, 5.1168999071852637, -2.0319658112299095, -4.9507215582761543e-9, 5.9626397294332597e-1, -4.4220765337238094e-1, 1.6079998700166273e-1, -2.4733786203223402e-8, -4.0307574759979762e-2, 2.7849050747097869e-2, -9.4751858992054221e-3, 6.419922235909132e-6, 2.1250180774699461e-3},
 246:     {2.1216837098382522e+2, 1.3107863022633868e+1, -4.9698285932871748e+2, 7.3121595266969204e+2, -4.8213821720890847e+2, -2.8817248692894889e-2, 3.2616720302947102e+2, -3.4389340280087117e+2, 1.7195193870816232e+2, 1.4038077378096158e-4, -7.52594195897599e+1, 6.651969984520934e+1, -2.8447519748152462e+1, -7.613702615875391e-7, 9.5402237105304373, -7.5175301113311376, 2.8943997568871961, -4.6612194999538201e-7, -8.0615149598794088e-1, 5.8483006570631029e-1, -2.0845408972964956e-1, 1.4765818959305817e-4, 5.1000433863753019e-2, -3.3066252141883665e-2, 1.5109265210467774e-2},
 247:     {-9.8959643098322368e+2, 2.1925555360905233e+3, -1.9283586782723356e+3, -1.5925738122215253e-1, 1.9569985945919857e+3, -2.4072514765081556e+3, 1.3756149959336496e+3, 1.2920735237496668e-3, -7.525941715948055e+2, 7.3171668742208716e+2, -3.4137023466220065e+2, -9.9857390260608043e-6, 1.3356313181291573e+2, -1.1276295161252794e+2, 4.6310396098204458e+1, -7.9237387133614756e-6, -1.4510726927018646e+1, 1.1111771248100563e+1, -4.1690817945270892, 3.1008219800117808e-3, 1.1220095449981468, -7.6052379926149916e-1, 3.6262236505085254e-1, 2.216867741940747e-1, 4.8683443692930507e-1}};
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 249-259
```cpp
 249:   int k, n, sgn;
 250:   int maxpow = 0;
 251:   constexpr accscalar_t MACHEP = std::is_same_v<accscalar_t, double> ?
 252:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
 253:   accscalar_t lambda = x / a;
 254:   accscalar_t sigma = (x - a) / a;
 255:   accscalar_t eta, res, ck, ckterm, term, absterm;
 256:   accscalar_t absoldterm = INFINITY;
 257:   accscalar_t etapow[25] = {1};
 258:   accscalar_t sum = 0;
 259:   accscalar_t afac = 1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 261-266
```cpp
 261:   if (igam) {
 262:     sgn = -1;
 263:   }
 264:   else {
 265:     sgn = 1;
 266:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 268-277
```cpp
 268:   if (lambda > 1) {
 269:     eta = ::sqrt(-2 * (::log1p(sigma) - sigma));
 270:   }
 271:   else if (lambda < 1) {
 272:     eta = -::sqrt(-2 * (::log1p(sigma) - sigma));
 273:   }
 274:   else {
 275:     eta = 0;
 276:   }
 277:   res = 0.5 * ::erfc(sgn * eta * ::sqrt(a / 2));
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 279-300
```cpp
 279:   for (k = 0; k < 25; k++) {
 280:     ck = d[k][0];
 281:     for (n = 1; n < 25; n++) {
 282:       if (n > maxpow) {
 283:         etapow[n] = eta * etapow[n-1];
 284:         maxpow += 1;
 285:       }
 286:       ckterm = d[k][n]*etapow[n];
 287:       ck += ckterm;
 288:       if (std::fabs(ckterm) < MACHEP * std::fabs(ck)) {
 289:         break;
 290:       }
 291:     }
 292:     term = ck * afac;
 293:     absterm = std::fabs(term);
 294:     if (absterm > absoldterm) {
 295:       break;
 296:     }
 297:     sum += term;
 298:     if (absterm < MACHEP * std::fabs(sum)) {
 299:       break;
 300:     }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 301-304
```cpp
 301:     absoldterm = absterm;
 302:     afac /= a;
 303:   }
 304:   res += sgn * ::exp(-0.5 * a * eta * eta) * sum / ::sqrt(2 * 3.1415926535 * a);
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 306-307
```cpp
 306:   return res;
 307: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 309-330
```cpp
 309: template <typename scalar_t>
 310: __host__ __device__ scalar_t _igamc_helper_continued_fraction(scalar_t a, scalar_t x) {
 311:   // Compute igamc using DLMF 8.9.2. [igam1]
 312: 
 313:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
 314:   int i;
 315:   accscalar_t ans, ax, c, yc, r, t, y, z;
 316:   accscalar_t pk, pkm1, pkm2, qk, qkm1, qkm2;
 317:   constexpr int MAXITER = 2000;
 318:   constexpr accscalar_t MACHEP = std::is_same_v<accscalar_t, double> ?
 319:     1.11022302462515654042E-16 : 5.9604644775390625E-8;
 320:   constexpr accscalar_t BIG = std::is_same_v<accscalar_t,double> ?
 321:     4.503599627370496e15 : 16777216.;
 322:   constexpr accscalar_t BIGINV = std::is_same_v<accscalar_t,double> ?
 323:     2.22044604925031308085e-16 : 5.9604644775390625E-8;
 324: 
 325:   ax = _igam_helper_fac(a, x);
 326:   if (ax == 0.0) {
 327:     return 0.0;
 328:   }
 329: 
 330:   /* continued fraction */
```
- EN: This block defines or continues the implementation of `_igamc_helper_continued_fraction`.
- CN: 该代码块定义或继续实现 `_igamc_helper_continued_fraction`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 331-338
```cpp
 331:   y = 1.0 - a;
 332:   z = x + y + 1.0;
 333:   c = 0.0;
 334:   pkm2 = 1.0;
 335:   qkm2 = x;
 336:   pkm1 = x + 1.0;
 337:   qkm1 = z * x;
 338:   ans = pkm1 / qkm1;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 340-361
```cpp
 340:   for (i = 0; i < MAXITER; i++) {
 341:     c += 1.0;
 342:     y += 1.0;
 343:     z += 2.0;
 344:     yc = y * c;
 345:     pk = pkm1 * z - pkm2 * yc;
 346:     qk = qkm1 * z - qkm2 * yc;
 347:     if (qk != 0) {
 348:       r = pk / qk;
 349:       t = ::fabs((ans - r) / r);
 350:       ans = r;
 351:     }
 352:     else {
 353:       t = 1.0;
 354:     }
 355:     pkm2 = pkm1;
 356:     pkm1 = pk;
 357:     qkm2 = qkm1;
 358:     qkm1 = qk;
 359:     if (::fabs(pk) > BIG) {
 360:       pkm2 *= BIGINV;
 361:       pkm1 *= BIGINV;
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 362-370
```cpp
 362:       qkm2 *= BIGINV;
 363:       qkm1 *= BIGINV;
 364:     }
 365:     if (t <= MACHEP) {
 366:       break;
 367:     }
 368:   }
 369:   return ans * ax;
 370: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 372-393
```cpp
 372: template <typename scalar_t>
 373: __noinline__ __host__ __device__ scalar_t calc_igammac(scalar_t a, scalar_t x) {
 374:   /* the calculation of the regularized upper incomplete gamma function
 375:    * is done differently based on the values of a and x:
 376:    * - if x and/or a is at the boundary of defined region, then assign the
 377:    *   result at the boundary
 378:    * - if a is large and a ~ x, then using Uniform Asymptotic Expansions for
 379:    *   Large Parameter (see DLMF 8.12.4 [igam1])
 380:    * - if x > 1.1 and x < a, using the subtraction from the regularized lower
 381:    *   incomplete gamma
 382:    * - otherwise, calculate the series from [igam2] eq (5)
 383:    */
 384: 
 385:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
 386:   accscalar_t absxma_a;
 387: 
 388:   constexpr accscalar_t SMALL = 20.0;
 389:   constexpr accscalar_t LARGE = 200.0;
 390:   constexpr accscalar_t SMALLRATIO = 0.3;
 391:   constexpr accscalar_t LARGERATIO = 4.5;
 392: 
 393:   if ((x < 0) || (a < 0)) {
```
- EN: This block defines or continues the implementation of `calc_igammac`.
- CN: 该代码块定义或继续实现 `calc_igammac`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 394-394
```cpp
 394:     // out of defined-region of the function
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 395-408
```cpp
 395:     return std::numeric_limits<accscalar_t>::quiet_NaN();
 396:   }
 397:   else if (a == 0) {
 398:     if (x > 0) {
 399:       return 0.0;
 400:     }
 401:     else {
 402:       return std::numeric_limits<accscalar_t>::quiet_NaN();
 403:     }
 404:   }
 405:   else if (x == 0) {
 406:     return 1.0;
 407:   }
 408:   else if (::isinf(static_cast<accscalar_t>(a))) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 409-416
```cpp
 409:     if (::isinf(static_cast<accscalar_t>(x))) {
 410:       return std::numeric_limits<accscalar_t>::quiet_NaN();
 411:     }
 412:     return 1.0;
 413:   }
 414:   else if (::isinf(static_cast<accscalar_t>(x))) {
 415:     return 0.0;
 416:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 418-424
```cpp
 418:   absxma_a = ::fabs(x - a) / a;
 419:   if ((a > SMALL) && (a < LARGE) && (absxma_a < SMALLRATIO)) {
 420:      return _igam_helper_asymptotic_series(a, x, 0);
 421:   }
 422:   else if ((a > LARGE) && (absxma_a < LARGERATIO / ::sqrt(a))) {
 423:      return _igam_helper_asymptotic_series(a, x, 0);
 424:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 426-441
```cpp
 426:   if (x > 1.1) {
 427:     if (x < a) {
 428:       return 1.0 - _igam_helper_series(a, x);
 429:     }
 430:     else {
 431:       return _igamc_helper_continued_fraction(a, x);
 432:     }
 433:   }
 434:   else if (x <= 0.5) {
 435:     if (-0.4 / ::log(x) < a) {
 436:       return 1.0 - _igam_helper_series(a, x);
 437:     }
 438:     else {
 439:       return _igamc_helper_series(a, x);
 440:     }
 441:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 442-450
```cpp
 442:   else {
 443:     if (x * 1.1 < a) {
 444:       return 1.0 - _igam_helper_series(a, x);
 445:     }
 446:     else {
 447:       return _igamc_helper_series(a, x);
 448:     }
 449:   }
 450: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 452-454
```cpp
 452: // NOTE: this __noinline__ is important -- otherwise, observed compile times significantly
 453: // increase.  The same kernel seems to get recompiled multiple times via gpu_kernel_with_scalars,
 454: // multiple dtypes, etc.
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。
- EN: This range launches or wires up CUDA-side computation, connecting prepared state to executable GPU work.
- CN: 这一段代码负责启动或衔接 CUDA 侧计算，把已准备好的状态连接到可执行的 GPU 工作。

### Lines 455-476
```cpp
 455: template <typename scalar_t>
 456: __noinline__ __host__ __device__ scalar_t calc_igamma(scalar_t a, scalar_t x) {
 457:   /* the calculation of the regularized lower incomplete gamma function
 458:    * is done differently based on the values of a and x:
 459:    * - if x and/or a is at the boundary of defined region, then assign the
 460:    *   result at the boundary
 461:    * - if a is large and a ~ x, then using Uniform Asymptotic Expansions for
 462:    *   Large Parameter (see DLMF 8.12.3 [igam1])
 463:    * - if x > 1 and x > a, using the subtraction from the regularized upper
 464:    *   incomplete gamma
 465:    * - otherwise, calculate the series from [igam2] eq (4)
 466:    */
 467: 
 468:   using accscalar_t = at::acc_type<scalar_t, /*is_cuda=*/true>;
 469:   accscalar_t absxma_a;
 470:   constexpr accscalar_t SMALL = 20.0;
 471:   constexpr accscalar_t LARGE = 200.0;
 472:   constexpr accscalar_t SMALLRATIO = 0.3;
 473:   constexpr accscalar_t LARGERATIO = 4.5;
 474: 
 475:   // boundary values following SciPy
 476:   if ((x < 0) || (a < 0)) {
```
- EN: This block defines or continues the implementation of `calc_igamma`.
- CN: 该代码块定义或继续实现 `calc_igamma`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 477-477
```cpp
 477:     // out of defined-region of the function
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 478-491
```cpp
 478:     return std::numeric_limits<accscalar_t>::quiet_NaN();
 479:   }
 480:   else if (a == 0) {
 481:     if (x > 0) {
 482:       return 1.0;
 483:     }
 484:     else {
 485:       return std::numeric_limits<accscalar_t>::quiet_NaN();
 486:     }
 487:   }
 488:   else if (x == 0) {
 489:     return 0.0; // zero integration limit
 490:   }
 491:   else if (::isinf(static_cast<accscalar_t>(a))) {
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 492-499
```cpp
 492:     if (::isinf(static_cast<accscalar_t>(x))) {
 493:       return std::numeric_limits<accscalar_t>::quiet_NaN();
 494:     }
 495:     return 0.0;
 496:   }
 497:   else if (::isinf(static_cast<accscalar_t>(x))) {
 498:     return 1.0;
 499:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 501-501
```cpp
 501:   /* Asymptotic regime where a ~ x. */
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 502-508
```cpp
 502:   absxma_a = ::fabs(x - a) / a;
 503:   if ((a > SMALL) && (a < LARGE) && (absxma_a < SMALLRATIO)) {
 504:     return _igam_helper_asymptotic_series(a, x, 1);
 505:   }
 506:   else if ((a > LARGE) && (absxma_a < LARGERATIO / ::sqrt(a))) {
 507:     return _igam_helper_asymptotic_series(a, x, 1);
 508:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 510-512
```cpp
 510:   if ((x > 1.0) && (x > a)) {
 511:     return 1.0 - calc_igammac(a, x);
 512:   }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 514-515
```cpp
 514:   return _igam_helper_series(a, x);
 515: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 517-528
```cpp
 517: template<typename scalar_t>
 518: struct CalcIgamma{
 519:   CalcIgamma(bool calc_igammac): calc_igammac_(calc_igammac){}
 520:   bool calc_igammac_;
 521:   __device__ scalar_t operator() (scalar_t a, scalar_t b) const {
 522:     if (calc_igammac_) {
 523:       return calc_igammac(a,b);
 524:     } else {
 525:       return calc_igamma(a,b);
 526:     }
 527:   }
 528: };
```
- EN: This block defines or continues the implementation of `CalcIgamma`.
- CN: 该代码块定义或继续实现 `CalcIgamma`。
- EN: CUDA qualifiers and loop helpers show that this logic is meant to execute massively in parallel on the GPU.
- CN: CUDA 限定符和循环辅助宏表明这部分逻辑会在 GPU 上以大规模并行方式执行。

### Lines 530-530
```cpp
 530: }
```
- EN: This range advances the local control flow and data plumbing required by the surrounding CUDA implementation.
- CN: 这一段推进了周围 CUDA 实现所需的局部控制流和数据传递。

### Lines 532-532
```cpp
 532: // end of regularized lower & upper incomplete gamma
```
- EN: This comment block marks a logical section boundary and documents the code that follows.
- CN: 这一注释块标记了逻辑分段，并说明了后续代码。

### Lines 534-554
```cpp
 534: namespace at::native {
 535: 
 536: void igamma_kernel_cuda(TensorIteratorBase& iter) {
 537:   AT_DISPATCH_FLOATING_TYPES(iter.common_dtype(), "igamma_cuda", [&]() {
 538:     gpu_kernel(iter, CalcIgamma<scalar_t>(false));
 539:   });
 540: }
 541: 
 542: void igammac_kernel_cuda(TensorIteratorBase& iter) {
 543:   AT_DISPATCH_FLOATING_TYPES(iter.common_dtype(), "igammac_cuda", [&]() {
 544:     gpu_kernel(iter, CalcIgamma<scalar_t>(true));
 545:   });
 546: }
 547: 
 548: REGISTER_DISPATCH(igamma_stub, &igamma_kernel_cuda)
 549: REGISTER_DISPATCH(igammac_stub, &igammac_kernel_cuda)
 550: 
 551: // DO NOT ADD ANY NEW KERNELS HERE
 552: // CUDA compilation times grow quickly.  It's perfectly acceptable to have a file per kernel.
 553: 
 554: } // namespace at::native
```
- EN: The namespace statements keep the symbols inside PyTorch's ATen native CUDA implementation layers.
- CN: 命名空间语句把这些符号限定在 PyTorch 的 ATen 原生 CUDA 实现层中。
- EN: This block defines or continues the implementation of `igamma_kernel_cuda`, `igammac_kernel_cuda`.
- CN: 该代码块定义或继续实现 `igamma_kernel_cuda`, `igammac_kernel_cuda`。
- EN: `TensorIterator` is used here to normalize shapes, strides, and elementwise traversal before CUDA execution.
- CN: 这里使用 `TensorIterator` 在 CUDA 执行前统一处理形状、步长和逐元素遍历。

## Key Concepts / 关键概念

- `TensorIterator` normalizes tensor shapes/strides and drives elementwise CUDA traversal. / `TensorIterator` 统一张量形状与步长，并驱动逐元素 CUDA 遍历。
- `AT_DISPATCH` expands one implementation across multiple scalar types. / `AT_DISPATCH` 会把同一实现扩展到多种标量类型。
- `gpu_kernel` applies a functor over iterator elements on CUDA. / `gpu_kernel` 在 CUDA 上对迭代器元素应用函数对象。
- `gpu_kernel_with_scalars` handles elementwise CUDA work while folding scalar operands efficiently. / `gpu_kernel_with_scalars` 在高效折叠标量操作数的同时执行逐元素 CUDA 计算。
- Dispatch registration exposes the CUDA specialization to higher PyTorch layers. / 分发注册把 CUDA 特化实现暴露给更高层的 PyTorch 调用链。
- `__device__` marks helpers callable from device code. / `__device__` 表示可由设备端代码调用的辅助函数。

## Dependencies / 依赖关系

- Headers / 头文件:
  - `<ATen/Dispatch.h>`
  - `<ATen/native/DispatchStub.h>`
  - `<ATen/native/cuda/Loops.cuh>`
  - `<ATen/native/cuda/Math.cuh>`
  - `<ATen/native/TensorIterator.h>`
  - `<ATen/native/BinaryOps.h>`
- Runtime symbols / 运行时符号:
  - `igamma_stub`
  - `igammac_stub`
  - `gpu_kernel_with_scalars`
  - `gpu_kernel`
  - `TensorIterator`
  - `TensorIteratorBase`
  - `AT_DISPATCH_FLOATING_TYPES`
  - `REGISTER_DISPATCH`
- EN: These symbols show which ATen, CUDA, or PyTorch dispatch facilities this file directly leans on.
- CN: 这些符号展示了本文件直接依赖的 ATen、CUDA 或 PyTorch 分发机制。
