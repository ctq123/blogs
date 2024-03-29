# 自动化生成代码几种方案的演变

今天我们聊一聊自动化生成代码的问题，试想一下，假如有一天机器替代你编写代码，你是应该感到开心还是难过？

## 方案
目前代码生成技术主要有以下几类：
![图片](https://cdn.poizon.com/node-common/13d6ed10dacd8d5c16c854ddca183e57.png)

### 1）基于模版编排生成代码
首先说下基于模版生成代码的方式，这种属于最原始最简单也是目前应用最广泛的一种代码生成方式，可以说，几乎所有的代码生成方式都是建立在模版的基础上繁衍而来。

前端最有名的莫过于vue-cli和create-react-app两款脚手架的代码生成了，他们分别基于vue和react框架进行的一键初始化项目生成代码，包括各公司内部的项目生成的脚手架，其实本质上是一样的，只是加入了公司内部集成的很多公共的组件和方法而已。

其实最早的代码生成并不是前端，因为那个时候还没有前端这个概念，在WWW万维网的时代，页面基本是由后端进行开发，因此在模版代码生成这一领域我们只是走了后端走过的路。

记得最早接触模版代码生成的是在刚开始开发后端项目的时候，那时候还是采用MVC架构，要开发一个新的功能模块的时候，往往需要分别对controller,service,dbService中进行新的代码注入，有点类似于egg中的结构（如果不熟悉后端开发的话），因此这种机械化比较固定的结构化组织中，往往通过命令一键生成即可，然后再对service/dbService中写入独特的逻辑即可。

这种开发模式能节省很多开发时间，且上手容易，开发者往往只需要关注逻辑开发即可，但往往会出现很多重复性的代码，这也是自动化代码生成一直存在的弊端。

### 2）基于可视化UI生成代码
基于可视化搭建的代码生成，这也是目前市场上最多的一种产物，也是很多可视化搭建文章强行给它带上的一种帽子，但准确的说它并不完全属于自动化生产代码，只是其中的一个环节而已，由于它需要过多的加入人为干预，最终才能得到我们想要的运行程序，感觉它一点都不自动化，如何对得起“自动化”三字？

但可视化搭建的确能带来一些节约生产力的功能，同时也给其他角色赋能，搭建应用的不一定是开发了，他们甚至可以不需要懂任何程序，可以是设计、产品和运营。

其中最为成熟的莫过于汽车行业生成代码。对，你没看错，就是汽车行业，在我们的印象中，汽车行业属于比较传统的行业，似乎与新兴产业互联网的代码编程八竿子打不着。然而，在2010年之前，大多数的汽车控制软件，如发动机控件、变速箱软件、车身控制软件等都是通过手写C代码完成的。开发流程如下：

![图片](https://cdn.poizon.com/node-common/2a4574b83f1735d78fd44cb863f6de45.jpeg)

其中设计师是不懂代码的，从设计图交付开始到最终控制器的完成，这一过程会出现两个问题，设计师与码农之间的沟通成本以及码农的交付质量，项目越大这两大问题就越突出，直接影响交付日期、成本和质量，通常情况下交付时间与成本成反比，与质量成正比，而时间就是金钱的今天，要同时保证成本和质量，更好的办法就是设法将将代码标准化，同时降低沟通成本。

因此，可视化UI生成代码似乎是最合适的途径。它从设计师的角度出发，将视图UI与命令行一一绑定，设计师拖动一个视图即生成一个对应的C代码。最终，码农下岗了。

![图片](https://cdn.poizon.com/node-common/b49efb7ae4cb9e2018dd4ef4a57d8e2b.jpeg)

上面这种成熟的Simulink软件处理的可视化逻辑的控制，只能处理简单的逻辑，与现代的逻辑编排的设计理念相同。当然它也有自己的缺点就是不适合复杂的项目工程，在这种场景，它反而比直接写代码的效率更低。

另一个著名的可视化UI生成代码例子，便是.NET时代的eclipse的web搭建，使用过eclipse编辑器开发web界面都应该知道这个工具，如图
![图片](https://cdn.poizon.com/node-common/4b5a9443e43f7e4bcc8e17b357dcdb04.jpeg)

只是它在开发者的用户体验上比不上网页三剑客，最终被淘汰了，毕竟它只是主打JAVA开发的IDE，开发Web可视化也只是其中一个扩展而已，这里也告诉我们一个道理，一个产品的精而美，比大而强反而更容易获得成功。专注而顶尖，是很多产品的成功奥秘。

当然目前可视化搭建的系统，是在太多了，目前主要分为两种方式，运行时搭建和生成源码搭建，这里就不一一展开介绍了。

### 3）基于代码语料生成代码

基于代码语料生产代码的前提是要有足够的语料，也就是代码片段。这种方式，通常都是基于IDE的插件而开发的应用。因为它最终的目的是为了开发提效，针对的用户群体是开发者，必须有开发介入生成一个半成品的代码片段或模块。

假如你有耐心阅读到此，说明你是对代码生成领域感兴趣的同行，为了表示感谢，接下来会安利一波福利。这里的福利也只是针对前端开发，主要是针对vscode插件展开介绍。

这里其实需要分为简单和复杂两种场景：
- 固定语料
- 智能化语料

#### 固定语料
用户提前设置的代码片段，通过监听用户输入快捷键值，搜索出对应的片段，提示用户。
针对vue的提示插件Vue 3 Snippets
针对react的提示插件ES7 React/Redux/GraphQL/React-Native snippets
上面两款插件的下载量都是百万级别以上的，所以流行程度是绝对杠杠的，当然如果你觉得它们不好用，或者不够贴切公司内部的代码片段，也可以自己定制。

这种生成代码方式，简单而快速，但也存在其弊端，固定化不好扩展，而且最令人不快的是使用者需要一定的学习成本，需要提前了解键值以及对应的代码片段。

为了解决这个弊端，于是聪明的程序员们便加入智能化，利用训练学习法找出对应的代码片段，于是有了接下来的智能化语料。
#### 智能化语料

**1）Kite Autocomplete插件**
首先介绍第一个智能化代码提示vscode插件Kite Autocomplete，在超过2500万个文件上训练的ML模型，Kite 为您的代码编辑器添加了 AI 驱动的代码完成功能，赋予开发人员超能力。
![图片](https://cdn.poizon.com/node-common/6dfd7ee1e2f5a00169438cf3d0ec16ac.png)

它会根据用户输入的上下文以及当前输入，预判用户将要输入的内容。

在安全方面，kite在本地运行，您的代码是私有的，不会离开您的机器。

这种智能化输入是依赖n-gram模型（不了解n-gram模型的童鞋可以自行搜索了解一下），只不过它比n-gram模型做得更前一步，会预先读取整个文件的上下文，结合当前的输入再推断出用户的行为。

总体上，Kite Autocomplete插件还是蛮好用的，它支持多种语言，**使用vscode编辑器的同学强烈建议使用，在这里给你们种草了，超好用，谁用谁知道**。

**2）GitHub Copilot插件**
接下来介绍另一个智能化代码提示vscode插件GitHub Copilot，它是OpenAI与微软联手推出的一款AI代码生成代码工具，可以说它是vscode官方亲儿子也不为过，不过要使用它需要注册。让我们看看你它的官方介绍
> GitHub Copilot 由 OpenAI 创建的新 AI 系统 Codex 提供支持。GitHub Copilot 比大多数代码助手理解的上下文要多得多。因此，无论是在文档字符串、注释、函数名称还是代码本身中，GitHub Copilot 都会使用您提供的上下文并合成代码以进行匹配。我们正在与 OpenAI 一起设计 GitHub Copilot，以便在开发人员使用它时更智能地生成安全有效的代码


![图片](https://cdn.poizon.com/node-common/2334679b5ce079e7a70730f2353cab56.png)

它最大的核心竞争力便是可以根据注释自动生成代码，也就是说你告诉编辑器功能描述，它便可以自动帮你生成你想要的代码了，听起来是不是很酷？它实现原理如下：

![图片](https://cdn.poizon.com/node-common/d407666677431c5f193c24ea58b69ed3.jpeg)

当然，它也可以自动填充重复代码，GitHub Copilot 非常适合快速生成样板和重复代码模式，这个功能完全就是将上述介绍的固定语料包含了进去，可以说对需要写大量模版代码的程序员是非常香的一个操作。
![图片](https://cdn.poizon.com/node-common/f91e6594bbbf6765688325c29e1dd07c.png)

另一个比较重要的功能便是测试代码，官方表示无需辛苦的测试。 测试是任何强大的软件工程项目的支柱。导入单元测试包，让 GitHub Copilot 建议与您的实现代码匹配的测试。
![图片](https://cdn.poizon.com/node-common/32254c5b6769a1e8c188e9c12d021ef9.png)

据官网介绍，它是一款基于训练集却可以生成从来没出现过的新代码的工具，听起来是不是很牛？它居然可以“创建”代码了。

最后也是最重要的一点，GitHub Copilot插件暂时只提供有限用户的试用体验，目前只有预览版本，总之一句话，**暂时用不了**。

### 4）基于智能化生成代码

**1）sketch2code**
介绍完GitHub Copilot后，下面让我们了解微软的另一款智能化生成工具[sketch2code](https://sketch2code.azurewebsites.net/)

![图片](https://cdn.poizon.com/node-common/fc064e49935962ac2b4e37ebee6fe7c9.png)

它的主要功能是将手绘图转化HTML代码。

它实现的过程如下：
1.首先，用户通过网站上传图片。
2.自定义视觉模型预测图像中存在哪些 HTML 元素及其位置。
3.手写文本识别服务读取预测元素内的文本。
4.布局算法使用来自预测元素的所有边界框的空间信息来生成容纳所有元素的网格结构。
5.HTML 生成引擎使用所有这些信息来生成反映结果的 HTML 标记代码。

于是我赶紧试了一波，效果如下：
![图片](https://cdn.poizon.com/node-common/34541cc54a57cc4394fdbc4de44aafb7.png)
左中右分别为草稿原图，识别分析图，结果html
由于是横屏的角度，嗯嗯，我原谅你，再来个正常的吧
![图片](https://cdn.poizon.com/node-common/86067f9995b213ef3c446a687583af56.png)
识别度还是不太理想呀。

同时它还支持摄像头识别，也就是说只要加上硬件设备，产品就可以一边在上面bibi……哦不对，是一边规划，一边生成代码，只不过是原生的html代码，我又赶紧试了一波，结果:
![图片](https://cdn.poizon.com/node-common/5358350a2139f046ca4ae12f585a2c28.png)
上面是摄像头拍到的快照，上传之后……
![图片](https://cdn.poizon.com/node-common/1e12c858348b413f6ba6656b651402da.png)
啥？识别不出，好吧。估计是我画得比较潦草，而且可能图片背景不是全白的，于是乎我用了它官方提供的示例图
![图片](https://cdn.poizon.com/node-common/f4237a4b21cdd9d622e93ba0880c4188.png)
效果好一点，但还是不能完全识别出来。

所以它目前还是有些缺陷的
- 生成结果只能是原生HTML
- 识别率较低，草稿图还原度有待提升


**2）teleporthq**
另一款AI智能化代码生成工具是[teleporthq](https://teleporthq.io/)，它与上面的sketch2code原理很相似，只是它更进一步，如图
![图片](https://cdn.poizon.com/node-common/68a658e787d11d31516b92c76f534cdf.png)

产品经理一边在小黑板上画图，后面的机器一边扫描，识别并生成代码，同时实时给出预览效果。**真正做到了一边画草稿一边生成代码**，实现自动化生成代码功能，原文请前往(https://teleporthq.io/blog/we-believe-in-ai-powered-code-generation)。

teleporthq它的官网似乎与普通的可视化搭建器并没有什么不同，然而，如果仔细观察，你会发现它其实比普通的可视化搭建器更多一些东西，比如支持草图到代码，视觉API，而这些东西正是结合硬件设备实现代码实时输出必不可少的东西。当然它识别草稿图的辨识度就ememem~，除了草稿图，它也是支持识别Sketch素材的。

目前它支持生成的目标代码有React/Vue/Angular等，更多可前往其官网https://teleporthq.io/

teleporthq与国内阿里的imgcook原理上很相似，只是imgcook识别的是各种设计师的素材(Sketch/PS/Figma)并将其生成多种目标代码，imgcook就不一一展开介绍，读者可自行鉴赏。
## 结论

今天我们从互联网的发展历史分别介绍了自动生成代码的几种方式：
- 基于模版编排生成代码
- 基于可视化UI生成代码
- 基于代码语料生成代码
- 基于智能化生成代码

无论如何发展，其本质其实从来没有变过，那就都是基于模版本身，变的只是转换规则。

下一篇文章将介绍我们对自动化生成代码的实践和探索（https://github.com/ctq123/idou）
