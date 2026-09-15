# Video Generation — Complete Code Templates

## Table of Contents
- [Python](#python)
- [Node.js / TypeScript](#nodejs--typescript)
- [cURL](#curl)

---

## Python

```python
import requests
import time
import os

ATLAS_API_KEY = os.environ.get("ATLASCLOUD_API_KEY")
BASE_URL = "https://api.atlascloud.ai/api/v1"

HEADERS = {
    "Authorization": f"Bearer {ATLAS_API_KEY}",
    "Content-Type": "application/json",
}


def generate_video(model: str, prompt: str, **kwargs) -> str:
    """
    Generate a video and return the output URL.

    Args:
        model: Model ID, e.g. "kwaivgi/kling-v3.0-std/text-to-video"
        prompt: Text description of the video
        **kwargs: Additional parameters (duration, aspect_ratio, image_url, etc.)

    Returns:
        URL of the generated video
    """
    # Step 1: Submit generation task
    payload = {"model": model, "prompt": prompt, **kwargs}
    resp = requests.post(f"{BASE_URL}/model/generateVideo", json=payload, headers=HEADERS, timeout=50)
    resp.raise_for_status()
    data = resp.json()

    prediction_id = data["data"]["id"]
    print(f"Task submitted. Prediction ID: {prediction_id}")

    # Step 2: Poll for result (videos take longer, up to 10 min)
    for _ in range(200):
        time.sleep(3)
        result = requests.get(f"{BASE_URL}/model/prediction/{prediction_id}", headers=HEADERS, timeout=30)
        result.raise_for_status()
        result_data = result.json()["data"]

        status = result_data.get("status", "unknown")
        if status in ("completed", "succeeded"):
            outputs = result_data.get("outputs") or result_data.get("output", [])
            if isinstance(outputs, str):
                outputs = [outputs]
            print(f"Generation completed: {outputs[0]}")
            return outputs[0]
        elif status == "failed":
            error = result_data.get("error", "Unknown error")
            raise RuntimeError(f"Generation failed: {error}")
        else:
            print(f"Status: {status}...")

    raise TimeoutError("Generation timed out")


# Text-to-Video
if __name__ == "__main__":
    url = generate_video(
        model="kwaivgi/kling-v3.0-std/text-to-video",
        prompt="A rocket launching into space with dramatic clouds",
        duration=5,
        aspect_ratio="16:9",
    )
    print(f"Video URL: {url}")
```

### Image-to-Video (Python)

```python
url = generate_video(
    model="kwaivgi/kling-v3.0-std/image-to-video",
    prompt="Camera slowly zooms in, petals gently falling",
    image_url="https://example.com/cherry-blossom.jpg",
    duration=5,
    aspect_ratio="16:9",
)
print(f"Video URL: {url}")
```

---

## Node.js / TypeScript

```typescript
const ATLAS_API_KEY = process.env.ATLASCLOUD_API_KEY;
const BASE_URL = 'https://api.atlascloud.ai/api/v1';

const headers = {
  Authorization: `Bearer ${ATLAS_API_KEY}`,
  'Content-Type': 'application/json',
};

async function generateVideo(
  model: string,
  prompt: string,
  extraParams: Record<string, unknown> = {}
): Promise<string> {
  // Step 1: Submit generation task
  const submitResp = await fetch(`${BASE_URL}/model/generateVideo`, {
    method: 'POST',
    headers,
    body: JSON.stringify({ model, prompt, ...extraParams }),
  });

  if (!submitResp.ok) {
    throw new Error(`Submit failed: ${submitResp.status} ${await submitResp.text()}`);
  }

  const submitData = await submitResp.json();
  const predictionId = submitData.data.id;
  console.log(`Task submitted. Prediction ID: ${predictionId}`);

  // Step 2: Poll for result (videos take 1-5 minutes)
  for (let i = 0; i < 200; i++) {
    await new Promise((r) => setTimeout(r, 3000));

    const pollResp = await fetch(`${BASE_URL}/model/prediction/${predictionId}`, { headers });
    if (!pollResp.ok) {
      throw new Error(`Poll failed: ${pollResp.status}`);
    }

    const result = (await pollResp.json()).data;

    if (result.status === 'completed' || result.status === 'succeeded') {
      const outputs = result.outputs ?? (Array.isArray(result.output) ? result.output : result.output ? [result.output] : []);
      console.log(`Generation completed: ${outputs[0]}`);
      return outputs[0];
    }

    if (result.status === 'failed') {
      throw new Error(`Generation failed: ${result.error || 'Unknown error'}`);
    }

    console.log(`Status: ${result.status}...`);
  }

  throw new Error('Generation timed out');
}

// Text-to-Video
const videoUrl = await generateVideo(
  'kwaivgi/kling-v3.0-std/text-to-video',
  'A rocket launching into space with dramatic clouds',
  { duration: 5, aspect_ratio: '16:9' }
);

// Image-to-Video
const videoUrl2 = await generateVideo(
  'kwaivgi/kling-v3.0-std/image-to-video',
  'Camera slowly zooms in, petals gently falling',
  { image_url: 'https://example.com/cherry-blossom.jpg', duration: 5 }
);
```

---

## cURL

### Text-to-Video

```bash
PREDICTION_ID=$(curl -s -X POST "https://api.atlascloud.ai/api/v1/model/generateVideo" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "kwaivgi/kling-v3.0-std/text-to-video",
    "prompt": "A rocket launching into space with dramatic clouds",
    "duration": 5,
    "aspect_ratio": "16:9"
  }' | jq -r '.data.id')

echo "Prediction ID: $PREDICTION_ID"

while true; do
  sleep 3
  RESULT=$(curl -s "https://api.atlascloud.ai/api/v1/model/prediction/$PREDICTION_ID" \
    -H "Authorization: Bearer $ATLASCLOUD_API_KEY")

  STATUS=$(echo "$RESULT" | jq -r '.data.status')

  if [ "$STATUS" = "completed" ] || [ "$STATUS" = "succeeded" ]; then
    echo "Video URL:"
    echo "$RESULT" | jq -r '.data.outputs[0]'
    break
  elif [ "$STATUS" = "failed" ]; then
    echo "Failed:"
    echo "$RESULT" | jq -r '.data.error'
    break
  else
    echo "Status: $STATUS..."
  fi
done
```

### Image-to-Video

```bash
PREDICTION_ID=$(curl -s -X POST "https://api.atlascloud.ai/api/v1/model/generateVideo" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "kwaivgi/kling-v3.0-std/image-to-video",
    "prompt": "Camera slowly zooms in, petals gently falling",
    "image_url": "https://example.com/cherry-blossom.jpg",
    "duration": 5,
    "aspect_ratio": "16:9"
  }' | jq -r '.data.id')

echo "Prediction ID: $PREDICTION_ID"
# ... same polling loop as above
```

---

## Seedance 2.5

Verified against the live schemas on 2026-09-15:
`https://static.atlascloud.ai/model/schema/bytedance-seedance-2.5-{text-to-video,image-to-video,reference-to-video}.json`

Three model IDs, one endpoint (`POST /api/v1/model/generateVideo`), one polling contract.

| Model ID | Required | Pricing mode |
|---|---|---|
| `bytedance/seedance-2.5/text-to-video` | `model`, `prompt` | per_second |
| `bytedance/seedance-2.5/image-to-video` | `model`, `image` | per_second |
| `bytedance/seedance-2.5/reference-to-video` | `model` | runtime_estimate (per-job quote) |

### Shared parameters

- `duration` — integer, `4`–`30`, or `-1` for auto. Default `5`.
- `resolution` — default `720p`. Enum: `480p`, `720p`, `720p-sr`, `720p-esr`, `1080p`, `1080p-sr`, `1080p-esr`, `1080p-esr & 60fps`, `1440p-sr`, `1440p-esr`, `4k-esr`.
  Native tiers are `480p`, `720p`, `1080p`. Every `-sr` / `-esr` tier generates the nearest native source, then upscales: `720p-*` from 480p, `1080p-*` from 720p, `1440p-*` and `4k-esr` from 1080p. `4k-esr` gives a 2160px short edge (3840x2160 at 16:9), not native 4K.
- `ratio` — default `adaptive`. T2V and R2V accept `16:9`, `4:3`, `1:1`, `3:4`, `9:16`, `21:9`, `adaptive`. **I2V accepts `adaptive` only** — the output keeps the source image's ratio.
- `generate_audio` — boolean, default `true`. Native synchronised voice / SFX / BGM.
- `output_format` — `mp4` (default) or `mov`. `mov` encodes yuv444p; use it for multi-round edit/extend chains where recompression loss stacks.
- `return_last_frame` — boolean, default `false`. Returns the last frame as a second output URL.
- `watermark` — boolean, default `false`.

Per-second price at 2026-09-15 (image-to-video and text-to-video, same table):
480p $0.1397 · 720p $0.3005 · 720p-sr $0.2193 · 720p-esr $0.2515 · 1080p $0.5914 · 1080p-sr $0.4056 · 1080p-esr $0.4507 · 1080p-esr & 60fps $0.4958 · 1440p-sr $0.7000 · 1440p-esr $0.7602 · 4k-esr $1.7006.

Note the inversion: native `1080p` costs more than `1080p-esr`, because the enhanced tier renders 720p natively and then upscales. They are different products, not quality tiers of one product.

### Image-to-video

- `image` (required) — first frame. URL, Base64, or `asset://<ASSET_ID>`.
- `last_image` (optional) — final frame; the video transitions first to last.
- Image limits: jpeg/png/webp/bmp/tiff/gif, 300–6000px each side, aspect ratio 0.4–2.5, max 30MB.

```bash
curl -s -X POST "https://api.atlascloud.ai/api/v1/model/generateVideo" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "bytedance/seedance-2.5/image-to-video",
    "image": "https://.../first.png",
    "last_image": "https://.../last.png",
    "prompt": "Slow push in, the box settles onto the tray",
    "duration": 8,
    "resolution": "1080p",
    "ratio": "adaptive",
    "generate_audio": true,
    "output_format": "mov"
  }' | jq -r '.data.id'
```

### Reference-to-video (omni)

- `reference_images` — up to **30**. Same per-image limits as above, plus heic/heif.
- `reference_videos` — up to **10**. mp4/mov (H.264/H.265 + AAC/MP3), 480p–4k, 2–30s each, 24–60 FPS, ≤200MB. Combined duration across all reference videos ≤30s per request.
- `reference_audios` — up to **10**. wav/mp3, 2–30s each, ≤15MB, combined ≤30s. **Audio-only referencing is unique to 2.5**: one BGM, voice, or SFX track can drive pacing, beat matching, and lip-sync.
- Cite inputs in the prompt in submission order with @-syntax: `@Image1`, `@Video1`, `@Audio1`.
- Real human faces cannot be uploaded. Use model-generated assets, preset digital characters, or authorised real-person assets.

`omni_reference_task_type` — subtask hint, API-only. Leave unset for `auto`, or set explicitly to get constraints validated at submit time instead of as an async task failure:

| Value | Meaning | Constraints |
|---|---|---|
| `reference` | New video from reference media | No extra ratio/duration constraints |
| `edit` | Edit an existing video | Exactly 1 `reference_videos` entry, 4–30s; `ratio` `adaptive`; `duration` `-1` |
| `extend` | Extend an existing video | Exactly 1 `reference_videos` entry; `ratio` `adaptive` |

Do not set it on prompt-only, first-frame, or first+last-frame requests — upstream rejects it. The model still re-derives the task type from the prompt, so a prompt that disagrees with the value still fails asynchronously. A prompt phrased as editing `@Video1`'s own scene flips the model into edit mode, which forces `duration: -1` (output tracks the input length, may run ~0.4s short).

```bash
curl -s -X POST "https://api.atlascloud.ai/api/v1/model/generateVideo" \
  -H "Authorization: Bearer $ATLASCLOUD_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "bytedance/seedance-2.5/reference-to-video",
    "reference_images": ["https://.../truck.png", "https://.../driver.png"],
    "reference_audios": ["https://.../bed.mp3"],
    "prompt": "@Image1 drives through the yard, cut on the beat of @Audio1",
    "omni_reference_task_type": "reference",
    "duration": 10,
    "resolution": "1080p",
    "ratio": "16:9"
  }' | jq -r '.data.id'
```

Reference-to-video is `runtime_estimate` — no flat catalog price. Quote a job first:

```bash
atlas generate cost video bytedance/seedance-2.5/reference-to-video \
  -p "..." --duration 10 --resolution 1080p --json
```

### CLI

`atlas` v0.1.18 works inside the agent shell. The TLS-handshake timeout that affected v0.1.16 is gone — verified 2026-09-15. Pass the key inline so it never enters context:

```bash
export ATLASCLOUD_API_KEY="$(bwsget ATLAS_CLOUD_API_KEY)"
atlas models list --type video --json
atlas models get bytedance/seedance-2.5/reference-to-video --json
atlas generate video bytedance/seedance-2.5/image-to-video -p "..." --no-wait --json
```

`generate cost` validates required params, so a missing `image` on an i2v quote returns `invalid_param` rather than a number. Sample quote: `bytedance/seedance-2.5/reference-to-video`, 10s, 1080p = $5.94 (2026-09-15).
