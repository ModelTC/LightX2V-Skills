# LightX2V-Skills

[中文](README.zh-CN.md)

Agent skills for **LightX2V** — AI image/video generation via [x2v.light-ai.top](https://x2v.light-ai.top).

**Skills directory:** https://skills.sh/ModelTC/LightX2V-Skills

## Skills

| Skill | Use |
| --- | --- |
| `lightx2v-ai-video-generation` | Use the `lightx2v` CLI / OpenAPI for image, video, digital-human, audio-video, VSR, animate, TTS, voice clone, and workflow tasks. Includes `references/workflow-nodes.md` and `references/voice-tts.md`. |

## Install

```bash
npx skills add ModelTC/LightX2V-Skills@lightx2v-ai-video-generation -g -y
```

Install all skills from this repo:

```bash
npx skills add ModelTC/LightX2V-Skills -g -y
```

## CLI

The skill teaches your agent how to use `lightx2v`. Install the CLI from the separate repo:

```bash
curl -fsSL https://raw.githubusercontent.com/ModelTC/lightx2v-studio-cli/main/install.sh | sh
```

Or: `pip install "git+https://github.com/ModelTC/lightx2v-studio-cli.git"`

CLI repo: https://github.com/ModelTC/lightx2v-studio-cli

## Links

- Platform: https://x2v.light-ai.top
- API docs: https://x2v.light-ai.top/api-docs
- OpenAPI: https://x2v.light-ai.top/openapi.json

## License

MIT
