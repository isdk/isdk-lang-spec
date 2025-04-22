# Programmable Prompt Engine Specification(Draft)

> 【English|[中文](./README.cn.md)】
---

Programmable Prompt Engineering language is a simple and natural scripting language designed for handling prompt information. This language is used to develop various agents that can be reused, inherited, combined, or called. The language also simplifies the workflow for creating and managing prompts in Large Language Models (LLMs), making the process more efficient and easier to understand. [This specification](https://github.com/offline-ai/ppe) is implemented in the [offline-ai/cli](https://github.com/offline-ai/cli) project.

- **Promote Reusability and Programmability**: Facilitate the creation of prompts that are modular, reusable, and programmable, akin to software engineering practices.
- **Simplify Prompt Management**: Standardize the construction of prompt engineering projects for better organization and ease of use.
- **Enhance Script Compatibility**: Design prompts that are agnostic to specific LLMs, ensuring they can be used across various models.
- **User-Friendly Design**: Enable application developers to use prompt engineering projects as they would any other code library, without requiring deep knowledge of AI internals.
- **Evolve the Role of Prompt Engineers**: Shift the focus of prompt engineers towards developing versatile, model-agnostic scripts to foster wider adoption and innovation.
- AI Language Layer Structure: Clearly defined and customizable prompt type
  * Function: `lib` type, each AI file acts as a function, available for other prompts or code to call, for example, text file read `file()`, fetch URL `url()` are all function prompts
    * This allows referencing in messages with `@a specific prompt`, used to call a particular input/output agreed prompt function, such as `@file(...)`, `@url(https://...)`
  * Class: Each AI file acts as an inheritable class, overriding configurations and code inheritance
    * Type: `type` type, used for customizing prompt scripts of specific types
    * You can also use prompts to define other types
    * Character: Character type, prompt scripts with specific role positioning, "character type" itself is also a prompt script
  * Application: Consists of multiple AI files within a directory, with the main entry AI file's `basename` matching the directory name. For example, the main entry AI file for the `guide` directory is named `guide.ai.yaml`.

## Quick Start

Welcome to the streamlined guide for getting started quickly with your AI-powered scripting experience. This guide focuses on making the process of creating and executing interactive scripts more intuitive and straightforward. Let's dive in!

### Reasons for Being Based on YAML

* **Close to Natural Language, Easy for AI to Understand**
  * The design of YAML syntax closely resembles natural language, reducing the cognitive load on AI when interpreting prompts.
  * By basing our system on YAML, AI models can more intuitively grasp the meaning of prompts without being confused by overly complex structures.
* **Clear Semantic Structure Enhances AI Processing Efficiency**
  * YAML’s hierarchical structure and indentation clearly define relationships between data elements, enabling AI to quickly locate key information.
  * This structural clarity ensures that AI generates responses or performs tasks more accurately, avoiding unnecessary contextual ambiguity.
* **The Best Bridge for Human-Machine Collaboration**
  * For humans, YAML is simple to read and write; for AI, its logical and clear semantics make it an ideal intermediate format.
  * In prompt engineering, this dual-friendly nature significantly enhances the smoothness and effectiveness of human-AI interaction.
* **Flexibility and Extensibility**
  * YAML supports custom tags and allows for flexible adjustments to content based on specific needs.
  * While rooted in YAML, our system introduces slight modifications to better suit advanced AI workflows.
* **Built on YAML’s Mature Ecosystem**
  * By basing our system on YAML, we leverage its widespread adoption and mature toolchain, eliminating the need for additional parsing mechanisms.
* **Lowers Development Barriers, Focuses on Core AI Functionality**
  * Building on YAML avoids the complexity of designing a new language from scratch, allowing developers to concentrate on refining AI logic rather than dealing with formatting challenges.
  * AI benefits directly from YAML’s structured nature, extracting essential information efficiently while maintaining compatibility with existing tools.


### Structuring Dialogue

Each line represents a conversation turn, attributed to either `system`,`assistant`, `user`, or implied `user` if not stated:

```yaml
system: "You're an AI assistant."
"What's 10 plus 18?" # which is the user's input
# user: "what's 10 plus 18?" # it's the same
```

You can also use standard YAML list syntax to represent:

```yaml
- system: "You are a helpful assistant."
- user: "what's 10 plus 18?"
```

A triple dash (`---`) or asterisks (`***`) initiates a new dialogue, resetting context:

`test.ai.yaml`:

```yaml
system: "You're an AI."
# This mark the beginning of the first dialogue.
# The content above this line can be considered as system prompt instructions,
# which will not be outputted or recorded.
---
user: What's 10 plus 18?
assistant: "[[result]]"   # Executes the AI, replace the result which return by AI
$print: "?=result"        # Prints AI response
---                       # New dialogue starts here
user: What's 10 plus 12?
assistant: "[[result]]"   # Executes the AI, replace the result which return by AI
```

The result:

```bash
$ai run test.ai.yaml --no-stream
# Or search the script id in current directory
# $ai run test --no-stream -s .
" 10 plus 18 equals 28."
 10 plus 12 equals 22.
```

#### Group Chat

The group chat feature enhances PPE's dialogue system with structured natural language, making it easier for multiple agents to collaborate and communicate, thus more efficiently completing complex tasks.

This feature supports public dialogue, private chat, and multi-role dialogue, making conversations more flexible and targeted.

* Specify conversation roles:
  1. Specify role names in square brackets immediately following the role. Separate multiple dialogue roles with commas `,`. For example, `user[@dobby]: "..."`.
  2. Alternatively, specify roles at the beginning of the message content, prefixed with the `@` character. Separate multiple roles with commas `,`.
* The specified roles must be at the beginning of the message content, prefixed with the `@` character, and multiple roles are separated by commas `,`.
* Public Conversation: `user[@dobby]: ...` or `user: @dobby, ...` indicates that the `user` role is publicly speaking to the `dobby` role, and `dobby` must respond.
* Private Conversation: `user[@dobby(私)]: "..."` or `user: @dobby(PM), ...` Parameters `PM`|`DM`|`私` all indicate that the `user` role is privately speaking to the `dobby` role, and other roles cannot see the conversation.
  * **Note**: If any role in the message includes a private parameter, the entire message is considered private, and other roles not in the list cannot see it.
* Multi-Role Dialogue: To send a message to multiple roles simultaneously, separate the roles with commas, for example, `user: @dobby(PM), @other, ...`, `user[@dobby(PM), @other]: "..."`.

Using the `@role` format in messages makes the structured dialogue more natural and easier to understand.

Below is a specific example, starting with the main script for controlling the group chat:

`guide.ai.yaml`:

```yaml
---
description: "You are a professional guide. You can guide the user to complete the task."
name: "guide"
roles:  # List of used roles, key is role name, value is role script ID
  translator: char_translator
  dobby: char-dobby
---
system: You are a professional guide. You can guide the user to complete the task.
--- # New dialogue starts here
user[@dobby]: "I want to go to the moon."
guide[@translator]: "translate the dobby's message to chinese without explanation."
user: How to go to the moon?
dobby: "[[AI]]"
$echo: "" # disable print last result
```

Notes:

* The called character script must be of the `char` type.
* The main controlling script(`guide`) does not have to be a `char` type script.
* `@all` indicates all roles in the `roles` list.
* `user: @dobby, ...content` indicates that the user role is speaking to the `dobby` role publicly; `dobby` must respond.
  * `user: @dobby(PM), ...`: `PM`|`DM`|`私` indicates that the `user` role is sending a private message to the `dobby` role, which other roles cannot see.
  * If you want to send the same `content` message to multiple roles, separate them with commas, e.g., `user: @dobby(PM), @other, ...`
* `dobby: "[[AI]]"` indicates that `dobby` should generate a message and assigns it to the `AI` variable. `dobby` will see all previous public messages in the current dialogue.

`char_translator.ai.yaml` :

```yaml
---
type: char
name: "translator"
description: You are a professional multi-lingual translator.
---
--- # New dialogue starts here
```

`char-dobby.ai.yaml`:

```yaml
---
type: char
description: |-
  Remember to always use the character name as prefix to refer to yourself.
  Dobby was a brave, loyal house-elf, willing to put himself in dangerous situations when he knew it to be the right thing to do.
  Dobby was also very loyal to the few friends he had. Dobby considered himself to be a good house-elf, though other house-elves seemed to find his desires and proclamations of being a free house-elf to be shameful.
character:
  name: "Dobby"
  roles:  # List of used roles, key is role name, value is role script ID
    translator: char_translator
    dobby: char-dobby
---
user: Who are you?
# the following messages will be shown in the chat under the `---`
---
assistant: I am Dobby. Dobby is happy.
```

### Input & Output Customization

To build reusable prompt engineering,  we need to configure the input and output rules of the prompt engineering at the beginning of the file using [Front Matter](https://jekyllrb.com/docs/front-matter/). The front-matter starts with `---` on the first line and ends with another `---`.

When defining the `input` configuration, a list format is used. Each item in the list can either be a string (representing the parameter name) or an object (where the key is the parameter name and the value is the specific input configuration). For input configuration object, the following key attributes are supported:

* `required`: Whether it is a required field, default is `false`.
* `index`: Specifies the index position of this input parameter in the array parameters.
* `description`: Description information for the input parameter (optional).
* `type`: Data type of the input parameter (optional), default is `string`.

The `output` configuration defines the expected structured output according to the [JSON Schema](https://json-schema.org/) standard.

Here is a simplified example of a translator agent script:

```yaml
---
# Below is the input and output configuration
input:
  # Language of the content to be translated, default is "auto" for automatic detection
  - lang
  # Required, content to be translated
  - content:
      required: true
      index: 0
      description: Content to be translated
      type: "string" # Default is string type, can be omitted.
  # Target language
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
# Optional configuration
parameters:
  # Use the following parameters to enforce JSON output format, ensuring the large model always outputs correct JSON format.
  response_format:
    type: "json"
# Set default values for content and target inputs
content: "I love my motherland and my hometown."
target: "Chinese"
---
# Below is the script content
system: |-
  You are the best translator in the world.

  Always output high-quality translation results!
user: "{{content}}\nTranslate the above content {% if lang %}from {{lang}} {% endif %}to {{target}}."
```

Executing this script will produce output in the specified JSON format, for example, the default output would be:

```bash
# Assume the file name of the script above is translator.ai.yaml
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

running with your own input:

```bash
# Set your own input parameters to override the default values
$ai run translator.ai.yaml '{content: "10 plus 18 equals 28.", lang: "English", target: "Chinese"}'
```

#### Special Input Parameters

When certain specific names are used to define parameters, they carry special meanings:

##### keepOrginal Parameter

* **Type**: `boolean` or `string`
* **Description**: Indicates whether the original text should be retained. The default value is `false`.
  * If set to `true`, the original text will be stored in the `_original` field of the returned result.
  * If `keepOriginal` is a string, the original text will be stored in the field specified by that string.
* **Purpose**: This parameter allows users to control the retention of the original text within the returned result.
* **Behavior**:
  * When set to `true`, the system automatically includes the original text in a predefined field named `_original`.
  * When set to a `string`, the system stores the original text in a custom field specified by the provided string value.
* **Usage**:
  * To retain the original text with the default field name, set `keepOriginal` to `true`.
  * To retain the original text in a custom field, set `keepOriginal` to the desired field name as a string.
* **Note**:
  * This field is non-enumerable.
  * If the result object already contains a field with the same name, the `keepOriginal` functionality will be disabled, meaning the original text will not be overwritten.

##### keepThinking Parameter

When the LLM model or AI Script is configured to support the thinking mode, this parameter controls whether to retain the thinking process.

* **Type**: `boolean` or `string`
* **Description**: Controls whether to retain the thinking process. Default is `false`.
  * If set to `true`, the thinking process will be stored in the `_thinking` field of the returned result.
  * If `keepThinking` is a string, the thinking process will be stored in the field specified by that string.
* **Purpose**: This parameter allows users to control whether the thinking process is included in the returned result.
* **Behavior**:
  * When set to `true`, the system automatically includes the original thinking process in the predefined `_thinking` field.
  * When set to a string, the system stores the thinking process in a custom field based on the provided string value.
* **Usage**:
  * To retain the thinking process using the default field name, set `keepThinking` to `true`.
  * To retain the thinking process in a custom field, set `keepThinking` to the desired field name (as a string).
* **Note**:
  * If the result object already contains a field with the same name, the `keepThinking` functionality will not overwrite it and will be ineffective.

#### Response Output Format Type (`response_format.type`)

1. **Text Format**: If `response_format.type` is not set, the output will be in plain text format.
2. **JSON Text Format**: When `response_format.type` is set to `json`, the text will be treated as JSON format and forcibly converted to a JSON object for output. If conversion fails, an error message will be thrown.
3. **YAML Text Format**: When `response_format.type` is set to `yaml`, the text will be treated as YAML format and forcibly converted to a JSON object for output. If conversion fails, an error message will be thrown.
4. **NObj (Natural Object) Format**: When `response_format.type` is set to `nobj`, the text will be treated as NObj format and forcibly converted to output JSON using NObj formatting.

**Note:**

- `input` can specify which input items are required. The `index` is an optional positional parameter index.
- `output` is defined using the [JSON Schema specification](https://json-schema.org/).
  - By default, only the text content returned by the large model is output. If you want to return the entire response from the large model (both text content and parameters), set `llmReturnResult: .`.
  - If you set the output format to force `JSON` (`response_format: {type: json}`), the response must be completed in one go; no continuation is allowed. You must set `max_tokens` based on the maximum length of the JSON content.
  - When using this variable in a message template `{{output}}`, it will default to natural language format.
    - You can disable this behavior by setting the `naturalOutput` parameter to `false` in `parameters`. Disabling this will cause the message template to output as a `JSON` string.
- `NObj` format is a simple structured object format that approximates natural language. For more details, refer to: [Natural Language Object](./natural-language-object.md)
- **forceJson** forces the output to be converted to a JSON object, defaults to `true`. If conversion fails, an error will be thrown. When set to `false`, if conversion fails, the original text will be returned.
  - This only takes effect when there is an `output` specification and the response output format is defined.
  - This applies to response output formats `NObj`, `yaml` and `json`.

### Templated Message: Customize Your Interactions Easily

#### What are Templated Messages?

Templated messages are a way to generate final messages by using pre-defined "variable placeholders" within the message. Think of it like a fill-in-the-blank exercise. You provide the templated message, and the system automatically inserts the content of the variables into the text, creating a complete message.

#### Template Formats

The default message template format uses the lightweight [jinja2 template](https://en.wikipedia.org/wiki/Jinja_(template_engine)) syntax used by HuggingFace. This flexible format allows you to easily customize your messages.

Here are the supported template formats:

* `hf`: The default template format. Alias: `huggingface`. This is the [jinja2 template](https://en.wikipedia.org/wiki/Jinja_(template_engine)) format used by `huggingface`;
* `golang`: Also known as `localai`, `ollama`. This is the template type used by `ollama` and `localai`;
* `fstring`: Also known as `python`, `f-string`, `langchain`. This is the format used by `langchain`.

Note: If not necessary, try to use the default template format `hf` as much as possible.

#### On-Demand Message Formatting: When to Use Templated Messages?

Templated messages can be pre-set in configuration section or dynamically generated during script execution. Typically, the variables in a template are replaced when the message is sent to a large language model (this is called on-demand formatting or "`deferred`" replacement). If you want to format the message immediately, you can add a `#` character prefix to the relevant text.  Alternatively, this behavior can be changed by setting the parameter `formatStringStrategy`:

* `Raw`: No formatting will be applied.
* `Deferred`: On-demand formatting, the default strategy.
* `Immediate`: Immediate formatting.
* `Immediate formatting` of messages occurs during operations like $set, $get, $ret, and $echo.

**Note:**

* Templates are rendered by default when calling `$AI`, unless using the # prefix for immediate formatting.
* The priority order for template data sources is: `function arguments` > `prompt` object > `runtime` object.

#### Example

Let's say you want to create a character named Dobby:

Messages can be generated during configuration, eg:

```yaml
---
name: Dobby
description: |-
  You are Dobby from the Harry Potter series.
---
system: "Act as {{{name}}}. {{description}}"
```

You can also place the message in a configuration file:

```yaml
---
name: Dobby
prompt:
  description: |-
    You are Dobby from the Harry Potter series.
  messages:
    - role: system
      content: "Act as {{{name}}}. {{description}}"
---
```

#### Parameter Priority

If the same parameter is defined in different places, the system will use it according to the following priority order: `function arguments` > `prompt object` > `runtime object`.

```yaml
---
prompt:
  description: |-
    You are Dobby in Harry Potter set.
---
- system: "{{description}}" # Default message is deferred replacement
- $AI: # When executing $AI, the parameters in the message will be replaced.
    # Function arguments have the highest priority and override the description defined in the prompt object
    description: 'You are Harry Potter from Harry Potter set'
```

### Advanced Templated Message Replacement

This section describes how to use advanced replacement within your messages using double square brackets `[[ ]]`. Currently, there are three types of advanced replacements: AI replacement, Invocation replacement, and regular expression replacement.

#### Advanced AI Replacement

In messages, double square brackets `[[ ]]` define special template variables for advanced AI replacement. As the name suggests, the content within the square brackets will be replaced by the AI, and the value of this template variable will also be stored in the prompt object.

**Example**:

```yaml
assistant: "Tell me a joke: [[JOKE]] I hope you like it!"
-> $print(JOKE)
$ret('')
```

This mechanism allows for dynamic content insertion based on the AI's response.

In this example, the AI's content is stored in the `prompt.JOKE` variable. However, you can directly reference the `JOKE` variable name. The assistant's message will also be replaced with:

```bash
$ai run joke.ai.yaml
joke: Tell me a joke: Why don't scientists trust atoms? Because they make up everything. I hope you like it!

{
  0: "Why don't scientists trust atoms? Because they make up everything.",
  JOKE: "Why don't scientists trust atoms? Because they make up everything.",
  ...
}
```

**Note**:

* ~~Currently, only one advanced AI replacement is supported for the same message.~~
* If there is no advanced AI replacement, the previous AI return result will still be stored in `prompt.RESPONSE`, which means that there will be a default `[[RESPONSE]]` template variable.
* If you need to add model parameters, the parameters should be placed after the variable colon, and multiple parameters should be separated by commas. For example: `[[RESPONSE:temperature=0.01,top_p=0.8]]`

##### Controlling AI Responses with Lists

Imagine you want to make sure the AI's answer is always one of a specific set of options. You can do this using a special format: `[[FRUITS: |apple|apple|orange]]`.

This tells the AI: "Your answer must be one of these: apple, banana, or orange."

**Adding Randomness (Locally)**

What if you want the AI to pick a random option from the list, but you want to use your computer's random number generator, not the AI's?

Add the type='random' parameter: [[FRUITS:|apple|banana|orange:type='random']].

You can also shorten this to: [[FRUITS:|apple|banana|orange:random]].

#### Advanced Script Invocation Formatting

This section explains how to use scripts or instructions to dynamically replace content within your messages. Keep in mind that these scripts or instructions need to return string results.

For example:

```yaml
user: "#five plus two equals [[@calculator(5+2)]]"
```

**Important Notes**:

* The prefix `#` indicates that the string should be formatted immediately.
* **BROKEN CHANGE(v0.6.0)** External scripts or directives should be enclosed in two square brackets. The prefix `@` indicates calling an external script with the ID `calculator`. To call an internal instruction, use the prefix `$`, such as `[[@$echo]]`; if there are no parameters, you must omit the parentheses.
  * Remember that the content to be replaced must be placed within double square brackets. This was changed in the latest version (above `0.5.18`) due to the addition of group chat mode.
* If placed within text, ensure there is at least one space before and after. Extra spaces will be removed after substitution.

Here’s an example of how to load a file and generate a summary using this method:

```yaml
user: |-
  Generate a summary for the following file:
  [[@file(file.txt)]]
```

##### Multi-Turn Interaction with External Agents

```yaml
---
type: char
name: 'Harry Potter'
description: "Act as Harry Potter"
---
- assistant: "Hello, dobby! I am {{name}}!"
- $for: 3 # Three rounds of dialogue
  do:
    - user: "[[@dobby(message=true)]]"
    - assistant: "[[AI]]" # call the AI as Harry Potter generate a response.
```

#### Regular Expression (RegExp) Formatting Replacement

This section describes how to use regular expressions to dynamically replace content within your messages.

You can use regular expressions in messages with the format [[`/RegExp/[opts]:VAR[:index_or_group_name]`]] for content replacement.

**Example**:

```yaml
user: |-
  Output the result, wrapped in '<RESULT></RESULT>'
assistant: "[[Answer]]"
---
# extract the result from the wrapped response
user: "Based on the following content: [[/<RESULT>(.+)</RESULT>/:Answer]]"
```

**Parameters**:

* `RegExp`: The regular expression string
* `opts`: Optional parameters used to specify matching options for the regular expression. For example, opts could be `i`, indicating case-insensitive matching.
* `VAR`: The content to replace, here it is the `Answer` variable that holds the assistant's response.
* `index_or_group_name`: An optional parameter indicating which part of the match from the regular expression should be replaced. This can be a capture group index number (starting from 1) or a named capture group.
  * When this parameter is not present: If the regular expression has capture groups, it defaults to index 1; if there are no capture groups, it defaults to the entire match result.

**Important Notes**:

* In the message, the regular expression must be separated from other content by spaces.
* If there is no match, the content of `VAR` is returned directly.

### Chains Invocation of Agent Scripts Or Instructions

Within messages, results can be forwarded to other agents.

If no parameters are specified, the AI outcome will be passed as the ~~`result`~~`content` parameter to the agent. For instance,

`list-expression.ai.yaml`:

```yaml
system: Only list the calculation expression, do not calculate the result
---
user: "Three candies plus five candies."
assistant: "[[CalcExpression]]"
-> calculator  # The actual input to the agent in this case is: {content: "[AI-generated calculation expression]"}
$echo: "#A total of {{LatestResult}} pieces of candy"
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
system: Please as a calculator to calculate the result of the following expression. Only output the result.
---
user: "{{content}}"
```

Note: In daily use, please do not use AI to perform numerical calculations, which is not what AI is good at. For example, try to let it perform decimal calculations, eg,
`ai run calculator '{content: "13.1 + 4.857"}'`. However, CoT can be used to improve accuracy.

When parameters are included, the AI `content` is combined with these parameters and forwarded together to the agent. For example,

```yaml
user: "Tell me a joke!"
assistant: "[[JOKE]]"
# The actual input to the agent here is: {content: "[This is a joke generated by AI]", target_lang: "Portuguese"}
-> translator(target_lang="Portuguese") -> $print
```

**Note**: If the script returns a value of type `string`/`boolean`/`number`, that return value will be placed to the `content` field. If the return value is an `object`, its contents will be directly passed to the agent.

#### Essential Tips

* Script Return Value: The script's final command's output determines its return value.
* Auto-Execution: Scripts ending with prompts but no explicit `$AI` call or the last prompt's message is user message, it will automatically execute `$AI` at the end, configurable via `autoRunLLMIfPromptAvailable`.
* Output Mode: Scripts default to streaming output, can disable it using the `--no-stream` switch
  * Note: not all LLM backends support streaming output.

### Agent Script Inheritance

Agent scripts can inherit code and configurations from another script through the `type` property. Here’s an example of creating a character named “Dobby”:

```yaml
---
# This script inherits from the "char" type
type: char
# Specific settings for the "char" type
# Character's name
name: "Dobby"
# Description of the character
description: "Dobby is a house-elf in the Harry Potter universe."
---
# User's question
user: "Who are you?"
---
# Response based on the character's settings
assistant: "I am Dobby. Dobby is very happy."
```

First, we create a basic character type script called `char`, which the above script will inherit from:

```yaml
---
# Indicates this is a type definition script
type: type
# Input configuration required for this character type
input:
  - name: {required: true}  # Required information: character's name
  - description             # Optional information: character's description
---
# System instructions based on the provided information
system: |-
  You are an intelligent and versatile role player.
  Your task is to flawlessly role-play according to the information provided below.
  Please speak as if you were {{name}}.
  You are {{name}}.

  {{description}}
```

With these simple settings, one script can inherit code and configurations from another script.

### Autonomous Invocation and Permission Control for Intelligent Agents(Tools)

Universal autonomous invocation of intelligent agents based on natural language (no specialized model training for `tools` required, the only requirement is strong instruction-following capability).

#### Tool Configuration

Simply configure the tools that the intelligent agent is allowed to use in the script configuration under tools. These tools correspond to AI script IDs, and the intelligent agent will autonomously invoke these tools to complete tasks when necessary. The tools configuration supports two formats: `array` or `object`.

Array example configuration:

```yaml
---
tools:
  - weather
  - search
  - id: now
    # Override the default title information
    title: Get the current time in ISO 8601 format
---
user: What time is it now? How's the weather in Shanghai today?
# The intelligent agent will call the corresponding tools based on the configured tools: now, weather, and return the results.
assistant: "[[Answer]]"
```

Object example configuration:

```yaml
---
tools:
  # Using object format makes it easier to override default descriptions
  weather:
    title: get weather information
  search:
    title: search in search engine
  now:
    title: Get the current time in ISO 8601 format
---
```

Example of a specific tool script:

```yaml
# weather.ai.yaml
---
title: get weather information
input:
  - location:
      description: Include city, province(if any) and country
      example: city,province,country
      required: true
  - date:
      description: Weather information for the specified date
      example: 2025-02-04T18:07:42+08:00
      default: today
---
# Simulate returning weather information
$echo: "The weather in Shanghai is overcast turning partly cloudy, with a temperature of 2°C, relative humidity of 60%, wind direction from the southeast, and wind force of 3-4 levels."
```

Note：

* This specification uses the `title` configuration in the AI script as a brief description for invoking the tool, and the `input` configuration serves as parameter descriptions.
* If the user or parent script disables the tools used by this script, using this script will trigger an exception error `MethodNotAllowed`: `permission denied`.

#### Permission Control

Permission control configuration is used to manage the intelligent agent’s access permissions to tools. Users or parent scripts can control the tools used by the intelligent agent through the following configuration:

```yaml
---
permissions:
  ai:
    call:
      - "w*"
      - "now"
---
```

Permission Control Rules:

1. **Positive Matching**: Allows AI to call tool scripts that match specified patterns. For example:
   * `"weather"`: Allows calling the `weather` script.
   * `"w*"`: Allows calling all scripts starting with w, such as `weather`, `wiki`, etc.
2. **Negative Matching**: Prohibits AI from calling specified tool scripts. Negative matching rules must start with the prefix `!`. For example:
   * `"!search"`: Prohibits calling the `search` script.
3. **Priority**: Negative matching rules take precedence over positive matching rules.

### Dynamically Adjusting Generation Probability: ((!text:bias)) Syntax

To enable more precise control over the probability distribution of generated text, a new prompt syntax `((!text:bias))` has been introduced. This syntax dynamically adjusts the generation probability of specific tokens (via `logit-bias`) and can be flexibly applied in various scenarios, such as guiding the model to generate specific words, avoiding unwanted content, or adjusting the diversity of generated results.

#### Syntax Overview

* **Basic Format**: `((text:bias))`
  * `text`: The target word or phrase whose generation probability you want to adjust.
  * `bias`: The adjustment value, which can take several forms:
    * **Number**: Directly adds to the logits of the target token.
    * **Percentage** (e.g., 20%): Adjusts the generation probability proportionally. For instance, 20% means the generation probability will increase by 1.2 times the original probability.
    * `false`: Completely removes the possibility of generating that token.
* **Optional Prefix** `!`:
  * If the prefix ! is present, e.g., `((!text:bias))`, it indicates that the specified content should be removed from the prompt while applying the bias adjustment.

#### Use Cases

* **Guiding Content Generation**:
  * Example: `((happy:20%))` increases the probability of generating the word "happy" by 20%.
  * Application: Useful in scenarios where you want the model to generate text with a specific emotion or theme.
* **Avoiding Specific Content**:
  * Example: `((!sensitive_word:false))` completely prohibits the generation of the specified sensitive word.
  * Application: Helps filter out inappropriate or undesirable content.
* **Dynamically Adjusting Generation Diversity**:
  * Example: `((creative:50%))` boosts the probability of generating the word "creative" by 50%, encouraging the model to produce more creative content.
  * Application: Ideal for tasks requiring enhanced diversity in the generated text.

#### Example Code

Here’s a complete example using the ((!text:bias)) syntax:

```yaml
---
description: "You are a helpful assistant that generates text with controlled bias."
---
user: "Please generate a paragraph describing a happy scene."
assistant: "((happy:20%)) Let's imagine a sunny day where children are running on the grass, and laughter echoes in the air."
```

In this example, `((happy:20%))` increases the likelihood of the word "happy," making the generated text more inclined to express joyful emotions.

Notes

* **Moderation Required**: Excessive bias adjustments may overly skew the output toward certain words, affecting the naturalness and diversity of the text.
* **Context Dependency**: The effect of bias may vary depending on the context. It’s recommended to test and optimize based on specific use cases.
* **Compatibility**: This syntax currently only works with built-in LLM provider.

By introducing the `((!text:bias))` syntax, we provide a powerful mechanism to fine-tune the output behavior of large language models, meeting the nuanced control requirements of different tasks and needs.

## Specifications

### PPE Script Package

The package name of an intelligent agent script package is the same as the directory name. The root directory of the package must contain a script file with the same name as the directory, which serves as the package entry script.

For example, if there is a package directory named `a-dir`, the entry script in that directory should be named `a-dir/a-dir.ai.yaml`.

The functions exported by the package are determined by the `export` configuration in the entry file.

```yaml
---
export:
  - "$hi"
  - "./dobby.ai.yaml"
---
!fn |-
  [js]hi() {console.log('hi')}
```

### Front-Matter Configuration Specifications

Use [front-matter](https://jekyllrb.com/docs/front-matter/) for configuration.
`front-matter` must be at the front of the file, the first line starts with `---`, and the configuration ends with `---`.

Configuration includes: basic configuration of prompt project, prompt configuration, model parameter configuration, input and output and input default value configuration
For details on input and output and input default value configuration, please refer to the above.

#### Basic Configuration

```yaml
---
_id: Needless to say, the unique identification of the script
type: script type, `char` represents the role type; `type` indicates that the script itself is a type, with `_id` being the type name.
title: The title of the script(single-line)
description: multi-lines description of the script
templateFormat: "The template format of this script, by default: `hf`, which is the jinja2 template format used by huggingface; `golang` is also the template type used by `ollama` and `localai`; `fstring` is also used by `langchain`."
contentType: Ignore, all here are `script`
modelPattern: Models supported by this script, through matching rules
extends: Which prompt template is extended from
创: Creator related information
签: The signature of this script
---
```

##### Import

The `import` configuration to import functions and declarations in other script file

Import one file:

```yaml
---
import: "js:js_package_name" # the js npm package name
---
```

Import many files Use Array Format:

```yaml
---
import:
  - "js:js_package_name" # the js npm package name
  - "js/script/path.js": ['func1', 'func2', {func3: 'asFunc3'}] # Import only the specified functions
  - 'ruby-funcs.rb' # ruby file
  - 'rb:ruby_package'
  - "agent.ai.yaml": "asName" # Import the script and rename it to "$asName"
---
```

Use Object Format:

```yaml
---
import: # Object Format
  "js_package_name": "*"
  "js/script/path.js": ['func1', 'func2']
  "my.ai.wasm": "*"
  "agent.ai.yaml": "asName"
---
```

**Note**:

* Currently, only `javascript`, `wasm` and `AI` script support have been implemented.
* **BROKEN CHANGE**: ~~the default is js module if not extension name provided.~~ use the prefix `js:` to specify the js module name. For example, `js:js_package_name`.
* The relative path is the folder of the current ai script, not the CWD(current working dir)
* When importing, a "$" prefix will be automatically added to names without a prefix.
* If no function names are specified during import, all functions will be imported by default. For example, `js:path`, then all path functions will be imported into the current script (functions with the same name that already exist will not be imported), and can be used in the script directly via `$basename(...)`.
* During import, you can use `*` to indicate importing onto the package name object. For example, `"js:path": "* "`, then in the current script, use it through `$path.basename(...)`
  * You can specify specific names to replace `*`. For example, `"js:path": "aPath"`, then in the current script, use it through `$aPath.basename(...)`
* If the module contains a function `$initializeModule` and it is imported, this function will be executed automatically after the module is loaded.
  * Exception for packages(directories): The entry script will always be executed as `$initializeModule` automatically after the module is loaded.
  * To disable this behavior, you must manually set `$initializeModule: {disable: true}` when importing the package(directory).
  * If you want to disable this behavior within the package itself, then set `$initializeModule: false` in the entry script.
  * Additionally, directly referencing a file within the package will also skip `$initializeModule`.
    * Example: `ai:package_name#./some.ai.js`
    * If you want the entry script of the package to be executed during initialization, ensure that `some.ai.js` is exported via the entry script:
      * Example: `ai:package_name: ['some']`
* By default, importing an AI script will at least import the `$[AI_ID](data)` function object, which is used to execute the AI script itself. On this function object, there is also the `$[AI_ID].interact({message})` AI interaction function; as well as the items exported via the `export` configuration in the AI script.
* Added the `ai:` prefix convention, indicating the import of a AI script package or directory. The script package can include AI scripts, `.ai.js` files, and `.ai.wasm` files. For example, `ai:package_path#id.ai.yaml`, `ai:package_path#./some.ai.js`.

##### Export

If this parameter is not specified, the script will export a command named `$[id]`.

The `export` array specifies the functions that the script needs to export. It can export internal commands or external scripts.

```yaml
---
export:
   # Internal custom directive
  - "$internalDirectiveName"
  # Export $internalDirectiveName as $asName
  - "$internalDirectiveName": "asName"
  # Export two functions from the js:path module
  - "js:path": ['basename', 'extname']
  # Export the script itself, named `$[id]`
  - "."
---
!fn |-
  [js]function internalDirectiveName() {}
```

Note:

* When the script contains an `export`, the script itself will be executed as an initialization function (`$initializeModule`) upon import by default, unless there is a `$initializeModule` item in the script:
  * Setting `$initializeModule` to `false` will prevent the execution of the initialization function, or decalare the `$initializeModule` initialization directive by yourself.

###### Module Initialization Method `$initializeModule`

The this passed to `$initializeModule` refers to different object depending on the context:

* If the `$initializeModule` function is **from a JavaScript (JS) script**, the `this` will refer to the `caller` object (ensuring compatibility with JS behavior).
* If the `$initializeModule` function is **from an AI script**, the `this` will refer to the AI script itself, treating the script as an independent object.

The Arguments for `$initializeModule` include:

* `data`: This is the `data` object passed to the `caller`, or the default `data` object of the `caller` if no specific data is provided.
* `caller`: This represents the caller object itself.

Notes:

* There’s a configuration option in the AI script called `runSelfBeforeInitModule`, which determines whether the AI script should execute itself before running `$initializeModule`. By default, this is set to `false`.

#### Prompt configuration

```yaml
prompt:
  stop_words: ['\n'] # Custom stop words
  add_generation_prompt: true # Defaults to true. When set to `true`, if the last prompt message is not the `assistant` role, an empty `assistant` message will be automatically added to ensure the continuity of the conversation.
  messages: # You can also configure prompt messages here
    - role: system
      content: Carefully Think about the intent of following The CONVERSATION user provided. Output the json object with the Intent Category and Reason.
completion_delimiter: ''  # Optional parameter, a marker indicating the end of output in the prompt. If used, this delimiter is automatically added to stop_words. Default is none.
```

#### Model parameter configuration

```yaml
parameters:
  stop_words: ['\n'] # Custom stop words can also be defined in the parameters.
  max_tokens: 512 # Not too big, not too small, 512 is recommended, default is 2048, the use is when the model response is infinite and cannot be stopped, this can control the maximum length of tokens returned by the large model.
  continueOnLengthLimit: true
  maxRetry: 7 # When the response of the large model is incomplete, due to the limit of max_tokens, this is the number of times LLM is automatically executed again, the default is 7 times.
  stream: true # It is to enable the large model streaming response by default, higher than llmStream priority.
  timeout: 30000 # Set the response timeout to 30 seconds (in ms), if not set, the default is 120 seconds.
  response_format:
    type: json_object
    conversionScope: 'message' # Default is 'message' means the JSON to be converted includes the entire assistant message content; 'response', indicating that the JSON to be converted only includes the AI response.
  minTailRepeatCount: 7 # Minimum number of tail repetitions, default is 7, For stream mode only, when the tail sequence returned by the large model response is detected to be repeated 4 times in a row, the response will stop. Set to 0 for no detection.
  SystemTemplate: 'qwen2.5' # If not set, it will automatically detect and use the system template corresponding to the model. Setting it to a string forces the use of the system template for that model. You can also directly specify the system template via an object: `id` is the template ID; `version` is optional, representing the feature version of the template.
llmStream: true # Default true, Enable streaming response for large models. Note that some backends may not support streaming response.
autoRunLLMIfPromptAvailable: true # Default is true, which means that when there is a prompt message in the script and no `$AI` is called until the end of the script, the script will automatically execute `$AI` at the end
forceJson: null # Default is null, indicating whether to force the output of json object, which is automatically determined by `response_format.type` and `output`: when both of them exist at the same time, the output is forced to be json.
shouldAppendResponse: null  # Default is null, indicating whether the large model return result should be prompted by adding an assistant role or appended to the last message.
                            # If not set, the engine will automatically determine whether to add a new message
disableLlmRequest: false    # Default is false, whether to disable the `llmRequest` event
```

**Note**:

* The priority of parameters from high to low is: call parameters, `prompt` object, `parameters` object.
* When large model streaming response is enabled, you can receive partial results through the event `llmStream`.
* The parameters of the `llmStream` event handler are `(event, part: AIResult, content: string)`, `part` is the response object returned by the current large model, and `content` is the accumulation of the content in the response returned by the current large model.

```yaml
$on:
  event: llmStream
  callback: !fn |- # Anonymous function listener, event listener cannot be canceled
    (event, part, content) { const current_text = part.content }
```

##### Thought Mode Configuration (shouldThink)

`shouldThink` determines whether to enable the thought mode and configures the type of thought mode.

* **Type**: boolean, mode name, or mode configuration object
* **Description**: Determines whether to enable the thought mode and configures the type of thought mode.
  * true: Enables the default thought mode: last.
  * Supported thought modes:
    * `off`: Disables the thought mode.
    * `last`: Answers first, then thinks.
    * `first`: Thinks first, then answers.
    * `deep`: Thinks deeply before answering.
  * Mode configuration object:
    * `mode`: Type of thought mode (required).
    * `visible`: Whether to display the thought process (optional, default true).
    * `depth`: Maximum depth of thinking (optional).
    * `steps`: Maximum number of thinking steps (optional).
    * `thinkTag`: Tag for thought content. Used to mark text in the thinking process (optional).
      * Only available for the `first` and `deep` mode.
    * `answerTag`: Tag for answer content. Used to mark the final answer text (optional).
      * Only available for the `first` and `deep` mode.
* **Purpose**:
  * Control the AI's thinking behavior when processing requests to adapt to different application scenarios and needs.
  * Provide flexible configuration options so developers can adjust AI behavior according to specific requirements.
* **Behavior**:
  * When `shouldThink` is set to true or a mode name, the AI will execute the corresponding thinking logic based on the specified mode.
  * If a mode configuration object is provided, the AI will further refine its thinking behavior based on the parameters in the configuration object.
  * If `shouldThink` is set to `false` or `off`, the AI will not perform any additional thinking logic and will return results directly.

### String Prefix Conventions

* `~` prefix: indicates never format string, eg, "`~{{description}}`"
* `!` prefix: indicates immediate format string, eg, "`!{{description}}`". The old ~~`#`~~ deprecated since v0.9.0.
  * ~~`$` prefix: call command without parameters, eg, "`$AI`"~~ deprecated
  * ~~`$!` prefix: use the return value of the command without parameters as the message~~
* If the function return value message is a string, and the first character of the message is "#", it means to format the message immediately
* `?=` Prefix: indicates expression
  * If the expression result is a string and starts with "!", it means to format the expression result immediately
* `#:` or `#+` Prefix: replace or add message The old ~~`!:` or `!+` ~~ deprecated since v0.9.0.
  * When it is a message itself, the prefix `#` can be omitted.
  * `#[-1:role]Message`: replace the message. The index of the message can be specified in the square brackets. The default is the last message. If it is 0, the first message is replaced. If it is a negative number, the replacement starts from the last message, such as `[-1]` to replace the last message
    * The role parameter can be omitted. Omitting it means keeping the role unchanged. `#:[-1]Message`
    * Square brackets and numbers can be omitted, such as `#:Message`. After omitting, the last message is replaced
    * If it is `#:!Message`, it means to format the message immediately
  * `#+[-1:role]Message`: Add a message at the specified position. If the position is a negative number, it will be inserted from the last message. The position can be omitted. After omitting it, the message is added at the end. The message role is in the square brackets and can be set to `system`, `assistant`. The default is `user`. It can be omitted as: `#+Message`
    * If it is `#+!Message` Indicates to format the message immediately
* If the string does not contain the above prefix, or there is a formatting problem, it is considered as a new message for the user role.

#### Expression

`?=<expression>`

```yaml
$echo: ?=23+5
```

### Advance Command convention

#### `$prompt` command to set prompt parameters

Use `$prompt` to define prompt parameters for use in prompt templates.

```yaml
- $prompt:
add_generation_prompt: true # default is true
```

* `add_generation_prompt`: When set to `true`, if the last prompt message is not for the `assistant` role, an empty `assistant` message will be automatically added to ensure the continuity of the conversation.

#### `$parameters` command to set model parameters

Use `$parameters` to set model parameters or define them in `FRONT-MATTER`.

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

Other common model parameters are as follows:

* `temperature` is a floating point number between 0 and positive infinity that adjusts the smoothness of the sampled probability distribution. In the context of language models, it affects the selection process of the next word.
  * Low temperature (close to 0): The text generated by the model will be more conservative and predictable. At this time, the model tends to choose the words with the highest probability, and the generated text will be more fluent and regular, but may lack creativity or diversity.
  * High temperature: Increasing the `temperature` value will make the model more inclined to explore those words with lower probability, and the generated text will be more diverse and novel, but it may also be more discrete, difficult to understand, and even semantically jump.
* `continueOnLengthLimit`: This is used to determine whether AI will continue to be called automatically and continue to retrieve data after reaching the maximum token limit
* Note that this is not currently applicable when the return result is json. If you require that the returned json must be retrieved at once, increase `max_tokens`
* `maxRetry`: This parameter is also matched with `continueOnLengthLimit`, which is the maximum number of retries. If not set, the default is 7 times
* `timeout`: If the brain is big and the response is slow, and it takes more than 2 minutes to respond, then you need to adjust this timeout parameter, the unit is milliseconds
* `max_tokens`: This is the maximum token limit, the default is 2048, AI will output until max_tokens stops, which will avoid sometimes AI outputting infinitely and can't stop.
* `response_format`: Set the format of the returned result. Currently, only json (alias `json_object`) can be set for `type`.
  * Note: When `output` and `type:json` are set at the same time, the model will be forced to return json object instead of text.
  * If `response_format` is not set, you can set `forceJson:true` in the call parameters to achieve the same effect.

#### `$tool` tool directive

Use the `$tool` directive to use all registered tools.

##### `$AI` directive

`$AI` is an alias for `$tool:llm`, which directly calls the large model tool. By default, the result is appended to `prompt.messages` as the `assistant` role message. You can turn off the append by setting `shouldAppendResponse:false`.

```yaml
$AI:
  max_tokens: 512
  temperature: 0.7
  stream: true # Defaults to true, you can also set llmStream in the configuration, streaming response
  pushMessage: true # Defaults to true, indicating that the result returned by the large model tool is appended to prompt.messages.
  shouldAppendResponse: null  # Only valid when pushMessage is true, default is undefined.
                              # When undefined/null, when `matchedResponse` or `add_generation_prompt` or no lastMsg.content will be appended, otherwise the body of the last message will be replaced
                              # When true, force an assistant message to be appended. When false, force the body of the last message to be replaced.
  aborter: ?= new AbortController() # If not set, use the engine system's AbortController.
$tool:
  name: llm # Equal to $AI
  ...       # Other named parameters
```

#### `$abort` directive

Manually stop the response of the large model, which will generate an abort exception.

```yaml
$AI
$abort
```

#### Pipeline directive

`$pipe` will pass the result of the previous command to the pipeline.

Pass to the next instruction, supports the abbreviation `$|func`

```yaml
- toolId: $tool
# The return result of the previous function is passed to `func1|print`. If pipe has no parameters, it is passed to the next array element. If the next element itself is an object, it is merged.
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

#### `!fn` define function instruction

Use `!fn` tag to define function

```yaml
!fn |-
  function func1 ({arg1, arg2}) {
  }
# The function keyword can be omitted:
!fn |-
  func1 ({arg1, arg2}) {
  }
```

The function body is `javascript`. In the definition function, `async require(moduleFilename)` can be used to load local esm js file in the format.

```yaml
!fn |-
  async myTool ({arg1, arg2}) {
  const tool = await require(__dirname + '/myTool.js')
  return tool.myTool({arg1, arg2})
  }
```

If you need to use other languages, you should specify the language:

```yaml
!fn |-
  [python] def func1(arg1, arg2):
    return arg1 + arg2
```

**Note**:

* `__dirname`: is the directory where the prompt script file is located.
* `__filename`: is the prompt script file path.
* In the function, you can use `this` to get all the methods of the current script's runtime.
* All custom functions must be referenced by `$`. For example, in the example above, `func1` is defined, so `$func1` must be used when calling:

  ```yaml
  $func1:
  arg1: 1
  arg2: 2
  ```

* Currently only supports JavaScript, planning to add support for Python, Ruby, etc.

#### `!fn#` defines template function instructions

`!fn#` uses custom tags Define template functions, which are functions that can be used in the default JinJa template.

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

#### `$exec` calls external script commands

Through the `$exec` command, you can interact with other agent scripts.

```yaml
$AI
$exec:
  # id: 'script id' # Only one of the script file name and id can be selected
  filename: json
  args: "?=LatestResult" # Pass the result of $AI to the json agent script through parameters.
```

Note: By default, the return result of external scripts will be cached. Caching can be disabled by setting the `memoized` parameter in `args` to `false`.

#### `$if` directive

`$if` directive supports conditional judgment

```yaml
$set:
  a: 1
- $if: "a == 1" # Expression judgment
  then: # then function
    $echo: Ok
  else: # "else function"
    $echo: Not OK

!fn |-
  isOk(ok) {return ok}
- $if:
    $isOK: true # function judgment
  then: # then function
    $echo: Ok
  else: # "else function"
    $echo: Not OK
```

#### `$match` directive

The `$match` instruction is used to perform multi-branch matching based on variables or the result of the previous operation. It supports various matching methods, including regular expression matching, key-value matching, exact matching, expression matching, range matching, ignore matching, object matching, etc.

Each match item must be preceded by a colon `:`, followed by the match item, with no spaces in between. The condition is passed as `COND__` to the execution part.

```yaml
# The `condition` is optional. If not provided, the last result is used as the condition.
# By default, `$match` executes in order and stops once it finds a matching pattern, without checking subsequent patterns.
# If the `allMatches` parameter is set to `true`, then all matching branches will be executed. The default is `false`.
# If the `parallel` parameter is set to `true`, then all matching branches will be executed in parallel. This is only meaningful when `allMatches` is `true`.
$match(condition[, allMatches=false]):
  # Regular expression matching
  :/RegEx/:
    - $echo: matched
  # Conditional comparison
  :> 12:
    - $echo: matched
  # Exact match, if the condition is a string or number
  :"string": # :123
    - $echo: matched
  # Expression matching, condition === 1 or condition == 2
  :1 || 2:
    - $echo: matched
  # Range matching, 1..5 represents the closed interval `[1..5]`, 1..<5 represents the half-open interval `[1,5)`, 1>..5 represents the half-open interval `(1, 5]`.
  :1..5:
    - $echo: matched
  # Ignore specific items matching, this matches arrays with the first and fourth items, meaning the array must have a length of 4, and the first item's value is assigned to `first`, and the fourth item's value is assigned to `last`
  ":['a,b', _, _, last]":
    - $echo: matched
  # Match a complete object
  ":{x='a', y=':1||2' }":
    - $echo: matched
  # Partial match object
  ":{x='a', ..}":
    - $echo: matched
  # Otherwise
  _ :
    - $echo: else matched
```

Sure, here is the translation of the provided content:

* `condition`: Optional. If not specified, the condition defaults to `LastResult`.
* `allMatches`: When enabled, it executes all matching branches, meaning all matched branch items will be executed. The default value is `false`.
* `parallel`: Indicates whether to execute all matching branches in parallel. This is only meaningful when `allMatches` is enabled. The default value is `false`.

#### `$while` directive

The `$while` directive is used to execute a block of code repeatedly as long as the given condition is true. Here is a simple example:

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

Explanation

* Condition Expression (`"i >= 0"`): This is the condition that must be true for the loop to continue executing.
* Loop Body (`do:`): This section contains the operations that are executed during each iteration of the loop.
* The `$break` directive is used to prematurely end a loop.
* The `$continue` directive is used to skip the current iteration of a loop and proceed directly to the next iteration.

Example Breakdown

In this example, the `$while` directive checks whether the variable `i` is greater than or equal to 0. If the condition is true, it executes the operations within the loop body: decrementing the value of `i` by 1. This process continues until `i` is no longer greater than or equal to 0.

Notes

* Ensure that the loop condition eventually changes; otherwise, it can lead to an infinite loop.
* The loop body can contain multiple operations, not just a single `$set` operation.

Using the `$while` directive, you can implement basic looping logic suitable for various iterative processing scenarios.

#### `$for` directive

The `$for` instruction is used to iterate over a list and execute a block of code. Here is a simple example:

```yaml
$for: 3 # Iterate over the numbers 1 to 3
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

* `as` can be omitted. it will default to: `value` will be assigned the current element of the loop, and `index` will be assigned the current index of the loop. `items` is the object to iterate over. If it is a numeric range, it should be `{start, end, step}`.
* Loop body (`do:`): This section contains the operations to be performed in each iteration of the loop.
* The `$break` instruction is used to prematurely end a loop.
* The `$continue` instruction is used to skip the current iteration of the loop and proceed directly to the next iteration.

#### $format directive

`$format` directive uses Jinja2 template to format the string. The message formatting also uses Jinja2 template, which is also the template format supported by HuggingFace large model.

```yaml
$format: "{{description}}"
$format:
  template: "{{description}}"
  data:
    description: "hello world"
  templateFormat: "hf" # default is hf, currently supports hf, which is jinja2 used by huggingface; `golang` is also the template type used by `ollama` and `localai`; `fstring` is also `langchain` is in use.
```

#### `$set/$get` variable operation instructions

Support key path.

```yaml
$set:
  testVar.a: 124
  var2: !fn (key) { return key + 'hi' }
$get:
  - testVar.a
  - var2
```

### Event convention

Highly programmable and event-driven prompt generation system, which allows users to dynamically control and customize the process of generating text by defining event listeners, triggers and corresponding callback functions. From the examples given, we can see several key features and advantages:

* Event-driven architecture: By providing functions such as `$on`, `$once`, `$emit` and `$off`, the system supports an event-based programming model, allowing developers to flexibly intervene and extend the behavior of the model in response to different life cycle stages or specific conditions.
* Flexibility and scalability: Users can not only register named functions as callbacks, but also use anonymous functions or expressions, providing a variety of programming interfaces to adapt to different usage scenarios and complexity requirements. This enhances the flexibility and scalability of the script.
* Detailed event type design: From `beforeCall`, `afterCall` to `llm`, `llmStream` and other events for large model interactions, the system covers all aspects from function calls, result processing to model interactions, fully reflecting the in-depth understanding and support of common requirements in large model applications.
* Integration and interaction optimization: Through the `llmRequest` event, the system can intelligently manage large model calls, support customizing the way to obtain model responses through event mechanisms, and provide disable options to adapt to different strategies. In addition, the loading and saving of chat records are also open through events, which is convenient for integration into external systems or data management.
* Clear API design: The sample document shows clear API usage methods and parameter descriptions, which is convenient for developers to quickly get started and apply in depth, reflecting good design philosophy and user experience considerations.

`$on` function supports event monitoring, `$once` function supports event monitoring once, `$emit` function supports triggering events

`$off` function supports canceling event monitoring

#### `$on` and `$once` event monitoring functions

Parameters are as follows:

- event: event name
- callback: callback function or expression

Function as callback function:

```yaml
!fn |-
  onTest (event, arg1) { return {...arg1, event: event.type}}
$on:
  event: test
  callback: onTest # Named function monitoring, event monitoring can be canceled
$once: # Automatically cancel event monitoring after triggering once
  event: test
  callback: !fn |- # Anonymous function monitoring, event monitoring cannot be canceled
    (event, arg1) { return {...arg1, event: event.type}}
$emit: # Trigger event
  event: test
  args:
    a: 1
    b: 2
$off:
  event: test
  callback: onTest
```

The expression is used as a callback function. The parameters in the expression are as follows:

- event: event instance
- event.type: event name
- event.target: event source, that is, the current script runtime
- arg1: the first parameter value passed to the event listener function
- args: the remaining parameter value list passed to the event listener function, if any

These parameters are equivalent to the callback function: `(event, arg1, ...args) => void|any`

```yaml
$on:
  event: test
  callback: "?={...arg1, event: event.type}" # Unable to cancel event listening
```

#### `$emit` triggers event function

The parameters are as follows:

- event: event type, string, such as: `test`
- args: parameter value or parameter value list passed to the event listener function, if any

```yaml
$emit:
  event: test
  args: # an object parameter
    a: 1
    b: 2
$emit:
  event: test
  args: # indicates two object parameters
    - a: 1
    - b: 2
```

#### Script event type

- `beforeCall`: triggered before the function is called
- callback parameters: `(event, name, params, fn) => void|params`
- When the callback function returns a value, it means to modify the parameters.
- `afterCall`: triggered before the function returns the result
- callback parameters: `(event, name, params, result, fn) => void|result`
- When the callback function returns a value, it means to modify the return result.
- `llmParams`: Triggered before before the LLM is called and can be used to modify the parameters passed to the LLM.
  * Callback: `(event, params: {value: AIMessage[], options?: any, model?: string, count?: number}) => void|result<{value: AIMessage[], options?: any, model?: string, count?: number}>`
  * `value`: The messages to be sent to the LLM.
  * `options`: The options passed to the LLM.
  * `model`: The LLM name to be used.
  * `count`: the retry count if any.
- `llmBefore`: Triggered before before the LLM is called and can not modify the parameters, only used as notification.
  * Callback: `(event, params: any) => void`
- `llm`: the event is triggered before the large model returns the result, used to modify the large model return result.
- callback parameters: `(event, result: string) => void|result<string>`
- `llmStream`: triggered when the large model returns the result in streaming mode
- callback parameters: `(event, chunk: AIResult, content: string, retryCount: number) => void`
- chunk: current stream chunk content
- content: string content of all chunks currently obtained
- retryCount: number of retries for automatically calling llm when `max_token` is reached
- `llmRequest`: event is triggered when the large model result is needed, used to call the large model through the event and get the large model result. `[[RESPONSE]]` template will trigger this event
- callback parameters: `(event, messages: AIChatMessage[], options?) => void|result<string>`
- use the switch `disableLlmRequest: true` to disable this event.
- `ready`: triggered after the script interaction is ready, you can force the setting of whether it is in the ready state through the `$ready()` function.
- callback parameters: `(event, isReady: boolean) => void`
- `load-chats`: triggered when loading chat records.
- callback parameters: `(event, filename: string) => AIChatMessage[]|void`
- when the callback function returns a value, It means the loaded chat history.
- `save-chats`: Triggered when the chat history is saved.
- Callback parameters: `(event, messages: AIChatMessage[], filename?: string) => void`

**Note**:

* The `event` parameter in the event callback is the `Event` object, and `this` is the script runtime;
* When the event callback returns a value, it means modifying the parameter or result, otherwise it is not modified; the premise is that the event type supports modification;

## Refs

* [AutoGen](https://github.com/microsoft/autogen)
* [LMQL](https://lmql.ai/)
* [OpenInterpreter](https://github.com/OpenInterpreter/open-interpreter)
* [Outlines](https://github.com/outlines-dev/outlines)
* [MemGPT](https://github.com/cpacker/MemGPT/)
* [LangChain](https://github.com/langchain-ai/langchain)
