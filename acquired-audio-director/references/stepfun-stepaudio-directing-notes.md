# StepFun StepAudio 2.5 TTS Directing Notes

> Extracted from: https://platform.stepfun.com/docs/zh/guides/models/stepaudio-2.5-tts
> Best practices: https://platform.stepfun.com/docs/zh/guides/developer/tts

## 模型概览

**StepAudio 2.5 TTS** — Contextual TTS，具备语境感知能力的语音合成模型。

核心卖点：不是念文本，而是**演文本**。

| 属性 | 值 |
|------|-----|
| 模型 ID | `stepaudio-2.5-tts` |
| 单次输入上限 | **1000 字符** |
| instruction 上限 | **200 字符**（全局语境自然语言指导） |
| API 兼容 | OpenAI TTS 格式 |
| 定价 | 5.8 元/万字符 |
| 音色复刻 | 9.9 元/音色，仅需 3s 参考音频 |
| 输出格式 | wav, mp3, flac, opus, pcm |
| 输出语言 | 中文、英文、中英混合、日语 |

## 核心能力：双档语境控制

### 1. Global Context（全局语境）— `instruction` 参数

定义**整段**表达基调。自然语言描述，不是固定标签。

```json
{
  "instruction": "声音极度紧绷，像在拼命压住快要失控的狂喜；语速快而断续，带明显的压抑感"
}
```

支持复合意图，例如：
- "克制的悲伤，不哭腔，轻轻发颤"
- "像在分享一个秘密，压低声音但藏不住兴奋"
- "专业播客主持风格，温暖但有分析深度"

**上限 200 字符。**

### 2. Inline Context（文中语境）— `()` 括号

在 `input` 文本中用圆括号 `()` 插入逐句精控指令。括号内文本**不会被朗读**，仅作为指令。

```json
{
  "input": "（压低声音）喂……你看我手机。（短促吸气）是不是我眼花了？（强装镇定）……算了，肯定是诈骗短信。"
}
```

支持的指令类型：
- 情绪：`（压抑的悲伤）`、`（极度兴奋）`
- 气息：`（短促吸气）`、`（长叹一口气）`
- 停顿：`（沉默三秒）`、`（停顿）`
- 语气转折：`（强装镇定）`、`（突然严肃）`
- 语速：`（放慢语速）`、`（加快）`
- 复合意图：`（克制但藏不住兴奋，声音微微发颤）`

**关键区别 vs ElevenLabs：**
- ElevenLabs 用 `[tag]` 前缀，StepFun 用 `(自然语言)` 内联
- ElevenLabs 标签是枚举集合，StepFun 支持任意自然语言描述
- StepFun 的 instruction 可以定义**全局基调**，ElevenLabs 没有
- StepFun 的 inline context 比 ElevenLabs 的 audio tags 更灵活

## API 调用

### 非流式合成

```bash
curl https://api.stepfun.com/v1/audio/speech \
  -H "Authorization: Bearer $STEP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "stepaudio-2.5-tts",
    "voice": "cixingnansheng",
    "input": "（压低声音）喂……你看我手机。（短促吸气）是不是我眼花了？",
    "instruction": "声音极度紧绷，像在拼命压住快要失控的狂喜",
    "speed": 1.0,
    "response_format": "mp3"
  }' \
  --output output.mp3
```

### 流式合成 (WebSocket)

```
WebSocket /v1/realtime/audio
```

低时延流式返回，适合对话与实时播放场景。

### 复刻试听

```bash
curl https://api.stepfun.com/v1/audio/voices/preview \
  -H "Authorization: Bearer $STEP_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "stepaudio-2.5-tts",
    "input": "试听文本",
    "voice": "reference_audio_id"
  }'
```

### 参数速查

| 参数 | 类型 | 说明 |
|------|------|------|
| `model` | string | `stepaudio-2.5-tts`, `step-tts-2`, `step-tts-mini` |
| `input` | string | 要合成的文本（含 `()` 内联指令），上限 1000 字符 |
| `voice` | string | 音色 ID，见下方音色表 |
| `instruction` | string | 全局语境自然语言指导，上限 200 字符 |
| `speed` | float | 语速：0.5-2.0（默认 1.0） |
| `volume` | float | 音量：0.1-2.0（默认 1.0） |
| `response_format` | string | `mp3`(默认), `wav`, `flac`, `opus`, `pcm` |
| `voice_label` | object | 标签控制：`{emotion, style}` |

## Voice Label 标签系统

除了 `instruction` 和 `()` 内联指令，还支持结构化标签：

### 情绪标签（voice_label.emotion）

| 标签 | 说明 |
|------|------|
| 高兴 | 基础高兴 |
| 非常高兴 | 强烈高兴 |
| 悲伤 | 基础悲伤 |
| 生气 | 基础生气 |
| 非常生气 | 强烈生气 |
| 撒娇 | 亲昵 |
| 恐惧 | 害怕 |
| 惊讶 | 惊异 |
| 兴奋 | 活力 |
| 钦佩 | 崇敬 |
| 困惑 | 不解 |

### 演绎风格标签（voice_label.style）

| 标签 | 说明 |
|------|------|
| 冷漠 | 疏离 |
| 尴尬 | 不自在 |
| 沮丧 | 低落 |
| 骄傲 | 自豪（仅 step-tts-2） |
| 温柔 | 柔和（仅 step-tts-2） |
| 甜美 | 可爱（仅 step-tts-2） |
| 豪爽 | 大气（仅 step-tts-2） |
| 严肃 | 庄重（仅 step-tts-2） |
| 傲慢 | 高高在上（仅 step-tts-2） |
| 老年 | 老年音色（仅 step-tts-2） |
| 吼叫 | 大声（仅 step-tts-2） |
| 阴阳怪气 | 讽刺（仅 step-tts-2） |
| 磕巴 | 结巴（仅 step-tts-2） |

### 语速标签（voice_label.style）

| 标签 | 说明 |
|------|------|
| 慢速 | 减速 |
| 极慢 | 大幅减速 |
| 快速 | 加速 |
| 极快 | 大幅加速 |

## 推荐音色（适合播客/有声书场景）

| 音色名 | Voice ID | 推荐场景 |
|--------|----------|---------|
| 磁性男声 | `cixingnansheng` | 有声书、情感陪伴 |
| 儒雅男士 | `ruyananshi` | 有声书、口播、语音助手 |
| 深沉男音 | `shenchennanyin` | 有声书、情感陪伴 |
| 温柔公子 | `wenrougongzi` | 有声书、情感陪伴 |
| 正派青年 | `zhengpaiqingnian` | 营销、有声书 |
| 播音男声 | `boyinnansheng` | 有声书、口播、新闻 |
| 自信男声 | `zixinnansheng` | 有声书、营销 |
| 温柔女声 | `wenrounvsheng` | 有声书、情感陪伴 |
| 经典女声 | `jingdiannvsheng` | 客服、情感陪伴 |

## 与 ElevenLabs v3 的对比

| 维度 | ElevenLabs v3 | StepAudio 2.5 TTS |
|------|---------------|-------------------|
| 控制方式 | `[tag]` 枚举标签 | `instruction` + `()` 自然语言 |
| 全局基调 | 无（stability 滑块） | `instruction` 200 字自然语言 |
| 局部控制 | `[excited] text` | `（激动地）文本` |
| 多 speaker | Text-to-Dialogue API | 无内置，需切分单独调用 |
| 单次上限 | ~2000 字符 | 1000 字符 |
| 音色复刻 | IVC/PVC | 3s 参考音频，9.9元/音色 |
| 中文质量 | 一般（英文为主） | **原生中文，质量极高** |
| 价格 | $0.18/1K chars (Creator) | ¥5.8/万字符 (~$0.08/1K chars) |
| SSML | v3 不支持 break | 不支持，用 `()` 指令替代 |

## Director Script 适配要点

### 对 Acquired 播客场景的独特优势

1. **instruction 就是天然的 beat-level 导演指令**：每个 chunk 可以有自己的全局基调
2. **`()` 内联指令比 `[tag]` 更精准**：可以写"（像发现了一个惊人秘密，压低声音但语速加快）"
3. **中文原生**：如果需要生成中文版 Acquired，StepFun 质量远超 ElevenLabs
4. **价格低**：长节目成本约为 ElevenLabs 的 40-50%

### 适配时的注意

1. **没有内置多 speaker**：每个 chunk 需要 ben_like 和 david_like **分开调用**，然后拼接
2. **1000 字符上限**：比 ElevenLabs 的 2000 短，需要切更小的 chunk
3. **括号冲突**：如果文本本身有圆括号内容，需要转义或改写
4. **voice_label 可搭配使用**：在 instruction + () 之外，还可以用 emotion/style 标签补充
