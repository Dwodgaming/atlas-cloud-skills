# Brayden’s preferred Atlas image workflows

Updated 2026-10-06 from Brayden’s explicit preferences; edit IDs and schemas
verified with `atlas models list --type image --json` and `atlas models get`.
Default to **Atlas Cloud directly** for these edits and Seedream v5 Pro generation.
An explicitly selected provider, skill or aggregator UI wins. Atlas is Brayden’s
preferred cost route, not a verified claim that it is always the cheapest host.
These are candidates to test, not local quality rankings or an instruction to run
all models. Re-fetch availability, schema and cost for the actual settings.

## Preferred edit shortlist

| Model | Atlas ID | Catalogue entry estimate (USD, 2026-10-06) | Input contract |
| --- | --- | --- | --- |
| MAI Image 2.6 Flash Edit | `microsoft/mai-image-2.6-flash/edit` | $0.0396 | `reference_images`, 1–5 JPEG/PNG, each <1,376,256 pixels; `size: default` preserves input ratio |
| MAI Image 2.6 Edit | `microsoft/mai-image-2.6/edit` | $0.0792 | Same reference and size contract as Flash |
| GPT Image 2.5 Flare Edit | `openai/gpt-image-2.5-flare/edit` | Estimate live for quality/size/input tokens | `images`, up to 16; optional `mask`; set quality and size explicitly |
| Qwen Image 3.0 Pro Edit | `qwen-image-3.0-pro/edit` | Estimate live | `reference_image_urls`, 1–3; 384–2048 px recommended, ≤10 MB; output 512*512–1440*1440 |
| Seedream v5.0 Flash Edit | `bytedance/seedream-v5.0-flash/edit` | $0.018 | `images`, up to 10; preset `WIDTH*HEIGHT` size; reference inputs free |
| Seedream v5.0 Pro Edit | `bytedance/seedream-v5.0-pro/edit` | $0.072 entry estimate | `images`, up to 10; preset `WIDTH*HEIGHT`; pixel tier and extra references affect cost |

Keep Flare standard and Developer endpoints distinct. The separately available
`openai/gpt-image-2.5-flare-developer/edit` has a $0.03 catalogue entry estimate;
that is not the standard Flare price or an instruction to silently substitute it.
Seedream Pro’s schema adds $0.003 for each reference after the first and uses
1.5K/2K pixel tiers; the catalogue entry estimate is not a quote for every request.

Previously saved candidates remain available for task-specific trials:
`alibaba/wan-2.7-pro/image-edit` and
`openai/gpt-image-2.5-sunburst-developer/edit`.

Other preferred workflows:

| Task | Preferred model IDs |
| --- | --- |
| Seedream v5 Pro text-to-image | `bytedance/seedream-v5.0-pro/text-to-image` |
| Separate a selected static into movable, individually editable objects/layers | `bytedance/seedream-v5.0-flash/layer-decomposition`; `bytedance/seedream-v5.0-pro/layer-decomposition` |
| Video reframing | `luma/ray-3.2/reframe` (video model, not static-image decomposition) |

## Matched parallel edit trials

1. Select a small task-appropriate subset (usually 2–3 candidates), or the models
   the user requested. Keep one immutable source set, ordered reference roles,
   the same requested change and preservation constraints. Write prompts in
   parallel when useful; use the same core edit instruction, adapting only the
   model’s documented reference syntax. Prompt-only requests stop at prompts.
2. Inspect each live schema and estimate each request with
   `atlas generate cost image <model> ... --json`; show the model/settings matrix,
   exact prompts and total estimated spend before any required production approval.
   Preserve the project’s existing execution authorization. New model discovery
   does not authorize extra paid trials.
3. Compile each request with `atlas generate image <model> ... --explain --json`.
   Bind references to its actual field (`images`, `reference_images` or
   `reference_image_urls`); avoid reusing a generic upload flag without checking
   the compiled request. Keep MAI web search off for source-preserving edits.
   Record Qwen prompt rewriting and Seedream optimization settings; shared seeds
   do not make different models deterministic equivalents.
4. Once execution is authorized, submit independent requests using the existing
   Atlas CLI with `--no-wait --no-download --json`, record every prediction ID,
   then poll with `atlas generate get <prediction-id> --json`. This lets tasks
   run concurrently without a new batch client. Bound submission concurrency
   (usually 2–3), collect partial successes, and avoid automatic POST retries.
   On timeout, inspect the existing prediction before considering another charge.
5. Compare side by side against the same source: requested change, untouched
   regions, identity/product geometry, camera/layout, text, shadows/reflections,
   artifacts, actual dimensions, elapsed time and actual cost when available.
   Disclose different model resolution ceilings and input copies; never shrink or
   overwrite the master just to fit MAI. Keep candidates pending until visual QA
   and human selection. Save model, exact prompt, settings, reference paths/hashes,
   prediction ID and result per candidate through the project’s existing receipts.

For `_atelier/project.json` projects, stage each candidate and use `atelier job
finish` with the returned URLs; follow `atelier-review` for reference state,
authorization and selection. Else use distinct candidate output paths.
Refresh the shortlist when asked for newer models or when an endpoint changes;
verify replacements before proposing them and retain explicit named versions.

Catalogue/schema verification is not an image-quality benchmark. No paid edits
were generated for this preference update.

## Production trial: editable static ads and other image compositions

Use `images/static-layer-decomposition` for source preservation, protected groups,
QA and packaging; use this Atlas skill for provider execution. The existing
`decompose.py` is fal-specific: it does not execute or parse Atlas responses.
Save the actual Atlas request, response, output files and costs before adapting
anything to the neutral layer manifest. Inspect the response rather than assuming
fal layer names, z-order, coordinates or bounding boxes exist.

The Flash decomposition schema inspected on 2026-10-01 accepts one `image`, an
optional `prompt`, `output_format` for the base, and `size`. Layers are transparent
PNG; each retains its own aspect ratio. Re-fetch the schema for the chosen model.

On one human-selected source, test separation, moving an object, editing that
object alone, and recomposing it over the reconstructed background. Preserve
contacting/held/worn groups and reflections where separation would break fidelity.
Check cut-out edges, hidden-area reconstruction, placement/scale, shadows,
reflections, product geometry and unchanged surrounding elements. Rebuild final
copy/logos as editable type/artwork. Keep originals immutable and outputs pending
until visual review passes; record the trial in the production project.

Prefer native layer transforms for moving/scaling usable cut-outs. Call an image
edit model only when the selected element’s pixels need to change. A generation
result is not itself an interactive editor or proof that all elements are editable.
