---
title: 聊一聊最近 AI 技术给气象工作者带来的焦虑
date: 2023-09-20 00:00:00 +08:00
modified: 2023-09-20 00:00:00 +08:00
tags: [气象]
description: 自从盘古气象模型点爆气象圈子以后，各种 AI 预报模型就像雨后春笋一样遍地开花，名字起得好听不说，还争相上了《Nature》。这些 AI 预报模型在赚足了眼球的同时，也迎来了业内的一些质疑，而 AI 预报模型的盛行也真真切切地让很多气象工作者感到焦虑，甚至有一些博主喊出了“留口饭吃”的口号。很多行业专家、学者、KOL都在热烈讨论 AI 预报的这个话题。我也看了很多观点，有一些感触，所以也想就着话题的热度，聊一聊我的看法。
---
*如果把数值天气预报与人工智能预报技术的 PK 比作一把王者荣耀，那毫无疑问人工智能队已经拔掉了二塔。*

自从盘古气象模型点爆气象圈子以后，各种 AI 预报模型就像雨后春笋一样遍地开花，名字起得好听不说，还争相上了《Nature》。这些 AI 预报模型在赚足了眼球的同时，也迎来了业内的一些质疑，而 AI 预报模型的盛行也真真切切地让很多气象工作者感到焦虑，甚至有一些博主喊出了“留口饭吃”的口号。很多行业专家、学者、KOL都在热烈讨论 AI 预报的这个话题。我也看了很多观点，有一些感触，所以也想就着话题的热度，聊一聊我的看法。

## 这次为什么不一样？
说起来 AI 技术在气象领域的应用那可不是新鲜事，得有一些年头了。就据我所知，市面上叫得出名字的短临预报产品，无一例外都是基于神经网络实现的。不管是彩云还是墨迹的分钟级降水预报早在七八年前就已经面世了，而且这么多年过来也比较受业内认可。并没有觉得给气象工作者带来了多么强烈的焦虑感，但为什么今年这一波会给行业内造成如此强烈的心理冲击？

关键词在于“替代性”。

之所以 AI 短临大行其道的时候人们并没有那么强的焦虑，是因为 AI 技术在短临上的应用属于是一种空白的填补，而并不会替代什么原有的岗位。

用过数值预报产品的人都知道，数值预报的时效性是一个硬伤，就拿 NCEP-GFS 和 ECMWF 的数值预报产品为例，前者发布的平均延迟4小时，后者平均7小时。也就是说，它今天早上8点做的预报，你最早得到下午2、3点才能拿到结果。这中间的几个小时实质上是无效预报，但是由于其求解的逻辑限制，你也不能跳过。

而且我们所见到的主流的数值预报产品，最高的时间颗粒度就是1小时间隔，想要制作分钟级的预报以现在的算力和代码架构在工程上根本不可能实现。

所以，一直以来数值预报都没有能力吃到 0-6 小时短临预报这块蛋糕，市面上 AI 短临预报的出现也是填补了数值预报无能为力的这部分空白，它是从无到有的一个过程，也就几乎不会对原有体系造成很大的冲击。

而盘古不一样，它从出现的那一刻就自带了“踢馆”属性。

我记得在去年盘古刚开始立项的时候就已经对外放出消息，要在天气预报领域搞事情。而从结果来看盘古的目标也很明确，初来乍到就是要在数值预报的地盘跟数值预报最强的对手板板手腕，于是它真的直接就去欧洲数值预报中心“踢馆”了，我猜想华为是想来一场气象界的“AlphaGO对战李世乭”的戏码，搞个大新闻。结果大家也都知道了，不仅搞了大新闻，也差点把气象工作者的心态搞崩了。

就目前的现状来说，ECMWF 几乎已经成了业内大部分人的一种“技术信仰”，它的很多产品已经成了气象数值预报领域实质上的标杆。很多机构研发团队在自研格点预报产品以后，也都会去跟 EC 的预报做做对比。所以在这么一个大背景下，盘古对 EC 的冲击，毫无疑问会也会对业界的研发工作者产生心理冲击。

排除技术信仰的因素不说，盘古的预报形式，本质上也就是在以替代数值预报为目的的，它能用排列组合的方式灵活地制作出1小时、3小时、6小时、12小时和24小时间隔的预报，这些都是模仿数值预报的预报间隔，并且运行效率还出奇的快，工程上能极大地减少无效预报的时间，而准确率还与数值预报有得一拼，甚至它可以直接在个人电脑上运行。

简直就是天气预报的卷王。职场上冒出来一个显眼包大家会不喜欢，那预报界来这么一个卷王，谁能受得了啊？所以从现实利益和心理情感的双重角度来看，盘古的出现必然也会受到行业内一定程度的抵制，大家即使嘴上不说，心理也会不服气。

因此对于 AI 预报的质疑声不断出现，比如大家质疑声中出现频率最高的，就是预报的可解释性。

## AI 可解释性问题
AI 的可解释性问题并不是气象预报才有，它是一个“历史悠久”的问题，几乎是与 AI 科学相伴而生的，甚至已经逐渐成了 AI 的一种“原罪”，有很多学者在这个问题上写了不少论文。所以我们首先要承认的是：AI 天气预报结果的可解释性的问题确实存在，并且会长期存在，且难以解决。

但同时我的另一个观点是：即使可解释性不明，但也不会阻碍它在应用领域的发展。就好像目前理论量子力学还有很多问题没有完全搞清楚，但也并不影响量子计算机的研发一样的道理。

在我看来，可解释性问题可能并不是“可不可解释”的问题，而是“我们是否具备了解释能力”的问题。

任何显性的知识都是经历了现象 -> 规律 -> 理论 -> 知识的过程，现代气象科学发展这一两百年时间，几乎所有写到课本里的知识都是从“无法解释”到“可解释”的，如果因为观察到的现象“无法解释”就抵制、排斥，那气象学早就不存在了。所以“可解释性问题”并不应该是 AI 的原罪，而应该是我们发现更大世界总结更多规律获取更多知识的机会。

此外，从现实意义出发，如果我们挣脱“可解释性问题”的枷锁，真真切切能够获得好处和实惠，那又何乐而不为呢？几千年前的老祖宗不知道大气的确切运行原理，但也不妨碍他们利用气候规律发展出发达的农耕文明，一样的道理。

如果我们经过实测 AI 预报的效果确实更好，那么就利用它的优势创造价值就好了，至于其中难以解释的原因，可以留给未来无聊的人生慢慢探索。

可能有人要问了，如果连内在原理都不知道，完全就是一个黑箱，那这预报的结果可信吗？敢用吗？

我的观点是不管黑猫白猫，抓到耗子就是好猫。对于预报结果可信不可信的评判，主要还是应该取决于长期的测评结果，而不应该取决于内在原理是否透明。古代的人不知道万有引力定理，但也不妨碍他们笃信抛到空中的物体一定会落到地面，这就是长期稳定观察得到的结果。所以不管是什么算法，只要它实际测试的预报效果好，并且是长期稳定地好，那当然可以相信，也可以放心地使用。

对于可解释性问题，还有一个观点认为 AI 的黑盒特性会造成其持续发展的路径不清晰，效果好也许只是撞大运。

聊到这里，我想起来之前看到一篇文章中一个有趣的比喻：

“看！我发明了汽车，它跑得比马快！” VS “看！我家骡子吃错了药，它跑得比马快！”

原作者认为盘古当前的表现可能是上述这两种情况中的一种，并且认为我们所需要的是前一种而非后一种。

首先这个例子里前一句是一个后入为主的比喻，它是以现代人已经普遍认知“汽车”概念以后再用回顾古代的方式叙述的。而如果真的让当时一个不知汽车为何物的人来描述，它的说法大概应该是：

“看！我家铁皮通了汽，它跑得比马快！”

如果用这种说法，给人的直观感觉似乎也并不比骡子吃错药好到哪去。当然我在这里并不是想咬文嚼字，而是当我们把二者拉到同一起跑线以后再看这个问题，才更公平。

从实用主义的角度考虑，如果我们首要考虑的是速度，无论是吃错药的骡子还是通汽的铁皮，都是OK的，甚至我们应该让吃药的骡子和通汽的铁皮也比一场，才能最终得出哪个是我们所想要的，不论谁胜出，我们都去用更好的那个就好了。

而且 AI 技术是一个动态发展自我迭代的过程，每年有成百上千的论文发表，也提出了大量的模型方案，这其中必然有吃错药的骡子也有通汽的铁皮，我们只需要在不断试错中淘汰掉那些掉队的骡子，而筛选出我们想要的“汽车”就行了，所以用汽车骡马论来质疑 AI 预报潜力在我看来也是不成立的。

再说回实用主义，在计算机领域，有一个非常有名的命题叫“鸭子类型”，它的描述很有趣：“如果一个实体它叫声像鸭子，走路像鸭子，那它就是鸭子。”乖乖，只要符合我的需求，连鸭子是不是真的都不在乎，也从侧面也可以看出实用主义在计算机领域是多么的普遍，所以永远不要低估一个工程师对实用主义的追求程度，而如今的 AI 技术也确实是在这样一群信奉实用主义的人的手底下成长起来的。

事实上，AI 科学能发展到今天这种程度，巨量的资金砸向这个领域，也都是拜实用主义所赐，如果每个投资人都是追求“可解释性”的严厉学者，那这个行业早就胎死腹中了。

## AI 是否会替代数值预报

还记得文章开头那个比喻吗？

> 如果把数值天气预报与人工智能预报技术的 PK 比作一把王者荣耀，那毫无疑问人工智能队已经拔掉了二塔。

之所以说 AI 只拔了二塔，是因为目前以盘古为首的各种 AI 大模型虽然取得了一定程度的成功，但是由于它在数据的种类的丰富度上并不能完全覆盖 NWP（Numerical Weather Prediction，数值天气预报，下同）的预报要素，所以 AI 预报目前仍然不具备完全替代 NWP 的实力，但是逐渐蚕食的过程已经在进行之中了。

很多人认为目前的 AI 模型严重依赖再分析数据，而再分析数据是 NWP 体系下资料同化的产品，所以当下的 AI 模型预报也不是真正独立于 NWP 的体系，而是寄生于 NWP 体系的一个分支。这个观点乍一看确实没有问题，因为从现在公开的资料来看，AI 预报大模型的训练数据几乎无一例外都是用 ERA5 数据作为训练集的。但问题是目前 AI 模型训练之所以都用 ERA5 再分析数据，是因为真实观测数据极度缺乏，除了 ERA5 几乎没有其他可选项。如果把 AI 天气大模型比作是一个婴儿，那 ERA5 就是它的婴儿车，而标志婴儿长大第一件事就是丢弃婴儿车。所以你怎么确定 AI 未来在真实观测数据充足的情况下，不能憋个大招来一个彻彻底底的端到端预报呢？如果你认为 AI 永远都离不开再分析数据，那格局就有点小了。

而且，说现在 AI 预报模型依赖再分析数据也不完全准确，因为从工程的角度来说，现在的模型在推理阶段实质上是与数据来源无关的，只要它的输入是按照预先设定好的变量种类顺序、维度组织的多维数组，就可以执行推理，它并不在乎输入的矩阵是来自什么机构的什么产品，不同的矩阵输入大概就类似于原装电池和山寨电池的区别，现在也有很多人在尝试用多种不同来源的输入场来驱动盘古以比较其预报效果。所以这些模型严格意义上并不是被卡脖子的技术。

从长远来看，我认为 AI 大概率是能够替代 NWP 的，而且是全链路的替代，包括资料同化。或者换一种说法，AI 最终追求的将是前面提到的端到端预报，也就是直接给模型输入非标准化的观测资料，模型直接吐出最终的预报结果，把同化和预报的界限打破，最终形成一套完全独立于 NWP 的预报体系，这几乎是 AI 进化的一个必经之路。

当 AI 的模型真的实现了这种端到端预报，并且预报效果能追平 NWP。那就说明 AI 预报已经从婴儿车里跳了出来，可以独立行走了。到那个时候，直接的面对面竞争才算真正开始。

*而到那个时候，AI 就攻上了高地。*

## AI 预报的阶段性展望 & NWP 还有什么机会
从短期来看（未来1年以内），除了更多基于 ERA5 的同质化模型跳出来发论文以外，应该不会有太多亮眼的新闻。可能比较值得期待的是国家队的表现，事实上现在这些模型无一例外都是基于欧洲的 ERA5 数据做的训练，而国家队实际上手里还握着比 ERA5 质量更好（中国区）的再分析数据 CRA40 没有放出来。如果能够以这套再分析做训练，是否可以得到更好的结果？值得期待。

从中期来看（未来1-5年），一定会陆陆续续出现独立于 NWP 体系的端到端 AI 预报大模型出来，各方应该都在憋大招，就看谁先放出来了。保守估计这个时间可能会在一年到一年半之间。

从长期来看（未来5-10年），NWP 将迎来与 AI 的直接竞争，并且 AI 模型可以凭借其数据驱动的灵活性，在资料同化、常规预报、气候预测、后处理、质控等气象预报的各个阶段和细分领域大放异彩，同时它也会不断在更终端的位置补齐从气象数据到用户真实需求的最后一公里，从而在气象的长尾市场里攻城略地，最终成为行业的主流方案。

上个月跟一个朋友骑车的时候聊过 AI 与 NWP 竞争的话题。他的一个观点我觉得是挺有道理的。他认为对于 NWP 来说，只要能够提高运算效率，计算速度能达到与 AI 的运算相同量级，那么 NWP 也是未来可期的。

事实上，确实有人在这方面做努力，比如我们都认识的一个前同事，就正在一家云计算公司里研发改造 WRF 让其在 GPU 上运行的项目，而且据说已经有所突破。

假如说，当 NWP 能够利用 GPU 实现计算性能大幅提升，提升到与 AI 的计算性能相当的水平，那未来的结局可能还真未可知。当然，即使最终 NWP 在业务预报领域难以抵挡 AI 预报的竞争，也并不意味着它没有任何生存空间。

打个不那么恰当的比方，我们可以认为 NWP 是在追求一种数学上完美的解析解（推导过程），而 AI 是通过不断迭代试错寻求一种近似值的数值解（无需推导）。众所周知，数学上大多数方程都是没有解析解的，在应用数学和工程领域，数值解才是主流。

那么同理 NWP 其实更适合在理论研究领域发挥作用，它可以用于对各种气象假说或猜想的计算机模拟验证，这一点是 AI 模型所无法做到的（黑盒）。所以学术研究将是 NWP 的一块无人能撼动的自留地。

## 人生苦短，我用 AI
我认为气象工作者并不需要焦虑，因为 AI 技术最终会是一种普惠的技术。它并不会让你失业，相反，它会让你工作得更幸福。

举个例子，比如说程序员，ChatGPT 出来以后，很多人也说程序员要被替代了。但从我的观察来看这件事似乎并没有发生，反而程序员们在 Copilot 的“按摩”下过上了神仙般的日子。

你可以想象一下，未来某个团队，基于开源大语言模型，加上几乎全部的天气过程分析的论文再加上全部的气象教材做语料，同时结合标签化的观测数据做多模态训练，训练了一个叫“首席”的 AI 模型，他每天自己去把观测数据拿下来，然后给你一通分析，最后把天气分析报告和结论都给你写好了，你只需要看一看没啥问题签个字然后发布就行了，这日子不美吗？而这个“首席”模型不也正是预报员真实需求的“最后一公里”吗？

AI 无法替代人的一个最重要的现实因素是，AI 不能作为责任体来承担责任，通俗来讲就是“AI不粘锅定理”，气象行业是一个需要背锅的行业，AI 没有软肋，不领工资，无法开除，你能让它背锅？所以说气象工作者还是无法替代的。

最后，人生苦短，用上 AI 好好享受生活它不香吗？