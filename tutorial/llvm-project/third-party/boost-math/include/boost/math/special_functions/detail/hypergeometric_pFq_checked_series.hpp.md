# hypergeometric_pFq_checked_series.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/detail/hypergeometric_pFq_checked_series.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This header supplies internal algorithms and tables for the hypergeometric pFq checked series special-function path.
- **作用（中文）**: 此头文件为 hypergeometric pFq checked series 特殊函数路径提供内部算法与查找表。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: ///////////////////////////////////////////////////////////////////////////////
   2: //  Copyright 2018 John Maddock
   3: //  Distributed under the Boost
   4: //  Software License, Version 1.0. (See accompanying file
   5: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6: 
   7: #ifndef BOOST_HYPERGEOMETRIC_PFQ_SERIES_HPP_
   8: #define BOOST_HYPERGEOMETRIC_PFQ_SERIES_HPP_
   9: 
  10: #ifndef BOOST_MATH_PFQ_MAX_B_TERMS
  11: #  define BOOST_MATH_PFQ_MAX_B_TERMS 5
  12: #endif
  13: 
  14: #include <array>
  15: #include <cstdint>
  16: #include <boost/math/special_functions/gamma.hpp>
  17: #include <boost/math/special_functions/expm1.hpp>
  18: #include <boost/math/special_functions/detail/hypergeometric_series.hpp>
~~~
- **EN:** This block imports dependencies such as array, cstdint, boost/math/special_functions/gamma.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 array, cstdint, boost/math/special_functions/gamma.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: 
  20:   namespace boost { namespace math { namespace detail {
  21: 
  22:      template <class Seq, class Real>
  23:      unsigned set_crossover_locations(const Seq& aj, const Seq& bj, const Real& z, unsigned int* crossover_locations)
  24:      {
  25:         BOOST_MATH_STD_USING
  26:         unsigned N_terms = 0;
  27: 
  28:         if(aj.size() == 1 && bj.size() == 1)
  29:         {
  30:            //
  31:            // For 1F1 we can work out where the peaks in the series occur,
  32:            //  which is to say when:
  33:            //
  34:            // (a + k)z / (k(b + k)) == +-1
  35:            //
  36:            // Then we are at either a maxima or a minima in the series, and the
~~~
- **EN:** The code enters namespace scope (boost::math::detail) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `Seq` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `Seq`，作为该文件核心抽象的一部分。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:            // last such point must be a maxima since the series is globally convergent.
  38:            // Potentially then we are solving 2 quadratic equations and have up to 4
  39:            // solutions, any solutions which are complex or negative are discarded,
  40:            // leaving us with 4 options:
  41:            //
  42:            // 0 solutions: The series is directly convergent.
  43:            // 1 solution : The series diverges to a maxima before converging.
  44:            // 2 solutions: The series is initially convergent, followed by divergence to a maxima before final convergence.
  45:            // 3 solutions: The series diverges to a maxima, converges to a minima before diverging again to a second maxima before final convergence.
  46:            // 4 solutions: The series converges to a minima before diverging to a maxima, converging to a minima, diverging to a second maxima and then converging.
  47:            //
  48:            // The first 2 situations are adequately handled by direct series evaluation, while the 2,3 and 4 solutions are not.
  49:            //
  50:            Real a = *aj.begin();
  51:            Real b = *bj.begin();
  52:            Real sq = 4 * a * z + b * b - 2 * b * z + z * z;
  53:            if (sq >= 0)
  54:            {
~~~
- **EN:** This range declares or defines callable logic such as begin. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 begin。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:               Real t = (-sqrt(sq) - b + z) / 2;
  56:               if (t >= 0)
  57:               {
  58:                  crossover_locations[N_terms] = itrunc(t);
  59:                  ++N_terms;
  60:               }
  61:               t = (sqrt(sq) - b + z) / 2;
  62:               if (t >= 0)
  63:               {
  64:                  crossover_locations[N_terms] = itrunc(t);
  65:                  ++N_terms;
  66:               }
  67:            }
  68:            sq = -4 * a * z + b * b + 2 * b * z + z * z;
  69:            if (sq >= 0)
  70:            {
  71:               Real t = (-sqrt(sq) - b - z) / 2;
  72:               if (t >= 0)
~~~
- **EN:** This range declares or defines callable logic such as itrunc. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:               {
  74:                  crossover_locations[N_terms] = itrunc(t);
  75:                  ++N_terms;
  76:               }
  77:               t = (sqrt(sq) - b - z) / 2;
  78:               if (t >= 0)
  79:               {
  80:                  crossover_locations[N_terms] = itrunc(t);
  81:                  ++N_terms;
  82:               }
  83:            }
  84:            std::sort(crossover_locations, crossover_locations + N_terms, std::less<Real>());
  85:            //
  86:            // Now we need to discard every other terms, as these are the minima:
  87:            //
  88:            switch (N_terms)
  89:            {
  90:            case 0:
~~~
- **EN:** This range declares or defines callable logic such as itrunc, std::sort. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc, std::sort。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:            case 1:
  92:               break;
  93:            case 2:
  94:               crossover_locations[0] = crossover_locations[1];
  95:               --N_terms;
  96:               break;
  97:            case 3:
  98:               crossover_locations[1] = crossover_locations[2];
  99:               --N_terms;
 100:               break;
 101:            case 4:
 102:               crossover_locations[0] = crossover_locations[1];
 103:               crossover_locations[1] = crossover_locations[3];
 104:               N_terms -= 2;
 105:               break;
 106:            }
 107:         }
 108:         else
~~~
- **EN:** This range continues the current implementation with a mix of declarations, structure, and low-level logic.
- **CN:** 此范围继续当前实现，混合了声明、结构与底层逻辑。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:         {
 110:            unsigned n = 0;
 111:            for (auto bi = bj.begin(); bi != bj.end(); ++bi, ++n)
 112:            {
 113:               crossover_locations[n] = *bi >= 0 ? 0 : itrunc(-*bi) + 1;
 114:            }
 115:            std::sort(crossover_locations, crossover_locations + bj.size(), std::less<Real>());
 116:            N_terms = (unsigned)bj.size();
 117:         }
 118:         return N_terms;
 119:      }
 120: 
 121:      template <class Seq, class Real, class Policy, class Terminal>
 122:      std::pair<Real, Real> hypergeometric_pFq_checked_series_impl(const Seq& aj, const Seq& bj, const Real& z, const Policy& pol, const Terminal& termination, long long& log_scale)
 123:      {
 124:         BOOST_MATH_STD_USING
 125:         Real result = 1;
 126:         Real abs_result = 1;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `Seq` as part of the file's main abstraction. This range declares or defines callable logic such as std::sort, size.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `Seq`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 std::sort, size。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:         Real term = 1;
 128:         Real term0 = 0;
 129:         Real tol = boost::math::policies::get_epsilon<Real, Policy>();
 130:         std::uintmax_t k = 0;
 131:         Real upper_limit(sqrt(boost::math::tools::max_value<Real>())), diff;
 132:         Real lower_limit(1 / upper_limit);
 133:         long long log_scaling_factor = lltrunc(boost::math::tools::log_max_value<Real>()) - 2;
 134:         Real scaling_factor = exp(Real(log_scaling_factor));
 135:         Real term_m1;
 136:         long long local_scaling = 0;
 137:         bool have_no_correct_bits = false;
 138: 
 139:         if ((aj.size() == 1) && (bj.size() == 0))
 140:         {
 141:            if (fabs(z) > 1)
 142:            {
 143:               if ((z > 0) && (floor(*aj.begin()) != *aj.begin()))
 144:               {
~~~
- **EN:** This range declares or defines callable logic such as lower_limit, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 lower_limit, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:                  Real r = policies::raise_domain_error("boost::math::hypergeometric_pFq", "Got p == 1 and q == 0 and |z| > 1, result is imaginary", z, pol);
 146:                  return std::make_pair(r, r);
 147:               }
 148:               std::pair<Real, Real> r = hypergeometric_pFq_checked_series_impl(aj, bj, Real(1 / z), pol, termination, log_scale);
 149: 
 150:               #if (defined(__GNUC__) && __GNUC__ == 13)
 151:               Real mul = pow(-z, Real(-*aj.begin()));
 152:               #else
 153:               Real mul = pow(-z, -*aj.begin());
 154:               #endif
 155: 
 156:               r.first *= mul;
 157:               r.second *= mul;
 158:               return r;
 159:            }
 160:         }
 161: 
 162:         if (aj.size() > bj.size())
~~~
- **EN:** This range declares or defines callable logic such as policies::raise_domain_error, hypergeometric_pFq_checked_series_impl, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 policies::raise_domain_error, hypergeometric_pFq_checked_series_impl, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:         {
 164:            if (aj.size() == bj.size() + 1)
 165:            {
 166:               if (fabs(z) > 1)
 167:               {
 168:                  Real r = policies::raise_domain_error("boost::math::hypergeometric_pFq", "Got p == q+1 and |z| > 1, series does not converge", z, pol);
 169:                  return std::make_pair(r, r);
 170:               }
 171:               if (fabs(z) == 1)
 172:               {
 173:                  Real s = 0;
 174:                  for (auto i = bj.begin(); i != bj.end(); ++i)
 175:                     s += *i;
 176:                  for (auto i = aj.begin(); i != aj.end(); ++i)
 177:                     s -= *i;
 178:                  if ((z == 1) && (s <= 0))
 179:                  {
 180:                     Real r = policies::raise_domain_error("boost::math::hypergeometric_pFq", "Got p == q+1 and |z| == 1, in a situation where the series does not converge", z, pol);
~~~
- **EN:** This range declares or defines callable logic such as policies::raise_domain_error. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 policies::raise_domain_error。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:                     return std::make_pair(r, r);
 182:                  }
 183:                  if ((z == -1) && (s <= -1))
 184:                  {
 185:                     Real r = policies::raise_domain_error("boost::math::hypergeometric_pFq", "Got p == q+1 and |z| == 1, in a situation where the series does not converge", z, pol);
 186:                     return std::make_pair(r, r);
 187:                  }
 188:               }
 189:            }
 190:            else
 191:            {
 192:               Real r = policies::raise_domain_error("boost::math::hypergeometric_pFq", "Got p > q+1, series does not converge", z, pol);
 193:               return std::make_pair(r, r);
 194:            }
 195:         }
 196: 
 197:         while (!termination(k))
 198:         {
~~~
- **EN:** This range declares or defines callable logic such as policies::raise_domain_error. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 policies::raise_domain_error。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:            for (auto ai = aj.begin(); ai != aj.end(); ++ai)
 200:            {
 201:               term *= *ai + k;
 202:            }
 203:            if (term == 0)
 204:            {
 205:               // There is a negative integer in the aj's:
 206:               return std::make_pair(result, abs_result);
 207:            }
 208:            for (auto bi = bj.begin(); bi != bj.end(); ++bi)
 209:            {
 210:               if (*bi + k == 0)
 211:               {
 212:                  // The series is undefined:
 213:                  result = boost::math::policies::raise_domain_error("boost::math::hypergeometric_pFq<%1%>", "One of the b values was the negative integer %1%", *bi, pol);
 214:                  return std::make_pair(result, result);
 215:               }
 216:               term /= *bi + k;
~~~
- **EN:** This range declares or defines callable logic such as boost::math::policies::raise_domain_error. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::policies::raise_domain_error。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:            }
 218:            term *= z;
 219:            ++k;
 220:            term /= k;
 221:            //std::cout << k << " " << *bj.begin() + k << " " << result << " " << term << /*" " << term_at_k(*aj.begin(), *bj.begin(), z, k, pol) <<*/ std::endl;
 222:            result += term;
 223:            abs_result += abs(term);
 224:            //std::cout << "k = " << k << " term = " << term * exp(log_scale) << " result = " << result * exp(log_scale) << " abs_result = " << abs_result * exp(log_scale) << std::endl;
 225: 
 226:            //
 227:            // Rescaling:
 228:            //
 229:            if (fabs(abs_result) >= upper_limit)
 230:            {
 231:               abs_result /= scaling_factor;
 232:               result /= scaling_factor;
 233:               term /= scaling_factor;
 234:               log_scale += log_scaling_factor;
~~~
- **EN:** This range declares or defines callable logic such as abs. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 abs。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:               local_scaling += log_scaling_factor;
 236:            }
 237:            if (fabs(abs_result) < lower_limit)
 238:            {
 239:               abs_result *= scaling_factor;
 240:               result *= scaling_factor;
 241:               term *= scaling_factor;
 242:               log_scale -= log_scaling_factor;
 243:               local_scaling -= log_scaling_factor;
 244:            }
 245: 
 246:            if ((abs(result * tol) > abs(term)) && (abs(term0) > abs(term)))
 247:               break;
 248:            if (abs_result * tol > abs(result))
 249:            {
 250:               // Check if result is so small compared to abs_resuslt that there are no longer any
 251:               // correct bits... we require two consecutive passes here before aborting to
 252:               // avoid false positives when result transiently drops to near zero then rebounds.
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:               if (have_no_correct_bits)
 254:               {
 255:                  // We have no correct bits in the result... just give up!
 256:                  result = boost::math::policies::raise_evaluation_error("boost::math::hypergeometric_pFq<%1%>", "Cancellation is so severe that no bits in the result are correct, last result was %1%", Real(result * exp(Real(log_scale))), pol);
 257:                  return std::make_pair(result, result);
 258:               }
 259:               else
 260:                  have_no_correct_bits = true;
 261:            }
 262:            else
 263:               have_no_correct_bits = false;
 264:            term0 = term;
 265:         }
 266:         //std::cout << "result = " << result << std::endl;
 267:         //std::cout << "local_scaling = " << local_scaling << std::endl;
 268:         //std::cout << "Norm result = " << std::setprecision(35) << boost::multiprecision::mpfr_float_50(result) * exp(boost::multiprecision::mpfr_float_50(local_scaling)) << std::endl;
 269:         //
 270:         // We have to be careful when one of the b's crosses the origin:
~~~
- **EN:** This range declares or defines callable logic such as boost::math::policies::raise_evaluation_error. Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::policies::raise_evaluation_error。 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:         //
 272:         if(bj.size() > BOOST_MATH_PFQ_MAX_B_TERMS)
 273:            policies::raise_domain_error<Real>("boost::math::hypergeometric_pFq<%1%>(Seq, Seq, %1%)",
 274:               "The number of b terms must be less than the value of BOOST_MATH_PFQ_MAX_B_TERMS (" BOOST_MATH_STRINGIZE(BOOST_MATH_PFQ_MAX_B_TERMS)  "), but got %1%.",
 275:               Real(bj.size()), pol);
 276: 
 277:         unsigned crossover_locations[BOOST_MATH_PFQ_MAX_B_TERMS];
 278: 
 279:         unsigned N_crossovers = set_crossover_locations(aj, bj, z, crossover_locations);
 280: 
 281:         bool terminate = false;   // Set to true if one of the a's passes through the origin and terminates the series.
 282: 
 283:         for (unsigned n = 0; n < N_crossovers; ++n)
 284:         {
 285:            if (k < crossover_locations[n])
 286:            {
 287:               for (auto ai = aj.begin(); ai != aj.end(); ++ai)
 288:               {
~~~
- **EN:** This range declares or defines callable logic such as Real, set_crossover_locations. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 Real, set_crossover_locations。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:                  if ((*ai < 0) && (floor(*ai) == *ai) && (*ai > static_cast<decltype(*ai)>(crossover_locations[n])))
 290:                     return std::make_pair(result, abs_result);  // b's will never cross the origin!
 291:               }
 292:               //
 293:               // local results:
 294:               //
 295:               Real loop_result = 0;
 296:               Real loop_abs_result = 0;
 297:               long long loop_scale = 0;
 298:               //
 299:               // loop_error_scale will be used to increase the size of the error
 300:               // estimate (absolute sum), based on the errors inherent in calculating
 301:               // the pochhammer symbols.
 302:               //
 303:               Real loop_error_scale = 0;
 304:               //boost::multiprecision::mpfi_float err_est = 0;
 305:               //
 306:               // b hasn't crossed the origin yet and the series may spring back into life at that point
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 return 语句会把计算结果或状态返回给调用方。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:               // so we need to jump forward to that term and then evaluate forwards and backwards from there:
 308:               //
 309:               unsigned s = crossover_locations[n];
 310:               std::uintmax_t backstop = k;
 311:               long long s1(1), s2(1);
 312:               term = 0;
 313:               for (auto ai = aj.begin(); ai != aj.end(); ++ai)
 314:               {
 315:                  if ((floor(*ai) == *ai) && (*ai < 0) && (-*ai <= static_cast<decltype(*ai)>(s)))
 316:                  {
 317:                     // One of the a terms has passed through zero and terminated the series:
 318:                     terminate = true;
 319:                     break;
 320:                  }
 321:                  else
 322:                  {
 323:                     int ls = 1;
 324:                     Real p = log_pochhammer(*ai, s, pol, &ls);
~~~
- **EN:** This range declares or defines callable logic such as s1, log_pochhammer. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 s1, log_pochhammer。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:                     s1 *= ls;
 326:                     term += p;
 327:                     loop_error_scale = (std::max)(p, loop_error_scale);
 328:                     //err_est += boost::multiprecision::mpfi_float(p);
 329:                  }
 330:               }
 331:               //std::cout << "term = " << term << std::endl;
 332:               if (terminate)
 333:                  break;
 334:               for (auto bi = bj.begin(); bi != bj.end(); ++bi)
 335:               {
 336:                  int ls = 1;
 337:                  Real p = log_pochhammer(*bi, s, pol, &ls);
 338:                  s2 *= ls;
 339:                  term -= p;
 340:                  loop_error_scale = (std::max)(p, loop_error_scale);
 341:                  //err_est -= boost::multiprecision::mpfi_float(p);
 342:               }
~~~
- **EN:** This range declares or defines callable logic such as boost::multiprecision::mpfi_float, log_pochhammer. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::multiprecision::mpfi_float, log_pochhammer。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:               //std::cout << "term = " << term << std::endl;
 344:               Real p = lgamma(Real(s + 1), pol);
 345:               term -= p;
 346:               loop_error_scale = (std::max)(p, loop_error_scale);
 347:               //err_est -= boost::multiprecision::mpfi_float(p);
 348:               p = s * log(fabs(z));
 349:               term += p;
 350:               loop_error_scale = (std::max)(p, loop_error_scale);
 351:               //err_est += boost::multiprecision::mpfi_float(p);
 352:               //err_est = exp(err_est);
 353:               //std::cout << err_est << std::endl;
 354:               //
 355:               // Convert loop_error scale to the absolute error
 356:               // in term after exp is applied:
 357:               //
 358:               loop_error_scale *= tools::epsilon<Real>();
 359:               //
 360:               // Convert to relative error after exp:
~~~
- **EN:** This range declares or defines callable logic such as lgamma, boost::multiprecision::mpfi_float, ....
- **CN:** 此范围声明或定义了可调用逻辑，例如 lgamma, boost::multiprecision::mpfi_float, ...。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:               //
 362:               loop_error_scale = fabs(expm1(loop_error_scale, pol));
 363:               //
 364:               // Convert to multiplier for the error term:
 365:               //
 366:               loop_error_scale /= tools::epsilon<Real>();
 367: 
 368:               if (z < 0)
 369:                  s1 *= (s & 1 ? -1 : 1);
 370: 
 371:               if (term <= tools::log_min_value<Real>())
 372:               {
 373:                  // rescale if we can:
 374:                  long long scale = lltrunc(floor(term - tools::log_min_value<Real>()) - 2);
 375:                  term -= scale;
 376:                  loop_scale += scale;
 377:               }
 378:                if (term > 10)
~~~
- **EN:** This range declares or defines callable logic such as fabs, lltrunc. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs, lltrunc。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:                {
 380:                   int scale = itrunc(floor(term));
 381:                   term -= scale;
 382:                   loop_scale += scale;
 383:                }
 384:                //std::cout << "term = " << term << std::endl;
 385:                term = s1 * s2 * exp(term);
 386:                //std::cout << "term = " << term << std::endl;
 387:                //std::cout << "loop_scale = " << loop_scale << std::endl;
 388:                k = s;
 389:                term0 = term;
 390:                long long saved_loop_scale = loop_scale;
 391:                bool terms_are_growing = true;
 392:                bool trivial_small_series_check = false;
 393:                do
 394:                {
 395:                   loop_result += term;
 396:                   loop_abs_result += fabs(term);
~~~
- **EN:** This range declares or defines callable logic such as itrunc, exp, ....
- **CN:** 此范围声明或定义了可调用逻辑，例如 itrunc, exp, ...。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:                   //std::cout << "k = " << k << " term = " << term * exp(loop_scale) << " result = " << loop_result * exp(loop_scale) << " abs_result = " << loop_abs_result * exp(loop_scale) << std::endl;
 398:                   if (fabs(loop_result) >= upper_limit)
 399:                   {
 400:                      loop_result /= scaling_factor;
 401:                      loop_abs_result /= scaling_factor;
 402:                      term /= scaling_factor;
 403:                      loop_scale += log_scaling_factor;
 404:                   }
 405:                   if (fabs(loop_result) < lower_limit)
 406:                   {
 407:                      loop_result *= scaling_factor;
 408:                      loop_abs_result *= scaling_factor;
 409:                      term *= scaling_factor;
 410:                      loop_scale -= log_scaling_factor;
 411:                   }
 412:                   term_m1 = term;
 413:                   for (auto ai = aj.begin(); ai != aj.end(); ++ai)
 414:                   {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:                      term *= *ai + k;
 416:                   }
 417:                   if (term == 0)
 418:                   {
 419:                      // There is a negative integer in the aj's:
 420:                      return std::make_pair(result, abs_result);
 421:                   }
 422:                   for (auto bi = bj.begin(); bi != bj.end(); ++bi)
 423:                   {
 424:                      if (*bi + k == 0)
 425:                      {
 426:                         // The series is undefined:
 427:                         result = boost::math::policies::raise_domain_error("boost::math::hypergeometric_pFq<%1%>", "One of the b values was the negative integer %1%", *bi, pol);
 428:                         return std::make_pair(result, result);
 429:                      }
 430:                      term /= *bi + k;
 431:                   }
 432:                   term *= z / (k + 1);
~~~
- **EN:** This range declares or defines callable logic such as boost::math::policies::raise_domain_error. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::policies::raise_domain_error。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433: 
 434:                   ++k;
 435:                   diff = fabs(term / loop_result);
 436:                   terms_are_growing = fabs(term) > fabs(term_m1);
 437:                   if (!trivial_small_series_check && !terms_are_growing)
 438:                   {
 439:                      //
 440:                      // Now that we have started to converge, check to see if the value of
 441:                      // this local sum is trivially small compared to the result.  If so
 442:                      // abort this part of the series.
 443:                      //
 444:                      trivial_small_series_check = true;
 445:                      Real d;
 446:                      if (loop_scale > local_scaling)
 447:                      {
 448:                         long long rescale = local_scaling - loop_scale;
 449:                         if (rescale < tools::log_min_value<Real>())
 450:                            d = 1;  // arbitrary value, we want to keep going
~~~
- **EN:** This range declares or defines callable logic such as fabs. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 451-468 / 第 451-468 行
~~~cpp
 451:                         else
 452:                            d = fabs(term / (result * exp(Real(rescale))));
 453:                      }
 454:                      else
 455:                      {
 456:                         long long rescale = loop_scale - local_scaling;
 457:                         if (rescale < tools::log_min_value<Real>())
 458:                            d = 0;  // terminate this loop
 459:                         else
 460:                            d = fabs(term * exp(Real(rescale)) / result);
 461:                      }
 462:                      if (d < boost::math::policies::get_epsilon<Real, Policy>())
 463:                         break;
 464:                   }
 465:                } while (!termination(k - s) && ((diff > boost::math::policies::get_epsilon<Real, Policy>()) || terms_are_growing));
 466: 
 467:                //std::cout << "Norm loop result = " << std::setprecision(35) << boost::multiprecision::mpfr_float_50(loop_result)* exp(boost::multiprecision::mpfr_float_50(loop_scale)) << std::endl;
 468:                //
~~~
- **EN:** This range declares or defines callable logic such as fabs, while. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs, while。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 469-486 / 第 469-486 行
~~~cpp
 469:                // We now need to combine the results of the first series summation with whatever
 470:                // local results we have now.  First though, rescale abs_result by loop_error_scale
 471:                // to factor in the error in the pochhammer terms at the start of this block:
 472:                //
 473:                std::uintmax_t next_backstop = k;
 474:                loop_abs_result += loop_error_scale * fabs(loop_result);
 475:                if (loop_scale > local_scaling)
 476:                {
 477:                   //
 478:                   // Need to shrink previous result:
 479:                   //
 480:                   long long rescale = local_scaling - loop_scale;
 481:                   local_scaling = loop_scale;
 482:                   log_scale -= rescale;
 483:                   Real ex = exp(Real(rescale));
 484:                   result *= ex;
 485:                   abs_result *= ex;
 486:                   result += loop_result;
~~~
- **EN:** This range declares or defines callable logic such as fabs, exp. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs, exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 487-504 / 第 487-504 行
~~~cpp
 487:                   abs_result += loop_abs_result;
 488:                }
 489:                else if (local_scaling > loop_scale)
 490:                {
 491:                   //
 492:                   // Need to shrink local result:
 493:                   //
 494:                   long long rescale = loop_scale - local_scaling;
 495:                   Real ex = exp(Real(rescale));
 496:                   loop_result *= ex;
 497:                   loop_abs_result *= ex;
 498:                   result += loop_result;
 499:                   abs_result += loop_abs_result;
 500:                }
 501:                else
 502:                {
 503:                   result += loop_result;
 504:                   abs_result += loop_abs_result;
~~~
- **EN:** This range declares or defines callable logic such as exp. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 505-522 / 第 505-522 行
~~~cpp
 505:                }
 506:                //
 507:                // Now go backwards as well:
 508:                //
 509:                k = s;
 510:                term = term0;
 511:                loop_result = 0;
 512:                loop_abs_result = 0;
 513:                loop_scale = saved_loop_scale;
 514:                trivial_small_series_check = false;
 515:                do
 516:                {
 517:                   --k;
 518:                   if (k == backstop)
 519:                      break;
 520:                   term_m1 = term;
 521:                   for (auto ai = aj.begin(); ai != aj.end(); ++ai)
 522:                   {
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 523-540 / 第 523-540 行
~~~cpp
 523:                      term /= *ai + k;
 524:                   }
 525:                   for (auto bi = bj.begin(); bi != bj.end(); ++bi)
 526:                   {
 527:                      if (*bi + k == 0)
 528:                      {
 529:                         // The series is undefined:
 530:                         result = boost::math::policies::raise_domain_error("boost::math::hypergeometric_pFq<%1%>", "One of the b values was the negative integer %1%", *bi, pol);
 531:                         return std::make_pair(result, result);
 532:                      }
 533:                      term *= *bi + k;
 534:                   }
 535:                   term *= (k + 1) / z;
 536:                   loop_result += term;
 537:                   loop_abs_result += fabs(term);
 538: 
 539:                   if (!trivial_small_series_check && (fabs(term) < fabs(term_m1)))
 540:                   {
~~~
- **EN:** This range declares or defines callable logic such as boost::math::policies::raise_domain_error, fabs. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 boost::math::policies::raise_domain_error, fabs。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 541-558 / 第 541-558 行
~~~cpp
 541:                      //
 542:                      // Now that we have started to converge, check to see if the value of
 543:                      // this local sum is trivially small compared to the result.  If so
 544:                      // abort this part of the series.
 545:                      //
 546:                      trivial_small_series_check = true;
 547:                      Real d;
 548:                      if (loop_scale > local_scaling)
 549:                      {
 550:                         long long rescale = local_scaling - loop_scale;
 551:                         if (rescale < tools::log_min_value<Real>())
 552:                            d = 1;  // keep going
 553:                         else
 554:                            d = fabs(term / (result * exp(Real(rescale))));
 555:                      }
 556:                      else
 557:                      {
 558:                         long long rescale = loop_scale - local_scaling;
~~~
- **EN:** This range declares or defines callable logic such as fabs. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 559-576 / 第 559-576 行
~~~cpp
 559:                         if (rescale < tools::log_min_value<Real>())
 560:                            d = 0;  // stop, underflow
 561:                         else
 562:                            d = fabs(term * exp(Real(rescale)) / result);
 563:                      }
 564:                      if (d < boost::math::policies::get_epsilon<Real, Policy>())
 565:                         break;
 566:                   }
 567: 
 568:                   //std::cout << "k = " << k << " result = " << result << " abs_result = " << abs_result << std::endl;
 569:                   if (fabs(loop_result) >= upper_limit)
 570:                   {
 571:                      loop_result /= scaling_factor;
 572:                      loop_abs_result /= scaling_factor;
 573:                      term /= scaling_factor;
 574:                      loop_scale += log_scaling_factor;
 575:                   }
 576:                   if (fabs(loop_result) < lower_limit)
~~~
- **EN:** This range declares or defines callable logic such as fabs. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 577-594 / 第 577-594 行
~~~cpp
 577:                   {
 578:                      loop_result *= scaling_factor;
 579:                      loop_abs_result *= scaling_factor;
 580:                      term *= scaling_factor;
 581:                      loop_scale -= log_scaling_factor;
 582:                   }
 583:                   diff = fabs(term / loop_result);
 584:                } while (!termination(s - k) && ((diff > boost::math::policies::get_epsilon<Real, Policy>()) || (fabs(term) > fabs(term_m1))));
 585: 
 586:                //std::cout << "Norm loop result = " << std::setprecision(35) << boost::multiprecision::mpfr_float_50(loop_result)* exp(boost::multiprecision::mpfr_float_50(loop_scale)) << std::endl;
 587:                //
 588:                // We now need to combine the results of the first series summation with whatever
 589:                // local results we have now.  First though, rescale abs_result by loop_error_scale
 590:                // to factor in the error in the pochhammer terms at the start of this block:
 591:                //
 592:                loop_abs_result += loop_error_scale * fabs(loop_result);
 593:                //
 594:                if (loop_scale > local_scaling)
~~~
- **EN:** This range declares or defines callable logic such as fabs, while. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 fabs, while。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 595-612 / 第 595-612 行
~~~cpp
 595:                {
 596:                   //
 597:                   // Need to shrink previous result:
 598:                   //
 599:                   long long rescale = local_scaling - loop_scale;
 600:                   local_scaling = loop_scale;
 601:                   log_scale -= rescale;
 602:                   Real ex = exp(Real(rescale));
 603:                   result *= ex;
 604:                   abs_result *= ex;
 605:                   result += loop_result;
 606:                   abs_result += loop_abs_result;
 607:                }
 608:                else if (local_scaling > loop_scale)
 609:                {
 610:                   //
 611:                   // Need to shrink local result:
 612:                   //
~~~
- **EN:** This range declares or defines callable logic such as exp. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 613-630 / 第 613-630 行
~~~cpp
 613:                   long long rescale = loop_scale - local_scaling;
 614:                   Real ex = exp(Real(rescale));
 615:                   loop_result *= ex;
 616:                   loop_abs_result *= ex;
 617:                   result += loop_result;
 618:                   abs_result += loop_abs_result;
 619:                }
 620:                else
 621:                {
 622:                   result += loop_result;
 623:                   abs_result += loop_abs_result;
 624:                }
 625:                //
 626:                // Reset k to the largest k we reached
 627:                //
 628:                k = next_backstop;
 629:            }
 630:         }
~~~
- **EN:** This range declares or defines callable logic such as exp.
- **CN:** 此范围声明或定义了可调用逻辑，例如 exp。

### Lines 631-648 / 第 631-648 行
~~~cpp
 631: 
 632:         return std::make_pair(result, abs_result);
 633:      }
 634: 
 635:      struct iteration_terminator
 636:      {
 637:         iteration_terminator(std::uintmax_t i) : m(i) {}
 638: 
 639:         bool operator()(std::uintmax_t v) const { return v >= m; }
 640: 
 641:         std::uintmax_t m;
 642:      };
 643: 
 644:      template <class Seq, class Real, class Policy>
 645:      Real hypergeometric_pFq_checked_series_impl(const Seq& aj, const Seq& bj, const Real& z, const Policy& pol, long long& log_scale)
 646:      {
 647:         BOOST_MATH_STD_USING
 648:         iteration_terminator term(boost::math::policies::get_max_series_iterations<Policy>());
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `iteration_terminator` as part of the file's main abstraction. This range declares or defines callable logic such as iteration_terminator, operator, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `iteration_terminator`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 iteration_terminator, operator, ...。

### Lines 649-666 / 第 649-666 行
~~~cpp
 649:         std::pair<Real, Real> result = hypergeometric_pFq_checked_series_impl(aj, bj, z, pol, term, log_scale);
 650:         //
 651:         // Check to see how many digits we've lost, if it's more than half, raise an evaluation error -
 652:         // this is an entirely arbitrary cut off, but not unreasonable.
 653:         //
 654:         if (result.second * sqrt(boost::math::policies::get_epsilon<Real, Policy>()) > abs(result.first))
 655:         {
 656:            return boost::math::policies::raise_evaluation_error("boost::math::hypergeometric_pFq<%1%>", "Cancellation is so severe that fewer than half the bits in the result are correct, last result was %1%", Real(result.first * exp(Real(log_scale))), pol);
 657:         }
 658:         return result.first;
 659:      }
 660: 
 661:      template <class Real, class Policy>
 662:      inline Real hypergeometric_1F1_checked_series_impl(const Real& a, const Real& b, const Real& z, const Policy& pol, long long& log_scale)
 663:      {
 664:         std::array<Real, 1> aj = { a };
 665:         std::array<Real, 1> bj = { b };
 666:         return hypergeometric_pFq_checked_series_impl(aj, bj, z, pol, log_scale);
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `Real` as part of the file's main abstraction. This range declares or defines callable logic such as hypergeometric_pFq_checked_series_impl.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `Real`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 hypergeometric_pFq_checked_series_impl。

### Lines 667-671 / 第 667-671 行
~~~cpp
 667:      }
 668: 
 669:   } } } // namespaces
 670: 
 671: #endif // BOOST_HYPERGEOMETRIC_PFQ_SERIES_HPP_
~~~
- **EN:** This short range closes a conditional-compilation or header-guard region. This range mainly closes the preceding type, namespace, or conditional scope. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这一小段用于结束条件编译区域或头文件保护区域。 此范围主要用于结束前面的类型、命名空间或条件作用域。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

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
- **Included headers / 包含头文件**: `array, cstdint, boost/math/special_functions/gamma.hpp, boost/math/special_functions/expm1.hpp, boost/math/special_functions/detail/hypergeometric_series.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `begin, itrunc, std::sort, size, lower_limit, exp, policies::raise_domain_error, hypergeometric_pFq_checked_series_impl, ...`
