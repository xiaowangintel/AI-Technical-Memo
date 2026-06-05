# pynvml.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/third_party/pynvml.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module contributes pynvml to the multimodal generation codebase. Key symbols include `NVMLLibraryMismatchError`, `NVMLError`, `nvmlExceptionClass`. / 该模块为多模态生成代码库提供 pynvml。 关键符号包括 `NVMLLibraryMismatchError`, `NVMLError`, `nvmlExceptionClass`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1203: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# copied from https://pypi.org/project/nvidia-ml-py
# version 12.570.86

#####
# Copyright (c) 2011-2023, NVIDIA Corporation.  All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
#    * Redistributions of source code must retain the above copyright notice,
#      this list of conditions and the following disclaimer.
# ...
NVML_CC_KEY_ROTATION_THRESH_ATTACKER_ADVANTAGE_MAX = 65

# GSP firmware
NVML_GSP_FIRMWARE_VERSION_BUF_SIZE = 0x40
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1206-1207: Class `NVMLLibraryMismatchError` / 类 `NVMLLibraryMismatchError`
```python
class NVMLLibraryMismatchError(Exception):
    pass
```
**EN:** This class models `NVMLLibraryMismatchError` as a specialization of `Exception`.
**CN:** 该类实现 `NVMLLibraryMismatchError`，并继承/扩展 `Exception`。

### Lines 1211-1259: Class `NVMLError` / 类 `NVMLError`
```python
class NVMLError(Exception):
    _valClassMapping = dict()
    # List of currently known error codes
    _errcode_to_string = {
        NVML_ERROR_UNINITIALIZED: "Uninitialized",
        NVML_ERROR_INVALID_ARGUMENT: "Invalid Argument",
        NVML_ERROR_NOT_SUPPORTED: "Not Supported",
        NVML_ERROR_NO_PERMISSION: "Insufficient Permissions",
        NVML_ERROR_ALREADY_INITIALIZED: "Already Initialized",
        NVML_ERROR_NOT_FOUND: "Not Found",
        NVML_ERROR_INSUFFICIENT_SIZE: "Insufficient Size",
        NVML_ERROR_INSUFFICIENT_POWER: "Insufficient External Power",
        NVML_ERROR_DRIVER_NOT_LOADED: "Driver Not Loaded",
        NVML_ERROR_TIMEOUT: "Timeout",
# ...
            return "NVML Error with code %d" % self.value

    def __eq__(self, other):
        return self.value == other.value
```
**EN:** This class models `NVMLError` as a specialization of `Exception`. Important methods include `__new__`, `__str__`, `__eq__`.
**CN:** 该类实现 `NVMLError`，并继承/扩展 `Exception`。 其中较重要的方法包括 `__new__`, `__str__`, `__eq__`。

### Lines 1262-1265: Function `nvmlExceptionClass` / 函数 `nvmlExceptionClass`
```python
def nvmlExceptionClass(nvmlErrorCode):
    if nvmlErrorCode not in NVMLError._valClassMapping:
        raise ValueError("nvmlErrorCode %s is not valid" % nvmlErrorCode)
    return NVMLError._valClassMapping[nvmlErrorCode]
```
**EN:** This function drives `nvmlExceptionClass` with inputs such as `nvmlErrorCode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlExceptionClass`，主要处理 `nvmlErrorCode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1268-1297: Function `_extractNVMLErrorsAsClasses` / 函数 `_extractNVMLErrorsAsClasses`
```python
def _extractNVMLErrorsAsClasses():
    """
    Generates a hierarchy of classes on top of NVMLError class.

    Each NVML Error gets a new NVMLError subclass. This way try,except blocks can filter appropriate
    exceptions more easily.

    NVMLError is a parent class. Each NVML_ERROR_* gets it's own subclass.
    e.g. NVML_ERROR_ALREADY_INITIALIZED will be turned into NVMLError_AlreadyInitialized
    """
    this_module = sys.modules[__name__]
    nvmlErrorsNames = [x for x in dir(this_module) if x.startswith("NVML_ERROR_")]
    for err_name in nvmlErrorsNames:
        # e.g. Turn NVML_ERROR_ALREADY_INITIALIZED into NVMLError_AlreadyInitialized
# ...
        new_error_class = type(class_name, (NVMLError,), {"__new__": gen_new(err_val)})
        new_error_class.__module__ = __name__
        setattr(this_module, class_name, new_error_class)
        NVMLError._valClassMapping[err_val] = new_error_class
```
**EN:** This function drives `_extractNVMLErrorsAsClasses`. Generates a hierarchy of classes on top of NVMLError class.
**CN:** 这个函数负责 `_extractNVMLErrorsAsClasses`。 文档字符串说明：Generates a hierarchy of classes on top of NVMLError class.

### Lines 1298-1300: Top-level configuration / 顶层配置
```python


_extractNVMLErrorsAsClasses()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1303-1306: Function `_nvmlCheckReturn` / 函数 `_nvmlCheckReturn`
```python
def _nvmlCheckReturn(ret):
    if ret != NVML_SUCCESS:
        raise NVMLError(ret)
    return ret
```
**EN:** This function drives `_nvmlCheckReturn` with inputs such as `ret`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_nvmlCheckReturn`，主要处理 `ret` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1307-1312: Top-level configuration / 顶层配置
```python


## Function access ##
_nvmlGetFunctionPointer_cache = (
    dict()
)  # function pointers are cached to prevent unnecessary libLoadLock locking
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1315-1333: Function `_nvmlGetFunctionPointer` / 函数 `_nvmlGetFunctionPointer`
```python
def _nvmlGetFunctionPointer(name):
    global nvmlLib

    if name in _nvmlGetFunctionPointer_cache:
        return _nvmlGetFunctionPointer_cache[name]

    libLoadLock.acquire()
    try:
        # ensure library was loaded
        if nvmlLib is None:
            raise NVMLError(NVML_ERROR_UNINITIALIZED)
        try:
            _nvmlGetFunctionPointer_cache[name] = getattr(nvmlLib, name)
            return _nvmlGetFunctionPointer_cache[name]
        except AttributeError:
            raise NVMLError(NVML_ERROR_FUNCTION_NOT_FOUND)
    finally:
        # lock is always freed
        libLoadLock.release()
```
**EN:** This function drives `_nvmlGetFunctionPointer` with inputs such as `name`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_nvmlGetFunctionPointer`，主要处理 `name` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1340-1346: Class `nvmlFriendlyObject` / 类 `nvmlFriendlyObject`
```python
class nvmlFriendlyObject(object):
    def __init__(self, dictionary):
        for x in dictionary:
            setattr(self, x, dictionary[x])

    def __str__(self):
        return self.__dict__.__str__()
```
**EN:** This class models `nvmlFriendlyObject` as a specialization of `object`. Important methods include `__init__`, `__str__`.
**CN:** 该类实现 `nvmlFriendlyObject`，并继承/扩展 `object`。 其中较重要的方法包括 `__init__`, `__str__`。

### Lines 1349-1357: Function `nvmlStructToFriendlyObject` / 函数 `nvmlStructToFriendlyObject`
```python
def nvmlStructToFriendlyObject(struct):
    d = {}
    for x in struct._fields_:
        key = x[0]
        value = getattr(struct, key)
        # only need to convert from bytes if bytes, no need to check python version.
        d[key] = value.decode() if isinstance(value, bytes) else value
    obj = nvmlFriendlyObject(d)
    return obj
```
**EN:** This function drives `nvmlStructToFriendlyObject` with inputs such as `struct`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlStructToFriendlyObject`，主要处理 `struct` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1361-1370: Function `nvmlFriendlyObjectToStruct` / 函数 `nvmlFriendlyObjectToStruct`
```python
def nvmlFriendlyObjectToStruct(obj, model):
    for x in model._fields_:
        key = x[0]
        value = obj.__dict__[key]
        # any c_char_p in python3 needs to be bytes, default encoding works fine.
        if sys.version_info >= (3,):
            setattr(model, key, value.encode())
        else:
            setattr(model, key, value)
    return model
```
**EN:** This function drives `nvmlFriendlyObjectToStruct` with inputs such as `obj`, `model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlFriendlyObjectToStruct`，主要处理 `obj`, `model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 1374-1375: Class `struct_c_nvmlUnit_t` / 类 `struct_c_nvmlUnit_t`
```python
class struct_c_nvmlUnit_t(Structure):
    pass  # opaque handle
```
**EN:** This class models `struct_c_nvmlUnit_t` as a specialization of `Structure`.
**CN:** 该类实现 `struct_c_nvmlUnit_t`，并继承/扩展 `Structure`。

### Lines 1376-1378: Top-level configuration / 顶层配置
```python


c_nvmlUnit_t = POINTER(struct_c_nvmlUnit_t)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1381-1433: Class `_PrintableStructure` / 类 `_PrintableStructure`
```python
class _PrintableStructure(Structure):
    """
    Abstract class that produces nicer __str__ output than ctypes.Structure.
    e.g. instead of:
      >>> print str(obj)
      <class_name object at 0x7fdf82fef9e0>
    this class will print
      class_name(field_name: formatted_value, field_name: formatted_value)

    _fmt_ dictionary of <str _field_ name> -> <str format>
    e.g. class that has _field_ 'hex_value', c_uint could be formatted with
      _fmt_ = {"hex_value" : "%08X"}
    to produce nicer output.
    Default formatting string for all fields can be set with key "<default>" like:
# ...
            # encoding a python2 string returns the same value, since python2 strings are bytes already
            # bytes passed in python3 will be ignored.
            value = value.encode()
        super(_PrintableStructure, self).__setattr__(name, value)
```
**EN:** This class models `_PrintableStructure` as a specialization of `Structure`. Abstract class that produces nicer __str__ output than ctypes.Structure. Important methods include `__str__`, `__getattribute__`, `__setattr__`.
**CN:** 该类实现 `_PrintableStructure`，并继承/扩展 `Structure`。 文档字符串指出：Abstract class that produces nicer __str__ output than ctypes.Structure. 其中较重要的方法包括 `__str__`, `__getattribute__`, `__setattr__`。

### Lines 1436-1442: Class `c_nvmlUnitInfo_t` / 类 `c_nvmlUnitInfo_t`
```python
class c_nvmlUnitInfo_t(_PrintableStructure):
    _fields_ = [
        ("name", c_char * 96),
        ("id", c_char * 96),
        ("serial", c_char * 96),
        ("firmwareVersion", c_char * 96),
    ]
```
**EN:** This class models `c_nvmlUnitInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlUnitInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1445-1446: Class `c_nvmlC2cModeInfo_v1_t` / 类 `c_nvmlC2cModeInfo_v1_t`
```python
class c_nvmlC2cModeInfo_v1_t(_PrintableStructure):
    _fields_ = [("isC2cEnabled", c_uint)]
```
**EN:** This class models `c_nvmlC2cModeInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlC2cModeInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1447-1449: Top-level configuration / 顶层配置
```python


nvmlC2cModeInfo_v1 = 0x1000008
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1452-1456: Class `c_nvmlLedState_t` / 类 `c_nvmlLedState_t`
```python
class c_nvmlLedState_t(_PrintableStructure):
    _fields_ = [
        ("cause", c_char * 256),
        ("color", _nvmlLedColor_t),
    ]
```
**EN:** This class models `c_nvmlLedState_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlLedState_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1459-1465: Class `c_nvmlPSUInfo_t` / 类 `c_nvmlPSUInfo_t`
```python
class c_nvmlPSUInfo_t(_PrintableStructure):
    _fields_ = [
        ("state", c_char * 256),
        ("current", c_uint),
        ("voltage", c_uint),
        ("power", c_uint),
    ]
```
**EN:** This class models `c_nvmlPSUInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlPSUInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1468-1472: Class `c_nvmlUnitFanInfo_t` / 类 `c_nvmlUnitFanInfo_t`
```python
class c_nvmlUnitFanInfo_t(_PrintableStructure):
    _fields_ = [
        ("speed", c_uint),
        ("state", _nvmlFanState_t),
    ]
```
**EN:** This class models `c_nvmlUnitFanInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlUnitFanInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1475-1476: Class `c_nvmlUnitFanSpeeds_t` / 类 `c_nvmlUnitFanSpeeds_t`
```python
class c_nvmlUnitFanSpeeds_t(_PrintableStructure):
    _fields_ = [("fans", c_nvmlUnitFanInfo_t * 24), ("count", c_uint)]
```
**EN:** This class models `c_nvmlUnitFanSpeeds_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlUnitFanSpeeds_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1480-1481: Class `struct_c_nvmlDevice_t` / 类 `struct_c_nvmlDevice_t`
```python
class struct_c_nvmlDevice_t(Structure):
    pass  # opaque handle
```
**EN:** This class models `struct_c_nvmlDevice_t` as a specialization of `Structure`.
**CN:** 该类实现 `struct_c_nvmlDevice_t`，并继承/扩展 `Structure`。

### Lines 1482-1484: Top-level configuration / 顶层配置
```python


c_nvmlDevice_t = POINTER(struct_c_nvmlDevice_t)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1487-1508: Class `nvmlPciInfoExt_v1_t` / 类 `nvmlPciInfoExt_v1_t`
```python
class nvmlPciInfoExt_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("domain", c_uint),
        ("bus", c_uint),
        ("device", c_uint),
        ("pciDeviceId", c_uint),
        ("pciSubSystemId", c_uint),
        ("baseClass", c_uint),
        ("subClass", c_uint),
        ("busId", c_char * NVML_DEVICE_PCI_BUS_ID_BUFFER_SIZE),
    ]
    _fmt_ = {
        "version": "0x%04X",
# ...
        "pciSubSystemId": "0x%08X",
        "baseClass": "0x%01X",
        "subClass": "0x%01X",
    }
```
**EN:** This class models `nvmlPciInfoExt_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `nvmlPciInfoExt_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1509-1511: Top-level configuration / 顶层配置
```python


nvmlPciInfoExt_v1 = 0x1000040
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1515-1535: Class `nvmlPciInfo_v2_t` / 类 `nvmlPciInfo_v2_t`
```python
class nvmlPciInfo_v2_t(_PrintableStructure):
    _fields_ = [
        ("busId", c_char * NVML_DEVICE_PCI_BUS_ID_BUFFER_V2_SIZE),
        ("domain", c_uint),
        ("bus", c_uint),
        ("device", c_uint),
        ("pciDeviceId", c_uint),
        # Added in 2.285
        ("pciSubSystemId", c_uint),
        ("reserved0", c_uint),
        ("reserved1", c_uint),
        ("reserved2", c_uint),
        ("reserved3", c_uint),
    ]
# ...
        "device": "0x%02X",
        "pciDeviceId": "0x%08X",
        "pciSubSystemId": "0x%08X",
    }
```
**EN:** This class models `nvmlPciInfo_v2_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `nvmlPciInfo_v2_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1538-1558: Class `nvmlPciInfo_t` / 类 `nvmlPciInfo_t`
```python
class nvmlPciInfo_t(_PrintableStructure):
    _fields_ = [
        # Moved to the new busId location below
        ("busIdLegacy", c_char * NVML_DEVICE_PCI_BUS_ID_BUFFER_V2_SIZE),
        ("domain", c_uint),
        ("bus", c_uint),
        ("device", c_uint),
        ("pciDeviceId", c_uint),
        # Added in 2.285
        ("pciSubSystemId", c_uint),
        # New busId replaced the long deprecated and reserved fields with a
        # field of the same size in 9.0
        ("busId", c_char * NVML_DEVICE_PCI_BUS_ID_BUFFER_SIZE),
    ]
# ...
        "device": "0x%02X",
        "pciDeviceId": "0x%08X",
        "pciSubSystemId": "0x%08X",
    }
```
**EN:** This class models `nvmlPciInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `nvmlPciInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1561-1565: Class `c_nvmlSystemDriverBranchInfo_v1_t` / 类 `c_nvmlSystemDriverBranchInfo_v1_t`
```python
class c_nvmlSystemDriverBranchInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("branch", c_char * NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE),
    ]
```
**EN:** This class models `c_nvmlSystemDriverBranchInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlSystemDriverBranchInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1566-1568: Top-level configuration / 顶层配置
```python


SystemDriverBranchInfo_v1 = 0x1000054
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1571-1572: Class `c_nvmlExcludedDeviceInfo_t` / 类 `c_nvmlExcludedDeviceInfo_t`
```python
class c_nvmlExcludedDeviceInfo_t(_PrintableStructure):
    _fields_ = [("pci", nvmlPciInfo_t), ("uuid", c_char * NVML_DEVICE_UUID_BUFFER_SIZE)]
```
**EN:** This class models `c_nvmlExcludedDeviceInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlExcludedDeviceInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1575-1579: Class `nvmlNvLinkUtilizationControl_t` / 类 `nvmlNvLinkUtilizationControl_t`
```python
class nvmlNvLinkUtilizationControl_t(_PrintableStructure):
    _fields_ = [
        ("units", _nvmlNvLinkUtilizationCountUnits_t),
        ("pktfilter", _nvmlNvLinkUtilizationCountPktTypes_t),
    ]
```
**EN:** This class models `nvmlNvLinkUtilizationControl_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `nvmlNvLinkUtilizationControl_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1582-1588: Class `c_nvmlMemory_t` / 类 `c_nvmlMemory_t`
```python
class c_nvmlMemory_t(_PrintableStructure):
    _fields_ = [
        ("total", c_ulonglong),
        ("free", c_ulonglong),
        ("used", c_ulonglong),
    ]
    _fmt_ = {"<default>": "%d B"}
```
**EN:** This class models `c_nvmlMemory_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlMemory_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1591-1599: Class `c_nvmlMemory_v2_t` / 类 `c_nvmlMemory_v2_t`
```python
class c_nvmlMemory_v2_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("total", c_ulonglong),
        ("reserved", c_ulonglong),
        ("free", c_ulonglong),
        ("used", c_ulonglong),
    ]
    _fmt_ = {"<default>": "%d B"}
```
**EN:** This class models `c_nvmlMemory_v2_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlMemory_v2_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1600-1602: Top-level configuration / 顶层配置
```python


nvmlMemory_v2 = 0x02000028
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1605-1611: Class `c_nvmlBAR1Memory_t` / 类 `c_nvmlBAR1Memory_t`
```python
class c_nvmlBAR1Memory_t(_PrintableStructure):
    _fields_ = [
        ("bar1Total", c_ulonglong),
        ("bar1Free", c_ulonglong),
        ("bar1Used", c_ulonglong),
    ]
    _fmt_ = {"<default>": "%d B"}
```
**EN:** This class models `c_nvmlBAR1Memory_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlBAR1Memory_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1614-1615: Class `nvmlClkMonFaultInfo_t` / 类 `nvmlClkMonFaultInfo_t`
```python
class nvmlClkMonFaultInfo_t(Structure):
    _fields_ = [("clkApiDomain", c_uint), ("clkDomainFaultMask", c_uint)]
```
**EN:** This class models `nvmlClkMonFaultInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `nvmlClkMonFaultInfo_t`，并继承/扩展 `Structure`。

### Lines 1616-1618: Top-level configuration / 顶层配置
```python


MAX_CLK_DOMAINS = 32
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1621-1626: Class `nvmlClkMonStatus_t` / 类 `nvmlClkMonStatus_t`
```python
class nvmlClkMonStatus_t(Structure):
    _fields_ = [
        ("bGlobalStatus", c_uint),
        ("clkMonListSize", c_uint),
        ("clkMonList", nvmlClkMonFaultInfo_t * MAX_CLK_DOMAINS),
    ]
```
**EN:** This class models `nvmlClkMonStatus_t` as a specialization of `Structure`.
**CN:** 该类实现 `nvmlClkMonStatus_t`，并继承/扩展 `Structure`。

### Lines 1640-1647: Class `c_nvmlProcessInfo_v2_t` / 类 `c_nvmlProcessInfo_v2_t`
```python
class c_nvmlProcessInfo_v2_t(_PrintableStructure):
    _fields_ = [
        ("pid", c_uint),
        ("usedGpuMemory", c_ulonglong),
        ("gpuInstanceId", c_uint),
        ("computeInstanceId", c_uint),
    ]
    _fmt_ = {"usedGpuMemory": "%d B"}
```
**EN:** This class models `c_nvmlProcessInfo_v2_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlProcessInfo_v2_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1648-1657: Top-level configuration / 顶层配置
```python


c_nvmlProcessInfo_v3_t = c_nvmlProcessInfo_v2_t

c_nvmlProcessInfo_t = c_nvmlProcessInfo_v3_t

_nvmlProcessMode_t = c_uint
NVML_PROCESS_MODE_COMPUTE = 0
NVML_PROCESS_MODE_GRAPHICS = 1
NVML_PROCESS_MODE_MPS = 2
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1660-1667: Class `c_nvmlProcessDetail_v1_t` / 类 `c_nvmlProcessDetail_v1_t`
```python
class c_nvmlProcessDetail_v1_t(Structure):
    _fields_ = [
        ("pid", c_uint),
        ("usedGpuMemory", c_ulonglong),
        ("gpuInstanceId", c_uint),
        ("computeInstanceId", c_uint),
        ("usedGpuCcProtectedMemory", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlProcessDetail_v1_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlProcessDetail_v1_t`，并继承/扩展 `Structure`。

### Lines 1670-1677: Class `c_nvmlProcessDetailList_v1_t` / 类 `c_nvmlProcessDetailList_v1_t`
```python
class c_nvmlProcessDetailList_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("mode", _nvmlProcessMode_t),
        ("numProcArrayEntries", c_uint),
        ("procArray", POINTER(c_nvmlProcessDetail_v1_t)),
    ]
    _fmt_ = {"numProcArrayEntries": "%d B"}
```
**EN:** This class models `c_nvmlProcessDetailList_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlProcessDetailList_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1678-1682: Top-level configuration / 顶层配置
```python


c_nvmlProcessDetailList_t = c_nvmlProcessDetailList_v1_t

nvmlProcessDetailList_v1 = 0x1000018
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1685-1689: Class `c_nvmlBridgeChipInfo_t` / 类 `c_nvmlBridgeChipInfo_t`
```python
class c_nvmlBridgeChipInfo_t(_PrintableStructure):
    _fields_ = [
        ("type", _nvmlBridgeChipType_t),
        ("fwVersion", c_uint),
    ]
```
**EN:** This class models `c_nvmlBridgeChipInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlBridgeChipInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1692-1696: Class `c_nvmlBridgeChipHierarchy_t` / 类 `c_nvmlBridgeChipHierarchy_t`
```python
class c_nvmlBridgeChipHierarchy_t(_PrintableStructure):
    _fields_ = [
        ("bridgeCount", c_uint),
        ("bridgeChipInfo", c_nvmlBridgeChipInfo_t * 128),
    ]
```
**EN:** This class models `c_nvmlBridgeChipHierarchy_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlBridgeChipHierarchy_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1699-1705: Class `c_nvmlEccErrorCounts_t` / 类 `c_nvmlEccErrorCounts_t`
```python
class c_nvmlEccErrorCounts_t(_PrintableStructure):
    _fields_ = [
        ("l1Cache", c_ulonglong),
        ("l2Cache", c_ulonglong),
        ("deviceMemory", c_ulonglong),
        ("registerFile", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlEccErrorCounts_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlEccErrorCounts_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1708-1713: Class `c_nvmlUtilization_t` / 类 `c_nvmlUtilization_t`
```python
class c_nvmlUtilization_t(_PrintableStructure):
    _fields_ = [
        ("gpu", c_uint),
        ("memory", c_uint),
    ]
    _fmt_ = {"<default>": "%d %%"}
```
**EN:** This class models `c_nvmlUtilization_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlUtilization_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1717-1721: Class `c_nvmlHwbcEntry_t` / 类 `c_nvmlHwbcEntry_t`
```python
class c_nvmlHwbcEntry_t(_PrintableStructure):
    _fields_ = [
        ("hwbcId", c_uint),
        ("firmwareVersion", c_char * 32),
    ]
```
**EN:** This class models `c_nvmlHwbcEntry_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlHwbcEntry_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1724-1733: Class `c_nvmlValue_t` / 类 `c_nvmlValue_t`
```python
class c_nvmlValue_t(Union):
    _fields_ = [
        ("dVal", c_double),
        ("uiVal", c_uint),
        ("ulVal", c_ulong),
        ("ullVal", c_ulonglong),
        ("sllVal", c_longlong),
        ("siVal", c_int),
        ("usVal", c_ushort),
    ]
```
**EN:** This class models `c_nvmlValue_t` as a specialization of `Union`.
**CN:** 该类实现 `c_nvmlValue_t`，并继承/扩展 `Union`。

### Lines 1736-1740: Class `c_nvmlSample_t` / 类 `c_nvmlSample_t`
```python
class c_nvmlSample_t(_PrintableStructure):
    _fields_ = [
        ("timeStamp", c_ulonglong),
        ("sampleValue", c_nvmlValue_t),
    ]
```
**EN:** This class models `c_nvmlSample_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlSample_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1743-1747: Class `c_nvmlViolationTime_t` / 类 `c_nvmlViolationTime_t`
```python
class c_nvmlViolationTime_t(_PrintableStructure):
    _fields_ = [
        ("referenceTime", c_ulonglong),
        ("violationTime", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlViolationTime_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlViolationTime_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1750-1759: Class `c_nvmlFieldValue_t` / 类 `c_nvmlFieldValue_t`
```python
class c_nvmlFieldValue_t(_PrintableStructure):
    _fields_ = [
        ("fieldId", c_uint32),
        ("scopeId", c_uint32),
        ("timestamp", c_int64),
        ("latencyUsec", c_int64),
        ("valueType", _nvmlValueType_t),
        ("nvmlReturn", _nvmlReturn_t),
        ("value", c_nvmlValue_t),
    ]
```
**EN:** This class models `c_nvmlFieldValue_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlFieldValue_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1760-1764: Top-level configuration / 顶层配置
```python


NVML_NVLINK_TOTAL_SUPPORTED_BW_MODES = 23

nvmlNvlinkSupportedBwModes_v1 = 0x100001C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1767-1777: Class `c_nvmlNvlinkSupportedBwModes_v1_t` / 类 `c_nvmlNvlinkSupportedBwModes_v1_t`
```python
class c_nvmlNvlinkSupportedBwModes_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("bwModes", c_uint8 * NVML_NVLINK_TOTAL_SUPPORTED_BW_MODES),
        ("totalBwModes", c_uint8),
    ]

    def __init__(self):
        super(c_nvmlNvlinkSupportedBwModes_v1_t, self).__init__(
            version=nvmlNvlinkSupportedBwModes_v1
        )
```
**EN:** This class models `c_nvmlNvlinkSupportedBwModes_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlNvlinkSupportedBwModes_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 1778-1780: Top-level configuration / 顶层配置
```python


nvmlNvlinkGetBwMode_v1 = 0x100000C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1783-1787: Class `c_nvmlNvlinkGetBwMode_v1_t` / 类 `c_nvmlNvlinkGetBwMode_v1_t`
```python
class c_nvmlNvlinkGetBwMode_v1_t(_PrintableStructure):
    _fields_ = [("version", c_uint), ("bIsBest", c_uint), ("bwMode", c_uint8)]

    def __init__(self):
        super(c_nvmlNvlinkGetBwMode_v1_t, self).__init__(version=nvmlNvlinkGetBwMode_v1)
```
**EN:** This class models `c_nvmlNvlinkGetBwMode_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlNvlinkGetBwMode_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 1788-1790: Top-level configuration / 顶层配置
```python


nvmlNvlinkSetBwMode_v1 = 0x100000C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1793-1797: Class `c_nvmlNvlinkSetBwMode_v1_t` / 类 `c_nvmlNvlinkSetBwMode_v1_t`
```python
class c_nvmlNvlinkSetBwMode_v1_t(_PrintableStructure):
    _fields_ = [("version", c_uint), ("bSetBest", c_uint), ("bwMode", c_uint8)]

    def __init__(self):
        super(c_nvmlNvlinkSetBwMode_v1_t, self).__init__(version=nvmlNvlinkSetBwMode_v1)
```
**EN:** This class models `c_nvmlNvlinkSetBwMode_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlNvlinkSetBwMode_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 1800-1804: Class `c_nvmlVgpuHeterogeneousMode_v1_t` / 类 `c_nvmlVgpuHeterogeneousMode_v1_t`
```python
class c_nvmlVgpuHeterogeneousMode_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("mode", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuHeterogeneousMode_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuHeterogeneousMode_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1805-1807: Top-level configuration / 顶层配置
```python


VgpuHeterogeneousMode_v1 = 0x1000008
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1810-1814: Class `c_nvmlVgpuPlacementId_v1_t` / 类 `c_nvmlVgpuPlacementId_v1_t`
```python
class c_nvmlVgpuPlacementId_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("placementId", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuPlacementId_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuPlacementId_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1815-1817: Top-level configuration / 顶层配置
```python


VgpuPlacementId_v1 = 0x1000008
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1820-1826: Class `c_nvmlVgpuPlacementList_v1_t` / 类 `c_nvmlVgpuPlacementList_v1_t`
```python
class c_nvmlVgpuPlacementList_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("count", c_uint),
        ("placementSize", c_uint),
        ("placementIds", POINTER(c_uint)),
    ]
```
**EN:** This class models `c_nvmlVgpuPlacementList_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuPlacementList_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1827-1832: Top-level configuration / 顶层配置
```python


VgpuPlacementList_v1 = 0x1000018

NVML_VGPU_PGPU_HETEROGENEOUS_MODE = 0
NVML_VGPU_PGPU_HOMOGENEOUS_MODE = 1
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1835-1842: Class `c_nvmlVgpuPlacementList_v2_t` / 类 `c_nvmlVgpuPlacementList_v2_t`
```python
class c_nvmlVgpuPlacementList_v2_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("placementSize", c_uint),
        ("count", c_uint),
        ("placementIds", POINTER(c_uint)),
        ("mode", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuPlacementList_v2_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuPlacementList_v2_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1843-1845: Top-level configuration / 顶层配置
```python


VgpuPlacementList_v2 = 0x2000020
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1848-1852: Class `c_nvmlVgpuTypeBar1Info_v1_t` / 类 `c_nvmlVgpuTypeBar1Info_v1_t`
```python
class c_nvmlVgpuTypeBar1Info_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("bar1Size", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlVgpuTypeBar1Info_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuTypeBar1Info_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1853-1855: Top-level configuration / 顶层配置
```python


VgpuTypeBar1Info_v1 = 0x1000010
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1858-1866: Class `c_nvmlVgpuInstanceUtilizationSample_t` / 类 `c_nvmlVgpuInstanceUtilizationSample_t`
```python
class c_nvmlVgpuInstanceUtilizationSample_t(_PrintableStructure):
    _fields_ = [
        ("vgpuInstance", _nvmlVgpuInstance_t),
        ("timeStamp", c_ulonglong),
        ("smUtil", c_nvmlValue_t),
        ("memUtil", c_nvmlValue_t),
        ("encUtil", c_nvmlValue_t),
        ("decUtil", c_nvmlValue_t),
    ]
```
**EN:** This class models `c_nvmlVgpuInstanceUtilizationSample_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuInstanceUtilizationSample_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1869-1879: Class `c_nvmlVgpuInstanceUtilizationInfo_v1_t` / 类 `c_nvmlVgpuInstanceUtilizationInfo_v1_t`
```python
class c_nvmlVgpuInstanceUtilizationInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("timeStamp", c_ulonglong),
        ("vgpuInstance", _nvmlVgpuInstance_t),
        ("smUtil", c_nvmlValue_t),
        ("memUtil", c_nvmlValue_t),
        ("encUtil", c_nvmlValue_t),
        ("decUtil", c_nvmlValue_t),
        ("jpgUtil", c_nvmlValue_t),
        ("ofaUtil", c_nvmlValue_t),
    ]
```
**EN:** This class models `c_nvmlVgpuInstanceUtilizationInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuInstanceUtilizationInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1882-1889: Class `c_nvmlVgpuInstancesUtilizationInfo_v1_t` / 类 `c_nvmlVgpuInstancesUtilizationInfo_v1_t`
```python
class c_nvmlVgpuInstancesUtilizationInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("sampleValType", _nvmlValueType_t),
        ("vgpuInstanceCount", c_uint),
        ("lastSeenTimeStamp", c_ulonglong),
        ("vgpuUtilArray", POINTER(c_nvmlVgpuInstanceUtilizationInfo_v1_t)),
    ]
```
**EN:** This class models `c_nvmlVgpuInstancesUtilizationInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuInstancesUtilizationInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1890-1892: Top-level configuration / 顶层配置
```python


VgpuInstancesUtilizationInfo_v1 = 0x01000020
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1895-1905: Class `c_nvmlVgpuProcessUtilizationSample_t` / 类 `c_nvmlVgpuProcessUtilizationSample_t`
```python
class c_nvmlVgpuProcessUtilizationSample_t(_PrintableStructure):
    _fields_ = [
        ("vgpuInstance", _nvmlVgpuInstance_t),
        ("pid", c_uint),
        ("processName", c_char * NVML_VGPU_NAME_BUFFER_SIZE),
        ("timeStamp", c_ulonglong),
        ("smUtil", c_uint),
        ("memUtil", c_uint),
        ("encUtil", c_uint),
        ("decUtil", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuProcessUtilizationSample_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuProcessUtilizationSample_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1908-1920: Class `c_nvmlVgpuProcessUtilizationInfo_v1_t` / 类 `c_nvmlVgpuProcessUtilizationInfo_v1_t`
```python
class c_nvmlVgpuProcessUtilizationInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("processName", c_char * NVML_VGPU_NAME_BUFFER_SIZE),
        ("timeStamp", c_ulonglong),
        ("vgpuInstance", _nvmlVgpuInstance_t),
        ("pid", c_uint),
        ("smUtil", c_uint),
        ("memUtil", c_uint),
        ("encUtil", c_uint),
        ("decUtil", c_uint),
        ("jpgUtil", c_uint),
        ("ofaUtil", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuProcessUtilizationInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuProcessUtilizationInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1923-1929: Class `c_nvmlVgpuProcessesUtilizationInfo_v1_t` / 类 `c_nvmlVgpuProcessesUtilizationInfo_v1_t`
```python
class c_nvmlVgpuProcessesUtilizationInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("vgpuProcessCount", c_uint),
        ("lastSeenTimeStamp", c_ulonglong),
        ("vgpuProcUtilArray", POINTER(c_nvmlVgpuProcessUtilizationInfo_v1_t)),
    ]
```
**EN:** This class models `c_nvmlVgpuProcessesUtilizationInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuProcessesUtilizationInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1930-1932: Top-level configuration / 顶层配置
```python


VgpuProcessesUtilizationInfo_v1 = 0x01000018
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1935-1939: Class `nvmlVgpuRuntimeState_v1_t` / 类 `nvmlVgpuRuntimeState_v1_t`
```python
class nvmlVgpuRuntimeState_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("size", c_ulonglong),
    ]
```
**EN:** This class models `nvmlVgpuRuntimeState_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `nvmlVgpuRuntimeState_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1940-1942: Top-level configuration / 顶层配置
```python


VgpuRuntimeState_v1 = 0x1000010
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1945-1954: Class `c_nvmlVgpuLicenseExpiry_t` / 类 `c_nvmlVgpuLicenseExpiry_t`
```python
class c_nvmlVgpuLicenseExpiry_t(_PrintableStructure):
    _fields_ = [
        ("year", c_uint32),
        ("month", c_uint16),
        ("day", c_uint16),
        ("hour", c_uint16),
        ("min", c_uint16),
        ("sec", c_uint16),
        ("status", c_uint8),
    ]
```
**EN:** This class models `c_nvmlVgpuLicenseExpiry_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuLicenseExpiry_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1955-1962: Top-level configuration / 顶层配置
```python


NVML_GRID_LICENSE_STATE_UNKNOWN = 0
NVML_GRID_LICENSE_STATE_UNINITIALIZED = 1
NVML_GRID_LICENSE_STATE_UNLICENSED_UNRESTRICTED = 2
NVML_GRID_LICENSE_STATE_UNLICENSED_RESTRICTED = 3
NVML_GRID_LICENSE_STATE_UNLICENSED = 4
NVML_GRID_LICENSE_STATE_LICENSED = 5
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 1965-1970: Class `c_nvmlVgpuLicenseInfo_t` / 类 `c_nvmlVgpuLicenseInfo_t`
```python
class c_nvmlVgpuLicenseInfo_t(_PrintableStructure):
    _fields_ = [
        ("isLicensed", c_uint8),
        ("licenseExpiry", c_nvmlVgpuLicenseExpiry_t),
        ("currentState", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuLicenseInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuLicenseInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1973-1983: Class `c_nvmlEncoderSession_t` / 类 `c_nvmlEncoderSession_t`
```python
class c_nvmlEncoderSession_t(_PrintableStructure):
    _fields_ = [
        ("sessionId", c_uint),
        ("pid", c_uint),
        ("vgpuInstance", _nvmlVgpuInstance_t),
        ("codecType", c_uint),
        ("hResolution", c_uint),
        ("vResolution", c_uint),
        ("averageFps", c_uint),
        ("encodeLatency", c_uint),
    ]
```
**EN:** This class models `c_nvmlEncoderSession_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlEncoderSession_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1986-1994: Class `c_nvmlProcessUtilizationSample_t` / 类 `c_nvmlProcessUtilizationSample_t`
```python
class c_nvmlProcessUtilizationSample_t(_PrintableStructure):
    _fields_ = [
        ("pid", c_uint),
        ("timeStamp", c_ulonglong),
        ("smUtil", c_uint),
        ("memUtil", c_uint),
        ("encUtil", c_uint),
        ("decUtil", c_uint),
    ]
```
**EN:** This class models `c_nvmlProcessUtilizationSample_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlProcessUtilizationSample_t`，并继承/扩展 `_PrintableStructure`。

### Lines 1997-2007: Class `c_nvmlProcessUtilizationInfo_v1_t` / 类 `c_nvmlProcessUtilizationInfo_v1_t`
```python
class c_nvmlProcessUtilizationInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("timeStamp", c_ulonglong),
        ("pid", c_uint),
        ("smUtil", c_uint),
        ("memUtil", c_uint),
        ("encUtil", c_uint),
        ("decUtil", c_uint),
        ("jpgUtil", c_uint),
        ("ofaUtil", c_uint),
    ]
```
**EN:** This class models `c_nvmlProcessUtilizationInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlProcessUtilizationInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2010-2016: Class `c_nvmlProcessesUtilizationInfo_v1_t` / 类 `c_nvmlProcessesUtilizationInfo_v1_t`
```python
class c_nvmlProcessesUtilizationInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("processSamplesCount", c_uint),
        ("lastSeenTimeStamp", c_ulonglong),
        ("procUtilArray", POINTER(c_nvmlProcessUtilizationInfo_v1_t)),
    ]
```
**EN:** This class models `c_nvmlProcessesUtilizationInfo_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlProcessesUtilizationInfo_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2017-2019: Top-level configuration / 顶层配置
```python


ProcessesUtilizationInfo_v1 = 0x01000018
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2022-2031: Class `c_nvmlGridLicenseExpiry_t` / 类 `c_nvmlGridLicenseExpiry_t`
```python
class c_nvmlGridLicenseExpiry_t(_PrintableStructure):
    _fields_ = [
        ("year", c_uint32),
        ("month", c_uint16),
        ("day", c_uint16),
        ("hour", c_uint16),
        ("min", c_uint16),
        ("sec", c_uint16),
        ("status", c_uint8),
    ]
```
**EN:** This class models `c_nvmlGridLicenseExpiry_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicenseExpiry_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2034-2042: Class `c_nvmlGridLicensableFeature_v4_t` / 类 `c_nvmlGridLicensableFeature_v4_t`
```python
class c_nvmlGridLicensableFeature_v4_t(_PrintableStructure):
    _fields_ = [
        ("featureCode", _nvmlGridLicenseFeatureCode_t),
        ("featureState", c_uint),
        ("licenseInfo", c_char * NVML_GRID_LICENSE_BUFFER_SIZE),
        ("productName", c_char * NVML_GRID_LICENSE_BUFFER_SIZE),
        ("featureEnabled", c_uint),
        ("licenseExpiry", c_nvmlGridLicenseExpiry_t),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeature_v4_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeature_v4_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2045-2053: Class `c_nvmlGridLicensableFeatures_v4_t` / 类 `c_nvmlGridLicensableFeatures_v4_t`
```python
class c_nvmlGridLicensableFeatures_v4_t(_PrintableStructure):
    _fields_ = [
        ("isGridLicenseSupported", c_int),
        ("licensableFeaturesCount", c_uint),
        (
            "gridLicensableFeatures",
            c_nvmlGridLicensableFeature_v4_t * NVML_GRID_LICENSE_FEATURE_MAX_COUNT,
        ),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeatures_v4_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeatures_v4_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2056-2063: Class `c_nvmlGridLicensableFeature_v3_t` / 类 `c_nvmlGridLicensableFeature_v3_t`
```python
class c_nvmlGridLicensableFeature_v3_t(_PrintableStructure):
    _fields_ = [
        ("featureCode", _nvmlGridLicenseFeatureCode_t),
        ("featureState", c_uint),
        ("licenseInfo", c_char * NVML_GRID_LICENSE_BUFFER_SIZE),
        ("productName", c_char * NVML_GRID_LICENSE_BUFFER_SIZE),
        ("featureEnabled", c_uint),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeature_v3_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeature_v3_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2066-2074: Class `c_nvmlGridLicensableFeatures_v3_t` / 类 `c_nvmlGridLicensableFeatures_v3_t`
```python
class c_nvmlGridLicensableFeatures_v3_t(_PrintableStructure):
    _fields_ = [
        ("isGridLicenseSupported", c_int),
        ("licensableFeaturesCount", c_uint),
        (
            "gridLicensableFeatures",
            c_nvmlGridLicensableFeature_v3_t * NVML_GRID_LICENSE_FEATURE_MAX_COUNT,
        ),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeatures_v3_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeatures_v3_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2077-2083: Class `c_nvmlGridLicensableFeature_v2_t` / 类 `c_nvmlGridLicensableFeature_v2_t`
```python
class c_nvmlGridLicensableFeature_v2_t(_PrintableStructure):
    _fields_ = [
        ("featureCode", _nvmlGridLicenseFeatureCode_t),
        ("featureState", c_uint),
        ("licenseInfo", c_char * NVML_GRID_LICENSE_BUFFER_SIZE),
        ("productName", c_char * NVML_GRID_LICENSE_BUFFER_SIZE),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeature_v2_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeature_v2_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2086-2094: Class `c_nvmlGridLicensableFeatures_v2_t` / 类 `c_nvmlGridLicensableFeatures_v2_t`
```python
class c_nvmlGridLicensableFeatures_v2_t(_PrintableStructure):
    _fields_ = [
        ("isGridLicenseSupported", c_int),
        ("licensableFeaturesCount", c_uint),
        (
            "gridLicensableFeatures",
            c_nvmlGridLicensableFeature_v2_t * NVML_GRID_LICENSE_FEATURE_MAX_COUNT,
        ),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeatures_v2_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeatures_v2_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2097-2102: Class `c_nvmlGridLicensableFeature_t` / 类 `c_nvmlGridLicensableFeature_t`
```python
class c_nvmlGridLicensableFeature_t(_PrintableStructure):
    _fields_ = [
        ("featureCode", _nvmlGridLicenseFeatureCode_t),
        ("featureState", c_uint),
        ("licenseInfo", c_char * NVML_GRID_LICENSE_BUFFER_SIZE),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeature_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeature_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2105-2113: Class `c_nvmlGridLicensableFeatures_t` / 类 `c_nvmlGridLicensableFeatures_t`
```python
class c_nvmlGridLicensableFeatures_t(_PrintableStructure):
    _fields_ = [
        ("isGridLicenseSupported", c_int),
        ("licensableFeaturesCount", c_uint),
        (
            "gridLicensableFeatures",
            c_nvmlGridLicensableFeature_t * NVML_GRID_LICENSE_FEATURE_MAX_COUNT,
        ),
    ]
```
**EN:** This class models `c_nvmlGridLicensableFeatures_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGridLicensableFeatures_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2116-2120: Class `c_nvmlMarginTemperature_v1_t` / 类 `c_nvmlMarginTemperature_v1_t`
```python
class c_nvmlMarginTemperature_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("marginTemperature", c_int),
    ]
```
**EN:** This class models `c_nvmlMarginTemperature_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlMarginTemperature_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2121-2123: Top-level configuration / 顶层配置
```python


nvmlMarginTemperature_v1 = 0x1000008
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2127-2128: Class `struct_c_nvmlEventSet_t` / 类 `struct_c_nvmlEventSet_t`
```python
class struct_c_nvmlEventSet_t(Structure):
    pass  # opaque handle
```
**EN:** This class models `struct_c_nvmlEventSet_t` as a specialization of `Structure`.
**CN:** 该类实现 `struct_c_nvmlEventSet_t`，并继承/扩展 `Structure`。

### Lines 2129-2214: Top-level configuration / 顶层配置
```python


c_nvmlEventSet_t = POINTER(struct_c_nvmlEventSet_t)

nvmlEventTypeSingleBitEccError = 0x0000000000000001
nvmlEventTypeDoubleBitEccError = 0x0000000000000002
nvmlEventTypePState = 0x0000000000000004
nvmlEventTypeXidCriticalError = 0x0000000000000008
nvmlEventTypeClock = 0x0000000000000010
nvmlEventTypePowerSourceChange = 0x0000000000000080
nvmlEventMigConfigChange = 0x0000000000000100
nvmlEventTypeSingleBitEccErrorStorm = 0x0000000000000200
nvmlEventTypeDramRetirementEvent = 0x0000000000000400
nvmlEventTypeDramRetirementFailure = 0x0000000000000800
# ...
    | nvmlClocksThrottleReasonHwThermalSlowdown
    | nvmlClocksThrottleReasonHwPowerBrakeSlowdown
    | nvmlClocksThrottleReasonDisplayClockSetting
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2217-2225: Class `c_nvmlEventData_t` / 类 `c_nvmlEventData_t`
```python
class c_nvmlEventData_t(_PrintableStructure):
    _fields_ = [
        ("device", c_nvmlDevice_t),
        ("eventType", c_ulonglong),
        ("eventData", c_ulonglong),
        ("gpuInstanceId", c_uint),
        ("computeInstanceId", c_uint),
    ]
    _fmt_ = {"eventType": "0x%08X"}
```
**EN:** This class models `c_nvmlEventData_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlEventData_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2228-2237: Class `c_nvmlAccountingStats_t` / 类 `c_nvmlAccountingStats_t`
```python
class c_nvmlAccountingStats_t(_PrintableStructure):
    _fields_ = [
        ("gpuUtilization", c_uint),
        ("memoryUtilization", c_uint),
        ("maxMemoryUsage", c_ulonglong),
        ("time", c_ulonglong),
        ("startTime", c_ulonglong),
        ("isRunning", c_uint),
        ("reserved", c_uint * 5),
    ]
```
**EN:** This class models `c_nvmlAccountingStats_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlAccountingStats_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2240-2241: Class `c_nvmlVgpuVersion_t` / 类 `c_nvmlVgpuVersion_t`
```python
class c_nvmlVgpuVersion_t(Structure):
    _fields_ = [("minVersion", c_uint), ("maxVersion", c_uint)]
```
**EN:** This class models `c_nvmlVgpuVersion_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlVgpuVersion_t`，并继承/扩展 `Structure`。

### Lines 2244-2256: Class `c_nvmlVgpuMetadata_t` / 类 `c_nvmlVgpuMetadata_t`
```python
class c_nvmlVgpuMetadata_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("revision", c_uint),
        ("guestInfoState", _nvmlVgpuGuestInfoState_t),
        ("guestDriverVersion", c_char * NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE),
        ("hostDriverVersion", c_char * NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE),
        ("reserved", c_uint * 6),
        ("vgpuVirtualizationCaps", c_uint),
        ("guestVgpuVersion", c_uint),
        ("opaqueDataSize", c_uint),
        ("opaqueData", c_char * NVML_VGPU_METADATA_OPAQUE_DATA_SIZE),
    ]
```
**EN:** This class models `c_nvmlVgpuMetadata_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuMetadata_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2259-2269: Class `c_nvmlVgpuPgpuMetadata_t` / 类 `c_nvmlVgpuPgpuMetadata_t`
```python
class c_nvmlVgpuPgpuMetadata_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("revision", c_uint),
        ("hostDriverVersion", c_char * NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE),
        ("pgpuVirtualizationCaps", c_uint),
        ("reserved", c_uint * 5),
        ("hostSupportedVgpuRange", c_nvmlVgpuVersion_t),
        ("opaqueDataSize", c_uint),
        ("opaqueData", c_char * NVML_VGPU_PGPU_METADATA_OPAQUE_DATA_SIZE),
    ]
```
**EN:** This class models `c_nvmlVgpuPgpuMetadata_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuPgpuMetadata_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2272-2276: Class `c_nvmlVgpuPgpuCompatibility_t` / 类 `c_nvmlVgpuPgpuCompatibility_t`
```python
class c_nvmlVgpuPgpuCompatibility_t(Structure):
    _fields_ = [
        ("vgpuVmCompatibility", _nvmlVgpuVmCompatibility_t),
        ("compatibilityLimitCode", _nvmlVgpuPgpuCompatibilityLimitCode_t),
    ]
```
**EN:** This class models `c_nvmlVgpuPgpuCompatibility_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlVgpuPgpuCompatibility_t`，并继承/扩展 `Structure`。

### Lines 2277-2292: Top-level configuration / 顶层配置
```python


## vGPU scheduler policy defines
NVML_VGPU_SCHEDULER_POLICY_UNKNOWN = 0
NVML_VGPU_SCHEDULER_POLICY_BEST_EFFORT = 1
NVML_VGPU_SCHEDULER_POLICY_EQUAL_SHARE = 2
NVML_VGPU_SCHEDULER_POLICY_FIXED_SHARE = 3

## Supported vGPU scheduler policy count
NVML_SUPPORTED_VGPU_SCHEDULER_POLICY_COUNT = 3

NVML_SCHEDULER_SW_MAX_LOG_ENTRIES = 200

NVML_VGPU_SCHEDULER_ARR_DEFAULT = 0
NVML_VGPU_SCHEDULER_ARR_DISABLE = 1
NVML_VGPU_SCHEDULER_ARR_ENABLE = 2
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2295-2299: Class `c_nvmlVgpuSchedDataWithARR_t` / 类 `c_nvmlVgpuSchedDataWithARR_t`
```python
class c_nvmlVgpuSchedDataWithARR_t(_PrintableStructure):
    _fields_ = [
        ("avgFactor", c_uint),
        ("timeslice", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedDataWithARR_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedDataWithARR_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2302-2305: Class `c_nvmlVgpuSchedData_t` / 类 `c_nvmlVgpuSchedData_t`
```python
class c_nvmlVgpuSchedData_t(_PrintableStructure):
    _fields_ = [
        ("timeslice", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedData_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedData_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2308-2312: Class `c_nvmlVgpuSchedulerParams_t` / 类 `c_nvmlVgpuSchedulerParams_t`
```python
class c_nvmlVgpuSchedulerParams_t(Union):
    _fields_ = [
        ("vgpuSchedDataWithARR", c_nvmlVgpuSchedDataWithARR_t),
        ("vgpuSchedData", c_nvmlVgpuSchedData_t),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedulerParams_t` as a specialization of `Union`.
**CN:** 该类实现 `c_nvmlVgpuSchedulerParams_t`，并继承/扩展 `Union`。

### Lines 2315-2323: Class `c_nvmlVgpuSchedulerLogEntry_t` / 类 `c_nvmlVgpuSchedulerLogEntry_t`
```python
class c_nvmlVgpuSchedulerLogEntry_t(_PrintableStructure):
    _fields_ = [
        ("timestamp", c_ulonglong),
        ("timeRunTotal", c_ulonglong),
        ("timeRun", c_ulonglong),
        ("swRunlistId", c_uint),
        ("targetTimeSlice", c_ulonglong),
        ("cumulativePreemptionTime", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedulerLogEntry_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedulerLogEntry_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2326-2337: Class `c_nvmlVgpuSchedulerLog_t` / 类 `c_nvmlVgpuSchedulerLog_t`
```python
class c_nvmlVgpuSchedulerLog_t(_PrintableStructure):
    _fields_ = [
        ("engineId", c_uint),
        ("schedulerPolicy", c_uint),
        ("arrMode", c_uint),
        ("schedulerParams", c_nvmlVgpuSchedulerParams_t),
        ("entriesCount", c_uint),
        (
            "logEntries",
            c_nvmlVgpuSchedulerLogEntry_t * NVML_SCHEDULER_SW_MAX_LOG_ENTRIES,
        ),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedulerLog_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedulerLog_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2340-2345: Class `c_nvmlVgpuSchedulerGetState_t` / 类 `c_nvmlVgpuSchedulerGetState_t`
```python
class c_nvmlVgpuSchedulerGetState_t(_PrintableStructure):
    _fields_ = [
        ("schedulerPolicy", c_uint),
        ("arrMode", c_uint),
        ("schedulerParams", c_nvmlVgpuSchedulerParams_t),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedulerGetState_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedulerGetState_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2348-2352: Class `c_nvmlVgpuSchedSetDataWithARR_t` / 类 `c_nvmlVgpuSchedSetDataWithARR_t`
```python
class c_nvmlVgpuSchedSetDataWithARR_t(_PrintableStructure):
    _fields_ = [
        ("avgFactor", c_uint),
        ("frequency", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedSetDataWithARR_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedSetDataWithARR_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2355-2358: Class `c_nvmlVgpuSchedSetData_t` / 类 `c_nvmlVgpuSchedSetData_t`
```python
class c_nvmlVgpuSchedSetData_t(_PrintableStructure):
    _fields_ = [
        ("timeslice", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedSetData_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedSetData_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2361-2365: Class `c_nvmlVgpuSchedulerSetParams_t` / 类 `c_nvmlVgpuSchedulerSetParams_t`
```python
class c_nvmlVgpuSchedulerSetParams_t(Union):
    _fields_ = [
        ("vgpuSchedDataWithARR", c_nvmlVgpuSchedSetDataWithARR_t),
        ("vgpuSchedData", c_nvmlVgpuSchedSetData_t),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedulerSetParams_t` as a specialization of `Union`.
**CN:** 该类实现 `c_nvmlVgpuSchedulerSetParams_t`，并继承/扩展 `Union`。

### Lines 2368-2373: Class `c_nvmlVgpuSchedulerSetState_t` / 类 `c_nvmlVgpuSchedulerSetState_t`
```python
class c_nvmlVgpuSchedulerSetState_t(_PrintableStructure):
    _fields_ = [
        ("schedulerPolicy", c_uint),
        ("enableARRMode", c_uint),
        ("schedulerParams", c_nvmlVgpuSchedulerSetParams_t),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedulerSetState_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedulerSetState_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2376-2386: Class `c_nvmlVgpuSchedulerCapabilities_t` / 类 `c_nvmlVgpuSchedulerCapabilities_t`
```python
class c_nvmlVgpuSchedulerCapabilities_t(_PrintableStructure):
    _fields_ = [
        ("supportedSchedulers", c_uint * NVML_SUPPORTED_VGPU_SCHEDULER_POLICY_COUNT),
        ("maxTimeslice", c_uint),
        ("minTimeslice", c_uint),
        ("isArrModeSupported", c_uint),
        ("maxFrequencyForARR", c_uint),
        ("minFrequencyForARR", c_uint),
        ("maxAvgFactorForARR", c_uint),
        ("minAvgFactorForARR", c_uint),
    ]
```
**EN:** This class models `c_nvmlVgpuSchedulerCapabilities_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlVgpuSchedulerCapabilities_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2389-2394: Class `c_nvmlFBCStats_t` / 类 `c_nvmlFBCStats_t`
```python
class c_nvmlFBCStats_t(Structure):
    _fields_ = [
        ("sessionsCount", c_uint),
        ("averageFPS", c_uint),
        ("averageLatency", c_uint),
    ]
```
**EN:** This class models `c_nvmlFBCStats_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlFBCStats_t`，并继承/扩展 `Structure`。

### Lines 2397-2411: Class `c_nvmlFBCSession_t` / 类 `c_nvmlFBCSession_t`
```python
class c_nvmlFBCSession_t(_PrintableStructure):
    _fields_ = [
        ("sessionId", c_uint),
        ("pid", c_uint),
        ("vgpuInstance", _nvmlVgpuInstance_t),
        ("displayOrdinal", c_uint),
        ("sessionType", c_uint),
        ("sessionFlags", c_uint),
        ("hMaxResolution", c_uint),
        ("vMaxResolution", c_uint),
        ("hResolution", c_uint),
        ("vResolution", c_uint),
        ("averageFPS", c_uint),
        ("averageLatency", c_uint),
    ]
```
**EN:** This class models `c_nvmlFBCSession_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlFBCSession_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2412-2430: Top-level configuration / 顶层配置
```python


NVML_DEVICE_MIG_DISABLE = 0x0
NVML_DEVICE_MIG_ENABLE = 0x1

NVML_GPU_INSTANCE_PROFILE_1_SLICE = 0x0
NVML_GPU_INSTANCE_PROFILE_2_SLICE = 0x1
NVML_GPU_INSTANCE_PROFILE_3_SLICE = 0x2
NVML_GPU_INSTANCE_PROFILE_4_SLICE = 0x3
NVML_GPU_INSTANCE_PROFILE_7_SLICE = 0x4
NVML_GPU_INSTANCE_PROFILE_8_SLICE = 0x5
NVML_GPU_INSTANCE_PROFILE_6_SLICE = 0x6
NVML_GPU_INSTANCE_PROFILE_1_SLICE_REV1 = 0x7
NVML_GPU_INSTANCE_PROFILE_2_SLICE_REV1 = 0x8
NVML_GPU_INSTANCE_PROFILE_1_SLICE_REV2 = 0x9
NVML_GPU_INSTANCE_PROFILE_1_SLICE_GFX = 0xA
NVML_GPU_INSTANCE_PROFILE_2_SLICE_GFX = 0xB
NVML_GPU_INSTANCE_PROFILE_4_SLICE_GFX = 0xC
NVML_GPU_INSTANCE_PROFILE_COUNT = 0xD
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2433-2434: Class `c_nvmlGpuInstancePlacement_t` / 类 `c_nvmlGpuInstancePlacement_t`
```python
class c_nvmlGpuInstancePlacement_t(Structure):
    _fields_ = [("start", c_uint), ("size", c_uint)]
```
**EN:** This class models `c_nvmlGpuInstancePlacement_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlGpuInstancePlacement_t`，并继承/扩展 `Structure`。

### Lines 2437-2450: Class `c_nvmlGpuInstanceProfileInfo_t` / 类 `c_nvmlGpuInstanceProfileInfo_t`
```python
class c_nvmlGpuInstanceProfileInfo_t(Structure):
    _fields_ = [
        ("id", c_uint),
        ("isP2pSupported", c_uint),
        ("sliceCount", c_uint),
        ("instanceCount", c_uint),
        ("multiprocessorCount", c_uint),
        ("copyEngineCount", c_uint),
        ("decoderCount", c_uint),
        ("encoderCount", c_uint),
        ("jpegCount", c_uint),
        ("ofaCount", c_uint),
        ("memorySizeMB", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlGpuInstanceProfileInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlGpuInstanceProfileInfo_t`，并继承/扩展 `Structure`。

### Lines 2451-2453: Top-level configuration / 顶层配置
```python


nvmlGpuInstanceProfileInfo_v2 = 0x02000098
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2456-2476: Class `c_nvmlGpuInstanceProfileInfo_v2_t` / 类 `c_nvmlGpuInstanceProfileInfo_v2_t`
```python
class c_nvmlGpuInstanceProfileInfo_v2_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("id", c_uint),
        ("isP2pSupported", c_uint),
        ("sliceCount", c_uint),
        ("instanceCount", c_uint),
        ("multiprocessorCount", c_uint),
        ("copyEngineCount", c_uint),
        ("decoderCount", c_uint),
        ("encoderCount", c_uint),
        ("jpegCount", c_uint),
        ("ofaCount", c_uint),
        ("memorySizeMB", c_ulonglong),
# ...
    def __init__(self):
        super(c_nvmlGpuInstanceProfileInfo_v2_t, self).__init__(
            version=nvmlGpuInstanceProfileInfo_v2
        )
```
**EN:** This class models `c_nvmlGpuInstanceProfileInfo_v2_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlGpuInstanceProfileInfo_v2_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 2479-2485: Class `c_nvmlGpuInstanceInfo_t` / 类 `c_nvmlGpuInstanceInfo_t`
```python
class c_nvmlGpuInstanceInfo_t(Structure):
    _fields_ = [
        ("device", c_nvmlDevice_t),
        ("id", c_uint),
        ("profileId", c_uint),
        ("placement", c_nvmlGpuInstancePlacement_t),
    ]
```
**EN:** This class models `c_nvmlGpuInstanceInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlGpuInstanceInfo_t`，并继承/扩展 `Structure`。

### Lines 2488-2489: Class `struct_c_nvmlGpuInstance_t` / 类 `struct_c_nvmlGpuInstance_t`
```python
class struct_c_nvmlGpuInstance_t(Structure):
    pass  # opaque handle
```
**EN:** This class models `struct_c_nvmlGpuInstance_t` as a specialization of `Structure`.
**CN:** 该类实现 `struct_c_nvmlGpuInstance_t`，并继承/扩展 `Structure`。

### Lines 2490-2505: Top-level configuration / 顶层配置
```python


c_nvmlGpuInstance_t = POINTER(struct_c_nvmlGpuInstance_t)

NVML_COMPUTE_INSTANCE_PROFILE_1_SLICE = 0x0
NVML_COMPUTE_INSTANCE_PROFILE_2_SLICE = 0x1
NVML_COMPUTE_INSTANCE_PROFILE_3_SLICE = 0x2
NVML_COMPUTE_INSTANCE_PROFILE_4_SLICE = 0x3
NVML_COMPUTE_INSTANCE_PROFILE_7_SLICE = 0x4
NVML_COMPUTE_INSTANCE_PROFILE_8_SLICE = 0x5
NVML_COMPUTE_INSTANCE_PROFILE_6_SLICE = 0x6
NVML_COMPUTE_INSTANCE_PROFILE_1_SLICE_REV1 = 0x7
NVML_COMPUTE_INSTANCE_PROFILE_COUNT = 0x8

NVML_COMPUTE_INSTANCE_ENGINE_PROFILE_SHARED = 0x0
NVML_COMPUTE_INSTANCE_ENGINE_PROFILE_COUNT = 0x1
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2508-2509: Class `c_nvmlComputeInstancePlacement_t` / 类 `c_nvmlComputeInstancePlacement_t`
```python
class c_nvmlComputeInstancePlacement_t(Structure):
    _fields_ = [("start", c_uint), ("size", c_uint)]
```
**EN:** This class models `c_nvmlComputeInstancePlacement_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlComputeInstancePlacement_t`，并继承/扩展 `Structure`。

### Lines 2512-2523: Class `c_nvmlComputeInstanceProfileInfo_t` / 类 `c_nvmlComputeInstanceProfileInfo_t`
```python
class c_nvmlComputeInstanceProfileInfo_t(Structure):
    _fields_ = [
        ("id", c_uint),
        ("sliceCount", c_uint),
        ("instanceCount", c_uint),
        ("multiprocessorCount", c_uint),
        ("sharedCopyEngineCount", c_uint),
        ("sharedDecoderCount", c_uint),
        ("sharedEncoderCount", c_uint),
        ("sharedJpegCount", c_uint),
        ("sharedOfaCount", c_uint),
    ]
```
**EN:** This class models `c_nvmlComputeInstanceProfileInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlComputeInstanceProfileInfo_t`，并继承/扩展 `Structure`。

### Lines 2524-2526: Top-level configuration / 顶层配置
```python


nvmlComputeInstanceProfileInfo_v2 = 0x02000088
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2529-2547: Class `c_nvmlComputeInstanceProfileInfo_v2_t` / 类 `c_nvmlComputeInstanceProfileInfo_v2_t`
```python
class c_nvmlComputeInstanceProfileInfo_v2_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("id", c_uint),
        ("sliceCount", c_uint),
        ("instanceCount", c_uint),
        ("multiprocessorCount", c_uint),
        ("sharedCopyEngineCount", c_uint),
        ("sharedDecoderCount", c_uint),
        ("sharedEncoderCount", c_uint),
        ("sharedJpegCount", c_uint),
        ("sharedOfaCount", c_uint),
        ("name", c_char * NVML_DEVICE_NAME_V2_BUFFER_SIZE),
    ]

    def __init__(self):
        super(c_nvmlComputeInstanceProfileInfo_v2_t, self).__init__(
            version=nvmlComputeInstanceProfileInfo_v2
        )
```
**EN:** This class models `c_nvmlComputeInstanceProfileInfo_v2_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlComputeInstanceProfileInfo_v2_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 2550-2557: Class `c_nvmlComputeInstanceInfo_t` / 类 `c_nvmlComputeInstanceInfo_t`
```python
class c_nvmlComputeInstanceInfo_t(Structure):
    _fields_ = [
        ("device", c_nvmlDevice_t),
        ("gpuInstance", c_nvmlGpuInstance_t),
        ("id", c_uint),
        ("profileId", c_uint),
        ("placement", c_nvmlComputeInstancePlacement_t),
    ]
```
**EN:** This class models `c_nvmlComputeInstanceInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlComputeInstanceInfo_t`，并继承/扩展 `Structure`。

### Lines 2558-2564: Top-level configuration / 顶层配置
```python


NVML_MAX_GPU_UTILIZATIONS = 8
NVML_GPU_UTILIZATION_DOMAIN_GPU = 0
NVML_GPU_UTILIZATION_DOMAIN_FB = 1
NVML_GPU_UTILIZATION_DOMAIN_VID = 2
NVML_GPU_UTILIZATION_DOMAIN_BUS = 3
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2567-2573: Class `c_nvmlGpuDynamicPstatesUtilization_t` / 类 `c_nvmlGpuDynamicPstatesUtilization_t`
```python
class c_nvmlGpuDynamicPstatesUtilization_t(Structure):
    _fields_ = [
        ("bIsPresent", c_uint, 1),
        ("percentage", c_uint),
        ("incThreshold", c_uint),
        ("decThreshold", c_uint),
    ]
```
**EN:** This class models `c_nvmlGpuDynamicPstatesUtilization_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlGpuDynamicPstatesUtilization_t`，并继承/扩展 `Structure`。

### Lines 2576-2583: Class `c_nvmlGpuDynamicPstatesInfo_t` / 类 `c_nvmlGpuDynamicPstatesInfo_t`
```python
class c_nvmlGpuDynamicPstatesInfo_t(Structure):
    _fields_ = [
        ("flags", c_uint),
        (
            "utilization",
            c_nvmlGpuDynamicPstatesUtilization_t * NVML_MAX_GPU_UTILIZATIONS,
        ),
    ]
```
**EN:** This class models `c_nvmlGpuDynamicPstatesInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlGpuDynamicPstatesInfo_t`，并继承/扩展 `Structure`。

### Lines 2584-2617: Top-level configuration / 顶层配置
```python


NVML_MAX_THERMAL_SENSORS_PER_GPU = 3

NVML_THERMAL_TARGET_NONE = 0
NVML_THERMAL_TARGET_GPU = 1
NVML_THERMAL_TARGET_MEMORY = 2
NVML_THERMAL_TARGET_POWER_SUPPLY = 4
NVML_THERMAL_TARGET_BOARD = 8
NVML_THERMAL_TARGET_VCD_BOARD = 9
NVML_THERMAL_TARGET_VCD_INLET = 10
NVML_THERMAL_TARGET_VCD_OUTLET = 11
NVML_THERMAL_TARGET_ALL = 15
NVML_THERMAL_TARGET_UNKNOWN = -1
# ...
NVML_THERMAL_CONTROLLER_NVSYSCON_E551 = 15
NVML_THERMAL_CONTROLLER_MAX6649R = 16
NVML_THERMAL_CONTROLLER_ADT7473S = 17
NVML_THERMAL_CONTROLLER_UNKNOWN = -1
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2620-2627: Class `c_nvmlGpuThermalSensor_t` / 类 `c_nvmlGpuThermalSensor_t`
```python
class c_nvmlGpuThermalSensor_t(Structure):
    _fields_ = [
        ("controller", c_int),
        ("defaultMinTemp", c_int),
        ("defaultMaxTemp", c_int),
        ("currentTemp", c_int),
        ("target", c_int),
    ]
```
**EN:** This class models `c_nvmlGpuThermalSensor_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlGpuThermalSensor_t`，并继承/扩展 `Structure`。

### Lines 2630-2634: Class `c_nvmlGpuThermalSettings_t` / 类 `c_nvmlGpuThermalSettings_t`
```python
class c_nvmlGpuThermalSettings_t(Structure):
    _fields_ = [
        ("count", c_uint),
        ("sensor", c_nvmlGpuThermalSensor_t * NVML_MAX_THERMAL_SENSORS_PER_GPU),
    ]
```
**EN:** This class models `c_nvmlGpuThermalSettings_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlGpuThermalSettings_t`，并继承/扩展 `Structure`。

### Lines 2635-2652: Top-level configuration / 顶层配置
```python


_nvmlCoolerControl_t = c_uint
NVML_THERMAL_COOLER_SIGNAL_NONE = 0
NVML_THERMAL_COOLER_SIGNAL_TOGGLE = 1
NVML_THERMAL_COOLER_SIGNAL_VARIABLE = 2
NVML_THERMAL_COOLER_SIGNAL_COUNT = 3

_nvmlCoolerTarget_t = c_uint
NVML_THERMAL_COOLER_TARGET_NONE = 1 << 0
NVML_THERMAL_COOLER_TARGET_GPU = 1 << 1
NVML_THERMAL_COOLER_TARGET_MEMORY = 1 << 2
NVML_THERMAL_COOLER_TARGET_POWER_SUPPLY = 1 << 3
NVML_THERMAL_COOLER_TARGET_GPU_RELATED = (
    NVML_THERMAL_COOLER_TARGET_GPU
    | NVML_THERMAL_COOLER_TARGET_MEMORY
    | NVML_THERMAL_COOLER_TARGET_POWER_SUPPLY
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2655-2661: Class `c_nvmlCoolerInfo_t` / 类 `c_nvmlCoolerInfo_t`
```python
class c_nvmlCoolerInfo_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("index", c_uint),
        ("coolerControlType", _nvmlCoolerControl_t),
        ("coolerTarget", _nvmlCoolerTarget_t),
    ]
```
**EN:** This class models `c_nvmlCoolerInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlCoolerInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 2662-2664: Top-level configuration / 顶层配置
```python


nvmlCoolerInfo_v1 = 0x1000010
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2667-2674: Function `nvmlDeviceGetCoolerInfo` / 函数 `nvmlDeviceGetCoolerInfo`
```python
def nvmlDeviceGetCoolerInfo(handle):
    c_coolerInfo = c_nvmlCoolerInfo_t()
    c_coolerInfo.version = nvmlCoolerInfo_v1
    c_coolerInfo.index = 0
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCoolerInfo")
    ret = fn(handle, byref(c_coolerInfo))
    _nvmlCheckReturn(ret)
    return [c_coolerInfo.coolerControlType, c_coolerInfo.coolerTarget]
```
**EN:** This function drives `nvmlDeviceGetCoolerInfo` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCoolerInfo`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2677-2678: Class `struct_c_nvmlComputeInstance_t` / 类 `struct_c_nvmlComputeInstance_t`
```python
class struct_c_nvmlComputeInstance_t(Structure):
    pass  # opaque handle
```
**EN:** This class models `struct_c_nvmlComputeInstance_t` as a specialization of `Structure`.
**CN:** 该类实现 `struct_c_nvmlComputeInstance_t`，并继承/扩展 `Structure`。

### Lines 2679-2681: Top-level configuration / 顶层配置
```python


c_nvmlComputeInstance_t = POINTER(struct_c_nvmlComputeInstance_t)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2684-2695: Class `c_nvmlDeviceAttributes` / 类 `c_nvmlDeviceAttributes`
```python
class c_nvmlDeviceAttributes(Structure):
    _fields_ = [
        ("multiprocessorCount", c_uint),
        ("sharedCopyEngineCount", c_uint),
        ("sharedDecoderCount", c_uint),
        ("sharedEncoderCount", c_uint),
        ("sharedJpegCount", c_uint),
        ("sharedOfaCount", c_uint),
        ("gpuInstanceSliceCount", c_uint),
        ("computeInstanceSliceCount", c_uint),
        ("memorySizeMB", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlDeviceAttributes` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlDeviceAttributes`，并继承/扩展 `Structure`。

### Lines 2698-2705: Class `c_nvmlRowRemapperHistogramValues` / 类 `c_nvmlRowRemapperHistogramValues`
```python
class c_nvmlRowRemapperHistogramValues(Structure):
    _fields_ = [
        ("max", c_uint),
        ("high", c_uint),
        ("partial", c_uint),
        ("low", c_uint),
        ("none", c_uint),
    ]
```
**EN:** This class models `c_nvmlRowRemapperHistogramValues` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlRowRemapperHistogramValues`，并继承/扩展 `Structure`。

### Lines 2706-2714: Top-level configuration / 顶层配置
```python


NVML_GPU_CERT_CHAIN_SIZE = 0x1000
NVML_GPU_ATTESTATION_CERT_CHAIN_SIZE = 0x1400
NVML_CC_GPU_CEC_NONCE_SIZE = 0x20
NVML_CC_GPU_ATTESTATION_REPORT_SIZE = 0x2000
NVML_CC_GPU_CEC_ATTESTATION_REPORT_SIZE = 0x1000
NVML_CC_CEC_ATTESTATION_REPORT_NOT_PRESENT = 0
NVML_CC_CEC_ATTESTATION_REPORT_PRESENT = 1
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2717-2722: Class `c_nvmlConfComputeSystemState_t` / 类 `c_nvmlConfComputeSystemState_t`
```python
class c_nvmlConfComputeSystemState_t(Structure):
    _fields_ = [
        ("environment", c_uint),
        ("ccFeature", c_uint),
        ("devToolsMode", c_uint),
    ]
```
**EN:** This class models `c_nvmlConfComputeSystemState_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlConfComputeSystemState_t`，并继承/扩展 `Structure`。

### Lines 2723-2725: Top-level configuration / 顶层配置
```python


nvmlSystemConfComputeSettings_v1 = 0x1000014
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2728-2740: Class `c_nvmlSystemConfComputeSettings_v1_t` / 类 `c_nvmlSystemConfComputeSettings_v1_t`
```python
class c_nvmlSystemConfComputeSettings_v1_t(Structure):
    _fields_ = [
        ("version", c_uint),
        ("environment", c_uint),
        ("ccFeature", c_uint),
        ("devToolsMode", c_uint),
        ("multiGpuMode", c_uint),
    ]

    def __init__(self):
        super(c_nvmlSystemConfComputeSettings_v1_t, self).__init__(
            version=nvmlSystemConfComputeSettings_v1
        )
```
**EN:** This class models `c_nvmlSystemConfComputeSettings_v1_t` as a specialization of `Structure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlSystemConfComputeSettings_v1_t`，并继承/扩展 `Structure`。 其中较重要的方法包括 `__init__`。

### Lines 2743-2747: Class `c_nvmlConfComputeSystemCaps_t` / 类 `c_nvmlConfComputeSystemCaps_t`
```python
class c_nvmlConfComputeSystemCaps_t(Structure):
    _fields_ = [
        ("cpuCaps", c_uint),
        ("gpusCaps", c_uint),
    ]
```
**EN:** This class models `c_nvmlConfComputeSystemCaps_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlConfComputeSystemCaps_t`，并继承/扩展 `Structure`。

### Lines 2750-2754: Class `c_nvmlConfComputeMemSizeInfo_t` / 类 `c_nvmlConfComputeMemSizeInfo_t`
```python
class c_nvmlConfComputeMemSizeInfo_t(Structure):
    _fields_ = [
        ("protectedMemSizeKib", c_ulonglong),
        ("unprotectedMemSizeKib", c_ulonglong),
    ]
```
**EN:** This class models `c_nvmlConfComputeMemSizeInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlConfComputeMemSizeInfo_t`，并继承/扩展 `Structure`。

### Lines 2757-2763: Class `c_nvmlConfComputeGpuCertificate_t` / 类 `c_nvmlConfComputeGpuCertificate_t`
```python
class c_nvmlConfComputeGpuCertificate_t(Structure):
    _fields_ = [
        ("certChainSize", c_uint),
        ("attestationCertChainSize", c_uint),
        ("certChain", c_uint8 * NVML_GPU_CERT_CHAIN_SIZE),
        ("attestationCertChain", c_uint8 * NVML_GPU_ATTESTATION_CERT_CHAIN_SIZE),
    ]
```
**EN:** This class models `c_nvmlConfComputeGpuCertificate_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlConfComputeGpuCertificate_t`，并继承/扩展 `Structure`。

### Lines 2766-2774: Class `c_nvmlConfComputeGpuAttestationReport_t` / 类 `c_nvmlConfComputeGpuAttestationReport_t`
```python
class c_nvmlConfComputeGpuAttestationReport_t(Structure):
    _fields_ = [
        ("isCecAttestationReportPresent", c_uint),
        ("attestationReportSize", c_uint),
        ("cecAttestationReportSize", c_uint),
        ("nonce", c_uint8 * NVML_CC_GPU_CEC_NONCE_SIZE),
        ("attestationReport", c_uint8 * NVML_CC_GPU_ATTESTATION_REPORT_SIZE),
        ("cecAttestationReport", c_uint8 * NVML_CC_GPU_CEC_ATTESTATION_REPORT_SIZE),
    ]
```
**EN:** This class models `c_nvmlConfComputeGpuAttestationReport_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlConfComputeGpuAttestationReport_t`，并继承/扩展 `Structure`。

### Lines 2777-2781: Class `c_nvmlConfComputeSetKeyRotationThresholdInfo_t` / 类 `c_nvmlConfComputeSetKeyRotationThresholdInfo_t`
```python
class c_nvmlConfComputeSetKeyRotationThresholdInfo_t(Structure):
    _fields_ = [
        ("version", c_uint),
        ("maxAttackerAdvantage", c_ulong),
    ]
```
**EN:** This class models `c_nvmlConfComputeSetKeyRotationThresholdInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlConfComputeSetKeyRotationThresholdInfo_t`，并继承/扩展 `Structure`。

### Lines 2782-2784: Top-level configuration / 顶层配置
```python


ConfComputeSetKeyRotationThresholdInfo_v1 = 0x1000010
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2787-2791: Class `c_nvmlConfComputeGetKeyRotationThresholdInfo_t` / 类 `c_nvmlConfComputeGetKeyRotationThresholdInfo_t`
```python
class c_nvmlConfComputeGetKeyRotationThresholdInfo_t(Structure):
    _fields_ = [
        ("version", c_uint),
        ("attackerAdvantage", c_ulong),
    ]
```
**EN:** This class models `c_nvmlConfComputeGetKeyRotationThresholdInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlConfComputeGetKeyRotationThresholdInfo_t`，并继承/扩展 `Structure`。

### Lines 2792-2794: Top-level configuration / 顶层配置
```python


ConfComputeGetKeyRotationThresholdInfo_v1 = 0x1000010
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 2798-2824: Function `convertStrBytes` / 函数 `convertStrBytes`
```python
def convertStrBytes(func):
    """
    In python 3, strings are unicode instead of bytes, and need to be converted for ctypes
    Args from caller: (1, 'string', <__main__.c_nvmlDevice_t at 0xFFFFFFFF>)
    Args passed to function: (1, b'string', <__main__.c_nvmlDevice_t at 0xFFFFFFFF)>
    ----
    Returned from function: b'returned string'
    Returned to caller: 'returned string'
    """

    @wraps(func)
    def wrapper(*args, **kwargs):
        # encoding a str returns bytes in python 2 and 3
        args = [arg.encode() if isinstance(arg, str) else arg for arg in args]
# ...

    if sys.version_info >= (3,):
        return wrapper
    return func
```
**EN:** This function drives `convertStrBytes` with inputs such as `func`. In python 3, strings are unicode instead of bytes, and need to be converted for ctypes
**CN:** 这个函数负责 `convertStrBytes`，主要处理 `func` 等输入。 文档字符串说明：In python 3, strings are unicode instead of bytes, and need to be converted for ctypes

### Lines 2827-2840: Function `throwOnVersionMismatch` / 函数 `throwOnVersionMismatch`
```python
def throwOnVersionMismatch(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        try:
            return func(*args, **kwargs)
        except NVMLError_FunctionNotFound:
            raise NVMLLibraryMismatchError(
                "Unversioned function called and the "
                "pyNVML version does not match the NVML lib version. "
                "Either use matching pyNVML and NVML lib versions or "
                "use a versioned function such as " + func.__name__ + "_v2"
            )

    return wrapper
```
**EN:** This function drives `throwOnVersionMismatch` with inputs such as `func`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `throwOnVersionMismatch`，主要处理 `func` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2844-2859: Function `nvmlInitWithFlags` / 函数 `nvmlInitWithFlags`
```python
def nvmlInitWithFlags(flags):
    _LoadNvmlLibrary()

    #
    # Initialize the library
    #
    fn = _nvmlGetFunctionPointer("nvmlInitWithFlags")
    ret = fn(flags)
    _nvmlCheckReturn(ret)

    # Atomically update refcount
    global _nvmlLib_refcount
    libLoadLock.acquire()
    _nvmlLib_refcount += 1
    libLoadLock.release()
    return None
```
**EN:** This function drives `nvmlInitWithFlags` with inputs such as `flags`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlInitWithFlags`，主要处理 `flags` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2862-2864: Function `nvmlInit` / 函数 `nvmlInit`
```python
def nvmlInit():
    nvmlInitWithFlags(0)
    return None
```
**EN:** This function drives `nvmlInit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlInit`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2867-2909: Function `_LoadNvmlLibrary` / 函数 `_LoadNvmlLibrary`
```python
def _LoadNvmlLibrary():
    """
    Load the library if it isn't loaded already
    """
    global nvmlLib

    if nvmlLib is None:
        # lock to ensure only one caller loads the library
        libLoadLock.acquire()

        try:
            # ensure the library still isn't loaded
            if nvmlLib is None:
                try:
# ...
                    _nvmlCheckReturn(NVML_ERROR_LIBRARY_NOT_FOUND)
        finally:
            # lock is always freed
            libLoadLock.release()
```
**EN:** This function drives `_LoadNvmlLibrary`. Load the library if it isn't loaded already
**CN:** 这个函数负责 `_LoadNvmlLibrary`。 文档字符串说明：Load the library if it isn't loaded already

### Lines 2912-2926: Function `nvmlShutdown` / 函数 `nvmlShutdown`
```python
def nvmlShutdown():
    #
    # Leave the library loaded, but shutdown the interface
    #
    fn = _nvmlGetFunctionPointer("nvmlShutdown")
    ret = fn()
    _nvmlCheckReturn(ret)

    # Atomically update refcount
    global _nvmlLib_refcount
    libLoadLock.acquire()
    if 0 < _nvmlLib_refcount:
        _nvmlLib_refcount -= 1
    libLoadLock.release()
    return None
```
**EN:** This function drives `nvmlShutdown`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlShutdown`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2930-2935: Function `nvmlErrorString` / 函数 `nvmlErrorString`
```python
@convertStrBytes
def nvmlErrorString(result):
    fn = _nvmlGetFunctionPointer("nvmlErrorString")
    fn.restype = c_char_p  # otherwise return is an int
    ret = fn(result)
    return ret
```
**EN:** This function drives `nvmlErrorString` with inputs such as `result`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlErrorString`，主要处理 `result` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2939-2945: Function `nvmlSystemGetNVMLVersion` / 函数 `nvmlSystemGetNVMLVersion`
```python
@convertStrBytes
def nvmlSystemGetNVMLVersion():
    c_version = create_string_buffer(NVML_SYSTEM_NVML_VERSION_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlSystemGetNVMLVersion")
    ret = fn(c_version, c_uint(NVML_SYSTEM_NVML_VERSION_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_version.value
```
**EN:** This function drives `nvmlSystemGetNVMLVersion`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetNVMLVersion`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2948-2953: Function `nvmlSystemGetCudaDriverVersion` / 函数 `nvmlSystemGetCudaDriverVersion`
```python
def nvmlSystemGetCudaDriverVersion():
    c_cuda_version = c_int()
    fn = _nvmlGetFunctionPointer("nvmlSystemGetCudaDriverVersion")
    ret = fn(byref(c_cuda_version))
    _nvmlCheckReturn(ret)
    return c_cuda_version.value
```
**EN:** This function drives `nvmlSystemGetCudaDriverVersion`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetCudaDriverVersion`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2956-2961: Function `nvmlSystemGetCudaDriverVersion_v2` / 函数 `nvmlSystemGetCudaDriverVersion_v2`
```python
def nvmlSystemGetCudaDriverVersion_v2():
    c_cuda_version = c_int()
    fn = _nvmlGetFunctionPointer("nvmlSystemGetCudaDriverVersion_v2")
    ret = fn(byref(c_cuda_version))
    _nvmlCheckReturn(ret)
    return c_cuda_version.value
```
**EN:** This function drives `nvmlSystemGetCudaDriverVersion_v2`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetCudaDriverVersion_v2`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2965-2971: Function `nvmlSystemGetProcessName` / 函数 `nvmlSystemGetProcessName`
```python
@convertStrBytes
def nvmlSystemGetProcessName(pid):
    c_name = create_string_buffer(1024)
    fn = _nvmlGetFunctionPointer("nvmlSystemGetProcessName")
    ret = fn(c_uint(pid), c_name, c_uint(1024))
    _nvmlCheckReturn(ret)
    return c_name.value
```
**EN:** This function drives `nvmlSystemGetProcessName` with inputs such as `pid`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetProcessName`，主要处理 `pid` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2974-2980: Function `nvmlSystemGetDriverVersion` / 函数 `nvmlSystemGetDriverVersion`
```python
@convertStrBytes
def nvmlSystemGetDriverVersion():
    c_version = create_string_buffer(NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlSystemGetDriverVersion")
    ret = fn(c_version, c_uint(NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_version.value
```
**EN:** This function drives `nvmlSystemGetDriverVersion`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetDriverVersion`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 2984-3004: Function `nvmlSystemGetHicVersion` / 函数 `nvmlSystemGetHicVersion`
```python
def nvmlSystemGetHicVersion():
    c_count = c_uint(0)
    hics = None
    fn = _nvmlGetFunctionPointer("nvmlSystemGetHicVersion")

    # get the count
    ret = fn(byref(c_count), None)

    # this should only fail with insufficient size
    if (ret != NVML_SUCCESS) and (ret != NVML_ERROR_INSUFFICIENT_SIZE):
        raise NVMLError(ret)

    # If there are no hics
    if c_count.value == 0:
# ...
    hics = hic_array()
    ret = fn(byref(c_count), hics)
    _nvmlCheckReturn(ret)
    return hics
```
**EN:** This function drives `nvmlSystemGetHicVersion`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetHicVersion`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3007-3013: Function `nvmlSystemGetDriverBranch` / 函数 `nvmlSystemGetDriverBranch`
```python
def nvmlSystemGetDriverBranch():
    c_branchInfo = c_nvmlSystemDriverBranchInfo_v1_t(0)
    c_branchInfo.version = SystemDriverBranchInfo_v1
    fn = _nvmlGetFunctionPointer("nvmlSystemGetDriverBranch")
    ret = fn(byref(c_branchInfo), c_uint(NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_branchInfo
```
**EN:** This function drives `nvmlSystemGetDriverBranch`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetDriverBranch`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3017-3022: Function `nvmlUnitGetCount` / 函数 `nvmlUnitGetCount`
```python
def nvmlUnitGetCount():
    c_count = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetCount")
    ret = fn(byref(c_count))
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlUnitGetCount`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetCount`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3025-3031: Function `nvmlUnitGetHandleByIndex` / 函数 `nvmlUnitGetHandleByIndex`
```python
def nvmlUnitGetHandleByIndex(index):
    c_index = c_uint(index)
    unit = c_nvmlUnit_t()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetHandleByIndex")
    ret = fn(c_index, byref(unit))
    _nvmlCheckReturn(ret)
    return unit
```
**EN:** This function drives `nvmlUnitGetHandleByIndex` with inputs such as `index`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetHandleByIndex`，主要处理 `index` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3034-3039: Function `nvmlUnitGetUnitInfo` / 函数 `nvmlUnitGetUnitInfo`
```python
def nvmlUnitGetUnitInfo(unit):
    c_info = c_nvmlUnitInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetUnitInfo")
    ret = fn(unit, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlUnitGetUnitInfo` with inputs such as `unit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetUnitInfo`，主要处理 `unit` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3042-3047: Function `nvmlUnitGetLedState` / 函数 `nvmlUnitGetLedState`
```python
def nvmlUnitGetLedState(unit):
    c_state = c_nvmlLedState_t()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetLedState")
    ret = fn(unit, byref(c_state))
    _nvmlCheckReturn(ret)
    return c_state
```
**EN:** This function drives `nvmlUnitGetLedState` with inputs such as `unit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetLedState`，主要处理 `unit` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3050-3055: Function `nvmlUnitGetPsuInfo` / 函数 `nvmlUnitGetPsuInfo`
```python
def nvmlUnitGetPsuInfo(unit):
    c_info = c_nvmlPSUInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetPsuInfo")
    ret = fn(unit, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlUnitGetPsuInfo` with inputs such as `unit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetPsuInfo`，主要处理 `unit` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3058-3063: Function `nvmlUnitGetTemperature` / 函数 `nvmlUnitGetTemperature`
```python
def nvmlUnitGetTemperature(unit, type):
    c_temp = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetTemperature")
    ret = fn(unit, c_uint(type), byref(c_temp))
    _nvmlCheckReturn(ret)
    return c_temp.value
```
**EN:** This function drives `nvmlUnitGetTemperature` with inputs such as `unit`, `type`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetTemperature`，主要处理 `unit`, `type` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3066-3071: Function `nvmlUnitGetFanSpeedInfo` / 函数 `nvmlUnitGetFanSpeedInfo`
```python
def nvmlUnitGetFanSpeedInfo(unit):
    c_speeds = c_nvmlUnitFanSpeeds_t()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetFanSpeedInfo")
    ret = fn(unit, byref(c_speeds))
    _nvmlCheckReturn(ret)
    return c_speeds
```
**EN:** This function drives `nvmlUnitGetFanSpeedInfo` with inputs such as `unit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetFanSpeedInfo`，主要处理 `unit` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3075-3083: Function `nvmlUnitGetDeviceCount` / 函数 `nvmlUnitGetDeviceCount`
```python
def nvmlUnitGetDeviceCount(unit):
    c_count = c_uint(0)
    # query the unit to determine device count
    fn = _nvmlGetFunctionPointer("nvmlUnitGetDevices")
    ret = fn(unit, byref(c_count), None)
    if ret == NVML_ERROR_INSUFFICIENT_SIZE:
        ret = NVML_SUCCESS
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlUnitGetDeviceCount` with inputs such as `unit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetDeviceCount`，主要处理 `unit` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3086-3093: Function `nvmlUnitGetDevices` / 函数 `nvmlUnitGetDevices`
```python
def nvmlUnitGetDevices(unit):
    c_count = c_uint(nvmlUnitGetDeviceCount(unit))
    device_array = c_nvmlDevice_t * c_count.value
    c_devices = device_array()
    fn = _nvmlGetFunctionPointer("nvmlUnitGetDevices")
    ret = fn(unit, byref(c_count), c_devices)
    _nvmlCheckReturn(ret)
    return c_devices
```
**EN:** This function drives `nvmlUnitGetDevices` with inputs such as `unit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitGetDevices`，主要处理 `unit` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3097-3102: Function `nvmlDeviceGetCount` / 函数 `nvmlDeviceGetCount`
```python
def nvmlDeviceGetCount():
    c_count = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCount_v2")
    ret = fn(byref(c_count))
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlDeviceGetCount`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCount`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3105-3111: Function `nvmlDeviceGetHandleByIndex` / 函数 `nvmlDeviceGetHandleByIndex`
```python
def nvmlDeviceGetHandleByIndex(index):
    c_index = c_uint(index)
    device = c_nvmlDevice_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetHandleByIndex_v2")
    ret = fn(c_index, byref(device))
    _nvmlCheckReturn(ret)
    return device
```
**EN:** This function drives `nvmlDeviceGetHandleByIndex` with inputs such as `index`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetHandleByIndex`，主要处理 `index` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3114-3121: Function `nvmlDeviceGetHandleBySerial` / 函数 `nvmlDeviceGetHandleBySerial`
```python
@convertStrBytes
def nvmlDeviceGetHandleBySerial(serial):
    c_serial = c_char_p(serial)
    device = c_nvmlDevice_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetHandleBySerial")
    ret = fn(c_serial, byref(device))
    _nvmlCheckReturn(ret)
    return device
```
**EN:** This function drives `nvmlDeviceGetHandleBySerial` with inputs such as `serial`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetHandleBySerial`，主要处理 `serial` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3124-3131: Function `nvmlDeviceGetHandleByUUID` / 函数 `nvmlDeviceGetHandleByUUID`
```python
@convertStrBytes
def nvmlDeviceGetHandleByUUID(uuid):
    c_uuid = c_char_p(uuid)
    device = c_nvmlDevice_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetHandleByUUID")
    ret = fn(c_uuid, byref(device))
    _nvmlCheckReturn(ret)
    return device
```
**EN:** This function drives `nvmlDeviceGetHandleByUUID` with inputs such as `uuid`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetHandleByUUID`，主要处理 `uuid` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3134-3141: Function `nvmlDeviceGetHandleByPciBusId` / 函数 `nvmlDeviceGetHandleByPciBusId`
```python
@convertStrBytes
def nvmlDeviceGetHandleByPciBusId(pciBusId):
    c_busId = c_char_p(pciBusId)
    device = c_nvmlDevice_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetHandleByPciBusId_v2")
    ret = fn(c_busId, byref(device))
    _nvmlCheckReturn(ret)
    return device
```
**EN:** This function drives `nvmlDeviceGetHandleByPciBusId` with inputs such as `pciBusId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetHandleByPciBusId`，主要处理 `pciBusId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3144-3150: Function `nvmlDeviceGetName` / 函数 `nvmlDeviceGetName`
```python
@convertStrBytes
def nvmlDeviceGetName(handle):
    c_name = create_string_buffer(NVML_DEVICE_NAME_V2_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetName")
    ret = fn(handle, c_name, c_uint(NVML_DEVICE_NAME_V2_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_name.value
```
**EN:** This function drives `nvmlDeviceGetName` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetName`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3153-3157: Class `c_nvmlDevicePerfModes_v1_t` / 类 `c_nvmlDevicePerfModes_v1_t`
```python
class c_nvmlDevicePerfModes_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("str", c_char * NVML_PERF_MODES_BUFFER_SIZE),
    ]
```
**EN:** This class models `c_nvmlDevicePerfModes_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlDevicePerfModes_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 3158-3160: Top-level configuration / 顶层配置
```python


nvmlDevicePerfModes_v1 = 0x1000804
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 3163-3170: Function `nvmlDeviceGetPerformanceModes` / 函数 `nvmlDeviceGetPerformanceModes`
```python
@convertStrBytes
def nvmlDeviceGetPerformanceModes(handle):
    perfModes = c_nvmlDevicePerfModes_v1_t()
    perfModes.version = nvmlDevicePerfModes_v1
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPerformanceModes")
    ret = fn(handle, byref(perfModes))
    _nvmlCheckReturn(ret)
    return perfModes.str
```
**EN:** This function drives `nvmlDeviceGetPerformanceModes` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPerformanceModes`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3173-3177: Class `c_nvmlDeviceCurrentClockFreqs_v1_t` / 类 `c_nvmlDeviceCurrentClockFreqs_v1_t`
```python
class c_nvmlDeviceCurrentClockFreqs_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("str", c_char * NVML_PERF_MODES_BUFFER_SIZE),
    ]
```
**EN:** This class models `c_nvmlDeviceCurrentClockFreqs_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlDeviceCurrentClockFreqs_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 3178-3180: Top-level configuration / 顶层配置
```python


nvmlDeviceCurrentClockFreqs_v1 = 0x1000804
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 3183-3190: Function `nvmlDeviceGetCurrentClockFreqs` / 函数 `nvmlDeviceGetCurrentClockFreqs`
```python
@convertStrBytes
def nvmlDeviceGetCurrentClockFreqs(handle):
    currentClockFreqs = c_nvmlDeviceCurrentClockFreqs_v1_t()
    currentClockFreqs.version = nvmlDeviceCurrentClockFreqs_v1
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCurrentClockFreqs")
    ret = fn(handle, byref(currentClockFreqs))
    _nvmlCheckReturn(ret)
    return currentClockFreqs.str
```
**EN:** This function drives `nvmlDeviceGetCurrentClockFreqs` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrentClockFreqs`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3193-3198: Function `nvmlDeviceGetBoardId` / 函数 `nvmlDeviceGetBoardId`
```python
def nvmlDeviceGetBoardId(handle):
    c_id = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetBoardId")
    ret = fn(handle, byref(c_id))
    _nvmlCheckReturn(ret)
    return c_id.value
```
**EN:** This function drives `nvmlDeviceGetBoardId` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetBoardId`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3201-3206: Function `nvmlDeviceGetMultiGpuBoard` / 函数 `nvmlDeviceGetMultiGpuBoard`
```python
def nvmlDeviceGetMultiGpuBoard(handle):
    c_multiGpu = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMultiGpuBoard")
    ret = fn(handle, byref(c_multiGpu))
    _nvmlCheckReturn(ret)
    return c_multiGpu.value
```
**EN:** This function drives `nvmlDeviceGetMultiGpuBoard` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMultiGpuBoard`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3209-3214: Function `nvmlDeviceGetBrand` / 函数 `nvmlDeviceGetBrand`
```python
def nvmlDeviceGetBrand(handle):
    c_type = _nvmlBrandType_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetBrand")
    ret = fn(handle, byref(c_type))
    _nvmlCheckReturn(ret)
    return c_type.value
```
**EN:** This function drives `nvmlDeviceGetBrand` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetBrand`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3217-3222: Function `nvmlDeviceGetC2cModeInfoV1` / 函数 `nvmlDeviceGetC2cModeInfoV1`
```python
def nvmlDeviceGetC2cModeInfoV1(handle):
    c_info = c_nvmlC2cModeInfo_v1_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetC2cModeInfoV")
    ret = fn(handle, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlDeviceGetC2cModeInfoV1` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetC2cModeInfoV1`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3225-3226: Function `nvmlDeviceGetC2cModeInfoV` / 函数 `nvmlDeviceGetC2cModeInfoV`
```python
def nvmlDeviceGetC2cModeInfoV(handle):
    return nvmlDeviceGetC2cModeInfoV1(handle)
```
**EN:** This function drives `nvmlDeviceGetC2cModeInfoV` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetC2cModeInfoV`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3229-3235: Function `nvmlDeviceGetBoardPartNumber` / 函数 `nvmlDeviceGetBoardPartNumber`
```python
@convertStrBytes
def nvmlDeviceGetBoardPartNumber(handle):
    c_part_number = create_string_buffer(NVML_DEVICE_PART_NUMBER_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetBoardPartNumber")
    ret = fn(handle, c_part_number, c_uint(NVML_DEVICE_PART_NUMBER_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_part_number.value
```
**EN:** This function drives `nvmlDeviceGetBoardPartNumber` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetBoardPartNumber`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3238-3244: Function `nvmlDeviceGetSerial` / 函数 `nvmlDeviceGetSerial`
```python
@convertStrBytes
def nvmlDeviceGetSerial(handle):
    c_serial = create_string_buffer(NVML_DEVICE_SERIAL_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSerial")
    ret = fn(handle, c_serial, c_uint(NVML_DEVICE_SERIAL_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_serial.value
```
**EN:** This function drives `nvmlDeviceGetSerial` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSerial`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3247-3256: Function `nvmlDeviceGetModuleId` / 函数 `nvmlDeviceGetModuleId`
```python
def nvmlDeviceGetModuleId(handle, moduleId=c_uint()):
    isReference = type(moduleId) is not c_uint
    moduleIdRef = moduleId if isReference else byref(moduleId)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetModuleId")
    ret = fn(handle, moduleIdRef)
    if isReference:
        return ret
    else:
        _nvmlCheckReturn(ret)
        return moduleId.value
```
**EN:** This function drives `nvmlDeviceGetModuleId` with inputs such as `handle`, `moduleId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetModuleId`，主要处理 `handle`, `moduleId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3259-3265: Function `nvmlDeviceGetMemoryAffinity` / 函数 `nvmlDeviceGetMemoryAffinity`
```python
def nvmlDeviceGetMemoryAffinity(handle, nodeSetSize, scope):
    affinity_array = c_ulonglong * nodeSetSize
    c_affinity = affinity_array()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMemoryAffinity")
    ret = fn(handle, nodeSetSize, byref(c_affinity), _nvmlAffinityScope_t(scope))
    _nvmlCheckReturn(ret)
    return c_affinity
```
**EN:** This function drives `nvmlDeviceGetMemoryAffinity` with inputs such as `handle`, `nodeSetSize`, `scope`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMemoryAffinity`，主要处理 `handle`, `nodeSetSize`, `scope` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3268-3274: Function `nvmlDeviceGetCpuAffinityWithinScope` / 函数 `nvmlDeviceGetCpuAffinityWithinScope`
```python
def nvmlDeviceGetCpuAffinityWithinScope(handle, cpuSetSize, scope):
    affinity_array = c_ulonglong * cpuSetSize
    c_affinity = affinity_array()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCpuAffinityWithinScope")
    ret = fn(handle, cpuSetSize, byref(c_affinity), _nvmlAffinityScope_t(scope))
    _nvmlCheckReturn(ret)
    return c_affinity
```
**EN:** This function drives `nvmlDeviceGetCpuAffinityWithinScope` with inputs such as `handle`, `cpuSetSize`, `scope`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCpuAffinityWithinScope`，主要处理 `handle`, `cpuSetSize`, `scope` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3277-3283: Function `nvmlDeviceGetCpuAffinity` / 函数 `nvmlDeviceGetCpuAffinity`
```python
def nvmlDeviceGetCpuAffinity(handle, cpuSetSize):
    affinity_array = c_ulonglong * cpuSetSize
    c_affinity = affinity_array()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCpuAffinity")
    ret = fn(handle, cpuSetSize, byref(c_affinity))
    _nvmlCheckReturn(ret)
    return c_affinity
```
**EN:** This function drives `nvmlDeviceGetCpuAffinity` with inputs such as `handle`, `cpuSetSize`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCpuAffinity`，主要处理 `handle`, `cpuSetSize` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3286-3290: Function `nvmlDeviceSetCpuAffinity` / 函数 `nvmlDeviceSetCpuAffinity`
```python
def nvmlDeviceSetCpuAffinity(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetCpuAffinity")
    ret = fn(handle)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetCpuAffinity` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetCpuAffinity`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3293-3297: Function `nvmlDeviceClearCpuAffinity` / 函数 `nvmlDeviceClearCpuAffinity`
```python
def nvmlDeviceClearCpuAffinity(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceClearCpuAffinity")
    ret = fn(handle)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceClearCpuAffinity` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceClearCpuAffinity`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3300-3305: Function `nvmlDeviceGetNumaNodeId` / 函数 `nvmlDeviceGetNumaNodeId`
```python
def nvmlDeviceGetNumaNodeId(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNumaNodeId")
    node = c_int()
    ret = fn(handle, byref(node))
    _nvmlCheckReturn(ret)
    return node.value
```
**EN:** This function drives `nvmlDeviceGetNumaNodeId` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNumaNodeId`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3308-3313: Function `nvmlDeviceGetMinorNumber` / 函数 `nvmlDeviceGetMinorNumber`
```python
def nvmlDeviceGetMinorNumber(handle):
    c_minor_number = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMinorNumber")
    ret = fn(handle, byref(c_minor_number))
    _nvmlCheckReturn(ret)
    return c_minor_number.value
```
**EN:** This function drives `nvmlDeviceGetMinorNumber` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMinorNumber`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3316-3322: Function `nvmlDeviceGetUUID` / 函数 `nvmlDeviceGetUUID`
```python
@convertStrBytes
def nvmlDeviceGetUUID(handle):
    c_uuid = create_string_buffer(NVML_DEVICE_UUID_V2_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetUUID")
    ret = fn(handle, c_uuid, c_uint(NVML_DEVICE_UUID_V2_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_uuid.value
```
**EN:** This function drives `nvmlDeviceGetUUID` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetUUID`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3325-3336: Function `nvmlDeviceGetInforomVersion` / 函数 `nvmlDeviceGetInforomVersion`
```python
@convertStrBytes
def nvmlDeviceGetInforomVersion(handle, infoRomObject):
    c_version = create_string_buffer(NVML_DEVICE_INFOROM_VERSION_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetInforomVersion")
    ret = fn(
        handle,
        _nvmlInforomObject_t(infoRomObject),
        c_version,
        c_uint(NVML_DEVICE_INFOROM_VERSION_BUFFER_SIZE),
    )
    _nvmlCheckReturn(ret)
    return c_version.value
```
**EN:** This function drives `nvmlDeviceGetInforomVersion` with inputs such as `handle`, `infoRomObject`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetInforomVersion`，主要处理 `handle`, `infoRomObject` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3340-3346: Function `nvmlDeviceGetInforomImageVersion` / 函数 `nvmlDeviceGetInforomImageVersion`
```python
@convertStrBytes
def nvmlDeviceGetInforomImageVersion(handle):
    c_version = create_string_buffer(NVML_DEVICE_INFOROM_VERSION_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetInforomImageVersion")
    ret = fn(handle, c_version, c_uint(NVML_DEVICE_INFOROM_VERSION_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_version.value
```
**EN:** This function drives `nvmlDeviceGetInforomImageVersion` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetInforomImageVersion`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3350-3355: Function `nvmlDeviceGetInforomConfigurationChecksum` / 函数 `nvmlDeviceGetInforomConfigurationChecksum`
```python
def nvmlDeviceGetInforomConfigurationChecksum(handle):
    c_checksum = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetInforomConfigurationChecksum")
    ret = fn(handle, byref(c_checksum))
    _nvmlCheckReturn(ret)
    return c_checksum.value
```
**EN:** This function drives `nvmlDeviceGetInforomConfigurationChecksum` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetInforomConfigurationChecksum`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3359-3363: Function `nvmlDeviceValidateInforom` / 函数 `nvmlDeviceValidateInforom`
```python
def nvmlDeviceValidateInforom(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceValidateInforom")
    ret = fn(handle)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceValidateInforom` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceValidateInforom`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3366-3372: Function `nvmlDeviceGetLastBBXFlushTime` / 函数 `nvmlDeviceGetLastBBXFlushTime`
```python
def nvmlDeviceGetLastBBXFlushTime(handle):
    c_timestamp = c_ulonglong()
    c_durationUs = c_ulong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetLastBBXFlushTime")
    ret = fn(handle, byref(c_timestamp), byref(c_durationUs))
    _nvmlCheckReturn(ret)
    return [c_timestamp.value, c_durationUs.value]
```
**EN:** This function drives `nvmlDeviceGetLastBBXFlushTime` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetLastBBXFlushTime`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3375-3380: Function `nvmlDeviceGetDisplayMode` / 函数 `nvmlDeviceGetDisplayMode`
```python
def nvmlDeviceGetDisplayMode(handle):
    c_mode = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDisplayMode")
    ret = fn(handle, byref(c_mode))
    _nvmlCheckReturn(ret)
    return c_mode.value
```
**EN:** This function drives `nvmlDeviceGetDisplayMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDisplayMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3383-3388: Function `nvmlDeviceGetDisplayActive` / 函数 `nvmlDeviceGetDisplayActive`
```python
def nvmlDeviceGetDisplayActive(handle):
    c_mode = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDisplayActive")
    ret = fn(handle, byref(c_mode))
    _nvmlCheckReturn(ret)
    return c_mode.value
```
**EN:** This function drives `nvmlDeviceGetDisplayActive` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDisplayActive`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3391-3396: Function `nvmlDeviceGetPersistenceMode` / 函数 `nvmlDeviceGetPersistenceMode`
```python
def nvmlDeviceGetPersistenceMode(handle):
    c_state = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPersistenceMode")
    ret = fn(handle, byref(c_state))
    _nvmlCheckReturn(ret)
    return c_state.value
```
**EN:** This function drives `nvmlDeviceGetPersistenceMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPersistenceMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3399-3403: Function `nvmlDeviceGetPciInfoExt` / 函数 `nvmlDeviceGetPciInfoExt`
```python
def nvmlDeviceGetPciInfoExt(handle, c_info):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPciInfoExt")
    ret = fn(handle, c_info)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceGetPciInfoExt` with inputs such as `handle`, `c_info`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPciInfoExt`，主要处理 `handle`, `c_info` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3406-3411: Function `nvmlDeviceGetPciInfo_v3` / 函数 `nvmlDeviceGetPciInfo_v3`
```python
def nvmlDeviceGetPciInfo_v3(handle):
    c_info = nvmlPciInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPciInfo_v3")
    ret = fn(handle, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlDeviceGetPciInfo_v3` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPciInfo_v3`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3414-3415: Function `nvmlDeviceGetPciInfo` / 函数 `nvmlDeviceGetPciInfo`
```python
def nvmlDeviceGetPciInfo(handle):
    return nvmlDeviceGetPciInfo_v3(handle)
```
**EN:** This function drives `nvmlDeviceGetPciInfo` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPciInfo`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3418-3423: Function `nvmlDeviceGetClockInfo` / 函数 `nvmlDeviceGetClockInfo`
```python
def nvmlDeviceGetClockInfo(handle, type):
    c_clock = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetClockInfo")
    ret = fn(handle, _nvmlClockType_t(type), byref(c_clock))
    _nvmlCheckReturn(ret)
    return c_clock.value
```
**EN:** This function drives `nvmlDeviceGetClockInfo` with inputs such as `handle`, `type`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetClockInfo`，主要处理 `handle`, `type` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3427-3432: Function `nvmlDeviceGetMaxClockInfo` / 函数 `nvmlDeviceGetMaxClockInfo`
```python
def nvmlDeviceGetMaxClockInfo(handle, type):
    c_clock = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMaxClockInfo")
    ret = fn(handle, _nvmlClockType_t(type), byref(c_clock))
    _nvmlCheckReturn(ret)
    return c_clock.value
```
**EN:** This function drives `nvmlDeviceGetMaxClockInfo` with inputs such as `handle`, `type`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMaxClockInfo`，主要处理 `handle`, `type` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3436-3441: Function `nvmlDeviceGetApplicationsClock` / 函数 `nvmlDeviceGetApplicationsClock`
```python
def nvmlDeviceGetApplicationsClock(handle, type):
    c_clock = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetApplicationsClock")
    ret = fn(handle, _nvmlClockType_t(type), byref(c_clock))
    _nvmlCheckReturn(ret)
    return c_clock.value
```
**EN:** This function drives `nvmlDeviceGetApplicationsClock` with inputs such as `handle`, `type`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetApplicationsClock`，主要处理 `handle`, `type` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3444-3449: Function `nvmlDeviceGetMaxCustomerBoostClock` / 函数 `nvmlDeviceGetMaxCustomerBoostClock`
```python
def nvmlDeviceGetMaxCustomerBoostClock(handle, type):
    c_clock = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMaxCustomerBoostClock")
    ret = fn(handle, _nvmlClockType_t(type), byref(c_clock))
    _nvmlCheckReturn(ret)
    return c_clock.value
```
**EN:** This function drives `nvmlDeviceGetMaxCustomerBoostClock` with inputs such as `handle`, `type`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMaxCustomerBoostClock`，主要处理 `handle`, `type` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3452-3457: Function `nvmlDeviceGetClock` / 函数 `nvmlDeviceGetClock`
```python
def nvmlDeviceGetClock(handle, type, id):
    c_clock = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetClock")
    ret = fn(handle, _nvmlClockType_t(type), _nvmlClockId_t(id), byref(c_clock))
    _nvmlCheckReturn(ret)
    return c_clock.value
```
**EN:** This function drives `nvmlDeviceGetClock` with inputs such as `handle`, `type`, `id`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetClock`，主要处理 `handle`, `type`, `id` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3461-3466: Function `nvmlDeviceGetDefaultApplicationsClock` / 函数 `nvmlDeviceGetDefaultApplicationsClock`
```python
def nvmlDeviceGetDefaultApplicationsClock(handle, type):
    c_clock = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDefaultApplicationsClock")
    ret = fn(handle, _nvmlClockType_t(type), byref(c_clock))
    _nvmlCheckReturn(ret)
    return c_clock.value
```
**EN:** This function drives `nvmlDeviceGetDefaultApplicationsClock` with inputs such as `handle`, `type`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDefaultApplicationsClock`，主要处理 `handle`, `type` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3470-3495: Function `nvmlDeviceGetSupportedMemoryClocks` / 函数 `nvmlDeviceGetSupportedMemoryClocks`
```python
def nvmlDeviceGetSupportedMemoryClocks(handle):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSupportedMemoryClocks")
    ret = fn(handle, byref(c_count), None)

    if ret == NVML_SUCCESS:
        # special case, no clocks
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        clocks_array = c_uint * c_count.value
        c_clocks = clocks_array()

# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetSupportedMemoryClocks` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSupportedMemoryClocks`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3499-3524: Function `nvmlDeviceGetSupportedGraphicsClocks` / 函数 `nvmlDeviceGetSupportedGraphicsClocks`
```python
def nvmlDeviceGetSupportedGraphicsClocks(handle, memoryClockMHz):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSupportedGraphicsClocks")
    ret = fn(handle, c_uint(memoryClockMHz), byref(c_count), None)

    if ret == NVML_SUCCESS:
        # special case, no clocks
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        clocks_array = c_uint * c_count.value
        c_clocks = clocks_array()

# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetSupportedGraphicsClocks` with inputs such as `handle`, `memoryClockMHz`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSupportedGraphicsClocks`，主要处理 `handle`, `memoryClockMHz` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3527-3532: Function `nvmlDeviceGetFanSpeed` / 函数 `nvmlDeviceGetFanSpeed`
```python
def nvmlDeviceGetFanSpeed(handle):
    c_speed = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetFanSpeed")
    ret = fn(handle, byref(c_speed))
    _nvmlCheckReturn(ret)
    return c_speed.value
```
**EN:** This function drives `nvmlDeviceGetFanSpeed` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetFanSpeed`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3535-3540: Function `nvmlDeviceGetFanSpeed_v2` / 函数 `nvmlDeviceGetFanSpeed_v2`
```python
def nvmlDeviceGetFanSpeed_v2(handle, fan):
    c_speed = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetFanSpeed_v2")
    ret = fn(handle, fan, byref(c_speed))
    _nvmlCheckReturn(ret)
    return c_speed.value
```
**EN:** This function drives `nvmlDeviceGetFanSpeed_v2` with inputs such as `handle`, `fan`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetFanSpeed_v2`，主要处理 `handle`, `fan` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3543-3548: Class `c_nvmlFanSpeedInfo_t` / 类 `c_nvmlFanSpeedInfo_t`
```python
class c_nvmlFanSpeedInfo_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("fan", c_uint),
        ("speed", c_uint),
    ]
```
**EN:** This class models `c_nvmlFanSpeedInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlFanSpeedInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 3549-3551: Top-level configuration / 顶层配置
```python


nvmlFanSpeedInfo_v1 = 0x100000C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 3554-3561: Function `nvmlDeviceGetFanSpeedRPM` / 函数 `nvmlDeviceGetFanSpeedRPM`
```python
def nvmlDeviceGetFanSpeedRPM(handle):
    c_fanSpeed = c_nvmlFanSpeedInfo_t()
    c_fanSpeed.fan = 0
    c_fanSpeed.version = nvmlFanSpeedInfo_v1
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetFanSpeedRPM")
    ret = fn(handle, byref(c_fanSpeed))
    _nvmlCheckReturn(ret)
    return c_fanSpeed.speed
```
**EN:** This function drives `nvmlDeviceGetFanSpeedRPM` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetFanSpeedRPM`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3564-3569: Function `nvmlDeviceGetTargetFanSpeed` / 函数 `nvmlDeviceGetTargetFanSpeed`
```python
def nvmlDeviceGetTargetFanSpeed(handle, fan):
    c_speed = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTargetFanSpeed")
    ret = fn(handle, fan, byref(c_speed))
    _nvmlCheckReturn(ret)
    return c_speed.value
```
**EN:** This function drives `nvmlDeviceGetTargetFanSpeed` with inputs such as `handle`, `fan`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTargetFanSpeed`，主要处理 `handle`, `fan` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3572-3577: Function `nvmlDeviceGetNumFans` / 函数 `nvmlDeviceGetNumFans`
```python
def nvmlDeviceGetNumFans(device):
    c_numFans = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNumFans")
    ret = fn(device, byref(c_numFans))
    _nvmlCheckReturn(ret)
    return c_numFans.value
```
**EN:** This function drives `nvmlDeviceGetNumFans` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNumFans`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3580-3584: Function `nvmlDeviceSetDefaultFanSpeed_v2` / 函数 `nvmlDeviceSetDefaultFanSpeed_v2`
```python
def nvmlDeviceSetDefaultFanSpeed_v2(handle, index):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetDefaultFanSpeed_v2")
    ret = fn(handle, index)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetDefaultFanSpeed_v2` with inputs such as `handle`, `index`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetDefaultFanSpeed_v2`，主要处理 `handle`, `index` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3587-3594: Function `nvmlDeviceGetMinMaxFanSpeed` / 函数 `nvmlDeviceGetMinMaxFanSpeed`
```python
def nvmlDeviceGetMinMaxFanSpeed(handle, minSpeed=c_uint(), maxSpeed=c_uint()):
    isReference = (type(minSpeed) is not c_uint) or (type(maxSpeed) is not c_uint)
    minSpeedRef = minSpeed if isReference else byref(minSpeed)
    maxSpeedRef = maxSpeed if isReference else byref(maxSpeed)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMinMaxFanSpeed")
    ret = fn(handle, minSpeedRef, maxSpeedRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else [minSpeed.value, maxSpeed.value]
```
**EN:** This function drives `nvmlDeviceGetMinMaxFanSpeed` with inputs such as `handle`, `minSpeed`, `maxSpeed`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMinMaxFanSpeed`，主要处理 `handle`, `minSpeed`, `maxSpeed` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3597-3603: Function `nvmlDeviceGetFanControlPolicy_v2` / 函数 `nvmlDeviceGetFanControlPolicy_v2`
```python
def nvmlDeviceGetFanControlPolicy_v2(handle, fan, fanControlPolicy=c_uint()):
    isReference = type(fanControlPolicy) is not c_uint
    fanControlPolicyRef = fanControlPolicy if isReference else byref(fanControlPolicy)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetFanControlPolicy_v2")
    ret = fn(handle, fan, fanControlPolicyRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else fanControlPolicy.value
```
**EN:** This function drives `nvmlDeviceGetFanControlPolicy_v2` with inputs such as `handle`, `fan`, `fanControlPolicy`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetFanControlPolicy_v2`，主要处理 `handle`, `fan`, `fanControlPolicy` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3606-3610: Function `nvmlDeviceSetFanControlPolicy` / 函数 `nvmlDeviceSetFanControlPolicy`
```python
def nvmlDeviceSetFanControlPolicy(handle, fan, fanControlPolicy):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetFanControlPolicy")
    ret = fn(handle, fan, _nvmlFanControlPolicy_t(fanControlPolicy))
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetFanControlPolicy` with inputs such as `handle`, `fan`, `fanControlPolicy`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetFanControlPolicy`，主要处理 `handle`, `fan`, `fanControlPolicy` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3613-3618: Class `c_nvmlTemperature_v1_t` / 类 `c_nvmlTemperature_v1_t`
```python
class c_nvmlTemperature_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("sensorType", _nvmlTemperatureSensors_t),
        ("temperature", c_int),
    ]
```
**EN:** This class models `c_nvmlTemperature_v1_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlTemperature_v1_t`，并继承/扩展 `_PrintableStructure`。

### Lines 3619-3621: Top-level configuration / 顶层配置
```python


nvmlTemperature_v1 = 0x100000C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 3624-3631: Function `nvmlDeviceGetTemperatureV1` / 函数 `nvmlDeviceGetTemperatureV1`
```python
def nvmlDeviceGetTemperatureV1(handle, sensor):
    c_temp = c_nvmlTemperature_v1_t()
    c_temp.version = nvmlTemperature_v1
    c_temp.sensorType = _nvmlTemperatureSensors_t(sensor)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTemperatureV")
    ret = fn(handle, byref(c_temp))
    _nvmlCheckReturn(ret)
    return c_temp.temperature
```
**EN:** This function drives `nvmlDeviceGetTemperatureV1` with inputs such as `handle`, `sensor`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTemperatureV1`，主要处理 `handle`, `sensor` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3634-3638: Function `nvmlDeviceGetTemperatureV` / 函数 `nvmlDeviceGetTemperatureV`
```python
def nvmlDeviceGetTemperatureV(handle, sensor, version=nvmlTemperature_v1):
    if version == nvmlTemperature_v1:
        return nvmlDeviceGetTemperatureV1(handle, sensor)
    else:
        raise NVMLError(NVML_ERROR_ARGUMENT_VERSION_MISMATCH)
```
**EN:** This function drives `nvmlDeviceGetTemperatureV` with inputs such as `handle`, `sensor`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTemperatureV`，主要处理 `handle`, `sensor`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3642-3647: Function `nvmlDeviceGetTemperature` / 函数 `nvmlDeviceGetTemperature`
```python
def nvmlDeviceGetTemperature(handle, sensor):
    c_temp = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTemperature")
    ret = fn(handle, _nvmlTemperatureSensors_t(sensor), byref(c_temp))
    _nvmlCheckReturn(ret)
    return c_temp.value
```
**EN:** This function drives `nvmlDeviceGetTemperature` with inputs such as `handle`, `sensor`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTemperature`，主要处理 `handle`, `sensor` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3650-3655: Function `nvmlDeviceGetTemperatureThreshold` / 函数 `nvmlDeviceGetTemperatureThreshold`
```python
def nvmlDeviceGetTemperatureThreshold(handle, threshold):
    c_temp = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTemperatureThreshold")
    ret = fn(handle, _nvmlTemperatureThresholds_t(threshold), byref(c_temp))
    _nvmlCheckReturn(ret)
    return c_temp.value
```
**EN:** This function drives `nvmlDeviceGetTemperatureThreshold` with inputs such as `handle`, `threshold`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTemperatureThreshold`，主要处理 `handle`, `threshold` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3658-3664: Function `nvmlDeviceSetTemperatureThreshold` / 函数 `nvmlDeviceSetTemperatureThreshold`
```python
def nvmlDeviceSetTemperatureThreshold(handle, threshold, temp):
    c_temp = c_uint()
    c_temp.value = temp
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetTemperatureThreshold")
    ret = fn(handle, _nvmlTemperatureThresholds_t(threshold), byref(c_temp))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetTemperatureThreshold` with inputs such as `handle`, `threshold`, `temp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetTemperatureThreshold`，主要处理 `handle`, `threshold`, `temp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3667-3673: Function `nvmlDeviceGetMarginTemperature` / 函数 `nvmlDeviceGetMarginTemperature`
```python
def nvmlDeviceGetMarginTemperature(handle):
    c_marginTempInfo = c_nvmlMarginTemperature_v1_t()
    c_marginTempInfo.version = nvmlMarginTemperature_v1
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMarginTemperature")
    ret = fn(handle, byref(c_marginTempInfo))
    _nvmlCheckReturn(ret)
    return c_marginTempInfo.marginTemperature
```
**EN:** This function drives `nvmlDeviceGetMarginTemperature` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMarginTemperature`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3677-3682: Function `nvmlDeviceGetPowerState` / 函数 `nvmlDeviceGetPowerState`
```python
def nvmlDeviceGetPowerState(handle):
    c_pstate = _nvmlPstates_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPowerState")
    ret = fn(handle, byref(c_pstate))
    _nvmlCheckReturn(ret)
    return c_pstate.value
```
**EN:** This function drives `nvmlDeviceGetPowerState` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPowerState`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3685-3690: Function `nvmlDeviceGetPerformanceState` / 函数 `nvmlDeviceGetPerformanceState`
```python
def nvmlDeviceGetPerformanceState(handle):
    c_pstate = _nvmlPstates_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPerformanceState")
    ret = fn(handle, byref(c_pstate))
    _nvmlCheckReturn(ret)
    return c_pstate.value
```
**EN:** This function drives `nvmlDeviceGetPerformanceState` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPerformanceState`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3693-3698: Function `nvmlDeviceGetPowerManagementMode` / 函数 `nvmlDeviceGetPowerManagementMode`
```python
def nvmlDeviceGetPowerManagementMode(handle):
    c_pcapMode = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPowerManagementMode")
    ret = fn(handle, byref(c_pcapMode))
    _nvmlCheckReturn(ret)
    return c_pcapMode.value
```
**EN:** This function drives `nvmlDeviceGetPowerManagementMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPowerManagementMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3701-3706: Function `nvmlDeviceGetPowerManagementLimit` / 函数 `nvmlDeviceGetPowerManagementLimit`
```python
def nvmlDeviceGetPowerManagementLimit(handle):
    c_limit = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPowerManagementLimit")
    ret = fn(handle, byref(c_limit))
    _nvmlCheckReturn(ret)
    return c_limit.value
```
**EN:** This function drives `nvmlDeviceGetPowerManagementLimit` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPowerManagementLimit`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3710-3716: Function `nvmlDeviceGetPowerManagementLimitConstraints` / 函数 `nvmlDeviceGetPowerManagementLimitConstraints`
```python
def nvmlDeviceGetPowerManagementLimitConstraints(handle):
    c_minLimit = c_uint()
    c_maxLimit = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPowerManagementLimitConstraints")
    ret = fn(handle, byref(c_minLimit), byref(c_maxLimit))
    _nvmlCheckReturn(ret)
    return [c_minLimit.value, c_maxLimit.value]
```
**EN:** This function drives `nvmlDeviceGetPowerManagementLimitConstraints` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPowerManagementLimitConstraints`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3720-3725: Function `nvmlDeviceGetPowerManagementDefaultLimit` / 函数 `nvmlDeviceGetPowerManagementDefaultLimit`
```python
def nvmlDeviceGetPowerManagementDefaultLimit(handle):
    c_limit = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPowerManagementDefaultLimit")
    ret = fn(handle, byref(c_limit))
    _nvmlCheckReturn(ret)
    return c_limit.value
```
**EN:** This function drives `nvmlDeviceGetPowerManagementDefaultLimit` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPowerManagementDefaultLimit`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3729-3734: Function `nvmlDeviceGetEnforcedPowerLimit` / 函数 `nvmlDeviceGetEnforcedPowerLimit`
```python
def nvmlDeviceGetEnforcedPowerLimit(handle):
    c_limit = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetEnforcedPowerLimit")
    ret = fn(handle, byref(c_limit))
    _nvmlCheckReturn(ret)
    return c_limit.value
```
**EN:** This function drives `nvmlDeviceGetEnforcedPowerLimit` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetEnforcedPowerLimit`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3737-3742: Function `nvmlDeviceGetPowerUsage` / 函数 `nvmlDeviceGetPowerUsage`
```python
def nvmlDeviceGetPowerUsage(handle):
    c_watts = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPowerUsage")
    ret = fn(handle, byref(c_watts))
    _nvmlCheckReturn(ret)
    return c_watts.value
```
**EN:** This function drives `nvmlDeviceGetPowerUsage` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPowerUsage`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3745-3750: Function `nvmlDeviceGetTotalEnergyConsumption` / 函数 `nvmlDeviceGetTotalEnergyConsumption`
```python
def nvmlDeviceGetTotalEnergyConsumption(handle):
    c_millijoules = c_uint64()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTotalEnergyConsumption")
    ret = fn(handle, byref(c_millijoules))
    _nvmlCheckReturn(ret)
    return c_millijoules.value
```
**EN:** This function drives `nvmlDeviceGetTotalEnergyConsumption` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTotalEnergyConsumption`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3754-3760: Function `nvmlDeviceGetGpuOperationMode` / 函数 `nvmlDeviceGetGpuOperationMode`
```python
def nvmlDeviceGetGpuOperationMode(handle):
    c_currState = _nvmlGpuOperationMode_t()
    c_pendingState = _nvmlGpuOperationMode_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuOperationMode")
    ret = fn(handle, byref(c_currState), byref(c_pendingState))
    _nvmlCheckReturn(ret)
    return [c_currState.value, c_pendingState.value]
```
**EN:** This function drives `nvmlDeviceGetGpuOperationMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuOperationMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3764-3765: Function `nvmlDeviceGetCurrentGpuOperationMode` / 函数 `nvmlDeviceGetCurrentGpuOperationMode`
```python
def nvmlDeviceGetCurrentGpuOperationMode(handle):
    return nvmlDeviceGetGpuOperationMode(handle)[0]
```
**EN:** This function drives `nvmlDeviceGetCurrentGpuOperationMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrentGpuOperationMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3769-3770: Function `nvmlDeviceGetPendingGpuOperationMode` / 函数 `nvmlDeviceGetPendingGpuOperationMode`
```python
def nvmlDeviceGetPendingGpuOperationMode(handle):
    return nvmlDeviceGetGpuOperationMode(handle)[1]
```
**EN:** This function drives `nvmlDeviceGetPendingGpuOperationMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPendingGpuOperationMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3773-3783: Function `nvmlDeviceGetMemoryInfo` / 函数 `nvmlDeviceGetMemoryInfo`
```python
def nvmlDeviceGetMemoryInfo(handle, version=None):
    if not version:
        c_memory = c_nvmlMemory_t()
        fn = _nvmlGetFunctionPointer("nvmlDeviceGetMemoryInfo")
    else:
        c_memory = c_nvmlMemory_v2_t()
        c_memory.version = version
        fn = _nvmlGetFunctionPointer("nvmlDeviceGetMemoryInfo_v2")
    ret = fn(handle, byref(c_memory))
    _nvmlCheckReturn(ret)
    return c_memory
```
**EN:** This function drives `nvmlDeviceGetMemoryInfo` with inputs such as `handle`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMemoryInfo`，主要处理 `handle`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3786-3791: Function `nvmlDeviceGetBAR1MemoryInfo` / 函数 `nvmlDeviceGetBAR1MemoryInfo`
```python
def nvmlDeviceGetBAR1MemoryInfo(handle):
    c_bar1_memory = c_nvmlBAR1Memory_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetBAR1MemoryInfo")
    ret = fn(handle, byref(c_bar1_memory))
    _nvmlCheckReturn(ret)
    return c_bar1_memory
```
**EN:** This function drives `nvmlDeviceGetBAR1MemoryInfo` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetBAR1MemoryInfo`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3794-3799: Function `nvmlDeviceGetComputeMode` / 函数 `nvmlDeviceGetComputeMode`
```python
def nvmlDeviceGetComputeMode(handle):
    c_mode = _nvmlComputeMode_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetComputeMode")
    ret = fn(handle, byref(c_mode))
    _nvmlCheckReturn(ret)
    return c_mode.value
```
**EN:** This function drives `nvmlDeviceGetComputeMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetComputeMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3802-3808: Function `nvmlDeviceGetCudaComputeCapability` / 函数 `nvmlDeviceGetCudaComputeCapability`
```python
def nvmlDeviceGetCudaComputeCapability(handle):
    c_major = c_int()
    c_minor = c_int()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCudaComputeCapability")
    ret = fn(handle, byref(c_major), byref(c_minor))
    _nvmlCheckReturn(ret)
    return (c_major.value, c_minor.value)
```
**EN:** This function drives `nvmlDeviceGetCudaComputeCapability` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCudaComputeCapability`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3811-3817: Function `nvmlDeviceGetEccMode` / 函数 `nvmlDeviceGetEccMode`
```python
def nvmlDeviceGetEccMode(handle):
    c_currState = _nvmlEnableState_t()
    c_pendingState = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetEccMode")
    ret = fn(handle, byref(c_currState), byref(c_pendingState))
    _nvmlCheckReturn(ret)
    return [c_currState.value, c_pendingState.value]
```
**EN:** This function drives `nvmlDeviceGetEccMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetEccMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3821-3822: Function `nvmlDeviceGetCurrentEccMode` / 函数 `nvmlDeviceGetCurrentEccMode`
```python
def nvmlDeviceGetCurrentEccMode(handle):
    return nvmlDeviceGetEccMode(handle)[0]
```
**EN:** This function drives `nvmlDeviceGetCurrentEccMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrentEccMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3826-3827: Function `nvmlDeviceGetPendingEccMode` / 函数 `nvmlDeviceGetPendingEccMode`
```python
def nvmlDeviceGetPendingEccMode(handle):
    return nvmlDeviceGetEccMode(handle)[1]
```
**EN:** This function drives `nvmlDeviceGetPendingEccMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPendingEccMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3830-3835: Function `nvmlDeviceGetDefaultEccMode` / 函数 `nvmlDeviceGetDefaultEccMode`
```python
def nvmlDeviceGetDefaultEccMode(handle):
    c_defaultState = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDefaultEccMode")
    ret = fn(handle, byref(c_defaultState))
    _nvmlCheckReturn(ret)
    return [c_defaultState.value]
```
**EN:** This function drives `nvmlDeviceGetDefaultEccMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDefaultEccMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3838-3848: Function `nvmlDeviceGetTotalEccErrors` / 函数 `nvmlDeviceGetTotalEccErrors`
```python
def nvmlDeviceGetTotalEccErrors(handle, errorType, counterType):
    c_count = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTotalEccErrors")
    ret = fn(
        handle,
        _nvmlMemoryErrorType_t(errorType),
        _nvmlEccCounterType_t(counterType),
        byref(c_count),
    )
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlDeviceGetTotalEccErrors` with inputs such as `handle`, `errorType`, `counterType`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTotalEccErrors`，主要处理 `handle`, `errorType`, `counterType` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3852-3862: Function `nvmlDeviceGetDetailedEccErrors` / 函数 `nvmlDeviceGetDetailedEccErrors`
```python
def nvmlDeviceGetDetailedEccErrors(handle, errorType, counterType):
    c_counts = c_nvmlEccErrorCounts_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDetailedEccErrors")
    ret = fn(
        handle,
        _nvmlMemoryErrorType_t(errorType),
        _nvmlEccCounterType_t(counterType),
        byref(c_counts),
    )
    _nvmlCheckReturn(ret)
    return c_counts
```
**EN:** This function drives `nvmlDeviceGetDetailedEccErrors` with inputs such as `handle`, `errorType`, `counterType`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDetailedEccErrors`，主要处理 `handle`, `errorType`, `counterType` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3866-3877: Function `nvmlDeviceGetMemoryErrorCounter` / 函数 `nvmlDeviceGetMemoryErrorCounter`
```python
def nvmlDeviceGetMemoryErrorCounter(handle, errorType, counterType, locationType):
    c_count = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMemoryErrorCounter")
    ret = fn(
        handle,
        _nvmlMemoryErrorType_t(errorType),
        _nvmlEccCounterType_t(counterType),
        _nvmlMemoryLocation_t(locationType),
        byref(c_count),
    )
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlDeviceGetMemoryErrorCounter` with inputs such as `handle`, `errorType`, `counterType`, `locationType`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMemoryErrorCounter`，主要处理 `handle`, `errorType`, `counterType`, `locationType` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3880-3885: Function `nvmlDeviceGetUtilizationRates` / 函数 `nvmlDeviceGetUtilizationRates`
```python
def nvmlDeviceGetUtilizationRates(handle):
    c_util = c_nvmlUtilization_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetUtilizationRates")
    ret = fn(handle, byref(c_util))
    _nvmlCheckReturn(ret)
    return c_util
```
**EN:** This function drives `nvmlDeviceGetUtilizationRates` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetUtilizationRates`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3888-3894: Function `nvmlDeviceGetEncoderUtilization` / 函数 `nvmlDeviceGetEncoderUtilization`
```python
def nvmlDeviceGetEncoderUtilization(handle):
    c_util = c_uint()
    c_samplingPeriod = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetEncoderUtilization")
    ret = fn(handle, byref(c_util), byref(c_samplingPeriod))
    _nvmlCheckReturn(ret)
    return [c_util.value, c_samplingPeriod.value]
```
**EN:** This function drives `nvmlDeviceGetEncoderUtilization` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetEncoderUtilization`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3897-3903: Function `nvmlDeviceGetDecoderUtilization` / 函数 `nvmlDeviceGetDecoderUtilization`
```python
def nvmlDeviceGetDecoderUtilization(handle):
    c_util = c_uint()
    c_samplingPeriod = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDecoderUtilization")
    ret = fn(handle, byref(c_util), byref(c_samplingPeriod))
    _nvmlCheckReturn(ret)
    return [c_util.value, c_samplingPeriod.value]
```
**EN:** This function drives `nvmlDeviceGetDecoderUtilization` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDecoderUtilization`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3906-3912: Function `nvmlDeviceGetJpgUtilization` / 函数 `nvmlDeviceGetJpgUtilization`
```python
def nvmlDeviceGetJpgUtilization(handle):
    c_util = c_uint()
    c_samplingPeriod = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetJpgUtilization")
    ret = fn(handle, byref(c_util), byref(c_samplingPeriod))
    _nvmlCheckReturn(ret)
    return [c_util.value, c_samplingPeriod.value]
```
**EN:** This function drives `nvmlDeviceGetJpgUtilization` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetJpgUtilization`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3915-3921: Function `nvmlDeviceGetOfaUtilization` / 函数 `nvmlDeviceGetOfaUtilization`
```python
def nvmlDeviceGetOfaUtilization(handle):
    c_util = c_uint()
    c_samplingPeriod = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetOfaUtilization")
    ret = fn(handle, byref(c_util), byref(c_samplingPeriod))
    _nvmlCheckReturn(ret)
    return [c_util.value, c_samplingPeriod.value]
```
**EN:** This function drives `nvmlDeviceGetOfaUtilization` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetOfaUtilization`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3924-3929: Function `nvmlDeviceGetPcieReplayCounter` / 函数 `nvmlDeviceGetPcieReplayCounter`
```python
def nvmlDeviceGetPcieReplayCounter(handle):
    c_replay = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPcieReplayCounter")
    ret = fn(handle, byref(c_replay))
    _nvmlCheckReturn(ret)
    return c_replay.value
```
**EN:** This function drives `nvmlDeviceGetPcieReplayCounter` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPcieReplayCounter`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3932-3938: Function `nvmlDeviceGetDriverModel` / 函数 `nvmlDeviceGetDriverModel`
```python
def nvmlDeviceGetDriverModel(handle):
    c_currModel = _nvmlDriverModel_t()
    c_pendingModel = _nvmlDriverModel_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDriverModel")
    ret = fn(handle, byref(c_currModel), byref(c_pendingModel))
    _nvmlCheckReturn(ret)
    return [c_currModel.value, c_pendingModel.value]
```
**EN:** This function drives `nvmlDeviceGetDriverModel` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDriverModel`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3942-3943: Function `nvmlDeviceGetCurrentDriverModel` / 函数 `nvmlDeviceGetCurrentDriverModel`
```python
def nvmlDeviceGetCurrentDriverModel(handle):
    return nvmlDeviceGetDriverModel(handle)[0]
```
**EN:** This function drives `nvmlDeviceGetCurrentDriverModel` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrentDriverModel`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3947-3948: Function `nvmlDeviceGetPendingDriverModel` / 函数 `nvmlDeviceGetPendingDriverModel`
```python
def nvmlDeviceGetPendingDriverModel(handle):
    return nvmlDeviceGetDriverModel(handle)[1]
```
**EN:** This function drives `nvmlDeviceGetPendingDriverModel` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPendingDriverModel`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3952-3958: Function `nvmlDeviceGetVbiosVersion` / 函数 `nvmlDeviceGetVbiosVersion`
```python
@convertStrBytes
def nvmlDeviceGetVbiosVersion(handle):
    c_version = create_string_buffer(NVML_DEVICE_VBIOS_VERSION_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVbiosVersion")
    ret = fn(handle, c_version, c_uint(NVML_DEVICE_VBIOS_VERSION_BUFFER_SIZE))
    _nvmlCheckReturn(ret)
    return c_version.value
```
**EN:** This function drives `nvmlDeviceGetVbiosVersion` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVbiosVersion`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3962-3990: Function `nvmlDeviceGetComputeRunningProcesses_v2` / 函数 `nvmlDeviceGetComputeRunningProcesses_v2`
```python
def nvmlDeviceGetComputeRunningProcesses_v2(handle):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetComputeRunningProcesses_v2")
    ret = fn(handle, byref(c_count), None)
    if ret == NVML_SUCCESS:
        # special case, no running processes
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        # oversize the array in case more processes are created
        c_count.value = c_count.value * 2 + 5
        proc_array = c_nvmlProcessInfo_v2_t * c_count.value
        c_procs = proc_array()
# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetComputeRunningProcesses_v2` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetComputeRunningProcesses_v2`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 3994-4026: Function `nvmlDeviceGetComputeRunningProcesses_v3` / 函数 `nvmlDeviceGetComputeRunningProcesses_v3`
```python
def nvmlDeviceGetComputeRunningProcesses_v3(handle):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetComputeRunningProcesses_v3")
    ret = fn(handle, byref(c_count), None)

    if ret == NVML_SUCCESS:
        # special case, no running processes
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        # oversize the array in case more processes are created
        c_count.value = c_count.value * 2 + 5
        proc_array = c_nvmlProcessInfo_v3_t * c_count.value
# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetComputeRunningProcesses_v3` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetComputeRunningProcesses_v3`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4029-4031: Function `nvmlDeviceGetComputeRunningProcesses` / 函数 `nvmlDeviceGetComputeRunningProcesses`
```python
@throwOnVersionMismatch
def nvmlDeviceGetComputeRunningProcesses(handle):
    return nvmlDeviceGetComputeRunningProcesses_v3(handle)
```
**EN:** This function drives `nvmlDeviceGetComputeRunningProcesses` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetComputeRunningProcesses`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4034-4062: Function `nvmlDeviceGetGraphicsRunningProcesses_v2` / 函数 `nvmlDeviceGetGraphicsRunningProcesses_v2`
```python
def nvmlDeviceGetGraphicsRunningProcesses_v2(handle):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGraphicsRunningProcesses_v2")
    ret = fn(handle, byref(c_count), None)
    if ret == NVML_SUCCESS:
        # special case, no running processes
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        # oversize the array in case more processes are created
        c_count.value = c_count.value * 2 + 5
        proc_array = c_nvmlProcessInfo_v2_t * c_count.value
        c_procs = proc_array()
# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetGraphicsRunningProcesses_v2` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGraphicsRunningProcesses_v2`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4065-4097: Function `nvmlDeviceGetGraphicsRunningProcesses_v3` / 函数 `nvmlDeviceGetGraphicsRunningProcesses_v3`
```python
def nvmlDeviceGetGraphicsRunningProcesses_v3(handle):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGraphicsRunningProcesses_v3")
    ret = fn(handle, byref(c_count), None)

    if ret == NVML_SUCCESS:
        # special case, no running processes
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        # oversize the array in case more processes are created
        c_count.value = c_count.value * 2 + 5
        proc_array = c_nvmlProcessInfo_v3_t * c_count.value
# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetGraphicsRunningProcesses_v3` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGraphicsRunningProcesses_v3`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4100-4102: Function `nvmlDeviceGetGraphicsRunningProcesses` / 函数 `nvmlDeviceGetGraphicsRunningProcesses`
```python
@throwOnVersionMismatch
def nvmlDeviceGetGraphicsRunningProcesses(handle):
    return nvmlDeviceGetGraphicsRunningProcesses_v3(handle)
```
**EN:** This function drives `nvmlDeviceGetGraphicsRunningProcesses` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGraphicsRunningProcesses`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4105-4107: Function `nvmlDeviceGetMPSComputeRunningProcesses` / 函数 `nvmlDeviceGetMPSComputeRunningProcesses`
```python
@throwOnVersionMismatch
def nvmlDeviceGetMPSComputeRunningProcesses(handle):
    return nvmlDeviceGetMPSComputeRunningProcesses_v3(handle)
```
**EN:** This function drives `nvmlDeviceGetMPSComputeRunningProcesses` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMPSComputeRunningProcesses`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4110-4142: Function `nvmlDeviceGetMPSComputeRunningProcesses_v2` / 函数 `nvmlDeviceGetMPSComputeRunningProcesses_v2`
```python
def nvmlDeviceGetMPSComputeRunningProcesses_v2(handle):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMPSComputeRunningProcesses_v2")
    ret = fn(handle, byref(c_count), None)

    if ret == NVML_SUCCESS:
        # special case, no running processes
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        # oversize the array in case more processes are created
        c_count.value = c_count.value * 2 + 5
        proc_array = c_nvmlProcessInfo_v2_t * c_count.value
# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetMPSComputeRunningProcesses_v2` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMPSComputeRunningProcesses_v2`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4145-4177: Function `nvmlDeviceGetMPSComputeRunningProcesses_v3` / 函数 `nvmlDeviceGetMPSComputeRunningProcesses_v3`
```python
def nvmlDeviceGetMPSComputeRunningProcesses_v3(handle):
    # first call to get the size
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMPSComputeRunningProcesses_v3")
    ret = fn(handle, byref(c_count), None)

    if ret == NVML_SUCCESS:
        # special case, no running processes
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        # oversize the array in case more processes are created
        c_count.value = c_count.value * 2 + 5
        proc_array = c_nvmlProcessInfo_v3_t * c_count.value
# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetMPSComputeRunningProcesses_v3` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMPSComputeRunningProcesses_v3`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4180-4215: Function `nvmlDeviceGetRunningProcessDetailList` / 函数 `nvmlDeviceGetRunningProcessDetailList`
```python
def nvmlDeviceGetRunningProcessDetailList(handle, version, mode):
    c_processDetailList = c_nvmlProcessDetailList_t()
    c_processDetailList.version = version
    c_processDetailList.mode = mode

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetRunningProcessDetailList")

    # first call to get the size
    ret = fn(handle, byref(c_processDetailList))
    if ret == NVML_SUCCESS:
        # special case, no running processes
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        c_procs = c_nvmlProcessDetail_v1_t * c_processDetailList.numProcArrayEntries
# ...
        return procs
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetRunningProcessDetailList` with inputs such as `handle`, `version`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetRunningProcessDetailList`，主要处理 `handle`, `version`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4218-4224: Function `nvmlDeviceGetAutoBoostedClocksEnabled` / 函数 `nvmlDeviceGetAutoBoostedClocksEnabled`
```python
def nvmlDeviceGetAutoBoostedClocksEnabled(handle):
    c_isEnabled = _nvmlEnableState_t()
    c_defaultIsEnabled = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAutoBoostedClocksEnabled")
    ret = fn(handle, byref(c_isEnabled), byref(c_defaultIsEnabled))
    _nvmlCheckReturn(ret)
    return [c_isEnabled.value, c_defaultIsEnabled.value]
```
**EN:** This function drives `nvmlDeviceGetAutoBoostedClocksEnabled` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAutoBoostedClocksEnabled`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4229-4233: Function `nvmlUnitSetLedState` / 函数 `nvmlUnitSetLedState`
```python
def nvmlUnitSetLedState(unit, color):
    fn = _nvmlGetFunctionPointer("nvmlUnitSetLedState")
    ret = fn(unit, _nvmlLedColor_t(color))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlUnitSetLedState` with inputs such as `unit`, `color`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlUnitSetLedState`，主要处理 `unit`, `color` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4236-4240: Function `nvmlDeviceSetPersistenceMode` / 函数 `nvmlDeviceSetPersistenceMode`
```python
def nvmlDeviceSetPersistenceMode(handle, mode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetPersistenceMode")
    ret = fn(handle, _nvmlEnableState_t(mode))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetPersistenceMode` with inputs such as `handle`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetPersistenceMode`，主要处理 `handle`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4243-4247: Function `nvmlDeviceSetComputeMode` / 函数 `nvmlDeviceSetComputeMode`
```python
def nvmlDeviceSetComputeMode(handle, mode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetComputeMode")
    ret = fn(handle, _nvmlComputeMode_t(mode))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetComputeMode` with inputs such as `handle`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetComputeMode`，主要处理 `handle`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4250-4254: Function `nvmlDeviceSetEccMode` / 函数 `nvmlDeviceSetEccMode`
```python
def nvmlDeviceSetEccMode(handle, mode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetEccMode")
    ret = fn(handle, _nvmlEnableState_t(mode))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetEccMode` with inputs such as `handle`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetEccMode`，主要处理 `handle`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4257-4261: Function `nvmlDeviceClearEccErrorCounts` / 函数 `nvmlDeviceClearEccErrorCounts`
```python
def nvmlDeviceClearEccErrorCounts(handle, counterType):
    fn = _nvmlGetFunctionPointer("nvmlDeviceClearEccErrorCounts")
    ret = fn(handle, _nvmlEccCounterType_t(counterType))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceClearEccErrorCounts` with inputs such as `handle`, `counterType`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceClearEccErrorCounts`，主要处理 `handle`, `counterType` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4264-4268: Function `nvmlDeviceSetDriverModel` / 函数 `nvmlDeviceSetDriverModel`
```python
def nvmlDeviceSetDriverModel(handle, model):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetDriverModel")
    ret = fn(handle, _nvmlDriverModel_t(model))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetDriverModel` with inputs such as `handle`, `model`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetDriverModel`，主要处理 `handle`, `model` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4271-4275: Function `nvmlDeviceSetAutoBoostedClocksEnabled` / 函数 `nvmlDeviceSetAutoBoostedClocksEnabled`
```python
def nvmlDeviceSetAutoBoostedClocksEnabled(handle, enabled):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetAutoBoostedClocksEnabled")
    ret = fn(handle, _nvmlEnableState_t(enabled))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetAutoBoostedClocksEnabled` with inputs such as `handle`, `enabled`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetAutoBoostedClocksEnabled`，主要处理 `handle`, `enabled` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4279-4283: Function `nvmlDeviceSetDefaultAutoBoostedClocksEnabled` / 函数 `nvmlDeviceSetDefaultAutoBoostedClocksEnabled`
```python
def nvmlDeviceSetDefaultAutoBoostedClocksEnabled(handle, enabled, flags):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetDefaultAutoBoostedClocksEnabled")
    ret = fn(handle, _nvmlEnableState_t(enabled), c_uint(flags))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetDefaultAutoBoostedClocksEnabled` with inputs such as `handle`, `enabled`, `flags`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetDefaultAutoBoostedClocksEnabled`，主要处理 `handle`, `enabled`, `flags` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4287-4291: Function `nvmlDeviceSetGpuLockedClocks` / 函数 `nvmlDeviceSetGpuLockedClocks`
```python
def nvmlDeviceSetGpuLockedClocks(handle, minGpuClockMHz, maxGpuClockMHz):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetGpuLockedClocks")
    ret = fn(handle, c_uint(minGpuClockMHz), c_uint(maxGpuClockMHz))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetGpuLockedClocks` with inputs such as `handle`, `minGpuClockMHz`, `maxGpuClockMHz`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetGpuLockedClocks`，主要处理 `handle`, `minGpuClockMHz`, `maxGpuClockMHz` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4294-4298: Function `nvmlDeviceResetGpuLockedClocks` / 函数 `nvmlDeviceResetGpuLockedClocks`
```python
def nvmlDeviceResetGpuLockedClocks(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceResetGpuLockedClocks")
    ret = fn(handle)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceResetGpuLockedClocks` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceResetGpuLockedClocks`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4301-4305: Function `nvmlDeviceSetMemoryLockedClocks` / 函数 `nvmlDeviceSetMemoryLockedClocks`
```python
def nvmlDeviceSetMemoryLockedClocks(handle, minMemClockMHz, maxMemClockMHz):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetMemoryLockedClocks")
    ret = fn(handle, c_uint(minMemClockMHz), c_uint(maxMemClockMHz))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetMemoryLockedClocks` with inputs such as `handle`, `minMemClockMHz`, `maxMemClockMHz`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetMemoryLockedClocks`，主要处理 `handle`, `minMemClockMHz`, `maxMemClockMHz` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4308-4312: Function `nvmlDeviceResetMemoryLockedClocks` / 函数 `nvmlDeviceResetMemoryLockedClocks`
```python
def nvmlDeviceResetMemoryLockedClocks(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceResetMemoryLockedClocks")
    ret = fn(handle)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceResetMemoryLockedClocks` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceResetMemoryLockedClocks`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4315-4321: Function `nvmlDeviceGetClkMonStatus` / 函数 `nvmlDeviceGetClkMonStatus`
```python
def nvmlDeviceGetClkMonStatus(handle, c_clkMonInfo=nvmlClkMonStatus_t()):
    isReference = type(c_clkMonInfo) is not nvmlClkMonStatus_t
    c_clkMonInfoRef = c_clkMonInfo if isReference else byref(c_clkMonInfo)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetClkMonStatus")
    ret = fn(handle, c_clkMonInfoRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else c_clkMonInfo
```
**EN:** This function drives `nvmlDeviceGetClkMonStatus` with inputs such as `handle`, `c_clkMonInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetClkMonStatus`，主要处理 `handle`, `c_clkMonInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4325-4329: Function `nvmlDeviceSetApplicationsClocks` / 函数 `nvmlDeviceSetApplicationsClocks`
```python
def nvmlDeviceSetApplicationsClocks(handle, maxMemClockMHz, maxGraphicsClockMHz):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetApplicationsClocks")
    ret = fn(handle, c_uint(maxMemClockMHz), c_uint(maxGraphicsClockMHz))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetApplicationsClocks` with inputs such as `handle`, `maxMemClockMHz`, `maxGraphicsClockMHz`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetApplicationsClocks`，主要处理 `handle`, `maxMemClockMHz`, `maxGraphicsClockMHz` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4333-4337: Function `nvmlDeviceResetApplicationsClocks` / 函数 `nvmlDeviceResetApplicationsClocks`
```python
def nvmlDeviceResetApplicationsClocks(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceResetApplicationsClocks")
    ret = fn(handle)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceResetApplicationsClocks` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceResetApplicationsClocks`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4341-4345: Function `nvmlDeviceSetPowerManagementLimit` / 函数 `nvmlDeviceSetPowerManagementLimit`
```python
def nvmlDeviceSetPowerManagementLimit(handle, limit):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetPowerManagementLimit")
    ret = fn(handle, c_uint(limit))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetPowerManagementLimit` with inputs such as `handle`, `limit`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetPowerManagementLimit`，主要处理 `handle`, `limit` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4349-4353: Function `nvmlDeviceSetGpuOperationMode` / 函数 `nvmlDeviceSetGpuOperationMode`
```python
def nvmlDeviceSetGpuOperationMode(handle, mode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetGpuOperationMode")
    ret = fn(handle, _nvmlGpuOperationMode_t(mode))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetGpuOperationMode` with inputs such as `handle`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetGpuOperationMode`，主要处理 `handle`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4357-4362: Function `nvmlEventSetCreate` / 函数 `nvmlEventSetCreate`
```python
def nvmlEventSetCreate():
    fn = _nvmlGetFunctionPointer("nvmlEventSetCreate")
    eventSet = c_nvmlEventSet_t()
    ret = fn(byref(eventSet))
    _nvmlCheckReturn(ret)
    return eventSet
```
**EN:** This function drives `nvmlEventSetCreate`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlEventSetCreate`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4366-4370: Function `nvmlDeviceRegisterEvents` / 函数 `nvmlDeviceRegisterEvents`
```python
def nvmlDeviceRegisterEvents(handle, eventTypes, eventSet):
    fn = _nvmlGetFunctionPointer("nvmlDeviceRegisterEvents")
    ret = fn(handle, c_ulonglong(eventTypes), eventSet)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceRegisterEvents` with inputs such as `handle`, `eventTypes`, `eventSet`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceRegisterEvents`，主要处理 `handle`, `eventTypes`, `eventSet` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4374-4379: Function `nvmlDeviceGetSupportedEventTypes` / 函数 `nvmlDeviceGetSupportedEventTypes`
```python
def nvmlDeviceGetSupportedEventTypes(handle):
    c_eventTypes = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSupportedEventTypes")
    ret = fn(handle, byref(c_eventTypes))
    _nvmlCheckReturn(ret)
    return c_eventTypes.value
```
**EN:** This function drives `nvmlDeviceGetSupportedEventTypes` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSupportedEventTypes`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4383-4388: Function `nvmlEventSetWait_v2` / 函数 `nvmlEventSetWait_v2`
```python
def nvmlEventSetWait_v2(eventSet, timeoutms):
    fn = _nvmlGetFunctionPointer("nvmlEventSetWait_v2")
    data = c_nvmlEventData_t()
    ret = fn(eventSet, byref(data), c_uint(timeoutms))
    _nvmlCheckReturn(ret)
    return data
```
**EN:** This function drives `nvmlEventSetWait_v2` with inputs such as `eventSet`, `timeoutms`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlEventSetWait_v2`，主要处理 `eventSet`, `timeoutms` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4391-4392: Function `nvmlEventSetWait` / 函数 `nvmlEventSetWait`
```python
def nvmlEventSetWait(eventSet, timeoutms):
    return nvmlEventSetWait_v2(eventSet, timeoutms)
```
**EN:** This function drives `nvmlEventSetWait` with inputs such as `eventSet`, `timeoutms`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlEventSetWait`，主要处理 `eventSet`, `timeoutms` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4396-4400: Function `nvmlEventSetFree` / 函数 `nvmlEventSetFree`
```python
def nvmlEventSetFree(eventSet):
    fn = _nvmlGetFunctionPointer("nvmlEventSetFree")
    ret = fn(eventSet)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlEventSetFree` with inputs such as `eventSet`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlEventSetFree`，主要处理 `eventSet` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4404-4409: Function `nvmlDeviceOnSameBoard` / 函数 `nvmlDeviceOnSameBoard`
```python
def nvmlDeviceOnSameBoard(handle1, handle2):
    fn = _nvmlGetFunctionPointer("nvmlDeviceOnSameBoard")
    onSameBoard = c_int()
    ret = fn(handle1, handle2, byref(onSameBoard))
    _nvmlCheckReturn(ret)
    return onSameBoard.value != 0
```
**EN:** This function drives `nvmlDeviceOnSameBoard` with inputs such as `handle1`, `handle2`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceOnSameBoard`，主要处理 `handle1`, `handle2` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4413-4418: Function `nvmlDeviceGetCurrPcieLinkGeneration` / 函数 `nvmlDeviceGetCurrPcieLinkGeneration`
```python
def nvmlDeviceGetCurrPcieLinkGeneration(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCurrPcieLinkGeneration")
    gen = c_uint()
    ret = fn(handle, byref(gen))
    _nvmlCheckReturn(ret)
    return gen.value
```
**EN:** This function drives `nvmlDeviceGetCurrPcieLinkGeneration` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrPcieLinkGeneration`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4422-4427: Function `nvmlDeviceGetMaxPcieLinkGeneration` / 函数 `nvmlDeviceGetMaxPcieLinkGeneration`
```python
def nvmlDeviceGetMaxPcieLinkGeneration(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMaxPcieLinkGeneration")
    gen = c_uint()
    ret = fn(handle, byref(gen))
    _nvmlCheckReturn(ret)
    return gen.value
```
**EN:** This function drives `nvmlDeviceGetMaxPcieLinkGeneration` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMaxPcieLinkGeneration`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4431-4436: Function `nvmlDeviceGetCurrPcieLinkWidth` / 函数 `nvmlDeviceGetCurrPcieLinkWidth`
```python
def nvmlDeviceGetCurrPcieLinkWidth(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCurrPcieLinkWidth")
    width = c_uint()
    ret = fn(handle, byref(width))
    _nvmlCheckReturn(ret)
    return width.value
```
**EN:** This function drives `nvmlDeviceGetCurrPcieLinkWidth` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrPcieLinkWidth`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4440-4445: Function `nvmlDeviceGetMaxPcieLinkWidth` / 函数 `nvmlDeviceGetMaxPcieLinkWidth`
```python
def nvmlDeviceGetMaxPcieLinkWidth(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMaxPcieLinkWidth")
    width = c_uint()
    ret = fn(handle, byref(width))
    _nvmlCheckReturn(ret)
    return width.value
```
**EN:** This function drives `nvmlDeviceGetMaxPcieLinkWidth` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMaxPcieLinkWidth`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4448-4453: Function `nvmlDeviceGetGpuMaxPcieLinkGeneration` / 函数 `nvmlDeviceGetGpuMaxPcieLinkGeneration`
```python
def nvmlDeviceGetGpuMaxPcieLinkGeneration(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuMaxPcieLinkGeneration")
    gen = c_uint()
    ret = fn(handle, byref(gen))
    _nvmlCheckReturn(ret)
    return gen.value
```
**EN:** This function drives `nvmlDeviceGetGpuMaxPcieLinkGeneration` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuMaxPcieLinkGeneration`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4457-4462: Function `nvmlDeviceGetSupportedClocksThrottleReasons` / 函数 `nvmlDeviceGetSupportedClocksThrottleReasons`
```python
def nvmlDeviceGetSupportedClocksThrottleReasons(handle):
    c_reasons = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSupportedClocksThrottleReasons")
    ret = fn(handle, byref(c_reasons))
    _nvmlCheckReturn(ret)
    return c_reasons.value
```
**EN:** This function drives `nvmlDeviceGetSupportedClocksThrottleReasons` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSupportedClocksThrottleReasons`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4465-4470: Function `nvmlDeviceGetSupportedClocksEventReasons` / 函数 `nvmlDeviceGetSupportedClocksEventReasons`
```python
def nvmlDeviceGetSupportedClocksEventReasons(handle):
    c_reasons = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSupportedClocksEventReasons")
    ret = fn(handle, byref(c_reasons))
    _nvmlCheckReturn(ret)
    return c_reasons.value
```
**EN:** This function drives `nvmlDeviceGetSupportedClocksEventReasons` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSupportedClocksEventReasons`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4474-4479: Function `nvmlDeviceGetCurrentClocksThrottleReasons` / 函数 `nvmlDeviceGetCurrentClocksThrottleReasons`
```python
def nvmlDeviceGetCurrentClocksThrottleReasons(handle):
    c_reasons = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCurrentClocksThrottleReasons")
    ret = fn(handle, byref(c_reasons))
    _nvmlCheckReturn(ret)
    return c_reasons.value
```
**EN:** This function drives `nvmlDeviceGetCurrentClocksThrottleReasons` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrentClocksThrottleReasons`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4482-4487: Function `nvmlDeviceGetCurrentClocksEventReasons` / 函数 `nvmlDeviceGetCurrentClocksEventReasons`
```python
def nvmlDeviceGetCurrentClocksEventReasons(handle):
    c_reasons = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCurrentClocksEventReasons")
    ret = fn(handle, byref(c_reasons))
    _nvmlCheckReturn(ret)
    return c_reasons.value
```
**EN:** This function drives `nvmlDeviceGetCurrentClocksEventReasons` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrentClocksEventReasons`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4491-4496: Function `nvmlDeviceGetIndex` / 函数 `nvmlDeviceGetIndex`
```python
def nvmlDeviceGetIndex(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetIndex")
    c_index = c_uint()
    ret = fn(handle, byref(c_index))
    _nvmlCheckReturn(ret)
    return c_index.value
```
**EN:** This function drives `nvmlDeviceGetIndex` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetIndex`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4500-4505: Function `nvmlDeviceGetAccountingMode` / 函数 `nvmlDeviceGetAccountingMode`
```python
def nvmlDeviceGetAccountingMode(handle):
    c_mode = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAccountingMode")
    ret = fn(handle, byref(c_mode))
    _nvmlCheckReturn(ret)
    return c_mode.value
```
**EN:** This function drives `nvmlDeviceGetAccountingMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAccountingMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4508-4512: Function `nvmlDeviceSetAccountingMode` / 函数 `nvmlDeviceSetAccountingMode`
```python
def nvmlDeviceSetAccountingMode(handle, mode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetAccountingMode")
    ret = fn(handle, _nvmlEnableState_t(mode))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetAccountingMode` with inputs such as `handle`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetAccountingMode`，主要处理 `handle`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4515-4519: Function `nvmlDeviceClearAccountingPids` / 函数 `nvmlDeviceClearAccountingPids`
```python
def nvmlDeviceClearAccountingPids(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceClearAccountingPids")
    ret = fn(handle)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceClearAccountingPids` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceClearAccountingPids`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4522-4530: Function `nvmlDeviceGetAccountingStats` / 函数 `nvmlDeviceGetAccountingStats`
```python
def nvmlDeviceGetAccountingStats(handle, pid):
    stats = c_nvmlAccountingStats_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAccountingStats")
    ret = fn(handle, c_uint(pid), byref(stats))
    _nvmlCheckReturn(ret)
    if stats.maxMemoryUsage == NVML_VALUE_NOT_AVAILABLE_ulonglong.value:
        # special case for WDDM on Windows, see comment above
        stats.maxMemoryUsage = None
    return stats
```
**EN:** This function drives `nvmlDeviceGetAccountingStats` with inputs such as `handle`, `pid`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAccountingStats`，主要处理 `handle`, `pid` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4533-4539: Function `nvmlDeviceGetAccountingPids` / 函数 `nvmlDeviceGetAccountingPids`
```python
def nvmlDeviceGetAccountingPids(handle):
    count = c_uint(nvmlDeviceGetAccountingBufferSize(handle))
    pids = (c_uint * count.value)()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAccountingPids")
    ret = fn(handle, byref(count), pids)
    _nvmlCheckReturn(ret)
    return list(map(int, pids[0 : count.value]))
```
**EN:** This function drives `nvmlDeviceGetAccountingPids` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAccountingPids`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4542-4547: Function `nvmlDeviceGetAccountingBufferSize` / 函数 `nvmlDeviceGetAccountingBufferSize`
```python
def nvmlDeviceGetAccountingBufferSize(handle):
    bufferSize = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAccountingBufferSize")
    ret = fn(handle, byref(bufferSize))
    _nvmlCheckReturn(ret)
    return int(bufferSize.value)
```
**EN:** This function drives `nvmlDeviceGetAccountingBufferSize` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAccountingBufferSize`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4550-4570: Function `nvmlDeviceGetRetiredPages` / 函数 `nvmlDeviceGetRetiredPages`
```python
def nvmlDeviceGetRetiredPages(device, sourceFilter):
    c_source = _nvmlPageRetirementCause_t(sourceFilter)
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetRetiredPages")

    # First call will get the size
    ret = fn(device, c_source, byref(c_count), None)

    # this should only fail with insufficient size
    if (ret != NVML_SUCCESS) and (ret != NVML_ERROR_INSUFFICIENT_SIZE):
        raise NVMLError(ret)

    # call again with a buffer
    # oversize the array for the rare cases where additional pages
# ...
    c_pages = page_array()
    ret = fn(device, c_source, byref(c_count), c_pages)
    _nvmlCheckReturn(ret)
    return list(map(int, c_pages[0 : c_count.value]))
```
**EN:** This function drives `nvmlDeviceGetRetiredPages` with inputs such as `device`, `sourceFilter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetRetiredPages`，主要处理 `device`, `sourceFilter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4573-4598: Function `nvmlDeviceGetRetiredPages_v2` / 函数 `nvmlDeviceGetRetiredPages_v2`
```python
def nvmlDeviceGetRetiredPages_v2(device, sourceFilter):
    c_source = _nvmlPageRetirementCause_t(sourceFilter)
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetRetiredPages_v2")

    # First call will get the size
    ret = fn(device, c_source, byref(c_count), None)

    # this should only fail with insufficient size
    if (ret != NVML_SUCCESS) and (ret != NVML_ERROR_INSUFFICIENT_SIZE):
        raise NVMLError(ret)

    # call again with a buffer
    # oversize the array for the rare cases where additional pages
# ...
    return [
        {"address": int(c_pages[i]), "timestamp": int(c_times[i])}
        for i in range(c_count.value)
    ]
```
**EN:** This function drives `nvmlDeviceGetRetiredPages_v2` with inputs such as `device`, `sourceFilter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetRetiredPages_v2`，主要处理 `device`, `sourceFilter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4601-4606: Function `nvmlDeviceGetRetiredPagesPendingStatus` / 函数 `nvmlDeviceGetRetiredPagesPendingStatus`
```python
def nvmlDeviceGetRetiredPagesPendingStatus(device):
    c_pending = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetRetiredPagesPendingStatus")
    ret = fn(device, byref(c_pending))
    _nvmlCheckReturn(ret)
    return int(c_pending.value)
```
**EN:** This function drives `nvmlDeviceGetRetiredPagesPendingStatus` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetRetiredPagesPendingStatus`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4609-4614: Function `nvmlDeviceGetAPIRestriction` / 函数 `nvmlDeviceGetAPIRestriction`
```python
def nvmlDeviceGetAPIRestriction(device, apiType):
    c_permission = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAPIRestriction")
    ret = fn(device, _nvmlRestrictedAPI_t(apiType), byref(c_permission))
    _nvmlCheckReturn(ret)
    return int(c_permission.value)
```
**EN:** This function drives `nvmlDeviceGetAPIRestriction` with inputs such as `device`, `apiType`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAPIRestriction`，主要处理 `device`, `apiType` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4617-4621: Function `nvmlDeviceSetAPIRestriction` / 函数 `nvmlDeviceSetAPIRestriction`
```python
def nvmlDeviceSetAPIRestriction(handle, apiType, isRestricted):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetAPIRestriction")
    ret = fn(handle, _nvmlRestrictedAPI_t(apiType), _nvmlEnableState_t(isRestricted))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetAPIRestriction` with inputs such as `handle`, `apiType`, `isRestricted`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetAPIRestriction`，主要处理 `handle`, `apiType`, `isRestricted` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4624-4629: Function `nvmlDeviceGetBridgeChipInfo` / 函数 `nvmlDeviceGetBridgeChipInfo`
```python
def nvmlDeviceGetBridgeChipInfo(handle):
    bridgeHierarchy = c_nvmlBridgeChipHierarchy_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetBridgeChipInfo")
    ret = fn(handle, byref(bridgeHierarchy))
    _nvmlCheckReturn(ret)
    return bridgeHierarchy
```
**EN:** This function drives `nvmlDeviceGetBridgeChipInfo` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetBridgeChipInfo`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4632-4664: Function `nvmlDeviceGetSamples` / 函数 `nvmlDeviceGetSamples`
```python
def nvmlDeviceGetSamples(device, sampling_type, timeStamp):
    c_sampling_type = _nvmlSamplingType_t(sampling_type)
    c_time_stamp = c_ulonglong(timeStamp)
    c_sample_count = c_uint(0)
    c_sample_value_type = _nvmlValueType_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSamples")

    ## First Call gets the size
    ret = fn(
        device,
        c_sampling_type,
        c_time_stamp,
        byref(c_sample_value_type),
        byref(c_sample_count),
# ...
        c_samples,
    )
    _nvmlCheckReturn(ret)
    return (c_sample_value_type.value, c_samples[0 : c_sample_count.value])
```
**EN:** This function drives `nvmlDeviceGetSamples` with inputs such as `device`, `sampling_type`, `timeStamp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSamples`，主要处理 `device`, `sampling_type`, `timeStamp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4667-4675: Function `nvmlDeviceGetViolationStatus` / 函数 `nvmlDeviceGetViolationStatus`
```python
def nvmlDeviceGetViolationStatus(device, perfPolicyType):
    c_perfPolicy_type = _nvmlPerfPolicyType_t(perfPolicyType)
    c_violTime = c_nvmlViolationTime_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetViolationStatus")

    ## Invoke the method to get violation time
    ret = fn(device, c_perfPolicy_type, byref(c_violTime))
    _nvmlCheckReturn(ret)
    return c_violTime
```
**EN:** This function drives `nvmlDeviceGetViolationStatus` with inputs such as `device`, `perfPolicyType`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetViolationStatus`，主要处理 `device`, `perfPolicyType` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4678-4683: Function `nvmlDeviceGetPcieThroughput` / 函数 `nvmlDeviceGetPcieThroughput`
```python
def nvmlDeviceGetPcieThroughput(device, counter):
    c_util = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPcieThroughput")
    ret = fn(device, _nvmlPcieUtilCounter_t(counter), byref(c_util))
    _nvmlCheckReturn(ret)
    return c_util.value
```
**EN:** This function drives `nvmlDeviceGetPcieThroughput` with inputs such as `device`, `counter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPcieThroughput`，主要处理 `device`, `counter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4686-4700: Function `nvmlSystemGetTopologyGpuSet` / 函数 `nvmlSystemGetTopologyGpuSet`
```python
def nvmlSystemGetTopologyGpuSet(cpuNumber):
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlSystemGetTopologyGpuSet")

    # First call will get the size
    ret = fn(cpuNumber, byref(c_count), None)

    if ret != NVML_SUCCESS:
        raise NVMLError(ret)
    # call again with a buffer
    device_array = c_nvmlDevice_t * c_count.value
    c_devices = device_array()
    ret = fn(cpuNumber, byref(c_count), c_devices)
    _nvmlCheckReturn(ret)
    return list(c_devices[0 : c_count.value])
```
**EN:** This function drives `nvmlSystemGetTopologyGpuSet` with inputs such as `cpuNumber`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetTopologyGpuSet`，主要处理 `cpuNumber` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4703-4718: Function `nvmlDeviceGetTopologyNearestGpus` / 函数 `nvmlDeviceGetTopologyNearestGpus`
```python
def nvmlDeviceGetTopologyNearestGpus(device, level):
    c_count = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTopologyNearestGpus")

    # First call will get the size
    ret = fn(device, level, byref(c_count), None)

    if ret != NVML_SUCCESS:
        raise NVMLError(ret)

    # call again with a buffer
    device_array = c_nvmlDevice_t * c_count.value
    c_devices = device_array()
    ret = fn(device, level, byref(c_count), c_devices)
    _nvmlCheckReturn(ret)
    return list(c_devices[0 : c_count.value])
```
**EN:** This function drives `nvmlDeviceGetTopologyNearestGpus` with inputs such as `device`, `level`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTopologyNearestGpus`，主要处理 `device`, `level` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4721-4726: Function `nvmlDeviceGetTopologyCommonAncestor` / 函数 `nvmlDeviceGetTopologyCommonAncestor`
```python
def nvmlDeviceGetTopologyCommonAncestor(device1, device2):
    c_level = _nvmlGpuTopologyLevel_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetTopologyCommonAncestor")
    ret = fn(device1, device2, byref(c_level))
    _nvmlCheckReturn(ret)
    return c_level.value
```
**EN:** This function drives `nvmlDeviceGetTopologyCommonAncestor` with inputs such as `device1`, `device2`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetTopologyCommonAncestor`，主要处理 `device1`, `device2` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4729-4735: Function `nvmlDeviceGetNvLinkUtilizationCounter` / 函数 `nvmlDeviceGetNvLinkUtilizationCounter`
```python
def nvmlDeviceGetNvLinkUtilizationCounter(device, link, counter):
    c_rxcounter = c_ulonglong()
    c_txcounter = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkUtilizationCounter")
    ret = fn(device, link, counter, byref(c_rxcounter), byref(c_txcounter))
    _nvmlCheckReturn(ret)
    return (c_rxcounter.value, c_txcounter.value)
```
**EN:** This function drives `nvmlDeviceGetNvLinkUtilizationCounter` with inputs such as `device`, `link`, `counter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkUtilizationCounter`，主要处理 `device`, `link`, `counter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4738-4742: Function `nvmlDeviceFreezeNvLinkUtilizationCounter` / 函数 `nvmlDeviceFreezeNvLinkUtilizationCounter`
```python
def nvmlDeviceFreezeNvLinkUtilizationCounter(device, link, counter, freeze):
    fn = _nvmlGetFunctionPointer("nvmlDeviceFreezeNvLinkUtilizationCounter")
    ret = fn(device, link, counter, freeze)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceFreezeNvLinkUtilizationCounter` with inputs such as `device`, `link`, `counter`, `freeze`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceFreezeNvLinkUtilizationCounter`，主要处理 `device`, `link`, `counter`, `freeze` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4745-4749: Function `nvmlDeviceResetNvLinkUtilizationCounter` / 函数 `nvmlDeviceResetNvLinkUtilizationCounter`
```python
def nvmlDeviceResetNvLinkUtilizationCounter(device, link, counter):
    fn = _nvmlGetFunctionPointer("nvmlDeviceResetNvLinkUtilizationCounter")
    ret = fn(device, link, counter)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceResetNvLinkUtilizationCounter` with inputs such as `device`, `link`, `counter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceResetNvLinkUtilizationCounter`，主要处理 `device`, `link`, `counter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4752-4756: Function `nvmlDeviceSetNvLinkUtilizationControl` / 函数 `nvmlDeviceSetNvLinkUtilizationControl`
```python
def nvmlDeviceSetNvLinkUtilizationControl(device, link, counter, control, reset):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetNvLinkUtilizationControl")
    ret = fn(device, link, counter, byref(control), reset)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetNvLinkUtilizationControl` with inputs such as `device`, `link`, `counter`, `control`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetNvLinkUtilizationControl`，主要处理 `device`, `link`, `counter`, `control` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4759-4764: Function `nvmlDeviceGetNvLinkUtilizationControl` / 函数 `nvmlDeviceGetNvLinkUtilizationControl`
```python
def nvmlDeviceGetNvLinkUtilizationControl(device, link, counter):
    c_control = nvmlNvLinkUtilizationControl_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkUtilizationControl")
    ret = fn(device, link, counter, byref(c_control))
    _nvmlCheckReturn(ret)
    return c_control
```
**EN:** This function drives `nvmlDeviceGetNvLinkUtilizationControl` with inputs such as `device`, `link`, `counter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkUtilizationControl`，主要处理 `device`, `link`, `counter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4767-4772: Function `nvmlDeviceGetNvLinkCapability` / 函数 `nvmlDeviceGetNvLinkCapability`
```python
def nvmlDeviceGetNvLinkCapability(device, link, capability):
    c_capResult = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkCapability")
    ret = fn(device, link, capability, byref(c_capResult))
    _nvmlCheckReturn(ret)
    return c_capResult.value
```
**EN:** This function drives `nvmlDeviceGetNvLinkCapability` with inputs such as `device`, `link`, `capability`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkCapability`，主要处理 `device`, `link`, `capability` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4775-4780: Function `nvmlDeviceGetNvLinkErrorCounter` / 函数 `nvmlDeviceGetNvLinkErrorCounter`
```python
def nvmlDeviceGetNvLinkErrorCounter(device, link, counter):
    c_result = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkErrorCounter")
    ret = fn(device, link, counter, byref(c_result))
    _nvmlCheckReturn(ret)
    return c_result.value
```
**EN:** This function drives `nvmlDeviceGetNvLinkErrorCounter` with inputs such as `device`, `link`, `counter`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkErrorCounter`，主要处理 `device`, `link`, `counter` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4783-4787: Function `nvmlDeviceResetNvLinkErrorCounters` / 函数 `nvmlDeviceResetNvLinkErrorCounters`
```python
def nvmlDeviceResetNvLinkErrorCounters(device, link):
    fn = _nvmlGetFunctionPointer("nvmlDeviceResetNvLinkErrorCounters")
    ret = fn(device, link)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceResetNvLinkErrorCounters` with inputs such as `device`, `link`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceResetNvLinkErrorCounters`，主要处理 `device`, `link` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4790-4795: Function `nvmlDeviceGetNvLinkRemotePciInfo` / 函数 `nvmlDeviceGetNvLinkRemotePciInfo`
```python
def nvmlDeviceGetNvLinkRemotePciInfo(device, link):
    c_pci = nvmlPciInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkRemotePciInfo_v2")
    ret = fn(device, link, byref(c_pci))
    _nvmlCheckReturn(ret)
    return c_pci
```
**EN:** This function drives `nvmlDeviceGetNvLinkRemotePciInfo` with inputs such as `device`, `link`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkRemotePciInfo`，主要处理 `device`, `link` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4798-4803: Function `nvmlDeviceGetNvLinkRemoteDeviceType` / 函数 `nvmlDeviceGetNvLinkRemoteDeviceType`
```python
def nvmlDeviceGetNvLinkRemoteDeviceType(handle, link):
    c_type = _nvmlNvLinkDeviceType_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkRemoteDeviceType")
    ret = fn(handle, link, byref(c_type))
    _nvmlCheckReturn(ret)
    return c_type.value
```
**EN:** This function drives `nvmlDeviceGetNvLinkRemoteDeviceType` with inputs such as `handle`, `link`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkRemoteDeviceType`，主要处理 `handle`, `link` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4806-4811: Function `nvmlDeviceGetNvLinkState` / 函数 `nvmlDeviceGetNvLinkState`
```python
def nvmlDeviceGetNvLinkState(device, link):
    c_isActive = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkState")
    ret = fn(device, link, byref(c_isActive))
    _nvmlCheckReturn(ret)
    return c_isActive.value
```
**EN:** This function drives `nvmlDeviceGetNvLinkState` with inputs such as `device`, `link`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkState`，主要处理 `device`, `link` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4814-4819: Function `nvmlDeviceGetNvLinkVersion` / 函数 `nvmlDeviceGetNvLinkVersion`
```python
def nvmlDeviceGetNvLinkVersion(device, link):
    c_version = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvLinkVersion")
    ret = fn(device, link, byref(c_version))
    _nvmlCheckReturn(ret)
    return c_version.value
```
**EN:** This function drives `nvmlDeviceGetNvLinkVersion` with inputs such as `device`, `link`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvLinkVersion`，主要处理 `device`, `link` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4822-4826: Function `nvmlDeviceModifyDrainState` / 函数 `nvmlDeviceModifyDrainState`
```python
def nvmlDeviceModifyDrainState(pciInfo, newState):
    fn = _nvmlGetFunctionPointer("nvmlDeviceModifyDrainState")
    ret = fn(pointer(pciInfo), newState)
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceModifyDrainState` with inputs such as `pciInfo`, `newState`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceModifyDrainState`，主要处理 `pciInfo`, `newState` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4829-4834: Function `nvmlDeviceQueryDrainState` / 函数 `nvmlDeviceQueryDrainState`
```python
def nvmlDeviceQueryDrainState(pciInfo):
    c_newState = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceQueryDrainState")
    ret = fn(pointer(pciInfo), byref(c_newState))
    _nvmlCheckReturn(ret)
    return c_newState.value
```
**EN:** This function drives `nvmlDeviceQueryDrainState` with inputs such as `pciInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceQueryDrainState`，主要处理 `pciInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4837-4841: Function `nvmlDeviceRemoveGpu` / 函数 `nvmlDeviceRemoveGpu`
```python
def nvmlDeviceRemoveGpu(pciInfo):
    fn = _nvmlGetFunctionPointer("nvmlDeviceRemoveGpu")
    ret = fn(pointer(pciInfo))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceRemoveGpu` with inputs such as `pciInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceRemoveGpu`，主要处理 `pciInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4844-4848: Function `nvmlDeviceDiscoverGpus` / 函数 `nvmlDeviceDiscoverGpus`
```python
def nvmlDeviceDiscoverGpus(pciInfo):
    fn = _nvmlGetFunctionPointer("nvmlDeviceDiscoverGpus")
    ret = fn(pointer(pciInfo))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceDiscoverGpus` with inputs such as `pciInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceDiscoverGpus`，主要处理 `pciInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4851-4864: Function `nvmlDeviceGetFieldValues` / 函数 `nvmlDeviceGetFieldValues`
```python
def nvmlDeviceGetFieldValues(handle, fieldIds):
    values_arr = c_nvmlFieldValue_t * len(fieldIds)
    values = values_arr()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetFieldValues")

    for i, fieldId in enumerate(fieldIds):
        try:
            values[i].fieldId, values[i].scopeId = fieldId
        except TypeError:
            values[i].fieldId = fieldId

    ret = fn(handle, c_int32(len(fieldIds)), byref(values))
    _nvmlCheckReturn(ret)
    return values
```
**EN:** This function drives `nvmlDeviceGetFieldValues` with inputs such as `handle`, `fieldIds`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetFieldValues`，主要处理 `handle`, `fieldIds` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4867-4880: Function `nvmlDeviceClearFieldValues` / 函数 `nvmlDeviceClearFieldValues`
```python
def nvmlDeviceClearFieldValues(handle, fieldIds):
    values_arr = c_nvmlFieldValue_t * len(fieldIds)
    values = values_arr()
    fn = _nvmlGetFunctionPointer("nvmlDeviceClearFieldValues")

    for i, fieldId in enumerate(fieldIds):
        try:
            values[i].fieldId, values[i].scopeId = fieldId
        except TypeError:
            values[i].fieldId = fieldId

    ret = fn(handle, c_int32(len(fieldIds)), byref(values))
    _nvmlCheckReturn(ret)
    return values
```
**EN:** This function drives `nvmlDeviceClearFieldValues` with inputs such as `handle`, `fieldIds`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceClearFieldValues`，主要处理 `handle`, `fieldIds` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4883-4888: Function `nvmlDeviceGetVirtualizationMode` / 函数 `nvmlDeviceGetVirtualizationMode`
```python
def nvmlDeviceGetVirtualizationMode(handle):
    c_virtualization_mode = c_ulonglong()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVirtualizationMode")
    ret = fn(handle, byref(c_virtualization_mode))
    _nvmlCheckReturn(ret)
    return c_virtualization_mode.value
```
**EN:** This function drives `nvmlDeviceGetVirtualizationMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVirtualizationMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4891-4893: Function `nvmlDeviceSetVirtualizationMode` / 函数 `nvmlDeviceSetVirtualizationMode`
```python
def nvmlDeviceSetVirtualizationMode(handle, virtualization_mode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetVirtualizationMode")
    return fn(handle, virtualization_mode)
```
**EN:** This function drives `nvmlDeviceSetVirtualizationMode` with inputs such as `handle`, `virtualization_mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetVirtualizationMode`，主要处理 `handle`, `virtualization_mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4896-4902: Function `nvmlDeviceGetVgpuHeterogeneousMode` / 函数 `nvmlDeviceGetVgpuHeterogeneousMode`
```python
def nvmlDeviceGetVgpuHeterogeneousMode(handle):
    c_vgpuHeterogeneousMode = c_nvmlVgpuHeterogeneousMode_v1_t(0)
    c_vgpuHeterogeneousMode.version = VgpuHeterogeneousMode_v1
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuHeterogeneousMode")
    ret = fn(handle, byref(c_vgpuHeterogeneousMode))
    _nvmlCheckReturn(ret)
    return c_vgpuHeterogeneousMode.mode
```
**EN:** This function drives `nvmlDeviceGetVgpuHeterogeneousMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuHeterogeneousMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4905-4912: Function `nvmlDeviceSetVgpuHeterogeneousMode` / 函数 `nvmlDeviceSetVgpuHeterogeneousMode`
```python
def nvmlDeviceSetVgpuHeterogeneousMode(handle, heterogeneous_mode):
    c_vgpuHeterogeneousMode = c_nvmlVgpuHeterogeneousMode_v1_t(0)
    c_vgpuHeterogeneousMode.version = VgpuHeterogeneousMode_v1
    c_vgpuHeterogeneousMode.mode = heterogeneous_mode
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetVgpuHeterogeneousMode")
    ret = fn(handle, byref(c_vgpuHeterogeneousMode))
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetVgpuHeterogeneousMode` with inputs such as `handle`, `heterogeneous_mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetVgpuHeterogeneousMode`，主要处理 `handle`, `heterogeneous_mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4915-4921: Function `nvmlVgpuInstanceGetPlacementId` / 函数 `nvmlVgpuInstanceGetPlacementId`
```python
def nvmlVgpuInstanceGetPlacementId(vgpuInstance):
    c_placement = c_nvmlVgpuPlacementId_v1_t(0)
    c_placement.version = VgpuPlacementId_v1
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetPlacementId")
    ret = fn(vgpuInstance, byref(c_placement))
    _nvmlCheckReturn(ret)
    return c_placement.placementId
```
**EN:** This function drives `nvmlVgpuInstanceGetPlacementId` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetPlacementId`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4924-4946: Function `nvmlDeviceGetVgpuTypeSupportedPlacements` / 函数 `nvmlDeviceGetVgpuTypeSupportedPlacements`
```python
def nvmlDeviceGetVgpuTypeSupportedPlacements(handle, vgpuTypeId, mode=0, version=1):
    c_max_instances = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetMaxInstances")
    ret = fn(handle, vgpuTypeId, byref(c_max_instances))
    _nvmlCheckReturn(ret)

    if version == 2:
        c_vgpu_placements = c_nvmlVgpuPlacementList_v2_t()
        c_vgpu_placements.version = VgpuPlacementList_v2
        c_vgpu_placements.count = c_max_instances.value
        c_vgpu_placements.mode = mode
    elif version == 1:
        c_vgpu_placements = c_nvmlVgpuPlacementList_v1_t()
        c_vgpu_placements.version = VgpuPlacementList_v1
# ...
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuTypeSupportedPlacements")
    ret = fn(handle, vgpuTypeId, byref(c_vgpu_placements))
    _nvmlCheckReturn(ret)
    return c_vgpu_placements
```
**EN:** This function drives `nvmlDeviceGetVgpuTypeSupportedPlacements` with inputs such as `handle`, `vgpuTypeId`, `mode`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuTypeSupportedPlacements`，主要处理 `handle`, `vgpuTypeId`, `mode`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4949-4968: Function `nvmlDeviceGetVgpuTypeCreatablePlacements` / 函数 `nvmlDeviceGetVgpuTypeCreatablePlacements`
```python
def nvmlDeviceGetVgpuTypeCreatablePlacements(handle, vgpuTypeId, version=1):
    c_max_instances = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetMaxInstances")
    ret = fn(handle, vgpuTypeId, byref(c_max_instances))
    _nvmlCheckReturn(ret)

    if version == 2:
        c_vgpu_placements = c_nvmlVgpuPlacementList_v2_t()
        c_vgpu_placements.version = VgpuPlacementList_v2
        c_vgpu_placements.count = c_max_instances.value
    elif version == 1:
        c_vgpu_placements = c_nvmlVgpuPlacementList_v1_t()
        c_vgpu_placements.version = VgpuPlacementList_v1

# ...
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuTypeCreatablePlacements")
    ret = fn(handle, vgpuTypeId, byref(c_vgpu_placements))
    _nvmlCheckReturn(ret)
    return c_vgpu_placements
```
**EN:** This function drives `nvmlDeviceGetVgpuTypeCreatablePlacements` with inputs such as `handle`, `vgpuTypeId`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuTypeCreatablePlacements`，主要处理 `handle`, `vgpuTypeId`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4971-4976: Function `nvmlGetVgpuDriverCapabilities` / 函数 `nvmlGetVgpuDriverCapabilities`
```python
def nvmlGetVgpuDriverCapabilities(capability):
    c_capResult = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlGetVgpuDriverCapabilities")
    ret = fn(_nvmlVgpuDriverCapability_t(capability), byref(c_capResult))
    _nvmlCheckReturn(ret)
    return c_capResult.value
```
**EN:** This function drives `nvmlGetVgpuDriverCapabilities` with inputs such as `capability`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGetVgpuDriverCapabilities`，主要处理 `capability` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4979-4984: Function `nvmlDeviceGetVgpuCapabilities` / 函数 `nvmlDeviceGetVgpuCapabilities`
```python
def nvmlDeviceGetVgpuCapabilities(handle, capability):
    c_capResult = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuCapabilities")
    ret = fn(handle, _nvmlDeviceVgpuCapability_t(capability), byref(c_capResult))
    _nvmlCheckReturn(ret)
    return c_capResult.value
```
**EN:** This function drives `nvmlDeviceGetVgpuCapabilities` with inputs such as `handle`, `capability`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuCapabilities`，主要处理 `handle`, `capability` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4987-4991: Function `nvmlDeviceSetVgpuCapabilities` / 函数 `nvmlDeviceSetVgpuCapabilities`
```python
def nvmlDeviceSetVgpuCapabilities(handle, capability, state):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetVgpuCapabilities")
    ret = fn(handle, _nvmlDeviceVgpuCapability_t(capability), state)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetVgpuCapabilities` with inputs such as `handle`, `capability`, `state`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetVgpuCapabilities`，主要处理 `handle`, `capability`, `state` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 4994-5018: Function `nvmlDeviceGetSupportedVgpus` / 函数 `nvmlDeviceGetSupportedVgpus`
```python
def nvmlDeviceGetSupportedVgpus(handle):
    # first call to get the size
    c_vgpu_count = c_uint(0)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSupportedVgpus")
    ret = fn(handle, byref(c_vgpu_count), None)

    if ret == NVML_SUCCESS:
        # special case, no supported vGPUs
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        vgpu_type_ids_array = _nvmlVgpuTypeId_t * c_vgpu_count.value
        c_vgpu_type_ids = vgpu_type_ids_array()
# ...
        return vgpus
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetSupportedVgpus` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSupportedVgpus`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5021-5045: Function `nvmlDeviceGetCreatableVgpus` / 函数 `nvmlDeviceGetCreatableVgpus`
```python
def nvmlDeviceGetCreatableVgpus(handle):
    # first call to get the size
    c_vgpu_count = c_uint(0)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCreatableVgpus")
    ret = fn(handle, byref(c_vgpu_count), None)

    if ret == NVML_SUCCESS:
        # special case, no supported vGPUs
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        vgpu_type_ids_array = _nvmlVgpuTypeId_t * c_vgpu_count.value
        c_vgpu_type_ids = vgpu_type_ids_array()
# ...
        return vgpus
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetCreatableVgpus` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCreatableVgpus`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5048-5053: Function `nvmlVgpuTypeGetGpuInstanceProfileId` / 函数 `nvmlVgpuTypeGetGpuInstanceProfileId`
```python
def nvmlVgpuTypeGetGpuInstanceProfileId(vgpuTypeId):
    c_profile_id = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetGpuInstanceProfileId")
    ret = fn(vgpuTypeId, byref(c_profile_id))
    _nvmlCheckReturn(ret)
    return c_profile_id.value
```
**EN:** This function drives `nvmlVgpuTypeGetGpuInstanceProfileId` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetGpuInstanceProfileId`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5056-5063: Function `nvmlVgpuTypeGetClass` / 函数 `nvmlVgpuTypeGetClass`
```python
@convertStrBytes
def nvmlVgpuTypeGetClass(vgpuTypeId):
    c_class = create_string_buffer(NVML_DEVICE_NAME_BUFFER_SIZE)
    c_buffer_size = c_uint(NVML_DEVICE_NAME_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetClass")
    ret = fn(vgpuTypeId, c_class, byref(c_buffer_size))
    _nvmlCheckReturn(ret)
    return c_class.value
```
**EN:** This function drives `nvmlVgpuTypeGetClass` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetClass`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5066-5073: Function `nvmlVgpuTypeGetName` / 函数 `nvmlVgpuTypeGetName`
```python
@convertStrBytes
def nvmlVgpuTypeGetName(vgpuTypeId):
    c_name = create_string_buffer(NVML_DEVICE_NAME_BUFFER_SIZE)
    c_buffer_size = c_uint(NVML_DEVICE_NAME_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetName")
    ret = fn(vgpuTypeId, c_name, byref(c_buffer_size))
    _nvmlCheckReturn(ret)
    return c_name.value
```
**EN:** This function drives `nvmlVgpuTypeGetName` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetName`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5076-5082: Function `nvmlVgpuTypeGetDeviceID` / 函数 `nvmlVgpuTypeGetDeviceID`
```python
def nvmlVgpuTypeGetDeviceID(vgpuTypeId):
    c_device_id = c_ulonglong(0)
    c_subsystem_id = c_ulonglong(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetDeviceID")
    ret = fn(vgpuTypeId, byref(c_device_id), byref(c_subsystem_id))
    _nvmlCheckReturn(ret)
    return (c_device_id.value, c_subsystem_id.value)
```
**EN:** This function drives `nvmlVgpuTypeGetDeviceID` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetDeviceID`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5085-5090: Function `nvmlVgpuTypeGetFramebufferSize` / 函数 `nvmlVgpuTypeGetFramebufferSize`
```python
def nvmlVgpuTypeGetFramebufferSize(vgpuTypeId):
    c_fb_size = c_ulonglong(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetFramebufferSize")
    ret = fn(vgpuTypeId, byref(c_fb_size))
    _nvmlCheckReturn(ret)
    return c_fb_size.value
```
**EN:** This function drives `nvmlVgpuTypeGetFramebufferSize` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetFramebufferSize`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5093-5098: Function `nvmlVgpuTypeGetNumDisplayHeads` / 函数 `nvmlVgpuTypeGetNumDisplayHeads`
```python
def nvmlVgpuTypeGetNumDisplayHeads(vgpuTypeId):
    c_num_heads = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetNumDisplayHeads")
    ret = fn(vgpuTypeId, byref(c_num_heads))
    _nvmlCheckReturn(ret)
    return c_num_heads.value
```
**EN:** This function drives `nvmlVgpuTypeGetNumDisplayHeads` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetNumDisplayHeads`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5101-5107: Function `nvmlVgpuTypeGetResolution` / 函数 `nvmlVgpuTypeGetResolution`
```python
def nvmlVgpuTypeGetResolution(vgpuTypeId):
    c_xdim = c_uint(0)
    c_ydim = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetResolution")
    ret = fn(vgpuTypeId, 0, byref(c_xdim), byref(c_ydim))
    _nvmlCheckReturn(ret)
    return (c_xdim.value, c_ydim.value)
```
**EN:** This function drives `nvmlVgpuTypeGetResolution` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetResolution`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5110-5117: Function `nvmlVgpuTypeGetLicense` / 函数 `nvmlVgpuTypeGetLicense`
```python
@convertStrBytes
def nvmlVgpuTypeGetLicense(vgpuTypeId):
    c_license = create_string_buffer(NVML_GRID_LICENSE_BUFFER_SIZE)
    c_buffer_size = c_uint(NVML_GRID_LICENSE_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetLicense")
    ret = fn(vgpuTypeId, c_license, c_buffer_size)
    _nvmlCheckReturn(ret)
    return c_license.value
```
**EN:** This function drives `nvmlVgpuTypeGetLicense` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetLicense`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5120-5125: Function `nvmlVgpuTypeGetFrameRateLimit` / 函数 `nvmlVgpuTypeGetFrameRateLimit`
```python
def nvmlVgpuTypeGetFrameRateLimit(vgpuTypeId):
    c_frl_config = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetFrameRateLimit")
    ret = fn(vgpuTypeId, byref(c_frl_config))
    _nvmlCheckReturn(ret)
    return c_frl_config.value
```
**EN:** This function drives `nvmlVgpuTypeGetFrameRateLimit` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetFrameRateLimit`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5128-5133: Function `nvmlVgpuTypeGetGspHeapSize` / 函数 `nvmlVgpuTypeGetGspHeapSize`
```python
def nvmlVgpuTypeGetGspHeapSize(vgpuTypeId):
    c_gsp_heap = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetGspHeapSize")
    ret = fn(vgpuTypeId, byref(c_gsp_heap))
    _nvmlCheckReturn(ret)
    return c_gsp_heap.value
```
**EN:** This function drives `nvmlVgpuTypeGetGspHeapSize` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetGspHeapSize`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5136-5141: Function `nvmlVgpuTypeGetFbReservation` / 函数 `nvmlVgpuTypeGetFbReservation`
```python
def nvmlVgpuTypeGetFbReservation(vgpuTypeId):
    c_fb_reservation = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetFbReservation")
    ret = fn(vgpuTypeId, byref(c_fb_reservation))
    _nvmlCheckReturn(ret)
    return c_fb_reservation.value
```
**EN:** This function drives `nvmlVgpuTypeGetFbReservation` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetFbReservation`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5144-5150: Function `nvmlVgpuInstanceGetRuntimeStateSize` / 函数 `nvmlVgpuInstanceGetRuntimeStateSize`
```python
def nvmlVgpuInstanceGetRuntimeStateSize(vgpuInstance):
    c_runtime_state = nvmlVgpuRuntimeState_v1_t()
    c_runtime_state.version = VgpuRuntimeState_v1
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetRuntimeStateSize")
    ret = fn(vgpuInstance, byref(c_runtime_state))
    _nvmlCheckReturn(ret)
    return c_runtime_state
```
**EN:** This function drives `nvmlVgpuInstanceGetRuntimeStateSize` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetRuntimeStateSize`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5153-5158: Function `nvmlVgpuTypeGetMaxInstances` / 函数 `nvmlVgpuTypeGetMaxInstances`
```python
def nvmlVgpuTypeGetMaxInstances(handle, vgpuTypeId):
    c_max_instances = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetMaxInstances")
    ret = fn(handle, vgpuTypeId, byref(c_max_instances))
    _nvmlCheckReturn(ret)
    return c_max_instances.value
```
**EN:** This function drives `nvmlVgpuTypeGetMaxInstances` with inputs such as `handle`, `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetMaxInstances`，主要处理 `handle`, `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5161-5166: Function `nvmlVgpuTypeGetMaxInstancesPerVm` / 函数 `nvmlVgpuTypeGetMaxInstancesPerVm`
```python
def nvmlVgpuTypeGetMaxInstancesPerVm(vgpuTypeId):
    c_max_instances_per_vm = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetMaxInstancesPerVm")
    ret = fn(vgpuTypeId, byref(c_max_instances_per_vm))
    _nvmlCheckReturn(ret)
    return c_max_instances_per_vm.value
```
**EN:** This function drives `nvmlVgpuTypeGetMaxInstancesPerVm` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetMaxInstancesPerVm`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5169-5175: Function `nvmlVgpuTypeGetBAR1Info` / 函数 `nvmlVgpuTypeGetBAR1Info`
```python
def nvmlVgpuTypeGetBAR1Info(vgpuTypeId):
    c_bar1Info = c_nvmlVgpuTypeBar1Info_v1_t(0)
    c_bar1Info.version = VgpuTypeBar1Info_v1
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetBAR1Info")
    ret = fn(vgpuTypeId, byref(c_bar1Info))
    _nvmlCheckReturn(ret)
    return c_bar1Info
```
**EN:** This function drives `nvmlVgpuTypeGetBAR1Info` with inputs such as `vgpuTypeId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetBAR1Info`，主要处理 `vgpuTypeId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5178-5202: Function `nvmlDeviceGetActiveVgpus` / 函数 `nvmlDeviceGetActiveVgpus`
```python
def nvmlDeviceGetActiveVgpus(handle):
    # first call to get the size
    c_vgpu_count = c_uint(0)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetActiveVgpus")
    ret = fn(handle, byref(c_vgpu_count), None)

    if ret == NVML_SUCCESS:
        # special case, no active vGPUs
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        vgpu_instance_array = _nvmlVgpuInstance_t * c_vgpu_count.value
        c_vgpu_instances = vgpu_instance_array()
# ...
        return vgpus
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetActiveVgpus` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetActiveVgpus`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5205-5213: Function `nvmlVgpuInstanceGetVmID` / 函数 `nvmlVgpuInstanceGetVmID`
```python
@convertStrBytes
def nvmlVgpuInstanceGetVmID(vgpuInstance):
    c_vm_id = create_string_buffer(NVML_DEVICE_UUID_BUFFER_SIZE)
    c_buffer_size = c_uint(NVML_GRID_LICENSE_BUFFER_SIZE)
    c_vm_id_type = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetVmID")
    ret = fn(vgpuInstance, byref(c_vm_id), c_buffer_size, byref(c_vm_id_type))
    _nvmlCheckReturn(ret)
    return (c_vm_id.value, c_vm_id_type.value)
```
**EN:** This function drives `nvmlVgpuInstanceGetVmID` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetVmID`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5216-5223: Function `nvmlVgpuInstanceGetUUID` / 函数 `nvmlVgpuInstanceGetUUID`
```python
@convertStrBytes
def nvmlVgpuInstanceGetUUID(vgpuInstance):
    c_uuid = create_string_buffer(NVML_DEVICE_UUID_BUFFER_SIZE)
    c_buffer_size = c_uint(NVML_DEVICE_UUID_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetUUID")
    ret = fn(vgpuInstance, byref(c_uuid), c_buffer_size)
    _nvmlCheckReturn(ret)
    return c_uuid.value
```
**EN:** This function drives `nvmlVgpuInstanceGetUUID` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetUUID`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5226-5233: Function `nvmlVgpuInstanceGetMdevUUID` / 函数 `nvmlVgpuInstanceGetMdevUUID`
```python
@convertStrBytes
def nvmlVgpuInstanceGetMdevUUID(vgpuInstance):
    c_uuid = create_string_buffer(NVML_DEVICE_UUID_BUFFER_SIZE)
    c_buffer_size = c_uint(NVML_DEVICE_UUID_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetMdevUUID")
    ret = fn(vgpuInstance, byref(c_uuid), c_buffer_size)
    _nvmlCheckReturn(ret)
    return c_uuid.value
```
**EN:** This function drives `nvmlVgpuInstanceGetMdevUUID` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetMdevUUID`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5236-5243: Function `nvmlVgpuInstanceGetVmDriverVersion` / 函数 `nvmlVgpuInstanceGetVmDriverVersion`
```python
@convertStrBytes
def nvmlVgpuInstanceGetVmDriverVersion(vgpuInstance):
    c_driver_version = create_string_buffer(NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE)
    c_buffer_size = c_uint(NVML_SYSTEM_DRIVER_VERSION_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetVmDriverVersion")
    ret = fn(vgpuInstance, byref(c_driver_version), c_buffer_size)
    _nvmlCheckReturn(ret)
    return c_driver_version.value
```
**EN:** This function drives `nvmlVgpuInstanceGetVmDriverVersion` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetVmDriverVersion`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5246-5251: Function `nvmlVgpuInstanceGetLicenseStatus` / 函数 `nvmlVgpuInstanceGetLicenseStatus`
```python
def nvmlVgpuInstanceGetLicenseStatus(vgpuInstance):
    c_license_status = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetLicenseStatus")
    ret = fn(vgpuInstance, byref(c_license_status))
    _nvmlCheckReturn(ret)
    return c_license_status.value
```
**EN:** This function drives `nvmlVgpuInstanceGetLicenseStatus` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetLicenseStatus`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5254-5259: Function `nvmlVgpuInstanceGetLicenseInfo_v2` / 函数 `nvmlVgpuInstanceGetLicenseInfo_v2`
```python
def nvmlVgpuInstanceGetLicenseInfo_v2(vgpuInstance):
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetLicenseInfo_v2")
    c_license_info = c_nvmlVgpuLicenseInfo_t()
    ret = fn(vgpuInstance, byref(c_license_info))
    _nvmlCheckReturn(ret)
    return c_license_info
```
**EN:** This function drives `nvmlVgpuInstanceGetLicenseInfo_v2` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetLicenseInfo_v2`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5262-5263: Function `nvmlVgpuInstanceGetLicenseInfo` / 函数 `nvmlVgpuInstanceGetLicenseInfo`
```python
def nvmlVgpuInstanceGetLicenseInfo(vgpuInstance):
    return nvmlVgpuInstanceGetLicenseInfo_v2(vgpuInstance)
```
**EN:** This function drives `nvmlVgpuInstanceGetLicenseInfo` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetLicenseInfo`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5266-5271: Function `nvmlVgpuInstanceGetFrameRateLimit` / 函数 `nvmlVgpuInstanceGetFrameRateLimit`
```python
def nvmlVgpuInstanceGetFrameRateLimit(vgpuInstance):
    c_frl = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetFrameRateLimit")
    ret = fn(vgpuInstance, byref(c_frl))
    _nvmlCheckReturn(ret)
    return c_frl.value
```
**EN:** This function drives `nvmlVgpuInstanceGetFrameRateLimit` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetFrameRateLimit`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5274-5279: Function `nvmlVgpuInstanceGetEccMode` / 函数 `nvmlVgpuInstanceGetEccMode`
```python
def nvmlVgpuInstanceGetEccMode(vgpuInstance):
    c_mode = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetEccMode")
    ret = fn(vgpuInstance, byref(c_mode))
    _nvmlCheckReturn(ret)
    return c_mode.value
```
**EN:** This function drives `nvmlVgpuInstanceGetEccMode` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetEccMode`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5282-5287: Function `nvmlVgpuInstanceGetType` / 函数 `nvmlVgpuInstanceGetType`
```python
def nvmlVgpuInstanceGetType(vgpuInstance):
    c_vgpu_type = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetType")
    ret = fn(vgpuInstance, byref(c_vgpu_type))
    _nvmlCheckReturn(ret)
    return c_vgpu_type.value
```
**EN:** This function drives `nvmlVgpuInstanceGetType` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetType`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5290-5295: Function `nvmlVgpuInstanceGetEncoderCapacity` / 函数 `nvmlVgpuInstanceGetEncoderCapacity`
```python
def nvmlVgpuInstanceGetEncoderCapacity(vgpuInstance):
    c_encoder_capacity = c_ulonglong(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetEncoderCapacity")
    ret = fn(vgpuInstance, byref(c_encoder_capacity))
    _nvmlCheckReturn(ret)
    return c_encoder_capacity.value
```
**EN:** This function drives `nvmlVgpuInstanceGetEncoderCapacity` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetEncoderCapacity`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5298-5300: Function `nvmlVgpuInstanceSetEncoderCapacity` / 函数 `nvmlVgpuInstanceSetEncoderCapacity`
```python
def nvmlVgpuInstanceSetEncoderCapacity(vgpuInstance, encoder_capacity):
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceSetEncoderCapacity")
    return fn(vgpuInstance, encoder_capacity)
```
**EN:** This function drives `nvmlVgpuInstanceSetEncoderCapacity` with inputs such as `vgpuInstance`, `encoder_capacity`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceSetEncoderCapacity`，主要处理 `vgpuInstance`, `encoder_capacity` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5303-5308: Function `nvmlVgpuInstanceGetFbUsage` / 函数 `nvmlVgpuInstanceGetFbUsage`
```python
def nvmlVgpuInstanceGetFbUsage(vgpuInstance):
    c_fb_usage = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetFbUsage")
    ret = fn(vgpuInstance, byref(c_fb_usage))
    _nvmlCheckReturn(ret)
    return c_fb_usage.value
```
**EN:** This function drives `nvmlVgpuInstanceGetFbUsage` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetFbUsage`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5311-5316: Function `nvmlVgpuTypeGetCapabilities` / 函数 `nvmlVgpuTypeGetCapabilities`
```python
def nvmlVgpuTypeGetCapabilities(vgpuTypeId, capability):
    c_cap_result = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuTypeGetCapabilities")
    ret = fn(vgpuTypeId, _nvmlVgpuCapability_t(capability), byref(c_cap_result))
    _nvmlCheckReturn(ret)
    return c_cap_result.value
```
**EN:** This function drives `nvmlVgpuTypeGetCapabilities` with inputs such as `vgpuTypeId`, `capability`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuTypeGetCapabilities`，主要处理 `vgpuTypeId`, `capability` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5319-5324: Function `nvmlVgpuInstanceGetGpuInstanceId` / 函数 `nvmlVgpuInstanceGetGpuInstanceId`
```python
def nvmlVgpuInstanceGetGpuInstanceId(vgpuInstance):
    c_id = c_uint(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetGpuInstanceId")
    ret = fn(vgpuInstance, byref(c_id))
    _nvmlCheckReturn(ret)
    return c_id.value
```
**EN:** This function drives `nvmlVgpuInstanceGetGpuInstanceId` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetGpuInstanceId`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5327-5335: Function `nvmlVgpuInstanceGetGpuPciId` / 函数 `nvmlVgpuInstanceGetGpuPciId`
```python
@convertStrBytes
def nvmlVgpuInstanceGetGpuPciId(vgpuInstance):
    c_vgpuPciId = create_string_buffer(NVML_DEVICE_PCI_BUS_ID_BUFFER_SIZE)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetGpuPciId")
    ret = fn(
        vgpuInstance, c_vgpuPciId, byref(c_uint(NVML_DEVICE_PCI_BUS_ID_BUFFER_SIZE))
    )
    _nvmlCheckReturn(ret)
    return c_vgpuPciId.value
```
**EN:** This function drives `nvmlVgpuInstanceGetGpuPciId` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetGpuPciId`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5338-5370: Function `nvmlDeviceGetVgpuUtilization` / 函数 `nvmlDeviceGetVgpuUtilization`
```python
def nvmlDeviceGetVgpuUtilization(handle, timeStamp):
    # first call to get the size
    c_vgpu_count = c_uint(0)
    c_time_stamp = c_ulonglong(timeStamp)
    c_sample_value_type = _nvmlValueType_t()

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuUtilization")
    ret = fn(
        handle, c_time_stamp, byref(c_sample_value_type), byref(c_vgpu_count), None
    )

    if ret == NVML_SUCCESS:
        # special case, no active vGPUs
        return []
# ...
        return c_samples[0 : c_vgpu_count.value]
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetVgpuUtilization` with inputs such as `handle`, `timeStamp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuUtilization`，主要处理 `handle`, `timeStamp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5373-5403: Function `nvmlDeviceGetVgpuInstancesUtilizationInfo` / 函数 `nvmlDeviceGetVgpuInstancesUtilizationInfo`
```python
def nvmlDeviceGetVgpuInstancesUtilizationInfo(handle, timeStamp):
    # first call to get the size
    c_time_stamp = c_ulonglong(timeStamp)
    c_vgpuUtilInfo = c_nvmlVgpuInstancesUtilizationInfo_v1_t(0)
    c_vgpuUtilInfo.version = VgpuInstancesUtilizationInfo_v1
    c_vgpuUtilInfo.sampleValType = _nvmlValueType_t()
    c_vgpuUtilInfo.vgpuInstanceCount = c_uint(0)
    c_vgpuUtilInfo.lastSeenTimeStamp = c_time_stamp

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuInstancesUtilizationInfo")
    ret = fn(handle, byref(c_vgpuUtilInfo))

    if ret == NVML_SUCCESS:
        # special case, no active vGPUs
# ...
        return c_samples[0 : c_vgpuUtilInfo.vgpuInstanceCount]
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetVgpuInstancesUtilizationInfo` with inputs such as `handle`, `timeStamp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuInstancesUtilizationInfo`，主要处理 `handle`, `timeStamp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5406-5411: Function `nvmlDeviceGetP2PStatus` / 函数 `nvmlDeviceGetP2PStatus`
```python
def nvmlDeviceGetP2PStatus(device1, device2, p2pIndex):
    c_p2pstatus = _nvmlGpuP2PStatus_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetP2PStatus")
    ret = fn(device1, device2, p2pIndex, byref(c_p2pstatus))
    _nvmlCheckReturn(ret)
    return c_p2pstatus.value
```
**EN:** This function drives `nvmlDeviceGetP2PStatus` with inputs such as `device1`, `device2`, `p2pIndex`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetP2PStatus`，主要处理 `device1`, `device2`, `p2pIndex` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5414-5420: Function `nvmlDeviceGetGridLicensableFeatures_v4` / 函数 `nvmlDeviceGetGridLicensableFeatures_v4`
```python
def nvmlDeviceGetGridLicensableFeatures_v4(handle):
    c_get_grid_licensable_features = c_nvmlGridLicensableFeatures_v4_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGridLicensableFeatures_v4")
    ret = fn(handle, byref(c_get_grid_licensable_features))
    _nvmlCheckReturn(ret)

    return c_get_grid_licensable_features
```
**EN:** This function drives `nvmlDeviceGetGridLicensableFeatures_v4` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGridLicensableFeatures_v4`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5423-5424: Function `nvmlDeviceGetGridLicensableFeatures` / 函数 `nvmlDeviceGetGridLicensableFeatures`
```python
def nvmlDeviceGetGridLicensableFeatures(handle):
    return nvmlDeviceGetGridLicensableFeatures_v4(handle)
```
**EN:** This function drives `nvmlDeviceGetGridLicensableFeatures` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGridLicensableFeatures`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5427-5434: Function `nvmlDeviceGetGspFirmwareVersion` / 函数 `nvmlDeviceGetGspFirmwareVersion`
```python
def nvmlDeviceGetGspFirmwareVersion(handle, version=None):
    isUserDefined = version is not None
    if not isUserDefined:
        version = (c_char * NVML_GSP_FIRMWARE_VERSION_BUF_SIZE)()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGspFirmwareVersion")
    ret = fn(handle, version)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isUserDefined else version.value
```
**EN:** This function drives `nvmlDeviceGetGspFirmwareVersion` with inputs such as `handle`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGspFirmwareVersion`，主要处理 `handle`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5437-5444: Function `nvmlDeviceGetGspFirmwareMode` / 函数 `nvmlDeviceGetGspFirmwareMode`
```python
def nvmlDeviceGetGspFirmwareMode(handle, isEnabled=c_uint(), defaultMode=c_uint()):
    isReference = type(isEnabled) is not c_uint
    isEnabledRef = isEnabled if isReference else byref(isEnabled)
    defaultModeRef = defaultMode if isReference else byref(defaultMode)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGspFirmwareMode")
    ret = fn(handle, isEnabledRef, defaultModeRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else [isEnabled.value, defaultMode.value]
```
**EN:** This function drives `nvmlDeviceGetGspFirmwareMode` with inputs such as `handle`, `isEnabled`, `defaultMode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGspFirmwareMode`，主要处理 `handle`, `isEnabled`, `defaultMode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5447-5454: Function `nvmlDeviceGetEncoderCapacity` / 函数 `nvmlDeviceGetEncoderCapacity`
```python
def nvmlDeviceGetEncoderCapacity(handle, encoderQueryType):
    c_encoder_capacity = c_ulonglong(0)
    c_encoderQuery_type = _nvmlEncoderQueryType_t(encoderQueryType)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetEncoderCapacity")
    ret = fn(handle, c_encoderQuery_type, byref(c_encoder_capacity))
    _nvmlCheckReturn(ret)
    return c_encoder_capacity.value
```
**EN:** This function drives `nvmlDeviceGetEncoderCapacity` with inputs such as `handle`, `encoderQueryType`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetEncoderCapacity`，主要处理 `handle`, `encoderQueryType` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5457-5480: Function `nvmlDeviceGetVgpuProcessUtilization` / 函数 `nvmlDeviceGetVgpuProcessUtilization`
```python
def nvmlDeviceGetVgpuProcessUtilization(handle, timeStamp):
    # first call to get the size
    c_vgpu_count = c_uint(0)
    c_time_stamp = c_ulonglong(timeStamp)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuProcessUtilization")
    ret = fn(handle, c_time_stamp, byref(c_vgpu_count), None)

    if ret == NVML_SUCCESS:
        # special case, no active vGPUs
        return []
    elif ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        sampleArray = c_vgpu_count.value * c_nvmlVgpuProcessUtilizationSample_t
# ...
        return c_samples[0 : c_vgpu_count.value]
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetVgpuProcessUtilization` with inputs such as `handle`, `timeStamp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuProcessUtilization`，主要处理 `handle`, `timeStamp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5483-5512: Function `nvmlDeviceGetVgpuProcessesUtilizationInfo` / 函数 `nvmlDeviceGetVgpuProcessesUtilizationInfo`
```python
def nvmlDeviceGetVgpuProcessesUtilizationInfo(handle, timeStamp):
    # first call to get the size
    c_time_stamp = c_ulonglong(timeStamp)
    c_vgpuProcUtilInfo = c_nvmlVgpuProcessesUtilizationInfo_v1_t(0)
    c_vgpuProcUtilInfo.version = VgpuProcessesUtilizationInfo_v1
    c_vgpuProcUtilInfo.vgpuProcessCount = c_uint(0)
    c_vgpuProcUtilInfo.lastSeenTimeStamp = c_time_stamp

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuProcessesUtilizationInfo")
    ret = fn(handle, byref(c_vgpuProcUtilInfo))

    if ret == NVML_SUCCESS:
        # special case, no active vGPUs
        return []
# ...
        return c_samples[0 : c_vgpuProcUtilInfo.vgpuProcessCount]
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetVgpuProcessesUtilizationInfo` with inputs such as `handle`, `timeStamp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuProcessesUtilizationInfo`，主要处理 `handle`, `timeStamp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5515-5522: Function `nvmlDeviceGetEncoderStats` / 函数 `nvmlDeviceGetEncoderStats`
```python
def nvmlDeviceGetEncoderStats(handle):
    c_encoderCount = c_ulonglong(0)
    c_encodeFps = c_ulonglong(0)
    c_encoderLatency = c_ulonglong(0)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetEncoderStats")
    ret = fn(handle, byref(c_encoderCount), byref(c_encodeFps), byref(c_encoderLatency))
    _nvmlCheckReturn(ret)
    return (c_encoderCount.value, c_encodeFps.value, c_encoderLatency.value)
```
**EN:** This function drives `nvmlDeviceGetEncoderStats` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetEncoderStats`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5525-5549: Function `nvmlDeviceGetEncoderSessions` / 函数 `nvmlDeviceGetEncoderSessions`
```python
def nvmlDeviceGetEncoderSessions(handle):
    # first call to get the size
    c_session_count = c_uint(0)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetEncoderSessions")
    ret = fn(handle, byref(c_session_count), None)

    if ret == NVML_SUCCESS:
        if c_session_count.value != 0:
            # typical case
            session_array = c_nvmlEncoderSession_t * c_session_count.value
            c_sessions = session_array()

            # make the call again
# ...
            return []  # no active sessions
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetEncoderSessions` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetEncoderSessions`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5552-5557: Function `nvmlDeviceGetFBCStats` / 函数 `nvmlDeviceGetFBCStats`
```python
def nvmlDeviceGetFBCStats(handle):
    c_fbcStats = c_nvmlFBCStats_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetFBCStats")
    ret = fn(handle, byref(c_fbcStats))
    _nvmlCheckReturn(ret)
    return c_fbcStats
```
**EN:** This function drives `nvmlDeviceGetFBCStats` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetFBCStats`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5560-5584: Function `nvmlDeviceGetFBCSessions` / 函数 `nvmlDeviceGetFBCSessions`
```python
def nvmlDeviceGetFBCSessions(handle):
    # first call to get the size
    c_session_count = c_uint(0)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetFBCSessions")
    ret = fn(handle, byref(c_session_count), None)

    if ret == NVML_SUCCESS:
        if c_session_count.value != 0:
            # typical case
            session_array = c_nvmlFBCSession_t * c_session_count.value
            c_sessions = session_array()

            # make the call again
# ...
            return []  # no active sessions
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetFBCSessions` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetFBCSessions`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5587-5596: Function `nvmlVgpuInstanceGetEncoderStats` / 函数 `nvmlVgpuInstanceGetEncoderStats`
```python
def nvmlVgpuInstanceGetEncoderStats(vgpuInstance):
    c_encoderCount = c_ulonglong(0)
    c_encodeFps = c_ulonglong(0)
    c_encoderLatency = c_ulonglong(0)
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetEncoderStats")
    ret = fn(
        vgpuInstance, byref(c_encoderCount), byref(c_encodeFps), byref(c_encoderLatency)
    )
    _nvmlCheckReturn(ret)
    return (c_encoderCount.value, c_encodeFps.value, c_encoderLatency.value)
```
**EN:** This function drives `nvmlVgpuInstanceGetEncoderStats` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetEncoderStats`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5599-5623: Function `nvmlVgpuInstanceGetEncoderSessions` / 函数 `nvmlVgpuInstanceGetEncoderSessions`
```python
def nvmlVgpuInstanceGetEncoderSessions(vgpuInstance):
    # first call to get the size
    c_session_count = c_uint(0)

    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetEncoderSessions")
    ret = fn(vgpuInstance, byref(c_session_count), None)

    if ret == NVML_SUCCESS:
        if c_session_count.value != 0:
            # typical case
            session_array = c_nvmlEncoderSession_t * c_session_count.value
            c_sessions = session_array()

            # make the call again
# ...
            return []  # no active sessions
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlVgpuInstanceGetEncoderSessions` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetEncoderSessions`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5626-5631: Function `nvmlVgpuInstanceGetFBCStats` / 函数 `nvmlVgpuInstanceGetFBCStats`
```python
def nvmlVgpuInstanceGetFBCStats(vgpuInstance):
    c_fbcStats = c_nvmlFBCStats_t()
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetFBCStats")
    ret = fn(vgpuInstance, byref(c_fbcStats))
    _nvmlCheckReturn(ret)
    return c_fbcStats
```
**EN:** This function drives `nvmlVgpuInstanceGetFBCStats` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetFBCStats`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5634-5658: Function `nvmlVgpuInstanceGetFBCSessions` / 函数 `nvmlVgpuInstanceGetFBCSessions`
```python
def nvmlVgpuInstanceGetFBCSessions(vgpuInstance):
    # first call to get the size
    c_session_count = c_uint(0)

    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetFBCSessions")
    ret = fn(vgpuInstance, byref(c_session_count), None)

    if ret == NVML_SUCCESS:
        if c_session_count.value != 0:
            # typical case
            session_array = c_nvmlFBCSession_t * c_session_count.value
            c_sessions = session_array()

            # make the call again
# ...
            return []  # no active sessions
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlVgpuInstanceGetFBCSessions` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetFBCSessions`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5661-5681: Function `nvmlDeviceGetProcessUtilization` / 函数 `nvmlDeviceGetProcessUtilization`
```python
def nvmlDeviceGetProcessUtilization(handle, timeStamp):
    # first call to get the size
    c_count = c_uint(0)
    c_time_stamp = c_ulonglong(timeStamp)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetProcessUtilization")
    ret = fn(handle, None, byref(c_count), c_time_stamp)

    if ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        sampleArray = c_count.value * c_nvmlProcessUtilizationSample_t
        c_samples = sampleArray()

        # make the call again
# ...
        return c_samples[0 : c_count.value]
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetProcessUtilization` with inputs such as `handle`, `timeStamp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetProcessUtilization`，主要处理 `handle`, `timeStamp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5684-5710: Function `nvmlDeviceGetProcessesUtilizationInfo` / 函数 `nvmlDeviceGetProcessesUtilizationInfo`
```python
def nvmlDeviceGetProcessesUtilizationInfo(handle, timeStamp):
    # first call to get the size
    c_time_stamp = c_ulonglong(timeStamp)
    c_processesUtilInfo = c_nvmlProcessesUtilizationInfo_v1_t(0)
    c_processesUtilInfo.version = ProcessesUtilizationInfo_v1
    c_processesUtilInfo.processSamplesCount = c_uint(0)
    c_processesUtilInfo.lastSeenTimeStamp = c_time_stamp

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetProcessesUtilizationInfo")
    ret = fn(handle, byref(c_processesUtilInfo))

    if ret == NVML_ERROR_INSUFFICIENT_SIZE:
        # typical case
        sampleArray = (
# ...
        return c_samples[0 : c_processesUtilInfo.processSamplesCount]
    else:
        # error case
        raise NVMLError(ret)
```
**EN:** This function drives `nvmlDeviceGetProcessesUtilizationInfo` with inputs such as `handle`, `timeStamp`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetProcessesUtilizationInfo`，主要处理 `handle`, `timeStamp` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5713-5725: Function `nvmlVgpuInstanceGetMetadata` / 函数 `nvmlVgpuInstanceGetMetadata`
```python
def nvmlVgpuInstanceGetMetadata(vgpuInstance):
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetMetadata")
    c_vgpuMetadata = c_nvmlVgpuMetadata_t()
    c_bufferSize = c_uint(0)
    # Make the first NVML API call to get the c_bufferSize value.
    # We have already allocated required buffer above.
    ret = fn(vgpuInstance, byref(c_vgpuMetadata), byref(c_bufferSize))
    if ret == NVML_ERROR_INSUFFICIENT_SIZE:
        ret = fn(vgpuInstance, byref(c_vgpuMetadata), byref(c_bufferSize))
        _nvmlCheckReturn(ret)
    else:
        raise NVMLError(ret)
    return c_vgpuMetadata
```
**EN:** This function drives `nvmlVgpuInstanceGetMetadata` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetMetadata`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5728-5740: Function `nvmlDeviceGetVgpuMetadata` / 函数 `nvmlDeviceGetVgpuMetadata`
```python
def nvmlDeviceGetVgpuMetadata(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuMetadata")
    c_vgpuPgpuMetadata = c_nvmlVgpuPgpuMetadata_t()
    c_bufferSize = c_uint(0)
    # Make the first NVML API call to get the c_bufferSize value.
    # We have already allocated required buffer above.
    ret = fn(handle, byref(c_vgpuPgpuMetadata), byref(c_bufferSize))
    if ret == NVML_ERROR_INSUFFICIENT_SIZE:
        ret = fn(handle, byref(c_vgpuPgpuMetadata), byref(c_bufferSize))
        _nvmlCheckReturn(ret)
    else:
        raise NVMLError(ret)
    return c_vgpuPgpuMetadata
```
**EN:** This function drives `nvmlDeviceGetVgpuMetadata` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuMetadata`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5743-5748: Function `nvmlGetVgpuCompatibility` / 函数 `nvmlGetVgpuCompatibility`
```python
def nvmlGetVgpuCompatibility(vgpuMetadata, pgpuMetadata):
    fn = _nvmlGetFunctionPointer("nvmlGetVgpuCompatibility")
    c_vgpuPgpuCompatibility = c_nvmlVgpuPgpuCompatibility_t()
    ret = fn(byref(vgpuMetadata), byref(pgpuMetadata), byref(c_vgpuPgpuCompatibility))
    _nvmlCheckReturn(ret)
    return c_vgpuPgpuCompatibility
```
**EN:** This function drives `nvmlGetVgpuCompatibility` with inputs such as `vgpuMetadata`, `pgpuMetadata`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGetVgpuCompatibility`，主要处理 `vgpuMetadata`, `pgpuMetadata` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5751-5764: Function `nvmlDeviceGetPgpuMetadataString` / 函数 `nvmlDeviceGetPgpuMetadataString`
```python
@convertStrBytes
def nvmlDeviceGetPgpuMetadataString(handle):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPgpuMetadataString")
    c_pgpuMetadata = create_string_buffer(NVML_VGPU_PGPU_METADATA_OPAQUE_DATA_SIZE)
    c_bufferSize = c_uint(0)
    # Make the first NVML API call to get the c_bufferSize value.
    # We have already allocated required buffer above.
    ret = fn(handle, byref(c_pgpuMetadata), byref(c_bufferSize))
    if ret == NVML_ERROR_INSUFFICIENT_SIZE:
        ret = fn(handle, byref(c_pgpuMetadata), byref(c_bufferSize))
        _nvmlCheckReturn(ret)
    else:
        raise NVMLError(ret)
    return (c_pgpuMetadata.value, c_bufferSize.value)
```
**EN:** This function drives `nvmlDeviceGetPgpuMetadataString` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPgpuMetadataString`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5767-5772: Function `nvmlDeviceGetVgpuSchedulerLog` / 函数 `nvmlDeviceGetVgpuSchedulerLog`
```python
def nvmlDeviceGetVgpuSchedulerLog(handle):
    c_vgpu_sched_log = c_nvmlVgpuSchedulerLog_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuSchedulerLog")
    ret = fn(handle, byref(c_vgpu_sched_log))
    _nvmlCheckReturn(ret)
    return c_vgpu_sched_log
```
**EN:** This function drives `nvmlDeviceGetVgpuSchedulerLog` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuSchedulerLog`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5775-5780: Function `nvmlDeviceGetVgpuSchedulerState` / 函数 `nvmlDeviceGetVgpuSchedulerState`
```python
def nvmlDeviceGetVgpuSchedulerState(handle):
    c_vgpu_sched_state = c_nvmlVgpuSchedulerGetState_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuSchedulerState")
    ret = fn(handle, byref(c_vgpu_sched_state))
    _nvmlCheckReturn(ret)
    return c_vgpu_sched_state
```
**EN:** This function drives `nvmlDeviceGetVgpuSchedulerState` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuSchedulerState`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5783-5788: Function `nvmlDeviceGetVgpuSchedulerCapabilities` / 函数 `nvmlDeviceGetVgpuSchedulerCapabilities`
```python
def nvmlDeviceGetVgpuSchedulerCapabilities(handle):
    c_vgpu_sched_caps = c_nvmlVgpuSchedulerCapabilities_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetVgpuSchedulerCapabilities")
    ret = fn(handle, byref(c_vgpu_sched_caps))
    _nvmlCheckReturn(ret)
    return c_vgpu_sched_caps
```
**EN:** This function drives `nvmlDeviceGetVgpuSchedulerCapabilities` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetVgpuSchedulerCapabilities`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5791-5795: Function `nvmlDeviceSetVgpuSchedulerState` / 函数 `nvmlDeviceSetVgpuSchedulerState`
```python
def nvmlDeviceSetVgpuSchedulerState(handle, sched_state):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetVgpuSchedulerState")
    ret = fn(handle, byref(sched_state))
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetVgpuSchedulerState` with inputs such as `handle`, `sched_state`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetVgpuSchedulerState`，主要处理 `handle`, `sched_state` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5798-5802: Function `nvmlSetVgpuVersion` / 函数 `nvmlSetVgpuVersion`
```python
def nvmlSetVgpuVersion(vgpuVersion):
    fn = _nvmlGetFunctionPointer("nvmlSetVgpuVersion")
    ret = fn(byref(vgpuVersion))
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlSetVgpuVersion` with inputs such as `vgpuVersion`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSetVgpuVersion`，主要处理 `vgpuVersion` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5805-5820: Function `nvmlGetVgpuVersion` / 函数 `nvmlGetVgpuVersion`
```python
def nvmlGetVgpuVersion(supported=None, current=None):
    isUserDefined = (supported is not None) or (current is not None)
    if not isUserDefined:
        supported = c_nvmlVgpuVersion_t()
        current = c_nvmlVgpuVersion_t()
    fn = _nvmlGetFunctionPointer("nvmlGetVgpuVersion")
    ret = fn(byref(supported), byref(current))
    _nvmlCheckReturn(ret)
    return (
        NVML_SUCCESS
        if isUserDefined
        else [
            (supported.minVersion, supported.maxVersion),
            (current.minVersion, current.maxVersion),
        ]
    )
```
**EN:** This function drives `nvmlGetVgpuVersion` with inputs such as `supported`, `current`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGetVgpuVersion`，主要处理 `supported`, `current` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5823-5828: Function `nvmlVgpuInstanceGetAccountingMode` / 函数 `nvmlVgpuInstanceGetAccountingMode`
```python
def nvmlVgpuInstanceGetAccountingMode(vgpuInstance):
    c_mode = _nvmlEnableState_t()
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetAccountingMode")
    ret = fn(vgpuInstance, byref(c_mode))
    _nvmlCheckReturn(ret)
    return c_mode.value
```
**EN:** This function drives `nvmlVgpuInstanceGetAccountingMode` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetAccountingMode`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5831-5842: Function `nvmlVgpuInstanceGetAccountingPids` / 函数 `nvmlVgpuInstanceGetAccountingPids`
```python
def nvmlVgpuInstanceGetAccountingPids(vgpuInstance):
    c_pidCount = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetAccountingPids")
    ret = fn(vgpuInstance, byref(c_pidCount), None)
    if ret == NVML_ERROR_INSUFFICIENT_SIZE:
        sampleArray = c_pidCount.value * c_uint
        c_pidArray = sampleArray()
        ret = fn(vgpuInstance, byref(c_pidCount), byref(c_pidArray))
        _nvmlCheckReturn(ret)
    else:
        raise NVMLError(ret)
    return (c_pidCount, c_pidArray)
```
**EN:** This function drives `nvmlVgpuInstanceGetAccountingPids` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetAccountingPids`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5845-5850: Function `nvmlVgpuInstanceGetAccountingStats` / 函数 `nvmlVgpuInstanceGetAccountingStats`
```python
def nvmlVgpuInstanceGetAccountingStats(vgpuInstance, pid):
    c_accountingStats = c_nvmlAccountingStats_t()
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceGetAccountingStats")
    ret = fn(vgpuInstance, pid, byref(c_accountingStats))
    _nvmlCheckReturn(ret)
    return c_accountingStats
```
**EN:** This function drives `nvmlVgpuInstanceGetAccountingStats` with inputs such as `vgpuInstance`, `pid`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceGetAccountingStats`，主要处理 `vgpuInstance`, `pid` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5853-5857: Function `nvmlVgpuInstanceClearAccountingPids` / 函数 `nvmlVgpuInstanceClearAccountingPids`
```python
def nvmlVgpuInstanceClearAccountingPids(vgpuInstance):
    fn = _nvmlGetFunctionPointer("nvmlVgpuInstanceClearAccountingPids")
    ret = fn(vgpuInstance)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlVgpuInstanceClearAccountingPids` with inputs such as `vgpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlVgpuInstanceClearAccountingPids`，主要处理 `vgpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5860-5865: Function `nvmlGetExcludedDeviceCount` / 函数 `nvmlGetExcludedDeviceCount`
```python
def nvmlGetExcludedDeviceCount():
    c_count = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlGetExcludedDeviceCount")
    ret = fn(byref(c_count))
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlGetExcludedDeviceCount`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGetExcludedDeviceCount`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5868-5874: Function `nvmlGetExcludedDeviceInfoByIndex` / 函数 `nvmlGetExcludedDeviceInfoByIndex`
```python
def nvmlGetExcludedDeviceInfoByIndex(index):
    c_index = c_uint(index)
    info = c_nvmlExcludedDeviceInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlGetExcludedDeviceInfoByIndex")
    ret = fn(c_index, byref(info))
    _nvmlCheckReturn(ret)
    return info
```
**EN:** This function drives `nvmlGetExcludedDeviceInfoByIndex` with inputs such as `index`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGetExcludedDeviceInfoByIndex`，主要处理 `index` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5877-5882: Function `nvmlDeviceGetHostVgpuMode` / 函数 `nvmlDeviceGetHostVgpuMode`
```python
def nvmlDeviceGetHostVgpuMode(handle):
    c_host_vgpu_mode = _nvmlHostVgpuMode_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetHostVgpuMode")
    ret = fn(handle, byref(c_host_vgpu_mode))
    _nvmlCheckReturn(ret)
    return c_host_vgpu_mode.value
```
**EN:** This function drives `nvmlDeviceGetHostVgpuMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetHostVgpuMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5885-5890: Function `nvmlDeviceSetMigMode` / 函数 `nvmlDeviceSetMigMode`
```python
def nvmlDeviceSetMigMode(device, mode):
    c_activationStatus = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetMigMode")
    ret = fn(device, mode, byref(c_activationStatus))
    _nvmlCheckReturn(ret)
    return c_activationStatus.value
```
**EN:** This function drives `nvmlDeviceSetMigMode` with inputs such as `device`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetMigMode`，主要处理 `device`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5893-5899: Function `nvmlDeviceGetMigMode` / 函数 `nvmlDeviceGetMigMode`
```python
def nvmlDeviceGetMigMode(device):
    c_currentMode = c_uint()
    c_pendingMode = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMigMode")
    ret = fn(device, byref(c_currentMode), byref(c_pendingMode))
    _nvmlCheckReturn(ret)
    return [c_currentMode.value, c_pendingMode.value]
```
**EN:** This function drives `nvmlDeviceGetMigMode` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMigMode`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5902-5913: Function `nvmlDeviceGetGpuInstanceProfileInfo` / 函数 `nvmlDeviceGetGpuInstanceProfileInfo`
```python
def nvmlDeviceGetGpuInstanceProfileInfo(device, profile, version=2):
    if version == 2:
        c_info = c_nvmlGpuInstanceProfileInfo_v2_t()
        fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuInstanceProfileInfoV")
    elif version == 1:
        c_info = c_nvmlGpuInstanceProfileInfo_t()
        fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuInstanceProfileInfo")
    else:
        raise NVMLError(NVML_ERROR_FUNCTION_NOT_FOUND)
    ret = fn(device, profile, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlDeviceGetGpuInstanceProfileInfo` with inputs such as `device`, `profile`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuInstanceProfileInfo`，主要处理 `device`, `profile`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5914-5917: Top-level configuration / 顶层配置
```python


# Define function alias for the API exposed by NVML
nvmlDeviceGetGpuInstanceProfileInfoV = nvmlDeviceGetGpuInstanceProfileInfo
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 5920-5925: Function `nvmlDeviceGetGpuInstanceRemainingCapacity` / 函数 `nvmlDeviceGetGpuInstanceRemainingCapacity`
```python
def nvmlDeviceGetGpuInstanceRemainingCapacity(device, profileId):
    c_count = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuInstanceRemainingCapacity")
    ret = fn(device, profileId, byref(c_count))
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlDeviceGetGpuInstanceRemainingCapacity` with inputs such as `device`, `profileId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuInstanceRemainingCapacity`，主要处理 `device`, `profileId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5928-5934: Function `nvmlDeviceGetGpuInstancePossiblePlacements` / 函数 `nvmlDeviceGetGpuInstancePossiblePlacements`
```python
def nvmlDeviceGetGpuInstancePossiblePlacements(
    device, profileId, placementsRef, countRef
):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuInstancePossiblePlacements_v2")
    ret = fn(device, profileId, placementsRef, countRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetGpuInstancePossiblePlacements` with inputs such as `device`, `profileId`, `placementsRef`, `countRef`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuInstancePossiblePlacements`，主要处理 `device`, `profileId`, `placementsRef`, `countRef` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5937-5942: Function `nvmlDeviceCreateGpuInstance` / 函数 `nvmlDeviceCreateGpuInstance`
```python
def nvmlDeviceCreateGpuInstance(device, profileId):
    c_instance = c_nvmlGpuInstance_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceCreateGpuInstance")
    ret = fn(device, profileId, byref(c_instance))
    _nvmlCheckReturn(ret)
    return c_instance
```
**EN:** This function drives `nvmlDeviceCreateGpuInstance` with inputs such as `device`, `profileId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceCreateGpuInstance`，主要处理 `device`, `profileId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5945-5950: Function `nvmlDeviceCreateGpuInstanceWithPlacement` / 函数 `nvmlDeviceCreateGpuInstanceWithPlacement`
```python
def nvmlDeviceCreateGpuInstanceWithPlacement(device, profileId, placement):
    c_instance = c_nvmlGpuInstance_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceCreateGpuInstanceWithPlacement")
    ret = fn(device, profileId, placement, byref(c_instance))
    _nvmlCheckReturn(ret)
    return c_instance
```
**EN:** This function drives `nvmlDeviceCreateGpuInstanceWithPlacement` with inputs such as `device`, `profileId`, `placement`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceCreateGpuInstanceWithPlacement`，主要处理 `device`, `profileId`, `placement` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5953-5957: Function `nvmlGpuInstanceDestroy` / 函数 `nvmlGpuInstanceDestroy`
```python
def nvmlGpuInstanceDestroy(gpuInstance):
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceDestroy")
    ret = fn(gpuInstance)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlGpuInstanceDestroy` with inputs such as `gpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceDestroy`，主要处理 `gpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5960-5964: Function `nvmlDeviceGetGpuInstances` / 函数 `nvmlDeviceGetGpuInstances`
```python
def nvmlDeviceGetGpuInstances(device, profileId, gpuInstancesRef, countRef):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuInstances")
    ret = fn(device, profileId, gpuInstancesRef, countRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetGpuInstances` with inputs such as `device`, `profileId`, `gpuInstancesRef`, `countRef`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuInstances`，主要处理 `device`, `profileId`, `gpuInstancesRef`, `countRef` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5967-5972: Function `nvmlDeviceGetGpuInstanceById` / 函数 `nvmlDeviceGetGpuInstanceById`
```python
def nvmlDeviceGetGpuInstanceById(device, gpuInstanceId):
    c_instance = c_nvmlGpuInstance_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuInstanceById")
    ret = fn(device, gpuInstanceId, byref(c_instance))
    _nvmlCheckReturn(ret)
    return c_instance
```
**EN:** This function drives `nvmlDeviceGetGpuInstanceById` with inputs such as `device`, `gpuInstanceId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuInstanceById`，主要处理 `device`, `gpuInstanceId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5975-5980: Function `nvmlGpuInstanceGetInfo` / 函数 `nvmlGpuInstanceGetInfo`
```python
def nvmlGpuInstanceGetInfo(gpuInstance):
    c_info = c_nvmlGpuInstanceInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceGetInfo")
    ret = fn(gpuInstance, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlGpuInstanceGetInfo` with inputs such as `gpuInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceGetInfo`，主要处理 `gpuInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5983-5996: Function `nvmlGpuInstanceGetComputeInstanceProfileInfo` / 函数 `nvmlGpuInstanceGetComputeInstanceProfileInfo`
```python
def nvmlGpuInstanceGetComputeInstanceProfileInfo(
    device, profile, engProfile, version=2
):
    if version == 2:
        c_info = c_nvmlComputeInstanceProfileInfo_v2_t()
        fn = _nvmlGetFunctionPointer("nvmlGpuInstanceGetComputeInstanceProfileInfoV")
    elif version == 1:
        c_info = c_nvmlComputeInstanceProfileInfo_t()
        fn = _nvmlGetFunctionPointer("nvmlGpuInstanceGetComputeInstanceProfileInfo")
    else:
        raise NVMLError(NVML_ERROR_FUNCTION_NOT_FOUND)
    ret = fn(device, profile, engProfile, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlGpuInstanceGetComputeInstanceProfileInfo` with inputs such as `device`, `profile`, `engProfile`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceGetComputeInstanceProfileInfo`，主要处理 `device`, `profile`, `engProfile`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 5997-6002: Top-level configuration / 顶层配置
```python


# Define function alias for the API exposed by NVML
nvmlGpuInstanceGetComputeInstanceProfileInfoV = (
    nvmlGpuInstanceGetComputeInstanceProfileInfo
)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6005-6010: Function `nvmlGpuInstanceGetComputeInstanceRemainingCapacity` / 函数 `nvmlGpuInstanceGetComputeInstanceRemainingCapacity`
```python
def nvmlGpuInstanceGetComputeInstanceRemainingCapacity(gpuInstance, profileId):
    c_count = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceGetComputeInstanceRemainingCapacity")
    ret = fn(gpuInstance, profileId, byref(c_count))
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlGpuInstanceGetComputeInstanceRemainingCapacity` with inputs such as `gpuInstance`, `profileId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceGetComputeInstanceRemainingCapacity`，主要处理 `gpuInstance`, `profileId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6013-6019: Function `nvmlGpuInstanceGetComputeInstancePossiblePlacements` / 函数 `nvmlGpuInstanceGetComputeInstancePossiblePlacements`
```python
def nvmlGpuInstanceGetComputeInstancePossiblePlacements(
    gpuInstance, profileId, placementsRef, countRef
):
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceGetComputeInstancePossiblePlacements")
    ret = fn(gpuInstance, profileId, placementsRef, countRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlGpuInstanceGetComputeInstancePossiblePlacements` with inputs such as `gpuInstance`, `profileId`, `placementsRef`, `countRef`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceGetComputeInstancePossiblePlacements`，主要处理 `gpuInstance`, `profileId`, `placementsRef`, `countRef` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6022-6027: Function `nvmlGpuInstanceCreateComputeInstance` / 函数 `nvmlGpuInstanceCreateComputeInstance`
```python
def nvmlGpuInstanceCreateComputeInstance(gpuInstance, profileId):
    c_instance = c_nvmlComputeInstance_t()
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceCreateComputeInstance")
    ret = fn(gpuInstance, profileId, byref(c_instance))
    _nvmlCheckReturn(ret)
    return c_instance
```
**EN:** This function drives `nvmlGpuInstanceCreateComputeInstance` with inputs such as `gpuInstance`, `profileId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceCreateComputeInstance`，主要处理 `gpuInstance`, `profileId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6030-6037: Function `nvmlGpuInstanceCreateComputeInstanceWithPlacement` / 函数 `nvmlGpuInstanceCreateComputeInstanceWithPlacement`
```python
def nvmlGpuInstanceCreateComputeInstanceWithPlacement(
    gpuInstance, profileId, placement
):
    c_instance = c_nvmlComputeInstance_t()
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceCreateComputeInstanceWithPlacement")
    ret = fn(gpuInstance, profileId, placement, byref(c_instance))
    _nvmlCheckReturn(ret)
    return c_instance
```
**EN:** This function drives `nvmlGpuInstanceCreateComputeInstanceWithPlacement` with inputs such as `gpuInstance`, `profileId`, `placement`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceCreateComputeInstanceWithPlacement`，主要处理 `gpuInstance`, `profileId`, `placement` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6040-6044: Function `nvmlComputeInstanceDestroy` / 函数 `nvmlComputeInstanceDestroy`
```python
def nvmlComputeInstanceDestroy(computeInstance):
    fn = _nvmlGetFunctionPointer("nvmlComputeInstanceDestroy")
    ret = fn(computeInstance)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlComputeInstanceDestroy` with inputs such as `computeInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlComputeInstanceDestroy`，主要处理 `computeInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6047-6053: Function `nvmlGpuInstanceGetComputeInstances` / 函数 `nvmlGpuInstanceGetComputeInstances`
```python
def nvmlGpuInstanceGetComputeInstances(
    gpuInstance, profileId, computeInstancesRef, countRef
):
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceGetComputeInstances")
    ret = fn(gpuInstance, profileId, computeInstancesRef, countRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlGpuInstanceGetComputeInstances` with inputs such as `gpuInstance`, `profileId`, `computeInstancesRef`, `countRef`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceGetComputeInstances`，主要处理 `gpuInstance`, `profileId`, `computeInstancesRef`, `countRef` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6056-6061: Function `nvmlGpuInstanceGetComputeInstanceById` / 函数 `nvmlGpuInstanceGetComputeInstanceById`
```python
def nvmlGpuInstanceGetComputeInstanceById(gpuInstance, computeInstanceId):
    c_instance = c_nvmlComputeInstance_t()
    fn = _nvmlGetFunctionPointer("nvmlGpuInstanceGetComputeInstanceById")
    ret = fn(gpuInstance, computeInstanceId, byref(c_instance))
    _nvmlCheckReturn(ret)
    return c_instance
```
**EN:** This function drives `nvmlGpuInstanceGetComputeInstanceById` with inputs such as `gpuInstance`, `computeInstanceId`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpuInstanceGetComputeInstanceById`，主要处理 `gpuInstance`, `computeInstanceId` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6064-6069: Function `nvmlComputeInstanceGetInfo_v2` / 函数 `nvmlComputeInstanceGetInfo_v2`
```python
def nvmlComputeInstanceGetInfo_v2(computeInstance):
    c_info = c_nvmlComputeInstanceInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlComputeInstanceGetInfo_v2")
    ret = fn(computeInstance, byref(c_info))
    _nvmlCheckReturn(ret)
    return c_info
```
**EN:** This function drives `nvmlComputeInstanceGetInfo_v2` with inputs such as `computeInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlComputeInstanceGetInfo_v2`，主要处理 `computeInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6072-6073: Function `nvmlComputeInstanceGetInfo` / 函数 `nvmlComputeInstanceGetInfo`
```python
def nvmlComputeInstanceGetInfo(computeInstance):
    return nvmlComputeInstanceGetInfo_v2(computeInstance)
```
**EN:** This function drives `nvmlComputeInstanceGetInfo` with inputs such as `computeInstance`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlComputeInstanceGetInfo`，主要处理 `computeInstance` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6076-6081: Function `nvmlDeviceIsMigDeviceHandle` / 函数 `nvmlDeviceIsMigDeviceHandle`
```python
def nvmlDeviceIsMigDeviceHandle(device):
    c_isMigDevice = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceIsMigDeviceHandle")
    ret = fn(device, byref(c_isMigDevice))
    _nvmlCheckReturn(ret)
    return c_isMigDevice
```
**EN:** This function drives `nvmlDeviceIsMigDeviceHandle` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceIsMigDeviceHandle`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6084-6089: Function `nvmlDeviceGetGpuInstanceId` / 函数 `nvmlDeviceGetGpuInstanceId`
```python
def nvmlDeviceGetGpuInstanceId(device):
    c_gpuInstanceId = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuInstanceId")
    ret = fn(device, byref(c_gpuInstanceId))
    _nvmlCheckReturn(ret)
    return c_gpuInstanceId.value
```
**EN:** This function drives `nvmlDeviceGetGpuInstanceId` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuInstanceId`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6092-6097: Function `nvmlDeviceGetComputeInstanceId` / 函数 `nvmlDeviceGetComputeInstanceId`
```python
def nvmlDeviceGetComputeInstanceId(device):
    c_computeInstanceId = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetComputeInstanceId")
    ret = fn(device, byref(c_computeInstanceId))
    _nvmlCheckReturn(ret)
    return c_computeInstanceId.value
```
**EN:** This function drives `nvmlDeviceGetComputeInstanceId` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetComputeInstanceId`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6100-6105: Function `nvmlDeviceGetMaxMigDeviceCount` / 函数 `nvmlDeviceGetMaxMigDeviceCount`
```python
def nvmlDeviceGetMaxMigDeviceCount(device):
    c_count = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMaxMigDeviceCount")
    ret = fn(device, byref(c_count))
    _nvmlCheckReturn(ret)
    return c_count.value
```
**EN:** This function drives `nvmlDeviceGetMaxMigDeviceCount` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMaxMigDeviceCount`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6108-6114: Function `nvmlDeviceGetMigDeviceHandleByIndex` / 函数 `nvmlDeviceGetMigDeviceHandleByIndex`
```python
def nvmlDeviceGetMigDeviceHandleByIndex(device, index):
    c_index = c_uint(index)
    migDevice = c_nvmlDevice_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMigDeviceHandleByIndex")
    ret = fn(device, c_index, byref(migDevice))
    _nvmlCheckReturn(ret)
    return migDevice
```
**EN:** This function drives `nvmlDeviceGetMigDeviceHandleByIndex` with inputs such as `device`, `index`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMigDeviceHandleByIndex`，主要处理 `device`, `index` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6117-6122: Function `nvmlDeviceGetDeviceHandleFromMigDeviceHandle` / 函数 `nvmlDeviceGetDeviceHandleFromMigDeviceHandle`
```python
def nvmlDeviceGetDeviceHandleFromMigDeviceHandle(migDevice):
    device = c_nvmlDevice_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDeviceHandleFromMigDeviceHandle")
    ret = fn(migDevice, byref(device))
    _nvmlCheckReturn(ret)
    return device
```
**EN:** This function drives `nvmlDeviceGetDeviceHandleFromMigDeviceHandle` with inputs such as `migDevice`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDeviceHandleFromMigDeviceHandle`，主要处理 `migDevice` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6125-6130: Function `nvmlDeviceGetAttributes_v2` / 函数 `nvmlDeviceGetAttributes_v2`
```python
def nvmlDeviceGetAttributes_v2(device):
    c_attrs = c_nvmlDeviceAttributes()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAttributes_v2")
    ret = fn(device, byref(c_attrs))
    _nvmlCheckReturn(ret)
    return c_attrs
```
**EN:** This function drives `nvmlDeviceGetAttributes_v2` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAttributes_v2`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6133-6134: Function `nvmlDeviceGetAttributes` / 函数 `nvmlDeviceGetAttributes`
```python
def nvmlDeviceGetAttributes(device):
    return nvmlDeviceGetAttributes_v2(device)
```
**EN:** This function drives `nvmlDeviceGetAttributes` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAttributes`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6137-6145: Function `nvmlDeviceGetRemappedRows` / 函数 `nvmlDeviceGetRemappedRows`
```python
def nvmlDeviceGetRemappedRows(device):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetRemappedRows")
    c_corr = c_uint()
    c_unc = c_uint()
    c_bpending = c_uint()
    c_bfailure = c_uint()
    ret = fn(device, byref(c_corr), byref(c_unc), byref(c_bpending), byref(c_bfailure))
    _nvmlCheckReturn(ret)
    return (c_corr.value, c_unc.value, c_bpending.value, c_bfailure.value)
```
**EN:** This function drives `nvmlDeviceGetRemappedRows` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetRemappedRows`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6148-6153: Function `nvmlDeviceGetRowRemapperHistogram` / 函数 `nvmlDeviceGetRowRemapperHistogram`
```python
def nvmlDeviceGetRowRemapperHistogram(device):
    c_vals = c_nvmlRowRemapperHistogramValues()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetRowRemapperHistogram")
    ret = fn(device, byref(c_vals))
    _nvmlCheckReturn(ret)
    return c_vals
```
**EN:** This function drives `nvmlDeviceGetRowRemapperHistogram` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetRowRemapperHistogram`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6156-6161: Function `nvmlDeviceGetArchitecture` / 函数 `nvmlDeviceGetArchitecture`
```python
def nvmlDeviceGetArchitecture(device):
    arch = _nvmlDeviceArchitecture_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetArchitecture")
    ret = fn(device, byref(arch))
    _nvmlCheckReturn(ret)
    return arch.value
```
**EN:** This function drives `nvmlDeviceGetArchitecture` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetArchitecture`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6164-6169: Function `nvmlDeviceGetBusType` / 函数 `nvmlDeviceGetBusType`
```python
def nvmlDeviceGetBusType(device):
    c_busType = _nvmlBusType_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetBusType")
    ret = fn(device, byref(c_busType))
    _nvmlCheckReturn(ret)
    return c_busType.value
```
**EN:** This function drives `nvmlDeviceGetBusType` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetBusType`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6172-6177: Function `nvmlDeviceGetIrqNum` / 函数 `nvmlDeviceGetIrqNum`
```python
def nvmlDeviceGetIrqNum(device):
    c_irqNum = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetIrqNum")
    ret = fn(device, byref(c_irqNum))
    _nvmlCheckReturn(ret)
    return c_irqNum.value
```
**EN:** This function drives `nvmlDeviceGetIrqNum` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetIrqNum`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6180-6185: Function `nvmlDeviceGetNumGpuCores` / 函数 `nvmlDeviceGetNumGpuCores`
```python
def nvmlDeviceGetNumGpuCores(device):
    c_numCores = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNumGpuCores")
    ret = fn(device, byref(c_numCores))
    _nvmlCheckReturn(ret)
    return c_numCores.value
```
**EN:** This function drives `nvmlDeviceGetNumGpuCores` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNumGpuCores`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6188-6193: Function `nvmlDeviceGetPowerSource` / 函数 `nvmlDeviceGetPowerSource`
```python
def nvmlDeviceGetPowerSource(device):
    c_powerSource = _nvmlPowerSource_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPowerSource")
    ret = fn(device, byref(c_powerSource))
    _nvmlCheckReturn(ret)
    return c_powerSource.value
```
**EN:** This function drives `nvmlDeviceGetPowerSource` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPowerSource`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6196-6201: Function `nvmlDeviceGetMemoryBusWidth` / 函数 `nvmlDeviceGetMemoryBusWidth`
```python
def nvmlDeviceGetMemoryBusWidth(device):
    c_memBusWidth = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMemoryBusWidth")
    ret = fn(device, byref(c_memBusWidth))
    _nvmlCheckReturn(ret)
    return c_memBusWidth.value
```
**EN:** This function drives `nvmlDeviceGetMemoryBusWidth` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMemoryBusWidth`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6204-6209: Function `nvmlDeviceGetPcieLinkMaxSpeed` / 函数 `nvmlDeviceGetPcieLinkMaxSpeed`
```python
def nvmlDeviceGetPcieLinkMaxSpeed(device):
    c_speed = _nvmlPcieLinkMaxSpeed_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPcieLinkMaxSpeed")
    ret = fn(device, byref(c_speed))
    _nvmlCheckReturn(ret)
    return c_speed.value
```
**EN:** This function drives `nvmlDeviceGetPcieLinkMaxSpeed` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPcieLinkMaxSpeed`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6212-6217: Function `nvmlDeviceGetAdaptiveClockInfoStatus` / 函数 `nvmlDeviceGetAdaptiveClockInfoStatus`
```python
def nvmlDeviceGetAdaptiveClockInfoStatus(device):
    c_adaptiveClockInfoStatus = _nvmlAdaptiveClockInfoStatus_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetAdaptiveClockInfoStatus")
    ret = fn(device, byref(c_adaptiveClockInfoStatus))
    _nvmlCheckReturn(ret)
    return c_adaptiveClockInfoStatus.value
```
**EN:** This function drives `nvmlDeviceGetAdaptiveClockInfoStatus` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetAdaptiveClockInfoStatus`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6220-6225: Function `nvmlDeviceGetPcieSpeed` / 函数 `nvmlDeviceGetPcieSpeed`
```python
def nvmlDeviceGetPcieSpeed(device):
    c_speed = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPcieSpeed")
    ret = fn(device, byref(c_speed))
    _nvmlCheckReturn(ret)
    return c_speed.value
```
**EN:** This function drives `nvmlDeviceGetPcieSpeed` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPcieSpeed`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6228-6239: Function `nvmlDeviceGetDynamicPstatesInfo` / 函数 `nvmlDeviceGetDynamicPstatesInfo`
```python
def nvmlDeviceGetDynamicPstatesInfo(
    device, c_dynamicpstatesinfo=c_nvmlGpuDynamicPstatesInfo_t()
):
    isReference = type(c_dynamicpstatesinfo) is not c_nvmlGpuDynamicPstatesInfo_t
    dynamicpstatesinfoRef = (
        c_dynamicpstatesinfo if isReference else byref(c_dynamicpstatesinfo)
    )

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDynamicPstatesInfo")
    ret = fn(device, dynamicpstatesinfoRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else c_dynamicpstatesinfo
```
**EN:** This function drives `nvmlDeviceGetDynamicPstatesInfo` with inputs such as `device`, `c_dynamicpstatesinfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDynamicPstatesInfo`，主要处理 `device`, `c_dynamicpstatesinfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6242-6246: Function `nvmlDeviceSetFanSpeed_v2` / 函数 `nvmlDeviceSetFanSpeed_v2`
```python
def nvmlDeviceSetFanSpeed_v2(handle, index, speed):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetFanSpeed_v2")
    ret = fn(handle, index, speed)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetFanSpeed_v2` with inputs such as `handle`, `index`, `speed`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetFanSpeed_v2`，主要处理 `handle`, `index`, `speed` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6249-6257: Function `nvmlDeviceGetThermalSettings` / 函数 `nvmlDeviceGetThermalSettings`
```python
def nvmlDeviceGetThermalSettings(
    device, sensorindex, c_thermalsettings=c_nvmlGpuThermalSettings_t()
):
    isReference = type(c_thermalsettings) is not c_nvmlGpuThermalSettings_t
    thermalsettingsRef = c_thermalsettings if isReference else byref(c_thermalsettings)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetThermalSettings")
    ret = fn(device, sensorindex, thermalsettingsRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else c_thermalsettings.sensor[:]
```
**EN:** This function drives `nvmlDeviceGetThermalSettings` with inputs such as `device`, `sensorindex`, `c_thermalsettings`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetThermalSettings`，主要处理 `device`, `sensorindex`, `c_thermalsettings` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6260-6275: Function `nvmlDeviceGetMinMaxClockOfPState` / 函数 `nvmlDeviceGetMinMaxClockOfPState`
```python
def nvmlDeviceGetMinMaxClockOfPState(
    device, clockType, pstate, minClockMHz=c_uint(), maxClockMHz=c_uint()
):
    isReference = (type(minClockMHz) is not c_uint) or (type(maxClockMHz) is not c_uint)
    minClockMHzRef = minClockMHz if isReference else byref(minClockMHz)
    maxClockMHzRef = maxClockMHz if isReference else byref(maxClockMHz)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMinMaxClockOfPState")
    ret = fn(
        device,
        _nvmlClockType_t(clockType),
        _nvmlClockType_t(pstate),
        minClockMHzRef,
        maxClockMHzRef,
    )
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else (minClockMHz.value, maxClockMHz.value)
```
**EN:** This function drives `nvmlDeviceGetMinMaxClockOfPState` with inputs such as `device`, `clockType`, `pstate`, `minClockMHz`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMinMaxClockOfPState`，主要处理 `device`, `clockType`, `pstate`, `minClockMHz` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6278-6286: Class `c_nvmlClockOffset_t` / 类 `c_nvmlClockOffset_t`
```python
class c_nvmlClockOffset_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("type", _nvmlClockType_t),
        ("pstate", _nvmlPstates_t),
        ("clockOffsetMHz", c_int),
        ("minClockOffsetMHz", c_int),
        ("maxClockOffsetMHz", c_int),
    ]
```
**EN:** This class models `c_nvmlClockOffset_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlClockOffset_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6287-6289: Top-level configuration / 顶层配置
```python


nvmlClockOffset_v1 = 0x1000018
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6292-6295: Function `nvmlDeviceGetClockOffsets` / 函数 `nvmlDeviceGetClockOffsets`
```python
def nvmlDeviceGetClockOffsets(device, info):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetClockOffsets")
    ret = fn(device, info)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetClockOffsets` with inputs such as `device`, `info`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetClockOffsets`，主要处理 `device`, `info` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6298-6301: Function `nvmlDeviceSetClockOffsets` / 函数 `nvmlDeviceSetClockOffsets`
```python
def nvmlDeviceSetClockOffsets(device, info):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetClockOffsets")
    ret = fn(device, info)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetClockOffsets` with inputs such as `device`, `info`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetClockOffsets`，主要处理 `device`, `info` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6304-6321: Function `nvmlDeviceGetSupportedPerformanceStates` / 函数 `nvmlDeviceGetSupportedPerformanceStates`
```python
def nvmlDeviceGetSupportedPerformanceStates(device):
    pstates = []
    c_count = c_uint(NVML_MAX_GPU_PERF_PSTATES)
    c_size = sizeof(c_uint) * c_count.value

    # NOTE: use 'c_uint' to represent the size of the nvmlPstate_t enumeration.
    pstates_array = _nvmlPstates_t * c_count.value
    c_pstates = pstates_array()

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSupportedPerformanceStates")
    ret = fn(device, c_pstates, c_size)
    _nvmlCheckReturn(ret)

    for value in c_pstates:
        if value != NVML_PSTATE_UNKNOWN:
            pstates.append(value)

    return pstates
```
**EN:** This function drives `nvmlDeviceGetSupportedPerformanceStates` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSupportedPerformanceStates`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6324-6329: Function `nvmlDeviceGetGpcClkVfOffset` / 函数 `nvmlDeviceGetGpcClkVfOffset`
```python
def nvmlDeviceGetGpcClkVfOffset(device):
    offset = c_int32()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpcClkVfOffset")
    ret = fn(device, byref(offset))
    _nvmlCheckReturn(ret)
    return offset.value
```
**EN:** This function drives `nvmlDeviceGetGpcClkVfOffset` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpcClkVfOffset`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6332-6337: Function `nvmlDeviceSetGpcClkVfOffset` / 函数 `nvmlDeviceSetGpcClkVfOffset`
```python
def nvmlDeviceSetGpcClkVfOffset(device, offset):
    c_offset = c_int32(offset)
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetGpcClkVfOffset")
    ret = fn(device, c_offset)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetGpcClkVfOffset` with inputs such as `device`, `offset`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetGpcClkVfOffset`，主要处理 `device`, `offset` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6340-6347: Function `nvmlDeviceGetGpcClkMinMaxVfOffset` / 函数 `nvmlDeviceGetGpcClkMinMaxVfOffset`
```python
def nvmlDeviceGetGpcClkMinMaxVfOffset(device, minOffset=c_int(), maxOffset=c_int()):
    isReference = (type(minOffset) is not c_int) or (type(maxOffset) is not c_int)
    minOffsetRef = minOffset if isReference else byref(minOffset)
    maxOffsetRef = maxOffset if isReference else byref(maxOffset)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpcClkMinMaxVfOffset")
    ret = fn(device, minOffsetRef, maxOffsetRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else (minOffset.value, maxOffset.value)
```
**EN:** This function drives `nvmlDeviceGetGpcClkMinMaxVfOffset` with inputs such as `device`, `minOffset`, `maxOffset`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpcClkMinMaxVfOffset`，主要处理 `device`, `minOffset`, `maxOffset` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6350-6355: Function `nvmlDeviceGetMemClkVfOffset` / 函数 `nvmlDeviceGetMemClkVfOffset`
```python
def nvmlDeviceGetMemClkVfOffset(device):
    offset = c_int32()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMemClkVfOffset")
    ret = fn(device, byref(offset))
    _nvmlCheckReturn(ret)
    return offset.value
```
**EN:** This function drives `nvmlDeviceGetMemClkVfOffset` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMemClkVfOffset`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6358-6363: Function `nvmlDeviceSetMemClkVfOffset` / 函数 `nvmlDeviceSetMemClkVfOffset`
```python
def nvmlDeviceSetMemClkVfOffset(device, offset):
    c_offset = c_int32(offset)
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetMemClkVfOffset")
    ret = fn(device, c_offset)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetMemClkVfOffset` with inputs such as `device`, `offset`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetMemClkVfOffset`，主要处理 `device`, `offset` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6366-6374: Function `nvmlDeviceGetMemClkMinMaxVfOffset` / 函数 `nvmlDeviceGetMemClkMinMaxVfOffset`
```python
def nvmlDeviceGetMemClkMinMaxVfOffset(device, minOffset=c_int(), maxOffset=c_int()):
    isReference = (type(minOffset) is not c_int) or (type(maxOffset) is not c_int)
    minOffsetRef = minOffset if isReference else byref(minOffset)
    maxOffsetRef = maxOffset if isReference else byref(maxOffset)

    fn = _nvmlGetFunctionPointer("nvmlDeviceGetMemClkMinMaxVfOffset")
    ret = fn(device, minOffsetRef, maxOffsetRef)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS if isReference else (minOffset.value, maxOffset.value)
```
**EN:** This function drives `nvmlDeviceGetMemClkMinMaxVfOffset` with inputs such as `device`, `minOffset`, `maxOffset`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetMemClkMinMaxVfOffset`，主要处理 `device`, `minOffset`, `maxOffset` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6377-6382: Function `nvmlSystemSetConfComputeGpusReadyState` / 函数 `nvmlSystemSetConfComputeGpusReadyState`
```python
def nvmlSystemSetConfComputeGpusReadyState(state):
    c_state = c_uint(state)
    fn = _nvmlGetFunctionPointer("nvmlSystemSetConfComputeGpusReadyState")
    ret = fn(c_state)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlSystemSetConfComputeGpusReadyState` with inputs such as `state`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemSetConfComputeGpusReadyState`，主要处理 `state` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6385-6390: Function `nvmlSystemGetConfComputeGpusReadyState` / 函数 `nvmlSystemGetConfComputeGpusReadyState`
```python
def nvmlSystemGetConfComputeGpusReadyState():
    c_state = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlSystemGetConfComputeGpusReadyState")
    ret = fn(byref(c_state))
    _nvmlCheckReturn(ret)
    return c_state.value
```
**EN:** This function drives `nvmlSystemGetConfComputeGpusReadyState`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetConfComputeGpusReadyState`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6393-6398: Function `nvmlSystemGetConfComputeCapabilities` / 函数 `nvmlSystemGetConfComputeCapabilities`
```python
def nvmlSystemGetConfComputeCapabilities():
    c_ccSysCaps = c_nvmlConfComputeSystemCaps_t()
    fn = _nvmlGetFunctionPointer("nvmlSystemGetConfComputeCapabilities")
    ret = fn(byref(c_ccSysCaps))
    _nvmlCheckReturn(ret)
    return c_ccSysCaps
```
**EN:** This function drives `nvmlSystemGetConfComputeCapabilities`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetConfComputeCapabilities`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6401-6406: Function `nvmlSystemGetConfComputeState` / 函数 `nvmlSystemGetConfComputeState`
```python
def nvmlSystemGetConfComputeState():
    c_state = c_nvmlConfComputeSystemState_t()
    fn = _nvmlGetFunctionPointer("nvmlSystemGetConfComputeState")
    ret = fn(byref(c_state))
    _nvmlCheckReturn(ret)
    return c_state
```
**EN:** This function drives `nvmlSystemGetConfComputeState`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetConfComputeState`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6409-6411: Function `nvmlSystemGetConfComputeSettings` / 函数 `nvmlSystemGetConfComputeSettings`
```python
def nvmlSystemGetConfComputeSettings(settings):
    fn = _nvmlGetFunctionPointer("nvmlSystemGetConfComputeSettings")
    return fn(settings)
```
**EN:** This function drives `nvmlSystemGetConfComputeSettings` with inputs such as `settings`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetConfComputeSettings`，主要处理 `settings` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6414-6418: Function `nvmlDeviceSetConfComputeUnprotectedMemSize` / 函数 `nvmlDeviceSetConfComputeUnprotectedMemSize`
```python
def nvmlDeviceSetConfComputeUnprotectedMemSize(device, c_ccMemSize):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetConfComputeUnprotectedMemSize")
    ret = fn(device, c_ccMemSize)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetConfComputeUnprotectedMemSize` with inputs such as `device`, `c_ccMemSize`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetConfComputeUnprotectedMemSize`，主要处理 `device`, `c_ccMemSize` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6421-6426: Function `nvmlDeviceGetConfComputeMemSizeInfo` / 函数 `nvmlDeviceGetConfComputeMemSizeInfo`
```python
def nvmlDeviceGetConfComputeMemSizeInfo(device):
    c_ccMemSize = c_nvmlConfComputeMemSizeInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetConfComputeMemSizeInfo")
    ret = fn(device, byref(c_ccMemSize))
    _nvmlCheckReturn(ret)
    return c_ccMemSize
```
**EN:** This function drives `nvmlDeviceGetConfComputeMemSizeInfo` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetConfComputeMemSizeInfo`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6429-6434: Function `nvmlDeviceGetConfComputeProtectedMemoryUsage` / 函数 `nvmlDeviceGetConfComputeProtectedMemoryUsage`
```python
def nvmlDeviceGetConfComputeProtectedMemoryUsage(device):
    c_memory = c_nvmlMemory_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetConfComputeProtectedMemoryUsage")
    ret = fn(device, byref(c_memory))
    _nvmlCheckReturn(ret)
    return c_memory
```
**EN:** This function drives `nvmlDeviceGetConfComputeProtectedMemoryUsage` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetConfComputeProtectedMemoryUsage`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6437-6442: Function `nvmlDeviceGetConfComputeGpuCertificate` / 函数 `nvmlDeviceGetConfComputeGpuCertificate`
```python
def nvmlDeviceGetConfComputeGpuCertificate(device):
    c_cert = c_nvmlConfComputeGpuCertificate_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetConfComputeGpuCertificate")
    ret = fn(device, byref(c_cert))
    _nvmlCheckReturn(ret)
    return c_cert
```
**EN:** This function drives `nvmlDeviceGetConfComputeGpuCertificate` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetConfComputeGpuCertificate`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6445-6452: Function `nvmlDeviceGetConfComputeGpuAttestationReport` / 函数 `nvmlDeviceGetConfComputeGpuAttestationReport`
```python
def nvmlDeviceGetConfComputeGpuAttestationReport(device, c_nonce):
    c_attestReport = c_nvmlConfComputeGpuAttestationReport_t()
    c_nonce_arr = (c_uint8 * len(c_nonce))(*(c_nonce))
    setattr(c_attestReport, "nonce", c_nonce_arr)
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetConfComputeGpuAttestationReport")
    ret = fn(device, byref(c_attestReport))
    _nvmlCheckReturn(ret)
    return c_attestReport
```
**EN:** This function drives `nvmlDeviceGetConfComputeGpuAttestationReport` with inputs such as `device`, `c_nonce`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetConfComputeGpuAttestationReport`，主要处理 `device`, `c_nonce` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6455-6462: Function `nvmlSystemSetConfComputeKeyRotationThresholdInfo` / 函数 `nvmlSystemSetConfComputeKeyRotationThresholdInfo`
```python
def nvmlSystemSetConfComputeKeyRotationThresholdInfo(max_atk_adv):
    c_keyRotationThrInfo = c_nvmlConfComputeSetKeyRotationThresholdInfo_t(0)
    c_keyRotationThrInfo.version = ConfComputeSetKeyRotationThresholdInfo_v1
    c_keyRotationThrInfo.maxAttackerAdvantage = max_atk_adv
    fn = _nvmlGetFunctionPointer("nvmlSystemSetConfComputeKeyRotationThresholdInfo")
    ret = fn(byref(c_keyRotationThrInfo))
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlSystemSetConfComputeKeyRotationThresholdInfo` with inputs such as `max_atk_adv`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemSetConfComputeKeyRotationThresholdInfo`，主要处理 `max_atk_adv` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6465-6471: Function `nvmlSystemGetConfComputeKeyRotationThresholdInfo` / 函数 `nvmlSystemGetConfComputeKeyRotationThresholdInfo`
```python
def nvmlSystemGetConfComputeKeyRotationThresholdInfo():
    c_keyRotationThrInfo = c_nvmlConfComputeGetKeyRotationThresholdInfo_t(0)
    c_keyRotationThrInfo.version = ConfComputeGetKeyRotationThresholdInfo_v1
    fn = _nvmlGetFunctionPointer("nvmlSystemGetConfComputeKeyRotationThresholdInfo")
    ret = fn(byref(c_keyRotationThrInfo))
    _nvmlCheckReturn(ret)
    return c_keyRotationThrInfo
```
**EN:** This function drives `nvmlSystemGetConfComputeKeyRotationThresholdInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetConfComputeKeyRotationThresholdInfo`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6472-6628: Top-level configuration / 顶层配置
```python


## GPM ##
#########

## Enums/defines

#### GPM Metric Identifiers
NVML_GPM_METRIC_GRAPHICS_UTIL = (
    1  # Percentage of time any compute/graphics app was active on the GPU. 0.0 - 100.0
)
NVML_GPM_METRIC_SM_UTIL = 2  # Percentage of SMs that were busy. 0.0 - 100.0
NVML_GPM_METRIC_SM_OCCUPANCY = (
    3  # Percentage of warps that were active vs theoretical maximum. 0.0 - 100.0
# ...
NVML_GPM_METRIC_NVLINK_L17_TX_PER_SEC = (
    97  # NvLink write bandwidth for link 17 in MiB/sec
)
NVML_GPM_METRIC_MAX = 98
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6633-6639: Class `c_nvmlUnitInfo_t` / 类 `c_nvmlUnitInfo_t`
```python
class c_nvmlUnitInfo_t(_PrintableStructure):
    _fields_ = [
        ("name", c_char * 96),
        ("id", c_char * 96),
        ("serial", c_char * 96),
        ("firmwareVersion", c_char * 96),
    ]
```
**EN:** This class models `c_nvmlUnitInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlUnitInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6642-6643: Class `struct_c_nvmlGpmSample_t` / 类 `struct_c_nvmlGpmSample_t`
```python
class struct_c_nvmlGpmSample_t(Structure):
    pass  # opaque handle
```
**EN:** This class models `struct_c_nvmlGpmSample_t` as a specialization of `Structure`.
**CN:** 该类实现 `struct_c_nvmlGpmSample_t`，并继承/扩展 `Structure`。

### Lines 6644-6646: Top-level configuration / 顶层配置
```python


c_nvmlGpmSample_t = POINTER(struct_c_nvmlGpmSample_t)
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6649-6654: Class `c_metricInfo_t` / 类 `c_metricInfo_t`
```python
class c_metricInfo_t(Structure):
    _fields_ = [
        ("shortName", c_char_p),
        ("longName", c_char_p),
        ("unit", c_char_p),
    ]
```
**EN:** This class models `c_metricInfo_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_metricInfo_t`，并继承/扩展 `Structure`。

### Lines 6657-6663: Class `c_nvmlGpmMetric_t` / 类 `c_nvmlGpmMetric_t`
```python
class c_nvmlGpmMetric_t(_PrintableStructure):
    _fields_ = [
        ("metricId", c_uint),
        ("nvmlReturn", _nvmlReturn_t),
        ("value", c_double),
        ("metricInfo", c_metricInfo_t),
    ]
```
**EN:** This class models `c_nvmlGpmMetric_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGpmMetric_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6666-6673: Class `c_nvmlGpmMetricsGet_t` / 类 `c_nvmlGpmMetricsGet_t`
```python
class c_nvmlGpmMetricsGet_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("numMetrics", c_uint),
        ("sample1", c_nvmlGpmSample_t),
        ("sample2", c_nvmlGpmSample_t),
        ("metrics", c_nvmlGpmMetric_t * NVML_GPM_METRIC_MAX),
    ]
```
**EN:** This class models `c_nvmlGpmMetricsGet_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGpmMetricsGet_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6674-6676: Top-level configuration / 顶层配置
```python


NVML_GPM_METRICS_GET_VERSION = 1
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6679-6683: Class `c_nvmlGpmSupport_t` / 类 `c_nvmlGpmSupport_t`
```python
class c_nvmlGpmSupport_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("isSupportedDevice", c_uint),
    ]
```
**EN:** This class models `c_nvmlGpmSupport_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGpmSupport_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6684-6686: Top-level configuration / 顶层配置
```python


NVML_GPM_SUPPORT_VERSION = 1
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6691-6695: Function `nvmlGpmMetricsGet` / 函数 `nvmlGpmMetricsGet`
```python
def nvmlGpmMetricsGet(metricsGet):
    fn = _nvmlGetFunctionPointer("nvmlGpmMetricsGet")
    ret = fn(byref(metricsGet))
    _nvmlCheckReturn(ret)
    return metricsGet
```
**EN:** This function drives `nvmlGpmMetricsGet` with inputs such as `metricsGet`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmMetricsGet`，主要处理 `metricsGet` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6698-6702: Function `nvmlGpmSampleFree` / 函数 `nvmlGpmSampleFree`
```python
def nvmlGpmSampleFree(gpmSample):
    fn = _nvmlGetFunctionPointer("nvmlGpmSampleFree")
    ret = fn(gpmSample)
    _nvmlCheckReturn(ret)
    return
```
**EN:** This function drives `nvmlGpmSampleFree` with inputs such as `gpmSample`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmSampleFree`，主要处理 `gpmSample` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6705-6710: Function `nvmlGpmSampleAlloc` / 函数 `nvmlGpmSampleAlloc`
```python
def nvmlGpmSampleAlloc():
    gpmSample = c_nvmlGpmSample_t()
    fn = _nvmlGetFunctionPointer("nvmlGpmSampleAlloc")
    ret = fn(byref(gpmSample))
    _nvmlCheckReturn(ret)
    return gpmSample
```
**EN:** This function drives `nvmlGpmSampleAlloc`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmSampleAlloc`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6713-6717: Function `nvmlGpmSampleGet` / 函数 `nvmlGpmSampleGet`
```python
def nvmlGpmSampleGet(device, gpmSample):
    fn = _nvmlGetFunctionPointer("nvmlGpmSampleGet")
    ret = fn(device, gpmSample)
    _nvmlCheckReturn(ret)
    return gpmSample
```
**EN:** This function drives `nvmlGpmSampleGet` with inputs such as `device`, `gpmSample`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmSampleGet`，主要处理 `device`, `gpmSample` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6720-6724: Function `nvmlGpmMigSampleGet` / 函数 `nvmlGpmMigSampleGet`
```python
def nvmlGpmMigSampleGet(device, gpuInstanceId, gpmSample):
    fn = _nvmlGetFunctionPointer("nvmlGpmMigSampleGet")
    ret = fn(device, gpuInstanceId, gpmSample)
    _nvmlCheckReturn(ret)
    return gpmSample
```
**EN:** This function drives `nvmlGpmMigSampleGet` with inputs such as `device`, `gpuInstanceId`, `gpmSample`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmMigSampleGet`，主要处理 `device`, `gpuInstanceId`, `gpmSample` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6727-6733: Function `nvmlGpmQueryDeviceSupport` / 函数 `nvmlGpmQueryDeviceSupport`
```python
def nvmlGpmQueryDeviceSupport(device):
    gpmSupport = c_nvmlGpmSupport_t()
    gpmSupport.version = NVML_GPM_SUPPORT_VERSION
    fn = _nvmlGetFunctionPointer("nvmlGpmQueryDeviceSupport")
    ret = fn(device, byref(gpmSupport))
    _nvmlCheckReturn(ret)
    return gpmSupport
```
**EN:** This function drives `nvmlGpmQueryDeviceSupport` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmQueryDeviceSupport`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6736-6741: Function `nvmlGpmSetStreamingEnabled` / 函数 `nvmlGpmSetStreamingEnabled`
```python
def nvmlGpmSetStreamingEnabled(device, state):
    c_state = c_uint(state)
    fn = _nvmlGetFunctionPointer("nvmlGpmSetStreamingEnabled")
    ret = fn(device, c_state)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlGpmSetStreamingEnabled` with inputs such as `device`, `state`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmSetStreamingEnabled`，主要处理 `device`, `state` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6744-6749: Function `nvmlGpmQueryIfStreamingEnabled` / 函数 `nvmlGpmQueryIfStreamingEnabled`
```python
def nvmlGpmQueryIfStreamingEnabled(device):
    c_state = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlGpmQueryIfStreamingEnabled")
    ret = fn(device, byref(c_state))
    _nvmlCheckReturn(ret)
    return c_state.value
```
**EN:** This function drives `nvmlGpmQueryIfStreamingEnabled` with inputs such as `device`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlGpmQueryIfStreamingEnabled`，主要处理 `device` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6750-6760: Top-level configuration / 顶层配置
```python


# Low Power Structure and Function

NVML_NVLINK_POWER_STATE_HIGH_SPEED = 0x0
NVML_NVLINK_POWER_STATE_LOW = 0x1

NVML_NVLINK_LOW_POWER_THRESHOLD_MIN = 0x1
NVML_NVLINK_LOW_POWER_THRESHOLD_MAX = 0x1FFF
NVML_NVLINK_LOW_POWER_THRESHOLD_RESET = 0xFFFFFFFF
NVML_NVLINK_LOW_POWER_THRESHOLD_DEFAULT = NVML_NVLINK_LOW_POWER_THRESHOLD_RESET
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6763-6766: Class `c_nvmlNvLinkPowerThres_t` / 类 `c_nvmlNvLinkPowerThres_t`
```python
class c_nvmlNvLinkPowerThres_t(Structure):
    _fields_ = [
        ("lowPwrThreshold", c_uint),
    ]
```
**EN:** This class models `c_nvmlNvLinkPowerThres_t` as a specialization of `Structure`.
**CN:** 该类实现 `c_nvmlNvLinkPowerThres_t`，并继承/扩展 `Structure`。

### Lines 6769-6775: Function `nvmlDeviceSetNvLinkDeviceLowPowerThreshold` / 函数 `nvmlDeviceSetNvLinkDeviceLowPowerThreshold`
```python
def nvmlDeviceSetNvLinkDeviceLowPowerThreshold(device, l1threshold):
    c_info = c_nvmlNvLinkPowerThres_t()
    c_info.lowPwrThreshold = l1threshold
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetNvLinkDeviceLowPowerThreshold")
    ret = fn(device, byref(c_info))
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetNvLinkDeviceLowPowerThreshold` with inputs such as `device`, `l1threshold`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetNvLinkDeviceLowPowerThreshold`，主要处理 `device`, `l1threshold` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6776-6784: Top-level configuration / 顶层配置
```python


NVML_GPU_FABRIC_UUID_LEN = 16

_nvmlGpuFabricState_t = c_uint
NVML_GPU_FABRIC_STATE_NOT_SUPPORTED = 0
NVML_GPU_FABRIC_STATE_NOT_STARTED = 1
NVML_GPU_FABRIC_STATE_IN_PROGRESS = 2
NVML_GPU_FABRIC_STATE_COMPLETED = 3
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6787-6793: Class `c_nvmlGpuFabricInfo_t` / 类 `c_nvmlGpuFabricInfo_t`
```python
class c_nvmlGpuFabricInfo_t(_PrintableStructure):
    _fields_ = [
        ("clusterUuid", c_char * NVML_DEVICE_UUID_BUFFER_SIZE),
        ("status", _nvmlReturn_t),
        ("cliqueId", c_uint32),
        ("state", _nvmlGpuFabricState_t),
    ]
```
**EN:** This class models `c_nvmlGpuFabricInfo_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlGpuFabricInfo_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6794-6820: Top-level configuration / 顶层配置
```python


NVML_GPU_FABRIC_HEALTH_MASK_DEGRADED_BW_NOT_SUPPORTED = 0
NVML_GPU_FABRIC_HEALTH_MASK_DEGRADED_BW_TRUE = 1
NVML_GPU_FABRIC_HEALTH_MASK_DEGRADED_BW_FALSE = 2
NVML_GPU_FABRIC_HEALTH_MASK_SHIFT_DEGRADED_BW = 0
NVML_GPU_FABRIC_HEALTH_MASK_WIDTH_DEGRADED_BW = 0x11

NVML_GPU_FABRIC_HEALTH_MASK_ROUTE_RECOVERY_NOT_SUPPORTED = 0
NVML_GPU_FABRIC_HEALTH_MASK_ROUTE_RECOVERY_TRUE = 1
NVML_GPU_FABRIC_HEALTH_MASK_ROUTE_RECOVERY_FALSE = 2
NVML_GPU_FABRIC_HEALTH_MASK_SHIFT_ROUTE_RECOVERY = 2
NVML_GPU_FABRIC_HEALTH_MASK_WIDTH_ROUTE_RECOVERY = 0x11

# ...
NVML_GPU_FABRIC_HEALTH_MASK_SHIFT_ACCESS_TIMEOUT_RECOVERY = 6
NVML_GPU_FABRIC_HEALTH_MASK_WIDTH_ACCESS_TIMEOUT_RECOVERY = 0x11

nvmlGpuFabricInfo_v2 = 0x02000024
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6823-6834: Class `c_nvmlGpuFabricInfoV_t` / 类 `c_nvmlGpuFabricInfoV_t`
```python
class c_nvmlGpuFabricInfoV_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("clusterUuid", c_char * NVML_GPU_FABRIC_UUID_LEN),
        ("status", _nvmlReturn_t),
        ("cliqueId", c_uint32),
        ("state", _nvmlGpuFabricState_t),
        ("healthMask", c_uint32),
    ]

    def __init__(self):
        super(c_nvmlGpuFabricInfoV_t, self).__init__(version=nvmlGpuFabricInfo_v2)
```
**EN:** This class models `c_nvmlGpuFabricInfoV_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlGpuFabricInfoV_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 6837-6841: Function `nvmlDeviceGetGpuFabricInfo` / 函数 `nvmlDeviceGetGpuFabricInfo`
```python
def nvmlDeviceGetGpuFabricInfo(device, gpuFabricInfo):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuFabricInfo")
    ret = fn(device, gpuFabricInfo)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetGpuFabricInfo` with inputs such as `device`, `gpuFabricInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuFabricInfo`，主要处理 `device`, `gpuFabricInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6844-6848: Function `nvmlDeviceGetGpuFabricInfoV` / 函数 `nvmlDeviceGetGpuFabricInfoV`
```python
def nvmlDeviceGetGpuFabricInfoV(device, gpuFabricInfo):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetGpuFabricInfoV")
    ret = fn(device, gpuFabricInfo)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetGpuFabricInfoV` with inputs such as `device`, `gpuFabricInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetGpuFabricInfoV`，主要处理 `device`, `gpuFabricInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6849-6859: Top-level configuration / 顶层配置
```python


######################
## Enums/defines
#### NVML GPU NVLINK BW MODE
NVML_GPU_NVLINK_BW_MODE_FULL = 0x0
NVML_GPU_NVLINK_BW_MODE_OFF = 0x1
NVML_GPU_NVLINK_BW_MODE_MIN = 0x2
NVML_GPU_NVLINK_BW_MODE_HALF = 0x3
NVML_GPU_NVLINK_BW_MODE_3QUARTER = 0x4
NVML_GPU_NVLINK_BW_MODE_COUNT = 0x5
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6862-6866: Function `nvmlSystemSetNvlinkBwMode` / 函数 `nvmlSystemSetNvlinkBwMode`
```python
def nvmlSystemSetNvlinkBwMode(mode):
    fn = _nvmlGetFunctionPointer("nvmlSystemSetNvlinkBwMode")
    ret = fn(mode)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlSystemSetNvlinkBwMode` with inputs such as `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemSetNvlinkBwMode`，主要处理 `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6869-6874: Function `nvmlSystemGetNvlinkBwMode` / 函数 `nvmlSystemGetNvlinkBwMode`
```python
def nvmlSystemGetNvlinkBwMode():
    mode = c_uint()
    fn = _nvmlGetFunctionPointer("nvmlSystemGetNvlinkBwMode")
    ret = fn(byref(mode))
    _nvmlCheckReturn(ret)
    return mode.value
```
**EN:** This function drives `nvmlSystemGetNvlinkBwMode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlSystemGetNvlinkBwMode`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6875-6880: Top-level configuration / 顶层配置
```python


_nvmlPowerScopeType_t = c_uint
NVML_POWER_SCOPE_GPU = 0
NVML_POWER_SCOPE_MODULE = 1
NVML_POWER_SCOPE_MEMORY = 2
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6883-6889: Class `c_nvmlPowerValue_v2_t` / 类 `c_nvmlPowerValue_v2_t`
```python
class c_nvmlPowerValue_v2_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("powerScope", _nvmlPowerScopeType_t),
        ("powerValueMw", c_uint),
    ]
    _fmt_ = {"<default>": "%d B"}
```
**EN:** This class models `c_nvmlPowerValue_v2_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlPowerValue_v2_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6890-6892: Top-level configuration / 顶层配置
```python


nvmlPowerValue_v2 = 0x0200000C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6895-6905: Function `nvmlDeviceSetPowerManagementLimit_v2` / 函数 `nvmlDeviceSetPowerManagementLimit_v2`
```python
def nvmlDeviceSetPowerManagementLimit_v2(
    device, powerScope, powerLimit, version=nvmlPowerValue_v2
):
    c_powerScope = _nvmlPowerScopeType_t(powerScope)
    c_powerValue = c_nvmlPowerValue_v2_t()
    c_powerValue.version = c_uint(version)
    c_powerValue.powerScope = c_powerScope
    c_powerValue.powerValueMw = c_uint(powerLimit)
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetPowerManagementLimit_v2")
    ret = fn(device, byref(c_powerValue))
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetPowerManagementLimit_v2` with inputs such as `device`, `powerScope`, `powerLimit`, `version`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetPowerManagementLimit_v2`，主要处理 `device`, `powerScope`, `powerLimit`, `version` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6908-6928: Class `c_nvmlEccSramErrorStatus_v1_t` / 类 `c_nvmlEccSramErrorStatus_v1_t`
```python
class c_nvmlEccSramErrorStatus_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("aggregateUncParity", c_ulonglong),
        ("aggregateUncSecDed", c_ulonglong),
        ("aggregateCor", c_ulonglong),
        ("volatileUncParity", c_ulonglong),
        ("volatileUncSecDed", c_ulonglong),
        ("volatileCor", c_ulonglong),
        ("aggregateUncBucketL2", c_ulonglong),
        ("aggregateUncBucketSm", c_ulonglong),
        ("aggregateUncBucketPcie", c_ulonglong),
        ("aggregateUncBucketMcu", c_ulonglong),
        ("aggregateUncBucketOther", c_ulonglong),
# ...
    def __init__(self):
        super(c_nvmlEccSramErrorStatus_v1_t, self).__init__(
            version=nvmlEccSramErrorStatus_v1
        )
```
**EN:** This class models `c_nvmlEccSramErrorStatus_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlEccSramErrorStatus_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 6929-6931: Top-level configuration / 顶层配置
```python


nvmlEccSramErrorStatus_v1 = 0x1000068
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6934-6938: Function `nvmlDeviceGetSramEccErrorStatus` / 函数 `nvmlDeviceGetSramEccErrorStatus`
```python
def nvmlDeviceGetSramEccErrorStatus(device, status):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetSramEccErrorStatus")
    ret = fn(device, status)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetSramEccErrorStatus` with inputs such as `device`, `status`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetSramEccErrorStatus`，主要处理 `device`, `status` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6939-6942: Top-level configuration / 顶层配置
```python


NVML_DEV_CAP_EGM = 1 << 0
nvmlDeviceCapabilities_v1 = 0x1000008
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6945-6954: Class `c_nvmlDeviceCapabilities_v1_t` / 类 `c_nvmlDeviceCapabilities_v1_t`
```python
class c_nvmlDeviceCapabilities_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("capMask", c_uint),
    ]

    def __init__(self):
        super(c_nvmlDeviceCapabilities_v1_t, self).__init__(
            version=nvmlDeviceCapabilities_v1
        )
```
**EN:** This class models `c_nvmlDeviceCapabilities_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlDeviceCapabilities_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 6957-6959: Function `nvmlDeviceGetCapabilities` / 函数 `nvmlDeviceGetCapabilities`
```python
def nvmlDeviceGetCapabilities(device, caps):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetCapabilities")
    return fn(device, caps)
```
**EN:** This function drives `nvmlDeviceGetCapabilities` with inputs such as `device`, `caps`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCapabilities`，主要处理 `device`, `caps` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6962-6975: Class `c_nvmlPlatformInfo_v1_t` / 类 `c_nvmlPlatformInfo_v1_t`
```python
class c_nvmlPlatformInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("ibGuid", c_char * 16),
        ("rackGuid", c_char * 16),
        ("chassisPhysicalSlotNumber", c_char),
        ("computeSlotIndex", c_char),
        ("nodeIndex", c_char),
        ("peerType", c_char),
        ("moduleId", c_char),
    ]

    def __init__(self):
        super(c_nvmlPlatformInfo_v1_t, self).__init__(version=nvmlPlatformInfo_v1)
```
**EN:** This class models `c_nvmlPlatformInfo_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlPlatformInfo_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 6976-6978: Top-level configuration / 顶层配置
```python


nvmlPlatformInfo_v1 = 0x100002C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 6981-6985: Function `nvmlDeviceGetPlatformInfo` / 函数 `nvmlDeviceGetPlatformInfo`
```python
def nvmlDeviceGetPlatformInfo(device, platformInfo):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetPlatformInfo")
    ret = fn(device, platformInfo)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetPlatformInfo` with inputs such as `device`, `platformInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPlatformInfo`，主要处理 `device`, `platformInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 6988-6991: Class `c_nvmlMask255_t` / 类 `c_nvmlMask255_t`
```python
class c_nvmlMask255_t(_PrintableStructure):
    _fields_ = [
        ("mask", c_uint * 8),
    ]
```
**EN:** This class models `c_nvmlMask255_t` as a specialization of `_PrintableStructure`.
**CN:** 该类实现 `c_nvmlMask255_t`，并继承/扩展 `_PrintableStructure`。

### Lines 6992-7012: Top-level configuration / 顶层配置
```python


NVML_WORKLOAD_POWER_MAX_PROFILES = 255
NVML_POWER_PROFILE_MAX_P = 0
NVML_POWER_PROFILE_MAX_Q = 1
NVML_POWER_PROFILE_COMPUTE = 2
NVML_POWER_PROFILE_MEMORY_BOUND = 3
NVML_POWER_PROFILE_NETWORK = 4
NVML_POWER_PROFILE_BALANCED = 5
NVML_POWER_PROFILE_LLM_INFERENCE = 6
NVML_POWER_PROFILE_LLM_TRAINING = 7
NVML_POWER_PROFILE_RBM = 8
NVML_POWER_PROFILE_DCPCIE = 9
NVML_POWER_PROFILE_HMMA_SPARSE = 10
# ...
NVML_POWER_PROFILE_MIG = 14
NVML_POWER_PROFILE_MAX = 15

nvmlWorkloadPowerProfileInfo_v1 = 0x100002C
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 7015-7026: Class `c_nvmlWorkloadPowerProfileInfo_v1_t` / 类 `c_nvmlWorkloadPowerProfileInfo_v1_t`
```python
class c_nvmlWorkloadPowerProfileInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("profileId", c_uint),
        ("priority", c_uint),
        ("conflictingmask", c_nvmlMask255_t),
    ]

    def __init__(self):
        super(c_nvmlWorkloadPowerProfileInfo_v1_t, self).__init__(
            version=nvmlWorkloadPowerProfileInfo_v1
        )
```
**EN:** This class models `c_nvmlWorkloadPowerProfileInfo_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlWorkloadPowerProfileInfo_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 7027-7029: Top-level configuration / 顶层配置
```python


nvmlWorkloadPowerProfileProfilesInfo_v1 = 0x1002BF8
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 7032-7045: Class `c_nvmlWorkloadPowerProfileProfilesInfo_v1_t` / 类 `c_nvmlWorkloadPowerProfileProfilesInfo_v1_t`
```python
class c_nvmlWorkloadPowerProfileProfilesInfo_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("perfProfilesMask", c_nvmlMask255_t),
        (
            "perfProfile",
            c_nvmlWorkloadPowerProfileInfo_v1_t * NVML_WORKLOAD_POWER_MAX_PROFILES,
        ),
    ]

    def __init__(self):
        super(c_nvmlWorkloadPowerProfileProfilesInfo_v1_t, self).__init__(
            version=nvmlWorkloadPowerProfileProfilesInfo_v1
        )
```
**EN:** This class models `c_nvmlWorkloadPowerProfileProfilesInfo_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlWorkloadPowerProfileProfilesInfo_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 7046-7048: Top-level configuration / 顶层配置
```python


nvmlWorkloadPowerProfileCurrentProfiles_v1 = 0x1000064
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 7051-7062: Class `c_nvmlWorkloadPowerProfileCurrentProfiles_v1_t` / 类 `c_nvmlWorkloadPowerProfileCurrentProfiles_v1_t`
```python
class c_nvmlWorkloadPowerProfileCurrentProfiles_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("perfProfilesMask", c_nvmlMask255_t),
        ("requestedProfilesMask", c_nvmlMask255_t),
        ("enforcedProfilesMask", c_nvmlMask255_t),
    ]

    def __init__(self):
        super(c_nvmlWorkloadPowerProfileCurrentProfiles_v1_t, self).__init__(
            version=nvmlWorkloadPowerProfileCurrentProfiles_v1
        )
```
**EN:** This class models `c_nvmlWorkloadPowerProfileCurrentProfiles_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlWorkloadPowerProfileCurrentProfiles_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 7063-7065: Top-level configuration / 顶层配置
```python


nvmlWorkloadPowerProfileRequestedProfiles_v1 = 0x1000024
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 7068-7077: Class `c_nvmlWorkloadPowerProfileRequestedProfiles_v1_t` / 类 `c_nvmlWorkloadPowerProfileRequestedProfiles_v1_t`
```python
class c_nvmlWorkloadPowerProfileRequestedProfiles_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("requestedProfilesMask", c_nvmlMask255_t),
    ]

    def __init__(self):
        super(c_nvmlWorkloadPowerProfileRequestedProfiles_v1_t, self).__init__(
            version=nvmlWorkloadPowerProfileRequestedProfiles_v1
        )
```
**EN:** This class models `c_nvmlWorkloadPowerProfileRequestedProfiles_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlWorkloadPowerProfileRequestedProfiles_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 7080-7084: Function `nvmlDeviceWorkloadPowerProfileGetProfilesInfo` / 函数 `nvmlDeviceWorkloadPowerProfileGetProfilesInfo`
```python
def nvmlDeviceWorkloadPowerProfileGetProfilesInfo(device, profilesInfo):
    fn = _nvmlGetFunctionPointer("nvmlDeviceWorkloadPowerProfileGetProfilesInfo")
    ret = fn(device, profilesInfo)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceWorkloadPowerProfileGetProfilesInfo` with inputs such as `device`, `profilesInfo`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceWorkloadPowerProfileGetProfilesInfo`，主要处理 `device`, `profilesInfo` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7087-7091: Function `nvmlDeviceWorkloadPowerProfileGetCurrentProfiles` / 函数 `nvmlDeviceWorkloadPowerProfileGetCurrentProfiles`
```python
def nvmlDeviceWorkloadPowerProfileGetCurrentProfiles(device, currentProfiles):
    fn = _nvmlGetFunctionPointer("nvmlDeviceWorkloadPowerProfileGetCurrentProfiles")
    ret = fn(device, currentProfiles)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceWorkloadPowerProfileGetCurrentProfiles` with inputs such as `device`, `currentProfiles`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceWorkloadPowerProfileGetCurrentProfiles`，主要处理 `device`, `currentProfiles` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7094-7098: Function `nvmlDeviceWorkloadPowerProfileSetRequestedProfiles` / 函数 `nvmlDeviceWorkloadPowerProfileSetRequestedProfiles`
```python
def nvmlDeviceWorkloadPowerProfileSetRequestedProfiles(device, requestedProfiles):
    fn = _nvmlGetFunctionPointer("nvmlDeviceWorkloadPowerProfileSetRequestedProfiles")
    ret = fn(device, requestedProfiles)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceWorkloadPowerProfileSetRequestedProfiles` with inputs such as `device`, `requestedProfiles`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceWorkloadPowerProfileSetRequestedProfiles`，主要处理 `device`, `requestedProfiles` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7101-7105: Function `nvmlDeviceWorkloadPowerProfileClearRequestedProfiles` / 函数 `nvmlDeviceWorkloadPowerProfileClearRequestedProfiles`
```python
def nvmlDeviceWorkloadPowerProfileClearRequestedProfiles(device, requestedProfiles):
    fn = _nvmlGetFunctionPointer("nvmlDeviceWorkloadPowerProfileClearRequestedProfiles")
    ret = fn(device, requestedProfiles)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceWorkloadPowerProfileClearRequestedProfiles` with inputs such as `device`, `requestedProfiles`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceWorkloadPowerProfileClearRequestedProfiles`，主要处理 `device`, `requestedProfiles` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7108-7112: Function `nvmlDeviceGetNvlinkSupportedBwModes` / 函数 `nvmlDeviceGetNvlinkSupportedBwModes`
```python
def nvmlDeviceGetNvlinkSupportedBwModes(device, supportedBwModes):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvlinkSupportedBwModes")
    ret = fn(device, supportedBwModes)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetNvlinkSupportedBwModes` with inputs such as `device`, `supportedBwModes`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvlinkSupportedBwModes`，主要处理 `device`, `supportedBwModes` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7115-7119: Function `nvmlDeviceGetNvlinkBwMode` / 函数 `nvmlDeviceGetNvlinkBwMode`
```python
def nvmlDeviceGetNvlinkBwMode(device, getBwMode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetNvlinkBwMode")
    ret = fn(device, getBwMode)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceGetNvlinkBwMode` with inputs such as `device`, `getBwMode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetNvlinkBwMode`，主要处理 `device`, `getBwMode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7122-7126: Function `nvmlDeviceSetNvlinkBwMode` / 函数 `nvmlDeviceSetNvlinkBwMode`
```python
def nvmlDeviceSetNvlinkBwMode(device, setBwMode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetNvlinkBwMode")
    ret = fn(device, setBwMode)
    _nvmlCheckReturn(ret)
    return NVML_SUCCESS
```
**EN:** This function drives `nvmlDeviceSetNvlinkBwMode` with inputs such as `device`, `setBwMode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetNvlinkBwMode`，主要处理 `device`, `setBwMode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7127-7129: Top-level configuration / 顶层配置
```python


nvmlDramEncryptionInfo_v1 = 0x01000008
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 7132-7141: Class `c_nvmlDramEncryptionInfo_t` / 类 `c_nvmlDramEncryptionInfo_t`
```python
class c_nvmlDramEncryptionInfo_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("encryptionState", _nvmlEnableState_t),
    ]

    def __init__(self):
        super(c_nvmlDramEncryptionInfo_t, self).__init__(
            version=nvmlDramEncryptionInfo_v1
        )
```
**EN:** This class models `c_nvmlDramEncryptionInfo_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlDramEncryptionInfo_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 7144-7150: Function `nvmlDeviceGetDramEncryptionMode` / 函数 `nvmlDeviceGetDramEncryptionMode`
```python
def nvmlDeviceGetDramEncryptionMode(handle):
    c_currState = c_nvmlDramEncryptionInfo_t()
    c_pendingState = c_nvmlDramEncryptionInfo_t()
    fn = _nvmlGetFunctionPointer("nvmlDeviceGetDramEncryptionMode")
    ret = fn(handle, byref(c_currState), byref(c_pendingState))
    _nvmlCheckReturn(ret)
    return [c_currState.encryptionState, c_pendingState.encryptionState]
```
**EN:** This function drives `nvmlDeviceGetDramEncryptionMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetDramEncryptionMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7154-7155: Function `nvmlDeviceGetCurrentDramEncryptionMode` / 函数 `nvmlDeviceGetCurrentDramEncryptionMode`
```python
def nvmlDeviceGetCurrentDramEncryptionMode(handle):
    return nvmlDeviceGetDramEncryptionMode(handle)[0]
```
**EN:** This function drives `nvmlDeviceGetCurrentDramEncryptionMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetCurrentDramEncryptionMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7159-7160: Function `nvmlDeviceGetPendingDramEncryptionMode` / 函数 `nvmlDeviceGetPendingDramEncryptionMode`
```python
def nvmlDeviceGetPendingDramEncryptionMode(handle):
    return nvmlDeviceGetDramEncryptionMode(handle)[1]
```
**EN:** This function drives `nvmlDeviceGetPendingDramEncryptionMode` with inputs such as `handle`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceGetPendingDramEncryptionMode`，主要处理 `handle` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7163-7169: Function `nvmlDeviceSetDramEncryptionMode` / 函数 `nvmlDeviceSetDramEncryptionMode`
```python
def nvmlDeviceSetDramEncryptionMode(handle, mode):
    fn = _nvmlGetFunctionPointer("nvmlDeviceSetDramEncryptionMode")
    c_dramEncryptionMode = c_nvmlDramEncryptionInfo_t()
    c_dramEncryptionMode.encryptionState = mode
    ret = fn(handle, byref(c_dramEncryptionMode))
    _nvmlCheckReturn(ret)
    return None
```
**EN:** This function drives `nvmlDeviceSetDramEncryptionMode` with inputs such as `handle`, `mode`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDeviceSetDramEncryptionMode`，主要处理 `handle`, `mode` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7170-7180: Top-level configuration / 顶层配置
```python


# Power Smoothing defines
NVML_POWER_SMOOTHING_MAX_NUM_PROFILES = 5
NVML_POWER_SMOOTHING_ADMIN_OVERRIDE_NOT_SET = 0xFFFFFFFF
NVML_POWER_SMOOTHING_PROFILE_PARAM_PERCENT_TMP_FLOOR = 0
NVML_POWER_SMOOTHING_PROFILE_PARAM_RAMP_UP_RATE = 1
NVML_POWER_SMOOTHING_PROFILE_PARAM_RAMP_DOWN_RATE = 2
NVML_POWER_SMOOTHING_PROFILE_PARAM_RAMP_DOWN_HYSTERESIS = 3

nvmlPowerSmoothingState_v1 = 0x1000008
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 7183-7192: Class `c_nvmlPowerSmoothingState_v1_t` / 类 `c_nvmlPowerSmoothingState_v1_t`
```python
class c_nvmlPowerSmoothingState_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("state", c_uint),
    ]

    def __init__(self):
        super(c_nvmlPowerSmoothingState_v1_t, self).__init__(
            version=nvmlPowerSmoothingState_v1
        )
```
**EN:** This class models `c_nvmlPowerSmoothingState_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlPowerSmoothingState_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 7193-7195: Top-level configuration / 顶层配置
```python


nvmlPowerSmoothingProfile_v1 = 0x1000018
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 7198-7209: Class `c_nvmlPowerSmoothingProfile_v1_t` / 类 `c_nvmlPowerSmoothingProfile_v1_t`
```python
class c_nvmlPowerSmoothingProfile_v1_t(_PrintableStructure):
    _fields_ = [
        ("version", c_uint),
        ("profileId", c_uint),
        ("paramId", c_uint),
        ("value", c_double),
    ]

    def __init__(self):
        super(c_nvmlPowerSmoothingProfile_v1_t, self).__init__(
            version=nvmlPowerSmoothingProfile_v1
        )
```
**EN:** This class models `c_nvmlPowerSmoothingProfile_v1_t` as a specialization of `_PrintableStructure`. Important methods include `__init__`.
**CN:** 该类实现 `c_nvmlPowerSmoothingProfile_v1_t`，并继承/扩展 `_PrintableStructure`。 其中较重要的方法包括 `__init__`。

### Lines 7212-7215: Function `nvmlDevicePowerSmoothingActivatePresetProfile` / 函数 `nvmlDevicePowerSmoothingActivatePresetProfile`
```python
def nvmlDevicePowerSmoothingActivatePresetProfile(device, profile):
    fn = _nvmlGetFunctionPointer("nvmlDevicePowerSmoothingActivatePresetProfile")
    ret = fn(device, profile)
    _nvmlCheckReturn(ret)
```
**EN:** This function drives `nvmlDevicePowerSmoothingActivatePresetProfile` with inputs such as `device`, `profile`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDevicePowerSmoothingActivatePresetProfile`，主要处理 `device`, `profile` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7218-7221: Function `nvmlDevicePowerSmoothingUpdatePresetProfileParam` / 函数 `nvmlDevicePowerSmoothingUpdatePresetProfileParam`
```python
def nvmlDevicePowerSmoothingUpdatePresetProfileParam(device, profile):
    fn = _nvmlGetFunctionPointer("nvmlDevicePowerSmoothingUpdatePresetProfileParam")
    ret = fn(device, profile)
    _nvmlCheckReturn(ret)
```
**EN:** This function drives `nvmlDevicePowerSmoothingUpdatePresetProfileParam` with inputs such as `device`, `profile`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDevicePowerSmoothingUpdatePresetProfileParam`，主要处理 `device`, `profile` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 7224-7227: Function `nvmlDevicePowerSmoothingSetState` / 函数 `nvmlDevicePowerSmoothingSetState`
```python
def nvmlDevicePowerSmoothingSetState(device, state):
    fn = _nvmlGetFunctionPointer("nvmlDevicePowerSmoothingSetState")
    ret = fn(device, state)
    _nvmlCheckReturn(ret)
```
**EN:** This function drives `nvmlDevicePowerSmoothingSetState` with inputs such as `device`, `state`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `nvmlDevicePowerSmoothingSetState`，主要处理 `device`, `state` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- Scheduling and batching / 调度与批处理
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Command-line interface / 命令行接口

## Dependencies / 依赖关系
- **External / 外部**: `ctypes`
- **Stdlib / 标准库**: `os`, `string`, `sys`, `threading`, `functools`
