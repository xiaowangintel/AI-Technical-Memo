# next.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/ccmath/next.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This Boost.Math header provides reusable support for ccmath next.
- **作用（中文）**: 此 Boost.Math 头文件为 ccmath next 提供可复用支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  (C) Copyright John Maddock 2008 - 2022.
   2: //  (C) Copyright Matt Borland 2022.
   3: //  Use, modification and distribution are subject to the
   4: //  Boost Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_MATH_CCMATH_NEXT_HPP
   8: #define BOOST_MATH_CCMATH_NEXT_HPP
   9: 
  10: #include <boost/math/ccmath/detail/config.hpp>
  11: 
  12: #ifdef BOOST_MATH_NO_CCMATH
  13: #error "The header <boost/math/next.hpp> can only be used in C++17 and later."
  14: #endif
  15: 
  16: #include <stdexcept>
  17: #include <cfloat>
  18: #include <cstdint>
~~~
- **EN:** This block imports dependencies such as boost/math/ccmath/detail/config.hpp, stdexcept, cfloat, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/ccmath/detail/config.hpp, stdexcept, cfloat, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/policies/policy.hpp>
  20: #include <boost/math/policies/error_handling.hpp>
  21: #include <boost/math/tools/assert.hpp>
  22: #include <boost/math/tools/config.hpp>
  23: #include <boost/math/tools/precision.hpp>
  24: #include <boost/math/tools/traits.hpp>
  25: #include <boost/math/tools/promotion.hpp>
  26: #include <boost/math/ccmath/ilogb.hpp>
  27: #include <boost/math/ccmath/ldexp.hpp>
  28: #include <boost/math/ccmath/scalbln.hpp>
  29: #include <boost/math/ccmath/round.hpp>
  30: #include <boost/math/ccmath/fabs.hpp>
  31: #include <boost/math/ccmath/fpclassify.hpp>
  32: #include <boost/math/ccmath/isfinite.hpp>
  33: #include <boost/math/ccmath/fmod.hpp>
  34: 
  35: namespace boost::math::ccmath {
  36: 
~~~
- **EN:** This block imports dependencies such as boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, boost/math/tools/assert.hpp, ... so the surrounding code can use external declarations. The code enters namespace scope (boost) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, boost/math/tools/assert.hpp, ... 等依赖，使周围代码可以使用外部声明。 代码进入命名空间作用域（boost），以保持符号组织清晰。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: namespace detail {
  38: 
  39: // Forward Declarations
  40: template <typename T, typename result_type = tools::promote_args_t<T>>
  41: constexpr result_type float_prior(const T& val);
  42: 
  43: template <typename T, typename result_type = tools::promote_args_t<T>>
  44: constexpr result_type float_next(const T& val);
  45: 
  46: template <typename T>
  47: struct has_hidden_guard_digits;
  48: template <>
  49: struct has_hidden_guard_digits<float> : public std::false_type {};
  50: template <>
  51: struct has_hidden_guard_digits<double> : public std::false_type {};
  52: template <>
  53: struct has_hidden_guard_digits<long double> : public std::false_type {};
  54: #ifdef BOOST_HAS_FLOAT128
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `has_hidden_guard_digits` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `has_hidden_guard_digits`，作为该文件核心抽象的一部分。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: template <>
  56: struct has_hidden_guard_digits<__float128> : public std::false_type {};
  57: #endif
  58: 
  59: template <typename T, bool b>
  60: struct has_hidden_guard_digits_10 : public std::false_type {};
  61: template <typename T>
  62: struct has_hidden_guard_digits_10<T, true> : public std::integral_constant<bool, (std::numeric_limits<T>::digits10 != std::numeric_limits<T>::max_digits10)> {};
  63: 
  64: template <typename T>
  65: struct has_hidden_guard_digits
  66:     : public has_hidden_guard_digits_10<T,
  67:     std::numeric_limits<T>::is_specialized
  68:     && (std::numeric_limits<T>::radix == 10) >
  69: {};
  70: 
  71: template <typename T>
  72: constexpr T normalize_value(const T& val, const std::false_type&) { return val; }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `has_hidden_guard_digits` as part of the file's main abstraction. This range declares or defines callable logic such as normalize_value.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `has_hidden_guard_digits`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 normalize_value。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73: template <typename T>
  74: constexpr T normalize_value(const T& val, const std::true_type&)
  75: {
  76:     static_assert(std::numeric_limits<T>::is_specialized, "Type T must be specialized.");
  77:     static_assert(std::numeric_limits<T>::radix != 2, "Type T must be specialized.");
  78: 
  79:     std::intmax_t shift = static_cast<std::intmax_t>(std::numeric_limits<T>::digits) - static_cast<std::intmax_t>(boost::math::ccmath::ilogb(val)) - 1;
  80:     T result = boost::math::ccmath::scalbn(val, shift);
  81:     result = boost::math::ccmath::round(result);
  82:     return boost::math::ccmath::scalbn(result, -shift);
  83: }
  84: 
  85: template <typename T>
  86: constexpr T get_smallest_value(const std::true_type&)
  87: {
  88:     //
  89:     // numeric_limits lies about denorms being present - particularly
  90:     // when this can be turned on or off at runtime, as is the case
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as static_assert, boost::math::ccmath::scalbn, .... Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 static_assert, boost::math::ccmath::scalbn, ...。 return 语句会把计算结果或状态返回给调用方。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:     // when using the SSE2 registers in DAZ or FTZ mode.
  92:     //
  93:     constexpr T m = std::numeric_limits<T>::denorm_min();
  94:     return ((tools::min_value<T>() / 2) == 0) ? tools::min_value<T>() : m;
  95: }
  96: 
  97: template <typename T>
  98: constexpr T get_smallest_value(const std::false_type&)
  99: {
 100:     return tools::min_value<T>();
 101: }
 102: 
 103: template <typename T>
 104: constexpr T get_smallest_value()
 105: {
 106:     return get_smallest_value<T>(std::integral_constant<bool, std::numeric_limits<T>::is_specialized>());
 107: }
 108: 
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as denorm_min. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 denorm_min。 return 语句会把计算结果或状态返回给调用方。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109: template <typename T>
 110: constexpr T calc_min_shifted(const std::true_type&)
 111: {
 112:    return boost::math::ccmath::ldexp(tools::min_value<T>(), tools::digits<T>() + 1);
 113: }
 114: 
 115: template <typename T>
 116: constexpr T calc_min_shifted(const std::false_type&)
 117: {
 118:    static_assert(std::numeric_limits<T>::is_specialized, "Type T must be specialized.");
 119:    static_assert(std::numeric_limits<T>::radix != 2, "Type T must be specialized.");
 120: 
 121:    return boost::math::ccmath::scalbn(tools::min_value<T>(), std::numeric_limits<T>::digits + 1);
 122: }
 123: 
 124: template <typename T>
 125: constexpr T get_min_shift_value()
 126: {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as static_assert. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 static_assert。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:    const T val = calc_min_shifted<T>(std::integral_constant<bool, !std::numeric_limits<T>::is_specialized || std::numeric_limits<T>::radix == 2>());
 128:    return val;
 129: }
 130: 
 131: template <typename T, bool b = boost::math::tools::detail::has_backend_type_v<T>>
 132: struct exponent_type
 133: {
 134:     using type = int;
 135: };
 136: 
 137: template <typename T>
 138: struct exponent_type<T, true>
 139: {
 140:     using type = typename T::backend_type::exponent_type;
 141: };
 142: 
 143: template <typename T, bool b = boost::math::tools::detail::has_backend_type_v<T>>
 144: using exponent_type_t = typename exponent_type<T>::type;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `exponent_type` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `exponent_type`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145: 
 146: template <typename T>
 147: constexpr T float_next_imp(const T& val, const std::true_type&)
 148: {
 149:     using exponent_type = exponent_type_t<T>;
 150: 
 151:     exponent_type expon {};
 152: 
 153:     int fpclass = boost::math::ccmath::fpclassify(val);
 154: 
 155:     if (fpclass == FP_NAN)
 156:     {
 157:         return val;
 158:     }
 159:     else if (fpclass == FP_INFINITE)
 160:     {
 161:         return val;
 162:     }
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as boost::math::ccmath::fpclassify. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 boost::math::ccmath::fpclassify。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:     else if (val <= -tools::max_value<T>())
 164:     {
 165:         return val;
 166:     }
 167: 
 168:     if (val == 0)
 169:     {
 170:         return detail::get_smallest_value<T>();
 171:     }
 172: 
 173:     if ((fpclass != FP_SUBNORMAL) && (fpclass != FP_ZERO)
 174:         && (boost::math::ccmath::fabs(val) < detail::get_min_shift_value<T>())
 175:         && (val != -tools::min_value<T>()))
 176:     {
 177:         //
 178:         // Special case: if the value of the least significant bit is a denorm, and the result
 179:         // would not be a denorm, then shift the input, increment, and shift back.
 180:         // This avoids issues with the Intel SSE2 registers when the FTZ or DAZ flags are set.
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:         //
 182:         return boost::math::ccmath::ldexp(boost::math::ccmath::detail::float_next(static_cast<T>(boost::math::ccmath::ldexp(val, 2 * tools::digits<T>()))), -2 * tools::digits<T>());
 183:     }
 184: 
 185:     if (-0.5f == boost::math::ccmath::frexp(val, &expon))
 186:     {
 187:         --expon; // reduce exponent when val is a power of two, and negative.
 188:     }
 189:     T diff = boost::math::ccmath::ldexp(static_cast<T>(1), expon - tools::digits<T>());
 190:     if(diff == 0)
 191:     {
 192:         diff = detail::get_smallest_value<T>();
 193:     }
 194:     return val + diff;
 195: }
 196: 
 197: //
 198: // Special version for some base other than 2:
~~~
- **EN:** This range declares or defines callable logic such as boost::math::ccmath::ldexp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::ccmath::ldexp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199: //
 200: template <typename T>
 201: constexpr T float_next_imp(const T& val, const std::false_type&)
 202: {
 203:     using exponent_type = exponent_type_t<T>;
 204: 
 205:     static_assert(std::numeric_limits<T>::is_specialized, "Type T must be specialized.");
 206:     static_assert(std::numeric_limits<T>::radix != 2, "Type T must be specialized.");
 207: 
 208:     exponent_type expon {};
 209: 
 210:     int fpclass = boost::math::ccmath::fpclassify(val);
 211: 
 212:     if (fpclass == FP_NAN)
 213:     {
 214:         return val;
 215:     }
 216:     else if (fpclass == FP_INFINITE)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as static_assert, boost::math::ccmath::fpclassify. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 static_assert, boost::math::ccmath::fpclassify。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:     {
 218:         return val;
 219:     }
 220:     else if (val <= -tools::max_value<T>())
 221:     {
 222:         return val;
 223:     }
 224: 
 225:     if (val == 0)
 226:     {
 227:         return detail::get_smallest_value<T>();
 228:     }
 229: 
 230:     if ((fpclass != FP_SUBNORMAL) && (fpclass != FP_ZERO)
 231:         && (boost::math::ccmath::fabs(val) < detail::get_min_shift_value<T>())
 232:         && (val != -tools::min_value<T>()))
 233:     {
 234:         //
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:         // Special case: if the value of the least significant bit is a denorm, and the result
 236:         // would not be a denorm, then shift the input, increment, and shift back.
 237:         // This avoids issues with the Intel SSE2 registers when the FTZ or DAZ flags are set.
 238:         //
 239:         return boost::math::ccmath::scalbn(boost::math::ccmath::detail::float_next(static_cast<T>(boost::math::ccmath::scalbn(val, 2 * std::numeric_limits<T>::digits))), -2 * std::numeric_limits<T>::digits);
 240:     }
 241: 
 242:     expon = 1 + boost::math::ccmath::ilogb(val);
 243:     if(-1 == boost::math::ccmath::scalbn(val, -expon) * std::numeric_limits<T>::radix)
 244:     {
 245:         --expon; // reduce exponent when val is a power of base, and negative.
 246:     }
 247: 
 248:     T diff = boost::math::ccmath::scalbn(static_cast<T>(1), expon - std::numeric_limits<T>::digits);
 249:     if(diff == 0)
 250:     {
 251:         diff = detail::get_smallest_value<T>();
 252:     }
~~~
- **EN:** This range declares or defines callable logic such as boost::math::ccmath::ilogb, boost::math::ccmath::scalbn. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::ccmath::ilogb, boost::math::ccmath::scalbn。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253: 
 254:     return val + diff;
 255: }
 256: 
 257: template <typename T, typename result_type>
 258: constexpr result_type float_next(const T& val)
 259: {
 260:     return detail::float_next_imp(detail::normalize_value(static_cast<result_type>(val), typename detail::has_hidden_guard_digits<result_type>::type()), std::integral_constant<bool, !std::numeric_limits<result_type>::is_specialized || (std::numeric_limits<result_type>::radix == 2)>());
 261: }
 262: 
 263: template <typename T>
 264: constexpr T float_prior_imp(const T& val, const std::true_type&)
 265: {
 266:     using exponent_type = exponent_type_t<T>;
 267: 
 268:     exponent_type expon {};
 269: 
 270:     int fpclass = boost::math::ccmath::fpclassify(val);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as boost::math::ccmath::fpclassify. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 boost::math::ccmath::fpclassify。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271: 
 272:     if (fpclass == FP_NAN)
 273:     {
 274:         return val;
 275:     }
 276:     else if (fpclass == FP_INFINITE)
 277:     {
 278:         return val;
 279:     }
 280:     else if (val <= -tools::max_value<T>())
 281:     {
 282:         return val;
 283:     }
 284: 
 285:     if (val == 0)
 286:     {
 287:         return -detail::get_smallest_value<T>();
 288:     }
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289: 
 290:     if ((fpclass != FP_SUBNORMAL) && (fpclass != FP_ZERO)
 291:         && (boost::math::ccmath::fabs(val) < detail::get_min_shift_value<T>())
 292:         && (val != tools::min_value<T>()))
 293:     {
 294:         //
 295:         // Special case: if the value of the least significant bit is a denorm, and the result
 296:         // would not be a denorm, then shift the input, increment, and shift back.
 297:         // This avoids issues with the Intel SSE2 registers when the FTZ or DAZ flags are set.
 298:         //
 299:         return boost::math::ccmath::ldexp(boost::math::ccmath::detail::float_prior(static_cast<T>(boost::math::ccmath::ldexp(val, 2 * tools::digits<T>()))), -2 * tools::digits<T>());
 300:     }
 301: 
 302:     if(T remain = boost::math::ccmath::frexp(val, &expon); remain == 0.5f)
 303:     {
 304:         --expon; // when val is a power of two we must reduce the exponent
 305:     }
 306: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:     T diff = boost::math::ccmath::ldexp(static_cast<T>(1), expon - tools::digits<T>());
 308:     if(diff == 0)
 309:     {
 310:         diff = detail::get_smallest_value<T>();
 311:     }
 312: 
 313:     return val - diff;
 314: }
 315: 
 316: //
 317: // Special version for bases other than 2:
 318: //
 319: template <typename T>
 320: constexpr T float_prior_imp(const T& val, const std::false_type&)
 321: {
 322:     using exponent_type = exponent_type_t<T>;
 323: 
 324:     static_assert(std::numeric_limits<T>::is_specialized, "Type T must be specialized.");
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as boost::math::ccmath::ldexp, static_assert. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 boost::math::ccmath::ldexp, static_assert。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:     static_assert(std::numeric_limits<T>::radix != 2, "Type T must be specialized.");
 326: 
 327:     exponent_type expon {};
 328: 
 329:     int fpclass = boost::math::ccmath::fpclassify(val);
 330: 
 331:     if (fpclass == FP_NAN)
 332:     {
 333:         return val;
 334:     }
 335:     else if (fpclass == FP_INFINITE)
 336:     {
 337:         return val;
 338:     }
 339:     else if (val <= -tools::max_value<T>())
 340:     {
 341:         return val;
 342:     }
~~~
- **EN:** This range declares or defines callable logic such as static_assert, boost::math::ccmath::fpclassify. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 static_assert, boost::math::ccmath::fpclassify。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343: 
 344:     if (val == 0)
 345:     {
 346:         return -detail::get_smallest_value<T>();
 347:     }
 348: 
 349:     if ((fpclass != FP_SUBNORMAL) && (fpclass != FP_ZERO)
 350:         && (boost::math::ccmath::fabs(val) < detail::get_min_shift_value<T>())
 351:         && (val != tools::min_value<T>()))
 352:     {
 353:         //
 354:         // Special case: if the value of the least significant bit is a denorm, and the result
 355:         // would not be a denorm, then shift the input, increment, and shift back.
 356:         // This avoids issues with the Intel SSE2 registers when the FTZ or DAZ flags are set.
 357:         //
 358:         return boost::math::ccmath::scalbn(boost::math::ccmath::detail::float_prior(static_cast<T>(boost::math::ccmath::scalbn(val, 2 * std::numeric_limits<T>::digits))), -2 * std::numeric_limits<T>::digits);
 359:     }
 360: 
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:     expon = 1 + boost::math::ccmath::ilogb(val);
 362: 
 363:     if (T remain = boost::math::ccmath::scalbn(val, -expon); remain * std::numeric_limits<T>::radix == 1)
 364:     {
 365:         --expon; // when val is a power of two we must reduce the exponent
 366:     }
 367: 
 368:     T diff = boost::math::ccmath::scalbn(static_cast<T>(1), expon - std::numeric_limits<T>::digits);
 369:     if (diff == 0)
 370:     {
 371:         diff = detail::get_smallest_value<T>();
 372:     }
 373:     return val - diff;
 374: } // float_prior_imp
 375: 
 376: template <typename T, typename result_type>
 377: constexpr result_type float_prior(const T& val)
 378: {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as boost::math::ccmath::ilogb, boost::math::ccmath::scalbn. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 boost::math::ccmath::ilogb, boost::math::ccmath::scalbn。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:     return detail::float_prior_imp(detail::normalize_value(static_cast<result_type>(val), typename detail::has_hidden_guard_digits<result_type>::type()), std::integral_constant<bool, !std::numeric_limits<result_type>::is_specialized || (std::numeric_limits<result_type>::radix == 2)>());
 380: }
 381: 
 382: } // namespace detail
 383: 
 384: template <typename T, typename U, typename result_type = tools::promote_args_t<T, U>>
 385: constexpr result_type nextafter(const T& val, const U& direction)
 386: {
 387:     if (BOOST_MATH_IS_CONSTANT_EVALUATED(val))
 388:     {
 389:         if (boost::math::ccmath::isnan(val))
 390:         {
 391:             return val;
 392:         }
 393:         else if (boost::math::ccmath::isnan(direction))
 394:         {
 395:             return direction;
 396:         }
~~~
- **EN:** The code enters namespace scope (detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 代码进入命名空间作用域（detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:         else if (val < direction)
 398:         {
 399:             return boost::math::ccmath::detail::float_next(val);
 400:         }
 401:         else if (val == direction)
 402:         {
 403:             // IEC 60559 recommends that from is returned whenever from == to. These functions return to instead,
 404:             // which makes the behavior around zero consistent: std::nextafter(-0.0, +0.0) returns +0.0 and
 405:             // std::nextafter(+0.0, -0.0) returns -0.0.
 406:             return direction;
 407:         }
 408: 
 409:         return boost::math::ccmath::detail::float_prior(val);
 410:     }
 411:     else
 412:     {
 413:         using std::nextafter;
 414:         return nextafter(static_cast<result_type>(val), static_cast<result_type>(direction));
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:     }
 416: }
 417: 
 418: constexpr float nextafterf(float val, float direction)
 419: {
 420:     return boost::math::ccmath::nextafter(val, direction);
 421: }
 422: 
 423: #ifndef BOOST_MATH_NO_LONG_DOUBLE_MATH_FUNCTIONS
 424: 
 425: constexpr long double nextafterl(long double val, long double direction)
 426: {
 427:     return boost::math::ccmath::nextafter(val, direction);
 428: }
 429: 
 430: template <typename T, typename result_type = tools::promote_args_t<T, long double>, typename return_type = std::conditional_t<std::is_integral_v<T>, double, T>>
 431: constexpr return_type nexttoward(T val, long double direction)
 432: {
~~~
- **EN:** It begins the file-level inclusion guard or portability wrapper. Templates in this range make the implementation reusable across types, precisions, or policies. Return statements hand the computed result or status back to the caller.
- **CN:** 它开始设置文件级防重包含或可移植性包装层。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 return 语句会把计算结果或状态返回给调用方。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:     if (BOOST_MATH_IS_CONSTANT_EVALUATED(val))
 434:     {
 435:         return static_cast<return_type>(boost::math::ccmath::nextafter(static_cast<result_type>(val), direction));
 436:     }
 437:     else
 438:     {
 439:         using std::nexttoward;
 440:         return nexttoward(val, direction);
 441:     }
 442: }
 443: 
 444: constexpr float nexttowardf(float val, long double direction)
 445: {
 446:     return boost::math::ccmath::nexttoward(val, direction);
 447: }
 448: 
 449: constexpr long double nexttowardl(long double val, long double direction)
 450: {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 451-458 / 第 451-458 行
~~~cpp
 451:     return boost::math::ccmath::nexttoward(val, direction);
 452: }
 453: 
 454: #endif
 455: 
 456: } // Namespaces
 457: 
 458: #endif // BOOST_MATH_SPECIAL_NEXT_HPP
~~~
- **EN:** Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/ccmath/detail/config.hpp, stdexcept, cfloat, cstdint, boost/math/policies/policy.hpp, boost/math/policies/error_handling.hpp, boost/math/tools/assert.hpp, boost/math/tools/config.hpp, boost/math/tools/precision.hpp, boost/math/tools/traits.hpp, boost/math/tools/promotion.hpp, boost/math/ccmath/ilogb.hpp, ...`
- **Namespaces / 命名空间**: `boost, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `float_prior, float_next, normalize_value, static_assert, boost::math::ccmath::scalbn, boost::math::ccmath::round, denorm_min, boost::math::ccmath::fpclassify, ...`
