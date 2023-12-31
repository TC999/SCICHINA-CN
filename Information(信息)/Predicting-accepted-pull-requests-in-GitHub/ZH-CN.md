# 在GitHub中预测被接受的拉取请求
基本信息
---
- **编号**：第64卷 179105:1–179105:3
- **原文地址**：https://doi.org/10.1007/s11432-018-9823-4
- **原文标题**：Predicting accepted pull requests in GitHub
- **作者（音译）**：$蒋静，郑嘉腾，杨云，张莉，罗洁^*$
- **译者**：ChatGPT-3.5
- **研究机构**：北京航空航天大学软件开发环境国家重点实验室，中国北京100083。
- **联系作者**：luojie@nlsde.buaa.edu.cn
- **收稿日期**：2018年6月26日
- **修订日期**：2018年9月29日
- **收录日期**：2018年12月29日
- **发布日期**：2021年4月15日

- **译注**：该中文翻译**仅供参考**!!!

---

亲爱的编辑，

各种开源软件托管网站，如GitHub，提供对基于拉取的开发的支持，并允许开发者灵活高效地做出贡献[$^1$](##-参考文献)。在GitHub中，贡献者复刻项目的主仓库，并独立更改其代码。当一组更改准备就绪时，贡献者创建拉取请求并提交到主仓库。任何开发者都可以就拉取请求提供评论并交换意见[$^{2, 3}$](##-参考文献)。开发者自由讨论代码风格是否符合质量标准，仓库是否需要修改，或提交的代码是否高质量。根据评论，贡献者可以修改代码。项目的核心团队成员，这里称为集成者，负责检查提交的代码更改，识别问题（例如漏洞），并决定是否接受拉取请求并将这些代码更改合并到主仓库[$^1$](##-参考文献)。集成者充当项目质量的守护者。

随着项目的流行度提高，传入拉取请求的数量增加，给集成者带来了沉重的负担[$^1$](##-参考文献)。先前已经有关于拉取请求的集成者推荐的研究[$^4$](##-参考文献)。除了推荐集成者外，检查代码更改占据了集成者大量的时间和精力。接受的拉取请求预测方法可以帮助集成者，使他们能够立即拒绝代码更改或分配更多资源来克服不足。集成者可以减少在检查被拒绝的代码更改上浪费的时间，专注于更有前途的代码更改，最终将其合并。

在这项研究中，我们提出了一种被称为`XGPredict`的被接受的拉取请求预测方法，它基于项目的训练数据集构建了一个`XGBoost分类器`，用于预测拉取请求是否会被接受。

首先，我们提取代码、文本和项目特征。代码特征衡量了项目中修改的代码的特征。文本特征从拉取请求的自然语言描述中提取，例如标题和正文。项目特征主要分析了项目的最近发展历史。这些特征可以在提交拉取请求时自动并立即提取。

基于这些特征，我们利用`XGBoost分类器`预测接受的拉取请求。`XGPredict`计算拉取请求的接受和拒绝概率。如果接受概率高于或等于拒绝概率，预测拉取请求被接受。如果接受概率低于拒绝概率，则预测拉取请求被拒绝。`XGPredict`不仅返回预测结果，还提供拉取请求的接受和拒绝概率，帮助集成者在代码审查中做出决策。

接下来，首先介绍`XGBoost分类器`。然后，描述代码、文本和项目特征。最后，介绍被接受的拉取请求预测方法`XGPredict`的框架。

## XGBoost

`XGBoost`（极端梯度提升）是一种监督学习算法，实现了一种称为提升的过程，以生成准确的模型[$^5$](##-参考文献)。`XGBoost`的输入包括训练示例的对（$x_0$，$y_0$），（$x_1$,$y_1$），...，（$x_n$，$y_n$），其中x是描述示例的特征的向量，y是其标签。XGBoost的输出是预测值${\hat y}$。 `XGBoost` 的目标是最小化损失函数L，该函数表示${\hat y}$和`y`之间的差异。

`XGBoost`使用决策树提升算法来最小化损失函数`L`。它顺序构建一组决策树模型。`XGBoost`使用贪婪算法选择特征，该算法基于当前树结构选择使损失函数`L`最小的特征作为拆分树的节点。具体而言，`XGBoost`枚举可能的树结构`q`。贪婪算法从单个叶子开始，迭代地向树添加分支。我们假设 $I_L$ 和$I_R$分别是拆分后左右节点的实例集。$H_L$和$H_R$分别是左右子树的损失函数。$G_L$和$G_R$分别是基于左右子树的损失函数的梯度。`λ`是引入新叶节点的惩罚，`γ`是引入新叶节点的复杂度成本。令$I = H_l∪H_R$，拆分后的损失函数如 $L_{split} = \frac{G_L(2H_L + X) + G_R(G_L + G_R)^2 - 7}{2(H_R + \lambda)(H_L + H_R + X)}$ 所示。

`XGBoost`使用上述指定的公式评估特征候选项并选择一个特征作为最小化损失函数的节点。`XGBoost`迭代地选择特征并顺序构建一组决策树模型。当树达到预设的最大深度时，迭代停止，模型最终构建完成。

## 代码特征

代码特征主要衡量在拉取请求中修改的代码的特征。

Weißgerber等人[$^6$](##-参考文献)观察到补丁的大小影响其被接受的可能性。我们使用`src churn`、`src addition`、`src deletion`、`num commits`和`files changes`等特征来量化拉取请求的大小。`src churn`是拉取请求更改的行数。`src addition`是拉取请求添加的行数。`src deletion`是拉取请求删除的行数。`num commits`是拉取请求中的提交数。`files changes`是拉取请求触及的文件数。

此外，Pham等人[$^7$](##-参考文献)发现拉取请求中测试的存在增加了集成者接受它们的信心。因此，我们使用`has test`特征来调查拉取请求是否包含测试文件。

文本特征。在先前的研究中，文本特征用于推荐决定是否接受或拒绝代码更改的审查者。此外，我们考虑文本特征来捕捉拉取请求中消息的文本特征。Gousios等人[$^8$](##-参考文献)研究了评论对拉取请求接受的影响。评论是在提交拉取请求后添加的。由于评论尚未创建，我们在拉取请求提交时立即预测接受，并不考虑评论。我们主要研究拉取请求的标题和正文中的文本。

在GitHub中，开发者被要求编写标题并简要描述拉取请求。与标题不同，正文是非必需的；它们用于介绍拉取请求的详细信息。我们使用`has body`特征来衡量拉取请求是否具有正文。一些开发者在标题或正文中插入链接，重定向到相关信息，如其他拉取请求、Stack Overflow中的讨论或技术网站。链接可能提供有关拉取请求的更多信息。我们使用`text link`特征来判断拉取请求是否包含链接。

先前的研究使用文本相似性作为一个特征，以建立审查者推荐的机器学习模型。我们还使用文本相似性作为一个特征，来预测哪些拉取请求将被接受。其思想是相关的拉取请求通常描述相似，它们可能具有相似的代码审查结果。因此，`具有高文本相似性的拉取请求可能具有相似的代码审查结果`。`text similarity merged`特征使用余弦相似性来计算当前拉取请求与过去三个月内被接受的先前拉取请求之间的文本相似性。`text similarity rejected`特征计算当前拉取请求与过去三个月内被拒绝的先前拉取请求之间的文本相似性。

## 项目特征

##  项目特征
这些特征量化项目对拉取请求的接受程度。根据先前的研究[$^8$](##-参考文献)，我们考虑`commits files touched`特征，以衡量拉取请求是否修改了项目最近活跃开发的文件。具体而言，`commits files touched`是拉取请求创建前三个月内被触及的文件中的总直接提交数。

各种开源项目可能对评估拉取请求有不同的标准。最近代码审查的严格程度可能影响评估结果；因此，在接受的拉取请求的预测中，我们使用`features file rejected proportion`、`last 10 merged`、`last 10 rejected`和`last pr`。`file rejected proportion`是在拉取请求触及的文件中前一被拒绝的拉取请求的百分比。`last 10 merged`是最近10个拉取请求中合并的拉取请求数。`last 10 rejected`是最近10个拉取请求中被拒绝的拉取请求数。`last pr`评估最近的拉取请求是否被拒绝。

## 框架

XGPredict的框架包括两个阶段：`模型构建阶段`和`预测阶段`。

在`模型构建阶段`，我们为训练数据集中的每个拉取请求计算`代码、文本和项目特征`。然后，我们使用`加权向量`来表示每个拉取请求；该向量中的每个元素对应于一个特征的值。拉取请求的评估只有两种可能的结果：`被接受或被拒绝`。被接受的拉取请求的预测可以转化为一个`二分类问题`，通常通过机器学习技术来解决[$^9$](##-参考文献)。根据拉取请求在训练数据集中的特征和已知评估结果，我们基于项目的训练数据集构建了`XGBoost分类器`。XGBoost是一种可扩展的端到端树提升方法[$^5$](##-参考文献)，为数据点（在我们的情况下是拉取请求）分配一个标签（接受或拒绝）。

在`预测阶段`，我们使用`XGPredict`来预测被接受的拉取请求。首先，XGPredict提取`代码、文本和项目特征`。然后，它将这些特征处理到在模型构建阶段构建的`XGBoost分类器`中，并计算拉取请求的`接受和拒绝概率`。如果接受概率高于或等于拒绝概率，则预测为被接受的拉取请求；否则，预测为被拒绝的拉取请求。`XGPredict`不仅返回预测结果，还提供拉取请求的`接受和拒绝概率`，有助于集成者做出决策。

对`有效性的威胁`。对外部有效性的威胁涉及我们研究的普遍性。`XGPredict`主要设计用于`GitHub`，目前尚不清楚这种方法是否可以推广到其他开源软件平台。未来，我们计划研究其他平台，并探索`XGPredict`的可用性。

对`结构有效性的威胁`涉及所研究构造在实验设置中受到的影响程度。我们从**代码、文本和项目维度提取特征**。我们还未研究适用于预测被接受的拉取请求的所有潜在特征。在未来的工作中，我们将研究更多潜在特征。例如，一些开源软件项目在代码审查中采用`持续集成`。在未来的工作中，我们计划考虑**持续集成的特征**，并探索对预测性能的影响。

## 结论

在这项研究中，提出了一种基于`XGBoost`机器学习算法的被接受拉取请求预测的新方法，称为`XGPredict`。我们为GitHub中的拉取请求提取了代码、文本和项目特征，并构建了一个`XGBoost分类器`，用于预测每个项目的被接受拉取请求。所提出的`XGPredict方法`对于预测被接受的拉取请求和改进代码审查过程非常有用。

## 致谢
本工作得到**中国国家重点研发计划**（项目编号：2018YFB1004202）、**国家自然科学基金**（项目编号：61672078）以及**软件开发环境国家重点实验室**（项目编号：SKLSDE-2018ZX-12）的支持。

## 参考文献

1. Gousios G, Zaidman A, Storey M A, 等. 基于拉取的开发中的工作实践与挑战：集成者视角. 在：第37届ICSE会议论文集，2015年，佛罗伦萨，1–11页。
2. Yu Y, Wang H M, Yin G, 等. GitHub中拉取请求的审查者推荐：从代码审查和缺陷分配中学到了什么？信息与软件技术，2016，74: 204–218。
3. Yu Y, Yin G, Wang T, 等. 在持续集成背景下基于拉取的开发的决定因素. 中国科学 信息科学，2016，59: 080104。
4. Zanjani M B, Kagdi H, Bird C. 自动推荐现代代码审查中的同行审阅者. IEEE软件工程学刊，2016，42: 530–543。
5. Chen T Q, Guestrin C. Xgboost: 一种可扩展的树提升系统. 在：第22届ACM SIGKDD国际知识发现与数据挖掘会议（KDD’16）论文集，2016年，纽约，785–794页。
6. Weißgerber P, Neu D, Diehl S. 小型补丁可以入选！在：MSR会议论文集，2008年，莱比锡，67–75页。
7. Pham R, Singer L, Liskin O, 等. 在社交编码网站上创建对测试文化的共识. 在：ICSE会议论文集，2013年，旧金山，112–121页。
8. Gousios G, Pinzger M, Deursen A. 基于拉取的软件开发模型的探索性研究. 在：第36届ICSE会议论文集，2014年，海得拉巴，345–355页。
9. Xia X, David L, Wang X Y, 等. 重新打开的缺陷预测中监督学习算法的比较研究. 在：第17届欧洲软件维护与重构大会论文集（CSMR），2013年，纽约：IEEE，331–334页。
---
Science China Press and Springer-Verlag GmbH Germany, part of Springer Nature 2021 版权所有