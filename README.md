# SRT Notebooks — KIENDOANTTS

> **Colab notebooks chuyển phụ đề SRT thành giọng đọc AI bằng OmniVoice TTS.**
> Public mirror của `doanquangkien/srt` (private). Push tự động lên GitHub → Colab load trực tiếp.

---

## Cách hoạt động

```
Người dùng paste SRT → notebook → model.generate() từng dòng → ffmpeg adelay → amix → MP3
```

### Pipeline

1. **Parse SRT** — regex tách `index, start, end, text`
2. **TTS tuần tự** — `model.generate(text=line['text'], voice_clone_prompt=..., language='vi', speed=0.95)`
3. **ffmpeg adelay** — `aresample + aformat + adelay={delay_ms}` — đặt audio vào timeline
4. **Pairwise amix** — mix 2 file/lần để tránh OOM trên T4
5. **WAV → MP3** — 128kbps
6. **EBU R128** — tùy chọn checkbox

### Nguyên tắc

- **KHÔNG trim** audio — mỗi segment là 1 lần `model.generate()` độc lập, giọng đọc tự nhiên
- **KHÔNG warmup skip** — không cần vì mỗi segment tách biệt
- **Sequential, không batch** — batch mode đã thử và thất bại (OOM + chậm hơn trên T4)
- **1 giọng = 1 notebook** — tách biệt hoàn toàn, không share code

---

## Cách thêm giọng mới

Tất cả notebook **giống hệt nhau**, khác đúng 3 dòng trong Cell 1. Dùng script:

```python
import json

# 1. Đọc notebook Nam Trầm Ấm (template)
with open('colab_srt-to-voice.ipynb', 'r', encoding='utf-8') as f:
    nb = json.load(f)

# 2. Replace voice slug + display name
SLUG_FROM = "nam-tram-am"
SLUG_TO   = "ten-giong-moi"
NAME_FROM = "Nam tram am"
NAME_TO   = "Ten giong moi"

for cell in nb['cells']:
    if isinstance(cell['source'], str):
        cell['source'] = cell['source'].replace(SLUG_FROM, SLUG_TO).replace(NAME_FROM, NAME_TO)
    elif isinstance(cell['source'], list):
        cell['source'] = [s.replace(SLUG_FROM, SLUG_TO).replace(NAME_FROM, NAME_TO) for s in cell['source']]

# 3. Ghi file mới
out = f'colab_srt-to-voice-{SLUG_TO}.ipynb'
with open(out, 'w', encoding='utf-8') as f:
    json.dump(nb, f, ensure_ascii=False, indent=1)

print(f'Created: {out}')
```

### Voice slugs & names

| Slug | Display Name | Sample URL |
|------|-------------|------------|
| `nam-tram-am` | Nam tram am | `.../samples/nam-tram-am.mp3` |
| `ngoc-huyen` | Ngoc Huyen | `.../samples/ngoc-huyen.mp3` |
| `thanh-nien-tu-tin` | Thanh nien tu tin | `.../samples/thanh-nien-tu-tin.mp3` |
| `adam` | Adam | `.../samples/adam.mp3` |
| `minh-anh` | Minh Anh | `.../samples/minh-anh.mp3` |
| `nam-cong-nghe` | Nam cong nghe | `.../samples/nam-cong-nghe.mp3` |
| `nho-ngot-ngao` | Nho Ngot Ngao | `.../samples/nho-ngot-ngao.mp3` |

Base URL: `https://raw.githubusercontent.com/doanquangkien/voice-notebooks/main/samples/`

---

## Cấu trúc file

```
srt-notebooks/
├── colab_srt-to-voice.ipynb                    ← Nam Trầm Ấm (template)
├── colab_srt-to-voice-ngoc-huyen.ipynb          ← Ngọc Huyền
├── colab_srt-to-voice-thanh-nien-tu-tin.ipynb   ← Thanh niên tự tin
├── test.srt                                     ← File test 10 dòng
└── README.md
```

Mỗi notebook: 3 cells — markdown intro → pip install + wget voice → model load + pipeline + UI.

---

## Colab URL Pattern

```
https://colab.research.google.com/github/doanquangkien/srt-notebooks/blob/main/colab_srt-to-voice-{slug}.ipynb
```

VD: `.../colab_srt-to-voice-ngoc-huyen.ipynb`

---

## Liên quan

- **Private repo:** `doanquangkien/srt` — code, docs, workmap, agent system
- **Voice samples:** `doanquangkien/voice-notebooks` — 7 file MP3 10s
- **OmniVoice:** `k2-fsa/OmniVoice` — TTS engine
- **Reference:** `doanquangkien/VOICE` — Gradio notebooks (pattern gốc)
