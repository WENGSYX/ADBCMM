<h3 align="center">
    <big><big><strong>ADBCMM</strong></big></big>  :Acronym Disambiguation
</h3>
<h3 align="center">
by Building Counterfactuals and Multilingual Mixing
</h3>
<h4 align="center">
    通过多语言中混合训练和构建反事实样本 提升缩略词消歧性能
</h4>
<hr>

<h4 align="center">
    <p>
        <a href="https://github.com/WENGSYX/ADBCMM/blob/master/README_en.md">English</a> |
        <b>简体中文</b> |
        <a href="https://github.com/WENGSYX/ADBCMM/blob/master/README_zh-hant.md">繁體中文</a>
    <p>
</h4>

<h3 align="center">
    <p>SDU@AAAI2022 缩略词消歧任务第一名</p></h3>


####                由于世界各国的科学文献中都包含了大量的首字母缩略词，但这些缩略词会阻碍研究人员阅读科学文献。因此[SDU@AAAI2022](https://sites.google.com/view/sdu-aaai22/) 提出了 [缩略词消歧任务（AD）](https://competitions.codalab.org/competitions/34899)。AD任务给定一篇从科学文献中截取的短文，并给定一份含多个缩略语—长句的字典，字典中每个对应的缩略语均含有2-14条长句，要求针对不同的短语，选出其中最合适的长句。

####               这个比赛给了四个不同语言或领域的数据集，分别是英语（法律），英语（科学），法语和西班牙语。对于英语数据集，本文不作对比（但使用我们的ADBCMM方法，mdeberta-base模型的效果在英语数据集上仍然能超过deberta-xxlarge模型的baseline性能），本文重点放在低资源数据集中。

<center><img src="img/1.png" alt="img" style="zoom:50%;" /></center>



####        在本存储库中，保存着我们在SDU@AAAI2022 任务中 冠军方案代码，您可基于本代码复现我们的成绩，或者可阅读我们的论文，了解具体信息。



## ADBCMM

#####         由于低资源数据集无论是预训练阶段还是微调阶段，相比起英语数据集，都更加难以学习到相关的信息,模型很容易造成偏见，更加偏向于选择训练集中存在的样本。因此我们选择认为想要获得更好的泛化性能，就需要额外的数据进行增强。但这些数据从哪来呢？总不能人工标注吧？

#####         因此，我们做了一个假设，像是法语和西班牙语这种数据集，在几千年前应该是源于同一种语言，这些语言在语法和结构上，应当存在着一些深层次的关系。因此能否让模型从不同语言的数据集中，都学到缩略词消歧的深层含义呢？

#####         多国语言预训练模型为我们提供了可能，毕竟多国语言预训练模型本身，就是在不同语言的语料库中进行训练的。显然，这个模型预训练时成功学会到不同语言之间深层次的关系。我们对这个模型也同样在下游任务中使用多国语言数据集进行微调，这是一个十分简单的想法。

#####                 具体来说，为了提升低资源数据集的性能，我们基于课程学习的方法，在预训练模型的基础上，我们首先对四个不同语言的数据集混合训练，之后再在相关的数据集上”微微调“。实验发现，这种方法，最多能比纯粹使用单语种数据微调，效果提升接近15%！这是十分惊人的。

<center><img src="img/3.png" alt="img" style="zoom:50%;" /></center>







## 其余方法



#### 模型框架

#####                 我们使用了多项选择模型，对于每一条短文，我们逐一将“长句-[SEP]-缩略词-[SEP]-短文“作为模型的输出，让模型选择最有可能的一条。为了更加并行化得训练模型，如果长句较少，我们使用"padding"进行填充，使每个batch，均在14条该短文样本中进行选择。

<center><img src="img/2.png" alt="img" style="zoom:50%;" /></center>

 ##### 注意：上下文所称Baseline，均指将预训练模型放入此框架中，而不使用本文所提及的其他方法。



#### 对抗学习与D-Drop

#####         众所周知，在NLP领域，使用对抗学习或D-Drop，能有效增加模型的鲁棒性，带来明显的效果提升。我们也尝试使用了这两种方法，并为我们的模型带来1-5%的提升。



#### Child-Tuning

#####         这是一种比较新颖的方法，在训练过程中，只微调小部分的权重（对于小部分权重的选择，有两种做法：Child-Tuning-D是任务无关做法，直接在伯努利分布中选择需要微调的权重；Child-Tuning-F是任务相关做法，在第一轮中使用全参数微调模型，之后会进行搜索，查找整个微调过程中，变化最大的那一批权重，在后面几轮的训练中，都只会对变化大的那一部分的权重进行微调，其余的保持不变。一般来说，Child-Tuning-F的效果好一些，但是时间会比较久，建议按需选择）。



#### 预训练模型

#####          鲁迅说过，不会用预训练模型的NLPer不是好NLPer。

#####          我们使用了[MDeberta-v3-base模型](https://huggingface.co/microsoft/mdeberta-v3-base)，这个模型在包含一百种语言的CC100数据集中进行了训练，应该算得上是最好的多语言预训练Encoder模型，效果是挺好的。



## 实验结果

<center><img src="img/4.png" alt="img" style="zoom:50%;" /></center>

#####         这是消融实验与对比实验的表格，头三栏是三个模型的baseline效果，BETO是西班牙语预训练模型，Flaubert-base-cased是法语预训练模型。mDeberta-v3-base我们也是在单语种中做了对比实验。由表可见，mDeberta-v3-base的如果论单语种微调的性能，其实还是远不如只在单个语种中进行预训练的另外两个模型。

#####         不过，如果加上我们的ADBCMM，也就是使用四份数据集，先进行训练，之后再在单语种中训练，这能大幅提升模型的效果。其中，”ALLs“表示单模型，使用所有方法在dev集中达到的最佳成绩。”Finally in Test“，使用了多个模型进行融合，其中包括五折融合/随机融合/加权融合在内的诸多融合策略，达到了最佳的效果。



## 最终排名

##### 虽然在本文开头就已经强调了一遍，但还是想再具体展示下：

<center><img src="img/5.png" alt="img" style="zoom:50%;" /></center>

#### 整整比第二名领先4-6%的F1，充分证明了我们的方法具有优越性与领先性。





## 复现代码

#### 最后更新日期：21年12月8日  剩余内容稍待片刻


## 论文引用
> @misc{weng2021adbcmm,
>       title={ADBCMM : Acronym Disambiguation by Building Counterfactuals and Multilingual Mixing}, 
>       author={Yixuan Weng and Fei Xia and Bin Li and Xiusheng Huang and Shizhu He and Kang Liu and Jun Zhao},
>       year={2021},
>       eprint={2112.08991},
>       archivePrefix={arXiv},
>       primaryClass={cs.CL}
> }


