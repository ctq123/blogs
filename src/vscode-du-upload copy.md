# 从vscode插件的角度看国际多语言一体化建设

![图片](https://cdn.poizon.com/node-common/a20a81941648ebcc4a077359ad177c1b.png)

今天跟大家分享一下从vscode插件的角度看国际多语言的变迁以及实践探索。

内容主要分四个部分。一是对国际多语言的解决方案介绍，二是我们项目遇到的问题和痛点，三是我们为了解决这些问题引出的vscode插件的开发介绍，四是对未来的展望

## 国际多语言i18n
首先来聊一聊实现国际多语言的解决方案
- 调用机器翻译API
- i18n
- react-intl
- 本地自动化翻译
- 翻译平台

#### 调用机器翻译API
首先是调用机器翻译API，比较经典就是调用google，有道，百度等提供出来的API接口，这种方案优点是相对比较简单且快捷。比如调用google翻译的API：

```javascript
const translate = require('google-translate-api');
translate('你好', { to: 'en'}).then(res => {
  console.log(res.text);
})
```

它缺点也很明显，准确率不达标，过于简单和机械，对公司内部的术语往往束手无策。同时，翻墙和收费等问题也是需要考虑的因素。因此这种方案通常只作为机器翻译用于参考。

#### i18n
它的全称为internationalization，意为国际化，取首尾字母+字母个数的形式缩写为i18n。它应该是目前最流行的一套解决方案，在vue应用中绝对处于垄断的地位，当然它不仅仅局限于vue应用，react，小程序和移动端都支持。它适合定制化的应用，翻译的效果也是杠杠的，缺点是不够智能化，扩展性也相对较弱。

#### react-intl
它跟i18n有点类似，但它有一定的局限性，往往只有react应用可以使用，也是一种定制化的解决方案，优缺点与i18n相当，但它的跨端和跨语言相对i18n要弱。

#### 本地自动化翻译
这种技术方案其实只是在i18n和react-intl上进行一层优化，技术的本质上并没有变，只是利用插件(webpack/vscode/node等)对项目文件进行扫描，提取文件中的文字进行提取和替换，自动生成源文件保存在本地，再根据源文件翻译各种目标语言文件。它的优点是自动化，缺点是只能处理文字，对css样式等额外功能无法处理。

#### 翻译平台
翻译平台分为外部翻译平台和内部翻译平台，顾名思义，外部翻译平台通常都要付费才能享受，而内部翻译平台通常需要一定的研发成本和精力，所以往往只有大厂才会出现这样的产物。它们的效果就不用多介绍了，但都需要付费才能享受此服务（内部研发费也是其中的一种）。

## 现状
目前我们国际的项目使用的技术栈是i18n，但由于历史原因使用的姿势也是各有不同，也造成了各种问题。首先是内部翻译平台刚起步，因此它与老的技术方案会同时并存一段时间。目前多语言方案对本地开发人员出现如下问题：
- 无法本地语言预览（代码层面）
- 使用场景多样化，方案没有统一
- 新增翻译文案教繁琐
- 查找翻译文案教麻烦
- 引用容易出错

https://poizon.feishu.cn/wiki/wikcnT0HyYNqNKfIWKWf6b9ctse#
## 分析
针对上述的痛点我们需要实现的目标：
+ 实现一键上传文件到CDN
+ 实现文件的预览功能

<video id="video" width="400" height="300" controls="" preload="none"><source id="mp4" src="https://cdn.poizon.com/node-common/8e93aa1202772b186b610d8fa402cf5a.mov"></video>
### 一键上传文件到CDN
#### 思路
要实现快捷上传到CDN的功能，首先需要自定义快捷命令或自定义右键菜单，其次出发命令后需要唤起弹窗供用户选择要上传的文件，然后将文件上传到CDN上，最后将路径显示在当前鼠标位置即可。
#### 实现

**1）自定义右键菜单**

由于自定义快捷命令对新手有一定的学习成本，这里采用更加友好的自定义右键菜单形式。

由于vscode插件开发有命令扩展形式，自定义右键的菜单只需要在vscode插件初始化项目中直接配置package.json文件即可。配置如下：

```javascript
// package.json
"contributes": {
  // 命令扩展
  "commands": [
    {
      // 自定义命令key
      "command": "extension.du.upload",
      // 自定义命令名称name（右键菜单显示的名称）
      "title": "上传文件到CDN"
    }
  ],
  // 右键菜单扩展
  "menus": {
    // 编辑内容类型
    "editor/context": [
      {
        // 当鼠标激活的时候启动命令
        "when": "editorFocus",
        // 命令绑定，调用对应的命令key
        "command": "extension.du.upload",
        // 菜单分组类型
        "group": "navigation"
      }
    ]
  }
}
```
配置的含义上面标注得很清楚了，这样自定义右键的菜单就完成了。

**2）唤起弹窗**

首次使用会自动转跳到配置页面，引导用户进行相关的配置，如图：

![图片](https://cdn.poizon.com/node-common/2293c406ce99ae0b059bc90899b4d922.png)

接下来通过设置监听extension.du.upload命令回调函数，调用vscode.window.showOpenDialog即可唤起用户本机的文件选择弹框，如下
![图片](https://cdn.poizon.com/node-common/dd105e32d8db471aa8303dd5d7ec428e7975da314e243d3a6479edd353838a8b.png)

当然你可以选择过滤掉一些文件类型，当用户选择好文件后，接下来就是要将选好的文件上传到CDN。

**3）上传CDN**

如何上传CDN，参考原有的绝大部分逻辑，不过需要注意的是vscode属于node环境，很多浏览器中的DOM和BOM属性是无法直接使用的，比如fetch，formData，文件数据等，都需要采用node环境的依赖包。

另外一点需要注意的是，新的文件名的生成会出现长度不一的情况，比如上传媒体类型的数据，新生成的文件名会相对较长，这里采用md5命名法。

上传文件到CDN这一步会遇到比较多的坑，主要是上传文件的调试阶段，一方面是老逻辑有坑，另一方面是node-fetch和form-data这两个库引起的一些问题。不过经过一番摸索之后，最终都成功解决掉。

上传成功之后，我们可以拿到对应的url了，不过在node-fetch中拿不到对应的返回url，浏览器中是可以的，这里需要自行拼接url。通过判断它是否上传成功状态来决定返回拼接好的url，还是返回null。

```javascript
const res = await fetch(host, options);
if (res.ok) {
  return `${host}/${key}`;
} else {
  return null;
}
```

**4）路径回填**

最后需要将对应的url插入到当前鼠标的位置，这里直接调用activeEditor.edit的回调函数，将内容注入即可

```javascript
// 获取当前编辑页面
const activeEditor = vscode.window.activeTextEditor;
if (!url || !activeEditor || !activeEditor.selection || !activeEditor.selection.active) {
  return;
}
// 获取当前鼠标位置
const activePosition = activeEditor.selection.active;
// 将url插入到鼠标的位置
activeEditor.edit((edit: any) => {
  edit.insert(activePosition, url);
});
```

至此，一件上传文件到CDN的功能就实现了。

### 文件预览
#### 思路
要实现文件的预览功能，首先需要获得当前鼠标悬浮的位置信息，其次是提取位置文本信息中的url，最后调用弹框显示对应的内容即可。
#### 实现

**1）悬浮位置**

首先想要获取当前鼠标悬浮的位置以及它对应的文本信息，我们需要用到它的钩子函数vscode.languages.registerHoverProvider，这样就可以在它的回调函数中我们就可以拿到它对应的位置信息。

```javascript
vscode.languages.registerHoverProvider('*', {
  provideHover(doc, position) {
    // do something
  }
})
```

**2）过滤url**

其次我们需要对它的文本信息进行过滤，提取其中的url信息。同时我们应该判断当前鼠标是否悬浮在url之上，如果是在url之上，我们才进行显示，否则不显示。

```javascript
// 获取当前的行
const line = doc.lineAt(position).text;
// 提取对应的url
const reg = /(http|https):\/\/([\w.]+\/?)[^';"\s]*/g;
const arr = reg.exec(line);
if (Array.isArray(arr) && arr.length) {
  const url = arr[0] || '';
  // url的位置
  const urlBeginIndex = line.indexOf(url);
  const urlEndIndex = urlBeginIndex + url.length;
  // 当前鼠标的位置
  const cursorPosition = position.character;
  // 鼠标处于url之上
	if (cursorPosition >= urlBeginIndex && cursorPosition <= urlEndIndex) {
    // do something
  }
}
```

**3）弹窗显示**

最后，我们需要根据不同的文件类型进行预览和显示。由于鼠标悬浮显示的弹窗内容只支持字符串类型，因此我们通过vscode.MarkdownString对文件url进行渲染。
```javascript
// 弹窗类型
const markdown = new vscode.MarkdownString();
// 鼠标处于url之上
if (cursorPosition >= urlBeginIndex && cursorPosition <= urlEndIndex) {
  switch(true) {
    // 判断是否为图片
    case imgReg.test(url):
      markdown.appendMarkdown(`![](${url})`);
      break;
    // 判断是否为视频文件
    case videoReg.test(url):
      // vscode.MarkdownString不支持媒体类
      markdown.appendMarkdown(`<video id="video" controls="" preload="none"><source id="mp4" src="${url}"></video>`);
      break;
    // 判断是否为音频文件
    case audioReg.test(url):
      // vscode.MarkdownString不支持媒体类
      markdown.appendMarkdown(`<audio id="audio" controls="" preload="none"><source id="mp3" src="${url}"></audio>`);
      break;
  }
  markdown.isTrusted = true;
  return new vscode.Hover(markdown);
```

由于鼠标悬浮窗口最终会合并所有的MarkdownString，比如你定义了自己的MarkdownString内容A，但假如它内部有内容B,最终显示的内容会成为B+A，也就是说你无法完全自定义自己的悬浮窗口内容，至少都会带上它内部默认的内容。

![图片](https://cdn.poizon.com/node-common/5d749d85538a470d08e51f4187ac7a6f2a568a8700756ffb7e1566bcf0f4c3d1.png)

如上图红色框就属于默认内置MarkdownString，图片才是我自己定义的内容。

同时markdown本身是支持媒体显示的，但vscode.MarkdownString暂时不支持媒体类渲染，查询了官网issue也有人遇到过相同的类型，暂时还没有解决。这里有试过加html之类的标志，但最终都不行，如果有童鞋知道解决方案，恳求告知～

这里我想到的是能不能自定义悬浮窗口，目前我所了解到的都是通过MarkdownString方式创建，能不能通过HTML之类的创建鼠标悬浮窗口，后续还需好好摸索一番。

至此，我们基本实现了预先设想的目标：一键上传和预览功能。日常工作中，如果你也想用这个便捷的工具，可以在商店中直接输入poizon或者dewu即可搜索到这个插件。

## 未来展望

通过实现文件的一件上传CDN和文件的预览功能，我们其实可以看到vscode插件的威力，只需要简单的代码实现，就能帮我们解决很多问题。

vscode插件能实现的功能远远不止于此，我们可以从研发流程，规范管理，问题排查等方面入手考虑，它最终给我们带来的提效点。

当然它也有自身的缺陷性，比如只能在vscode本身使用，只支持node相关的语法，工程化的插件无法在外面调用自动化工作，插件的效率也是相对的，假如插件代码写得不好的话，也会拖慢编辑器。

任何工具都有利弊，我们只需要根据自身业务的实际需要，权衡利弊的同时，朝着更有利的方向前进即可。


