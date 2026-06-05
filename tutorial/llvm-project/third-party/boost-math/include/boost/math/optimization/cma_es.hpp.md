# cma_es.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/optimization/cma_es.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This Boost.Math header provides reusable support for optimization cma es.
- **作用（中文）**: 此 Boost.Math 头文件为 optimization cma es 提供可复用支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: /*
   2:  * Copyright Nick Thompson, 2024
   3:  * Use, modification and distribution are subject to the
   4:  * Boost Software License, Version 1.0. (See accompanying file
   5:  * LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   6:  */
   7: #ifndef BOOST_MATH_OPTIMIZATION_CMA_ES_HPP
   8: #define BOOST_MATH_OPTIMIZATION_CMA_ES_HPP
   9: #include <atomic>
  10: #include <cmath>
  11: #include <iostream>
  12: #include <limits>
  13: #include <random>
  14: #include <sstream>
  15: #include <stdexcept>
  16: #include <utility>
  17: #include <vector>
  18: #include <boost/math/optimization/detail/common.hpp>
~~~
- **EN:** This block imports dependencies such as atomic, cmath, iostream, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 atomic, cmath, iostream, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: #include <boost/math/tools/assert.hpp>
  20: #if __has_include(<Eigen/Dense>)
  21: #include <Eigen/Dense>
  22: #else
  23: #error "CMA-ES requires Eigen."
  24: #endif
  25: 
  26: // Follows the notation in:
  27: // https://arxiv.org/pdf/1604.00772.pdf
  28: // This is a (hopefully) faithful reproduction of the pseudocode in the arxiv review
  29: // by Nikolaus Hansen.
  30: // Comments referring to equations all refer to this arxiv review.
  31: // A slide deck by the same author is given here:
  32: // http://www.cmap.polytechnique.fr/~nikolaus.hansen/CmaTutorialGecco2023-no-audio.pdf
  33: // which is also a very useful reference.
  34: 
  35: #ifndef BOOST_MATH_DEBUG_CMA_ES
  36: #define BOOST_MATH_DEBUG_CMA_ES 0
~~~
- **EN:** This block imports dependencies such as boost/math/tools/assert.hpp, Eigen/Dense so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此代码块引入了 boost/math/tools/assert.hpp, Eigen/Dense 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37: #endif
  38: 
  39: namespace boost::math::optimization {
  40: 
  41: template <typename ArgumentContainer> struct cma_es_parameters {
  42:   using Real = typename ArgumentContainer::value_type;
  43:   using DimensionlessReal = decltype(Real()/Real());
  44:   ArgumentContainer lower_bounds;
  45:   ArgumentContainer upper_bounds;
  46:   size_t max_generations = 1000;
  47:   ArgumentContainer const *initial_guess = nullptr;
  48:   // In the reference, population size = \lambda.
  49:   // If the population size is zero, it is set to equation (48) of the reference
  50:   // and rounded up to the nearest multiple of threads:
  51:   size_t population_size = 0;
  52:   // In the reference, learning_rate = c_m:
  53:   DimensionlessReal learning_rate = 1;
  54: };
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the struct `cma_es_parameters` as part of the file's main abstraction.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 struct `cma_es_parameters`，作为该文件核心抽象的一部分。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55: 
  56: template <typename ArgumentContainer>
  57: void validate_cma_es_parameters(cma_es_parameters<ArgumentContainer> &params) {
  58:   using Real = typename ArgumentContainer::value_type;
  59:   using DimensionlessReal = decltype(Real()/Real());
  60:   using std::isfinite;
  61:   using std::isnan;
  62:   using std::log;
  63:   using std::ceil;
  64:   using std::floor;
  65: 
  66:   std::ostringstream oss;
  67:   detail::validate_bounds(params.lower_bounds, params.upper_bounds);
  68:   if (params.initial_guess) {
  69:     detail::validate_initial_guess(*params.initial_guess, params.lower_bounds, params.upper_bounds);
  70:   }
  71:   const size_t n = params.upper_bounds.size();
  72:   // Equation 48 of the arxiv review:
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. This range declares or defines callable logic such as validate_cma_es_parameters, decltype, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 此范围声明或定义了可调用逻辑，例如 validate_cma_es_parameters, decltype, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:   if (params.population_size == 0) {
  74:     //auto tmp = 4.0 + floor(3*log(n));
  75:     // But round to the nearest multiple of the thread count:
  76:     //auto k = static_cast<size_t>(std::ceil(tmp/params.threads));
  77:     //params.population_size = k*params.threads;
  78:     params.population_size = static_cast<size_t>(4 + floor(3*log(n)));
  79:   }
  80:   if (params.learning_rate <= DimensionlessReal(0) || !isfinite(params.learning_rate)) {
  81:     oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
  82:     oss << ": The learning rate must be > 0, but got " << params.learning_rate << ".";
  83:     throw std::invalid_argument(oss.str());
  84:   }
  85: }
  86: 
  87: template <typename ArgumentContainer, class Func, class URBG>
  88: ArgumentContainer cma_es(
  89:     const Func cost_function,
  90:     cma_es_parameters<ArgumentContainer> &params,
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `Func` as part of the file's main abstraction. This range declares or defines callable logic such as floor, std::ceil, ....
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `Func`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 floor, std::ceil, ...。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:     URBG &gen,
  92:     std::invoke_result_t<Func, ArgumentContainer> target_value = std::numeric_limits<std::invoke_result_t<Func, ArgumentContainer>>::quiet_NaN(),
  93:     std::atomic<bool> *cancellation = nullptr,
  94:     std::atomic<std::invoke_result_t<Func, ArgumentContainer>> *current_minimum_cost = nullptr,
  95:     std::vector<std::pair<ArgumentContainer, std::invoke_result_t<Func, ArgumentContainer>>> *queries = nullptr)
  96:  {
  97:   using Real = typename ArgumentContainer::value_type;
  98:   using DimensionlessReal = decltype(Real()/Real());
  99:   using ResultType = std::invoke_result_t<Func, ArgumentContainer>;
 100:   using std::abs;
 101:   using std::log;
 102:   using std::exp;
 103:   using std::pow;
 104:   using std::min;
 105:   using std::max;
 106:   using std::sqrt;
 107:   using std::isnan;
 108:   using std::isfinite;
~~~
- **EN:** This range declares or defines callable logic such as decltype. Type aliases simplify verbose template names and make later declarations easier to read.
- **CN:** 此范围声明或定义了可调用逻辑，例如 decltype。 类型别名可以简化冗长的模板名称，使后续声明更易阅读。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:   using std::uniform_real_distribution;
 110:   using std::normal_distribution;
 111:   validate_cma_es_parameters(params);
 112:   // n = dimension of problem:
 113:   const size_t n = params.lower_bounds.size();
 114:   std::atomic<bool> target_attained = false;
 115:   std::atomic<ResultType> lowest_cost = std::numeric_limits<ResultType>::infinity();
 116:   ArgumentContainer best_vector;
 117:   // p_{c} := evolution path, equation (24) of the arxiv review:
 118:   Eigen::Vector<DimensionlessReal, Eigen::Dynamic> p_c(n);
 119:   // p_{\sigma} := conjugate evolution path, equation (31) of the arxiv review:
 120:   Eigen::Vector<DimensionlessReal, Eigen::Dynamic> p_sigma(n);
 121:   if constexpr (detail::has_resize_v<ArgumentContainer>) {
 122:     best_vector.resize(n, std::numeric_limits<Real>::quiet_NaN());
 123:   }
 124:   for (size_t i = 0; i < n; ++i) {
 125:     p_c[i] = DimensionlessReal(0);
 126:     p_sigma[i] = DimensionlessReal(0);
~~~
- **EN:** This range declares or defines callable logic such as validate_cma_es_parameters, size, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 validate_cma_es_parameters, size, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:   }
 128:   // Table 1, \mu = floor(\lambda/2):
 129:   size_t mu = params.population_size/2;
 130:   std::vector<DimensionlessReal> w_prime(params.population_size, std::numeric_limits<DimensionlessReal>::quiet_NaN());
 131:   for (size_t i = 0; i < params.population_size; ++i) {
 132:     // Equation (49), but 0-indexed:
 133:     w_prime[i] = log(static_cast<DimensionlessReal>(params.population_size + 1)/(2*(i+1)));
 134:   }
 135:   // Table 1, notes at top:
 136:   DimensionlessReal positive_weight_sum = 0;
 137:   DimensionlessReal sq_weight_sum = 0;
 138:   for (size_t i = 0; i < mu; ++i) {
 139:     BOOST_MATH_ASSERT(w_prime[i] > 0);
 140:     positive_weight_sum += w_prime[i];
 141:     sq_weight_sum += w_prime[i]*w_prime[i];
 142:   }
 143:   DimensionlessReal mu_eff = positive_weight_sum*positive_weight_sum/sq_weight_sum;
 144:   BOOST_MATH_ASSERT(1 <= mu_eff);
~~~
- **EN:** This range declares or defines callable logic such as w_prime, log, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 w_prime, log, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:   BOOST_MATH_ASSERT(mu_eff <= mu);
 146:   DimensionlessReal negative_weight_sum = 0;
 147:   sq_weight_sum = 0;
 148:   for (size_t i = mu; i < params.population_size; ++i) {
 149:     BOOST_MATH_ASSERT(w_prime[i] <= 0);
 150:     negative_weight_sum += w_prime[i];
 151:     sq_weight_sum += w_prime[i]*w_prime[i];
 152:   }
 153:   DimensionlessReal mu_eff_m = negative_weight_sum*negative_weight_sum/sq_weight_sum;
 154:   // Equation (54):
 155:   DimensionlessReal c_m = params.learning_rate;
 156:   // Equation (55):
 157:   DimensionlessReal c_sigma = (mu_eff + 2)/(n + mu_eff + 5);
 158:   BOOST_MATH_ASSERT(c_sigma < 1);
 159:   DimensionlessReal d_sigma = 1 + 2*(max)(DimensionlessReal(0), sqrt(DimensionlessReal((mu_eff - 1)/(n + 1))) - DimensionlessReal(1)) + c_sigma;
 160:   // Equation (56):
 161:   DimensionlessReal c_c = (4 + mu_eff/n)/(n + 4 + 2*mu_eff/n);
 162:   BOOST_MATH_ASSERT(c_c <= 1);
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT. Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:   // Equation (57):
 164:   DimensionlessReal c_1 = DimensionlessReal(2)/(pow(n + 1.3, 2) + mu_eff);
 165:   // Equation (58)
 166:   DimensionlessReal c_mu = (min)(1 - c_1, 2*(DimensionlessReal(0.25)  + mu_eff  + 1/mu_eff - 2)/((n+2)*(n+2) + mu_eff));
 167:   BOOST_MATH_ASSERT(c_1 + c_mu <= DimensionlessReal(1));
 168:   // Equation (50):
 169:   DimensionlessReal alpha_mu_m = 1 + c_1/c_mu;
 170:   // Equation (51):
 171:   DimensionlessReal alpha_mu_eff_m = 1 + 2*mu_eff_m/(mu_eff + 2);
 172:   // Equation (52):
 173:   DimensionlessReal alpha_m_pos_def = (1- c_1 - c_mu)/(n*c_mu);
 174:   // Equation (53):
 175:   std::vector<DimensionlessReal> weights(params.population_size, std::numeric_limits<DimensionlessReal>::quiet_NaN());
 176:   for (size_t i = 0; i < mu; ++i) {
 177:     weights[i] = w_prime[i]/positive_weight_sum;
 178:   }
 179:   DimensionlessReal min_alpha = (min)(alpha_mu_m, (min)(alpha_mu_eff_m, alpha_m_pos_def));
 180:   for (size_t i = mu; i < params.population_size; ++i) {
~~~
- **EN:** This range declares or defines callable logic such as DimensionlessReal, BOOST_MATH_ASSERT, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 DimensionlessReal, BOOST_MATH_ASSERT, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:     weights[i] = min_alpha*w_prime[i]/abs(negative_weight_sum);
 182:   }
 183:   // mu:= number of parents, lambda := number of offspring.
 184:   Eigen::Matrix<DimensionlessReal, Eigen::Dynamic, Eigen::Dynamic> C = Eigen::Matrix<DimensionlessReal, Eigen::Dynamic, Eigen::Dynamic>::Identity(n, n);
 185:   ArgumentContainer mean_vector;
 186:   // See the footnote in Figure 6 of the arxiv review:
 187:   // We should consider the more robust initialization described there. . .
 188:   Real sigma = DimensionlessReal(0.3)*(params.upper_bounds[0] - params.lower_bounds[0]);;
 189:   if (params.initial_guess) {
 190:     mean_vector = *params.initial_guess;
 191:   }
 192:   else {
 193:     mean_vector = detail::random_initial_population(params.lower_bounds, params.upper_bounds, 1, gen)[0];
 194:   }
 195:   auto initial_cost = cost_function(mean_vector);
 196:   if (!isnan(initial_cost)) {
 197:     best_vector = mean_vector;
 198:     lowest_cost = initial_cost;
~~~
- **EN:** This range declares or defines callable logic such as abs, Identity, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 abs, Identity, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:     if (current_minimum_cost) {
 200:       *current_minimum_cost = initial_cost;
 201:     }
 202:   }
 203: #if BOOST_MATH_DEBUG_CMA_ES
 204:   {
 205:     std::cout << __FILE__ << ":" << __LINE__ << ":" << __func__ << "\n";
 206:     std::cout << "\tRunning a (" << params.population_size/2 << "/" << params.population_size/2 << "_W, " << params.population_size << ")-aCMA Evolutionary Strategy on " << params.threads << " threads.\n";
 207:     std::cout << "\tInitial mean vector: {";
 208:     for (size_t i = 0; i < n - 1; ++i) {
 209:       std::cout << mean_vector[i] << ", ";
 210:     }
 211:     std::cout << mean_vector[n - 1] << "}.\n";
 212:     std::cout << "\tCost: " << lowest_cost << ".\n";
 213:     std::cout << "\tInitial step length: " << sigma << ".\n";
 214:     std::cout << "\tVariance effective selection mass: " << mu_eff << ".\n";
 215:     std::cout << "\tLearning rate for rank-one update of covariance matrix: " << c_1 << ".\n";
 216:     std::cout << "\tLearning rate for rank-mu update of covariance matrix: " << c_mu << ".\n";
~~~
- **EN:** Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:     std::cout << "\tDecay rate for cumulation path for step-size control: " << c_sigma << ".\n";
 218:     std::cout << "\tLearning rate for the mean: " << c_m << ".\n";
 219:     std::cout << "\tDamping parameter for step-size update: " << d_sigma << ".\n";
 220:   }
 221: #endif
 222:   size_t generation = 0;
 223: 
 224:   std::vector<Eigen::Vector<DimensionlessReal, Eigen::Dynamic>> ys(params.population_size);
 225:   std::vector<ArgumentContainer> xs(params.population_size);
 226:   std::vector<ResultType> costs(params.population_size, std::numeric_limits<ResultType>::quiet_NaN());
 227:   Eigen::Vector<DimensionlessReal, Eigen::Dynamic> weighted_avg_y(n);
 228:   Eigen::Vector<DimensionlessReal, Eigen::Dynamic> z(n);
 229:   if constexpr (detail::has_resize_v<ArgumentContainer>) {
 230:     for (auto & x : xs) {
 231:       x.resize(n, std::numeric_limits<Real>::quiet_NaN());
 232:     }
 233:   }
 234:   for (auto & y : ys) {
~~~
- **EN:** This range declares or defines callable logic such as ys, xs, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 ys, xs, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:     y.resize(n);
 236:   }
 237:   normal_distribution<DimensionlessReal> dis(DimensionlessReal(0), DimensionlessReal(1));
 238:   do {
 239:     if (cancellation && *cancellation) {
 240:       break;
 241:     }
 242:     // TODO: The reference contends the following in
 243:     // Section B.2 "Strategy internal numerical effort":
 244:     // "In practice, the re-calculation of B and D needs to be done not until about
 245:     // max(1, floor(1/(10n(c_1+c_mu)))) generations."
 246:     // Note that sigma can be dimensionless, in which case C carries the units.
 247:     // This is a weird decision-we're not gonna do that!
 248:     Eigen::SelfAdjointEigenSolver<Eigen::Matrix<DimensionlessReal, Eigen::Dynamic, Eigen::Dynamic>> eigensolver(C);
 249:     if (eigensolver.info() != Eigen::Success) {
 250:       std::ostringstream oss;
 251:       oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
 252:       oss << ": Could not decompose the covariance matrix as BDB^{T}.";
~~~
- **EN:** This range declares or defines callable logic such as resize, dis, .... Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 resize, dis, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:       throw std::logic_error(oss.str());
 254:     }
 255:     Eigen::Matrix<DimensionlessReal, Eigen::Dynamic, Eigen::Dynamic> B = eigensolver.eigenvectors();
 256:     // Eigen returns D^2, in the notation of the survey:
 257:     auto D = eigensolver.eigenvalues();
 258:     // So make it better:
 259:     for (auto & d : D) {
 260:       if (d <= 0 || isnan(d)) {
 261:         std::ostringstream oss;
 262:         oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
 263:         oss << ": The covariance matrix is not positive definite. This breaks the evolution path computation downstream.\n";
 264:         oss << "C=\n" << C << "\n";
 265:         oss << "Eigenvalues: " << D;
 266:         throw std::domain_error(oss.str());
 267:       }
 268:       d = sqrt(d);
 269:     }
 270: 
~~~
- **EN:** This range declares or defines callable logic such as std::logic_error, eigenvectors, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 std::logic_error, eigenvectors, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271:     for (size_t k = 0; k < params.population_size; ++k) {
 272:       auto & y = ys[k];
 273:       auto & x = xs[k];
 274:       BOOST_MATH_ASSERT(static_cast<size_t>(x.size()) == n);
 275:       BOOST_MATH_ASSERT(static_cast<size_t>(y.size()) == n);
 276:       size_t resample_counter = 0;
 277:       do {
 278:         // equation (39) of Figure 6:
 279:         // Also see equation (4):
 280:         for (size_t i = 0; i < n; ++i) {
 281:           z[i] = dis(gen);
 282:         }
 283:         Eigen::Vector<DimensionlessReal, Eigen::Dynamic> Dz(n);
 284:         for (size_t i = 0; i < n; ++i) {
 285:           Dz[i] = D[i]*z[i];
 286:         }
 287:         y = B*Dz;
 288:         for (size_t i = 0; i < n; ++i) {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, dis, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, dis, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 289-306 / 第 289-306 行
~~~cpp
 289:           BOOST_MATH_ASSERT(!isnan(mean_vector[i]));
 290:           BOOST_MATH_ASSERT(!isnan(y[i]));
 291:           x[i] = mean_vector[i] + sigma*y[i]; // equation (40) of Figure 6.
 292:         }
 293:         costs[k] = cost_function(x);
 294:         if (resample_counter++ == 50) {
 295:           std::ostringstream oss;
 296:           oss << __FILE__ << ":" << __LINE__ << ":" << __func__;
 297:           oss << ": 50 resamples was not sufficient to find an argument to the cost function which did not return NaN.";
 298:           oss << " Giving up.";
 299:           throw std::domain_error(oss.str());
 300:         }
 301:       } while (isnan(costs[k]));
 302: 
 303:       if (queries) {
 304:         queries->emplace_back(std::make_pair(x, costs[k]));
 305:       }
 306:       if (costs[k] < lowest_cost) {
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT, cost_function, .... Conditional branches split behavior across input ranges, error cases, or configuration modes. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT, cost_function, ...。 条件分支会根据输入区间、错误情况或配置模式切换行为。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 307-324 / 第 307-324 行
~~~cpp
 307:         lowest_cost = costs[k];
 308:         best_vector = x;
 309:         if (current_minimum_cost && costs[k] < *current_minimum_cost) {
 310:           *current_minimum_cost = costs[k];
 311:         }
 312:         if (lowest_cost < target_value) {
 313:           target_attained = true;
 314:           break;
 315:         }
 316:       }
 317:     }
 318:     if (target_attained) {
 319:       break;
 320:     }
 321:     if (cancellation && *cancellation) {
 322:       break;
 323:     }
 324:     auto indices = detail::best_indices(costs);
~~~
- **EN:** This range declares or defines callable logic such as detail::best_indices. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 detail::best_indices。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 325-342 / 第 325-342 行
~~~cpp
 325:     // Equation (41), Figure 6:
 326:     for (size_t j = 0; j < n; ++j) {
 327:       weighted_avg_y[j] = 0;
 328:       for (size_t i = 0; i < mu; ++i) {
 329:         BOOST_MATH_ASSERT(!isnan(weights[i]));
 330:         BOOST_MATH_ASSERT(!isnan(ys[indices[i]][j]));
 331:         weighted_avg_y[j] += weights[i]*ys[indices[i]][j];
 332:       }
 333:     }
 334:     // Equation (42), Figure 6:
 335:     for (size_t j = 0; j < n; ++j) {
 336:       mean_vector[j] = mean_vector[j] + c_m*sigma*weighted_avg_y[j];
 337:     }
 338:     // Equation (43), Figure 6: Start with C^{-1/2}<y>_{w}
 339:     Eigen::Vector<DimensionlessReal, Eigen::Dynamic> inv_D_B_transpose_y = B.transpose()*weighted_avg_y;
 340:     for (long j = 0; j < inv_D_B_transpose_y.size(); ++j) {
 341:       inv_D_B_transpose_y[j] /= D[j];
 342:     }
~~~
- **EN:** This range declares or defines callable logic such as BOOST_MATH_ASSERT. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围声明或定义了可调用逻辑，例如 BOOST_MATH_ASSERT。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 343-360 / 第 343-360 行
~~~cpp
 343:     Eigen::Vector<DimensionlessReal, Eigen::Dynamic> C_inv_sqrt_y_avg = B*inv_D_B_transpose_y;
 344:     // Equation (43), Figure 6:
 345:     DimensionlessReal p_sigma_norm = 0;
 346:     for (size_t j = 0; j < n; ++j) {
 347:       p_sigma[j] = (1-c_sigma)*p_sigma[j] + sqrt(c_sigma*(2-c_sigma)*mu_eff)*C_inv_sqrt_y_avg[j];
 348:       p_sigma_norm += p_sigma[j]*p_sigma[j];
 349:     }
 350:     p_sigma_norm = sqrt(p_sigma_norm);
 351:     // A: Algorithm Summary: E[||N(0,1)||]:
 352:     const DimensionlessReal expectation_norm_0I = sqrt(static_cast<DimensionlessReal>(n))*(DimensionlessReal(1) - DimensionlessReal(1)/(4*n) + DimensionlessReal(1)/(21*n*n));
 353:     // Equation (44), Figure 6:
 354:     sigma = sigma*exp(c_sigma*(p_sigma_norm/expectation_norm_0I -1)/d_sigma);
 355:     // A: Algorithm Summary:
 356:     DimensionlessReal h_sigma = 0;
 357:     DimensionlessReal rhs = (DimensionlessReal(1.4) + DimensionlessReal(2)/(n+1))*expectation_norm_0I*sqrt(1 - pow(1-c_sigma, 2*(generation+1)));
 358:     if (p_sigma_norm < rhs) {
 359:       h_sigma = 1;
 360:     }
~~~
- **EN:** This range declares or defines callable logic such as sqrt, exp, .... Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Conditional branches split behavior across input ranges, error cases, or configuration modes.
- **CN:** 此范围声明或定义了可调用逻辑，例如 sqrt, exp, ...。 这里的许多行是数值常量或系数表，用来支撑周围的算法。 条件分支会根据输入区间、错误情况或配置模式切换行为。

### Lines 361-378 / 第 361-378 行
~~~cpp
 361:     // Equation (45), Figure 6:
 362:     p_c = (1-c_c)*p_c + h_sigma*sqrt(c_c*(2-c_c)*mu_eff)*weighted_avg_y;
 363:     DimensionlessReal delta_h_sigma = (1-h_sigma)*c_c*(2-c_c);
 364:     DimensionlessReal weight_sum = 0;
 365:     for (auto & w : weights) {
 366:       weight_sum += w;
 367:     }
 368:     // Equation (47), Figure 6:
 369:     DimensionlessReal K = (1 + c_1*delta_h_sigma - c_1 - c_mu*weight_sum);
 370:     // Can these operations be sped up using `.selfadjointView<Eigen::Upper>`?
 371:     // Maybe: A.selfadjointView<Eigen::Lower>().rankUpdate(p_c, c_1);?
 372:     C = K*C + c_1*p_c*p_c.transpose();
 373:     // Incorporate positive weights of Equation (46):
 374:     for (size_t i = 0; i < params.population_size/2; ++i) {
 375:       C += c_mu*weights[i]*ys[indices[i]]*ys[indices[i]].transpose();
 376:     }
 377:     for (size_t i = params.population_size/2; i < params.population_size; ++i) {
 378:       Eigen::Vector<DimensionlessReal, Eigen::Dynamic> D_inv_BTy = B.transpose()*ys[indices[i]];
~~~
- **EN:** This range declares or defines callable logic such as rankUpdate, transpose. Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 此范围声明或定义了可调用逻辑，例如 rankUpdate, transpose。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

### Lines 379-392 / 第 379-392 行
~~~cpp
 379:       for (size_t j = 0; j < n; ++j) {
 380:         D_inv_BTy[j] /= D[j];
 381:       }
 382:       DimensionlessReal squared_norm = D_inv_BTy.squaredNorm();
 383:       DimensionlessReal K2 = c_mu*weights[i]/squared_norm;
 384:       C += K2*ys[indices[i]]*ys[indices[i]].transpose();
 385:     }
 386:   } while (generation++ < params.max_generations);
 387: 
 388:   return best_vector;
 389: }
 390: 
 391: } // namespace boost::math::optimization
 392: #endif
~~~
- **EN:** The code enters namespace scope (boost) to keep symbols organized. This range declares or defines callable logic such as squaredNorm, transpose, .... Loop constructs repeat work over benchmark iterations, table entries, or numeric refinement steps.
- **CN:** 代码进入命名空间作用域（boost），以保持符号组织清晰。 此范围声明或定义了可调用逻辑，例如 squaredNorm, transpose, ...。 循环结构会在基准迭代、表项处理或数值细化步骤中重复执行工作。

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
- **Included headers / 包含头文件**: `atomic, cmath, iostream, limits, random, sstream, stdexcept, utility, vector, boost/math/optimization/detail/common.hpp, boost/math/tools/assert.hpp, Eigen/Dense`
- **Namespaces / 命名空间**: `boost`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `decltype, validate_cma_es_parameters, detail::validate_bounds, detail::validate_initial_guess, size, floor, std::ceil, std::invalid_argument, ...`
