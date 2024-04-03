### Instruction-Tuning

本仓库记录学习几年来在LLM领域(以及多模态大模型领域)的指令微调相关知识，指令微调(instruction finetuning)，又作指令调优(instruction tuning)，指令跟随(instruction following)。

大语言模型在下游任务上的性能，可以通过在指定任务上进行微调(fientune)提升，或者给定少量提示，也叫做few-shot prompt，指令微调是为了让模型能够理解人类指令，在不给出少量提示的情况下，增强模型的zero-shot能力的一种方式，比如，你可以在许多任务上进行指令微调，在其他任务上(unseen, 模型未见过)进行测试，你会发现模型的理解能力有所提升，可以给出更好的答案。

指令微调数据一般如下 {###Insruction:...., ###Input:...., ###Answer:....} 组成，举个例子，问模型1+1等于几， 微调数据就是{###Insruction:在接下来的input中，我给你两个数字，你能够告诉我两个数字的和吗, ###Input: 1，1, ###Answer: 2}。以上就是一条微调数据，当然，你可以更换数据模板，比如不用Input，或者给一些提示(few-shot)，或者在答案中给出CoT, 也就是整个答案的推理过程。

或许你曾经听过FLAN(finetune language net)，这篇论文是谷歌提出Instruction tuning的经典，文章将微调后的模型和175B的GPT-3进行了比较，证明了指令微调的优越性：
https://openreview.net/forum?id=gEZrGCozdqR

同年，谷歌针对CoT(chain of thought，链式思维)和few-shot在Instruction-Tuning中能否提升模型的能力，以及模型大小和数据集大小对指令调优的影响进行了进一步探究，这篇文章用了146种任务，473个数据集，最终确定指令微调任务的多样性可以提升模型的能力, CoT数据可以提升模型的推理能力, 模型规模越大，经过指令微调的效果越好，还没看到上限，文中最大540B：
https://doi.org/10.48550/arXiv.2210.11416

有没有想过为什么指令微调效果会变好呢，这篇文章给出了一些定量分析，从token的偏移角度出发，对比微调后的模型和微调前的模型的回答的token分布，发现token会产生%5-8%左右的偏移(使用的模型为 Llama-2-7b -> Llama-2-7b-chat,  Llama-2-7b -> Vicuna-7b-v1.5, Mistral-7b -> Mistral-7b-instruct), 而且产生偏移的多数都是风格token，比如however, if这种词汇，会让模型的回答更加流畅和清晰，而模型回答中的关键词汇和信息，是模型本身经过预训练以后就具有的，也就是base模型和chat模型(tuned)产生的回答中关键信息一致，很有趣，所以这篇文章使用ICL作为提示+系统提示来代替指令微调。  但是是否产生这种情况是因为使用的测试模型的微调数据相近或者一致呢，还有待考究：https://arxiv.org/abs/2312.01552 

从上述问题出发，国内软件所联合美团提出以下论文，并发表观点：(1) 对于指令微调而言，学习与模型参数知识不一致的世界知识无法带来增益，甚至会造成额外的损害。(2) 有效指令微调的本质在于完成行为模式转换的同时，保持指令微调前后模型参数知识的一致性。换句话说，指令微调的核心作用机制并不是让模型去“学习”额外的知识，而是将模型内部现有的知识进行一种自我的对齐。这个其实就很符合一直以来的猜想，模型学不到额外知识，毕竟你参数也没怎么更新，指令微调数据里面的内容也都是用的已有的数据，所以只是让模型学会回答，这篇文章也对应上面的使用ICL去替代指令调优，因为本质上ICL连参数更新都没有，更是教会模型如何去回答信息，从而可以产生和IFT相似的效果：https://arxiv.org/abs/2402.18243

近两年指令微调不仅在LLM领域，在多模态领域也是一大热点，经过探究和实验，许多学者认为，指令调优意在对齐模型的内部知识，让模型学会回答，这才是指令调优提升模型能力的重点，所以经过浓缩后的指令调优数据集可能仅有几千条，但是足以提升模型的能力。
 
