<div align="center">
    <h1>LLM Deploy Action</h1>
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

# 推理和部署框架
有了模型接下来就是确定推理部署框架，随着LLM的蓬勃发展，LLM对应的推理部署框架也是百花齐放、眼花缭乱。有常用的transformers、onnx框架，也有专门针对LLM做了优化的部署框架，如vLLM、TensorRT-LLM等。有C++语言实现的llama.cpp，也有移动端部署框架ChatGLM-MNN，还有支持在TPU上部署的框架ChatGLM2-TPU等等。

| 推理和部署框架                                               | 特点                                                         | 优化方法                                                     | 量化方法                                   | 流式推理           | 技术报告                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------ | ------------------ | ------------------------------------------------------------ |
| [HuggingFace transformers, HF](https://github.com/huggingface/transformers) ![GitHub Repo stars](https://img.shields.io/github/stars/huggingface/transformers) | 原生transformers推理接口                                     | KV Cache                                                     |                                            |                    |                                                              |
| [HuggingFace Transformer Text Generation Inference, TGI](https://github.com/huggingface/text-generation-inference) ![GitHub Repo stars](https://img.shields.io/github/stars/huggingface/text-generation-inference) | huggingface提供了LLM文本生成推理库                           | TP, CB, FA, PA                                               | bitsandbytes, GPTQ, EETQ, AWQ              | :white_check_mark: |                                                              |
| [llama.cpp](https://github.com/ggerganov/llama.cpp) ![GitHub Repo stars](https://img.shields.io/github/stars/ggerganov/llama.cpp) | 纯C/C++开发的部署框架，还支持Apple芯片和多种量化方式，提供了多种语言的接口 | 2/3/4/5/6/8-bit quantization, 用户自定义CUDA算子，CPU+GPU混合推理 |                                            |                    |                                                              |
| [text-generation-webui](https://github.com/oobabooga/text-generation-webui) ![GitHub Repo stars](https://img.shields.io/github/stars/oobabooga/text-generation-webui) | 主要用于需要提供web UI交互界面的部署方式，所以推理速度和并发量并不是其考虑的要点 |                                                              |                                            |                    |                                                              |
| [vLLM](https://github.com/vllm-project/vllm) ![GitHub Repo stars](https://img.shields.io/github/stars/vllm-project/vllm) | 使用了PagedAttention技术降低显存占用，推理吞吐量比HF高24x，比TGI高3.5x | PA, CB, 优化了CUDA算子, TP                                   | GPTQ, AWQ, SqueezeLLM, FP8 KV Cache        | :white_check_mark: | [LINK](https://blog.vllm.ai/2023/06/20/vllm.html)            |
| [TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) ![GitHub Repo stars](https://img.shields.io/github/stars/NVIDIA/TensorRT-LLM) | 集成了TensorRT的专门用于在英伟达GPU上部署LLM的SDK，提供了python和C++接口；支持集成英伟达自家的Triton推理服务器 | TP, PP, A16W8, A16W4, CB, PA, FMHA                           | SmoothQuant                                |                    | [LINK](https://github.com/NVIDIA/TensorRT-LLM/blob/main/docs/source/performance.md) |
| [JittorLLMs](https://github.com/Jittor/JittorLLMs) ![GitHub Repo stars](https://img.shields.io/github/stars/Jittor/JittorLLMs) | 大幅降低硬件要求，2GB内存即可部署LLM；模型可快速适配各种计算设备；通过零拷贝技术，实现40%加载加速；通过元算子编译优化，计算速度提升20%+ |                                                              |                                            |                    |                                                              |
| [InferLLM](https://github.com/MegEngine/InferLLM) ![GitHub Repo stars](https://img.shields.io/github/stars/MegEngine/InferLLM) | 旷视推出的LLM推理框架，在llama.cpp的基础上优化了编码；至于与llama.cpp的推理速度对比，readme中并没有提及 |                                                              |                                            |                    |                                                              |
| [mnn-llm](https://github.com/wangzhaode/mnn-llm) ![GitHub Repo stars](https://img.shields.io/github/stars/wangzhaode/mnn-llm) | 用于在安卓系统上部署LLM的框架，支持onnx和mnn模型格式         |                                                              |                                            |                    | [LINK](https://github.com/wangzhaode/mnn-llm?tab=readme-ov-file#cpu-4%E7%BA%BF%E7%A8%8B%E9%80%9F%E5%BA%A6-prefill--decode-toks) |
| [LMDeploy](https://github.com/InternLM/lmdeploy) ![GitHub Repo stars](https://img.shields.io/github/stars/InternLM/lmdeploy) | 由MMDeploy和MMRazor团队联合开发，在InternLM项目推出的部署框架。推理性能为vLLM的1.8倍。后端支持TurboMind和PyTorch两种引擎，前者注重推理速度，后者注重开发效率。 | CB， Blocked KV Cache, 动态拆分和融合, TP                    |                                            |                    | [LINK](https://github.com/InternLM/lmdeploy/blob/main/docs/en/benchmark/a100_fp16.md) |
| [CTranslate2](https://github.com/OpenNMT/CTranslate2) ![GitHub Repo stars](https://img.shields.io/github/stars/OpenNMT/CTranslate2) | 纯C++实现的提高Transformer结构模型推理速度的框架，提供python接口。从笔者使用的情况来看加速效果一般，并且不太适合LLM推理 |                                                              |                                            |                    | [LINK](https://github.com/OpenNMT/CTranslate2?tab=readme-ov-file#cpu) |
| [OpenLLM](https://github.com/bentoml/OpenLLM) ![GitHub Repo stars](https://img.shields.io/github/stars/bentoml/OpenLLM) | 通过集成Pytorch、vLLM、PEFT等工具，支持LoRA微调和部署的开源框架。还支持与BentoML、OpenAI、LangChain等工具集成。 | CB,                                                          | LLM.int8, SpQR(int4), AWQ, GPTQ, SqueezeLM | :white_check_mark: |                                                              |
| [MLC-LLM](https://github.com/mlc-ai/mlc-llm) ![GitHub Repo stars](https://img.shields.io/github/stars/mlc-ai/mlc-llm) | 支持在多种设备和平台上，使用原生api开发各种LLM模型，以实现最大程度的优化。推理速度是vLLM的1.4~2.5倍。 |                                                              |                                            |                    |                                                              |
| [LightLLM](https://github.com/ModelTC/lightllm) ![GitHub Repo stars](https://img.shields.io/github/stars/ModelTC/lightllm) | LightLLM更多的是一个服务分发和处理框架，主要通过异步处理tokenization、模型推理和detokenization来提高模型推理性能 | CB，FA，TP，TA，Int8 K/V Cache                               |                                            |                    |                                                              |
| [AirLLM](https://github.com/lyogavin/Anima/tree/main/air_llm) ![GitHub Repo stars](https://img.shields.io/github/stars/lyogavin/Anima) | 实现了在4GB显存上运行70B模型，使用了分层推理、FA、模型文件拆解、meta设备等技术 |                                                              |                                            |                    | [Link1](https://arxiv.org/abs/2212.09720)  [Link2](https://zhuanlan.zhihu.com/p/667474769) |



**备注**： Tensor Parallelism (TP), Pipeline Parallelism (PP), Continuous Batching (CB), Flash Attention (FA), Paged Attention(PA),  Fused MultiHeadAttention (FMHA), Token Attention (TA)

通过上表可以看到，目前开源了非常多用于LLM推理部署的框架，其中使用较多的有：vLLM, TensorRT-LLM, JittorLM, LMDeploy, OpenLLM, MLC-LLM, AirLLM等。这些框架普遍使用了Attention加速技术、多种量化方法、Continuous Batching、张量并行等技术。

# 显卡
有了模型和推理矿框架，接下来就是部署时使用的硬件设备。因为LLM的计算需求，一般来说都是在GPU上部署LLM。目前市面上Nvidia、AMD、Apple和Intel几家出的GPU，但是应用最广泛的是Nvidia的GPU。

**GPU有几个关键的参数需要关注：**

+** cuda core和tensor core**： 这两个概念是用在Nvidia的GPU上的，它们都是运算单元，主要的差异的算力大小。cuda core是全能型浮点运算单元，在Fermi架构之间被称为streaming processors；而tensor core是从Volta架构开始推出的专门用于矩阵运算的运算单元。在深度学习中，大部分的、耗时的运算都是矩阵相关的运算，特别是矩阵乘法，因此tensor core专门为矩阵运算进行了加速。
+ **显存大小**：显存的大小决定了GPU可以加载多少数据，当显存越大可以计算的数据就越多。
+ **显存带宽**：决定了GPU将数据从显存移动到计算核心的速度，速度越快完成某个运算的速度也更快。
+ **计算能力**：表示每秒的浮点运算量。根据数据的精度又分为双精度（fp64）、单精度（fp32）和半精度（fp16）浮点计算量，一般来说使用双精度浮点计算量来衡量算力。



| 显卡      | 架构         | 显存（GB） | FP64 Tensor Cores（TFLOPS） | 带宽（TB/s） | TF32               | FP16               | BF16               | FP8                | INT8               | 连接                                                         |
| --------- | ------------ | ---------- | --------------------------- | ------------ | ------------------ | ------------------ | ------------------ | ------------------ | ------------------ | ------------------------------------------------------------ |
| H200-SXM  | Hopper       | 141        | 67                          | 4.8          | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | [LINK](https://www.nvidia.com/zh-tw/data-center/h200/)       |
| H100-NVL  | Hopper       | 188        | 134                         | 7.8          | :x:                | :white_check_mark: | :white_check_mark: | :white_check_mark: | :white_check_mark: | [LINK](https://www.nvidia.com/zh-tw/data-center/h100/)       |
| A100-PCIe | Ampere       | 80         | 19.5                        | 1.935        | :white_check_mark: | :white_check_mark: | :white_check_mark: | :x:                | :white_check_mark: | [LINK](https://www.nvidia.com/zh-tw/data-center/a100/)       |
| V100-NVL  | Volta        | 32         | 7.8                         | 0.9          | :white_check_mark: | :white_check_mark: | :x:                | :x:                | :white_check_mark: | [LINK](https://www.nvidia.com/zh-tw/data-center/v100/)       |
| V100-PCIE | Volta        | 32         | 7                           | 0.9          | :white_check_mark: | :white_check_mark: | :x:                | :x:                | :white_check_mark: | [LINK](https://www.nvidia.com/zh-tw/data-center/v100/)       |
| RTX 3090  | Ampere       | 24         |                             |              | :x:                | :x:                | :x:                | :x:                | :x:                | [LINK](https://www.nvidia.cn/geforce/graphics-cards/30-series/rtx-3090-3090ti/) |
| RTX 4090  | Ada Lovelace | 24         |                             |              | :x:                | :x:                | :x:                | :x:                | :x:                | [LINK](https://www.nvidia.cn/geforce/graphics-cards/40-series/rtx-4090-d/) |



**参考资料：**
+ GPU核心指标：https://blog.csdn.net/chongbin007/article/details/123897149
+ cuda core和tensor core的区别：https://www.zhihu.com/question/451127498
+ 浮点计算能力：https://blog.csdn.net/zong596568821xp/article/details/103957058
+ 浮点计算能力：https://www.jishulink.com/post/1205946
+ 巅峰对决：英伟达 V100、A100/800、H100/800 GPU 对比: https://zhuanlan.zhihu.com/p/658054971


