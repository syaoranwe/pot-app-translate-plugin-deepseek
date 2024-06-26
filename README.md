# Pot-App DeepSeek模型 翻译插件（OpenAI API格式接口）

## 介绍

1. 你可以自定义几乎所有的接口参数配置项，包括系统人设、提示词、采样温度、Top-P、惩罚因子等。
2. 由于Pot的插件不支持打印机流式效果，所以只支持获取到所有翻译内容后再一并显示。蹲一个官方更新，以支持插件流式输出。
3. 代码里写死了超时时间为50秒，最大回复Token长度为2K。如有需要可自行修改。
4. 对DeepSeek平台的审查机制特别新增了报错处理，如果你使用的是DeepSeek官方平台，当你的翻译请求包含敏感内容被审查时，会有相应的报错提示。

## 配置参数

### 获取API Key

DeepSeek官方平台：[DeepSeek开放平台](https://platform.deepseek.com/api_keys)

API Key：注册官方平台后在[这里](https://platform.deepseek.com/api_keys)生成。注意保存好，只会显示一次。

### 必填配置项
1. API Key：你的API Key，见上文获取方法。
2. 请求地址：如果你使用DeepSeek官方平台，那么就是`https://api.deepseek.com/chat/completions`
3. 模型名：如果你使用DeepSeek官方平台，那么目前只有两种选择：`deepseek-chat`, `deepseek-coder`。前者是通用聊天模型，后者是编程模型。如果你使用的是其他平台，那么请填写你的自定义模型名。

### 可选配置项
1. System人设：系统人设提示词字符串, 留空时默认为`You are a professional translation engine.`
2. 翻译提示词：用户自定义提示词列表, 由一行json字符串表示, 列表中元素值中的role有两种：user: 表示用户；assistant: 表示对话助手, content表示内容。content中的`$to$`会自动替换为译文语言描述, 例如`Traditional Chinese(繁體中文)`, `$src_text$`会自动被替换为原文文本。 如果留空则使用默认提示词（你可以在 json.cn 上编辑该提示词，之后压缩为一行即可导入程序）：
```text
[{"role":"user","content":"You are a professional translation engine, skilled in translating text into accurate, professional, fluent, and natural translations, avoiding mechanical literal translations like machine translation. You only translate the text without interpreting it. You only respond with the translated text and do not include any additional content."},{"role":"assistant","content":"OK, I will only translate the text content you provided, never interpret it."},{"role":"user","content":"Translate the text delimited by ``` below to Simplified Chinese(简体中文), only return translation:\n```\nHello, world!\n```\n"},{"role":"assistant","content":"你好，世界！"},{"role":"user","content":"Translate the text delimited by ``` below to English, only return translation:\n```\n再见，小明\n```\n"},{"role":"assistant","content":"Bye, Xiaoming."},{"role":"user","content":"Translate the text delimited by ``` below to $to$, only return translation:\n```\n$src_text$\n```\n"}]
```
3. temperature：留空时默认0.75, 范围 [0, 2.0], 值越小, 生成的内容越固定。当取0时，模型生成时几乎总是会选取概率最大的Token。越低的采样发散度模型将越倾向于使用机翻风格逐句翻译，越高的采样发散度模型的译文将越随机，可能导致译文丢失部分信息，也有可能会给出更流畅的译文。
4. top_p：留空时默认为0.95, 取值范围：(0, 1.0], 值越大, 生成的内容多样性越丰富，但在temperature较低的情况下越接近1，提升效果越趋近于没有。建议只调整temperature，不建议调整top_p。
5. presence_penalty：留空时默认为0, 取值范围：[-2.0, 2.0]. 值越高，模型给出下一个字词时，越不可能选择那些已经在原文和已生成的译文中重复过的字词。为负数时，模型会被鼓励生成原文和已生成译文中已有的字词。如果你希望模型给出译文时使用表达更丰富的词汇，可以适当增加该值，但这有可能生成更多疑难少用词汇。默认值设置为0，意味着既不鼓励模型沿用重复旧词，也不反对。翻译时建议略大于0，如0.1。
6. frequency_penalty：留空时默认为0, 取值范围：[-2.0, 2.0]. 值越高，模型在给出下一个字词时，越不可能选择那些已经在原文和已生成的译文中重复了多次的字词。为负数时，相当于鼓励模型多采用重复了多次的字词。默认值设置为0，意味着既不鼓励模型多次沿用重复旧词，也不反对。翻译时建议略大于0，如0.1。
