# 自动化生成代码的实践和探索

由上一篇文章我们介绍了[自动化生成代码几种方案的演变](https://zhuanlan.zhihu.com/p/419795613)，今天跟大家分享一下我们对自动化生成代码的实践和探索。

本文主要是对之前开发的一个总结，当初由于某些原因没有进行分享和总结，当时我们的开发模式和搭建器还没有那么地完善。

本文主要解决的是开发效率低的问题，通过一键生成源码的方式，减少开发成本和提高效率，效果如下

![图片](https://cdn.poizon.com/node-common/777a27cad438288e1757c08519342135.gif)
<!-- <iframe src="https://cdn.poizon.com/node-common/777a27cad438288e1757c08519342135.gif" allowfullscreen></iframe> -->
<div align='center'>输入一行命令直接生成源码</div>

## 现状
目前我们中后台的很多页面都很相似，这些页面大多为列表页，编辑页和详情页，也就是“四表一单”中的几个，这些页面写起来也是挺繁琐重复的事情。

目前我们的开发模式大多数是这样，若要开发一个新的模块，先拷贝之前写好的相似模块页面到新的模块中，然后再此基础上进行逻辑的删改处理，然后与根据后端提供的API接口，到mock平台中修改其中的字段，自测通过后与后端进行联调，联调完成并执行测试用例并提测，到此开发阶段结束。后续再进行跟测。

目前采用这种开发模式存在以下几个痛点：
![图片](https://cdn.poizon.com/node-common/810b4013f8cb3503d7b6dc39384f2f42.png)
## 分析
针对这些痛点通常我们第一时间想到的是可视化搭建，关于可视化搭建，是指通过可视化的环境平台，以更快捷的配置方式实现应用程序的生成。它有两个主要特征：一是可视化，二是可配置。

按目标代码划分，可分为以下两类

|     |  低代码  |  无代码  | 
|  ----  | ----   | ----  |
| 平台  | 可视化   | 可视化 |
| 编码  | 少编码 | 无编码 |
| 面向群体  | <div style="width: 150pt">开发</div> | <div style="width: 150pt">设计/运营/产品</div> |

按技术实现，可分为以下两类

|     |  运行时模式  |  非运行时模式  | 
|  ----  | ----   | ----  |
| 出码能力  | 不输出源码 | 输出源码 |
| 编排时机  | 运行时编排 | 编译时编排 |
| 运行效率  | 低（项目越复杂越明显） | 较高 |
| 灵活性  | 低 | 较高 |
| 维护成本  | 较低 | 较高 |
| 风险性  | 较高且影响广 | 较低 |
| 面向群体  | <div style="width: 150pt">运营/产品</div> | <div style="width: 150pt">一般是开发</div> |

> 思考：
> 非运行时模式，输出的是源码，在什么样的搭建器中不完全不需要开发介入？

首先我们内部的中后台搭建器属于运行时模式，意味着需要使用整套服务，对独立的新系统友好，但也存在一定的局限性：
- 扩展性较弱，业务需求要与搭建器开发者跟进
- 抗风险弱，哪天需求变了，搭建器无法搭建时，二次的迁移成本和风险随之而来
- 兼容性弱，无法很好地与老系统共存，包括UI风格
- 风险高，搭建器的修改，直接影响所有系统，稳定性的风险提高

当然这里不是否定搭建器的作用，相反我是搭建器的开发者和支持者，任何一种设计都有其适用场景和缺陷，它与现成的搭建器不是对立的关系，而是一种补充。针对这种场景，我认为输出源码的搭建器才更适合传统的开发模式，原因如下：
- 拥有代码掌控权，意味着你拥有绝对控制权，无信任危机
- 稳定性得到保证，业务与搭建器完全隔离，即使搭建器出bug也不会影响正常业务系统的运行
- 消除潜在的未知风险，业务需求怎么变都可以灵活修改，因为源码在你手上，不存在二次迁移成本和风险
- 扩展性和兼容性都很强

当然，非运行时搭建器，即输出源码的形式也有其短板，后面我会讲到，这里的提到的优点，只是针对快速与开发模式结合的场景。

> 非运行时模式，不一定是一个完整的UI搭建器，它也可以是一个接口服务，只要最终输出源码即可

针对目前存在的痛点，我们的解决方案如下：
![图片](https://cdn.poizon.com/node-common/0a3baab31a1b0292b0f637bd2920558d.png)

## 实现
#### 架构
![图片](https://cdn.poizon.com/node-common/bbcb9a7e46abf5e85d0498c6334de054.png)
前两步已实现，要实现自动生成源码，其实主要就是将第一步【用户搭建】改成了【AI搭建】，省去人工手动搭建的过程，架构如下：
![图片](https://cdn.poizon.com/node-common/c9606259017ae89847efab0f576d8b3e.png)
它由两部分构成，智能识别器和代码生成器：
**智能识别器**：主要的功能是识别并生成数据，读取用户输入和爬mock接口数据，生成代码生成器中要使用的数据。
**代码生成器**：主要功能是输出源码，这里是一个搭建器，它具有非常强的灵活性和可控性，结合传统的开发模式，直接输出Vue2/Vue3/React源码。

它的核心是**数据驱动生成代码，也就是以接口数据为核心，自动生成对应页面的源码**。
#### 流程
![图片](https://cdn.poizon.com/node-common/68f8a07c76c837c91cecabf9df6a6858.png)

通过读取用户输入的API接口，使用爬虫登陆Mock平台，抓取到对应的API接口数据，解析数据，根据接口数据判断要生成的模版页面，同时转换成平台能识别到的数据，这个数据转换器处理的好坏直接影响最终生成结果，所以它也是自动化脚本的核心。转换器处理完数据后，关闭mock窗口。

接着，自动登陆搭建器平台，从而模拟我们真实环境生成的页面，从而省去我们手动配置这一步。将前面数据转换器处理的结果，选择对应的模版，自动填充对应的数据，最终生成源码。

#### 智能识别器
智能识别器它主要有三大核心流程，分别是解析、转换和识别：
**解析**：在mock平台接口数据，不同的接口可能会有不同的数据结构，提取过程需要对请求头和返回body分别进行判断，提取数据，它们跟普通请求返回的固定结构略有不同，并不是读取固定的字段就能读到数据。
**识别**：识别数据，首先要识别模版类型，不同的模版意味着不同的上下文，即使对同一个字段，在不同的上下文中含义是不一样的。比如一个状态字段，在表单模版中是一个选择框，然而它在列表中就是一个枚举。
**转换**：准确转化数据是自动化生成代码的关键，在不同的上下文中需要转换不同的数据。

**1）识别**
![图片](https://cdn.poizon.com/node-common/de3b131b937ce267c1dbfde61682bf04.png)

以识别模版类型为例，我们这里采用了三层逻辑判断，首先判断用户的指令中属否指定了类型，若用户指定了模版类型直接采用，若用户没有制定；接着根据接口名称判断页面类型，是否包含特殊关键字，若接口名称不能判断，最后再根据返回的结构进一步判断，最终若都不能识别模版类型，直接退出程序，不再往下走。

**2）转换**
转换器其实是跟代码生成器有很强的耦合性，转换器转化的准确率其实跟你的代码生成器要接收什么样的数据存在很大的关系。

转换器需要在不同的上下文中处理不同的转换逻辑，以列表页面为例，需要将对应的数据拆分两块，搜索栏和表格数据，首先需要对先对分页字段进行提取，然后需要将不同的字段与对应的组件进行绑定，如选择器（状态/类型），日期范围（时间/日期），数字输入框（金额/次数/数量）等；这里需要对一些字段做特殊处理，如日期范围类型组件的字段，需要由两个字段合并成一个字段，提交时再拆分成两个字段。

同理，对于表格数据，转换器需要将字段与处理器进行绑定，同时根据字段名称和类型实时模拟生成mock数据。

转换器需要在实践过程中不断进行调整，才能更好的提高它的准确率。
#### 搭建器平台
![图片](https://cdn.poizon.com/node-common/e04f11138ddadc1ba8c07dca09d423ae.png)
搭建器([idou](https://github.com/ctq123/idou))主要功能就是输出源码，它以一份DSL作为桥梁，在UI组件和源码之间承担承上启下的角色，用户在搭建器编辑UI，其实是在编辑DSL，在点击生成源码的时候，再根据DSL和特定的转化规则生成源码。

![图片](https://cdn.poizon.com/node-common/9cb4a6ef07801dd83109cdb3125e5cb2.png)
涉及到的功能模块有很多，这里就不一一展开介绍，这里简单介绍下DSL转化源码的大体思路。

**DSL转化源码**
![图片](https://cdn.poizon.com/node-common/26cd5af7f9df4a7c13e41178217f2aa5.png)

首先生成源码的过程，不同DSL/schema/AST结构，它们跟DSL的结构耦合性很强，定义不同的结构基本上方法是不能共用的，但有一个很重要的点就是它们大体思路是一样的，**都是采用分治的思想，即先将其拆分成不同模块，分别处理，最后再合并在一起**。

以vue代码为例，手把手教你转换源码
![图片](https://cdn.poizon.com/node-common/ba1d666f603d0a050516459c739191b1.png)
我们首先需要对源码进行分析，将其按模块进行分块，它由三大模块构成，如上，分别为template模块、script模块、style模块，script模块再进一步拆分，data数据块、lifecycle生命周期块、methods方法块。

模块拆解对应的DSL分别如下：
```javascript
// template模块
<template>
  <div class="page">
    hello world
  </div>
</template>
```
```javascript
// template模块DSL
const TemplateDSL = {
  componentName: 'div',
  props: {
    className: 'page',
  },
  children: 'hello world',
}
```
```javascript
// script模块
<script>
// import块
import * as API from './api';

export default {
  // data块
  data() {
    return {
      form: {
        trueName: '',
      },
    };
  },
  // lifecycle块
  mounted() {
    this.queryList();
  },
  // methods块
  methods: {
    queryList() {},
  },
};
</script>
```
```javascript
// script模块DSL
const ScriptDSL = {
  dataSource: {
    form: {
      trueName: '',
    },
  },
  lifeCycle: {
    mounted: `function mounted() {
      this.queryList();
    }`,
  },
  methods: {
    queryList: `function queryList() {}`,
  },
  imports: {
    '* as API': './api',
  },
}
```
```javascript
// style模块
<style lang="scss" scoped>
.page {
  padding: 0px;
}
</style>
```
```javascript
// style模块DSL
const StyleDSL = {
  styles: [
    `.page {
      padding: 0px;
    }`
  ]
}
```
然后收集对应的DSL数据，结构如下：
```javascript
const renderData = {
  template: '',
  imports: [],
  data: {},
  methods: [],
  lifeCycles: [],
  styles: [],
}
```
最后根据renderData对象重新组合成vue文件，如下：
```javascript
// 组装template最核心的代码，价值一个亿，拿去用不谢^_^
const CreateDom = (name, attrStr, childStr) => 
`<${name} ${attrStr}>
  ${childStr}
  </${name}>`;
```
```javascript
// 重新组装vue代码
const VueTemplate = (renderData) => `
  <template>
    ${renderData.template}
  </template>

  <script>
    ${renderData.imports.join(';\n')};

    export default {
      data() {
        return ${JSON.stringify(renderData.data, null, 2)}
      },${
      renderData.lifeCycles.length
      ? renderData.lifeCycles.join(',\n') + ','
      : ''
      }
      methods: {
        ${renderData.methods.join(',\n')}
      }
    }
  </script>

  <style lang="scss" scoped>
    ${renderData.styles.join('\n')}
  </style>
`
```

当然这只是最简单的一个例子，这里只是为了向大家阐述DSL生成源码的大体思路，实际例子会这更复杂，比如跨组件绑定数据、rules动态生成、事件的绑定、UI转换等。

> 思考: 同样的这份DSL如何生成React源码？
[前往idou查看解决方案](https://github1s.com/ctq123/idou)

## 总结
本文介绍了我们对自动化生成代码的探索与实践，主要包含两部分智能识别器和搭建器，利用自动化脚本抓取数据，经过智能转换处理数据后，再打开搭建器，自动输入配置数据，最终生成源码。

实现了保持足够的简单，一行命令后便输出可运行且可维护的源码，避免重复工作，提高研发效率，并且保持足够强的灵活性和扩展性，快速与传统开发模式结合。

## 未来展望

![图片](https://cdn.poizon.com/node-common/797e8277e63894000d8cec1bc05ddb87.png)

**相关链接：** 
项目地址：https://github.com/ctq123/idou
预览地址：https://idou100.netlify.app
自动上传源码：https://github.com/ctq123/dslService
