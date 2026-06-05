# jacobi_theta.hpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `third-party/boost-math/include/boost/math/special_functions/jacobi_theta.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/` (`llvm-project`)
- **Purpose / 目的**:
  - **EN**: This header jacobi theta functions Four main theta functions with various flavors of parameterization, floating-point policies, and bonus "minus 1" versions of functions 3 and 4 designed to preserve accuracy for small q..
  - **CN**: 声明 Boost.Math 的特殊函数、近似计算内核以及按类型泛化的数值入口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````cpp
   1: // Jacobi theta functions
   2: // Copyright Evan Miller 2020
   3: //
   4: // Use, modification and distribution are subject to the
   5: // Boost Software License, Version 1.0. (See accompanying file
   6: // LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   7: //
   8: // Four main theta functions with various flavors of parameterization,
   9: // floating-point policies, and bonus "minus 1" versions of functions 3 and 4
  10: // designed to preserve accuracy for small q. Twenty-four C++ functions are
  11: // provided in all.
  12: //
  13: // The functions take a real argument z and a parameter known as q, or its close
  14: // relative tau.
  15: //
  16: // The mathematical functions are best understood in terms of their Fourier
  17: // series. Using the q parameterization, and summing from n = 0 to INF:
  18: //
  19: // theta_1(z,q) = 2 SUM (-1)^n * q^(n+1/2)^2 * sin((2n+1)z)
  20: // theta_2(z,q) = 2 SUM q^(n+1/2)^2 * cos((2n+1)z)
````
- **L1 EN**: Comment documents nearby intent or usage notes: `Jacobi theta functions`.
  - **L1 CN**: 注释说明附近代码的意图或使用说明：`Jacobi theta functions`。
- **L2 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L2 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L3 EN**: Separator comment used for visual grouping.
  - **L3 CN**: 分隔注释，用于视觉分组。
- **L4 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L4 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L5 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L5 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L6 EN**: License or provenance comment documenting ownership and reuse terms.
  - **L6 CN**: 许可或来源注释，用于说明归属与复用条款。
- **L7 EN**: Separator comment used for visual grouping.
  - **L7 CN**: 分隔注释，用于视觉分组。
- **L8 EN**: Comment documents nearby intent or usage notes: `Four main theta functions with various flavors of parameterization,`.
  - **L8 CN**: 注释说明附近代码的意图或使用说明：`Four main theta functions with various flavors of parameterization,`。
- **L9 EN**: Comment documents nearby intent or usage notes: `floating-point policies, and bonus "minus 1" versions of functions 3 and 4`.
  - **L9 CN**: 注释说明附近代码的意图或使用说明：`floating-point policies, and bonus "minus 1" versions of functions 3 and 4`。
- **L10 EN**: Comment documents nearby intent or usage notes: `designed to preserve accuracy for small q. Twenty-four C++ functions are`.
  - **L10 CN**: 注释说明附近代码的意图或使用说明：`designed to preserve accuracy for small q. Twenty-four C++ functions are`。
- **L11 EN**: Comment documents nearby intent or usage notes: `provided in all.`.
  - **L11 CN**: 注释说明附近代码的意图或使用说明：`provided in all.`。
- **L12 EN**: Separator comment used for visual grouping.
  - **L12 CN**: 分隔注释，用于视觉分组。
- **L13 EN**: Comment documents nearby intent or usage notes: `The functions take a real argument z and a parameter known as q, or its close`.
  - **L13 CN**: 注释说明附近代码的意图或使用说明：`The functions take a real argument z and a parameter known as q, or its close`。
- **L14 EN**: Comment documents nearby intent or usage notes: `relative tau.`.
  - **L14 CN**: 注释说明附近代码的意图或使用说明：`relative tau.`。
- **L15 EN**: Separator comment used for visual grouping.
  - **L15 CN**: 分隔注释，用于视觉分组。
- **L16 EN**: Comment documents nearby intent or usage notes: `The mathematical functions are best understood in terms of their Fourier`.
  - **L16 CN**: 注释说明附近代码的意图或使用说明：`The mathematical functions are best understood in terms of their Fourier`。
- **L17 EN**: Comment documents nearby intent or usage notes: `series. Using the q parameterization, and summing from n = 0 to INF:`.
  - **L17 CN**: 注释说明附近代码的意图或使用说明：`series. Using the q parameterization, and summing from n = 0 to INF:`。
- **L18 EN**: Separator comment used for visual grouping.
  - **L18 CN**: 分隔注释，用于视觉分组。
- **L19 EN**: Comment documents nearby intent or usage notes: `theta_1(z,q) = 2 SUM (-1)^n * q^(n+1/2)^2 * sin((2n+1)z)`.
  - **L19 CN**: 注释说明附近代码的意图或使用说明：`theta_1(z,q) = 2 SUM (-1)^n * q^(n+1/2)^2 * sin((2n+1)z)`。
- **L20 EN**: Comment documents nearby intent or usage notes: `theta_2(z,q) = 2 SUM q^(n+1/2)^2 * cos((2n+1)z)`.
  - **L20 CN**: 注释说明附近代码的意图或使用说明：`theta_2(z,q) = 2 SUM q^(n+1/2)^2 * cos((2n+1)z)`。

### Lines 21-40 / 第 21-40 行

````cpp
  21: // theta_3(z,q) = 1 + 2 SUM q^n^2 * cos(2nz)
  22: // theta_4(z,q) = 1 + 2 SUM (-1)^n * q^n^2 * cos(2nz)
  23: //
  24: // Appropriately multiplied and divided, these four theta functions can be used
  25: // to implement the famous Jacabi elliptic functions - but this is not really
  26: // recommended, as the existing Boost implementations are likely faster and
  27: // more accurate.  More saliently, setting z = 0 on the fourth theta function
  28: // will produce the limiting CDF of the Kolmogorov-Smirnov distribution, which
  29: // is this particular implementation's raison d'etre.
  30: //
  31: // Separate C++ functions are provided for q and for tau. The main q functions are:
  32: //
  33: // template <class T> inline T jacobi_theta1(T z, T q);
  34: // template <class T> inline T jacobi_theta2(T z, T q);
  35: // template <class T> inline T jacobi_theta3(T z, T q);
  36: // template <class T> inline T jacobi_theta4(T z, T q);
  37: //
  38: // The parameter q, also known as the nome, is restricted to the domain (0, 1),
  39: // and will throw a domain error otherwise.
  40: //
````
- **L21 EN**: Comment documents nearby intent or usage notes: `theta_3(z,q) = 1 + 2 SUM q^n^2 * cos(2nz)`.
  - **L21 CN**: 注释说明附近代码的意图或使用说明：`theta_3(z,q) = 1 + 2 SUM q^n^2 * cos(2nz)`。
- **L22 EN**: Comment documents nearby intent or usage notes: `theta_4(z,q) = 1 + 2 SUM (-1)^n * q^n^2 * cos(2nz)`.
  - **L22 CN**: 注释说明附近代码的意图或使用说明：`theta_4(z,q) = 1 + 2 SUM (-1)^n * q^n^2 * cos(2nz)`。
- **L23 EN**: Separator comment used for visual grouping.
  - **L23 CN**: 分隔注释，用于视觉分组。
- **L24 EN**: Comment documents nearby intent or usage notes: `Appropriately multiplied and divided, these four theta functions can be used`.
  - **L24 CN**: 注释说明附近代码的意图或使用说明：`Appropriately multiplied and divided, these four theta functions can be used`。
- **L25 EN**: Comment documents nearby intent or usage notes: `to implement the famous Jacabi elliptic functions - but this is not really`.
  - **L25 CN**: 注释说明附近代码的意图或使用说明：`to implement the famous Jacabi elliptic functions - but this is not really`。
- **L26 EN**: Comment documents nearby intent or usage notes: `recommended, as the existing Boost implementations are likely faster and`.
  - **L26 CN**: 注释说明附近代码的意图或使用说明：`recommended, as the existing Boost implementations are likely faster and`。
- **L27 EN**: Comment documents nearby intent or usage notes: `more accurate.  More saliently, setting z = 0 on the fourth theta function`.
  - **L27 CN**: 注释说明附近代码的意图或使用说明：`more accurate.  More saliently, setting z = 0 on the fourth theta function`。
- **L28 EN**: Comment documents nearby intent or usage notes: `will produce the limiting CDF of the Kolmogorov-Smirnov distribution, which`.
  - **L28 CN**: 注释说明附近代码的意图或使用说明：`will produce the limiting CDF of the Kolmogorov-Smirnov distribution, which`。
- **L29 EN**: Comment documents nearby intent or usage notes: `is this particular implementation's raison d'etre.`.
  - **L29 CN**: 注释说明附近代码的意图或使用说明：`is this particular implementation's raison d'etre.`。
- **L30 EN**: Separator comment used for visual grouping.
  - **L30 CN**: 分隔注释，用于视觉分组。
- **L31 EN**: Comment documents nearby intent or usage notes: `Separate C++ functions are provided for q and for tau. The main q functions are:`.
  - **L31 CN**: 注释说明附近代码的意图或使用说明：`Separate C++ functions are provided for q and for tau. The main q functions are:`。
- **L32 EN**: Separator comment used for visual grouping.
  - **L32 CN**: 分隔注释，用于视觉分组。
- **L33 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta1(T z, T q);`.
  - **L33 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta1(T z, T q);`。
- **L34 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta2(T z, T q);`.
  - **L34 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta2(T z, T q);`。
- **L35 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta3(T z, T q);`.
  - **L35 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta3(T z, T q);`。
- **L36 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta4(T z, T q);`.
  - **L36 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta4(T z, T q);`。
- **L37 EN**: Separator comment used for visual grouping.
  - **L37 CN**: 分隔注释，用于视觉分组。
- **L38 EN**: Comment documents nearby intent or usage notes: `The parameter q, also known as the nome, is restricted to the domain (0, 1),`.
  - **L38 CN**: 注释说明附近代码的意图或使用说明：`The parameter q, also known as the nome, is restricted to the domain (0, 1),`。
- **L39 EN**: Comment documents nearby intent or usage notes: `and will throw a domain error otherwise.`.
  - **L39 CN**: 注释说明附近代码的意图或使用说明：`and will throw a domain error otherwise.`。
- **L40 EN**: Separator comment used for visual grouping.
  - **L40 CN**: 分隔注释，用于视觉分组。

### Lines 41-60 / 第 41-60 行

````cpp
  41: // The equivalent functions that use tau instead of q are:
  42: //
  43: // template <class T> inline T jacobi_theta1tau(T z, T tau);
  44: // template <class T> inline T jacobi_theta2tau(T z, T tau);
  45: // template <class T> inline T jacobi_theta3tau(T z, T tau);
  46: // template <class T> inline T jacobi_theta4tau(T z, T tau);
  47: //
  48: // Mathematically, q and tau are related by:
  49: //
  50: // q = exp(i PI*Tau)
  51: //
  52: // However, the tau in the equation above is *not* identical to the tau in the function
  53: // signature. Instead, `tau` is the imaginary component of tau. Mathematically, tau can
  54: // be complex - but practically, most applications call for a purely imaginary tau.
  55: // Rather than provide a full complex-number API, the author decided to treat the
  56: // parameter `tau` as an imaginary number. So in computational terms, the
  57: // relationship between `q` and `tau` is given by:
  58: //
  59: // q = exp(-constants::pi<T>() * tau)
  60: //
````
- **L41 EN**: Comment documents nearby intent or usage notes: `The equivalent functions that use tau instead of q are:`.
  - **L41 CN**: 注释说明附近代码的意图或使用说明：`The equivalent functions that use tau instead of q are:`。
- **L42 EN**: Separator comment used for visual grouping.
  - **L42 CN**: 分隔注释，用于视觉分组。
- **L43 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta1tau(T z, T tau);`.
  - **L43 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta1tau(T z, T tau);`。
- **L44 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta2tau(T z, T tau);`.
  - **L44 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta2tau(T z, T tau);`。
- **L45 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta3tau(T z, T tau);`.
  - **L45 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta3tau(T z, T tau);`。
- **L46 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta4tau(T z, T tau);`.
  - **L46 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta4tau(T z, T tau);`。
- **L47 EN**: Separator comment used for visual grouping.
  - **L47 CN**: 分隔注释，用于视觉分组。
- **L48 EN**: Comment documents nearby intent or usage notes: `Mathematically, q and tau are related by:`.
  - **L48 CN**: 注释说明附近代码的意图或使用说明：`Mathematically, q and tau are related by:`。
- **L49 EN**: Separator comment used for visual grouping.
  - **L49 CN**: 分隔注释，用于视觉分组。
- **L50 EN**: Comment documents nearby intent or usage notes: `q = exp(i PI*Tau)`.
  - **L50 CN**: 注释说明附近代码的意图或使用说明：`q = exp(i PI*Tau)`。
- **L51 EN**: Separator comment used for visual grouping.
  - **L51 CN**: 分隔注释，用于视觉分组。
- **L52 EN**: Comment documents nearby intent or usage notes: `However, the tau in the equation above is *not* identical to the tau in the function`.
  - **L52 CN**: 注释说明附近代码的意图或使用说明：`However, the tau in the equation above is *not* identical to the tau in the function`。
- **L53 EN**: Comment documents nearby intent or usage notes: `signature. Instead, `tau` is the imaginary component of tau. Mathematically, tau can`.
  - **L53 CN**: 注释说明附近代码的意图或使用说明：`signature. Instead, `tau` is the imaginary component of tau. Mathematically, tau can`。
- **L54 EN**: Comment documents nearby intent or usage notes: `be complex - but practically, most applications call for a purely imaginary tau.`.
  - **L54 CN**: 注释说明附近代码的意图或使用说明：`be complex - but practically, most applications call for a purely imaginary tau.`。
- **L55 EN**: Comment documents nearby intent or usage notes: `Rather than provide a full complex-number API, the author decided to treat the`.
  - **L55 CN**: 注释说明附近代码的意图或使用说明：`Rather than provide a full complex-number API, the author decided to treat the`。
- **L56 EN**: Comment documents nearby intent or usage notes: `parameter `tau` as an imaginary number. So in computational terms, the`.
  - **L56 CN**: 注释说明附近代码的意图或使用说明：`parameter `tau` as an imaginary number. So in computational terms, the`。
- **L57 EN**: Comment documents nearby intent or usage notes: `relationship between `q` and `tau` is given by:`.
  - **L57 CN**: 注释说明附近代码的意图或使用说明：`relationship between `q` and `tau` is given by:`。
- **L58 EN**: Separator comment used for visual grouping.
  - **L58 CN**: 分隔注释，用于视觉分组。
- **L59 EN**: Comment documents nearby intent or usage notes: `q = exp(-constants::pi<T>() * tau)`.
  - **L59 CN**: 注释说明附近代码的意图或使用说明：`q = exp(-constants::pi<T>() * tau)`。
- **L60 EN**: Separator comment used for visual grouping.
  - **L60 CN**: 分隔注释，用于视觉分组。

### Lines 61-80 / 第 61-80 行

````cpp
  61: // The tau versions are provided for the sake of accuracy, as well as conformance
  62: // with common notation. If your q is an exponential, you are better off using
  63: // the tau versions, e.g.
  64: //
  65: // jacobi_theta1(z, exp(-a)); // rather poor accuracy
  66: // jacobi_theta1tau(z, a / constants::pi<T>()); // better accuracy
  67: //
  68: // Similarly, if you have a precise (small positive) value for the complement
  69: // of q, you can obtain a more precise answer overall by passing the result of
  70: // `log1p` to the tau parameter:
  71: //
  72: // jacobi_theta1(z, 1-q_complement); // precision lost in subtraction
  73: // jacobi_theta1tau(z, -log1p(-q_complement) / constants::pi<T>()); // better!
  74: //
  75: // A third quartet of functions are provided for improving accuracy in cases
  76: // where q is small, specifically |q| < exp(-PI) = 0.04 (or, equivalently, tau
  77: // greater than unity). In this domain of q values, the third and fourth theta
  78: // functions always return values close to 1. So the following "m1" functions
  79: // are provided, similar in spirit to `expm1`, which return one less than their
  80: // regular counterparts:
````
- **L61 EN**: Comment documents nearby intent or usage notes: `The tau versions are provided for the sake of accuracy, as well as conformance`.
  - **L61 CN**: 注释说明附近代码的意图或使用说明：`The tau versions are provided for the sake of accuracy, as well as conformance`。
- **L62 EN**: Comment documents nearby intent or usage notes: `with common notation. If your q is an exponential, you are better off using`.
  - **L62 CN**: 注释说明附近代码的意图或使用说明：`with common notation. If your q is an exponential, you are better off using`。
- **L63 EN**: Comment documents nearby intent or usage notes: `the tau versions, e.g.`.
  - **L63 CN**: 注释说明附近代码的意图或使用说明：`the tau versions, e.g.`。
- **L64 EN**: Separator comment used for visual grouping.
  - **L64 CN**: 分隔注释，用于视觉分组。
- **L65 EN**: Comment documents nearby intent or usage notes: `jacobi_theta1(z, exp(-a)); // rather poor accuracy`.
  - **L65 CN**: 注释说明附近代码的意图或使用说明：`jacobi_theta1(z, exp(-a)); // rather poor accuracy`。
- **L66 EN**: Comment documents nearby intent or usage notes: `jacobi_theta1tau(z, a / constants::pi<T>()); // better accuracy`.
  - **L66 CN**: 注释说明附近代码的意图或使用说明：`jacobi_theta1tau(z, a / constants::pi<T>()); // better accuracy`。
- **L67 EN**: Separator comment used for visual grouping.
  - **L67 CN**: 分隔注释，用于视觉分组。
- **L68 EN**: Comment documents nearby intent or usage notes: `Similarly, if you have a precise (small positive) value for the complement`.
  - **L68 CN**: 注释说明附近代码的意图或使用说明：`Similarly, if you have a precise (small positive) value for the complement`。
- **L69 EN**: Comment documents nearby intent or usage notes: `of q, you can obtain a more precise answer overall by passing the result of`.
  - **L69 CN**: 注释说明附近代码的意图或使用说明：`of q, you can obtain a more precise answer overall by passing the result of`。
- **L70 EN**: Comment documents nearby intent or usage notes: ``log1p` to the tau parameter:`.
  - **L70 CN**: 注释说明附近代码的意图或使用说明：``log1p` to the tau parameter:`。
- **L71 EN**: Separator comment used for visual grouping.
  - **L71 CN**: 分隔注释，用于视觉分组。
- **L72 EN**: Comment documents nearby intent or usage notes: `jacobi_theta1(z, 1-q_complement); // precision lost in subtraction`.
  - **L72 CN**: 注释说明附近代码的意图或使用说明：`jacobi_theta1(z, 1-q_complement); // precision lost in subtraction`。
- **L73 EN**: Comment documents nearby intent or usage notes: `jacobi_theta1tau(z, -log1p(-q_complement) / constants::pi<T>()); // better!`.
  - **L73 CN**: 注释说明附近代码的意图或使用说明：`jacobi_theta1tau(z, -log1p(-q_complement) / constants::pi<T>()); // better!`。
- **L74 EN**: Separator comment used for visual grouping.
  - **L74 CN**: 分隔注释，用于视觉分组。
- **L75 EN**: Comment documents nearby intent or usage notes: `A third quartet of functions are provided for improving accuracy in cases`.
  - **L75 CN**: 注释说明附近代码的意图或使用说明：`A third quartet of functions are provided for improving accuracy in cases`。
- **L76 EN**: Comment documents nearby intent or usage notes: `where q is small, specifically |q| < exp(-PI) = 0.04 (or, equivalently, tau`.
  - **L76 CN**: 注释说明附近代码的意图或使用说明：`where q is small, specifically |q| < exp(-PI) = 0.04 (or, equivalently, tau`。
- **L77 EN**: Comment documents nearby intent or usage notes: `greater than unity). In this domain of q values, the third and fourth theta`.
  - **L77 CN**: 注释说明附近代码的意图或使用说明：`greater than unity). In this domain of q values, the third and fourth theta`。
- **L78 EN**: Comment documents nearby intent or usage notes: `functions always return values close to 1. So the following "m1" functions`.
  - **L78 CN**: 注释说明附近代码的意图或使用说明：`functions always return values close to 1. So the following "m1" functions`。
- **L79 EN**: Comment documents nearby intent or usage notes: `are provided, similar in spirit to `expm1`, which return one less than their`.
  - **L79 CN**: 注释说明附近代码的意图或使用说明：`are provided, similar in spirit to `expm1`, which return one less than their`。
- **L80 EN**: Comment documents nearby intent or usage notes: `regular counterparts:`.
  - **L80 CN**: 注释说明附近代码的意图或使用说明：`regular counterparts:`。

### Lines 81-100 / 第 81-100 行

````cpp
  81: //
  82: // template <class T> inline T jacobi_theta3m1(T z, T q);
  83: // template <class T> inline T jacobi_theta4m1(T z, T q);
  84: // template <class T> inline T jacobi_theta3m1tau(T z, T tau);
  85: // template <class T> inline T jacobi_theta4m1tau(T z, T tau);
  86: //
  87: // Note that "m1" versions of the first and second theta would not be useful,
  88: // as their ranges are not confined to a neighborhood around 1 (see the Fourier
  89: // transform representations above).
  90: //
  91: // Finally, the twelve functions above are each available with a third Policy
  92: // argument, which can be used to define a custom epsilon value. These Policy
  93: // versions bring the total number of functions provided by jacobi_theta.hpp
  94: // to twenty-four.
  95: //
  96: // See:
  97: // https://mathworld.wolfram.com/JacobiThetaFunctions.html
  98: // https://dlmf.nist.gov/20
  99: 
 100: #ifndef BOOST_MATH_JACOBI_THETA_HPP
````
- **L81 EN**: Separator comment used for visual grouping.
  - **L81 CN**: 分隔注释，用于视觉分组。
- **L82 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta3m1(T z, T q);`.
  - **L82 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta3m1(T z, T q);`。
- **L83 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta4m1(T z, T q);`.
  - **L83 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta4m1(T z, T q);`。
- **L84 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta3m1tau(T z, T tau);`.
  - **L84 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta3m1tau(T z, T tau);`。
- **L85 EN**: Comment documents nearby intent or usage notes: `template <class T> inline T jacobi_theta4m1tau(T z, T tau);`.
  - **L85 CN**: 注释说明附近代码的意图或使用说明：`template <class T> inline T jacobi_theta4m1tau(T z, T tau);`。
- **L86 EN**: Separator comment used for visual grouping.
  - **L86 CN**: 分隔注释，用于视觉分组。
- **L87 EN**: Comment documents nearby intent or usage notes: `Note that "m1" versions of the first and second theta would not be useful,`.
  - **L87 CN**: 注释说明附近代码的意图或使用说明：`Note that "m1" versions of the first and second theta would not be useful,`。
- **L88 EN**: Comment documents nearby intent or usage notes: `as their ranges are not confined to a neighborhood around 1 (see the Fourier`.
  - **L88 CN**: 注释说明附近代码的意图或使用说明：`as their ranges are not confined to a neighborhood around 1 (see the Fourier`。
- **L89 EN**: Comment documents nearby intent or usage notes: `transform representations above).`.
  - **L89 CN**: 注释说明附近代码的意图或使用说明：`transform representations above).`。
- **L90 EN**: Separator comment used for visual grouping.
  - **L90 CN**: 分隔注释，用于视觉分组。
- **L91 EN**: Comment documents nearby intent or usage notes: `Finally, the twelve functions above are each available with a third Policy`.
  - **L91 CN**: 注释说明附近代码的意图或使用说明：`Finally, the twelve functions above are each available with a third Policy`。
- **L92 EN**: Comment documents nearby intent or usage notes: `argument, which can be used to define a custom epsilon value. These Policy`.
  - **L92 CN**: 注释说明附近代码的意图或使用说明：`argument, which can be used to define a custom epsilon value. These Policy`。
- **L93 EN**: Comment documents nearby intent or usage notes: `versions bring the total number of functions provided by jacobi_theta.hpp`.
  - **L93 CN**: 注释说明附近代码的意图或使用说明：`versions bring the total number of functions provided by jacobi_theta.hpp`。
- **L94 EN**: Comment documents nearby intent or usage notes: `to twenty-four.`.
  - **L94 CN**: 注释说明附近代码的意图或使用说明：`to twenty-four.`。
- **L95 EN**: Separator comment used for visual grouping.
  - **L95 CN**: 分隔注释，用于视觉分组。
- **L96 EN**: Comment documents nearby intent or usage notes: `See:`.
  - **L96 CN**: 注释说明附近代码的意图或使用说明：`See:`。
- **L97 EN**: Comment documents nearby intent or usage notes: `https://mathworld.wolfram.com/JacobiThetaFunctions.html`.
  - **L97 CN**: 注释说明附近代码的意图或使用说明：`https://mathworld.wolfram.com/JacobiThetaFunctions.html`。
- **L98 EN**: Comment documents nearby intent or usage notes: `https://dlmf.nist.gov/20`.
  - **L98 CN**: 注释说明附近代码的意图或使用说明：`https://dlmf.nist.gov/20`。
- **L99 EN**: Blank line separating nearby declarations or logic.
  - **L99 CN**: 空行，用于分隔相邻声明或逻辑。
- **L100 EN**: Starts a header guard condition: `#ifndef BOOST_MATH_JACOBI_THETA_HPP`.
  - **L100 CN**: 开始头文件保护条件：`#ifndef BOOST_MATH_JACOBI_THETA_HPP`。

### Lines 101-120 / 第 101-120 行

````cpp
 101: #define BOOST_MATH_JACOBI_THETA_HPP
 102: 
 103: #include <boost/math/tools/complex.hpp>
 104: #include <boost/math/tools/precision.hpp>
 105: #include <boost/math/tools/promotion.hpp>
 106: #include <boost/math/policies/error_handling.hpp>
 107: #include <boost/math/constants/constants.hpp>
 108: 
 109: namespace boost{ namespace math{
 110: 
 111: // Simple functions - parameterized by q
 112: template <class T, class U>
 113: inline typename tools::promote_args<T, U>::type jacobi_theta1(T z, U q);
 114: template <class T, class U>
 115: inline typename tools::promote_args<T, U>::type jacobi_theta2(T z, U q);
 116: template <class T, class U>
 117: inline typename tools::promote_args<T, U>::type jacobi_theta3(T z, U q);
 118: template <class T, class U>
 119: inline typename tools::promote_args<T, U>::type jacobi_theta4(T z, U q);
 120: 
````
- **L101 EN**: Defines macro `BOOST_MATH_JACOBI_THETA_HPP` for compile-time control, shorthand, or generated boilerplate.
  - **L101 CN**: 定义宏 `BOOST_MATH_JACOBI_THETA_HPP`，用于编译期控制、简写或生成样板代码。
- **L102 EN**: Blank line separating nearby declarations or logic.
  - **L102 CN**: 空行，用于分隔相邻声明或逻辑。
- **L103 EN**: Includes <boost/math/tools/complex.hpp> to access Boost.Math numeric tool helpers.
  - **L103 CN**: 引入 <boost/math/tools/complex.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L104 EN**: Includes <boost/math/tools/precision.hpp> to access Boost.Math numeric tool helpers.
  - **L104 CN**: 引入 <boost/math/tools/precision.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L105 EN**: Includes <boost/math/tools/promotion.hpp> to access Boost.Math numeric tool helpers.
  - **L105 CN**: 引入 <boost/math/tools/promotion.hpp> 以使用Boost.Math 数值工具辅助逻辑。
- **L106 EN**: Includes <boost/math/policies/error_handling.hpp> to access Boost.Math policy configuration.
  - **L106 CN**: 引入 <boost/math/policies/error_handling.hpp> 以使用Boost.Math 策略配置。
- **L107 EN**: Includes <boost/math/constants/constants.hpp> to access Boost.Math numeric constants.
  - **L107 CN**: 引入 <boost/math/constants/constants.hpp> 以使用Boost.Math 数值常量。
- **L108 EN**: Blank line separating nearby declarations or logic.
  - **L108 CN**: 空行，用于分隔相邻声明或逻辑。
- **L109 EN**: Opens namespace scope `boost{ namespace math`.
  - **L109 CN**: 打开命名空间作用域 `boost{ namespace math`。
- **L110 EN**: Blank line separating nearby declarations or logic.
  - **L110 CN**: 空行，用于分隔相邻声明或逻辑。
- **L111 EN**: Comment documents nearby intent or usage notes: `Simple functions - parameterized by q`.
  - **L111 CN**: 注释说明附近代码的意图或使用说明：`Simple functions - parameterized by q`。
- **L112 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L112 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L113 EN**: Executes a call or declaration centered on `jacobi_theta1`.
  - **L113 CN**: 执行以 `jacobi_theta1` 为核心的调用或声明。
- **L114 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L114 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L115 EN**: Executes a call or declaration centered on `jacobi_theta2`.
  - **L115 CN**: 执行以 `jacobi_theta2` 为核心的调用或声明。
- **L116 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L116 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L117 EN**: Executes a call or declaration centered on `jacobi_theta3`.
  - **L117 CN**: 执行以 `jacobi_theta3` 为核心的调用或声明。
- **L118 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L118 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L119 EN**: Executes a call or declaration centered on `jacobi_theta4`.
  - **L119 CN**: 执行以 `jacobi_theta4` 为核心的调用或声明。
- **L120 EN**: Blank line separating nearby declarations or logic.
  - **L120 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 121-140 / 第 121-140 行

````cpp
 121: // Simple functions - parameterized by tau (assumed imaginary)
 122: // q = exp(i*PI*TAU)
 123: // tau = -log(q)/PI
 124: template <class T, class U>
 125: inline typename tools::promote_args<T, U>::type jacobi_theta1tau(T z, U tau);
 126: template <class T, class U>
 127: inline typename tools::promote_args<T, U>::type jacobi_theta2tau(T z, U tau);
 128: template <class T, class U>
 129: inline typename tools::promote_args<T, U>::type jacobi_theta3tau(T z, U tau);
 130: template <class T, class U>
 131: inline typename tools::promote_args<T, U>::type jacobi_theta4tau(T z, U tau);
 132: 
 133: // Minus one versions for small q / large tau
 134: template <class T, class U>
 135: inline typename tools::promote_args<T, U>::type jacobi_theta3m1(T z, U q);
 136: template <class T, class U>
 137: inline typename tools::promote_args<T, U>::type jacobi_theta4m1(T z, U q);
 138: template <class T, class U>
 139: inline typename tools::promote_args<T, U>::type jacobi_theta3m1tau(T z, U tau);
 140: template <class T, class U>
````
- **L121 EN**: Comment documents nearby intent or usage notes: `Simple functions - parameterized by tau (assumed imaginary)`.
  - **L121 CN**: 注释说明附近代码的意图或使用说明：`Simple functions - parameterized by tau (assumed imaginary)`。
- **L122 EN**: Comment documents nearby intent or usage notes: `q = exp(i*PI*TAU)`.
  - **L122 CN**: 注释说明附近代码的意图或使用说明：`q = exp(i*PI*TAU)`。
- **L123 EN**: Comment documents nearby intent or usage notes: `tau = -log(q)/PI`.
  - **L123 CN**: 注释说明附近代码的意图或使用说明：`tau = -log(q)/PI`。
- **L124 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L124 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L125 EN**: Executes a call or declaration centered on `jacobi_theta1tau`.
  - **L125 CN**: 执行以 `jacobi_theta1tau` 为核心的调用或声明。
- **L126 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L126 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L127 EN**: Executes a call or declaration centered on `jacobi_theta2tau`.
  - **L127 CN**: 执行以 `jacobi_theta2tau` 为核心的调用或声明。
- **L128 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L128 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L129 EN**: Executes a call or declaration centered on `jacobi_theta3tau`.
  - **L129 CN**: 执行以 `jacobi_theta3tau` 为核心的调用或声明。
- **L130 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L130 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L131 EN**: Executes a call or declaration centered on `jacobi_theta4tau`.
  - **L131 CN**: 执行以 `jacobi_theta4tau` 为核心的调用或声明。
- **L132 EN**: Blank line separating nearby declarations or logic.
  - **L132 CN**: 空行，用于分隔相邻声明或逻辑。
- **L133 EN**: Comment documents nearby intent or usage notes: `Minus one versions for small q / large tau`.
  - **L133 CN**: 注释说明附近代码的意图或使用说明：`Minus one versions for small q / large tau`。
- **L134 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L134 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L135 EN**: Executes a call or declaration centered on `jacobi_theta3m1`.
  - **L135 CN**: 执行以 `jacobi_theta3m1` 为核心的调用或声明。
- **L136 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L136 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L137 EN**: Executes a call or declaration centered on `jacobi_theta4m1`.
  - **L137 CN**: 执行以 `jacobi_theta4m1` 为核心的调用或声明。
- **L138 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L138 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L139 EN**: Executes a call or declaration centered on `jacobi_theta3m1tau`.
  - **L139 CN**: 执行以 `jacobi_theta3m1tau` 为核心的调用或声明。
- **L140 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L140 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。

### Lines 141-160 / 第 141-160 行

````cpp
 141: inline typename tools::promote_args<T, U>::type jacobi_theta4m1tau(T z, U tau);
 142: 
 143: // Policied versions - parameterized by q
 144: template <class T, class U, class Policy>
 145: inline typename tools::promote_args<T, U>::type jacobi_theta1(T z, U q, const Policy& pol);
 146: template <class T, class U, class Policy>
 147: inline typename tools::promote_args<T, U>::type jacobi_theta2(T z, U q, const Policy& pol);
 148: template <class T, class U, class Policy>
 149: inline typename tools::promote_args<T, U>::type jacobi_theta3(T z, U q, const Policy& pol);
 150: template <class T, class U, class Policy>
 151: inline typename tools::promote_args<T, U>::type jacobi_theta4(T z, U q, const Policy& pol);
 152: 
 153: // Policied versions - parameterized by tau
 154: template <class T, class U, class Policy>
 155: inline typename tools::promote_args<T, U>::type jacobi_theta1tau(T z, U tau, const Policy& pol);
 156: template <class T, class U, class Policy>
 157: inline typename tools::promote_args<T, U>::type jacobi_theta2tau(T z, U tau, const Policy& pol);
 158: template <class T, class U, class Policy>
 159: inline typename tools::promote_args<T, U>::type jacobi_theta3tau(T z, U tau, const Policy& pol);
 160: template <class T, class U, class Policy>
````
- **L141 EN**: Executes a call or declaration centered on `jacobi_theta4m1tau`.
  - **L141 CN**: 执行以 `jacobi_theta4m1tau` 为核心的调用或声明。
- **L142 EN**: Blank line separating nearby declarations or logic.
  - **L142 CN**: 空行，用于分隔相邻声明或逻辑。
- **L143 EN**: Comment documents nearby intent or usage notes: `Policied versions - parameterized by q`.
  - **L143 CN**: 注释说明附近代码的意图或使用说明：`Policied versions - parameterized by q`。
- **L144 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L144 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L145 EN**: Executes a call or declaration centered on `jacobi_theta1`.
  - **L145 CN**: 执行以 `jacobi_theta1` 为核心的调用或声明。
- **L146 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L146 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L147 EN**: Executes a call or declaration centered on `jacobi_theta2`.
  - **L147 CN**: 执行以 `jacobi_theta2` 为核心的调用或声明。
- **L148 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L148 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L149 EN**: Executes a call or declaration centered on `jacobi_theta3`.
  - **L149 CN**: 执行以 `jacobi_theta3` 为核心的调用或声明。
- **L150 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L150 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L151 EN**: Executes a call or declaration centered on `jacobi_theta4`.
  - **L151 CN**: 执行以 `jacobi_theta4` 为核心的调用或声明。
- **L152 EN**: Blank line separating nearby declarations or logic.
  - **L152 CN**: 空行，用于分隔相邻声明或逻辑。
- **L153 EN**: Comment documents nearby intent or usage notes: `Policied versions - parameterized by tau`.
  - **L153 CN**: 注释说明附近代码的意图或使用说明：`Policied versions - parameterized by tau`。
- **L154 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L154 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L155 EN**: Executes a call or declaration centered on `jacobi_theta1tau`.
  - **L155 CN**: 执行以 `jacobi_theta1tau` 为核心的调用或声明。
- **L156 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L156 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L157 EN**: Executes a call or declaration centered on `jacobi_theta2tau`.
  - **L157 CN**: 执行以 `jacobi_theta2tau` 为核心的调用或声明。
- **L158 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L158 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L159 EN**: Executes a call or declaration centered on `jacobi_theta3tau`.
  - **L159 CN**: 执行以 `jacobi_theta3tau` 为核心的调用或声明。
- **L160 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L160 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。

### Lines 161-180 / 第 161-180 行

````cpp
 161: inline typename tools::promote_args<T, U>::type jacobi_theta4tau(T z, U tau, const Policy& pol);
 162: 
 163: // Policied m1 functions
 164: template <class T, class U, class Policy>
 165: inline typename tools::promote_args<T, U>::type jacobi_theta3m1(T z, U q, const Policy& pol);
 166: template <class T, class U, class Policy>
 167: inline typename tools::promote_args<T, U>::type jacobi_theta4m1(T z, U q, const Policy& pol);
 168: template <class T, class U, class Policy>
 169: inline typename tools::promote_args<T, U>::type jacobi_theta3m1tau(T z, U tau, const Policy& pol);
 170: template <class T, class U, class Policy>
 171: inline typename tools::promote_args<T, U>::type jacobi_theta4m1tau(T z, U tau, const Policy& pol);
 172: 
 173: // Compare the non-oscillating component of the delta to the previous delta.
 174: // Both are assumed to be non-negative.
 175: template <class RealType>
 176: inline bool
 177: _jacobi_theta_converged(RealType last_delta, RealType delta, RealType eps) {
 178:     return delta == 0.0 || delta < eps*last_delta;
 179: }
 180: 
````
- **L161 EN**: Executes a call or declaration centered on `jacobi_theta4tau`.
  - **L161 CN**: 执行以 `jacobi_theta4tau` 为核心的调用或声明。
- **L162 EN**: Blank line separating nearby declarations or logic.
  - **L162 CN**: 空行，用于分隔相邻声明或逻辑。
- **L163 EN**: Comment documents nearby intent or usage notes: `Policied m1 functions`.
  - **L163 CN**: 注释说明附近代码的意图或使用说明：`Policied m1 functions`。
- **L164 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L164 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L165 EN**: Executes a call or declaration centered on `jacobi_theta3m1`.
  - **L165 CN**: 执行以 `jacobi_theta3m1` 为核心的调用或声明。
- **L166 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L166 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L167 EN**: Executes a call or declaration centered on `jacobi_theta4m1`.
  - **L167 CN**: 执行以 `jacobi_theta4m1` 为核心的调用或声明。
- **L168 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L168 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L169 EN**: Executes a call or declaration centered on `jacobi_theta3m1tau`.
  - **L169 CN**: 执行以 `jacobi_theta3m1tau` 为核心的调用或声明。
- **L170 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L170 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L171 EN**: Executes a call or declaration centered on `jacobi_theta4m1tau`.
  - **L171 CN**: 执行以 `jacobi_theta4m1tau` 为核心的调用或声明。
- **L172 EN**: Blank line separating nearby declarations or logic.
  - **L172 CN**: 空行，用于分隔相邻声明或逻辑。
- **L173 EN**: Comment documents nearby intent or usage notes: `Compare the non-oscillating component of the delta to the previous delta.`.
  - **L173 CN**: 注释说明附近代码的意图或使用说明：`Compare the non-oscillating component of the delta to the previous delta.`。
- **L174 EN**: Comment documents nearby intent or usage notes: `Both are assumed to be non-negative.`.
  - **L174 CN**: 注释说明附近代码的意图或使用说明：`Both are assumed to be non-negative.`。
- **L175 EN**: Introduces template parameters or specialization context: `template <class RealType>`.
  - **L175 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType>`。
- **L176 EN**: Continues the surrounding expression or declaration: `inline bool`.
  - **L176 CN**: 继续构造周围的表达式或声明：`inline bool`。
- **L177 EN**: Starts a function, method, lambda, or structured scope: `_jacobi_theta_converged(RealType last_delta, RealType delta, RealType eps) {`.
  - **L177 CN**: 开始一个函数、方法、lambda 或结构化作用域：`_jacobi_theta_converged(RealType last_delta, RealType delta, RealType eps) {`。
- **L178 EN**: Returns from the current function with `delta == 0.0 || delta < eps*last_delta`.
  - **L178 CN**: 以 `delta == 0.0 || delta < eps*last_delta` 从当前函数返回。
- **L179 EN**: Closes the current lexical scope or compound statement.
  - **L179 CN**: 结束当前词法作用域或复合语句块。
- **L180 EN**: Blank line separating nearby declarations or logic.
  - **L180 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 181-200 / 第 181-200 行

````cpp
 181: template <class RealType>
 182: inline RealType
 183: _jacobi_theta_sum(RealType tau, RealType z_n, RealType z_increment, RealType eps) {
 184:     BOOST_MATH_STD_USING
 185:     RealType delta = 0, partial_result = 0;
 186:     RealType last_delta = 0;
 187: 
 188:     do {
 189:         last_delta = delta;
 190:         delta = exp(-tau*z_n*z_n/constants::pi<RealType>());
 191:         partial_result += delta;
 192:         z_n += z_increment;
 193:     } while (!_jacobi_theta_converged(last_delta, delta, eps));
 194: 
 195:     return partial_result;
 196: }
 197: 
 198: // The following _IMAGINARY theta functions assume imaginary z and are for
 199: // internal use only. They are designed to increase accuracy and reduce the
 200: // number of iterations required for convergence for large |q|. The z argument
````
- **L181 EN**: Introduces template parameters or specialization context: `template <class RealType>`.
  - **L181 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType>`。
- **L182 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L182 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L183 EN**: Starts a function, method, lambda, or structured scope: `_jacobi_theta_sum(RealType tau, RealType z_n, RealType z_increment, RealType eps) {`.
  - **L183 CN**: 开始一个函数、方法、lambda 或结构化作用域：`_jacobi_theta_sum(RealType tau, RealType z_n, RealType z_increment, RealType eps) {`。
- **L184 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L184 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L185 EN**: Initializes variable `delta` from the right-hand expression.
  - **L185 CN**: 使用右侧表达式初始化变量 `delta`。
- **L186 EN**: Initializes variable `last_delta` from the right-hand expression.
  - **L186 CN**: 使用右侧表达式初始化变量 `last_delta`。
- **L187 EN**: Blank line separating nearby declarations or logic.
  - **L187 CN**: 空行，用于分隔相邻声明或逻辑。
- **L188 EN**: Continues the surrounding expression or declaration: `do {`.
  - **L188 CN**: 继续构造周围的表达式或声明：`do {`。
- **L189 EN**: Executes a standalone statement or declaration: `last_delta = delta;`.
  - **L189 CN**: 执行一条独立语句或声明：`last_delta = delta;`。
- **L190 EN**: Executes a call or declaration centered on `exp`.
  - **L190 CN**: 执行以 `exp` 为核心的调用或声明。
- **L191 EN**: Executes a standalone statement or declaration: `partial_result += delta;`.
  - **L191 CN**: 执行一条独立语句或声明：`partial_result += delta;`。
- **L192 EN**: Executes a standalone statement or declaration: `z_n += z_increment;`.
  - **L192 CN**: 执行一条独立语句或声明：`z_n += z_increment;`。
- **L193 EN**: Executes a call or declaration centered on `while`.
  - **L193 CN**: 执行以 `while` 为核心的调用或声明。
- **L194 EN**: Blank line separating nearby declarations or logic.
  - **L194 CN**: 空行，用于分隔相邻声明或逻辑。
- **L195 EN**: Returns from the current function with `partial_result`.
  - **L195 CN**: 以 `partial_result` 从当前函数返回。
- **L196 EN**: Closes the current lexical scope or compound statement.
  - **L196 CN**: 结束当前词法作用域或复合语句块。
- **L197 EN**: Blank line separating nearby declarations or logic.
  - **L197 CN**: 空行，用于分隔相邻声明或逻辑。
- **L198 EN**: Comment documents nearby intent or usage notes: `The following _IMAGINARY theta functions assume imaginary z and are for`.
  - **L198 CN**: 注释说明附近代码的意图或使用说明：`The following _IMAGINARY theta functions assume imaginary z and are for`。
- **L199 EN**: Comment documents nearby intent or usage notes: `internal use only. They are designed to increase accuracy and reduce the`.
  - **L199 CN**: 注释说明附近代码的意图或使用说明：`internal use only. They are designed to increase accuracy and reduce the`。
- **L200 EN**: Comment documents nearby intent or usage notes: `number of iterations required for convergence for large |q|. The z argument`.
  - **L200 CN**: 注释说明附近代码的意图或使用说明：`number of iterations required for convergence for large |q|. The z argument`。

### Lines 201-220 / 第 201-220 行

````cpp
 201: // is scaled by tau, and the summations are rewritten to be double-sided
 202: // following DLMF 20.13.4 and 20.13.5. The return values are scaled by
 203: // exp(-tau*z^2/Pi)/sqrt(tau).
 204: //
 205: // These functions are triggered when tau < 1, i.e. |q| > exp(-Pi) = 0.043
 206: //
 207: // Note that jacobi_theta4 uses the imaginary version of jacobi_theta2 (and
 208: // vice-versa). jacobi_theta1 and jacobi_theta3 use the imaginary versions of
 209: // themselves, following DLMF 20.7.30 - 20.7.33.
 210: template <class RealType, class Policy>
 211: inline RealType
 212: _IMAGINARY_jacobi_theta1tau(RealType z, RealType tau, const Policy&) {
 213:     BOOST_MATH_STD_USING
 214:     RealType eps = policies::get_epsilon<RealType, Policy>();
 215:     RealType result = RealType(0);
 216: 
 217:     // n>=0 even
 218:     result -= _jacobi_theta_sum(tau, RealType(z + constants::half_pi<RealType>()), constants::two_pi<RealType>(), eps);
 219:     // n>0 odd
 220:     result += _jacobi_theta_sum(tau, RealType(z + constants::half_pi<RealType>() + constants::pi<RealType>()), constants::two_pi<RealType>(), eps);
````
- **L201 EN**: Comment documents nearby intent or usage notes: `is scaled by tau, and the summations are rewritten to be double-sided`.
  - **L201 CN**: 注释说明附近代码的意图或使用说明：`is scaled by tau, and the summations are rewritten to be double-sided`。
- **L202 EN**: Comment documents nearby intent or usage notes: `following DLMF 20.13.4 and 20.13.5. The return values are scaled by`.
  - **L202 CN**: 注释说明附近代码的意图或使用说明：`following DLMF 20.13.4 and 20.13.5. The return values are scaled by`。
- **L203 EN**: Comment documents nearby intent or usage notes: `exp(-tau*z^2/Pi)/sqrt(tau).`.
  - **L203 CN**: 注释说明附近代码的意图或使用说明：`exp(-tau*z^2/Pi)/sqrt(tau).`。
- **L204 EN**: Separator comment used for visual grouping.
  - **L204 CN**: 分隔注释，用于视觉分组。
- **L205 EN**: Comment documents nearby intent or usage notes: `These functions are triggered when tau < 1, i.e. |q| > exp(-Pi) = 0.043`.
  - **L205 CN**: 注释说明附近代码的意图或使用说明：`These functions are triggered when tau < 1, i.e. |q| > exp(-Pi) = 0.043`。
- **L206 EN**: Separator comment used for visual grouping.
  - **L206 CN**: 分隔注释，用于视觉分组。
- **L207 EN**: Comment documents nearby intent or usage notes: `Note that jacobi_theta4 uses the imaginary version of jacobi_theta2 (and`.
  - **L207 CN**: 注释说明附近代码的意图或使用说明：`Note that jacobi_theta4 uses the imaginary version of jacobi_theta2 (and`。
- **L208 EN**: Comment documents nearby intent or usage notes: `vice-versa). jacobi_theta1 and jacobi_theta3 use the imaginary versions of`.
  - **L208 CN**: 注释说明附近代码的意图或使用说明：`vice-versa). jacobi_theta1 and jacobi_theta3 use the imaginary versions of`。
- **L209 EN**: Comment documents nearby intent or usage notes: `themselves, following DLMF 20.7.30 - 20.7.33.`.
  - **L209 CN**: 注释说明附近代码的意图或使用说明：`themselves, following DLMF 20.7.30 - 20.7.33.`。
- **L210 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L210 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L211 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L211 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L212 EN**: Starts a function, method, lambda, or structured scope: `_IMAGINARY_jacobi_theta1tau(RealType z, RealType tau, const Policy&) {`.
  - **L212 CN**: 开始一个函数、方法、lambda 或结构化作用域：`_IMAGINARY_jacobi_theta1tau(RealType z, RealType tau, const Policy&) {`。
- **L213 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L213 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L214 EN**: Initializes variable `eps` from the right-hand expression.
  - **L214 CN**: 使用右侧表达式初始化变量 `eps`。
- **L215 EN**: Initializes variable `result` from the right-hand expression.
  - **L215 CN**: 使用右侧表达式初始化变量 `result`。
- **L216 EN**: Blank line separating nearby declarations or logic.
  - **L216 CN**: 空行，用于分隔相邻声明或逻辑。
- **L217 EN**: Comment documents nearby intent or usage notes: `n>=0 even`.
  - **L217 CN**: 注释说明附近代码的意图或使用说明：`n>=0 even`。
- **L218 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L218 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L219 EN**: Comment documents nearby intent or usage notes: `n>0 odd`.
  - **L219 CN**: 注释说明附近代码的意图或使用说明：`n>0 odd`。
- **L220 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L220 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。

### Lines 221-240 / 第 221-240 行

````cpp
 221:     // n<0 odd
 222:     result += _jacobi_theta_sum(tau, RealType(z - constants::half_pi<RealType>()), RealType (-constants::two_pi<RealType>()), eps);
 223:     // n<0 even
 224:     result -= _jacobi_theta_sum(tau, RealType(z - constants::half_pi<RealType>() - constants::pi<RealType>()), RealType (-constants::two_pi<RealType>()), eps);
 225: 
 226:     return result * sqrt(tau);
 227: }
 228: 
 229: template <class RealType, class Policy>
 230: inline RealType
 231: _IMAGINARY_jacobi_theta2tau(RealType z, RealType tau, const Policy&) {
 232:     BOOST_MATH_STD_USING
 233:     RealType eps = policies::get_epsilon<RealType, Policy>();
 234:     RealType result = RealType(0);
 235: 
 236:     // n>=0
 237:     result += _jacobi_theta_sum(tau, RealType(z + constants::half_pi<RealType>()), constants::pi<RealType>(), eps);
 238:     // n<0
 239:     result += _jacobi_theta_sum(tau, RealType(z - constants::half_pi<RealType>()), RealType (-constants::pi<RealType>()), eps);
 240: 
````
- **L221 EN**: Comment documents nearby intent or usage notes: `n<0 odd`.
  - **L221 CN**: 注释说明附近代码的意图或使用说明：`n<0 odd`。
- **L222 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L222 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L223 EN**: Comment documents nearby intent or usage notes: `n<0 even`.
  - **L223 CN**: 注释说明附近代码的意图或使用说明：`n<0 even`。
- **L224 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L224 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L225 EN**: Blank line separating nearby declarations or logic.
  - **L225 CN**: 空行，用于分隔相邻声明或逻辑。
- **L226 EN**: Returns from the current function with `result * sqrt(tau)`.
  - **L226 CN**: 以 `result * sqrt(tau)` 从当前函数返回。
- **L227 EN**: Closes the current lexical scope or compound statement.
  - **L227 CN**: 结束当前词法作用域或复合语句块。
- **L228 EN**: Blank line separating nearby declarations or logic.
  - **L228 CN**: 空行，用于分隔相邻声明或逻辑。
- **L229 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L229 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L230 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L230 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L231 EN**: Starts a function, method, lambda, or structured scope: `_IMAGINARY_jacobi_theta2tau(RealType z, RealType tau, const Policy&) {`.
  - **L231 CN**: 开始一个函数、方法、lambda 或结构化作用域：`_IMAGINARY_jacobi_theta2tau(RealType z, RealType tau, const Policy&) {`。
- **L232 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L232 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L233 EN**: Initializes variable `eps` from the right-hand expression.
  - **L233 CN**: 使用右侧表达式初始化变量 `eps`。
- **L234 EN**: Initializes variable `result` from the right-hand expression.
  - **L234 CN**: 使用右侧表达式初始化变量 `result`。
- **L235 EN**: Blank line separating nearby declarations or logic.
  - **L235 CN**: 空行，用于分隔相邻声明或逻辑。
- **L236 EN**: Comment documents nearby intent or usage notes: `n>=0`.
  - **L236 CN**: 注释说明附近代码的意图或使用说明：`n>=0`。
- **L237 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L237 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L238 EN**: Comment documents nearby intent or usage notes: `n<0`.
  - **L238 CN**: 注释说明附近代码的意图或使用说明：`n<0`。
- **L239 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L239 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L240 EN**: Blank line separating nearby declarations or logic.
  - **L240 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 241-260 / 第 241-260 行

````cpp
 241:     return result * sqrt(tau);
 242: }
 243: 
 244: template <class RealType, class Policy>
 245: inline RealType
 246: _IMAGINARY_jacobi_theta3tau(RealType z, RealType tau, const Policy&) {
 247:     BOOST_MATH_STD_USING
 248:     RealType eps = policies::get_epsilon<RealType, Policy>();
 249:     RealType result = 0;
 250: 
 251:     // n=0
 252:     result += exp(-z*z*tau/constants::pi<RealType>());
 253:     // n>0
 254:     result += _jacobi_theta_sum(tau, RealType(z + constants::pi<RealType>()), constants::pi<RealType>(), eps);
 255:     // n<0
 256:     result += _jacobi_theta_sum(tau, RealType(z - constants::pi<RealType>()), RealType(-constants::pi<RealType>()), eps);
 257: 
 258:     return result * sqrt(tau);
 259: }
 260: 
````
- **L241 EN**: Returns from the current function with `result * sqrt(tau)`.
  - **L241 CN**: 以 `result * sqrt(tau)` 从当前函数返回。
- **L242 EN**: Closes the current lexical scope or compound statement.
  - **L242 CN**: 结束当前词法作用域或复合语句块。
- **L243 EN**: Blank line separating nearby declarations or logic.
  - **L243 CN**: 空行，用于分隔相邻声明或逻辑。
- **L244 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L244 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L245 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L245 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L246 EN**: Starts a function, method, lambda, or structured scope: `_IMAGINARY_jacobi_theta3tau(RealType z, RealType tau, const Policy&) {`.
  - **L246 CN**: 开始一个函数、方法、lambda 或结构化作用域：`_IMAGINARY_jacobi_theta3tau(RealType z, RealType tau, const Policy&) {`。
- **L247 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L247 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L248 EN**: Initializes variable `eps` from the right-hand expression.
  - **L248 CN**: 使用右侧表达式初始化变量 `eps`。
- **L249 EN**: Initializes variable `result` from the right-hand expression.
  - **L249 CN**: 使用右侧表达式初始化变量 `result`。
- **L250 EN**: Blank line separating nearby declarations or logic.
  - **L250 CN**: 空行，用于分隔相邻声明或逻辑。
- **L251 EN**: Comment documents nearby intent or usage notes: `n=0`.
  - **L251 CN**: 注释说明附近代码的意图或使用说明：`n=0`。
- **L252 EN**: Executes a call or declaration centered on `exp`.
  - **L252 CN**: 执行以 `exp` 为核心的调用或声明。
- **L253 EN**: Comment documents nearby intent or usage notes: `n>0`.
  - **L253 CN**: 注释说明附近代码的意图或使用说明：`n>0`。
- **L254 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L254 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L255 EN**: Comment documents nearby intent or usage notes: `n<0`.
  - **L255 CN**: 注释说明附近代码的意图或使用说明：`n<0`。
- **L256 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L256 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L257 EN**: Blank line separating nearby declarations or logic.
  - **L257 CN**: 空行，用于分隔相邻声明或逻辑。
- **L258 EN**: Returns from the current function with `result * sqrt(tau)`.
  - **L258 CN**: 以 `result * sqrt(tau)` 从当前函数返回。
- **L259 EN**: Closes the current lexical scope or compound statement.
  - **L259 CN**: 结束当前词法作用域或复合语句块。
- **L260 EN**: Blank line separating nearby declarations or logic.
  - **L260 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 261-280 / 第 261-280 行

````cpp
 261: template <class RealType, class Policy>
 262: inline RealType
 263: _IMAGINARY_jacobi_theta4tau(RealType z, RealType tau, const Policy&) {
 264:     BOOST_MATH_STD_USING
 265:     RealType eps = policies::get_epsilon<RealType, Policy>();
 266:     RealType result = 0;
 267: 
 268:     // n = 0
 269:     result += exp(-z*z*tau/constants::pi<RealType>());
 270: 
 271:     // n > 0 odd
 272:     result -= _jacobi_theta_sum(tau, RealType(z + constants::pi<RealType>()), constants::two_pi<RealType>(), eps);
 273:     // n < 0 odd
 274:     result -= _jacobi_theta_sum(tau, RealType(z - constants::pi<RealType>()), RealType (-constants::two_pi<RealType>()), eps);
 275:     // n > 0 even
 276:     result += _jacobi_theta_sum(tau, RealType(z + constants::two_pi<RealType>()), constants::two_pi<RealType>(), eps);
 277:     // n < 0 even
 278:     result += _jacobi_theta_sum(tau, RealType(z - constants::two_pi<RealType>()), RealType (-constants::two_pi<RealType>()), eps);
 279: 
 280:     return result * sqrt(tau);
````
- **L261 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L261 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L262 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L262 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L263 EN**: Starts a function, method, lambda, or structured scope: `_IMAGINARY_jacobi_theta4tau(RealType z, RealType tau, const Policy&) {`.
  - **L263 CN**: 开始一个函数、方法、lambda 或结构化作用域：`_IMAGINARY_jacobi_theta4tau(RealType z, RealType tau, const Policy&) {`。
- **L264 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L264 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L265 EN**: Initializes variable `eps` from the right-hand expression.
  - **L265 CN**: 使用右侧表达式初始化变量 `eps`。
- **L266 EN**: Initializes variable `result` from the right-hand expression.
  - **L266 CN**: 使用右侧表达式初始化变量 `result`。
- **L267 EN**: Blank line separating nearby declarations or logic.
  - **L267 CN**: 空行，用于分隔相邻声明或逻辑。
- **L268 EN**: Comment documents nearby intent or usage notes: `n = 0`.
  - **L268 CN**: 注释说明附近代码的意图或使用说明：`n = 0`。
- **L269 EN**: Executes a call or declaration centered on `exp`.
  - **L269 CN**: 执行以 `exp` 为核心的调用或声明。
- **L270 EN**: Blank line separating nearby declarations or logic.
  - **L270 CN**: 空行，用于分隔相邻声明或逻辑。
- **L271 EN**: Comment documents nearby intent or usage notes: `n > 0 odd`.
  - **L271 CN**: 注释说明附近代码的意图或使用说明：`n > 0 odd`。
- **L272 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L272 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L273 EN**: Comment documents nearby intent or usage notes: `n < 0 odd`.
  - **L273 CN**: 注释说明附近代码的意图或使用说明：`n < 0 odd`。
- **L274 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L274 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L275 EN**: Comment documents nearby intent or usage notes: `n > 0 even`.
  - **L275 CN**: 注释说明附近代码的意图或使用说明：`n > 0 even`。
- **L276 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L276 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L277 EN**: Comment documents nearby intent or usage notes: `n < 0 even`.
  - **L277 CN**: 注释说明附近代码的意图或使用说明：`n < 0 even`。
- **L278 EN**: Executes a call or declaration centered on `_jacobi_theta_sum`.
  - **L278 CN**: 执行以 `_jacobi_theta_sum` 为核心的调用或声明。
- **L279 EN**: Blank line separating nearby declarations or logic.
  - **L279 CN**: 空行，用于分隔相邻声明或逻辑。
- **L280 EN**: Returns from the current function with `result * sqrt(tau)`.
  - **L280 CN**: 以 `result * sqrt(tau)` 从当前函数返回。

### Lines 281-300 / 第 281-300 行

````cpp
 281: }
 282: 
 283: // First Jacobi theta function (Parameterized by tau - assumed imaginary)
 284: // = 2 * SUM (-1)^n * exp(i*Pi*Tau*(n+1/2)^2) * sin((2n+1)z)
 285: template <class RealType, class Policy>
 286: inline RealType
 287: jacobi_theta1tau_imp(RealType z, RealType tau, const Policy& pol, const char *function)
 288: {
 289:     BOOST_MATH_STD_USING
 290:     unsigned n = 0;
 291:     RealType eps = policies::get_epsilon<RealType, Policy>();
 292:     RealType q_n = 0, last_q_n, delta, result = 0;
 293: 
 294:     if (tau <= 0.0)
 295:         return policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol);
 296: 
 297:     if (abs(z) == 0.0)
 298:         return result;
 299: 
 300:     if (tau < 1.0) {
````
- **L281 EN**: Closes the current lexical scope or compound statement.
  - **L281 CN**: 结束当前词法作用域或复合语句块。
- **L282 EN**: Blank line separating nearby declarations or logic.
  - **L282 CN**: 空行，用于分隔相邻声明或逻辑。
- **L283 EN**: Comment documents nearby intent or usage notes: `First Jacobi theta function (Parameterized by tau - assumed imaginary)`.
  - **L283 CN**: 注释说明附近代码的意图或使用说明：`First Jacobi theta function (Parameterized by tau - assumed imaginary)`。
- **L284 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM (-1)^n * exp(i*Pi*Tau*(n+1/2)^2) * sin((2n+1)z)`.
  - **L284 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM (-1)^n * exp(i*Pi*Tau*(n+1/2)^2) * sin((2n+1)z)`。
- **L285 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L285 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L286 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L286 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L287 EN**: Continues logic associated with callable symbol `jacobi_theta1tau_imp`.
  - **L287 CN**: 继续与可调用符号 `jacobi_theta1tau_imp` 相关的逻辑。
- **L288 EN**: Opens a new lexical scope or compound statement.
  - **L288 CN**: 打开一个新的词法作用域或复合语句块。
- **L289 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L289 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L290 EN**: Initializes variable `n` from the right-hand expression.
  - **L290 CN**: 使用右侧表达式初始化变量 `n`。
- **L291 EN**: Initializes variable `eps` from the right-hand expression.
  - **L291 CN**: 使用右侧表达式初始化变量 `eps`。
- **L292 EN**: Initializes variable `q_n` from the right-hand expression.
  - **L292 CN**: 使用右侧表达式初始化变量 `q_n`。
- **L293 EN**: Blank line separating nearby declarations or logic.
  - **L293 CN**: 空行，用于分隔相邻声明或逻辑。
- **L294 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L294 CN**: 开始 `if` 控制流语句并计算其条件。
- **L295 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)`.
  - **L295 CN**: 以 `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)` 从当前函数返回。
- **L296 EN**: Blank line separating nearby declarations or logic.
  - **L296 CN**: 空行，用于分隔相邻声明或逻辑。
- **L297 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L297 CN**: 开始 `if` 控制流语句并计算其条件。
- **L298 EN**: Returns from the current function with `result`.
  - **L298 CN**: 以 `result` 从当前函数返回。
- **L299 EN**: Blank line separating nearby declarations or logic.
  - **L299 CN**: 空行，用于分隔相邻声明或逻辑。
- **L300 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L300 CN**: 开始 `if` 控制流语句并计算其条件。

### Lines 301-320 / 第 301-320 行

````cpp
 301:         z = fmod(z, constants::two_pi<RealType>());
 302:         while (z > constants::pi<RealType>()) {
 303:             z -= constants::two_pi<RealType>();
 304:         }
 305:         while (z < -constants::pi<RealType>()) {
 306:             z += constants::two_pi<RealType>();
 307:         }
 308: 
 309:         return _IMAGINARY_jacobi_theta1tau(z, RealType(1/tau), pol);
 310:     }
 311: 
 312:     do {
 313:         last_q_n = q_n;
 314:         q_n = exp(-tau * constants::pi<RealType>() * RealType(n + 0.5)*RealType(n + 0.5) );
 315:         delta = q_n * sin(RealType(2*n+1)*z);
 316:         if (n%2)
 317:             delta = -delta;
 318: 
 319:         result += delta + delta;
 320:         n++;
````
- **L301 EN**: Executes a call or declaration centered on `fmod`.
  - **L301 CN**: 执行以 `fmod` 为核心的调用或声明。
- **L302 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L302 CN**: 开始 `while` 控制流语句并计算其条件。
- **L303 EN**: Executes a call or declaration centered on `constants::two_pi<RealType>`.
  - **L303 CN**: 执行以 `constants::two_pi<RealType>` 为核心的调用或声明。
- **L304 EN**: Closes the current lexical scope or compound statement.
  - **L304 CN**: 结束当前词法作用域或复合语句块。
- **L305 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L305 CN**: 开始 `while` 控制流语句并计算其条件。
- **L306 EN**: Executes a call or declaration centered on `constants::two_pi<RealType>`.
  - **L306 CN**: 执行以 `constants::two_pi<RealType>` 为核心的调用或声明。
- **L307 EN**: Closes the current lexical scope or compound statement.
  - **L307 CN**: 结束当前词法作用域或复合语句块。
- **L308 EN**: Blank line separating nearby declarations or logic.
  - **L308 CN**: 空行，用于分隔相邻声明或逻辑。
- **L309 EN**: Returns from the current function with `_IMAGINARY_jacobi_theta1tau(z, RealType(1/tau), pol)`.
  - **L309 CN**: 以 `_IMAGINARY_jacobi_theta1tau(z, RealType(1/tau), pol)` 从当前函数返回。
- **L310 EN**: Closes the current lexical scope or compound statement.
  - **L310 CN**: 结束当前词法作用域或复合语句块。
- **L311 EN**: Blank line separating nearby declarations or logic.
  - **L311 CN**: 空行，用于分隔相邻声明或逻辑。
- **L312 EN**: Continues the surrounding expression or declaration: `do {`.
  - **L312 CN**: 继续构造周围的表达式或声明：`do {`。
- **L313 EN**: Executes a standalone statement or declaration: `last_q_n = q_n;`.
  - **L313 CN**: 执行一条独立语句或声明：`last_q_n = q_n;`。
- **L314 EN**: Executes a call or declaration centered on `exp`.
  - **L314 CN**: 执行以 `exp` 为核心的调用或声明。
- **L315 EN**: Executes a call or declaration centered on `sin`.
  - **L315 CN**: 执行以 `sin` 为核心的调用或声明。
- **L316 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L316 CN**: 开始 `if` 控制流语句并计算其条件。
- **L317 EN**: Executes a standalone statement or declaration: `delta = -delta;`.
  - **L317 CN**: 执行一条独立语句或声明：`delta = -delta;`。
- **L318 EN**: Blank line separating nearby declarations or logic.
  - **L318 CN**: 空行，用于分隔相邻声明或逻辑。
- **L319 EN**: Executes a standalone statement or declaration: `result += delta + delta;`.
  - **L319 CN**: 执行一条独立语句或声明：`result += delta + delta;`。
- **L320 EN**: Executes a standalone statement or declaration: `n++;`.
  - **L320 CN**: 执行一条独立语句或声明：`n++;`。

### Lines 321-340 / 第 321-340 行

````cpp
 321:     } while (!_jacobi_theta_converged(last_q_n, q_n, eps));
 322: 
 323:     return result;
 324: }
 325: 
 326: // First Jacobi theta function (Parameterized by q)
 327: // = 2 * SUM (-1)^n * q^(n+1/2)^2 * sin((2n+1)z)
 328: template <class RealType, class Policy>
 329: inline RealType
 330: jacobi_theta1_imp(RealType z, RealType q, const Policy& pol, const char *function) {
 331:     BOOST_MATH_STD_USING
 332:     if (q <= 0.0 || q >= 1.0) {
 333:         return policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol);
 334:     }
 335:     return jacobi_theta1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function);
 336: }
 337: 
 338: // Second Jacobi theta function (Parameterized by tau - assumed imaginary)
 339: // = 2 * SUM exp(i*Pi*Tau*(n+1/2)^2) * cos((2n+1)z)
 340: template <class RealType, class Policy>
````
- **L321 EN**: Executes a call or declaration centered on `while`.
  - **L321 CN**: 执行以 `while` 为核心的调用或声明。
- **L322 EN**: Blank line separating nearby declarations or logic.
  - **L322 CN**: 空行，用于分隔相邻声明或逻辑。
- **L323 EN**: Returns from the current function with `result`.
  - **L323 CN**: 以 `result` 从当前函数返回。
- **L324 EN**: Closes the current lexical scope or compound statement.
  - **L324 CN**: 结束当前词法作用域或复合语句块。
- **L325 EN**: Blank line separating nearby declarations or logic.
  - **L325 CN**: 空行，用于分隔相邻声明或逻辑。
- **L326 EN**: Comment documents nearby intent or usage notes: `First Jacobi theta function (Parameterized by q)`.
  - **L326 CN**: 注释说明附近代码的意图或使用说明：`First Jacobi theta function (Parameterized by q)`。
- **L327 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM (-1)^n * q^(n+1/2)^2 * sin((2n+1)z)`.
  - **L327 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM (-1)^n * q^(n+1/2)^2 * sin((2n+1)z)`。
- **L328 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L328 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L329 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L329 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L330 EN**: Starts a function, method, lambda, or structured scope: `jacobi_theta1_imp(RealType z, RealType q, const Policy& pol, const char *function) {`.
  - **L330 CN**: 开始一个函数、方法、lambda 或结构化作用域：`jacobi_theta1_imp(RealType z, RealType q, const Policy& pol, const char *function) {`。
- **L331 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L331 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L332 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L332 CN**: 开始 `if` 控制流语句并计算其条件。
- **L333 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)`.
  - **L333 CN**: 以 `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)` 从当前函数返回。
- **L334 EN**: Closes the current lexical scope or compound statement.
  - **L334 CN**: 结束当前词法作用域或复合语句块。
- **L335 EN**: Returns from the current function with `jacobi_theta1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function)`.
  - **L335 CN**: 以 `jacobi_theta1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function)` 从当前函数返回。
- **L336 EN**: Closes the current lexical scope or compound statement.
  - **L336 CN**: 结束当前词法作用域或复合语句块。
- **L337 EN**: Blank line separating nearby declarations or logic.
  - **L337 CN**: 空行，用于分隔相邻声明或逻辑。
- **L338 EN**: Comment documents nearby intent or usage notes: `Second Jacobi theta function (Parameterized by tau - assumed imaginary)`.
  - **L338 CN**: 注释说明附近代码的意图或使用说明：`Second Jacobi theta function (Parameterized by tau - assumed imaginary)`。
- **L339 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM exp(i*Pi*Tau*(n+1/2)^2) * cos((2n+1)z)`.
  - **L339 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM exp(i*Pi*Tau*(n+1/2)^2) * cos((2n+1)z)`。
- **L340 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L340 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。

### Lines 341-360 / 第 341-360 行

````cpp
 341: inline RealType
 342: jacobi_theta2tau_imp(RealType z, RealType tau, const Policy& pol, const char *function)
 343: {
 344:     BOOST_MATH_STD_USING
 345:     unsigned n = 0;
 346:     RealType eps = policies::get_epsilon<RealType, Policy>();
 347:     RealType q_n = 0, last_q_n, delta, result = 0;
 348: 
 349:     if (tau <= 0.0) {
 350:         return policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol);
 351:     } else if (tau < 1.0 && abs(z) == 0.0) {
 352:         return jacobi_theta4tau(z, 1/tau, pol) / sqrt(tau);
 353:     } else if (tau < 1.0) { // DLMF 20.7.31
 354:         z = fmod(z, constants::two_pi<RealType>());
 355:         while (z > constants::pi<RealType>()) {
 356:             z -= constants::two_pi<RealType>();
 357:         }
 358:         while (z < -constants::pi<RealType>()) {
 359:             z += constants::two_pi<RealType>();
 360:         }
````
- **L341 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L341 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L342 EN**: Continues logic associated with callable symbol `jacobi_theta2tau_imp`.
  - **L342 CN**: 继续与可调用符号 `jacobi_theta2tau_imp` 相关的逻辑。
- **L343 EN**: Opens a new lexical scope or compound statement.
  - **L343 CN**: 打开一个新的词法作用域或复合语句块。
- **L344 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L344 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L345 EN**: Initializes variable `n` from the right-hand expression.
  - **L345 CN**: 使用右侧表达式初始化变量 `n`。
- **L346 EN**: Initializes variable `eps` from the right-hand expression.
  - **L346 CN**: 使用右侧表达式初始化变量 `eps`。
- **L347 EN**: Initializes variable `q_n` from the right-hand expression.
  - **L347 CN**: 使用右侧表达式初始化变量 `q_n`。
- **L348 EN**: Blank line separating nearby declarations or logic.
  - **L348 CN**: 空行，用于分隔相邻声明或逻辑。
- **L349 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L349 CN**: 开始 `if` 控制流语句并计算其条件。
- **L350 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)`.
  - **L350 CN**: 以 `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)` 从当前函数返回。
- **L351 EN**: Starts a function, method, lambda, or structured scope: `} else if (tau < 1.0 && abs(z) == 0.0) {`.
  - **L351 CN**: 开始一个函数、方法、lambda 或结构化作用域：`} else if (tau < 1.0 && abs(z) == 0.0) {`。
- **L352 EN**: Returns from the current function with `jacobi_theta4tau(z, 1/tau, pol) / sqrt(tau)`.
  - **L352 CN**: 以 `jacobi_theta4tau(z, 1/tau, pol) / sqrt(tau)` 从当前函数返回。
- **L353 EN**: Continues the surrounding expression or declaration: `} else if (tau < 1.0) { // DLMF 20.7.31`.
  - **L353 CN**: 继续构造周围的表达式或声明：`} else if (tau < 1.0) { // DLMF 20.7.31`。
- **L354 EN**: Executes a call or declaration centered on `fmod`.
  - **L354 CN**: 执行以 `fmod` 为核心的调用或声明。
- **L355 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L355 CN**: 开始 `while` 控制流语句并计算其条件。
- **L356 EN**: Executes a call or declaration centered on `constants::two_pi<RealType>`.
  - **L356 CN**: 执行以 `constants::two_pi<RealType>` 为核心的调用或声明。
- **L357 EN**: Closes the current lexical scope or compound statement.
  - **L357 CN**: 结束当前词法作用域或复合语句块。
- **L358 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L358 CN**: 开始 `while` 控制流语句并计算其条件。
- **L359 EN**: Executes a call or declaration centered on `constants::two_pi<RealType>`.
  - **L359 CN**: 执行以 `constants::two_pi<RealType>` 为核心的调用或声明。
- **L360 EN**: Closes the current lexical scope or compound statement.
  - **L360 CN**: 结束当前词法作用域或复合语句块。

### Lines 361-380 / 第 361-380 行

````cpp
 361: 
 362:         return _IMAGINARY_jacobi_theta4tau(z, RealType(1/tau), pol);
 363:     }
 364: 
 365:     do {
 366:         last_q_n = q_n;
 367:         q_n = exp(-tau * constants::pi<RealType>() * RealType(n + 0.5)*RealType(n + 0.5));
 368:         delta = q_n * cos(RealType(2*n+1)*z);
 369:         result += delta + delta;
 370:         n++;
 371:     } while (!_jacobi_theta_converged(last_q_n, q_n, eps));
 372: 
 373:     return result;
 374: }
 375: 
 376: // Second Jacobi theta function, parameterized by q
 377: // = 2 * SUM q^(n+1/2)^2 * cos((2n+1)z)
 378: template <class RealType, class Policy>
 379: inline RealType
 380: jacobi_theta2_imp(RealType z, RealType q, const Policy& pol, const char *function) {
````
- **L361 EN**: Blank line separating nearby declarations or logic.
  - **L361 CN**: 空行，用于分隔相邻声明或逻辑。
- **L362 EN**: Returns from the current function with `_IMAGINARY_jacobi_theta4tau(z, RealType(1/tau), pol)`.
  - **L362 CN**: 以 `_IMAGINARY_jacobi_theta4tau(z, RealType(1/tau), pol)` 从当前函数返回。
- **L363 EN**: Closes the current lexical scope or compound statement.
  - **L363 CN**: 结束当前词法作用域或复合语句块。
- **L364 EN**: Blank line separating nearby declarations or logic.
  - **L364 CN**: 空行，用于分隔相邻声明或逻辑。
- **L365 EN**: Continues the surrounding expression or declaration: `do {`.
  - **L365 CN**: 继续构造周围的表达式或声明：`do {`。
- **L366 EN**: Executes a standalone statement or declaration: `last_q_n = q_n;`.
  - **L366 CN**: 执行一条独立语句或声明：`last_q_n = q_n;`。
- **L367 EN**: Executes a call or declaration centered on `exp`.
  - **L367 CN**: 执行以 `exp` 为核心的调用或声明。
- **L368 EN**: Executes a call or declaration centered on `cos`.
  - **L368 CN**: 执行以 `cos` 为核心的调用或声明。
- **L369 EN**: Executes a standalone statement or declaration: `result += delta + delta;`.
  - **L369 CN**: 执行一条独立语句或声明：`result += delta + delta;`。
- **L370 EN**: Executes a standalone statement or declaration: `n++;`.
  - **L370 CN**: 执行一条独立语句或声明：`n++;`。
- **L371 EN**: Executes a call or declaration centered on `while`.
  - **L371 CN**: 执行以 `while` 为核心的调用或声明。
- **L372 EN**: Blank line separating nearby declarations or logic.
  - **L372 CN**: 空行，用于分隔相邻声明或逻辑。
- **L373 EN**: Returns from the current function with `result`.
  - **L373 CN**: 以 `result` 从当前函数返回。
- **L374 EN**: Closes the current lexical scope or compound statement.
  - **L374 CN**: 结束当前词法作用域或复合语句块。
- **L375 EN**: Blank line separating nearby declarations or logic.
  - **L375 CN**: 空行，用于分隔相邻声明或逻辑。
- **L376 EN**: Comment documents nearby intent or usage notes: `Second Jacobi theta function, parameterized by q`.
  - **L376 CN**: 注释说明附近代码的意图或使用说明：`Second Jacobi theta function, parameterized by q`。
- **L377 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM q^(n+1/2)^2 * cos((2n+1)z)`.
  - **L377 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM q^(n+1/2)^2 * cos((2n+1)z)`。
- **L378 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L378 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L379 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L379 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L380 EN**: Starts a function, method, lambda, or structured scope: `jacobi_theta2_imp(RealType z, RealType q, const Policy& pol, const char *function) {`.
  - **L380 CN**: 开始一个函数、方法、lambda 或结构化作用域：`jacobi_theta2_imp(RealType z, RealType q, const Policy& pol, const char *function) {`。

### Lines 381-400 / 第 381-400 行

````cpp
 381:     BOOST_MATH_STD_USING
 382:     if (q <= 0.0 || q >= 1.0) {
 383:         return policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol);
 384:     }
 385:     return jacobi_theta2tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function);
 386: }
 387: 
 388: // Third Jacobi theta function, minus one (Parameterized by tau - assumed imaginary)
 389: // This function preserves accuracy for small values of q (i.e. |q| < exp(-Pi) = 0.043)
 390: // For larger values of q, the minus one version usually won't help.
 391: // = 2 * SUM exp(i*Pi*Tau*(n)^2) * cos(2nz)
 392: template <class RealType, class Policy>
 393: inline RealType
 394: jacobi_theta3m1tau_imp(RealType z, RealType tau, const Policy& pol)
 395: {
 396:     BOOST_MATH_STD_USING
 397: 
 398:     RealType eps = policies::get_epsilon<RealType, Policy>();
 399:     RealType q_n = 0, last_q_n, delta, result = 0;
 400:     unsigned n = 1;
````
- **L381 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L381 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L382 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L382 CN**: 开始 `if` 控制流语句并计算其条件。
- **L383 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)`.
  - **L383 CN**: 以 `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)` 从当前函数返回。
- **L384 EN**: Closes the current lexical scope or compound statement.
  - **L384 CN**: 结束当前词法作用域或复合语句块。
- **L385 EN**: Returns from the current function with `jacobi_theta2tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function)`.
  - **L385 CN**: 以 `jacobi_theta2tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function)` 从当前函数返回。
- **L386 EN**: Closes the current lexical scope or compound statement.
  - **L386 CN**: 结束当前词法作用域或复合语句块。
- **L387 EN**: Blank line separating nearby declarations or logic.
  - **L387 CN**: 空行，用于分隔相邻声明或逻辑。
- **L388 EN**: Comment documents nearby intent or usage notes: `Third Jacobi theta function, minus one (Parameterized by tau - assumed imaginary)`.
  - **L388 CN**: 注释说明附近代码的意图或使用说明：`Third Jacobi theta function, minus one (Parameterized by tau - assumed imaginary)`。
- **L389 EN**: Comment documents nearby intent or usage notes: `This function preserves accuracy for small values of q (i.e. |q| < exp(-Pi) = 0.043)`.
  - **L389 CN**: 注释说明附近代码的意图或使用说明：`This function preserves accuracy for small values of q (i.e. |q| < exp(-Pi) = 0.043)`。
- **L390 EN**: Comment documents nearby intent or usage notes: `For larger values of q, the minus one version usually won't help.`.
  - **L390 CN**: 注释说明附近代码的意图或使用说明：`For larger values of q, the minus one version usually won't help.`。
- **L391 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM exp(i*Pi*Tau*(n)^2) * cos(2nz)`.
  - **L391 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM exp(i*Pi*Tau*(n)^2) * cos(2nz)`。
- **L392 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L392 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L393 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L393 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L394 EN**: Continues logic associated with callable symbol `jacobi_theta3m1tau_imp`.
  - **L394 CN**: 继续与可调用符号 `jacobi_theta3m1tau_imp` 相关的逻辑。
- **L395 EN**: Opens a new lexical scope or compound statement.
  - **L395 CN**: 打开一个新的词法作用域或复合语句块。
- **L396 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L396 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L397 EN**: Blank line separating nearby declarations or logic.
  - **L397 CN**: 空行，用于分隔相邻声明或逻辑。
- **L398 EN**: Initializes variable `eps` from the right-hand expression.
  - **L398 CN**: 使用右侧表达式初始化变量 `eps`。
- **L399 EN**: Initializes variable `q_n` from the right-hand expression.
  - **L399 CN**: 使用右侧表达式初始化变量 `q_n`。
- **L400 EN**: Initializes variable `n` from the right-hand expression.
  - **L400 CN**: 使用右侧表达式初始化变量 `n`。

### Lines 401-420 / 第 401-420 行

````cpp
 401: 
 402:     if (tau < 1.0)
 403:         return jacobi_theta3tau(z, tau, pol) - RealType(1);
 404: 
 405:     do {
 406:         last_q_n = q_n;
 407:         q_n = exp(-tau * constants::pi<RealType>() * RealType(n)*RealType(n));
 408:         delta = q_n * cos(RealType(2*n)*z);
 409:         result += delta + delta;
 410:         n++;
 411:     } while (!_jacobi_theta_converged(last_q_n, q_n, eps));
 412: 
 413:     return result;
 414: }
 415: 
 416: // Third Jacobi theta function, parameterized by tau
 417: // = 1 + 2 * SUM exp(i*Pi*Tau*(n)^2) * cos(2nz)
 418: template <class RealType, class Policy>
 419: inline RealType
 420: jacobi_theta3tau_imp(RealType z, RealType tau, const Policy& pol, const char *function)
````
- **L401 EN**: Blank line separating nearby declarations or logic.
  - **L401 CN**: 空行，用于分隔相邻声明或逻辑。
- **L402 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L402 CN**: 开始 `if` 控制流语句并计算其条件。
- **L403 EN**: Returns from the current function with `jacobi_theta3tau(z, tau, pol) - RealType(1)`.
  - **L403 CN**: 以 `jacobi_theta3tau(z, tau, pol) - RealType(1)` 从当前函数返回。
- **L404 EN**: Blank line separating nearby declarations or logic.
  - **L404 CN**: 空行，用于分隔相邻声明或逻辑。
- **L405 EN**: Continues the surrounding expression or declaration: `do {`.
  - **L405 CN**: 继续构造周围的表达式或声明：`do {`。
- **L406 EN**: Executes a standalone statement or declaration: `last_q_n = q_n;`.
  - **L406 CN**: 执行一条独立语句或声明：`last_q_n = q_n;`。
- **L407 EN**: Executes a call or declaration centered on `exp`.
  - **L407 CN**: 执行以 `exp` 为核心的调用或声明。
- **L408 EN**: Executes a call or declaration centered on `cos`.
  - **L408 CN**: 执行以 `cos` 为核心的调用或声明。
- **L409 EN**: Executes a standalone statement or declaration: `result += delta + delta;`.
  - **L409 CN**: 执行一条独立语句或声明：`result += delta + delta;`。
- **L410 EN**: Executes a standalone statement or declaration: `n++;`.
  - **L410 CN**: 执行一条独立语句或声明：`n++;`。
- **L411 EN**: Executes a call or declaration centered on `while`.
  - **L411 CN**: 执行以 `while` 为核心的调用或声明。
- **L412 EN**: Blank line separating nearby declarations or logic.
  - **L412 CN**: 空行，用于分隔相邻声明或逻辑。
- **L413 EN**: Returns from the current function with `result`.
  - **L413 CN**: 以 `result` 从当前函数返回。
- **L414 EN**: Closes the current lexical scope or compound statement.
  - **L414 CN**: 结束当前词法作用域或复合语句块。
- **L415 EN**: Blank line separating nearby declarations or logic.
  - **L415 CN**: 空行，用于分隔相邻声明或逻辑。
- **L416 EN**: Comment documents nearby intent or usage notes: `Third Jacobi theta function, parameterized by tau`.
  - **L416 CN**: 注释说明附近代码的意图或使用说明：`Third Jacobi theta function, parameterized by tau`。
- **L417 EN**: Comment documents nearby intent or usage notes: `= 1 + 2 * SUM exp(i*Pi*Tau*(n)^2) * cos(2nz)`.
  - **L417 CN**: 注释说明附近代码的意图或使用说明：`= 1 + 2 * SUM exp(i*Pi*Tau*(n)^2) * cos(2nz)`。
- **L418 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L418 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L419 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L419 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L420 EN**: Continues logic associated with callable symbol `jacobi_theta3tau_imp`.
  - **L420 CN**: 继续与可调用符号 `jacobi_theta3tau_imp` 相关的逻辑。

### Lines 421-440 / 第 421-440 行

````cpp
 421: {
 422:     BOOST_MATH_STD_USING
 423:     if (tau <= 0.0) {
 424:         return policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol);
 425:     } else if (tau < 1.0 && abs(z) == 0.0) {
 426:         return jacobi_theta3tau(z, RealType(1/tau), pol) / sqrt(tau);
 427:     } else if (tau < 1.0) { // DLMF 20.7.32
 428:         z = fmod(z, constants::pi<RealType>());
 429:         while (z > constants::half_pi<RealType>()) {
 430:             z -= constants::pi<RealType>();
 431:         }
 432:         while (z < -constants::half_pi<RealType>()) {
 433:             z += constants::pi<RealType>();
 434:         }
 435:         return _IMAGINARY_jacobi_theta3tau(z, RealType(1/tau), pol);
 436:     }
 437:     return RealType(1) + jacobi_theta3m1tau_imp(z, tau, pol);
 438: }
 439: 
 440: // Third Jacobi theta function, minus one (parameterized by q)
````
- **L421 EN**: Opens a new lexical scope or compound statement.
  - **L421 CN**: 打开一个新的词法作用域或复合语句块。
- **L422 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L422 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L423 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L423 CN**: 开始 `if` 控制流语句并计算其条件。
- **L424 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)`.
  - **L424 CN**: 以 `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)` 从当前函数返回。
- **L425 EN**: Starts a function, method, lambda, or structured scope: `} else if (tau < 1.0 && abs(z) == 0.0) {`.
  - **L425 CN**: 开始一个函数、方法、lambda 或结构化作用域：`} else if (tau < 1.0 && abs(z) == 0.0) {`。
- **L426 EN**: Returns from the current function with `jacobi_theta3tau(z, RealType(1/tau), pol) / sqrt(tau)`.
  - **L426 CN**: 以 `jacobi_theta3tau(z, RealType(1/tau), pol) / sqrt(tau)` 从当前函数返回。
- **L427 EN**: Continues the surrounding expression or declaration: `} else if (tau < 1.0) { // DLMF 20.7.32`.
  - **L427 CN**: 继续构造周围的表达式或声明：`} else if (tau < 1.0) { // DLMF 20.7.32`。
- **L428 EN**: Executes a call or declaration centered on `fmod`.
  - **L428 CN**: 执行以 `fmod` 为核心的调用或声明。
- **L429 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L429 CN**: 开始 `while` 控制流语句并计算其条件。
- **L430 EN**: Executes a call or declaration centered on `constants::pi<RealType>`.
  - **L430 CN**: 执行以 `constants::pi<RealType>` 为核心的调用或声明。
- **L431 EN**: Closes the current lexical scope or compound statement.
  - **L431 CN**: 结束当前词法作用域或复合语句块。
- **L432 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L432 CN**: 开始 `while` 控制流语句并计算其条件。
- **L433 EN**: Executes a call or declaration centered on `constants::pi<RealType>`.
  - **L433 CN**: 执行以 `constants::pi<RealType>` 为核心的调用或声明。
- **L434 EN**: Closes the current lexical scope or compound statement.
  - **L434 CN**: 结束当前词法作用域或复合语句块。
- **L435 EN**: Returns from the current function with `_IMAGINARY_jacobi_theta3tau(z, RealType(1/tau), pol)`.
  - **L435 CN**: 以 `_IMAGINARY_jacobi_theta3tau(z, RealType(1/tau), pol)` 从当前函数返回。
- **L436 EN**: Closes the current lexical scope or compound statement.
  - **L436 CN**: 结束当前词法作用域或复合语句块。
- **L437 EN**: Returns from the current function with `RealType(1) + jacobi_theta3m1tau_imp(z, tau, pol)`.
  - **L437 CN**: 以 `RealType(1) + jacobi_theta3m1tau_imp(z, tau, pol)` 从当前函数返回。
- **L438 EN**: Closes the current lexical scope or compound statement.
  - **L438 CN**: 结束当前词法作用域或复合语句块。
- **L439 EN**: Blank line separating nearby declarations or logic.
  - **L439 CN**: 空行，用于分隔相邻声明或逻辑。
- **L440 EN**: Comment documents nearby intent or usage notes: `Third Jacobi theta function, minus one (parameterized by q)`.
  - **L440 CN**: 注释说明附近代码的意图或使用说明：`Third Jacobi theta function, minus one (parameterized by q)`。

### Lines 441-460 / 第 441-460 行

````cpp
 441: // = 2 * SUM q^n^2 * cos(2nz)
 442: template <class RealType, class Policy>
 443: inline RealType
 444: jacobi_theta3m1_imp(RealType z, RealType q, const Policy& pol, const char *function) {
 445:     BOOST_MATH_STD_USING
 446:     if (q <= 0.0 || q >= 1.0) {
 447:         return policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol);
 448:     }
 449:     return jacobi_theta3m1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol);
 450: }
 451: 
 452: // Third Jacobi theta function (parameterized by q)
 453: // = 1 + 2 * SUM q^n^2 * cos(2nz)
 454: template <class RealType, class Policy>
 455: inline RealType
 456: jacobi_theta3_imp(RealType z, RealType q, const Policy& pol, const char *function) {
 457:     BOOST_MATH_STD_USING
 458:     if (q <= 0.0 || q >= 1.0) {
 459:         return policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol);
 460:     }
````
- **L441 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM q^n^2 * cos(2nz)`.
  - **L441 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM q^n^2 * cos(2nz)`。
- **L442 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L442 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L443 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L443 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L444 EN**: Starts a function, method, lambda, or structured scope: `jacobi_theta3m1_imp(RealType z, RealType q, const Policy& pol, const char *function) {`.
  - **L444 CN**: 开始一个函数、方法、lambda 或结构化作用域：`jacobi_theta3m1_imp(RealType z, RealType q, const Policy& pol, const char *function) {`。
- **L445 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L445 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L446 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L446 CN**: 开始 `if` 控制流语句并计算其条件。
- **L447 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)`.
  - **L447 CN**: 以 `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)` 从当前函数返回。
- **L448 EN**: Closes the current lexical scope or compound statement.
  - **L448 CN**: 结束当前词法作用域或复合语句块。
- **L449 EN**: Returns from the current function with `jacobi_theta3m1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol)`.
  - **L449 CN**: 以 `jacobi_theta3m1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol)` 从当前函数返回。
- **L450 EN**: Closes the current lexical scope or compound statement.
  - **L450 CN**: 结束当前词法作用域或复合语句块。
- **L451 EN**: Blank line separating nearby declarations or logic.
  - **L451 CN**: 空行，用于分隔相邻声明或逻辑。
- **L452 EN**: Comment documents nearby intent or usage notes: `Third Jacobi theta function (parameterized by q)`.
  - **L452 CN**: 注释说明附近代码的意图或使用说明：`Third Jacobi theta function (parameterized by q)`。
- **L453 EN**: Comment documents nearby intent or usage notes: `= 1 + 2 * SUM q^n^2 * cos(2nz)`.
  - **L453 CN**: 注释说明附近代码的意图或使用说明：`= 1 + 2 * SUM q^n^2 * cos(2nz)`。
- **L454 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L454 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L455 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L455 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L456 EN**: Starts a function, method, lambda, or structured scope: `jacobi_theta3_imp(RealType z, RealType q, const Policy& pol, const char *function) {`.
  - **L456 CN**: 开始一个函数、方法、lambda 或结构化作用域：`jacobi_theta3_imp(RealType z, RealType q, const Policy& pol, const char *function) {`。
- **L457 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L457 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L458 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L458 CN**: 开始 `if` 控制流语句并计算其条件。
- **L459 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)`.
  - **L459 CN**: 以 `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)` 从当前函数返回。
- **L460 EN**: Closes the current lexical scope or compound statement.
  - **L460 CN**: 结束当前词法作用域或复合语句块。

### Lines 461-480 / 第 461-480 行

````cpp
 461:     return jacobi_theta3tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function);
 462: }
 463: 
 464: // Fourth Jacobi theta function, minus one (Parameterized by tau)
 465: // This function preserves accuracy for small values of q (i.e. tau > 1)
 466: // = 2 * SUM (-1)^n exp(i*Pi*Tau*(n)^2) * cos(2nz)
 467: template <class RealType, class Policy>
 468: inline RealType
 469: jacobi_theta4m1tau_imp(RealType z, RealType tau, const Policy& pol)
 470: {
 471:     BOOST_MATH_STD_USING
 472: 
 473:     RealType eps = policies::get_epsilon<RealType, Policy>();
 474:     RealType q_n = 0, last_q_n, delta, result = 0;
 475:     unsigned n = 1;
 476: 
 477:     if (tau < 1.0)
 478:         return jacobi_theta4tau(z, tau, pol) - RealType(1);
 479: 
 480:     do {
````
- **L461 EN**: Returns from the current function with `jacobi_theta3tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function)`.
  - **L461 CN**: 以 `jacobi_theta3tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol, function)` 从当前函数返回。
- **L462 EN**: Closes the current lexical scope or compound statement.
  - **L462 CN**: 结束当前词法作用域或复合语句块。
- **L463 EN**: Blank line separating nearby declarations or logic.
  - **L463 CN**: 空行，用于分隔相邻声明或逻辑。
- **L464 EN**: Comment documents nearby intent or usage notes: `Fourth Jacobi theta function, minus one (Parameterized by tau)`.
  - **L464 CN**: 注释说明附近代码的意图或使用说明：`Fourth Jacobi theta function, minus one (Parameterized by tau)`。
- **L465 EN**: Comment documents nearby intent or usage notes: `This function preserves accuracy for small values of q (i.e. tau > 1)`.
  - **L465 CN**: 注释说明附近代码的意图或使用说明：`This function preserves accuracy for small values of q (i.e. tau > 1)`。
- **L466 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM (-1)^n exp(i*Pi*Tau*(n)^2) * cos(2nz)`.
  - **L466 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM (-1)^n exp(i*Pi*Tau*(n)^2) * cos(2nz)`。
- **L467 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L467 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L468 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L468 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L469 EN**: Continues logic associated with callable symbol `jacobi_theta4m1tau_imp`.
  - **L469 CN**: 继续与可调用符号 `jacobi_theta4m1tau_imp` 相关的逻辑。
- **L470 EN**: Opens a new lexical scope or compound statement.
  - **L470 CN**: 打开一个新的词法作用域或复合语句块。
- **L471 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L471 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L472 EN**: Blank line separating nearby declarations or logic.
  - **L472 CN**: 空行，用于分隔相邻声明或逻辑。
- **L473 EN**: Initializes variable `eps` from the right-hand expression.
  - **L473 CN**: 使用右侧表达式初始化变量 `eps`。
- **L474 EN**: Initializes variable `q_n` from the right-hand expression.
  - **L474 CN**: 使用右侧表达式初始化变量 `q_n`。
- **L475 EN**: Initializes variable `n` from the right-hand expression.
  - **L475 CN**: 使用右侧表达式初始化变量 `n`。
- **L476 EN**: Blank line separating nearby declarations or logic.
  - **L476 CN**: 空行，用于分隔相邻声明或逻辑。
- **L477 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L477 CN**: 开始 `if` 控制流语句并计算其条件。
- **L478 EN**: Returns from the current function with `jacobi_theta4tau(z, tau, pol) - RealType(1)`.
  - **L478 CN**: 以 `jacobi_theta4tau(z, tau, pol) - RealType(1)` 从当前函数返回。
- **L479 EN**: Blank line separating nearby declarations or logic.
  - **L479 CN**: 空行，用于分隔相邻声明或逻辑。
- **L480 EN**: Continues the surrounding expression or declaration: `do {`.
  - **L480 CN**: 继续构造周围的表达式或声明：`do {`。

### Lines 481-500 / 第 481-500 行

````cpp
 481:         last_q_n = q_n;
 482:         q_n = exp(-tau * constants::pi<RealType>() * RealType(n)*RealType(n));
 483:         delta = q_n * cos(RealType(2*n)*z);
 484:         if (n%2)
 485:             delta = -delta;
 486: 
 487:         result += delta + delta;
 488:         n++;
 489:     } while (!_jacobi_theta_converged(last_q_n, q_n, eps));
 490: 
 491:     return result;
 492: }
 493: 
 494: // Fourth Jacobi theta function (Parameterized by tau)
 495: // = 1 + 2 * SUM (-1)^n exp(i*Pi*Tau*(n)^2) * cos(2nz)
 496: template <class RealType, class Policy>
 497: inline RealType
 498: jacobi_theta4tau_imp(RealType z, RealType tau, const Policy& pol, const char *function)
 499: {
 500:     BOOST_MATH_STD_USING
````
- **L481 EN**: Executes a standalone statement or declaration: `last_q_n = q_n;`.
  - **L481 CN**: 执行一条独立语句或声明：`last_q_n = q_n;`。
- **L482 EN**: Executes a call or declaration centered on `exp`.
  - **L482 CN**: 执行以 `exp` 为核心的调用或声明。
- **L483 EN**: Executes a call or declaration centered on `cos`.
  - **L483 CN**: 执行以 `cos` 为核心的调用或声明。
- **L484 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L484 CN**: 开始 `if` 控制流语句并计算其条件。
- **L485 EN**: Executes a standalone statement or declaration: `delta = -delta;`.
  - **L485 CN**: 执行一条独立语句或声明：`delta = -delta;`。
- **L486 EN**: Blank line separating nearby declarations or logic.
  - **L486 CN**: 空行，用于分隔相邻声明或逻辑。
- **L487 EN**: Executes a standalone statement or declaration: `result += delta + delta;`.
  - **L487 CN**: 执行一条独立语句或声明：`result += delta + delta;`。
- **L488 EN**: Executes a standalone statement or declaration: `n++;`.
  - **L488 CN**: 执行一条独立语句或声明：`n++;`。
- **L489 EN**: Executes a call or declaration centered on `while`.
  - **L489 CN**: 执行以 `while` 为核心的调用或声明。
- **L490 EN**: Blank line separating nearby declarations or logic.
  - **L490 CN**: 空行，用于分隔相邻声明或逻辑。
- **L491 EN**: Returns from the current function with `result`.
  - **L491 CN**: 以 `result` 从当前函数返回。
- **L492 EN**: Closes the current lexical scope or compound statement.
  - **L492 CN**: 结束当前词法作用域或复合语句块。
- **L493 EN**: Blank line separating nearby declarations or logic.
  - **L493 CN**: 空行，用于分隔相邻声明或逻辑。
- **L494 EN**: Comment documents nearby intent or usage notes: `Fourth Jacobi theta function (Parameterized by tau)`.
  - **L494 CN**: 注释说明附近代码的意图或使用说明：`Fourth Jacobi theta function (Parameterized by tau)`。
- **L495 EN**: Comment documents nearby intent or usage notes: `= 1 + 2 * SUM (-1)^n exp(i*Pi*Tau*(n)^2) * cos(2nz)`.
  - **L495 CN**: 注释说明附近代码的意图或使用说明：`= 1 + 2 * SUM (-1)^n exp(i*Pi*Tau*(n)^2) * cos(2nz)`。
- **L496 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L496 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L497 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L497 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L498 EN**: Continues logic associated with callable symbol `jacobi_theta4tau_imp`.
  - **L498 CN**: 继续与可调用符号 `jacobi_theta4tau_imp` 相关的逻辑。
- **L499 EN**: Opens a new lexical scope or compound statement.
  - **L499 CN**: 打开一个新的词法作用域或复合语句块。
- **L500 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L500 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。

### Lines 501-520 / 第 501-520 行

````cpp
 501:     if (tau <= 0.0) {
 502:         return policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol);
 503:     } else if (tau < 1.0 && abs(z) == 0.0) {
 504:         return jacobi_theta2tau(z, 1/tau, pol) / sqrt(tau);
 505:     } else if (tau < 1.0) { // DLMF 20.7.33
 506:         z = fmod(z, constants::pi<RealType>());
 507:         while (z > constants::half_pi<RealType>()) {
 508:             z -= constants::pi<RealType>();
 509:         }
 510:         while (z < -constants::half_pi<RealType>()) {
 511:             z += constants::pi<RealType>();
 512:         }
 513:         return _IMAGINARY_jacobi_theta2tau(z, RealType(1/tau), pol);
 514:     }
 515: 
 516:     return RealType(1) + jacobi_theta4m1tau_imp(z, tau, pol);
 517: }
 518: 
 519: // Fourth Jacobi theta function, minus one (Parameterized by q)
 520: // This function preserves accuracy for small values of q
````
- **L501 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L501 CN**: 开始 `if` 控制流语句并计算其条件。
- **L502 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)`.
  - **L502 CN**: 以 `policies::raise_domain_error<RealType>(function, "tau must be greater than 0 but got %1%.", tau, pol)` 从当前函数返回。
- **L503 EN**: Starts a function, method, lambda, or structured scope: `} else if (tau < 1.0 && abs(z) == 0.0) {`.
  - **L503 CN**: 开始一个函数、方法、lambda 或结构化作用域：`} else if (tau < 1.0 && abs(z) == 0.0) {`。
- **L504 EN**: Returns from the current function with `jacobi_theta2tau(z, 1/tau, pol) / sqrt(tau)`.
  - **L504 CN**: 以 `jacobi_theta2tau(z, 1/tau, pol) / sqrt(tau)` 从当前函数返回。
- **L505 EN**: Continues the surrounding expression or declaration: `} else if (tau < 1.0) { // DLMF 20.7.33`.
  - **L505 CN**: 继续构造周围的表达式或声明：`} else if (tau < 1.0) { // DLMF 20.7.33`。
- **L506 EN**: Executes a call or declaration centered on `fmod`.
  - **L506 CN**: 执行以 `fmod` 为核心的调用或声明。
- **L507 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L507 CN**: 开始 `while` 控制流语句并计算其条件。
- **L508 EN**: Executes a call or declaration centered on `constants::pi<RealType>`.
  - **L508 CN**: 执行以 `constants::pi<RealType>` 为核心的调用或声明。
- **L509 EN**: Closes the current lexical scope or compound statement.
  - **L509 CN**: 结束当前词法作用域或复合语句块。
- **L510 EN**: Begins a `while` control-flow statement and evaluates its condition.
  - **L510 CN**: 开始 `while` 控制流语句并计算其条件。
- **L511 EN**: Executes a call or declaration centered on `constants::pi<RealType>`.
  - **L511 CN**: 执行以 `constants::pi<RealType>` 为核心的调用或声明。
- **L512 EN**: Closes the current lexical scope or compound statement.
  - **L512 CN**: 结束当前词法作用域或复合语句块。
- **L513 EN**: Returns from the current function with `_IMAGINARY_jacobi_theta2tau(z, RealType(1/tau), pol)`.
  - **L513 CN**: 以 `_IMAGINARY_jacobi_theta2tau(z, RealType(1/tau), pol)` 从当前函数返回。
- **L514 EN**: Closes the current lexical scope or compound statement.
  - **L514 CN**: 结束当前词法作用域或复合语句块。
- **L515 EN**: Blank line separating nearby declarations or logic.
  - **L515 CN**: 空行，用于分隔相邻声明或逻辑。
- **L516 EN**: Returns from the current function with `RealType(1) + jacobi_theta4m1tau_imp(z, tau, pol)`.
  - **L516 CN**: 以 `RealType(1) + jacobi_theta4m1tau_imp(z, tau, pol)` 从当前函数返回。
- **L517 EN**: Closes the current lexical scope or compound statement.
  - **L517 CN**: 结束当前词法作用域或复合语句块。
- **L518 EN**: Blank line separating nearby declarations or logic.
  - **L518 CN**: 空行，用于分隔相邻声明或逻辑。
- **L519 EN**: Comment documents nearby intent or usage notes: `Fourth Jacobi theta function, minus one (Parameterized by q)`.
  - **L519 CN**: 注释说明附近代码的意图或使用说明：`Fourth Jacobi theta function, minus one (Parameterized by q)`。
- **L520 EN**: Comment documents nearby intent or usage notes: `This function preserves accuracy for small values of q`.
  - **L520 CN**: 注释说明附近代码的意图或使用说明：`This function preserves accuracy for small values of q`。

### Lines 521-540 / 第 521-540 行

````cpp
 521: // = 2 * SUM q^n^2 * cos(2nz)
 522: template <class RealType, class Policy>
 523: inline RealType
 524: jacobi_theta4m1_imp(RealType z, RealType q, const Policy& pol, const char *function) {
 525:     BOOST_MATH_STD_USING
 526:     if (q <= 0.0 || q >= 1.0) {
 527:         return policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol);
 528:     }
 529:     return jacobi_theta4m1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol);
 530: }
 531: 
 532: // Fourth Jacobi theta function, parameterized by q
 533: // = 1 + 2 * SUM q^n^2 * cos(2nz)
 534: template <class RealType, class Policy>
 535: inline RealType
 536: jacobi_theta4_imp(RealType z, RealType q, const Policy& pol, const char *function) {
 537:     BOOST_MATH_STD_USING
 538:     if (q <= 0.0 || q >= 1.0) {
 539:         return policies::raise_domain_error<RealType>(function, "|q| must be greater than zero and less than 1, but got %1%.", q, pol);
 540:     }
````
- **L521 EN**: Comment documents nearby intent or usage notes: `= 2 * SUM q^n^2 * cos(2nz)`.
  - **L521 CN**: 注释说明附近代码的意图或使用说明：`= 2 * SUM q^n^2 * cos(2nz)`。
- **L522 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L522 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L523 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L523 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L524 EN**: Starts a function, method, lambda, or structured scope: `jacobi_theta4m1_imp(RealType z, RealType q, const Policy& pol, const char *function) {`.
  - **L524 CN**: 开始一个函数、方法、lambda 或结构化作用域：`jacobi_theta4m1_imp(RealType z, RealType q, const Policy& pol, const char *function) {`。
- **L525 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L525 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L526 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L526 CN**: 开始 `if` 控制流语句并计算其条件。
- **L527 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)`.
  - **L527 CN**: 以 `policies::raise_domain_error<RealType>(function, "q must be greater than 0 and less than 1 but got %1%.", q, pol)` 从当前函数返回。
- **L528 EN**: Closes the current lexical scope or compound statement.
  - **L528 CN**: 结束当前词法作用域或复合语句块。
- **L529 EN**: Returns from the current function with `jacobi_theta4m1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol)`.
  - **L529 CN**: 以 `jacobi_theta4m1tau_imp(z, RealType (-log(q)/constants::pi<RealType>()), pol)` 从当前函数返回。
- **L530 EN**: Closes the current lexical scope or compound statement.
  - **L530 CN**: 结束当前词法作用域或复合语句块。
- **L531 EN**: Blank line separating nearby declarations or logic.
  - **L531 CN**: 空行，用于分隔相邻声明或逻辑。
- **L532 EN**: Comment documents nearby intent or usage notes: `Fourth Jacobi theta function, parameterized by q`.
  - **L532 CN**: 注释说明附近代码的意图或使用说明：`Fourth Jacobi theta function, parameterized by q`。
- **L533 EN**: Comment documents nearby intent or usage notes: `= 1 + 2 * SUM q^n^2 * cos(2nz)`.
  - **L533 CN**: 注释说明附近代码的意图或使用说明：`= 1 + 2 * SUM q^n^2 * cos(2nz)`。
- **L534 EN**: Introduces template parameters or specialization context: `template <class RealType, class Policy>`.
  - **L534 CN**: 为后续声明引入模板参数或特化上下文：`template <class RealType, class Policy>`。
- **L535 EN**: Continues the surrounding expression or declaration: `inline RealType`.
  - **L535 CN**: 继续构造周围的表达式或声明：`inline RealType`。
- **L536 EN**: Starts a function, method, lambda, or structured scope: `jacobi_theta4_imp(RealType z, RealType q, const Policy& pol, const char *function) {`.
  - **L536 CN**: 开始一个函数、方法、lambda 或结构化作用域：`jacobi_theta4_imp(RealType z, RealType q, const Policy& pol, const char *function) {`。
- **L537 EN**: Uses a Boost.Math macro for portability, policy plumbing, or compile-time configuration.
  - **L537 CN**: 使用 Boost.Math 宏处理可移植性、策略连接或编译期配置。
- **L538 EN**: Begins a `if` control-flow statement and evaluates its condition.
  - **L538 CN**: 开始 `if` 控制流语句并计算其条件。
- **L539 EN**: Returns from the current function with `policies::raise_domain_error<RealType>(function, "|q| must be greater than zero and less than 1, but got %1%.", q, pol)`.
  - **L539 CN**: 以 `policies::raise_domain_error<RealType>(function, "|q| must be greater than zero and less than 1, but got %1%.", q, pol)` 从当前函数返回。
- **L540 EN**: Closes the current lexical scope or compound statement.
  - **L540 CN**: 结束当前词法作用域或复合语句块。

### Lines 541-560 / 第 541-560 行

````cpp
 541:     return jacobi_theta4tau_imp(z, RealType(-log(q)/constants::pi<RealType>()), pol, function);
 542: }
 543: 
 544: // Begin public API
 545: 
 546: template <class T, class U, class Policy>
 547: inline typename tools::promote_args<T, U>::type jacobi_theta1tau(T z, U tau, const Policy&) {
 548:    BOOST_FPU_EXCEPTION_GUARD
 549:    typedef typename tools::promote_args<T, U>::type result_type;
 550:    typedef typename policies::normalise<
 551:       Policy,
 552:       policies::promote_float<false>,
 553:       policies::promote_double<false>,
 554:       policies::discrete_quantile<>,
 555:       policies::assert_undefined<> >::type forwarding_policy;
 556: 
 557:    static const char* function = "boost::math::jacobi_theta1tau<%1%>(%1%)";
 558: 
 559:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta1tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function);
 560: }
````
- **L541 EN**: Returns from the current function with `jacobi_theta4tau_imp(z, RealType(-log(q)/constants::pi<RealType>()), pol, function)`.
  - **L541 CN**: 以 `jacobi_theta4tau_imp(z, RealType(-log(q)/constants::pi<RealType>()), pol, function)` 从当前函数返回。
- **L542 EN**: Closes the current lexical scope or compound statement.
  - **L542 CN**: 结束当前词法作用域或复合语句块。
- **L543 EN**: Blank line separating nearby declarations or logic.
  - **L543 CN**: 空行，用于分隔相邻声明或逻辑。
- **L544 EN**: Comment documents nearby intent or usage notes: `Begin public API`.
  - **L544 CN**: 注释说明附近代码的意图或使用说明：`Begin public API`。
- **L545 EN**: Blank line separating nearby declarations or logic.
  - **L545 CN**: 空行，用于分隔相邻声明或逻辑。
- **L546 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L546 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L547 EN**: Starts a function or method definition for `jacobi_theta1tau`.
  - **L547 CN**: 开始定义函数或方法 `jacobi_theta1tau`。
- **L548 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L548 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L549 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L549 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L550 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L550 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L551 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L551 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L552 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L552 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L553 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L553 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L554 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L554 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L555 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L555 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L556 EN**: Blank line separating nearby declarations or logic.
  - **L556 CN**: 空行，用于分隔相邻声明或逻辑。
- **L557 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L557 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L558 EN**: Blank line separating nearby declarations or logic.
  - **L558 CN**: 空行，用于分隔相邻声明或逻辑。
- **L559 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta1tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)`.
  - **L559 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta1tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)` 从当前函数返回。
- **L560 EN**: Closes the current lexical scope or compound statement.
  - **L560 CN**: 结束当前词法作用域或复合语句块。

### Lines 561-580 / 第 561-580 行

````cpp
 561: 
 562: template <class T, class U>
 563: inline typename tools::promote_args<T, U>::type jacobi_theta1tau(T z, U tau) {
 564:     return jacobi_theta1tau(z, tau, policies::policy<>());
 565: }
 566: 
 567: template <class T, class U, class Policy>
 568: inline typename tools::promote_args<T, U>::type jacobi_theta1(T z, U q, const Policy&) {
 569:    BOOST_FPU_EXCEPTION_GUARD
 570:    typedef typename tools::promote_args<T, U>::type result_type;
 571:    typedef typename policies::normalise<
 572:       Policy,
 573:       policies::promote_float<false>,
 574:       policies::promote_double<false>,
 575:       policies::discrete_quantile<>,
 576:       policies::assert_undefined<> >::type forwarding_policy;
 577: 
 578:    static const char* function = "boost::math::jacobi_theta1<%1%>(%1%)";
 579: 
 580:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function);
````
- **L561 EN**: Blank line separating nearby declarations or logic.
  - **L561 CN**: 空行，用于分隔相邻声明或逻辑。
- **L562 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L562 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L563 EN**: Starts a function or method definition for `jacobi_theta1tau`.
  - **L563 CN**: 开始定义函数或方法 `jacobi_theta1tau`。
- **L564 EN**: Returns from the current function with `jacobi_theta1tau(z, tau, policies::policy<>())`.
  - **L564 CN**: 以 `jacobi_theta1tau(z, tau, policies::policy<>())` 从当前函数返回。
- **L565 EN**: Closes the current lexical scope or compound statement.
  - **L565 CN**: 结束当前词法作用域或复合语句块。
- **L566 EN**: Blank line separating nearby declarations or logic.
  - **L566 CN**: 空行，用于分隔相邻声明或逻辑。
- **L567 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L567 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L568 EN**: Starts a function or method definition for `jacobi_theta1`.
  - **L568 CN**: 开始定义函数或方法 `jacobi_theta1`。
- **L569 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L569 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L570 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L570 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L571 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L571 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L572 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L572 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L573 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L573 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L574 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L574 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L575 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L575 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L576 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L576 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L577 EN**: Blank line separating nearby declarations or logic.
  - **L577 CN**: 空行，用于分隔相邻声明或逻辑。
- **L578 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L578 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L579 EN**: Blank line separating nearby declarations or logic.
  - **L579 CN**: 空行，用于分隔相邻声明或逻辑。
- **L580 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)`.
  - **L580 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)` 从当前函数返回。

### Lines 581-600 / 第 581-600 行

````cpp
 581: }
 582: 
 583: template <class T, class U>
 584: inline typename tools::promote_args<T, U>::type jacobi_theta1(T z, U q) {
 585:     return jacobi_theta1(z, q, policies::policy<>());
 586: }
 587: 
 588: template <class T, class U, class Policy>
 589: inline typename tools::promote_args<T, U>::type jacobi_theta2tau(T z, U tau, const Policy&) {
 590:    BOOST_FPU_EXCEPTION_GUARD
 591:    typedef typename tools::promote_args<T, U>::type result_type;
 592:    typedef typename policies::normalise<
 593:       Policy,
 594:       policies::promote_float<false>,
 595:       policies::promote_double<false>,
 596:       policies::discrete_quantile<>,
 597:       policies::assert_undefined<> >::type forwarding_policy;
 598: 
 599:    static const char* function = "boost::math::jacobi_theta2tau<%1%>(%1%)";
 600: 
````
- **L581 EN**: Closes the current lexical scope or compound statement.
  - **L581 CN**: 结束当前词法作用域或复合语句块。
- **L582 EN**: Blank line separating nearby declarations or logic.
  - **L582 CN**: 空行，用于分隔相邻声明或逻辑。
- **L583 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L583 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L584 EN**: Starts a function or method definition for `jacobi_theta1`.
  - **L584 CN**: 开始定义函数或方法 `jacobi_theta1`。
- **L585 EN**: Returns from the current function with `jacobi_theta1(z, q, policies::policy<>())`.
  - **L585 CN**: 以 `jacobi_theta1(z, q, policies::policy<>())` 从当前函数返回。
- **L586 EN**: Closes the current lexical scope or compound statement.
  - **L586 CN**: 结束当前词法作用域或复合语句块。
- **L587 EN**: Blank line separating nearby declarations or logic.
  - **L587 CN**: 空行，用于分隔相邻声明或逻辑。
- **L588 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L588 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L589 EN**: Starts a function or method definition for `jacobi_theta2tau`.
  - **L589 CN**: 开始定义函数或方法 `jacobi_theta2tau`。
- **L590 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L590 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L591 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L591 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L592 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L592 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L593 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L593 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L594 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L594 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L595 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L595 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L596 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L596 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L597 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L597 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L598 EN**: Blank line separating nearby declarations or logic.
  - **L598 CN**: 空行，用于分隔相邻声明或逻辑。
- **L599 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L599 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L600 EN**: Blank line separating nearby declarations or logic.
  - **L600 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 601-620 / 第 601-620 行

````cpp
 601:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta2tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function);
 602: }
 603: 
 604: template <class T, class U>
 605: inline typename tools::promote_args<T, U>::type jacobi_theta2tau(T z, U tau) {
 606:     return jacobi_theta2tau(z, tau, policies::policy<>());
 607: }
 608: 
 609: template <class T, class U, class Policy>
 610: inline typename tools::promote_args<T, U>::type jacobi_theta2(T z, U q, const Policy&) {
 611:    BOOST_FPU_EXCEPTION_GUARD
 612:    typedef typename tools::promote_args<T, U>::type result_type;
 613:    typedef typename policies::normalise<
 614:       Policy,
 615:       policies::promote_float<false>,
 616:       policies::promote_double<false>,
 617:       policies::discrete_quantile<>,
 618:       policies::assert_undefined<> >::type forwarding_policy;
 619: 
 620:    static const char* function = "boost::math::jacobi_theta2<%1%>(%1%)";
````
- **L601 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta2tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)`.
  - **L601 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta2tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)` 从当前函数返回。
- **L602 EN**: Closes the current lexical scope or compound statement.
  - **L602 CN**: 结束当前词法作用域或复合语句块。
- **L603 EN**: Blank line separating nearby declarations or logic.
  - **L603 CN**: 空行，用于分隔相邻声明或逻辑。
- **L604 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L604 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L605 EN**: Starts a function or method definition for `jacobi_theta2tau`.
  - **L605 CN**: 开始定义函数或方法 `jacobi_theta2tau`。
- **L606 EN**: Returns from the current function with `jacobi_theta2tau(z, tau, policies::policy<>())`.
  - **L606 CN**: 以 `jacobi_theta2tau(z, tau, policies::policy<>())` 从当前函数返回。
- **L607 EN**: Closes the current lexical scope or compound statement.
  - **L607 CN**: 结束当前词法作用域或复合语句块。
- **L608 EN**: Blank line separating nearby declarations or logic.
  - **L608 CN**: 空行，用于分隔相邻声明或逻辑。
- **L609 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L609 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L610 EN**: Starts a function or method definition for `jacobi_theta2`.
  - **L610 CN**: 开始定义函数或方法 `jacobi_theta2`。
- **L611 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L611 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L612 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L612 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L613 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L613 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L614 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L614 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L615 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L615 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L616 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L616 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L617 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L617 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L618 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L618 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L619 EN**: Blank line separating nearby declarations or logic.
  - **L619 CN**: 空行，用于分隔相邻声明或逻辑。
- **L620 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L620 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。

### Lines 621-640 / 第 621-640 行

````cpp
 621: 
 622:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta2_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function);
 623: }
 624: 
 625: template <class T, class U>
 626: inline typename tools::promote_args<T, U>::type jacobi_theta2(T z, U q) {
 627:     return jacobi_theta2(z, q, policies::policy<>());
 628: }
 629: 
 630: template <class T, class U, class Policy>
 631: inline typename tools::promote_args<T, U>::type jacobi_theta3m1tau(T z, U tau, const Policy&) {
 632:    BOOST_FPU_EXCEPTION_GUARD
 633:    typedef typename tools::promote_args<T, U>::type result_type;
 634:    typedef typename policies::normalise<
 635:       Policy,
 636:       policies::promote_float<false>,
 637:       policies::promote_double<false>,
 638:       policies::discrete_quantile<>,
 639:       policies::assert_undefined<> >::type forwarding_policy;
 640: 
````
- **L621 EN**: Blank line separating nearby declarations or logic.
  - **L621 CN**: 空行，用于分隔相邻声明或逻辑。
- **L622 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta2_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)`.
  - **L622 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta2_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)` 从当前函数返回。
- **L623 EN**: Closes the current lexical scope or compound statement.
  - **L623 CN**: 结束当前词法作用域或复合语句块。
- **L624 EN**: Blank line separating nearby declarations or logic.
  - **L624 CN**: 空行，用于分隔相邻声明或逻辑。
- **L625 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L625 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L626 EN**: Starts a function or method definition for `jacobi_theta2`.
  - **L626 CN**: 开始定义函数或方法 `jacobi_theta2`。
- **L627 EN**: Returns from the current function with `jacobi_theta2(z, q, policies::policy<>())`.
  - **L627 CN**: 以 `jacobi_theta2(z, q, policies::policy<>())` 从当前函数返回。
- **L628 EN**: Closes the current lexical scope or compound statement.
  - **L628 CN**: 结束当前词法作用域或复合语句块。
- **L629 EN**: Blank line separating nearby declarations or logic.
  - **L629 CN**: 空行，用于分隔相邻声明或逻辑。
- **L630 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L630 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L631 EN**: Starts a function or method definition for `jacobi_theta3m1tau`.
  - **L631 CN**: 开始定义函数或方法 `jacobi_theta3m1tau`。
- **L632 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L632 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L633 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L633 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L634 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L634 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L635 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L635 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L636 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L636 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L637 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L637 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L638 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L638 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L639 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L639 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L640 EN**: Blank line separating nearby declarations or logic.
  - **L640 CN**: 空行，用于分隔相邻声明或逻辑。

### Lines 641-660 / 第 641-660 行

````cpp
 641:    static const char* function = "boost::math::jacobi_theta3m1tau<%1%>(%1%)";
 642: 
 643:    return policies::checked_narrowing_cast<result_type, Policy>(
 644:            jacobi_theta3m1tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy()), function);
 645: }
 646: 
 647: template <class T, class U>
 648: inline typename tools::promote_args<T, U>::type jacobi_theta3m1tau(T z, U tau) {
 649:     return jacobi_theta3m1tau(z, tau, policies::policy<>());
 650: }
 651: 
 652: template <class T, class U, class Policy>
 653: inline typename tools::promote_args<T, U>::type jacobi_theta3tau(T z, U tau, const Policy&) {
 654:    BOOST_FPU_EXCEPTION_GUARD
 655:    typedef typename tools::promote_args<T, U>::type result_type;
 656:    typedef typename policies::normalise<
 657:       Policy,
 658:       policies::promote_float<false>,
 659:       policies::promote_double<false>,
 660:       policies::discrete_quantile<>,
````
- **L641 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L641 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L642 EN**: Blank line separating nearby declarations or logic.
  - **L642 CN**: 空行，用于分隔相邻声明或逻辑。
- **L643 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(`.
  - **L643 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(` 从当前函数返回。
- **L644 EN**: Executes a call or declaration centered on `jacobi_theta3m1tau_imp`.
  - **L644 CN**: 执行以 `jacobi_theta3m1tau_imp` 为核心的调用或声明。
- **L645 EN**: Closes the current lexical scope or compound statement.
  - **L645 CN**: 结束当前词法作用域或复合语句块。
- **L646 EN**: Blank line separating nearby declarations or logic.
  - **L646 CN**: 空行，用于分隔相邻声明或逻辑。
- **L647 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L647 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L648 EN**: Starts a function or method definition for `jacobi_theta3m1tau`.
  - **L648 CN**: 开始定义函数或方法 `jacobi_theta3m1tau`。
- **L649 EN**: Returns from the current function with `jacobi_theta3m1tau(z, tau, policies::policy<>())`.
  - **L649 CN**: 以 `jacobi_theta3m1tau(z, tau, policies::policy<>())` 从当前函数返回。
- **L650 EN**: Closes the current lexical scope or compound statement.
  - **L650 CN**: 结束当前词法作用域或复合语句块。
- **L651 EN**: Blank line separating nearby declarations or logic.
  - **L651 CN**: 空行，用于分隔相邻声明或逻辑。
- **L652 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L652 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L653 EN**: Starts a function or method definition for `jacobi_theta3tau`.
  - **L653 CN**: 开始定义函数或方法 `jacobi_theta3tau`。
- **L654 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L654 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L655 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L655 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L656 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L656 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L657 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L657 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L658 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L658 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L659 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L659 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L660 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L660 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。

### Lines 661-680 / 第 661-680 行

````cpp
 661:       policies::assert_undefined<> >::type forwarding_policy;
 662: 
 663:    static const char* function = "boost::math::jacobi_theta3tau<%1%>(%1%)";
 664: 
 665:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function);
 666: }
 667: 
 668: template <class T, class U>
 669: inline typename tools::promote_args<T, U>::type jacobi_theta3tau(T z, U tau) {
 670:     return jacobi_theta3tau(z, tau, policies::policy<>());
 671: }
 672: 
 673: 
 674: template <class T, class U, class Policy>
 675: inline typename tools::promote_args<T, U>::type jacobi_theta3m1(T z, U q, const Policy&) {
 676:    BOOST_FPU_EXCEPTION_GUARD
 677:    typedef typename tools::promote_args<T, U>::type result_type;
 678:    typedef typename policies::normalise<
 679:       Policy,
 680:       policies::promote_float<false>,
````
- **L661 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L661 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L662 EN**: Blank line separating nearby declarations or logic.
  - **L662 CN**: 空行，用于分隔相邻声明或逻辑。
- **L663 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L663 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L664 EN**: Blank line separating nearby declarations or logic.
  - **L664 CN**: 空行，用于分隔相邻声明或逻辑。
- **L665 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)`.
  - **L665 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)` 从当前函数返回。
- **L666 EN**: Closes the current lexical scope or compound statement.
  - **L666 CN**: 结束当前词法作用域或复合语句块。
- **L667 EN**: Blank line separating nearby declarations or logic.
  - **L667 CN**: 空行，用于分隔相邻声明或逻辑。
- **L668 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L668 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L669 EN**: Starts a function or method definition for `jacobi_theta3tau`.
  - **L669 CN**: 开始定义函数或方法 `jacobi_theta3tau`。
- **L670 EN**: Returns from the current function with `jacobi_theta3tau(z, tau, policies::policy<>())`.
  - **L670 CN**: 以 `jacobi_theta3tau(z, tau, policies::policy<>())` 从当前函数返回。
- **L671 EN**: Closes the current lexical scope or compound statement.
  - **L671 CN**: 结束当前词法作用域或复合语句块。
- **L672 EN**: Blank line separating nearby declarations or logic.
  - **L672 CN**: 空行，用于分隔相邻声明或逻辑。
- **L673 EN**: Blank line separating nearby declarations or logic.
  - **L673 CN**: 空行，用于分隔相邻声明或逻辑。
- **L674 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L674 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L675 EN**: Starts a function or method definition for `jacobi_theta3m1`.
  - **L675 CN**: 开始定义函数或方法 `jacobi_theta3m1`。
- **L676 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L676 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L677 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L677 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L678 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L678 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L679 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L679 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L680 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L680 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。

### Lines 681-700 / 第 681-700 行

````cpp
 681:       policies::promote_double<false>,
 682:       policies::discrete_quantile<>,
 683:       policies::assert_undefined<> >::type forwarding_policy;
 684: 
 685:    static const char* function = "boost::math::jacobi_theta3m1<%1%>(%1%)";
 686: 
 687:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3m1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function);
 688: }
 689: 
 690: template <class T, class U>
 691: inline typename tools::promote_args<T, U>::type jacobi_theta3m1(T z, U q) {
 692:     return jacobi_theta3m1(z, q, policies::policy<>());
 693: }
 694: 
 695: template <class T, class U, class Policy>
 696: inline typename tools::promote_args<T, U>::type jacobi_theta3(T z, U q, const Policy&) {
 697:    BOOST_FPU_EXCEPTION_GUARD
 698:    typedef typename tools::promote_args<T, U>::type result_type;
 699:    typedef typename policies::normalise<
 700:       Policy,
````
- **L681 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L681 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L682 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L682 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L683 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L683 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L684 EN**: Blank line separating nearby declarations or logic.
  - **L684 CN**: 空行，用于分隔相邻声明或逻辑。
- **L685 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L685 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L686 EN**: Blank line separating nearby declarations or logic.
  - **L686 CN**: 空行，用于分隔相邻声明或逻辑。
- **L687 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3m1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)`.
  - **L687 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3m1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)` 从当前函数返回。
- **L688 EN**: Closes the current lexical scope or compound statement.
  - **L688 CN**: 结束当前词法作用域或复合语句块。
- **L689 EN**: Blank line separating nearby declarations or logic.
  - **L689 CN**: 空行，用于分隔相邻声明或逻辑。
- **L690 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L690 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L691 EN**: Starts a function or method definition for `jacobi_theta3m1`.
  - **L691 CN**: 开始定义函数或方法 `jacobi_theta3m1`。
- **L692 EN**: Returns from the current function with `jacobi_theta3m1(z, q, policies::policy<>())`.
  - **L692 CN**: 以 `jacobi_theta3m1(z, q, policies::policy<>())` 从当前函数返回。
- **L693 EN**: Closes the current lexical scope or compound statement.
  - **L693 CN**: 结束当前词法作用域或复合语句块。
- **L694 EN**: Blank line separating nearby declarations or logic.
  - **L694 CN**: 空行，用于分隔相邻声明或逻辑。
- **L695 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L695 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L696 EN**: Starts a function or method definition for `jacobi_theta3`.
  - **L696 CN**: 开始定义函数或方法 `jacobi_theta3`。
- **L697 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L697 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L698 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L698 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L699 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L699 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L700 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L700 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。

### Lines 701-720 / 第 701-720 行

````cpp
 701:       policies::promote_float<false>,
 702:       policies::promote_double<false>,
 703:       policies::discrete_quantile<>,
 704:       policies::assert_undefined<> >::type forwarding_policy;
 705: 
 706:    static const char* function = "boost::math::jacobi_theta3<%1%>(%1%)";
 707: 
 708:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function);
 709: }
 710: 
 711: template <class T, class U>
 712: inline typename tools::promote_args<T, U>::type jacobi_theta3(T z, U q) {
 713:     return jacobi_theta3(z, q, policies::policy<>());
 714: }
 715: 
 716: template <class T, class U, class Policy>
 717: inline typename tools::promote_args<T, U>::type jacobi_theta4m1tau(T z, U tau, const Policy&) {
 718:    BOOST_FPU_EXCEPTION_GUARD
 719:    typedef typename tools::promote_args<T, U>::type result_type;
 720:    typedef typename policies::normalise<
````
- **L701 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L701 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L702 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L702 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L703 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L703 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L704 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L704 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L705 EN**: Blank line separating nearby declarations or logic.
  - **L705 CN**: 空行，用于分隔相邻声明或逻辑。
- **L706 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L706 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L707 EN**: Blank line separating nearby declarations or logic.
  - **L707 CN**: 空行，用于分隔相邻声明或逻辑。
- **L708 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)`.
  - **L708 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta3_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)` 从当前函数返回。
- **L709 EN**: Closes the current lexical scope or compound statement.
  - **L709 CN**: 结束当前词法作用域或复合语句块。
- **L710 EN**: Blank line separating nearby declarations or logic.
  - **L710 CN**: 空行，用于分隔相邻声明或逻辑。
- **L711 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L711 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L712 EN**: Starts a function or method definition for `jacobi_theta3`.
  - **L712 CN**: 开始定义函数或方法 `jacobi_theta3`。
- **L713 EN**: Returns from the current function with `jacobi_theta3(z, q, policies::policy<>())`.
  - **L713 CN**: 以 `jacobi_theta3(z, q, policies::policy<>())` 从当前函数返回。
- **L714 EN**: Closes the current lexical scope or compound statement.
  - **L714 CN**: 结束当前词法作用域或复合语句块。
- **L715 EN**: Blank line separating nearby declarations or logic.
  - **L715 CN**: 空行，用于分隔相邻声明或逻辑。
- **L716 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L716 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L717 EN**: Starts a function or method definition for `jacobi_theta4m1tau`.
  - **L717 CN**: 开始定义函数或方法 `jacobi_theta4m1tau`。
- **L718 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L718 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L719 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L719 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L720 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L720 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。

### Lines 721-740 / 第 721-740 行

````cpp
 721:       Policy,
 722:       policies::promote_float<false>,
 723:       policies::promote_double<false>,
 724:       policies::discrete_quantile<>,
 725:       policies::assert_undefined<> >::type forwarding_policy;
 726: 
 727:    static const char* function = "boost::math::jacobi_theta4m1tau<%1%>(%1%)";
 728: 
 729:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4m1tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy()), function);
 730: }
 731: 
 732: template <class T, class U>
 733: inline typename tools::promote_args<T, U>::type jacobi_theta4m1tau(T z, U tau) {
 734:     return jacobi_theta4m1tau(z, tau, policies::policy<>());
 735: }
 736: 
 737: template <class T, class U, class Policy>
 738: inline typename tools::promote_args<T, U>::type jacobi_theta4tau(T z, U tau, const Policy&) {
 739:    BOOST_FPU_EXCEPTION_GUARD
 740:    typedef typename tools::promote_args<T, U>::type result_type;
````
- **L721 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L721 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L722 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L722 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L723 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L723 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L724 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L724 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L725 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L725 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L726 EN**: Blank line separating nearby declarations or logic.
  - **L726 CN**: 空行，用于分隔相邻声明或逻辑。
- **L727 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L727 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L728 EN**: Blank line separating nearby declarations or logic.
  - **L728 CN**: 空行，用于分隔相邻声明或逻辑。
- **L729 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4m1tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy()), function)`.
  - **L729 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4m1tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy()), function)` 从当前函数返回。
- **L730 EN**: Closes the current lexical scope or compound statement.
  - **L730 CN**: 结束当前词法作用域或复合语句块。
- **L731 EN**: Blank line separating nearby declarations or logic.
  - **L731 CN**: 空行，用于分隔相邻声明或逻辑。
- **L732 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L732 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L733 EN**: Starts a function or method definition for `jacobi_theta4m1tau`.
  - **L733 CN**: 开始定义函数或方法 `jacobi_theta4m1tau`。
- **L734 EN**: Returns from the current function with `jacobi_theta4m1tau(z, tau, policies::policy<>())`.
  - **L734 CN**: 以 `jacobi_theta4m1tau(z, tau, policies::policy<>())` 从当前函数返回。
- **L735 EN**: Closes the current lexical scope or compound statement.
  - **L735 CN**: 结束当前词法作用域或复合语句块。
- **L736 EN**: Blank line separating nearby declarations or logic.
  - **L736 CN**: 空行，用于分隔相邻声明或逻辑。
- **L737 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L737 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L738 EN**: Starts a function or method definition for `jacobi_theta4tau`.
  - **L738 CN**: 开始定义函数或方法 `jacobi_theta4tau`。
- **L739 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L739 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L740 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L740 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。

### Lines 741-760 / 第 741-760 行

````cpp
 741:    typedef typename policies::normalise<
 742:       Policy,
 743:       policies::promote_float<false>,
 744:       policies::promote_double<false>,
 745:       policies::discrete_quantile<>,
 746:       policies::assert_undefined<> >::type forwarding_policy;
 747: 
 748:    static const char* function = "boost::math::jacobi_theta4tau<%1%>(%1%)";
 749: 
 750:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function);
 751: }
 752: 
 753: template <class T, class U>
 754: inline typename tools::promote_args<T, U>::type jacobi_theta4tau(T z, U tau) {
 755:     return jacobi_theta4tau(z, tau, policies::policy<>());
 756: }
 757: 
 758: template <class T, class U, class Policy>
 759: inline typename tools::promote_args<T, U>::type jacobi_theta4m1(T z, U q, const Policy&) {
 760:    BOOST_FPU_EXCEPTION_GUARD
````
- **L741 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L741 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L742 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L742 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L743 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L743 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L744 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L744 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L745 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L745 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L746 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L746 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L747 EN**: Blank line separating nearby declarations or logic.
  - **L747 CN**: 空行，用于分隔相邻声明或逻辑。
- **L748 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L748 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L749 EN**: Blank line separating nearby declarations or logic.
  - **L749 CN**: 空行，用于分隔相邻声明或逻辑。
- **L750 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)`.
  - **L750 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4tau_imp(static_cast<result_type>(z), static_cast<result_type>(tau), forwarding_policy(), function), function)` 从当前函数返回。
- **L751 EN**: Closes the current lexical scope or compound statement.
  - **L751 CN**: 结束当前词法作用域或复合语句块。
- **L752 EN**: Blank line separating nearby declarations or logic.
  - **L752 CN**: 空行，用于分隔相邻声明或逻辑。
- **L753 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L753 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L754 EN**: Starts a function or method definition for `jacobi_theta4tau`.
  - **L754 CN**: 开始定义函数或方法 `jacobi_theta4tau`。
- **L755 EN**: Returns from the current function with `jacobi_theta4tau(z, tau, policies::policy<>())`.
  - **L755 CN**: 以 `jacobi_theta4tau(z, tau, policies::policy<>())` 从当前函数返回。
- **L756 EN**: Closes the current lexical scope or compound statement.
  - **L756 CN**: 结束当前词法作用域或复合语句块。
- **L757 EN**: Blank line separating nearby declarations or logic.
  - **L757 CN**: 空行，用于分隔相邻声明或逻辑。
- **L758 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L758 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L759 EN**: Starts a function or method definition for `jacobi_theta4m1`.
  - **L759 CN**: 开始定义函数或方法 `jacobi_theta4m1`。
- **L760 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L760 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。

### Lines 761-780 / 第 761-780 行

````cpp
 761:    typedef typename tools::promote_args<T, U>::type result_type;
 762:    typedef typename policies::normalise<
 763:       Policy,
 764:       policies::promote_float<false>,
 765:       policies::promote_double<false>,
 766:       policies::discrete_quantile<>,
 767:       policies::assert_undefined<> >::type forwarding_policy;
 768: 
 769:    static const char* function = "boost::math::jacobi_theta4m1<%1%>(%1%)";
 770: 
 771:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4m1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function);
 772: }
 773: 
 774: template <class T, class U>
 775: inline typename tools::promote_args<T, U>::type jacobi_theta4m1(T z, U q) {
 776:     return jacobi_theta4m1(z, q, policies::policy<>());
 777: }
 778: 
 779: template <class T, class U, class Policy>
 780: inline typename tools::promote_args<T, U>::type jacobi_theta4(T z, U q, const Policy&) {
````
- **L761 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L761 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L762 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L762 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L763 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L763 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L764 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L764 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L765 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L765 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L766 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L766 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L767 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L767 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L768 EN**: Blank line separating nearby declarations or logic.
  - **L768 CN**: 空行，用于分隔相邻声明或逻辑。
- **L769 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L769 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L770 EN**: Blank line separating nearby declarations or logic.
  - **L770 CN**: 空行，用于分隔相邻声明或逻辑。
- **L771 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4m1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)`.
  - **L771 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4m1_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)` 从当前函数返回。
- **L772 EN**: Closes the current lexical scope or compound statement.
  - **L772 CN**: 结束当前词法作用域或复合语句块。
- **L773 EN**: Blank line separating nearby declarations or logic.
  - **L773 CN**: 空行，用于分隔相邻声明或逻辑。
- **L774 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L774 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L775 EN**: Starts a function or method definition for `jacobi_theta4m1`.
  - **L775 CN**: 开始定义函数或方法 `jacobi_theta4m1`。
- **L776 EN**: Returns from the current function with `jacobi_theta4m1(z, q, policies::policy<>())`.
  - **L776 CN**: 以 `jacobi_theta4m1(z, q, policies::policy<>())` 从当前函数返回。
- **L777 EN**: Closes the current lexical scope or compound statement.
  - **L777 CN**: 结束当前词法作用域或复合语句块。
- **L778 EN**: Blank line separating nearby declarations or logic.
  - **L778 CN**: 空行，用于分隔相邻声明或逻辑。
- **L779 EN**: Introduces template parameters or specialization context: `template <class T, class U, class Policy>`.
  - **L779 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U, class Policy>`。
- **L780 EN**: Starts a function or method definition for `jacobi_theta4`.
  - **L780 CN**: 开始定义函数或方法 `jacobi_theta4`。

### Lines 781-800 / 第 781-800 行

````cpp
 781:    BOOST_FPU_EXCEPTION_GUARD
 782:    typedef typename tools::promote_args<T, U>::type result_type;
 783:    typedef typename policies::normalise<
 784:       Policy,
 785:       policies::promote_float<false>,
 786:       policies::promote_double<false>,
 787:       policies::discrete_quantile<>,
 788:       policies::assert_undefined<> >::type forwarding_policy;
 789: 
 790:    static const char* function = "boost::math::jacobi_theta4<%1%>(%1%)";
 791: 
 792:    return policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function);
 793: }
 794: 
 795: template <class T, class U>
 796: inline typename tools::promote_args<T, U>::type jacobi_theta4(T z, U q) {
 797:     return jacobi_theta4(z, q, policies::policy<>());
 798: }
 799: 
 800: }}
````
- **L781 EN**: Continues the surrounding expression or declaration: `BOOST_FPU_EXCEPTION_GUARD`.
  - **L781 CN**: 继续构造周围的表达式或声明：`BOOST_FPU_EXCEPTION_GUARD`。
- **L782 EN**: Introduces a legacy type alias or function typedef: `typedef typename tools::promote_args<T, U>::type result_type;`.
  - **L782 CN**: 引入传统类型别名或函数 typedef：`typedef typename tools::promote_args<T, U>::type result_type;`。
- **L783 EN**: Introduces a legacy type alias or function typedef: `typedef typename policies::normalise<`.
  - **L783 CN**: 引入传统类型别名或函数 typedef：`typedef typename policies::normalise<`。
- **L784 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `Policy,`.
  - **L784 CN**: 继续一个多行参数列表、初始化器或聚合项：`Policy,`。
- **L785 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_float<false>,`.
  - **L785 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_float<false>,`。
- **L786 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::promote_double<false>,`.
  - **L786 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::promote_double<false>,`。
- **L787 EN**: Continues a multi-line argument list, initializer, or aggregate entry: `policies::discrete_quantile<>,`.
  - **L787 CN**: 继续一个多行参数列表、初始化器或聚合项：`policies::discrete_quantile<>,`。
- **L788 EN**: Executes a standalone statement or declaration: `policies::assert_undefined<> >::type forwarding_policy;`.
  - **L788 CN**: 执行一条独立语句或声明：`policies::assert_undefined<> >::type forwarding_policy;`。
- **L789 EN**: Blank line separating nearby declarations or logic.
  - **L789 CN**: 空行，用于分隔相邻声明或逻辑。
- **L790 EN**: Uses Boost facilities or Boost.Math helpers to build the surrounding numeric logic.
  - **L790 CN**: 使用 Boost 设施或 Boost.Math 辅助逻辑来构建周边数值实现。
- **L791 EN**: Blank line separating nearby declarations or logic.
  - **L791 CN**: 空行，用于分隔相邻声明或逻辑。
- **L792 EN**: Returns from the current function with `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)`.
  - **L792 CN**: 以 `policies::checked_narrowing_cast<result_type, Policy>(jacobi_theta4_imp(static_cast<result_type>(z), static_cast<result_type>(q), forwarding_policy(), function), function)` 从当前函数返回。
- **L793 EN**: Closes the current lexical scope or compound statement.
  - **L793 CN**: 结束当前词法作用域或复合语句块。
- **L794 EN**: Blank line separating nearby declarations or logic.
  - **L794 CN**: 空行，用于分隔相邻声明或逻辑。
- **L795 EN**: Introduces template parameters or specialization context: `template <class T, class U>`.
  - **L795 CN**: 为后续声明引入模板参数或特化上下文：`template <class T, class U>`。
- **L796 EN**: Starts a function or method definition for `jacobi_theta4`.
  - **L796 CN**: 开始定义函数或方法 `jacobi_theta4`。
- **L797 EN**: Returns from the current function with `jacobi_theta4(z, q, policies::policy<>())`.
  - **L797 CN**: 以 `jacobi_theta4(z, q, policies::policy<>())` 从当前函数返回。
- **L798 EN**: Closes the current lexical scope or compound statement.
  - **L798 CN**: 结束当前词法作用域或复合语句块。
- **L799 EN**: Blank line separating nearby declarations or logic.
  - **L799 CN**: 空行，用于分隔相邻声明或逻辑。
- **L800 EN**: Continues the surrounding expression or declaration: `}}`.
  - **L800 CN**: 继续构造周围的表达式或声明：`}}`。

### Lines 801-802 / 第 801-802 行

````cpp
 801: 
 802: #endif
````
- **L801 EN**: Blank line separating nearby declarations or logic.
  - **L801 CN**: 空行，用于分隔相邻声明或逻辑。
- **L802 EN**: Closes the current preprocessor conditional block or header guard.
  - **L802 CN**: 结束当前预处理条件块或头文件保护。

## Key Concepts / 关键概念

- **Special-function numerics / 特殊函数数值计算**:
  - **EN**: Implements or exposes numerically stable special functions such as gamma, elliptic, or transcendental routines.
  - **CN**: 实现或暴露数值稳定的特殊函数，例如 gamma、椭圆积分或超越函数例程。
- **Template genericity / 模板泛型**:
  - **EN**: Uses C++ templates to adapt behavior across types, policies, or compile-time parameters.
  - **CN**: 使用 C++ 模板使行为适配不同类型、策略或编译期参数。
- **Policy customization / 策略定制**:
  - **EN**: Routes behavior through policy objects or compile-time policy choices.
  - **CN**: 通过策略对象或编译期策略选择来路由行为。
- **Assertions and contracts / 断言与契约**:
  - **EN**: Checks preconditions, postconditions, or test outcomes to keep behavior well defined.
  - **CN**: 检查前置条件、后置条件或测试结果，以保持行为有明确定义。
- **Boost integration / Boost 集成**:
  - **EN**: Fits the implementation into Boost naming, configuration, and portability conventions.
  - **CN**: 让实现融入 Boost 的命名、配置与可移植性约定。
- **Dependency layering / 依赖分层**:
  - **EN**: Builds on included headers instead of re-declaring reusable infrastructure.
  - **CN**: 构建在已包含头文件之上，而不是重复声明可复用基础设施。
- **Header contracts / 头文件契约**:
  - **EN**: Provides declarations, inline logic, or macros that other translation units consume.
  - **CN**: 提供供其他编译单元使用的声明、内联逻辑或宏。

## Dependencies / 依赖关系

- **Direct local/internal includes / 直接本地或内部包含**: `boost/math/tools/complex.hpp`, `boost/math/tools/precision.hpp`, `boost/math/tools/promotion.hpp`, `boost/math/policies/error_handling.hpp`, `boost/math/constants/constants.hpp`
- **Dependency categories / 依赖类别**: Boost.Math numeric tool helpers / Boost.Math 数值工具辅助逻辑 (3), Boost.Math policy configuration / Boost.Math 策略配置 (1), Boost.Math numeric constants / Boost.Math 数值常量 (1)

- **EN**: `boost/math/tools/complex.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/complex.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/precision.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/precision.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/tools/promotion.hpp` provides Boost.Math numeric tool helpers.
  - **CN**: `boost/math/tools/promotion.hpp` 提供Boost.Math 数值工具辅助逻辑。
- **EN**: `boost/math/policies/error_handling.hpp` provides Boost.Math policy configuration.
  - **CN**: `boost/math/policies/error_handling.hpp` 提供Boost.Math 策略配置。
- **EN**: `boost/math/constants/constants.hpp` provides Boost.Math numeric constants.
  - **CN**: `boost/math/constants/constants.hpp` 提供Boost.Math 数值常量。
