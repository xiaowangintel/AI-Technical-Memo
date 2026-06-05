# file_io_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/file_io_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7 / 第 1-7 行

````python
import json
import re
import shutil
from pathlib import Path
from typing import Any

import boto3  # type: ignore[import]
````

- EN: This block imports dependencies such as `json`, `re`, `shutil`, `pathlib`; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；序列化或解析结构化数据文件。

### Lines 10-22 / 第 10-22 行

````python
def zip_folder(folder_to_zip: Path, dest_file_base_name: Path) -> Path:
    """
    Returns the path to the resulting zip file, with the appropriate extension added if needed
    """
    # shutil.make_archive will append .zip to the dest_file_name, so we need to remove it if it's already there
    if dest_file_base_name.suffix == ".zip":
        dest_file_base_name = dest_file_base_name.with_suffix("")

    ensure_dir_exists(dest_file_base_name.parent)

    print(f"Zipping {folder_to_zip}\n     to {dest_file_base_name}")
    # Convert to string because shutil.make_archive doesn't like Path objects
    return Path(shutil.make_archive(str(dest_file_base_name), "zip", folder_to_zip))
````

- EN: This block defines callable units such as `zip_folder`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 25-35 / 第 25-35 行

````python
def unzip_folder(zip_file_path: Path, unzip_to_folder: Path) -> None:
    """
    Returns the path to the unzipped folder
    """
    print(f"Unzipping {zip_file_path}")
    print(f"       to {unzip_to_folder}")
    shutil.unpack_archive(zip_file_path, unzip_to_folder, "zip")


def ensure_dir_exists(dir: Path) -> None:
    dir.mkdir(parents=True, exist_ok=True)
````

- EN: This block defines callable units such as `unzip_folder`, `ensure_dir_exists`.
- CN: 该代码块定义可调用函数来执行具体自动化步骤。

### Lines 38-48 / 第 38-48 行

````python
def copy_file(source_file: Path, dest_file: Path) -> None:
    ensure_dir_exists(dest_file.parent)
    shutil.copyfile(source_file, dest_file)


def load_json_file(file_path: Path) -> Any:
    """
    Returns the deserialized json object
    """
    with open(file_path) as f:
        return json.load(f)
````

- EN: This block defines callable units such as `copy_file`, `load_json_file`; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 51-64 / 第 51-64 行

````python
def write_json_file(file_path: Path, content: Any) -> None:
    dir = file_path.parent
    ensure_dir_exists(dir)

    with open(file_path, "w") as f:
        json.dump(content, f, indent=2)


def sanitize_for_s3(text: str) -> str:
    """
    S3 keys can only contain alphanumeric characters, underscores, and dashes.
    This function replaces all other characters with underscores.
    """
    return re.sub(r"[^a-zA-Z0-9_-]", "_", text)
````

- EN: This block defines callable units such as `write_json_file`, `sanitize_for_s3`; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 67-75 / 第 67-75 行

````python
def upload_file_to_s3(file_name: Path, bucket: str, key: str) -> None:
    print(f"Uploading {file_name}")
    print(f"       to s3://{bucket}/{key}")

    boto3.client("s3").upload_file(
        str(file_name),
        bucket,
        key,
    )
````

- EN: This block defines callable units such as `upload_file_to_s3`.
- CN: 该代码块定义可调用函数来执行具体自动化步骤。

### Lines 78-87 / 第 78-87 行

````python
def download_s3_objects_with_prefix(
    bucket_name: str, prefix: str, download_folder: Path
) -> list[Path]:
    s3 = boto3.resource("s3")
    bucket = s3.Bucket(bucket_name)

    downloads = []

    for obj in bucket.objects.filter(Prefix=prefix):
        download_path = download_folder / obj.key
````

- EN: This block defines callable units such as `download_s3_objects_with_prefix`; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项。

### Lines 89-99 / 第 89-99 行

````python
        ensure_dir_exists(download_path.parent)
        print(f"Downloading s3://{bucket.name}/{obj.key}")
        print(f"         to {download_path}")

        s3.Object(bucket.name, obj.key).download_file(str(download_path))
        downloads.append(download_path)

    if len(downloads) == 0:
        print(
            f"There were no files matching the prefix `{prefix}` in bucket `{bucket.name}`"
        )
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 101-101 / 第 101-101 行

````python
    return downloads
````

- EN: This block returns computed values to callers.
- CN: 该代码块向调用方返回计算结果。


## Key Concepts / 关键概念
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `json`, `re`, `shutil`, `pathlib`, `typing`
- Internal imports / 内部导入: none
- External imports / 外部导入: `boto3`
- Classes / 类: none
- Functions / 函数: `zip_folder`, `unzip_folder`, `ensure_dir_exists`, `copy_file`, `load_json_file`, `write_json_file`, `sanitize_for_s3`, `upload_file_to_s3`, `download_s3_objects_with_prefix`
- Test entry points / 测试入口: none
