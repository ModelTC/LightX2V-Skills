# LightX2V-Skills

[English](README.md)

**LightX2V** 的 Agent Skills 仓库，让 Cursor 等 AI 助手通过 [x2v.light-ai.top](https://x2v.light-ai.top) 完成 AI 图像/视频生成。

**Skills 目录：** https://skills.sh/ModelTC/LightX2V-Skills

## Skills

| Skill | 用途 |
| --- | --- |
| `lightx2v-ai-video-generation` | 使用直接 HTTP 或 `lightx2v` CLI 生成图片、视频、数字人口播、音视频、图片超分 ISR、视频超分 VSR、动作迁移、TTS、音色克隆和工作流任务，并按实时接口发现能力。 |

## 安装 Skill

```bash
npx skills add ModelTC/LightX2V-Skills@lightx2v-ai-video-generation -g -y
```

安装本仓库全部 Skills：

```bash
npx skills add ModelTC/LightX2V-Skills -g -y
```

## API 信息来源

- 网站文档：https://x2v.light-ai.top/api-docs
- Agent 总索引：https://x2v.light-ai.top/llms.txt
- Agent 可读 Markdown：https://x2v.light-ai.top/api-docs.md
- OpenAPI 结构：https://x2v.light-ai.top/openapi.json
- 实时模型目录：https://x2v.light-ai.top/api/v1/model/list
- 按任务查询实时规格：`https://x2v.light-ai.top/api/v1/model/specs?task=...`
- Skill：发现并组合上述来源，再选择直接 HTTP 或 CLI 执行。

当前模型和可能变化的媒体限制以实时模型规格为准；请求与响应字段以 OpenAPI 为准；Markdown 负责操作流程和恢复策略。

## CLI

Skill 会教 Agent 如何使用 `lightx2v` 命令行。CLI 需从独立仓库单独安装：

```bash
curl -fsSL https://raw.githubusercontent.com/ModelTC/lightx2v-studio-cli/main/install.sh | sh
```

或：`pip install "git+https://github.com/ModelTC/lightx2v-studio-cli.git"`

CLI 仓库：https://github.com/ModelTC/lightx2v-studio-cli

## 相关链接

- 平台：https://x2v.light-ai.top
- API 文档：https://x2v.light-ai.top/api-docs
- Agent 可读文档：https://x2v.light-ai.top/llms.txt
- OpenAPI：https://x2v.light-ai.top/openapi.json

## 许可证

MIT
