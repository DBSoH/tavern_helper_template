---
applyTo: '**/角色卡*/**'
description: 当用户输入中明确提及 MVU 时, 你应该参考本文件
---

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
      - fixed optional keys + the same type: use `z.partialRecord(z.enum(['key1', 'key2', ...]), ${value type})`
      - dynamic optional keys + the same type: use `z.record(z.string(), ${value type})`
      - fixed required keys + different types: 'use `z.object({ key1: ${type1}, key2: ${type2}, ... })`'
      - dynamic keys but some keys are required + the same type:
          'use `z.intersection(z.object({ requiredKey1: ${type1}, requiredKey2: ${type2}, ... }), z.record(z.string(),
          ${value type}))`'
  - for special format (rare to happen): prefer `z.templateLiteral` over regex or manual parsing
  - for restrictions:
      when accepting a update that breaks the schema, users are tend to expect the update takes some effect instead of
      being discarded completely; therefore, you should try your best to use `z.transform` to convert the broken input
      to a valid input.
  - on default value:
      - prefer `z.prefault` over `z.default`
      - if a `z.object` or the whole Schema is complicated enough, set `.prefault('${suitable default value}')` or
        `.or(z.literal('待初始化')).prefault('待初始化')` for every field of it
  - when to describe: use `z.describe` only when there's no field name to explain the usage of the schema
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
