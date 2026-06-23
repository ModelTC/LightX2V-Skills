# LightX2V-Skills

[English](README.md)

**LightX2V** 的 Agent Skills 仓库，让 Cursor 等 AI 助手通过 [x2v.light-ai.top](https://x2v.light-ai.top) 完成 AI 图像/视频生成。

**Skills 目录：** https://skills.sh/ModelTC/LightX2V-Skills

## 安装 Skill

```bash
npx skills add ModelTC/LightX2V-Skills@lightx2v-ai-video-generation -g -y
```

安装本仓库全部 Skills：

```bash
npx skills add ModelTC/LightX2V-Skills -g -y
```

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
- OpenAPI：https://x2v.light-ai.top/openapi.json

## 许可证

MIT
