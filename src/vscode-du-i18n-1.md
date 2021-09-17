# 从vscode插件的角度看国际多语言一体化建设（上）

![图片](https://cdn.poizon.com/node-common/a20a81941648ebcc4a077359ad177c1b.png)

今天跟大家分享一下从vscode插件的角度看国际多语言开发的实践和探索。

国际多语言一体化的建设主要分为上下两篇，分别对线上和线下两种开发模式展开介绍。

本文主要是针对线下本地的开发模式，也是目前行业内比较通用简单的开发模式。主要内容分三个部分，一是本地开发遇到的问题和痛点，二是针对这些痛点进行分析和解决，最后是内容的总结。

## 现状
不知道大家在国际多语言开发的时候，有没有遇到这样一个问题，这个key代表啥意思呀？然后根据这个key好不容易找到对应的语言描述，发现有的key真的跟描述对应不上。新增语言文案需要在多个语言文件中切换，太繁琐了，尤其是输入一批文案时，新增引用就增多，一不小心就引用出错。这些都是传统开发模式下所面临的问题。

同时我们引用方式也是多样化的，包含js，json，yaml等格式，如下
![图片](https://cdn.poizon.com/node-common/c9525b8dc4dd88fcb1fe1dc2e993a528.png)
![图片](https://cdn.poizon.com/node-common/b9d3023f37fafddad858dcdbe50ba970.png)
![图片](https://cdn.poizon.com/node-common/d1203f560c29060555c1979806383a10.png)

总体来说，本地开发模式的体验并不是很好。

总结下我们日常开发所面临的问题：
- 无法本地语言预览（代码层面）
- 使用场景多样化，方案没有统一
- 新增翻译文案教繁琐
- 查找翻译文案教麻烦
- 引用容易出错

## 分析
针对上述的痛点我们需要实现的目标：
- 实现本地语言预览
- 支持配置化，满足不同场景
- 支持输入自动补全
- 支持点击转跳变量声明处
- 支持显示语言切换
- 支持批量新增翻译文案

### 实现本地语言预览
![图片](https://cdn.poizon.com/node-common/4bd1d9e31014dab8c691c7d601840297.png)
#### 思路
要实现vscode本地语言的预览功能，首先获取语言的map集合，然后扫描文件中所有的key值，通过vscode插件将对应的value值显示出来。
#### 实现

由于不同的项目引用的方式存在差异，因此需要支持配置化，首先是文件的路径是不固定的，其次文件中引用的key方式也未必相同，因此识别key的方式也需要支持配置化，最后需要确定默认显示的语言（一般是中文），中文也会分简体中文和繁体中文，简体中文不一定会存在，再有语言文件名也不是是固定，需要支持配置化。如下：

![图片](https://cdn.poizon.com/node-common/23f8143b2f9513896ca6d652e9caa68f.png)

同时首次使用插件时需要自动转跳到改配置页面，引导用户进行配置。

**1）获取语言的map集合**

首先根据配置化的语言文件夹路径读取所有的语言map集合，但也有一些H5项目是没有所有语言集合的概念，因为高性能的原因，语言集合是跟着单个页面走的，它内嵌在页面之中，这种就需要判断它是不是内嵌在页面之中。

通过vscode.workspace.findFiles，先扫描用户配置的语言文件夹目录下是否存在语言文件，若存在则提取出文件中的对象进行map保存处理。

然后再扫描当前文件是否属于包含语言集合，若存在则提取对应的语言集合

![图片](https://cdn.poizon.com/node-common/4392ee6ff11b36f24bf0f3bdb74d682e.png)

如上这种是内嵌在页面之内的语言集合，它与普通的所有语言文件存储在一个文件夹下是有所区别的。

**2）扫描文件中key值并显示**

通过vscode.window.activeTextEditor.document获取当前文件路径和文件内容，根据正则表达式当前匹配文件中的所有key值，以其它们所有的位置信息。

最后根据map提取对应的value值，这里提取值的时候有两个地方需要注意：
- 一是对key的转换处理，如{ "a.b": 1, a: { c: 2, d: [{e: 'e'}, {f: 'f'}] } }，获取obj, 'a.d.1.f'
- 二是对value的转换处理，如value的值不是基本类型，需要转换成string，否则用户将会看到的是一个[object Object]这样的信息，对用户来说这是无用信息。

最后通过调用editor.setDecorations(decorationType, ranges)方法将对应的信息显示出来。

这里需要特别注意的是每次调用decorationType前，需要调用decorationType.dispose()对其进行注销清空处理，否则当你监听文件保存的变化时，会显示多份相同的提示信息。

```javascript
function showDecoration(editor: vscode.TextEditor, positionObj: object, lang: object) {
	if (editor && positionObj) {
    const foregroundColor = new vscode.ThemeColor('editorCodeLens.foreground');
    
    // 一定要先清空，否则会出现重复的情况，即使将全局变量decorationType改成内部局部变量也无效
    if(decorationType != null) {
      // 释放操作
      decorationType.dispose();
    }

    decorationType = vscode.window.createTextEditorDecorationType({
      isWholeLine: true,
      rangeBehavior: vscode.DecorationRangeBehavior.ClosedClosed,
      overviewRulerColor: 'grey',
      overviewRulerLane: vscode.OverviewRulerLane.Left,
    });
    
    // 设置提示
    const decorationOptions: any = [];
		Object.entries(positionObj).forEach(([k, v]: any) => {
			const p: any = k.split('-');
			if (p && p.length === 2) {
				const startPosition = editor.document.positionAt(p[0]);
				const endPosition = editor.document.positionAt(p[1]);
				const range = new vscode.Range(startPosition, endPosition);
				const value = getObjectValue(lang, v);
				const text = getStringText(value); 
				const item = {
					range,
					renderOptions: {
						after: {
							contentText: ` ${text}`,
							color: foregroundColor,
              opacity: '0.6',
						},
					}
				};
        decorationOptions.push(item);
			};
		});
    // 设置回显
    editor.setDecorations(decorationType, decorationOptions);
	}
}
```

### 支持输入自动补全以及key值预览
#### 思路
要实现输入的自动补全，需要监听当前用户的输入，判断是否属于i18n的引用，若是则需要给出对应的提示供用户提供选择，以提高用户的开发效率和引用的正确率。
#### 实现

**监听用户输入并预判用户行为**

通过vscode.languages.registerCompletionItemProvider监听用户的输入，通过提取当前用户的输入，判断是否要引用i18n，若用户要引用i18n，则应该将该语言库中或当前页面所包含的key-value设置入vscode.CompletionItem中，同时需要注意转换value为object和array的问题。

![图片](https://cdn.poizon.com/node-common/698cf95e377044dc2357d242eb7cc8a4.png)

这里需要注意判断当前用户输入最后的一个字符，根据最后一个字符处理不同的回填内容。

```javascript
const provideCompletionItems = (document, position) => {
	const lang = getCurLang();
	// 初步语言集合以及配置是否有效
	if (includeKeys && lang && Object.keys(lang).length) {
		const lineText = document.lineAt(position).text;
		const inputText = lineText.substring(0, position.character);
		const reg = getRegExp(includeKeys);
		// 判断用户输入是否开始引用i18n
		if(reg.test(inputText)) {
			const lastChar = inputText[inputText.length - 1];
			return Object.entries(lang).map(([k, v]: any) => {
				const item = new vscode.CompletionItem(k);
				item.kind = vscode.CompletionItemKind.Value;
				// 转换对象和数组成string
				item.detail = getStringValue(v);
				// 根据最后一个字符处理回填的内容
				item.insertText = lastChar === '(' ? `'${k}'` : `${k}`;
				return item;
			});
		}
	}
}
```

### 支持显示语言切换

一般情况下，用户看到中文预览就已经足够了，但也存在这样一种用户确实想切换其他语言来查看的场景，或者检查该字段在该语言是否已填写，起到一种预检查的作用，如下：
![图片](https://cdn.poizon.com/node-common/d477a8b5b8fb042c95f6695183ae1085.png)
![图片](https://cdn.poizon.com/node-common/99bf857f6f397b62644e7f3d7edf47e9.png)
![图片](https://cdn.poizon.com/node-common/556ceca561db6a9a66eaf0f0a9a6caf4.png)

这种就是还没有填写的情况，起到一定的预检作用

### 支持点击转跳变量声明处

对于还没有填写内容的情况需要快速地转跳到对应的位置，引导用户填写，以提高开发和查找效率，降低维护成本。

通过监听用户的点击转跳功能，根据引用的路径反向查找对应语言对应的key所处的位置，然后转跳到对应的位置。这里需要注意的是key的来源可能是多个的，需要考虑混合使用的情况。

增加快速查找功能，按ctrl健点击即可转跳至声明处

![图片](https://cdn.poizon.com/node-common/7693dfda018d04f434a63d246d2a49b8.png)
![图片](https://cdn.poizon.com/node-common/5fc90d9d88c371957ef9b88bef1a264c.png)

### 支持批量新增翻译文案

我们每次需要新增文案的时候都需要在多个文件一个一个地将翻译写进入，这样处理起来效率比较繁琐且低效，这些工作其实可以交给机器去处理。

同时我们注意到翻译员大多给出到开发的翻译文本一般都是excel，因此如果需要支持批量输入翻译的话，那么就需要对其进行特殊处理，转换成我们想要的格式。

翻译人员给的结果：
![图片](https://cdn.poizon.com/node-common/7063ce6785efa37cdfd731ec63d4ebd7.png)

我们需要对其进行批量输入和转换
![图片](https://cdn.poizon.com/node-common/3a18535668345c446b2a294f0169f388.png)
里面包含要转换的目标语言，生产格式，以及编码前缀。并且根据当前项目自动适配对应的目标语言和生产格式，目前转换格式支持json和yaml，用户直接从excel中拷贝目标语言，点击批量新增即可，如下
![图片](https://cdn.poizon.com/node-common/86be1f53d439c9c668579ca5cea643d9.png)
![图片](https://cdn.poizon.com/node-common/2fc21c8800fb0fa9092689dddd294198.png)
![图片](https://cdn.poizon.com/node-common/51d178776085fa0094ef4dcc9f5a0f61.png)

## 总结

针对传统线下的开发模式我们已实现的目标：
- 实现本地语言预览
- 支持配置化，满足不同场景
- 支持输入自动补全
- 支持点击转跳变量声明处
- 支持显示语言切换
- 支持批量新增翻译文案

当然这是对目前普遍流行的本地开发模式，并且已发布上线，在vscode插件市场中输入Du I18N即可下载使用，下一篇我们会聊聊线上国际多语言平台相关的介绍。

