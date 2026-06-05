# hypergeometric_pdf.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/distributions/detail/hypergeometric_pdf.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header implements the hypergeometric pdf distribution together with policy-aware statistical helpers.
- **作用（中文）**: 此头文件实现 hypergeometric pdf 分布，并提供带策略控制的统计辅助接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: // Copyright 2008 Gautam Sewani
   2: // Copyright 2008 John Maddock
   3: //
   4: // Use, modification and distribution are subject to the
   5: // Boost Software License, Version 1.0.
   6: // (See accompanying file LICENSE_1_0.txt
   7: // or copy at http://www.boost.org/LICENSE_1_0.txt)
   8: 
   9: #ifndef BOOST_MATH_DISTRIBUTIONS_DETAIL_HG_PDF_HPP
  10: #define BOOST_MATH_DISTRIBUTIONS_DETAIL_HG_PDF_HPP
  11: 
  12: #include <boost/math/constants/constants.hpp>
  13: #include <boost/math/special_functions/lanczos.hpp>
  14: #include <boost/math/special_functions/gamma.hpp>
  15: #include <boost/math/special_functions/pow.hpp>
  16: #include <boost/math/special_functions/prime.hpp>
  17: #include <boost/math/policies/error_handling.hpp>
  18: #include <algorithm>
~~~
- **EN:** This block imports dependencies such as boost/math/constants/constants.hpp, boost/math/special_functions/lanczos.hpp, boost/math/special_functions/gamma.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/constants/constants.hpp, boost/math/special_functions/lanczos.hpp, boost/math/special_functions/gamma.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <cstdint>
  20: 
  21: #ifdef BOOST_MATH_INSTRUMENT
  22: #include <typeinfo>
  23: #endif
  24: 
  25: namespace boost{ namespace math{ namespace detail{
  26: 
  27: template <class T, class Func>
  28: void bubble_down_one(T* first, T* last, Func f)
  29: {
  30:    using std::swap;
  31:    T* next = first;
  32:    ++next;
  33:    while((next != last) && (!f(*first, *next)))
  34:    {
  35:       swap(*first, *next);
  36:       ++first;
~~~
- **EN:** This block imports dependencies such as cstdint, typeinfo so the surrounding code can use external declarations. The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies.
- **CN:** 此代码块引入了 cstdint, typeinfo 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:       ++next;
  38:    }
  39: }
  40: 
  41: template <class T>
  42: struct sort_functor
  43: {
  44:    sort_functor(const T* exponents) : m_exponents(exponents){}
  45:    bool operator()(std::size_t i, std::size_t j)
  46:    {
  47:       return m_exponents[i] > m_exponents[j];
  48:    }
  49: private:
  50:    const T* m_exponents;
  51: };
  52: 
  53: template <class T, class Lanczos, class Policy>
  54: T hypergeometric_pdf_lanczos_imp(T /*dummy*/, std::uint64_t x, std::uint64_t r, std::uint64_t n, std::uint64_t N, const Lanczos&, const Policy&)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as sort_functor.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 sort_functor。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: {
  56:    BOOST_MATH_STD_USING
  57: 
  58:    BOOST_MATH_INSTRUMENT_FPU
  59:    BOOST_MATH_INSTRUMENT_VARIABLE(x);
  60:    BOOST_MATH_INSTRUMENT_VARIABLE(r);
  61:    BOOST_MATH_INSTRUMENT_VARIABLE(n);
  62:    BOOST_MATH_INSTRUMENT_VARIABLE(N);
  63:    BOOST_MATH_INSTRUMENT_VARIABLE(typeid(Lanczos).name());
  64: 
  65:    T bases[9] = {
  66:       T(n) + static_cast<T>(Lanczos::g()) + 0.5f,
  67:       T(r) + static_cast<T>(Lanczos::g()) + 0.5f,
  68:       T(N - n) + static_cast<T>(Lanczos::g()) + 0.5f,
  69:       T(N - r) + static_cast<T>(Lanczos::g()) + 0.5f,
  70:       1 / (T(N) + static_cast<T>(Lanczos::g()) + 0.5f),
  71:       1 / (T(x) + static_cast<T>(Lanczos::g()) + 0.5f),
  72:       1 / (T(n - x) + static_cast<T>(Lanczos::g()) + 0.5f),
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:       1 / (T(r - x) + static_cast<T>(Lanczos::g()) + 0.5f),
  74:       1 / (T(N - n - r + x) + static_cast<T>(Lanczos::g()) + 0.5f)
  75:    };
  76:    T exponents[9] = {
  77:       n + T(0.5f),
  78:       r + T(0.5f),
  79:       N - n + T(0.5f),
  80:       N - r + T(0.5f),
  81:       N + T(0.5f),
  82:       x + T(0.5f),
  83:       n - x + T(0.5f),
  84:       r - x + T(0.5f),
  85:       N - n - r + x + T(0.5f)
  86:    };
  87:    int base_e_factors[9] = {
  88:       -1, -1, -1, -1, 1, 1, 1, 1, 1
  89:    };
  90:    int sorted_indexes[9] = {
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:       0, 1, 2, 3, 4, 5, 6, 7, 8
  92:    };
  93: #ifdef BOOST_MATH_INSTRUMENT
  94:    BOOST_MATH_INSTRUMENT_FPU
  95:    for(unsigned i = 0; i < 9; ++i)
  96:    {
  97:       BOOST_MATH_INSTRUMENT_VARIABLE(i);
  98:       BOOST_MATH_INSTRUMENT_VARIABLE(bases[i]);
  99:       BOOST_MATH_INSTRUMENT_VARIABLE(exponents[i]);
 100:       BOOST_MATH_INSTRUMENT_VARIABLE(base_e_factors[i]);
 101:       BOOST_MATH_INSTRUMENT_VARIABLE(sorted_indexes[i]);
 102:    }
 103: #endif
 104:    std::sort(sorted_indexes, sorted_indexes + 9, sort_functor<T>(exponents));
 105: #ifdef BOOST_MATH_INSTRUMENT
 106:    BOOST_MATH_INSTRUMENT_FPU
 107:    for(unsigned i = 0; i < 9; ++i)
 108:    {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, std::sort. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, std::sort。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:       BOOST_MATH_INSTRUMENT_VARIABLE(i);
 110:       BOOST_MATH_INSTRUMENT_VARIABLE(bases[i]);
 111:       BOOST_MATH_INSTRUMENT_VARIABLE(exponents[i]);
 112:       BOOST_MATH_INSTRUMENT_VARIABLE(base_e_factors[i]);
 113:       BOOST_MATH_INSTRUMENT_VARIABLE(sorted_indexes[i]);
 114:    }
 115: #endif
 116: 
 117:    do{
 118:       exponents[sorted_indexes[0]] -= exponents[sorted_indexes[1]];
 119:       bases[sorted_indexes[1]] *= bases[sorted_indexes[0]];
 120:       if((bases[sorted_indexes[1]] < tools::min_value<T>()) && (exponents[sorted_indexes[1]] != 0))
 121:       {
 122:          return 0;
 123:       }
 124:       base_e_factors[sorted_indexes[1]] += base_e_factors[sorted_indexes[0]];
 125:       bubble_down_one(sorted_indexes, sorted_indexes + 9, sort_functor<T>(exponents));
 126: 
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, bubble_down_one. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, bubble_down_one。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127: #ifdef BOOST_MATH_INSTRUMENT
 128:       for(unsigned i = 0; i < 9; ++i)
 129:       {
 130:          BOOST_MATH_INSTRUMENT_VARIABLE(i);
 131:          BOOST_MATH_INSTRUMENT_VARIABLE(bases[i]);
 132:          BOOST_MATH_INSTRUMENT_VARIABLE(exponents[i]);
 133:          BOOST_MATH_INSTRUMENT_VARIABLE(base_e_factors[i]);
 134:          BOOST_MATH_INSTRUMENT_VARIABLE(sorted_indexes[i]);
 135:       }
 136: #endif
 137:    }while(exponents[sorted_indexes[1]] > 1);
 138: 
 139:    //
 140:    // Combine equal powers:
 141:    //
 142:    std::size_t j = 8;
 143:    while(exponents[sorted_indexes[j]] == 0) --j;
 144:    while(j)
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, while. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, while。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:    {
 146:       while(j && (exponents[sorted_indexes[j-1]] == exponents[sorted_indexes[j]]))
 147:       {
 148:          bases[sorted_indexes[j-1]] *= bases[sorted_indexes[j]];
 149:          exponents[sorted_indexes[j]] = 0;
 150:          base_e_factors[sorted_indexes[j-1]] += base_e_factors[sorted_indexes[j]];
 151:          bubble_down_one(sorted_indexes + j, sorted_indexes + 9, sort_functor<T>(exponents));
 152:          --j;
 153:       }
 154:       --j;
 155: 
 156: #ifdef BOOST_MATH_INSTRUMENT
 157:       BOOST_MATH_INSTRUMENT_VARIABLE(j);
 158:       for(unsigned i = 0; i < 9; ++i)
 159:       {
 160:          BOOST_MATH_INSTRUMENT_VARIABLE(i);
 161:          BOOST_MATH_INSTRUMENT_VARIABLE(bases[i]);
 162:          BOOST_MATH_INSTRUMENT_VARIABLE(exponents[i]);
~~~
- **EN:** This range declares or defines callable logic such as bubble_down_one, BOOST_MATH_INSTRUMENT_VARIABLE. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 bubble_down_one, BOOST_MATH_INSTRUMENT_VARIABLE。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:          BOOST_MATH_INSTRUMENT_VARIABLE(base_e_factors[i]);
 164:          BOOST_MATH_INSTRUMENT_VARIABLE(sorted_indexes[i]);
 165:       }
 166: #endif
 167:    }
 168: 
 169: #ifdef BOOST_MATH_INSTRUMENT
 170:    BOOST_MATH_INSTRUMENT_FPU
 171:    for(unsigned i = 0; i < 9; ++i)
 172:    {
 173:       BOOST_MATH_INSTRUMENT_VARIABLE(i);
 174:       BOOST_MATH_INSTRUMENT_VARIABLE(bases[i]);
 175:       BOOST_MATH_INSTRUMENT_VARIABLE(exponents[i]);
 176:       BOOST_MATH_INSTRUMENT_VARIABLE(base_e_factors[i]);
 177:       BOOST_MATH_INSTRUMENT_VARIABLE(sorted_indexes[i]);
 178:    }
 179: #endif
 180: 
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:    T result;
 182:    BOOST_MATH_INSTRUMENT_VARIABLE(bases[sorted_indexes[0]] * exp(static_cast<T>(base_e_factors[sorted_indexes[0]])));
 183:    BOOST_MATH_INSTRUMENT_VARIABLE(exponents[sorted_indexes[0]]);
 184:    {
 185:       BOOST_FPU_EXCEPTION_GUARD
 186:       result = pow(bases[sorted_indexes[0]] * exp(static_cast<T>(base_e_factors[sorted_indexes[0]])), exponents[sorted_indexes[0]]);
 187:    }
 188:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 189:    for(std::size_t i = 1; (i < 9) && (exponents[sorted_indexes[i]] > 0); ++i)
 190:    {
 191:       BOOST_FPU_EXCEPTION_GUARD
 192:       if(result < tools::min_value<T>())
 193:          return 0; // short circuit further evaluation
 194:       if(exponents[sorted_indexes[i]] == 1)
 195:          result *= bases[sorted_indexes[i]] * exp(static_cast<T>(base_e_factors[sorted_indexes[i]]));
 196:       else if(exponents[sorted_indexes[i]] == 0.5f)
 197:          result *= sqrt(bases[sorted_indexes[i]] * exp(static_cast<T>(base_e_factors[sorted_indexes[i]])));
 198:       else
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_INSTRUMENT_VARIABLE, pow, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_INSTRUMENT_VARIABLE, pow, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:          result *= pow(bases[sorted_indexes[i]] * exp(static_cast<T>(base_e_factors[sorted_indexes[i]])), exponents[sorted_indexes[i]]);
 200: 
 201:       BOOST_MATH_INSTRUMENT_VARIABLE(result);
 202:    }
 203: 
 204:    result *= Lanczos::lanczos_sum_expG_scaled(static_cast<T>(n + 1))
 205:       * Lanczos::lanczos_sum_expG_scaled(static_cast<T>(r + 1))
 206:       * Lanczos::lanczos_sum_expG_scaled(static_cast<T>(N - n + 1))
 207:       * Lanczos::lanczos_sum_expG_scaled(static_cast<T>(N - r + 1))
 208:       /
 209:       ( Lanczos::lanczos_sum_expG_scaled(static_cast<T>(N + 1))
 210:          * Lanczos::lanczos_sum_expG_scaled(static_cast<T>(x + 1))
 211:          * Lanczos::lanczos_sum_expG_scaled(static_cast<T>(n - x + 1))
 212:          * Lanczos::lanczos_sum_expG_scaled(static_cast<T>(r - x + 1))
 213:          * Lanczos::lanczos_sum_expG_scaled(static_cast<T>(N - n - r + x + 1)));
 214: 
 215:    BOOST_MATH_INSTRUMENT_VARIABLE(result);
 216:    return result;
~~~
- **EN:** This range declares or defines callable logic such as pow, BOOST_MATH_INSTRUMENT_VARIABLE, .... Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 pow, BOOST_MATH_INSTRUMENT_VARIABLE, ...。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217: }
 218: 
 219: template <class T, class Policy>
 220: T hypergeometric_pdf_lanczos_imp(T /*dummy*/, std::uint64_t x, std::uint64_t r, std::uint64_t n, std::uint64_t N, const boost::math::lanczos::undefined_lanczos&, const Policy& pol)
 221: {
 222:    BOOST_MATH_STD_USING
 223:    return exp(
 224:       boost::math::lgamma(T(n + 1), pol)
 225:       + boost::math::lgamma(T(r + 1), pol)
 226:       + boost::math::lgamma(T(N - n + 1), pol)
 227:       + boost::math::lgamma(T(N - r + 1), pol)
 228:       - boost::math::lgamma(T(N + 1), pol)
 229:       - boost::math::lgamma(T(x + 1), pol)
 230:       - boost::math::lgamma(T(n - x + 1), pol)
 231:       - boost::math::lgamma(T(r - x + 1), pol)
 232:       - boost::math::lgamma(T(N - n - r + x + 1), pol));
 233: }
 234: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as boost::math::lgamma.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 boost::math::lgamma。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235: template <class T>
 236: inline T integer_power(const T& x, int ex)
 237: {
 238:    if(ex < 0)
 239:       return 1 / integer_power(x, -ex);
 240:    switch(ex)
 241:    {
 242:    case 0:
 243:       return 1;
 244:    case 1:
 245:       return x;
 246:    case 2:
 247:       return x * x;
 248:    case 3:
 249:       return x * x * x;
 250:    case 4:
 251:       return boost::math::pow<4>(x);
 252:    case 5:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:       return boost::math::pow<5>(x);
 254:    case 6:
 255:       return boost::math::pow<6>(x);
 256:    case 7:
 257:       return boost::math::pow<7>(x);
 258:    case 8:
 259:       return boost::math::pow<8>(x);
 260:    }
 261:    BOOST_MATH_STD_USING
 262: #ifdef __SUNPRO_CC
 263:    return pow(x, T(ex));
 264: #else
 265:    return static_cast<T>(pow(x, ex));
 266: #endif
 267: }
 268: template <class T>
 269: struct hypergeometric_pdf_prime_loop_result_entry
 270: {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:    T value;
 272:    const hypergeometric_pdf_prime_loop_result_entry* next;
 273: };
 274: 
 275: #ifdef _MSC_VER
 276: #pragma warning(push)
 277: #pragma warning(disable:4510 4512 4610)
 278: #endif
 279: 
 280: struct hypergeometric_pdf_prime_loop_data
 281: {
 282:    const std::uint64_t x;
 283:    const std::uint64_t r;
 284:    const std::uint64_t n;
 285:    const std::uint64_t N;
 286:    std::size_t prime_index;
 287:    std::uint64_t current_prime;
 288: };
~~~
- **EN:** It introduces the struct `hypergeometric_pdf_prime_loop_data` as part of the file's main abstraction. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 它引入了 struct `hypergeometric_pdf_prime_loop_data`，作为该文件核心抽象的一部分。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289: 
 290: #ifdef _MSC_VER
 291: #pragma warning(pop)
 292: #endif
 293: 
 294: template <class T>
 295: T hypergeometric_pdf_prime_loop_imp(hypergeometric_pdf_prime_loop_data& data, hypergeometric_pdf_prime_loop_result_entry<T>& result)
 296: {
 297:    while(data.current_prime <= data.N)
 298:    {
 299:       std::uint64_t base = data.current_prime;
 300:       std::uint64_t prime_powers = 0;
 301:       while(base <= data.N)
 302:       {
 303:          prime_powers += data.n / base;
 304:          prime_powers += data.r / base;
 305:          prime_powers += (data.N - data.n) / base;
 306:          prime_powers += (data.N - data.r) / base;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:          prime_powers -= data.N / base;
 308:          prime_powers -= data.x / base;
 309:          prime_powers -= (data.n - data.x) / base;
 310:          prime_powers -= (data.r - data.x) / base;
 311:          prime_powers -= (data.N - data.n - data.r + data.x) / base;
 312:          base *= data.current_prime;
 313:       }
 314:       if(prime_powers)
 315:       {
 316:          T p = integer_power<T>(static_cast<T>(data.current_prime), static_cast<int>(prime_powers));
 317:          if((p > 1) && (tools::max_value<T>() / p < result.value))
 318:          {
 319:             //
 320:             // The next calculation would overflow, use recursion
 321:             // to sidestep the issue:
 322:             //
 323:             hypergeometric_pdf_prime_loop_result_entry<T> t = { p, &result };
 324:             data.current_prime = prime(static_cast<unsigned>(++data.prime_index));
~~~
- **EN:** This range declares or defines callable logic such as prime. Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 prime。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:             return hypergeometric_pdf_prime_loop_imp<T>(data, t);
 326:          }
 327:          if((p < 1) && (tools::min_value<T>() / p > result.value))
 328:          {
 329:             //
 330:             // The next calculation would underflow, use recursion
 331:             // to sidestep the issue:
 332:             //
 333:             hypergeometric_pdf_prime_loop_result_entry<T> t = { p, &result };
 334:             data.current_prime = prime(static_cast<unsigned>(++data.prime_index));
 335:             return hypergeometric_pdf_prime_loop_imp<T>(data, t);
 336:          }
 337:          result.value *= p;
 338:       }
 339:       data.current_prime = prime(static_cast<unsigned>(++data.prime_index));
 340:    }
 341:    //
 342:    // When we get to here we have run out of prime factors,
~~~
- **EN:** This range declares or defines callable logic such as prime. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 prime。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:    // the overall result is the product of all the partial
 344:    // results we have accumulated on the stack so far, these
 345:    // are in a linked list starting with "data.head" and ending
 346:    // with "result".
 347:    //
 348:    // All that remains is to multiply them together, taking
 349:    // care not to overflow or underflow.
 350:    //
 351:    // Enumerate partial results >= 1 in variable i
 352:    // and partial results < 1 in variable j:
 353:    //
 354:    hypergeometric_pdf_prime_loop_result_entry<T> const *i, *j;
 355:    i = &result;
 356:    while(i && i->value < 1)
 357:       i = i->next;
 358:    j = &result;
 359:    while(j && j->value >= 1)
 360:       j = j->next;
~~~
- **EN:** Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361: 
 362:    T prod = 1;
 363: 
 364:    while(i || j)
 365:    {
 366:       while(i && ((prod <= 1) || (j == 0)))
 367:       {
 368:          prod *= i->value;
 369:          i = i->next;
 370:          while(i && i->value < 1)
 371:             i = i->next;
 372:       }
 373:       while(j && ((prod >= 1) || (i == 0)))
 374:       {
 375:          prod *= j->value;
 376:          j = j->next;
 377:          while(j && j->value >= 1)
 378:             j = j->next;
~~~
- **EN:** Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:       }
 380:    }
 381: 
 382:    return prod;
 383: }
 384: 
 385: template <class T, class Policy>
 386: inline T hypergeometric_pdf_prime_imp(std::uint64_t x, std::uint64_t r, std::uint64_t n, std::uint64_t N, const Policy&)
 387: {
 388:    hypergeometric_pdf_prime_loop_result_entry<T> result = { 1, 0 };
 389:    hypergeometric_pdf_prime_loop_data data = { x, r, n, N, 0, prime(0) };
 390:    return hypergeometric_pdf_prime_loop_imp<T>(data, result);
 391: }
 392: 
 393: template <class T, class Policy>
 394: T hypergeometric_pdf_factorial_imp(std::uint64_t x, std::uint64_t r, std::uint64_t n, std::uint64_t N, const Policy&)
 395: {
 396:    BOOST_MATH_STD_USING
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:    BOOST_MATH_ASSERT(N <= boost::math::max_factorial<T>::value);
 398:    T result = boost::math::unchecked_factorial<T>(static_cast<unsigned>(n));
 399:    T num[3] = {
 400:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(r)),
 401:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(N - n)),
 402:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(N - r))
 403:    };
 404:    T denom[5] = {
 405:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(N)),
 406:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(x)),
 407:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(n - x)),
 408:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(r - x)),
 409:       boost::math::unchecked_factorial<T>(static_cast<unsigned>(N - n - r + x))
 410:    };
 411:    std::size_t i = 0;
 412:    std::size_t j = 0;
 413:    while((i < 3) || (j < 5))
 414:    {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:       while((j < 5) && ((result >= 1) || (i >= 3)))
 416:       {
 417:          result /= denom[j];
 418:          ++j;
 419:       }
 420:       while((i < 3) && ((result <= 1) || (j >= 5)))
 421:       {
 422:          result *= num[i];
 423:          ++i;
 424:       }
 425:    }
 426:    return result;
 427: }
 428: 
 429: 
 430: template <class T, class Policy>
 431: inline typename tools::promote_args<T>::type
 432:    hypergeometric_pdf(std::uint64_t x, std::uint64_t r, std::uint64_t n, std::uint64_t N, const Policy&)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433: {
 434:    BOOST_FPU_EXCEPTION_GUARD
 435:    typedef typename tools::promote_args<T>::type result_type;
 436:    typedef typename policies::evaluation<result_type, Policy>::type value_type;
 437:    typedef typename lanczos::lanczos<value_type, Policy>::type evaluation_type;
 438:    typedef typename policies::normalise<
 439:       Policy,
 440:       policies::promote_float<false>,
 441:       policies::promote_double<false>,
 442:       policies::discrete_quantile<>,
 443:       policies::assert_undefined<> >::type forwarding_policy;
 444: 
 445:    value_type result;
 446:    if(N <= boost::math::max_factorial<value_type>::value)
 447:    {
 448:       //
 449:       // If N is small enough then we can evaluate the PDF via the factorials
 450:       // directly: table lookup of the factorials gives the best performance
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:       // of the methods available:
 452:       //
 453:       result = detail::hypergeometric_pdf_factorial_imp<value_type>(x, r, n, N, forwarding_policy());
 454:    }
 455:    else if(N <= boost::math::prime(boost::math::max_prime - 1))
 456:    {
 457:       //
 458:       // If N is no larger than the largest prime number in our lookup table
 459:       // (104729) then we can use prime factorisation to evaluate the PDF,
 460:       // this is slow but accurate:
 461:       //
 462:       result = detail::hypergeometric_pdf_prime_imp<value_type>(x, r, n, N, forwarding_policy());
 463:    }
 464:    else
 465:    {
 466:       //
 467:       // Catch all case - use the lanczos approximation - where available -
 468:       // to evaluate the ratio of factorials.  This is reasonably fast
~~~
- **EN:** This range declares or defines callable logic such as forwarding_policy. Conditional branches split behavior across input ranges, error cases, or configuration modes. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 此范围声明或定义了可调用逻辑，例如 forwarding_policy。 条件分支会根据输入区间、错误情况或配置模式切换行为。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:       // (almost as quick as using logarithmic evaluation in terms of lgamma)
 470:       // but only a few digits better in accuracy than using lgamma:
 471:       //
 472:       result = detail::hypergeometric_pdf_lanczos_imp(value_type(), x, r, n, N, evaluation_type(), forwarding_policy());
 473:    }
 474: 
 475:    if(result > 1)
 476:    {
 477:       result = 1;
 478:    }
 479:    if(result < 0)
 480:    {
 481:       result = 0;
 482:    }
 483: 
 484:    return policies::checked_narrowing_cast<result_type, forwarding_policy>(result, "boost::math::hypergeometric_pdf<%1%>(%1%,%1%,%1%,%1%)");
 485: }
 486: 
~~~
- **EN:** This range declares or defines callable logic such as detail::hypergeometric_pdf_lanczos_imp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::hypergeometric_pdf_lanczos_imp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 487-490 / 第 487-490 行
~~~cpp
 487: }}} // namespaces
 488: 
 489: #endif
 490: 
~~~
- **EN:** This short range closes a conditional-compilation or header-guard region. This range mainly closes the preceding type, namespace, or conditional scope. The implementation focuses on distribution mathematics such as density, cumulative probability, or inverse transforms.
- **CN:** 这一小段用于结束条件编译区域或头文件保护区域。 此范围主要用于结束前面的类型、命名空间或条件作用域。 该实现聚焦于分布数学，例如密度、累积概率或反变换。

## Key Concepts / 关键概念
- **Header guard / 头文件保护**: Prevents repeated inclusion during compilation. / 防止编译期间被重复包含。
- **Header dependencies / 头文件依赖**: Pulls in other declarations, traits, and standard facilities. / 引入其他声明、类型萃取和标准库能力。
- **Namespaces / 命名空间**: Keeps third-party symbols scoped and avoids collisions. / 通过命名空间约束第三方符号，避免冲突。
- **Templates / 模板**: Generalizes the implementation across numeric types or policies. / 让实现可适配不同数值类型或策略。
- **Types / 类型定义**: Introduces classes or structs that model the library abstraction. / 引入类或结构体来表达库中的抽象。
- **Functions / 函数**: Provides callable entry points or helpers. / 提供可调用的入口或辅助函数。
- **Macros / 宏**: Handles portability, visibility, or compile-time switches. / 处理可移植性、可见性或编译期开关。
- **Precomputed coefficients / 预计算系数**: Stores constants or tables used by numerical approximations. / 存储数值逼近使用的常量或查找表。

## Dependencies / 依赖关系
- **Included headers / 包含头文件**: `boost/math/constants/constants.hpp, boost/math/special_functions/lanczos.hpp, boost/math/special_functions/gamma.hpp, boost/math/special_functions/pow.hpp, boost/math/special_functions/prime.hpp, boost/math/policies/error_handling.hpp, algorithm, cstdint, typeinfo`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `swap, sort_functor, BOOST_MATH_INSTRUMENT_VARIABLE, std::sort, bubble_down_one, while, pow, exp, ...`
