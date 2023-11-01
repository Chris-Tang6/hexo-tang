---
title: openGauss用户操作模型-结项报告
tags: [项目, openGauss, ]
date: 2023-09-02 17:13:05
---

## 项目信息
项目要求探索openGauss已有的机器学习模块和用户操作模型构建算法，完成技术洞察博客一篇。 在审计日志中提取出用户操作时间、操作客体等相关信息。根据用户操作信息提取用户操作特点，完成程序开发，实现用户操作画像功能，完成设计文档。
### 方案描述
首先我们明确一下将用户的画像问题看作一个多分类任务，确定好模型的输入输出。模型的输入是设计好的用户操作特征，模型输出的是该用户的类型。而要从原始的审计日志到用户特征需要经过特征工程模块；在用户操作模型中选用的是openGauss中以逻辑回归为基础分类器的多分类模型。![用户操作模型](https://cdn.nlark.com/yuque/0/2023/png/21528568/1689931339917-568c1737-b9d2-4e40-b013-776b846ec967.png#averageHue=%23f9f7f4&clientId=u862145d8-5212-4&from=paste&id=uc944a0f2&originHeight=302&originWidth=1217&originalType=url&ratio=1&rotation=0&showTitle=true&size=44504&status=done&style=none&taskId=u1ea3bcff-0c69-4d9d-91db-57306164042&title=%E7%94%A8%E6%88%B7%E6%93%8D%E4%BD%9C%E6%A8%A1%E5%9E%8B#averageHue=%23f9f7f4&id=rtGFC&originHeight=302&originWidth=1217&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E7%94%A8%E6%88%B7%E6%93%8D%E4%BD%9C%E6%A8%A1%E5%9E%8B "用户操作模型")
整体的流程如下图所示：

1. 首先需要收集指定时间段的审计审计日志
2. 通过分析改日志来筛除一些无效的数据，设计一些用户特征，例如用户的一次操作需要包括主体、客体、操作类型（哪个用户对谁做了什么）等信息；
3. 接着需要处理数据集，例如给类别数据编号，划分数据集；
4. 使用为了评估模型分类的好坏，这里选择准确率作为指标；
5. 根据指标的好坏，我们可以调整模型的超参数，使得模型在训练集上分类得更好；
6. 选定好超参数后模型就可以使用了，之后数据库中又会产生许多的用户操作日志，我们可以重新收集日志来迭代模型；![用户画像执行流程](https://cdn.nlark.com/yuque/0/2023/png/21528568/1689930304365-e9e46716-bd55-4c3c-aa54-8af83ed6a634.png#averageHue=%23f8f8f8&clientId=u862145d8-5212-4&from=paste&id=uf685c4ef&originHeight=812&originWidth=1005&originalType=url&ratio=1&rotation=0&showTitle=true&size=73439&status=done&style=none&taskId=u79639d8f-757c-4152-935f-d420127696a&title=%E7%94%A8%E6%88%B7%E7%94%BB%E5%83%8F%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B#averageHue=%23f8f8f8&height=609&id=M6WJ7&originHeight=812&originWidth=1005&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E7%94%A8%E6%88%B7%E7%94%BB%E5%83%8F%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B&width=754 "用户画像执行流程")
### 时间规划
**6.26 - 7.9，前期调研**

1. 安装openGauss
2. 学习openGauss中AI特性、审计日志
3. 搜集常用的用户画像模型
4. 搜集数据库审计日志数据
5. 对调研内容进行总结形成技术博客

**7.10 - 8.15，项目开发第一阶段**

1. 撰写项目设计文档
2. 从审计日志进行清洗，从中提取用户操作时间、操作客体等信息
3. 使用AI模块对清洗后的数据提取特征，进行用户画像
4. 模型调优

**8.16 - 9.30，项目开发第二阶段**

1. 整理并解决中期检查中发现的问题
2. 对第一阶段的开发进行测试和完善
3. 继续完善文档
## 项目进度
### 已完成工作

- 对openGauss中审计日志、AI特性调研，完成两篇技术博客
- 完成项目设计文档
- 通过模拟一个多用户的教务管理系统获取到审计日志数据
- 对审计日志进行筛选、分析，统计了日志中主体、客体、操作类型的频数分布
- 基于openGauss中提供的AI模块构建了用户操作模型
- 基于用户操作模型实现了两个应用：用户画像、识别危险用户
### 遇到的问题及解决方案
#### 获取审计日志数据
我们的目标就是使用审计日志来进行用户画像，但是因为自己先前并没有使用过openGauss数据库手上没有现成的日志数据，于是只好在网上找找看有没有别人公开的。但是对于开发者来说数据库的日志属于很机密的数据很少会公开出来，查找了几天都没有结果，便只好自己模拟出来。所以在日志数据获取上花费的时间要比预期多些。
那具体怎么模拟呢？首先确定的是需要一个多用户操作的数据库，不然只给一名用户画像就没意义了；其次模拟既要尽可能的贴近真实同时也要简单。最后我选择设计一个多用户的教务管理系统并使用python脚本来自动模拟用户的操作，模拟结束后在openGauss中导出审计日志。
**数据库设计**
![](https://cdn.nlark.com/yuque/0/2023/png/21528568/1688179079909-62ff4099-5583-4b60-a785-1ff39dc1c060.png#averageHue=%23f9f9f9&clientId=ue9f5a0b9-ad20-4&from=paste&height=1022&id=u3bbe4e2d&originHeight=1022&originWidth=1502&originalType=binary&ratio=1&rotation=0&showTitle=false&size=115219&status=done&style=none&taskId=u53db3b75-75f7-40ef-8519-4068c15efa7&title=&width=1502#averageHue=%23f9f9f9&height=767&id=SlM13&originHeight=1022&originWidth=1502&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=&width=1127)
**多用户设计**
opengauss：超级管理员，创建用户pm
pm：项目架构师；负责设计、创建数据库、表格、视图；创建下面的用户 
t_coder：老师端，主要对成绩表增、改，对教师表查询
s_coder：学生端，成绩、学生表查询
affair：教务处，负责增删学生表、老师表、课程表
#### 用户画像怎么实现
一开始看到要对用其实并不好怎么下手，于是我调研了一些用户画像的案例：

| 案例 | 算法 |  |
| --- | --- | --- |
| [银行贷款用户的画像](https://www.heywhale.com/mw/project/5ed9c13fb772f5002d6dc07c) | 二分类 | 统计不同类型卡的用户年龄、收入分布实现画像；最后训练模型决定是否放贷 |
| [商店会员画像]() | 统计学 | 统计各年龄、性别消费者消费行为，以及不同时间段的消费行为 |
| [酒店会员行为分析](https://www.kaggle.com/code/liamwoodly/eda-prediction) | 二分类 | 根据以往的预定信息判断是否会取消预定 |
| [黑客画像预警模型](https://dl.ccf.org.cn/article/articleDetail.html?type=qkwz&_ack=1&id=5460231971866624) | 聚类 | 提取黑客的攻击时间、攻击方式、目标特征，进行聚类，建模群体画像 |

可以看到大多对用户行为的数据经过统计学分析，才确定好用户的特征，这些特征就是对用户的画像；接下来可以使用机器学习模型对用户进行分类。于是我认真分析了审计日志，并统计了主体、客体、操作类型的分布。
原始的审计日志中每条数据包括下面这些属性：
```
time: 操作的时间戳
type: 操作类型
result: 执行结果
user_id: 用户ID
username: 用户名
database: 操作涉及到的数据库
client_conninfo: 客户端连接信息,
object_name: 操作的客体，例如用户是查询的哪个表，创建的哪个数据库，给哪个用户赋权等
detail-info: 详细的操作信息
```
![type字段统计](https://cdn.nlark.com/yuque/0/2023/jpeg/21528568/1689042106419-5f21bae8-05ac-4d94-8585-3089a042066a.jpeg#averageHue=%23f9f9f9&clientId=u7e264589-bb57-4&from=paste&height=463&id=u95efe2f8&originHeight=2071&originWidth=2307&originalType=binary&ratio=1&rotation=0&showTitle=false&size=205448&status=done&style=none&taskId=uab06e9d3-4bbb-4eee-bb23-c4713d7d78e&title=&width=516#averageHue=%23f9f9f9&height=518&id=SGb2V&originHeight=2071&originWidth=2307&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=type%E5%AD%97%E6%AE%B5%E7%BB%9F%E8%AE%A1&width=577 "type字段统计")
依据前面对操作类型的观察，可以发现操作主要分为：

- **系统级**别的操作，如登录，创建用户，分配权限，设置参数
- **数据库级**操作，创建数据库，模式，表
- **表级**操作，对表的增删改查

因此，在设计操作特征的时候，也按照这些类别创建了各类操作的视图（所有视图均是统计每天的次数），最后便得到了下面的用户特征表，每一条数据表示某一天该用户的操作特征，包括一下8个属性：

- query_login，用户每天的登录次数
- query_sys，除了登录操作，其他的系统级操作次数
- query_db，数据库级的操作次数
- query_insert_all，所有对表的插入操作
- query_insert_score，所有对score表的插入操作
- query_insert_score，所有对score表的插入操作
- query_sel_info，各类信息表的查询
- query_sel_score，对成绩表的查询

![用户操作特征](https://cdn.nlark.com/yuque/0/2023/png/21528568/1689043106450-c9e76bcb-3fad-4491-9f63-163318767599.png#averageHue=%23eaeeb6&clientId=u7e264589-bb57-4&from=paste&height=365&id=u7eece92c&originHeight=365&originWidth=895&originalType=binary&ratio=1&rotation=0&showTitle=true&size=26882&status=done&style=none&taskId=u385bde95-dbce-452a-920c-c0284dcee1e&title=log_refined%E5%90%88%E5%B9%B6%E6%89%80%E6%9C%89%E6%93%8D%E4%BD%9C%E7%89%B9%E5%BE%81%E7%9A%84%E8%A7%86%E5%9B%BE&width=895#averageHue=%23eaeeb6&id=FYD51&originHeight=365&originWidth=895&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E7%94%A8%E6%88%B7%E6%93%8D%E4%BD%9C%E7%89%B9%E5%BE%81 "用户操作特征")
从日志中提取到操作特征后，计算出训练数据中各个特征的下四分位值_a_25_作为我们的判定阈值。即有75%的数据是大于这个a_25；同时防止该特征出现的很少例如操作数据库的特征，绝大部分数据都会等于0此时，对于这种情况我们手动设置该阈值为1。如果用户在该属性上大于阈值就赋予对应的标签于是就实现了用户画像。
![用户画像](https://cdn.nlark.com/yuque/0/2023/png/21528568/1690164189609-2541fb66-bffc-475e-af43-7b908739247d.png#averageHue=%23090705&clientId=u0ffef368-6ac2-4&from=paste&height=402&id=u31c1d3fc&originHeight=402&originWidth=724&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39358&status=done&style=none&taskId=u1d7804c6-4b07-4175-b672-706550a6d98&title=&width=724#averageHue=%23090705&id=ABMg5&originHeight=402&originWidth=724&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E7%94%A8%E6%88%B7%E7%94%BB%E5%83%8F "用户画像")
最后我还提出了使用用户操作特征的来识别危险的用户，具体做法是训练一个机器学习模型让模型学习区分各种类型的用户，这属于多分类模型，如果遇到异常的用户行为，由于与以往该用户的行为有区别，模型便会将其判断出来时可疑的用户。
```plsql
SELECT id, username, date,
PREDICT BY log_m1 (FEATURES login, sys, db, insert_all, insert_score, sel_info, sel_score)
 	as "PREDICT",
name_id as "LABEL"
FROM log_test2  where "PREDICT"!="LABEL" ;
```
例如在下面的测试中，将学生端的insert_score属性改为了1，模拟学生端用户修改了成绩表，这是与异常操作，最后我们的模型确实能够识别出来。
![](https://cdn.nlark.com/yuque/0/2023/png/21528568/1689999783279-6885413d-8123-4e63-b029-264a6c42d35c.png#averageHue=%23090604&clientId=uc27e51f0-0ebc-4&from=paste&height=224&id=ufc7fd24f&originHeight=224&originWidth=1097&originalType=binary&ratio=1&rotation=0&showTitle=false&size=17543&status=done&style=none&taskId=ucd3bbee4-e03e-46b6-85fd-9c4ebb8d330&title=&width=1097#averageHue=%23090604&id=nHQ7R&originHeight=224&originWidth=1097&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
![识别可疑用户](https://cdn.nlark.com/yuque/0/2023/png/21528568/1689999736604-8ce52013-fbf7-4305-b5a3-1efe00f80cda.png#averageHue=%23090706&clientId=uc27e51f0-0ebc-4&from=paste&height=85&id=u15cbe58a&originHeight=85&originWidth=499&originalType=binary&ratio=1&rotation=0&showTitle=false&size=4571&status=done&style=none&taskId=ub12e1d95-938b-4d62-ab95-ee3b76335a7&title=&width=499#averageHue=%23090706&id=nwSNY&originHeight=85&originWidth=499&originalType=binary&ratio=1&rotation=0&showTitle=true&status=done&style=none&title=%E8%AF%86%E5%88%AB%E5%8F%AF%E7%96%91%E7%94%A8%E6%88%B7 "识别可疑用户")
## 总结
特别感谢开源之夏还有openGauss组织提供的宝贵实践机会，参加这次活动让我收获了很多。在几个月的时间内不断驱动自己学习新知识来完成项目，为国产开源数据库openGauss贡献出自己的一份力量。同时也感谢汤学明老师、李浩然师兄的倾情指导。通过不断与浩然师兄交流想法，改进代码，最终完成了任务。
