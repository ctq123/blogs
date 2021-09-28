# json-schema入门教程

> 文章简介：
> 大白话介绍json-schema，基本概念到高阶用法，由浅入深，结合实际应用分析json-schema的实际作用

## 一、缘起

**什么是json-schema？**
在回答这个问题之前，我们先了解一下它产生的背景

随着互联网的发展，前后端交互，由最初的text/html，image/*图片等文件流，到目前的application/x-www-form-urlencoded，multipart/form-data，application/json流，以及未来的application/octet-stream二进制。但毫无疑问，目前最流行的前后端交互的格式是application/json，也是我们开发者开发过程中应用最多的格式。

除此之外，json在前端的应用也越来越重要，无论未来如何发展，它始终有着不可或缺的重要地位，为什么呢？笔者认为最根本的原因是它本质上是一个对象，面向对象编程横行霸道的今天，自然而然，它的霸主地位便无法被撼动，同时它的轻量化，高可读，强扩展，高传输效率，也是一种重要原因，它正在为人与机器之间的交互扮演着一个重要的媒介角色。

正是由于json应用越来越广泛，与它相关的工具因此应用而生，json-schema就是其中之一。

假设我们有这样一种JSON数据格式
```json
{
    "id": "1432423230",
    "name": "Jeck",
    "sex": "male"
}
```
尽管这段json对开发的人来说简单明了，我们很容易就知道它是表示一个Person的字符串，但仍然存在一些问题，例如：

1）id可以是数字吗？
2）name有多少字符限制吗？
3）sex是必须的吗？可以是man和woman吗？

也许作为项目的创始人会十分清楚这些字段代表的含义，然而随着项目的增大，过了一年以后，他未必能想起来这些字段的含义，作为代码的初始开发者尚且如此，那接管项目的其他维护人更不必说了。

尽管上述json代码我们都知道什么含义（Person），里面的字段每个人的理解可能就会不一样，我们只能简单地根据属性英文的意思理解它的含义，但我们不能假定每个人的英语水平都很高，制定的属性值都能让其他人能一眼就看出什么含义。

这时候也许就有人出来抬杠了，我们团队的开发者都是中国人，属性值为什么不用中文来表示呢？中文本质上也是一种字符串呀，简单又好理解是吧，谁规定了一定要用英文，再说了……出去！对于这种杠精，我们也只能很礼貌地请他出去了，嗯嗯，言归正传……

所以，在团队日益凸显重要的今天，为团队共同指定一套json的规范就十分必要，让团队对它的理解是一致的，以此达到这样的目的：
1）减少理解成本；
2）提高开发效率；
3）降低维护成本；

那么回过头来，再回答一下什么是json-schema呢？相信聪明的你已经猜到了。
对，没错，就是json的一套规范，也有人说它是校验json的一套利器，是一个提议的IETF标准……其实都是一个含义。

如果你熟悉[typescript](https://www.typescriptlang.org/)或者[flow](https://flow.org/en/docs/getting-started/)，那么很快就能帮你理出这样的一套关系：

> **json-schema之于json，就如同typescript(或flow)之于javascript**

![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606468364189-6706aeed-c6c7-4ac4-be85-502e4a07563b.png#align=left&display=inline&height=187&margin=%5Bobject%20Object%5D&originHeight=187&originWidth=748&size=0&status=done&style=none&width=748)

## 二、介绍
#### 1）基本类型
构成JSON的两种基本类型：Object和Array
![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606468364414-89e451c4-905c-4ef8-982b-242216b67a1a.png#align=left&display=inline&height=425&margin=%5Bobject%20Object%5D&originHeight=425&originWidth=1089&size=0&status=done&style=none&width=1089)

其中value的值为：string,number,object,array,boolean, null

| 关键字 | 含义 |
| --- | --- |
| string | 字符串类型 |
| number | 数值类型，包括int和float |
| object | 对象类型 |
| array | 数组类型 |
| boolean | 布尔类型 |
| null | 空类型 |
> **_tips：没有undefined类型_**

#### 2）基本概念
既然是一套规范，那么就会有很多的语义，那么我们从最简单的例子开始介绍，如下：
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/person.schema.json",
  "title": "Person",
  "description": "it is a person object",
  "type": "object",
  "properties": {
    "id": {
      "description": "The unique identifier for a person",
      "type": "string"
    }
  },
  "required": [ "id" ]
}
```
| 关键字 | 含义 | 备注 |
| --- | --- | --- |
| $schema | 表示使用的标准特定版本，即版本控制 | 非必填 |
| $id | 表示校验json的url引用，与$ref类似，用于处理重复引用的问题 | 非必填 |
| title | 标题关键字，用于描述本json | 非必填 |
| description | 详情内容，用于描述本json | 非必填 |
| type | 描述类型，定义我们的JSON数据的第一个约束，通常为object或array | 非必填（一般都要，特殊情况下可不填） |
| properties | 对象属性的描述和限制条件 | 非必填 |
| required | 校验必填的属性 | 非必填 |

**_通常情况下，我们见得最多的格式和属性是type和properties，用于描述一个对象，用于描述一个数组的是items_**

#### 3）定义属性值
id时描述人的唯一标志，类似我们的身份证号码，这是对这条数据的唯一标识符，对数据库而言，它是必须的，同时我们限定了它的类型为string。

name是描述人的符号，这更接近人类的使用习惯，计算机通常注重标识符ID，而人类通常更注重姓名，因此它通常也是必须的，且限定它的最大长度为50

description表示该字段的描述，使人更明白这个属性值想要表达的意思

type限制该字段的类型

required是验证必填的属性列表，不填表示不验证，属性突然的增加和减少都不会验证

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/person.schema.json",
  "title": "Person",
  "description": "it is a person object",
  "type": "object",
  "properties": {
    "id": {
      "description": "The unique identifier for a person",
      "type": "string"
    },
    "name": {
      "description": "The identifier for a person",
      "type": "string",
      "maxLength": 50
    }
  },
  "required": [ "id", "name" ]
}
```
#### 4）深入了解属性
同时我们需要对性别进行规范，不然对同一含义的男性，一个人的理解为male，另一个理解为man，更有甚者会理解为“哼，男人”

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/person.schema.json",
  "title": "Person",
  "description": "it is a person object",
  "type": "object",
  "properties": {
    "id": {
      "description": "The unique identifier for a person",
      "type": "string"
    },
    "name": {
      "description": "The identifier for a person",
      "type": "string",
      "maxLength": 50
    },
    "sex": {
      "description": "The gender of the person",
      "type": "string",
      "enum": ["male", "female"]
    },
  },
  "required": [ "id", "name" ]
}
```

突然有一天，需求变了，产品经理说需要加入另外一个属性age年龄，年龄不能低于0吧，更不能大于1000，否则就成千年老妖了，木得问题，安排～

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/person.schema.json",
  "title": "Person",
  "description": "it is a person object",
  "type": "object",
  "properties": {
    "id": {
      "description": "The unique identifier for a person",
      "type": "string"
    },
    "name": {
      "description": "The identifier for a person",
      "type": "string",
      "maxLength": 50
    },
    "sex": {
      "description": "The gender of the person",
      "type": "string",
      "enum": ["male", "female"]
    },
    "age": {
        "description": "The age for a person",
        "type": "number",
        "exclusiveMinimum": 0,
        "maximum": 1000
    }
  },
  "required": [ "id", "name" ]
}
```

其中exclusiveMinimum是大于的意思，如果想包含0作为有效年龄，我们将指定minimum作为关键字，表示大于等于的意思，对应地，maximum和exclusiveMaximum则分别取值为小于等于和小于。

知道了一个人的姓名和年龄，似乎还缺少了什么，试想一下，你见到帅哥或美女后，很幸运的是你们成功搭讪上了，而且还聊得挺开心的，那么将要离别的最后一件很重要事是啥？

直男直女还在苦苦思索中，1000 years later……千年等一回，等一回啊……
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/person.schema.json",
  "title": "Person",
  "description": "it is a person object",
  "type": "object",
  "properties": {
    "id": {
      "description": "The unique identifier for a person",
      "type": "string"
    },
    "name": {
      "description": "The identifier for a person",
      "type": "string",
      "maxLength": 50
    },
    "sex": {
      "description": "The gender of the person",
      "type": "string",
      "enum": ["male", "female"]
    },
    "age": {
        "description": "The age for a person",
        "type": "number",
        "exclusiveMinimum": 0,
        "maximum": 1000
    },
    "phone": {
        "description": "The contact for a person",
        "type": "string",
        "pattern": "^[13|14|15|16|17|18|19][0-9]{9}$"
    }
  },
  "required": [ "id", "name" ]
}
```
字符串约束支持正则表达式的描述，使用pattern关键字。

随着90及00后的崛起，他们是一群有着梦想和追求自我的人，因此他们会越来越注重自己的个性和标签，而我们的起初项目居然没有这一内容字段，终于产品经理熬过了一番社会的毒打后，轮着一根20米长的棒球棍坐到了开发的旁边，鼻青脸肿的开发含泪默默的敲起了键盘……看什么看，说的就是屏幕前你！

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/person.schema.json",
  "title": "Person",
  "description": "it is a person object",
  "type": "object",
  "properties": {
    "id": {
      "description": "The unique identifier for a person",
      "type": "string"
    },
    "name": {
      "description": "The identifier for a person",
      "type": "string",
      "maxLength": 50
    },
    "sex": {
      "description": "The gender of the person",
      "type": "string",
      "enum": ["male", "female"]
    },
    "age": {
        "description": "The age for a person",
        "type": "number",
        "exclusiveMinimum": 0,
        "maximum": 1000
    },
    "phone": {
        "description": "The contact for a person",
        "type": "string",
        "pattern": "^[13|14|15|16|17|18|19][0-9]{9}$"
    },
    "tags": {
        "description": "The labels to describe a person",
        "type": "array",
        "items": [
            { "type": "string" }
        ],
        "minItems": 1,
        "uniqueItems": true
    },
  },
  "required": [ "id", "name" ]
}
```

引入items规定数组的每一项，要求数组中的内容为string类型，同时如果声明了这个字段tags，那么minItems规定它的标签至少有一个以上，且uniqueItems规定每个标签都是唯一的。

#### 5）嵌套结构
我们通常遇到的数据都不是扁平的，层级一般都会比较深，这里引入给它添加上一个地址字段。

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "$id": "http://example.com/person.schema.json",
  "title": "Person",
  "description": "it is a person object",
  "type": "object",
  "properties": {
    "id": {
      "description": "The unique identifier for a person",
      "type": "string"
    },
    "name": {
      "description": "The identifier for a person",
      "type": "string",
      "maxLength": 50
    },
    "sex": {
      "description": "The gender of the person",
      "type": "string",
      "enum": ["male", "female"]
    },
    "age": {
        "description": "The age for a person",
        "type": "number",
        "exclusiveMinimum": 0,
        "maximum": 1000
    },
    "phone": {
        "description": "The contact for a person",
        "type": "string",
        "pattern": "^[13|14|15|16|17|18|19][0-9]{9}$"
    },
    "tags": {
        "description": "The labels to describe a person",
        "type": "array",
        "items": [
            { "type": "string" }
        ],
        "minItems": 1,
        "uniqueItems": true
    },
    "address": {
        "description": "The address for a person",
        "type": "object",
        "properties": {
            "country": {
                "type": "string"
            },
            "province": {
                "type": "string"
            },
            "city": {
                "type": "string"
            },
            "region": {
                "type": "string"
            },
            "detail": {
                "type": "string"
            }
        },
        "required": ["country", province", "city", "region"]
    },
  },
  "required": [ "id", "name" ]
}
```

最后，让我们来看一下我们定义的json应该是怎样的

```json
{
    "id": "A000000000",
    "name": "溥仪",
    "sex": "male",
    "age": 112,
    "phone": "1300000000",
    "tags": ["吃饭","睡觉","发呆","末代皇帝"],
    "address": {
        "country": "天朝",
        "province": "帝都",
        "city": "帝都",
        "region": "东城区",
        "detail": "景山前街4号故宫博物馆",
    }
}
```
更多的属性规范请移步官网：[https://json-schema.org/draft/2019-09/json-schema-validation.html](https://json-schema.org/draft/2019-09/json-schema-validation.html)

## 三、高阶用法
#### 1）重用
聪明的人通常情况下都比较懒，也正因如此，他们创造了很多省事的东西，大到宇宙飞船，小到家常日用手机，这个世界上很多东西正因为懒才创造出来的，可爱的攻城狮们也是这样一群有智慧的动物。很多程序我们只想写一遍，比如上述的地址定义，有这样一种场景，一个人的地址可以有很多种，比如收件地址和发送地址，它们可能包括学校地址，公司地址，家庭地址，租房地址等等。而它们的结构都是相同的，那么我们就不可能重复定义那么多地址信息了，而json-schema也是支持这一操作的。

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "definitions": {
      "address": {
        "type": "object",
        "properties": {
            "country": {
                "type": "string"
            },
            "province": {
                "type": "string"
            },
            "city": {
                "type": "string"
            },
            "region": {
                "type": "string"
            },
            "detail": {
                "type": "string"
            }
        },
        "required": ["country", province", "city", "region"]
    },
  },
  
  "type": "object",
  
  "properties": {
        "receipt_address": {
            "#ref": "#/definitions/address"
        },
        "send_address": {
            "#ref": "#/definitions/address"
        }
  }
}
```
亦或者结合$id和$ref进行引用处理
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "definitions": {
      "address": {
        "$id": "#address",
        "type": "object",
        "properties": {
            "country": {
                "type": "string"
            },
            "province": {
                "type": "string"
            },
            "city": {
                "type": "string"
            },
            "region": {
                "type": "string"
            },
            "detail": {
                "type": "string"
            }
        },
        "required": ["country", province", "city", "region"]
    },
  },
  
  "type": "object",
  
  "properties": {
        "receipt_address": {
            "#ref": "#address"
        },
        "send_address": {
            "#ref": "#address"
        }
  }
}
```


#### 2）递归


通过上面我们知道用$ref可以处理共用的问题，那么进一步我们可以是否可以理解为自己可以调用自己，进而形成递归？yes, we can!


最常见的就是html的dom结构，它本身就是用递归来生成dom的，举个栗子:


```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "definitions": {
      "element": {
        "$id": "#element",
        "type": "object",
        "properties": {
            "name": {
                "type": "string"
            },
            "props": {
                "type": "object",
                "properties": {},
            },
            "children": {
                "type": "array",
                "items": {"$ref": "#element"},
            },
        }
    },
  },
  
  "type": "object",
  
  "properties": {
        "element": {
            "#ref": "#element"
        }
  }
}
```
> **需要注意的是，递归调用的时候，不要出现a和b的相互引用，否则会形成死循环**

## 四、实际应用
考虑到网络带宽和开发效率，通常实际的应用中我们都会省略很多，上面的描述有些会显得冗余，让我们来看一个实际的应用。以[@formily/antd](https://formilyjs.org/#/0yTeT0/jbUzUluaIG)为例

![](https://cdn.nlark.com/yuque/0/2020/png/1387052/1606468364208-a1f73db3-4a78-46df-8a85-9b945c9565b5.png#align=left&display=inline&height=214&margin=%5Bobject%20Object%5D&originHeight=214&originWidth=932&size=0&status=done&style=none&width=932)

上述内联布局对应json-schema为
```json
{
  "type": "object",
  "properties": {
    "aaa": {
      "key": "aaa",
      "name": "aaa",
      "type": "string",
      "title": "字段1",
      "x-component": "input"
    },
    "bbb": {
      "key": "bbb",
      "name": "bbb",
      "type": "number",
      "title": "字段2",
      "x-component": "numberpicker"
    },
    "ccc": {
      "key": "ccc",
      "name": "ccc",
      "type": "date",
      "title": "字段3",
      "x-component": "datepicker"
    }
  }
}
```
上述type规定了属性值的输入值类型应该是什么，对比你会发现用于验证的json-schema数据就只有

```json
{
  "type": "object",
  "properties": {
    "aaa": {
      "type": "string",
    },
    "bbb": {
      "type": "number",
    },
    "ccc": {
      "type": "date",
    }
  }
}
```
> **_tips: 上述的"type":"date"是draft7新增的类型_**

在@formily/antd实际应用中，校验属性与业务属性是混合在一起的，严格意义上说它并不是一个标准的json-schema，它结合自己的实际业务制定了一套属于自己的json-schema，你可以理解为伪json-schema，但它值得我们借鉴和学习。

通常情况下，我们还可以通过使用第三方工具来主动验证我们写的json的合法性，如[jsonschema](https://github.com/tdegrunt/jsonschema)，react的form表单schema校验[react-jsonschema-form](https://github.com/rjsf-team/react-jsonschema-form)

## 五、总结

今天这里只是介绍了json-schema的一些基础用法，只是它的冰山一角，它还有很多强大的功能，如dependencies，additionalItems，consts, allOf, anyOf, oneOf, not, if……then……else等等，更多的玩法可以去[json-schema官网](https://json-schema.org/)

最后，让我们总结一下json-schema的基础知识，它是一套用于校验json的规范，让读写都同时遵循的一套规则。

由小及大，我们再将json-schema拆解一下，那么什么是schema呢？

维基百科给出的定义是:

> The word schema comes from the Greek word σχῆμα (skhēma), which means shape, or more generally, plan.

严格意义上，schema是一种架构或模式，在数据库系统中是形式语言描述的一种结构，是对象的集合。

举一反三，就会有xml-schema，yaml-schema……它们指的都是XXX数据的一种规范和模式。

#### 参考
1）json-schema官网：[https://json-schema.org/](https://json-schema.org/)
2）json-schema使用教程文档：[http://xaber.co/2015/10/20/JSON-schema-使用教程文档/](http://xaber.co/2015/10/20/JSON-schema-%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B%E6%96%87%E6%A1%A3/)
3）@formily/antd的SchemaForm：[https://formilyjs.org/#/0yTeT0/jbUzUluaIG](https://formilyjs.org/#/0yTeT0/jbUzUluaIG)
