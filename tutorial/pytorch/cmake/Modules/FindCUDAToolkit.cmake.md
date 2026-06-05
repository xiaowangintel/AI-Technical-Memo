# FindCUDAToolkit.cmake — Documentation Analysis / 文档分析

## Source / 来源

- **File / 文件**: `cmake/Modules/FindCUDAToolkit.cmake`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines a CMake discovery/configuration module that locates external dependencies or build capabilities. Dependency discovery and platform-specific probing are central themes in the file. The opening comment frames the file as: "This module is back-ported from CMake 3.17 and above to work with CMake 3.10."
- **Purpose (CN)**: 定义一个 CMake 发现/配置模块，用于定位外部依赖或构建能力。 依赖发现与平台特定探测是该文件的核心主题。 开头注释将该文件概括为：“This module is back-ported from CMake 3.17 and above to work with CMake 3.10”。

## Content Analysis / 内容分析

### Lines 1-16 / 第 1-16 行

```cmake

# This module is back-ported from CMake 3.17 and above to work with CMake 3.10

# Distributed under the OSI-approved BSD 3-Clause License.  See accompanying
# file Copyright.txt or https://cmake.org/licensing for details.

#[=======================================================================[.rst:
FindCUDAToolkit
---------------

.. versionadded:: 3.17

This script locates the NVIDIA CUDA toolkit and the associated libraries, but
does not require the ``CUDA`` language be enabled for a given project. This
module does not search for the NVIDIA CUDA Samples.

```

- **EN:** This chunk introduces sections such as This module is back-ported from CMake 3.17 and above to work with CMake 3.10, Distributed under the OSI-approved BSD 3-Clause License.  See accompanying, file Copyright.txt or https://cmake.org/licensing for details., [=======================================================================[.rst:, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 This module is back-ported from CMake 3.17 and above to work with CMake 3.10、Distributed under the OSI-approved BSD 3-Clause License.  See accompanying、file Copyright.txt or https://cmake.org/licensing for details.、[=======================================================================[.rst: 等标题组织周边说明或配置。

### Lines 17-33 / 第 17-33 行

```cmake
.. versionadded:: 3.19
  QNX support.

Search Behavior
^^^^^^^^^^^^^^^

The CUDA Toolkit search behavior uses the following order:

1. If the ``CUDA`` language has been enabled we will use the directory
   containing the compiler as the first search location for ``nvcc``.

2. If the ``CUDAToolkit_ROOT`` cmake configuration variable (e.g.,
   ``-DCUDAToolkit_ROOT=/some/path``) *or* environment variable is defined, it
   will be searched.  If both an environment variable **and** a
   configuration variable are specified, the *configuration* variable takes
   precedence.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 34-49 / 第 34-49 行

```cmake
   The directory specified here must be such that the executable ``nvcc`` or
   the appropriate ``version.txt`` file can be found underneath the specified
   directory.

3. If the CUDA_PATH environment variable is defined, it will be searched
   for ``nvcc``.

4. The user's path is searched for ``nvcc`` using :command:`find_program`.  If
   this is found, no subsequent search attempts are performed.  Users are
   responsible for ensuring that the first ``nvcc`` to show up in the path is
   the desired path in the event that multiple CUDA Toolkits are installed.

5. On Unix systems, if the symbolic link ``/usr/local/cuda`` exists, this is
   used.  No subsequent search attempts are performed.  No default symbolic link
   location exists for the Windows platform.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 50-67 / 第 50-67 行

```cmake
6. The platform specific default install locations are searched.  If exactly one
   candidate is found, this is used.  The default CUDA Toolkit install locations
   searched are:

   +-------------+-------------------------------------------------------------+
   | Platform    | Search Pattern                                              |
   +=============+=============================================================+
   | macOS       | ``/Developer/NVIDIA/CUDA-X.Y``                              |
   +-------------+-------------------------------------------------------------+
   | Other Unix  | ``/usr/local/cuda-X.Y``                                     |
   +-------------+-------------------------------------------------------------+
   | Windows     | ``C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\vX.Y`` |
   +-------------+-------------------------------------------------------------+

   Where ``X.Y`` would be a specific version of the CUDA Toolkit, such as
   ``/usr/local/cuda-9.0`` or
   ``C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v9.0``

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 68-83 / 第 68-83 行

```cmake
   .. note::

       When multiple CUDA Toolkits are installed in the default location of a
       system(e.g., both ``/usr/local/cuda-9.0`` and ``/usr/local/cuda-10.0``
       exist but the ``/usr/local/cuda`` symbolic link does **not** exist), this
       package is marked as **not** found.

       There are too many factors involved in making an automatic decision in
       the presence of multiple CUDA Toolkits being installed.  In this
       situation, users are encouraged to either (1) set ``CUDAToolkit_ROOT`` or
       (2) ensure that the correct ``nvcc`` executable shows up in ``$PATH`` for
       :command:`find_program` to find.

Arguments
^^^^^^^^^

```

- **EN:** CMake commands like system drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 system 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 84-99 / 第 84-99 行

```cmake
``[<version>]``
    The ``[<version>]`` argument requests a version with which the package found
    should be compatible. See :ref:`find_package version format <FIND_PACKAGE_VERSION_FORMAT>`
    for more details.

Options
^^^^^^^

``REQUIRED``
    If specified, configuration will error if a suitable CUDA Toolkit is not
    found.

``QUIET``
    If specified, the search for a suitable CUDA Toolkit will not produce any
    messages.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 100-115 / 第 100-115 行

```cmake
``EXACT``
    If specified, the CUDA Toolkit is considered found only if the exact
    ``VERSION`` specified is recovered.

Imported targets
^^^^^^^^^^^^^^^^

An :ref:`imported target <Imported targets>` named ``CUDA::toolkit`` is provided.

This module defines :prop_tgt:`IMPORTED` targets for each
of the following libraries that are part of the CUDAToolkit:

- :ref:`CUDA Runtime Library<cuda_toolkit_rt_lib>`
- :ref:`CUDA Driver Library<cuda_toolkit_driver_lib>`
- :ref:`cuBLAS<cuda_toolkit_cuBLAS>`
- :ref:`cuFFT<cuda_toolkit_cuFFT>`
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 116-131 / 第 116-131 行

```cmake
- :ref:`cuRAND<cuda_toolkit_cuRAND>`
- :ref:`cuSOLVER<cuda_toolkit_cuSOLVER>`
- :ref:`cuSPARSE<cuda_toolkit_cuSPARSE>`
- :ref:`cuPTI<cuda_toolkit_cupti>`
- :ref:`NPP<cuda_toolkit_NPP>`
- :ref:`nvBLAS<cuda_toolkit_nvBLAS>`
- :ref:`nvGRAPH<cuda_toolkit_nvGRAPH>`
- :ref:`nvJPEG<cuda_toolkit_nvJPEG>`
- :ref:`nvidia-ML<cuda_toolkit_nvML>`
- :ref:`nvRTC<cuda_toolkit_nvRTC>`
- :ref:`nvToolsExt<cuda_toolkit_nvToolsExt>`
- :ref:`OpenCL<cuda_toolkit_opencl>`
- :ref:`cuLIBOS<cuda_toolkit_cuLIBOS>`

.. _`cuda_toolkit_rt_lib`:

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 132-147 / 第 132-147 行

```cmake
CUDA Runtime Library
""""""""""""""""""""

The CUDA Runtime library (cudart) are what most applications will typically
need to link against to make any calls such as `cudaMalloc`, and `cudaFree`.

Targets Created:

- ``CUDA::cudart``
- ``CUDA::cudart_static``

.. _`cuda_toolkit_driver_lib`:

CUDA Driver Library
""""""""""""""""""""

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 148-163 / 第 148-163 行

```cmake
The CUDA Driver library (cuda) are used by applications that use calls
such as `cuMemAlloc`, and `cuMemFree`.

Targets Created:

- ``CUDA::cuda_driver``

.. _`cuda_toolkit_cuBLAS`:

cuBLAS
""""""

The `cuBLAS <https://docs.nvidia.com/cuda/cublas/index.html>`_ library.

Targets Created:

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 164-179 / 第 164-179 行

```cmake
- ``CUDA::cublas``
- ``CUDA::cublas_static``
- ``CUDA::cublasLt`` starting in CUDA 10.1
- ``CUDA::cublasLt_static`` starting in CUDA 10.1

.. _`cuda_toolkit_cuFFT`:

cuFFT
"""""

The `cuFFT <https://docs.nvidia.com/cuda/cufft/index.html>`_ library.

Targets Created:

- ``CUDA::cufft``
- ``CUDA::cufftw``
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 180-195 / 第 180-195 行

```cmake
- ``CUDA::cufft_static``
- ``CUDA::cufft_static_nocallback`` starting in CUDA 9.2, requires CMake 3.23+
- ``CUDA::cufftw_static``

cuRAND
""""""

The `cuRAND <https://docs.nvidia.com/cuda/curand/index.html>`_ library.

Targets Created:

- ``CUDA::curand``
- ``CUDA::curand_static``

.. _`cuda_toolkit_cuSOLVER`:

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 196-212 / 第 196-212 行

```cmake
cuSOLVER
""""""""

The `cuSOLVER <https://docs.nvidia.com/cuda/cusolver/index.html>`_ library.

Targets Created:

- ``CUDA::cusolver``
- ``CUDA::cusolver_static``

.. _`cuda_toolkit_cuSPARSE`:

cuSPARSE
""""""""

The `cuSPARSE <https://docs.nvidia.com/cuda/cusparse/index.html>`_ library.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 213-228 / 第 213-228 行

```cmake
Targets Created:

- ``CUDA::cusparse``
- ``CUDA::cusparse_static``

.. _`cuda_toolkit_cupti`:

cupti
"""""

The `NVIDIA CUDA Profiling Tools Interface <https://developer.nvidia.com/CUPTI>`_.

Targets Created:

- ``CUDA::cupti``
- ``CUDA::cupti_static``
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 229-244 / 第 229-244 行

```cmake

.. _`cuda_toolkit_NPP`:

NPP
"""

The `NPP <https://docs.nvidia.com/cuda/npp/index.html>`_ libraries.

Targets Created:

- `nppc`:

  - ``CUDA::nppc``
  - ``CUDA::nppc_static``

- `nppial`: Arithmetic and logical operation functions in `nppi_arithmetic_and_logical_operations.h`
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 245-260 / 第 245-260 行

```cmake

  - ``CUDA::nppial``
  - ``CUDA::nppial_static``

- `nppicc`: Color conversion and sampling functions in `nppi_color_conversion.h`

  - ``CUDA::nppicc``
  - ``CUDA::nppicc_static``

- `nppicom`: JPEG compression and decompression functions in `nppi_compression_functions.h`
  Removed starting in CUDA 11.0, use :ref:`nvJPEG<cuda_toolkit_nvJPEG>` instead.

  - ``CUDA::nppicom``
  - ``CUDA::nppicom_static``

- `nppidei`: Data exchange and initialization functions in `nppi_data_exchange_and_initialization.h`
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 261-276 / 第 261-276 行

```cmake

  - ``CUDA::nppidei``
  - ``CUDA::nppidei_static``

- `nppif`: Filtering and computer vision functions in `nppi_filter_functions.h`

  - ``CUDA::nppif``
  - ``CUDA::nppif_static``

- `nppig`: Geometry transformation functions found in `nppi_geometry_transforms.h`

  - ``CUDA::nppig``
  - ``CUDA::nppig_static``

- `nppim`: Morphological operation functions found in `nppi_morphological_operations.h`

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 277-292 / 第 277-292 行

```cmake
  - ``CUDA::nppim``
  - ``CUDA::nppim_static``

- `nppist`: Statistics and linear transform in `nppi_statistics_functions.h` and `nppi_linear_transforms.h`

  - ``CUDA::nppist``
  - ``CUDA::nppist_static``

- `nppisu`: Memory support functions in `nppi_support_functions.h`

  - ``CUDA::nppisu``
  - ``CUDA::nppisu_static``

- `nppitc`: Threshold and compare operation functions in `nppi_threshold_and_compare_operations.h`

  - ``CUDA::nppitc``
```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 293-309 / 第 293-309 行

```cmake
  - ``CUDA::nppitc_static``

- `npps`:

  - ``CUDA::npps``
  - ``CUDA::npps_static``

.. _`cuda_toolkit_nvBLAS`:

nvBLAS
""""""

The `nvBLAS <https://docs.nvidia.com/cuda/nvblas/index.html>`_ libraries.
This is a shared library only.

Targets Created:

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 310-325 / 第 310-325 行

```cmake
- ``CUDA::nvblas``

.. _`cuda_toolkit_nvGRAPH`:

nvGRAPH
"""""""

The `nvGRAPH <https://docs.nvidia.com/cuda/nvgraph/index.html>`_ library.
Removed starting in CUDA 11.0

Targets Created:

- ``CUDA::nvgraph``
- ``CUDA::nvgraph_static``


```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 326-343 / 第 326-343 行

```cmake
.. _`cuda_toolkit_nvJPEG`:

nvJPEG
""""""

The `nvJPEG <https://docs.nvidia.com/cuda/nvjpeg/index.html>`_ library.
Introduced in CUDA 10.

Targets Created:

- ``CUDA::nvjpeg``
- ``CUDA::nvjpeg_static``

.. _`cuda_toolkit_nvRTC`:

nvRTC
"""""

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 344-360 / 第 344-360 行

```cmake
The `nvRTC <https://docs.nvidia.com/cuda/nvrtc/index.html>`_ (Runtime Compilation) library.
This is a shared library only.

Targets Created:

- ``CUDA::nvrtc``

.. _`cuda_toolkit_nvml`:

nvidia-ML
"""""""""

The `NVIDIA Management Library <https://developer.nvidia.com/nvidia-management-library-nvml>`_.
This is a shared library only.

Targets Created:

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 361-376 / 第 361-376 行

```cmake
- ``CUDA::nvml``

.. _`cuda_toolkit_nvToolsExt`:

nvToolsExt
""""""""""

The `NVIDIA Tools Extension <https://docs.nvidia.com/gameworks/content/gameworkslibrary/nvtx/nvidia_tools_extension_library_nvtx.htm>`_.
This is a shared library only.

Targets Created:

- ``CUDA::nvToolsExt``

.. _`cuda_toolkit_opencl`:

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 377-396 / 第 377-396 行

```cmake
OpenCL
""""""

The `NVIDIA OpenCL Library <https://developer.nvidia.com/opencl>`_.
This is a shared library only.

Targets Created:

- ``CUDA::OpenCL``

.. _`cuda_toolkit_cuLIBOS`:

cuLIBOS
"""""""

The cuLIBOS library is a backend thread abstraction layer library which is
static only.  The ``CUDA::cublas_static``, ``CUDA::cusparse_static``,
``CUDA::cufft_static``, ``CUDA::curand_static``, and (when implemented) NPP
libraries all automatically have this dependency linked.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 397-412 / 第 397-412 行

```cmake
Target Created:

- ``CUDA::culibos``

**Note**: direct usage of this target by consumers should not be necessary.

.. _`cuda_toolkit_cuRAND`:



Result variables
^^^^^^^^^^^^^^^^

``CUDAToolkit_FOUND``
    A boolean specifying whether or not the CUDA Toolkit was found.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 413-429 / 第 413-429 行

```cmake
``CUDAToolkit_VERSION``
    The exact version of the CUDA Toolkit found (as reported by
    ``nvcc --version`` or ``version.txt``).

``CUDAToolkit_VERSION_MAJOR``
    The major version of the CUDA Toolkit.

``CUDAToolkit_VERSION_MINOR``
    The minor version of the CUDA Toolkit.

``CUDAToolkit_VERSION_PATCH``
    The patch version of the CUDA Toolkit.

``CUDAToolkit_BIN_DIR``
    The path to the CUDA Toolkit library directory that contains the CUDA
    executable ``nvcc``.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 430-448 / 第 430-448 行

```cmake
``CUDAToolkit_INCLUDE_DIRS``
    The path to the CUDA Toolkit ``include`` folder containing the header files
    required to compile a project linking against CUDA.

``CUDAToolkit_LIBRARY_DIR``
    The path to the CUDA Toolkit library directory that contains the CUDA
    Runtime library ``cudart``.

``CUDAToolkit_LIBRARY_ROOT``
    .. versionadded:: 3.18

    The path to the CUDA Toolkit directory containing the nvvm directory and
    version.txt.

``CUDAToolkit_TARGET_DIR``
    The path to the CUDA Toolkit directory including the target architecture
    when cross-compiling. When not cross-compiling this will be equivalent to
    the parent directory of ``CUDAToolkit_BIN_DIR``.

```

- **EN:** This chunk continues the surrounding documentation or configuration by refining local settings, examples, or workflow steps.
- **CN:** 这一段继续补充周边文档或配置，细化局部设置、示例或工作流步骤。

### Lines 449-464 / 第 449-464 行

```cmake
``CUDAToolkit_NVCC_EXECUTABLE``
    The path to the NVIDIA CUDA compiler ``nvcc``.  Note that this path may
    **not** be the same as
    :variable:`CMAKE_CUDA_COMPILER <CMAKE_<LANG>_COMPILER>`.  ``nvcc`` must be
    found to determine the CUDA Toolkit version as well as determining other
    features of the Toolkit.  This variable is set for the convenience of
    modules that depend on this one.


#]=======================================================================]

# NOTE: much of this was simply extracted from FindCUDA.cmake.

#   James Bigler, NVIDIA Corp (nvidia.com - jbigler)
#   Abe Stephens, SCI Institute -- http://www.sci.utah.edu/~abe/FindCuda.html
#
```

- **EN:** This chunk introduces sections such as ]=======================================================================], NOTE: much of this was simply extracted from FindCUDA.cmake., James Bigler, NVIDIA Corp (nvidia.com - jbigler), Abe Stephens, SCI Institute -- http://www.sci.utah.edu/~abe/FindCuda.html, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 ]=======================================================================]、NOTE: much of this was simply extracted from FindCUDA.cmake.、James Bigler, NVIDIA Corp (nvidia.com - jbigler)、Abe Stephens, SCI Institute -- http://www.sci.utah.edu/~abe/FindCuda.html 等标题组织周边说明或配置。

### Lines 465-480 / 第 465-480 行

```cmake
#   Copyright (c) 2008 - 2009 NVIDIA Corporation.  All rights reserved.
#
#   Copyright (c) 2007-2009
#   Scientific Computing and Imaging Institute, University of Utah
#
#   This code is licensed under the MIT License.  See the FindCUDA.cmake script
#   for the text of the license.

# The MIT License
#
# License for the specific language governing rights and limitations under
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
```

- **EN:** This chunk introduces sections such as Copyright (c) 2008 - 2009 NVIDIA Corporation.  All rights reserved., , Copyright (c) 2007-2009, Scientific Computing and Imaging Institute, University of Utah, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Copyright (c) 2008 - 2009 NVIDIA Corporation.  All rights reserved.、、Copyright (c) 2007-2009、Scientific Computing and Imaging Institute, University of Utah 等标题组织周边说明或配置。

### Lines 481-496 / 第 481-496 行

```cmake
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
#
###############################################################################

# The toolkit is located during compiler detection for CUDA and stored in CMakeCUDACompiler.cmake as
```

- **EN:** This chunk introduces sections such as Software is furnished to do so, subject to the following conditions:, , The above copyright notice and this permission notice shall be included, in all copies or substantial portions of the Software., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Software is furnished to do so, subject to the following conditions:、、The above copyright notice and this permission notice shall be included、in all copies or substantial portions of the Software. 等标题组织周边说明或配置。

### Lines 497-513 / 第 497-513 行

```cmake
# CMAKE_CUDA_COMPILER_TOOLKIT_ROOT and CMAKE_CUDA_COMPILER_LIBRARY_ROOT.
# We compute the rest based on those here to avoid re-searching and to avoid finding a possibly
# different installation.
if(CMAKE_CUDA_COMPILER_TOOLKIT_ROOT)
  set(CUDAToolkit_ROOT_DIR "${CMAKE_CUDA_COMPILER_TOOLKIT_ROOT}")
  set(CUDAToolkit_LIBRARY_ROOT "${CMAKE_CUDA_COMPILER_LIBRARY_ROOT}")
  set(CUDAToolkit_VERSION "${CMAKE_CUDA_COMPILER_TOOLKIT_VERSION}")

  if(CUDAToolkit_VERSION MATCHES [=[([0-9]+)\.([0-9]+)\.([0-9]+)]=])
    set(CUDAToolkit_VERSION_MAJOR "${CMAKE_MATCH_1}")
    set(CUDAToolkit_VERSION_MINOR "${CMAKE_MATCH_2}")
    set(CUDAToolkit_VERSION_PATCH "${CMAKE_MATCH_3}")
  endif()
else()
  function(_CUDAToolkit_find_root_dir )
    cmake_parse_arguments(arg "" "" "SEARCH_PATHS;FIND_FLAGS" ${ARGN})

```

- **EN:** This chunk introduces sections such as CMAKE_CUDA_COMPILER_TOOLKIT_ROOT and CMAKE_CUDA_COMPILER_LIBRARY_ROOT., We compute the rest based on those here to avoid re-searching and to avoid finding a possibly, different installation., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 CMAKE_CUDA_COMPILER_TOOLKIT_ROOT and CMAKE_CUDA_COMPILER_LIBRARY_ROOT.、We compute the rest based on those here to avoid re-searching and to avoid finding a possibly、different installation. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, endif, else, function, cmake_parse_arguments drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、endif、else、function、cmake_parse_arguments 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 514-530 / 第 514-530 行

```cmake
    if(NOT CUDAToolkit_BIN_DIR)
      if(NOT CUDAToolkit_SENTINEL_FILE)
        find_program(CUDAToolkit_NVCC_EXECUTABLE
          NAMES nvcc nvcc.exe
          PATHS ${arg_SEARCH_PATHS}
          ${arg_FIND_FLAGS}
        )
      endif()

      if(NOT CUDAToolkit_NVCC_EXECUTABLE)
        find_file(CUDAToolkit_SENTINEL_FILE
          NAMES version.txt
          PATHS ${arg_SEARCH_PATHS}
          NO_DEFAULT_PATH
        )
      endif()

```

- **EN:** CMake commands like if, find_program, endif, find_file drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_program、endif、find_file 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 531-547 / 第 531-547 行

```cmake
      if(EXISTS "${CUDAToolkit_NVCC_EXECUTABLE}")
        # If NVCC exists  then invoke it to find the toolkit location.
        # This allows us to support wrapper scripts (e.g. ccache or colornvcc), CUDA Toolkit,
        # NVIDIA HPC SDK, and distro's splayed layouts
        execute_process(COMMAND ${CUDAToolkit_NVCC_EXECUTABLE} "-v" "__cmake_determine_cuda"
          OUTPUT_VARIABLE _CUDA_NVCC_OUT ERROR_VARIABLE _CUDA_NVCC_OUT)
        if(_CUDA_NVCC_OUT MATCHES "\\#\\$ TOP=([^\r\n]*)")
          get_filename_component(CUDAToolkit_BIN_DIR "${CMAKE_MATCH_1}/bin" ABSOLUTE)
        else()
          get_filename_component(CUDAToolkit_BIN_DIR "${CUDAToolkit_NVCC_EXECUTABLE}" DIRECTORY)
        endif()
        unset(_CUDA_NVCC_OUT)

        mark_as_advanced(CUDAToolkit_BIN_DIR)
        set(CUDAToolkit_BIN_DIR "${CUDAToolkit_BIN_DIR}" CACHE PATH "" FORCE)
      endif()

```

- **EN:** This chunk introduces sections such as If NVCC exists  then invoke it to find the toolkit location., This allows us to support wrapper scripts (e.g. ccache or colornvcc), CUDA Toolkit,, NVIDIA HPC SDK, and distro's splayed layouts, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 If NVCC exists  then invoke it to find the toolkit location.、This allows us to support wrapper scripts (e.g. ccache or colornvcc), CUDA Toolkit,、NVIDIA HPC SDK, and distro's splayed layouts 等标题组织周边说明或配置。
- **EN:** CMake commands like if, execute_process, get_filename_component, else, endif, unset drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、execute_process、get_filename_component、else、endif、unset 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 548-563 / 第 548-563 行

```cmake
      if(CUDAToolkit_SENTINEL_FILE)
        get_filename_component(CUDAToolkit_BIN_DIR ${CUDAToolkit_SENTINEL_FILE} DIRECTORY ABSOLUTE)
        set(CUDAToolkit_BIN_DIR "${CUDAToolkit_BIN_DIR}/bin")

        set(CUDAToolkit_BIN_DIR "${CUDAToolkit_BIN_DIR}" CACHE PATH "" FORCE)
        mark_as_advanced(CUDAToolkit_BIN_DIR)
      endif()
    endif()

    if(CUDAToolkit_BIN_DIR)
      get_filename_component(CUDAToolkit_ROOT_DIR ${CUDAToolkit_BIN_DIR} DIRECTORY ABSOLUTE)
      set(CUDAToolkit_ROOT_DIR "${CUDAToolkit_ROOT_DIR}" PARENT_SCOPE)
    endif()

  endfunction()

```

- **EN:** CMake commands like if, get_filename_component, set, mark_as_advanced, endif, endfunction drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、get_filename_component、set、mark_as_advanced、endif、endfunction 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 564-580 / 第 564-580 行

```cmake
  # For NVCC we can easily deduce the SDK binary directory from the compiler path.
  if(CMAKE_CUDA_COMPILER_LOADED AND NOT CUDAToolkit_BIN_DIR AND CMAKE_CUDA_COMPILER_ID STREQUAL "NVIDIA")
    get_filename_component(CUDAToolkit_BIN_DIR "${CMAKE_CUDA_COMPILER}" DIRECTORY)
    set(CUDAToolkit_BIN_DIR "${CUDAToolkit_BIN_DIR}" CACHE PATH "")
    # Try language provided path first.
    _CUDAToolkit_find_root_dir(SEARCH_PATHS "${CUDAToolkit_BIN_DIR}" FIND_FLAGS NO_DEFAULT_PATH)
    mark_as_advanced(CUDAToolkit_BIN_DIR)
  endif()

  # Try user provided path
  if(NOT CUDAToolkit_ROOT_DIR AND CUDAToolkit_ROOT)
    _CUDAToolkit_find_root_dir(SEARCH_PATHS "${CUDAToolkit_ROOT}" FIND_FLAGS PATH_SUFFIXES bin NO_DEFAULT_PATH)
  endif()
  if(NOT CUDAToolkit_ROOT_DIR)
    _CUDAToolkit_find_root_dir(FIND_FLAGS PATHS ENV CUDA_PATH PATH_SUFFIXES bin)
  endif()

```

- **EN:** This chunk introduces sections such as For NVCC we can easily deduce the SDK binary directory from the compiler path., Try language provided path first., Try user provided path, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 For NVCC we can easily deduce the SDK binary directory from the compiler path.、Try language provided path first.、Try user provided path 等标题组织周边说明或配置。
- **EN:** CMake commands like if, get_filename_component, set, _CUDAToolkit_find_root_dir, mark_as_advanced, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、get_filename_component、set、_CUDAToolkit_find_root_dir、mark_as_advanced、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 581-609 / 第 581-609 行

```cmake
  # If the user specified CUDAToolkit_ROOT but the toolkit could not be found, this is an error.
  if(NOT CUDAToolkit_ROOT_DIR AND (DEFINED CUDAToolkit_ROOT OR DEFINED ENV{CUDAToolkit_ROOT}))
    # Declare error messages now, print later depending on find_package args.
    set(fail_base "Could not find nvcc executable in path specified by")
    set(cuda_root_fail "${fail_base} CUDAToolkit_ROOT=${CUDAToolkit_ROOT}")
    set(env_cuda_root_fail "${fail_base} environment variable CUDAToolkit_ROOT=$ENV{CUDAToolkit_ROOT}")

    if(CUDAToolkit_FIND_REQUIRED)
      if(DEFINED CUDAToolkit_ROOT)
        message(FATAL_ERROR ${cuda_root_fail})
      elseif(DEFINED ENV{CUDAToolkit_ROOT})
        message(FATAL_ERROR ${env_cuda_root_fail})
      endif()
    else()
      if(NOT CUDAToolkit_FIND_QUIETLY)
        if(DEFINED CUDAToolkit_ROOT)
          message(STATUS ${cuda_root_fail})
        elseif(DEFINED ENV{CUDAToolkit_ROOT})
          message(STATUS ${env_cuda_root_fail})
        endif()
      endif()
      set(CUDAToolkit_FOUND FALSE)
      unset(fail_base)
      unset(cuda_root_fail)
      unset(env_cuda_root_fail)
      return()
    endif()
  endif()

```

- **EN:** This chunk introduces sections such as If the user specified CUDAToolkit_ROOT but the toolkit could not be found, this is an error., Declare error messages now, print later depending on find_package args., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 If the user specified CUDAToolkit_ROOT but the toolkit could not be found, this is an error.、Declare error messages now, print later depending on find_package args. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, message, elseif, endif, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、message、elseif、endif、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 610-629 / 第 610-629 行

```cmake
  # CUDAToolkit_ROOT cmake / env variable not specified, try platform defaults.
  #
  # - Linux: /usr/local/cuda-X.Y
  # - macOS: /Developer/NVIDIA/CUDA-X.Y
  # - Windows: C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\vX.Y
  #
  # We will also search the default symlink location /usr/local/cuda first since
  # if CUDAToolkit_ROOT is not specified, it is assumed that the symlinked
  # directory is the desired location.
  if(NOT CUDAToolkit_ROOT_DIR)
    if(UNIX)
      if(NOT APPLE)
        set(platform_base "/usr/local/cuda-")
      else()
        set(platform_base "/Developer/NVIDIA/CUDA-")
      endif()
    else()
      set(platform_base "C:\\Program Files\\NVIDIA GPU Computing Toolkit\\CUDA\\v")
    endif()

```

- **EN:** This chunk introduces sections such as CUDAToolkit_ROOT cmake / env variable not specified, try platform defaults., , - Linux: /usr/local/cuda-X.Y, - macOS: /Developer/NVIDIA/CUDA-X.Y, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 CUDAToolkit_ROOT cmake / env variable not specified, try platform defaults.、、- Linux: /usr/local/cuda-X.Y、- macOS: /Developer/NVIDIA/CUDA-X.Y 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 630-646 / 第 630-646 行

```cmake
    # Build out a descending list of possible cuda installations, e.g.
    file(GLOB possible_paths "${platform_base}*")
    # Iterate the glob results and create a descending list.
    set(versions)
    foreach(p ${possible_paths})
      # Extract version number from end of string
      string(REGEX MATCH "[0-9][0-9]?\\.[0-9]$" p_version ${p})
      if(IS_DIRECTORY ${p} AND p_version)
        list(APPEND versions ${p_version})
      endif()
    endforeach()

    # Sort numerically in descending order, so we try the newest versions first.
    if(CMAKE_VERSION VERSION_GREATER_EQUAL 3.18)
      list(SORT versions COMPARE NATURAL ORDER DESCENDING)
    elseif(versions)
      # Alphabetical sort here is not ideal but better than nothing
```

- **EN:** This chunk introduces sections such as Build out a descending list of possible cuda installations, e.g., Iterate the glob results and create a descending list., Extract version number from end of string, Sort numerically in descending order, so we try the newest versions first., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Build out a descending list of possible cuda installations, e.g.、Iterate the glob results and create a descending list.、Extract version number from end of string、Sort numerically in descending order, so we try the newest versions first. 等标题组织周边说明或配置。
- **EN:** CMake commands like file, set, foreach, string, if, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 file、set、foreach、string、if、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 647-662 / 第 647-662 行

```cmake
      list(SORT versions)
      list(REVERSE versions)
    endif()

    # With a descending list of versions, populate possible paths to search.
    set(search_paths)
    foreach(v ${versions})
      list(APPEND search_paths "${platform_base}${v}")
    endforeach()

    # Force the global default /usr/local/cuda to the front on Unix.
    if(UNIX)
      list(INSERT search_paths 0 "/usr/local/cuda")
    endif()

    # Now search for the toolkit again using the platform default search paths.
```

- **EN:** This chunk introduces sections such as With a descending list of versions, populate possible paths to search., Force the global default /usr/local/cuda to the front on Unix., Now search for the toolkit again using the platform default search paths., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 With a descending list of versions, populate possible paths to search.、Force the global default /usr/local/cuda to the front on Unix.、Now search for the toolkit again using the platform default search paths. 等标题组织周边说明或配置。
- **EN:** CMake commands like list, endif, set, foreach, endforeach, if drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 list、endif、set、foreach、endforeach、if 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 663-683 / 第 663-683 行

```cmake
    _CUDAToolkit_find_root_dir(SEARCH_PATHS "${search_paths}" FIND_FLAGS PATH_SUFFIXES bin)

    # We are done with these variables now, cleanup for caller.
    unset(platform_base)
    unset(possible_paths)
    unset(versions)
    unset(search_paths)

    if(NOT CUDAToolkit_ROOT_DIR)
      if(CUDAToolkit_FIND_REQUIRED)
        message(FATAL_ERROR "Could not find nvcc, please set CUDAToolkit_ROOT.")
      elseif(NOT CUDAToolkit_FIND_QUIETLY)
        message(STATUS "Could not find nvcc, please set CUDAToolkit_ROOT.")
      endif()

      set(CUDAToolkit_FOUND FALSE)
      return()
    endif()
  endif()
endif()

```

- **EN:** This chunk introduces sections such as We are done with these variables now, cleanup for caller., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 We are done with these variables now, cleanup for caller. 等标题组织周边说明或配置。
- **EN:** CMake commands like _CUDAToolkit_find_root_dir, unset, if, message, elseif, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 _CUDAToolkit_find_root_dir、unset、if、message、elseif、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 684-707 / 第 684-707 行

```cmake
if(NOT CUDAToolkit_BIN_DIR)
  set(CUDAToolkit_BIN_DIR "${CUDAToolkit_ROOT_DIR}/bin")
endif()

if(NOT CUDAToolkit_NVCC_EXECUTABLE)
  set(CUDAToolkit_NVCC_EXECUTABLE "${CUDAToolkit_BIN_DIR}/nvcc${CMAKE_EXECUTABLE_SUFFIX}")
endif()

if(CMAKE_CUDA_COMPILER_TOOLKIT_VERSION)
  set(CUDAToolkit_VERSION "${CMAKE_CUDA_COMPILER_TOOLKIT_VERSION}")
else()
  function(_CUDAToolkit_find_version_file result_variable)
    # We first check for a non-scattered installation to prefer it over a scattered installation.
    if(CUDAToolkit_ROOT AND EXISTS "${CUDAToolkit_ROOT}/version.txt")
      set(${result_variable} "${CUDAToolkit_ROOT}/version.txt" PARENT_SCOPE)
    elseif(CUDAToolkit_ROOT_DIR AND EXISTS "${CUDAToolkit_ROOT_DIR}/version.txt")
      set(${result_variable} "${CUDAToolkit_ROOT_DIR}/version.txt" PARENT_SCOPE)
    elseif(CMAKE_SYSROOT_LINK AND EXISTS "${CMAKE_SYSROOT_LINK}/usr/lib/cuda/version.txt")
      set(${result_variable} "${CMAKE_SYSROOT_LINK}/usr/lib/cuda/version.txt" PARENT_SCOPE)
    elseif(EXISTS "${CMAKE_SYSROOT}/usr/lib/cuda/version.txt")
      set(${result_variable} "${CMAKE_SYSROOT}/usr/lib/cuda/version.txt" PARENT_SCOPE)
    endif()
  endfunction()

```

- **EN:** This chunk introduces sections such as We first check for a non-scattered installation to prefer it over a scattered installation., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 We first check for a non-scattered installation to prefer it over a scattered installation. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, endif, else, function, elseif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、endif、else、function、elseif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 708-727 / 第 708-727 行

```cmake
  _CUDAToolkit_find_version_file( _CUDAToolkit_version_file )
  if(_CUDAToolkit_version_file)
    # CUDAToolkit_LIBRARY_ROOT contains the device library and version file.
    get_filename_component(CUDAToolkit_LIBRARY_ROOT "${_CUDAToolkit_version_file}" DIRECTORY ABSOLUTE)
  endif()
  unset(_CUDAToolkit_version_file)

  if(CUDAToolkit_NVCC_EXECUTABLE AND
     CMAKE_CUDA_COMPILER_VERSION AND
     CUDAToolkit_NVCC_EXECUTABLE STREQUAL CMAKE_CUDA_COMPILER)
    # Need to set these based off the already computed CMAKE_CUDA_COMPILER_VERSION value
    # This if statement will always match, but is used to provide variables for MATCH 1,2,3...
    if(CMAKE_CUDA_COMPILER_VERSION MATCHES [=[([0-9]+)\.([0-9]+)\.([0-9]+)]=])
      set(CUDAToolkit_VERSION_MAJOR "${CMAKE_MATCH_1}")
      set(CUDAToolkit_VERSION_MINOR "${CMAKE_MATCH_2}")
      set(CUDAToolkit_VERSION_PATCH "${CMAKE_MATCH_3}")
      set(CUDAToolkit_VERSION "${CMAKE_CUDA_COMPILER_VERSION}")
    endif()
  elseif(CUDAToolkit_NVCC_EXECUTABLE)
    # Compute the version by invoking nvcc
```

- **EN:** This chunk introduces sections such as CUDAToolkit_LIBRARY_ROOT contains the device library and version file., Need to set these based off the already computed CMAKE_CUDA_COMPILER_VERSION value, This if statement will always match, but is used to provide variables for MATCH 1,2,3..., Compute the version by invoking nvcc, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 CUDAToolkit_LIBRARY_ROOT contains the device library and version file.、Need to set these based off the already computed CMAKE_CUDA_COMPILER_VERSION value、This if statement will always match, but is used to provide variables for MATCH 1,2,3...、Compute the version by invoking nvcc 等标题组织周边说明或配置。
- **EN:** CMake commands like _CUDAToolkit_find_version_file, if, get_filename_component, endif, unset, set drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 _CUDAToolkit_find_version_file、if、get_filename_component、endif、unset、set 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 728-749 / 第 728-749 行

```cmake
    execute_process(COMMAND ${CUDAToolkit_NVCC_EXECUTABLE} "--version" OUTPUT_VARIABLE NVCC_OUT)
    if(NVCC_OUT MATCHES [=[ V([0-9]+)\.([0-9]+)\.([0-9]+)]=])
      set(CUDAToolkit_VERSION_MAJOR "${CMAKE_MATCH_1}")
      set(CUDAToolkit_VERSION_MINOR "${CMAKE_MATCH_2}")
      set(CUDAToolkit_VERSION_PATCH "${CMAKE_MATCH_3}")
      set(CUDAToolkit_VERSION "${CMAKE_MATCH_1}.${CMAKE_MATCH_2}.${CMAKE_MATCH_3}")
    endif()
    unset(NVCC_OUT)
  else()
    _CUDAToolkit_find_version_file(version_file)
    if(version_file)
      file(READ "${version_file}" VERSION_INFO)
      if(VERSION_INFO MATCHES [=[CUDA Version ([0-9]+)\.([0-9]+)\.([0-9]+)]=])
        set(CUDAToolkit_VERSION_MAJOR "${CMAKE_MATCH_1}")
        set(CUDAToolkit_VERSION_MINOR "${CMAKE_MATCH_2}")
        set(CUDAToolkit_VERSION_PATCH "${CMAKE_MATCH_3}")
        set(CUDAToolkit_VERSION "${CMAKE_MATCH_1}.${CMAKE_MATCH_2}.${CMAKE_MATCH_3}")
      endif()
    endif()
  endif()
endif()

```

- **EN:** CMake commands like execute_process, if, set, endif, unset, else drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 execute_process、if、set、endif、unset、else 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 750-768 / 第 750-768 行

```cmake
# Find target directory when crosscompiling.
if(CMAKE_CROSSCOMPILING)
  if(CMAKE_SYSTEM_PROCESSOR STREQUAL "armv7-a")
    # Support for NVPACK
    set(CUDAToolkit_TARGET_NAME "armv7-linux-androideabi")
  elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "arm")
    set(CUDAToolkit_TARGET_NAME "armv7-linux-gnueabihf")
  elseif(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
    if(ANDROID_ARCH_NAME STREQUAL "arm64")
      set(CUDAToolkit_TARGET_NAME "aarch64-linux-androideabi")
    elseif(CMAKE_SYSTEM_NAME STREQUAL "QNX")
      set(CUDAToolkit_TARGET_NAME "aarch64-qnx")
    else()
      set(CUDAToolkit_TARGET_NAME "aarch64-linux")
    endif(ANDROID_ARCH_NAME STREQUAL "arm64")
  elseif(CMAKE_SYSTEM_PROCESSOR STREQUAL "x86_64")
    set(CUDAToolkit_TARGET_NAME "x86_64-linux")
  endif()

```

- **EN:** This chunk introduces sections such as Find target directory when crosscompiling., Support for NVPACK, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Find target directory when crosscompiling.、Support for NVPACK 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, elseif, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、elseif、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 769-784 / 第 769-784 行

```cmake
  if(EXISTS "${CUDAToolkit_ROOT_DIR}/targets/${CUDAToolkit_TARGET_NAME}")
    set(CUDAToolkit_TARGET_DIR "${CUDAToolkit_ROOT_DIR}/targets/${CUDAToolkit_TARGET_NAME}")
    # add known CUDA target root path to the set of directories we search for programs, libraries and headers
    list(PREPEND CMAKE_FIND_ROOT_PATH "${CUDAToolkit_TARGET_DIR}")

    # Mark that we need to pop the root search path changes after we have
    # found all cuda libraries so that searches for our cross-compilation
    # libraries work when another cuda sdk is in CMAKE_PREFIX_PATH or
    # PATh
    set(_CUDAToolkit_Pop_ROOT_PATH True)
  endif()
endif()

# If not already set we can simply use the toolkit root or it's a scattered installation.
if(NOT CUDAToolkit_TARGET_DIR)
  # Not cross compiling
```

- **EN:** This chunk introduces sections such as add known CUDA target root path to the set of directories we search for programs, libraries and headers, Mark that we need to pop the root search path changes after we have, found all cuda libraries so that searches for our cross-compilation, libraries work when another cuda sdk is in CMAKE_PREFIX_PATH or, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 add known CUDA target root path to the set of directories we search for programs, libraries and headers、Mark that we need to pop the root search path changes after we have、found all cuda libraries so that searches for our cross-compilation、libraries work when another cuda sdk is in CMAKE_PREFIX_PATH or 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, list, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、list、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 785-801 / 第 785-801 行

```cmake
  set(CUDAToolkit_TARGET_DIR "${CUDAToolkit_ROOT_DIR}")
  # Now that we have the real ROOT_DIR, find components inside it.
  list(APPEND CMAKE_PREFIX_PATH ${CUDAToolkit_ROOT_DIR})

  # Mark that we need to pop the prefix path changes after we have
  # found the cudart library.
  set(_CUDAToolkit_Pop_Prefix True)
endif()

# CUDAToolkit_TARGET_DIR always points to the directory containing the include directory.
# On a scattered installation /usr, on a non-scattered something like /usr/local/cuda or /usr/local/cuda-10.2/targets/aarch64-linux.
if(EXISTS "${CUDAToolkit_TARGET_DIR}/include/cuda_runtime.h")
  set(CUDAToolkit_INCLUDE_DIR "${CUDAToolkit_TARGET_DIR}/include")
elseif(NOT CUDAToolkit_FIND_QUIETLY)
  message(STATUS "Unable to find cuda_runtime.h in \"${CUDAToolkit_TARGET_DIR}/include\" for CUDAToolkit_INCLUDE_DIR.")
endif()

```

- **EN:** This chunk introduces sections such as Now that we have the real ROOT_DIR, find components inside it., Mark that we need to pop the prefix path changes after we have, found the cudart library., CUDAToolkit_TARGET_DIR always points to the directory containing the include directory., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Now that we have the real ROOT_DIR, find components inside it.、Mark that we need to pop the prefix path changes after we have、found the cudart library.、CUDAToolkit_TARGET_DIR always points to the directory containing the include directory. 等标题组织周边说明或配置。
- **EN:** CMake commands like set, list, endif, if, elseif, message drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 set、list、endif、if、elseif、message 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 802-824 / 第 802-824 行

```cmake
# The NVHPC layout moves math library headers and libraries to a sibling directory.
# Create a separate variable so this directory can be selectively added to math targets.
if(NOT EXISTS "${CUDAToolkit_INCLUDE_DIR}/cublas_v2.h")
  set(CUDAToolkit_MATH_INCLUDE_DIR "${CUDAToolkit_TARGET_DIR}/../../math_libs/include")
  get_filename_component(CUDAToolkit_MATH_INCLUDE_DIR "${CUDAToolkit_MATH_INCLUDE_DIR}" ABSOLUTE)
  if(NOT EXISTS "${CUDAToolkit_MATH_INCLUDE_DIR}/cublas_v2.h")
    if(NOT CUDAToolkit_FIND_QUIETLY)
      message(STATUS "Unable to find cublas_v2.h in either \"${CUDAToolkit_INCLUDE_DIR}\" or \"${CUDAToolkit_MATH_INCLUDE_DIR}\"")
    endif()
    unset(CUDAToolkit_MATH_INCLUDE_DIR)
  endif()
endif()

# Find the CUDA Runtime Library libcudart
find_library(CUDA_CUDART
  NAMES cudart
  PATH_SUFFIXES lib64 lib/x64
)
find_library(CUDA_CUDART
  NAMES cudart
  PATH_SUFFIXES lib64/stubs lib/x64/stubs
)

```

- **EN:** This chunk introduces sections such as The NVHPC layout moves math library headers and libraries to a sibling directory., Create a separate variable so this directory can be selectively added to math targets., Find the CUDA Runtime Library libcudart, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 The NVHPC layout moves math library headers and libraries to a sibling directory.、Create a separate variable so this directory can be selectively added to math targets.、Find the CUDA Runtime Library libcudart 等标题组织周边说明或配置。
- **EN:** CMake commands like if, set, get_filename_component, message, endif, unset drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、set、get_filename_component、message、endif、unset 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 825-846 / 第 825-846 行

```cmake
if(NOT CUDA_CUDART AND NOT CUDAToolkit_FIND_QUIETLY)
  message(STATUS "Unable to find cudart library.")
endif()

if(_CUDAToolkit_Pop_Prefix)
  list(REMOVE_AT CMAKE_PREFIX_PATH -1)
  unset(_CUDAToolkit_Pop_Prefix)
endif()

#-----------------------------------------------------------------------------
# Perform version comparison and validate all required variables are set.
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(CUDAToolkit
  REQUIRED_VARS
    CUDAToolkit_INCLUDE_DIR
    CUDAToolkit_VERSION
    CUDA_CUDART
    CUDAToolkit_BIN_DIR
  VERSION_VAR
    CUDAToolkit_VERSION
)

```

- **EN:** This chunk introduces sections such as -----------------------------------------------------------------------------, Perform version comparison and validate all required variables are set., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 -----------------------------------------------------------------------------、Perform version comparison and validate all required variables are set. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, message, endif, list, unset, include drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、message、endif、list、unset、include 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 847-863 / 第 847-863 行

```cmake
mark_as_advanced(CUDA_CUDART
                 CUDAToolkit_INCLUDE_DIR
                 CUDAToolkit_NVCC_EXECUTABLE
                 CUDAToolkit_SENTINEL_FILE
                 )

#-----------------------------------------------------------------------------
# Construct result variables
if(CUDAToolkit_FOUND)
  set(CUDAToolkit_INCLUDE_DIRS ${CUDAToolkit_INCLUDE_DIR})
  get_filename_component(CUDAToolkit_LIBRARY_DIR ${CUDA_CUDART} DIRECTORY ABSOLUTE)
endif()

#-----------------------------------------------------------------------------
# Construct import targets
if(CUDAToolkit_FOUND)

```

- **EN:** This chunk introduces sections such as -----------------------------------------------------------------------------, Construct result variables, -----------------------------------------------------------------------------, Construct import targets, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 -----------------------------------------------------------------------------、Construct result variables、-----------------------------------------------------------------------------、Construct import targets 等标题组织周边说明或配置。
- **EN:** CMake commands like mark_as_advanced, if, set, get_filename_component, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 mark_as_advanced、if、set、get_filename_component、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 864-885 / 第 864-885 行

```cmake
  function(_CUDAToolkit_find_and_add_import_lib lib_name)
    cmake_parse_arguments(arg "" "" "ALT;DEPS;EXTRA_HINTS;EXTRA_PATH_SUFFIXES;EXTRA_INCLUDE_DIRS" ${ARGN})

    set(search_names ${lib_name} ${arg_ALT})

    find_library(CUDA_${lib_name}_LIBRARY
      NAMES ${search_names}
      HINTS ${CUDAToolkit_LIBRARY_DIR}
            ENV CUDA_PATH
            ${arg_EXTRA_HINTS}
      PATH_SUFFIXES nvidia/current lib64 lib/x64 lib
                    ${arg_EXTRA_PATH_SUFFIXES}
    )
    # Don't try any stub directories until we have exhausted all other
    # search locations.
    find_library(CUDA_${lib_name}_LIBRARY
      NAMES ${search_names}
      HINTS ${CUDAToolkit_LIBRARY_DIR}
            ENV CUDA_PATH
            ${arg_EXTRA_HINTS}
      PATH_SUFFIXES lib64/stubs lib/x64/stubs lib/stubs stubs
                    # Support NVHPC splayed math library layout
```

- **EN:** This chunk introduces sections such as Don't try any stub directories until we have exhausted all other, search locations., Support NVHPC splayed math library layout, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 Don't try any stub directories until we have exhausted all other、search locations.、Support NVHPC splayed math library layout 等标题组织周边说明或配置。
- **EN:** CMake commands like function, cmake_parse_arguments, set, find_library drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 function、cmake_parse_arguments、set、find_library 等 CMake 命令完成依赖探测、变量设置或平台检查。

### Lines 886-917 / 第 886-917 行

```cmake
                    ../../math_libs/${CUDAToolkit_VERSION_MAJOR}.${CUDAToolkit_VERSION_MINOR}/lib64
                    ../../math_libs/lib64
    )

    mark_as_advanced(CUDA_${lib_name}_LIBRARY)

    if(NOT TARGET CUDA::${lib_name} AND CUDA_${lib_name}_LIBRARY)
      add_library(CUDA::${lib_name} UNKNOWN IMPORTED)
      set_property(TARGET CUDA::${lib_name} APPEND PROPERTY
          INTERFACE_INCLUDE_DIRECTORIES "${CUDAToolkit_INCLUDE_DIRS}")
      set_property(TARGET CUDA::${lib_name} APPEND PROPERTY
          INTERFACE_SYSTEM_INCLUDE_DIRECTORIES "${CUDAToolkit_INCLUDE_DIRS}")
      if(DEFINED CUDAToolkit_MATH_INCLUDE_DIR)
        string(FIND ${CUDA_${lib_name}_LIBRARY} "math_libs" math_libs)
        if(NOT ${math_libs} EQUAL -1)
          set_property(TARGET CUDA::${lib_name} APPEND PROPERTY
              INTERFACE_INCLUDE_DIRECTORIES "${CUDAToolkit_MATH_INCLUDE_DIRS}")
          set_property(TARGET CUDA::${lib_name} APPEND PROPERTY
              INTERFACE_SYSTEM_INCLUDE_DIRECTORIES "${CUDAToolkit_MATH_INCLUDE_DIRS}")
        endif()
      endif()
      set_property(TARGET CUDA::${lib_name} PROPERTY IMPORTED_LOCATION "${CUDA_${lib_name}_LIBRARY}")
      foreach(dep ${arg_DEPS})
        if(TARGET CUDA::${dep})
          set_property(TARGET CUDA::${lib_name} APPEND PROPERTY
              INTERFACE_LINK_LIBRARIES CUDA::${dep})
        endif()
      endforeach()
      if(arg_EXTRA_INCLUDE_DIRS)
        set_property(TARGET CUDA::${lib_name} APPEND PROPERTY
            INTERFACE_INCLUDE_DIRECTORIES "${arg_EXTRA_INCLUDE_DIRS}")
        set_property(TARGET CUDA::${lib_name} APPEND PROPERTY
```

- **EN:** CMake commands like mark_as_advanced, if, add_library, set_property, string, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 mark_as_advanced、if、add_library、set_property、string、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 918-935 / 第 918-935 行

```cmake
            INTERFACE_SYSTEM_INCLUDE_DIRECTORIES "${arg_EXTRA_INCLUDE_DIRS}")
      endif()
    endif()
  endfunction()

  if(NOT TARGET CUDA::toolkit)
    add_library(CUDA::toolkit IMPORTED INTERFACE)
    set_property(TARGET CUDA::toolkit APPEND PROPERTY
        INTERFACE_INCLUDE_DIRECTORIES "${CUDAToolkit_INCLUDE_DIRS}")
    set_property(TARGET CUDA::toolkit APPEND PROPERTY
        INTERFACE_SYSTEM_INCLUDE_DIRECTORIES "${CUDAToolkit_INCLUDE_DIRS}")
  endif()

  _CUDAToolkit_find_and_add_import_lib(cuda_driver ALT cuda)

  _CUDAToolkit_find_and_add_import_lib(cudart)
  _CUDAToolkit_find_and_add_import_lib(cudart_static)

```

- **EN:** CMake commands like endif, endfunction, if, add_library, set_property, _CUDAToolkit_find_and_add_import_lib drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 endif、endfunction、if、add_library、set_property、_CUDAToolkit_find_and_add_import_lib 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 936-951 / 第 936-951 行

```cmake
  # setup dependencies that are required for cudart_static when building
  # on linux. These are generally only required when using the CUDA toolkit
  # when CUDA language is disabled
  if(NOT TARGET CUDA::cudart_static_deps
     AND TARGET CUDA::cudart_static)

    add_library(CUDA::cudart_static_deps IMPORTED INTERFACE)
    set_property(TARGET CUDA::cudart_static APPEND PROPERTY
        INTERFACE_LINK_LIBRARIES CUDA::cudart_static_deps)

    if(UNIX AND (CMAKE_C_COMPILER OR CMAKE_CXX_COMPILER))
      find_package(Threads REQUIRED)
      set_property(TARGET CUDA::cudart_static_deps APPEND PROPERTY
          INTERFACE_LINK_LIBRARIES Threads::Threads ${CMAKE_DL_LIBS})
    endif()

```

- **EN:** This chunk introduces sections such as setup dependencies that are required for cudart_static when building, on linux. These are generally only required when using the CUDA toolkit, when CUDA language is disabled, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 setup dependencies that are required for cudart_static when building、on linux. These are generally only required when using the CUDA toolkit、when CUDA language is disabled 等标题组织周边说明或配置。
- **EN:** CMake commands like if, add_library, set_property, find_package, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、add_library、set_property、find_package、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 952-970 / 第 952-970 行

```cmake
    if(UNIX AND NOT APPLE AND NOT (CMAKE_SYSTEM_NAME STREQUAL "QNX"))
      # On Linux, you must link against librt when using the static cuda runtime.
      find_library(CUDAToolkit_rt_LIBRARY rt)
      mark_as_advanced(CUDAToolkit_rt_LIBRARY)
      if(NOT CUDAToolkit_rt_LIBRARY)
        message(WARNING "Could not find librt library, needed by CUDA::cudart_static")
      else()
        set_property(TARGET CUDA::cudart_static_deps APPEND PROPERTY
            INTERFACE_LINK_LIBRARIES ${CUDAToolkit_rt_LIBRARY})
      endif()
    endif()
  endif()

  _CUDAToolkit_find_and_add_import_lib(culibos) # it's a static library
  foreach(cuda_lib cublasLt cufft curand cusparse nppc nvjpeg)
    _CUDAToolkit_find_and_add_import_lib(${cuda_lib})
    _CUDAToolkit_find_and_add_import_lib(${cuda_lib}_static DEPS culibos)
  endforeach()

```

- **EN:** This chunk introduces sections such as On Linux, you must link against librt when using the static cuda runtime., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 On Linux, you must link against librt when using the static cuda runtime. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, find_library, mark_as_advanced, message, else, set_property drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、find_library、mark_as_advanced、message、else、set_property 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 971-988 / 第 971-988 行

```cmake
  if(CUDAToolkit_VERSION VERSION_GREATER_EQUAL 11.0.0)
    # cublas depends on cublasLt
    # https://docs.nvidia.com/cuda/archive/11.0/cublas/index.html#static-library
    _CUDAToolkit_find_and_add_import_lib(cublas DEPS cublasLt)
    _CUDAToolkit_find_and_add_import_lib(cublas_static DEPS cublasLt_static)
  else()
    _CUDAToolkit_find_and_add_import_lib(cublas)
    _CUDAToolkit_find_and_add_import_lib(cublas_static DEPS culibos)
  endif()

  if(CUDAToolkit_VERSION VERSION_GREATER_EQUAL 11.4)
    _CUDAToolkit_find_and_add_import_lib(cuFile ALT cufile DEPS culibos)
    _CUDAToolkit_find_and_add_import_lib(cuFile_static ALT cufile_static DEPS culibos)

    _CUDAToolkit_find_and_add_import_lib(cuFile_rdma ALT cufile_rdma DEPS cuFile culibos)
    _CUDAToolkit_find_and_add_import_lib(cuFile_rdma_static ALT cufile_rdma_static DEPS cuFile_static culibos)
  endif()

```

- **EN:** This chunk introduces sections such as cublas depends on cublasLt, https://docs.nvidia.com/cuda/archive/11.0/cublas/index.html#static-library, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cublas depends on cublasLt、https://docs.nvidia.com/cuda/archive/11.0/cublas/index.html#static-library 等标题组织周边说明或配置。
- **EN:** CMake commands like if, _CUDAToolkit_find_and_add_import_lib, else, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、_CUDAToolkit_find_and_add_import_lib、else、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 989-1007 / 第 989-1007 行

```cmake
  # cuFFTW depends on cuFFT
  _CUDAToolkit_find_and_add_import_lib(cufftw DEPS cufft)
  _CUDAToolkit_find_and_add_import_lib(cufftw_static DEPS cufft_static)
  if(CUDAToolkit_VERSION VERSION_GREATER_EQUAL 9.2)
    _CUDAToolkit_find_and_add_import_lib(cufft_static_nocallback DEPS culibos)
  endif()

  # cuSOLVER depends on cuBLAS, and cuSPARSE
  _CUDAToolkit_find_and_add_import_lib(cusolver DEPS cublas cusparse)
  _CUDAToolkit_find_and_add_import_lib(cusolver_static DEPS cublas_static cusparse_static culibos)


  if(CUDAToolkit_VERSION VERSION_GREATER_EQUAL 10.1.2)
    # cusolver depends on liblapack_static.a starting with CUDA 10.1 update 2,
    # https://docs.nvidia.com/cuda/archive/11.5.0/cusolver/index.html#static-link-lapack
    _CUDAToolkit_find_and_add_import_lib(cusolver_lapack_static ALT lapack_static) # implementation detail static lib
    _CUDAToolkit_find_and_add_import_lib(cusolver_static DEPS cusolver_lapack_static)
  endif()

```

- **EN:** This chunk introduces sections such as cuFFTW depends on cuFFT, cuSOLVER depends on cuBLAS, and cuSPARSE, cusolver depends on liblapack_static.a starting with CUDA 10.1 update 2,, https://docs.nvidia.com/cuda/archive/11.5.0/cusolver/index.html#static-link-lapack, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cuFFTW depends on cuFFT、cuSOLVER depends on cuBLAS, and cuSPARSE、cusolver depends on liblapack_static.a starting with CUDA 10.1 update 2,、https://docs.nvidia.com/cuda/archive/11.5.0/cusolver/index.html#static-link-lapack 等标题组织周边说明或配置。
- **EN:** CMake commands like _CUDAToolkit_find_and_add_import_lib, if, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 _CUDAToolkit_find_and_add_import_lib、if、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1008-1026 / 第 1008-1026 行

```cmake
  if(CUDAToolkit_VERSION VERSION_GREATER 11.2.1)
    # cusolver depends on libcusolver_metis and cublasLt
    # https://docs.nvidia.com/cuda/archive/11.2.2/cusolver/index.html#link-dependency
    _CUDAToolkit_find_and_add_import_lib(cusolver DEPS cublasLt)

    _CUDAToolkit_find_and_add_import_lib(cusolver_metis_static ALT metis_static) # implementation detail static lib
    _CUDAToolkit_find_and_add_import_lib(cusolver_static DEPS cusolver_metis_static cublasLt_static)
  endif()

  # nvGRAPH depends on cuRAND, and cuSOLVER.
  _CUDAToolkit_find_and_add_import_lib(nvgraph DEPS curand cusolver)
  _CUDAToolkit_find_and_add_import_lib(nvgraph_static DEPS curand_static cusolver_static)

  # Process the majority of the NPP libraries.
  foreach(cuda_lib nppial nppicc nppidei nppif nppig nppim nppist nppitc npps nppicom nppisu)
    _CUDAToolkit_find_and_add_import_lib(${cuda_lib} DEPS nppc)
    _CUDAToolkit_find_and_add_import_lib(${cuda_lib}_static DEPS nppc_static)
  endforeach()

```

- **EN:** This chunk introduces sections such as cusolver depends on libcusolver_metis and cublasLt, https://docs.nvidia.com/cuda/archive/11.2.2/cusolver/index.html#link-dependency, nvGRAPH depends on cuRAND, and cuSOLVER., Process the majority of the NPP libraries., which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 cusolver depends on libcusolver_metis and cublasLt、https://docs.nvidia.com/cuda/archive/11.2.2/cusolver/index.html#link-dependency、nvGRAPH depends on cuRAND, and cuSOLVER.、Process the majority of the NPP libraries. 等标题组织周边说明或配置。
- **EN:** CMake commands like if, _CUDAToolkit_find_and_add_import_lib, endif, foreach, endforeach drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、_CUDAToolkit_find_and_add_import_lib、endif、foreach、endforeach 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1027-1044 / 第 1027-1044 行

```cmake
  find_path(CUDAToolkit_CUPTI_INCLUDE_DIR cupti.h PATHS
      "${CUDAToolkit_ROOT_DIR}/extras/CUPTI/include"
      "${CUDAToolkit_INCLUDE_DIR}/../extras/CUPTI/include"
      "${CUDAToolkit_INCLUDE_DIR}"
      NO_DEFAULT_PATH)
  mark_as_advanced(CUDAToolkit_CUPTI_INCLUDE_DIR)

  if(CUDAToolkit_CUPTI_INCLUDE_DIR)
    _CUDAToolkit_find_and_add_import_lib(cupti
                                        EXTRA_PATH_SUFFIXES ../extras/CUPTI/lib64/
                                                            ../extras/CUPTI/lib/
                                        EXTRA_INCLUDE_DIRS "${CUDAToolkit_CUPTI_INCLUDE_DIR}")
    _CUDAToolkit_find_and_add_import_lib(cupti_static
                                        EXTRA_PATH_SUFFIXES ../extras/CUPTI/lib64/
                                                            ../extras/CUPTI/lib/
                                        EXTRA_INCLUDE_DIRS "${CUDAToolkit_CUPTI_INCLUDE_DIR}")
  endif()

```

- **EN:** CMake commands like find_path, mark_as_advanced, if, _CUDAToolkit_find_and_add_import_lib, endif drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 find_path、mark_as_advanced、if、_CUDAToolkit_find_and_add_import_lib、endif 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1045-1064 / 第 1045-1064 行

```cmake
  _CUDAToolkit_find_and_add_import_lib(nvrtc DEPS cuda_driver)

  _CUDAToolkit_find_and_add_import_lib(nvml ALT nvidia-ml nvml)

  # nvtools can be installed outside the CUDA toolkit directory,
  # so search the NVTOOLSEXT_PATH windows only environment variable
  set(nvToolsExt_EXTRA_PATH)
  if(WIN32)
     set(nvToolsExt_EXTRA_PATH "C:\\Program Files\\NVIDIA Corporation\\NvToolsExt")
  endif()

  find_path(CUDAToolkit_nvToolsExt_INCLUDE_DIR nvToolsExt.h
      PATHS "${CUDAToolkit_INCLUDE_DIR}"
            "${CUDAToolkit_ROOT_DIR}"
            ENV NVTOOLSEXT_PATH
            "${nvToolsExt_EXTRA_PATH}"
      PATH_SUFFIXES include
      NO_DEFAULT_PATH)
  mark_as_advanced(CUDAToolkit_nvToolsExt_INCLUDE_DIR)

```

- **EN:** This chunk introduces sections such as nvtools can be installed outside the CUDA toolkit directory,, so search the NVTOOLSEXT_PATH windows only environment variable, which organize the surrounding guidance or configuration.
- **CN:** 这一段通过 nvtools can be installed outside the CUDA toolkit directory,、so search the NVTOOLSEXT_PATH windows only environment variable 等标题组织周边说明或配置。
- **EN:** CMake commands like _CUDAToolkit_find_and_add_import_lib, set, if, endif, find_path, mark_as_advanced drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 _CUDAToolkit_find_and_add_import_lib、set、if、endif、find_path、mark_as_advanced 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

### Lines 1065-1081 / 第 1065-1081 行

```cmake
  if(CUDAToolkit_nvToolsExt_INCLUDE_DIR)
    _CUDAToolkit_find_and_add_import_lib(nvToolsExt
        ALT nvToolsExt64 nvToolsExt64_1
        EXTRA_HINTS ENV NVTOOLSEXT_PATH
                    "${nvToolsExt_EXTRA_PATH}"
        EXTRA_INCLUDE_DIRS "${CUDAToolkit_nvToolsExt_INCLUDE_DIR}")
  endif()

  _CUDAToolkit_find_and_add_import_lib(OpenCL)
endif()

unset(CUDAToolkit_ROOT_DIR)

if(_CUDAToolkit_Pop_ROOT_PATH)
  list(REMOVE_AT CMAKE_FIND_ROOT_PATH 0)
  unset(_CUDAToolkit_Pop_ROOT_PATH)
endif()
```

- **EN:** CMake commands like if, _CUDAToolkit_find_and_add_import_lib, endif, unset, list drive dependency detection, variable setup, or platform checks here.
- **CN:** 这里通过 if、_CUDAToolkit_find_and_add_import_lib、endif、unset、list 等 CMake 命令完成依赖探测、变量设置或平台检查。
- **EN:** Platform- or feature-specific branches are used to adapt configuration to different environments.
- **CN:** 这里使用平台或特性分支来让配置适配不同环境。

## Key Concepts / 关键概念

- **CMake dependency module** — CMake 依赖发现模块
- **Device coverage** — 突出 CPU/CUDA/移动端等后端相关执行路径。
- **CMake build logic** — 控制构建系统配置、特性检查或依赖发现。
- **External package discovery** — 搜索 PyTorch 构建所需的外部库。
- **Representative symbols: system, if, set, endif, else, function, cmake_parse_arguments, find_program** — 代表性符号：system、if、set、endif、else、function、cmake_parse_arguments、find_program

## Dependencies / 依赖关系

- `FindPackageHandleStandardArgs`
- `Threads`
- `CUDA_CUDART`
- `CUDA_`
- `CUDAToolkit_rt_LIBRARY`
