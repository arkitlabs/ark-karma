# vditor源码解析

```@orgs/v1/slip
ID: SLIP-11008292-0adb1a2
CREATE_TIME: 2024-11-13:09:51:32
TAGS: UNASSIMILATED
CREATOR: zhu
```
## 背景介绍 

这是一篇vditor源码阅读记录。帮助记忆和理解vditor编辑器的实现原理 

## Vditor实例对象

```js
const vditor = {
   currentMode, // wysiwyg, sv, ir
   id, //  container 
   hint, // 
   lute, // markdown引擎 core
   options, // 外部配置
   originalInnerHTML, // 初始化文案
   outline, // 大纲
   tip, //
   sv, // 分屏模块
   ir, // 即时渲染模块
   wysiwyg, // wysiwyg模块
   undo,
   toolbar,
   resize,
   devtools,
   upload,
   // methods 
}
```

## 引入md解析引擎

```js
addScript(

mergedOptions._lutePath ||

`${mergedOptions.cdn}/dist/js/lute/lute.min.js`,

"vditorLuteScript",

).then(() => {

this.vditor.lute = setLute({/* options */});
```

初始化UI
* 设置主题
* 设置尺寸
* 添加各个模块 tookbar ，wysiwyg， sv，ir...
* 设置编辑模式，初始化容器内的文本
	```js
	
	const getMarkdown = (vditor: IVditor) => {
	
		if (vditor.currentMode === "sv") {
		
		return code160to32(`${vditor.sv.element.textContent}\n`.replace(/\n\n$/, "\n"));
		
		} else if (vditor.currentMode === "wysiwyg") {
		
		return vditor.lute.VditorDOM2Md(vditor.wysiwyg.element.innerHTML);
		
		} else if (vditor.currentMode === "ir") {
		
		return vditor.lute.VditorIRDOM2Md(vditor.ir.element.innerHTML);
		
		}
		
		return "";
	
	};
	
	let markdownText;
	
		if (typeof event !== "string") {
		
		hidePanel(vditor, ["subToolbar", "hint"]);
		
		event.preventDefault();
		
		markdownText = getMarkdown(vditor);
		
		} else {
		
		markdownText = event;
	}
	
	if(type === 'ir') {
		vditor.currentMode = "ir";
		vditor.ir.element.innerHTML = vditor.lute.Md2VditorIRDOM(markdownText);
	} else if （type === 'wysiwyg'）{
	    vditor.currentMode = "wysiwyg"; 
		setPadding(vditor);
		renderDomByMd(vditor, markdownText, {
			enableAddUndoStack: true,
			enableHint: false,
			enableInput: false,
		});
	} else if （type === 'sv'）{
	   vditor.currentMode = "sv";
	  let svHTML = processSpinVditorSVDOM(markdownText, vditor);
	
	if (svHTML === "<div data-block='0'></div>") {
	
	// https://github.com/Vanessa219/vditor/issues/654 SV 模式 Placeholder 显示问题
	
	svHTML = "";
	
	}
	
	vditor.sv.element.innerHTML = svHTML;
		}

* 设备兼容

## 编辑器主体模块 - IR

1. 使用`<pre>`标签动态创建container
2. 给container 添加事件
3. 通过range对象精准操作dom的编辑，选择等事件
- **文本选择**：可以用 `Range` 来选择文档中的一段文本或节点。它允许更精确的控制选中的起始点和终止点，甚至可以选择部分节点内容。
- **动态内容编辑**：`Range` 对象常用于内容编辑器，比如实现富文本编辑器（WYSIWYG）。可以用它来在选定的范围内插入、删除、或替换内容，提供灵活的文本或 HTML 片段插入功能。
- **复制和高亮显示文本**：可以利用 `Range` 对象选择特定文本，然后应用样式，比如背景色，以便高亮显示文本。常见于搜索功能的结果高亮显示，或特定文本的动态样式应用。
- **复杂的 DOM 操作**：通过 `Range` 对象，可以实现比单纯使用 `innerHTML` 更复杂的 DOM 操作。例如，您可以在两个节点之间插入内容，或只替换节点的一部分，而不会影响其他内容。 
4. 及时渲染： 内部实现Input函数，如果用户的输入涉及列表、脚注或引用等块元素，代码会将整个块元素重新渲染为 Markdown 的相应格式，代码在更新内容后会调用 Vditor 的 `SpinVditorIRDOM` 函数来解析 HTML 并重新渲染，确保内容与 Markdown 语法一致。渲染后的内容将根据编辑器的设置重新插入编辑区。当用户输入\`\`\` 时，Vditor 编辑器会识别为代码块开始标记，并自动触发代码块的 DOM 渲染、样式清除和光标位置更新，确保用户可以在代码块`<code>`中输入内容，同时保持文档的结构和 Markdown 语法一致。
## range & section
1. range表示一个包含节点与文本节点的一部分的文档片段，可以精准获取文档的起始&结束位置，获取内部节点，对文档进行插入移动复制删除等操作。
2. section代表用户当前的文本选区，和range可以配合使用选择文档，操作文档。
3. 创建range： document.createRange() 或者从selection.getRangeAt() 

# 资源链接 
* markdown引擎 lute https://github.com/88250/lute
* js实现的在线编辑器vditor https://github.com/Vanessa219/vditor
