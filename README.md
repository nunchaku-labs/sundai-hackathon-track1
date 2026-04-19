# Sundai Hackathon Track 1 — Nunchaku API

Build apps with state-of-the-art image and video generation.

## Get Your API Key

1. Sign up at **https://sundai.nunchaku.dev/** (Google login). You get **$10 in credits** automatically.
2. Go to **Dashboard → API Keys → Create API Key**.
3. Copy the key immediately — you won't be able to see it again. It looks like `sk-nunchaku-...`.
4. Run out of credits? Come find the organizers.

## Quick Start

```bash
# 1. Set your API key (from the dashboard above)
export NUNCHAKU_API_KEY="sk-nunchaku-..."

# 2. Install dependencies
pip install requests Pillow

# 3. Generate your first image
python examples/python/text_to_image.py

# 4. Or run the interactive demo
pip install gradio
python demo/app.py
```

---

## Available Models

### Image Generation

| Model | Category | Endpoint | Tiers | Default Steps | Default Guidance |
|-------|----------|----------|-------|---------------|-----------------|
| `nunchaku-qwen-image` | Text-to-Image | `POST /v1/images/generations` | fast, radically_fast | 28 (rf: 4) | — (rf: true_cfg=1.0) |
| `nunchaku-qwen-image-edit` | Image-to-Image | `POST /v1/images/edits` | fast, radically_fast | 28 (rf: 4) | true_cfg=4.0 (rf: 1.0) |
| `nunchaku-flux.2-klein-9b` | Text-to-Image | `POST /v1/images/generations` | fast | 4 | 1.0 |
| `nunchaku-flux.2-klein-9b-edit` | Image-to-Image | `POST /v1/images/edits` | fast | 4 | 1.0 |

> FLUX models are pre-distilled (4-step) — already fast at default settings.
> Qwen `radically_fast` (rf) tier uses a distilled 4-step variant.

### Video Generation

| Model | Category | Endpoint | Tier | Default Steps | Default Guidance | Default Frames |
|-------|----------|----------|------|---------------|-----------------|----------------|
| `nunchaku-wan2.2-lightning-t2v` | Text-to-Video | `POST /v1/video/generations` | normal | 4 | 1.0 | 81 (3.4s @24fps) |
| `nunchaku-wan2.2-lightning-i2v` | Image-to-Video | `POST /v1/video/animations` | normal | 4 | 1.0 | 81 (3.4s @24fps) |

> Video models are pre-distilled (4-step Lightning). Tier defaults to `normal` — no need to set it explicitly.

---

## Pricing

Pay per successful request. `1 credit = $1 USD`. Your $10 signup bonus already covers a lot of hackathon iteration — here's the budget math.

| Model | Tier | Price / request | $10 gets you |
|-------|------|----------------:|-------------:|
| `nunchaku-qwen-image` | `radically_fast` | $0.002 | ~5,000 images |
| `nunchaku-qwen-image` | `fast` | $0.004 | ~2,500 images |
| `nunchaku-qwen-image-edit` | `radically_fast` | $0.0024 | ~4,166 edits |
| `nunchaku-qwen-image-edit` | `fast` | $0.0048 | ~2,083 edits |
| `nunchaku-flux.2-klein-9b` | `fast` | $0.0032 | ~3,125 images |
| `nunchaku-flux.2-klein-9b-edit` | `fast` | $0.004 | ~2,500 edits |
| `nunchaku-wan2.2-lightning-t2v` | `normal` | $0.020 | ~500 videos |
| `nunchaku-wan2.2-lightning-i2v` | `normal` | $0.025 | ~400 videos |

> **Tip:** default to `radically_fast` for Qwen (~10× faster, ~60% cheaper). Only switch to `fast` if you find the extra steps are genuinely needed for your final demo. Full tier comparison is under **Speed Tiers** below.

---

## API Reference

**Base URL:** `https://api.nunchaku.dev`

**Auth:** pass your API key in *either* header — both work:
```
Authorization: Bearer sk-nunchaku-...
# — or —
X-API-Key: sk-nunchaku-...
```

Each endpoint below shows three equivalent examples: **cURL**, **Python (`requests`)**, and **Python (OpenAI SDK)**. Video endpoints skip the OpenAI SDK — it doesn't support video generation.

### 1. Text-to-Image

```
POST /v1/images/generations
```

**cURL:**
```bash
curl -X POST https://api.nunchaku.dev/v1/images/generations \
  -H "Authorization: Bearer $NUNCHAKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "nunchaku-qwen-image",
    "prompt": "a red apple on a wooden table, photorealistic",
    "n": 1,
    "size": "1024x1024",
    "tier": "radically_fast",
    "response_format": "b64_json",
    "seed": 42
  }' | jq -r '.data[0].b64_json' | base64 -d > output.jpg
```

**Python (`requests`):**
```python
import requests, base64

response = requests.post(
    "https://api.nunchaku.dev/v1/images/generations",
    headers={"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"},
    json={
        "model": "nunchaku-qwen-image",
        "prompt": "a red apple on a wooden table, photorealistic",
        "n": 1,
        "size": "1024x1024",
        "tier": "radically_fast",
        "response_format": "b64_json",
        "seed": 42,
    },
)
img = base64.b64decode(response.json()["data"][0]["b64_json"])
```

**Python (OpenAI SDK):**
```python
import base64
from openai import OpenAI

client = OpenAI(base_url="https://api.nunchaku.dev/v1", api_key=API_KEY)

response = client.images.generate(
    model="nunchaku-qwen-image",
    prompt="a red apple on a wooden table, photorealistic",
    n=1,
    size="1024x1024",
    response_format="b64_json",
    extra_body={"tier": "radically_fast", "seed": 42},
)
img = base64.b64decode(response.data[0].b64_json)
```

> Swap `model` to `nunchaku-flux.2-klein-9b` for FLUX (distilled, `tier: "fast"` only — no `num_inference_steps` needed).

**Parameters:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model` | string | required | Model name |
| `prompt` | string | required | What to generate |
| `n` | int | 1 | Number of images (1-10) |
| `size` | string | `1024x1024` | `WxH` format |
| `tier` | string | `radically_fast` (recommended) | Speed tier (`fast` or `radically_fast` for Qwen) |
| `response_format` | string | `b64_json` | `b64_json` or `url` |
| `seed` | int | random | For reproducibility |
| `negative_prompt` | string | — | What to avoid |
| `num_inference_steps` | int | tier default | Override diffusion steps |
| `guidance_scale` | float | tier default | Classifier-free guidance |

---

### 2. Image-to-Image (Edit)

```
POST /v1/images/edits
```

> **Key:** the input image goes in the `url` field as a `data:` URI — *not* an `image` field.

**cURL:**
```bash
IMG_B64=$(base64 -w0 input.jpg)
curl -X POST https://api.nunchaku.dev/v1/images/edits \
  -H "Authorization: Bearer $NUNCHAKU_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"nunchaku-qwen-image-edit\",
    \"prompt\": \"add a sunset in the background\",
    \"url\": \"data:image/jpeg;base64,$IMG_B64\",
    \"n\": 1,
    \"size\": \"1024x1024\",
    \"tier\": \"radically_fast\",
    \"response_format\": \"b64_json\"
  }" | jq -r '.data[0].b64_json' | base64 -d > edited.jpg
```

**Python (`requests`):**
```python
import base64, requests

with open("input.jpg", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

response = requests.post(
    "https://api.nunchaku.dev/v1/images/edits",
    headers={"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"},
    json={
        "model": "nunchaku-qwen-image-edit",
        "prompt": "add a sunset in the background",
        "url": f"data:image/jpeg;base64,{img_b64}",
        "n": 1,
        "size": "1024x1024",
        "tier": "radically_fast",
        "response_format": "b64_json",
    },
)
```

**Python (OpenAI SDK):**
```python
import base64
from openai import OpenAI

client = OpenAI(base_url="https://api.nunchaku.dev/v1", api_key=API_KEY)

with open("input.jpg", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

# The SDK's images.edit() requires an `image` kwarg, but Nunchaku ignores it
# and reads the input image from `url` in extra_body.
response = client.images.edit(
    model="nunchaku-qwen-image-edit",
    image=b"",  # ignored by Nunchaku — placeholder to satisfy the SDK
    prompt="add a sunset in the background",
    n=1,
    size="1024x1024",
    response_format="b64_json",
    extra_body={
        "url": f"data:image/jpeg;base64,{img_b64}",
        "tier": "radically_fast",
    },
)
edited = base64.b64decode(response.data[0].b64_json)
```

> Swap `model` to `nunchaku-flux.2-klein-9b-edit` (`tier: "fast"`) for the FLUX edit model.

---

### 3. Text-to-Video

```
POST /v1/video/generations
```

**cURL:**
```bash
curl -X POST https://api.nunchaku.dev/v1/video/generations \
  -H "Authorization: Bearer $NUNCHAKU_API_KEY" \
  -H "Content-Type: application/json" \
  --max-time 120 \
  -d '{
    "model": "nunchaku-wan2.2-lightning-t2v",
    "prompt": "A golden retriever running on a beach at sunset, cinematic",
    "n": 1,
    "size": "1280x720",
    "num_frames": 81,
    "num_inference_steps": 4,
    "guidance_scale": 1.0,
    "response_format": "b64_json"
  }' | jq -r '.data[0].b64_json' | base64 -d > output.mp4
```

**Python (`requests`):**
```python
response = requests.post(
    "https://api.nunchaku.dev/v1/video/generations",
    headers={"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"},
    json={
        "model": "nunchaku-wan2.2-lightning-t2v",
        "prompt": "A golden retriever running on a beach at sunset, cinematic",
        "n": 1,
        "size": "1280x720",
        "num_frames": 81,
        "num_inference_steps": 4,
        "guidance_scale": 1.0,
        "response_format": "b64_json",
    },
    timeout=120,
)
video = base64.b64decode(response.json()["data"][0]["b64_json"])
with open("output.mp4", "wb") as f:
    f.write(video)
```

> The OpenAI SDK has no video-generation method — use cURL or `requests`.

**Video parameters:**

| Parameter | Default | Description |
|-----------|---------|-------------|
| `num_frames` | 81 | Frames to generate (81 = 3.4s at 24fps) |
| `num_inference_steps` | 4 | Denoising steps (Lightning models) |
| `guidance_scale` | 1.0 | CFG scale |
| `size` | `1280x720` | Video dimensions |

---

### 4. Image-to-Video (Animate)

```
POST /v1/video/animations
```

> **Key:** unlike the other endpoints, the image goes inside `messages[0].content` as an `image_url` block — not a top-level `image` or `url` field.

**cURL:**
```bash
IMG_B64=$(base64 -w0 input.jpg)
PROMPT="the scene comes to life with gentle motion"
curl -X POST https://api.nunchaku.dev/v1/video/animations \
  -H "Authorization: Bearer $NUNCHAKU_API_KEY" \
  -H "Content-Type: application/json" \
  --max-time 120 \
  -d "{
    \"model\": \"nunchaku-wan2.2-lightning-i2v\",
    \"prompt\": \"$PROMPT\",
    \"n\": 1,
    \"size\": \"1280x720\",
    \"num_frames\": 81,
    \"num_inference_steps\": 4,
    \"guidance_scale\": 1.0,
    \"response_format\": \"b64_json\",
    \"messages\": [{
      \"role\": \"user\",
      \"content\": [
        {\"type\": \"image_url\", \"image_url\": {\"url\": \"data:image/jpeg;base64,$IMG_B64\"}},
        {\"type\": \"text\", \"text\": \"$PROMPT\"}
      ]
    }]
  }" | jq -r '.data[0].b64_json' | base64 -d > output.mp4
```

**Python (`requests`):**
```python
with open("input.jpg", "rb") as f:
    img_b64 = base64.b64encode(f.read()).decode()

data_uri = f"data:image/jpeg;base64,{img_b64}"
prompt = "the scene comes to life with gentle motion"

response = requests.post(
    "https://api.nunchaku.dev/v1/video/animations",
    headers={"Authorization": f"Bearer {API_KEY}", "Content-Type": "application/json"},
    json={
        "model": "nunchaku-wan2.2-lightning-i2v",
        "prompt": prompt,
        "n": 1,
        "size": "1280x720",
        "num_frames": 81,
        "num_inference_steps": 4,
        "guidance_scale": 1.0,
        "response_format": "b64_json",
        "messages": [
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": data_uri}},
                    {"type": "text", "text": prompt},
                ],
            }
        ],
    },
    timeout=120,
)
```

> The OpenAI SDK has no video-generation method — use cURL or `requests`.

---

### Response Format

All endpoints return:

```json
{
  "created": 1234567890,
  "data": [
    {
      "b64_json": "<base64-encoded image or video>",
      "url": null,
      "revised_prompt": null
    }
  ]
}
```

- Images are JPEG by default (change with `output_format`: jpeg, png, webp)
- Videos are MP4

---

### Speed Tiers

Trade quality for speed with the `tier` parameter:

| Tier | Qwen latency | Cost | Recommendation |
|------|--------------|------|----------------|
| `radically_fast` | ~3s | 0.4× | **Default for Qwen.** Distilled 4-step variant. |
| `fast` | ~60s (p95 ~120s) | 0.8× | Higher-quality mode — only when you've confirmed `radically_fast` isn't enough for a final render. Expect client-side timeouts if your HTTP timeout is below ~180s. |

FLUX models support `fast` tier only (already distilled, ~1s). Qwen models support both `fast` and `radically_fast`. Video Lightning models are already distilled (4 steps) — no tier parameter.

---

### Errors

| Code | Meaning | What to do |
|------|---------|------------|
| 401 | Invalid or missing API key | Check your `NUNCHAKU_API_KEY` |
| 402 | Insufficient credits | Check balance on the dashboard; ask organizers if you need a top-up |
| 429 | Rate limited (RPM or concurrent) | Wait and retry (check `Retry-After` header) |
| 504 | Generation timeout | Retry, or use a simpler prompt |

---

## Examples

Four flavors per endpoint — pick the one closest to your hackathon stack.

### Python (`requests`)
| File | Endpoint |
|------|----------|
| [`examples/python/text_to_image.py`](examples/python/text_to_image.py) | `/v1/images/generations` |
| [`examples/python/image_to_image.py`](examples/python/image_to_image.py) | `/v1/images/edits` |
| [`examples/python/text_to_video.py`](examples/python/text_to_video.py) | `/v1/video/generations` |
| [`examples/python/image_to_video.py`](examples/python/image_to_video.py) | `/v1/video/animations` (shows `messages` format) |

### Python (OpenAI SDK)
| File | Notes |
|------|-------|
| [`examples/python/openai_sdk.py`](examples/python/openai_sdk.py) | Works for text-to-image. Edit and video need raw `requests` — see notes in the file. |

### JavaScript (fetch)
| File | Endpoint |
|------|----------|
| [`examples/javascript/text_to_image.mjs`](examples/javascript/text_to_image.mjs) | `/v1/images/generations` |
| [`examples/javascript/image_to_image.mjs`](examples/javascript/image_to_image.mjs) | `/v1/images/edits` |
| [`examples/javascript/text_to_video.mjs`](examples/javascript/text_to_video.mjs) | `/v1/video/generations` |
| [`examples/javascript/image_to_video.mjs`](examples/javascript/image_to_video.mjs) | `/v1/video/animations` |

### cURL
| File | Endpoint |
|------|----------|
| [`examples/curl/text_to_image.sh`](examples/curl/text_to_image.sh) | `/v1/images/generations` |
| [`examples/curl/image_to_image.sh`](examples/curl/image_to_image.sh) | `/v1/images/edits` |
| [`examples/curl/text_to_video.sh`](examples/curl/text_to_video.sh) | `/v1/video/generations` |
| [`examples/curl/image_to_video.sh`](examples/curl/image_to_video.sh) | `/v1/video/animations` |

## Demo App

Interactive Gradio app with 5 tabs — including a **pipeline** that chains generate → edit → animate:

```bash
pip install -r requirements.txt
python demo/app.py
```

---

## Hackathon Tips

- **Start with `nunchaku-qwen-image`** — supports all speed tiers, great for iteration
- **Try `nunchaku-flux.2-klein-9b`** — distilled model, fast results
- **Default to `tier: "radically_fast"`** (Qwen only) — ~10× faster, 60% cheaper, and best availability right now. Only reach for `tier: "fast"` if you find the extra steps are truly needed.
- **Set a `seed`** while iterating on prompts — reproducible results
- **Chain endpoints** — generate an image, edit it, then animate it
- **Video takes ~30s** — plan your demo flow around this
- **Use `response_format: "url"`** if you want a link instead of base64

## Running Tests

```bash
export NUNCHAKU_API_KEY="sk-nunchaku-..."
pip install pytest requests Pillow
pytest tests/ -v
```

## License

MIT
