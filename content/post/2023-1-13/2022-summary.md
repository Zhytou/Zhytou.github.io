---
title: "2022年总结"
date: 2023-01-13T00:52:50+08:00
draft: false
author: Zhytou
---

- [22春招](#22春招)
- [后22春招学习](#后22春招学习)
- [最后的大四时光](#最后的大四时光)
- [毕业啦](#毕业啦)
- [旅行](#旅行)
- [愚蠢的决定](#愚蠢的决定)
- [IELTS与申请出国](#ielts与申请出国)
- [10月之后的学习](#10月之后的学习)
- [后记](#后记)
- [TODO-List](#todo-list)

## 22春招

考完研之后我也没有完全放开玩，还是认真的在收集信息谋划我的未来。在此期间，我又去仔细地了解了出国的事项，因为我发现通过出国留学最终成功在国外上班正是最符合我对未来生活预期的。我也恨自己过去只模糊地感觉出国读书好贵，完全没有认真的调研就做出了一个草率的判断。通过这些了解，我又决定找一份互联网工作，并用互联网的高工资来支撑自己出国留学，这样似乎是我最好的选择。

我在2月开始正式复习，主要以分类做Leetcode算法题和看面经为主。3月底开始投递，最后只收获了俩个因裁员临时终止面试的遗憾和一个TP-Link的保底offer。具体情况如下：

|投递公司| 笔试/面试结果 | 个人反思 |
|-------|--------------| -------- |
|  腾讯 |    无校招HC   | |
|  字节 |    一面挂     | 当时脑淤血了，想着用字节来适应面试强度，因为之前实习面试三面都感觉难度适中，没有复习好就上场了。|
|  美团 |    没投递     | 最主要的原因是Java实习的时候没学好，担心面试被拷打。虽然实习也修了两三个bug，但还是非常不熟悉。|
|  网易 |    简历挂     | |
|  百度 |    简历挂     | |
|  虾皮 |    笔试挂     | 虽然我4题里面3题过了，但还是挂了。 |
|  Akuna Capital | 一面挂 | 第一次英语面试，非常紧张。错了一个没有scoped_lock的死锁问题加上优先队列实现失败（虽然我大概知道堆排序怎么写，但我根本没想到，也是因为这场面试才有了MultiThreadPractice和SimpleSTL这两个Toy Project）。|
| TP-Link | Offer | 大白菜，甚至比我完全没复习直接收走的做硬件的同学工资低。|
| Zoom | 笔试挂？ | 做完笔试完全没有任何消息，不过我当时心态已经爆炸，间歇性摆烂和焦虑，甚至是后悔没练到这个点。|
| 小红书 | 三面中止 | 基本上已经稳拿offer的一个了，不过受裁员潮影响，直接GG。 |

现在看下来，排除裁员潮的客观因素，我还犯了几个致命错误：

- 心高气傲。抬手上来就是用字节面试机会热身，纯纯的把机会浪费了。也可能是21年实习面试的时候给我增长太多自信心了。
- 投递策略失败。虽然我个人当时，认为我已经把我能投的公司全投了，其实现在看来，那真是投递的太少了。特别是和那些投递四五十家的同学来说，我在投递策略上真的是大失误。有很多我自己一直追求的WLB杭州公司都没投递，跟别说还有不少苦一点但钱还行的公司也没投递。当然这跟我信息源匮乏也有关系，也间接促使我后面学会经常去关注工作方面的信息。
- 心态不好。从最初的自信满满必拿下，到最后因为越来越渺茫的上岸机会而心态爆炸。我真的是开始飘的有多高，后面摔的就有多惨。
- 复习准备不足。过分相信面经和所谓的总结了，忽略了算法和语言基础。当时实习面试给我一种错觉，让我认为面试官着重考察的是八股文背的好不好。实际上，这只是一种面试风格而已，甚至可以说是不推崇的一种面试风格。更多的面试官还是青睐于考察算法和语言基础。此时我这300出头的Leetcode刷题量就略显不足了。此外，我C++的使用缺乏工程实践，或者说很多公司中常用的点我根本不熟悉甚至没听说过。还有很多概念我在当时只是知道个大概用法，这也是显得我基础很差的重要原因。

## 后22春招学习

针对当时面试出现的问题，我又进行了一波总结，写到了我的[CS-Notes](https://github.com/Zhytou/CS-Notes)里面。特别是，重读C++ Primer和参考[system design repo](https://github.com/donnemartin/system-design-primer)对系统涉及进行的总结，当时写完感觉自己又升华了。还有关于算法分题型总结的那部分，我自己感觉也写的特别好。

同时，我在春招复习期间接触到了[CS自学指南](https://csdiy.wiki/)，于是趁着复习结束有空余时间就开始了学习。虽然很早就受到CC98中胡神的影响，开始了解和学习公开课。事实上，我就是通过CS106L学习C++的。但这个自学指南与我之前所了解的信息相比，更加完善和丰富。同时受知乎上另一位迟先生的影响，开始对数据库产生兴趣，想要多了解数据库相关知识，或者说想成为像迟先生一样成功的人。总之呢，在结束春招之后，我开始学习CMU-445。虽然到毕业设计结题答辩之前（到现在也是）我只完成了两个lab，但掌握了一种新的学习方法，加之我确确实实做出来两个lab，我当时是非常开心的。而且自己也更想要暑假狂学一场，以便自己好好在研究生大展拳脚。不过后面的事情，却打破了我的如意算盘。

另外一点，我前面也提到了我觉得自己非常缺乏C++的工程经验，特别是看到自己当时简历上那几个项目有点寒碜（全是课程大作业）。于是找个C++项目练手丰满简历的想法也就应运而生。后来，碰巧刷到Milo Yip的[从零开始的JSON教程](https://zhuanlan.zhihu.com/json-tutorial)，我也由此开始模仿着做了一个[AtomJson](https://github.com/Zhytou/MyJsonParser)。这个项目总共花了我大概40多个小时（wakatime记录的纯写时间，不包括查阅资料等等）。写完这个项目之后呢，我反正非常开心，感觉自己C++又熟悉了不少。

除此之外，我在这期间还捡回了在公司学会的git使用，并且把它用来管理自己的一些项目。然后完善了自己github profile包括建站和开始使用wakatime记录代码时间等等。

## 最后的大四时光

![校园](https://zhytou.top/post/2023-1-13/zju.jpeg)

寒假结束回学校之后，为了不让我自己的本科生涯留下遗憾，我加入了学院的篮球队。虽然说上场时间不多而且经常被队友批评漏洞，但还是快乐和骄傲的吧。唯一的遗憾就是没有让DJ来看一次我的比赛。

最后一学期最让我难受的肯定是我的毕业设计。被迫选择了个自己完全不熟悉的天线设计作为自己的毕业课题，而且老师也特别严格。我一度感觉自己做不出来毕不了业了，好在最后还是在父母的鼓励下艰难地完成了毕业设计。只不过我还是因为重复率超过20%而被迫延期到二次答辩。虽然说这件事肯定怪我自己，但是在屎上雕花的降重是真的太恶心我了，我完全做不下去。

## 毕业啦

毕设答辩顺利结束、毕业照、毕业典礼。大学四年就这样结束了。虽说今年已经回忆过很多次自己的大学生涯了，但这里还是简单写下吧。

大一和大二的时候我还搞不清状况，还是按照高中的学习方法一股脑地往课程上堆时间。结果自然是虽然努力的但绩点还是不太好看，甚至一度产生摆烂的想法。现在回想起来，感觉大一大二真应该好好的去感受一下大学生活，参与各种各样的文体活动、广泛地结交朋友，而不是让自己的生活里只剩学习。

进入大三之后，可能是因为我发现我保研无望而并且真的是不喜欢那些信号处理的课程。于是，我改变了自己之前的学习乃至生活的方式——不喜欢的课凑合就行、多出门和朋友去玩玩、多去打打自己喜欢的篮球。特别是从校内论坛98中接触了转码，我开始把大量的时间花在学习C++，并且有计划的去选择了一些计院开设的软件课程。转码良好的就业前景是一方面原因，但更重要的是我在写代码时真的很有成就感。那段时间可能是我大学过的最爽一段时间了吧，有压力也有动力、有开心也有焦虑。功夫不负有心人，我也在大三下收获了美团实习offer，并在21年暑假在美团实习了两个月。

结束实习之后，我意识到以我现在的情况出去就业，尤其是去大厂其实完全就是炮灰，发展空间很小。加上我导师的生活状态——研究生毕业工作两年就结婚生子，下一步是家里支持买套房。我只感觉喘不过气来，这种生活好像都没为自己活过啊。此外，大厂on-call和ping的制度也真的是在把员工当干电池在压榨，自己的不受别人打扰的时间变得很少。于是，我决定考研读硕，积累经验的同时，看看大厂之外的可能性。之后就是2022年的故事了。

现在回头看来，大三那年做的选择还是很幼稚，或者说我信息源太匮乏才让我做出这么二极管的选择。我大可以上班之后跳槽，或者挣钱出国深造。本身已经拿到挺好的结果，却硬生生放弃换了另外一条路。

## 旅行

六月初的时候，和DJ一起去了躺千岛湖。

![温州之行](https://zhytou.top/post/2023-1-13/wenzhou.jpg)

之后，紧接着和DJ、老余还有卷毛一起去了趟温州洞头当作我们的毕业旅行。

紧接着之后就和DJ一起回了成都。回成都当然就是我尽地主之谊了，带她去吃去玩，然后和我高中时期的好朋友打麻将，顺便见了一直活在对话里的睿姐。不过比较可惜的是，DJ没有吃到乐山菜。

## 愚蠢的决定

![考研](https://zhytou.top/post/2023-1-13/exam.jpg)

3月刚出考研成绩的时候，我还只是稍微有点小遗憾，毕竟离上线只差一分。而且是碰上今年专业爆热，院线狂涨20多分的情况下，我甚至还觉得自己有点牛逼，毕竟自己也没准备多久。因为我之前想的是考研保留学生身份，进而争取继续成长，刷更多实习经历，所以我也不太焦虑。毕竟我已经想好了如果考不上，我参加春招就是了。而且我之前考完研也就是马不停蹄开始准备了，所以我还是对自己有信心的，但是现实是真的残酷啊。

我春招的结果非常惨不忍睹，加之太久没有经历面试。我后期心态已经爆炸，甚至觉得自己就是一手好牌打得稀烂的典型。于是，当听到有调剂机会的时候，我几乎都没怎么想就同意了，感觉像抓住了救命稻草一样。只可惜这个决定只让我轻松了两个月。七月份，一通电话打过来叫我回学校做暑期科研。我当时才和DJ到家没两天，而且我暑假已经打算按照CS自学指南狂学公开课了，感觉这学院非常NT。不过我还是去了，通过和实验室学长交流加上我自己的观察，我感觉这里是个大坑，不仅学生毕业有困难（小老板组内一个七年博士，一个六年博士），而且是以工程为导向的（大老板开了公司），最后我小老板方向还和我差比较远。加上之前我为了能留在杭州方便实习，选择调剂的还是个类似硕博连读的项目（根本没有读完的打算，当时就想读硕士），我对四月份做的决定感到深深的绝望。

再接连联系了之前学院的辅导员、实习的同事和出去工作的同学之后，我发现如果我想退出这个项目，目前只有三种选择：1、和导师摊牌，争取保留读硕的机会，但会和我之前的设想有很大出入，因为实验室没有放人实习的先例，但因为招聘形势恶化，实习又是必须的，所以也算一个不太完美的选择。2、直接放弃继续读书，转而去工作，但在我联系了好几个公司之后，这条路基本上也被堵死了。很多公司根本不认我应届生身份，而且招聘形势恶劣，我很难找到一份还算满意的工作。而且父母也非常不支持我这样做，他们完全不能理解要丢弃学位的做法。3、暂时接受现状，积极准备出国，去香港读一年制水硕，既可以有时间刷实习经历，而且还能完全做自己想做的事，不被什么导师束缚。但这条路的问题是钱和不确定性，因为春招的大失败让我感觉如果一年后环境持续恶化，我真的不太确定我自己能找到一份还行的工作。到头来又是浪费青春加金钱，然后拿到比本科毕业还差的东西。

算是仔细考虑了之后吧，我还是决定遵从内心选择我喜欢的自己掌控时间的路。因此，我在抓紧完成暑期科研的相关工作之后就赶快回到了成都和父母交流我的出路。那段时间真的是煎熬，好在是周末还可以去千岛湖找DJ玩，有她和我父母的支持，我才能继续前进。

## IELTS与申请出国

回到成都之后，我和父母交换了意见。当时我精神状况不太对，最糟糕的几天甚至是早上三四点醒来就睡不着了，只能麻木的看b站让我脑子不要去想那些糟心事情。乃至于之后我还人生第一次去传说中的成都市四医院。总之，就是在这种精神状态下，我敲定了中介，然后开始准备IELTS。每天就是从早上起来刷题一直刷到晚上，不让自己想太多其他糟心的事情，专注在这个英语考试上。这期间唯一一件让我现在都挺开心的事就是了解到Episoden这个网站。当时我基本每天都上线和上面的外国友人聊天，把自己的经历告诉他们，总之就是聊天说地，能很好的缓解我的焦虑。至于具体的复习方法和总结，可以参考我的另一篇博客[IELTS学习总结](https://zhytou.top/post/2022-10-18/ietls-learning/)。

十月中旬拿到语言成绩之后，我就开始和中介联系准备文书。期间最让我难受的其实是找老师拿推荐信，唯一一个之前给他做过科研的老师还不是我们学院的。好在最后还是有一个课程老师愿意帮我。我还是顺利的完成了文书的准备以及投递。

## 10月之后的学习

> [wakatime-2022-report](https://wakatime.com/a-look-back-at-2022/a7b329b7-d489-40d2-9239-8be7cf83b65e/ptrvgjpvtt)

在完成出国申请的准备之后，我终于有时间回到自己的代码学习当中来。在这期间内，我其实最主要的工作就是完成了6.824的lab2——实现了Raft一致性协议，并且完成了一篇[总结](https://zhytou.top/post/2022-11-14/raft/)。而完成6.824的lab1和lab2一共花了我快90个小时。

之后因为一直好奇迭代器的实现，索性参考着STL模板库剖析写了个SimpleSTL的Toy Project来帮助自己理解和强化模板类写法。

## 后记

2022对我来说算是波澜起伏的一年吧。虽然说自己也思考了很多，也对人生有了新的规划，但自己对未来还是感到很迷茫。有些时候想起来这些事情，感觉自己又回到了7月份那个炎热的青年旅舍里。但无论如何，2022暂且就是过去了，希望新的一年能万事如意吧。

天赋也是一种诅咒。越是觉得自己看到了别人看不见的东西，越会被蒙蔽双眼。

最后放上我对23年的期待吧！

## TODO-List

**Coding**：

- [ ] 参加两段实习
- [ ] 参加一次OSPP或GSoC
- [ ] 继续刷题和面试保持状态
- [ ] 完善之前的一些Toy Project，包括：
  - [ ] MultiThreadPracticeInCPP
  - [ ] SimpleSTL
  - [ ] SimpleRenderEngine
- [ ] 给开源社区提一次PR
- [ ] 找到自己的方向，可以通过读paper和做项目来实现。潜在的几个方向包括：
  - [ ] 数据库或基础架构方向：1、做完6.824；2、读DDIA；3、参加TinyKV训练营；4、寻找一份数据库方向实习。
  - [ ] 图形学或GPU加速：1、完善SimpleRenderEngine；2、阅读相关文献（待了解）；3、学习CUDA和shading language，了解openCL和openGL协作；4、参与Taichi开源社区；4、寻找一份图形学方向实习。
  - [ ] QT开发：1、重新捡起QT，完成相关笔记；2、相关实习。
  - [ ] 云原生容器及Web3方向：1、做完6.824；2、读DDIA；3、补足go基础知识（go秒杀）；4、相关实习。

**With my girl**：

- [ ] 去影楼拍照片，算是某种莫名的理想？
- [ ] 去听gali或者其他某个我们都喜欢的饶舌歌手的Live
- [ ] 两个3-5天的短期旅行（复刻温州之行）

**English**：

- [ ] 继续和Episoden上的朋友聊天
- [ ] 参加一次英文的mock面试
- [ ] 准备Gre

**Health**：

- [ ] 体重降回到大三时期
- [ ] 继续篮球