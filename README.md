# pot-app-translate-plugin-SiliconFlow

一个 [Pot](https://pot-app.com/) 划词翻译外部插件，接入 [硅基流动 SiliconFlow](https://siliconflow.cn) 平台的**免费**翻译模型：

| 模型 | 特点 |
| --- | --- |
| `tencent/Hunyuan-MT-7B` | 腾讯混元翻译专用模型（WMT25 多语种第一），句子 / 段落翻译质量高、速度快 |
| `Qwen/Qwen2.5-7B-Instruct` | 通用指令模型，擅长按指令做词典式结构化输出 |

## 功能特性

- **智能模式（默认）**：自动识别待翻译内容——
  - **单词 / 短语 / 固定搭配**（如 `hello`、`break down`、`画蛇添足`）：**并行请求两个模型**——混元几秒内先给出快速译文，Qwen2.5-7B 输出结构化词典，就绪后以 **pot 原生词典卡片**自动替换（与内置 ecdict / 必应词典效果一致）：音标（英语词条同时给美式 us 和英式 uk 的 IPA；日文给假名；中文给拼音）、分词性释义、复数 / 过去式等屈折变化、例句对照；
  - **短句 / 长句 / 段落**：调用 Hunyuan-MT-7B 直接给出流畅译文。
- **双模型对照**：同时调用两个模型，句子结果并列展示；单词的词典结果**任一先就绪即显示**；单个模型失败不影响另一个。
- **仅混元 / 仅 Qwen**：固定使用其中一个模型。
- **自定义 Prompt**：词典模式 Prompt、句子翻译 Prompt、System Prompt 均可在服务配置中修改，留空则使用内置默认值。
- 支持自定义 API 地址（默认官方 `https://api.siliconflow.cn/v1/chat/completions`），方便接入兼容接口或代理。
- 词典 JSON 输出解析失败（含字段全空）时，自动改用**纯文本词典格式重试一次**，保证不会把原始 JSON 展示给用户。

> 关于速度：免费档 7B 模型的生成速度有限（约 30~50 token/s），词典 JSON 内容较多，完整卡片通常需要数秒到十余秒——这是模型输出速度的硬限制。插件已通过"快速译文先行 + 词典卡片后到"、压缩输出要求、词典请求低温（0.3）等方式尽量缩短等待。

## 安装

1. 从本仓库 [Releases](https://github.com/LargeNumberZZ/pot-app-translate-plugin-SiliconFlow/releases)（或 Actions 构建产物）下载 `plugin.com.pot-app.siliconflow.potext`；
2. 双击安装，或在 Pot 的 偏好设置 → 服务设置 中导入；
3. 添加 "硅基流动 SiliconFlow" 翻译服务并填写 API Key。

API Key 在 [硅基流动控制台](https://cloud.siliconflow.cn) 注册后即可免费获取。

## 配置项

| 配置 | 说明 |
| --- | --- |
| API Key | 硅基流动 API Key（必填） |
| 翻译模式 | 智能模式（默认）/ 双模型对照 / 仅 Hunyuan-MT-7B / 仅 Qwen2.5-7B |
| 词典模式 | 自动判断（默认）/ 总是按词典格式输出 / 总是直接输出译文 |
| 词典模式 Prompt | 单词 / 短语使用的 Prompt，留空用默认 |
| 句子翻译 Prompt | 句子 / 段落使用的 Prompt，留空用默认 |
| System Prompt | 系统消息，留空用默认 |
| API 地址 | 留空用官方默认，可指向兼容 OpenAI Chat API 的地址 |

## 自定义 Prompt

通过自定义 Prompt 自定义 AI 行为，`$text` `$from` `$to` `$detect` 将会被替换为待翻译文本、源语言、目标语言和检测到的语言（与 Pot 内置 AI 服务一致）。语言会被替换为英文名称（如 `zh_cn` → `Simplified Chinese`），便于模型理解。

例如，想让句子翻译总是输出更书面化的结果，可以把"句子翻译 Prompt"改为：

```text
请将下面的内容从 $from 翻译成 $to，使用正式书面语体，只输出译文：
"""
$text
"""
```

> 提示：
> - 如果希望所有输入都按词典格式（或都直接翻译）输出，把"词典模式"设为"总是词典"或"总是直接翻译"即可，无需修改 Prompt。
> - 默认的"词典模式 Prompt"把待查词条放在最前面（`<<< >>>` 包裹）并要求输出 pot 词典 JSON 结构；若你在旧版本自定义过该 Prompt，建议清空以使用新的默认值（新版对 7B 小模型的遵循度更好）。解析不出词典 JSON 时，插件会自动用内置的纯文本词典格式重试一次。

## 词典模式判定规则（自动判断时）

- 文本含换行，或以 `.。!！?？;；:：` 等句末标点结尾 → 按句子翻译；
- 以中日韩文字为主：4 字及以内 → 词 / 短语 / 成语，走词典模式；
- 其他语言：4 个单词及以内 → 词 / 短语，走词典模式；
- 其余情况 → 句子翻译。

## 开发

插件结构遵循 [pot-app 插件规范](https://github.com/pot-app/pot-desktop)，仅包含三个参与打包的文件：

- `info.json` —— 插件声明（id、语言映射、配置字段）
- `main.js` —— 翻译逻辑入口 `translate(text, from, to, options)`
- `siliconflow.svg` —— 图标

推送后 GitHub Actions 会自动打包 `info.json + siliconflow.svg + main.js` 为 `.potext`；打 tag（如 `v1.0.0`）时会自动发布到 Releases。

本地构建：

```bash
zip plugin.com.pot-app.siliconflow.potext info.json siliconflow.svg main.js
```

仓库外附带的 `run_tests.py` 是本地逻辑测试脚本（headless Edge 模拟 pot 的插件加载与请求，25 项断言），不影响插件打包：

```bash
python run_tests.py
```

## License

遵循上游模板的开源协议（见 [LICENSE](LICENSE)）。
