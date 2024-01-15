## 第五次课程笔记

#### 1.大模型部署
定义：将训练好的模型在特定软硬件环境中启动，使模型能够接收输入并返回预测结果，通常需要一些模型优化，例如模型的压缩和硬件加速
大模型：内存开销大、 
自回归生成token需要缓存Attention的K和V    
LLM的结构大部分是decoder-only，比较简单  

大模型部署挑战：
1、设备：如何应对巨大的存储问题
2、推理：如何加速token的生成速度、如何解决动态shape，让推理可以不间断、如何有效管理和利用内存
3、服务：如何提高系统吞吐量、如何降低个体用户的响应时间

部署方案：
huggingface transformers
专门的推理加速框架，云端例如lmdeploy\vllm，本地例如llama.cpp等等

#### 2.lmdeploy
在英伟达设备上部署的全流程解决方案，包括模型轻量化、推理和服务
4bit权重、8bit K/V
推理支持turbomind pytorch
服务apiserver,gradio

为什么做量化：降低显存     
weight only的量化，可以解决计算密集和访存密集的问题
![Alt text](/image5/note1.png)

持续批处理：队列
有状态的推理：请求不带历史记录，历史记录在清理测缓存    
blocked K/V cache:K/V在推理过程中需要一直用到，
block状态：free不用放现存，active正在推理，chche被缓存的序列占用，三种状态可以迁移
![Alt text](/image5/note1.png)
高性能的cuda kernel

推理服务api serversss
![Alt text](/image5/note3.png)

C/S架构
1. 在线转换：本地的huggingface模型：
```bash
lmdeploy chat turbomind /share/temp/model_repos/internlm-chat-7b/  --model-name internlm-chat-7b
```
速度变得比之前快了很多
![Alt text](/image5/chat.png)

2. 离线转换：将模型转为lmdeploy trubomind的格式，在root/下生成一个workspace的文件夹，包含模型推理需要用到的文件
```bash
lmdeploy convert internlm-chat-7b  /root/share/temp/model_repos/internlm-chat-7b/
```
![Alt text](/image5/quanzhong.png)
```bash
#开启本地对话
lmdeploy chat turbomind ./workspace
```
3. turbomind推理+API服务
```bash
lmdeploy serve api_server ./workspace \
	--server_name 0.0.0.0 \
	--server_port 23333 \
	--instance_num 64 \
	--tp 1
```
![Alt text](/image5/api.png)
新开一个窗口激活环境，然后输入
```bash
# ChatApiClient+ApiServer
lmdeploy serve api_client http://localhost:23333
```
就可以实现对话
![Alt text](image-7.png)
本地输入
```bash
ssh -CNg -L 23333:127.0.0.1:23333 root@ssh.intern-ai.org.cn -p 34802
```
即可在浏览器打开
![Alt text](/image5/api11.png)


