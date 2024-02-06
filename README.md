<div align="center">
    <h1>LLM Deploy Note</h1>
</div>

自从ChatGPT为代表的大语言模型(Large Language Model, LLM)兴起之后，各大巨头公司、创业公司、科研机构都在卷LLM训练，忙着发布自家的大模型。身边的人都在忙着如何在垂直领域里怼数据，各种花式微调。本人卷不过，卷不动，没脑子，不懂卷，遂换赛道，决定整LLM部署。谨以此文记录破卷之路的学习心得。希望本repo可以坚持更新。🐷

# 模型
大模型练手的前提之一就是有模型，这意味着咱只能用开源模型，闭源模型咱够不着，所以先来看看有哪些可用的中文开源模型。至于为什么要用中文模型，这个问题你觉得呢？

## GLM基座
+ ChatGLM系列
  -  [ChatGLM-6b](https://github.com/THUDM/ChatGLM-6B) ![](https://img.shields.io/github/stars/THUDM/ChatGLM-6B.svg) 模型是由清华大学和智源联合发布的中文开源模型，基于General Language Model (GM) 架构，在1T token上训练，支持2K上下文。在INT4量化之后只需要6GB显存即可完成部署，可以运行在消费级显卡上。
  -   [ChatGLM2-6b](https://github.com/THUDM/ChatGLM2-6B) ![](https://img.shields.io/github/stars/THUDM/ChatGLM2-6B.svg)  在ChatGLM-6b的基础上做了三个方面的改进：1）使用了GLM的混合目标函数，在1.4T token上对齐人类偏好，性能上大幅提升；2）基于FlashAttention技术，支持32K上下文；3）基于Multi-Query-Attention技术，推理速度提升42%
  -    [ChatGLM3-6b](https://github.com/THUDM/ChatGLM3) ![](https://img.shields.io/github/stars/THUDM/ChatGLM3.svg)  使用了更多样的训练数据，拥有10b以下基础模型模型中最强性能；采用了全新的Prompt格式，原生支持工具代调用、代码执行和Agent等任务。

## LLaMA基座
+ LLaMA系列
  - [Chinese-LLaMA-Alpaca](https://github.com/ymcui/Chinese-LLaMA-Alpaca) ![](https://img.shields.io/github/stars/ymcui/Chinese-LLaMA-Alpaca.svg) 项目包括中文LLaMA和中文Alpaca两个系列模型，前者采用因果语言模型（Causual Lanugage Model, CLM）的方式进行二次预训练，后者使用指令微调的方式进行训练。而且由于使用的是未正式开源的LLaMA模型，所以这个项目中发布的是LoRA权重。
  -  [Llama2-Chinese](https://github.com/FlagAlpha/Llama2-Chinese)   ![](https://img.shields.io/github/stars/FlagAlpha/Llama2-Chinese.svg) 项目同样包括两个系列的LLaMA模型，一个是基于LLaMA2-Chat进行指令微调的Llama2-Chinese-Chat系列模型，一个基于中文数据进行预训练的Atom系列模型。

+ Baichuan系列
	- Baichuan是由百川智能开发的中文LLM模型，采用了与LLaMA一样的模型设计。在此基础上，使用了FlashAttention、算子切分、通讯优化等技。[Baichuan-7b](https://github.com/baichuan-inc/baichuan-7B) ![](https://img.shields.io/github/stars/baichuan-inc/baichuan-7B.svg)  和[baichuan-13b](https://github.com/baichuan-inc/Baichuan-13B) ![](https://img.shields.io/github/stars/baichuan-inc/baichuan-13B.svg) 支持的上下文长度都是4096，后者在更多数据上进行训练，并且使用了ALiBi位置编码。百川智能还开源了int4和int8的量化版本，用于推理加速，可以在消费级显卡上部署。
	- [Baichuan2](https://github.com/baichuan-inc/Baichuan2) ![](https://img.shields.io/github/stars/baichuan-inc/Baichuan2.svg) 在更大、更多样的数据集上进行训练

+ Qwen系列
	- [Qwen](https://github.com/QwenLM/Qwen) ![](https://img.shields.io/github/stars/QwenLM/Qwen.svg) 是阿里发布的通义千问系列大模型，同样采用了LLaMA的模型架构，在3T token上进行了训练，支持8K上下文。

+ 书生·浦语（InternLM）系列
  - [书生·浦语2](https://github.com/InternLM/InternLM) ![](https://img.shields.io/github/stars/InternLM/InternLM.svg)  由商汤科技、上海AI实验室联合香港中文大学、复旦大学和上海交通大学共同发布，在书生·浦语的基础上使用了更大、更多样的数据，在代码、数理、创作等方面的性能都有大幅提升。

+ ChatRWKV系列
  - [ChatRWKV](https://github.com/BlinkDL/ChatRWKV) ![](https://img.shields.io/github/stars/BlinkDL/ChatRWKV.svg) 开源了一系列基于RWKV结构的模型，RWKV是一个使用RNN结构的模型，其特点是无论上下文长度是多少，其速度不变，占用显存不变。

其他更多的开源中文模型介绍可以参考[这个repo](https://github.com/HqWu-HITCS/Awesome-Chinese-LLM)。从上面列出的star较多的模型来看，国内目前开源的LLM主要基于GLM、LLaMA和RWKV三种模型架构。
