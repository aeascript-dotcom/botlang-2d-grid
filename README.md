# BotLang 2D Grid

**A Chinese-character-based token compression encoding for LLM data transmission**

Created by **Eddy Suphakorn** — February 2026

---

## What is BotLang 2D Grid?

BotLang 2D Grid is a data encoding method that compresses business data (customer records, CRM data, etc.) into a compact 2D grid format using single Chinese characters, designed to minimize token usage when sending data to Large Language Models (LLMs).

Instead of sending verbose JSON objects, BotLang 2D Grid represents each data field with a single Chinese character and arranges records in a pipe-separated (`|`) grid — like a compressed spreadsheet in plain text.

## The Problem

Sending business data to LLMs via API is expensive because of token costs. A typical JSON customer record uses ~120 tokens:

```json
{
  "customer": "สมชาย",
  "status": "กำลังดำเนินการ",
  "progress": 50,
  "satisfaction": "พอใจ",
  "score": 2,
  "type": "บ้าน",
  "budget": 120
}
```

## The Solution

BotLang 2D Grid encodes the same data in ~8 tokens:

```
陈|进|50|满|2|房|120
```

### How it works

**Dimension 1 (Horizontal → Columns):** Each pipe-separated field represents a data attribute.

```
陈 | 进 | 50 | 满 | 2  | 房  | 120
Name Status  %  Satisfaction Score Type Budget
```

**Dimension 2 (Vertical → Rows):** Each line represents one record (e.g., one customer).

```
名|状|%|意|分|类|额       ← Header (field names)
陈|进|50|满|2|房|120      ← Customer 1
林|完|100|很|5|车|80      ← Customer 2
王|等|0|无|0|房|200       ← Customer 3
```

## Key Design Principles

1. **Chinese characters as compression** — A single Chinese character (e.g., 进 = "in progress") replaces a multi-word Thai/English phrase, using only ~0.7 tokens instead of 3-5 tokens.

2. **No emoji** — Emoji cost ~2 tokens each on most LLM tokenizers. Chinese characters carry equal semantic meaning at 1/3 the token cost.

3. **Single mapping dictionary** — Only one Thai↔Chinese dictionary is needed. No separate emoji table required.

4. **LLM-native readability** — LLMs (especially DeepSeek, Qwen, etc.) are pre-trained on Chinese text and understand character meanings without extra legend explanation.

## Token Savings

| Format | Tokens per record | Savings |
|--------|------------------|---------|
| JSON (Thai) | ~120 tokens | baseline |
| JSON (English) | ~80 tokens | 33% |
| BotLang 2D Grid (Chinese) | ~8 tokens | **93%** |

## Recommended Stack

```
Language:    Chinese (pure Chinese characters, no emoji)
Encoding:    BotLang 2D Grid
Model:       DeepSeek V3.2 (or any Chinese-optimized LLM)
Caching:     Prompt caching for header/legend (90% discount)
Mapping:     Single Thai↔Chinese dictionary
```

## Example: Full Implementation

### Dictionary (Thai → Chinese)

| Thai | Chinese | Meaning |
|------|---------|---------|
| สมชาย | 陈 | Customer name |
| กำลังดำเนินการ | 进 | In progress |
| เสร็จแล้ว | 完 | Completed |
| รอดำเนินการ | 等 | Waiting |
| พอใจ | 满 | Satisfied |
| พอใจมาก | 很 | Very satisfied |
| ไม่มีข้อมูล | 无 | No data |
| บ้าน | 房 | House |
| รถ | 车 | Car |

### Encoded Data

```
名|状|%|意|分|类|额
陈|进|50|满|2|房|120
林|完|100|很|5|车|80
王|等|0|无|0|房|200
```

### How LLM Reads It

The header row tells the LLM what each column means. The LLM can then process hundreds of records efficiently, answering questions like:
- "Which customers are still in progress?"
- "What's the average satisfaction score?"
- "List all house-type customers with budget over 100."

## Origin

This encoding method was conceived by **Eddy Suphakorn** in February 2026 during conversations about optimizing LLM API costs for Thai business applications. The original idea was to arrange data in a 2D grid format (like a chessboard) and use Chinese characters for maximum token efficiency.

As of February 2026, no similar approach combining Chinese character compression with pipe-separated 2D grid encoding for LLM business data transmission exists in published literature or open-source projects.

## License

MIT License — Free to use with attribution to Eddy Suphakorn as the original creator.

---

*"Simple, cheap, and easy to maintain."* — Eddy Suphakorn
