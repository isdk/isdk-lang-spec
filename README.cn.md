# ISDK Language Specification(Draft)

> 【[English](./README.md)|中文】
---

> **ISDK** (**Intelligent Structure Definition of Knowledge**, 智能结构化知识定义语言): AI 工程领域缺失的那门编程语言。

ISDK Language 是一门完整的高级编程脚本语言，旨在将提示词上下文工程转变为一个健壮的软件工程学科。它将零散的一次性提示转化为结构化、可复用、可继承、可组合、可调用的智能模块（智能体/流程），并以更接近自然语言的语法降低书写与协作成本。
这也使得该语言成为能为AI轻松掌握使用的原生AI语言。

它通过在面向开发者的 **`Prompt Context`** （源代码）和面向机器的 **`Prompt Packet`** （编译渲染后的 AI 输入）之间建立清晰的层次来实现这一目标，这可以被称为**Just-in-Time Context (JIT Context)**。

**语言分层结构：**

ISDK 将AI提示词脚本组织成开发者熟悉的、面向对象的结构：

* **AI 函数 (`function`)**： 用于特定任务的可复用、可调用的AI脚本（例如 `@file()`, `@url()`）。**一切皆“函数”**。
* **AI 类 (`type`)**： 封装的、可继承的智能体蓝图。这允许您创建基础智能体并对其进行扩展，覆盖其配置和逻辑。
  * **角色类(`char`)**：供开发者使用的，内置的基础角色类。
  * **智能体类 (`agent`)**： 封装的、可继承的智能体。
* **AI 应用或库项目**： 在AI项目目录下由多个 ISDK 文件组成的独立项目，拥有清晰的入口点(`index.ai.yaml`文件或与项目目录名称同名的脚本文件作为入口点)。

**核心工程原则：**

* **声明式接口：** YAML **front-matter** 用于定义显式的 `input` 输入槽位（支持 Jinja2 模板）和 `output` 输出契约（使用 JSON Schema），使智能体成为可预测和可测试的组件。
* **可编程对话：** 语言主体区允许使用内置命令（`$if`, `$print`）、AI 驱动的内容替换（`[[JOKE]]`）和行内函数调用（`@calculator(5+2)`）来动态构建对话。`---` 分隔符支持在单个文件中进行多轮对话模拟和上下文管理。
* **高级编排：** 通过**智能体链式调用**（`->`）和复杂的**角色群聊**模型，原生支持复杂工作流，允许多个智能体公开或私下协作以解决任务。
* **工具使用与 API 集成：** 在统一的语法内，无缝集成外部**工具（`tools`）**或通过 JSON Schema 定义作为 API 调用者的智能体。开发者始终保持完全控制：是您的脚本调用 AI，而不是反过来。

**安全性与可移植性：**
ISDK 的设计旨在实现**模型无关**，允许您构建一个可在各种 LLM 之间移植的通用提示词脚本库。对于敏感应用，它提供**提示词上下文加密**功能，以在使用本地 LLM 提供商时保护知识产权。
**新使命：** 有了 ISDK，提示词工程师的工作重心将从一次性的调整，转向开发一个健壮、通用、可复用的 AI 组件库，从而为所有人加速技术创新。

> 💡 名词解释: 在 AI ISDK Language 中，**`提示词上下文`** 本质上是按角色组织的消息序列（结构化对话）。每次运行推理，ISDK 会依据作用域、优先级与合并规则，将其渲染为 **`Prompt Packet`**（LLM Input）。同时提供显式输入槽位（支持可扩展的模板语法，默认 Jinja2）、输出契约，以及继承与组合能力，便于构建可复用、可维护的智能体与流程。

## Quick Start

欢迎来到简化版的快速入门指南，助您轻松开启 AI 脚本编写体验。本指南专注于让创建和执行交互式脚本的过程更加直观和简便。让我们开始吧！

### 结构化对话消息

每一行代表对话中的一次交流。可以通过“角色: 消息”的形式指定发言者，角色可以是系统（`system`）、助手（`assistant`）或用户（`user`）。如果未明确指定角色，则默认为用户发言。

> 💡 提示: **system** 角色是AI的"身份设定卡"，告诉AI"你是谁"和"该怎么做"，是构建高质量对话体验的基础。

可编程提示词引擎脚本的示例如下:

```yaml
system: "您是一位AI助手。"
"10加18等于多少？" # 这是用户角色消息
# user: "what's 10 plus 18?" # 这是同样的角色消息
```

你也可以使用标准的YAML列表语法来表示:

```yaml
- system: "You are a helpful assistant."
- user: "what's 10 plus 18?"
```

三个短划线(`---`) 或星号 `***` 表示一个新的对话开始,并且之前的上下文会被清除，从头开始新的对话。eg:

`test.ai.yaml`:

```yaml
system: "您是一位AI助手。"
# 下面为第一个对话的起点,而分隔线上面的对话内容会被隐藏，可以当作系统提示词,它们不会被输出或记录.
---
"10加18等于多少？"
assistant: "[[result]]" # 执行AI,替换为AI传回的结果result
$print: "?=result"      # 打印大模型传回的结果
# 下面开始新对话,回到第一次的起点
---
user: "10加12等于多少？"
assistant: "[[result]]" # 执行AI,替换为AI传回的结果result
```

注: 以`$`开头的名字表示的是脚本内置指令,

结果：

```bash
$ai run test.ai.yaml --no-stream
# Or search the script id in current directory
# $ai run test --no-stream -s .
" 10加18等于28。"
 10加12等于22。
```

#### 结构化系统（角色）消息规范

系统（角色）消息是大型语言模型（LLM）对话流程中的核心指令，作为对话的初始上下文对模型行为起决定性引导作用。该消息需始终以`system`角色声明，并置于对话开头。

系统消息内容（`content`）支持两种形式：

1. 纯文本格式：直接输入引导模型的文本内容
2. 结构化对象格式（推荐）
   * `background`: 可选的背景信息文本内容
   * `content`: 必填的系统消息正文
   * `output`: 可选的输出约定列表,
   * `notes`: 可选的注意事项列表

**多系统消息合并规则**： 当AI脚本中存在多个系统消息时

1. 若内容为纯文本字符串，视为结构化系统消息的`content`内容进行处理
2. 若为结构化对象，各字段将进行深度合并:
   * `background`字段： 各个`background`字段之间用`\n`分隔，并最终合并为单个字符串。
   * `content`字段： 各个`content`字段之间用`\n`分隔，并最终合并为单个字符串。
   * `notes`字段： 各个`notes`字段合并为一个字符串数组

例子：

```yaml
# 系统消息1（结构化）
system:
  background: "你是一位学术论文翻译专家"

# 系统消息2（纯文本）
system: "优先考虑翻译准确性"

# 系统消息3（结构化）
system:
  content: "采用专业术语"
  notes: ["核对参考文献格式"]

# 最终标准化格式：
system:
   background: "你是一位学术论文翻译专家"
   content: |
     优先考虑翻译准确性
     采用专业术语
   notes:
     - 核对参考文献格式

# 最终合并结果
system: |-
  你是一位学术论文翻译专家

  优先考虑翻译准确性
  采用专业术语

  Notes:
  * 核对参考文献格式
```

这种设计保证了配置的向下兼容性，既支持传统纯文本配置的平滑过渡，又能通过结构化对象对系统指令进行定制化扩展。

注意:

* 使用`SystemNotesTitle`配置选项可以控制修改系统消息`notes`当中的标题。
* 使用`SystemOutputTitle`配置选项可以控制修改系统消息`output`当中的标题。注意`SystemOutputTitle`中必须存在模板变量`{{type}}`，否则视为无效。

#### 角色群聊

角色群聊功能以结构化自然语言的方式进一步完善了对话系统，同时使得多个智能体之间能够方便地进行协作和交流，从而更高效地完成复杂的任务。

角色群聊支持公开对话、私聊对话和多角色对话，使得对话更加灵活和有针对性。

* 指定对话角色：
  1. 在角色紧跟的方括号中指定角色名，指定的多个对话角色之间用逗号`,`分隔。例如 `user[@dobby]: "..."`
  2. 或者在消息内容的最前面指定角色，同样要加上前缀`@`字符，多个角色之间用逗号`,`分隔。
* 公开对话: `user[@dobby]: ...` 或 `user: "@dobby, ..."` 表示 `user` 角色对 `dobby` 角色公开说的话， `dobby` 角色必须回应。
* 私聊对话: `user[@dobby(私)]: "..."` 或 `user: "@dobby(私),..."` 参数 `PM`|`DM`|`私` 均表示 `user` 角色对 `dobby` 角色私聊说的话，其他角色看不见。
  * **注意**： 消息中只要任意一个角色带上了私聊参数，那么该消息就是是私聊，其他角色看不见。
* 多角色对话: 如果要把消息同时发送给多个角色，角色之间用逗号分隔，例如 `user[@dobby(PM), @other]: "..."`, `user: "@dobby(PM), @other ..."`。

在消息内容中使用 `@role` 的方式，使得结构化消息更加接近自然语言。

下面具体来看一个例子，首先是主要控制群聊的脚本:

`guide.ai.yaml` 脚本：

```yaml
---
description: "You are a professional guide. You can guide the user to complete the task."
name: "guide"
roles: # 使用的角色列表，key为角色名，值为角色脚本ID
  translator: char_translator
  dobby: char-dobby
---
system: You are a professional guide. You can guide the user to complete the task.
--- # New dialogue starts here
user: "@dobby, I want to go to the moon."
guide: "@translator, translate the dobby's message to chinese without explanation."
user: How to go to the moon?
dobby: "[[AI]]"
$echo: "" # disable print last result
```

注意:

* 调用方脚本必须是角色`char`类型
* 主控方脚本(`guide`)可以不必是`char`类型脚本
* `@all` 表示`roles`列表中的所有角色
* `user: '@dobby, ...'` 表示`user`角色对`dobby`角色公开说的话, `dobby`必须回应。
  * `user: '@dobby(私), ...'`: `PM`|`DM`|`私` 表示`user`角色对`dobby`角色私聊说的话，其他角色看不见。
  * 如果要把该消息同时发送给多个角色，那么角色之间用逗号分隔，eg, "`user: '@dobby(PM), @other, ...'`"
* `dobby: "[[AI]]"` 表示调用`dobby`生成一条消息并赋值给`AI`变量，`dobby`会看到前面当前dialogue中所有公开的消息。

`char_translator.ai.yaml` 角色脚本:

```yaml
---
type: char
name: "translator"
description: You are a professional multi-lingual translator.
---
--- # New dialogue starts here
```

`char-dobby.ai.yaml` 角色脚本:

```yaml
---
type: char
description: |-
  Remember to always use the character name as prefix to refer to yourself.
  Dobby was a brave, loyal house-elf, willing to put himself in dangerous situations when he knew it to be the right thing to do.
  Dobby was also very loyal to the few friends he had. Dobby considered himself to be a good house-elf, though other house-elves seemed to find his desires and proclamations of being a free house-elf to be shameful.
character:
  name: "Dobby"
  roles: # 使用的角色列表，key为角色名，值为角色脚本ID
    dobby: char-dobby
---
user: Who are you?
# the following messages will be shown in the chat under the `---`
---
assistant: I am Dobby. Dobby is happy.
```

### 定义输入与输出

为了构建可复用的提示词工程,我们需要在文件的开头使用 [front-matter](https://jekyllrb.com/docs/front-matter/) 来配置提示词工程的输入和输出规则。`front-matter`第一行以`---`开始, 配置最后以`---`行结束.

在定义输入(`input`)配置时，采用列表的形式来进行设置。列表中的每一项可以是一个字符串（代表参数名称），也可以是一个对象（其中 key 为参数名，value 为具体的输入配置）。对于输入配置对象，支持以下几个关键属性：

* `required`: 是否为必填项，默认为`false`
* `index`: 指定该输入参数在数组参数中的索引位置
* `description`: 输入参数的描述信息(可选项)
* `default`: 当作为可选项时，描述默认值内容
* `type`: 输入参数的数据类型(可选项)，默认为`string`
* `hidden`: 是否隐藏该工具的输入参数(可选项)，默认为`false`，仅当为可选项或有默认值时生效
* `hiddenInResult`: 是否在工具返回结果中隐藏该输入参数(可选项)，默认为`false`, 如果`hidden`为`true`，则`hiddenInResult`会自动生效
* `enum`: 该参数只允许的值列表(可选项)
* `example(s)`: 该参数的示例值或列表(可选项)

输出(`output`)配置按照[JSON Schema](https://json-schema.org/)规范来定义预期的结构化输出。

以下是一个简化的翻译智能体角色的脚本示例:

```yaml
---
# 下面是输入输出配置
input:
  # 待翻译内容的语言，默认为"auto"自动检测
  - lang
  # 必填，待翻译内容
  - content:
      required: true
      index: 0
      description: 待翻译内容
      type: "string" # 默认为string类型，可省略。
  # 目标语言
  - target: {required: true}
output:
  type: "object"
  properties:
    target_text:
      type: "string"
    source_text:
      type: "string"
    source_lang:
      type: "string"
    target_lang:
      type: "string"
  required: ["target_text", "source_text", "target_lang"]
# 可选配置
parameters:
  # 使用后面的参数,将设置强制json输出格式,确保大模型总是输出正确的json格式.
  response_format:
    type: "json"
# 设置 content 和 target 输入项的默认值
content: "I love my motherland and my hometown."
target: "Chinese"
---
# 下面为脚本内容
system: |-
  You are the best translator in the world.

  Output high-quality translation result always!
user: "{{content}}\nTranslate the above content {% if lang %}from {{lang}} {% endif %}to {{target}}."
```

执行该脚本会按照指定的json格式输出, eg, 上面默认项的输出为:

```bash
#假设上面脚本的文件名为translator.ai.yaml
$ai run translator.ai.yaml
```

```json
{
  "target_text": "我爱我的祖国、我的家乡。",
  "source_text": "I love my motherland and my hometown.",
  "target_lang": "Chinese",
  "reason": ""
}
```

```bash
# 设置自己的输入参数,替换默认值
$ai run translator.ai.yaml '{content: "10加18等于28。", lang: "中文", target: "English"}'
```

#### 特殊的预定义的输入参数

当使用某些特定名称定义参数时，它们具有特殊含义：

##### keepOrginal 参数

是否保留转换前的原始文本

* **类型**: `boolean` or `string`
* **描述**: 表示是否保留原始文本，默认值为 `false`
  * 如果设置为 `true`，原始文本将存储在返回结果的 `_original` 字段中。
  * 如果 `keepOriginal` 是字符串，则原始文本将存储在该字符串指定的字段中。
* **目的**: 该参数允许用户控制返回结果中是否包含原始文本。
* **行为**:
  * 当设置为 `true` 时，系统会自动在预定义字段 `_original` 中包含原始文本。
  * 当设置为字符串时，系统会根据提供的字符串值，将原始文本存储在自定义字段中。
* **使用方法**:
  * 若要使用默认字段名保留原始文本，将 `keepOriginal` 设置为 `true`。
  * 若要在自定义字段中保留原始文本，将 `keepOriginal` 设置为所需的字段名称（作为字符串）。
* 注意:
  * 该字段是不可枚举的。
  * 如果结果对象中已经有同名字段，`keepOriginal` 的功能将失效，即不会覆盖保存原始文本。

##### keepThinking 参数

当LLM模型或AI脚本配置为支持思考模式时，该参数可以控制是否保留思考过程。

* **类型**: `boolean` or `string`
* **描述**: 是否保留思考过程。默认值为 `false`
  * 如果设置为 `true`，思考过程将存储在返回结果的 `_thinking` 字段中。
  * 如果 `keepThinking` 是字符串，则思考过程将存储在该字符串指定的字段中。
* **目的**: 该参数允许用户控制返回结果中是否包含思考过程。
* **行为**:
  * 当设置为 `true` 时，系统会自动在预定义字段 `_thinking` 中包含原始文本。
  * 当设置为字符串时，系统会根据提供的字符串值，将思考过程存储在自定义字段中。
* **使用方法**:
  * 若要使用默认字段名保留思考过程，将 `keepThinking` 设置为 `true`。
  * 若要在自定义字段中保留思考过程，将 `keepThinking` 设置为所需的字段名称（作为字符串）。
* 注意:
  * 如果结果对象中已经有同名字段，`keepThinking` 的功能将失效，即不会覆盖保存思考过程。

#### 响应结构化输出格式类型(response_format.type)

1. 文本格式,当没有设置`response_format.type`时，则为原始文本输出
2. JSON 文本格式，当设置`response_format.type`为`json`时，则会将文本视作json格式, 并强制转为JSON对象输出,如果无法转换，则会抛出错误信息。
3. YAML 文本格式，当设置`response_format.type`为`yaml`时，则会将文本视作YAML格式, 并强制转为JSON对象输出,如果无法转换，则会抛出错误信息。
4. [NObj(Natural Object)](./natural-language-object.cn.md) 格式，当设置`response_format.type`为`nobj`时，则会将文本视作NObj格式，并强制尝试使用NObj格式转换为JSON对象。

**注意:**

* `input` 可以约定输入项中哪些是必填项. 其中 `index` 为可选的基于位置的参数索引。
* `output` 是用 [JSON Schema 规范](https://json-schema.org/)约定的输出
  * 默认只输出大模型返回的文本内容,如果希望返回大模型的全部内容(文本内容和参数),那么请设定`llmReturnResult: .`.
  * 如果设置了强制输出为`JSON`(`response_format: {type: json}`),那么就只能一次完成,不能续写,必须根据输出json内容的最大长度设置`max_tokens`.
  * 当在消息模板中使用该变量时`{{output}}`,默认会以自然语言的格式输出。
    * 可以通过在`parameters`设置参数`naturalOutput`为`false`禁用该功能。禁用后该消息模板会以`JSON`字符串的形式输出。
  * `autoBuildOutputPrompt` 参数控制是否自动构建`output`的输出内容提示作为系统提示。默认为`true`。
* `NObj` 格式是近似自然语言的结构化简单对象格式，详细介绍参阅： [Natural Language Object](./natural-language-object.cn.md)
* **forceJson** 强制将输出转换为JSON对象，默认为`true`, 当强制转换失败,则会抛出错误信息。当设置为`false`,如果转换失败,则会返回原始文本。
  * 仅当存在`ouput`规范以及响应输出格式才有效。
  * 对响应输出格式类型 `NObj`, `yaml` 和 `json` 都有效。

### 模板化消息 - 轻松定制你的消息

#### 什么是模板化消息？

模板化消息是一种通过在消息中使用预定义的“`变量占位符`”来生成最终消息的方法。就像填空题一样，用户只需要提供模板化的消息，系统会自动将变量的内容插入到文本中，生成完整的消息。

#### 模板格式

模板格式默认采用的是HuggingFace使用的轻量级[jinja2模板](https://en.wikipedia.org/wiki/Jinja_(template_engine))语法，这种模板格式非常灵活，可以让你轻松地定制消息。

目前支持的模板格式有:

* `hf`: 默认模板格式. 别名: `huggingface`. 也就是`huggingface`用的[jinja2模板](https://en.wikipedia.org/wiki/Jinja_(template_engine))格式;
* `golang`: 别名: '`localai`', `'ollama`'. 也是 `ollama` 和 `localai`用的模板类型;
* `fstring`: 别名: `python`, `f-string`, `langchain`. 是 `langchain`在用的.

注意： 如无必要，请尽量使用模板的默认格式`hf`。

#### 按需格式化消息: 何时使用模板化消息？

模板化消息可以在配置中预设，也可以在脚本执行时动态生成。通常情况下，模板中的变量会在消息传递给大型模型时才被替换（这称为按需格式化或“`延后`”替换）。如果你希望立即格式化消息，可以在相关文本前加上`#`字符作为前缀。或者通过设置参数`formatStringStrategy`进行改变:

* `Raw`: 不做任何的格式化
* `Deferred`: 按需格式化，默认策略
* `Immediate`: 立即格式化

**注意事项**:

* 模板默认在调用`$AI`时渲染，除非使用`#`前缀进行即时格式化。
* 模板数据来源的优先级顺序是：`函数参数` > `prompt`对象 > `runtime`对象。
* 当执行`$set`, `$get`, `$ret`，`$echo`操作时会立即对消息进行格式化。

#### 示例

假设你想创建一个角色Dobby，你可以这样写：

```yaml
---
name: Dobby
description: |-
  你是《哈利·波特》系列中的Dobby。
---
system: "扮演{{{name}}。{{description}}"
```

你也可以将消息放在配置文件中：

```yaml
---
name: Dobby
prompt:
  description: |-
    你是《哈利·波特》系列中的Dobby。
  messages:
    - role: system
      content: "扮演{{{name}}。{{description}}"
---
```

#### 参数优先级

如果在不同的地方定义了相同的参数，系统会按照以下优先级顺序使用： `函数参数` > `prompt`对象 > `runtime`对象。

```yaml
---
prompt:
  description: |-
    You are Dobby in Harry Potter set.
---
- system: "{{description}}" # 默认消息是延后替换
- $AI: # 当执行$AI时,消息中的参数才会被替换.
    # 调用参数的优先级最高,覆盖了prompt对象中定义的description
    description: 'You are Harry Potter in Harry Potter set'
```

### 高级模板化消息替换

在消息中,使用两个方括号`[[ ]]`定义高级替换，目前高级替换有`AI替换`，`调用替换`和`正则替换`.

#### 高级AI替换

在消息中,使用双方括号`[[ ]]`定义特别的模板变量进行高级AI替换,顾名思义,方括号的内容将被AI替换, 同时该模板变量的值也被存放在`prompt`对象中. eg,

```yaml
assistant: "讲个笑话：[[JOKE]] 希望您喜欢！"
-> $print(JOKE)
$ret('')
```

此机制允许根据AI响应动态插入内容。

在该例子中AI的内容被存放在 `prompt.JOKE` 变量中，不过你可以直接引用`JOKE`变量名. assistant的消息也将被替换为:

```bash
$ai run joke.ai.yaml
joke: 讲个笑话： Why don't scientists trust atoms? Because they make up everything. 希望您喜欢！

{ 0: "Why don't scientists trust atoms? Because they make up everything."
  JOKE: "Why don't scientists trust atoms? Because they make up everything."
  ...
```

**注意**:

* 如果没有高级AI替换,上一次的大模型返回结果依然会被存放在`prompt.RESPONSE`上,也就是默认会有`[[RESPONSE]]`模板变量.
* 如果需要添加模型参数，参数应放在变量冒号后面，多个参数之间用逗号分隔。例如：`[[RESPONSE:temperature=0.01,top_p=0.8]]`

##### 限定 AI 回答内容为列表中的选项

若要强制 AI 只能从列表中选择，可以使用以下格式：`[[FRUITS:|苹果|香蕉|橙子]]`。这意味着 AI 只能从中挑选苹果、香蕉或橙子其中之一。

如果需要从本地随机选择一个（使用计算机本地的随机数生成器而非 AI），则需加上 `type='random'` 参数：`[[FRUITS:|苹果|香蕉|橙子:type='random']]`,该参数可以缩写为: `[[FRUITS:|苹果|香蕉|橙子:random]]`

#### 高级脚本调用消息替换

在消息中，我们支持通过调用外部脚本或指令来进行内容替换。注意这些脚本或指令需要返回字符串结果。例如：

```yaml
user: "#五加二等于 [[@calculator(5+2)]]"
```

注意事项:

* 前缀`#`表示立即对字符串进行格式化处理。
* **BROKEN CHANGE(v0.6.0)** 外部脚本或指令应放于两个方括号内，前缀`@`表示调用外部脚本，其ID为`calculator`。若要调用内部指令，则使用前缀`$`，如`[[@$echo]]`；若无参数，则需省略括号。
  * 注意，必须放置于两个方括号内，表示替换的内容。以前版本(0.5.18)是不需要加方括号的，现在加上了群聊模式，为了区分，所以更改了格式。
* 若插入在文本中间，请确保前后各有一个空格。格式化后，多余的空格将被自动移除。

现在有一个例子，是关于如何用这种方式来加载并生成文件摘要的脚本:

```yaml
user: |-
  为下面文件生成摘要:
  [[@file(file.txt)]]
```

##### 外部智能体多轮交互调用方式

```yaml
---
type: char
name: 'Harry Potter'
description: "Act as Harry Potter"
---
- assistant: "你好,dobby!,我是{{name}}!"
- $for: 3 # for 循环建立3轮对话
  do:
    - user: "[[@dobby(message=true)]]"
    - assistant: "[[AI]]" # 调用AI产生Harry Potter的回答
```

#### 正则表达式(RegExp)格式化替换

在消息中可以使用正则表达式[[`/RegExp/[opts]:VAR[:index_or_group_name]`]]进行内容替换。例如：

```yaml
user: |-
  输出结果,并用 '<RESULT></RESULT>' 包裹
assistant: "[[Answer]]"
---
user: "基于如下的内容: [[/<RESULT>(.+)</RESULT>/:Answer]]"
```

参数说明:

* `RegExp`: 正则表达式字符串
* `opts`: 可选参数，用于指定正则表达式的匹配选项。例如，`opts`可以是`i`，表示忽略大小写。
* `VAR`: 要替换的内容,这里是保存了助手回答的`Answer`变量;
* `index_or_group_name`: 可选参数，表示要替换的内容是正则表达式匹配到的哪一部分。可以是正则表达式中的捕获组索引号（从1开始）或命名的捕获组。
  * 当该参数不存在时: 如果正则存在捕获组,则默认为索引号1; 如果没有捕获组,则默认为整个匹配结果.

注意事项:

* 在消息中间的正则表达式必须用空格和其它内容区分开来.
* 如果没有匹配则直接返回`VAR`的内容.

### 智能体脚本的链式调用

在消息中,可以将结果发给其它智能体

如果没有带参数,那么会把AI结果作为`content`参数传给智能体,eg,

`list-expression.ai.yaml`:

```yaml
system: "只列出计算表达式, 不要计算结果"
---
user: "三块糖加上5块糖"
assistant: "[[CalcExpression]]"
-> calculator  # 传入智能体的实际输入参数就是上一次的结果: {content: "[AI生成的计算表达式]"},返回值在`LatestResult`中
$echo: "#一共是{{LatestResult}}块糖"
```

`calculator.ai.yaml`:

```yaml
---
parameters:
  response_format:
    type: "json"
output:
  type: "number"
---
system: 请作为一个计算器，计算表达式结果. 只输出结果。
---
user: "{{content}}"
```

注意： 在日常使用中，请勿使用AI进行数字运算，这不是AI所擅长的,比如，请尝试让它进行小数运算，eg: `ai run calculator '{content: "13.1 + 4.857"}'`,不过是可以用CoT提高准确度。

如果带参数,那么会把AI结果`content`合并传入参数一起传给智能体, eg,

```yaml
user: "讲个笑话吧！"
assistant: "[[JOKE]]"
# 传入智能体的实际输入参数是: {content: "[这里是由AI生成的笑话]", target_lang: "葡萄牙语"}
-> translator(target_lang="葡萄牙语") -> $print
```

**注**: 如果脚本返回值是`string`/`boolean`/`number`,那么都会将该返回值放到结果(`result`)的`content`字段;如果返回值是`object`,则会直接将对象里的内容作为结果(`result`)传递给智能体.

**Tips**:

* **脚本返回值**：脚本执行的最后一条指令的返回值,为脚本的返回值
* **自动执行AI**：当脚本中存在提示消息并且一直到脚本结束也没有执行调用过`$AI`或者提示消息的最后一条是user消息,那么脚本会自动在结束时执行一次`$AI`调用, 此行为可通过`autoRunLLMIfPromptAvailable`配置
* **输出模式**：脚本默认采用流式输出，可使用`--no-stream` 开关禁用流式输出
  * 注意: 并非所有LLM后端均支持流式输出.

### 智能体脚本的继承

智能体脚本可以通过`type`特性来继承另一个脚本的内容和配置。这里举个例子，假设我们要创建一个像“Dobby”这样的角色：

```yaml
---
# 继承自char角色类型脚本
type: char
# 这里是“char”角色的一些具体设置
# 角色的名字
name: "Dobby"
# 对角色的描述
description: "Dobby 是哈利波特世界里的一个小精灵"
---
# 用户提问
user: "你是谁?"
---
# 根据角色设定的回答
assistant: "我是 Dobby。Dobby 很开心。"
```

接下来我们先简单地创建一个基础的角色类型脚本叫做 char，这样上面的脚本就可以继承它了：

```yaml
---
# 表明该脚本是一个类型脚本
type: type
# 定义该角色类型的输入配置
input:
  - name: {required: true}  # 必须提供的信息：角色的名字
  - description             # 可选的信息：对角色的描述
---
# 系统根据提供的信息来指导角色的行为
system: |-
  你是一个聪明、多才多艺的角色扮演者。
  你的任务是根据下面提供的信息完美地扮演角色。
  请说话就像{{name}}一样。
  你就是{{name}}。

  {{description}}
```

这样通过简单的设置，我们就让一个脚本能够继承另一个脚本的内容和配置了.

### 智能体自主调用及权限控制

基于自然语言的通用智能体自主调用（无需模型专门训练支持`tools`,唯一要求是遵循指令能力要强）。

#### 使用工具

只要在脚本配置中设定允许智能体使用的工具`tools`， 这些工具`tools`对应 AI 脚本ID，智能体就会在需要的时候自主调用这些工具完成任务。使用工具的配置`tools`支持两种格式: 数组或对象

> 💡 提示: 当前,工具仅提供给本地内置LLM Provider使用。

数组示例配置如下:

```yaml
---
tools:
  - weather
  - search
  - id: now
    # 重载默认的title信息
    title: 以ISO 8601格式获取当前时间
---
user: 现在几点了？今天上海的天气如何？
#智能体会根据配置的tools调用对应的工具: now, weather,并返回结果
assistant: "[[Answer]]"
```

对象示例配置如下:

```yaml
---
tools:
  # 使用对象格式方便重载默认的说明信息
  weather:
    title: get weather information
    defaults: # 重载调用工具参数值
      location: "上海"
      date: today
  search:
    title: search in search engine
  now:
    title: 以ISO 8601格式获取当前时间
---
```

工具高级配置项:

* 工具的动态路由: 当工具名称以`$`开头时，此名称将作为动态路由，运行时将被作为指令被调用，该指令应该返回一个工具列表。
  * 动态路由函数的参数如下:
    * `context`: 当前用户的聊天内容文字版，可用于评估该使用哪些工具
    * `globalDefaults`: 配置该路由特殊的全局默认值，工具函数中已经设置的默认值无需再次配置。
  * 是否需要一个权限控制允许动态路由的执行？
* `trigger`: 工具被触发执行的时机，默认为`AI决定`，支持配置多个触发时机。
  * 选择执行时机：
    * AI决定（AI/Auto）：由 AI 自主判断何时调用该工具。如果没有设置这个就是默认值。
    * 每次用户发言后（AfterUserMessage）：每当用户发送一条消息后自动触发, 会传入用户的消息作为参数。设定了这个选项后，`ai`选项将无意义，因为每次都会触发了。
    * 每次AI回复后（AfterAIResponse）：每当 AI 完成一次回复后自动触发。会传入 AI 的回复作为参数。
    * 程序开始时（AtStartup）：在程序初始化阶段自动触发。会传入程序的启动参数作为参数。
    * 程序结束前（BeforeShutdown）：在程序即将终止前触发。
    * 当包含关键词时（Keyword）：当输入中检测到特定关键词时触发。会传入用户的消息以及匹配的关键词作为参数。
    * 当记忆改变时（MemoryChanged）：会传入新增或修改的记忆对象`memory`，以及操作名称`operation`(新增，修改，删除)作为参数。
      * 新增记忆（MemoryAdded）: 传入参数同上，只不过操作名称固定为`新增`
      * 修改记忆（MemoryUpdated）: 传入参数同上，操作名称固定为`修改`
      * 删除记忆（MemoryRemoved）: 传入参数同上，操作名称固定为`删除`
* 增加工具执行策略配置(RunMode/action_mode)：
  * 并行执行策略： 让LLM尽可能一次性输出多个待执行的工具，让程序可以一次执行多个工具
  * 串行执行策略： 让LLM一次只输出一个待执行的工具
  * 执行规划器：由执行规划决定执行策略,感觉并行、串行也是规划的执行策略中一种，相当于预定义的
    * 级联执行策略: 让LLM 按照任务中对结果的依赖先后顺序，输出待执行的工具
      * 注意，这个依赖关系是由具体任务而定的，不是工具之间的依赖关系。例子，对当前Top10的品牌电视进行评价，那么首先必须要通过search工具获得前10的品牌名称列表，然后再将列表作为输入，传入工具得到各个品牌的电视信息，再分析。
  * AI作答并发与后台同时执行工具（可附加在任意执行策略上）

```json
{
  "run_mode": {
    "type": "string",
    "description": "Defines how the tools are executed.  Choose one: 'parallel', 'serial', 'cascaded'.",
    "enum": ["parallel", "sequential", "cascaded", "planer"],
    "default": "sequential"
  },
  "concurrency_level": {
    "type": "integer",
    "description": "Maximum number of tools to run concurrently when 'action_mode' is 'parallel'.  Set to 1 for single execution.",
    "default": 5,
    "minimum": 1,
    "maximum": 10  // 可以根据系统资源调整上限
  },
  "planner": {
    "type": "string",
    "description": "The planning algorithm determines how the Action Mode is chosen, and whether cascading is needed.",
    "enum": ["default", "advanced"], // 例子，可以扩展更多规划算法
    "default": "default"
  },
  "concurrent_answer": {
    "type": "boolean",
    "description": "Enable concurrent answer generation while tools are running in the background.",
    "default": false
  }
}
```

#### 创建工具

具体Tool脚本示例如下：

```yaml
# weather.ai.yaml
---
# one line description for prompt
title: get weather information
input:
  - location:
      # one line description for prompt
      description: the weather forecast location includes city, province(if any) and country
      example: ChongQing, China
      required: true
  - date:
      description: the weather information of the specified date
      example: 2025-02-04T18:07:42+08:00
      default: today
---
# 模拟返回天气信息
$echo: "上海的天气阴转多云，气温 2°C，相对湿度 60%，风向 东南风，风力 3-4级。"
```

注意：

* 该规范会使用 AI 脚本中的`title`配置作为调用工具的简短说明，`input`配置作为参数说明
  * 成为工具函数的前提是必须要配置`title`或`description`字段。
* 如果用户或者上级脚本禁用了该脚本使用的工具，使用该脚本则会触发异常错误`MethodNotAllowed`: `permission denied`。
* 参数 `example(s)` 配置不仅可以表示单个示例值，还可以是一个示例数组，用于展示多个使用示例。需要注意的是，如果某个参数设置了 default 默认值，那么这个默认值会自动被插入到 examples 列表中，并作为第一个示例项进行展示。
* `hidden` 配置项表示该参数是否被隐藏，默认为 false。如果设置为 true，则该参数将不会显示在工具调用参数列表中。注意，当参数被设置为必填项时，则 hidden 配置项将无效。

#### 工具集配置

你可以将多个工具集中到一个工具集脚本中`myTools.ai.yaml`, 只要配置`export`字段:

```yaml
---
export:
  - weather.ai.yaml
  - now.ai.yaml
  - search.ai.yaml
---
# the following codes will be as the initiaization（$initializeExec） if any
...
```

使用和方式和其他工具一样:

```yaml
---
tools:
  - myTools:
      defaults: # 也可设置默认参数，注意：这些参数会传入工具的参数中以及`$initializeExec`初始化模块函数中
        ...
---
```

**Key Points**:

* **模块化设计**：将相关的工具分组可以提升模块化程度，简化维护工作。
* **初始化行为**: 当导入时，工具集中定义的脚本（例如 `myTools.ai.yaml`）会自动作为 `$initializeExec` 执行，除非显式禁用。
* **默认参数继承机制**: 工具集支持层级化的默认参数解析策略，使大型 AI 项目中的配置更加灵活且易于维护。
  * 优先级: 工具默认参数 > 工具集默认参数 > 工具集脚本对象参数
  * 默认参数采用深度合并（deep merge）策略进行处理。

### 权限控制

权限控制配置用于管理智能体对工具或外部指令的调用权限，用户或上级脚本通过如下配置对智能体使用的工具进行权限控制：

```yaml
---
permissions:
  ai: # 针对 ai 的调用权限（LLM发出的调用）
    call:
      - "!delete_*" # 禁止所有`delete_`打头的工具函数，匹配规则使用glob模式
      - "w*"  # 匹配所有w打头的工具函数
      - read_file: # 只匹配read_file工具函数
          action: allow
          parameters: # 参数匹配
            file_path:
              - "!/dangerous_directory/*" # 禁止访问 dangerous_directory 下的文件
              - "/safe_directory/*" # 只有 safe_directory 下的文件才能访问
  defaultAction: allow|deny|confirm # 如果没有，默认为 confirm
  overwriteAction: allow|deny|confirm # 如果存在强制所有的call(不包括"!"禁止的，"!"始终有效)
---
```

权限控制规则

* 字符串规则为glob规则:
  1. **正匹配**：允许AI调用匹配指定模式的工具脚本。例如：
     - `"weather"`：允许调用`weather`脚本。
     - `"w*"`：允许调用所有以`w`开头的脚本，如`weather`、`wiki`等。
  2. **负匹配**：禁止AI调用指定工具脚本。负匹配规则需以`!`前缀开头。例如：
     - `"!search"`：禁止调用`search`脚本。
  3. **优先级**：负匹配规则优先于正匹配规则。
* 对象规则支持单字段对象和多字段对象:
  1. 单字段对象时，字段名为操作名称，字段值为操作参数。`{read_file: {action: 'allow'}}`
  2. 多字段对象时，必须存在`name`字段，其他字段为操作参数。`{name: 'read_file', action: 'confirm'}`
  3. 对象的操作名称允许正匹配，但不支持负匹配，负匹配由`action: 'deny'`体现。
* 字符串规则承载逻辑复杂性，对象规则承载动作扩展性。
* `overwriteAction` 为了安全，暂时不会实现，等实现非对称加密的签名机制后

注意：这里的权限控制规则是静态的，即权限控制规则在配置文件中定义，并适用于所有用户或上级脚本。

### 动态调整生成概率：`((!text:bias))` 语法

为了更精细地控制生成文本的概率分布，引入了新的提示词语法 `((!text:bias))`，用于动态调整某些 token 的生成概率（`logit-bias`）。该语法可以灵活应用于多种场景，例如引导模型生成特定词汇、避免不希望的内容或调整生成结果的多样性。

#### 语法说明

* 基本格式：`((text:bias))`
  * `text`：需要调整生成概率的目标文本。
  * `bias`：偏置值，可以是以下几种形式：
    * **数值**：直接增加到目标 token 的 logits 上。
    * **百分比**（如 `20%`）：按比例提升或降低生成概率。例如：20% 表示生成概率会按原概率的 1.2 倍计算。
    * `false|never`：完全移除该 token 的生成可能性。
* 可选前缀 `!`：
  * 如果存在前缀 `!`，例如 `((!text:bias))`，则表示从提示词中移除该内容，同时应用指定的偏置值。

#### 使用场景

1. 引导生成内容：
   * 示例：`((happy:20%))` 表示将单词 "happy" 的生成概率提高 20%。
   * 应用：适用于需要引导模型生成特定情感或主题的场景。
2. 避免特定内容：
   * 示例：`((!敏感词:false))` 表示完全禁止生成指定的敏感词。
   * 应用：用于过滤不适当或不符合要求的内容。

#### 示例代码

以下是一个使用 ((!text:bias)) 语法的完整示例：

```yaml
---
description: "You are a helpful assistant that generates text with controlled bias."
---
user: "请生成一段描述快乐场景的文字。"
assistant: "((happy:20%)) 让我们想象一个阳光明媚的日子，孩子们在草地上奔跑，笑声回荡在空气中。"
```

在这个例子中，((happy:20%)) 提高了单词 "happy" 的生成概率，使生成的文本更倾向于表达快乐的情感。

注意事项

* 适度调整：过大的偏置值可能导致生成结果过于偏向某些词汇，影响文本的自然性和多样性。
* 上下文依赖：偏置效果可能因上下文而异，建议根据具体应用场景进行测试和优化。
* 兼容性：该语法目前仅支持在内置本地LLM提供者中使用。

通过引入 `((!text:bias))` 语法，提供了一种微调大语言模型的输出行为的强大机制，满足不同任务和需求的精细化控制。

## 规范

### 智能体脚本包(Package)

智能体脚本包(Package)的包名就是目录名，在包的根目录内必须包含一个与目录同名的脚本文件或`index.ai.yaml`作为package入口脚本。

例如，如果有一个名为 `a-dir` 的package目录，则该目录下的入口脚本应命名为 `a-dir/a-dir.ai.yaml`。

> 💡注意: 如果包的根目录下两个文件同时存在，则与目录同名的脚本文件为入口，而`index.ai.yaml`文件会被忽略。

通过入口文件的 `export` 配置，决定该包导出的函数。

```yaml
---
export:
  - "$hi"
  - "./dobby.ai.yaml"
---
!fn |-
  [js]hi() {console.log('hi')}
```

### Front-Matter 配置规范

使用 [front-matter](https://jekyllrb.com/docs/front-matter/) 进行配置.
`front-matter`必须是文件最前面,第一行以`---`开始, 配置最后以`---`行结束.

配置包括:提示词工程的基础配置,提示词配置,模型参数配置,输入输出以及输入默认值配置
输入输出以及输入默认值配置详见前述.

#### 基础配置

```yaml
---
_id: 不用说了,该脚本的唯一识别标识
type: 脚本类型, `char` 表示脚色类型；`type` 表示该脚本本身就是是一个类型，其_id为类型名。
title: 该脚本的标题(单行)
description: 该脚本的多行说明
templateFormat: "该脚本的模板格式,默认为: `hf`, 也就是huggingface用的jinja2模板格式; `golang` 也是 `ollama` 和 `localai`用的模板类型; `fstring` 也是 `langchain`在用的."
contentType: 忽略,这里都是`script`
modelPattern: 该脚本支持的模型,通过匹配规则
extends: 是扩展自哪个提示词模板
import: # 当导入的声明是函数时会自动给没有前缀的函数名称增加前缀"$"
  - "js package name"
  - "js/script/path.js": ['func1', 'func2'] # 只导入指定的函数
  - 'ruby-funcs.rb'
  - "agent.ai.yaml": "asName" # 导入脚本, 并重命名为$asName
# import: # Object Format
#   "js package name": "*"
#   "js/script/path.js": ['func1', 'func2']
#   "agent.ai.yaml": "asName"
创: 创建者相关信息
签: 该脚本的签名
---
```

##### Import

`import` 配置用于从其他脚本文件中导入函数和声明。

导入单个文件:

```yaml
---
import: "js:js_package_name" # the js npm package name
---
```

导入多个文件**数组格式**:

```yaml
---
import:
  - "js:js_package_name" # the js npm package name
  - "js/script/path.js": ['func1', 'func2', {func3: 'asFunc3'}] # 只导入指定的函数
  - 'ruby-funcs.rb' # ruby file
  - 'rb:ruby_package'
  - "agent.ai.yaml": "asName" # 导入AI脚本函数并重命名为 "$asName"
---
```

**对象格式**:

```yaml
---
import:
  "js:js_package_name": "*"
  "js/script/path.js": ['func1', 'func2']
  "my.ai.wasm": "*"
  "agent.ai.yaml": "asName"
  "ai:package_path#./some.ai.js": "*"
---
```

**注意事项**

* 当前只实现了 `javascript`, `wasm` 以及 `AI`脚本 的支持
* **BROKEN CHANGE**: ~~如果没有提供扩展名，默认为 JavaScript 模块。~~ 通过前缀区分模块类型。js npm 模块必须加上`js:`前缀
* 相对路径基于当前 AI 脚本所在的文件夹，而不是当前工作目录 (CWD)。
* 当导入时，会自动为没有前缀的名称添加 "$" 前缀。
* 导入时没有指定导入的函数名，则默认导入所有函数。eg， `js:path`, 那么所有path函数都会被导入到当前脚本（已经存在同名的函数不会导入）中，可以在脚本中直接通过`$basename(...)`使用。
* 导入时，可用 `*` 表示导入在包名对象上。eg, `"js:path": "*"`, 那么在当前脚本中通过`$path.basename(...)`使用
  * 可以指定特定的名称替换 `*`。eg, `"js:path": "aPath"`, 那么在当前脚本中通过`$aPath.basename(...)`使用
* 如果模块中存在函数`$initializeExec`并且被导入,那么该函数会在模块加载后的初始化执行阶段时自动执行.
  * 包（目录）例外，入口脚本会始终作为`$initializeExec`在初始化执行阶段时自动执行.
    * 禁用必须手动在导入包（目录）时设置`$initializeExec: {disable: true}`。
    * 如果要在包本身中禁用该习惯，那么请在入口脚本中设置`$initializeExec: false`。
    * 另外直接引用包中文件也会跳过`$initializeExec`
      * `ai:package_name#./some.ai.js`
      * 如果希望初始化时候执行包中的入口脚本必须让some.ai.js通过入口脚本导出:
      * `ai:package_name: ['some']`
* 默认导入AI脚本则至少会导入`$[AI_ID](data)`函数对象，该函数用于执行该AI脚本本身，在该函数对象上还有`$[AI_ID].interact({message})`AI交互函数;以及AI脚本中通过`export`配置导出的项目。
* 新增 `ai:` 前缀约定, 表示 AI 脚本包或目录导入。脚本包可以包括AI脚本,`.ai.js`文件以及`.ai.wasm`文件。例如，`ai:package_path#id.ai.yaml`, `ai:package_path#./some.ai.js`
* 模板函数命名约定:以`template$`打头的函数名为jinja2模板函数，其他函数为普通函数。
  * 也可以在导入的时候，通过`type: 'template'`指定某个函数为模板函数。

  ```yaml
  import:
    - "js:js_package_name": [{someFunc: {type: 'template'}}]
  ```

##### Export

如果没有该参数，那么脚本导出的就是自身名称为`$[id]`的指令.

`export`数组约定脚本需要导出的函数。可以导出内部指令，也可以导出外部脚本。

```yaml
---
export:
   # 内部自定义的指令
  - "$internalDirectiveName"
  # 作为 $asName 导出$internalDirectiveName
  - "$internalDirectiveName": "asName"
  # 导出js path模块中的两个函数
  - "js:path": ['basename', 'extname']
  # 导出脚本自身，名称为 `$[id]`
  - "."
---
!fn |-
  [js]function internalDirectiveName() {}
```

注意:

* 当脚本存在`export`后，默认脚本会把自身作为导入时的初始化函数(`$initializeExec`)执行, 除非在`脚本`中存在`$initializeExec`项：
  * 设置 `$initializeExec` 为 `false` 则禁用该行为，不会执行初始化函数；或者设置`$initializeExec`为一个初始化函数。

###### `$initializeExec` 模块初始化方法

执行阶段的初始化方法，在每次运行前期，对于`import`的模块会尝试调用一次该方法，初始化导入的模块。

`$initializeExec`的参数为

* `data`: 传入`调用者`的args数据对象 或`调用者`的默认args data对象
  * `_initCaller`属性: 调用者对象, 因为AI脚本传参是对象参数，第二个`caller`参数无法传入,所以需要该属性。
* `caller`: 调用者对象

注意:

* 如果初始化来自js脚本，那么传入的`this`为caller(保持兼容)
* 如果初始化来自AI脚本，那么传入的`this`为该脚本本身,将脚本视作对象.
  * 脚本配置选项 `runSelfBeforeInitializeExec`，决定是否将AI脚本本身作为初始化过程的一部分先执行，然后再执行`$initializeExec`。默认为 `false`

#### 提示词配置

```yaml
prompt:
  stop_words: ['\n'] # 自定义停止词
  add_generation_prompt: true # 默认为true, 当设置为`true`时, 如果最后一条提示消息不是 `assistant` 角色, 那么会自定增加一条空`assistant`消息,用于确保对话的连贯性.
  messages:          # 也可以在这里配置提示词消息
    - role: system
      content: Carefully Think about the intent of following The CONVERSATION user provided. Output the json object with the Intent Category and Reason.
completion_delimiter: ''  # 可选的参数, 用于在提示词中指示输出结束的标记, 如果使用,该结束标记会自动加入到 stop_words 中 默认无
```

#### 模型参数配置

```yaml
parameters:
  stop_words: ['\n'] # 自定义停止词也可以定义在参数里.
  max_tokens: 512  # 别太大,也别太小, 建议512, 默认2048, 用处是当模型响应无限不能停止时,这个就能控制大模型最多返回的token长度.
  continueOnLengthLimit: true
  maxRetry: 7    # 当大模型的响应不全时, 因受到max_tokens的限制,这个是自动再次执行LLM的次数,默认7次.
  stream: true # 就是默认启用大模型流式响应,高于llmStream优先级.
  timeout: 30000 # 设置响应30秒(单位是ms)超时,不设置就默认为120秒.
  response_format:
    type: json_object
    conversionScope: 'message' # 默认为'message' 则表示参与转换的json内容为整个助手消息内容; 'response'，表示参与转换json内容仅为AI响应内容。
  minTailRepeatCount: 7 # 最少尾部重复次数, 默认为7, For stream mode only, 当检测到大模型响应返回的尾部序列连续重复4次,就停止响应. 设置为0则不检测.
  SystemTemplate: 'qwen2.5' # 如果不设置默认会自动检测使用模型对应的系统模板，设置为字符串后强制使用该模型的系统模板. 也可通过对象直接指定系统模板: `id`为模板id；`version`可选，为该模板的功能版本。
llmStream: true # 默认true, 启用大模型流式响应,注意,可能有的后端并不支持流式响应.
autoRunLLMIfPromptAvailable: true # 默认true, 表示当脚本中存在提示消息并且一直到脚本结束也没有执行调用过`$AI`,那么脚本会自动在结束时执行`$AI`
forceJson: null # 默认为null,表示是否强制输出json对象, 由`response_format.type`和`output`自动决定: 当它们两个配置同时存在的时候就强制输出为json.
shouldAppendResponse: null # 默认为null, 表示大模型返回结果是否应该通过新增一个助手角色提示消息,还是在加在最后一个消息上.
                           # 如果不设置,则由引擎自动判断是否新增消息
disableLlmRequest: false  # 默认 false, 是否禁用`llmRequest`事件
```

其它常见模型参数如下:

* `temperature` 是一个介于0和正无穷之间的浮点数，用来调节采样概率分布的平滑程度。在语言模型的上下文中，它影响着下一个词汇的选择过程。
  * 低温度 (接近0)：模型生成的文本会更加保守、可预测。这时，模型倾向于选择最高概率的词汇，生成的文本会更加流畅、常规，但可能会缺乏创造性或多样性。
  * 高温度：增加`temperature`值会使模型更加倾向于探索那些概率较低的词汇，生成的文本会更加多样、新颖，但也可能更加离散、难以理解，甚至出现语义上的跳跃。
* `continueOnLengthLimit`: 这个的作用是,当到达最大token限制后,是否会自动继续调用ai,继续取数据
  * 注意,这个目前不适用于当返回结果为json的情况,如果要求返回json必须一次取回,改大 `max_tokens`
* `maxRetry`: 与`continueOnLengthLimit`配套的还有这个参数,继续重试的最大次数.如果不设置,默认是7次
* `timeout`: 如果脑子比较大,响应比较慢,超过2分钟都没有响应完,那么就需要调整这个超时参数,单位是毫秒
* `max_tokens`: 这个就是最大token限制,默认是2048,ai会输出直到max_tokens停止,这会避免有时候ai无限输出停不下来.
* `response_format`: 设定返回结果的格式,目前`type`只有json(别名`json_object`)可以设置.
  * 注意: 当`output`和`type:json`同时被设置的时候,就会强制模型返回json object, 而非文本.
  * 如果没有设置`response_format`可以在调用参数中设置`forceJson:true`也是同样的效果.

**注意**:

* 参数的优先级从高到低是: 调用参数, `prompt`对象, `parameters`对象.
* 当启用大模型流式响应后,你可以通过事件`llmStream`接收部分结果.
  * `llmStream`事件处理器的参数是`(event, part: AIResult, content: string)`, `part`是当前大模型返回的响应对象, `content`是当前大模型返回的响应中的内容的累积.

```yaml
$on:
  event: llmStream
  callback: !fn |- # 注意：使用匿名函数监听时, 无法取消事件监听
    (event, part, content) { const current_text = part.content }
```

##### 思维模式配置(shouldThink)

`shouldThink` 决定是否启用思维模式, 以及配置思维模式的类型。

* **类型**: `boolean` 或 模式名称 或 模式配置对象
* **描述**: 决定是否启用思维模式, 以及配置思维模式的类型。
  * `true`: 启用默认的思维模式: `last`。
  * 支持的思维模式如下:
    * `off`: 禁用思维模式。
    * `last`: 先回答再思考。
    * `first`: 先思考再回答。
    * `deep`: 深度思考后再回答。
  * 当作为`模式配置对象`，有如下配置:
    * `mode`: 思维模式的类型。
    * `visible`: 是否展示思维过程(可选)
    * `depth`: 最多思考深度(可选)
    * `steps`: 最多思考步骤(可选)
    * `thinkTag: string|RegExp|[string, string]`: 思考内容的标签。用于标记思考过程中的文本。(可选)
      * 仅在`first`,`deep`模式下有效。
      * 如果是数组，则表示开始和结束的标签。
      * 如果没有配置，则默认为标签为“Think”
      * RegExp: `/<\|channel\|>analysis<\|message\|>(?<think>.*?)<\|end\|>/s`
        * `think` 命名捕获组为思考内容，如果没有则寻找第一个捕获组，否则就是全部匹配内容。
    * `answerTag: string|RegExp|[string, string]`: 回答内容的标签。用于标记最终的回答文本。(可选)
      * 仅在`first`,`deep`模式下有效。
      * 如果是数组，则表示开始和结束的标签。
      * 如果没有配置，则默认为标签为“Answer”
      * RegExp: `/<\|channel\|>answer<\|message\|>(?<answer>.*?)<\|end\|>/s`
        * `answer` 命名捕获组为回答内容，如果没有则寻找第一个捕获组，否则就是全部匹配内容。
    * `skipDefaultPrompt`: 是否强制跳过默认的思考提示词。(可选)
    * `defaultThinkPrompt`: 使用自己的思考提示词替换（当skipDefaultPrompt时无效）。(可选)
      * `prompt`: 思考提示词对象
        * `lastThinkPrompt`: 先回答再思考提示词
        * `firstThinkPrompt`: 先思考再回答提示词
        * `deepThinkPrompt`: 深度思考提示词
        * `rolePrompt`: 角色提示（可选）
      * `templateFormat`: 提示词格式（可选）
* **目的**:
  * 控制AI在处理请求时的思考行为，以适应不同的应用场景和需求。
  * 提供灵活的配置选项，使得开发者可以根据具体需求调整AI的行为。
* **行为**:
  * 当 `shouldThink` 设置为 `true` 或者模式名称时，AI将根据指定的模式执行相应的思考逻辑。
  * 如果提供了模式配置对象，则会根据配置对象中的参数进一步细化思考行为。
  * 如果 `shouldThink` 设置为 `false` 或 `off`，则AI不会执行任何额外的思考逻辑，直接返回结果。

### 字符串前缀约定

* `~` 前缀: 表示不对接着的字符串进行格式化,原样返回, eg, "`~{{description}}`"
* `!` 前缀: 表示立即进行 format string, eg, "`!{{description}}`"。
  * 如果是函数返回值消息是字符串,并且作为消息的第一个字符为"!" 表示立即格式化消息
* `?=` 前缀: 表示表达式
  * 如果表达式结果是字符串,并且以"#"开头,则表示立即格式化表达式结果
* `#:` or `#+`
  * 当本身是消息的话, 前缀`#`可以省略
  * `:[-1:role]Message`: 替换消息,方括号中可以指定消息的索引,默认为最后一条消息,如果为0则替换第一条消息,如果为负数则从倒数第几条开始替换,如`[-1]`则替换最后一条消息
    * role角色参数可以省略, 省略为保持角色不变, `#:[-1]Message`
    * 方括号和数字可以省略,如`#:Message`,省略后为替换最后一条消息
    * 如果是`#:!Message` 表示立即格式化消息
  * `+[-1:role]Message`: 在指定位置新增消息,位置如果为负数则从倒数第几条开始插入, 位置可以省略,省略后为在最后处新增消息, 方括号内为消息角色,可以设置为`system`,`assistant`,默认为`user`,可以省略为: `!+Message`
    * 如果是`#+!Message` 表示立即格式化消息
* 如果字符串中没有上述前缀，或者出现格式问题，就视作用户角色的新增消息。

#### 表达式

`?=<expression>`

```yaml
$echo: ?=23+5
```

### URI 协议约定

#### Pack 协议

用于表示文档型NoSQL知识库资源：`pack://id[?tbl=collection&db=dbName]`

参数:

* `id`: 代表该知识的是唯一标识符;
* `tbl`: 指定了数据集(collection)的名称。如果未指定该参数，则默认使用当前脚本所在的数据集
* `db`: 该参数定义了数据库(database)的名称。如果没有明确给出，则会使用当前脚本所在的数据库名称作为默认值。

最简单的形式只包含`id`参数：`pack://my_knowledge_id` 表示访问当前AI脚本的知识库中的`my_knowledge_id`资源。

## Refs

* [AutoGen](https://github.com/microsoft/autogen)
* [LMQL](https://lmql.ai/)
* [OpenInterpreter](https://github.com/OpenInterpreter/open-interpreter)
* [Outlines](https://github.com/outlines-dev/outlines)
* [MemGPT](https://github.com/cpacker/MemGPT/)
* [LangChain](https://github.com/langchain-ai/langchain)
