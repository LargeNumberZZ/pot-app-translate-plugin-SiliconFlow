# pot-app-translate-plugin-SiliconFlow

[中文说明](README.zh-CN.md)

A [Pot](https://pot-app.com/) translate plugin powered by the **free** models on [SiliconFlow](https://siliconflow.cn):

| Model | Role |
| --- | --- |
| `Qwen/Qwen2.5-7B-Instruct` | Primary for everything you see as the final result: the lean dictionary card, the detailed entry, sentence translation |
| `tencent/Hunyuan-MT-7B` | Fallback for all of the above |

## Features

- **Dictionary card for words and phrases**: UK & US IPA plus senses grouped by POS (Qwen). A "More details" link loads a richer entry on demand — more senses, collocations, inflections, synonyms and two example pairs — with instant back/refresh switching.
- **Sentence / paragraph translation** (Qwen, Hunyuan fallback): translate only — no markdown, no re-formatting; paragraphs and line breaks mirror the source exactly.
- **Streaming display with a 20 s timeout**: text appears as it is generated; on timeout the partial output is returned instead of erroring out.
- **Custom prompts**: dictionary / sentence / system prompts with `$text` `$from` `$to` `$detect` placeholders (same as Pot's built-in AI services); languages are expanded to English names (`zh_cn` → `Simplified Chinese`) for better model understanding.

## Installation

1. Download `plugin.com.pot-app.siliconflow.potext` from [Releases](https://github.com/LargeNumberZZ/pot-app-translate-plugin-SiliconFlow/releases) (or the Actions build artifacts);
2. Install it in Pot via Preferences → Service Settings;
3. Add the "硅基流动 SiliconFlow" service and fill in your API Key.

Get a free API key by registering on the [SiliconFlow platform](https://cloud.siliconflow.cn). You may also sign up via the author's invite link: https://cloud.siliconflow.cn/i/fEgcWvLa

## Configuration

| Option | Description |
| --- | --- |
| API Key | Your SiliconFlow API key (required) |
| Mode | Default (Qwen-first) / Dual comparison / Hunyuan-only / Qwen-only |
| Dictionary mode | Auto (default) / Always dictionary / Always plain translation |
| Dictionary prompt | Used for word/phrase lookups; empty = built-in default |
| Sentence prompt | Used for sentences/paragraphs; empty = built-in default |
| System prompt | System message; empty = built-in default |
| API URL | Empty = official endpoint; can point to any OpenAI-compatible Chat API |

## Custom prompts

Customize the AI behavior through the prompts; `$text`, `$from`, `$to` and `$detect` are replaced with the source text, source language, target language and detected language (same as Pot's built-in AI services). Languages are expanded to English names (`zh_cn` → `Simplified Chinese`) for better model understanding.

Example — a more formal sentence translation:

```text
Translate the following from $from to $to in a formal register. Output the translation only:
"""
$text
"""
```

> Notes:
> - The default dictionary prompt asks the model for a strict JSON structure so Pot renders the rich dictionary card; if you supply your own, output falls back to plain text (a parse failure triggers one built-in plain-text retry).
> - The built-in prompts are intentionally lean: the simple card contains IPA + senses only, while collocations/examples/synonyms/usage are loaded on demand via the More details button.

## How entries are classified (dictionary mode = Auto)

- Text containing line breaks, or ending with `.。!！?？;；:：` → sentence translation;
- Mostly CJK: 4 characters or fewer → word/phrase/idiom, dictionary mode;
- Otherwise: 4 words or fewer → word/phrase, dictionary mode;
- Everything else → sentence translation.

## Known limitations

- **The API key is shown in plain text** in the service settings: Pot renders external-plugin options as plain inputs and dropdowns only (no password field type). Masking requires upstream support — consider filing a feature request at [pot-desktop](https://github.com/pot-app/pot-desktop/issues).
- The dropdown labels in the service settings are intentionally short (no hover tooltips for external plugin configs); see this README for details.
- Generation speed of the free 7B models is limited (~30-50 tokens/s). Rich dictionary cards and long paragraphs can take several seconds — that is a hard limit of the models themselves. The plugin mitigates it with the quick-translation preview, real streaming, lean outputs and a low dictionary temperature (0.1).

## Development

The plugin follows the [pot-app plugin spec](https://github.com/pot-app/pot-desktop) and ships three files:

- `info.json` — plugin manifest (id, language mapping, config fields)
- `main.js` — entry point `translate(text, from, to, options)`
- `siliconflow.svg` — icon

Pushing to GitHub runs GitHub Actions, which packages `info.json + siliconflow.svg + main.js` into a `.potext`; pushing a tag (e.g. `v1.2.0`) also publishes a release.

Build locally:

```bash
zip plugin.com.pot-app.siliconflow.potext info.json siliconflow.svg main.js
```

## License

Follows the upstream template license (see [LICENSE](LICENSE)).
