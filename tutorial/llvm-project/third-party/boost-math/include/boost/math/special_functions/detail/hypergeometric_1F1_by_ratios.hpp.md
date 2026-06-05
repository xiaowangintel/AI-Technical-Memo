# hypergeometric_1F1_by_ratios.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/hypergeometric_1F1_by_ratios.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the hypergeometric 1F1 by ratios special-function path.
- **作用（中文）**: 此头文件为 hypergeometric 1F1 by ratios 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: 
   2: ///////////////////////////////////////////////////////////////////////////////
   3: //  Copyright 2018 John Maddock
   4: //  Distributed under the Boost
   5: //  Software License, Version 1.0. (See accompanying file
   6: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   7: 
   8: #ifndef BOOST_HYPERGEOMETRIC_1F1_BY_RATIOS_HPP_
   9: #define BOOST_HYPERGEOMETRIC_1F1_BY_RATIOS_HPP_
  10: 
  11: #include <boost/math/tools/recurrence.hpp>
  12: #include <boost/math/policies/error_handling.hpp>
  13: 
  14:   namespace boost { namespace math { namespace detail {
  15: 
  16:      template <class T, class Policy>
  17:      T hypergeometric_1F1_imp(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling);
  18: 
~~~
- **EN:** This block imports dependencies such as boost/math/tools/recurrence.hpp, boost/math/policies/error_handling.hpp so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/tools/recurrence.hpp, boost/math/policies/error_handling.hpp 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19:      /*
  20:       Evaluation by method of ratios for domain b < 0 < a,z
  21: 
  22:       We first convert the recurrence relation into a ratio
  23:       of M(a+1, b+1, z) / M(a, b, z) using Shintan's equivalence
  24:       between solving a recurrence relation using Miller's method
  25:       and continued fractions.  The continued fraction is VERY rapid
  26:       to converge (typically < 10 terms), but may converge to entirely
  27:       the wrong value if we're in a bad part of the domain.  Strangely
  28:       it seems to matter not whether we use recurrence on a, b or a and b
  29:       they all work or not work about the same, so we might as well make
  30:       life easy for ourselves and use the a and b recurrence to avoid
  31:       having to apply one extra recurrence to convert from an a or b
  32:       recurrence to an a+b one.
  33: 
  34:       See: H. Shintan, Note on Miller's recurrence algorithm, J. Sci. Hiroshima Univ. Ser. A-I Math., 29 (1965), pp. 121-133.
  35:       Also: Computational Aspects of Three Term Recurrence Relations, SIAM Review, January 1967.
  36: 
~~~
- **EN:** This range continues the current implementation with a mix of declarations, structure, and low-level logic.
- **CN:** 此范围继续当前实现，混合了声明、结构与底层逻辑。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:       The following table lists by experiment, how large z can be in order to
  38:       ensure the continued fraction converges to the correct value:
  39: 
  40:           a         b    max  z
  41:          13,      -130,      22
  42:          13,     -1300,     335
  43:          13,    -13000,    3585
  44:         130,      -130,       8
  45:         130,     -1300,     263
  46:         130,   - 13000,    3420
  47:        1300,      -130,       1
  48:        1300,     -1300,      90
  49:        1300,    -13000,    2650
  50:       13000,       -13,       -
  51:       13000,      -130,       -
  52:       13000,     -1300,      13
  53:       13000,    -13000,     750
  54: 
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:       So try z_limit = -b / (4 - 5 * sqrt(log(a)) * a / b);
  56:       or     z_limit = -b / (4 - 5 * (log(a)) * a / b)  for a < 100
  57: 
  58:       This still isn't quite right for both a and b small, but we'll be using a Bessel approximation
  59:       in that region anyway.
  60: 
  61:       Normalization using wronksian {1,2} from A&S 13.1.20 (also 13.1.12, 13.1.13):
  62: 
  63:       W{ M(a,b,z), z^(1-b)M(1+a-b, 2-b, z) } = (1-b)z^-b e^z
  64: 
  65:        = M(a,b,z) M2'(a,b,z) - M'(a,b,z) M2(a,b,z)
  66:        = M(a,b,z) [(a-b+1)z^(1-b)/(2-b) M2(a+1,b+1,z) + (1-b)z^-b M2(a,b,z)] - a/b M(a+1,b+1,z) z^(1-b)M2(a,b,z)
  67:        = M(a,b,z) [(a-b+1)z^(1-b)/(2-b) M2(a+1,b+1,z) + (1-b)z^-b M2(a,b,z)] - a/b R(a,b,z) M(a,b,z) z^(1-b)M2(a,b,z)
  68:        = M(a,b,z) [(a-b+1)z^(1-b)/(2-b) M2(a+1,b+1,z) + (1-b)z^-b M2(a,b,z) - a/b R(a,b,z) z^(1-b)M2(a,b,z) ]
  69:        so:
  70:        (1-b)e^z = M(a,b,z) [(a-b+1)z/(2-b) M2(a+1,b+1,z) + (1-b) M2(a,b,z) - a/b z R(a,b,z) M2(a,b,z) ]
  71: 
  72:       */
~~~
- **EN:** This range declares or defines callable logic such as sqrt. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73: 
  74:      template <class T>
  75:      inline bool is_in_hypergeometric_1F1_from_function_ratio_negative_b_region(const T& a, const T& b, const T& z)
  76:      {
  77:         BOOST_MATH_STD_USING
  78:         if (a < 100)
  79:            return z < -b / (4 - 5 * (log(a)) * a / b);
  80:         else
  81:            return z < -b / (4 - 5 * sqrt(log(a)) * a / b);
  82:      }
  83: 
  84:      template <class T, class Policy>
  85:      T hypergeometric_1F1_from_function_ratio_negative_b(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling, const T& ratio)
  86:      {
  87:         BOOST_MATH_STD_USING
  88:         //
  89:         // Let M2 = M(1+a-b, 2-b, z)
  90:         // This is going to be a mighty big number:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:         //
  92:         long long local_scaling = 0;
  93:         T M2 = boost::math::detail::hypergeometric_1F1_imp(T(1 + a - b), T(2 - b), z, pol, local_scaling);
  94:         log_scaling -= local_scaling; // all the M2 terms are in the denominator
  95:         //
  96:         // Since a, b and z are all likely to be large we need the Wronksian
  97:         // calculation below to not overflow, so scale everything right down:
  98:         //
  99:         if (fabs(M2) > 1)
 100:         {
 101:            long long s = lltrunc(log(fabs(M2)));
 102:            log_scaling -= s;  // M2 will be in the denominator, so subtract the scaling!
 103:            local_scaling += s;
 104:            M2 *= exp(T(-s));
 105:         }
 106:         //
 107:         // Let M3 = M(1+a-b + 1, 2-b + 1, z)
 108:         // we can get to this from the ratio which is cheaper to calculate:
~~~
- **EN:** This range declares or defines callable logic such as boost::math::detail::hypergeometric_1F1_imp, lltrunc, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::detail::hypergeometric_1F1_imp, lltrunc, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:         //
 110:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 111:         boost::math::detail::hypergeometric_1F1_recurrence_a_and_b_coefficients<T> coef2(1 + a - b + 1, 2 - b + 1, z);
 112:         T M3 = boost::math::tools::function_ratio_from_backwards_recurrence(coef2, boost::math::policies::get_epsilon<T, Policy>(), max_iter) * M2;
 113:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_from_function_ratio_negative_b_positive_a<%1%>(%1%,%1%,%1%)", max_iter, pol);
 114:         //
 115:         // Get the RHS of the Wronksian:
 116:         //
 117:         long long fz = lltrunc(z);
 118:         log_scaling += fz;
 119:         T rhs = (1 - b) * exp(z - fz);
 120:         //
 121:         // We need to divide that by:
 122:         // [(a-b+1)z/(2-b) M2(a+1,b+1,z) + (1-b) M2(a,b,z) - a/b z^b R(a,b,z) M2(a,b,z) ]
 123:         // Note that at this stage, both M3 and M2 are scaled by exp(local_scaling).
 124:         //
 125:         T lhs = (a - b + 1) * z * M3 / (2 - b) + (1 - b) * M2 - a * z * ratio * M2 / b;
 126: 
~~~
- **EN:** This range declares or defines callable logic such as coef2, lltrunc, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围声明或定义了可调用逻辑，例如 coef2, lltrunc, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:         return rhs / lhs;
 128:      }
 129: 
 130:      template <class T, class Policy>
 131:      T hypergeometric_1F1_from_function_ratio_negative_b(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling)
 132:      {
 133:         BOOST_MATH_STD_USING
 134:         //
 135:         // Get the function ratio, M(a+1, b+1, z)/M(a,b,z):
 136:         //
 137:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 138:         boost::math::detail::hypergeometric_1F1_recurrence_a_and_b_coefficients<T> coef(a + 1, b + 1, z);
 139:         T ratio = boost::math::tools::function_ratio_from_backwards_recurrence(coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 140:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_from_function_ratio_negative_b_positive_a<%1%>(%1%,%1%,%1%)", max_iter, pol);
 141:         return hypergeometric_1F1_from_function_ratio_negative_b(a, b, z, pol, log_scaling, ratio);
 142:      }
 143:      //
 144:      // And over again, this time via forwards recurrence when z is large enough:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as coef, boost::math::tools::function_ratio_from_backwards_recurrence.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 coef, boost::math::tools::function_ratio_from_backwards_recurrence。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:      //
 146:      template <class T>
 147:      bool hypergeometric_1F1_is_in_forwards_recurence_for_negative_b_region(const T& a, const T& b, const T& z)
 148:      {
 149:         //
 150:         // There's no easy relation between a, b and z that tells us whether we're in the region
 151:         // where forwards recursion is stable, so use a lookup table, note that the minimum
 152:         // permissible z-value is decreasing with a, and increasing with |b|:
 153:         //
 154:         static const float data[][3] = {
 155:            {7.500e+00f, -7.500e+00f, 8.409e+00f },
 156:            {7.500e+00f, -1.125e+01f, 8.409e+00f },
 157:            {7.500e+00f, -1.688e+01f, 9.250e+00f },
 158:            {7.500e+00f, -2.531e+01f, 1.119e+01f },
 159:            {7.500e+00f, -3.797e+01f, 1.354e+01f },
 160:            {7.500e+00f, -5.695e+01f, 1.983e+01f },
 161:            {7.500e+00f, -8.543e+01f, 2.639e+01f },
 162:            {7.500e+00f, -1.281e+02f, 3.864e+01f },
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:            {7.500e+00f, -1.922e+02f, 5.657e+01f },
 164:            {7.500e+00f, -2.883e+02f, 8.283e+01f },
 165:            {7.500e+00f, -4.325e+02f, 1.213e+02f },
 166:            {7.500e+00f, -6.487e+02f, 1.953e+02f },
 167:            {7.500e+00f, -9.731e+02f, 2.860e+02f },
 168:            {7.500e+00f, -1.460e+03f, 4.187e+02f },
 169:            {7.500e+00f, -2.189e+03f, 6.130e+02f },
 170:            {7.500e+00f, -3.284e+03f, 9.872e+02f },
 171:            {7.500e+00f, -4.926e+03f, 1.445e+03f },
 172:            {7.500e+00f, -7.389e+03f, 2.116e+03f },
 173:            {7.500e+00f, -1.108e+04f, 3.098e+03f },
 174:            {7.500e+00f, -1.663e+04f, 4.990e+03f },
 175:            {1.125e+01f, -7.500e+00f, 6.318e+00f },
 176:            {1.125e+01f, -1.125e+01f, 6.950e+00f },
 177:            {1.125e+01f, -1.688e+01f, 7.645e+00f },
 178:            {1.125e+01f, -2.531e+01f, 9.250e+00f },
 179:            {1.125e+01f, -3.797e+01f, 1.231e+01f },
 180:            {1.125e+01f, -5.695e+01f, 1.639e+01f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:            {1.125e+01f, -8.543e+01f, 2.399e+01f },
 182:            {1.125e+01f, -1.281e+02f, 3.513e+01f },
 183:            {1.125e+01f, -1.922e+02f, 5.657e+01f },
 184:            {1.125e+01f, -2.883e+02f, 8.283e+01f },
 185:            {1.125e+01f, -4.325e+02f, 1.213e+02f },
 186:            {1.125e+01f, -6.487e+02f, 1.776e+02f },
 187:            {1.125e+01f, -9.731e+02f, 2.860e+02f },
 188:            {1.125e+01f, -1.460e+03f, 4.187e+02f },
 189:            {1.125e+01f, -2.189e+03f, 6.130e+02f },
 190:            {1.125e+01f, -3.284e+03f, 9.872e+02f },
 191:            {1.125e+01f, -4.926e+03f, 1.445e+03f },
 192:            {1.125e+01f, -7.389e+03f, 2.116e+03f },
 193:            {1.125e+01f, -1.108e+04f, 3.098e+03f },
 194:            {1.125e+01f, -1.663e+04f, 4.990e+03f },
 195:            {1.688e+01f, -7.500e+00f, 4.747e+00f },
 196:            {1.688e+01f, -1.125e+01f, 5.222e+00f },
 197:            {1.688e+01f, -1.688e+01f, 5.744e+00f },
 198:            {1.688e+01f, -2.531e+01f, 7.645e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:            {1.688e+01f, -3.797e+01f, 1.018e+01f },
 200:            {1.688e+01f, -5.695e+01f, 1.490e+01f },
 201:            {1.688e+01f, -8.543e+01f, 2.181e+01f },
 202:            {1.688e+01f, -1.281e+02f, 3.193e+01f },
 203:            {1.688e+01f, -1.922e+02f, 5.143e+01f },
 204:            {1.688e+01f, -2.883e+02f, 7.530e+01f },
 205:            {1.688e+01f, -4.325e+02f, 1.213e+02f },
 206:            {1.688e+01f, -6.487e+02f, 1.776e+02f },
 207:            {1.688e+01f, -9.731e+02f, 2.600e+02f },
 208:            {1.688e+01f, -1.460e+03f, 4.187e+02f },
 209:            {1.688e+01f, -2.189e+03f, 6.130e+02f },
 210:            {1.688e+01f, -3.284e+03f, 9.872e+02f },
 211:            {1.688e+01f, -4.926e+03f, 1.445e+03f },
 212:            {1.688e+01f, -7.389e+03f, 2.116e+03f },
 213:            {1.688e+01f, -1.108e+04f, 3.098e+03f },
 214:            {1.688e+01f, -1.663e+04f, 4.990e+03f },
 215:            {2.531e+01f, -7.500e+00f, 3.242e+00f },
 216:            {2.531e+01f, -1.125e+01f, 3.566e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:            {2.531e+01f, -1.688e+01f, 4.315e+00f },
 218:            {2.531e+01f, -2.531e+01f, 5.744e+00f },
 219:            {2.531e+01f, -3.797e+01f, 7.645e+00f },
 220:            {2.531e+01f, -5.695e+01f, 1.231e+01f },
 221:            {2.531e+01f, -8.543e+01f, 1.803e+01f },
 222:            {2.531e+01f, -1.281e+02f, 2.903e+01f },
 223:            {2.531e+01f, -1.922e+02f, 4.676e+01f },
 224:            {2.531e+01f, -2.883e+02f, 6.845e+01f },
 225:            {2.531e+01f, -4.325e+02f, 1.102e+02f },
 226:            {2.531e+01f, -6.487e+02f, 1.776e+02f },
 227:            {2.531e+01f, -9.731e+02f, 2.600e+02f },
 228:            {2.531e+01f, -1.460e+03f, 4.187e+02f },
 229:            {2.531e+01f, -2.189e+03f, 6.130e+02f },
 230:            {2.531e+01f, -3.284e+03f, 8.974e+02f },
 231:            {2.531e+01f, -4.926e+03f, 1.445e+03f },
 232:            {2.531e+01f, -7.389e+03f, 2.116e+03f },
 233:            {2.531e+01f, -1.108e+04f, 3.098e+03f },
 234:            {2.531e+01f, -1.663e+04f, 4.990e+03f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:            {3.797e+01f, -7.500e+00f, 2.214e+00f },
 236:            {3.797e+01f, -1.125e+01f, 2.679e+00f },
 237:            {3.797e+01f, -1.688e+01f, 3.242e+00f },
 238:            {3.797e+01f, -2.531e+01f, 4.315e+00f },
 239:            {3.797e+01f, -3.797e+01f, 6.318e+00f },
 240:            {3.797e+01f, -5.695e+01f, 9.250e+00f },
 241:            {3.797e+01f, -8.543e+01f, 1.490e+01f },
 242:            {3.797e+01f, -1.281e+02f, 2.399e+01f },
 243:            {3.797e+01f, -1.922e+02f, 3.864e+01f },
 244:            {3.797e+01f, -2.883e+02f, 6.223e+01f },
 245:            {3.797e+01f, -4.325e+02f, 1.002e+02f },
 246:            {3.797e+01f, -6.487e+02f, 1.614e+02f },
 247:            {3.797e+01f, -9.731e+02f, 2.600e+02f },
 248:            {3.797e+01f, -1.460e+03f, 3.806e+02f },
 249:            {3.797e+01f, -2.189e+03f, 6.130e+02f },
 250:            {3.797e+01f, -3.284e+03f, 8.974e+02f },
 251:            {3.797e+01f, -4.926e+03f, 1.445e+03f },
 252:            {3.797e+01f, -7.389e+03f, 2.116e+03f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:            {3.797e+01f, -1.108e+04f, 3.098e+03f },
 254:            {3.797e+01f, -1.663e+04f, 4.990e+03f },
 255:            {5.695e+01f, -7.500e+00f, 1.513e+00f },
 256:            {5.695e+01f, -1.125e+01f, 1.830e+00f },
 257:            {5.695e+01f, -1.688e+01f, 2.214e+00f },
 258:            {5.695e+01f, -2.531e+01f, 3.242e+00f },
 259:            {5.695e+01f, -3.797e+01f, 4.315e+00f },
 260:            {5.695e+01f, -5.695e+01f, 7.645e+00f },
 261:            {5.695e+01f, -8.543e+01f, 1.231e+01f },
 262:            {5.695e+01f, -1.281e+02f, 1.983e+01f },
 263:            {5.695e+01f, -1.922e+02f, 3.513e+01f },
 264:            {5.695e+01f, -2.883e+02f, 5.657e+01f },
 265:            {5.695e+01f, -4.325e+02f, 9.111e+01f },
 266:            {5.695e+01f, -6.487e+02f, 1.467e+02f },
 267:            {5.695e+01f, -9.731e+02f, 2.363e+02f },
 268:            {5.695e+01f, -1.460e+03f, 3.806e+02f },
 269:            {5.695e+01f, -2.189e+03f, 5.572e+02f },
 270:            {5.695e+01f, -3.284e+03f, 8.974e+02f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:            {5.695e+01f, -4.926e+03f, 1.314e+03f },
 272:            {5.695e+01f, -7.389e+03f, 2.116e+03f },
 273:            {5.695e+01f, -1.108e+04f, 3.098e+03f },
 274:            {5.695e+01f, -1.663e+04f, 4.990e+03f },
 275:            {8.543e+01f, -7.500e+00f, 1.250e+00f },
 276:            {8.543e+01f, -1.125e+01f, 1.250e+00f },
 277:            {8.543e+01f, -1.688e+01f, 1.513e+00f },
 278:            {8.543e+01f, -2.531e+01f, 2.214e+00f },
 279:            {8.543e+01f, -3.797e+01f, 3.242e+00f },
 280:            {8.543e+01f, -5.695e+01f, 5.222e+00f },
 281:            {8.543e+01f, -8.543e+01f, 9.250e+00f },
 282:            {8.543e+01f, -1.281e+02f, 1.639e+01f },
 283:            {8.543e+01f, -1.922e+02f, 2.903e+01f },
 284:            {8.543e+01f, -2.883e+02f, 5.143e+01f },
 285:            {8.543e+01f, -4.325e+02f, 8.283e+01f },
 286:            {8.543e+01f, -6.487e+02f, 1.334e+02f },
 287:            {8.543e+01f, -9.731e+02f, 2.148e+02f },
 288:            {8.543e+01f, -1.460e+03f, 3.460e+02f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:            {8.543e+01f, -2.189e+03f, 5.572e+02f },
 290:            {8.543e+01f, -3.284e+03f, 8.974e+02f },
 291:            {8.543e+01f, -4.926e+03f, 1.314e+03f },
 292:            {8.543e+01f, -7.389e+03f, 2.116e+03f },
 293:            {8.543e+01f, -1.108e+04f, 3.098e+03f },
 294:            {8.543e+01f, -1.663e+04f, 4.536e+03f },
 295:            {1.281e+02f, -7.500e+00f, 1.250e+00f },
 296:            {1.281e+02f, -1.125e+01f, 1.250e+00f },
 297:            {1.281e+02f, -1.688e+01f, 1.250e+00f },
 298:            {1.281e+02f, -2.531e+01f, 1.513e+00f },
 299:            {1.281e+02f, -3.797e+01f, 2.214e+00f },
 300:            {1.281e+02f, -5.695e+01f, 3.923e+00f },
 301:            {1.281e+02f, -8.543e+01f, 6.950e+00f },
 302:            {1.281e+02f, -1.281e+02f, 1.231e+01f },
 303:            {1.281e+02f, -1.922e+02f, 2.181e+01f },
 304:            {1.281e+02f, -2.883e+02f, 4.250e+01f },
 305:            {1.281e+02f, -4.325e+02f, 6.845e+01f },
 306:            {1.281e+02f, -6.487e+02f, 1.213e+02f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:            {1.281e+02f, -9.731e+02f, 1.953e+02f },
 308:            {1.281e+02f, -1.460e+03f, 3.460e+02f },
 309:            {1.281e+02f, -2.189e+03f, 5.572e+02f },
 310:            {1.281e+02f, -3.284e+03f, 8.159e+02f },
 311:            {1.281e+02f, -4.926e+03f, 1.314e+03f },
 312:            {1.281e+02f, -7.389e+03f, 1.924e+03f },
 313:            {1.281e+02f, -1.108e+04f, 3.098e+03f },
 314:            {1.281e+02f, -1.663e+04f, 4.536e+03f },
 315:            {1.922e+02f, -7.500e+00f, 1.250e+00f },
 316:            {1.922e+02f, -1.125e+01f, 1.250e+00f },
 317:            {1.922e+02f, -1.688e+01f, 1.250e+00f },
 318:            {1.922e+02f, -2.531e+01f, 1.250e+00f },
 319:            {1.922e+02f, -3.797e+01f, 1.664e+00f },
 320:            {1.922e+02f, -5.695e+01f, 2.679e+00f },
 321:            {1.922e+02f, -8.543e+01f, 5.222e+00f },
 322:            {1.922e+02f, -1.281e+02f, 9.250e+00f },
 323:            {1.922e+02f, -1.922e+02f, 1.803e+01f },
 324:            {1.922e+02f, -2.883e+02f, 3.193e+01f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:            {1.922e+02f, -4.325e+02f, 5.657e+01f },
 326:            {1.922e+02f, -6.487e+02f, 1.002e+02f },
 327:            {1.922e+02f, -9.731e+02f, 1.776e+02f },
 328:            {1.922e+02f, -1.460e+03f, 3.145e+02f },
 329:            {1.922e+02f, -2.189e+03f, 5.066e+02f },
 330:            {1.922e+02f, -3.284e+03f, 8.159e+02f },
 331:            {1.922e+02f, -4.926e+03f, 1.194e+03f },
 332:            {1.922e+02f, -7.389e+03f, 1.924e+03f },
 333:            {1.922e+02f, -1.108e+04f, 3.098e+03f },
 334:            {1.922e+02f, -1.663e+04f, 4.536e+03f },
 335:            {2.883e+02f, -7.500e+00f, 1.250e+00f },
 336:            {2.883e+02f, -1.125e+01f, 1.250e+00f },
 337:            {2.883e+02f, -1.688e+01f, 1.250e+00f },
 338:            {2.883e+02f, -2.531e+01f, 1.250e+00f },
 339:            {2.883e+02f, -3.797e+01f, 1.250e+00f },
 340:            {2.883e+02f, -5.695e+01f, 2.013e+00f },
 341:            {2.883e+02f, -8.543e+01f, 3.566e+00f },
 342:            {2.883e+02f, -1.281e+02f, 6.950e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:            {2.883e+02f, -1.922e+02f, 1.354e+01f },
 344:            {2.883e+02f, -2.883e+02f, 2.399e+01f },
 345:            {2.883e+02f, -4.325e+02f, 4.676e+01f },
 346:            {2.883e+02f, -6.487e+02f, 8.283e+01f },
 347:            {2.883e+02f, -9.731e+02f, 1.614e+02f },
 348:            {2.883e+02f, -1.460e+03f, 2.600e+02f },
 349:            {2.883e+02f, -2.189e+03f, 4.605e+02f },
 350:            {2.883e+02f, -3.284e+03f, 7.417e+02f },
 351:            {2.883e+02f, -4.926e+03f, 1.194e+03f },
 352:            {2.883e+02f, -7.389e+03f, 1.924e+03f },
 353:            {2.883e+02f, -1.108e+04f, 2.817e+03f },
 354:            {2.883e+02f, -1.663e+04f, 4.536e+03f },
 355:            {4.325e+02f, -7.500e+00f, 1.250e+00f },
 356:            {4.325e+02f, -1.125e+01f, 1.250e+00f },
 357:            {4.325e+02f, -1.688e+01f, 1.250e+00f },
 358:            {4.325e+02f, -2.531e+01f, 1.250e+00f },
 359:            {4.325e+02f, -3.797e+01f, 1.250e+00f },
 360:            {4.325e+02f, -5.695e+01f, 1.375e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:            {4.325e+02f, -8.543e+01f, 2.436e+00f },
 362:            {4.325e+02f, -1.281e+02f, 4.747e+00f },
 363:            {4.325e+02f, -1.922e+02f, 9.250e+00f },
 364:            {4.325e+02f, -2.883e+02f, 1.803e+01f },
 365:            {4.325e+02f, -4.325e+02f, 3.513e+01f },
 366:            {4.325e+02f, -6.487e+02f, 6.845e+01f },
 367:            {4.325e+02f, -9.731e+02f, 1.334e+02f },
 368:            {4.325e+02f, -1.460e+03f, 2.363e+02f },
 369:            {4.325e+02f, -2.189e+03f, 3.806e+02f },
 370:            {4.325e+02f, -3.284e+03f, 6.743e+02f },
 371:            {4.325e+02f, -4.926e+03f, 1.086e+03f },
 372:            {4.325e+02f, -7.389e+03f, 1.749e+03f },
 373:            {4.325e+02f, -1.108e+04f, 2.817e+03f },
 374:            {4.325e+02f, -1.663e+04f, 4.536e+03f },
 375:            {6.487e+02f, -7.500e+00f, 1.250e+00f },
 376:            {6.487e+02f, -1.125e+01f, 1.250e+00f },
 377:            {6.487e+02f, -1.688e+01f, 1.250e+00f },
 378:            {6.487e+02f, -2.531e+01f, 1.250e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:            {6.487e+02f, -3.797e+01f, 1.250e+00f },
 380:            {6.487e+02f, -5.695e+01f, 1.250e+00f },
 381:            {6.487e+02f, -8.543e+01f, 1.664e+00f },
 382:            {6.487e+02f, -1.281e+02f, 3.242e+00f },
 383:            {6.487e+02f, -1.922e+02f, 6.950e+00f },
 384:            {6.487e+02f, -2.883e+02f, 1.354e+01f },
 385:            {6.487e+02f, -4.325e+02f, 2.639e+01f },
 386:            {6.487e+02f, -6.487e+02f, 5.143e+01f },
 387:            {6.487e+02f, -9.731e+02f, 1.002e+02f },
 388:            {6.487e+02f, -1.460e+03f, 1.953e+02f },
 389:            {6.487e+02f, -2.189e+03f, 3.460e+02f },
 390:            {6.487e+02f, -3.284e+03f, 6.130e+02f },
 391:            {6.487e+02f, -4.926e+03f, 9.872e+02f },
 392:            {6.487e+02f, -7.389e+03f, 1.590e+03f },
 393:            {6.487e+02f, -1.108e+04f, 2.561e+03f },
 394:            {6.487e+02f, -1.663e+04f, 4.124e+03f },
 395:            {9.731e+02f, -7.500e+00f, 1.250e+00f },
 396:            {9.731e+02f, -1.125e+01f, 1.250e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:            {9.731e+02f, -1.688e+01f, 1.250e+00f },
 398:            {9.731e+02f, -2.531e+01f, 1.250e+00f },
 399:            {9.731e+02f, -3.797e+01f, 1.250e+00f },
 400:            {9.731e+02f, -5.695e+01f, 1.250e+00f },
 401:            {9.731e+02f, -8.543e+01f, 1.250e+00f },
 402:            {9.731e+02f, -1.281e+02f, 2.214e+00f },
 403:            {9.731e+02f, -1.922e+02f, 4.747e+00f },
 404:            {9.731e+02f, -2.883e+02f, 9.250e+00f },
 405:            {9.731e+02f, -4.325e+02f, 1.983e+01f },
 406:            {9.731e+02f, -6.487e+02f, 3.864e+01f },
 407:            {9.731e+02f, -9.731e+02f, 7.530e+01f },
 408:            {9.731e+02f, -1.460e+03f, 1.467e+02f },
 409:            {9.731e+02f, -2.189e+03f, 2.860e+02f },
 410:            {9.731e+02f, -3.284e+03f, 5.066e+02f },
 411:            {9.731e+02f, -4.926e+03f, 8.974e+02f },
 412:            {9.731e+02f, -7.389e+03f, 1.445e+03f },
 413:            {9.731e+02f, -1.108e+04f, 2.561e+03f },
 414:            {9.731e+02f, -1.663e+04f, 4.124e+03f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:            {1.460e+03f, -7.500e+00f, 1.250e+00f },
 416:            {1.460e+03f, -1.125e+01f, 1.250e+00f },
 417:            {1.460e+03f, -1.688e+01f, 1.250e+00f },
 418:            {1.460e+03f, -2.531e+01f, 1.250e+00f },
 419:            {1.460e+03f, -3.797e+01f, 1.250e+00f },
 420:            {1.460e+03f, -5.695e+01f, 1.250e+00f },
 421:            {1.460e+03f, -8.543e+01f, 1.250e+00f },
 422:            {1.460e+03f, -1.281e+02f, 1.513e+00f },
 423:            {1.460e+03f, -1.922e+02f, 3.242e+00f },
 424:            {1.460e+03f, -2.883e+02f, 6.950e+00f },
 425:            {1.460e+03f, -4.325e+02f, 1.354e+01f },
 426:            {1.460e+03f, -6.487e+02f, 2.903e+01f },
 427:            {1.460e+03f, -9.731e+02f, 5.657e+01f },
 428:            {1.460e+03f, -1.460e+03f, 1.213e+02f },
 429:            {1.460e+03f, -2.189e+03f, 2.148e+02f },
 430:            {1.460e+03f, -3.284e+03f, 4.187e+02f },
 431:            {1.460e+03f, -4.926e+03f, 7.417e+02f },
 432:            {1.460e+03f, -7.389e+03f, 1.314e+03f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:            {1.460e+03f, -1.108e+04f, 2.328e+03f },
 434:            {1.460e+03f, -1.663e+04f, 3.749e+03f },
 435:            {2.189e+03f, -7.500e+00f, 1.250e+00f },
 436:            {2.189e+03f, -1.125e+01f, 1.250e+00f },
 437:            {2.189e+03f, -1.688e+01f, 1.250e+00f },
 438:            {2.189e+03f, -2.531e+01f, 1.250e+00f },
 439:            {2.189e+03f, -3.797e+01f, 1.250e+00f },
 440:            {2.189e+03f, -5.695e+01f, 1.250e+00f },
 441:            {2.189e+03f, -8.543e+01f, 1.250e+00f },
 442:            {2.189e+03f, -1.281e+02f, 1.250e+00f },
 443:            {2.189e+03f, -1.922e+02f, 2.214e+00f },
 444:            {2.189e+03f, -2.883e+02f, 4.747e+00f },
 445:            {2.189e+03f, -4.325e+02f, 9.250e+00f },
 446:            {2.189e+03f, -6.487e+02f, 1.983e+01f },
 447:            {2.189e+03f, -9.731e+02f, 4.250e+01f },
 448:            {2.189e+03f, -1.460e+03f, 8.283e+01f },
 449:            {2.189e+03f, -2.189e+03f, 1.776e+02f },
 450:            {2.189e+03f, -3.284e+03f, 3.460e+02f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:            {2.189e+03f, -4.926e+03f, 6.130e+02f },
 452:            {2.189e+03f, -7.389e+03f, 1.086e+03f },
 453:            {2.189e+03f, -1.108e+04f, 1.924e+03f },
 454:            {2.189e+03f, -1.663e+04f, 3.408e+03f },
 455:            {3.284e+03f, -7.500e+00f, 1.250e+00f },
 456:            {3.284e+03f, -1.125e+01f, 1.250e+00f },
 457:            {3.284e+03f, -1.688e+01f, 1.250e+00f },
 458:            {3.284e+03f, -2.531e+01f, 1.250e+00f },
 459:            {3.284e+03f, -3.797e+01f, 1.250e+00f },
 460:            {3.284e+03f, -5.695e+01f, 1.250e+00f },
 461:            {3.284e+03f, -8.543e+01f, 1.250e+00f },
 462:            {3.284e+03f, -1.281e+02f, 1.250e+00f },
 463:            {3.284e+03f, -1.922e+02f, 1.513e+00f },
 464:            {3.284e+03f, -2.883e+02f, 3.242e+00f },
 465:            {3.284e+03f, -4.325e+02f, 6.318e+00f },
 466:            {3.284e+03f, -6.487e+02f, 1.354e+01f },
 467:            {3.284e+03f, -9.731e+02f, 2.903e+01f },
 468:            {3.284e+03f, -1.460e+03f, 6.223e+01f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:            {3.284e+03f, -2.189e+03f, 1.334e+02f },
 470:            {3.284e+03f, -3.284e+03f, 2.600e+02f },
 471:            {3.284e+03f, -4.926e+03f, 5.066e+02f },
 472:            {3.284e+03f, -7.389e+03f, 9.872e+02f },
 473:            {3.284e+03f, -1.108e+04f, 1.749e+03f },
 474:            {3.284e+03f, -1.663e+04f, 3.098e+03f },
 475:            {4.926e+03f, -7.500e+00f, 1.250e+00f },
 476:            {4.926e+03f, -1.125e+01f, 1.250e+00f },
 477:            {4.926e+03f, -1.688e+01f, 1.250e+00f },
 478:            {4.926e+03f, -2.531e+01f, 1.250e+00f },
 479:            {4.926e+03f, -3.797e+01f, 1.250e+00f },
 480:            {4.926e+03f, -5.695e+01f, 1.250e+00f },
 481:            {4.926e+03f, -8.543e+01f, 1.250e+00f },
 482:            {4.926e+03f, -1.281e+02f, 1.250e+00f },
 483:            {4.926e+03f, -1.922e+02f, 1.250e+00f },
 484:            {4.926e+03f, -2.883e+02f, 2.013e+00f },
 485:            {4.926e+03f, -4.325e+02f, 4.315e+00f },
 486:            {4.926e+03f, -6.487e+02f, 9.250e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:            {4.926e+03f, -9.731e+02f, 2.181e+01f },
 488:            {4.926e+03f, -1.460e+03f, 4.250e+01f },
 489:            {4.926e+03f, -2.189e+03f, 9.111e+01f },
 490:            {4.926e+03f, -3.284e+03f, 1.953e+02f },
 491:            {4.926e+03f, -4.926e+03f, 3.806e+02f },
 492:            {4.926e+03f, -7.389e+03f, 7.417e+02f },
 493:            {4.926e+03f, -1.108e+04f, 1.445e+03f },
 494:            {4.926e+03f, -1.663e+04f, 2.561e+03f },
 495:            {7.389e+03f, -7.500e+00f, 1.250e+00f },
 496:            {7.389e+03f, -1.125e+01f, 1.250e+00f },
 497:            {7.389e+03f, -1.688e+01f, 1.250e+00f },
 498:            {7.389e+03f, -2.531e+01f, 1.250e+00f },
 499:            {7.389e+03f, -3.797e+01f, 1.250e+00f },
 500:            {7.389e+03f, -5.695e+01f, 1.250e+00f },
 501:            {7.389e+03f, -8.543e+01f, 1.250e+00f },
 502:            {7.389e+03f, -1.281e+02f, 1.250e+00f },
 503:            {7.389e+03f, -1.922e+02f, 1.250e+00f },
 504:            {7.389e+03f, -2.883e+02f, 1.375e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:            {7.389e+03f, -4.325e+02f, 2.947e+00f },
 506:            {7.389e+03f, -6.487e+02f, 6.318e+00f },
 507:            {7.389e+03f, -9.731e+02f, 1.490e+01f },
 508:            {7.389e+03f, -1.460e+03f, 3.193e+01f },
 509:            {7.389e+03f, -2.189e+03f, 6.845e+01f },
 510:            {7.389e+03f, -3.284e+03f, 1.334e+02f },
 511:            {7.389e+03f, -4.926e+03f, 2.860e+02f },
 512:            {7.389e+03f, -7.389e+03f, 5.572e+02f },
 513:            {7.389e+03f, -1.108e+04f, 1.086e+03f },
 514:            {7.389e+03f, -1.663e+04f, 2.116e+03f },
 515:            {1.108e+04f, -7.500e+00f, 1.250e+00f },
 516:            {1.108e+04f, -1.125e+01f, 1.250e+00f },
 517:            {1.108e+04f, -1.688e+01f, 1.250e+00f },
 518:            {1.108e+04f, -2.531e+01f, 1.250e+00f },
 519:            {1.108e+04f, -3.797e+01f, 1.250e+00f },
 520:            {1.108e+04f, -5.695e+01f, 1.250e+00f },
 521:            {1.108e+04f, -8.543e+01f, 1.250e+00f },
 522:            {1.108e+04f, -1.281e+02f, 1.250e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:            {1.108e+04f, -1.922e+02f, 1.250e+00f },
 524:            {1.108e+04f, -2.883e+02f, 1.250e+00f },
 525:            {1.108e+04f, -4.325e+02f, 2.013e+00f },
 526:            {1.108e+04f, -6.487e+02f, 4.315e+00f },
 527:            {1.108e+04f, -9.731e+02f, 1.018e+01f },
 528:            {1.108e+04f, -1.460e+03f, 2.181e+01f },
 529:            {1.108e+04f, -2.189e+03f, 4.676e+01f },
 530:            {1.108e+04f, -3.284e+03f, 1.002e+02f },
 531:            {1.108e+04f, -4.926e+03f, 2.148e+02f },
 532:            {1.108e+04f, -7.389e+03f, 4.187e+02f },
 533:            {1.108e+04f, -1.108e+04f, 8.974e+02f },
 534:            {1.108e+04f, -1.663e+04f, 1.749e+03f },
 535:            {1.663e+04f, -7.500e+00f, 1.250e+00f },
 536:            {1.663e+04f, -1.125e+01f, 1.250e+00f },
 537:            {1.663e+04f, -1.688e+01f, 1.250e+00f },
 538:            {1.663e+04f, -2.531e+01f, 1.250e+00f },
 539:            {1.663e+04f, -3.797e+01f, 1.250e+00f },
 540:            {1.663e+04f, -5.695e+01f, 1.250e+00f },
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:            {1.663e+04f, -8.543e+01f, 1.250e+00f },
 542:            {1.663e+04f, -1.281e+02f, 1.250e+00f },
 543:            {1.663e+04f, -1.922e+02f, 1.250e+00f },
 544:            {1.663e+04f, -2.883e+02f, 1.250e+00f },
 545:            {1.663e+04f, -4.325e+02f, 1.375e+00f },
 546:            {1.663e+04f, -6.487e+02f, 2.947e+00f },
 547:            {1.663e+04f, -9.731e+02f, 6.318e+00f },
 548:            {1.663e+04f, -1.460e+03f, 1.490e+01f },
 549:            {1.663e+04f, -2.189e+03f, 3.193e+01f },
 550:            {1.663e+04f, -3.284e+03f, 6.845e+01f },
 551:            {1.663e+04f, -4.926e+03f, 1.467e+02f },
 552:            {1.663e+04f, -7.389e+03f, 3.145e+02f },
 553:            {1.663e+04f, -1.108e+04f, 6.743e+02f },
 554:            {1.663e+04f, -1.663e+04f, 1.314e+03f },
 555:         };
 556:         if ((a > 1.663e+04) || (-b > 1.663e+04))
 557:            return z > -b;  // Way overly conservative?
 558:         if (a < data[0][0])
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 559-576 / 第 559-576 行
~~~cpp
 559:            return false;
 560:         int index = 0;
 561:         while (data[index][0] < a)
 562:            ++index;
 563:         if(a != data[index][0])
 564:            --index;
 565:         while ((data[index][1] < b) && (data[index][2] > 1.25))
 566:            --index;
 567:         ++index;
 568:         BOOST_MATH_ASSERT(a > data[index][0]);
 569:         BOOST_MATH_ASSERT(-b < -data[index][1]);
 570:         return z > data[index][2];
 571:      }
 572:      template <class T, class Policy>
 573:      T hypergeometric_1F1_from_function_ratio_negative_b_forwards(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling)
 574:      {
 575:         BOOST_MATH_STD_USING
 576:         //
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as BOOST_MATH_ASSERT.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。

### Lines 577-594 / 第 577-594 行
~~~cpp
 577:         // Get the function ratio, M(a+1, b+1, z)/M(a,b,z):
 578:         //
 579:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 580:         boost::math::detail::hypergeometric_1F1_recurrence_a_and_b_coefficients<T> coef(a, b, z);
 581:         T ratio = 1 / boost::math::tools::function_ratio_from_forwards_recurrence(coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
 582:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_from_function_ratio_negative_b_positive_a<%1%>(%1%,%1%,%1%)", max_iter, pol);
 583:         //
 584:         // We can't normalise via the Wronksian as the subtraction in the Wronksian will suffer an exquisite amount of cancellation -
 585:         // potentially many hundreds of digits in this region.  However, if forwards iteration is stable at this point
 586:         // it will also be stable for M(a, b+1, z) etc all the way up to the origin, and hopefully one step beyond.  So
 587:         // use a reference value just over the origin to normalise:
 588:         //
 589:         long long scale = 0;
 590:         int steps = itrunc(ceil(-b));
 591:         T reference_value = hypergeometric_1F1_imp(T(a + steps), T(b + steps), z, pol, log_scaling);
 592:         T found = boost::math::tools::apply_recurrence_relation_forward(boost::math::detail::hypergeometric_1F1_recurrence_a_and_b_coefficients<T>(a + 1, b + 1, z), steps - 1, T(1), ratio, &scale);
 593:         log_scaling -= scale;
 594:         if ((fabs(reference_value) < 1) && (fabs(reference_value) < tools::min_value<T>() * fabs(found)))
~~~
- **EN:** This range declares or defines callable logic such as coef, boost::math::tools::function_ratio_from_forwards_recurrence, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 coef, boost::math::tools::function_ratio_from_forwards_recurrence, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 595-612 / 第 595-612 行
~~~cpp
 595:         {
 596:            // Possible underflow, rescale
 597:            long long s = lltrunc(tools::log_max_value<T>());
 598:            log_scaling -= s;
 599:            reference_value *= exp(T(s));
 600:         }
 601:         else if ((fabs(found) < 1) && (fabs(reference_value) > tools::max_value<T>() * fabs(found)))
 602:         {
 603:            // Overflow, rescale:
 604:            long long s = lltrunc(tools::log_max_value<T>());
 605:            log_scaling += s;
 606:            reference_value /= exp(T(s));
 607:         }
 608:         return reference_value / found;
 609:      }
 610: 
 611: 
 612: 
~~~
- **EN:** This range declares or defines callable logic such as lltrunc, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lltrunc, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 613-630 / 第 613-630 行
~~~cpp
 613:      //
 614:      // This next version is largely the same as above, but calculates the ratio for the b recurrence relation
 615:      // which has a larger area of stability than the ab recurrence when a,b < 0.  We can then use a single
 616:      // recurrence step to convert this to the ratio for the ab recursion and proceed largely as before.
 617:      // The routine is quite insensitive to the size of z, but requires |a| < |5b| for accuracy.
 618:      // Fortunately the accuracy outside this domain falls off steadily rather than suddenly switching
 619:      // to a different behaviour.
 620:      //
 621:      template <class T, class Policy>
 622:      T hypergeometric_1F1_from_function_ratio_negative_ab(const T& a, const T& b, const T& z, const Policy& pol, long long& log_scaling)
 623:      {
 624:         BOOST_MATH_STD_USING
 625:         //
 626:         // Get the function ratio, M(a+1, b+1, z)/M(a,b,z):
 627:         //
 628:         std::uintmax_t max_iter = boost::math::policies::get_max_series_iterations<Policy>();
 629:         boost::math::detail::hypergeometric_1F1_recurrence_b_coefficients<T> coef(a, b + 1, z);
 630:         T ratio = boost::math::tools::function_ratio_from_backwards_recurrence(coef, boost::math::policies::get_epsilon<T, Policy>(), max_iter);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as coef, boost::math::tools::function_ratio_from_backwards_recurrence.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 coef, boost::math::tools::function_ratio_from_backwards_recurrence。

### Lines 631-648 / 第 631-648 行
~~~cpp
 631:         boost::math::policies::check_series_iterations<T>("boost::math::hypergeometric_1F1_from_function_ratio_negative_b_positive_a<%1%>(%1%,%1%,%1%)", max_iter, pol);
 632:         //
 633:         // We need to use A&S 13.4.3 to convert a ratio for M(a, b + 1, z) / M(a, b, z)
 634:         // to M(a+1, b+1, z) / M(a, b, z)
 635:         //
 636:         // We have:        (a-b)M(a, b+1, z) - aM(a+1, b+1, z) + bM(a, b, z) = 0
 637:         // and therefore:  M(a + 1, b + 1, z) / M(a, b, z) = ((a - b)M(a, b + 1, z) / M(a, b, z) + b) / a
 638:         //
 639:         ratio = ((a - b) * ratio + b) / a;
 640:         //
 641:         // Let M2 = M(1+a-b, 2-b, z)
 642:         // This is going to be a mighty big number:
 643:         //
 644:         long long local_scaling = 0;
 645:         T M2 = boost::math::detail::hypergeometric_1F1_imp(T(1 + a - b), T(2 - b), z, pol, local_scaling);
 646:         log_scaling -= local_scaling; // all the M2 terms are in the denominator
 647:         //
 648:         // Let M3 = M(1+a-b + 1, 2-b + 1, z)
~~~
- **EN:** This range declares or defines callable logic such as boost::math::detail::hypergeometric_1F1_imp. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::detail::hypergeometric_1F1_imp。 这里的许多行是数值常量或系数表，用来支撑周围的算法。

### Lines 649-666 / 第 649-666 行
~~~cpp
 649:         // We don't use the ratio to get this as it's not clear that it's reliable:
 650:         //
 651:         long long local_scaling2 = 0;
 652:         T M3 = boost::math::detail::hypergeometric_1F1_imp(T(2 + a - b), T(3 - b), z, pol, local_scaling2);
 653:         //
 654:         // M2 and M3 must be identically scaled:
 655:         //
 656:         if (local_scaling != local_scaling2)
 657:         {
 658:            M3 *= exp(T(local_scaling2 - local_scaling));
 659:         }
 660:         //
 661:         // Get the RHS of the Wronksian:
 662:         //
 663:         long long fz = lltrunc(z);
 664:         log_scaling += fz;
 665:         T rhs = (1 - b) * exp(z - fz);
 666:         //
~~~
- **EN:** This range declares or defines callable logic such as boost::math::detail::hypergeometric_1F1_imp, exp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::detail::hypergeometric_1F1_imp, exp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 667-678 / 第 667-678 行
~~~cpp
 667:         // We need to divide that by:
 668:         // [(a-b+1)z/(2-b) M2(a+1,b+1,z) + (1-b) M2(a,b,z) - a/b z^b R(a,b,z) M2(a,b,z) ]
 669:         // Note that at this stage, both M3 and M2 are scaled by exp(local_scaling).
 670:         //
 671:         T lhs = (a - b + 1) * z * M3 / (2 - b) + (1 - b) * M2 - a * z * ratio * M2 / b;
 672: 
 673:         return rhs / lhs;
 674:      }
 675: 
 676:   } } } // namespaces
 677: 
 678: #endif // BOOST_HYPERGEOMETRIC_1F1_BY_RATIOS_HPP_
~~~
- **EN:** This range mainly closes the preceding type, namespace, or conditional scope. Return statements hand the computed result or status back to the caller. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围主要用于结束前面的类型、命名空间或条件作用域。 return 语句会把计算结果或状态返回给调用方。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `boost/math/tools/recurrence.hpp, boost/math/policies/error_handling.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `hypergeometric_1F1_imp, sqrt, boost::math::detail::hypergeometric_1F1_imp, lltrunc, exp, coef2, coef, boost::math::tools::function_ratio_from_backwards_recurrence, ...`
