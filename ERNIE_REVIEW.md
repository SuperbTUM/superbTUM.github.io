---
title: "Literature Review of ERNIE 2.0, Baidu, 2020"
permalink: /ERNIE_REVIEW/
layout: page
---

# Literature Review of ERNIE 2.0



Declaration: There are two proposals with same abbreviation ERNIE. To separate them, I use ERNIE THU and ERNIE Baidu to represent different papers.



In a survey of deep learning in named entity recognition (NER), CNN and RNN have been applied to do the task. The drawback is CNN and RNN are not able to handle long-term dependency, while CNN performs worse and time-costly since it has a bunch of filters (but it can be parallelized). Overall, the mainstream solution for NLP problem is LSTM and transformer. 



A very common proposal for NER is to use BiLSTM with CRF. CRF is the abbreviation of condition random field. It is widely used in image segmentation in fully connected layer and semantic analysis, both of which need context. In the linear chain version, it takes four parameters: sentence, word location i, scores of current (i) and previous (i-1) semantic prediction. The score is either 1 or 0. It is based on hidden Markov model with wider context and flexible weights. With such strategy, there should be a huge improvement on the F1 score (considers precision and recall). It is commonly used as the tag encoder in the model.



One of the drawbacks of CRF is that it can't leverage segment information since CRF is based on word-level representations. One optimization is gated recursive semi-markov CRF, with 0.2% improvement compared with baseline (BiLSTM+CRF).



In 2018, BERT was came up by Google and immediately went viral in the recent years. BERT is based on transformers raised in 2017. To optimize BERT, apart from deploying novel masking method which will be introduced accordingly, we can modify the way of training (learning rate, warm-up, etc). Since the original BERT has limit in length of sequence, there should be a proposal of breaking the boundary.



The most popular proposal is pre-trained BERT with fine-tuning. The fine-tuning with BERT shows the way of input embedding. In ERNIE THU, it uses two kinds of encoders, token and knowledge. In token encoder, the embedding is based on token input while in knowledge encoder, the embedding is based on entity input. The model first goes through the token encoder and then goes through knowledge encoder. The results will be stacked and go through an *Information Fusion* part to extract to two outputs. The information fusion is a kind of multi-layer perception (FCN) and the activation function is GELU.



There is another novel optimization of BERT, multi-task learning. This concept is implemented by both DT-CNN from Microsoft and ERNIE from Baidu. The latter one will be introduced in detail in the following paragraph. DT-CNN leverages three pre-trained tasks and trains them with SGD. There are three losses and multi-task training means back propagation is based on the average loss of the three tasks.



In another work: Cloze-driven pretraining in self-attention neural network, it made its attempt to modify the transformer by switching the layer normalization to before the self-attention module rather than after (normally this is placed between self-attention and FFN). The paper also proposed that when fine tuning the downstream task, it's better not to do masking in order to provide more context information.



You can also think of the masking strategy. The initial strategy is to sample once, 15~20% tokens are available for masking, in which 10% remain immutable. RoBERTa leverages a dynamic masking approach by duplicating the dataset several times and assigns different masking.



In recent two years, most of the proposals had better scores/rankings in the SIGHAN 2006 dataset challenge with transformers, where ERNIE from Baidu ranked top-5. ERNIE base uses a basic transformer mentioned from *Attention* paper as the encoder. Base model has 12 encoders, 768 hidden layers and 12 attention heads with no shared weights. Through the transformer, each embedded token will become contextual embeddings. It seems that both the sequence loss and token loss are cross entropy loss, with separate backward propagation? One model takes up multiple tasks and tasks are encoded with serial numbers ranging 0~N-1. 



Continual training is one of the key components of ERNIE, Baidu. Traditionally multi-task training may lead to knowledge retention, which the model may perform well on current task while worse on previous tasks. Continual training takes the model of every task and trains them from scratch, this makes it impossible to get a worse result. But clearly there is an efficiency issue. In this case, ERNIE, Baidu leverages sequential multi-task training. (first sequential then separately, each repetition will add one more task?)



In the initial version published in 2019, there are three-level maskings: word masking, phrase masking and entity masking. The tasks will be trained with different masking. During training, there is an additional embedding except three regular embeddings in BERT base named task embedding (This is pretty simple from 0 ~ N-1). These embeddings are summed to form the ultimate inputs of the model. Knowledge masking is the initial attempt to get a pretrained model. I don't know how other tasks are trained since it is not open-source.



There are questions and doubts on the Internet saying ERNIE 2.0 is quite similar to MT-DNN. Also since there is no ablation study, we don't know which pretrained task contributes more and which one contributes less. The pretrained model is not open-source so we don't know what continual learning really is.



A few months ago, ERNIE 3.0 was released. The model is divided into low level general semantic training and high level task-specific training. Low level will be frozen after general pretraining. This will save time in further high level training. One of the key components of the model is the import of the large-scale unsupervised knowledge graph. It is said that it reached top-1 in over 54 NLP tasks but there is no test of SIGHAN dataset.



One paper use an optimized Dice loss for imbalanced training data. Another paper use FLAT BERT. FLAT only uses span positional encoding that cares about the start and end index of entity. Four distances are heads across entities, tails across entities, first head and second tail and first tail and second head. Final encoding is the RELU activation of XOR (non-linear transformation) of positional encoding of four distances across two entities.  There is also a paper introducing a method that NER can be viewed as machine reading comprehension (MRC). It firstly uses start and end index prediction and does start-end matching based on the probability distribution. The best paper on the leaderboard (Dice coefficient) is based on this paper.



My question about NER:

Is there any difference between a English entity task and a Chinese entity task?

Why not consider transformer (encoder) with CRF (decoder)?

Why not consider using transformer-XL or XLNET as baseline?

What is the trade-off between embedding and encoding?



## References

Internet Articles or Blogs

[1] [Tech Report from Meituan, in Chinese](https://tech.meituan.com/2020/07/23/ner-in-meituan-nlp.html)

[2] [Scientific Blog about NER from Zhihu, in Chinese](https://zhuanlan.zhihu.com/p/61227299)

[3] [Scientific Blog about Post-BERT Generation, in Chinese](https://www.jiqizhixin.com/articles/2019-08-26-16)

Papers

[1] [MSRA-NER Challenge](https://paperswithcode.com/sota/chinese-named-entity-recognition-on-msra)

[2] [BERT-MRC Ranked as Third](https://aclanthology.org/2020.acl-main.519.pdf)

[3] [FLattice transformer Ranked as Second](https://aclanthology.org/2020.acl-main.611.pdf)

[4] [Dice Loss for Data Imbalanced NLP Tasks Ranked as First](https://arxiv.org/pdf/1911.02855v3.pdf)

[5] [Survey on DL for NER](https://arxiv.org/pdf/1812.09449.pdf)

[6] [A Proposal of CRF: Hybrid Semi-markov CRF](https://aclanthology.org/P18-2038.pdf)

[7] [Baseline of Where Everything Gets Started: BERT 2018](https://arxiv.org/pdf/1810.04805.pdf)

[8] [ERNIE from THU with Information Fusion](https://arxiv.org/pdf/1905.07129.pdf)

[9] [MTDNN from Microsoft with Multi-task Learning](https://arxiv.org/pdf/1901.11504.pdf)

[10] [ERNIE 1.0 from Baidu with Optimized Masking](https://arxiv.org/pdf/1904.09223.pdf)

[11] [ERNIE 3.0 with Large-scale Knowledge Graph](https://arxiv.org/abs/2107.02137)

[12] [Cloze-driven Pretraining of Self-attention Networks](https://arxiv.org/pdf/1903.07785.pdf)

[13] [RoBERTa](https://arxiv.org/pdf/1907.11692.pdf)

