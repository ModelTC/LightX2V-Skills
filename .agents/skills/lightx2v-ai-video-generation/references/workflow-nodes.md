# LightX2V Workflow Nodes

This reference is for agents creating or editing LightX2V workflows from scratch. Use it with `lightx2v workflow create --input @workflow.json` and `lightx2v workflow run ...`.

## Workflow JSON Shape

```json
{
  "workflow_id": "NEW-UUID",
  "name": "Readable workflow name",
  "description": "What the workflow does",
  "visibility": "private",
  "nodes": [
    {
      "id": "stable_node_id",
      "tool_id": "text-input",
      "name": "Canvas label",
      "position": {"x": 80, "y": 120},
      "data": {}
    }
  ],
  "connections": [
    {
      "id": "stable_connection_id",
      "source_node_id": "source_node",
      "source_port_id": "out-text",
      "target_node_id": "target_node",
      "target_port_id": "in-text"
    }
  ]
}
```

Rules:

- `workflow_id` must be a real UUID. Generate one; do not use placeholders.
- Use stable lowercase node ids such as `script_input`, `portrait_input`, `voice_tts`, `avatar_video`.
- `position` only affects canvas layout, but include it so the workflow opens cleanly.
- Put default values in Input node `data.value` only when the workflow needs a default. For one run, pass values through `workflow run --inputs`.
- Use `model` in node `data` for generation nodes. `model` must be the actual deployed `model_cls`, not a display name.
- Discover available model classes with `lightx2v models --json`.

## Input Node Values

Input nodes are the only nodes users should fill at run time.

| tool_id | Output port | Runtime value |
| --- | --- | --- |
| `text-input` | `out-text` | String |
| `image-input` | `out-image` | URL, data URL, `studio:...`, `task:.../output/...`, or local file via `--input-file` |
| `audio-input` | `out-audio` | Same media formats |
| `video-input` | `out-video` | Same media formats |

Example:

```bash
lightx2v workflow run WORKFLOW_ID \
  --inputs '{"script_input":"说一段欢迎词"}' \
  --input-file portrait_input=./portrait.png \
  --input-file backing_audio=./speech.wav \
  --poll
```

## Node Catalog

### Input Nodes

| tool_id | Purpose | Inputs | Outputs | data keys |
| --- | --- | --- | --- | --- |
| `text-input` | User text/script/prompt | none | `out-text` | `value` |
| `image-input` | User image(s) | none | `out-image` | `value`, `image_edits` |
| `audio-input` | User audio | none | `out-audio` | `value` |
| `video-input` | User video | none | `out-video` | `value` |

Notes:

- `image-input` output is normalized as a list internally; downstream nodes usually accept it directly.
- `image_edits` may be an empty list. Use it only when matching existing UI crop/edit behavior.
- Runtime values can be passed as raw strings in `--inputs`; local files should use `--input-file`.

### Text And Audio Utility Nodes

| tool_id | Purpose | Inputs | Outputs | Important data |
| --- | --- | --- | --- | --- |
| `text-generation` | LLM planning, prompt rewriting, structured text | `in-text`, `in-image` optional | `out-text` or custom text ports | `model`, `mode`, `customInstruction`, `custom_outputs`, `useSearch` |
| `tts` | Text-to-speech | `in-text`, `in-context-tone` optional | `out-audio` | `voiceType`, `resourceId`, `voiceTab`, `speakerId`, `emotion`, `emotionScale`, `speechRate`, `loudnessRate`, `pitch` |
| `asr-transcribe` | Audio/video to text | exactly one of `in-audio` or `in-video` | `out-text` | `languageHint` |

For TTS CLI commands, built-in voice discovery, clone voice commands, and full `tts` node examples, read `references/voice-tts.md`.

### Image Generation Nodes

| tool_id | Task | Inputs | Outputs | Important data |
| --- | --- | --- | --- | --- |
| `text-to-image` | `t2i` | `in-text` | `out-image` | `model`, `aspectRatio`, `resolutionLevel` |
| `image-to-image` | `i2i` | `in-image`, `in-text` | `out-image` | `model`, `aspectRatio`, `resolutionLevel` |
| `image-super-resolution` | `vsr` image slot | `in-image` | `out-image` | `model`, `vsr_preset` |

Example:

```json
{
  "id": "hero_image",
  "tool_id": "text-to-image",
  "name": "文生图",
  "position": {"x": 420, "y": 120},
  "data": {
    "model": "gpt-image-2",
    "aspectRatio": "9:16",
    "resolutionLevel": "720p"
  }
}
```

### Video Generation Nodes

| tool_id | Task | Inputs | Outputs | Important data |
| --- | --- | --- | --- | --- |
| `video-gen-text` | `t2v` | `in-text` | `out-video` | `model`, `aspectRatio`, `resolutionLevel` |
| `video-gen-image` | `i2v` | `in-image`, `in-text` | `out-video` | `model`, `aspectRatio`, `resolutionLevel` |
| `video-gen-dual-frame` | `flf2v` | `in-image-start`, `in-image-end`, `in-text` optional | `out-video` | `model`, `aspectRatio`, `resolutionLevel` |
| `video-gen-text-with-audio` | `t2av` | `in-text` | `out-video` | `model`, `aspectRatio`, `resolutionLevel`, `videoDurationSeconds`, `loraStylePlugins` |
| `video-gen-image-with-audio` | `i2av` | `in-text`, `in-image` | `out-video` | `model`, `aspectRatio`, `resolutionLevel`, `videoDurationSeconds`, `imageKeyframeTimes`, `image_strength`, `i2av_row_order`, `loraStylePlugins` |
| `video-super-resolution` | `vsr` video slot | `in-video` | `out-video` | `model`, `vsr_preset` |

For MiniMax-H3 nodes, set `resolutionLevel` explicitly when the user requests a tier:
`544p` or `768p`. Omitting it selects the platform default (`544p`). A custom pixel
shape is not a substitute for `resolutionLevel`, and 768P does not require a separate
super-resolution node.

Example image-to-video node:

```json
{
  "id": "motion_video",
  "tool_id": "video-gen-image",
  "name": "图生视频",
  "position": {"x": 760, "y": 180},
  "data": {
    "model": "Wan2.2_I2V_A14B_distilled",
    "aspectRatio": "9:16",
    "resolutionLevel": "720p"
  }
}
```

For `video-gen-image-with-audio` multi-keyframes, connect multiple image-producing nodes into `in-image`; the workflow runtime flattens the images. If using direct submit JSON outside workflow, use one wrapper:

```json
{
  "input_image": {"type": "base64", "data": ["IMG1_BASE64", "IMG2_BASE64"]},
  "image_keyframe_times": [0, 2],
  "image_strength": [0.9, 0.9]
}
```

### Digital Human And Motion Nodes

| tool_id | Task | Inputs | Outputs | Important data |
| --- | --- | --- | --- | --- |
| `avatar-gen` | `s2v` | `in-image`, `in-video` optional, `in-audio`, `in-text` optional | `out-video` | `model`, `prompt`, `aspectRatio` |
| `animate` | `animate` | `in-video`, `in-image`, `in-text` optional | `out-video` | `model`, `prompt`, `replace_flag`, `retarget_flag` |

`avatar-gen` accepts either an image or a video as the visual source depending on the model. For the common single portrait talking-head case, connect `image-input.out-image` to `avatar-gen.in-image` and `tts.out-audio` or `audio-input.out-audio` to `avatar-gen.in-audio`.

Example:

```json
{
  "id": "avatar_video",
  "tool_id": "avatar-gen",
  "name": "数字人视频",
  "position": {"x": 780, "y": 240},
  "data": {
    "model": "SekoTalk-V3",
    "prompt": "Natural talking-head delivery",
    "aspectRatio": "9:16"
  }
}
```

`animate` modes:

- `replace_flag: true`: character replacement / motion transfer style.
- `replace_flag: false` with `retarget_flag: true`: retarget motion where supported.

## Connection Rules

Connect matching data types:

| Source port | Target ports |
| --- | --- |
| `out-text` | `in-text`, `in-context-tone` |
| `out-image` | `in-image`, `in-image-start`, `in-image-end` |
| `out-audio` | `in-audio` |
| `out-video` | `in-video` |

Common chains:

```text
text-input.out-text -> text-to-image.in-text -> text-to-image.out-image
image-input.out-image + text-input.out-text -> video-gen-image -> out-video
text-input.out-text -> tts.in-text -> tts.out-audio
image-input.out-image + tts.out-audio -> avatar-gen -> out-video
audio-input.out-audio -> asr-transcribe.in-audio -> text-generation.in-text
video-input.out-video + image-input.out-image -> animate -> out-video
```

Multiple connections into the same `in-image` are allowed for image-list-capable tools such as `image-to-image` and `video-gen-image-with-audio`. Avoid multiple connections into `in-audio` or `in-video` unless the backend explicitly supports that model.

## Data Key Translation To Submit Payload

Generation-node `data` is translated to task submit fields:

| node.data key | submit field |
| --- | --- |
| `model` | `model_cls` |
| `aspectRatio` | `aspect_ratio` |
| `resolutionLevel` | `resolution_level` |
| `videoDurationSeconds` | `video_duration_seconds` |
| `imageKeyframeTimes` | `image_keyframe_times` |
| `image_strength` | `image_strength` |
| `loraStylePlugins` | `lora_style_plugins` |
| `vsr_preset` | `vsr_preset` |
| `replace_flag` | `replace_flag` |
| `retarget_flag` | `retarget_flag` |

Text connected to `in-text` becomes `prompt` for generation nodes. Media connected to `in-image`, `in-audio`, `in-video`, `in-image-start`, or `in-image-end` becomes `input_image`, `input_audio`, `input_video`, or `input_last_frame`.

## Workflow Patterns

### Portrait + Script To Digital Human

Nodes:

- `text-input` `script_input`
- `image-input` `portrait_input`
- `tts` `voice_tts`
- `avatar-gen` `avatar_video`

Connections:

- `script_input.out-text -> voice_tts.in-text`
- `voice_tts.out-audio -> avatar_video.in-audio`
- `portrait_input.out-image -> avatar_video.in-image`

### Existing Audio + Portrait To Digital Human

Nodes:

- `image-input` `portrait_input`
- `audio-input` `speech_input`
- `avatar-gen` `avatar_video`

Connections:

- `portrait_input.out-image -> avatar_video.in-image`
- `speech_input.out-audio -> avatar_video.in-audio`

### Prompt To Image To Video

Nodes:

- `text-input` `prompt_input`
- `text-to-image` `image_gen`
- `video-gen-image` `video_gen`

Connections:

- `prompt_input.out-text -> image_gen.in-text`
- `image_gen.out-image -> video_gen.in-image`
- `prompt_input.out-text -> video_gen.in-text`

### Product Image To Edited Image To Product Video

Nodes:

- `image-input` `product_input`
- `text-input` `creative_prompt`
- `image-to-image` `style_image`
- `video-gen-image-with-audio` `product_video`

Connections:

- `product_input.out-image -> style_image.in-image`
- `creative_prompt.out-text -> style_image.in-text`
- `style_image.out-image -> product_video.in-image`
- `creative_prompt.out-text -> product_video.in-text`

### Audio Or Video To Transcript To Rewrite

Nodes:

- `audio-input` or `video-input`
- `asr-transcribe`
- `text-generation`

Connections:

- `audio_input.out-audio -> transcript.in-audio` or `video_input.out-video -> transcript.in-video`
- `transcript.out-text -> rewrite.in-text`

## Validation Checklist

Before creating/running a workflow:

- Every non-input node has all required input ports connected.
- Port data types match.
- Every generation node has a real `data.model`.
- TTS preset nodes have both `voiceType` and `resourceId`; clone TTS has `speakerId`.
- `text-generation.custom_outputs` ids match any downstream `source_port_id` that consumes them.
- `video-gen-dual-frame` uses `in-image-start` and `in-image-end`, not generic `in-image`.
- Runtime-only values are passed via `workflow run --inputs` / `--input-file`, not by editing the workflow for one run.
- Localhost runs behind corporate proxies include `NO_PROXY=127.0.0.1,localhost no_proxy=127.0.0.1,localhost`.
