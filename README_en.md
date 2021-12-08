<h3 align="center">ADBCMM  :Acronym Disambiguation
</h3>
<h3 align="center">
by Building Counterfactuals and Multilingual Mixing

</h3>

<hr>
<h4 align="center">
    <p>
        <b>English</b> |
        <a href="https://github.com/WENGSYX/ADBCMM/blob/master/README.md">简体中文</a> |
        <a href="https://github.com/WENGSYX/ADBCMM/blob/master/README_zh-hant.md">繁體中文</a>
    <p>
</h4>


<h3 align="center">
    <p>State-of-the-art of SUD@AAAI2022 AD task</p>
</h3>


#####         Scientific document around the world contains a large number of acronyms, these abbreviations affect researchers’ understanding of scientific document.  Therefore, [SDU@AAAI2022](https://sites.google.com/view/sdu-aaai22/) proposed the [Acronym Disambiguation (AD)](https://competitions.codalab.org/competitions/34899). The AD task gives a short text extracted from scientific literature and gives a dictionary containing multiple abbreviations—long sentences, each corresponding abbreviation in the dictionary contains 2-14 long sentences, requiring the most suitable long sentences for different phrases to be selected.

#####         This competition gives data sets in four different languages or fields, namely English (Legal), English (Science), French and Spanish. For English data sets, this article is not comparable (but using our ADBCMM method, the effect of the mdeberta-base model can still outperform the baseline performance of the deberta-xxlarge model on the English data set), this article focuses on low-resource data sets.

<center><img src="img/1.png" alt="img" style="zoom:50%;" /></center

#####             In this repository, the championship program code is stored in our SDU@AAAI2022 mission, you can reproduce our achievements based on this code, or you can read our paper to learn specific information.



## ADBCMM

#####         Because low-resource datasets are more difficult to learn relevant information than English data sets, either in the pre-training or fine-tuning phase, models are more likely to be biased and more inclined to select samples that are focused on training.Therefore, we chose to believe that additional data is needed to be enhanced to better generalization performance.

#####         The multilingual pre-training model gives us the possibility, after all, that the multilingual pre-training model itself is trained in the language library of different languages. Clearly, this model successfully learns the deeper relationships between different languages during pre-training. We also fine-tune this model using multilingual datasets in downstream tasks, which is a very simple idea.

#####         Specifically, in order to improve the performance of low-resource datasets, our course-based learning approach, based on a pre-training model, we first mixed up data sets in four different languages, and then on the related datasets in “micro-tuning”.

#####         In our experiments, we found that this method improved data tuning by 15 percent compared to using purely monolingual data!

<center><img src="img/3.png" alt="img" style="zoom:50%;" /></center>

## The Else Method

##### In this section, we will introduce the rest of the ways to improve the performance of the model.



### The model framework

#####         We used multiple selection models, and for each phrase, we would use "long sentence-[SEP]-abbreviation-[SEP]-phrase" as the output of the model, so that the model would choose the most likely one. In order to more parallel the training model, if the long sentence is less, we would use "padding" for filling, so that each batch, all 14 selected in the short text sample.

<center><img src="img/2.png" alt="img" style="zoom:50%;" /></center>

#####         Note: Baseline referred to above refers to the introduction of a pre-training model into this framework without the use of other methods mentioned in this article.



### Adversarial Learning and D-Drop

#####         It is well known that in the field of NLP, the use of adversarial learning or D-Drop can effectively increase the robustness of the model, resulting in a noticeable improvement of the effect. We also tried to use these two approaches and bring 1-5% improvement to our model.



### Pre-Training Language Models

#####         We used the [MDeberta-v3-base model](https://huggingface.co/microsoft/mdeberta-v3-base), which was trained in a CC100 dataset containing a hundred languages and should be considered the best multi-language pre-trained Encoder model, and the results were good.



## Results of Experiment

<center><img src="img/4.png" alt="img" style="zoom:50%;" /></center>

#####         BETO is a Spanish pre-training model, tested only on Spanish data in AD; Flaubert-base-cased is a French pre-training model, tested only on French data in AD; mDeberta is a multi-language pre-training model, we test in both French and Spanish. Additionally, methods including "ADBCMM'', "Child-Tuning", "R-Drop" and "Alls" are fine-tuned on mDeberta models, "Alls" refers to using all of the above methods. In addition to "Finally in Test", we test the results of the Dev series. "Finally in Test'' also uses model fusion to improve our performance.



## The Final Ranking

<center><img src="img/5.png" alt="img" style="zoom:50%;" /></center>

#### Our approach leads second to 4%-6%, which proves our approach is effective.



## The code



#### Last updated on December 9th, 21st.

#### The code is fixed.

