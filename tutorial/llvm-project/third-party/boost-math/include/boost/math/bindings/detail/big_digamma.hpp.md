# big_digamma.hpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `third-party/boost-math/include/boost/math/bindings/detail/big_digamma.hpp`
- **Repository / 仓库**: `/root/xw/llvm-project/`
- **Purpose (EN)**: This Boost.Math header provides reusable support for big digamma.
- **作用（中文）**: 此 Boost.Math 头文件为 big digamma 提供可复用支持。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18 / 第 1-18 行
~~~cpp
   1: //  (C) Copyright John Maddock 2006-8.
   2: //  Use, modification and distribution are subject to the
   3: //  Boost Software License, Version 1.0. (See accompanying file
   4: //  LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)
   5: 
   6: #ifndef BOOST_MATH_NTL_DIGAMMA
   7: #define BOOST_MATH_NTL_DIGAMMA
   8: 
   9: #include <boost/math/tools/rational.hpp>
  10: #include <boost/math/tools/config.hpp>
  11: #include <boost/math/policies/error_handling.hpp>
  12: #include <boost/math/constants/constants.hpp>
  13: #include <boost/math/tools/big_constant.hpp>
  14: 
  15: namespace boost{ namespace math{ namespace detail{
  16: 
  17: template <class T>
  18: T big_digamma_helper(T x)
~~~
- **EN:** This block imports dependencies such as boost/math/tools/rational.hpp, boost/math/tools/config.hpp, boost/math/policies/error_handling.hpp, ... so the surrounding code can use external declarations. It establishes an inclusion guard so the header is processed only once per translation unit. The code enters namespace scope (boost::math::detail) to keep symbols organized.
- **CN:** 此代码块引入了 boost/math/tools/rational.hpp, boost/math/tools/config.hpp, boost/math/policies/error_handling.hpp, ... 等依赖，使周围代码可以使用外部声明。 它建立了防重复包含机制，使头文件在同一翻译单元中只被处理一次。 代码进入命名空间作用域（boost::math::detail），以保持符号组织清晰。

### Lines 19-36 / 第 19-36 行
~~~cpp
  19: {
  20:       static const T P[61] = {
  21:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6660133691143982067148122682345055274952e81),
  22:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6365271516829242456324234577164675383137e81),
  23:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2991038873096202943405966144203628966976e81),
  24:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9211116495503170498076013367421231351115e80),
  25:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2090792764676090716286400360584443891749e80),
  26:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3730037777359591428226035156377978092809e79),
  27:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5446396536956682043376492370432031543834e78),
  28:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6692523966335177847425047827449069256345e77),
  29:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7062543624100864681625612653756619116848e76),
  30:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6499914905966283735005256964443226879158e75),
  31:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5280364564853225211197557708655426736091e74),
  32:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3823205608981176913075543599005095206953e73),
  33:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2486733714214237704739129972671154532415e72),
  34:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1462562139602039577983434547171318011675e71),
  35:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7821169065036815012381267259559910324285e69),
  36:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3820552182348155468636157988764435365078e68),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 37-54 / 第 37-54 行
~~~cpp
  37:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1711618296983598244658239925535632505062e67),
  38:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7056661618357643731419080738521475204245e65),
  39:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2685246896473614017356264531791459936036e64),
  40:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9455168125599643085283071944864977592391e62),
  41:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3087541626972538362237309145177486236219e61),
  42:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9367928873352980208052601301625005737407e59),
  43:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2645306130689794942883818547314327466007e58),
  44:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6961815141171454309161007351079576190079e56),
  45:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1709637824471794552313802669803885946843e55),
  46:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3921553258481531526663112728778759311158e53),
  47:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.8409006354449988687714450897575728228696e51),
  48:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1686755204461325935742097669030363344927e50),
  49:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3166653542877070999007425197729038754254e48),
  50:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5566029092358215049069560272835654229637e46),
  51:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9161766287916328133080586672953875116242e44),
  52:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1412317772330871298317974693514430627922000),
  53:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 20387991986727877473732570146112459874790),
  54:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 275557928713904105182512535678580359839.3),
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 55-72 / 第 55-72 行
~~~cpp
  55:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3485719851040516559072031256589598330.723),
  56:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 41247046743564028399938106707656877.40859),
  57:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 456274078125709314602601667471879.0147312),
  58:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 4714450683242899367025707077155.310613012),
  59:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 45453933537925041680009544258.75073849996),
  60:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 408437900487067278846361972.302331241052),
  61:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3415719344386166273085838.705771571751035),
  62:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 26541502879185876562320.93134691487351145),
  63:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 191261415065918713661.1571433274648417668),
  64:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1275349770108718421.645275944284937551702),
  65:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 7849171120971773.318910987434906905704272),
  66:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 44455946386549.80866460312682983576538056),
  67:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 230920362395.3198137186361608905136598046),
  68:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1095700096.240863858624279930600654130254),
  69:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 4727085.467506050153744334085516289728134),
  70:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 18440.75118859447173303252421991479005424),
  71:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 64.62515887799460295677071749181651317052),
  72:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.201851568864688406206528472883512147547),
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 73-90 / 第 73-90 行
~~~cpp
  73:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.0005565091674187978029138500039504078098143),
  74:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1338097668312907986354698683493366559613e-5),
  75:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.276308225077464312820179030238305271638e-8),
  76:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4801582970473168520375942100071070575043e-11),
  77:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6829184144212920949740376186058541800175e-14),
  78:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7634080076358511276617829524639455399182e-17),
  79:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6290035083727140966418512608156646142409e-20),
  80:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.339652245667538733044036638506893821352e-23),
  81:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9017518064256388530773585529891677854909e-27)
  82:       };
  83:       static const T Q[61] = {
  84:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0),
  85:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1386831185456898357379390197203894063459e81),
  86:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6467076379487574703291056110838151259438e81),
  87:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1394967823848615838336194279565285465161e82),
  88:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1872927317344192945218570366455046340458e82),
  89:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1772461045338946243584650759986310355937e82),
  90:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1267294892200258648315971144069595555118e82),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 91-108 / 第 91-108 行
~~~cpp
  91:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7157764838362416821508872117623058626589e81),
  92:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.329447266909948668265277828268378274513e81),
  93:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1264376077317689779509250183194342571207e81),
  94:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4118230304191980787640446056583623228873e80),
  95:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1154393529762694616405952270558316515261e80),
  96:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.281655612889423906125295485693696744275e79),
  97:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6037483524928743102724159846414025482077e78),
  98:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1145927995397835468123576831800276999614e78),
  99:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1938624296151985600348534009382865995154e77),
 100:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.293980925856227626211879961219188406675e76),
 101:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4015574518216966910319562902099567437832e75),
 102:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4961475457509727343545565970423431880907e74),
 103:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5565482348278933960215521991000378896338e73),
 104:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5686112924615820754631098622770303094938e72),
 105:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5305988545844796293285410303747469932856e71),
 106:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4533363413802585060568537458067343491358e70),
 107:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3553932059473516064068322757331575565718e69),
 108:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2561198565218704414618802902533972354203e68),
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 109-126 / 第 109-126 行
~~~cpp
 109:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1699519313292900324098102065697454295572e67),
 110:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1039830160862334505389615281373574959236e66),
 111:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5873082967977428281000961954715372504986e64),
 112:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3065255179030575882202133042549783442446e63),
 113:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1479494813481364701208655943688307245459e62),
 114:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6608150467921598615495180659808895663164e60),
 115:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2732535313770902021791888953487787496976e59),
 116:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1046402297662493314531194338414508049069e58),
 117:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3711375077192882936085049147920021549622e56),
 118:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1219154482883895482637944309702972234576e55),
 119:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3708359374149458741391374452286837880162e53),
 120:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1044095509971707189716913168889769471468e52),
 121:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.271951506225063286130946773813524945052e50),
 122:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6548016291215163843464133978454065823866e48),
 123:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1456062447610542135403751730809295219344e47),
 124:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2986690175077969760978388356833006028929e45),
 125:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 5643149706574013350061247429006443326844000),
 126:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 98047545414467090421964387960743688053480),
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 127-144 / 第 127-144 行
~~~cpp
 127:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1563378767746846395507385099301468978550),
 128:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 22823360528584500077862274918382796495),
 129:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 304215527004115213046601295970388750),
 130:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3690289075895685793844344966820325),
 131:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 40584512015702371433911456606050),
 132:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 402834190897282802772754873905),
 133:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3589522158493606918146495750),
 134:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 28530557707503483723634725),
 135:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 200714561335055753000730),
 136:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1237953783437761888641),
 137:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 6614698701445762950),
 138:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 30155495647727505),
 139:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 114953256021450),
 140:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 356398020013),
 141:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 863113950),
 142:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1531345),
 143:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1770),
 144:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1)
~~~
- **EN:** Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 145-162 / 第 145-162 行
~~~cpp
 145:       };
 146:       static const T PD[60] = {
 147:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6365271516829242456324234577164675383137e81),
 148:          2*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2991038873096202943405966144203628966976e81),
 149:          3*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9211116495503170498076013367421231351115e80),
 150:          4*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2090792764676090716286400360584443891749e80),
 151:          5*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3730037777359591428226035156377978092809e79),
 152:          6*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5446396536956682043376492370432031543834e78),
 153:          7*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6692523966335177847425047827449069256345e77),
 154:          8*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7062543624100864681625612653756619116848e76),
 155:          9*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6499914905966283735005256964443226879158e75),
 156:          10*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5280364564853225211197557708655426736091e74),
 157:          11*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3823205608981176913075543599005095206953e73),
 158:          12*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2486733714214237704739129972671154532415e72),
 159:          13*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1462562139602039577983434547171318011675e71),
 160:          14*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7821169065036815012381267259559910324285e69),
 161:          15*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3820552182348155468636157988764435365078e68),
 162:          16*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1711618296983598244658239925535632505062e67),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 163-180 / 第 163-180 行
~~~cpp
 163:          17*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7056661618357643731419080738521475204245e65),
 164:          18*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2685246896473614017356264531791459936036e64),
 165:          19*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9455168125599643085283071944864977592391e62),
 166:          20*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3087541626972538362237309145177486236219e61),
 167:          21*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9367928873352980208052601301625005737407e59),
 168:          22*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2645306130689794942883818547314327466007e58),
 169:          23*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6961815141171454309161007351079576190079e56),
 170:          24*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1709637824471794552313802669803885946843e55),
 171:          25*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3921553258481531526663112728778759311158e53),
 172:          26*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.8409006354449988687714450897575728228696e51),
 173:          27*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1686755204461325935742097669030363344927e50),
 174:          28*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3166653542877070999007425197729038754254e48),
 175:          29*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5566029092358215049069560272835654229637e46),
 176:          30*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9161766287916328133080586672953875116242e44),
 177:          31*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1412317772330871298317974693514430627922000),
 178:          32*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 20387991986727877473732570146112459874790),
 179:          33*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 275557928713904105182512535678580359839.3),
 180:          34*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3485719851040516559072031256589598330.723),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 181-198 / 第 181-198 行
~~~cpp
 181:          35*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 41247046743564028399938106707656877.40859),
 182:          36*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 456274078125709314602601667471879.0147312),
 183:          37*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 4714450683242899367025707077155.310613012),
 184:          38*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 45453933537925041680009544258.75073849996),
 185:          39*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 408437900487067278846361972.302331241052),
 186:          40*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3415719344386166273085838.705771571751035),
 187:          41*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 26541502879185876562320.93134691487351145),
 188:          42*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 191261415065918713661.1571433274648417668),
 189:          43*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1275349770108718421.645275944284937551702),
 190:          44*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 7849171120971773.318910987434906905704272),
 191:          45*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 44455946386549.80866460312682983576538056),
 192:          46*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 230920362395.3198137186361608905136598046),
 193:          47*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1095700096.240863858624279930600654130254),
 194:          48*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 4727085.467506050153744334085516289728134),
 195:          49*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 18440.75118859447173303252421991479005424),
 196:          50*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 64.62515887799460295677071749181651317052),
 197:          51*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.201851568864688406206528472883512147547),
 198:          52*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.0005565091674187978029138500039504078098143),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 199-216 / 第 199-216 行
~~~cpp
 199:          53*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1338097668312907986354698683493366559613e-5),
 200:          54*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.276308225077464312820179030238305271638e-8),
 201:          55*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4801582970473168520375942100071070575043e-11),
 202:          56*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6829184144212920949740376186058541800175e-14),
 203:          57*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7634080076358511276617829524639455399182e-17),
 204:          58*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6290035083727140966418512608156646142409e-20),
 205:          59*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.339652245667538733044036638506893821352e-23),
 206:          60*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.9017518064256388530773585529891677854909e-27)
 207:       };
 208:       static const T QD[60] = {
 209:          BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1386831185456898357379390197203894063459e81),
 210:          2*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6467076379487574703291056110838151259438e81),
 211:          3*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1394967823848615838336194279565285465161e82),
 212:          4*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1872927317344192945218570366455046340458e82),
 213:          5*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1772461045338946243584650759986310355937e82),
 214:          6*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1267294892200258648315971144069595555118e82),
 215:          7*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.7157764838362416821508872117623058626589e81),
 216:          8*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.329447266909948668265277828268378274513e81),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 217-234 / 第 217-234 行
~~~cpp
 217:          9*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1264376077317689779509250183194342571207e81),
 218:          10*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4118230304191980787640446056583623228873e80),
 219:          11*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1154393529762694616405952270558316515261e80),
 220:          12*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.281655612889423906125295485693696744275e79),
 221:          13*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6037483524928743102724159846414025482077e78),
 222:          14*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1145927995397835468123576831800276999614e78),
 223:          15*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1938624296151985600348534009382865995154e77),
 224:          16*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.293980925856227626211879961219188406675e76),
 225:          17*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4015574518216966910319562902099567437832e75),
 226:          18*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4961475457509727343545565970423431880907e74),
 227:          19*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5565482348278933960215521991000378896338e73),
 228:          20*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5686112924615820754631098622770303094938e72),
 229:          21*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5305988545844796293285410303747469932856e71),
 230:          22*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.4533363413802585060568537458067343491358e70),
 231:          23*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3553932059473516064068322757331575565718e69),
 232:          24*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2561198565218704414618802902533972354203e68),
 233:          25*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1699519313292900324098102065697454295572e67),
 234:          26*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1039830160862334505389615281373574959236e66),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 235-252 / 第 235-252 行
~~~cpp
 235:          27*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.5873082967977428281000961954715372504986e64),
 236:          28*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3065255179030575882202133042549783442446e63),
 237:          29*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1479494813481364701208655943688307245459e62),
 238:          30*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6608150467921598615495180659808895663164e60),
 239:          31*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2732535313770902021791888953487787496976e59),
 240:          32*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1046402297662493314531194338414508049069e58),
 241:          33*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3711375077192882936085049147920021549622e56),
 242:          34*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1219154482883895482637944309702972234576e55),
 243:          35*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.3708359374149458741391374452286837880162e53),
 244:          36*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1044095509971707189716913168889769471468e52),
 245:          37*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.271951506225063286130946773813524945052e50),
 246:          38*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.6548016291215163843464133978454065823866e48),
 247:          39*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.1456062447610542135403751730809295219344e47),
 248:          40*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 0.2986690175077969760978388356833006028929e45),
 249:          41*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 5643149706574013350061247429006443326844000),
 250:          42*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 98047545414467090421964387960743688053480),
 251:          43*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1563378767746846395507385099301468978550),
 252:          44*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 22823360528584500077862274918382796495),
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 253-270 / 第 253-270 行
~~~cpp
 253:          45*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 304215527004115213046601295970388750),
 254:          46*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3690289075895685793844344966820325),
 255:          47*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 40584512015702371433911456606050),
 256:          48*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 402834190897282802772754873905),
 257:          49*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 3589522158493606918146495750),
 258:          50*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 28530557707503483723634725),
 259:          51*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 200714561335055753000730),
 260:          52*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1237953783437761888641),
 261:          53*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 6614698701445762950),
 262:          54*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 30155495647727505),
 263:          55*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 114953256021450),
 264:          56*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 356398020013),
 265:          57*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 863113950),
 266:          58*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1531345),
 267:          59*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1770),
 268:          60*BOOST_MATH_BIG_CONSTANT(T, boost::math::tools::numeric_traits<T>::digits, 1)
 269:       };
 270:    static const double g = 63.192152;
~~~
- **EN:** Many lines here are numeric constants or coefficient tables that feed the surrounding algorithm. Macros in this range encode portability hooks, API decoration, or registration boilerplate.
- **CN:** 这里的许多行是数值常量或系数表，用来支撑周围的算法。 此范围中的宏用于表达可移植性钩子、API 修饰或注册样板。

### Lines 271-288 / 第 271-288 行
~~~cpp
 271: 
 272:    T zgh = x + g - 0.5;
 273: 
 274:    T result = (x - 0.5) / zgh;
 275:    result += log(zgh);
 276:    result += tools::evaluate_polynomial(PD, x) / tools::evaluate_polynomial(P, x);
 277:    result -= tools::evaluate_polynomial(QD, x) / tools::evaluate_polynomial(Q, x);
 278:    result -= 1;
 279: 
 280:    return result;
 281: }
 282: 
 283: template <class T>
 284: T big_digamma(T x)
 285: {
 286:    BOOST_MATH_STD_USING
 287: 
 288:    if(x < 0)
~~~
- **EN:** Templates in this range make the implementation reusable across types, precisions, or policies. It introduces the class `T` as part of the file's main abstraction. This range declares or defines callable logic such as log, tools::evaluate_polynomial.
- **CN:** 此范围中的模板让实现可在不同类型、精度或策略之间复用。 它引入了 class `T`，作为该文件核心抽象的一部分。 此范围声明或定义了可调用逻辑，例如 log, tools::evaluate_polynomial。

### Lines 289-297 / 第 289-297 行
~~~cpp
 289:    {
 290:       return big_digamma_helper(static_cast<T>(1-x)) + constants::pi<T>() / tan(constants::pi<T>() * (1-x));
 291:    }
 292:    return big_digamma_helper(x);
 293: }
 294: 
 295: }}}
 296: 
 297: #endif // include guard
~~~
- **EN:** Return statements hand the computed result or status back to the caller.
- **CN:** return 语句会把计算结果或状态返回给调用方。

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
- **Included headers / 包含头文件**: `boost/math/tools/rational.hpp, boost/math/tools/config.hpp, boost/math/policies/error_handling.hpp, boost/math/constants/constants.hpp, boost/math/tools/big_constant.hpp`
- **Namespaces / 命名空间**: `boost, math, detail`
- **Library context / 所属库上下文**: Boost.Math numeric traits, policies, and helper layers. / Boost.Math 的数值 traits、策略与辅助层。
- **Macro/config layer / 宏与配置层**: Uses compile-time macros to adapt visibility, warnings, or feature switches. / 使用编译期宏适配可见性、警告或特性开关。
- **Representative symbols / 代表性符号**: `log, tools::evaluate_polynomial`
