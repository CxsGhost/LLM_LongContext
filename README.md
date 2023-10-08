# 总结拓展LLM上下文长度的相关文献
---
本repo主要总结了近年来关于拓展大模型上下文输入长度的相关方法及文献，这些方法大多都包含Finetune或即插即用两种模式，目前主要技术路线可以总结为以下两种：

 - **修改位置编码实现内插、外推**
 - **稀疏注意力（Sparse Attention）**

| 年份 | 技术路线 | 标题（arxiv链接） | 简介 | 代码 | 实测效果 |
|--|--|--|--|--|--|
| 2023.10 | 稀疏注意力 | [Ring Attention with Blockwise Transformers for Near-Infinite Context](http://arxiv.org/abs/2310.01889) | 。。 | 。。 | / |
| 2023.9 | 位置编码PI+高效Finetune探索、继\续\从头Pretrain | [Effective Long-Context Scaling of Foundation Models](http://arxiv.org/abs/2309.16039) | 全文以实验和分析为主，通过线性缩放插值改进位置编码（rope、xpos），然后在混合长上下文数据上“继续”预训练+指令微调，以较低的成本达到了较好的效果。还从头训练llama，对比了在不同阶段切换到长文本数据的最终效果，结果显示先以短文本训练，在中后期切换到长文本，最终性能几乎没有损失，并极大减少了计算量。 | / | / |
| 2023.9 | 稀疏注意力+位置编码PI+Finetune | [LongLoRA: Efficient Fine-tuning of Long-Context Large Language Models](http://arxiv.org/abs/2309.12307) | 提出了一种稀疏注意力：shift short attention，以及新微调方法LoRA+（LoRA+解冻embedding层和norm层），结合位置编码改进（PI、NTK-aware）对llama进行微调，效果接近全量微调，极大节省计算成本。 | [官方](https://github.com/dvlab-research/LongLoRA) | / |
| 2023.9 | 稀疏注意力（即插即用） | [LM-Infinite: Simple On-the-Fly Length Generalization for Large Language Models](http://arxiv.org/abs/2308.16137) | 这篇进行比较深入的理论分析，llm面对超长输入崩溃的主要原因有三：1. 对于较远处没见过的距离，注意力分数会发生大幅度不稳定震荡。2. 过多的token数导致注意力稀释，注意力熵爆炸理论，无法有效利用信息。3. attention会隐式的编码绝对位置信息，导致输入序列中前部和中后部vectors的分布空间明显不同，而局部注意力会导致忽略前部token，进而导致模型失效。针对以上问题，提出了稀疏注意力“λ attention”，以及距离限制（超过最大距离都设为最大），无需finetune即插即用。 | [非官方](https://github.com/Glaciohound/LM-Infinite) | / |
| 2023.8 | 探索位置编码外推+Benchmark | [Giraffe: Adventures in Expanding Context Lengths in LLMs](http://arxiv.org/abs/2308.10882) | 主要研究位置编码的外推能力，提出一个由三个数据集组成的Benchmark（key-value检索、基于wiki的QA）用于评估llm长上下文能力，然后在rope、线性缩放PI、xpos、alibi、randomPE以及自己提出的Truncated-rope上finetune再外推，最终结论是对rope进行线性缩放PI外推能力是最好的。 | [官方](https://github.com/abacusai/Long-Context) | / |
| 2023.8 | 位置编码外推+Finetune | [YaRN: Efficient Context Window Extension of Large Language Models](http://arxiv.org/abs/2309.00071) | 提出了一种名为YaRN（Yet another RoPE extensioN method）的rope插值方法，借鉴并结合了线性PI、ntk-aware等方法。文中对位置编码的问题进行了深入分析，并说明YaRN能实现“短训练，长测试”，即具有良好的外推能力。 | [官方](https://github.com/jquesnelle/yarn) | / |
| 2023.7 | 外部数据库+稀疏注意力 | [Focused Transformer: Contrastive Training for Context Scaling](http://arxiv.org/abs/2307.03170) | 这篇中了NeurIPS23，但是写的有点乱，看起来比较头疼。主要是把Memorizing Transformers的方法重新拿来用，新东西是提出了一个“CrossBatch”的训练方法，借鉴对比学习思想，从同batch里抽其他样本的token作为“负样本上下文”，然后最终是说训练的kv更有区分度。文章里介绍完以上东西，又说为简化实现什么的在训练时把knn改成了“dense attention”但是根本没说清楚是咋回事，这个地方有点怪。又说crossbatch在实现上也有点出入（本身就没介绍清楚，看网上大家读完也都有点迷糊）。又说训练时和推理时位置编码实现的行为也不一样（这块也没说清楚咋回事）。知乎有人解析代码，大概是说只有Memorizing Transformers那部分有，其余的都没，和文章里介绍的有出入。 | [官方](https://github.com/CStanKonrad/long_llama) | / |
| 2023.7 | 稀疏注意力+大规模分布式 | [LongNet: Scaling Transformers to 1,000,000,000 Tokens](http://arxiv.org/abs/2307.02486) | 微软搞的，提出了一种名为“dilated attention”的稀疏注意力，说是结合大规模分布式算法，能输入10亿tokens而模型性能不崩溃，速度也可以。dilated attention通过**分块大小**和**扩张率**来控制采样，多种模式结合能兼顾局部和全局信息。 | / | / |
| 2023.5 | 稀疏注意力+外部数据库（可选） | [Unlimiformer: Long-Range Transformers with Unlimited Length Input](http://arxiv.org/abs/2305.01625) | 文章基于encoder-decoder架构，但代码提供了对llama这种decoder-only架构的支持。在encoder阶段对长输入以chunks编码，建立外部数据库，在decoder阶段通过KNN检索指定窗口大小数量的vectors做attention。可仅用于验证、测试阶段，训练阶段成本大。主要在小说摘要数据集上实验。 | [官方](https://github.com/abertsch72/unlimiformer) | / |
| 2022.12 | 稀疏注意力（分段式输入） | [Recurrent Memory Transformer](http://arxiv.org/abs/2207.06881) | 。。 | [官方](https://github.com/booydar/lm-rmt) | / |
| 2022.3 | 稀疏注意力+外部缓存（分段式输入） | [Memorizing Transformers](http://arxiv.org/abs/2203.08913) | 。。 | [非官方](https://github.com/lucidrains/memorizing-transformers-pytorch) | / |
| 2019 | 稀疏注意力+外部缓存（分段式输入） | [Transformer-XL: Attentive Language Models Beyond a Fixed-Length Context](http://arxiv.org/abs/1901.02860) | 。。 | [官方](https://github.com/kimiyoung/transformer-xl) | / |
