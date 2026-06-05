# hypergeometric_1F1_recurrence.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/hypergeometric_1F1_recurrence.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the hypergeometric 1F1 recurrence special-function path.
- **作用（中文）**: 此头文件为 hypergeometric 1F1 recurrence 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: 
   2: ///////////////////////////////////////////////////////////////////////////////
   3: //  Copyright 2014 Anton Bikineev
   4: //  Copyright 2014 Christopher Kormanyos
   5: //  Copyright 2014 John Maddock
   6: //  Copyright 2014 Paul Bristow
   7: //  Distributed under the Boost
   8: //  Software License, Version 1.0. (See accompanying file
   9: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
  10: 
  11: #ifndef BOOST_HYPERGEOMETRIC_1F1_RECURRENCE_HPP_
  12: #define BOOST_HYPERGEOMETRIC_1F1_RECURRENCE_HPP_
  13: 
  14: #include <boost/math/special_functions/modf.hpp>
  15: #include <boost/math/special_functions/next.hpp>
  16: 
  17: #include <boost/math/tools/recurrence.hpp>
  18: #include <boost/math/special_functions/detail/hypergeometric_pFq_checked_series.hpp>
~~~
- **EN:** This block imports dependencies such as boost/math/special_functions/modf.hpp, boost/math/special_functions/next.hpp, boost/math/tools/recurrence.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/special_functions/modf.hpp, boost/math/special_functions/next.hpp, boost/math/tools/recurrence.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: 
  20:   namespace boost { namespace math { namespace detail {
  21: 
  22:   // forward declaration for initial values
  23:   template <class T, class Policy>
  24:   inline T hypergeometric_1F1_imp(const T& a, const T& b, const T& z, const Policy& pol);
  25: 
  26:   template <class T, class Policy>
  27:   inline T hypergeometric_1F1_imp(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling);
  28: 
  29:   template <class T>
  30:   struct hypergeometric_1F1_recurrence_a_coefficients
  31:   {
  32:     using result_type = boost::math::tuple<T, T, T>;
  33: 
  34:     hypergeometric_1F1_recurrence_a_coefficients(const T& a, const T& b, const T& z):
  35:     a(a), b(b), z(z)
  36:     {
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:     }
  38: 
  39:     hypergeometric_1F1_recurrence_a_coefficients(const hypergeometric_1F1_recurrence_a_coefficients&) = default;
  40: 
  41:     hypergeometric_1F1_recurrence_a_coefficients operator=(const hypergeometric_1F1_recurrence_a_coefficients&) = delete;
  42: 
  43:     result_type operator()(std::intmax_t i) const
  44:     {
  45:       const T ai = a + i;
  46: 
  47:       const T an = b - ai;
  48:       const T bn = (2 * ai - b + z);
  49:       const T cn = -ai;
  50: 
  51:       return boost::math::make_tuple(an, bn, cn);
  52:     }
  53: 
  54:   private:
~~~
- **EN:** Return statements hand the computed result or status back to the caller.
- **CN:** return 语句会把计算结果或状态返回给调用方。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:     const T a;
  56:     const T b;
  57:     const T z;
  58:   };
  59: 
  60:   template <class T>
  61:   struct hypergeometric_1F1_recurrence_b_coefficients
  62:   {
  63:     using result_type = boost::math::tuple<T, T, T>;
  64: 
  65:     hypergeometric_1F1_recurrence_b_coefficients(const T& a, const T& b, const T& z):
  66:     a(a), b(b), z(z)
  67:     {
  68:     }
  69: 
  70:     hypergeometric_1F1_recurrence_b_coefficients(const hypergeometric_1F1_recurrence_b_coefficients&) = default;
  71: 
  72:     hypergeometric_1F1_recurrence_b_coefficients& operator=(const hypergeometric_1F1_recurrence_b_coefficients&) = delete;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73: 
  74:     result_type operator()(std::intmax_t i) const
  75:     {
  76:       const T bi = b + i;
  77: 
  78:       const T an = bi * (bi - 1);
  79:       const T bn = bi * (1 - bi - z);
  80:       const T cn = z * (bi - a);
  81: 
  82:       return boost::math::make_tuple(an, bn, cn);
  83:     }
  84: 
  85:   private:
  86:     const T a;
  87:     const T b;
  88:     const T z;
  89:   };
  90:   //
~~~
- **EN:** Return statements hand the computed result or status back to the caller.
- **CN:** return 语句会把计算结果或状态返回给调用方。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:   // for use when we're recursing to a small b:
  92:   //
  93:   template <class T>
  94:   struct hypergeometric_1F1_recurrence_small_b_coefficients
  95:   {
  96:      using result_type = boost::math::tuple<T, T, T>;
  97: 
  98:      hypergeometric_1F1_recurrence_small_b_coefficients(const T& a, const T& b, const T& z, int N) :
  99:         a(a), b(b), z(z), N(N)
 100:      {
 101:      }
 102: 
 103:      hypergeometric_1F1_recurrence_small_b_coefficients(const hypergeometric_1F1_recurrence_small_b_coefficients&) = default;
 104: 
 105:      hypergeometric_1F1_recurrence_small_b_coefficients operator=(const hypergeometric_1F1_recurrence_small_b_coefficients&) = delete;
 106: 
 107:      result_type operator()(std::intmax_t i) const
 108:      {
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:         const T bi = b + (i + N);
 110:         const T bi_minus_1 = b + (i + N - 1);
 111: 
 112:         const T an = bi * bi_minus_1;
 113:         const T bn = bi * (-bi_minus_1 - z);
 114:         const T cn = z * (bi - a);
 115: 
 116:         return boost::math::make_tuple(an, bn, cn);
 117:      }
 118: 
 119:   private:
 120:      const T a;
 121:      const T b;
 122:      const T z;
 123:      int N;
 124:   };
 125: 
 126:   template <class T>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 return 语句会把计算结果或状态返回给调用方。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:   struct hypergeometric_1F1_recurrence_a_and_b_coefficients
 128:   {
 129:     using result_type = boost::math::tuple<T, T, T>;
 130: 
 131:     hypergeometric_1F1_recurrence_a_and_b_coefficients(const T& a, const T& b, const T& z, int offset = 0):
 132:     a(a), b(b), z(z), offset(offset)
 133:     {
 134:     }
 135: 
 136:     hypergeometric_1F1_recurrence_a_and_b_coefficients(const hypergeometric_1F1_recurrence_a_and_b_coefficients&) = default;
 137: 
 138:     hypergeometric_1F1_recurrence_a_and_b_coefficients operator=(const hypergeometric_1F1_recurrence_a_and_b_coefficients&) = delete;
 139: 
 140:     result_type operator()(std::intmax_t i) const
 141:     {
 142:       const T ai = a + (offset + i);
 143:       const T bi = b + (offset + i);
 144: 
~~~
- **EN:** It introduces the struct `hypergeometric_1F1_recurrence_a_and_b_coefficients` as part of the file's main abstraction. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 它引入了 struct `hypergeometric_1F1_recurrence_a_and_b_coefficients`，作为该文件核心抽象的一部分。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:       const T an = bi * (b + (offset + i - 1));
 146:       const T bn = bi * (z - (b + (offset + i - 1)));
 147:       const T cn = -ai * z;
 148: 
 149:       return boost::math::make_tuple(an, bn, cn);
 150:     }
 151: 
 152:   private:
 153:     const T a;
 154:     const T b;
 155:     const T z;
 156:     int offset;
 157:   };
 158: #if 0
 159:   //
 160:   // These next few recurrence relations are archived for future reference, some of them are novel, though all
 161:   // are trivially derived from the existing well known relations:
 162:   //
~~~
- **EN:** Return statements hand the computed result or status back to the caller.
- **CN:** return 语句会把计算结果或状态返回给调用方。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:   // Recurrence relation for double-stepping on both a and b:
 164:   // - b(b-1)(b-2) / (2-b+z) M(a-2,b-2,z) + [b(a-1)z / (2-b+z) + b(1-b+z) + abz(b+1) /(b+1)(z-b)] M(a,b,z) - a(a+1)z^2 / (b+1)(z-b) M(a+2,b+2,z)
 165:   //
 166:   template <class T>
 167:   struct hypergeometric_1F1_recurrence_2a_and_2b_coefficients
 168:   {
 169:      typedef boost::math::tuple<T, T, T> result_type;
 170: 
 171:      hypergeometric_1F1_recurrence_2a_and_2b_coefficients(const T& a, const T& b, const T& z, int offset = 0) :
 172:         a(a), b(b), z(z), offset(offset)
 173:      {
 174:      }
 175: 
 176:      result_type operator()(std::intmax_t i) const
 177:      {
 178:         i *= 2;
 179:         const T ai = a + (offset + i);
 180:         const T bi = b + (offset + i);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181: 
 182:         const T an = -bi * (b + (offset + i - 1)) * (b + (offset + i - 2)) / (-(b + (offset + i - 2)) + z);
 183:         const T bn = bi * (a + (offset + i - 1)) * z / (z - (b + (offset + i - 2)))
 184:            + bi * (z - (b + (offset + i - 1)))
 185:            + ai * bi * z * (b + (offset + i + 1)) / ((b + (offset + i + 1)) * (z - bi));
 186:         const T cn = -ai * (a + (offset + i + 1)) * z * z / ((b + (offset + i + 1)) * (z - bi));
 187: 
 188:         return boost::math::make_tuple(an, bn, cn);
 189:      }
 190: 
 191:   private:
 192:      const T a, b, z;
 193:      int offset;
 194:      hypergeometric_1F1_recurrence_2a_and_2b_coefficients operator=(const hypergeometric_1F1_recurrence_2a_and_2b_coefficients&);
 195:   };
 196: 
 197:   //
 198:   // Recurrence relation for double-stepping on a:
~~~
- **EN:** Return statements hand the computed result or status back to the caller.
- **CN:** return 语句会把计算结果或状态返回给调用方。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:   // -(b-a)(1 + b - a)/(2a-2-b+z)M(a-2,b,z)  + [(b-a)(a-1)/(2a-2-b+z) + (2a-b+z) + a(b-a-1)/(2a+2-b+z)]M(a,b,z)   -a(a+1)/(2a+2-b+z)M(a+2,b,z)
 200:   //
 201:   template <class T>
 202:   struct hypergeometric_1F1_recurrence_2a_coefficients
 203:   {
 204:      typedef boost::math::tuple<T, T, T> result_type;
 205: 
 206:      hypergeometric_1F1_recurrence_2a_coefficients(const T& a, const T& b, const T& z, int offset = 0) :
 207:         a(a), b(b), z(z), offset(offset)
 208:      {
 209:      }
 210: 
 211:      result_type operator()(std::intmax_t i) const
 212:      {
 213:         i *= 2;
 214:         const T ai = a + (offset + i);
 215:         // -(b-a)(1 + b - a)/(2a-2-b+z)
 216:         const T an = -(b - ai) * (b - (a + (offset + i - 1))) / (2 * (a + (offset + i - 1)) - b + z);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:         const T bn = (b - ai) * (a + (offset + i - 1)) / (2 * (a + (offset + i - 1)) - b + z) + (2 * ai - b + z) + ai * (b - (a + (offset + i + 1))) / (2 * (a + (offset + i + 1)) - b + z);
 218:         const T cn = -ai * (a + (offset + i + 1)) / (2 * (a + (offset + i + 1)) - b + z);
 219: 
 220:         return boost::math::make_tuple(an, bn, cn);
 221:      }
 222: 
 223:   private:
 224:      const T a, b, z;
 225:      int offset;
 226:      hypergeometric_1F1_recurrence_2a_coefficients operator=(const hypergeometric_1F1_recurrence_2a_coefficients&);
 227:   };
 228: 
 229:   //
 230:   // Recurrence relation for double-stepping on b:
 231:   // b(b-1)^2(b-2)/((1-b)(2-b-z)) M(a,b-2,z)  + [zb(b-1)(b-1-a)/((1-b)(2-b-z)) + b(1-b-z) + z(b-a)(b+1)b/((b+1)(b+z)) ] M(a,b,z) + z^2(b-a)(b+1-a)/((b+1)(b+z)) M(a,b+2,z)
 232:   //
 233:   template <class T>
 234:   struct hypergeometric_1F1_recurrence_2b_coefficients
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:   {
 236:      typedef boost::math::tuple<T, T, T> result_type;
 237: 
 238:      hypergeometric_1F1_recurrence_2b_coefficients(const T& a, const T& b, const T& z, int offset = 0) :
 239:         a(a), b(b), z(z), offset(offset)
 240:      {
 241:      }
 242: 
 243:      result_type operator()(std::intmax_t i) const
 244:      {
 245:         i *= 2;
 246:         const T bi = b + (offset + i);
 247:         const T bi_m1 = b + (offset + i - 1);
 248:         const T bi_p1 = b + (offset + i + 1);
 249:         const T bi_m2 = b + (offset + i - 2);
 250: 
 251:         const T an = bi * (bi_m1) * (bi_m1) * (bi_m2) / (-bi_m1 * (-bi_m2 - z));
 252:         const T bn = z * bi * bi_m1 * (bi_m1 - a) / (-bi_m1 * (-bi_m2 - z)) + bi * (-bi_m1 - z) + z * (bi - a) * bi_p1 * bi / (bi_p1 * (bi + z));
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:         const T cn = z * z * (bi - a) * (bi_p1 - a) / (bi_p1 * (bi + z));
 254: 
 255:         return boost::math::make_tuple(an, bn, cn);
 256:      }
 257: 
 258:   private:
 259:      const T a, b, z;
 260:      int offset;
 261:      hypergeometric_1F1_recurrence_2b_coefficients operator=(const hypergeometric_1F1_recurrence_2b_coefficients&);
 262:   };
 263: 
 264:   //
 265:   // Recurrence relation for a+ b-:
 266:   // -z(b-a)(a-1-b)/(b(a-1+z)) M(a-1,b+1,z) + [(b-a)(a-1)b/(b(a-1+z)) + (2a-b+z) + a(b-a-1)/(a+z)] M(a,b,z) + a(1-b)/(a+z) M(a+1,b-1,z)
 267:   //
 268:   // This is potentially the most useful of these novel recurrences.
 269:   //              -                                      -                  +        -                           +
 270:   template <class T>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:   struct hypergeometric_1F1_recurrence_a_plus_b_minus_coefficients
 272:   {
 273:      typedef boost::math::tuple<T, T, T> result_type;
 274: 
 275:      hypergeometric_1F1_recurrence_a_plus_b_minus_coefficients(const T& a, const T& b, const T& z, int offset = 0) :
 276:         a(a), b(b), z(z), offset(offset)
 277:      {
 278:      }
 279: 
 280:      result_type operator()(std::intmax_t i) const
 281:      {
 282:         const T ai = a + (offset + i);
 283:         const T bi = b - (offset + i);
 284: 
 285:         const T an = -z * (bi - ai) * (ai - 1 - bi) / (bi * (ai - 1 + z));
 286:         const T bn = z * ((-1 / (ai + z) - 1 / (ai + z - 1)) * (bi + z - 1) + 3) + bi - 1;
 287:         const T cn = ai * (1 - bi) / (ai + z);
 288: 
~~~
- **EN:** It introduces the struct `hypergeometric_1F1_recurrence_a_plus_b_minus_coefficients` as part of the file's main abstraction. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 它引入了 struct `hypergeometric_1F1_recurrence_a_plus_b_minus_coefficients`，作为该文件核心抽象的一部分。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:         return boost::math::make_tuple(an, bn, cn);
 290:      }
 291: 
 292:   private:
 293:      const T a, b, z;
 294:      int offset;
 295:      hypergeometric_1F1_recurrence_a_plus_b_minus_coefficients operator=(const hypergeometric_1F1_recurrence_a_plus_b_minus_coefficients&);
 296:   };
 297: #endif
 298: 
 299:   template <class T, class Policy>
 300:   inline T hypergeometric_1F1_backward_recurrence_for_negative_a(const T& a, const T& b, const T& z, const Policy& pol, const char* function, long long& log_scaling)
 301:   {
 302:     BOOST_MATH_STD_USING // modf, frexp, fabs, pow
 303: 
 304:     std::intmax_t integer_part = 0;
 305:     T ak = modf(a, &integer_part);
 306:     //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as modf.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 modf。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:     // We need ak-1 positive to avoid infinite recursion below:
 308:     //
 309:     if (0 != ak)
 310:     {
 311:        ak += 2;
 312:        integer_part -= 2;
 313:     }
 314:     if (ak - 1 == b)
 315:     {
 316:        // When ak - 1 == b are recursion coefficients disappear to zero and
 317:        // we end up with a NaN result.  Reduce the recursion steps by 1 to
 318:        // avoid this.  We rely on |b| small and therefore no infinite recursion.
 319:        ak -= 1;
 320:        integer_part += 1;
 321:     }
 322: 
 323:     if (-integer_part > static_cast<std::intmax_t>(policies::get_max_series_iterations<Policy>()))
 324:        return policies::raise_evaluation_error<T>(function, "1F1 arguments sit in a range with a so negative that we have no evaluation method, got a = %1%", std::numeric_limits<T>::quiet_NaN(), pol);
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325: 
 326:     T first {};
 327:     T second {};
 328:     if(ak == 0)
 329:     {
 330:        first = 1;
 331:        ak -= 1;
 332:        second = 1 - z / b;
 333:        if (fabs(second) < 0.5)
 334:           second = (b - z) / b;  // cancellation avoidance
 335:     }
 336:     else
 337:     {
 338:        long long scaling1 {};
 339:        long long scaling2 {};
 340:        first = detail::hypergeometric_1F1_imp(ak, b, z, pol, scaling1);
 341:        ak -= 1;
 342:        second = detail::hypergeometric_1F1_imp(ak, b, z, pol, scaling2);
~~~
- **EN:** This range declares or defines callable logic such as detail::hypergeometric_1F1_imp. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::hypergeometric_1F1_imp。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:        if (scaling1 != scaling2)
 344:        {
 345:           second *= exp(T(scaling2 - scaling1));
 346:        }
 347:        log_scaling += scaling1;
 348:     }
 349:     ++integer_part;
 350: 
 351:     detail::hypergeometric_1F1_recurrence_a_coefficients<T> s(ak, b, z);
 352: 
 353:     return tools::apply_recurrence_relation_backward(s,
 354:                                                      static_cast<unsigned int>(std::abs(integer_part)),
 355:                                                      first,
 356:                                                      second, &log_scaling);
 357:   }
 358: 
 359: 
 360:   template <class T, class Policy>
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as exp, s.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 exp, s。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:   T hypergeometric_1F1_backwards_recursion_on_b_for_negative_a(const T& a, const T& b, const T& z, const Policy& pol, const char*, long long& log_scaling)
 362:   {
 363:      using std::swap;
 364:      BOOST_MATH_STD_USING // modf, frexp, fabs, pow
 365:      //
 366:      // We compute
 367:      //
 368:      // M[a + a_shift, b + b_shift; z]
 369:      //
 370:      // and recurse backwards on a and b down to
 371:      //
 372:      // M[a, b, z]
 373:      //
 374:      // With a + a_shift > 1 and b + b_shift > z
 375:      //
 376:      // There are 3 distinct regions to ensure stability during the recursions:
 377:      //
 378:      // a > 0         :  stable for backwards on a
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:      // a < 0, b > 0  :  stable for backwards on a and b
 380:      // a < 0, b < 0  :  stable for backwards on b (as long as |b| is small).
 381:      //
 382:      // We could simplify things by ignoring the middle region, but it's more efficient
 383:      // to recurse on a and b together when we can.
 384:      //
 385: 
 386:      BOOST_MATH_ASSERT(a < -1); // Not tested nor taken for -1 < a < 0
 387: 
 388:      int b_shift = itrunc(z - b) + 2;
 389: 
 390:      int a_shift = itrunc(-a);
 391:      if (a + a_shift != 0)
 392:      {
 393:         a_shift += 2;
 394:      }
 395:      //
 396:      // If the shifts are so large that we would throw an evaluation_error, try the series instead,
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, itrunc. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, itrunc。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:      // even though this will almost certainly throw as well:
 398:      //
 399:      if (b_shift > static_cast<std::intmax_t>(boost::math::policies::get_max_series_iterations<Policy>()))
 400:         return hypergeometric_1F1_checked_series_impl(a, b, z, pol, log_scaling);
 401: 
 402:      if (a_shift > static_cast<std::intmax_t>(boost::math::policies::get_max_series_iterations<Policy>()))
 403:         return hypergeometric_1F1_checked_series_impl(a, b, z, pol, log_scaling);
 404: 
 405:      int a_b_shift = b < 0 ? itrunc(b + b_shift) : b_shift;   // The max we can shift on a and b together
 406:      int leading_a_shift = (std::min)(3, a_shift);        // Just enough to make a negative
 407:      if (a_b_shift > a_shift - 3)
 408:      {
 409:         a_b_shift = a_shift < 3 ? 0 : a_shift - 3;
 410:      }
 411:      else
 412:      {
 413:         // Need to ensure that leading_a_shift is large enough that a will reach it's target
 414:         // after the first 2 phases (-,0) and (-,-) are over:
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:         leading_a_shift = a_shift - a_b_shift;
 416:      }
 417:      int trailing_b_shift = b_shift - a_b_shift;
 418:      if (a_b_shift < 5)
 419:      {
 420:         // Might as well do things in two steps rather than 3:
 421:         if (a_b_shift > 0)
 422:         {
 423:            leading_a_shift += a_b_shift;
 424:            trailing_b_shift += a_b_shift;
 425:         }
 426:         a_b_shift = 0;
 427:         --leading_a_shift;
 428:      }
 429: 
 430:      BOOST_MATH_ASSERT(leading_a_shift > 1);
 431:      BOOST_MATH_ASSERT(a_b_shift + leading_a_shift + (a_b_shift == 0 ? 1 : 0) == a_shift);
 432:      BOOST_MATH_ASSERT(a_b_shift + trailing_b_shift == b_shift);
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT. Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433: 
 434:      if ((trailing_b_shift == 0) && (fabs(b) < 0.5) && a_b_shift)
 435:      {
 436:         // Better to have the final recursion on b alone, otherwise we lose precision when b is very small:
 437:         int diff = (std::min)(a_b_shift, 3);
 438:         a_b_shift -= diff;
 439:         leading_a_shift += diff;
 440:         trailing_b_shift += diff;
 441:      }
 442: 
 443:      T first {};
 444:      T second {};
 445:      long long scale1 {};
 446:      long long scale2 {};
 447:      first = boost::math::detail::hypergeometric_1F1_imp(T(a + a_shift), T(b + b_shift), z, pol, scale1);
 448:      //
 449:      // It would be good to compute "second" from first and the ratio - unfortunately we are right on the cusp
 450:      // recursion on a switching from stable backwards to stable forwards behaviour and so this is not possible here.
~~~
- **EN:** This range declares or defines callable logic such as boost::math::detail::hypergeometric_1F1_imp. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::detail::hypergeometric_1F1_imp。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:      //
 452:      second = boost::math::detail::hypergeometric_1F1_imp(T(a + a_shift - 1), T(b + b_shift), z, pol, scale2);
 453:      if (scale1 != scale2)
 454:         second *= exp(T(scale2 - scale1));
 455:      log_scaling += scale1;
 456: 
 457:      //
 458:      // Now we have [a + a_shift, b + b_shift, z] and [a + a_shift - 1, b + b_shift, z]
 459:      // and want to recurse until [a + a_shift - leading_a_shift, b + b_shift, z] and [a + a_shift - leadng_a_shift - 1, b + b_shift, z]
 460:      // which is leading_a_shift -1 steps.
 461:      //
 462:      second = boost::math::tools::apply_recurrence_relation_backward(
 463:         hypergeometric_1F1_recurrence_a_coefficients<T>(a + a_shift - 1, b + b_shift, z),
 464:         leading_a_shift, first, second, &log_scaling, &first);
 465: 
 466:      if (a_b_shift)
 467:      {
 468:         //
~~~
- **EN:** This range declares or defines callable logic such as boost::math::detail::hypergeometric_1F1_imp, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::detail::hypergeometric_1F1_imp, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:         // Now we need to switch to an a+b shift so that we have:
 470:         // [a + a_shift - leading_a_shift, b + b_shift, z] and [a + a_shift - leadng_a_shift - 1, b + b_shift - 1, z]
 471:         // A&S 13.4.3 gives us what we need:
 472:         //
 473:         {
 474:            // local a's and b's:
 475:            T la = a + a_shift - leading_a_shift - 1;
 476:            T lb = b + b_shift;
 477:            second = ((1 + la - lb) * second - la * first) / (1 - lb);
 478:         }
 479:         //
 480:         // Now apply a_b_shift - 1 recursions to get down to
 481:         // [a + 1, b + trailing_b_shift + 1, z] and [a, b + trailing_b_shift, z]
 482:         //
 483:         second = boost::math::tools::apply_recurrence_relation_backward(
 484:            hypergeometric_1F1_recurrence_a_and_b_coefficients<T>(a, b + b_shift - a_b_shift, z, a_b_shift - 1),
 485:            a_b_shift - 1, first, second, &log_scaling, &first);
 486:         //
~~~
- **EN:** This range continues the current implementation with a mix of declarations, structure, and low-level logic.
- **CN:** 此范围继续当前实现，混合了声明、结构与底层逻辑。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:         // Now we need to switch to a b shift, a different application of A&S 13.4.3
 488:         // will get us there, we leave "second" where it is, and move "first" sideways:
 489:         //
 490:         {
 491:            T lb = b + trailing_b_shift + 1;
 492:            first = (second * (lb - 1) - a * first) / -(1 + a - lb);
 493:         }
 494:      }
 495:      else
 496:      {
 497:         //
 498:         // We have M[a+1, b+b_shift, z] and M[a, b+b_shift, z] and need M[a, b+b_shift-1, z] for
 499:         // recursion on b: A&S 13.4.3 gives us what we need.
 500:         //
 501:         T third = -(second * (1 + a - b - b_shift) - first * a) / (b + b_shift - 1);
 502:         swap(first, second);
 503:         swap(second, third);
 504:         --trailing_b_shift;
~~~
- **EN:** This range declares or defines callable logic such as swap.
- **CN:** 此范围声明或定义了可调用逻辑，例如 swap。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:      }
 506:      //
 507:      // Finish off by applying trailing_b_shift recursions:
 508:      //
 509:      if (trailing_b_shift)
 510:      {
 511:         second = boost::math::tools::apply_recurrence_relation_backward(
 512:            hypergeometric_1F1_recurrence_small_b_coefficients<T>(a, b, z, trailing_b_shift),
 513:            trailing_b_shift, first, second, &log_scaling);
 514:      }
 515:      return second;
 516:   }
 517: 
 518: 
 519: 
 520:   } } } // namespaces
 521: 
 522: #endif // BOOST_HYPERGEOMETRIC_1F1_RECURRENCE_HPP_
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

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
- **Included headers / 包含头文件**: `boost/math/special_functions/modf.hpp, boost/math/special_functions/next.hpp, boost/math/tools/recurrence.hpp, boost/math/special_functions/detail/hypergeometric_pFq_checked_series.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `hypergeometric_1F1_imp, modf, detail::hypergeometric_1F1_imp, exp, s, BOOST_MATH_ASSERT, itrunc, boost::math::detail::hypergeometric_1F1_imp, ...`
