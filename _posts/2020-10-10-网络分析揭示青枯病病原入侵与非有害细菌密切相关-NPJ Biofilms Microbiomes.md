---
layout: post
title: "NPJ Biofilms Microbiomes | 网络分析揭示青枯病病原入侵与非有害细菌密切相关 | 2020"
date:  2020-10-10
tags: [文献分享]
comments: true
author: YiYan
---

> 标题：Network analysis infers the wilt pathogen invasion associated with non-detrimental bacteria
>
> 单位：中国科学院生态环境研究中心
>
> 日期：2020年2月14日
>
> 作者：胡秋龙 / 邓晔

### 摘要

**背景：**微生物在根围土壤和根系内生中定殖和迁移，对植物的生长和健康起着重要的作用，但我们目前对于参与抑制或协助病原体入侵的关键微生物物种依然知之甚少。

**方法和结果：**本研究利用16S高通量测序和分子生态网络方法（MENA），系统研究了青枯病入侵过程中植物内生（茎和根）菌群与根围土壤（根际和根周边）菌群中的物种互作网络。通过对微生物多样性和群落结构组成的分析，发现植物内生菌群受病原菌入侵的影响会发生显著性改变。根据物种溯源分析（SourceTracker），受青枯病原菌感染的植物内生细菌群落组成主要来源于周边土壤，但健康植物中的内生细菌群落几乎无法从土壤群落中追溯。分子生态网络分析结果表明，受青枯病原菌感染的植物土壤和内生微生物群落均呈现出拓扑特征结构，并较健康植物具有更复杂的网络结构和更多的相互作用关系。此外，微生物物种之间的互作网络结果表明，青枯病原菌*Ralstonia*与受感染植物根内生中的微生物物种（*Delftia*, *Stenotrophomonas*, *Bacillus,* *Clostridium XlVa*, *Fontibacillus*, *Acidovorax*, *Herminiimonas*和3个未分类的细菌属）呈正相关互作关系。

**结论：**青枯病原菌在根际和根系内生的入侵可能与通常对植物无害甚至有益的某些功能细菌密切相关。

### 材料与方法

本研究以茄科植物-**烟草**为研究对象，共采集80个样品，包括健康和受青枯病严重感染的植株（5-9级）和土壤样品：（健康 / 感病）植株的（根周边土壤（HBS）/ 根际土壤（HRS）/ 根（HR）/ 茎（IS））各10个。样品进行16S rRNA基因的高通量测序，并采用微生物溯源分析（http://mem.rcees.ac.cn:8080）和分子生态网络分析（http://ieg2.ou.edu/MENA/）揭示青枯病原菌关键迁移节点和微生物互作关系。

### 主要结果

#### 01. 健康和感染样本的细菌多样性和群落组成
根据多样性指数Chao1和PD（图1），健康和受青枯病感染的植株根周边和根际土壤的细菌群落多样性无显著差异。然而，受青枯病感染的根和茎样品中内生细菌多样性均显著高于健康的根和茎样品（P < 0.01）。

<img src="https://raw.githubusercontent.com/zhangzl96/zhangzl96.github.io/master/images/41522_2020_117_Fig1_HTML.png" alt="41522_2020_117_Fig1_HTML" style="zoom:25%;" />

> **图1 基于16S rRNA测序的根周边土壤、根际土壤、根、茎微生物群落多样性**

所有OTUs归属于828属（土壤，730; 内生，242），归属于28门（土壤，28; 内生, 13）。土壤和内生样品中相对丰度排名前10的OTUs （相对丰度≥1.0%）见图2a，植物内生细菌群落中相对丰度排名前20个属（相对丰度≥1.0%）见图2b。结果表明，*Arthrobacter, Acinetobacter, Massilia, Sphingomonas, Falsibacillus, Bradyrhizobium, Rhodanobacte, Sphingobium, Gaiella* 和 *Terrabacter*在根周边和根际土壤细菌群落中占优势地位，但这些属的相对丰度在内生菌群中明显减少或消失。此外，一些细菌属的相对丰度也因青枯病的入侵而发生显著变化。*Ralstonia*（病原菌）、*Chryseobacterium, Rhodococcus, Burkholderia, Noviherbaspirillum, Acinetobacter, Sphingomonas, Falsibacillus, Bradyrhizobium* 在受青枯病感染的根周边和根际土壤中的相对丰度均高于健康土壤。在植物内生菌群中（图2b），感染样品中*Ralstonia、Stenotrophomonas、Paenibacillus、Achromobacter*和*Rhizobium* 的相对丰度较健康样品中显著增加，然而，公认的植物有益菌*Pseudomonas, Bacillus* 和 *Falsibacillus* 的相对丰度较健康样品中显著下降。

<img src="https://raw.githubusercontent.com/zhangzl96/zhangzl96.github.io/master/images/41522_2020_117_Fig2_HTML.png" alt="41522_2020_117_Fig2_HTML" style="zoom:25%;" />

> **图2 健康和感染样本的土壤和植物内生群落结构组成属水平上的比较**
>
> (a) 健康和受感染的根周边土壤、根际土壤、根、茎样品中排名前10优势OTUs的相对丰度；
>
> (b) 健康和受感染的根和茎样本样品中排名前20优势OTUs的相对丰度。


#### 02. 从土壤到内生细菌群落的溯源分析

利用微生物溯源(SourceTracker)来研究从土壤到内生细菌群落的迁移比例。根据源解析结果，受感染和健康植物内生细菌群落组成源存在显著差异。在健康样品中(图3)，根际土壤细菌群落组成绝大多数(74.98%)来自于根周边土壤，而内生细菌群落几乎无法从土壤群落中追溯，说明健康植物的内部和外部细菌群落有着明确的界限。感染样品中(图3)，根际土壤细菌群落主要来源于根周边土壤(71.4%)，茎内生细菌群落主要来源于根内生(94.9%)。更重要的是，大量的根内生细菌群落组成来自于根周边 (50.1%)和根际(11.7%)土壤，这表明大多数受感染植物内生细菌群落可从土壤中追溯。

<img src="https://raw.githubusercontent.com/zhangzl96/zhangzl96.github.io/master/images/41522_2020_117_Fig3_HTML.png" alt="41522_2020_117_Fig3_HTML" style="zoom:25%;" />

> **图3 健康(左)和感染(右)样品的微生物溯源（SourceTracker）分析结果**


#### 03. 土壤和内生群落的分子生态网络分析

利用分子生态网络分析(MENA)揭示土壤和内生细菌群落之间的相互作用。受感染的植物土壤和内生细菌群落网络结构 (图4 b, d)较健康的土壤和内生细菌(图4 a，c)呈现出更高的复杂性和更多的链接。在受感染的土壤和内生细菌群落中，呈现出了致病菌Ralstonia和其他物种之间的交互网络关系，而在健康样品中没有呈现。此外，在受感染的内生细菌群落网络结构中，致病菌Ralstonia节点(9个)和其它细菌物种之间的链接（38条）显著多于受感染的土壤网络(1个节点和12条链接)，这表明，与受感染的土壤相比，受感染的内生细菌菌落中致病菌Ralstonia与其他物种之间的互作关系更多，这可能对确定在青枯病入侵过程中病原菌从土壤向内生群落的迁移起着重要作用。

基于上述结果，我们对受感染的根和茎的内生细菌群落的分子生态网络进一步地深入分析，以验证哪个植株部位在青枯病入侵过程中发挥了更重要的作用。受感染的根(图4f)的内生细菌群落比受感染的茎(图4e)具有更复杂和更高度链接的细菌群落。此外, 相比较于感染茎(4节点, 6链接)的网络结构, 在受感染的根内生网络中具有更多的致病菌Ralstonia节点 (8)和更多的Ralstonia与其他微生物物种之间的链接(69)，这揭示了青枯病原菌入侵可能和植物根系内生中丰富的病原菌与其他微生物物种互作关系密切相关。

<img src="https://raw.githubusercontent.com/zhangzl96/zhangzl96.github.io/master/images/41522_2020_117_Fig4_HTML.png" alt="41522_2020_117_Fig4_HTML" style="zoom:25%;" />

> **图4 健康与感染样品细菌群落中微生物之间互作的分子生态网络图**
>
> (a) 健康样品中土壤群落的细菌网络图(根周边+根际土壤)；
>
> (b) 受感染样品中土壤群落的细菌网络图(根周边+根际土壤)；
>
> (c) 健康样品中内生细菌群落的网络图(根+茎)；
>
> (d) 受感染样品中内生细菌群落的网络图(根+茎)；
>
> (e) 受感染茎内生细菌群落的网络图；
>
> (f) 受感染根内生细菌群落的网络图。
>
> 每个节点的颜色代表一个门级的微生物物种，在属水平上标记了病原菌*Ralstonia*。蓝色链接表示节点之间的正相关关系，红色链接表示负相关关系。


#### 04. 根系内生中致病菌*Ralstonia*和其他微生物物种之间的相互作用网络

为揭示关键内生微生物可能在协助或抑制青枯病病原爆发中发挥重要作用，我们进一步分析了受感染的根内生分子生态网络中致病菌*Ralstonia*与其他微生物物种之间的互作关系，以推断关键微生物物种的作用。如图5所示，致病菌(*Ralstonia*)与多个细菌属呈负相关互作关系，包括*Pigmentiphaga, Bosea, Variovorax, Sphingobacterium, and 1 unclassified bacteria (family: Enterobacteriaceae)，*但与*Delftia, Stenotrophomonas, Bacillus, Clostridium XlVa, Fontibacillus, Acidovorax, Herminiimonas和3 unclassified bacterial genera (family:2 Burkholderiaceae, Rhizobiales)* 呈正相关互作关系。这些与致病菌*Ralstonia* 呈正相关和负相关互作关系的细菌可能在协助和抑制青枯病爆发过程中发挥重要作用。我们的结果表明，在青枯病入侵植物过程中，植物根内的原生细菌可能通过与病原菌的互作互惠关系协助病原菌定植和/或富集，从而帮助青枯病原菌入侵。

<img src="https://raw.githubusercontent.com/zhangzl96/zhangzl96.github.io/master/images/41522_2020_117_Fig5_HTML.png" alt="41522_2020_117_Fig5_HTML" style="zoom: 50%;" />

> **图5 受感染植物根中病原菌(红色)与其他微生物物种之间互作的分子生态网络图**
>
> 每个节点都标记为属水平平。蓝色链接表示节点之间的正相关互作关系，红色链接表示负相关互作关系。

### 结论

综上所述，我们推断致病菌*Ralstonia*的入侵可能与非有害细菌（包括*Delftia, Stenotrophomonas, Bacillus, Clostridium XlVa, Fontibacillus, Acidovorax, Herminiimonas* 和3个高度疑似的根瘤菌）的互作关系密切相关，这些非有害细菌与病原菌建立互惠关系，可能导致大量的的细菌菌群从根周边土壤向植物根和茎内生中迁移，从而最终导致植物青枯病的暴发。本研究将为防控植物青枯病的发生提供潜在的新思路和理论依据。



#### 参考文献：

Qiulong Hu, Lin Tan, Songsong Gu, Yansong Xiao, Xingyao Xiong, Wei-ai Zeng, Kai Feng, Zhong Wei, Ye Deng. Network analysis infers the wilt pathogen invasion associated with non-detrimental bacteria[J]. NPJ Biofilms and Microbiomes, 2020. DOI: 10.1038/s41522-020-0117-2

#### 原文链接

[https://www.nature.com/articles/s41522-020-0117-2](https://www.nature.com/articles/s41522-020-0117-2)










