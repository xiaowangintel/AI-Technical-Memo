# classifier.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/release_notes/classifier.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements release-note tooling that classifies commits, applies categories, and assembles changelog inputs.
- **用途 (CN)**: 实现发布说明工具，用于分类提交、应用类别并组装变更日志输入。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17 / 第 1-17 行

````python
import argparse
import math
import pickle
import random
from dataclasses import dataclass
from itertools import chain
from pathlib import Path
from typing import Dict, List

import common
import pandas as pd
import torchtext
from torchtext.functional import to_tensor
from tqdm import tqdm

import torch
import torch.nn as nn
````

- EN: This block imports dependencies such as `argparse`, `math`, `pickle`, `random`; parses command-line arguments for script entry points.
- CN: 该代码块导入当前模块运行所需的依赖；为脚本入口解析命令行参数。

### Lines 20-35 / 第 20-35 行

````python
XLMR_BASE = torchtext.models.XLMR_BASE_ENCODER
# This should not be here but it works for now
device = "cuda" if torch.cuda.is_available() else "cpu"

HAS_IMBLEARN = False
try:
    import imblearn

    HAS_IMBLEARN = True
except ImportError:
    HAS_IMBLEARN = False

# 94% of all files are captured at len 5, good hyperparameter to play around with.
MAX_LEN_FILE = 6

UNKNOWN_TOKEN = "<Unknown>"
````

- EN: This block imports dependencies such as `imblearn`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块导入当前模块运行所需的依赖；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 37-53 / 第 37-53 行

````python
# Utilities for working with a truncated file graph


def truncate_file(file: Path, max_len: int = 5):
    return ("/").join(file.parts[:max_len])


def build_file_set(all_files: List[Path], max_len: int):
    truncated_files = [truncate_file(file, max_len) for file in all_files]
    return set(truncated_files)


@dataclass
class CommitClassifierInputs:
    title: List[str]
    files: List[str]
    author: List[str]
````

- EN: This block introduces classes like `CommitClassifierInputs`; defines callable units such as `truncate_file`, `build_file_set`; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 56-67 / 第 56-67 行

````python
@dataclass
class CategoryConfig:
    categories: List[str]
    input_dim: int = 768
    inner_dim: int = 128
    dropout: float = 0.1
    activation = nn.ReLU
    embedding_dim: int = 8
    file_embedding_dim: int = 32


class CommitClassifier(nn.Module):
````

- EN: This block introduces classes like `CategoryConfig`, `CommitClassifier`.
- CN: 该代码块定义新的类来封装状态与行为。

### Lines 68-85 / 第 68-85 行

````python
    def __init__(
        self,
        encoder_base: torchtext.models.XLMR_BASE_ENCODER,
        author_map: Dict[str, int],
        file_map: [str, int],
        config: CategoryConfig,
    ):
        super().__init__()
        self.encoder = encoder_base.get_model().requires_grad_(False)
        self.transform = encoder_base.transform()
        self.author_map = author_map
        self.file_map = file_map
        self.categories = config.categories
        self.num_authors = len(author_map)
        self.num_files = len(file_map)
        self.embedding_table = nn.Embedding(self.num_authors, config.embedding_dim)
        self.file_embedding_bag = nn.EmbeddingBag(
            self.num_files, config.file_embedding_dim, mode="sum"
````

- EN: This block defines callable units such as `__init__`.
- CN: 该代码块定义可调用函数来执行具体自动化步骤。

### Lines 86-103 / 第 86-103 行

````python
        )
        self.dense_title = nn.Linear(config.input_dim, config.inner_dim)
        self.dense_files = nn.Linear(config.file_embedding_dim, config.inner_dim)
        self.dense_author = nn.Linear(config.embedding_dim, config.inner_dim)
        self.dropout = nn.Dropout(config.dropout)
        self.out_proj_title = nn.Linear(config.inner_dim, len(self.categories))
        self.out_proj_files = nn.Linear(config.inner_dim, len(self.categories))
        self.out_proj_author = nn.Linear(config.inner_dim, len(self.categories))
        self.activation_fn = config.activation()

    def forward(self, input_batch: CommitClassifierInputs):
        # Encode input title
        title: List[str] = input_batch.title
        model_input = to_tensor(self.transform(title), padding_value=1).to(device)
        title_features = self.encoder(model_input)
        title_embed = title_features[:, 0, :]
        title_embed = self.dropout(title_embed)
        title_embed = self.dense_title(title_embed)
````

- EN: This block defines callable units such as `forward`.
- CN: 该代码块定义可调用函数来执行具体自动化步骤。

### Lines 104-120 / 第 104-120 行

````python
        title_embed = self.activation_fn(title_embed)
        title_embed = self.dropout(title_embed)
        title_embed = self.out_proj_title(title_embed)

        files: list[str] = input_batch.files
        batch_file_indexes = []
        for file in files:
            paths = [
                truncate_file(Path(file_part), MAX_LEN_FILE)
                for file_part in file.split(" ")
            ]
            batch_file_indexes.append(
                [
                    self.file_map.get(file, self.file_map[UNKNOWN_TOKEN])
                    for file in paths
                ]
            )
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 122-136 / 第 122-136 行

````python
        flat_indexes = torch.tensor(
            list(chain.from_iterable(batch_file_indexes)),
            dtype=torch.long,
            device=device,
        )
        offsets = [0]
        offsets.extend(len(files) for files in batch_file_indexes[:-1])
        offsets = torch.tensor(offsets, dtype=torch.long, device=device)
        offsets = offsets.cumsum(dim=0)

        files_embed = self.file_embedding_bag(flat_indexes, offsets)
        files_embed = self.dense_files(files_embed)
        files_embed = self.activation_fn(files_embed)
        files_embed = self.dropout(files_embed)
        files_embed = self.out_proj_files(files_embed)
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 138-151 / 第 138-151 行

````python
        # Add author embedding
        authors: List[str] = input_batch.author
        author_ids = [
            self.author_map.get(author, self.author_map[UNKNOWN_TOKEN])
            for author in authors
        ]
        author_ids = torch.tensor(author_ids).to(device)
        author_embed = self.embedding_table(author_ids)
        author_embed = self.dense_author(author_embed)
        author_embed = self.activation_fn(author_embed)
        author_embed = self.dropout(author_embed)
        author_embed = self.out_proj_author(author_embed)

        return title_embed + files_embed + author_embed
````

- EN: This block iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 153-163 / 第 153-163 行

````python
    def convert_index_to_category_name(self, most_likely_index):
        if isinstance(most_likely_index, int):
            return self.categories[most_likely_index]
        elif isinstance(most_likely_index, torch.Tensor):
            return [self.categories[i] for i in most_likely_index]

    def get_most_likely_category_name(self, input):
        # Input will be a dict with title and author keys
        logits = self.forward(input)
        most_likely_index = torch.argmax(logits, dim=1)
        return self.convert_index_to_category_name(most_likely_index)
````

- EN: This block defines callable units such as `convert_index_to_category_name`, `get_most_likely_category_name`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 166-183 / 第 166-183 行

````python
def get_train_val_data(data_folder: Path, regen_data: bool, train_percentage=0.95):
    if (
        not regen_data
        and Path(data_folder / "train_df.csv").exists()
        and Path(data_folder / "val_df.csv").exists()
    ):
        train_data = pd.read_csv(data_folder / "train_df.csv")
        val_data = pd.read_csv(data_folder / "val_df.csv")
        return train_data, val_data
    else:
        print("Train, Val, Test Split not found generating from scratch.")
        commit_list_df = pd.read_csv(data_folder / "commitlist.csv")
        test_df = commit_list_df[commit_list_df["category"] == "Uncategorized"]
        all_train_df = commit_list_df[commit_list_df["category"] != "Uncategorized"]
        # We are going to drop skip from training set since it is so imbalanced
        print(
            "We are removing skip categories, YOU MIGHT WANT TO CHANGE THIS, BUT THIS IS A MORE HELPFUL CLASSIFIER FOR LABELING."
        )
````

- EN: This block defines callable units such as `get_train_val_data`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 184-195 / 第 184-195 行

````python
        all_train_df = all_train_df[all_train_df["category"] != "skip"]
        all_train_df = all_train_df.sample(frac=1).reset_index(drop=True)
        split_index = math.floor(train_percentage * len(all_train_df))
        train_df = all_train_df[:split_index]
        val_df = all_train_df[split_index:]
        print("Train data size: ", len(train_df))
        print("Val data size: ", len(val_df))

        test_df.to_csv(data_folder / "test_df.csv", index=False)
        train_df.to_csv(data_folder / "train_df.csv", index=False)
        val_df.to_csv(data_folder / "val_df.csv", index=False)
        return train_df, val_df
````

- EN: This block returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 198-214 / 第 198-214 行

````python
def get_author_map(data_folder: Path, regen_data, assert_stored=False):
    if not regen_data and Path(data_folder / "author_map.pkl").exists():
        with open(data_folder / "author_map.pkl", "rb") as f:
            return pickle.load(f)
    else:
        if assert_stored:
            raise FileNotFoundError(
                "Author map not found, you are loading for inference you need to have an author map!"
            )
        print("Regenerating Author Map")
        all_data = pd.read_csv(data_folder / "commitlist.csv")
        authors = all_data.author.unique().tolist()
        authors.append(UNKNOWN_TOKEN)
        author_map = {author: i for i, author in enumerate(authors)}
        with open(data_folder / "author_map.pkl", "wb") as f:
            pickle.dump(author_map, f)
        return author_map
````

- EN: This block defines callable units such as `get_author_map`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败；序列化或解析结构化数据文件。

### Lines 217-229 / 第 217-229 行

````python
def get_file_map(data_folder: Path, regen_data, assert_stored=False):
    if not regen_data and Path(data_folder / "file_map.pkl").exists():
        with open(data_folder / "file_map.pkl", "rb") as f:
            return pickle.load(f)
    else:
        if assert_stored:
            raise FileNotFoundError(
                "File map not found, you are loading for inference you need to have a file map!"
            )
        print("Regenerating File Map")
        all_data = pd.read_csv(data_folder / "commitlist.csv")
        # Lets explore files
        files = all_data.files_changed.to_list()
````

- EN: This block defines callable units such as `get_file_map`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败；序列化或解析结构化数据文件。

### Lines 231-243 / 第 231-243 行

````python
        all_files = []
        for file in files:
            paths = [Path(file_part) for file_part in file.split(" ")]
            all_files.extend(paths)
        all_files.append(Path(UNKNOWN_TOKEN))
        file_set = build_file_set(all_files, MAX_LEN_FILE)
        file_map = {file: i for i, file in enumerate(file_set)}
        with open(data_folder / "file_map.pkl", "wb") as f:
            pickle.dump(file_map, f)
        return file_map


#  Generate a dataset for training
````

- EN: This block iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 246-263 / 第 246-263 行

````python
def get_title_files_author_categories_zip_list(dataframe: pd.DataFrame):
    title = dataframe.title.to_list()
    files_str = dataframe.files_changed.to_list()
    author = dataframe.author.fillna(UNKNOWN_TOKEN).to_list()
    category = dataframe.category.to_list()
    return list(zip(title, files_str, author, category))


def generate_batch(batch):
    title, files, author, category = zip(*batch)
    title = list(title)
    files = list(files)
    author = list(author)
    category = list(category)
    targets = torch.tensor([common.categories.index(cat) for cat in category]).to(
        device
    )
    return CommitClassifierInputs(title, files, author), targets
````

- EN: This block defines callable units such as `get_title_files_author_categories_zip_list`, `generate_batch`; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 266-281 / 第 266-281 行

````python
def train_step(batch, model, optimizer, loss):
    input, targets = batch
    optimizer.zero_grad()
    output = model(input)
    l = loss(output, targets)
    l.backward()
    optimizer.step()
    return l


@torch.no_grad()
def eval_step(batch, model, loss):
    input, targets = batch
    output = model(input)
    l = loss(output, targets)
    return l
````

- EN: This block defines callable units such as `train_step`, `eval_step`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 284-301 / 第 284-301 行

````python
def balance_dataset(dataset: List):
    if not HAS_IMBLEARN:
        return dataset
    title, files, author, category = zip(*dataset)
    category = [common.categories.index(cat) for cat in category]
    inpt_data = list(zip(title, files, author))
    from imblearn.over_sampling import RandomOverSampler

    # from imblearn.under_sampling import RandomUnderSampler
    rus = RandomOverSampler(random_state=42)
    X, y = rus.fit_resample(inpt_data, category)
    merged = list(zip(X, y))
    merged = random.sample(merged, k=2 * len(dataset))
    X, y = zip(*merged)
    rebuilt_dataset = []
    for i in range(len(X)):
        rebuilt_dataset.append((*X[i], common.categories[y[i]]))
    return rebuilt_dataset
````

- EN: This block imports dependencies such as `imblearn.over_sampling`; defines callable units such as `balance_dataset`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果。

### Lines 304-321 / 第 304-321 行

````python
def gen_class_weights(dataset: List):
    from collections import Counter

    epsilon = 1e-1
    title, files, author, category = zip(*dataset)
    category = [common.categories.index(cat) for cat in category]
    counter = Counter(category)
    percentile_33 = len(category) // 3
    most_common = counter.most_common(percentile_33)
    least_common = counter.most_common()[-percentile_33:]
    smoothed_top = sum(i[1] + epsilon for i in most_common) / len(most_common)
    smoothed_bottom = sum(i[1] + epsilon for i in least_common) / len(least_common) // 3
    class_weights = torch.tensor(
        [
            1.0 / (min(max(counter[i], smoothed_bottom), smoothed_top) + epsilon)
            for i in range(len(common.categories))
        ],
        device=device,
````

- EN: This block imports dependencies such as `collections`; defines callable units such as `gen_class_weights`; iterates through collections, records, or generated items.
- CN: 该代码块导入当前模块运行所需的依赖；定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项。

### Lines 322-336 / 第 322-336 行

````python
    )
    return class_weights


def train(save_path: Path, data_folder: Path, regen_data: bool, resample: bool):
    train_data, val_data = get_train_val_data(data_folder, regen_data)
    train_zip_list = get_title_files_author_categories_zip_list(train_data)
    val_zip_list = get_title_files_author_categories_zip_list(val_data)

    classifier_config = CategoryConfig(common.categories)
    author_map = get_author_map(data_folder, regen_data)
    file_map = get_file_map(data_folder, regen_data)
    commit_classifier = CommitClassifier(
        XLMR_BASE, author_map, file_map, classifier_config
    ).to(device)
````

- EN: This block defines callable units such as `train`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 338-353 / 第 338-353 行

````python
    # Lets train this bag of bits
    class_weights = gen_class_weights(train_zip_list)
    loss = torch.nn.CrossEntropyLoss(weight=class_weights)
    optimizer = torch.optim.Adam(commit_classifier.parameters(), lr=3e-3)

    num_epochs = 25
    batch_size = 256

    if resample:
        # Lets not use this
        train_zip_list = balance_dataset(train_zip_list)
    data_size = len(train_zip_list)

    print(f"Training on {data_size} examples.")
    # We can fit all of val into one batch
    val_batch = generate_batch(val_zip_list)
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 355-371 / 第 355-371 行

````python
    for i in tqdm(range(num_epochs), desc="Epochs"):
        start = 0
        random.shuffle(train_zip_list)
        while start < data_size:
            end = start + batch_size
            # make the last batch bigger if needed
            if end > data_size:
                end = data_size
            train_batch = train_zip_list[start:end]
            train_batch = generate_batch(train_batch)
            l = train_step(train_batch, commit_classifier, optimizer, loss)
            start = end

        val_l = eval_step(val_batch, commit_classifier, loss)
        tqdm.write(
            f"Finished epoch {i} with a train loss of: {l.item()} and a val_loss of: {val_l.item()}"
        )
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项。

### Lines 373-388 / 第 373-388 行

````python
    with torch.no_grad():
        commit_classifier.eval()
        val_inpts, val_targets = val_batch
        val_output = commit_classifier(val_inpts)
        val_preds = torch.argmax(val_output, dim=1)
        val_acc = torch.sum(val_preds == val_targets).item() / len(val_preds)
        print(f"Final Validation accuracy is {val_acc}")

    print(f"Jobs done! Saving to {save_path}")
    torch.save(commit_classifier.state_dict(), save_path)


def main():
    parser = argparse.ArgumentParser(
        description="Tool to create a classifier for helping to categorize commits"
    )
````

- EN: This block defines callable units such as `main`; parses command-line arguments for script entry points; iterates through collections, records, or generated items.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；遍历集合、记录或生成项。

### Lines 390-405 / 第 390-405 行

````python
    parser.add_argument("--train", action="store_true", help="Train a new classifier")
    parser.add_argument("--commit_data_folder", default="results/classifier/")
    parser.add_argument(
        "--save_path", default="results/classifier/commit_classifier.pt"
    )
    parser.add_argument(
        "--regen_data",
        action="store_true",
        help="Regenerate the training data, helps if labeled more examples and want to re-train.",
    )
    parser.add_argument(
        "--resample",
        action="store_true",
        help="Resample the training data to be balanced. (Only works if imblearn is installed.)",
    )
    args = parser.parse_args()
````

- EN: This block parses command-line arguments for script entry points; branches on runtime conditions or configuration choices.
- CN: 该代码块为脚本入口解析命令行参数；依据运行时条件或配置选择进行分支。

### Lines 407-418 / 第 407-418 行

````python
    if args.train:
        train(
            Path(args.save_path),
            Path(args.commit_data_folder),
            args.regen_data,
            args.resample,
        )
        return

    print(
        "Currently this file only trains a new classifier please pass in --train to train a new classifier"
    )
````

- EN: This block branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 421-422 / 第 421-422 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `argparse`, `math`, `pickle`, `random`, `dataclasses`, `itertools`, `pathlib`, `typing`
- Internal imports / 内部导入: `torch`, `torch.nn`
- External imports / 外部导入: `common`, `pandas`, `torchtext`, `torchtext.functional`, `tqdm`
- Classes / 类: `CommitClassifierInputs`, `CategoryConfig`, `CommitClassifier`
- Functions / 函数: `truncate_file`, `build_file_set`, `get_train_val_data`, `get_author_map`, `get_file_map`, `get_title_files_author_categories_zip_list`, `generate_batch`, `train_step`, `eval_step`, `balance_dataset`, ...
- Test entry points / 测试入口: none
