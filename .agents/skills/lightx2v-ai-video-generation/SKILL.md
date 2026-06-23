---
name: lightx2v-ai-video-generation
description: "Run LightX2V AI generation tasks from the command line via OpenAPI. Commands: lightx2v login, models, run, query, list, cancel, resume, delete, result, completion. Tasks: t2i, t2v, i2v, s2v, flf2v, t2av, i2av, vsr, animate, i2i. Use for: scriptable video/image generation, CI pipelines, agent automation, batch jobs. Triggers: lightx2v cli, lightx2v run, api key generation, text to video cli, text to image cli, download generated video, poll task status"
allowed-tools: Bash(lightx2v *)
---

> **Install skill:** `npx skills add ModelTC/LightX2V-Skills@lightx2v-ai-video-generation`
>
> **Install CLI:** `curl -fsSL https://raw.githubusercontent.com/ModelTC/lightx2v-studio-cli/main/install.sh | sh`
>
> **Skills directory:** https://skills.sh/ModelTC/LightX2V-Skills

# LightX2V AI Video Generation

Submit and download AI generation tasks via the LightX2V OpenAPI.

**Online API docs:** https://x2v.light-ai.top/api-docs

## Quick Start

```bash
curl -fsSL https://raw.githubusercontent.com/ModelTC/lightx2v-studio-cli/main/install.sh | sh
lightx2v login
lightx2v models
lightx2v run t2i/Qwen-Image-2512 --prompt "a cute cat" --shape 512,512 -o out.png
```

## Commands

| Command | Description |
| --- | --- |
| `lightx2v login [--base-url URL]` | Save apikey_ to `~/.config/lightx2v/config.json` |
| `lightx2v models [--json]` | List task/model combinations |
| `lightx2v run <task>/<model_cls> ...` | Submit, poll, download (main workflow) |
| `lightx2v query <task_id> [--json]` | Query single task |
| `lightx2v list [--status] [--page]` | Paginated task list |
| `lightx2v cancel <task_id>` | Cancel running task |
| `lightx2v resume <task_id>` | Resume failed/cancelled task |
| `lightx2v delete <task_id>` | Delete finished task |
| `lightx2v result <task_id> [-o PATH]` | Get result URL / download |
| `lightx2v completion bash` or `zsh` | Shell completion script |

## `run` options

| Flag | Description |
| --- | --- |
| `--prompt` | Prompt text |
| `--input JSON` or `--input @file.json` | Extra submit body fields |
| `--image` / `--video` / `--audio` | Local file to base64 input |
| `--shape H,W` | `custom_shape` e.g. `720,1280` |
| `--aspect-ratio` | e.g. `16:9` |
| `--duration SEC` | Seconds (t2av/i2av/s2v) |
| `--vsr-preset` / `--vsr-input-slot` | VSR options |
| `-o PATH` | Output file |
| `--quote` | Pre-submit billing quote |
| `--json` | Structured JSON event stream |
| `-q` / `--quiet` | Minimal output |
| `--no-download` | Return URL only |

## Examples

### Text-to-image

```bash
lightx2v run t2i/Qwen-Image-2512 --prompt "watercolor portrait" --aspect-ratio 16:9 -o portrait.png
```

### Text-to-video

```bash
lightx2v run t2v/Wan2.2_T2V_A14B_distilled --prompt "waves at sunset" --shape 720,1280 -o beach.mp4
```

### Image-to-video

```bash
lightx2v run i2v/Wan2.2_I2V_A14B_distilled --prompt "subject turns and smiles" --image ./ref.png -o animated.mp4
```

### Digital human (s2v)

```bash
lightx2v run s2v/SekoTalk --prompt "natural speaking" --image ./portrait.png --audio ./speech.wav -o avatar.mp4
```

### Task management

```bash
lightx2v query TASK_ID
lightx2v list --status SUCCEED --page 1 --page-size 20
lightx2v cancel TASK_ID
lightx2v result TASK_ID -o output.png
```

## Notes

- API channel costs **2x** web UI credits; `run` prints `quoted_credits`.
- Config: `~/.config/lightx2v/config.json` (mode 0600).
- Env: `LIGHTX2V_API_KEY`, `LIGHTX2V_CLOUD_API_KEY`, `LIGHTX2V_BASE_URL`.
- Large media: prefer `url` in `--input` JSON over base64.

## Related

- Skills directory: https://skills.sh/ModelTC/LightX2V-Skills
- Skills repo: https://github.com/ModelTC/LightX2V-Skills
- CLI repo: https://github.com/ModelTC/lightx2v-studio-cli
- OpenAPI: https://x2v.light-ai.top/openapi.json
- Platform: https://x2v.light-ai.top
