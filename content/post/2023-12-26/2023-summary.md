---
title: "2023年总结"
date: 2023-12-26T14:19:14+08:00
draft: false
---

- [Timeline](#timeline)
- [Workout](#workout)
- [Coding](#coding)
- [TODO-List](#todo-list)
- [Outro](#outro)

## Timeline

**CG大作业**：

22年学校因为疫情很早就安排学生回家了，我大概是12月19号到的家。接着一个月都在写那一学期课程的大作业，其中耗费我最多精力的就是图形学的大作业——[SimpleRenderEngine](https://github.com/Zhytou/SimpleRenderEngine)，一直拖拖拉拉的写到我去虹软实习。虽然效果是依托答辩，但我是真的尽力了，当时大晚上还发了条[推特](https://twitter.com/_ZhoY_/status/1624779747271245824)。

**艰难的决定**：

大概是过年的时候吧，我接到了香港几个学校的Offer，我大概有两周时间决定是不是接受这个机会。尽管最终我还是决定了放弃出国就安安心心的在学校读研，但我也因为自己糟糕的状态和相处3年的女友分了手。那段时间是真的挺糟糕的，好在是工作和运动把我拖出了泥潭。不过也感谢这段时间，让我爱上了运动并在整个虹软实习期间都坚持了下来。

![图1 滨江夜景](http://zhytou.github.io/post/2023-12-26/binjiang.jpg)

现在回过头去看，最开始准备出国是因为发现实验室很烂，如果按部就班的完成那个硕博联培的项目，按实验室前辈的经验我大概率要延毕；因为大四毕业暑假才开始急急忙忙的准备申请出国，根据家庭经济条件和自己的意愿选择了港新这种好申请的区域；再后来研一报道，自己去和导师、学院沟通争取到了只读硕士，不读博士的机会；然后我又在焦虑到底出国还是读研，因为疫情三年发生的荒唐事加上互联网疯狂裁员让我对在国内找工作、生活产生了不少的绝望，但是我也不敢拿爸妈的积蓄去香港赌一个看起来收益不高的博。说到底，我还是思路不够清晰，目标不够坚定。从21年决定转码去美团实习，然后发现互联网大头兵迟早要完转去考研延长学生生涯，再到发现实验室是个大坑打算出国，似乎我每个时间节点都做了看似正确的选择，但到最后我还是会焦虑痛苦。或许是我过去的路都太顺利了，这也让我对自己总是期待不低，进而导致常常心态失衡。不过经历了这些糟心事之后，我自认为还是成长了，只希望25年我能比之前做的更好吧。即使结果不尽如意，也尽量要做到自己无怨无悔吧。

**Arcsoft实习**：

开年之后，我就回到了杭州准备租房入职实习，最终在江陵路附近找到一个月租近2000的短租房。虽然小区环境不怎么样，但好在离公司很近，每天上班骑共享单车也就20分钟左右。我有的时候下班还去逛一逛滨江天街，买点菜和水果，然后回家自己做两个菜。虽然工资不太高，但同事和领导人都挺不错的，而且实习生也不用担心加班。我几乎就是到点就走。加上写Qt+GLSL的工作对我来说也算简单，所以实习那段时间总体还是挺舒服的。唯一需要担心的就是导师突然找我，让我去华家池干活之类的事。好在我老板放养的确实彻底，我请假的次数也不算多。

![图2 自己做的菜](http://zhytou.github.io/post/2023-12-26/food.jpg)

说回实习期间的具体工作，我主要完成了三项工作。一是使用Qt和OpenGL开发一个简单的图像标注工具。这个工具旨在取代团队目前正在使用的MarkTool工具，并且通过引入OpenGL来实现颜色空间的快速转变。二是基于QNX和OpenGL的车载系统图形库开发，也就是下面那张图片我正在干的事情。三是对团队视频标注工具的软件解码、视频帧提取、播放组件进行了优化，利用双缓冲进行软件优化，并修改了原有的FFmpeg解码逻辑。

![图3 Arcsoft工作](http://zhytou.github.io/post/2023-12-26/arcsoft-work.jpg)

实习期间还有一件让我高兴的事是我和本科关系好的朋友又经常联系了。因为我忽然发现我住的地方离他们的所在的校区，也就是科创中心也不远，于是我又经常和他们打球吃饭了。庆幸充实的工作以及和朋友们一起玩耍的快乐让我从1月份那种焦虑情绪中抽离出来，生活也走向正轨了。

4月底快清明节的时候，我和lh、tl两兄弟一起去hcy家玩，这大概是我上半年最开心的时光了。错开五一高峰期，加上抓住禁渔期尾巴，还有个本地人带我们逛吃逛喝，甚至还去了个亲戚家蹭了一顿饭，简直不要让这次旅游太爽。面线糊和炸醋肉是真好吃啊，有机会一定还要再去一次福建玩。

**摆烂的5、6月**：

结束Arcsoft的实习之后，我摆烂休息了两周。接着就是忙着处理一些乱七八糟的事，什么补交党政资料、OSPP项目申请之类的。在了解和选择OSPP项目期间，我也尝试了去加入CubeFS开源社区。虽然也仔细看了文档尝试去部署并且试着提了两个[issue](https://github.com/cubefs/cubefs/issues/2024)，但因为实在是有点远离自己平时的应用而且我和相关项目导师沟通的时间有点晚，所以也没能顺利参与OSPP。

在这之后，我也把去年没写完的6.824的项目捡起来继续完成。期间也看了一些课程推荐的论文，最终形成了三篇博客：[基于Raft搭建一个简单KV存储服务](https://zhytou.github.io/post/2023-6-3/build-a-kvstore-on-raft/)、[数据库概念总结](https://zhytou.github.io/post/2023-6-11/introduction-to-database/)和[从GFS论文了解分布式文件系统](https://zhytou.github.io/post/2023-6-17/gfs/)。紧接着就是一个很临时起意的回家，本来想给爸妈一个惊喜的，结果回去就感冒了。和朋友们吃了两顿饭打了个麻将。感觉大家都好厉害，出国的出国、工作的工作、二战考研的也顺利上岸，我自己也要加油。待了一周就匆匆会杭州了，不过返程的时候顺道去了趟重庆。

![图4 重庆](http://zhytou.github.io/post/2023-12-26/chongqing.jpg)

**实验室任务、学习新内容和准备下一次实习**：

转眼到了7月中旬，我按照着[CS自学指南](https://csdiy.wiki/)先后学习了NJU-OS和CS144，并且完成了其配套的实验，同时也产出了几篇博客。尤其是CS144这门课，它几乎是重塑了我对网络的认识，让我把之前死记硬背的好多知识给理解了；另外，NJU-OS的参考教材[OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/)也让我对操作系统中的许多知识有了更深刻的认识。总之这两门课程还是给了我很多收获的。

临近暑假的时候，被老师安排去写一个上位机的软件。本来以为是件比较容易的事情，当时还在考虑还要不要趁暑假再回次家或者直接就接个实习去上班。谁知道这个横向的需求真的是一大堆，而且测试的人也是拖拖拉拉的，明明两周之前就写完的东西，非要等我马上要放假了来恶心一下我。好在是这个软件整体难度都比较低，我也趁此机会把Qt拿来练了练手，算是巩固了在Arcsoft学到的东西。最终也是在8月底的时候，我和老师要到了假期，就跑到上海去见世面了。

那段时间里，我也一直关注着实习的机会。因为上一段实习的原因，我对实习的选择更加谨慎。我当时的想法是至少要和前两段是不同的方向和不同类型的企业，因为我希望体验不同领域的工作。我给自己划定的可能的工作领域包括：

- 数据库：因为自己做过MIT6.824和CMU15-445，而且当时PingCAP作为很有名的开源公司，我在推特上看到挺多厉害的人都在那里工作，所以自己也想去看一看。还有一点是98之前有个叫冰冰的小冰的大佬也是杭州数据库公司工作，并且他也经常宣传他的公司——DolphinDB。
- 量化 & 银行：前者无他，钱多事少；后者稳定，据说也是一个不错的选择。
- 自动驾驶：非常火爆，看起来像一个冉冉升起的新领域，特别是22年比亚迪校招收了很多offer。
- 游戏 & 图形学：第一点是这个方向都是以C++语言为主；第二点是我之前上过CG的课程，而且大作业也做的特别折磨，也想去看看工业界都在干嘛。

那段时间也面试了不少公司，各种各样的都有，天天就是在boss直聘上给HR们发简历求捞，最终很幸运的得到了一次百亿私募白鹭资管的实习机会。

**量化实习**：

![图5 上海](http://zhytou.github.io/post/2023-12-26/shanghai.jpg)

在实习正式开始前，我先是到公司的上海分部待了三天，算是学习和参观，后续则是回到杭州分部正式入职。上海分部的具体办公地点位于陆家嘴的花旗大厦顶层。第一次去的时候，既兴奋又紧张，活脱脱的刘姥姥进大观园。虽然对量化行业福利好和薪资高早有所耳闻，但员工福利和办公环境让我觉得好的有些不可思议。可以看到外滩江景的落地窗、可升降的办公桌和两块超大显示屏；提供早午餐、各种各样的零食和饮料；给新人也发很多文化衫、文创；几乎从不加班等等。至于工作内容，主要都是由我的领导昊驰总安排的，总体都比较轻松。除了最开始的组播接行情项目，我因为一个测试把整个公司的网络搞瘫痪了两三个小时，好在是处于休市时间，没造成经济损失，不然篓子就捅大了。现在想起来还真的有点后怕，也算是给我上了一课——要时时刻刻注意自己的操作，就像当初在美团处理脏数据那样，先进行线下测试，然后影子库测试，最后才是生产环境灰度上线。

回到杭州实习没多久就迎来到十一小长假，我和龙哥也是约好两天自驾游，强度非常适合我们这种懒但看朋友圈容易心动的社畜。我们从杭州出发，先到淳安，吃了我心心念念的鱼头，沿着千岛湖畔逛了逛；晚上就赶到衢州，又是一顿逛吃逛吃，吃了烤饼、鸭头、水晶糕，第二天起床又去了衢州孔子博物馆，又是一顿逛吃逛吃；下午返程杭州，途径兰溪，逛了诸葛八卦村，然后吃了太爷鸡。总结：我个人感觉衢州很不错，特别适合能吃辣的朋友去特种兵旅游，小吃实惠好吃，地方不大，景点集中，甚至可以靠双腿走。

![图5 衢州](http://zhytou.github.io/post/2023-12-26/quzhou.jpg)

三个月的实习转眼就结束了。虽然上班时天天想着赶紧润，但真到分别的时候似乎又有些不舍。但考虑到我的毕业小论文，我还是毅然决然的拒绝了HR让我继续实习的建议。回顾一下这段实习，我大致完成了四个主要项目，分别是Basecom、Asset-Monitor、HFT-App和EMS-Monitor。其中，Basecom是一个C++的基础工具库，提供CSV读写、交易日快速计算、配置文件解析以及字符串split和strip功能。Asset-Monitor是一个用于监控和对比实时市值与目标市值的脚本工具。HFT-App是对回测系统和策略模型解耦的一次尝试。具体来说，就是HFT-Sys写死，只通过动态链接的方式加载不同model，并利用model类名实例化对象，调用内部函数完成回测功能。EMS-Monitor是一个基于Python Pandas和Bokeh交易监控与风险管理系统WEB界面。总的来说，HFT-App应该是我这段实习中技术层面最有挑战性的项目了，它的核心就是在使用动态加载的前提下，实现反射，即：在程序的运行状态中，仅根据类名就可以实例化该类对象的机制。总的来说，我这段时间掌握了pandas常见功能，可以熟练进行数据清洗、转换和分析计算。例如：用DataFrame实现行列选择、数据类型转换、为空值处理等；利用Series对数据做统计和滤波，并且可以使用它们配合matplotlib和bokeh进行可视化展示。同时，我对C++的反射实现、动态链接也有了更深的认识，并且接触到一些模板元编程的技术与constexpr all the things的思想。除此之外，我在金融领域方面也收获了一些知识，包括基金投资的盈利模式和股票期货交易的常见“黑话”等等。

**年末学习**：

在12初结束实习之后，我先是报复性的躺了一周。然后就在打游戏打到百无聊赖中，开始继续学习CS231n。因为我本来的打算就是在23年末到第二年3月份之前看能不能糊一篇小论文，如果不能至少也给我明年的毕业论文一点信心。毕竟放养型的导师，毕业还是全靠自己，更何况我对之后的工作还有期待。在拖拖拉拉看完CS231n之后，我也是火速看了好多篇使用深度学习进行星图识别的论文，在大致了解了课题背景和解决方案之后，我决定复现一篇论文，但卡在数据集获取这一步。因为星图识别不比其他深度学习的课题，没有开源的数据集，它的训练集都是通过软件或硬件（传感器）生成的。好在有师兄留下来的Matlab代码遗产，我打算之后好好研究尝试一下。

## Workout

1月初因为失恋了，所以决定用运动来避免胡思乱想，并且帮我脱离焦虑和烦躁的情绪。开始也只是跑步加上在出租房里做俯卧撑之类的自重训练。后来逐渐爱上了运动，坚持每天下班结束都沿着江边跑步，有时候甚至周末也会去跑步。后来和科创的朋友们联系多了，也捡起了打篮球的习惯。紧接着发现科创中心竟然有免费的健身房，于是便和hcy相约一起健身。尽管每次在健身房都是照着b站视频瞎练，但我们还是一步步摸索出了该怎么正确的做动作。

结束Arcsoft的实习回到玉泉之后，我就开始有规律的去健身房进行训练。虽然一开始也和所有人一样为了虚荣心，盲目上重量，卧推推半程深蹲只半蹲。好在是顺利的度过了前两个月的小白时期，大致学明白了三大项的动作。就这样有一搭没一搭的练，和hcy还有db一起，一直到了我找到第二份实习。期间也在通过b站或者小红书学习一些健身方面的知识，也算是马马虎虎形成了自己的知识体系。

![图6 训练记录](http://zhytou.github.io/post/2023-12-26/train-schedule.jpg)

于是在9月份，我决定正式开始走计划，并且使用APP记录自己的训练量化自己的进步幅度，从而让自己不会因看不见进步而放弃。从9月初到12月底，我每天6.下班，大概7.到玉泉。中途吃个沙县小吃或者兰州拉面，接着回宿舍换衣服略作休息就直接到健身房练到关门。三分化间隔日的时候，就安排吃顿好的，比如去二食堂二楼点菜或者到外面去吃饭。那段时间，我卧推和深蹲一直执行的是主项5×5的计划，背部因为俯身划船和硬拉都做的不太好，就以容量为主。整个计划执行完效果还是不错的，到12月中旬的时候，我的卧推从开始60kg勉强触胸做1个到现在随便做8个，最大重量也来到80kg；深蹲的话则是从最大重量90kg到现在100kg可以做5*5；背部则是可以几乎插满高位下拉68kg×8×3。12结束那边量化实习之后，我也把计划改为增肌计划，以容量为目标，希望明年3月份下一个增力期有惊喜吧。

## Coding

**课程**：

今年还是又上了一些不错的课程，包括：NJU-OS、CS144、CS231n、WorldQuant的Data Science Lab以及去年没上完的6.824。其中，NJU-OS、CS144和6.824都是6、7月比较闲的时候学的，而WorldQuant的Data Science Lab则是在白鹭资管实习上班抽空看的（主要是为了它可以放在[Linkin展示的证书](https://www.linkedin.com/in/yang-zhong-190072218/)），CS231n则是12月火急火燎看完的。总的来说，NJU-OS和CS144对我的提升最大，算是很好的帮我重构了操作系统和计算机网络的体系，理解了很多之前死记硬背的八股知识点，避免面试一问深入就单纯的像一张白纸的情况出现。

**语言**：

今年语言使用最多的还是C++，其次则是Python，然后也写了一点Golang和Javascript。其中，Python是最令我惊喜的，虽然一直被誉为最简单的语言，但我之前因为语法记不清，总是要Google查，现在写多了之后，尤其是量化实习写的比较多，现在也算是能写一点简单的东西了。希望之后写毕业论文的时候，这里学到的东西能够帮上我吧。

**面试**：

这里也顺道总结一下去年的面试吧：

| 公司名称 | 面试时间 | 感受 & 总结 |
| ------- | -------- | -----------|
| DolphinDB | 1.5 | 面试整体难度不太大，但当时我才开始准备找实习，复习不是很到位，然后手撕代码也写的磕磕碰碰（题目是滑动窗口众数），于是就顺利成章的被淘汰了。其中，影响比较深的问题是虚拟内存、TLB缓存、C++迭代器失效和TCP滑动窗口。总体来说，面试官给我的感觉是挺好的，加上98上胡神的一再推荐，这家公司应该还是不错的。 |
| Arcsoft | 1.12 | 面试问题包括多态、智能指针、C++内存管理、模板编程和锁。其他还问了一些过去项目和实习主要技术点的问题。当时因为正在做图形学作业感觉还算对口就去了。实习下来感觉也还不错，除了工资低一点其他都挺好的。 |
| 阿里云存储 | 2.11 | 感觉问了很对CSAPP中的内容，自己答的不是很好。后续也是果断重新翻了翻CSAPP这本书，重点看了其中编译链接的部分。 |
| 同花顺 | 6.19 | 一面基本上可以说是没有准备的，因为上周自己忙着写课程论文，个人的时间大部分都投入到了6.824课程要求的论文阅读之中去了。好在是难度不大，顺利过关。二面面试官是一位浙大前辈，主要是和我讨论自己未来的发展方向，甚至建议我读一下浪潮这本书。后续和小Leader聊了具体可能做什么，个人不是很感兴趣就放弃了这次机会。 |
| 飞步科技 | 7.4 | 一个控院老师开的公司，我面试只是打算保持一下状态顺便查漏补缺。整体难度不大，不过听到他们实习生也基本八九点下班，就赶紧拒绝逃了。 |
| 凌迪科技 Style3D | 7.12 | 一家开着紫金港附近的公司，是线下面试。感觉挺不错的，尤其是一进门的大显示屏，特别的fashion。面试也还算顺利，只不过对方要求至少半年实习，所以也无奈被拒。 |
| Momenta | 7.26 | 本来看名字以为这个公司会具有类似外企的文化，没想到面试官问完八股就是一通加班PUA，那我肯定逃啊。又是被名字和实习工资欺骗的一天。|
| 白鹭资管 | 8.5 | 终于来到我第二段实习的东家了。面试一共两轮，外加一轮笔试和一轮HR主管面试。感觉笔试和面试和之前的公司难度都不同，好在当时才学完NJU-OS和CS144不久，操作系统和计算机网络理解还比较深刻，顺利拿到量化实习Offer。 |

## TODO-List

先是去年目标的完成情况，大部分都做了，还是值得表扬的，嘻嘻。

**Coding**：

- [x] 参加两段实习
- [ ] ~~参加一次OSPP或GSoC~~（尝试了去申请ospp中cubefs的一个简单任务，但可能是沟通的太晚了就没通过，后续也没有继续关注这方面的消息了。）
- [x] 继续刷题和面试保持状态
- [x] 完善之前的一些Toy Project，包括：
  - [x] MultiThreadPracticeInCPP
  - [x] SimpleSTL
  - [x] SimpleRenderEngine
- [x] ~~给开源社区提一次PR~~（给cubefs的文档提了两个issue，然后尝试给microsoft的一个文档添加一个中文翻译的pr，不过没有合并。虽然都是比较简单的人物，但对我来说还是一次比较好的尝试。）
- [ ] 找到自己的方向，可以通过读paper和做项目来实现。潜在的几个方向包括：
  - [ ] 数据库或基础架构方向：1、做完6.824；2、读DDIA；3、参加TinyKV训练营；4、寻找一份数据库方向实习。
  - [ ] 图形学或GPU加速：1、完善SimpleRenderEngine；2、阅读相关文献（待了解）；3、学习CUDA和shading language，了解openCL和openGL协作；4、参与Taichi开源社区；4、寻找一份图形学方向实习。
  - [x] QT开发：1、重新捡起QT，完成相关笔记；2、相关实习。
  - [ ] 云原生容器及Web3方向：1、做完6.824；2、读DDIA；3、补足go基础知识（go秒杀）；4、相关实习。

**With my girl**：

- [ ] ~~去影楼拍照片，算是某种莫名的理想？~~
- [ ] ~~去听gali或者其他某个我们都喜欢的饶舌歌手的Live~~
- [x] ~~两个3-5天的短期旅行（复刻温州之行）~~(没和女朋友，但是和朋友去了两趟，也不错)

**English**：

- [ ] ~~继续和Episoden上的朋友聊天~~（有聊，但是不多）
- [ ] 参加一次英文的mock面试
- [ ] 准备Gre

**Health**：

- [x] 体重降回到大三时期
- [x] 继续篮球

接着是明年的计划，分成学习（毕业问题）、代码（找工作）、娱乐和健身四大块。

**Graduation**：

首先是毕业问题，大约三月底就要完成开题答辩，所以我至少需要在年前确定课题然后做出点东西来，最好能写一份综述和开题报告，避免到时候因为和春招撞上而忙的焦头烂额。根据我目前收集到的信息，我能写毕设和小论文的时间就两部分，一块是春招找实习之前，第二是秋招结束确定工作之后，所以尽量一月份要做点东西出来。

- [ ] 希望能顺利投一篇小论文。目前的目标是星图识别方向，找一个EI会议投了，达成申请学位的要求。
- [ ] 希望做毕设能顺顺利利。不求做多好，只求达成毕业要求。

**Coding**：

其次是找工作，毕竟找份好工作是我读研的原动力。24年又是我25届毕业生暑期实习和秋招的重要时期，所以务必要好好准备。

- [ ] 一二月份以做课题为主，但也要刷刷Leetcode，然后关注实习信息，尤其是汇丰、微软、亚马逊、AMD、Apple、英伟达这几家公司的招聘信息，及时投递不要错过了。
- [ ] 尽管实习很重要，特别是那种主要靠实习转正的公司，但也要谨慎选择。尤其要注意和HR确认是否有转正headcount。此外，如果没有特别好的机会，宁愿不去实习，早点开始秋招。
- [ ] 第三，尽管我明白实习和找工作特别重要，但也要以毕业任务为主。特别是老师如果要安排其他横向给你做，也要尽量争取以毕业课题为主。毕竟还有25年春招一次机会。

**Workout**：

接下来是健身，23年健身算是小有成果，三大项卧推从开始60kg勉强触胸做1个到现在随便做8个，最大重量也来到80kg；深蹲的话则是从最大重量90kg到现在100kg可以做5*5；背部则是可以几乎插满高位下拉68kg×8×3。

- [ ] 首先是指标性的目标。希望明年结束卧推最大重量能够到100kg，或者100kg能够做组；深蹲则是120kg做组；至于背部的话，希望能够标准引体能够完成3*8，因为现在对握引体已经可以做组了。
- [ ] 其次是增肌、增力和减脂周期的规划。承接上一年的增肌周期，持续到24年3月初；接着转入增力周期，完成2到3个月的增力计划；然后开始控制饮食并加入有氧，进入3个月左右的减脂期，为夏天做准备，也是检验我前一年训练成果的好时机；最后再次进入增肌期。

**Entertainment**：

最后是娱乐，说到底人活着还是为了开心。无聊而只会倒苦水的人甚至交不到朋友，所以让自己的生活变得更精彩和丰富的娱乐活动也很重要。

- [ ] 第一件事是去一次livehouse或者音乐节，这个想了很久了，明年必拿下。
- [ ] 第二件事是多逛一逛杭州，比如：去坐一下水上公交，去一趟宋城、湘湖、西溪湿地之类的景点，吃一下国宾里的正宗杭帮菜或者巷子里的爆鳝面片儿川，毕竟在杭州的日子也快占我人生的五分之一了。
- [ ] 第三件事是希望能够和23年一样去两次短途旅游。毕竟自己最喜欢干的的事就是羡慕别人在朋友圈里晒出旅游的精彩瞬间。（但自己真正有机会有时间去行动的时候又会打退堂鼓。）
- [ ] 第四件事是希望自己能在减脂结束的时候，换个发型或者发色。毕竟也快要进入工作了，再不年轻一把似乎都没机会了。
- [ ] 最后一件事就是多记录自己的生活，拍拍照。不一定要搞定特别酷，特别装逼然后发到社交平台上，只是给自己留个记忆通道的入口。

## Outro

2023对我来说总体还是挺开心而充实的一年吧！虽然经历了分手，但我也因此成长了不少。其中最重要的感悟就是，不要剖析自己，不要和总是谈论烦恼和选择，即使是面对最亲密的恋人或者父母也是一样。因为它除了给真正关心的人徒增烦恼，就是暴露我的软弱让假装关心我的人远离自己。在外人面前，只需要当个存粹的赌徒，上座下注，愿赌服输。

![图7 摘录](http://zhytou.github.io/post/2023-12-26/quote.jpg)

前几天朋友给我分享了图7里面的这段话，真的挺有感触的。太多的人把生活当作一个超大的TODO-List来一步步完成，在人生的路程上飞奔。因为所有人都享受给TODO-List某一项画上勾的爽感，但这种频繁的成就感只会让多巴胺的阈值不断升高，让开心越来越难。所以啊不被欲望操纵，知足常乐才是正道。另外，很多普通人每天被社交媒体上的“精彩人生”所拷打之后，审视自己一潭死水的生活往往会陷入深深的失落。但必须要明白的一点是，那些所谓的网红、博主、Vlogger都只是把他们生活中最好的一面展示给观众，进而让白日梦想家们沉沦。所以啊，我们要接纳自己在生活上的平庸，和自己和解，但这并不是不再热爱生活、懒惰消极的对待一切，而是不要被社媒上勇敢自由的定义裹挟。把眼光放开一些，热爱生活从来不只有这一个固定模板。或许我们不能成为社交媒体上精通滑雪跳伞、全世界飞的Cool Boy/Girl，但我们可以成为把家安排的井井有条、日子过得有滋有味的生活管家。希望在未来的日子里，我看到朋友圈里晒出的精彩生活，只浅浅感叹一句真好啊就继续专注在自己的生活对抗。因为英雄主义的最好写照就是在认清生活的真相之后，依然热爱生活。