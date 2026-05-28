skills: https://aws.amazon.com/cn/blogs/china/ai-understanding-component-library-intelligent-d2c-architecture-aws-kiro-mcp-skills/

## 各种概念

[LLM、Agent、MCP、Skill 是什么？它们之间有什么关系？](https://blog.csdn.net/Aric_Jones/article/details/158292255)

[10分钟讲清楚 Prompt, Agent, MCP 是什么](https://www.bilibili.com/video/BV1aeLqzUE6L/?spm_id_from=333.788.top_right_bar_window_custom_collection.content.click&vd_source=41ed998ac767425fb616fd9071ce9682)

[【闪客】一口气拆穿Skill/MCP/RAG/Agent/OpenClaw底层逻辑](https://www.bilibili.com/video/BV1ojfDBSEPv/?spm_id_from=333.788.recommend_more_video.1&trackid=web_related_0.router-related-2479604-9xr68.1776935746345.533&vd_source=41ed998ac767425fb616fd9071ce9682)

[从 LLM 到 Agent Skill，一期视频带你打通底层逻辑！](https://www.bilibili.com/video/BV1E7wtzaEdq/?spm_id_from=333.788.recommend_more_video.2&trackid=web_related_0.router-related-2479604-dplt2.1776741772597.515&vd_source=41ed998ac767425fb616fd9071ce9682)

[Agent Skill 从使用到原理，一次讲清](https://www.bilibili.com/video/BV1cGigBQE6n/?spm_id_from=333.788.recommend_more_video.2&trackid=web_related_0.router-related-2479604-9xr68.1776935746345.533&vd_source=41ed998ac767425fb616fd9071ce9682)

### LLM

**LLM**（Large Language Model ，大语言模型）是一种基于深度学习的 AI 模型，通过海量文本数据训练，具备理解和生成自然语言的能力。

### Prompt

Prompt（提示词/提示工程），这是用户与 AI 模型交互的最基本形式。它是你输入给大语言模型（LLM）的一段文本，用来引导模型生成你想要的回复。

可以看作是**写给 AI 的“使用说明书”或“指令”**。它本身不执行动作，只是告诉模型应该做什么、扮演什么角色、输出什么格式。

用户发送的聊天文本叫做`User prompt`。给LLM设置背景、时间、角色等条件的prompt叫做`System prompt`

### Memory

基础的LLM是没有记忆功能的，当发起追问的时候它就不知道你在说什么了，比如问`1 + 1 等于几？`，回答了`2`，再继续追问`再加1呢？`，他可能就不知道你在说啥了。LLM只能一问一答，那么可以将之前的对话内容都处理成`System prompt`一同与新问题发送给LLM这样就有了记忆（`Memory`）以及追问功能了。

在对话越来越多之后`Memory`就会越来越大，消耗的`token`也就越来越多，这时我们可以将`Memory`发给LLM，让它进行浓缩、总结。以此来压缩`Memory`

### RAG

LLM的知识都是平时训练得到的，或者是已经过时的信息，如果问关于你家有几只猫、几个狗或者你朋友喜欢吃什么菜这种问题，他是不知道答案的。此时可以给LLM添加额外的知识，这个额外的知识就是`RAG`(检索增强生成 Retrieval-Augmented Generation)

### Agent

LLM除了能够进行一问一答的聊天回复外，其他事情都做不了，比如如果让LLM给你写一遍文章并导出生成为一个`文章.word`文件的话，它自能做到写文章而不能生成文件，因为它没有这个能力。

我们可以先准备好一些文件读写方法或者服务，并将这些方法的描述生成`system prompt`一同发给LLM，LLM在写完文章后就可以告诉我们执行文件写方法生成文件了

我们将这个方法、生成`system prompt`、执行方法的操作做成一个程序，我们只需要执行发送问题`user prompt`即可，这个程序就是`Agent`。那些方法和服务可以叫做`Agent tools`

### LangChain

`Agent`程序一般使用`LangChain`框架进行开发，Langchain的工作流程可以概括为以下几个步骤：

1. **提问**：用户提出问题。
2. **向语言模型查询**：问题被转换成向量表示，用于在向量数据库中进行相似性搜索。
3. **获取相关信息**：从向量数据库中提取相关信息块，并将其输入给语言模型。
4. **生成答案或执行操作**：语言模型现在拥有了初始问题和相关信息，能够提供答案或执行操作。

### Function calling

`Agent`就是一个程序，它需要接受一个明确的输入才能进行判断处理，而LLM的在不加约束的时候返回数据可能是文本、也可能是xml数据格式是不确定的，所以要想 `Agent` 能处理好任务就必须要给LLM的输出回复添加约束，`Function calling`就是定义这个约束的，一般情况下它要求`Agent tool`的数据格式描述为一个JSON

```json
{
  name:"wirte_article",
  desc:"写入文章",
  params:{
    content:"str",
    filename:"str_filename",
  }
}
```

同时`Function calling`要求了LLM返回的`Agent`可以解析的数据格式，比如JSON

```json
{
  type:"call",
  name:"wirte_article",
  args:{
    content:"文章内容",
    filename:"文章.word"
  }
}
```

这些要求都可以通过从`System prompt`中抽取出来放到单独的字段里面（LLM支持的话），并且LLM可以自己检查输出是否符合规范，不符合就重试不需要`Agent`重试，这样就可以节省大量token了。

但是有个问题各个LLM厂商的`function calling`并不统一，而且有些厂商还不支持`function calling`所以还是要使用`System prompt`

### MCP

`Agent`和`Agent tool`一般都是放到一个程序里面的，但是如果我们的`Agent tool`非常通用在别的`Agent`中也要使用的话就要在另一个`Agent`在写一遍了，所以我们需要一个公共的服务来处理`Agent tool`

**MCP**（Model Context Protocol，模型上下文协议）是 Anthropic 公司于 2024 年 11 月发布的一个**开放标准协议**，用于统一 AI 模型与外部数据源、工具之间的通信方式

简单来说，MCP 就是 AI 世界的 **USB 接口**——不管什么设备，只要支持 USB 接口，就能即插即用。MCP 也一样，不管什么工具或数据源，只要实现了 MCP 协议，Agent 就能直接使用

`Agent tool`的服务叫`MCP server`，使用这个服务的`Agent`叫`MCP client`，MCP规定了`MCP server`和`MCP client`如何进行通信，以及`MCP server`要提供哪些接口、有哪些`tool`、数据格式、工具描述、参数等等。

MCP也可以直接提供数据`resources`和提示词模版`prompts`

`function calling`和`prompt`是`Agent`和`LLM`的通信方式，`MCP`是`Agent`和`Agent tool`的通信方式

![image-20260425150853863](./AI/image-20260425150853863.png)

上面图片的流程

1. 提问：女朋友肚子疼怎么办？
2. Agent 向 MCP server请求相关的工具列表
3. Agent 将工具列表通过Function calling方式于用户问题一同发给LLM
4. LLM接收到问题后分析得出需要调用`web_browse`工具上网查询资料，将结果发回给 Agent
5. Agent 将调用工具请求发给MCP服务，由MCP server调用工具回去资粮
6. MCP获取资料之后给Agent，Agent继续LLM发送资料
7. LLM结合资料得出答案：多喝热水

### Skills

**Skill**（技能）是预定义的、可复用的**指令集合**，告诉 Agent 在特定场景下应该如何行动。

如果 MCP 是"工具箱"，那 Skill 就是"操作手册"。工具箱里有锤子、螺丝刀、扳手，但你还需要一本手册告诉你"装柜子的时候，先用螺丝刀拧底板，再用扳手固定支架"。

比如说在`MCP server`中有对文件的工具，比如读取文件`form_doc.py form_txt.py form_ppt.py`，写文件`to_html.py to_pdf.py to_png.py`。

现在要将用户传入的文件判断并转换成转换成html，编写一个skill告诉LLM应该怎么工作

`convert_everting/SKILL.md`

```markdown
---
name: convert_everting
description: 将用户输入的文件处理后转换成另一种格式的文件
---

## 执行步骤

1. 根据用户输入的文件格式，寻找合适的脚本进行内容提取。
2. 根据用户的需求，翻译成指定语言。
3. 根据用户要求的输出格式，寻找合适的脚本输出为指定格式的文件。
```

在发送`prompt`时自动添加上

```text
先读取 SKILL.md 中的要求，然后遵守这个要求的前提下，完成下面用户的任务
```

整体的`prompt`就是

```text
先读取 SKILL.md 中的要求，然后遵守这个要求的前提下，完成下面用户的任务把 hh.pdf 翻译成中文，输出 html 文件
```



## RAG 工作流程

[RAG 工作机制详解——一个高质量知识库背后的技术全流程](https://www.bilibili.com/video/BV1JLN2z4EZQ?spm_id_from=333.788.videopod.sections&vd_source=41ed998ac767425fb616fd9071ce9682)

RAG：检索增强生成，先从资料库里检索相关内容，再基于这些内容来生成答案。一般用来做智能客服或者专属知识库

在给LLM添加额外知识的时候如果内容过长就容易丢失上下文，也会消耗大量的token，所以在添加需要对内容进行精简，将问题相关的内容发给LLM

RAG的基本工作流程

### 1.分片

将大的内容分成各个小的部分，可以按`字数/段落/章节/页面`分片。

### 2.索引

1. 通过`Embedding`将片段文本转换为向量
2. 将片段文本和片段向量存入向量数据库中

> `Embedding`：将文本转换为向量的过程，有专门的库来实现这一过程
>
> https://huggingface.co/spaces/mteb/leaderboard

### 3.召回

用户提问，用`Embedding`模型将用户问题转换向量，然后通过向量数据库查询相关性最大的10个相似的结果。

查询相关性的算法有很多种，常用的有`余弦相似度/欧氏距离/点积`

召回采用的是`向量相似度`，这种方法成本低，耗时短，准确率低，适合初步筛选

### 4.重排

将召回查询到的10个相似的结果，进行重排找出最高关联度的3个相关结果

召回采用的是`cross-encoder`，这种方法成本高，耗时长，准确率高，适合精细挑选

### 5.生成

将重排得到的3个结果和问题一同发给LLM得到结果

![image-20260429141450569](./AI/image-20260429141450569.png)

首先创建项目和安装依赖

```shell
mkdir rag && cd rag
uv init .
uv add sentence_transformers chromadb google-genai python-dotenv
```

> + `sentence_transformers`：加载 embedding 和 cross-encoder 模型
> + `chromadb`：一个非常流行的向量数据库
> + `google-genai`：Google 的 AI SDK，调用 gemini-2.5-flash 必备
> + `python-dotenv`：将Gemini API Key 映射到环境变量中

```markdown
# 《魔气斩》修炼秘籍

## 功法总纲
魔气斩，乃以自身魔念为引，汲取天地煞气凝于锋芒，化无形杀意为有形之刃的秘传武学。此功法非正途所容，却威力绝伦，修炼者须先堕魔道，方可驾驭。

## 修炼方式
### 第一阶段：引气入体（三月为期）
修炼者须于每月十五子时，独处极阴之地（古战场、乱葬岗、万年古木之下），以特殊手印引导天地煞气渗入丹田。此阶段需承受万蚁噬骨之痛，心智不坚者极易走火入魔。
_具体法门：盘膝而坐，双手掐“地魔印”，口诵九字真言反转咒，意念观想丹田处有一黑色漩涡缓慢旋转，吸纳周身的暗青色煞气。_
### 第二阶段：凝气成刃（一年为期）
待体内煞气充盈，需寻一头活物（妖兽为佳，牲畜亦可），将双掌贴于其头颅，令体内魔气沿手臂经脉贯出，在掌心前三寸处凝聚成半透明暗刃。初时刃长不过三寸，修至高深可达丈余。
_标志成就：凝刃可维持三十息不散，斩断三寸厚铁板。_
### 第三阶段：化刃为气（三年为期）
此为大成之境，体内魔气与意念完全合一，无需手动凝聚，心念一动即可自周身任何部位激发无形斩击。此阶段修炼者须经历“斩执念”一关——在幻境中亲手斩杀心中最在意之人，否则功法永难圆满。

## 功法威力
- **初成者**（第一至二年）：发出暗色气刃，射程五丈，可断木碎石，对活物造成腐蚀性伤口，普通灵力难以愈合。
- **小成者**（第三至五年）：气刃近乎透明，射程三十丈，可分化为三道同时攻击，斩铁如泥，被击中者魔气入体，神智渐失。
- **大成者**（十年以上）：无形无相，出则无声，可于百丈外取人性命而不觉。最强者可斩开空间裂隙，断人生死轮回。

## 使用限制
1. **寿命折损**：每施展一次强力的魔气斩，折损七日阳寿。全力一击可损耗三月寿命。
2. **魔气反噬**：若一日内施展超过九次，体内魔气暴走，轻则经脉寸断，重则化为毫无理智的妖魔。
3. **心智侵蚀**：长期修炼者会逐渐丧失七情六欲，最终六亲不认，唯余杀意。
4. **天谴劫数**：修至大成者，每十年必遭一次天雷追剿，躲得过续命十年，躲不过形神俱灭。

## 发动条件
1. **前置条件**：修炼者必须亲手斩杀过至少一名生灵（人、妖、魔皆可），以此开启杀意之门。
2. **环境依赖**：月圆之夜威力倍增；正午时分威力减半；光明圣地、佛寺道观中功法失效。
3. **气血要求**：施展时气血充盈为佳，若重伤失血过多或饥饿乏力，则无法凝聚气刃。
4. **情绪触发**：杀意越盛，威力越强。若心怀仁慈或犹豫，斩击会自动消散。
5. **距离限制**：超过百丈后，气刃将自动消散，无法维持形态。

## 修炼禁忌
- 不可与光明系功法同修，否则体内灵气冲突，丹田爆裂而亡。
- 每月初七、二十三两日，煞气紊乱，不可修炼，否则必遭反噬。
- 孕妇及未满十六岁者切勿修炼，胎儿/少年魂魄未稳，会被煞气吞噬。
- 若修炼者真心忏悔、放下杀念，已修炼的魔气斩会全数反噬自身。

## 备注
此功法源于魔道，正道人士得之应毁去，邪道人士得之亦当慎之又慎。须知：**斩人者，人恒斩之；以魔气斩人，终将斩己。**

```

```python
from typing import List
import chromadb
from dotenv import load_dotenv
from google import genai
from sentence_transformers import CrossEncoder, SentenceTransformer

embedding_model = SentenceTransformer("./embedding_models/text2vec-base-chinese",token="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")
chromadb_client = chromadb.EphemeralClient()
chromadb_collection = chromadb_client.get_or_create_collection(name="article")
load_dotenv()
google_client = genai.Client()


## ------------------------------- 准备阶段 -------------------------------

def split_into_chunks(doc_file:str) -> List[str]:
    """
    1.分割: 将文档文件分割成多个段落
    """
    with open(doc_file, "r") as f:
        content = f.read()

    return [chunk for chunk in content.split("\n\n") if chunk.strip()]

def trans_text_to_vector(text:str) -> List[float]:
    """
    2.转换: 将文本转换为向量
    """
    embedding = embedding_model.encode([text], normalize_embeddings=True)[0]
    return embedding.tolist()

def save_vectors_to_db(chunks:List[str],vectors:List[List[float]]):
    """
    3.保存: 将文本和向量保存到数据库中
    """
    for idx,(chunk,embedding) in enumerate(zip(chunks,vectors)):
        chromadb_collection.add(
            documents=[chunk],
            embeddings=[embedding],
            metadatas=[{"chunk":chunk}],
            ids=[f"chunk_{idx}"],
        )

def prepare():
    # 分割文档
    chunks = split_into_chunks("article.md")
    for idx,chunk in enumerate(chunks):
        print(f"[{idx + 1}] {chunk}\n")

    # 转换为向量
    vectors = [trans_text_to_vector(chunk) for chunk in chunks]
    
    # 保存到数据库
    save_vectors_to_db(chunks,vectors)

## ------------------------------- 查询阶段 -------------------------------

def retrieve(query:str,top_k:int = 5):
    """
    4.检索: 从数据库中检索与查询最相似的段落
    """
    results = chromadb_collection.query(
        query_embeddings=[trans_text_to_vector(query)],
        n_results=top_k,
    )
    return results["documents"][0]

def rerank(query: str, retrieved_chunks: List[str], top_k: int) -> List[str]:
    """
    5.重排: 对检索到的段落进行重排，根据查询的相似度
    """
    cross_encoder = CrossEncoder('cross-encoder/mmarco-mMiniLMv2-L12-H384-v1')
    pairs = [(query, chunk) for chunk in retrieved_chunks]
    scores = cross_encoder.predict(pairs)

    scored_chunks = list(zip(retrieved_chunks, scores))
    scored_chunks.sort(key=lambda x: x[1], reverse=True)

    return [chunk for chunk, _ in scored_chunks][:top_k]

def generate(query: str, chunks: List[str]) -> str:
    prompt = f"""你是一位知识助手，请根据用户的问题和下列片段生成准确的回答。

用户问题: {query}

相关片段:
{"\n\n".join(chunks)}

请基于上述内容作答，不要编造信息。"""

    print(f"{prompt}\n\n---\n")

    response = google_client.models.generate_content(
        model="gemini-2.5-flash",
        contents=prompt
    )

    return response.text

def query():
    query_text = "《魔气斩》的使用限制"
    retrieved_chunks = retrieve(query_text)
    reranked_chunks = rerank(query_text, retrieved_chunks, top_k=2)
    answer = generate(query_text, reranked_chunks)
    print(answer)
        
if __name__ == "__main__":
    prepare()
    query()
```



## MCP server

[简易MCP服务](https://developer.aliyun.com/article/1716024)