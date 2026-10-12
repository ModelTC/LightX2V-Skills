# LightX2V-Skills

[中文](README.zh-CN.md)

Agent skills for **LightX2V** — AI image/video generation via [x2v.light-ai.top](https://x2v.light-ai.top).

**Skills directory:** https://skills.sh/ModelTC/LightX2V-Skills

## Skills

| Skill | Use |
| --- | --- |
| `lightx2v-ai-video-generation` | Use direct HTTP or the `lightx2v` CLI for image, video, digital-human, audio-video, ISR/VSR, animate, TTS, voice clone, and workflow tasks. Includes live API discovery and execution references. |

## Install

```bash
npx skills add ModelTC/LightX2V-Skills@lightx2v-ai-video-generation -g -y
```

Install all skills from this repo:

```bash
npx skills add ModelTC/LightX2V-Skills -g -y
```

## API sources

- Website: https://x2v.light-ai.top/api-docs
- Agent index: https://x2v.light-ai.top/llms.txt
- Agent-readable Markdown: https://x2v.light-ai.top/en/api-docs.md
- OpenAPI schema: https://x2v.light-ai.top/openapi.json
- Live model catalog: https://x2v.light-ai.top/api/v1/model/list
- Task-scoped model specs: `https://x2v.light-ai.top/api/v1/model/specs?task=...`
- Skill: discovers and combines these sources, then chooses direct HTTP or the CLI.

Live model specs are authoritative for current models and mutable media constraints. OpenAPI defines exact request and response shapes; Markdown explains workflows and recovery behavior.

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
- Agent-readable docs: https://x2v.light-ai.top/llms.txt
- OpenAPI: https://x2v.light-ai.top/openapi.json

## License

MIT
