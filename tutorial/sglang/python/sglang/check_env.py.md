# check_env.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/check_env.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Check environment configurations and dependency versions. / 该文件围绕上述模块职责展开实现，组织核心流程、辅助逻辑以及与相邻组件的协作。

## Line-by-Line Analysis / 逐行分析

### Lines 1-1: Module-level supporting statements
```python
"""Check environment configurations and dependency versions."""
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 2-2: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 3-3: Module-level supporting statements
```python
import importlib.metadata
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 4-4: Module-level supporting statements
```python
import os
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 5-5: Module-level supporting statements
```python
import resource
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 6-6: Module-level supporting statements
```python
import subprocess
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 7-7: Module-level supporting statements
```python
import sys
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 8-8: Module-level supporting statements
```python
from abc import abstractmethod
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 9-9: Module-level supporting statements
```python
from collections import OrderedDict, defaultdict
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 10-10: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 11-11: Module-level supporting statements
```python
import torch
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 12-12: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 13-13: Module-level supporting statements
```python
from sglang.srt.utils import is_hip, is_mps, is_musa, is_npu
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 14-15: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 16-17: is cuda v2 function
```python
def is_cuda_v2():
    return torch.version.cuda is not None
```
**EN:** This block uses `is_cuda_v2` to implement the core logic for this symbol. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `is_cuda_v2` 来实现该符号的核心逻辑。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 18-20: Module-level supporting statements
```python


# List of packages to check versions
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 21-53: Module-level supporting statements
```python
PACKAGE_LIST = [
    "sglang",
    "sglang-kernel",
    "flashinfer_python",
    "flashinfer_cubin",
    "flashinfer_jit_cache",
    "triton",
    "transformers",
    "torchao",
    "numpy",
    "aiohttp",
    "fastapi",
    "huggingface_hub",
    "interegular",
    "modelscope",
    "orjson",
    "outlines",
    "packaging",
    "psutil",
    "pydantic",
    "python-multipart",
    "pyzmq",
    "torchao",
    "uvicorn",
    "uvloop",
    "vllm",
    "xgrammar",
    "openai",
    "tiktoken",
    "anthropic",
    "litellm",
    "torchcodec",
]
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 54-55: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 56-56: BaseEnv class declaration
```python
class BaseEnv:
```
**EN:** This block declares the `BaseEnv` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `BaseEnv` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 57-57: Class-level supporting statements
```python
    """Base class for environment check"""
```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 58-58: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 59-60: BaseEnv initializer
```python
    def __init__(self):
        self.package_list = PACKAGE_LIST
```
**EN:** This block initializes the `BaseEnv` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `BaseEnv` 对象，连接后续方法使用的状态与依赖。

### Lines 61-62: Class-level supporting statements
```python

    @abstractmethod
```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 63-67: BaseEnv.get_info method
```python
    def get_info(self) -> dict:
        """
        Get CUDA-related information if available.
        """
        raise NotImplementedError
```
**EN:** This block uses `BaseEnv.get_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseEnv.get_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 68-69: Class-level supporting statements
```python

    @abstractmethod
```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 70-71: BaseEnv.get_topology method
```python
    def get_topology(self) -> dict:
        raise NotImplementedError
```
**EN:** This block uses `BaseEnv.get_topology` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseEnv.get_topology` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 72-72: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 73-85: BaseEnv.get_package_versions method
```python
    def get_package_versions(self) -> dict:
        """
        Get versions of specified packages.
        """
        versions = {}
        for package in self.package_list:
            package_name = package.split("==")[0].split(">=")[0].split("<=")[0]
            try:
                version = importlib.metadata.version(package_name)
                versions[package_name] = version
            except ModuleNotFoundError:
                versions[package_name] = "Module Not Found"
        return versions
```
**EN:** This block uses `BaseEnv.get_package_versions` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseEnv.get_package_versions` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 86-86: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 87-111: BaseEnv.get_device_info method
```python
    def get_device_info(self):
        """
        Get information about available GPU devices.
        """
        devices = defaultdict(list)
        capabilities = defaultdict(list)
        for k in range(torch.cuda.device_count()):
            devices[torch.cuda.get_device_name(k)].append(str(k))
            capability = torch.cuda.get_device_capability(k)
            capabilities[f"{capability[0]}.{capability[1]}"].append(str(k))

        gpu_info = {}
        for name, device_ids in devices.items():
            gpu_info[f"GPU {','.join(device_ids)}"] = name

        if len(capabilities) == 1:
            # All GPUs have the same compute capability
            cap, gpu_ids = list(capabilities.items())[0]
            gpu_info[f"GPU {','.join(gpu_ids)} Compute Capability"] = cap
        else:
            # GPUs have different compute capabilities
            for cap, gpu_ids in capabilities.items():
                gpu_info[f"GPU {','.join(gpu_ids)} Compute Capability"] = cap

        return gpu_info
```
**EN:** This block uses `BaseEnv.get_device_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseEnv.get_device_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 112-112: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 113-121: BaseEnv.get_hypervisor_vendor method
```python
    def get_hypervisor_vendor(self) -> dict:
        try:
            output = subprocess.check_output(["lscpu"], text=True)
            for line in output.split("\n"):
                if "Hypervisor vendor:" in line:
                    return {"Hypervisor vendor:": line.split(":")[1].strip()}
            return {}
        except:
            return {}
```
**EN:** This block uses `BaseEnv.get_hypervisor_vendor` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseEnv.get_hypervisor_vendor` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 122-122: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 123-125: BaseEnv.get_ulimit_soft method
```python
    def get_ulimit_soft(self) -> dict:
        ulimit_soft, _ = resource.getrlimit(resource.RLIMIT_NOFILE)
        return {"ulimit soft": ulimit_soft}
```
**EN:** This block uses `BaseEnv.get_ulimit_soft` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseEnv.get_ulimit_soft` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 126-126: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `BaseEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`BaseEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 127-141: BaseEnv.check_env method
```python
    def check_env(self):
        """
        Check and print environment information.
        """
        env_info = OrderedDict()
        env_info["Python"] = sys.version.replace("\n", "")
        env_info.update(self.get_info())
        env_info["PyTorch"] = torch.__version__
        env_info.update(self.get_package_versions())
        env_info.update(self.get_topology())
        env_info.update(self.get_hypervisor_vendor())
        env_info.update(self.get_ulimit_soft())

        for k, v in env_info.items():
            print(f"{k}: {v}")
```
**EN:** This block uses `BaseEnv.check_env` to validate conditions or environment state. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `BaseEnv.check_env` 来校验条件或环境状态。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 142-143: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 144-144: GPUEnv class declaration
```python
class GPUEnv(BaseEnv):
```
**EN:** This block declares the `GPUEnv` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `GPUEnv` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 145-145: Class-level supporting statements
```python
    """Environment checker for Nvidia GPU"""
```
**EN:** This block contains supporting statements for the `GPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 146-146: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `GPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 147-154: GPUEnv.get_info method
```python
    def get_info(self):
        cuda_info = {"CUDA available": torch.cuda.is_available()}

        if cuda_info["CUDA available"]:
            cuda_info.update(self.get_device_info())
            cuda_info.update(self._get_cuda_version_info())

        return cuda_info
```
**EN:** This block uses `GPUEnv.get_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GPUEnv.get_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 155-155: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `GPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 156-168: GPUEnv._get_cuda_version_info method
```python
    def _get_cuda_version_info(self):
        """
        Get CUDA version information.
        """
        from torch.utils.cpp_extension import CUDA_HOME

        cuda_info = {"CUDA_HOME": CUDA_HOME}

        if CUDA_HOME and os.path.isdir(CUDA_HOME):
            cuda_info.update(self._get_nvcc_info())
            cuda_info.update(self._get_cuda_driver_version())

        return cuda_info
```
**EN:** This block uses `GPUEnv._get_cuda_version_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GPUEnv._get_cuda_version_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 169-169: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `GPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 170-191: GPUEnv._get_nvcc_info method
```python
    def _get_nvcc_info(self):
        """
        Get NVCC version information.
        """
        from torch.utils.cpp_extension import CUDA_HOME

        try:
            nvcc = os.path.join(CUDA_HOME, "bin/nvcc")
            nvcc_output = (
                subprocess.check_output(f'"{nvcc}" -V', shell=True)
                .decode("utf-8")
                .strip()
            )
            return {
                "NVCC": nvcc_output[
                    nvcc_output.rfind("Cuda compilation tools") : nvcc_output.rfind(
                        "Build"
                    )
                ].strip()
            }
        except subprocess.SubprocessError:
            return {"NVCC": "Not Available"}
```
**EN:** This block uses `GPUEnv._get_nvcc_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GPUEnv._get_nvcc_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 192-192: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `GPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 193-202: GPUEnv._get_cuda_driver_version method
```python
    def _get_cuda_driver_version(self):
        """
        Get CUDA driver version.
        """
        from sglang.srt.utils.common import get_nvidia_driver_version_str

        ver = get_nvidia_driver_version_str()
        if ver is None:
            return {"CUDA Driver Version": "Not Available"}
        return {"CUDA Driver Version": ver}
```
**EN:** This block uses `GPUEnv._get_cuda_driver_version` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GPUEnv._get_cuda_driver_version` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 203-203: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `GPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`GPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 204-222: GPUEnv.get_topology method
```python
    def get_topology(self):
        """
        Get GPU topology information.
        """
        try:
            result = subprocess.run(
                ["nvidia-smi", "topo", "-m"],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
                check=True,
            )
            return {
                "NVIDIA Topology": (
                    "\n" + result.stdout if result.returncode == 0 else None
                )
            }
        except subprocess.SubprocessError:
            return {}
```
**EN:** This block uses `GPUEnv.get_topology` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `GPUEnv.get_topology` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 223-224: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 225-225: HIPEnv class declaration
```python
class HIPEnv(BaseEnv):
```
**EN:** This block declares the `HIPEnv` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `HIPEnv` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 226-226: Class-level supporting statements
```python
    """Environment checker for ROCm/HIP"""
```
**EN:** This block contains supporting statements for the `HIPEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`HIPEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 227-227: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `HIPEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`HIPEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 228-235: HIPEnv.get_info method
```python
    def get_info(self):
        cuda_info = {"ROCM available": torch.cuda.is_available()}

        if cuda_info["ROCM available"]:
            cuda_info.update(self.get_device_info())
            cuda_info.update(self._get_cuda_version_info())

        return cuda_info
```
**EN:** This block uses `HIPEnv.get_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `HIPEnv.get_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 236-236: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `HIPEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`HIPEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 237-246: HIPEnv._get_cuda_version_info method
```python
    def _get_cuda_version_info(self):
        from torch.utils.cpp_extension import ROCM_HOME as ROCM_HOME

        cuda_info = {"ROCM_HOME": ROCM_HOME}

        if ROCM_HOME and os.path.isdir(ROCM_HOME):
            cuda_info.update(self._get_hipcc_info())
            cuda_info.update(self._get_rocm_driver_version())

        return cuda_info
```
**EN:** This block uses `HIPEnv._get_cuda_version_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `HIPEnv._get_cuda_version_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 247-247: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `HIPEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`HIPEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 248-264: HIPEnv._get_hipcc_info method
```python
    def _get_hipcc_info(self):
        from torch.utils.cpp_extension import ROCM_HOME

        try:
            hipcc = os.path.join(ROCM_HOME, "bin/hipcc")
            hipcc_output = (
                subprocess.check_output(f'"{hipcc}" --version', shell=True)
                .decode("utf-8")
                .strip()
            )
            return {
                "HIPCC": hipcc_output[
                    hipcc_output.rfind("HIP version") : hipcc_output.rfind("AMD clang")
                ].strip()
            }
        except subprocess.SubprocessError:
            return {"HIPCC": "Not Available"}
```
**EN:** This block uses `HIPEnv._get_hipcc_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `HIPEnv._get_hipcc_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 265-265: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `HIPEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`HIPEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 266-282: HIPEnv._get_rocm_driver_version method
```python
    def _get_rocm_driver_version(self):
        try:
            output = subprocess.check_output(
                [
                    "rocm-smi",
                    "--showdriverversion",
                    "--csv",
                ]
            )
            versions = set(output.decode().strip().split("\n"))
            versions.discard("name, value")
            ver = versions.pop()
            ver = ver.replace('"Driver version", ', "").replace('"', "")

            return {"ROCM Driver Version": ver}
        except subprocess.SubprocessError:
            return {"ROCM Driver Version": "Not Available"}
```
**EN:** This block uses `HIPEnv._get_rocm_driver_version` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `HIPEnv._get_rocm_driver_version` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 283-283: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `HIPEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`HIPEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 284-297: HIPEnv.get_topology method
```python
    def get_topology(self):
        try:
            result = subprocess.run(
                ["rocm-smi", "--showtopotype"],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
                check=True,
            )
            return {
                "AMD Topology": "\n" + result.stdout if result.returncode == 0 else None
            }
        except subprocess.SubprocessError:
            return {}
```
**EN:** This block uses `HIPEnv.get_topology` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `HIPEnv.get_topology` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 298-299: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 300-300: NPUEnv class declaration
```python
class NPUEnv(BaseEnv):
```
**EN:** This block declares the `NPUEnv` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `NPUEnv` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 301-301: Class-level supporting statements
```python
    """Environment checker for Ascend NPU"""
```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 302-302: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 303-307: Class-level supporting statements
```python
    EXTRA_PACKAGE_LIST = [
        "torch_npu",
        "sgl-kernel-npu",
        "deep_ep",
    ]
```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 308-308: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 309-311: NPUEnv initializer
```python
    def __init__(self):
        super().__init__()
        self.package_list.extend(NPUEnv.EXTRA_PACKAGE_LIST)
```
**EN:** This block initializes the `NPUEnv` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `NPUEnv` 对象，连接后续方法使用的状态与依赖。

### Lines 312-312: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 313-319: NPUEnv.get_info method
```python
    def get_info(self):
        cuda_info = {"NPU available": torch.npu.is_available()}
        if cuda_info["NPU available"]:
            cuda_info.update(self.get_device_info())
            cuda_info.update(self._get_cann_version_info())

        return cuda_info
```
**EN:** This block uses `NPUEnv.get_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NPUEnv.get_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 320-320: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 321-334: NPUEnv.get_device_info method
```python
    def get_device_info(self):
        """
        Get information about available NPUs.
        Need to override due to torch_npu interface differences.
        """
        devices = defaultdict(list)
        for k in range(torch.npu.device_count()):
            devices[torch.npu.get_device_name(k)].append(str(k))

        npu_info = {}
        for name, device_ids in devices.items():
            npu_info[f"NPU {','.join(device_ids)}"] = name

        return npu_info
```
**EN:** This block uses `NPUEnv.get_device_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NPUEnv.get_device_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 335-335: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 336-353: NPUEnv._get_cann_version_info method
```python
    def _get_cann_version_info(self):
        cann_envs = ["ASCEND_TOOLKIT_HOME", "ASCEND_INSTALL_PATH"]
        for var in cann_envs:
            path = os.environ.get(var)
            if path and os.path.exists(path):
                CANN_HOME = path
                break
        else:
            default_path = "/usr/local/Ascend/ascend-toolkit/latest"
            CANN_HOME = default_path if os.path.exists(default_path) else None

        if CANN_HOME:
            npu_info = {"CANN_HOME": CANN_HOME}
            npu_info.update(self._get_cann_info(CANN_HOME))
            npu_info.update(self._get_ascend_driver_version())
            return npu_info
        else:
            return {"CANN_HOME": "Not found"}
```
**EN:** This block uses `NPUEnv._get_cann_version_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NPUEnv._get_cann_version_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 354-354: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 355-375: NPUEnv._get_cann_info method
```python
    def _get_cann_info(self, CANN_HOME: str):
        cann_info = {}
        cann_version_file = os.path.join(CANN_HOME, "version.cfg")
        if os.path.exists(cann_version_file):
            with open(cann_version_file, "r", encoding="utf-8") as f:
                f.readline()  # discard first line comment in version.cfg
                cann_info["CANN"] = f.readline().split("[")[1].split("]")[0]
        else:
            cann_info["CANN"] = "Not Available"
        try:
            bisheng = os.path.join(CANN_HOME, "tools/bisheng_compiler/bin/bisheng")
            if not os.path.isfile(bisheng):
                # Check path for old CANN version
                bisheng = os.path.join(CANN_HOME, "compiler/ccec_compiler/bin/bisheng")
            bisheng_output = (
                subprocess.check_output([bisheng, "--version"]).decode("utf-8").strip()
            )
            cann_info["BiSheng"] = bisheng_output.split("\n")[0].strip()
        except subprocess.SubprocessError:
            cann_info["BiSheng"] = "Not Available"
        return cann_info
```
**EN:** This block uses `NPUEnv._get_cann_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NPUEnv._get_cann_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 376-376: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 377-398: NPUEnv._get_ascend_driver_version method
```python
    def _get_ascend_driver_version(self):
        try:
            output = subprocess.check_output(
                [
                    "npu-smi",
                    "info",
                    "-t",
                    "board",
                    "-i",
                    "0",
                ]
            )
            for line in output.decode().strip().split("\n"):
                if "Software Version" in line:
                    version = line.split(":")[-1].strip()
                    break
            else:
                version = "Not Available"

            return {"Ascend Driver Version": version}
        except subprocess.SubprocessError:
            return {"Ascend Driver Version": "Not Available"}
```
**EN:** This block uses `NPUEnv._get_ascend_driver_version` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NPUEnv._get_ascend_driver_version` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 399-399: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `NPUEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`NPUEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 400-415: NPUEnv.get_topology method
```python
    def get_topology(self):
        try:
            result = subprocess.run(
                ["npu-smi", "info", "-t", "topo"],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
                check=True,
            )
            return {
                "Ascend Topology": (
                    "\n" + result.stdout if result.returncode == 0 else None
                )
            }
        except subprocess.SubprocessError:
            return {}
```
**EN:** This block uses `NPUEnv.get_topology` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `NPUEnv.get_topology` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 416-417: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 418-418: MUSAEnv class declaration
```python
class MUSAEnv(BaseEnv):
```
**EN:** This block declares the `MUSAEnv` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MUSAEnv` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 419-419: Class-level supporting statements
```python
    """Environment checker for MThreads GPU"""
```
**EN:** This block contains supporting statements for the `MUSAEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MUSAEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 420-420: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MUSAEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MUSAEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 421-428: MUSAEnv.get_info method
```python
    def get_info(self):
        musa_info = {"MUSA available": torch.musa.is_available()}

        if musa_info["MUSA available"]:
            musa_info.update(self.get_device_info())
            musa_info.update(self._get_musa_version_info())

        return musa_info
```
**EN:** This block uses `MUSAEnv.get_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MUSAEnv.get_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 429-429: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MUSAEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MUSAEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 430-442: MUSAEnv._get_musa_version_info method
```python
    def _get_musa_version_info(self):
        """
        Get MUSA version information.
        """
        from torch_musa.utils.musa_extension import MUSA_HOME

        musa_info = {"MUSA_HOME": MUSA_HOME}

        if MUSA_HOME and os.path.isdir(MUSA_HOME):
            musa_info.update(self._get_mcc_info())
            musa_info.update(self._get_musa_driver_version())

        return musa_info
```
**EN:** This block uses `MUSAEnv._get_musa_version_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MUSAEnv._get_musa_version_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 443-443: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MUSAEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MUSAEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 444-463: MUSAEnv._get_mcc_info method
```python
    def _get_mcc_info(self):
        """
        Get MCC version information.
        """
        from torch_musa.utils.musa_extension import MUSA_HOME

        try:
            mcc = os.path.join(MUSA_HOME, "bin/mcc")
            mcc_output = (
                subprocess.check_output(f'"{mcc}" --version', shell=True)
                .decode("utf-8")
                .strip()
            )
            return {
                "MCC": mcc_output[
                    mcc_output.rfind("mcc version") : mcc_output.rfind("Target")
                ].strip()
            }
        except subprocess.SubprocessError:
            return {"MCC": "Not Available"}
```
**EN:** This block uses `MUSAEnv._get_mcc_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MUSAEnv._get_mcc_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 464-464: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MUSAEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MUSAEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 465-485: MUSAEnv._get_musa_driver_version method
```python
    def _get_musa_driver_version(self):
        """
        Get MUSA driver version.
        """
        try:
            output = subprocess.check_output(
                [
                    "mthreads-gmi",
                    "-q",
                ],
                text=True,
            )
            driver_version = None
            for line in output.splitlines():
                if "Driver Version" in line:
                    driver_version = line.split(":", 1)[1].strip()
                    break

            return {"MUSA Driver Version": driver_version}
        except subprocess.SubprocessError:
            return {"MUSA Driver Version": "Not Available"}
```
**EN:** This block uses `MUSAEnv._get_musa_driver_version` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MUSAEnv._get_musa_driver_version` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 486-486: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MUSAEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MUSAEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 487-505: MUSAEnv.get_topology method
```python
    def get_topology(self):
        """
        Get GPU topology information.
        """
        try:
            result = subprocess.run(
                ["mthreads-gmi", "topo", "-m"],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
                check=True,
            )
            return {
                "MTHREADS Topology": (
                    "\n" + result.stdout if result.returncode == 0 else None
                )
            }
        except subprocess.SubprocessError:
            return {}
```
**EN:** This block uses `MUSAEnv.get_topology` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MUSAEnv.get_topology` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 506-507: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 508-508: MPSEnv class declaration
```python
class MPSEnv(BaseEnv):
```
**EN:** This block declares the `MPSEnv` class, which exists to implement the core logic for this symbol. It defines the structure and member layout used by the rest of the file.
**CN:** 该代码块声明了 `MPSEnv` 类，其职责是实现该符号的核心逻辑。它定义了本文件其余部分使用的结构与成员布局。

### Lines 509-509: Class-level supporting statements
```python
    """Environment checker for Apple Silicon MPS"""
```
**EN:** This block contains supporting statements for the `MPSEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MPSEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 510-510: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MPSEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MPSEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 511-511: Class-level supporting statements
```python
    EXTRA_PACKAGE_LIST = ["mlx", "mlx-lm", "mlx-metal"]
```
**EN:** This block contains supporting statements for the `MPSEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MPSEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 512-512: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MPSEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MPSEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 513-515: MPSEnv initializer
```python
    def __init__(self):
        super().__init__()
        self.package_list.extend(MPSEnv.EXTRA_PACKAGE_LIST)
```
**EN:** This block initializes the `MPSEnv` object, wiring together the state and dependencies used by later methods.
**CN:** 该代码块初始化 `MPSEnv` 对象，连接后续方法使用的状态与依赖。

### Lines 516-516: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MPSEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MPSEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 517-576: MPSEnv.get_info method (part 1/2)
```python
    def get_info(self):
        import platform

        info = {"MPS available": torch.backends.mps.is_available()}
        if not info["MPS available"]:
            return info

        info["macOS Version"] = platform.mac_ver()[0]

        try:
            info["macOS Build"] = subprocess.check_output(
                ["sw_vers", "-buildVersion"], text=True
            ).strip()
        except Exception:
            info["macOS Build"] = "Not Available"

        for label, key in [
            ("Apple Silicon", "machdep.cpu.brand_string"),
            ("Unified Memory", "hw.memsize"),
            ("CPU Cores (Total)", "hw.ncpu"),
        ]:
            try:
                info[label] = subprocess.check_output(
                    ["sysctl", "-n", key], text=True
                ).strip()
            except Exception:
                info[label] = "Not Available"

        try:
            mem_bytes = int(info["Unified Memory"])
            info["Unified Memory"] = f"{mem_bytes / 1024**3:.1f} GB"
        except Exception:
            pass

        for label, key in [
            ("CPU Cores (Performance)", "hw.perflevel0.logicalcpu"),
            ("CPU Cores (Efficiency)", "hw.perflevel1.logicalcpu"),
        ]:
            try:
                info[label] = subprocess.check_output(
                    ["sysctl", "-n", key], text=True
                ).strip()
            except Exception:
                pass

        # Single system_profiler call for both Metal support and GPU cores
        info["Metal Support"] = "Not Available"
        info["GPU Cores"] = "Not Available"
        try:
            sp = subprocess.check_output(
                ["system_profiler", "SPDisplaysDataType"], text=True
            )
            for line in sp.splitlines():
                line = line.strip()
                if "Metal Support" in line or "Metal Family" in line:
                    info["Metal Support"] = line.partition(":")[2].strip()
                if "Total Number of Cores" in line:
                    info["GPU Cores"] = line.partition(":")[2].strip()
        except Exception:
            pass
```
**EN:** This block uses `MPSEnv.get_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the routine entry and its initial setup.
**CN:** 该代码块通过 `MPSEnv.get_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程入口及其初始准备逻辑。

### Lines 577-578: MPSEnv.get_info method (part 2/2)
```python

        return info
```
**EN:** This block uses `MPSEnv.get_info` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components. This chunk covers the later stage of the routine, including final updates or return values.
**CN:** 该代码块通过 `MPSEnv.get_info` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。 这一部分覆盖了例程后段，包括最终状态更新或返回值。

### Lines 579-579: Class-level supporting statements
```python

```
**EN:** This block contains supporting statements for the `MPSEnv` class, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含`MPSEnv` 类的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 580-581: MPSEnv.get_topology method
```python
    def get_topology(self):
        return {}
```
**EN:** This block uses `MPSEnv.get_topology` to fetch or compute a value. It participates in the file's main execution path and exposes behavior consumed by neighboring components.
**CN:** 该代码块通过 `MPSEnv.get_topology` 来获取或计算某个值。它属于本文件的主要执行路径，并向相邻组件暴露可复用行为。

### Lines 582-583: Module-level supporting statements
```python

```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

### Lines 584-595: Module-level supporting statements
```python
if __name__ == "__main__":
    if is_cuda_v2():
        env = GPUEnv()
    elif is_hip():
        env = HIPEnv()
    elif is_npu():
        env = NPUEnv()
    elif is_musa():
        env = MUSAEnv()
    elif is_mps():
        env = MPSEnv()
    env.check_env()
```
**EN:** This block contains supporting statements for the module, such as assignments, registrations, or small helper expressions that shape nearby control flow.
**CN:** 该代码块包含模块的配套语句，例如赋值、注册逻辑或小型辅助表达式，用于组织邻近的控制流程。

## Key Concepts / 关键概念
- Tensor computation with PyTorch / 基于 PyTorch 的张量计算

## Dependencies / 依赖关系
### Internal / 内部
- `sglang.srt.utils`
- `sglang.srt.utils.common`
### External / 外部
- `importlib`
- `platform`
- `resource`
- `torch`
- `torch_musa`
- `abc` (stdlib)
- `collections` (stdlib)
- `os` (stdlib)
- `subprocess` (stdlib)
- `sys` (stdlib)
