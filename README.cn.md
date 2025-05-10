# Programmable Prompt Engine Specification(Draft)

> 【[English](./README.md)|中文】
---

可编程提示词工程语言是一种简单且自然的AI语言，专门用于处理提示词信息。这种语言用于开发各种智能体，这些智能体可以被重用、继承、组合或调用。该语言也可以用于简化大型语言模型 (LLM) 的提示词创建与重用管理工作流程，使这一过程更加高效且易于理解。[本规范](https://github.com/offline-ai/ppe)在 [offline-ai/cli](https://github.com/offline-ai/cli) 项目中实现。

可编程提示词语言是

## 特色功能

* 构建一个像软件工程那样可重复利用、能编程的提示系统，专为大型语言模型（LLM）设计，这样做的目的是让工作更高效、易于理解
* 简化了提示词管理
* 保持通用性和兼容性：按照功能组织调用提示词工程脚本库，追求独立于特定大模型的通用性，让脚本能灵活应用于各种不同的大模型
* 让用户轻松上手：应用程序开发者可以直接使用提示词工程，就像使用任何常规代码库一样，无需深入了解复杂的AI内部机制，降低了使用门槛
* AI 语言分层结构: 清晰划分并自定义提示词类型
  * 函数: `lib`类型, 每个AI文件可以定义为一个函数，供其他提示词或代码调用，例如，文本文件读入`file()`，fetch url `url()` 都是函数提示词
    * 引伸出在消息中 `@某个提示词`, 用于调用特定输入输出约定的提示词函数，如: `@file(...)`, `@url(https://...)`
  * 类: 每个AI文件可以被封装为一个可继承的类，覆盖配置和代码的继承
    * type: `type`类型, 用于自定义类型的提示词脚本
    * 你也可以用提示词自定义其它类型
    * char: 脚色类型, 具有特定角色定位的提示词脚本,"脚色类型"自身也是一个提示词脚本
  * 应用: 由目录下的若干AI文件组成，主入口AI文件的`basename`与目录名相同， 例子，`guide`目录的主入口AI文件名为`guide.ai.yaml`
* 脚本调用AI是第一位,而AI调用脚本则完全在您的控制下
* 在同一脚本中可以同时实现API调用(通过`JSON Schema`)功能和多轮交互对话功能

提示词工程师的新使命：未来，提示词工程师的工作重心将转向开发更多能广泛兼容大模型的通用型提示词脚本，推动技术普及和应用创新。

## Quick Start

### 选择基于YAML的理由

1. 接近自然语言，便于AI理解
   * YAML的语法设计非常贴近自然语言表达，减少了AI在解析复杂格式时的认知负担。
   * AI模型能够更直观地理解提示词内容，避免因格式复杂而引入歧义或误解。
2. 清晰的语义结构，提升AI处理效率
   * YAML通过缩进和层级结构，明确表达了数据之间的关系，帮助AI快速定位关键信息。
   * 这种结构化的表达方式让AI在生成响应或执行任务时更加精准，减少了不必要的上下文混淆。
3. 人机协作的最佳桥梁
   * 对人类来说，YAML易于阅读和编写；对AI来说，它的语义清晰、逻辑明确，是双方都能高效理解的中间格式。
   * 在提示词工程中，这种“双向友好”的特性尤为重要，能够显著提升人机协作的流畅性。
4. 灵活性和扩展性
   * YAML 可以自定义 tag
   * YAML 可以根据需要自由调整内容，适应不同场景下的需求。
5. YAML被广泛使用和成熟生态
   * 基于YAML可以直接复用现有工具链，无需额外开发解析逻辑。
6. 降低开发门槛，专注AI核心功能
   * 使用YAML避免了从头设计一套新语言的复杂性，开发者可以专注于优化AI语言逻辑，而非纠结于格式问题。
   * AI也能够直接利用YAML的结构化特性，快速提取关键信息并生成高质量的响应。

### 结构化对话消息

在使用 YAML 格式来表示对话消息时，每一行代表了一次对话中的交流。对话可以通过“角色: 消息”的形式指定说话人，其中角色可以是系统（system）、助手（assistant）或用户（user）。如果省略了角色，默认就是用户在说话。

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

角色群聊功能以结构化自然语言的方式进一步完善了PPE的对话系统，同时使得多个智能体之间能够方便地进行协作和交流，从而更高效地完成复杂的任务。

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
    translator: char_translator
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
* `type`: 输入参数的数据类型(可选项)，默认为`string`

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

如果没有带参数,那么会把AI结果作为~~`result`~~`content`参数传给智能体,eg,

`list-expression.ai.yaml`:

```yaml
system: "只列出计算表达式, 不要计算结果"
---
user: "三块糖加上5块糖"
assistant: "[[CalcExpression]]"
-> calculator  # 传入智能体的实际输入参数就是上一次的结果: {content: "[AI生成的计算表达式]"}
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

**注**: 如果脚本返回值是`string`/`boolean`/`number`,那么都会将该返回值放到`content`字段;如果返回值是`object`,则会直接将对象里的内容传递给智能体.

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

#### 工具配置

只要在脚本配置中设定允许智能体使用的工具`tools`， 这些工具`tools`对应 AI 脚本ID，智能体就会在需要的时候自主调用这些工具完成任务。工具配置`tools`支持两种格式: 数组或对象

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
  search:
    title: search in search engine
  now:
    title: 以ISO 8601格式获取当前时间
---
```

具体Tool脚本示例如下：

```yaml
# weather.ai.yaml
---
# one line description for prompt
title: get weather information
input:
  - location:
      # one line description for prompt
      description: contain city, province(if any) and country
      example: city,province,country
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
* 如果用户或者上级脚本禁用了该脚本使用的工具，使用该脚本则会触发异常错误`MethodNotAllowed`: `permission denied`。

#### 权限控制

权限控制配置用于管理智能体对工具的调用权限，用户或上级脚本通过如下配置对智能体使用的工具进行权限控制：

```yaml
---
permissions:
  ai:
    call:
      - "w*"
      - "now"
---
```

权限控制规则

1. **正匹配**：允许AI调用匹配指定模式的工具脚本。例如：
   - `"weather"`：允许调用`weather`脚本。
   - `"w*"`：允许调用所有以`w`开头的脚本，如`weather`、`wiki`等。
2. **负匹配**：禁止AI调用指定工具脚本。负匹配规则需以`!`前缀开头。例如：
   - `"!search"`：禁止调用`search`脚本。
3. **优先级**：负匹配规则优先于正匹配规则。

### 动态调整生成概率：`((!text:bias))` 语法

为了更精细地控制生成文本的概率分布，引入了新的提示词语法 `((!text:bias))`，用于动态调整某些 token 的生成概率（`logit-bias`）。该语法可以灵活应用于多种场景，例如引导模型生成特定词汇、避免不希望的内容或调整生成结果的多样性。

#### 语法说明

* 基本格式：`((text:bias))`
  * `text`：需要调整生成概率的目标文本。
  * `bias`：偏置值，可以是以下几种形式：
    * 数值：直接增加到目标 token 的 logits 上。
    * 百分比（如 `20%`）：按比例提升或降低生成概率。例如：20% 表示生成概率会按原概率的 1.2 倍计算。
    * `false`：完全移除该 token 的生成可能性。
* 可选前缀 `!`：
  * 如果存在前缀 `!`，例如 `((!text:bias))`，则表示从提示词中移除该内容，同时应用指定的偏置值。

#### 使用场景

1. 引导生成内容：
   * 示例：`((happy:20%))` 表示将单词 "happy" 的生成概率提高 20%。
   * 应用：适用于需要引导模型生成特定情感或主题的场景。
2. 避免特定内容：
   * 示例：`((!敏感词:false))` 表示完全禁止生成指定的敏感词。
   * 应用：用于过滤不适当或不符合要求的内容。
3. 动态调整生成多样性：
   * 示例：`((creative:50%))` 表示将单词 "creative" 的生成概率提高 50%，从而鼓励模型生成更具创造性的内容。
   * 应用：适合需要增强生成文本多样性的任务。

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
* 兼容性：该语法目前仅支持在内置LLM提供者中使用。

通过引入 `((!text:bias))` 语法，提供了一种微调大语言模型的输出行为的强大机制，满足不同任务和需求的精细化控制。

## 规范

### 智能体脚本包(Package)

智能体脚本包(Package)的包名就是目录名，在包的根目录内必须包含一个与目录同名的脚本文件作为package入口脚本。

例如，如果有一个名为 `a-dir` 的package目录，则该目录下的入口脚本应命名为 `a-dir/a-dir.ai.yaml`。

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
* 如果模块中存在函数`$initializeModule`并且被导入,那么该函数会在模块加载后自动执行.
  * 包（目录）例外，入口脚本会始终作为`$initializeModule`在模块加载后自动执行.
    * 禁用必须手动在导入包（目录）时设置`$initializeModule: {disable: true}`。
    * 如果要在包本身中禁用该习惯，那么请在入口脚本中设置`$initializeModule: false`。
    * 另外直接引用包中文件也会跳过`$initializeModule`
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

* 当脚本存在`export`后，默认脚本会把自身作为导入时的初始化函数(`$initializeModule`)执行, 除非在`脚本`中存在`$initializeModule`项：
  * 设置 `$initializeModule` 为 `false` 则禁用该行为，不会执行初始化函数；或者设置`$initializeModule`为一个初始化函数。

###### `$initializeModule` 模块初始化方法

传入`$initializeModule`的`this`为调用者的对象,

`$initializeModule`的参数为

* `data`: 传入`调用者`的args数据对象 或`调用者`的默认args data对象
* `caller`: 调用者对象

注意:

* 如果初始化来自js脚本，那么传入的`this`为caller(保持兼容)
* 如果初始化来自AI脚本，那么传入的`this`为该脚本本身,将脚本视作对象.
  * 脚本配置选项 `runSelfBeforeInitModule`，决定是否将AI脚本本身作为初始化过程的一部分先执行，然后再执行`$initializeModule`。默认为 `false`

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
    * `thinkTag: string | [string, string]`: 思考内容的标签。用于标记思考过程中的文本。(可选)
      * 仅在`first`,`deep`模式下有效。
      * 如果是数组，则表示开始和结束的标签。
      * 如果没有配置，则默认为标签为“Think”
    * `answerTag: string | [string, string]`: 回答内容的标签。用于标记最终的回答文本。(可选)
      * 仅在`first`,`deep`模式下有效。
      * 如果是数组，则表示开始和结束的标签。
      * 如果没有配置，则默认为标签为“Answer”
    * `thinkRegex`: 用于分析AI的返回内容的正则表达式，仅限thinkTag为字符串时有效。(可选)
    * `answerRegex`: 用于分析AI的返回内容的正则表达式，仅限answerTag为字符串时有效。(可选)
* **目的**:
  * 控制AI在处理请求时的思考行为，以适应不同的应用场景和需求。
  * 提供灵活的配置选项，使得开发者可以根据具体需求调整AI的行为。
* **行为**:
  * 当 `shouldThink` 设置为 `true` 或者模式名称时，AI将根据指定的模式执行相应的思考逻辑。
  * 如果提供了模式配置对象，则会根据配置对象中的参数进一步细化思考行为。
  * 如果 `shouldThink` 设置为 `false` 或 `off`，则AI不会执行任何额外的思考逻辑，直接返回结果。

### 字符串前缀约定

* `~` 前缀: 表示不对接着的字符串进行格式化,原样返回, eg, "`~{{description}}`"
* `!` 前缀: 表示立即进行 format string, eg, "`!{{description}}`"  以前的 ~~`#`~~ 前缀从版本`v0.9.0`开始被废弃。
* ~~`$` 前缀: 无参数指令调用, eg, "`$AI`"~~ deprecated
* ~~`$#`前缀: 把无参数指令的返回值作为消息~~
  * 如果是函数返回值消息是字符串,并且作为消息的第一个字符为"!" 表示立即格式化消息
* `?=` 前缀: 表示表达式
  * 如果表达式结果是字符串,并且以"#"开头,则表示立即格式化表达式结果
* `#:` or `#+` 以前的 ~~`!:` or `!+`~~ 前缀从版本`v0.9.0`开始被废弃。
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

### 高级指令约定

#### `$prompt`设置提示参数指令

使用`$prompt`定义提示参数，供提示词模板使用.

```yaml
- $prompt:
  add_generation_prompt: true # 默认为true
```

* `add_generation_prompt`: 当设置为`true`时, 如果最后一条提示消息不是 `assistant` 角色, 那么会自定增加一条空`assistant`消息,用于确保对话的连贯性.

#### `$parameters`设置模型参数指令

用`$parameters`设置模型参数或在`FRONT-MATTER`中定义.

```yaml
---
parameters:
  max_tokens: 512
  temperature: 0.01
---
- $parameters:
  max_tokens: 512
  temperature: 0.01
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

#### `$tool`工具指令

用`$tool`指令使用注册的所有工具。

##### `$AI` 指令

`$AI` 是`$tool:llm`别名,直接调用大模型工具. 默认是将结果作为`assistant`角色消息追加到`prompt.messages`中,可以通过设置`shouldAppendResponse:false`关闭追加.

```yaml
$AI:
  max_tokens: 512
  temperature: 0.7
  stream: true      # 默认为真,也可在配置中设置 llmStream, 流式响应
  pushMessage: true # 默认为true,表示将大模型工具返回的结果追加到prompt.messages中.
  shouldAppendResponse: null  # 仅当 pushMessage 为true时候有效, 默认为 undefined.
                              # 当 undefined/null时, 当 `matchedResponse` 或 `add_generation_prompt` 或没有 lastMsg.content 会追加,否则会替换最后一条消息的正文
                              # 当为true时,强制追加一条助手消息,false时,为强制替换最后一条消息的正文.
  aborter: ?= new AbortController() # 如果没有设置,则使用引擎系统的AbortController.
$tool:
  name: llm # 等于$AI
  ...       # 其它named参数
```

#### `$abort` 指令

手动停止大模型的响应,这会产生一个abort异常.

```yaml
$AI
$abort
```

#### 管道指令

`$pipe`将上一次指令的结果传递给下一个指令, 支持缩写`$|func`

```yaml
- toolId: $tool
# 上一个函数的返回结果传递到`func1|print`, 如果pipe没有参数,就传递给下一个数组元素.如果下一个元素本身是对象,就合并.
- |
- $func1
- $pipe
- $print
```

```yaml
- llm: $tool
- $|func1
- $|print
```

#### `!fn` 定义函数指令

使用`!fn` tag 定义函数

```yaml
!fn |-
  function func1 ({arg1, arg2}) {
  }
# function 关键字 可以省略:
!fn |-
  func1 ({arg1, arg2}) {
  }
```

函数体是默认是js. 定义函数中可以使用 `async require(moduleFilename)` 加载本地 esm 格式的js文件.

```yaml
!fn |-
  async myTool ({arg1, arg2}) {
    const tool = await require(__dirname + '/myTool.js')
    return tool.myTool({arg1, arg2})
  }
```

如果要使用其他语言，需要指定语言:

```yaml
!fn |-
  [python] def func1(arg1, arg2):
    return arg1 + arg2
```

**注意**:

* `__dirname`: 为提示词脚本文件所在目录.
* `__filename`: 为提示词脚本文件路径.
* 在函数中,可以使用`this`来获取当前脚本的runtime的所有方法.
* 所有的自定义函数都必须通过`$`打头引用. 如,在上面例子定义了`func1`,那么调用的时候必须用`$func1`:

  ```yaml
  $func1:
    arg1: 1
    arg2: 2
  ```

* 目前只支持js，准备添加python,ruby等支持.

#### `!fn#` 定义模板函数指令

`!fn#` 将定义在Jinja模板中使用的函数,其他同`!fn`.

```yaml
---
content:
  a: 1
  b: 2
---
!fn# |-
  function toString(value) {
    return JSON.stringify(value)
  }
$format: "{{toString(content)}}"
```

#### `$exec` 调用外部脚本指令

透过`$exec`指令可以与其它智能体脚本进行交互.

```yaml
$AI
$exec:
  # id: 'script id'   #  脚本文件名与id只能选一个
  filename: json
  args: "?=LatestResult"  # 将$AI的结果通过参数传给 json 智能体脚本.
```

注意：默认会对外部脚本的输出结果进行缓存，可以通过设置参数中的`memoized`为`false`来关闭缓存。

#### `$if` 指令

`$if` 函数支持条件判断

```yaml
$set:
  a: 1
- $if: "a == 1" # 表达式判断
  then: # then 函数
    $echo: Ok
  else: # "else 函数"
    $echo: Not OK

!fn |-
  isOk(ok) {return ok}
- $if:
    $isOK: true # 函数判断
  then: # then 函数
    $echo: Ok
  else: # "else 函数"
    $echo: Not OK
```

#### `$match` 指令

`$match` 指令用于根据变量或上一次操作的结果进行多分支匹配。支持多种匹配方式，包括正则匹配、键值匹配、完整匹配、表达式匹配、范围匹配、忽略匹配、对象匹配等。

匹配项前必须有冒号`:`，冒号后面是匹配项，中间不能有空格。
condition 将 作为`COND__` 传入执行部分.

```yaml
# condition 可选, 如果没有则是以LastResult为条件,
# $match默认为顺序执行一旦找到一个匹配的模式，就会执行对应的分支，并且不会继续检查后续的模式。
# 如果设置参数allMatches为 true，那么就会执行所有匹配的分支项，默认为false。
# 如果设置参数 parallel 为 true，那么就会并行执行所有匹配的分支项，仅当allMatches为真时才有意义。
$match(condition[, allMatches=false]):
  # 正则匹配
  :/RegEx/:
    - $echo: matched
  # 可以是条件比较
  :> 12:
    - $echo: matched
  # 完整匹配,如果condition是字符串 or 数字
  :"string": # :123
    - $echo: matched
  # 表达式匹配, condition === 1 or condition == 2
  :1 || 2:
    - $echo: matched
  # 范围匹配,  1..5 表示`[1..5]`全闭区间, `1..<5` 表示`[1,5)` 从1到小于5的半闭区间, 1>..5 表示`(1, 5]` 大于1到5的半闭区间。
  :1..5:
    - $echo: matched
  # 忽略特定项匹配,这个只匹配数组的第一项和第四项,也就是必须存在第一项和第四项并且数组长度是4,并且将第一项的值赋予first,第四项的值赋予last
  ":['a,b', _, _, last]":
    - $echo: matched
  # 表示匹配完整对象
  ":{x='a', y=':1||2' }":
    - $echo: matched
  # 表示部分匹配
  ":{x='a', ..}":
    - $echo: matched
  # 否则
  _ :
    - $echo: else matched
```

* `condition`: 可选项, 如果没有则是以LastResult为条件。
* `allMatches`: 启用则执行全部匹配，也就是会执行所有匹配的分支项，默认为`false`。
* `parallel`: 是否并发执行所有匹配的分支项，仅当启用allMatches时才有意义，默认为`false`。

#### `$while` 指令

`$while` 指令用于执行一段代码块，只要给定的条件为真就会一直循环执行。下面是一个简单的示例：

```yaml
- $set:
    i: 5
- $while: "i >= 0"
  do:
    - $set:
        i: ?=i-1
    - $if: "i == 2"
      then: $break
```

说明

* 条件表达式 (`"i >= 0"`): 这是循环的条件。只有当这个条件为真的时候，循环体内的代码才会被执行。
* 循环体 (`do:`): 这部分包含了在每次循环时需要执行的操作。
* `$break` 指令用于在循环体中提前结束循环。
* `$continue` 指令用于在循环体中跳过当前循环，直接进入下一次循环。

示例解析

在这个例子中，`$while` 指令会检查变量 `i` 是否大于或等于 0。如果条件满足，则执行循环体中的操作：将 `i` 的值减 1。这个过程会一直重复直到 `i` 不再大于或等于 0 为止。

注意事项

* 确保循环条件最终会被改变，否则会导致无限循环。
* 循环体内可以包含多个操作步骤，不仅仅限于单个 `$set` 操作。

通过 `$while` 指令，你可以实现基本的循环逻辑，适用于多种场景下的迭代处理。

#### `$for` 指令

`$for` 指令用于迭代一个列表，并执行一段代码块。下面是一个简单的示例：

```yaml
$for: 3 # 遍历从 1 到 3
  as:
    value: item
  do:
    - $print("The current item is:{{item}}")
```

```yaml
$for: 1.5..5.5 # 遍历从 1.5 到 5.5,步长为0.5
  step: 0.5 # 默认步长为1
  as:
    value: item
  do:
    - $print("The current item is:{{item}}")
```

```yaml
$for: "[1, 2, 3, 4, 5]"
  as:
    value: item
  do:
    - $print("The current item is:{{item}}")
```

```yaml
$for: "{a:1, b:2}"
  as:
    index: k
    value: v
  do:
    - $print("The current item is:{{k}}={{v}}")
```

* `as` 可省略，如果省略, 将默认定义为: `value` 会被赋予当前循环的元素, `index` 会被赋予当前循环的索引。`items` 为遍历的对象，如果是数值范围则为`{start, end, step}`。
* 循环体 (`do:`): 这部分包含了在每次循环时需要执行的操作。
* `$break` 指令用于在循环体中提前结束循环。
* `$continue` 指令用于在循环体中跳过当前循环，直接进入下一次循环。

#### `$format` 指令

`$format` 函数使用Jinja2模板格式化字符串,消息的格式化也是使用的Jinja2模板,这也是HuggingFace大模型支持的模板格式.

```yaml
$format: "{{description}}"
$format:
  template: "{{description}}"
  data:
    description: "hello world"
  templateFormat: "hf"  # 默认为 hf, 目前支持 hf 也就是huggingface用的jinja2; `golang` 也是 `ollama` 和 `localai`用的模板类型; `fstring` 也是 `langchain`在用的.
```

#### `$set/$get` 变量操作指令

支持 key path.

```yaml
$set:
  testVar.a: 124
  var2: !fn (key) { return key + ' hi' }
$get:
  - testVar.a
  - var2
```

#### `$ctx` 获得调用方为我分配的变量操作指令

用于统一访问和操作脚本或智能体中调用方为我分配的**变量（variables）**存储对象。这种设计为主脚本提供的一种机制，用于为每个外部脚本分配一个独立的状态存储空间，帮助外部脚本在多次调用之间保持状态，特别是外部脚本或智能体需要在主脚本（调用方）保持自己的上下文状态时。

参数:

* `key`: 外部变量的键，当没有时，返回指定`id`下的所有外部变量
* `value`: 外部变量的值，当`value`没有定义时候，返回`key`定义的外部变量，否则设置为`key`的值
* `id`: 脚本ID，默认为当前脚本ID

该指令提供了一种统一的方式，来获取或设置 extVars 中的数据：

```yaml
$ctx # 获取调用方上所有外部变量

$ctx: # 获取调用方上my_script的外部变量
  id: "my_script"

$ctx: # 获取调用方上的外部变量my_var
  key: "my_var"

$ctx: # 设置调用方上的外部变量my_var的值为new value
  key: "my_var"
  value: "new value"
```

🧠 设计理念

`$ctx` 的设计灵感来自于“沙盒式变量空间”模式，外部脚本可以通过它，在多次调用之间保持自己的状态，而无需直接操作全局变量或自身的成员变量（因为它们可能被重复调用或嵌套调用）.

* 让外部脚本能够**安全地保存自己的上下文状态**
* **避免**外部脚本**污染全局环境或修改主脚本状态**
* 实现**多外部脚本之间的状态隔离**
* 支持"长期"运行脚本的**状态持久化**

📌 实际应用场景

场景一： 根据调用方上下文调整脚本的行为状态

外部脚本 validator.ai.yaml 需要在特定条件下跳过验证：

```yaml
$set:
  skipValidation: "?= this.$ctx({key: 'skipValidation'})"
$if: "{{skipValidation}}"
  then:
    $ret: "Validation skipped."
  else:
    # 正常执行验证逻辑...
```

### 事件约定

高度可编程和事件驱动的提示生成系统，它允许用户通过定义事件监听器、触发器以及相应的回调函数，动态地控制和定制化生成文本的过程。从给出的示例中，我们可以看到几个关键特点和优势：

* 事件驱动架构：通过提供`$on`、`$once`、`$emit`和`$off`这样的函数，系统支持基于事件的编程模型，使得开发者能够响应不同的生命周期阶段或特定条件，灵活地干预和扩展模型的行为。
* 灵活性与可扩展性：用户不仅可以注册命名函数作为回调，还能使用匿名函数或表达式，提供了多样化的编程接口，适应不同的使用场景和复杂度需求。这增强了脚本的灵活性和可扩展性。
* 细致的事件类型设计：从`beforeCall`、`afterCall`到针对大模型交互的`llm`、`llmStream`等事件，系统覆盖了从函数调用、结果处理到模型交互的各个环节，充分体现了对大模型应用中常见需求的深入理解和支持。
* 集成与交互优化：通过`llmRequest`事件，系统能够智能化地管理大模型调用，支持通过事件机制来定制获取模型响应的方式，同时提供了禁用选项以适应不同策略。此外，对聊天记录的加载和保存也通过事件开放，方便集成到外部系统或进行数据管理。
* 清晰的API设计：示例文档展示了清晰的API使用方法和参数说明，便于开发者快速上手并深入应用，体现了良好的设计哲学和用户体验考量。

`$on` 函数支持事件监听, `$once`函数支持事件监听一次, `$emit`函数支持触发事件
`$off`函数支持取消事件监听

#### `$on` 和 `$once` 事件监听函数

参数如下:

- event: 事件名称
- callback: 回调函数或表达式

函数作为回调函数:

```yaml
!fn |-
  onTest (event, arg1) { return {...arg1, event: event.type}}
$on:
  event: test
  callback: onTest # 命名函数监听,可以取消事件监听
$once:             # 触发一次后自动取消事件监听
  event: test
  callback: !fn |- # 匿名函数监听, 不能取消事件监听
    (event, arg1) { return {...arg1, event: event.type}}
$emit:             # 触发事件
  event: test
  args:
    a: 1
    b: 2
$off:
  event: test
  callback: onTest
```

表达式作为回调函数, 表达式中的参数如下:

- `event`: 事件对象实例
  - event.type: 事件名称
  - event.target: 事件源,也就是当前脚本runtime
  - event.stopped: 设置为`true`会阻止后续回调函数执行。
  - event.result: 上一个回调函数设置的返回值
- `arg1`: 事件监听函数传入的第一个参数值
- `args`: 事件监听函数传入的剩下参数值列表,如果有的话

这些参数等同于回调函数中的: `(event, arg1, ...args) => void|any`

```yaml
$on:
  event: test
  callback: "?={...arg1, event: event.type}" # 无法取消事件监听
```

注意:

* 如果回调函数有返回值，那么在其他回调函数中可以通过参数`event.result`看到，以最后一个回调函数的返回值作为最终结果返回给`$emit`事件触发函数。
* 设置参数`event.stopped`为`true`会阻止后续回调函数执行。

#### `$emit` 触发事件函数

参数如下:

- event: 事件类型,字符串,如: `test`
- args: 事件监听函数传入的参数值或参数值列表,如果有的话

```yaml
$emit:
  event: test
  args: # 一个对象参数
    a: 1
    b: 2
$emit:
  event: test
  args: # 表示两个对象参数
    - a: 1
    - b: 2

```

#### 内置脚本事件

- `beforeCall`: 在指令调用前触发
  - 回调参数: `(event, name, params, fn) => void|params`
  - 当回调函数返回值的时候, 则表示修改参数.
- `afterCall`: 在指令返回结果前触发
  - 回调参数: `(event, name, params, result, fn) => void|result`
  - 当回调函数返回值的时候, 则表示修改返回结果.
- `llmParams`: 事件在大模型调用前触发,可用于修改传递给大模型的参数.
  - 回调参数: `(event, params: any) => void|result<any>`
- `llmBefore`: 事件在大模型调用前触发,不可修改参数，仅作为通知.
  - 回调参数: `(event, params: any) => void`
- `llm`: 事件在大模型返回结果前触发,用于修改大模型返回结果.
  - 回调参数: `(event, result: string) => void|result<string>`
- `llmStream`: 当大模型以流式返回结果的时候触发
  - 回调参数: `(event, chunk: AIResult, content: string, retryCount: number) => void`
    - chunk: 当前流块内容
    - content: 当前得到的所有chunk的字符串内容
    - retryCount: 当因为达到`max_token`后自动调用llm的重试次数
- `llmMessagePushed`: 当大模型的返回结果作为助手消息推送到消息列表后触发
  - 回调参数: `(event, message: AIChatMessageParam) => void`
    - message: 为新增的助手消息
- `llmRequest`: 事件在需要得到大模型结果的时候触发,用于通过事件调用大模型,得到大模型结果. `[[RESPONSE]]`模板会触发该事件
  - 回调参数: `(event, messages: AIChatMessage[], options?) => void|result<string>`
  - 使用开关`disableLlmRequest: true`禁用该事件.
- `ready`: 脚本交互准备完成后触发,可以通过`$ready()`函数强制设置是否处于准备状态.
  - 回调参数: `(event, isReady: boolean) => void`
- `load-chats`: 加载聊天记录时触发.
  - 回调参数: `(event, filename: string) => AIChatMessage[]|void`
  - 当回调函数返回值的时候, 则表示加载的聊天记录.
- `save-chats`: 保存聊天记录时触发.
  - 回调参数: `(event, messages: AIChatMessage[], filename?: string) => void`
- `beforeGenerateChats`: 在生成聊天记录前触发,可用于修改原始的聊天记录.
  - 回调参数: `(event, data: {messages: AIChatMessage[], fromIndex: number, toIndex?: number}) => {messages: AIChatMessage[], fromIndex: number, toIndex?: number}|void`
- `afterGenerateChats`: 在生成聊天记录后触发,可用于修改生成的聊天记录.
  - 回调参数: `(event, data: {messages: AIChatMessage[], fromIndex: number, toIndex: number, result: AIChatMessage[]}) => AIChatMessage[]|void`

**注意**:

* 事件回调中的`event`参数是`Event`对象, `this`为脚本runtime;
* 当事件回调返回值, 则表示修改参数或结果, 否则不修改;前提是该事件类型支持修改;

## Refs

* [AutoGen](https://github.com/microsoft/autogen)
* [LMQL](https://lmql.ai/)
* [OpenInterpreter](https://github.com/OpenInterpreter/open-interpreter)
* [Outlines](https://github.com/outlines-dev/outlines)
* [MemGPT](https://github.com/cpacker/MemGPT/)
* [LangChain](https://github.com/langchain-ai/langchain)
