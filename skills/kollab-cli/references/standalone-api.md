# Standalone CLI and API

## Authentication

Use a workspace login for interactive `model run` / `tool run`, or supply an
account API key through `KOLLAB_API_KEY` / `KOLLAB_STANDALONE_API_KEY` for automation.
Create a key from an authenticated shell, capturing the response privately:

```bash
umask 077
kollab api-key create --name "terminal agent" --space-id <space-id> > ./kollab-key-response.json
```

Transfer the key to a secret store through a controlled local process, then delete
the response file. Do not display it in tool output. The raw key is returned once;
never paste it into chat, source control or command logs. Inspect key metadata with `kollab api-key list`; revoke
unused keys with `kollab api-key revoke <key-id>`.

The API key also supports Hosted Apps, `kollab social fetch` and optional
`kollab mcp serve`. Other workspace resource commands require login/token auth.
`KOLLAB_SPACE_ID` can select a workspace for model/tool calls; the server checks
access before execution and billing. With no override the key's default workspace
or login workspace applies. Calls spend that workspace's credits.

External Anthropic Messages clients can use `https://kollab.im/api/llm` as
`ANTHROPIC_BASE_URL` and the same account key as `ANTHROPIC_AUTH_TOKEN`, supplied
through secure process configuration. Use a priced catalog model for the main
model and any client background model. Never supply provider keys or direct
provider endpoints. Authentication/credit/rate-limit failures must be resolved
before retrying paid calls.

## Text models

```bash
kollab model list
kollab model list --source openrouter
kollab model info google/gemini-2.5-flash
kollab model run --model google/gemini-2.5-flash --prompt "Summarize this text" --output-file ./answer.txt
```

`--model` takes a concrete catalog ID. `--reasoning` is supported only by catalog
OpenRouter models advertising that effort; managed short IDs reject it.
`--session-id <uuid>` retains successful turns for follow-ups. One session can
have only one running request. Increase `--max-tokens` if thinking exhausts the
budget before visible text is produced.

`--request-id` identifies a paid attempt. Reuse it only with identical parameters
to recover/replay the result without another charge; changed parameters return
409. Use a new ID or omit it for a new generation. Inspect errors before retrying.

## Managed tool commands

```bash
kollab tool list
kollab tool run generate_image --input-json '{"prompt":"红色纸飞机","model":"gemini-3.1-flash-image"}' --output-dir ./output
kollab tool run generate_image --input-json '{"prompt":"透明背景的 logo","model":"gpt-image-2","size":"1024x1024","quality":"high","background":"transparent","num_images":2}' --output-dir ./output
kollab tool run edit_image --input-json '{"prompt":"把背景换成海边","model":"gpt-image-2","image_refs":[{"media_type":"image","artifact_id":"<image-artifact-id>"}]}' --output-dir ./output
kollab tool run generate_video --input-json '{"prompt":"东京夜景","model":"seedance-2"}' --output-dir ./output
kollab tool run generate_video --input-json '{"prompt":"让图片中的主体自然动起来","model":"seedance-2","media_refs":[{"media_type":"image","artifact_id":"<image-artifact-id>"}]}' --output-dir ./output
kollab tool run text_to_speech --input-json '{"text":"欢迎使用 Kollab","model":"openrouter-tts/x-ai/grok-voice-tts-1.0"}' --output-dir ./output
kollab tool run speech_to_text --input-json '{"audio_artifact_id":"<artifact-id>","model":"openrouter-stt/x-ai/grok-stt-1.0"}' --output-dir ./output
kollab tool run jev --input-file req.json --json
```

图片模型也可用 `model run`：

```bash
kollab model run --model google/gemini-3.1-flash-image --prompt "一只红色纸飞机" --output-dir ./images --json
```

成功时 `data.images` 是带 UUID 形态 `artifact_id`（已登记的真实 artifact）、`file_name`、`media_type`、`download_url`、`preview_url` 的引用，`data.local_paths` 是 CLI 下载后的本地文件路径。纯图片结果允许 `data.text` 为空。示例回执（字段形状示意，非实测）：

```json
{"ok":true,"data":{"text":"","model":"openrouter/google/gemini-3.1-flash-image","images":[{"artifact_id":"a59acf7c-07a7-45c1-83f2-109ee6fc5de9","file_name":"image.png","media_type":"image","download_url":"https://<signed-download-link>","preview_url":"https://<signed-preview-link>"}],"local_paths":["./images/image.png"],"usage":{"input_tokens":12,"output_tokens":1}}}
```

多张图部分失败时，成功项仍在 `data.images`，失败项在有界 `data.errors`（`index` + `message`）；逐项检查，不要把部分成功当作全部成功。Nano Banana 2 (`gemini-3.1-flash-image`)、Nano Banana Pro (`gemini-3-pro-image`)、`gemini-2.5-flash-image` 和 Grok Image 的图片回执均可提供 UUID artifact 引用；同空间可原样交给 `edit_image`，无需上传。链式使用（需 `jq`）：

```bash
mkdir -p ./images
kollab tool run generate_image --input-json '{"prompt":"a green frog on a lily pad","model":"gemini-3.1-flash-image"}' --output-dir ./images --json > ./images/generated.json
IMAGE_ID="$(jq -er '.data.images[0].artifact_id' ./images/generated.json)"
kollab tool run edit_image --input-json "$(jq -nc --arg id "$IMAGE_ID" '{prompt:"make the frog orange",model:"gemini-3.1-flash-image",image_refs:[{media_type:"image",artifact_id:$id}]}')" --output-dir ./images --json
```

`/api/llm` 保持 Anthropic Messages 文字输出；图片使用 `tool run generate_image`，或使用支持图片输出的目录模型配合 `model run --output-dir`。所有调用经 Kollab Server 和托管网关，费用扣 API key 绑定 Space 的额度。媒体完成后由 Server 返回 artifact 下载链接，CLI 才将文件保存到本地；链接需要按现有有效期使用。

### generate_image 参数

| 参数 | 必填 | 说明 |
|---|---|---|
| `prompt` | 是 | 图片描述 |
| `model` | 是 | `gpt-image-2`、`gemini-3.1-flash-image`、`gemini-3-pro-image`、`gemini-2.5-flash-image` |
| `size` | 否 | 尺寸，如 `1024x1024`、`1536x1024`（横）、`1024x1536`（竖）、`auto` |
| `quality` | 否 | `low`、`medium`、`high` |
| `background` | 否 | `opaque`（默认）或 `transparent` |
| `num_images` | 否 | 生成数量，整数 |
| `output_format` | 否 | 输出格式 |
| `output_compression` | 否 | 压缩率，整数 |
| `output_name` | 否 | 指定输出文件名 |
| `moderation` | 否 | 内容审核等级 |

### edit_image 参数

| 参数 | 必填 | 说明 |
|---|---|---|
| `prompt` | 是 | 编辑指令，描述要对图片做什么修改 |
| `model` | 是 | `gpt-image-2`、`gemini-3.1-flash-image`、`gemini-3-pro-image`、`gemini-2.5-flash-image` |
| `image_refs` | 是 | 原图引用数组，格式 `[{"media_type":"image","artifact_id":"<id>"}]` |
| `size` | 否 | 输出尺寸 |
| `quality` | 否 | `low`、`medium`、`high` |
| `background` | 否 | `opaque` 或 `transparent` |
| `output_format` | 否 | 输出格式 |
| `output_compression` | 否 | 压缩率 |
| `output_name` | 否 | 指定输出文件名 |
| `moderation` | 否 | 内容审核等级 |

`edit_image` 与 `generate_image` 共享相同的后端模型和大部分参数，区别在于 `edit_image` 必须提供
`image_refs` 作为编辑输入，且不支持 `num_images`（每次编辑只输出一张）。`image_refs` 中的
`artifact_id` 必须是当前空间内已有的图片 artifact。

## 3D and current availability

```bash
kollab tool run generate_3d --input-json '{"prompt":"a small ceramic vase","model":"tripo-3d"}' --output-dir ./output
```

Run `kollab tool list` first for currently enabled tools, model IDs and required
inputs. Examples require those routes to be available to the account. Do not
substitute provider endpoints or credentials when a route is unavailable.
Video calls poll until terminal status; completed media is downloaded to
`--output-dir` (current directory by default). 3D returns GLB files. Inspect local
files and the returned result before claiming a successful generation.
Treat partial image successes separately from errors. Use same-workspace image
artifact IDs for editing and image-to-video, without continuation/wakeup fields.

`jev` accepts a plain-text `state` and a `questions` map. Question types are
`noul` (instructions), `choice` (instructions plus an ID-to-description criteria
map), and `score` (instructions plus ordered criteria descriptions). It accepts
no model field. See the example in SKILL.md and the current tool inventory.
