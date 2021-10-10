# 自动化生成代码的实践和探索

由上一篇文章我们介绍了几种代码自动化生成的方式，今天跟大家分享一下我们对自动化生成代码的实践和探索。

本文主要是对之前开发的一个总结，当初由于某些原因没有进行分享和总结，当时我们的开发模式和搭建器还没有那么地完善。

本文主要解决的是开发效率低的问题，通过一键生成可调试源码的方式减少开发成本和提高效率，效果如下

## 现状
目前我们中后台的很多页面都很相似，这些页面大多为列表页，编辑页和详情页，也就是“四表一单”中的几个，这些页面写起来也是挺繁琐重复的事情。

目前我们的开发模式大多数是这样，若要开发一个新的模块，先拷贝之前写好的相似模块页面到新的模块中，然后再此基础上进行逻辑的删改处理，然后与根据后端提供的API接口，到mock平台中修改其中的字段，自测通过后与后端进行联调，联调完成并执行测试用例并提测，到此开发阶段结束。后续再进行跟测。

目前采用这种开发模式会出现以下几个问题：
- 页面重复率高，开发繁琐
- 页面的字段与API接口非常密切
- 修改字段多且繁琐，容易出错
- 团队各成员的代码风格不统一
- 开发效率相对较低
## 分析
关于可视化编程，是指通过可视化的环境平台，以更快捷的配置方式实现应用程序的生成。它有两个主要特征：一是可视化，二是可配置。

按目标代码划分，可分为以下两类

|     |  低代码  |  无代码  | 
|  ----  | ----   | ----  |
| 平台  | 可视化 | 可视化 |
| 编码  | 少编码 | 无编码 |
| 面向群体  | 开发 | 运营 |

按技术实现，可分为以下两类

|     |  运行时模式  |  非运行时模式  | 
|  ----  | ----   | ----  |
| 出码能力  | 不输出源码 | 输出源码 |
| 编排时机  | 运行时编排 | 编译时编排 |
| 运行效率  | 低（项目越复杂越明显） | 较高 |
| 灵活性  | 低 | 较高 |
| 维护成本  | 较低 | 较高 |
| 风险性  | 较高且影响广 | 较低 |
| 面向群体  | 运营/产品 | 一般是开发 |

> 思考：非运行时模式，输出的是源码，在什么样的搭建器中不完全不需要开发介入？

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
## 目标
针对上述现状，我们需要想要实现的目标：
1）快速与传统开发模式结合
2）保持足够强的灵活性
3）避免重复工作，读取接口字段自动填充字段
4）保持风格统一，降低维护成本
5）消除未知的潜在迁移成本和风险
6）输出可运行且可维护的源码
7）保持足够的简单，最好一键生成

## 实现
#### 架构
要实现自动化上述目标，基本就是自动化生成代码的探索，如图
![图片](https://cdn.poizon.com/node-common/2c3e7ed2e492bd8dc9120e37c10d0ec6.png)
它由两部分构成，数据转换器和搭建器：
**数据转换器**：主要的功能是读取用户输入和爬mock接口数据，识别生成的页面类型和数据。
**搭建器**：根据配置数据，输出源码，我这里做成了一个搭建器，具有非常强的灵活性和可控性，与现有的传统开发模式不冲突。如果你们公司没有还不具备搭建器，采用的又是传统的开发模式，这个工具将非常适合你，目前支持生成Vue2/Vue3/React。

它的核心是数据驱动生成代码，也就是以接口数据为核心，通过prettier自动生成我们对应的页面
#### 流程
![图片](https://cdn.poizon.com/node-common/68f8a07c76c837c91cecabf9df6a6858.png)

通过读取用户输入的API接口，使用爬虫登陆Mock平台，抓取到对应的API接口数据，解析数据，根据接口数据判断要生成的模版页面，同时转换成平台能识别到的数据，这个数据转换器处理的好坏直接影响最终生成结果，所以它也是自动化脚本的核心。转换器处理完数据后，关闭mock窗口。

接着，自动登陆搭建器平台，从而模拟我们真实环境生成的页面，从而省去我们手动配置这一步。将前面数据转换器处理的结果，选择对应的模版，自动填充对应的数据，最终生成源码。

#### 数据转换器
要获取到想要的数据，有两个主要的流程，分别是提取和转化：
**提取**：在mock平台接口数据，不同的接口可能会有不同的数据结构，提取过程需要对请求头和返回body分别进行判断，提取数据
**转化**：作为自动化脚本的核心，数据转换器有两大核心功能，一是识别模版类型，二是准确转化数据。

**1）识别模版类型**
![图片](https://cdn.poizon.com/node-common/de3b131b937ce267c1dbfde61682bf04.png)

要准确识别生成的模版类型，我们这里采用了三层逻辑判断，首先判断用户的指令中属否指定了类型，若用户指定了模版类型直接采用，若用户没有制定；接着根据接口名称判断页面类型，是否包含特殊关键字，若接口名称不能判断，最后再根据返回的结构进一步判断，最终若都不能识别模版类型，直接退出程序，不再往下走。

**2）准确转化数据**
数据转换器其实是跟代码生成器有很强的耦合性，转换器转化的准确率其实跟你的代码生成器要接收什么样的数据存在很大的关系。也就是说你需要很了解你的代码生成器，这对多人合作的团队是一个挑战，这需要你们密切合作，尤其在研发初期阶段。

我们这里代码生成器也就是搭建器接受的数据类似配置化，对每种模版类型的数据是不一样的，所以需要根据模版类型分别进行判断。

以列表页面为例，需要将对应的数据拆分两块，搜索栏和表格数据，首先需要对先对分页字段进行提取，然后需要将不同的字段与对应的组件进行绑定，如选择器（状态/类型），日期范围（时间/日期），数字输入框（金额/次数/数量）等；

```javascript
Object.entries(request).forEach(([k, v]) => {
  if (['page', 'pageNum'].some((s) => k === s)) {
    search.pageKey = k;
  } else if (['pageSize'].some((s) => k === s)) {
    search.pageSizeKey = k;
  } else {
    if (isObject(v)) {
      const typeObj = {
        default: '输入框',
        number: '数字输入框',
        price: '数字输入框',
        enum: '选择器',
        date: '日期范围',
      };
      // 获取字段类型
      const obj = checkFiledType(v);
      // 将字段与组件绑定
      obj['componentType'] = typeObj[obj.fileType] || '输入框';
      // 处理日期字段
      if (obj.componentType === '日期范围') {
        if (/Start$|Begin$|End$/i.test(k)) {
          k = k.replace(/Begin$/i, '');
          k = k.replace(/Start$/i, '');
          k = k.replace(/End$/i, '');
        } else if (/^start|^end/i.test(k)) {
          k = k.replace(/^start/i, '');
          k = k.replace(/^end/i, '');
        } else if (/^lt|^gt/i.test(k)) {
          k = k.replace(/^lt/i, '');
          k = k.replace(/^gt/i, '');
        }
      }
      form[k] = obj;
    }
  }
});
```
同理，对于表格数据，需要将字段与处理器进行绑定。

数据转换器需要在实践过程中不断进行调整，才能更好的提高它的准确率。
#### 搭建器
![图片](https://cdn.poizon.com/node-common/e04f11138ddadc1ba8c07dca09d423ae.png)
搭建器([idou](https://github.com/ctq123/idou))主要功能就是输出源码，它以一份DSL作为桥梁，在UI组件和源码之间承担承上启下的角色，用户操作搭建器，操作的是DSL，在点击生成源码的时候，再根据DSL和特定的转化规则生成源码。因此从流程上，搭建器的主要功能有两大块，分别是生成DSL和转化源码。

**转化源码**
作为搭建器的核心功能，这里介绍一下如何利用一份DSL生成多分不同的源码。

首先生成源码的过程，不同DSL/schema/AST结构，所使用的方法函数是不一样的，它们跟DSL的结构耦合性很强，但是一个很重要的点就是它们大体思路是一样的，**都是采用分治的思想，即先将其拆分成不同模块，分别处理，最后再合并在一起**。

以vue代码为例：
```javascript
<template>
  <div class="page">
    hello world
  </div>
</template>

<script>
import * as API from './api';

export default {
  data() {
    return {
      form: {
        trueName: '',
      },
    };
  },
  mounted() {
    this.queryList();
  },
  methods: {
    queryList() {},
  },
};
</script>

<style lang="scss" scoped>
.page {
  padding: 0px;
}
</style>
```
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
本文介绍了我们对自动化生成代码的探索与实践，主要包含两部分数据转换器和搭建器，利用自动化脚本抓取数据，经过数据转换器处理数据后，再打开搭建器，自动输入配置数据，最终生成源码。

实现了保持足够的简单，一行命令后便输出可运行且可维护的源码，避免重复工作，提高研发效率，并且保持足够强的灵活性和扩展性，快速与传统开发模式结合。

如果你们公司没有还不具备搭建器，采用的又是传统的开发模式，这个工具将非常适合你，目前支持生成Vue2/Vue3/React（https://github.com/ctq123/idou）
## 未来展望

1）UI库的选择，允许用户引入其他UI库

2）预览功能，打通codesandbox，实现项目的在线预览

3）允许增加自定义的模版，除了四表一局（列表，表格，表单，图表，布局）扩展外，希望能处理增加自定义模版

4）增加物料市场管理，实现物料共享和编辑

5）接通github仓库，自动上传源码（nodejs），形成整条链路的闭环。[已实现demo](https://github.com/ctq123/dslService)，但现实中肯定会遇到很多问题，比如如何解决代码冲突的问题。

6）改善UI交互
