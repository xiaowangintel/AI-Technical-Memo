# model_file_verifier.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/utils/model_file_verifier.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Provides reusable runtime utilities centered on `model_file_verifier` and the surrounding SGLang serving stack. / 提供围绕 `model_file_verifier` 及其所处 SGLang 服务栈的可复用运行时工具。

## Line-by-Line Analysis / 逐行分析
### Lines 1-21: Module setup and shared state / 模块设置与共享状态
```python
"""
Model File Verifier - Verify model file integrity using SHA256 checksums.

Example commands:
    # Verify using HuggingFace model online metadata
    python -m sglang.srt.utils.model_file_verifier verify --model-path /path/to/model --model-checksum Qwen/Qwen3-0.6B

    # Verify using locally generated checksum
    python -m sglang.srt.utils.model_file_verifier generate --model-path <hf-id-or-model-path> --model-checksum checksums.json
    python -m sglang.srt.utils.model_file_verifier verify --model-path /path/to/model --model-checksum checksums.json
"""

import argparse
import fnmatch
import hashlib
import json
import warnings
from concurrent.futures import ThreadPoolExecutor
from dataclasses import asdict, dataclass
from pathlib import Path
from typing import Dict, List, Optional, Tuple
```
**EN:** This block prepares the module by collecting imports, constants, and top-level state. Key dependencies introduced here include `argparse`, `fnmatch`, `hashlib`, `json`, `warnings`, `concurrent.futures`.
**CN:** 该代码块通过组织导入、常量和顶层状态来完成模块初始化。 这里引入的关键依赖包括 `argparse`, `fnmatch`, `hashlib`, `json`, `warnings`, `concurrent.futures`。

### Lines 26-29: Class `FileInfo` declaration / 类 `FileInfo` 声明
```python
@dataclass
class FileInfo:
    sha256: str
    size: int
```
**EN:** This class establishes `FileInfo` as a compact data container for the surrounding logic. The main stored fields include `sha256`, `size`.
**CN:** 该类将 `FileInfo` 定义为周边逻辑的紧凑的数据容器。 其主要存储字段包括 `sha256`, `size`。

### Lines 32-34: Class `Manifest` declaration / 类 `Manifest` 声明
```python
@dataclass
class Manifest:
    files: Dict[str, FileInfo]
```
**EN:** This class establishes `Manifest` as a compact data container for the surrounding logic. Its core interface includes methods such as `from_dict`, `to_dict`.
**CN:** 该类将 `Manifest` 定义为周边逻辑的紧凑的数据容器。 其核心接口包括 `from_dict`, `to_dict` 等方法。

### Lines 36-50: Method `Manifest.from_dict` / 方法 `Manifest.from_dict`
```python
    @classmethod
    def from_dict(cls, data: dict) -> "Manifest":
        if "checksums" in data:
            warnings.warn(
                "The 'checksums' format is deprecated. "
                "Please regenerate with the latest version to use the new 'files' format.",
                DeprecationWarning,
                stacklevel=3,
            )
            return cls(
                files={
                    k: FileInfo(sha256=v, size=-1) for k, v in data["checksums"].items()
                }
            )
        return cls(files={k: FileInfo(**v) for k, v in data["files"].items()})
```
**EN:** This method implements `from_dict` on `Manifest`. It primarily calls `cls`, `warnings.warn`, `FileInfo`, `data.items` to complete its work. The implementation relies on conditional branches.
**CN:** 该方法（属于 `Manifest`）实现了 `from_dict`。 它主要通过调用 `cls`, `warnings.warn`, `FileInfo`, `data.items` 来完成任务。 实现中使用了条件分支。

### Lines 52-53: Method `Manifest.to_dict` / 方法 `Manifest.to_dict`
```python
    def to_dict(self) -> dict:
        return asdict(self)
```
**EN:** This method implements `to_dict` on `Manifest`. It primarily calls `asdict` to complete its work.
**CN:** 该方法（属于 `Manifest`）实现了 `to_dict`。 它主要通过调用 `asdict` 来完成任务。

### Lines 59-68: Constants and shared state / 常量与共享状态
```python
IGNORE_PATTERNS = [
    ".DS_Store",
    "*.lock",
    ".gitattributes",
    "LICENSE",
    "LICENSE.*",
    "README.md",
    "README.*",
    "NOTICE",
]
```
**EN:** This block defines constants, docstrings, or shared state that other routines consume later in the file. The key names introduced here are `IGNORE_PATTERNS`.
**CN:** 该代码块定义了常量、文档字符串或共享状态，供文件中的其他例程后续使用。 这里引入的关键名称包括 `IGNORE_PATTERNS`。

### Lines 74-83: Function `verify` / 函数 `verify`
```python
def verify(*, model_path: str, checksums_source: str, max_workers: int = 4) -> None:
    model_path = Path(model_path).resolve()
    expected = _load_checksums(checksums_source)
    actual = _compute_manifest_from_folder(
        model_path=model_path,
        filenames=list(expected.files.keys()),
        max_workers=max_workers,
    )
    _compare_manifests(expected=expected, actual=actual)
    print(f"[ModelFileVerifier] All {len(expected.files)} files verified successfully.")
```
**EN:** This function implements `verify`. It primarily calls `Path.resolve`, `_load_checksums`, `_compute_manifest_from_folder`, `_compare_manifests`, `print`, `Path` to complete its work. State updates are written into `model_path`, `expected`, `actual`.
**CN:** 该函数实现了 `verify`。 它主要通过调用 `Path.resolve`, `_load_checksums`, `_compute_manifest_from_folder`, `_compare_manifests`, `print`, `Path` 来完成任务。 状态更新主要写入 `model_path`, `expected`, `actual`。

### Lines 86-98: Function `_compare_manifests` / 函数 `_compare_manifests`
```python
def _compare_manifests(*, expected: Manifest, actual: Manifest) -> None:
    errors = []
    for filename, exp in expected.files.items():
        if filename not in actual.files:
            errors.append(f"{filename}: missing (expected size={exp.size})")
        elif actual.files[filename].sha256 != exp.sha256:
            act = actual.files[filename]
            errors.append(
                f"{filename}: mismatch (expected={exp.sha256[:16]}... size={exp.size}, actual={act.sha256[:16]}... size={act.size})"
            )

    if errors:
        raise IntegrityError("Integrity check failed: " + "; ".join(errors))
```
**EN:** This function implements `_compare_manifests`. It primarily calls `expected.files.items`, `IntegrityError`, `errors.append`, `join` to complete its work. State updates are written into `errors`, `act`. The implementation relies on conditional branches, iteration.
**CN:** 该函数实现了 `_compare_manifests`。 它主要通过调用 `expected.files.items`, `IntegrityError`, `errors.append`, `join` 来完成任务。 状态更新主要写入 `errors`, `act`。 实现中使用了条件分支、迭代逻辑。

### Lines 104-125: Function `generate_checksums` / 函数 `generate_checksums`
```python
def generate_checksums(
    *, source: str, output_path: str, max_workers: int = 4
) -> Manifest:
    if Path(source).is_dir():
        model_path = Path(source).resolve()
        files = _discover_files(model_path)
        if not files:
            raise IntegrityError(f"No model files found in {model_path}")
        manifest = _compute_manifest_from_folder(
            model_path=model_path, filenames=files, max_workers=max_workers
        )
    else:
        manifest = Manifest(files=_load_file_infos_from_hf(repo_id=source))

    Path(output_path).write_text(
        json.dumps(manifest.to_dict(), indent=2, sort_keys=True)
    )

    print(
        f"[ModelFileVerifier] Generated checksums for {len(manifest.files)} files -> {output_path}"
    )
    return manifest
```
**EN:** This function implements `generate_checksums`. It primarily calls `Path.is_dir`, `Path.write_text`, `print`, `Path.resolve`, `_discover_files`, `_compute_manifest_from_folder` to complete its work. State updates are written into `model_path`, `files`, `manifest`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `generate_checksums`。 它主要通过调用 `Path.is_dir`, `Path.write_text`, `print`, `Path.resolve`, `_discover_files`, `_compute_manifest_from_folder` 来完成任务。 状态更新主要写入 `model_path`, `files`, `manifest`。 实现中使用了条件分支。

### Lines 128-135: Function `_discover_files` / 函数 `_discover_files`
```python
def _discover_files(model_path: Path) -> List[str]:
    return sorted(
        e.name
        for e in model_path.iterdir()
        if e.is_file()
        and not e.name.startswith(".")
        and not any(fnmatch.fnmatch(e.name, p) for p in IGNORE_PATTERNS)
    )
```
**EN:** This function implements `_discover_files`. It primarily calls `sorted`, `model_path.iterdir`, `e.is_file`, `e.name.startswith`, `any`, `fnmatch.fnmatch` to complete its work.
**CN:** 该函数实现了 `_discover_files`。 它主要通过调用 `sorted`, `model_path.iterdir`, `e.is_file`, `e.name.startswith`, `any`, `fnmatch.fnmatch` 来完成任务。

### Lines 141-145: Function `_load_checksums` / 函数 `_load_checksums`
```python
def _load_checksums(source: str) -> Manifest:
    if Path(source).is_file():
        data = json.loads(Path(source).read_text())
        return Manifest.from_dict(data)
    return Manifest(files=_load_file_infos_from_hf(repo_id=source))
```
**EN:** This function implements `_load_checksums`. It primarily calls `Path.is_file`, `Manifest`, `json.loads`, `Manifest.from_dict`, `Path`, `Path.read_text` to complete its work. State updates are written into `data`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_load_checksums`。 它主要通过调用 `Path.is_file`, `Manifest`, `json.loads`, `Manifest.from_dict`, `Path`, `Path.read_text` 来完成任务。 状态更新主要写入 `data`。 实现中使用了条件分支。

### Lines 148-160: Function `_load_file_infos_from_hf` / 函数 `_load_file_infos_from_hf`
```python
def _load_file_infos_from_hf(*, repo_id: str) -> Dict[str, FileInfo]:
    from huggingface_hub import HfFileSystem

    fs = HfFileSystem()
    files = fs.ls(repo_id, detail=True)

    file_infos = dict(
        r for r in map(lambda f: _get_filename_and_info_from_hf_file(fs, f), files) if r
    )
    if not file_infos:
        raise IntegrityError(f"No files found in HF repo {repo_id}.")

    return file_infos
```
**EN:** This function implements `_load_file_infos_from_hf`. It primarily calls `HfFileSystem`, `fs.ls`, `dict`, `IntegrityError`, `map`, `_get_filename_and_info_from_hf_file` to complete its work. State updates are written into `fs`, `files`, `file_infos`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_load_file_infos_from_hf`。 它主要通过调用 `HfFileSystem`, `fs.ls`, `dict`, `IntegrityError`, `map`, `_get_filename_and_info_from_hf_file` 来完成任务。 状态更新主要写入 `fs`, `files`, `file_infos`。 实现中使用了条件分支。

### Lines 163-184: Function `_get_filename_and_info_from_hf_file` / 函数 `_get_filename_and_info_from_hf_file`
```python
def _get_filename_and_info_from_hf_file(
    fs, file_info
) -> Optional[Tuple[str, FileInfo]]:
    if file_info.get("type") != "file":
        return None

    filename = Path(file_info.get("name", "")).name
    if any(fnmatch.fnmatch(filename, pat) for pat in IGNORE_PATTERNS):
        return None

    size = file_info.get("size", -1)
    lfs_info = file_info.get("lfs")
    if lfs_info and "sha256" in lfs_info:
        return filename, FileInfo(sha256=lfs_info["sha256"], size=size)

    if "sha256" in file_info:
        return filename, FileInfo(sha256=file_info["sha256"], size=size)

    content = fs.read_bytes(file_info.get("name", ""))
    return filename, FileInfo(
        sha256=hashlib.sha256(content).hexdigest(), size=len(content)
    )
```
**EN:** This function implements `_get_filename_and_info_from_hf_file`. It primarily calls `any`, `file_info.get`, `fs.read_bytes`, `Path`, `FileInfo`, `fnmatch.fnmatch` to complete its work. State updates are written into `filename`, `size`, `lfs_info`, `content`. The implementation relies on conditional branches.
**CN:** 该函数实现了 `_get_filename_and_info_from_hf_file`。 它主要通过调用 `any`, `file_info.get`, `fs.read_bytes`, `Path`, `FileInfo`, `fnmatch.fnmatch` 来完成任务。 状态更新主要写入 `filename`, `size`, `lfs_info`, `content`。 实现中使用了条件分支。

### Lines 190-212: Function `_compute_manifest_from_folder` / 函数 `_compute_manifest_from_folder`
```python
def _compute_manifest_from_folder(
    *, model_path: Path, filenames: List[str], max_workers: int
) -> Manifest:
    from tqdm import tqdm

    def compute_one(filename: str) -> Tuple[str, Optional[FileInfo]]:
        full_path = model_path / filename
        if not full_path.exists():
            return filename, None
        sha256 = compute_sha256(file_path=full_path)
        size = full_path.stat().st_size
        return filename, FileInfo(sha256=sha256, size=size)

    with ThreadPoolExecutor(max_workers=max_workers) as executor:
        results = list(
            tqdm(
                executor.map(compute_one, filenames),
                total=len(filenames),
                desc="Computing checksums",
            )
        )

    return Manifest(files={k: v for k, v in results if v is not None})
```
**EN:** This function implements `_compute_manifest_from_folder`. It primarily calls `Manifest`, `compute_sha256`, `ThreadPoolExecutor`, `list`, `full_path.exists`, `full_path.stat` to complete its work. State updates are written into `full_path`, `sha256`, `size`, `results`. The implementation relies on conditional branches, context-managed resources.
**CN:** 该函数实现了 `_compute_manifest_from_folder`。 它主要通过调用 `Manifest`, `compute_sha256`, `ThreadPoolExecutor`, `list`, `full_path.exists`, `full_path.stat` 来完成任务。 状态更新主要写入 `full_path`, `sha256`, `size`, `results`。 实现中使用了条件分支、上下文管理资源。

### Lines 215-220: Function `compute_sha256` / 函数 `compute_sha256`
```python
def compute_sha256(*, file_path) -> str:
    sha256 = hashlib.sha256()
    with open(file_path, "rb") as f:
        while chunk := f.read(64 * 1024):
            sha256.update(chunk)
    return sha256.hexdigest()
```
**EN:** This function implements `compute_sha256`. It primarily calls `hashlib.sha256`, `sha256.hexdigest`, `open`, `f.read`, `sha256.update` to complete its work. State updates are written into `sha256`. The implementation relies on iteration, context-managed resources.
**CN:** 该函数实现了 `compute_sha256`。 它主要通过调用 `hashlib.sha256`, `sha256.hexdigest`, `open`, `f.read`, `sha256.update` 来完成任务。 状态更新主要写入 `sha256`。 实现中使用了迭代逻辑、上下文管理资源。

### Lines 226-227: Class `IntegrityError` declaration / 类 `IntegrityError` 声明
```python
class IntegrityError(Exception):
    pass
```
**EN:** This class establishes `IntegrityError` as the main container/coordinator for the surrounding logic. It inherits from `Exception`.
**CN:** 该类将 `IntegrityError` 定义为周边逻辑的主要封装体或协调者。 它继承自 `Exception`。

### Lines 233-246: Function `_add_common_args` / 函数 `_add_common_args`
```python
def _add_common_args(parser):
    parser.add_argument(
        "--model-path",
        required=True,
        help="Local model directory or HuggingFace repo ID",
    )
    parser.add_argument(
        "--model-checksum",
        required=True,
        help="Checksums JSON file path",
    )
    parser.add_argument(
        "--workers", type=int, default=4, help="Number of parallel workers"
    )
```
**EN:** This function implements `_add_common_args`. It primarily calls `parser.add_argument` to complete its work.
**CN:** 该函数实现了 `_add_common_args`。 它主要通过调用 `parser.add_argument` 来完成任务。

### Lines 249-280: Function `main` / 函数 `main`
```python
def main():
    parser = argparse.ArgumentParser(
        description="Model File Verifier - Verify model file integrity using checksums"
    )
    subparsers = parser.add_subparsers(dest="command", required=True)

    gen_parser = subparsers.add_parser(
        "generate", help="Generate checksums.json for a model"
    )
    _add_common_args(gen_parser)
    gen_parser.set_defaults(
        func=lambda args: generate_checksums(
            source=args.model_path,
            output_path=args.model_checksum,
            max_workers=args.workers,
        )
    )

    verify_parser = subparsers.add_parser(
        "verify", help="Verify model files against checksums"
    )
    _add_common_args(verify_parser)
    verify_parser.set_defaults(
        func=lambda args: verify(
            model_path=args.model_path,
            checksums_source=args.model_checksum,
            max_workers=args.workers,
        )
    )

    args = parser.parse_args()
    args.func(args)
```
**EN:** This function implements `main`. It primarily calls `argparse.ArgumentParser`, `parser.add_subparsers`, `subparsers.add_parser`, `_add_common_args`, `gen_parser.set_defaults`, `verify_parser.set_defaults` to complete its work. State updates are written into `parser`, `subparsers`, `gen_parser`, `verify_parser`, `args`.
**CN:** 该函数实现了 `main`。 它主要通过调用 `argparse.ArgumentParser`, `parser.add_subparsers`, `subparsers.add_parser`, `_add_common_args`, `gen_parser.set_defaults`, `verify_parser.set_defaults` 来完成任务。 状态更新主要写入 `parser`, `subparsers`, `gen_parser`, `verify_parser`, `args`。

### Lines 283-284: Conditional and fallback logic / 条件与回退逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This block contains conditional, fallback, or guarded runtime logic that adapts behavior to the current environment.
**CN:** 该代码块包含条件分支、回退路径或受保护的运行时逻辑，用于根据当前环境调整行为。

## Key Concepts / 关键概念
- **Classes / 类**: `FileInfo`, `Manifest`, `IntegrityError`
- **Functions / 函数**: `verify`, `_compare_manifests`, `generate_checksums`, `_discover_files`, `_load_checksums`, `_load_file_infos_from_hf`, `_get_filename_and_info_from_hf_file`, `_compute_manifest_from_folder`

## Dependencies / 依赖关系
- **Internal / 内部依赖**: None / 无
- **External / 外部依赖**: `huggingface_hub`, `tqdm`
- **Standard library / 标准库**: `argparse`, `fnmatch`, `hashlib`, `json`, `warnings`, `concurrent.futures`, `dataclasses`, `pathlib`, `typing`
