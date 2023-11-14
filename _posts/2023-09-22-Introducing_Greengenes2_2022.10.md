---
layout: post
title: "Introducing Greengenes2 2022.10"
date:  2023-09-22
tags: [生物信息工具]
comments: true
author: 一言
---

#### 介绍

很高兴地宣布Greengenes数据库的新版本，完全从头开始重新设计，以全基因组为支持，重点是协调16S rRNA和shotgun宏基因组数据集。由于设计上的重大变化，将新数据库命名为Greengenes2。然而，就像最初的Greengenes一样，它依赖于一个全新的系统发育，并表达了一个来自系统发育的分类学。

这篇论坛文章的重点是如何使用新的Greengenes2数据库。关于资源的设计和评估的细节可以在[预印本](https://www.biorxiv.org/content/10.1101/2022.12.19.520774v1)中找到。还提供了一个[交互式网站](https://greengenes2.ucsd.edu/)来探索数据库。原始文件在BSD-3许可下发布，并托管在一个公共匿名FTP上。还有一个用于与数据库交互的[QIIME 2插件](https://github.com/biocore/q2-greengenes2/)。

####  背景

Greengenes2是如何构建的?简单地说，Greengenes2依赖于Web of Life的第2版作为主干系统发育。Web of Life是一个基于381个标记基因的系统基因组树。然后使用uDance更新系统发育，其中包括来自Living Tree Project的330k个独特的高质量全长16S rRNA，来自全长核糖体operons(数据来自地球微生物组计划和人类粪便样本)和基因组分类数据库Genome Taxonomy Database。然后，使用DEPP将多余20M的16S rRNA V4 Deblur扩增子序列变体放入树中，该变体来自Qiita的30多万份公共和私人微生物组样本。最后，通过整合基因组分类数据库和Living Tree Project构建了一个分类法，并使用tax2tree将其添加到系统发育上。

结果是一个单一的系统发育和分类，适用于短读shotgun宏基因组学和16S rRNA测序研究。它允许直接整合16S rRNA和shotgun宏基因组数据集。此外，还保存了来自GTDB的分类管理，包括其多种标记。并且，对于集中于16S rRNA基因V4区的研究，可以直接从系统发育中得出分类，而不需要使用朴素贝叶斯，相对于朴素贝叶斯，这似乎产生了更高的分辨率结果。

#### 如何使用Greengenes2 ?

使用Greengenes2最简单的方法是通过[q2-greengenes2](https://github.com/biocore/q2-greengenes2/)。该插件提供了将数据与资源进行比较的操作。如何比较数据取决于所拥有的数据类型。我们将在下面介绍几个不同的场景，并提供具体的交互示例。

在此之前，先安装Greengenes2插件。

```shell
$ pip install q2-greengenes2
```

…下次运行“qiime”命令时，它将需要重新缓存其环境，这将花费几秒钟的时间。

在下面的例子中，将使用从Qiita获取数据，以此来演示使用Greengenes2的几种方法。Qiita和redbiom都不是必需的，它们只是一个数据源。在这一点上，让我们继续安装redbiom：

```shell
$ pip install redbiom
```

##### V4数据处理

Greengenes2包含超过20,000,000个16S rRNA V4扩增子测序片段，来自Qiita令人眼花缭乱的公共和私有微生物组样本，代表了非常大的环境类型横截面。选择V4片段，而不是其他可变区域，因为V4是Qiita的主要区域（注：在未来，正在考虑使用其他可变区域）。

如果你有V4数据，那么很有可能你的大部分测序数据已经由Greengenes2表示了——这极大地简化了与数据库的交互：只需要在你的`FeatureTable[Frequency]`和数据库中存在的内容之间执行一组交集!

让我们来看一个例子。为了获取数据，将依赖redbiom，它充当Qiita的缓存层，并提供命令行驱动的方式来获取`FeatureTables`和样本元数据`metadata`。你可以在[QIIME 2教程](https://forum.qiime2.org/t/querying-for-public-microbiome-data-in-qiita-using-redbiom)中了解更多关于redbiom的知识。

首先，将为redbiom设置一个“context”，它表示一组一致处理的样本(提示:您可以使用redbiom总结上下文找到上下文)。然后，将从Qiita下载[“极端”微生物组研究](https://pubmed.ncbi.nlm.nih.gov/27602409/)，这是研究[2136](https://qiita.ucsd.edu/study/description/2136)。最后，将数据导入QIIME 2:

 ```shell
$ ctx=Deblur_2021.09-Illumina-16S-V4-150nt-ac8c0b
$ redbiom fetch qiita-study \
>    --context $ctx \
>    --study-id 2136 \
>    --remove-blanks \
>    --output-basename icu
$ qiime tools import \
>    --input-path icu.biom \
>    --output-path icu.biom.qza \
>    --type FeatureTable[Frequency]
Imported icu.biom as BIOMV210DirFmt to icu.biom.qza
 ```

接下来，将针对Greengenes2过滤`FeatureTable[Frequency]`。为了做到这一点，将下载分类法的`Phylogeny[Rooted]`表示。在Greengenes2中，我们提供了两个经典的制表符分隔的分类法文件(.tsv)，以及树表示。树表示被大大压缩，简化了一些下游计算。

要下载的文件的名称是"2022.10.taxonomy.asv.nwk.qza"，表示它是“taxonomy”数据，feature IDs表示为实际扩增子序列变异(amplicon sequence variants)；“nwk”表示它内部是`NewickFormat`，这是一种表示树结构的方式。此外，我们将使用分类法的变体，将扩增子序列变体编码为ASVs本身(与MD5哈希相反)。我们使用“asv”表示的原因是，默认情况下，redbiom 将扩增子序列变体作为序列本身输出。

如果我的ASVs是*hash*的呢?简单!只需使用"2022.10.taxonomy.md5.nwk.qza"。

注意：`filter-features`命令现在需要大约8-10GB的内存。虽然从概念上讲，只是简单地取一组特征的交集，但数据集本身非常大——这是未来要优化的东西。

 ```
$ wget http://ftp.microbio.me/greengenes_release/2022.10/2022.10.taxonomy.asv.nwk.qza
$ qiime greengenes2 filter-features \
>     --i-feature-table icu.biom.qza \
>     --i-reference 2022.10.taxonomy.asv.nwk.qza \
>     --o-filtered-feature-table icu_gg2.biom.qza
Saved FeatureTable[Frequency] to: icu_gg2.biom.qza
 ```

```
source activate qiime2-2023.2
cd ~/test/gg2
qiime greengenes2 filter-features \
  --i-feature-table rep-seqs.qza \
  --i-reference 2022.10.taxonomy.asv.nwk.qza \
  --o-filtered-feature-table gg2_rep-seqs.qza
```


现在我们已经过滤了我们的表，可以收集所表示的扩增子序列变体的分类信息。

注意：就像过滤器功能一样，这个命令现在将需要大约8-10GB的内存。

```
$ qiime greengenes2 taxonomy-from-table \
>     --i-reference-taxonomy 2022.10.taxonomy.asv.nwk.qza \
>     --i-table icu_gg2.biom.qza \
>     --o-classification icu_gg2.taxonomy.qza
Saved FeatureData[Taxonomy] to: icu_gg2.taxonomy.qza
```

就像你已经用Greengenes2分类了你的序列数据!

##### 非V4数据处理

许多QIIME 2用户从其他可变区域生成扩增，这是有很好的理由的!一些用户可能有完整的16S rRNA序列，或者其他可能没有在已经放置的片段集中表示的长片段。对于这些情况，建议使用`non-v4-16s`操作，它将对Greengenes2中的全长16S序列使用q2-vsearch执行封闭参考OTU选择。

来试一下。正如在“V4”场景中所做的那样，我们将使用redbiom来获取数据集。在写这篇文章时，redbiom不会自动写出`FeatureData[Sequence]`，所以将使用biom命令提取该信息，然后用awk将其重新格式化为FASTA文件。然后我们将这些序列导入QIIME 2。注意：下面的命令假设运行了上面的`redbiom fetch qiita-study`命令块。

 ```shell
$ biom table-ids \
>     -i icu.biom \
>     --observations | \
>         awk '{ print ">" $1 "\n" $1 }' > icu.fna
$ qiime tools import \
>     --input-path icu.fna \
>     --output-path icu.fna.qza \
>     --type FeatureData[Sequence]
Imported icu.fna as DNASequencesDirectoryFormat to icu.fna.qza
 ```

回顾一下，现在有了来自icu.bio.qza文件的`FeatureTable[Frequency]`，刚刚生成了`FeatureData[Sequence]`。现在几乎拥有了纳入Greengenes2所需的所需数据。最后一个缺失的部分是我们需要下载主干(backbone)的16S rRNA序列。backbone代表Greengenes2中所有独特的全长16S rRNA序列:

 ```shell
$ wget http://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.full-length.fna.qza
 ```

既然我们已经完成了所有的输入，我们就可以开始Greengenes2操作。你也可以使用多个线程来指定这个动作，以使它运行得更快：

```
$ qiime greengenes2 non-v4-16s \
>    --i-table icu.biom.qza \
>    --i-sequences icu.fna.qza \
>    --i-backbone 2022.10.backbone.full-length.fna.qza \
>    --o-mapped-table icu.gg2.biom.qza \
>    --o-representatives icu.gg2.fna.qza
Saved FeatureTable[Frequency] to: icu.gg2.biom.qza
Saved FeatureData[Sequence] to: icu.gg2.fna.qza
```

现在我们已经将数据映射到Greengenes2，让我们对序列的分类进行分类!您可能会注意到，这个命令与我们对V4数据使用的命令相同，只是输入文件名略有不同：

```shell
$ qiime greengenes2 taxonomy-from-table \
>     --i-reference-taxonomy 2022.10.taxonomy.asv.nwk.qza \
>     --i-table icu.gg2.biom.qza \
>     --o-classification icu.gg2.taxonomy.qza
Saved FeatureData[Taxonomy] to: icu.gg2.taxonomy.qza
```

##### shotgun宏基因组数据

建议使用[Woltka 7工具包](https://journals.asm.org/doi/10.1128/msystems.00167-22)来处理短读数据，而不是使用Web of Life数据库的版本2(注意，在本文发布时，版本2还没有公开发布，但预计随时会上线)。在用Woltka处理你的短读数之后，你可以使用得到的`FeatureTable[Frequency]`和`q2-greengenes2`的`filter-features`命令。然后可以使用`taxonomy -from-table`操作生成`FeatureData[Taxonomy]`对象。

首先使用redbiom从Qiita获得Woltka处理的短读宏基因组。为此，需要使用之前不同处理获取宏基因组数据而非16S rRNA数据。让我们看看Woltka有哪些可用的上下文：

```shell
$ redbiom summarize contexts | grep -i woltka | cut -f 1,2,3 | column -t
Woltka-KEGG-Enzyme-WoLr2-4db94e    37367  3403
Woltka-per-genome-WoLr1-422c8e     35364  10052
Woltka-KEGG-Ontology-WoLr2-7dd29a  37367  9678
Woltka-KEGG-Pathway-WoLr2-58cdd3   37367  110
Woltka-per-genome-WoLr2-3ab352     38110  15326
Woltka-per-genome-WoLr1-1f9978     96     4109
```

正如您所看到的，有几个可能的选项可供选择。显示的第一列是样本ID，第二列是表示的样本数量，第三列是观察到的特征数量。当我们进行分类学评估时，我们希望使用含有"per-genome"文本。此外，我们将使用第二版的生命之网(“WoLr2”)，因为它支持Greengenes2：

```shell
$ ctx=Woltka-per-genome-WoLr2-3ab352
```

接下来，提取一些数据。在本教程中，使用 head 只抓取一小部分数据，然后将数据导入 QIIME 2：

```shell
$ redbiom fetch samples-contained --context $ctx | \
>     head | \
>     redbiom fetch samples \
>         --context $ctx \
>         --resolve-ambiguities merge \
>         --output woltka.example.biom
$ qiime tools import \
>     --input-path woltka.example.biom \
>     --output-path woltka.example.biom.qza \
>     --type FeatureTable[Frequency]
Imported woltka.example.biom as BIOMV210DirFmt to woltka.example.biom.qza
```

太好了！现在我们有一些猎枪样本。我们怎样才能得到分类？首先，我们将像上面一样使用`filter-features`进行过滤，因为一小组 WoLr2记录不在 Greengenes2中。然后，我们将使用`taxonomy-from-table`。下面的这些命令假设已经运行了上面命令块中提到的 Greengenes2数据的 wgets：

```shell
$ qiime greengenes2 filter-features \
>     --i-feature-table woltka.example.biom.qza \
>     --i-reference 2022.10.taxonomy.asv.nwk.qza \
>     --o-filtered-feature-table woltka_gg2.example.biom.qza
Saved FeatureTable[Frequency] to: woltka_gg2.example.biom.qza
$ qiime greengenes2 taxonomy-from-table \
>     --i-reference-taxonomy 2022.10.taxon
>     --i-reference-taxonomy 2022.10.taxonomy.asv.nwk.qza \
>     --i-table woltka_gg2.example.biom.qza \
>     --o-classification woltka_gg2.example.taxonomy.qza
Saved FeatureData[Taxonomy] to: woltka_gg2.example.taxonomy.qza
```

##### 双端测序数据

有两种选择。放置在Greengenes2中的片段只是正向序列，因此，如果您有V4数据，第一种选择是简单地忽略反向序列并且不合并序列。在这种情况下，您将遵循上面的“如果您有V4数据”步骤。第二个选项是使用非V4 -16s操作，如果您有V4或非V4数据，则可以使用该选项。在这种情况下，您将遵循上面的非v4数据步骤。

##### 可以用朴素贝叶斯进行分类吗

在Greengenes2的发布文件中，提供了与QIIME 2数据资源类似的用于朴素贝叶斯进行分类的V4区域和完整长度的分类器数据。

虽然我们确实提供了这些分类器，但通常建议对V4数据使用系统发生分类法(例如，`taxonomy-from-table`)。我们已经观察到，与朴素贝叶斯相比，使用全基因组测序数据在物种水平上每个样本的相关性得到了提高。

##### 系统发育学分析

一旦您的数据被Greengenes2过滤，通过上面提到的各种场景，您就可以利用`Phylogeny[Rooted]`来计算UniFrac、Faith’s PD或任何其他您想要的系统发育分析。关键的一步是获得系统发育对象。在上面的例子中，我们将asv表示为序列本身，而不是MD5散列序列，因此我们将获取以下树：

```shell
$ wget http://ftp.microbio.me/greengenes_release/current/2022.10.phylogeny.asv.nwk.qza
```

该树与`q2-diversity`兼容。

##### 使用Deblur或者DADA2有影响吗？

在运行非v4-16s操作时，数据是使用Deblur还是DADA2处理并不重要。根据所执行的上游处理类型，在使用filter-features操作时可能会产生影响。我们从Qiita插入的大部分数据都没有引物。如果引物仍然在序列上，则可能需要执行“左”修剪以使过滤器功能工作良好。类似地，放置在Greengenes2中的大多数asv的长度为90nt, 100nt或150nt，因此您最好的重叠选择是“正确”修剪片段到这些长度之一。

##### Greengenes2会持续更新吗？

会

##### 如何获取帮助？

目前，如果您对Greengenes2有任何疑问、担忧或评论，请给Daniel McDonald发电子邮件:d3mcdonald@eng.ucsd.edu。如果您对Greengenes2和QIIME 2的使用有任何疑问、担忧或评论，请在原帖回复。链接：https://forum.qiime2.org/t/introducing-greengenes2-2022-10/25291

#### 基于greengene2训练分类器及结果注释

```
# V4区分类器在官方文件中提供了，不需要再自己训练，如果是其他区段则可以参考以下命令进行操作：
qiime feature-classifier extract-reads \
    --i-sequences 2022.10.backbone.full-length.fna.qza \
    --p-f-primer ACTCCTACGGGAGGCAGCAG --p-r-primer GGACTACNVGGGTWTCTAAT \
    --p-read-orientation both \
    --o-reads 2022.10.backbone.v3v4.fna.qza \
    --p-n-jobs 4
# 2022.10.backbone.full-length.fna.qza在http://ftp.microbio.me/greengenes_release/2022.10/下载
# --p-f-primer 和 --p-r-primer参数需要根据你测序数据区段调整
qiime feature-classifier fit-classifier-naive-bayes \
    --i-reference-reads 2022.10.backbone.v3v4.fna.qza \
    --i-reference-taxonomy 2022.10.backbone.tax.qza \
    --o-classifier 2022.10.backbone.v3v4.nb.qza

# 新建工作目录
mkdir -p test/gg2 && cd test/gg2
# 获取V4分类器(新/旧)
wget http://ftp.microbio.me/greengenes_release/2022.10/2022.10.backbone.v4.nb.qza
wget https://docs.qiime2.org/2021.11/data/tutorials/moving-pictures-usage/gg-13-8-99-515-806-nb-classifier.qza

# 获取示例数据，以“Moving Pictures” tutorial中基于dada2生成的rep-seqs.qza和table.qza为测试数据
wget https://docs.qiime2.org/2021.11/data/tutorials/moving-pictures-usage/rep-seqs.qza
wget https://docs.qiime2.org/2021.11/data/tutorials/moving-pictures-usage/table.qza
wget https://docs.qiime2.org/2021.11/data/tutorials/moving-pictures-usage/sample-metadata.tsv

# 序列注释及可视化
conda activate qiime2-2023.2
time qiime feature-classifier classify-sklearn \
  --i-classifier 2022.10.backbone.v4.nb.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy-gg2.qza
# Saved FeatureData[Taxonomy] to: taxonomy-gg2.qza
# 
# real    4m43.991s
# user    4m30.594s
# sys     0m26.263s

qiime metadata tabulate \
  --m-input-file taxonomy-gg2.qza \
  --o-visualization taxonomy-gg2.qzv
# Saved Visualization to: taxonomy-gg2.qzv

qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy-gg2.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots-gg2.qzv
# Saved Visualization to: taxa-bar-plots-gg2.qzv

time qiime feature-classifier classify-sklearn \
  --i-classifier gg-13-8-99-515-806-nb-classifier.qza \
  --i-reads rep-seqs.qza \
  --o-classification taxonomy-gg-13-8.qza
# Saved FeatureData[Taxonomy] to: taxonomy-gg-13-8.qza
# 
# real    0m45.766s
# user    0m47.036s
# sys     0m10.040s

qiime metadata tabulate \
  --m-input-file taxonomy-gg-13-8.qza \
  --o-visualization taxonomy-gg-13-8.qzv
# Saved Visualization to: taxonomy-gg-13-8.qzv

qiime taxa barplot \
  --i-table table.qza \
  --i-taxonomy taxonomy-gg-13-8.qza \
  --m-metadata-file sample-metadata.tsv \
  --o-visualization taxa-bar-plots-gg-13-8.qzv
# Saved Visualization to: taxa-bar-plots-gg-13-8.qzv
```

















