# ChatGLM模型

ChatGLM系列模型由清华大学开发的、支持中英文双语问答的对话LLM。目前开源了6b级别的模型，包括[ChatGLLM-6B](https://github.com/THUDM/ChatGLM-6B)、[ChatGLM2-6B](https://github.com/THUDM/ChatGLM2-6B)和[ChatGLM3-6B](https://github.com/THUDM/ChatGLM3)三个模型。ChatGLM4-6B并没有开源，但是可以在[智谱清言](https://www.chatglm.cn/)和[API平台](https://open.bigmodel.cn/)体验。



## General Language Model

ChatGLM使用[General Language Model （GLM）](https://github.com/THUDM/GLM)作为backbone，先来看看这个架构是怎么样的，和Transformer区别在哪里。一般来说预训练架构主要分为自回归（Auto-Regressive, AR）、自动编码（Auto-Encoding，AE）和编解码（Encoder-Decoder，ED）三种结构。ChatGPTs系列的模型就是第一种AR结构，这种结构为从左到右的语言模型，即根据给定的上下文预测下一个词，是一种从左往右的单向模式。AE架构以BERT为代表，以Maksed Language Model（MLM）作为预训练任务，从给定的上下文中学习双向文本特征，预测被Masked掉的词。ED架构则包括了解码器和编码器，解码器使用双向注意力和解码器使用单向注意力。

GLM则期望可以统一这些结构可以完成的任务，既可以完成文本生成类任务，也可以完成文本理解、文本总结类任务。因此GLM提出了基于AR结构的填空预训练任务：


![image-20240208092533234](./images/image-20240208092533234.png)

+ 从原始的文本中随机采样不同长度的文本得到若干个span，将span的位置使用MASK替换，如何将文本分层A和B两部分：被MASK掉的文本和随机采样得到的span
+ 对于模型来说，输入是由A和B组成的文本，输出就是预测的span内容
+ 对于attention来说，MASK掉的文本相互之间是可见的，而在span中的文本则是单向可见的。如此就实现了既可以学习双向的上下文信息，也可以完成文本生成类任务。



## ChatGLM

基于GLM架构，清华首先训练了GLM-120B模型。为了适应大量参数的模型训练，在GLM的基础上做了如下几点优化：

+ 使用了DeepNorm保证模型训练稳定性：
  $$
  DeepNorm(x) = LayerNorm(\alpha \times x + Network(x)), \alpha=(2N)^{1/2}
  $$

在`ffn, v_proj, out_proj`使用了调节分子$(2N)^{-1/2}$ 的Xavier初始化。



+ 使用了旋转位置编码(Rotary Positional Encoding, RoPE)

+ 使用GeLU作为激活函数

+ 使用了fp16混合精度训练方式

+ 为了避免loss尖刺，在词嵌入层使用了梯度收缩：
  $$
  word\_embedding = word\_embedding \times \alpha + word\_embedding.detach() \times (1 - \alpha)
  $$
  

GLM-130B支持2048长度上下文，在1T tokens上完成了预训练，并且使用了74个数据集完成指令微调。总共使用了96个DGX-A100 GPU（8*40G）花费了60天完成训练。为了加快训练速度，使用了数据并行（Data Parallel， DP），张量模型并行（Tensor Model Parallel， TP）和流水线模型并行（Pipeline Model Parallelism， PP）组成了3D并行策略（4路TP，8路PP）。



为了加速GLM-130B的推理速度，研究者们使用了FasterTransformer实现GLM-130B，相比于PyTorch推理速度提升了7-8.4倍。为了避免量化带来的异常激活值，研究者们采用了W8A16方式（即权重使用INT8量化，激活值保持FP16类型）。为了进一步降低显存消耗，GLM-130B最终采用了INT4量化方法，可以在消费级显卡（RTX 3090/2080 Ti）上运行。



# ChatGLM-6B系列模型

清华在2023年开源了GLM-6B版本模型，包括ChatGLM-6B、ChatGLM2-6B和ChatGLM3-6B。

| 模型        | 特点                                                         | 连接                                         |
| ----------- | ------------------------------------------------------------ | -------------------------------------------- |
| ChatGLM-6B  | 支持中英文双语的对话语言模型，经过~1T tokens训练，INT4量化下最低只需6GB显存 | [LINK](https://github.com/THUDM/ChatGLM-6B)  |
| ChatGLM2-6B | 使用了GLM的混合目标函数，经过了~1.4T tokens训练并对齐人类偏好；使用了FlashAttention基础，上下文长度从2K扩展到32K；使用Multi-Query Attention技术，推理速度提升42% | [LINK](https://github.com/THUDM/ChatGLM2-6B) |
| ChatGLM3-6B | 采用了更多样的训练数据、更充分的训练步数和更合理的训练策略；使用了全新的Prompt格式；还支持工具调用、代码执行和Agent等复杂任务 | [LINK](https://github.com/THUDM/ChatGLM3)    |

