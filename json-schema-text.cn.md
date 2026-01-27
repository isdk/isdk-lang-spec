# JSON Schema 人类可读文本转换规范 v2.0

> 设计给人与 AI 共同阅读、理解和实现

---

## 设计理念

JSON Schema 是给机器验证用的，但人类需要快速理解"这个字段是什么、有什么限制"。

**核心原则**：

1. **字符串是默认**：人类写文档时不会特别说明"这是一段文字"，只有非字符串才需要说明类型
2. **描述优先**：`user name` 比 `string` 更有意义
3. **例外标注**：不说"必填、必填、必填、选填"，而是"默认必填，只标注选填的"
4. **层次清晰**：用缩进表达嵌套关系，一目了然

---

## 基础约定

### 缩进

每层嵌套增加 2 个空格：

```
根层级
  第一层
    第二层
```

### 标记符号

| 符号 | 用途 |
|------|------|
| `*` | 对象的固定属性 |
| `-` | 数组元素、oneOf/anyOf 选项 |
| `{...}` | 动态属性（patternProperties） |
| `...` | 附加属性（additionalProperties） |

* 对象属性使用 `*` 开头。
* 数组元素/选项使用 `-` 开头。

### 字段的基本构成

格式：`* 键名 (标签): Title. Description (类型与约束)`

* **键名**：始终用双引号包裹 `"key"`。
* **Title (技术名称)**：可选。作为简短标识。与描述之间用句号 `.` 分隔。
* **Description (描述)**：详细说明。
  * **去重规则**：如果描述文本中已包含 Title（不区分大小写），则省略 Title，只展示 Description。
* **类型**：放在行尾括号内。
  * **String (默认)**：若无约束且非 Nullable，**完全隐藏类型标记**。
  * **非 String**：必须标记，如 `(integer)`, `(boolean)`。

**对比示例**：

```json
{
  "type": "object",
  "properties": {
    "uuid": { "type": "string", "title": "UUID", "description": "用户唯一标识" },
    "user": { "type": "object", "description": "The User Object", "title": "User" },
    "username": { "type": "string", "title": "用户名" },
    "age": { "type": "integer", "description": "年龄" },
    "count": { "type": "integer", "description": "数量 integer" },
    "isEnabled": { "type": "boolean", "description": "是否启用" }
  }
}
```

输出:

```md
* "uuid": UUID. 用户唯一标识             <-- Title + Description
* "user": The User Object              <-- Description 包含 Title ("User")，省略 Title
* "username": 用户名                    <-- String, 简洁
* "age": 年龄 (integer)                 <-- 无 Title，仅 Description
* "count": 数量 integer                 <-- Integer, 描述已包含类型名，隐藏 (integer)
* "isEnabled": 是否启用 (boolean)        <-- Boolean, 显式
```

### 必填与选填 (上下文感知模式)

为了减少视觉噪音，不重复标注每一个字段，而是根据**当前层级**的统计特征进行标注。

计算当前对象层级中 `required` 字段的占比：

| 占比 | 父级标记 | 子级标注策略 |
| --- | --- | --- |
| **> 50% (严格模式)** | `(fields required by default)` | 只标注特例 `(optional)` |
| **≤ 50% (宽松模式)** | `(fields optional by default)` | 只标注特例 `(required)` |

_注：若字段数少于 3 个，可省略父级标记，直接全量标注。_

### 描述来源优先级

```
description > title > (无描述)
```

---

## 简单类型

### 转换规则

字符串不标类型，非字符串标类型：

| 类型 | 有描述 | 描述包含类型名 | 无描述 |
|------|--------|---------------|--------|
| string | `* "name": 用户名` | `* "name": 字符名` | `* "name"` |
| integer | `* "age": 年龄 (integer)` | `* "age": 年龄 integer` | `* "age" (integer)` |
| number | `* "score": 分数 (number)` | `* "score": 分数 number` | `* "score" (number)` |
| boolean | `* "active": 是否激活 (boolean)` | `* "active": boolean开关` | `* "active" (boolean)` |

### format 处理

字符串有 `format` 时，format 值本身就是类型说明：

```
{ "type": "string", "format": "email" }
→ * "email" (email)

{ "type": "string", "format": "date-time", "description": "创建时间" }
→ * "createdAt": 创建时间 (date-time)
```

常见 format：`email`, `uri`, `uuid`, `date`, `time`, `date-time`, `hostname`, `ipv4`, `ipv6`

### 特殊标记

| 关键字 | 输出 |
|--------|------|
| `default: "foo"` | `(default: "foo")` |
| `deprecated: true` | `(deprecated)` |
| `readOnly: true` | `(read-only)` |
| `writeOnly: true` | `(write-only)` |

#### 默认值 (Default)

* 置于约束最后。
* 格式：`default: value`
* 示例：`* "retry": (integer, default: 3)`

示例：

```
* "version": API版本 (default: "v1")
* "oldId": 旧ID (deprecated)
```

---

## 对象类型

### 根对象

有描述时先说明，再列字段：

```json
{
  "type": "object",
  "description": "用户信息",
  "properties": {
    "name": { "type": "string", "description": "用户名" },
    "age": { "type": "integer", "description": "年龄" }
  }
}
```

输出：

```
用户信息:
  * "name": 用户名
  * "age": 年龄 (integer)
```

无描述时直接列字段（不缩进）：

```
* "name": 用户名
* "age": 年龄 (integer)
```

### 嵌套对象

展开属性，保持层次：

```json
{
  "profile": {
    "type": "object",
    "description": "个人资料",
    "properties": {
      "avatar": { "type": "string", "description": "头像链接" }
    }
  }
}
```

输出：

```
* "profile": 个人资料
  * "avatar": 头像链接
```

如果嵌套对象没有 properties，才标注 `(object)`：

```
* "metadata": 额外数据 (object)
```

---

## 数组类型

### 根数组

```json
{
  "type": "array",
  "description": "用户列表",
  "items": { "type": "object", "description": "用户对象", "properties": {...} }
}
```

输出：

```
The list of 用户列表, each is a 用户对象:
  * "name": 用户名
  * "age" (integer)
```

简单 items：

```
{ "type": "array", "description": "标签", "items": { "type": "string" } }
→ The list of 标签

{ "type": "array", "description": "分数", "items": { "type": "number" } }
→ The list of 分数, each is a number
```

### 属性中的数组

```
* "tags": 标签列表 (list)
* "scores": 分数列表 (list of number)
* "friends": 好友列表 (list of object)
  * "name": 好友名
  * "since": 认识年份 (integer)
```

### 嵌套数组

用 `-` 标记数组元素：

```json
{
  "type": "array",
  "description": "坐标集合",
  "items": {
    "type": "array",
    "description": "单个坐标",
    "items": { "type": "number" }
  }
}
```

输出：

```
The list of 坐标集合, each is a 单个坐标:
  - list of number
```

---

## 约束条件

约束放在行末括号内，顺序：`(类型, 格式约束, 范围约束, 其他)`

### 字符串约束

| 约束 | 输出 |
|------|------|
| `enum: ["a", "b"]` | `(enum: "a", "b")` |
| `enum` > 6 项 | `(enum: "a", "b", "c", ... +4 more)` |
| `const: "x"` | `(const: "x")` |
| `pattern: "^[A-Z]+$"` | `(pattern: /^[A-Z]+$/)` |
| `minLength: 6, maxLength: 6` | `(6 chars)` |
| `minLength: 1, maxLength: 100` | `(1-100 chars)` |
| `minLength: 10` | `(at least 10 chars)` |
| `maxLength: 500` | `(at most 500 chars)` |

为了避免枚举刷屏，对枚举 (Enum) 折叠:

* **≤ 6 项**：全列出 `(enum: "A", "B")`
* **> 6 项**：前 3 项 + 剩余数 `(enum: "CN", "US", "JP", ... +190 more)`

### 数值约束

| 约束 | 输出 |
|------|------|
| `minimum: 0, maximum: 100` | `(integer, 0-100)` |
| `exclusiveMinimum: 0` | `(number, > 0)` |
| `minimum: 0` | `(integer, >= 0)` |
| `maximum: 100` | `(integer, <= 100)` |
| `multipleOf: 5` | `(integer, multiple of 5)` |

### 数组约束

| 约束 | 输出 |
|------|------|
| `minItems: 1, maxItems: 1` | `(list, exactly 1 item)` |
| `minItems: 2, maxItems: 2` | `(list, exactly 2 items)` |
| `minItems: 1, maxItems: 5` | `(list, 1-5 items)` |
| `uniqueItems: true` | `(list, unique)` |

### 对象约束

| 约束 | 输出 |
|------|------|
| `minProperties: 1, maxProperties: 10` | `(1-10 properties)` |
| `additionalProperties: false` | `(no extra properties)` |

### 值的表示

| 类型 | 格式 |
|------|------|
| 字符串 | `"value"` |
| 数字 | `123` / `3.14` |
| 布尔 | `true` / `false` |
| null | `null` |

---

## 组合类型

### oneOf（互斥，恰好匹配一个）

```json
{
  "oneOf": [
    { "type": "string", "description": "错误信息" },
    { "type": "object", "description": "详细错误", "properties": {...} }
  ]
}
```

**摘要**：优先取各选项的 Title，若无则取描述（或 format，或 type）生成：

```
(one of: 错误信息 | 详细错误)
```

**展开规则**：任一选项有描述或是复杂类型 → 展开

```
* "error" (one of: 错误信息 | 详细错误)
  - 错误信息
  - 详细错误:
    * "code" (integer)
    * "message"
```

全是简单类型且无描述 → 只显示摘要：

```
* "value" (one of: string | integer | null)
```

### anyOf（兼容，匹配至少一个）

格式与 oneOf 相同，但使用 `any of`：

```
* "contact" (any of: 邮箱 | 电话 | 地址)
  - 邮箱 (email)
  - 电话 (pattern: ^\d{11}$)
  - 地址
```

### allOf（同时满足）

**先合并，再渲染**：

```json
{
  "allOf": [
    { "properties": { "id": { "type": "integer" } }, "required": ["id"] },
    { "properties": { "name": { "type": "string" } } }
  ]
}
```

合并后按普通对象处理：

```
* "id" (required, integer)
* "name"
```

合并规则：

- `properties` → 后者覆盖同名
- `required` → 取并集
- 约束 → 取更严格的值

---

## 混合类型

`type` 为数组时的处理：

* 格式：`(typeA | typeB)`
* 示例：`* "id": (integer | string)`

### 简单情况

```
{ "type": ["string", "integer"] }
→ (string | integer)

{ "type": ["string", "null"] }
→ (nullable)

{ "type": ["integer", "null"] }
→ (integer, nullable)
```

### 有类型特定约束

视为隐式 anyOf，按类型分组约束：

```json
{
  "type": ["string", "integer"],
  "minLength": 3,
  "minimum": 100
}
```

输出：

```
* "code" (any of: string | integer)
  - string (at least 3 chars)
  - integer (>= 100)
```

---

## nullable 标注

`nullable` 表示值可以为 `null`，与 `optional`（字段可不存在）是两个独立概念：

| 情况 | 字段存在性 | 值 |
|------|-----------|-----|
| required | 必须存在 | 不能为 null |
| optional | 可不存在 | 不能为 null |
| required + nullable | 必须存在 | 可以为 null |
| optional + nullable | 可不存在 | 可以为 null |

示例：

```
* "name" (required): 用户名
* "bio" (optional): 个人简介
* "avatar" (required, nullable): 头像URL
* "nickname" (optional, nullable): 昵称
```

---

## 动态属性

### patternProperties

当键名不是固定的，而是正则匹配或动态定义时，使用花括号 `{}` 包裹，以暗示这是“变量”。

格式：`* {/正则/}: 描述`

* **有描述**：`* {/^[a-z]{2}$/}: 国家代码...`
* **无描述**：`* {/^[0-9]+$/}: ...`

举例:

```json
{
  "patternProperties": {
    "^x-": { "type": "string", "description": "扩展字段" },
    "^data_\\d+$": { "type": "object" }
  }
}
```

输出：

```
* {/^x-/}: 扩展字段
* {/^data_\d+$/}: (object)
```

### additionalProperties

附加属性处理：

| 值 | 处理 |
|---|------|
| `true` | 不输出（默认行为） |
| `false` | `(no extra properties)` |
| `{ schema }` | 用 `...` 表示，按普通属性处理 |

举例:

```json
{
  "additionalProperties": {
    "type": "object",
    "description": "插件配置",
    "properties": { "enabled": { "type": "boolean" } }
  }
}
```

输出：

```
* "name": 插件名
* ...: 插件配置
  * "enabled" (boolean)
```

---

## 引用 ($ref)

### 可解析的引用

展开内容，同内联处理。属性自身的描述优先于引用目标的描述。

### 不可解析的引用

```
* "config" (see: #/definitions/Config)
```

### 支持的路径格式

- `#/definitions/Name`（draft-04/07）
- `#/$defs/Name`（draft 2019-09+）
- `#/components/schemas/Name`（OpenAPI）

---

## 边界情况

| 情况 | 处理 |
|------|------|
| `null` / `undefined` | 返回空字符串 |
| `{}` 或 `true` | `(any)` |
| `false` | `(never)` |
| 对象只有 description | `描述:` |
| 数组无 items | `The list` |
| 循环引用 | `(circular: #/path)` |
| 超过深度限制（10层） | `(see: #/path)` |
| 未知 type | `(unknown type: xxx)` |

---

## 完整示例

输入：

```json
{
  "type": "object",
  "description": "API响应",
  "required": ["success", "data"],
  "properties": {
    "success": { "type": "boolean", "description": "请求是否成功" },
    "data": {
      "type": "array",
      "description": "结果列表",
      "items": {
        "type": "object",
        "required": ["id"],
        "properties": {
          "id": { "type": "integer", "minimum": 1 },
          "name": { "type": "string", "maxLength": 100 },
          "email": { "type": "string", "format": "email" },
          "status": { "type": "string", "enum": ["active", "inactive", "pending"] }
        }
      }
    },
    "error": {
      "oneOf": [
        { "type": "string", "description": "错误信息" },
        {
          "type": "object",
          "description": "详细错误",
          "properties": {
            "code": { "type": "integer" },
            "message": { "type": "string" }
          }
        }
      ]
    },
    "meta": {
      "type": ["object", "null"],
      "description": "元信息"
    }
  }
}
```

输出：

```
API响应 (unmarked: required):
  * "success": 请求是否成功 (boolean)
  * "data": 结果列表 (list of object)
    * "id" (required, integer, >= 1)
    * "name": (at most 100 chars)
    * "email" (email)
    * "status": (enum: "active", "inactive", "pending")
  * "error" (optional, one of: 错误信息 | 详细错误)
    - 错误信息
    - 详细错误:
      * "code" (integer)
      * "message"
  * "meta" (optional): 元信息 (object, nullable)
```

---

## 实现指南

### 处理流程

```
输入 JSON Schema
       ↓
┌─────────────────────────────────────┐
│ 1. 规范化                            │
│    - 解析 $ref                       │
│    - 合并 allOf                      │
│    - 展开 type 数组（含 null 时提取） │
└─────────────────────────────────────┘
       ↓
┌─────────────────────────────────────┐
│ 2. 分析上下文                        │
│    - 识别根类型                      │
│    - 计算 required 比例              │
│    - 确定标注策略                    │
└─────────────────────────────────────┘
       ↓
┌─────────────────────────────────────┐
│ 3. 递归渲染                          │
│    - 按类型分发处理                  │
│    - 维护缩进深度                    │
│    - 检测循环引用                    │
└─────────────────────────────────────┘
       ↓
输出 Markdown 文本
```

### 规范化伪代码

```
function normalize(schema, visited):
    if schema is null or undefined:
        return null

    if schema.$ref:
        if schema.$ref in visited:
            return { $circular: schema.$ref }
        visited.add(schema.$ref)
        resolved = resolve(schema.$ref)
        schema = merge(resolved, schemaWithout$ref)

    if schema.allOf:
        schema = mergeAllOf(schema.allOf)

    if isArray(schema.type):
        if "null" in schema.type:
            schema.nullable = true
            schema.type = schema.type.filter(t => t != "null")
        if schema.type.length == 1:
            schema.type = schema.type[0]
        else:
            schema = convertToAnyOf(schema)

    return schema
```

### 渲染分发

```
function render(schema, depth, context):
    schema = normalize(schema, context.visited)

    if schema.$circular:
        return "(circular: " + schema.$circular + ")"

    if schema.oneOf or schema.anyOf:
        return renderCombination(schema, depth, context)

    switch schema.type:
        case "object":
            return renderObject(schema, depth, context)
        case "array":
            return renderArray(schema, depth, context)
        case undefined:
            return "(any)"
        default:
            return renderSimple(schema)
```

### 上下文对象

```
context = {
    visited: Set(),           // 已访问的 $ref 路径
    depth: 0,                 // 当前嵌套深度
    maxDepth: 10,             // 最大深度限制
    definitions: {},          // schema 定义集合
    requiredFields: Set(),    // 当前对象的 required 字段
    requiredRatio: 0,         // 必填比例
    labelStrategy: "optional" // 标注策略: "optional" | "required" | "explicit"
}
```

---

### AI System Prompt

以下指令用于配置 ChatGPT/Claude 等 LLM，使其严格执行上述规范：

```md
You are a JSON Schema documentation generator. Convert schemas into concise, human-readable format.

### PROCESS
1. **Normalize**: Resolve $ref → Merge allOf → Extract null from type arrays
2. **Analyze**: For each object, calculate required field ratio
3. **Render**: Recursively process by type with proper indentation

### RULES:
1. **Contextual Requirement**:
   - Before outputting an object, count its required properties.
   - If >50% are required: Label parent `(fields required by default)` and only mark `(optional)` children.
   - If <=50% are required: Label parent `(fields optional by default)` and only mark `(required)` children.
2. **Title & Description**:
   - **Format**: `Title. Description`.
   - **Deduplication**: If `Description` contains `Title` (case-insensitive), omit `Title`.
   - **Summary**: In `oneOf`/`anyOf` summaries, strictly prioritize `Title` over `Description`.
3. **Type Simplification**:
   - **String**: If type is `string` and NOT nullable, hide the type label. Just show the description.
   - **Nullable**: If `type: ["x", "null"]`, explicitly label as `(nullable)`.
   - **Mixed**: If multiple types (e.g., string/int), write `(string | integer)`.
4. **Combinations**:
   - `oneOf` → `(one of: A | B)` - exactly one must match
   - `anyOf` → `(any of: A | B)` - at least one must match
   - `allOf` → Merge schemas first, then render as single object
5. **Formatting Constraints (Shorthand)**:
   - **Enums**: If >6 items, list top 3 and add `... +N others`.
   - **Arrays**: Format as `Description (list of ItemDescription)`.
   - **Numbers**: Use shorthand `(min N)`, `(max N)`, `(> N)`.
   - **Strings**: Use shorthand `(max N chars)`, `(email)`, `(uuid)`, `(datetime)`.
   - **Refs**: If `$ref` is found, resolve it if possible, otherwise write `(see DefinitionName)`.
6. **Dynamic Properties**:
   - `patternProperties` → `* {/regex/}: Description`
   - `additionalProperties: schema` → `* ...: Description`
7. **Arrays**:
   - Property: `* "items": Description (list of type)`
   - Root: `The list of <ArrayDescription>, each is a <ItemDescription>:`
   - Complex items → Expand nested structure
8. **References**:
   - Resolvable `$ref` → Expand inline
   - Unresolvable → `(see: #/path/to/def)`
   - Circular → `(circular: #/path)`

### FORMAT
- 2-space indentation per level
- `*` for object properties
- `-` for array items, oneOf/anyOf options
- `* "key": Description (constraints)`

### EXAMPLE INPUT:
{
  "type": "object",
  "required": ["id"],
  "properties": {
    "id": { "type": "integer" },
    "status": { "type": "string", "enum": ["open", "closed"], "default": "open" },
    "meta": {
      "type": "object",
      "properties": { "msg": { "type": "string" } }
    },
    "attrs": {
      "type": "object",
      "patternProperties": { "^x-": { "description": "Custom attr" } }
    }
  }
}

### EXAMPLE OUTPUT:
(fields optional by default):
  * "id" (required): (integer)
  * "status": (enum: "open", "closed", default: "open")
  * "meta": (fields optional by default)
    * "msg": (string) note: implicit string label hidden if description existed
  * "attrs":
    * { /^x-/ Custom attr }: (string)
### TASK:
Convert the provided JSON Schema now.
```

## 版本兼容

本规范针对 JSON Schema draft-07，兼容：

| 版本 | 特殊处理 |
|------|----------|
| draft-04 | `definitions` 替代 `$defs` |
| draft 2019-09+ | `$ref` 可与其他关键字共存 |
| OpenAPI 3.x | `nullable` 关键字、`components/schemas` 路径 |
