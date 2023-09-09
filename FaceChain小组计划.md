# FaceChain

## 01 基本信息

### 项目信息

- 原项目地址：[modelscope/facechain: FaceChain is a deep-learning toolchain for generating your Digital-Twin. (github.com)](https://github.com/modelscope/facechain)

- 原项目说明：

  - 官方引导：[FaceChain人物写真 · 魔搭社区 (modelscope.cn)](https://modelscope.cn/brand/view/FaceChain)

  - 主要结构：<img src="https://github.com/Aziily/facechain/raw/main/resources/framework_eng.jpg" alt="pipeline" style="zoom:30%;" />

### 小组信息

- 小组成员：陈哲恺、唐嘉俊、康致宁
- github昵称：Aziily、iotang、potazinc
- 小组fork连接：
  - [Aziily/facechain: FaceChain is a deep-learning toolchain for generating your Digital-Twin. (github.com)](https://github.com/Aziily/facechain/tree/main)



## 02 项目思考

当前项目仍然处于开发阶段，虽然主要pipeline已经完成，但是在功能拓展和用户体验上仍有较大提升空间

需要注意的是，本项目和传统的diffusion的id控制不同，是通过生成图片替换人脸的方式实现的，类似此工作的一部分[tencent-ailab/IP-Adapter: The image prompt adapter is designed to enable a pretrained text-to-image diffusion model to generate images with image prompt. (github.com)](https://github.com/tencent-ailab/IP-Adapter)

考虑到短学期时长以及小组能力，我们打算聚焦在以下五个方面

- 风格lora的适配添加
- 优化用户体验
- 添加拓展功能
- 修复过程bug
- 尝试一些实现



## 03 任务说明

### (1) 基本任务

在现有base模型基础上，使用civitai上训好的lora模型，在`facechain/constants.py`文件中调参（可能需要部分代码更改）以实现对新风格的兼容以及稳定使用

> 1. [C站本身 civitai](https://civitai.com/)
>
> 2. [国内镜像站1 炼丹阁 - 官网,专业全面的模型平台社区,高手AI作者都在这∩•ﻌ•⊃ (liandange.com)](https://www.liandange.com/)
> 3. [国内镜像站2 DESAI|爱得思](https://www.desai.art/#/)



**要点**

1. 选择一个可用的lora，最好是直接基于基础模型（`leosamsMoonfilm_filmGrain20/MajicmixRealistic_v6`）训练的lora
2. 注意使用真人lora（当然如果是为了风格化使用二次元lora也不是不行）
3. 调参本身应该不会太难，因为涉及内容不多，但是可能涉及部分代码修改，有问题及时分享



### (2) 拓展任务

> 本部分任务如果能完成，基本能保证必定被merge，但是该部分任务可能需要学习一些额外知识

#### a. lora即插即用

本部分内容也是中文文档的TODO List中的第一个任务，可以理解为给用户一个可自行选择lora的前端，以及两个调控human lora和style lora的组件，使用户可以选择自己要使用的lora以完成自定义风格化



**要点**

1. 基于gradio搭建
2. 不能破坏现有项目的base model和lora下载保存体系
3. 分为三步
   1. 允许用户自由调整human lora和style lora的权重，但是只能使用已有的lora
   2. 允许用户据上传lora
   3. 允许用户提供lora链接或lora名称，自动下载使用



#### b. inference生成图片的保存和查看

当前pipeline中，用户只能在inefrence生成的当时进行生成图像的保存，而无法查看过往生成的图片结果

通过修改保存方式和保存位置，在做好用户区分、人脸区分和风格区分的情况下，允许用户查看历史生成记录，优化使用体验



**要点**

1. 基于gradio搭建
2. 要注意用户不同、人脸模型不同或风格不同时，应该保存在不同的根目录下
3. 分为两步
   1. 修改当前保存体系，做到对用户、人脸、风格的存储控制
   2. 提供可供用户查看历史记录的ui接口



#### c. pose预览和自定义

facechain项目的艺术照部分，主要使用的是基于[controlnet](https://arxiv.org/abs/2302.05543)实现的姿势控制（应该还使用了其他技术进行背景控制啥的）。但是，现在的版本是从图像中提取人物keypose信息，然后根据keypose信息进行实现（大致过程如下图）。可以考虑通过允许用户自由绘制keypose图片以实现自定义艺术照

<img src="https://github.com/lllyasviel/ControlNet/blob/main/github_page/p12.png?raw=true" alt="keypose" style="zoom:50%;" />



**要点**

1. 得先了解keypose的大致组成和作用

2. 参考stable diffusion webui一个插件的实现[openpose-editor](https://github.com/huchenlei/sd-webui-openpose-editor)

   <img src="https://github.com/huchenlei/sd-webui-openpose-editor/raw/main/readme_assets/editor_in_modal.png" alt="editor" style="zoom:30%;" />

3. 分为两步
   1. 在用户上传图片或使用项目中pose图片之后，产生openpose预览结果
   3. 允许用户直接通过gradio的ui绘制keypose

> 本部分可能比较困难，考虑只实现第一部分



#### d. 基于mixofshow重构facechain的实现方式

> 本部分工作需要大规模重构现有facechain代码，只是给出想法但应该不会进行实现

原工作链接：[Mix-of-Show (showlab.github.io)](https://showlab.github.io/Mix-of-Show/)



正常来说，lora是只对attention层进行，而在mixofshow中使用了`P+`工作中提出的一种lora方式，通过将一个词对应到`tokenizer_attention_layers * 2`个词上，实现了一种更为精细的lora学习，对于保持id和保证较好的生成质量有不错的效果

在此之上，mixofshow提出了一种新的merge方式，不是通过简单的加权计算，二是通过对tensor的concat和对单层的优化，实现多风格的融合，这种融合被证明有更好的效果（不过原论文只有对多人物的实验，对于人物+风格是否有效有待验证）

我们可以通过重构human lora的学习方式为edlora，并通过style lora产生图片重新进行style edlora的训练，来实现学习到human edlora和style edlora，再基于mixofshow的merge方式进行lora的融合。



**要点**

1. edlora和lora本身概念不一致，不存在兼容性，所以本部分内容修改幅度较大、难度较大
2. 分为两步
   1. 测试human edlora和style edlora的训练可行性，以及二者的merge可行性
   2. 修改facechain的训练过程使用edlora，merge过程使用mixofshow方式



## 04 任务分工

- 风格适配实现：唐嘉俊、陈哲恺、康志宁
- ui功能拓展和使用优化：陈哲恺
- 过程bug修复：康志宁
- 其他实现



## 05 任务进度

1. 基本任务
   - 当前进度：已进行多组风格适配尝试，并在三个风格上取得了不错的效果，已merge到主分支
   - Style_lora：traditional Chinese Style、zhuangzu、european fields
   - PR_link：[提供 3 个新的风格模型 by iotang · Pull Request #189 · modelscope/facechain (github.com)](https://github.com/modelscope/facechain/pull/189)
   - 未来计划：进一步进行更多lora风格适配
2. 拓展任务
   1. lora即插即用
      - 当前进度：已实现用户上传lora的safetensors文件以使用，并允许选用用户历史上传lora文件，同时保留了原有预设风格使用的同时优化了部分bug和使用体验，已merge到主分支
      - PR_link：[Lora即插即用和历史图片结果查看的实现 by Aziily · Pull Request #188 · modelscope/facechain (github.com)](https://github.com/modelscope/facechain/pull/188)
      - 未来计划：本来计划中是有用户提供链接直接下载lora，减少上传lora所需的网络开销，但是感觉这样反而没有上传lora来的直观便捷，并且对于可能存在的lora差异和lora链接差异可能存在较大的处理困难。在通过允许用户调用历史上传lora之后，不需要多次上传lora相同，可以部分减小网络开销，所以直接下载的方式可能不考虑实现
   2. inference生成图片的保存和查看
      - 当前进度：已实现对保存方式的重构，完成了对不同用户、不同模型、不同人脸和不同风格的分置保存，并通过ui接口提供了对用户自己的历史生成记录的查看和删除，已merge到主分支
      - PR_link：[Lora即插即用和历史图片结果查看的实现 by Aziily · Pull Request #188 · modelscope/facechain (github.com)](https://github.com/modelscope/facechain/pull/188)
      - 未来计划：在当前ui接口当中，是通过两个button实现的查看和删除功能，这并不是十分用户友好，可能考虑后续实现更加便于使用的ui设计
   3. pose预览和自定义
      - 当前进度：已实现对pose图片的openpose结果生成预览，已merge到主分支
      - PR_link：[添加pose预览 by Aziily · Pull Request #192 · modelscope/facechain (github.com)](https://github.com/modelscope/facechain/pull/192)
      - 未来计划：在实际查看了openpose-editor的实现之后，我们认为这一基于webui开发的extension本身体量过大，并且不便于直接接入当前项目，可能不进行pose自定义编辑的实现
   4. 基于mixofshow重构facechain的实现方式
      - 此仅为一个构想，暂时没有实现考虑
3. 已知bug修复
   1. train：
      - 如果train过程中出现类似OOM或者download失败的问题，不会在ui界面提示失败，而是会显示训练完成——待修复
   2. inference：
      - 切换base model时可能出现cloth style显示错误的问题——已修复（在lora即插即用中修复）



## 06 总结

我们小组当前已经基本完成了课程要求，并在此基础上为facechain项目本身进行了一定程度的贡献。

后续我们会进行学习和贡献，主要尝试功能的拓展和ui的优化，不再将新风格lora的适配作为重点。
