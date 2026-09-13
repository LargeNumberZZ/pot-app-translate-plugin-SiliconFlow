# pot-app-translate-plugin-SiliconFlow

[English](README.md)

一个 [Pot](https://pot-app.com/) 划词翻译外部插件，接入 [硅基流动 SiliconFlow](https://siliconflow.cn) 平台的**免费**模型：

| 模型 | 分工 |
| --- | --- |
| `Qwen/Qwen2.5-7B-Instruct` | 所有环节的主力：词典查询（词性标注准确）、详细解释、句子翻译 |
| `tencent/Hunyuan-MT-7B` | 所有环节的兜底 + 词典卡片生成期间的快速译文先行展示 |

## 功能特性

- **单词 / 短语 / 固定搭配**（如 `hello`、`break down`、`画蛇添足`）：极简词典卡片——英美音标 + 按词性分组的词义，由千问生成，以 **pot 原生词典卡片**渲染。卡片底部有灰色"详细解释"链接，点击后原地加载详尽版：每词性 2~4 条释义（含引申义）、3~5 条搭配、屈折变化、近义词/反义词、用法说明、两组例句对照，并带 **◂ 返回简明版**（瞬时，有缓存）与 **↻ 刷新**（强制重新获取）链接。千问失败时自动使用混元的行格式词典，且解析后有合并兜底，详尽版永远包含简明版的全部内容。
- **短句 / 长句 / 段落**：千问优先，混元兜底。Prompt 严格约束：只翻译——不用 markdown、不重排格式，分段换行与原文完全一致。
- **翻译模式**：千问优先、混元兜底（默认）；双模型对照（两栏并排实时流式）；仅混元；仅千问。
- **自定义 Prompt**：词典模式 Prompt、句子翻译 Prompt、System Prompt 均可在服务配置中修改，支持 `$text` `$from` `$to` `$detect` 占位符（与 pot 内置 AI 服务一致）。语言会替换为英文名（如 `zh_cn` → `Simplified Chinese`），便于模型理解。
- **自定义 API 地址**（默认 `https://api.siliconflow.cn/v1/chat/completions`），可指向兼容 OpenAI Chat API 的网关或代理。
- **稳定性**：优先走 webview 原生 fetch 真流式（文字边生成边出现，硅基流动 CORS 开放）；不可用时自动回退 tauriFetch 并以约 0.4 秒的快速打字动画展示。**请求带 20 秒超时**：超时若已有部分生成内容则直接截断返回，完全没有内容才报超时错误。词典输出经过去重加固（重复的音标/词性/联想/例句只保留一条），模型复读不会撑爆卡片；解析失败自动降级纯文本；详细加载失败时提供重试链接。

## 安装

1. 从 [Releases](https://github.com/LargeNumberZZ/pot-app-translate-plugin-SiliconFlow/releases)（或 Actions 构建产物）下载 `plugin.com.pot-app.siliconflow.potext`；
2. 双击安装，或在 Pot 的 偏好设置 → 服务设置 中导入；
3. 添加 "硅基流动 SiliconFlow" 翻译服务并填写 API Key。

API Key 在 [硅基流动控制台](https://cloud.siliconflow.cn) 注册后即可免费获取。也欢迎通过作者的邀请链接注册：**https://cloud.siliconflow.cn/i/fEgcWvLa**（这是作者的邀请码，自行注册也完全可以）。

## 配置项

| 配置 | 说明 |
| --- | --- |
| API Key | 硅基流动 API Key（必填） |
| 翻译模式 | 千问优先，混元兜底（默认）/ 双模型对照 / 仅混元 / 仅千问 |
| 词典模式 | 自动判断（默认）/ 总是词典格式 / 总是直接翻译 |
| 词典模式 Prompt | 单词/短语查询使用，留空用默认 |
| 句子翻译 Prompt | 句子/段落使用，留空用默认 |
| System Prompt | 系统消息，留空用默认 |
| API 地址 | 留空用官方默认，可指向兼容 OpenAI Chat API 的地址 |

## 自定义 Prompt

通过自定义 Prompt 自定义 AI 行为，`$text` `$from` `$to` `$detect` 将会被替换为待翻译文本、源语言、目标语言和检测到的语言（与 pot 内置 AI 服务一致）。语言会被替换为英文名（如 `zh_cn` → `Simplified Chinese`），便于模型理解。

例如，想让句子翻译使用正式语体：

```text
请将下面的内容从 $from 翻译成 $to，使用正式书面语体，只输出译文：
"""
$text
"""
```

> 提示：
> - 默认的"词典模式 Prompt"要求模型输出严格的 JSON 结构以获得词典卡片效果；若你自定义该 Prompt，输出将按纯文本展示（解析失败时插件会自动用内置的纯文本格式重试一次）。
> - 内置 Prompt 有意保持精简：简明卡片只含音标与词义，搭配/例句/近义词/用法说明由"详细解释"按钮按需加载。

## 词典模式判定规则（自动判断时）

- 文本含换行，或以 `.。!！?？;；:：` 等句末标点结尾 → 按句子翻译；
- 以中日韩文字为主：4 字及以内 → 词 / 短语 / 成语，走词典模式；
- 其他语言：4 个单词及以内 → 词 / 短语，走词典模式；
- 其余情况 → 句子翻译。

## 已知限制

- **API Key 在配置界面为明文显示**：pot 对外部插件的配置项只支持普通文本框和下拉框（没有密码框类型），遮蔽显示需要 pot 上游支持，可在 [pot-desktop](https://github.com/pot-app/pot-desktop/issues) 提功能建议。
- 服务配置中的下拉选项文字较短（界面不支持下拉悬停提示），各选项的详细行为说明以本 README 为准。
- 免费档 7B 模型的生成速度有限（约 30~50 token/s），详尽词典卡片与长段落翻译需要生成大量 token，完整输出通常需要数秒到十几秒——这是模型输出速度的硬限制。插件通过"快速译文先行 + 词典卡片后到"、真流式显示、精简输出、词典请求低温（0.1）等方式尽量缩短可感知的等待。

## 开发

插件遵循 [pot-app 插件规范](https://github.com/pot-app/pot-desktop)，参与打包的文件只有三个：

- `info.json` —— 插件声明（id、语言映射、配置字段）
- `main.js` —— 翻译逻辑入口 `translate(text, from, to, options)`
- `siliconflow.svg` —— 图标

推送后 GitHub Actions 会自动打包 `info.json + siliconflow.svg + main.js` 为 `.potext`；打 tag（如 `v1.2.0`）时会自动发布到 Releases。

本地构建：

```bash
zip plugin.com.pot-app.siliconflow.potext info.json siliconflow.svg main.js
```

## 许可证

遵循上游模板的开源协议（见 [LICENSE](LICENSE)）。
