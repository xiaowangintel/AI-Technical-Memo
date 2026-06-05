# omniglot_loaders.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `functorch/examples/maml_omniglot/support/omniglot_loaders.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Provides runnable examples that demonstrate functorch transforms, compilation flows, and research-style use cases.
- **Purpose (CN)**: 提供可运行示例，展示 functorch 变换、编译流程以及研究型用例。
## Line-by-Line Analysis / 逐行分析

### Lines 1-13
```python
# Copyright (c) Facebook, Inc. and its affiliates.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
```
- **EN**: This comment block records assumptions, caveats, or usage guidance that frame the implementation below.
- **CN**: 这一段注释记录了后续实现所依赖的假设、注意事项或使用说明。

### Lines 15-29
```python
# These Omniglot loaders are from Jackie Loong's PyTorch MAML implementation:
#     https://github.com/dragen1860/MAML-Pytorch
#     https://github.com/dragen1860/MAML-Pytorch/blob/master/omniglot.py
#     https://github.com/dragen1860/MAML-Pytorch/blob/master/omniglotNShot.py

import errno
import os
import os.path

import numpy as np
from PIL import Image
from torchvision import transforms

import torch
import torch.utils.data as data
```
- **EN**: The import section wires together PyTorch-local modules such as torch, torch.utils.data; third-party modules such as numpy, PIL, torchvision; standard-library modules such as errno, os, os.path for the logic below.
- **CN**: 导入区把PyTorch 本地模块，如 torch、torch.utils.data；第三方模块，如 numpy、PIL、torchvision；标准库模块，如 errno、os、os.path组织在一起，供下方逻辑使用。

### Lines 32-49
```python
class Omniglot(data.Dataset):
    urls = [
        "https://github.com/brendenlake/omniglot/raw/master/python/images_background.zip",
        "https://github.com/brendenlake/omniglot/raw/master/python/images_evaluation.zip",
    ]
    raw_folder = "raw"
    processed_folder = "processed"
    training_file = "training.pt"
    test_file = "test.pt"

    """
    The items are (filename,category). The index of all the categories can be found in self.idx_classes
    Args:
    - root: the directory where the dataset will be stored
    - transform: how to transform the input
    - target_transform: how to transform the target
    - download: need to download the dataset
    """
```
- **EN**: It introduces or extends Omniglot, which hold the primary data model or public surface for this slice of the file. This chunk continues `Omniglot` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 它引入或扩展了 Omniglot，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `Omniglot`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 51-65
```python
    def __init__(self, root, transform=None, target_transform=None, download=False):
        self.root = root
        self.transform = transform
        self.target_transform = target_transform

        if not self._check_exists():
            if download:
                self.download()
            else:
                raise RuntimeError(
                    "Dataset not found." + " You can use download=True to download it"
                )

        self.all_items = find_classes(os.path.join(self.root, self.processed_folder))
        self.idx_classes = index_classes(self.all_items)
```
- **EN**: This chunk defines `__init__`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `__init__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 67-80
```python
    def __getitem__(self, index):
        filename = self.all_items[index][0]
        img = str.join("/", [self.all_items[index][2], filename])

        target = self.idx_classes[self.all_items[index][1]]
        if self.transform is not None:
            img = self.transform(img)
        if self.target_transform is not None:
            target = self.target_transform(target)

        return img, target

    def __len__(self):
        return len(self.all_items)
```
- **EN**: This chunk defines `__len__`, which implements one step in a functional transform, example, or package export flow. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `__len__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 82-94
```python
    def _check_exists(self):
        return os.path.exists(
            os.path.join(self.root, self.processed_folder, "images_evaluation")
        ) and os.path.exists(
            os.path.join(self.root, self.processed_folder, "images_background")
        )

    def download(self):
        import urllib
        import zipfile

        if self._check_exists():
            return
```
- **EN**: The import section wires together standard-library modules such as urllib, zipfile for the logic below. This chunk defines `download`, which loads external data or stored state into the active pipeline. Conditional branches preserve special cases and keep invariants explicit. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把标准库模块，如 urllib、zipfile组织在一起，供下方逻辑使用。 这一段定义了 `download`，其作用是把外部数据或已存储状态加载到当前流水线中。 条件分支用于保留特殊情况，并显式维护不变量。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 96-113
```python
        # download files
        try:
            os.makedirs(os.path.join(self.root, self.raw_folder))
            os.makedirs(os.path.join(self.root, self.processed_folder))
        except OSError as e:
            if e.errno == errno.EEXIST:
                pass
            else:
                raise

        for url in self.urls:
            print("== Downloading " + url)
            data = urllib.request.urlopen(url)
            filename = url.rpartition("/")[2]
            file_path = os.path.join(self.root, self.raw_folder, filename)
            with open(file_path, "wb") as f:
                f.write(data.read())
            file_processed = os.path.join(self.root, self.processed_folder)
```
- **EN**: This chunk continues `download` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Exception-handling structure makes cleanup or fallback behavior explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `download`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 异常处理结构使清理逻辑或回退行为保持明确。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 114-130
```python
            print("== Unzip from " + file_path + " to " + file_processed)
            zip_ref = zipfile.ZipFile(file_path, "r")
            zip_ref.extractall(file_processed)
            zip_ref.close()
        print("Download finished.")


def find_classes(root_dir):
    retour = []
    for root, dirs, files in os.walk(root_dir):
        for f in files:
            if f.endswith("png"):
                r = root.split("/")
                lr = len(r)
                retour.append((f, r[lr - 2] + "/" + r[lr - 1], root))
    print(f"== Found {len(retour)} items ")
    return retour
```
- **EN**: This chunk defines `find_classes`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `find_classes`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 133-150
```python
def index_classes(items):
    idx = {}
    for i in items:
        if i[1] not in idx:
            idx[i[1]] = len(idx)
    print(f"== Found {len(idx)} classes")
    return idx


class OmniglotNShot:
    def __init__(self, root, batchsz, n_way, k_shot, k_query, imgsz, device=None):
        """
        Different from mnistNShot, the
        :param root:
        :param batchsz: task num
        :param n_way:
        :param k_shot:
        :param k_query:
```
- **EN**: It introduces or extends OmniglotNShot, which hold the primary data model or public surface for this slice of the file. This chunk defines `__init__`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 OmniglotNShot，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `__init__`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 151-168
```python
        :param imgsz:
        """

        self.resize = imgsz
        self.device = device
        if not os.path.isfile(os.path.join(root, "omniglot.npy")):
            # if root/data.npy does not exist, just download it
            self.x = Omniglot(
                root,
                download=True,
                transform=transforms.Compose(
                    [
                        lambda x: Image.open(x).convert("L"),
                        lambda x: x.resize((imgsz, imgsz)),
                        lambda x: np.reshape(x, (imgsz, imgsz, 1)),
                        lambda x: np.transpose(x, [2, 0, 1]),
                        lambda x: x / 255.0,
                    ]
```
- **EN**: This chunk continues `__init__` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `__init__`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 169-184
```python
                ),
            )

            temp = {}  # {label:img1, img2..., 20 imgs, label2: img1, img2,... in total, 1623 label}
            for img, label in self.x:
                if label in temp:
                    temp[label].append(img)
                else:
                    temp[label] = [img]

            self.x = []
            for (
                label,
                imgs,
            ) in temp.items():  # labels info deserted , each label contains 20imgs
                self.x.append(np.array(imgs))
```
- **EN**: This chunk continues `__init__` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `__init__`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 186-199
```python
            # as different class may have different number of imgs
            self.x = np.array(self.x).astype(
                np.float64
            )  # [[20 imgs],..., 1623 classes in total]
            # each character contains 20 imgs
            print("data shape:", self.x.shape)  # [1623, 20, 84, 84, 1]
            temp = []  # Free memory
            # save all dataset into npy file.
            np.save(os.path.join(root, "omniglot.npy"), self.x)
            print("write into omniglot.npy.")
        else:
            # if data.npy exists, just load it.
            self.x = np.load(os.path.join(root, "omniglot.npy"))
            print("load from omniglot.npy.")
```
- **EN**: It introduces or extends may, which hold the primary data model or public surface for this slice of the file. This chunk continues `may` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 它引入或扩展了 may，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `may`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 201-215
```python
        # [1623, 20, 84, 84, 1]
        # TODO: can not shuffle here, we must keep training and test set distinct!
        self.x_train, self.x_test = self.x[:1200], self.x[1200:]

        # self.normalization()

        self.batchsz = batchsz
        self.n_cls = self.x.shape[0]  # 1623
        self.n_way = n_way  # n way
        self.k_shot = k_shot  # k shot
        self.k_query = k_query  # k query
        if (k_shot + k_query) > 20:
            raise AssertionError(
                f"k_shot + k_query must be <= 20, got {k_shot + k_query}"
            )
```
- **EN**: This chunk continues `may` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `may`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 217-230
```python
        # save pointer of current read batch in total cache
        self.indexes = {"train": 0, "test": 0}
        self.datasets = {
            "train": self.x_train,
            "test": self.x_test,
        }  # original data cached
        print("DB: train", self.x_train.shape, "test", self.x_test.shape)

        self.datasets_cache = {
            "train": self.load_data_cache(
                self.datasets["train"]
            ),  # current epoch data cached
            "test": self.load_data_cache(self.datasets["test"]),
        }
```
- **EN**: This chunk continues `may` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `may`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 232-247
```python
    def normalization(self):
        """
        Normalizes our data, to have a mean of 0 and sdt of 1
        """
        self.mean = np.mean(self.x_train)
        self.std = np.std(self.x_train)
        self.max = np.max(self.x_train)
        self.min = np.min(self.x_train)
        # print("before norm:", "mean", self.mean, "max", self.max, "min", self.min, "std", self.std)
        self.x_train = (self.x_train - self.mean) / self.std
        self.x_test = (self.x_test - self.mean) / self.std

        self.mean = np.mean(self.x_train)
        self.std = np.std(self.x_train)
        self.max = np.max(self.x_train)
        self.min = np.min(self.x_train)
```
- **EN**: This chunk defines `normalization`, which implements one step in a functional transform, example, or package export flow. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `normalization`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 249-260
```python
    # print("after norm:", "mean", self.mean, "max", self.max, "min", self.min, "std", self.std)

    def load_data_cache(self, data_pack):
        """
        Collects several batches data for N-shot learning
        :param data_pack: [cls_num, 20, 84, 84, 1]
        :return: A list with [support_set_x, support_set_y, target_x, target_y] ready to be fed to our networks
        """
        #  take 5 way 1 shot as example: 5 * 1
        setsz = self.k_shot * self.n_way
        querysz = self.k_query * self.n_way
        data_cache = []
```
- **EN**: This chunk defines `load_data_cache`, which loads external data or stored state into the active pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `load_data_cache`，其作用是把外部数据或已存储状态加载到当前流水线中。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 262-278
```python
        # print('preload next 50 caches of batchsz of batch.')
        for sample in range(10):  # num of episodes
            x_spts, y_spts, x_qrys, y_qrys = [], [], [], []
            for i in range(self.batchsz):  # one batch means one set
                x_spt, y_spt, x_qry, y_qry = [], [], [], []
                selected_cls = np.random.choice(data_pack.shape[0], self.n_way, False)

                for j, cur_class in enumerate(selected_cls):
                    selected_img = np.random.choice(
                        20, self.k_shot + self.k_query, False
                    )

                    # meta-training and meta-test
                    x_spt.append(data_pack[cur_class][selected_img[: self.k_shot]])
                    x_qry.append(data_pack[cur_class][selected_img[self.k_shot :]])
                    y_spt.append([j for _ in range(self.k_shot)])
                    y_qry.append([j for _ in range(self.k_query)])
```
- **EN**: This chunk continues `load_data_cache` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `load_data_cache`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 280-296
```python
                # shuffle inside a batch
                perm = np.random.permutation(self.n_way * self.k_shot)
                x_spt = np.array(x_spt).reshape(
                    self.n_way * self.k_shot, 1, self.resize, self.resize
                )[perm]
                y_spt = np.array(y_spt).reshape(self.n_way * self.k_shot)[perm]
                perm = np.random.permutation(self.n_way * self.k_query)
                x_qry = np.array(x_qry).reshape(
                    self.n_way * self.k_query, 1, self.resize, self.resize
                )[perm]
                y_qry = np.array(y_qry).reshape(self.n_way * self.k_query)[perm]

                # append [sptsz, 1, 84, 84] => [b, setsz, 1, 84, 84]
                x_spts.append(x_spt)
                y_spts.append(y_spt)
                x_qrys.append(x_qry)
                y_qrys.append(y_qry)
```
- **EN**: This chunk continues `load_data_cache` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `load_data_cache`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 298-311
```python
            # [b, setsz, 1, 84, 84]
            x_spts = (
                np.array(x_spts)
                .astype(np.float32)
                .reshape(self.batchsz, setsz, 1, self.resize, self.resize)
            )
            y_spts = np.array(y_spts).astype(int).reshape(self.batchsz, setsz)
            # [b, qrysz, 1, 84, 84]
            x_qrys = (
                np.array(x_qrys)
                .astype(np.float32)
                .reshape(self.batchsz, querysz, 1, self.resize, self.resize)
            )
            y_qrys = np.array(y_qrys).astype(int).reshape(self.batchsz, querysz)
```
- **EN**: This chunk continues `load_data_cache` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `load_data_cache`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 313-330
```python
            x_spts, y_spts, x_qrys, y_qrys = (
                torch.from_numpy(z).to(self.device)
                for z in [x_spts, y_spts, x_qrys, y_qrys]
            )

            data_cache.append([x_spts, y_spts, x_qrys, y_qrys])

        return data_cache

    def next(self, mode="train"):
        """
        Gets next batch from the dataset with name.
        :param mode: The name of the splitting (one of "train", "val", "test")
        :return:
        """
        # update cache if indexes is larger cached num
        if self.indexes[mode] >= len(self.datasets_cache[mode]):
            self.indexes[mode] = 0
```
- **EN**: This chunk defines `next`, which implements one step in a functional transform, example, or package export flow. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `next`，其作用是实现函数式变换、示例或包导出流程中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 331-336
```python
            self.datasets_cache[mode] = self.load_data_cache(self.datasets[mode])

        next_batch = self.datasets_cache[mode][self.indexes[mode]]
        self.indexes[mode] += 1

        return next_batch
```
- **EN**: This chunk continues `next` and expands its control flow, data preparation, or emitted structure. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `next`，继续展开其控制流、数据准备或生成结构。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Functional tensor transforms**
  - EN: Applies composable transforms such as batching, gradients, and tracing.
  - CN: 应用可组合的变换，例如 batching、梯度和 tracing。
- **Jacobian/Hessian transforms**
  - EN: Works with Jacobian- or Hessian-style higher-order differentiation APIs.
  - CN: 处理 Jacobian/Hessian 风格的高阶微分 API。
- **Omniglot**
  - EN: `Omniglot` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `Omniglot` 是本文件声明、导出或驱动的显著符号之一。
- **__init__**
  - EN: `__init__` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `__init__` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch`, `torch.utils.data`
- **Third-party modules / 第三方模块**: `numpy`, `PIL`, `torchvision`
- **Standard library / 标准库**: `errno`, `os`, `os.path`, `urllib`, `zipfile`
- **Primary symbols / 核心符号**: `Omniglot`, `__init__`, `__getitem__`, `__len__`, `_check_exists`, `download`, `find_classes`, `index_classes`, `OmniglotNShot`, `normalization`
