# jso.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/optimization/jso.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This Boost.Math header provides reusable support for optimization jso.
- **作用（中文）**: 此 Boost.Math 头文件为 optimization jso 提供可复用支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: /*
   2:  * Copyright Nick Thompson, 2024
   3:  * Use, modification and distribution are subject to the
   4:  * Boost Software License, Version 1.0. (See accompanying file
   5:  * LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6:  */
   7: #ifndef BOOST_MATH_OPTIMIZATION_JSO_HPP
   8: #define BOOST_MATH_OPTIMIZATION_JSO_HPP
   9: #include <atomic>
  10: #include <boost/math/optimization/detail/common.hpp>
  11: #include <cmath>
  12: #include <iostream>
  13: #include <limits>
  14: #include <random>
  15: #include <sstream>
  16: #include <stdexcept>
  17: #include <thread>
  18: #include <type_traits>
~~~
- **EN:** This block imports dependencies such as atomic, boost/math/optimization/detail/common.hpp, cmath, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 atomic, boost/math/optimization/detail/common.hpp, cmath, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <utility>
  20: #include <vector>
  21: 
  22: namespace boost::math::optimization {
  23: 
  24: #ifndef BOOST_MATH_DEBUG_JSO
  25: #define BOOST_MATH_DEBUG_JSO 0
  26: #endif
  27: // Follows: Brest, Janez, Mirjam Sepesy Maucec, and Borko Boskovic. "Single
  28: // objective real-parameter optimization: Algorithm jSO." 2017 IEEE congress on
  29: // evolutionary computation (CEC). IEEE, 2017. In the sequel, this wil be
  30: // referred to as "the reference". Note that the reference is rather difficult
  31: // to understand without also reading: Zhang, J., & Sanderson, A. C. (2009).
  32: // JADE: adaptive differential evolution with optional external archive.
  33: // IEEE Transactions on evolutionary computation, 13(5), 945-958."
  34: template <typename ArgumentContainer> struct jso_parameters {
  35:   using Real = typename ArgumentContainer::value_type;
  36:   using DimensionlessReal = decltype(Real()/Real());
~~~
- **EN:** This block imports dependencies such as utility, vector so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost) to keep symbols organized.
- **CN:** 此代码块引入了 utility, vector 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost），以保持符号组织清晰。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:   ArgumentContainer lower_bounds;
  38:   ArgumentContainer upper_bounds;
  39:   // Population in the first generation.
  40:   // This defaults to 0, which indicates "use the default specified in the
  41:   // referenced paper". To wit, initial population size
  42:   // =ceil(25log(D+1)sqrt(D)), where D is the dimension of the problem.
  43:   size_t initial_population_size = 0;
  44:   // Maximum number of function evaluations.
  45:   // The default of 0 indicates "use max_function_evaluations = 10,000D", where
  46:   // D is the dimension of the problem.
  47:   size_t max_function_evaluations = 0;
  48:   size_t threads = std::thread::hardware_concurrency();
  49:   ArgumentContainer const *initial_guess = nullptr;
  50: };
  51: 
  52: template <typename ArgumentContainer>
  53: void validate_jso_parameters(jso_parameters<ArgumentContainer> &jso_params) {
  54:   using std::isfinite;
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as std::thread::hardware_concurrency, validate_jso_parameters.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 std::thread::hardware_concurrency, validate_jso_parameters。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:   using std::isnan;
  56:   std::ostringstream oss;
  57:   if (jso_params.threads == 0) {
  58:     oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
  59:     oss << ": Requested zero threads to perform the calculation, but at least "
  60:            "1 is required.";
  61:     throw std::invalid_argument(oss.str());
  62:   }
  63:   detail::validate_bounds(jso_params.lower_bounds, jso_params.upper_bounds);
  64:   auto const dimension = jso_params.lower_bounds.size();
  65:   if (jso_params.initial_population_size == 0) {
  66:     // Ever so slightly different than the reference-the dimension can be 1,
  67:     // but if we followed the reference, the population size would then be zero.
  68:     jso_params.initial_population_size = static_cast<size_t>(
  69:         std::ceil(25 * std::log(dimension + 1.0) * sqrt(dimension)));
  70:   }
  71:   if (jso_params.max_function_evaluations == 0) {
  72:     // Recommended value from the reference:
~~~
- **EN:** This range declares or defines callable logic such as std::invalid_argument, detail::validate_bounds, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::invalid_argument, detail::validate_bounds, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:     jso_params.max_function_evaluations = 10000 * dimension;
  74:   }
  75:   if (jso_params.initial_population_size < 4) {
  76:     oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
  77:     oss << ": The population size must be at least 4, but requested population "
  78:            "size of "
  79:         << jso_params.initial_population_size << ".";
  80:     throw std::invalid_argument(oss.str());
  81:   }
  82:   if (jso_params.threads > jso_params.initial_population_size) {
  83:     oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
  84:     oss << ": There must be more individuals in the population than threads.";
  85:     throw std::invalid_argument(oss.str());
  86:   }
  87:   if (jso_params.initial_guess) {
  88:     detail::validate_initial_guess(*jso_params.initial_guess,
  89:                                    jso_params.lower_bounds,
  90:                                    jso_params.upper_bounds);
~~~
- **EN:** This range declares or defines callable logic such as std::invalid_argument. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::invalid_argument。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:   }
  92: }
  93: 
  94: template <typename ArgumentContainer, class Func, class URBG>
  95: ArgumentContainer
  96: jso(const Func cost_function, jso_parameters<ArgumentContainer> &jso_params,
  97:     URBG &gen,
  98:     std::invoke_result_t<Func, ArgumentContainer> target_value =
  99:         std::numeric_limits<
 100:             std::invoke_result_t<Func, ArgumentContainer>>::quiet_NaN(),
 101:     std::atomic<bool> *cancellation = nullptr,
 102:     std::atomic<std::invoke_result_t<Func, ArgumentContainer>>
 103:         *current_minimum_cost = nullptr,
 104:     std::vector<std::pair<ArgumentContainer,
 105:                           std::invoke_result_t<Func, ArgumentContainer>>>
 106:         *queries = nullptr) {
 107:   using Real = typename ArgumentContainer::value_type;
 108:   using DimensionlessReal = decltype(Real()/Real());
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `Func` as part of the file's main abstraction. This range declares or defines callable logic such as decltype.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `Func`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 decltype。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:   validate_jso_parameters(jso_params);
 110: 
 111:   using ResultType = std::invoke_result_t<Func, ArgumentContainer>;
 112:   using std::abs;
 113:   using std::cauchy_distribution;
 114:   using std::clamp;
 115:   using std::isnan;
 116:   using std::max;
 117:   using std::round;
 118:   using std::isfinite;
 119:   using std::uniform_real_distribution;
 120: 
 121:   // Refer to the referenced paper, pseudocode on page 1313:
 122:   // Algorithm 1, jSO, Line 1:
 123:   std::vector<ArgumentContainer> archive;
 124: 
 125:   // Algorithm 1, jSO, Line 2
 126:   // "Initialize population P_g = (x_0,g, ..., x_{np-1},g) randomly.
~~~
- **EN:** This range declares or defines callable logic such as validate_jso_parameters. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围声明或定义了可调用逻辑，例如 validate_jso_parameters。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:   auto population = detail::random_initial_population(
 128:       jso_params.lower_bounds, jso_params.upper_bounds,
 129:       jso_params.initial_population_size, gen);
 130:   if (jso_params.initial_guess) {
 131:     population[0] = *jso_params.initial_guess;
 132:   }
 133:   size_t dimension = jso_params.lower_bounds.size();
 134:   // Don't force the user to initialize to sane value:
 135:   if (current_minimum_cost) {
 136:     *current_minimum_cost = std::numeric_limits<ResultType>::infinity();
 137:   }
 138:   std::atomic<bool> target_attained = false;
 139:   std::vector<ResultType> cost(jso_params.initial_population_size,
 140:                                std::numeric_limits<ResultType>::quiet_NaN());
 141:   for (size_t i = 0; i < cost.size(); ++i) {
 142:     cost[i] = cost_function(population[i]);
 143:     if (!isnan(target_value) && cost[i] <= target_value) {
 144:       target_attained = true;
~~~
- **EN:** This range declares or defines callable logic such as size, infinity, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 size, infinity, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:     }
 146:     if (current_minimum_cost && cost[i] < *current_minimum_cost) {
 147:       *current_minimum_cost = cost[i];
 148:     }
 149:     if (queries) {
 150:       queries->push_back(std::make_pair(population[i], cost[i]));
 151:     }
 152:   }
 153:   std::vector<size_t> indices = detail::best_indices(cost);
 154:   std::atomic<size_t> function_evaluations = population.size();
 155: #if BOOST_MATH_DEBUG_JSO
 156:   {
 157:     auto const &min_cost = cost[indices[0]];
 158:     auto const &location = population[indices[0]];
 159:     std::cout << __FILE__ << ":" << __LINE__ << ":" << __func__;
 160:     std::cout << "\n\tMinimum cost after evaluation of initial random "
 161:                  "population of size "
 162:               << population.size() << " is " << min_cost << "."
~~~
- **EN:** This range declares or defines callable logic such as push_back, detail::best_indices, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 push_back, detail::best_indices, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:               << "\n\tLocation {";
 164:     for (size_t i = 0; i < location.size() - 1; ++i) {
 165:       std::cout << location[i] << ", ";
 166:     }
 167:     std::cout << location.back() << "}.\n";
 168:   }
 169: #endif
 170:   // Algorithm 1: jSO algorithm, Line 3:
 171:   // "Set all values in M_F to 0.5":
 172:   // I believe this is a typo: Compare with "Section B. Experimental Results",
 173:   // last bullet, which claims this should be set to 0.3. The reference
 174:   // implementation also does 0.3:
 175:   size_t H = 5;
 176:   std::vector<DimensionlessReal> M_F(H, static_cast<DimensionlessReal>(0.3));
 177:   // Algorithm 1: jSO algorithm, Line 4:
 178:   // "Set all values in M_CR to 0.8":
 179:   std::vector<DimensionlessReal> M_CR(H, static_cast<DimensionlessReal>(0.8));
 180: 
~~~
- **EN:** This range declares or defines callable logic such as M_F, M_CR. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 M_F, M_CR。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:   std::uniform_real_distribution<DimensionlessReal> unif01(DimensionlessReal(0), DimensionlessReal(1));
 182:   bool keep_going = !target_attained;
 183:   if (cancellation && *cancellation) {
 184:     keep_going = false;
 185:   }
 186:   // k from:
 187:   // http://metahack.org/CEC2014-Tanabe-Fukunaga.pdf, Algorithm 1:
 188:   // Determines where Lehmer means are stored in the memory:
 189:   size_t k = 0;
 190:   size_t minimum_population_size = (max)(size_t(4), size_t(jso_params.threads));
 191:   while (keep_going) {
 192:     // Algorithm 1, jSO, Line 7:
 193:     std::vector<DimensionlessReal> S_CR;
 194:     std::vector<DimensionlessReal> S_F;
 195:     // Equation 9 of L-SHADE:
 196:     std::vector<ResultType> delta_f;
 197:     for (size_t i = 0; i < population.size(); ++i) {
 198:       // Algorithm 1, jSO, Line 9:
~~~
- **EN:** This range declares or defines callable logic such as unif01, size_t. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 unif01, size_t。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:       auto ri = gen() % H;
 200:       // Algorithm 1, jSO, Line 10-13:
 201:       // Again, what is written in the pseudocode is not quite right.
 202:       // What they mean is mu_F = 0.9-the historical memory is not used.
 203:       // I confess I find it weird to store the historical memory if we're just
 204:       // gonna ignore it, but that's what the paper and the reference
 205:       // implementation says!
 206:       DimensionlessReal mu_F = static_cast<DimensionlessReal>(0.9);
 207:       DimensionlessReal mu_CR = static_cast<DimensionlessReal>(0.9);
 208:       if (ri != H - 1) {
 209:         mu_F = M_F[ri];
 210:         mu_CR = M_CR[ri];
 211:       }
 212:       // Algorithm 1, jSO, Line 14-18:
 213:       DimensionlessReal crossover_probability = static_cast<DimensionlessReal>(0);
 214:       if (mu_CR >= 0) {
 215:         using std::normal_distribution;
 216:         normal_distribution<DimensionlessReal> normal(mu_CR, static_cast<DimensionlessReal>(0.1));
~~~
- **EN:** This range declares or defines callable logic such as normal. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 normal。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:         crossover_probability = normal(gen);
 218:         // Clamp comes from L-SHADE description:
 219:         crossover_probability = clamp(crossover_probability, DimensionlessReal(0), DimensionlessReal(1));
 220:       }
 221:       // Algorithm 1, jSO, Line 19-23:
 222:       // Note that the pseudocode uses a "max_generations parameter",
 223:       // but the reference implementation does not.
 224:       // Since we already require specification of max_function_evaluations,
 225:       // the pseudocode adds an unnecessary parameter.
 226:       if (4 * function_evaluations < jso_params.max_function_evaluations) {
 227:         crossover_probability = (max)(crossover_probability, DimensionlessReal(0.7));
 228:       } else if (2 * function_evaluations <
 229:                  jso_params.max_function_evaluations) {
 230:         crossover_probability = (max)(crossover_probability, DimensionlessReal(0.6));
 231:       }
 232: 
 233:       // Algorithm 1, jSO, Line 24-27:
 234:       // Note the adjustments to the pseudocode given in the reference
~~~
- **EN:** This range declares or defines callable logic such as normal, clamp, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 normal, clamp, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:       // implementation.
 236:       cauchy_distribution<DimensionlessReal> cauchy(mu_F, static_cast<DimensionlessReal>(0.1));
 237:       DimensionlessReal F;
 238:       do {
 239:         F = cauchy(gen);
 240:         if (F > 1) {
 241:           F = 1;
 242:         }
 243:       } while (F <= 0);
 244:       DimensionlessReal threshold = static_cast<DimensionlessReal>(7) / static_cast<DimensionlessReal>(10);
 245:       if ((10 * function_evaluations <
 246:            6 * jso_params.max_function_evaluations) &&
 247:           (F > threshold)) {
 248:         F = threshold;
 249:       }
 250:       // > p value for mutation strategy linearly decreases from pmax to pmin
 251:       // during the evolutionary process, > where pmax = 0.25 in jSO and pmin =
 252:       // pmax/2.
~~~
- **EN:** This range declares or defines callable logic such as cauchy, while. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 cauchy, while。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:       DimensionlessReal p = DimensionlessReal(0.25) * (1 - static_cast<DimensionlessReal>(function_evaluations) /
 254:                                      (2 * jso_params.max_function_evaluations));
 255:       // Equation (4) of the reference:
 256:       DimensionlessReal Fw = static_cast<DimensionlessReal>(1.2) * F;
 257:       if (10 * function_evaluations < 4 * jso_params.max_function_evaluations) {
 258:         if (10 * function_evaluations <
 259:             2 * jso_params.max_function_evaluations) {
 260:           Fw = static_cast<DimensionlessReal>(0.7) * F;
 261:         } else {
 262:           Fw = static_cast<DimensionlessReal>(0.8) * F;
 263:         }
 264:       }
 265:       // Algorithm 1, jSO, Line 28:
 266:       // "ui,g := current-to-pBest-w/1/bin using Eq. (3)"
 267:       // This is not explained in the reference, but "current-to-pBest"
 268:       // strategy means "select randomly among the best values available."
 269:       // See:
 270:       // Zhang, J., & Sanderson, A. C. (2009).
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:       // JADE: adaptive differential evolution with optional external archive.
 272:       // IEEE Transactions on evolutionary computation, 13(5), 945-958.
 273:       // > As a generalization of DE/current-to- best,
 274:       // > DE/current-to-pbest utilizes not only the best solution information
 275:       // > but also the information of other good solutions.
 276:       // > To be specific, any of the top 100p%, p in (0, 1],
 277:       // > solutions can be randomly chosen in DE/current-to-pbest to play the
 278:       // role > designed exclusively for the single best solution in
 279:       // DE/current-to-best."
 280:       size_t max_p_index = static_cast<size_t>(std::ceil(p * indices.size()));
 281:       size_t p_best_idx = gen() % max_p_index;
 282:       // We now need r1, r2 so that r1 != r2 != i:
 283:       size_t r1;
 284:       do {
 285:         r1 = gen() % population.size();
 286:       } while (r1 == i);
 287:       size_t r2;
 288:       do {
~~~
- **EN:** This range declares or defines callable logic such as std::ceil, gen, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::ceil, gen, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:         r2 = gen() % (population.size() + archive.size());
 290:       } while (r2 == r1 || r2 == i);
 291: 
 292:       ArgumentContainer trial_vector;
 293:       if constexpr (detail::has_resize_v<ArgumentContainer>) {
 294:         trial_vector.resize(dimension);
 295:       }
 296:       auto const &xi = population[i];
 297:       auto const &xr1 = population[r1];
 298:       ArgumentContainer xr2;
 299:       if (r2 < population.size()) {
 300:         xr2 = population[r2];
 301:       } else {
 302:         xr2 = archive[r2 - population.size()];
 303:       }
 304:       auto const &x_p_best = population[p_best_idx];
 305:       for (size_t j = 0; j < dimension; ++j) {
 306:         auto guaranteed_changed_idx = gen() % dimension;
~~~
- **EN:** This range declares or defines callable logic such as gen, while, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 gen, while, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:         if (unif01(gen) < crossover_probability ||
 308:             j == guaranteed_changed_idx) {
 309:           auto tmp = xi[j] + Fw * (x_p_best[j] - xi[j]) + F * (xr1[j] - xr2[j]);
 310:           auto const &lb = jso_params.lower_bounds[j];
 311:           auto const &ub = jso_params.upper_bounds[j];
 312:           // Some others recommend regenerating the indices rather than
 313:           // clamping; I dunno seems like it could get stuck regenerating . . .
 314:           // Another suggestion is provided in:
 315:           // "JADE: Adaptive Differential Evolution with Optional External
 316:           // Archive" page 947. Perhaps we should implement it!
 317:           trial_vector[j] = clamp(tmp, lb, ub);
 318:         } else {
 319:           trial_vector[j] = population[i][j];
 320:         }
 321:       }
 322:       auto trial_cost = cost_function(trial_vector);
 323:       function_evaluations++;
 324:       if (isnan(trial_cost)) {
~~~
- **EN:** This range declares or defines callable logic such as clamp, cost_function. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 clamp, cost_function。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:         continue;
 326:       }
 327:       if (queries) {
 328:         queries->push_back(std::make_pair(trial_vector, trial_cost));
 329:       }
 330: 
 331:       // Successful trial:
 332:       if (trial_cost < cost[i] || isnan(cost[i])) {
 333:         if (!isnan(target_value) && trial_cost <= target_value) {
 334:           target_attained = true;
 335:         }
 336:         if (current_minimum_cost && trial_cost < *current_minimum_cost) {
 337:           *current_minimum_cost = trial_cost;
 338:         }
 339:         // Can't decide on improvement if the previous evaluation was a NaN:
 340:         if (!isnan(cost[i])) {
 341:           if (crossover_probability > 1 || crossover_probability < 0) {
 342:             throw std::domain_error("Crossover probability is weird.");
~~~
- **EN:** This range declares or defines callable logic such as push_back, std::domain_error. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 push_back, std::domain_error。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:           }
 344:           if (F > 1 || F < 0) {
 345:             throw std::domain_error("Scale factor (F) is weird.");
 346:           }
 347:           S_CR.push_back(crossover_probability);
 348:           S_F.push_back(F);
 349:           delta_f.push_back(abs(cost[i] - trial_cost));
 350:         }
 351:         // Build the historical archive:
 352:         // The historical archive stores inferior solutions for the purpose of maintaining diversity.
 353:         if (archive.size() < cost.size()) {
 354:           archive.push_back(population[i]);
 355:         } else {
 356:           // If it's already built, then put the eliminated individual in a random index:
 357:           archive.resize(cost.size());
 358:           auto idx = gen() % archive.size();
 359:           archive[idx] = population[i];
 360:         }
~~~
- **EN:** This range declares or defines callable logic such as std::domain_error, push_back, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::domain_error, push_back, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:         cost[i] = trial_cost;
 362:         population[i] = trial_vector;
 363:       }
 364:     }
 365: 
 366:     indices = detail::best_indices(cost);
 367: 
 368:     // If there are no successful updates this generation, we do not update the
 369:     // historical memory:
 370:     if (S_CR.size() > 0) {
 371:       std::vector<DimensionlessReal> weights(S_CR.size(),
 372:                                 std::numeric_limits<DimensionlessReal>::quiet_NaN());
 373:       ResultType delta_sum = static_cast<ResultType>(0);
 374:       for (auto const &delta : delta_f) {
 375:         delta_sum += delta;
 376:       }
 377:       if (delta_sum <= 0 || !isfinite(delta_sum)) {
 378:         std::ostringstream oss;
~~~
- **EN:** This range declares or defines callable logic such as detail::best_indices, quiet_NaN. Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::best_indices, quiet_NaN。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 379-396 / 第 379-396 行
~~~cpp
 379:         oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
 380:         oss << "\n\tYou've hit a bug: The sum of improvements must be strictly "
 381:                "positive, but got "
 382:             << delta_sum << " improvement from " << S_CR.size()
 383:             << " successful updates.\n";
 384:         oss << "\tImprovements: {" << std::hexfloat;
 385:         for (size_t l = 0; l < delta_f.size() -1; ++l) {
 386:           oss << delta_f[l] << ", ";
 387:         }
 388:         oss << delta_f.back() << "}.\n";
 389:         throw std::logic_error(oss.str());
 390:       }
 391:       for (size_t i = 0; i < weights.size(); ++i) {
 392:         weights[i] = delta_f[i] / delta_sum;
 393:       }
 394: 
 395:       M_CR[k] = detail::weighted_lehmer_mean(S_CR, weights);
 396:       M_F[k] = detail::weighted_lehmer_mean(S_F, weights);
~~~
- **EN:** This range declares or defines callable logic such as std::logic_error, detail::weighted_lehmer_mean. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::logic_error, detail::weighted_lehmer_mean。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 397-414 / 第 397-414 行
~~~cpp
 397:     }
 398:     k++;
 399:     if (k == M_F.size()) {
 400:       k = 0;
 401:     }
 402:     if (target_attained) {
 403:       break;
 404:     }
 405:     if (cancellation && *cancellation) {
 406:       break;
 407:     }
 408:     if (function_evaluations >= jso_params.max_function_evaluations) {
 409:       break;
 410:     }
 411:     // Linear population size reduction:
 412:     size_t new_population_size = static_cast<size_t>(round(
 413:         -double(jso_params.initial_population_size - minimum_population_size) *
 414:             function_evaluations /
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 415-432 / 第 415-432 行
~~~cpp
 415:             static_cast<double>(jso_params.max_function_evaluations) +
 416:         jso_params.initial_population_size));
 417:     size_t num_erased = population.size() - new_population_size;
 418:     std::vector<size_t> indices_to_erase(num_erased);
 419:     size_t j = 0;
 420:     for (size_t i = population.size() - 1; i >= new_population_size; --i) {
 421:       indices_to_erase[j++] = indices[i];
 422:     }
 423:     std::sort(indices_to_erase.rbegin(), indices_to_erase.rend());
 424:     for (auto const &idx : indices_to_erase) {
 425:       population.erase(population.begin() + idx);
 426:       cost.erase(cost.begin() + idx);
 427:     }
 428:     indices.resize(new_population_size);
 429:   }
 430: 
 431: #if BOOST_MATH_DEBUG_JSO
 432:   {
~~~
- **EN:** This range declares or defines callable logic such as indices_to_erase, std::sort, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 indices_to_erase, std::sort, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 433-450 / 第 433-450 行
~~~cpp
 433:     auto const &min_cost = cost[indices[0]];
 434:     auto const &location = population[indices[0]];
 435:     std::cout << __FILE__ << ":" << __LINE__ << ":" << __func__;
 436:     std::cout << "\n\tMinimum cost after completion is " << min_cost
 437:               << ".\n\tLocation: {";
 438:     for (size_t i = 0; i < location.size() - 1; ++i) {
 439:       std::cout << location[i] << ", ";
 440:     }
 441:     std::cout << location.back() << "}.\n";
 442:     std::cout << "\tRequired " << function_evaluations
 443:               << " function calls out of "
 444:               << jso_params.max_function_evaluations << " allowed.\n";
 445:     if (target_attained) {
 446:       std::cout << "\tReason for return: Target value attained.\n";
 447:     }
 448:     std::cout << "\tHistorical crossover probabilities (M_CR): {";
 449:     for (size_t i = 0; i < M_CR.size() - 1; ++i) {
 450:       std::cout << M_CR[i] << ", ";
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Return statements hand the computed result or status back to the caller.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 return 语句会把计算结果或状态返回给调用方。

### Lines 451-465 / 第 451-465 行
~~~cpp
 451:     }
 452:     std::cout << M_CR.back() << "}.\n";
 453:     std::cout << "\tHistorical scale factors (M_F): {";
 454:     for (size_t i = 0; i < M_F.size() - 1; ++i) {
 455:       std::cout << M_F[i] << ", ";
 456:     }
 457:     std::cout << M_F.back() << "}.\n";
 458:     std::cout << "\tFinal population size: " << population.size() << ".\n";
 459:   }
 460: #endif
 461:   return population[indices[0]];
 462: }
 463: 
 464: } // namespace boost::math::optimization
 465: #endif
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Return statements hand the computed result or status back to the caller.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 return 语句会把计算结果或状态返回给调用方。

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
- **Included headers / 包含头文件**: `atomic, boost/math/optimization/detail/common.hpp, cmath, iostream, limits, random, sstream, stdexcept, thread, type_traits, utility, vector`
- **Namespaces / 命名空间**: `boost`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `decltype, std::thread::hardware_concurrency, validate_jso_parameters, std::invalid_argument, detail::validate_bounds, size, std::ceil, infinity, ...`
