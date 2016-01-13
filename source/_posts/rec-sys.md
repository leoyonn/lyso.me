title: 推荐系统综述 
date: 2012-09-01 18:44:12
updated: 2012-09-10 18:44:12
permalink: rec-sys
tags:
 - 推荐系统
 - 机器学习
 - rec-sys
 - 个性化推荐
 - 饭饭
 - 大数据
 - 数据挖掘
categories:

---
![mampa](http://7jprdp.com1.z0.glb.clouddn.com/fanfan.jpg)

# 简介

一个完整的推荐系统由三个部分组成：
* 收集模块：收集用户行为数据，例如评分、购买、下载、浏览等行为。
* 分析模块：分析用户的潜在喜好产品和喜好程度，并建立适当的模型以描述用户的喜好信息；
* 推荐模块：用特定的推荐算法从产品列表中筛选出用户喜好的产品排序后给出推荐结果；推荐算法是推荐系统中最为核心的部分。
## 所需数据源

1. 要推荐物品或内容的*元数据*，例如关键字，基因描述等；
1. 系统*用户*的基本信息，例如性别，年龄等
1. 用户对物品或者信息的*偏好*，根据应用本身的不同，可能包括用户对物品的评分，用户查看物品的记录，用户的购买记录等。其实这些用户的偏好信息可以分为两类：
  1. *显式的用户反馈*：这类是用户在网站上自然浏览或者使用网站以外，显式的提供反馈信息，例如用户对物品的评分，或者对物品的评论。
  1. *隐式的用户反馈*：这类是用户在使用网站是产生的数据，隐式的反应了用户对物品的喜好，例如用户购买了某物品，用户查看了某物品的信息等等。

## 推荐算法的分类

### 根据推荐是否因人而异

1. 大众化推荐；
1. 个性化推荐；

### 根据推荐系统的数据源或研究对象方法上分类

考虑如何发现数据的相关性：大部分推荐系统的工作原理还是基于物品或者用户的相似性进行推荐，大致分为如下几种：
1. *基于人口统计学*的推荐（Demographic-based Recommendation）：根据系统用户的基本信息发现用户的相关程度；
1. *基于内容*的推荐（Content-based Recommendation）；
1. *基于协同过滤*的推荐（Collaborative Filtering）；基于协同过滤的推荐可以分为三个子类：
  1. *基于用户*的推荐（User-based）；
  1. *基于物品*的推荐（Item-bsed）；
  1. *基于模型*的推荐（Model-based）；
1. *基于社交网络*的推荐；
1. *混合推荐系统*（Hybrid）；

### 根据推荐模型的建立方式或算法架构的分类

1. *基于物品和用户本身的启发式推荐*（又称memory-based）；
1. *基于关联规则*的推荐：关注用户行为的关联模式。关联规则的挖掘已经是数据挖掘中的一个经典的问题，主要是挖掘一些数据的依赖关系，例 如在线商城的“购物篮问题”，通过关联规则的挖掘找出哪些物品经常被同时购买，或者用户购买了一些物品后通常会购买哪些其他的物品，并基于这些规则给用户进行推荐。一个关联规则由两部分组成：关联决策和置信度。关联决策是指，如果A => B, C, 那么如果Item A出现在某人的历史记录里面，那么可以推断B和C也很可能出现在那里；置信度用以说明关联决策有多么可靠。
1. *基于模型*的推荐；
1. *混合的推荐机制*：其实在现在的主要推荐系统中，很少有只使用了一个推荐算法的，一般都是在不同的场景下使用不同的推荐策略从而达到最好的推荐效果，例如 Amazon 的推荐，它将基于用户本身历史购买数据的推荐，和基于用户当前浏览的物品的推荐，以及基于大众喜好的当下比较流行的物品都在不同的区域推荐给用户，让用户可以从全方位的推荐中找到自己真正感兴趣的物品。对不同推荐算法的混合方法又可分为：加权的混合、切换的混合、分区的混合、分层的混合等等。

## 典型推荐算法概述及优缺点比较

### 基于内容的推荐

是指，根据事先抽取出的产品或用户特征产生推荐，即根据推荐物品或内容的元数据，发现物品或者内容的相关性。有些应用还请专业的人员对物品进行基因编码，比如Pandora在一份报告中说每首歌有超过 100 个元数据特征，包括歌曲的风格，年份，演唱者等等。可以分别对用户和物品建立配置文件，通过分析用户已选择或购买、浏览的物品内容，建立或更新用户的配置文件，系统比较用户与物品配置文件的相关度，并直接向用户推荐与其配置文件最相似的物品。模型有决策树、神经网络等。
基于内容的推荐根本在于信息获取和信息过滤。在信息获取中，表征文本最常用的方法是TF-IDF（term frequency-inverse document frequency），IDF的主要思想是：如果包含词条t的文档越少，也就是n越小，IDF越大，则说明词条t具有很好的类别区分能力。

则文档可以表达成一个向量*dj=(w1j, w2j,..., wkj)*。类似于Lucene搜索的方式。

#### 优点：

1. 对new item没有冷启动问题；
1. 不受打分稀疏性问题的限制；
1. 具有较好的可解释性（展示推荐物品的内容特征）；

#### 缺点：

1. 需要预处理产品以得到代表它们的特征，受信息获取技术的制约、难以处理复杂对象；
1. 无法发现用户并不熟悉但具有潜在兴趣的产品种类；
1. 可扩展性不好（针对一个数据集合训练的数据模型未必适合更大数据，并且加入新数据后效果也有待评定。）

### 基于协同过滤的推荐

是指，收集用户过去的行为以获得其对产品的显式或隐式信息，即根据用户对物品或者信息的偏好，发现物品或者内容本身的相关性、或用户的相关性，然后再基于这些关联性进行推荐。根据前文所述，基于协同过滤的推荐可以分基于用户的推荐（User-based Recommendation），基于物品的推荐（Item-based Recommendation）、基于模型的推荐（Model-based Recommendation）等子类。用户对物品的喜好或评分矩阵往往是一个很大的稀疏矩阵，为了减少计算量，可采用对物品或用户进行聚类的方法，但会降低推荐的准确性。

#### memory-based推荐
基于用户的推荐和基于物品的推荐都是memory-based，即基于系统中的历史记录（如所有被打过分的产品）进行预测，最简单的方法是计算邻居的打分平均值，如公式(1a)；更精确的是计算相似度加权，如(1b/1c)；

* 基于用户的协同过滤

用相似统计的方法得到具有相似爱好或者兴趣的相邻使用者，最早是在1994年由来自美国Minnesota大学Paul Resnick等人发表的《GroupLens: An Open Architecture for Collaborative Filtering of Netnews》一文中提出的。 方法步骤：
 * 收集使用者信息，主要分为两类：
   * 主动评分（显式评分）：基于用户的直接打分数据，如评分，喜爱等级，like/dislike
   * 被动评分（隐式评分）：是根据使用者的行为模式由系统代替使用者完成评价，不需要使用者直接打分或输入评价资料，如电子商务中的购买记录，视频网站用户观看记录、收藏记录，甚至是评论文本观点意见挖掘等进行广泛深度的数据挖掘。

 * 最近邻搜索(Nearest neighbor search, NNS)：
以使用者为基础（User-based）的协同过滤的出发点是与使用者兴趣爱好相同的另一组使用者，就是计算两个使用者的相似度。例如：寻找n个和A有相似兴趣使用者，把他们对M的评分作为A对M的评分预测。一般会根据场景的不同选择不同的算法。目前有多种相似度算法，如皮尔森相关系数、余弦相似度等，在后文介绍。
 * 产生推荐结果：有了最近邻集合，就可以对目标使用者的兴趣进行预测，产生推荐结果。
依据推荐目的不同形式的推荐，较常见的推荐结果有Top-N 推荐和关联推荐。
  * Top-N 推荐：是针对个体使用者产生，对每个人产生不一样的结果，例如：透过对A使用者的最近邻使用者进行统计，选择出现频率高且在A使用者的评分物品中不存在的，作为推荐结果。
  * 关联推荐：对最近邻使用者的记录进行关联规则(association rules)挖掘。

 * 优点：在数据集完善，内容丰富下，准确率较高，而且能够避开物品内容上的挖掘进行准确推荐，对够对物品关联性，用户偏好进行隐式透明的挖掘。
 * 缺点：随着使用者数量的增多，计算的时间就会变长，新用户问题，以及数据稀疏性问题是导致效率与伸缩性上均不足。

* 基于物品的协同过滤（Item-based）

鉴于基于用户的协同推荐算法随着使用者数量的增多，计算的时间就会变长，最早是在2001年由Sarwar提出了基于物品的协同过滤推荐算法《Item-based Collaborative Filtering Algorithms》中所提出的。
基于物品协同过滤在于透过计算物品之间的相似性来代替使用者之间的相似性。所建立的一个基本假设是，能够引起使用者兴趣的物品，必定与其之前评分高的物品相似，举个例子，喜欢《长尾理论》的人，都会去看《世界是平的》。 方法步骤：
 * 收集使用者资讯：同以使用者为基础（User-based）的协同过滤。
 * 针对物品的最近邻搜索：先计算已评价物品和待预测物品的相似度，并以相似度作为权重，加权各已评价物品的分数，得到待预测物品的预测值。例如：要对物品 A 和物品 B 进行相似性计算，要先找出同时对 A 和 B 打过分的组合，对这些组合进行相似度计算，常用的算法同基于使用者（User-based）的协同过滤。
 * 产生推荐结果：在用户使用评价一个商品感兴趣后，会自动搜寻改商品相似度最大的前N项条目。 Item-Based方法总体来说是一种启发式方法，对目标的拟合能力有限。但把多个启发式方法的结果（以及其他的一些特征）做个线性组合，就可以有很好的拟合能力。
 * 优点：
  * 实现简单：不需要用户的历史资料或进行用户识别，复杂的模型在短平快的互联网行业很难施展手脚；
  * 实时响应：相似性要稳定很多，因此可以离线完成工作量最大的相似性计算步骤，从而降低了线上计算量，提高推荐效率，可以实时响应用户的请求，尤其是在使用者多于物品的情形下尤为显著。比如用户新添加了几个感兴趣的商品之后，可以马上更新对用户的推荐；
  * 可解释性：用户总是希望自己有最后的决定权，如果系统推荐的商品不满意，得有办法让用户改进它。采用Item-based方法，很容易让用户理解为什么推荐了某个商品，用户通过在兴趣列表里添加或删除商品，可以调整系统的推荐结果。这可能是其他方法最难做到的一点。
 * 缺点：
  * 以物品为基础的协同过滤不用考虑使用者间的差别，所以精度比较差；
  * 其仍有许多问题需要解决，最典型的有稀疏问题(Sparsity)和冷启动问题(Cold-start)，开始时效果较差。
  * 算法健壮性等问题。

* User-based CF vs. Item-based CF
 * 计算复杂度：User CF 是很早以前就提出来了，Item CF 是从 Amazon 的论文和专利发表之后（2001 年）开始流行，对于一个用户数量大大超过物品数量且物品数据相对稳定的应用，往往 Item CF 从性能和复杂度上比 User CF 更优，因为计算物品的相似度不但计算量较小，且不必频繁更新；而对于诸如新闻，博客或者微内容等物品数量海量且更新频繁的应用中，User CF往往更具优势，推荐系统的设计者需要根据自己应用的特点选择更加合适的算法。
 * 适用场景：在非社交网络的网站中，内容内在的联系是很重要的推荐原则，它比基于相似用户的推荐原则更加有效。比如在购书网站上，当你看一本书的时候，推荐引擎会给你推荐相关的书籍，这个推荐的重要性远远超过了网站首页对该用户的综合推荐。可以看到，在这种情况下，Item CF 的推荐成为了引导用户浏览的重要手段。同时 Item CF 便于为推荐做出解释，在一个非社交网络的网站中，给某个用户推荐一本书，同时给出的解释是某某和你有相似兴趣的人也看了这本书，这很难让用户信服，因为用户可能根本不认识那个人；但如果解释说是因为这本书和你以前看的某本书相似，用户可能就觉得合理而采纳了此推荐。相反的，在社交网络站点中，User CF 是一个更不错的选择，User CF 加上社会网络信息，可以增加用户对推荐解释的信服程度。
推荐多样性和精度：研究推荐引擎的学者们在相同的数据集合上分别用 User CF 和 Item CF 计算推荐结果，发现推荐列表中，只有 50% 是一样的，还有 50% 完全不同。但是这两个算法确有相似的精度，所以可以说，这两个算法是很互补的。
关于推荐的多样性，有两种度量方法 ：
  * 第一种度量方法是从单个用户的角度度量，就是说给定一个用户，查看系统给出的推荐列表是否多样，也就是要比较推荐列表中的物品之间两两的相似度，不难想到，对这种度量方法，Item CF 的多样性显然不如 User CF 的好，因为 Item CF 的推荐就是和以前看的东西最相似的。
  * 第二种度量方法是考虑系统的多样性，也被称为覆盖率 (Coverage)，它是指一个推荐系统是否能够提供给所有用户丰富的选择。在这种指标下，Item CF 的多样性要远远好于 User CF, 因为 User CF 总是倾向于推荐热门的，从另一个侧面看，也就是说，Item CF 的推荐有很好的新颖性，很擅长推荐长尾里的物品。所以，尽管大多数情况，Item CF 的精度略小于 User CF， 但如果考虑多样性，Item CF 却比 User CF 好很多。
  * 用户对推荐算法的适应度：User CF推荐的原则是假设用户会喜欢那些和他有相同喜好的用户喜欢的东西，但如果一个用户没有相同喜好的朋友，那 User CF 的算法的效果就会很差，所以一个用户对的 CF 算法的适应度是和他有多少共同喜好用户成正比的；Item CF 算法也有一个基本假设，就是用户会喜欢和他以前喜欢的东西相似的东西，那么我们可以计算一个用户喜欢的物品的自相似度。一个用户喜欢物品的自相似度大，就说明他喜欢的东西都是比较相似的，也就是说他比较符合 Item CF 方法的基本假设，那么他对 Item CF 的适应度自然比较好；反之，如果自相似度小，就说明这个用户的喜好习惯并不满足 Item CF 方法的基本假设，那么对于这种用户，用 Item CF 方法做出好的推荐的可能性非常低。
* 结合 User CF 和 Item CF：
考虑到精度、多样性：当采用 Item CF 导致系统对个人推荐的多样性不足时，加入 User CF 增加个人推荐的多样性，从而提高精度；而当因为采用 User CF 而使系统的整体多样性不足时，可以通过加入 Item CF 增加整体的多样性，同样可以提高推荐的精度。

#### model-based推荐
user-based或item-based方法共有的缺点是数据稀疏，难以处理大数据量下的即时结果，因此发展出以模型为基础的协同过滤技术，先用历史资料得到一个模型，再用此模型进行预测。以模型为基础的协同过滤广泛使用的技术包括Latent Semantic Indexing、Bayesian Networks等， 收集打分数据进行分析和学习并推断出用户行为模型，进而对某个产品进行预测打分，这种方式不是基于一些启发规则进行预测计算，而是对于已有数据应用统计和机器学习得到的模型进行预测。这种方法的问题在于如何将用户实时或者近期的喜好信息反馈给训练好的模型，从而提高推荐的准确度。
例如Breese基于概率的协同过滤算法：

可以使用的选择概率模型如聚类模型、Bayes网络等。
其他：概率相关模型、极大熵模型、线性回归、基于聚类的gibbs抽样算法、bayes模型，邻居（kNN）模型、受限玻尔兹曼机模型、因子模型（包括矩阵分解模型、二项矩阵分解模型、修正模糊聚类模型）以及模型组合方法等。

#### Slope-One
计算item i和item j之间的相似度并用以预测打分值。

#### 改进Slope one
The BI-Polar Slope One Scheme
The Weighted Slope One

#### hybrid混合方法
混合推荐系统是推荐系统的另一个研究热点，它是指将多种推荐技术进行混合相互弥补缺点，从而可以获得更好的推荐效果。最常见的是将协同过滤技术和其他技术相结合，克服cold start的问题。

* Weighted加权： 就是将多种推荐技术的计算结果加权混合产生推荐，最简单的方式是线性混合，首先将协同过滤的推荐结果和基于内容的推荐结果赋予相同的权重值，然后比较用户对项的评价与系统的预测是否相符，然后调整权重值，形如w(a)*r(a) + w(b)*r(b)... 。这种方法有一个假设的是对于整个空间中所有可能的项，使用不同技术的相关参数值都基本相同。
* Switch切换：根据问题背景和实际情况采用不同的推荐技术。比如，使用基于内容推荐和协同过滤混合的方式，系统首先使用基于内容的推荐技术，如果它不能产生高可信度的推荐，然后再尝试使用协同过滤技术。因为需要各种情况比较转换标准，所以这种方法会增加算法的复杂度和参数化，当然这样做的好处是对各种推荐技术的优点和弱点比较敏感。
* Mixed合并：同时将多种不同的推荐算法推荐出来的结果混合在一起，其难点是如何重排序。
* Feature Combination特征组合：将来自不同推荐数据源的特征组合起来，由另一种推荐技术采用。一般会将协同过滤的信息作为增加的特征向量，然后在这增加的数据集上采用基于内容的推荐技术。特征组合的混合方式使得系统不再仅仅考虑协同过滤的数据源，所以它降低了用户对物品评分数量的敏感度，相反的，它允许系统拥有项的内部相似信息，其对协同系统是不透明的。
* Cascade瀑布型：后一个推荐方法优化前一个推荐方法：它是一个分阶段的过程，首先用一种推荐技术产生一个较为粗略的候选结果，在此基础上使用第二种推荐技术对其作出进一步精确地推荐。瀑布型允许系统对某些项避免采用低优先级的技术，这些项可能是通过第一种推荐技术被较好的予以区分了的，或者是很少被用户评价从来都不会被推荐的物品。因为瀑布型的第二步，仅仅是集中在需要另外判断的项上。另外，瀑布型在低优先级技术上具有较高的容错性，因为高优先级得出的评分会变得更加精确，而不是被完全修改。
* Feature Augmentation特征递增：前一个推荐方法的输出作为后一个推荐方法的输入。例如将聚类分析作为关联规则的预处理：首先对会话文件进行聚类，再针对每个聚类进行关联规则挖掘，得到不同聚类的关联规则。当一个访问会话获得后，首先计算该访问会话与各聚类的匹配值，确认其属于哪个聚类，再应用这个聚类对应的关联规则进行推荐。这里第二种推荐方法使用的特征包括了第一种的输出，而在瀑布型中第二种推荐方法并没有使用第一种产生的任何等级排列的输出，其两种推荐方法的结果以一种优化的方式进行混合。
* Meta-level元层次：用一种推荐方法产生的模型作为另一种推荐方法的输入。与特征递增型的不同在于：在特征递增型中使用一个学习模型产生某些特征作为第二种算法的输入，而在元层次型中，整个模型都会作为输入。比如，你可以通过组合基于用户的协同过滤和基于物品的协同过滤算法，先求解目标物品的相似物品集，在目标物品的相似物品集上再采用基于用户的协同过滤算法。这种基于相似物品的邻居用户协同推荐方法，能很好地处理用户多兴趣下的个性化推荐问题，尤其是候选推荐物品的内容属性相差很大的时候，该方法性能会更好。

### 协同过滤优缺点

#### 优点：

* 不需要预处理产品或用户的特征，故而不依赖于特定的应用领域，能够分析机器难以自动内容分析的数据，可处理复杂对象，如艺术品，音乐等；
* 共用其他人的经验，避免了内容分析的不完全或不精确，并且能够基于一些复杂的，难以表述的概念（如物品品质、个人品味）进行过滤；
* 推荐新物品的能力，User-based方法可以发现内容上完全不相似的物品，即用户潜在的但自己尚未发现的兴趣偏好；
* 推荐个性化、自动化程度高：能够有效的利用其他相似用户的回馈信息，加快个性化学习的速度。

#### 缺点：

* 冷启动问题：对新用户及新物品，无法产生可靠推荐；
* 稀疏问题；
* 可扩展性问题；

## 基于用户-产品二部图网络结构的推荐系统 network-based

基于网络的推荐不考虑用户和物品的内容，仅将其看做抽象的节点。所有算法利用的信息都藏在用户和产品的选择关系之中，如下：

wij表示产品j愿意分配给产品i的资源配额。


# 评估

## 评估方法

应用设计者需要衡量推荐系统使用何种算法时最能够达到其目标，这些决策需要通过实验来辅助。所有的实验场景一般都需要有以下性质：

1. *假设Hypothesis*：依照假设设计实验（例如假设算法A比B的评分预测精度高，则实验测试预测精度，而不预测其他）；
1. *参数控制*：所有不被测试的参数需要保持不变；
1. *一般化能力*：实验结论的普适性，比如用多种数据集来尽量满足这一点。
评估推荐系统的实验方法有三种：离线评测、用户调研和在线评测。

### 离线评测

通过把数据集分成训练集（probe set）和测试集（quiz set），在训练集上学习和调整模型参数，在测试集上进行测试，计算精确度、运行效率等指标，达到评测的目的。这种方法实施简单方便，只需要收集数据，不需要与用户或被试者的交互。但能说明的问题较少，比如不能评估出推荐系统对用户行为的影响。一般用以衡量预测能力，淘汰掉预测不准确的算法。

实施过程：
1. 准备数据：对user、item、rating的选取尽量保证随机和均匀，不能有bias；
1. 模拟用户行为：共有以下几种实施方法：
 a. 按时间序从0开始预测用户选择，并用已经预测的选择作为后续的预测输入；
 a. 随机选择测试组用户及一个时间戳T，隐藏所有T之后的用户数据并做预测；
 a. 同b，但隐藏所有T后加入的物品数据；
 a. 为每个测试用户选一个时间戳T，隐藏此用户在T之后的所有物品，忽略用户间时间序的影响，这种方法注重选择的顺序而非具体时间；
 a. 忽略时间，随机选择测试组用户，为每个用户a随机选择一个na，作为要隐藏掉的物品数量，并随机隐藏掉na个物品。
 a. 更复杂的用户行为模型：用更复杂的用户行为模型可以模拟用户与系统更多的交互行为，但存在以下问题：对用户行为建模较为复杂（[Fischer01] ）；如果模型不准确，可能优化的是完全不相关的性能。

### 用户调研

*实施方法*：召集被试，使之与推荐系统交互，在此过程中对被试进行观察记录，收集量化信息；且根据实验需要可以在被试使用系统前、中、后对其提问问题以收集更多无法观测到的信息。
*优点*：更广阔的解答问题；获取定性数据（解释定量数据的）；通过监控等手段获取更多定量数据；
*缺点*：昂贵（时间/金钱）；覆盖率低；bias（被试知道这是实验）；

### 线上评测

推荐系统的实际效果依赖于多种因素，如用户意图、用户背景知识（对系统的熟悉度、信任度）、用户界面，这些因素在离线评测用户调研中不易重现。一般线上评测可以使用A-B test的方法实施。

结论的可靠性（显著性检验significance testing）：
置信度Confidence 和 p-values：the probability that the obtained results were due to luck. 一般给一个阈值，如0.05（即95%置信度）或0.01等，如果p-value较大则置信度过低，不予采纳结论。p-value一般的计算方法如sign test[Demˇsar06]：


其中n=nA+nB，nA为算法A胜于B的用户数，nB为算法B胜于A的用户数。

## 系统指标

所谓系统指标有两种含义：技术评价指标 和 业务指标。技术评价指标包括诸如RMSE、NDCG，MAP，Recall、Precision等，业务指标也不难定义，例如CTR或者成交转化率，在商城系统中推荐系统为顾客推荐的产品有多少转化成了顾客的购买行为时他们最最关心的，而在音乐，电影，资讯服务的网站中，推荐系统为用户推荐的音乐，电影，资讯有多少被用户点击并且表达出一种喜欢的态度很重要。关键点在于如何让技术指标真实地反映业务指标。
例如Hulu在实验报告《Do clicks measure recommendation relevancy》中讨论了点击率是否适用于评测推荐系统，报告认为在搜索领域有个被广泛认可或验证了的一个假定—— “排在靠前位置的搜索结果得到的点击会比靠后位置的结果多得多” 并不适用于推荐系统。他们的实验表明，推荐产品的排列位置对点击影响甚微，在以NDCG为指标的离线测评中性能好的算法，在在线测评中点击率有可能反而比较低。
目前评估推荐系统的指标（或属性）可分为准确度（accuracy）与可用性（usefulness）两种。

其中准确度衡量的是推荐系统的预测结果与用户行为之间的误差，还可以再细分为 预测准确度（Prediction Accuracy） 和 决策支持准确度（Decision-Support Accuracy）。
预测准确度又可分为评分预测准确度、使用预 预测准确度、排序准确度等，以 MAE、NMAE、RMSE、ARMSE 等常用的统计工具，计算推荐系统对消费者喜好的预测与消费者实际的喜好间的误差平均值；
决策支持准确度则以Correlation（关联度，包括：Pearson、Spearman、Kendall Tau等）、Reversal Rate 、Precision-Recall 、F指标：F = 2PR/(P + R) （同时考察准确率和召回率）、或 ROC curve（Receiver Operating Characteristic）、Swet′A指标（曲线下面积比例）、AVS（Average Ranking Score，平均排序分）等为主要工具。

### 评分预测准确度（Ratings Prediction Accuracy）

RMSE（Root Mean Squared Error，根均方差）是最流行的度量，预测的是用户对每个商品感兴趣的程度，优化RMSE，实际上就是要预测用户对每个商品的评分。

比如Netflix Prize 竞赛，就是以RMSE为度量，竞赛者比Netflix公司之前使用的推荐系统 Cinematch 低（愈低愈好）百分之十，就可获得百万大奖（团队BPC，BellKor's Pragmatic Chaos获奖）。
另一个比较常用的度量是MAE（Mean Absolute Error，平均绝对误差）

两者相比，RMSE对大误差比较敏感。
与之类似的有NMRSE（Normalized RMSE） 和 NMAE（Normalized MAE）：两者的归一化版本，例如可以除以rmax-rmin进行归一化。针对物品或用户在测试集中分布不均的情况可以使用Average RMSE 和 Average MAE，例如对于物品分布不均时，单独计算每个物品的RMSE然后再平均；对于用户分布不均时单独计算每个用户的RMSE然后再平均。
RMSE、MAE仅度量误差幅度，容易理解且计算方式也不复杂，但其缺点也正是失之于简单，过度简化事实，在有些场合可能不能说明问题。例如用户打分1、2的差别和打分4、5的差别都是1，但意义不一样。在这种情况下可以定义适当的扭曲程度度量distortion measure d(ˆr, r)来代替差值以改进度量度量方法。
与之类似的还有MAP（平均准确率，Mean Average Precision），评估是否用户期望相关结果尽可能排在前面。MAP是信息检索中解决P·R·F指标的不足而提出的，单个主题的平均准确率是每篇相关文档检索出后的准确率的平均值，主集合的平均准确率(MAP)是每个主题的平均准确率的平均值。 MAP 是反映系统在全部相关文档上性能的单值指标。系统检索出来的相关文档越靠前(rank 越高)，MAP就可能越高。

其中：U为测试用户集，|U|表示用户集的数目，| Ri |表示用户 ui 相关的项目（如电影）数据， ri,j 表示系统为用户ui 推荐的第 j 个相关项目对于用户 ui 实际的偏好排名。
例如，设有两个主题，主题1有4个相关网页，主题2有5个相关网页。某系统对于主题1检索出4个相关网页，其rank分别为1, 2, 4, 7；对于主题2检索出3个相关网页，其rank分别为1,3,5。则主题1平均准确率为(1/1+2/2+3/4+4/7)/4 = 0.83； 主题 2平均准确率为(1/1+2/3+3/5+0+0)/5 = 0.45；MAP= (0.83+0.45)/2=0.64。

### 使用预测准确度（Usage Prediction）

对评分类的应用可以使用Reversal Rate：首先选择 High 和 Low 两个基准数值，然后比较推荐预测与消费者所给的评分，如果消费者的评分大过 High 这个参考基准，但是推荐系统的预测是小于 Low ，我们叫这种情况是 High Reversal ；反之，则称这种情况为 Low Reversal 。计算 Reversal Rate 可以反映这个系统产生的预测的准确程度。
而对于选择类的应用，可以使用源自信息检索（Information Retrieval）中的Precision-Recall。评分类的应用也可将评分按某阈值分为“选择”与“不选择”后应用此度量方法。 对用户是否选择和是否推荐某个物品有以下四种情况： 


然后计算每一种情况的物品数量，得到以下度量：


这些度量之间是相互关联的，例如提高Recall相应的就会降低Precision。对于推荐数量预定的场合可使用 P@N （Precision at N）度量，而推荐数量未指定的使用PR曲线或ROC曲线，描述的是喜好的物品被推荐的比例。其中：

PR（Precision-Recall）曲线：强调推荐的物品有多少是喜好的，与其对应的统计量分别有F-measure [Rijsbergen79]等；
ROC（Receiver Operating Characteristic）曲线 最早 ROC 是应用于信号处理领域，目的是在于评估一个过滤器（information filtering system）区分讯号（signal）与噪音（noise）的能力，用ROC曲线则强调有多少不喜好的物品却被推荐的。与其对应的曲线有AUC（Area Under the ROC Curve）等（AUC越大，表示系统能够推荐出越多的好的物品）。
已经有学者证明，大部分状况下， 计算 ROC 和 Precision-Recall 时，会得到相同的 confusion matrix ，而且从其中一个曲线，可以推演出另外一种曲线的状况。不过PR 比较适合数据分布高度不平均（highly-skewed）的情况，因此在实际应用中要根据推荐系统选择相应的评估方式。

### 排序准确度（Ranking Measures）

为了能够评估一个推荐系统的排序准确度，首先要给出一个RR（Reference Ranking，参考排序），例如可以将用户已评分物品按照评分降序排列（评分相同的在同一级别）。这样对每两个物品都可以给出一个相对排序：A在B前、A在B后或A与B同级，即如果A与B相对关系未知，则放在同级中。对算法排序结果进行评估时，就应该忽略掉在RR中相对关系未知物品对，可以使用如下度量：
NDPM（Normalized Distance-based Performance Measure）[Yao95]

 
其中，Cu是RR给出相对关系的物品对数，C+和C− 分别是推荐系统排序顺序正确和错误的物品对数；Cu0 是RR没有归为同级但推荐系统归为同级的对数；NDPM如下计算：

NDPM在成功预测每个物品对的序时得到最好分值0；在对每个物品对的序都预测错时得到最差分值1；推荐系统没有给出预测的对的惩罚度仅为预测错的0.5倍；而预测了RR没有给出序的则不被惩罚。
但当用户的偏好完全知晓时，没有序的对就不能预测为有序的，可以使用Spearman's Rou 或 Kendall's Tau [Kendall38,45]


(上划线代表平均值，sigma代表标准差)

除给出参考排序外，还有其它标准度量排序准确度，例如基于效用的排序（Utility-based ranking）：定义推荐的效用可以累加，且一次推荐的效用为推荐中所有物品效用的加权和。根据不同场景有不同的算法：

* 场景1：用户仅需要1个或很少个推荐的物品，认为用户仅查看排序比较靠前的几个，对排序靠后的不太关心。这样一般使用衰减较快的分值，如使用R-Score metric [Breese98]，假定推荐的值沿推荐列表指数级下降，对每个用户u给出如下分数：

ij表示推荐列表中第j个物品，rui是用户u对i的打分。


* 场景2：用户需要所有或大多推荐的物品，衰减速度就需要比较慢，一般使用信息检索领域的一个度量NDCG（Normalized Discounted Cumulative Gain）[J¨arvelin02]，推荐的值沿列表对数下降。


其中DCG*为理想DCG。

## 可用性

推荐系统的预测准确度当然是评估的重要指标，但是“准确度”不是唯一的标准。

### 覆盖率

1. 物品空间覆盖率：
* catalog coverage：所有物品被推荐的比例；
* sales diversity：Gini Index：如果所有物品都被均等的选中，则为0，如果仅1个物品总是被选中，则为1.
* shannon entropy：有n个物品被均匀的选中时为logn，总是1个物品被选中时为0。

1. 用户空间覆盖率：推荐系统能够推荐的用户占总用户中的比率。

# 其他可用性指标

* 置信度Confidence：system trust in its ratings
* 可信度Trust：user's trust in the system recommendation，只能用在线评测或用户调研来评测。
* 新颖性novelty：
* 惊奇性Serendipity：
* 多样性Diversity：
* 实用性Utility：
* 风险Risk：
* 鲁棒性Robustness：
* 隐私性privacy：
* 自适应性Adaptivity：例如item的时效性等；
* 可扩展性Scalability：大数据集
* 推荐效率Efficiency；
* 可解释性Explanation；
* 点击率CTR（Click-Through rate）

## 补充

*推荐长尾*：一个推荐系统的好坏不能直接以预测评分的精确度来测量，而应该以用户的满意度来考虑。推荐系统应该以“发现(Discovery)”为核心终极目标。而现在存在的一些推荐技术通常会倾向于推荐流行度很高的，用户已经知道的Item。这样，存在长尾中的Item也就不能很好的推荐给相应的用户，但是，这些长尾中的Items通常也标注了用户的兴趣偏好。所以，在推荐系统的设计过程中，我们不仅要考虑预测的精度，而且还要考虑用户真正的兴趣点在哪里。
最近，有很多人也开始考虑长尾在推荐系统设计过程中的应用，开始考虑怎样将长尾中的Item推荐给用户。根据[Truong07]，可以将Item聚类成几个small cluster，然后对每个cluster中的item运用CF。利用这一思想，在文章[Park08]中，作者将Item分成head和tail两部分，而针对tail中的Item，作者提出了Clustered Tail算法来做推荐。另外[Ishikawa08]利用Diffusion理论来解决推荐问题。

# 相似度计算算法

关于相似度的计算，现有的几种基本方法都是基于向量（Vector）的：两个向量的距离越近相似度越大。在推荐的场景中，在用户-物品偏好的二维矩阵中，我们可以将一个用户对所有物品的偏好作为一个向量来计算用户之间的相似度，或者将所有用户对某个物品的偏好作为一个向量来计算物品之间的相似度。


## 欧氏距离（Euclidean Distance）

最初用于计算欧氏空间中两个点的距离，假设 x，y 是 n 维空间的两个点，它们之间的欧氏距离是：

n=2时就是平面上两个点的距离。
当用欧氏距离表示相似度，一般采用以下公式进行转换：距离越小，相似度越大：


## 皮尔逊相关系数（Pearson Correlation Coefficient）

皮尔逊相关系数一般用于计算两个定距变量间联系的紧密程度，它的取值在 [-1，+1] 之间。


## 余弦相似度（Cosine Similarity）

余弦相似度被广泛应用于计算文档数据的相似度：


## 矫正余弦相似度

余弦相似度计算并没有考虑到不同的用户的评分尺度差异性，也就是说有的用户评分更宽容普遍打分较高，有的用户评分更严格，普遍打分较低。矫正余弦相似度正式为了克服这一缺点，通过求出每位用户的平均打分，调整评分向量为评分偏差向量，再进行求解余弦相似度。


## Tanimoto 系数（Tanimoto Coefficient）

Tanimoto 系数也称为 Jaccard 系数，是另一种余弦相似度的扩展，也多用于计算文档数据的相似度：


## 相似邻居的计算

相似邻居即根据相似度找到用户-物品的邻居，常用的挑选原则可以分为两类：


### 固定数量的邻居：K-neighborhoods 或者 Fix-size neighborhoods

不论邻居的“远近”，只取最近的 K 个，作为其邻居。如图中的 A，假设要计算点 1 的 5- 邻居，那么根据点之间的距离，我们取最近的 5 个点，分别是点 2，点 3，点 4，点 7 和点 5。但很明显我们可以看出，这种方法对于孤立点的计算效果不好，因为要取固定个数的邻居，当它附近没有足够多比较相似的点，就被迫取一些不太相似的点作为邻居，这样就影响了邻居相似的程度，比如图 1 中，点 1 和点 5 其实并不是很相似。

### 基于相似度门槛的邻居：Threshold-based neighborhoods

与计算固定数量的邻居的原则不同，基于相似度门槛的邻居计算是对邻居的远近进行最大值的限制，落在以当前点为中心，距离为 K 的区域中的所有点都作为当前点的邻居，这种方法计算得到的邻居个数不确定，但相似度不会出现较大的误差。如图 1 中的 B，从点 1 出发，计算相似度在 K 内的邻居，得到点 2，点 3，点 4 和点 7，这种方法计算出的邻居的相似度程度比前一种优，尤其是对孤立点的处理。

## 相似度计算不足与改进

基于评分矩阵相似度计算所面临的一个性能问题，数据稀疏下，精度很差，因为相似度计算是基于寻找拥有共同用户评分的项目或者共同项目评分的用户，在数据稀疏下，两个向量中空值过多，导致计算共同评分维度过低，甚至就没有共同评分。
对于数据稀疏性这种情况下，一个尝试的途径是进行对评分矩阵进行填充。
填充规则有很多，一种通用的方法是，填充均分，具体如下两种：

a 基于用户均分：填充对应用户的平均打分 a 基于项目均分：基于对应项目的平均打分

（基于用户均分填充） （基于项目均分填充）

# 典型应用

## 常用数据集

* grouplens
http://www.grouplens.org/node/12
* netflix
这个数据集来自于电影租赁网址Netflix的数据库。Netflix于2005年底公布此数据集并设立百万美元的奖金，征集能够使其推荐系统性能上升10%的推荐算法和架构。数据集包含了480 189个匿名用户对大约17 770部电影作的大约10亿次评分。网址：http://www.netflixprize.com/。大奖已在09年揭晓，目前数据集已经不提供下载，可以在这里下载到：http://www.lifecrunch.biz/wp-content/uploads/2011/04/nf_prize_dataset.tar.gz
* kddcup 2011 music recommendation system dataset
http://kddcup.yahoo.com/datasets.php
* MovieLens
MovieLens数据集中，用户对自己看过的电影进行评分，分值为1~5。MovieLens包括两个不同大小的库，适用于不同规模的算法：小规模的库是943个独立用户对1 682部电影作的10 000次评分的数据；大规模的库是6 040个独立用户对3 900部电影作的大约100万次评分。
* EachMovie
HP/Compaq的DEC研究中心曾经在网上架设EachMovie电影推荐系统对公众开放。之后，这个推荐系统关闭了一段时间，其数据作为研究用途对外公布，MovieLens的部分数据就是来自于这个数据集的。这个数据集有72 916个用户对1 628部电影进行的2 811 983次评分。早期大量的协同过滤的研究工作都是基于这个数据集的，2004年HP重新开放EachMovie，这个数据集就不提供公开下载了。
* BookCrossing
这个数据集是网上的Book-Crossing图书社区的278 858个用户对271 379本书进行的评分，包括显式和隐式的评分。这些用户的年龄等人口统计学属性(demographic feature)都以匿名的形式保存并供分析。这个数据集是由Cai-Nicolas Ziegler使用爬虫程序在2004年从Book-Crossing图书社区上采集的。
* Jester Joke
Jester Joke是一个网上推荐和分享笑话的网站。这个数据集有73 496个用户对100个笑话作的410万次评分。评分范围是−10~10的连续实数。这些数据是由加州大学伯克利分校的Ken Goldberg公布的。
* Usenet Newsgroups
这个数据集包括20个新闻组的用户浏览数据。最新的应用是在KDD 2007上的论文。新闻组的内容和讨论的话题包括计算机技术、摩托车、篮球、政治等.用户们对这些话题进行评价和反馈。
* UCI知识库
UCI知识库是Blake等人在1998年开放的一个用于机器学习和评测的数据库，其中存储大量用于模型训练的标注样本。

## 商务应用

* 电子商务领域：
Amazon、eBay、淘宝、dangdang、京东商城等；
百分点科技为别人做推荐系统
* 网页标签：
Fab、del.icio.us、Foxtrot等
* 新闻与阅读：
GroupLens、PHOAKS、zite、flipboard、Trap.it(http://techcrunch.com/2012/01/28/trapit-lets-get-personalized/)等；
国内有指阅、牛赞网、无觅网（专门为各大博客和论坛提供相似文章推荐的网站）等；
* 电影：
MovieLens、Moviefinder、Netflix、hulu、豆瓣、猜道等；
国内的一些影视类网站大都有自己的推荐系统，比如奇艺，优酷，土豆等等；
* 音乐：
Pandora、Ringo、CDNOW、last.fm、豆瓣电台、新浪音乐等；
* 餐馆/饮食类：
饭饭@网易有道

## 商务应用举例：Amazon

在amazon的商城系统中多处应用了推荐算法，例如：

* 今日推荐 (Today's Recommendation For You)：通常是根据用户的近期的历史购买或者查看记录，并结合时下流行的物品给出一个折中的推荐。
* 新产品的推荐 (New For You)：采用了基于内容的推荐机制 (Content-based Recommendation)，将一些新到物品推荐给用户。在方法选择上由于新物品没有大量的用户喜好信息，所以基于内容的推荐能很好的解决这个“冷启动”的问题。
* 捆绑销售 (Frequently Bought Together)：采用数据挖掘技术对用户的购买行为进行分析，找到经常被一起或同一个人购买的物品集，进行捆绑销售，这是一种典型的基于项目的协同过滤推荐机制。
* 别人购买 / 浏览的商品 (Customers Who Bought/See This Item Also Bought/See)：这也是一个典型的基于物品的协同过滤推荐的应用，通过社会化机制用户能更快更方便的找到自己感兴趣的物品。

值得一提的是，Amazon 在做推荐时，设计和用户体验也做得特别独到：

* Amazon 利用有它大量历史数据的优势，量化推荐原因。
* 基于社会化的推荐，Amazon 会给你事实的数据，让用户信服，例如：购买此物品的用户百分之多少也购买了那个物品；
* 基于物品本身的推荐，Amazon 也会列出推荐的理由，例如：因为你的购物框中有 xx，或者因为你购买过 xx，所以给你推荐类似的 xx。
* 另外，Amazon 很多推荐是基于用户的 profile 计算出来的，用户的 profile 中记录了用户在 Amazon 上的行为，包括看了那些物品，买了那些物品，收藏夹和 wish list 里的物品等等，当然 Amazon 里还集成了评分等其他的用户反馈的方式，它们都是 profile 的一部分，同时，Amazon 提供了让用户自主管理自己 profile 的功能，通过这种方式用户可以更明确的告诉推荐引擎他的品味和意图是什么。
据VentureBeat统计，Amazon的推荐系统为其提供了35%的销售额。
另外，亚马逊早期推荐系统的主要贡献者Greg Linden在博文《YouTube uses Amazon's recommendation algorithm》中讨论了YouTube在RecSys 2010上的一篇论文，该文报告YouTube的推荐算法主要采用Item-based方法和Ranking方法。

# 当前推荐系统的一些问题及解决办法

## Data Sparsity

### 问题表现

冷启动问题：新物品问题、新用户问题等；
Reduced Coverage问题：rating相对于item来说太少了
Neighbour Transitivity Problem：因为数据量过小，没有用户对相同的item进行过评价，因此没法计算他们之间的相似性（例如在电影推荐系统中，有很多电影只被小部分用户评级，而且这些电影会很少被推荐，即使那小部分用户给予很高评级。同样，对于那些有着不同品味的小众群体，找不到相同特定同口味的用户，也导致较差的推荐结果了）。

### 解决方法

降维技术（Dimensionality Reduction）： 通过奇异值分解（SVD, Singular Value Decomposition）来降低稀疏矩阵的维度，为原始矩阵求的最好的低维近似。但存在大数据量运算成本及对效果影响问题（因为一些insignificant用户或者物品被扔掉了，对这类用户或者物品的推荐效果就要打折扣）。
使用Hybrid方法：混合使用多种推荐方法能弥补其中某种方法的问题，弥补其造成的冷启动问题。
Model-Based方法
Demographic filtering：解决数据稀疏性问题的另一种方法是通过使用用户资料信息来计算用户相似度。也就是，两个用户会被认为相似不只单在相同的电影评级类似，而且也有可能属于同一个人口统计区块（demographic），比如，用户的性别，年龄，居住地，教育情况，工作信息。

## Scalability

### 问题表现

数据量大
对响应时间要求高
数据更新问题：用户从肉食者变成素食者（CF Demographic Content都无法立即反应这种变化）

### 解决方法

降维：SVD（matrix factorization steps代价很大，可使用增量的SVD方法）
Item-Based推荐算法
Model-Based方法（因为有些用户会出在两个类的边缘因此推荐的时候会影响效果）

## Synonymy

### 问题表现：
同一类物品被归为不同的名字（导致数据稀疏性） 

### 解决方法：

intellectual or automatic term expansion, or the construction of a thesaurus
SVD

## Gray Sheep

### 问题描述：
有些人的口味与任何人都不同（Black Sheep是那些口味与正常人完全相反，根本没有办法向他们推荐的人群。因为在现实中也无法解决这个问题，因此这是acceptable failure） 

### 解决办法：
Hybrid（结合content-based和CF，并根据不同的user对两种不同的推荐方法的结果选取不同的权重）

## Shilling Attack

### 问题描述：
作弊（对于自己的东西或者对自己有利的东西打高分，竞争对手的东西打低分——类似于作弊） 

### 解决办法：
Item-Based（在shilling attack这个问题上，Item-based的效果要比User-based的效果要好）
Hybrid（hybrid方法能够在部分的解决bias injection problem）

## Other Challenges

* 隐私问题
* 噪声问题
* 解释性问题

# 参考文献

* [Alag08] Satnam Alag. Collective Intelligence in Action. October, 2008. 424 pages. Manning Publications. URL http://www.manning.com/alag/
* [Ricci11] Ricci, F.; Rokach, L.; Shapira, B.; Kantor, P.B. (Eds.). Recommender Systems Handbook. 1st Edition., 2011, XXIX, 842 p. 20 illus. URL http://www.springer.com/computer/ai/book/978-0-387-85819-7
* [Segaran07] Toby Segaran. Collective Intelligence Programming. O'Reilly Media. August 2007 URL http://shop.oreilly.com/product/9780596529321.do
* [Shi10]Yue Shi, Martha Larson, Alan Hanjalic. Mining mood-specific movie similarity with matrix factorization for context-aware recommendation. CAMRa '10 Proceedings of the Workshop on Context-Aware Movie Recommendation. ACM New York, NY, USA. 2010
* [Wang10]Licai Wang,Xiangwu Meng, Yujie Zhang, Yancui Shi. New approaches to mood-based hybrid collaborative filtering. CAMRa '10 Proceedings of the Workshop on Context-Aware Movie Recommendation. ACM New York, NY, USA 2010
* [Celma10]Òscar Celma.Music Recommendation and Discovery. The Long Tail, Long Fail, and Long Play in the Digital Music Space. Springer Heidelberg Dordrecht London New York.
* [Fischer01]Gerhard Fischer. User modeling in human-computer interaction. User Model. User-Adapt. Interact., 11(1-2):65–86, 2001.
* [Demˇsar06]Janez Demˇsar. Statistical comparisons of classifiers over multiple data sets. J. Mach. Learn. Res., 7:1–30, 2006. ISSN 1533-7928.
* [Herlocker06] J. L. Herlocker, J. A. Konstan, L. G. Terveen, and J. T. Riedl, "Evaluating collaborative filtering recommender systems," ACM Trans. Inf. Syst., vol. 22, no. 1, pp. 5-53, January 2004. [Online]. Available: http://portal.acm.org/citation.cfm?id=963772
* [Sarwar98] B. M. Sarwar, J. A. Konstan, A. Borchers, J. Herlocker, B. Miller, and J. Riedl, "Using filtering agents to improve prediction quality in the grouplens research collaborative filtering system," in CSCW '98: Proceedings of the 1998 ACM conference on Computer supported cooperative work. New York, NY, USA: ACM, 1998, pp. 345-354. [Online]. Available: http://portal.acm.org/citation.cfm?id=289509
* [Rijsbergen79]C. J. Van Rijsbergen. Information Retrieval. Butterworth-Heinemann, Newton, MA, USA, 1979. ISBN 0408709294. URL http://portal.acm.org/citation.cfm?id=539927.
[Yao95]Y. Y. Yao. Measuring retrieval effectiveness based on user preference of documents. J. Amer. Soc. Inf. Sys, 46(2):133–145, 1995.
* [Kendall38]M. G. Kendall. A new measure of rank correlation. Biometrika, 30(1–2):81–93, 1938.
* [Kendall45]M. G. Kendall. The treatment of ties in ranking problems. Biometrika, 33(3):239–251, 1945.
* [Breese98]John S. Breese, David Heckerman, and Carl Myers Kadie. Empirical analysis of predictive algorithms for collaborative filtering. In UAI, pages 43–52, 1998.
* [J¨arvelin02]Kalervo J¨arvelin and Jaana Kek¨al¨ainen. Cumulated gain-based evaluation of ir techniques. ACM Trans. Inf. Syst., 20(4):422–446, 2002. ISSN 1046-8188. doi: http://doi.acm.org/10.1145/582415.582418.
* [Zheng10]Hua Zheng, Dong Wang, Qi Zhang, Hang Li, Tinghao Yang. Do clicks measure recommendation relevancy?: an empirical user study.RecSys '10 Proceedings of the fourth ACM conference on Recommender systems. ACM New York, NY, USA. 2010 URL http://dl.acm.org/citation.cfm?id=1864759&dl=ACM&coll=DL&CFID=64378782&CFTOKEN=90062204
* [Castells11] P. Castells, S. Vargas, J. Wang. Novelty and Diversity Metrics for Recommender Systems: Choice, Discovery and Relevance. In International Workshop on Diversity in Document Retrieval (DDR 2011) at the 33rd European Conference on Information Retrieval (ECIR 2011) (April 2011) http://www.citeulike.org/user/saulvargas/article/9136077
* [Shani09]Guy Shani and Asela Gunawardana. Evaluating Recommendation Systems. Nov. 2009. http://research.microsoft.com/pubs/115396/EvaluationMetrics.TR.pdf
* [Truong07] Truong, K.Q., Ishikawa, F., Honiden, S. 2007. Improving Accuracy of Recommender System by Item Clustering, IEICE TRANSACTIONS on Information and Systems, E90-D-I(9).
* [Park08 ] Y. Park and A. Tuzhilin. The long tail of recommender systems and how to leverage it. In RecSys. ACM, 2008.
* [Ishikawa08] M. Ishikawa, P. Geczy, N. Izumi, and T. Yamaguchi. Long Tail Recommender Utilizing Information Diffusion Theory. In WI-IAT, 2008.
resyschina.org
* 《智能web算法》
《collective intelligence in action》：http://ishare.iask.sina.com.cn/f/14652916.html
* 《集体智慧编程》Collective Intelligence Programming：http://ishare.iask.sina.com.cn/f/16265737.html
* 《Recommender Systems Handbook》：http://ishare.iask.sina.com.cn/f/19991234.html
这本书主要分成五个部分：
 * 第一部分：介绍构建推荐系统的基础理论，比如协作过滤，基于内容的过滤，数据挖掘方法和上下文感知的方法；
 * 第二部分：介绍评估推荐系统的方法，构建实际的推荐系统需要考虑的问题；
 * 第三部分：介绍实际的推荐系统的界面展现相关内容
 * 第四部分：介绍了推挤系统的一个新主题，协作推荐，挖掘UGC相关的内容构建一个新的和更加可靠的推荐系统。
 * 第五部分：介绍了推荐系统相关的一些新的paper，包括推荐系统的对恶意攻击的鲁棒性，收集用户更多的信息和反馈提供更可靠的推荐。
http://www.lifecrunch.biz/wp-content/uploads/2010/11/0387858199Systems.pdf
* 指标详细界定参考论文《Mining mood-specific movie similarity with matrix factorization for context-aware recommendation》及《New Approaches to Mood-based Hybrid Collaborative Filtering》
* 一种通过使用用户资料信息来计算用户相似度的基于传统协同过滤的扩展方法称为demographic filtering。详见M. Pazzani, 《A Framework for Collaborative, Content-Based, and Demographic Filtering, Artificial Intelligence Rev》
* 降维技术：SVD是目前一个已知的矩阵分解技术，来为原始矩阵求的最好的低维近似。详见B. Sarwar, 的《Application of Dimensionality Reduction in Recommender Systems—A Case Study》
