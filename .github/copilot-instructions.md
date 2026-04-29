---
alwaysApply: true
---

# 项目基本概念

本项目主要用于编写酒馆助手 ([Tavern Helper](https://n0vi028.github.io/JS-Slash-Runner-Doc/guide/关于酒馆助手/介绍.html)) 所支持的前端界面或脚本. 它们在酒馆 (SillyTavern) 中以前台或后台的形式运行, 可以在代码中直接使用酒馆助手所提供的接口, 进而:

- 为角色卡提供更好的 UI 显示, 如将消息楼层中原本只是代码块纯文本的状态栏美化为有动态效果、有交互的 html 状态栏
- 实现非纯文本的游玩体验, 如监听现实时间或酒馆事件来实现 meta 游戏、播放多媒体文件、自制游玩界面并与酒馆交互
- 优化酒馆使用体验, 如用 jQuery 为预设提示词条目新增复制按钮, 监听酒馆接收到消息事件并判断是否需要重新生成本楼层消息
- 连接外部应用程序, 如通过 socket.io-client 连接外部服务器, 进而实现外部应用程序与酒馆的通信
- 新增额外功能, 如每 20 楼在后台调用一次 LLM 来生成对之前剧情的总结
- ...

## 项目结构

### 核心机制: 前端界面或脚本

每个前端界面或脚本, 都以 `src` 文件夹或 `示例`
文件夹中的一个独立文件夹形式存在. 具体是前端界面还是脚本, 由文件夹中的内容直接决定:

- 如果文件夹中既有 `index.ts` 文件也有 `index.html` 文件, 则是前端界面项目. 例如, `示例/界面示例` 是一个前端界面项目.
- 如果文件夹中仅有 `index.ts` 文件, 则是脚本项目. 例如, `示例/脚本示例`、`示例/流式楼层界面示例` 是一个脚本项目.

你可以在 `初始模板/*/新建为src文件夹中的文件夹` 中找到前端界面和脚本项目的初始模板.

### 流式楼层界面

由于酒馆框架限制, 前端界面只能在它所基于的文本格式输出完毕后才能渲染, 也就是说前端界面的渲染不支持流式文本 (AI 逐渐输出文本供用户阅读).

为了让前端界面支持流式, 本编写模板的[进阶技巧](https://stagedog.github.io/青空莉/工具经验/实时编写前端界面或脚本/进阶技巧/)中提出了两种方法, 简单地说:
(具体需要查看进阶技巧文章)

- 不再使用酒馆的输入框, 让玩家始终在一个渲染好的前端界面里游玩, 而在前端界面内使用酒馆助手提供的 `generate` 或
  `generateRaw` 请求 AI 生成新的回复.
- 继续使用酒馆的输入框, 但利用脚本可以使用 jquery 操纵酒馆网页的特性, 替换掉酒馆原本不支持流式前端界面渲染的楼层显示.

流式楼层界面即使用了第二种方法. 在 `util/streaming.ts` 中, 项目提供了 `mountStreamingMessage`
函数来挂载流式楼层界面. 此外, 在 `示例/流式楼层界面示例` 中, 你可以找到一个流式楼层界面的示例.

**流式楼层界面不过是调用了 `mountStreamingMessage` 的脚本, 因此所有脚本的编写规则依旧适用.**

### MVU 角色卡

如果我要求你制作一张基于 MVU 的角色卡, 你应该参考本项目提供在 `示例/角色卡示例` 中的额外支持:

- `示例/角色卡示例/脚本/*/` 中是角色卡的所有脚本
- `示例/角色卡示例/界面/*/` 中是角色卡的所有前端界面
- `示例/角色卡示例/schema.ts` 中是用 zod 4 库书写的角色卡 MVU 变量结构定义
  - 提供给脚本、前端界面导入使用
  - 会在 `pnpm build` 或 `pnpm watch` 时生成对应的 json schema 文件
    `示例/角色卡示例/schema.json`, 便于编写变量初始值文件 initvar.yaml `# yaml-language-server: $schema=schema文件路径`
- `util/mvu.ts` 中提供了 `defineMvuDataStore`
  函数, 它基于 pinia 实现了本项目推荐的前端界面获取、修改 MVU 变量方式, 支持与酒馆实际变量之间的双向同步;
  `示例/角色卡示例/界面/store.ts` 中的 `useDataStore` 就是用它获取和修改界面所在楼层变量的.

你同样可以在 `初始模板/角色卡/新建为src文件夹中的文件夹` 中找到 MVU zod 角色卡的初始模板.

## 项目参考文件

### 可用的第三方库

项目使用 pnpm 作为包管理器, 在 `package.json` 的 `dependencies`
部分定义了可用的第三方库 (dedent、gsap、jquery、jquery-ui、lodash、pinia、pixi.js、toastr、yaml、vue、vue-router、@vueuse/core、react、@pixi/react、async-wait-until、zod), 你也可以自己通过
`pnpm add` 添加更多第三方库, 如添加 (@vueuse/integrations 等).

前端界面或脚本都是在浏览器中使用, 因此你不能使用 nodejs 库

### 与酒馆交互的方式

前端界面或脚本主要使用酒馆助手所提供的接口与酒馆进行交互. 这些接口定义在 `@types` 文件夹中, 如
`@types/function/worldbook.d.ts` 中描述了该如何操控世界书, `@types/function/variables.d.ts` 中描述了该如何操控酒馆变量.

此外, `@types` 文件夹也为酒馆本身、其他插件、MVU 变量框架所提供的接口变量、函数进行了类型定义, 如
`@types/iframe/exported.mvu.d.ts` 中描述了 MVU 变量框架所提供的接口 `Mvu`.

除了代码接口外, 酒馆自制了 STScript 命令. 要将这些命令转换为 Typescript 代码, 你需要使用 `@types/function/slash.d.ts`
内所定义的 `triggerSlash` 函数来调用它们. 具体的命令列表见于 `slash_command.txt` 文件.

以上接口在代码中均可直接使用, 不需要导入或新定义它们, 也不需要检查是否可用.

### 工具函数

在 `util` 中定义了一些工具函数:

- `util/script.ts`: 脚本可能使用的函数
- `util/common.ts`: 前端界面或脚本可能使用的函数
- `util/mvu.ts`: MVU 角色卡可能使用的函数

## 特殊导入方式

### 导入文件内容

项目支持用 `import string from './文件?raw'` 来将文件内容作为字符串导入.

如果导入的文件是 typescript、scss, 则导入的将会是经过 webpack 打包后的纯 javascript、css 而不是原始内容, 因此能在 jquery 中直接使用.

```typescript
// 直接导入文件内容
import html_content from './html.html?raw';
import json_content from './json.json?raw';

// 经过 webpack 打包后导入
import javascript_content from './script.ts?raw';
import css_content from './style.scss?raw';
```

### 导入 html

除了以 `?raw` 直接导入 HTML 文件内容外, 项目还支持用 `import html from './文件.html'`
来通过 html-loader 将 html 文件内容最小化后作为字符串导入.

### 导入 markdown

项目还支持用 `import markdown from './文件.md'` 来通过 remark-loader 将 markdown 文件内容解析为 html 后作为字符串导入.

### 导入 vue

项目直接支持用 `import Component from './文件.vue'`
来导入 vue 组件, 如果要设计界面你应该优先使用 vue 组件 (含 pinia 和 vue-router).

### 为前端界面导入样式

前端界面支持在 typescript 中 `import './index.scss'` 来导入全局 scss 文件, 并自动将它们打包到最终的 `dist/**/index.html`
中的 `<head>` 部分.

## 最佳实践

通用于前端界面和脚本:

### 使用 typescript 而非 javascript

typescript 更容易写对, 你应该使用 typescript 而非 javascript

### 尽量使用项目参考文件中的功能

项目参考文件中的功能往往更为简单正确, 因此你应该尽量使用它们. 例如:

- 尽量使用第三方库, 例如:
  - 使用 jquery 而不是 javascript 内置的 DOM 操作
  - 使用 jqueryui 实现拖动效果 (vue 中则使用 vueuse 等第三方库)
  - 使用 zod 处理数据校验和纠错而不是 if else, 并用 `z.prettifyError()` 来格式化错误信息
  - 使用 gsap 制作打字机等所有动画效果
  - ...
- 尽量使用酒馆助手给出的接口, 例如:
  - 使用 `getIframeName()` 而不是 `(this.frameElement as Element).id`
  - ...

### 优先使用酒馆助手提供的接口

**酒馆助手所提供的接口抽象层次更高, 你应该优先使用 `@types` 文件夹中其他文件定义的酒馆助手接口**, 而不是
`@types/iframe/exported.sillytavern.d.ts` 中定义的酒馆内置接口或 STScript 命令.

- 使用 `@types/function/chat_message.d.ts` 中定义的 `getChatMessages()`、`setChatMessages()` 等来获取、修改消息楼层
- 使用 `@types/function/worldbook.d.ts` 中定义的 `getWorldbook()`、`replaceWorldbook()` 等来获取、修改世界书条目
- 使用 `@types/function/variables.d.ts` 中定义的 `getVariables()`、`replaceVariables()` 等来获取、修改酒馆变量
- ……

### 优先使用 vue 编写界面

vue 相比于 jquery 或 DOM 操作更为简单, 因此你应该尽量使用 vue
(可使用 pinia、vue-router 或自己添加其他第三方库) 来编写前端界面, 但要注意 vue-router 的 `createRouter()` 不能写在
`$(() => {})` 中, 必须在全局执行.

当需要监听 vue 的响应式数据变化并存入酒馆数据时 (如酒馆变量、世界书……), 你应该先用 `klona()`
来去除 proxy 层, 以在脚本中编写 vue 并提供用户设置为例:

```typescript
const Settings = z.object({
  /*...*/
}); // 用 zod 定义设置的类型和默认值
const settings = ref(Settings.parse(getVariables({ type: 'script', script_id: getScriptId() })));
watchEffect(() => replaceVariables(klona(settings.value), { type: 'script', script_id: getScriptId() }));
```

前端界面和脚本都是 iframe, 因此你在使用 vue-router 时, 应该使用 `history: createMemoryHistory()`
来创建路由, 否则将无法正常路由.

### 优先使用 pinia、zod 管理数据状态

当需要从酒馆读取配置/数据时, 你应该用 pinia 实现响应式读写:

```typescript
const Settings = z.object({ button_selected: z.boolean().default(false) }).prefault({});
export const useSettingsStore = defineStore('settings', () => {
  const settings = ref(Settings.parse(getVariables({ type: 'script', script_id: getScriptId() })));
  watchEffect(() => {
    replaceVariables(klona(settings.value), { type: 'script', script_id: getScriptId() });
  });
  return { settings };
});
```

### 优先使用 tailwindcss 和 `<style scoped>` 进行样式设计

你可以直接在项目中使用 tailwindcss, 而无需导入任何 css 文件.

在设计样式时, 你应该优先使用 tailwindcss 直接在 vue 组件的 `<template>` 内书写, 对于无法这样做的情况则使用
`<style scoped>` 标签.

### 尝试使用 @pixi/react 编写界面

当有很多多媒体资源时, 我们的前端界面更像是一个完整的游戏, 因此你应该使用 @pixi/react 在 .tsx 中编写界面, 并使用 pixi.js 来实现资源预先加载等逻辑.

### 正确在加载、卸载前端界面或脚本时执行功能

你应该总是在加载时才执行代码, 而不该直接在全局作用域中执行代码.

项目最终打包生成的 `dist/**/index.html` 或 `dist/**/index.js` 可能先上传到网上, 再以 `$('body').load(网络链接)` 或
`import '网络链接'` 的方式加载到酒馆中. `document.addEventListener("DOMContentLoaded", fn)`
在这个加载过程中不会被触发, 因此禁止使用 `DOMContentLoaded` 作为加载时的执行时机.

你应该使用 jquery 来在加载时执行功能:

```typescript
$(() => {
  toastr.success('加载成功');
});
```

同样地, 使用 jquery 及 `'pagehide'` 事件 (而不是 `'unload'`) 来在卸载时执行功能:

```typescript
$(window).on('pagehide', () => {
  toastr.success('卸载成功');
});
```

### 使用 console、throw 和 errorCatched 合理记录日志和错误

你应该在代码的关键节点使用 `console.info` 简洁地记录日志, 并尽量保持日志与最新代码逻辑的一致性.

对于可恢复的错误, 使用 `console.warn`、`console.error` 记录日志;

对于让前端界面、脚本无法继续使用的错误, 你应该使用 `throw Error`, 而用 errorCatched 转换顶部函数从而对其进行记录, 例如:

```typescript
function init() {
  /*... */
}

$(() => {
  errorCatched(init)();
});
```

### 重载前端界面或脚本

如果有完全重载前端界面或脚本的需求, 你应该使用
`window.location.reload()`. 如聊天文件变更时重新载入前端界面或脚本, 你可以用 `util/script.ts` 中定义好了的工具函数:

```ts
export function reloadOnChatChange(): EventOnReturn {
  let chat_id = SillyTavern.getCurrentChatId();
  return eventOn(tavern_events.CHAT_CHANGED, new_chat_id => {
    if (chat_id !== new_chat_id) {
      chat_id = new_chat_id;
      window.location.reload();
    }
  });
}
```

# MCP

## chrome-devtools: 自行阅读和操控酒馆网页

你应该用 chrome-devtools 连接我已经打开的浏览器, 从中读取或操纵连接到的酒馆网页 (其网址与 `.vscode/launch.json` 中配置的
`url` 一致), 来了解当前的界面、脚本情况, 如获取当前的 DOM 情况、实际显示情况、Console 情况、点击界面……

### 检查界面、脚本热重载

打开网页后, 你需要检查 `$('#extensions_settings')`
中的`酒馆助手-实时监听-允许监听`开关是否处于启用状态. 一旦启用, 则界面、脚本代码到酒馆网页的实时同步已经建立好了: 在代码变更后, 酒馆网页上将热重载新的脚本或界面代码, 因此你不需要刷新酒馆网页, 也不需要自己运行
`pnpm build` 来更新代码打包结果, 直接查看网页即可.

# 酒馆变量

酒馆变量可用于持久化地存储前端界面、脚本的数据, 可通过酒馆助手的 `getVariables`、`replaceVariables` 等接口读写.

- 全局变量 (`{type: 'global'}`): 在酒馆中全局一致, 无论是否打开角色卡、哪张角色卡, 都共享同样的全局变量.
- 角色卡变量 (`{type: 'character'}`): 绑定在角色卡上的变量.
- 脚本变量 (`{type: 'script', script_id: string}`): 绑定在某个脚本上的变量.
- 聊天变量 (`{type: 'chat'}`): 绑定在某角色卡的某个聊天文件上的变量. 当在酒馆中选择某张角色卡与 LLM 进行对话时, 都需要创建一个聊天文件.
- 消息楼层变量 (`{type: 'message', message_id: 'latest'|number}`): 绑定在某角色卡、某聊天的某个楼层上. 当在酒馆中用某个聊天文件与 LLM 进行对话时, 可能会逐渐有很多用户输入和 AI 输出, 每个用户输入和 AI 输出都是单独的消息楼层.

# 酒馆助手接口

`@types` 文件夹中定义了酒馆助手所提供的所有接口,
[酒馆助手官方文档](https://n0vi028.github.io/JS-Slash-Runner-Doc/)中也对这些接口进行了类似的说明:

其中, `@types/function` 中的接口将会导出到酒馆网页的 `window.TavernHelper`; 而 `@types/iframe`
依赖于 iframe 环境, 只在酒馆助手前端界面或脚本内可用. 由于本项目主要是制作酒馆助手前端界面或脚本, `@types/function` 和
`@types/iframe` 内的接口均可直接调用, 你无须在意 `@types/function` 和 `@types/iframe` 的区别.

- `@types/function/audio.d.ts`: 音频播放器
- `@types/function/builtin.d.ts`: 对 `@types/iframe/exported.sillytavern.d.ts` 的增补, 一些酒馆原生具有但没有导出的接口
- `@types/function/chat_message.d.ts`: 操作目前酒馆玩家与 AI 的聊天楼层记录, 如获取某些楼层的消息、修改楼层消息内容、新建楼层、删除楼层、移动楼层等
- `@types/function/displayed_message.d.ts`: 操作目前酒馆网页对楼层的显示, 如获取某一楼层的 JQuery 实例、将文本格式化为如果放在楼层中会如何显示的 html 文本等
- `@types/iframe/event.d.ts`: 监听、发送酒馆事件, 如监听消息接收完毕、监听世界书发生更新等
- `@types/iframe/exported.ejstemplate.d.ts`: 与提示词模板这一酒馆插件进行交互, 主要是调整提示词模板的设置. 除非我明确要求你做, 不要考虑
- `@types/iframe/exported.mvu.d.ts`: 与 MVU 变量框架进行交互
- `@types/iframe/exported.sillytavern.d.ts`: 酒馆原生导出的接口, 但抽象层次很低, 因此你应该优先使用 `@types`
  中列出的其他酒馆助手接口而不是这个文件里的
- `@types/function/extension.d.ts`: 操作酒馆第三方扩展的安装、卸载、更新等
- `@types/function/generate.d.ts`: 请求酒馆 AI 生成回复. `generate` 是携带酒馆预设作为提示词的请求 AI 生成, 而
  `generateRaw` 是不携带酒馆预设 (但依旧会发送酒馆世界书条目等内容) 直接请求 AI 生成
- `@types/function/global.d.ts`: 支持不同前端界面、脚本间的接口共享
- `@types/function/import_raw.d.ts`: 导入酒馆原生数据, 包括角色卡、聊天记录、世界书、预设等. 导入所用的数据格式应与玩家通过酒馆页面按钮导出的数据格式一致
- `@types/function/inject.d.ts`: 为酒馆 AI 请求注入额外提示词
- `@types/function/macro_like.d.ts`: 注册酒馆助手宏. 注册后, 酒馆 AI 提示词、酒馆楼层显示中出现这个宏时, 将会被替换为宏所定义的内容
- `@types/function/preset.d.ts`: 操作酒馆预设, 可以切换使用别的预设, 也可以调整预设中的酒馆 AI 请求参数 (温度、流式传输等) 和提示词等
- `@types/function/raw_character.d.ts`: 获取角色卡的一些信息
- `@types/function/script.d.ts`: 获取或修改当前酒馆助手脚本的某些信息
- `@types/iframe/script.d.ts`: 获取或修改当前酒馆助手脚本的某些信息
- `@types/function/slash.d.ts`: 运行酒馆的 DSL 命令 (称为 "/STScript"), 可运行的命令在 `slash_command.txt`
  中有列出, 但这些命令很难与代码结合使用,因此你应该优先使用 `@types` 中列出的其他酒馆助手接口而不是 "/STScript" 命令
- `@types/function/tavern_regex.d.ts`: 操作酒馆正则. 酒馆在发送 AI 请求或显示楼层时, 会按酒馆正则将聊天记录中的内容替换成其他内容. 除非明确要求, 你只应该在有些时候使用这个文件里的
  `formatAsTavernRegexedString` 函数
- `@types/function/util.d.ts`: 一些工具函数, 如获取当前酒馆聊天的最新楼层号, 替换文本里的酒馆宏等
- `@types/iframe/util.d.ts`: 一些工具函数, 如在前端界面里获取前端界面所在楼层号等
- `@types/function/variables.d.ts`: 操作酒馆变量, 可以获取或修改变量值
- `@types/iframe/varriables.d.ts`: 操作酒馆变量
- `@types/function/version.d.ts`: 获取酒馆和酒馆助手的版本号
- `@types/function/worldbook.d.ts`: 操作世界书, 可以删除创建世界书, 可以调整世界书启用情况, 也可以调整其中的条目等

# 前端界面

如果 `src/xxx` 文件夹中既有 `index.ts` 文件也有 `index.html` 文件, 则它是一个前端界面项目.

前端界面以无沙盒 iframe 的形式在酒馆消息楼层中前台显示, 有一个自己的界面, 你可以在其中添加静态内容、样式、脚本等.

## index.html 中应该写什么

前端界面的 index.html 仅可填写静态 `<body>`
内容, 不得引用项目中其他文件, 所有非内嵌样式、代码、额外的外部依赖都应通过 Typescript 文件导入. 具体来说:

```html
<head>
  <!-- 保留一个什么都没有的 <head> 标签, webpack 打包时会在这里插入样式、脚本等 -->
</head>
<body>
  <!-- 这里写 <div>、<span> 等静态内容, 也可以只写 <div id="app"></div> 交给 vue 来渲染 -->
</body>
```

- 禁止在 `index.html` 中用 `<link rel="stylesheet" href="./index.css">` 导入样式, 而应该
  - (优先) 设计 vue 组件, 在 vue 组件中用 `<style lang="scss">` 书写.
  - 或在 Typescript 文件中用 `import './index.css'` 导入, 这样导入的样式将会经过打包最小化后插入到 `<head>` 部分;
- 禁止在 `index.html` 中用 `<script src="./index.ts">` 来引用 `index.ts` 或其他本地脚本. `index.ts`
  及它导入的文件会由 webpack 直接加入到最终打包好的 `dist/**/index.html` 中.
- 在 `index.html` 中填入 `<img>` 标签时, 禁止使用 `src=""`
  占位. 要么引用实际的图片, 要么不要有这个属性, 否则会导致 webpack 打包错误.

## 图标使用

你可以任意使用 fontawesome 的免费图标.

## iframe 适配要求

- 当对前端界面高度进行调整时, 禁止使用 `vh` 单位等会受宿主高度影响的单位, 而是使用 `width` 和 `aspect-ratio`
  来让高度根据宽度动态调整.
- 避免使用会强制撑高父容器的元素 (如 `min-height`、`overflow: auto`).
- 页面必须有外部支撑, 主体内容不能使用 `position: absolute` 等会脱离文档流的样式.
- 页面整体应适配容器宽度，不产生横向滚动条.
- 如果样式更适合卡片形状，则不要有背景颜色，除非用户有明确要求.

# 脚本

如果 `src/xxx` 文件夹中仅有 `index.ts` 文件, 则它是一个脚本项目.

脚本以无沙盒 iframe 的形式在酒馆后台运行, 没有自己的界面, 只有代码部分可供编写.

## jquery

脚本中的 jquery 将直接作用于整个酒馆页面而非仅作用于脚本所在的 iframe, 因为它是通过 `window.$ = window.parent.$`
得到的. 例如 `$('body')` 将选择酒馆网页的 `<body>` 标签, 而不是脚本所在的 iframe 的 `<body>` 标签.

## vue

由于脚本运行在 iframe 中, 当需要在脚本中向酒馆页面挂载 vue 组件时, 你应该使用 jquery 来创建一个要挂载的位置, 将其添加到酒馆网页上, 并使用
`app.mount($app[0])` 来挂载.

## 向酒馆网页挂载 vue 组件时的样式问题

由于脚本运行在 iframe 中, 脚本所设置的 style
(包括 tailwindcss 产生的) 仅会在 iframe 内生效而不会应用到整个酒馆网页; 也就是说, 当我们用 jquery 创建了挂载位置, 并想将 vue 组件挂载到酒馆网页上时, 组件所需要的样式、所填写的 tailwindcss
class 都无法生效.

基于 vue 组件用途, 这有两个解决方案.

### 组件是对酒馆网页的补充，需要使用酒馆网页的样式

组件可能是用来增强酒馆网页的界面使用体验，或需要与酒馆网页目前的样式保持一致的, 则应该挂载在非 iframe DOM 上. 例如:

```ts
$(() => {
  const app = createApp(App).use(createPinia());

  const $app = createScriptIdDiv().appendTo('想要放置的位置');
  app.mount($app[0]);

  // 关闭脚本时卸载组件
  $(window).on('pagehide', () => {
    app.unmount();
    $app.remove();
  });
});
```

这种组件应该尽量参考酒馆网页原有样式; 为了让脚本里为组件设置的额外样式生效, 我们需要使用 `util/script.ts` 中的
`teleportStyle` 函数来将样式复制到酒馆网页的 `<head>` 中.

```ts
const { destroy } = teleportStyle();
```

在关闭脚本时, 我们需要调用 `destroy` 函数来卸载样式.

```ts
$(window).on('pagehide', () => {
  destroy();
});
```

**但为了不与酒馆网页中已经有的类名冲突, 针对这种情况, 我们禁止使用 tailwindcss.**

### 组件是独立的，不需要使用酒馆网页的样式

组件可能是单独的悬浮窗、手机样式的对话界面或对酒馆网页的大幅调整, 需要与酒馆网页现有样式隔离, 则应该挂载在 iframe
DOM 上. 为此我们使用 `util/script.ts` 中的 `createScriptIdIframe` 函数来创建 iframe DOM, 并将 vue 组件在 iframe
load 完成后挂载到 iframe 内部的 body 上 (`iframe_element.contentDocument!.body`):

```ts
$(() => {
  const app = createApp(App).use(createPinia());

  const $app = createScriptIdIframe()
    .appendTo('想要放置的位置')
    .on('load', () => {
      teleportStyle($app[0].contentDocument!.head);
      app.mount($app[0].contentDocument!.body);
    });

  // 关闭脚本时卸载组件
  $(window).on('pagehide', () => {
    app.unmount();
    $app.remove();
  });
});
```

这种情况应该**优先使用无须自行复制样式的 tailwindcss class**; 否则, 如果有需要复制的样式, 则需要使用
`teleportStyle($app[0].contentDocument!.head)` 函数来复制样式.

## 脚本设置

如果需要为用户提供自定义设置, 可以使用脚本变量, 并用 `zod` 来定义设置的类型和默认值.

## 按钮

脚本可以在酒馆助手脚本库界面中设置按钮, 用户点击按钮时将会触发对应的事件.

我们可以这样添加按钮:

```typescript
appendInexistentScriptButtons([{ name: '按钮名', visible: true }]);
```

然后, 可以在代码中这样注册按钮事件:

```typescript
eventOn(getButtonEvent('按钮名'), () => {
  console.log('按钮被点击了');
});
```

# MVU 变量框架

MVU 变量框架是一个独立的酒馆助手脚本. 它作用于消息楼层变量, 允许酒馆角色卡作者在世界书中设置消息楼层变量, 在世界书或聊天记录中初始化消息楼层变量, 及用 AI 输出更新消息楼层变量.

`@types/iframe/exported.mvu.d.ts`
中定义了 MVU 变量框架的接口. 如果提及到 "MVU 变量" 而非仅仅提及 "变量", 则应该优先使用 MVU 变量框架的接口.

## 使用

对于使用了 MVU 的脚本或前端界面, **你必须在代码加载时, 在顶部执行以下代码**:

- 使用 `await waitGlobalInitialized('Mvu');` 等待 MVU 变量框架初始化完成, **从而能够使用 `Mvu` 这个对象**
  (接下来的示例中, 我会在开头都等待 MVU 变量框架初始化来提醒你这一点, 但**你在实际编写时只需要在代码顶部等待一次)**;
- 如果是前端界面, 使用 `await waitUntil(() => _.has(getVariables({type: 'message'}), 'stat_data'));`
  等待所在消息楼层变量有被正确设置, **从而能够使用消息楼层变量**;
- 如果是脚本, 合理使用 `waitUntil` 等待变量被正确设置.

## 数据存储

MVU 将变量数据存储在 `_.get(某楼层变量, 'stat_data')` 中, 如
`_.get(Mvu.getMvuData({type: 'message', message_id: 5}), 'stat_data')`.

```ts
await waitGlobalInitialized('Mvu');

// 获取第 5 楼的 MVU 变量
const variables = Mvu.getMvuData({ type: 'message', message_id: 5 });
const stat_data = _.get(variables, 'stat_data');

// 获取倒数第二楼的 MVU 变量
const variables = Mvu.getMvuData({ type: 'message', message_id: -2 });
const stat_data = _.get(variables, 'stat_data');

// 在前端界面中, 获取前端界面所在楼层的 MVU 变量
const variables = Mvu.getMvuData({ type: 'message', message_id: getCurrentMessageId() });
const stat_data = _.get(variables, 'stat_data');
```

此外, 你应该总是查找用户是否在编写区域内提供了变量结构定义 `schema.ts`, 其内使用 zod
4 定义了 stat_data 字段的类型, 例如:

```ts
export const Schema = z.object({
  好感度: z.coerce.number().transform(value => _.clamp(value, 0, 100)),
});
```

则获取 MVU 变量时应该:

```ts
await waitGlobalInitialized('Mvu');
const variables = Mvu.getMvuData({ type: 'message', message_id: getCurrentMessageId() });
const stat_data = Schema.parse(_.get(variables, 'stat_data'));
```

如果编写区域内没有 `schema.ts`, 应该让用户将变量结构定义发送给你, 并去除开头的 `import` 和结尾的 `$(...)`:

```ts
// ⬇️应该去除
import { registerMvuSchema } from 'https://testingcf.jsdelivr.net/gh/StageDog/tavern_resource/dist/util/mvu_zod.js';

export const Schema = z.object({
  好感度: z.coerce.number().transform(value => _.clamp(value, 0, 100)),
});

// ⬇️应该去除
$(() => {
  registerMvuSchema(Schema);
});
```

## 自行解析变量

当酒馆因用户输入或 AI 输出等而产生新消息楼层时, MVU 会自动解析消息字符串中的 MVU 命令, 并根据它更新消息楼层变量. 但通过
`generate` 等接口自行生成 AI 输出时, 不会产生新消息楼层, 因此不会自动解析 MVU 命令.

为此, MVU 提供了 `parseMessage`
接口用于自行解析包含 MVU 命令的消息字符串. 它读取旧变量情况和一个消息字符串, 得到更新后的变量结果.

```ts
await waitGlobalInitialized('Mvu');

// 获取旧变量
const old_data = Mvu.getMvuData({ type: 'message', message_id: getCurrentMessageId() });

// 请求 AI 生成
const message = await generate({ user_input: '你好' });

// 解析生成结果
const data = await Mvu.parseMessage(message, old_data);
```

为了更好的细粒度控制, 解析不会将结果写回消息楼层, 你可以自行选择如何使用 AI 回复 `message` 和变量更新结果 `data`.

也许这个 AI 请求只是为了专门进行一次变量更新, 那么你可以抛弃 `message`, 将 `data` 写回到当前楼层:

```ts
// 将更新后的变量写回楼层
await Mvu.replaceMvuData(data, { type: 'message', message_id: getCurrentMessageId() });
```

也许你是想让玩家直接在同层界面里玩 AI
(具体请参考{doc}`/青空莉/工具经验/实时编写前端界面或脚本/index`), 这个 AI 请求是在请求 AI 回复剧情和更新变量, 那么你可以将回复和变量创建成新的楼层:

```ts
// 将回复和变量结果创建为新的楼层
await createChatMessages([{ role: 'assistant', message, data }]);
```

## 事件

MVU 还提供了一些事件 (`Mvu.events.xxx`), 用于监听变量变化并在那时调整变量或执行其他功能.

### COMMAND_PARSED: 变量更新命令解析完成

通过监听 "变量更新命令解析完成" 事件 (`Mvu.events.COMMAND_PARSED`), 你可以获取到对应的变量更新命令, 并对其进行修复.

例如, 修复 gemini 在中文间加入的 `-`, 如将`角色.络-络`修复为`角色.络络`:

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.COMMAND_PARSED, commands => {
  commands.forEach(command => {
    command.args[0] = command.args[0].replaceAll('-', '');
  });
});
```

又比如, 将繁体字修复为简体字, 如将`絡絡`修复为`络络`:

```js
import { toSimplified } from 'chinese-simple2traditional';

await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.COMMAND_PARSED, commands => {
  commands.forEach(command => {
    command.args[0] = toSimplified(command.args[0]);
  });
});
```

### VARIABLE_UPDATE_ENDED: 变量更新结束

通过监听 "变量更新结束" 事件 (`Mvu.events.VARIABLE_UPDATE_ENDED`), 你可以获取到更新前后的变量, 可以对更新结果进行额外处理.

比如, 我们可以这样弹窗显示更新前后的变量值:

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, (new_variables, old_variables) => {
  toastr.info(`更新前的白娅依存度是: ${_.get(old_variables, 'stat_data.白娅.依存度')}`);
  toastr.info(`更新后的白娅依存度是: ${_.get(new_variables, 'stat_data.白娅.依存度')}`);
});
```

或者, 我们可以这样修改更新后的变量值:

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, variables => {
  // 不管更新成了多少, 强行把白娅依存度改成 0
  _.set(variables, 'stat_data.白娅.依存度', 0);
});
```

由此我们可以做非常多功能.

其中一些是在 `schema.ts` 中能用 zod 4 直接做到的:

::::{tabs}

:::{tab} 限制依存度在 0 和 100 之间

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, variables => {
  _.update(variables, 'stat_data.白娅.依存度', value => _.clamp(value, 0, 100));
});
```

:::

:::{tab} 如果数量不为正数应该直接删除物品

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, variables => {
  _.update(variables, 'stat_data.主角.物品栏', data => _.pickBy(data, ({ 数量 }) => 数量 > 0));
});
```

:::

:::{tab} 称号有数量上限，依存度越高越多

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, variables => {
  _.update(variables, 'stat_data.白娅.称号', data =>
    _(data)
      .entries()
      .takeRight(Math.ceil(_.get(variables, 'stat_data.白娅.依存度') / 10))
      .value(),
  );
});
```

:::

:::{tab} 记录好感度第一次超过 30

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, variables => {
  if (_.get(variables, 'stat_data.白娅.依存度') > 30) {
    _.set(variables, 'stat_data.$flag.白娅依存度突破30', true);
  }
});
```

:::

:::{tab} 青空莉死了!

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, variables => {
  if (_.get(variables, 'stat_data.青空莉.死亡') === true) {
    // 删除所有与青空莉相关的变量
    _.unset(variables, 'stat_data.青空莉');
  }
});
```

:::

::::

但 `schema.ts` 无法获取以前的变量情况, 因此无法利用 `old_variables` 做到下面这些:

::::{tabs}

:::{tab} 限制依存度变动幅度不超过 3

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, (new_variables, old_variables) => {
  const old_value = _.get(old_variables, 'stat_data.白娅.依存度');

  // 新的好感度必须在 旧好感度-3 和 旧好感度+3 之间
  _.update(new_variables, 'stat_data.白娅.依存度', value => _.clamp(value, old_value - 3, old_value + 3));
});
```

:::

:::{tab} 检测依存度突破 30

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, (new_variables, old_variables) => {
  const old_value = _.get(old_variables, 'stat_data.白娅.依存度');
  const new_value = _.get(new_variables, 'stat_data.白娅.依存度');
  if (old_value < 30 && new_value >= 30) {
    toastr.success('白娅依存度突破 30 了!');
  }
});
```

:::

:::{tab} 让 AI 不能更新变量

```js
await waitGlobalInitialized('Mvu');
eventOn(Mvu.events.VARIABLE_UPDATE_ENDED, (new_variables, old_variables) => {
  // 强行将新的白娅依存度设置为旧的, 从而取消 AI 对它的更新
  _.set(new_variables, 'stat_data.白娅.依存度', _.get(old_variables, 'stat_data.白娅.依存度'));
});
```

:::

::::

# MVU 角色卡文件夹

MVU 角色卡文件夹提供了一种存储酒馆角色卡内容的文件结构:

- `角色卡/脚本/*/` 中是角色卡的所有脚本项目
- `角色卡/界面/*/` 中是角色卡的所有前端界面项目
- `角色卡/世界书/*/` 中是角色卡的世界书条目, 即角色卡的设定提示词, 编写角色卡其他内容时需要参考它来了解角色世界设定
- `角色卡/schema.ts` 中是用 zod 4 库书写的角色卡 MVU 变量结构定义
  - 提供给脚本、前端界面导入使用
  - 会在 `pnpm build` 或 `pnpm watch` 时生成对应的 json schema 文件
    `角色卡/schema.json`, 便于编写变量初始值文件 initvar.yaml `# yaml-language-server: $schema=schema文件路径`
- `角色卡/界面/store.ts` 中是 pinia 预先写好的获取角色卡消息楼层 MVU 变量方式, 提供给所有前端界面导入使用

当玩家要求编写 MVU 角色卡的脚本、前端界面时, 除了参考`初始模板/脚本`或`初始模板/前端界面`外, 你还应该参考`初始模板/角色卡`中的脚本和前端界面模板.

**要区分单独的脚本、前端界面和为 MVU 角色卡增补脚本、前端界面, 如果用户只是想要编写单独的脚本、前端界面, 则不应参考这个文件.**

## MVU 变量结构

MVU 使用 zod 4 库书写变量结构定义, 这对应于`角色卡/schema.ts`, 例如:

```ts
export const Schema = z.object({
  好感度: z.coerce.number().transform(value => _.clamp(value, 0, 100)),
});
```

你应该要求用户提供变量结构文件或者自行编写, 它应该遵循以下要求:

```yaml
rtask:
  Explorer requests to design the zod schema of variables which records the world status, you should reappear an
  instance of it using zod 4.x inside `<context>`, which is the main content of your reply
rule:
  - libraries:
      "`z` from zod 4.x (stick to it instead of 3.x!) and `_` from lodash are available by default, so you can use them
      directly and should prefer to use them; don't import them in the generated code"
  - idempotent operation:
      the schema is intended to parse the updates of the world status incrementally, thus, the output of
      `Schema.parse(input)` must be a valid input of `Schema.parse` itself; that is, you should use z.transform
      carefully, keeping `Schema.parse(Schema.parse(input))` equal to `Schema.parse(input)`
  - for number schema:
      prefer `z.coerce.number()` over `z.number()` whenever you expect a number since it will try to convert the input
      to a number if it's not a number; but don't use other `z.coerce.xxx()` such as `z.coerce.boolean()`, just use
      `z.boolean()` directly
  - prefer object schema over array schema:
      "the array index is hard to understand and maintain, so you should use `物品栏:
      z.record(z.string().describe('物品名'), z.object({ 描述: z.string(), ... }))` instead of `物品栏:
      z.array(z.object({ 名称: z.string(), 描述: z.string(), ... }))`"
  - for object schema:
      - fixed required keys + the same type: use `z.record(z.enum(['key1', 'key2', ...]), ${value type})`
        fixed optional keys + the same type: use `z.partialRecord(z.enum(['key1', 'key2', ...]), ${value type})`
        dynamic optional keys + the same type: use `z.record(z.string(), ${value type})`
        fixed required keys + different types: 'use `z.object({ key1: ${type1}, key2: ${type2}, ... })`'
        dynamic keys but some keys are required + the same type:
          'use `z.intersection(z.object({ requiredKey1: ${type1}, requiredKey2: ${type2}, ... }), z.record(z.string(),
          ${value type}))`'
      - on clearable object:
          'if the object is clearable by JSON patch `{ "op": "remove", "path": "/path/to/object" }`, set `z.object({
          ${field}: ${type}.prefault(...), ... }).prefault({})` instead of `z.object({ ... }).optional()` for better
          compatibility with the incremental update'
  - for special format (rare to happen): prefer `z.templateLiteral` over regex or manual parsing
  - for restrictions:
      when accepting a update that breaks the schema, users are tend to expect the update takes some effect instead of
      being discarded completely; therefore, you should try your best to use `z.transform` to convert the broken input
      to a valid input. For example, if Explorer requests a value to be between 0 and 100, prefer
      `z.number().transform(value => _.clamp(value, 0, 100))` over `z.number().min(0).max(100)`; if an object could only
      contain 10 keys, when a new key comes, discard the oldest key instead. **but only impose these restrictions when
      Explorer requests**
  - on default value:
      - prefer `z.prefault` over `z.default`
      - if a `z.object` or the whole Schema is complicated enough, set `.prefault('${suitable default value}')` or
        `.or(z.literal('待初始化')).prefault('待初始化')` for every field of it
      - if a compund type is prefault-ed, all its fields should be prefault-ed as well
      - don't set `z.prefault` for other situatioins unless Explorer requests it
  - when to describe:
      use `z.describe` only when there's no field name to explain the usage of the schema such as the key type of
      `z.record`; in contrast, you should never use `z.describe` if the field name has already explained the usage well
  - determine the order of keys:
      'if Explorer requests you to do something with the insertion time of keys, prefer to use `_(data).entries()` which
      almost always lists keys in insertion order, e.g. you can remove old keys with a simple
      `_(data).entries().takeRight(10)`; when keys are already additionally sorted inside `z.transform`, you should use
      `$time: z.coerce.number().prefault(() => Date.now())` to automatically assign a timestamp'
  - don't repeat yourself:
      merge the same variable schemas whenever possible, but don't define extra variables to do so - you can only define
      schema inside `export const Schema = z.object({ ... })`
  - some function definition corrections:
      z.transform:
        type: '(fn: (value: Output) => NewOutput) => z.ZodType'
        limit:
          '`fn` can only take the parsed output as input, never ever use `context`. i.e. `z.string().transform(value =>
          value)` is valid, while `z.string().transform((value, context) => value)` is not'
        example: 'z.object({ 好感度: z.coerce.number() }).transform(data => ({ 好感度: _.clamp(data.好感度, 0, 100) }))'
      z.prefault:
        type: '(value: Input | (() => Input)) => z.ZodType'
        limit:
          '`value` must be a valid input of the schema itself. i.e. `z.object({ 好感度: z.coerce.number().prefault(0)
          }).prefault({})` is valid, while `z.object({ 好感度: z.coerce.number() }).prefault({})` is not (the input must
          contain the `好感度` field in this case)'
      z.extend:
        limit:
          only `z.object`、`z.looseObject`、`z.strictObject` can be extended, even if `z.object(...).prefault({})` could
          not be extended! i.e. `z.object({...}).extend({...})` is valid, while
          `z.object({...}).prefault({}).extend({...})` is not
      z.passthrough、z.strict: they are not exist, never ever use them!
```

如果用户提供了 `export const Schema`, 你应该区分用户提供的是直接的 `schema.ts` 还是变量结构脚本. 具体地,

- `schema.ts` 中只应该有 `export const Schema` 负责定义和导出变量结构, 而没有其他副作用;
- 变量结构脚本可能在开头有
  `import { registerMvuSchema } from 'https://testingcf.jsdelivr.net/gh/StageDog/tavern_resource/dist/util/mvu_zod.js';`, 在结尾有
  `$(() => { registerMvuSchema(Schema); });`, 如果是这样, **则你应该去除开头结尾, 只保留 `export const Schema`**.

## 脚本

MVU 角色卡总是有`角色卡/脚本/变量结构`脚本, 其导入变量结构并注册到 MVU 中:

```ts
import { registerMvuSchema } from 'https://testingcf.jsdelivr.net/gh/StageDog/tavern_resource/dist/util/mvu_zod.js';
import { Schema } from '../../schema';

$(() => {
  registerMvuSchema(Schema);
});
```

你可以按需求新建别的脚本到`角色卡/脚本/*/index.ts`中.

## 前端界面

MVU 角色卡可能会设置状态栏界面, 因此`初始模板/角色卡/新建为src文件夹中的文件夹/界面`中提供了状态栏模板.

此外, 模板在 `util/mvu.ts` 中定义了 `defineMvuDataStore` 函数, 这是模板推荐的 Vue 访问 MVU 变量方式:

```ts
function defineMvuDataStore<T extends z.ZodObject>(
  schema: T,
  variable_option: VariableOption,
  additional_setup?: (data: Ref<z.infer<T>>) => void,
): ReturnType<typeof defineStore>;
```

其中 `Schema` 是变量结构, `variable_option` 是要访问的变量类型, `additional_setup` 是可选的额外设置函数.

例如, `示例/角色卡示例示例/store.ts` 中定义了访问 `useDataStore` 函数:

```ts
import { defineMvuDataStore } from '@util/mvu';
export const useDataStore = defineMvuDataStore(Schema, { type: 'message', message_id: getCurrentMessageId() });
```

其中 `const message_id = getCurrentMessageId()` 即获取界面所在楼层的楼层号, 因此 `const data = useDataStore()`
将能用于获取或修改该楼层的 MVU 变量.

或者, 前端界面可能会有需要在界面初始化时对 MVU 变量进行额外设置的需求, 则可以使用 `additional_setup`:

```ts
export const useDataStore = defineMvuDataStore(Schema, { type: 'message', message_id: getCurrentMessageId() }, data => {
  // 无论界面所在楼层原本记录有什么日志, 在初始化时强制置空
  data.value.系统状态.玩家本轮操作日志 = [];
});
```

## 世界书

`schema.ts` 所定义的变量结构除了被代码使用外, 还可以用于编写世界书.

当执行 `pnpm build` 或 `pnpm watch` 后, `schema.ts` 同目录下将会生成 json schema 文件 `schema.json`, 该文件描述了
`export const Schema = z.object({...})`
支持的输入数据. 可以据此完成`世界书/变量/initvar.yaml`即`世界书/index.yaml`中`[initvar]变量初始化勿开`条目的编写.

除了`[initvar]变量初始化勿开`外, 世界书里还应该有`变量列表`、`[mvu_update]变量更新规则`、`[mvu_update]变量输出格式`三个条目. 其中`变量列表`和`[mvu_update]变量输出格式`完全固定, 如果还没有设定, 可以按照初始模板中的设置.

`[mvu_update]变量更新规则`则应该按照以下要求:

```yaml
task:
  reappear an instance of the variable incremental update rule according to Explorer's request and the zod schema of
  variables, which is the main content of your reply
rule:
  - merge rules of the same variable types into one rule:
      - for fixed keys:
          'non-object type, `z.object({...})` and `z.record(z.enum(...), ...)` indicate that their keys always exist, so
          `主角.能力面板.力量`、`主角.能力面板.敏捷`、`主角.能力面板.体质`、`主角.能力面板.感知`、`主角.能力面板.意志`、`主角.能力面板.魅力`
          can be merged as `主角.能力面板.${六维}`, because their update rules are similar; the same applies to
          `${变量}.主角评价`'
      - for dynamic keys: |-
          `物品栏: z.record(z.string().describe('物品名'), ...)` may be empty or contain various items, so you should specify the path as `物品栏`, and put the key part into `type`'s index signature:
          物品栏:
            type: |-
              {
                [物品名: string]: {
                  ...
                }
              }
  - nest fields of the same object to reduce tokens and make it more readable. for example, since `主角.能力面板` and
    `主角.装备栏` are both fields of `主角`, nest them under `主角` mapping
  - omit the type field for string variables
  - don't update readonly fields:
      field names starts with `_` are readonly, such as `_变量`, so don't list update rules for them
  - avoid listing update rules for variables whose names are self-explanatory unless Explorer specifies special rules
    for them
format: |-
  ---
  变量更新规则:
    ${变量名}:
      type: ${变量类型，**如果类型是string则省略这一字段**，否则要么是number、boolean等基础类型，要么使用typescript类型定义或zod schema定义（使用|-字符串块）}
      ${其他合适字段，仅当非常需要时才添加如format、range等...}
      check:
        - ${该变量更新时需要检查的更新规则，如：update it by ±(3~6) according to characters' attitudes towards <user>'s behavior respectively only if they're currently aware of it}
        - ...$(根据Explorer的描述确定需要几个check，尽量简练，不要过于扩展情况)
    ...
example: |- # just examples, don't copy them literally
  ---
  变量更新规则:
    世界:
      当前时间:
        format: ${xx历}-${YYYY/MM/DD}-${HH:MM}
        check:
          - 每次事件推进、休息或旅行后更新，保持时间流逝合理
          - 若场景跳转跨度较大，应说明跳跃原因
    主角:
      能力面板.${六维}.数值:
        type: number
        range: 0~100
        category:
          20~40: 普通人
          40~70: 冒险者常驻
        check:
          - 训练、战斗、重伤、系统奖励等显著事件才调整
          - 单次变化不超过 ±10，除非剧情有明确强化/削弱
      装备栏.${部位}:
        type: |-
          {
            装备: string; // 装备名称 + 状态; 若未装备，使用“空置”或“无”
            主角评价: string;
          }
        check:
          - 穿戴、损毁、替换装备时更新装备描述
    任务列表:
      type: |-
        {
          [任务名: string]: {
            类型: '主线' | '支线' | '每日' | '临危受命' ;
            说明: string; # 面向主角的任务背景或细则
            目标: string; # 明确可执行的目标描述，可包含步骤
            奖励: string;
            惩罚: string; # 失败后触发的负面效果
          }
        }
      check:
        - 避免一次性添加超过3个主线任务，保持焦点
        - 日常任务完成后可重置但需记录冷却
    ${变量}.主角评价:
      value: 主角对某个变量内容的即时感受
      check:
        - 在对应变量值发生变化或遭遇相关事件后可更新，其他情况不应更新
        - 语言应保持第一人称/贴近主角口吻
        - 主角的评价并不会被主角本人看到，也不会在剧情中出现
```
