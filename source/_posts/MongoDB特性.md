---
title: MongoDB新特性与案例
tags: [NoSQL]
date: 2023-12-12 22:21:00
---

最近有幸参加了MongoDB的研讨会，了解业界主流的NoSQL数据库——MongoDB的应用案例和模型设计，对于MongoDB的了解又更近一分。
根据下面这份排名，MongoDB在NoSQL家族中已经是当之无愧的老大了：
[https://db-engines.com/en/ranking](https://db-engines.com/en/ranking)
## What is NoSQL?
作为一个计算机的科班生，学习且使用过不少数据库：SQL Server、MySQL、Redis、SQLite、openGauss等等。最初接触到的就是关系型数据库RDBMS，简单的理解就是一些行列组成的表格，一张表中没条数据都是有规则的（有多少列、列数据格式是固定的），表格之间通过关系可以连接起来，可以回想一下ER图中实体之间的如何关联。
而NoSQL则是非关系型数据库，注意这里的No是指Not only的意思，NoSQL的出现并不是为了取代关系型数据库，而是在一些场景下使用RDBMS不合适了，这是采用NoSQL反而会更好。

- 例如Redis属于键值数据库，在高并发场景、热点数据、消息队列、用Redis很好的选择；
- 文档存储，将数据以文档的形式储存，类似 JSON，是一系列数据项的集合。每个数据项都有一个名称与对应的值，值既可以是简单的数据类型，如字符串、数字和日期等；也可以是复杂的类型，如有序列表和关联对象。MongoDB就属于这一类。
- 还有用于存储图数据的Neo4j，注意这里的图并不是图像，而是由节点和边构成的图（数据结构中的图），在推荐系统、知识图谱、社交网络中用的很多。这种图数据如果用RDMS就得使用邻接表、邻接矩阵存储。他们要么查询效率不高，要么存储消耗大。
- 向量存储，MongoDB也推出的Atlas Vector Search也支持对向量存储功能，openAI就用了到这个。在2023年由ChatGPT掀起的大模型热潮，大家都在讨论起人工智能，在数据库上也增加了对AI的支持，这些数据库通过存储模型训练中或是输出的向量存到，可以对这些向量进行搜索、计算相似度、实现动态个性化和LLM的长期记忆。

![20231212222343](https://raw.githubusercontent.com/Chris-Tang6/PicGo-Hub/master/blog/20231212222343.png)

## MongoDB应用场景
MongoDB作为一种NoSQL，具备对许多种非结构化数据的存储：文档、地理信息、时序数据、向量等
在各种行业都有着广泛的应用：

- 游戏：主要是对角色、装备等信息利用文档存储的功能，因为这些数据往往是多变的
- 社交：地理信息、推荐（向量）、评论、点赞、关注信息
- 金融：时序数据
- Web3：数字藏品NFT，类比游戏中的道具
- 视频：视频的meta data，弹幕
- IoT：设备信息，传感器数据（时序）

![20231212222407](https://raw.githubusercontent.com/Chris-Tang6/PicGo-Hub/master/blog/20231212222407.png)

在MongoDB最新释出的7.0版本中新增了向量存储、搜素功能，极大的方便了AI应用的开发：
通过计算两向量的“距离”来筛选出相似的向量：
目前提供了3种距离计算方法：

- 欧几里得距离
- 余弦相似度
- 点积，dot product
![20231212222414](https://raw.githubusercontent.com/Chris-Tang6/PicGo-Hub/master/blog/20231212222414.png)

## MongoDB官网案例的启发

1. 地理位置存储和查询

第一个例子是借助简单的查询规则就能查询到指定范围内的位置点，如果是系统中需要存储地理位置信息，相比RDBMS，这个真的很方便了。
我之前参加比赛开发的一个众包寻找走失老人的平台就需要处理各种位置信息，我们开始在MySQL中存下老人走失的位置（经纬度），然后根据志愿者当前坐标给他们推送走失老人信息；在走失人数不多的时候还好，可以遍历计算所有的距离，筛选出距离在500M内的再推给用户。
![20231212222451](https://raw.githubusercontent.com/Chris-Tang6/PicGo-Hub/master/blog/20231212222451.png)

![20231212222524](https://raw.githubusercontent.com/Chris-Tang6/PicGo-Hub/master/blog/20231212222524.png)


1. 向量存储
![20231212222540](https://raw.githubusercontent.com/Chris-Tang6/PicGo-Hub/master/blog/20231212222540.png)

第二个是关于向量数据库的功能，通过在数据库中查询到与输入prompt相似的向量，喂到LLM。借助先进的ML已经可以提取到prompt的语义表示，这样输入的提示词只要是语义相近的，都能够得到准确的输出。
例如在下面的LLM的例子中，会将预训练的资料向量话存在vector db中，在微调时也就将prompt向量化存储，统一了核心的数据。给LLM提供记忆性；

   1. ChatGPT可以再多个历史对话中切换，并且仍然保持之前的“记忆”
   2. ChatGPT最新的“分身”功能，用户可以使用自己的行业\需求“增强数据”来创建GPT分身，相当于让用户提供自己的数据，openAI就能帮你在这些数据上微调出一个新的GPT

![20231212222556](https://raw.githubusercontent.com/Chris-Tang6/PicGo-Hub/master/blog/20231212222556.png)

在推荐系统中，vector db中存储了各类用户的特征，如果现在需要给一个新来的用户进行推荐，我们可以将新用户向量化表示，然后进行向量搜索，便可以找到“相似用户”，推送给新用户的推荐信息也就更加准确。
在我所做的DTA任务中，实际场景中往往需要预测一种新药物可能的靶标，在以往的实践中需要将这个新药物和数据库中所有的蛋白质组成pairs，然后输入到DTA model预测绑定的亲和力；而有了向量数据库之后，这个预测流程可以进行优化：

   1. 选取DTA模型中药物编码器，对数据库中所有药物进行特征嵌入，并存入到vector db，同时在我们的数据库中还存储由这些药物绑定的蛋白质有哪些。
   2. 此时需要对一个新药物进行预测，我们使用这个训练好的药物编码器，提取这个药物的特征向量，然后使用vector search，找到与其相似的药物，然后根据相似的药物绑定的蛋白质也相似，便可以预测出这个新药可能的靶标；
   3. 这种基于相似性原理的靶标预测，在流程上比起前面的预测要方便，因为只需要计算一个药物特征，然后进行搜索即可；但是这样忽略了蛋白质特征、分子间交互作用对药物靶标绑定的影响。所以可以将其作为一种快速筛选的方式，选出一系列候选的蛋白质分子后，再存储pairs输入到DTA model中预测绑定亲和力。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/21528568/1701925779702-aed12fc4-8a59-46ac-8200-22ceb4c3df11.png#averageHue=%23dcee9d&clientId=u241e97ab-37a6-4&from=paste&height=531&id=ud57bdc73&originHeight=531&originWidth=934&originalType=binary&ratio=1&rotation=0&showTitle=false&size=338257&status=done&style=none&taskId=u4b70c3f0-74c5-4c7f-961a-f0c2d7f3b46&title=&width=934)

3. MongoDB支持多种海内外的云平台，方便应用出海

MongoDB除了在国内与阿里云有合作，对海外3大云厂商全部支持（GCP、AWS、Azure）
在会上，有不上游戏公司都有提到，就是为了解让自家游戏拓展海外业务的。本身在需要游戏上都使用MongoDB来存储Jason文件（角色、道具、武器；这些实体的字段多、变化多、关系性不够明显）

