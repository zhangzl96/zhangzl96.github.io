---
layout: post
title: "在Linux HPC集群上配置SMETANA与CPLEX 22.1.2的完整过程"
date:  2026-05-19
tags: [生物信息工具, CPLEX, SMETANA, 基因组代谢网络模型, 物种互作潜力, 代谢互养]
comments: true
author: 一言 & Gemini Pro
---

SMETANA 是一款用于分析微生物群落代谢相互作用的工具。在处理复杂的基因组尺度代谢模型时，使用商业求解器 CPLEX 能够显著提升计算效率。本文记录了在 Linux 高性能计算集群（HPC）上，基于 Python 3.12 和 CPLEX 22.1.2 版本从零开始配置 SMETANA 环境的完整流程，并提供了一种隔离环境变量的解决方案。

---

## 1. 安装 CPLEX 22.1.2

首先需要获取 IBM ILOG CPLEX Optimization Studio 的 Linux 安装包（通常为 `.bin` 格式）。

1. 赋予安装包执行权限并运行：
   ```bash
   chmod +x cplex_studio2212.linux_x86_64.bin
   ./cplex_studio2212.linux_x86_64.bin
   ```
2. 按照终端提示完成安装，输入 `1` 接受许可协议。
3. 记录安装路径。如果是普通用户权限，默认路径通常为 `~/ibm/ILOG/CPLEX_Studio2212`。
---

## 2. 创建 Python 3.12 虚拟环境

使用 Micromamba（或 Conda）创建一个独立的 Python 3.12 环境。

```bash
# 创建环境
micromamba create -n smetana_env python=3.12 -y

# 激活环境
micromamba activate smetana_env
```

---

## 3. 配置 CPLEX Python API (V22.1.2 新机制)

与旧版本（V22.1.1 及更早版本）不同，CPLEX V22.1.2 不再在安装目录下提供 `cplex/python` 文件夹，而是改用 pip 下载社区版后进行本地升级的机制。

在激活的虚拟环境中，执行以下命令：

```bash
# 安装基础接口
pip install cplex docplex

# 绑定本地完整版 CPLEX 以解除社区版的规模限制（请替换为实际路径）
docplex config --upgrade ~/ibm/ILOG/CPLEX_Studio2212
```

---

## 4. 安装 SMETANA 及其依赖

在 HPC 环境中，直接使用 pip 安装底层由 C/C++ 编写的依赖（如 `numpy`、`pandas`）容易触发系统的 GCC 编译报错或动态链接库版本过低的问题（如常见的 `CXXABI_1.3.9 not found`）。推荐先通过 conda-forge 渠道安装预编译的二进制包。

```bash
# 1. 安装核心科学计算包和较新的 C++ 标准库
micromamba install -c conda-forge pandas numpy reframed libstdcxx-ng -y

# 2. 依赖就绪后，使用 pip 安装纯 Python 编写的 SMETANA
pip install smetana
```

---

## 5. 配置环境变量隔离机制 (HPC Hooks)

为了让系统正确调用虚拟环境中的新版 C++ 库，同时不污染集群中其他生信分析流程的全局环境变量（避免引发 `Segmentation fault`），可以通过环境管理器自带的 Hooks 机制实现 `LD_LIBRARY_PATH` 的动态挂载与卸载。

依次执行以下命令创建自动化脚本：

```bash
# 进入环境配置目录
cd ~/micromamba/envs/smetana_env
mkdir -p ./etc/conda/activate.d
mkdir -p ./etc/conda/deactivate.d

# 创建激活脚本：备份旧变量并优先使用当前环境的 lib 目录
echo 'export OLD_LD_LIBRARY_PATH=$LD_LIBRARY_PATH' > ./etc/conda/activate.d/env_vars.sh
echo 'export LD_LIBRARY_PATH="$PWD/lib:$LD_LIBRARY_PATH"' >> ./etc/conda/activate.d/env_vars.sh

# 创建退出脚本：恢复旧变量并清理内存
echo 'export LD_LIBRARY_PATH=$OLD_LD_LIBRARY_PATH' > ./etc/conda/deactivate.d/env_vars.sh
echo 'unset OLD_LD_LIBRARY_PATH' >> ./etc/conda/deactivate.d/env_vars.sh
```

完成配置后，每次执行 `micromamba activate smetana_env` 时，环境变量会自动适配当前环境；退出环境时则自动恢复系统默认状态。

---

## 6. 软件测试与结果解读

### 6.1 验证 CPLEX 引擎挂载

在终端输入以下命令：

```bash
python -c "import cplex; c = cplex.Cplex(); print('CPLEX 版本:', c.get_version(), '挂载成功！')"
```

若正常输出版本号，则说明底层求解器已就绪。

### 6.2 运行测试数据

切换到 SMETANA 源码包中的 `tests/data` 目录（假设该目录下有官方提供的测试 `.xml` 模型），运行全局分析：

```bash
smetana -g --solver cplex tests/data/*.xml
```

*注：`-g` 参数代表运行较快的全局分析模式，`--solver cplex` 指定使用 CPLEX 求解器。*

继续进行详细的微观物种/代谢物级分析：

```bash
smetana -d --solver cplex tests/data/*.xml
```

### 6.3 结果指标解读

运行结束后，当前目录会生成`.tsv` 格式的结果文件。

SMETANA全局分析结果`global.tsv`：

| community | medium   | size | mip  | mro                |
| --------- | -------- | ---- | ---- | ------------------ |
| all       | complete | 2    | 0    | 0.5714285714285714 |

> mip：Metabolic Interaction Potential（代谢互作潜力）。0 表示这俩菌株在当前的 **complete** 培养基下，完全不需要靠互相交换代谢物就能活得很好（没有合作的必要）。
>
> mro：Metabolic Resource Overlap（代谢资源重叠度）。0.57 表示它们对培养基里营养物质的需求有 57% 是重合的，说明它们之间存在**中等偏强的竞争关系**。

SMETANA 详细分析结果`detailed.tsv`：

| community | medium  | receiver  | donor     | compound | scs  | mus  | mps  | smetana |
| --------- | ------- | --------- | --------- | -------- | ---- | ---- | ---- | ------- |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.1  | 1    | 0.1     |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.3  | 1    | 0.3     |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.2  | 1    | 0.2     |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.1  | 1    | 0.1     |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.2  | 1    | 0.2     |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.2  | 1    | 0.2     |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.2  | 1    | 0.2     |
| all       | minimal | ec_nh4_ko | ec_glc_ko | M_ac_e   | 1.0  | 0.1  | 1    | 0.1     |

> scs：物种耦合分数。衡量某一物种的生存对群落中另一物种的依赖程度（存在方向性）。这里1.0代表在当前的 `minimal`（基础/极简）培养基中，受体 `ec_nh4_ko` 对供体 `ec_glc_ko` 的依赖度是 100%。如果没有供体存在，受体会立刻死亡（无法在模型中求解出正常的生长速率）。
>
> mus：代谢物摄取分数。用于明确**受体**摄取特定物质的**概率/必需性**，以维持自身生长。
>
> mps：代谢物产生分数。用于明确**供体**产出该物质的概率。
>
> smetana：最终互养得分。由前三者计算出的综合得分（通常是连乘：`scs * mps * mus`）。分数越高，代表这条“具体的物质传输通道”在整个群落生态网络中越核心、越脆弱、越不可替代。

至此，适用于大规模微生物群落代谢网络分析的 SMETANA 运行环境配置完毕。

### 6.4 SMETANA设置培养条件参数（培养基）

SMETANA 针对 `-g` (宏观全局) 和 `-d` (微观详细) 模式，有着完全不同的默认培养基行为：

全局模式（你目前用的 -g）：如果你不加任何参数，SMETANA 默认会使用“完全培养基 (Complete Medium)”（即假设环境中存在所有营养物质，模型能够无限制吸收）。这就是为什么你之前看到 global.tsv 里的 medium 列显示为 complete，且 mip（代谢互作潜力）为 0 的原因——因为在营养极度丰富的环境里，微生物完全可以各自独活，不存在任何互作和依赖的必要。

详细模式（-d）：在此模式下，你不需要手动指定，SMETANA 底层会自动计算出能维持当前特定群落生存的群落特异性最小培养基 (Community-specific minimal media)，并在这种极限资源压力下计算交叉互养的细致得分。

如果你想强制模型在某种你自定义的最小培养基下运行（例如你明确想要测试某种限制性碳源环境），可以通过以下两种方式来实现：

#### 方法一：调用内置或模型预设的培养基

如果你使用的模型或者工具底层已经预设了经典的最小培养基（例如广泛使用的 `M9` 基础培养基），你可以直接用 `-m` 传入其名称：

```bash
smetana -g --solver cplex -m M9 tests/data/*.xml
```

#### 方法二：使用自定义的媒体库文件（最推荐的严谨做法）

在真实的科研分析中（特别是涉及土壤和植物根际的复杂环境），我们通常需要定义非常具体的无机盐和碳氮源成分。 你可以自己创建一个制表符（Tab）分隔的文本文件（例如命名为 `media_library.tsv`），里面定义好你的最小培养基中包含的具体代谢物 ID。然后使用 `-m` 结合 `--mediadb` 参数来调用它：

```bash
smetana -g --solver cplex -m my_minimal_media --mediadb media_library.tsv tests/data/*.xml
```

*小技巧：你可以一次性在 `-m` 后面用逗号隔开写入多个培养基（如 `-m M9,ARE`），SMETANA 会自动帮你把同一个群落放在不同培养基下分别跑一遍，这对于对比不同土壤营养条件下的群落互作网络非常有用。*

---

> **整理与技术支持**：Gemini Pro 大师