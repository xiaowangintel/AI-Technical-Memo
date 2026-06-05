# data_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/mmmu/data_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on mmmu data. It primarily loads, filters, or serializes benchmark datasets, computes evaluation scores and aggregate statistics, and reads or writes local files and intermediate artifacts. / 该 Python 模块聚焦于 mmmu data 相关流程。它主要用于加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果、读写本地文件及中间产物。

## Line-by-Line Analysis / 逐行分析
### Lines 1-75: Imports and setup / 导入与初始化
```python
"""Utils for data load, save, and process (e.g., prompt construction)"""

import json
import os
import re

import yaml

DOMAIN_CAT2SUB_CAT = {
    "Art and Design": ["Art", "Art_Theory", "Design", "Music"],
    "Business": ["Accounting", "Economics", "Finance", "Manage", "Marketing"],
    "Science": [
        "Biology",
        "Chemistry",
        "Geography",
        "Math",
        "Physics",
    ],
    "Health and Medicine": [
        "Basic_Medical_Science",
        "Clinical_Medicine",
        "Diagnostics_and_Laboratory_Medicine",
        "Pharmacy",
        "Public_Health",
    ],
    "Humanities and Social Science": [
        "History",
        "Literature",
        "Sociology",
        "Psychology",
    ],
    "Tech and Engineering": [
        "Agriculture",
        "Architecture_and_Engineering",
        "Computer_Science",
        "Electronics",
        "Energy_and_Power",
        "Materials",
        "Mechanical_Engineering",
    ],
}


CAT_SHORT2LONG = {
    "acc": "Accounting",
    "agri": "Agriculture",
    "arch": "Architecture_and_Engineering",
    "art": "Art",
    "art_theory": "Art_Theory",
    "bas_med": "Basic_Medical_Science",
    "bio": "Biology",
    "chem": "Chemistry",
    "cli_med": "Clinical_Medicine",
    "cs": "Computer_Science",
    "design": "Design",
    "diag_med": "Diagnostics_and_Laboratory_Medicine",
    "econ": "Economics",
    "elec": "Electronics",
    "ep": "Energy_and_Power",
    "fin": "Finance",
    "geo": "Geography",
    "his": "History",
    "liter": "Literature",
    "manage": "Manage",
    "mark": "Marketing",
    "mate": "Materials",
    "math": "Math",
    "mech": "Mechanical_Engineering",
    "music": "Music",
    "phar": "Pharmacy",
    "phys": "Physics",
    "psy": "Psychology",
    "pub_health": "Public_Health",
    "socio": "Sociology",
}
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it loads, filters, or serializes benchmark datasets and computes evaluation scores and aggregate statistics.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果。

### Lines 78-91: Function `get_multi_choice_info` / 函数 `get_multi_choice_info`
```python
def get_multi_choice_info(options):
    """
    Given the list of options for multiple choice question
    Return the index2ans and all_choices
    """

    start_chr = "A"
    all_choices = []
    index2ans = {}
    for i, option in enumerate(options):
        index2ans[chr(ord(start_chr) + i)] = option
        all_choices.append(chr(ord(start_chr) + i))

    return index2ans, all_choices
```
**EN:** `get_multi_choice_info` is a function that implements the core logic for this scope. The docstring frames it as: Given the list of options for multiple choice question Return the index2ans and all_choices It returns `(index2ans, all_choices)` to the caller. Notable calls include `enumerate`, `all_choices.append`, `chr`.
**CN:** `get_multi_choice_info` 是一个函数，用于实现当前作用域的核心逻辑。文档字符串进一步说明了该逻辑的用途与调用语义。它会向调用方返回 `(index2ans, all_choices)`。其中较关键的调用包括 `enumerate`, `all_choices.append`, `chr`。

### Lines 94-101: Function `load_yaml` / 函数 `load_yaml`
```python
def load_yaml(file_path):
    with open(file_path, "r") as stream:
        try:
            yaml_dict = yaml.safe_load(stream)
        except yaml.YAMLError as exc:
            print(exc)

    return yaml_dict
```
**EN:** `load_yaml` is a function that reads or writes local files and intermediate artifacts. It returns `yaml_dict` to the caller. Notable calls include `open`, `yaml.safe_load`, `print`.
**CN:** `load_yaml` 是一个函数，用于读写本地文件及中间产物。它会向调用方返回 `yaml_dict`。其中较关键的调用包括 `open`, `yaml.safe_load`, `print`。

### Lines 104-106: Function `parse_img_path` / 函数 `parse_img_path`
```python
def parse_img_path(text):
    matches = re.findall("<img='(.*?)'>", text)
    return matches
```
**EN:** `parse_img_path` is a function that implements the core logic for this scope. It returns `matches` to the caller. Notable calls include `re.findall`.
**CN:** `parse_img_path` 是一个函数，用于实现当前作用域的核心逻辑。它会向调用方返回 `matches`。其中较关键的调用包括 `re.findall`。

### Lines 109-134: Function `process_single_sample` / 函数 `process_single_sample`
```python
def process_single_sample(data):
    question = data["question"]
    o_imgs_paths = []
    for option in data["options"]:
        current_o_imgs_paths = parse_img_path(option)
        for img_path in current_o_imgs_paths:
            o_imgs_paths.append(img_path)

    if len(o_imgs_paths) > 1:  # multiple images in options, used for random selection
        return {
            "id": data["id"],
            "question": question,
            "options": data["options"],
            "answer": data["answer"],
            "image": None,
            "question_type": data["question_type"],
        }
    else:
        return {
            "id": data["id"],
            "question": question,
            "options": data["options"],
            "answer": data["answer"],
            "image": data["image_1"],
            "question_type": data["question_type"],
        }
```
**EN:** `process_single_sample` is a function that computes evaluation scores and aggregate statistics. It returns `{'id': data['id'], 'question': question, 'options': data['options'], 'answer': data['answer'], 'i...` to the caller. Notable calls include `parse_img_path`, `len`, `o_imgs_paths.append`.
**CN:** `process_single_sample` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `{'id': data['id'], 'question': question, 'options': data['options'], 'answer': data['answer'], 'i...`。其中较关键的调用包括 `parse_img_path`, `len`, `o_imgs_paths.append`。

### Lines 138-142: Function `save_json` / 函数 `save_json`
```python
def save_json(filename, ds):
    print(f"answers saved to: {filename}")
    os.makedirs(os.path.dirname(filename), exist_ok=True)
    with open(filename, "w") as f:
        json.dump(ds, f, indent=4)
```
**EN:** `save_json` is a function that loads, filters, or serializes benchmark datasets, computes evaluation scores and aggregate statistics, and reads or writes local files and intermediate artifacts. Notable calls include `print`, `os.makedirs`, `os.path.dirname`.
**CN:** `save_json` 是一个函数，用于加载、筛选或序列化基准测试数据集、计算评测分数与聚合统计结果、读写本地文件及中间产物。其中较关键的调用包括 `print`, `os.makedirs`, `os.path.dirname`。

### Lines 145-160: Function `save_jsonl` / 函数 `save_jsonl`
```python
def save_jsonl(filename, data):
    """
    Save a dictionary of data to a JSON Lines file with the filename as key and caption as value.

    Args:
        filename (str): The path to the file where the data should be saved.
        data (dict): The dictionary containing the data to save where key is the image path and value is the caption.
    """
    with open(filename, "w", encoding="utf-8") as f:
        for img_path, caption in data.items():
            # Extract the base filename without the extension
            base_filename = os.path.basename(img_path)
            # Create a JSON object with the filename as the key and caption as the value
            json_record = json.dumps({base_filename: caption}, ensure_ascii=False)
            # Write the JSON object to the file, one per line
            f.write(json_record + "\n")
```
**EN:** `save_jsonl` is a function that loads, filters, or serializes benchmark datasets and reads or writes local files and intermediate artifacts. The docstring frames it as: Save a dictionary of data to a JSON Lines file with the filename as key and caption as value. Notable calls include `open`, `data.items`, `os.path.basename`.
**CN:** `save_jsonl` 是一个函数，用于加载、筛选或序列化基准测试数据集、读写本地文件及中间产物。文档字符串进一步说明了该逻辑的用途与调用语义。其中较关键的调用包括 `open`, `data.items`, `os.path.basename`。

### Lines 163-169: Function `save_args` / 函数 `save_args`
```python
def save_args(args, path_dir):
    argsDict = args.__dict__
    with open(path_dir + "setting.txt", "w") as f:
        f.writelines("------------------ start ------------------" + "\n")
        for eachArg, value in argsDict.items():
            f.writelines(eachArg + " : " + str(value) + "\n")
        f.writelines("------------------- end -------------------")
```
**EN:** `save_args` is a function that reads or writes local files and intermediate artifacts. Notable calls include `open`, `f.writelines`, `argsDict.items`.
**CN:** `save_args` 是一个函数，用于读写本地文件及中间产物。其中较关键的调用包括 `open`, `f.writelines`, `argsDict.items`。

### Lines 173-215: Function `construct_prompt` / 函数 `construct_prompt`
```python
def construct_prompt(sample, config):
    question = sample["question"]
    options = eval(sample["options"])
    example = ""
    if sample["question_type"] == "multiple-choice":
        start_chr = "A"
        prediction_range = []
        index2ans = {}
        for option in options:
            prediction_range.append(start_chr)
            example += f"({start_chr}) {option}\n"
            index2ans[start_chr] = option
            start_chr = chr(ord(start_chr) + 1)
        empty_prompt_sample_structure = config["multi_choice_example_format"]
        empty_prompt = empty_prompt_sample_structure.format(question, example)
        res_dict = {}
        res_dict["index2ans"] = index2ans
        res_dict["correct_choice"] = sample["answer"]
        res_dict["all_choices"] = prediction_range
        res_dict["empty_prompt"] = empty_prompt
        if config["task_instructions"]:
            res_dict["final_input_prompt"] = (
                config["task_instructions"].strip() + "\n\n" + empty_prompt
            )
        else:
            res_dict["final_input_prompt"] = empty_prompt

        res_dict["gt_content"] = options[ord(sample["answer"].upper()) - ord("A")]
    else:
        empty_prompt_sample_structure = config["short_ans_example_format"]
        empty_prompt = empty_prompt_sample_structure.format(question)
        res_dict = {}
        res_dict["empty_prompt"] = empty_prompt
        if config["task_instructions"]:
            res_dict["final_input_prompt"] = (
                config["task_instructions"].strip() + "\n\n" + empty_prompt
            )
        else:
            res_dict["final_input_prompt"] = empty_prompt
        res_dict["gt_content"] = sample["answer"]

    res_dict.update(sample)
    return res_dict
```
**EN:** `construct_prompt` is a function that computes evaluation scores and aggregate statistics. It returns `res_dict` to the caller. Notable calls include `eval`, `res_dict.update`, `empty_prompt_sample_structure.format`.
**CN:** `construct_prompt` 是一个函数，用于计算评测分数与聚合统计结果。它会向调用方返回 `res_dict`。其中较关键的调用包括 `eval`, `res_dict.update`, `empty_prompt_sample_structure.format`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Datasets / 数据集**: Loads or reshapes benchmark inputs from structured files. / 从结构化文件中加载或整理基准测试输入。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `re`
- **Third-party / 第三方依赖**: `yaml`
