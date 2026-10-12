# 总结归类 · Bilibili 视频分析
本仓库提供 Bilibili 视频分析技能的最小实现规格及完整说明、输入示例、输出模板和验收用例。版本：0.1.0。

**交付状态：规格与提示词技能初稿。** 可让具备文件读取能力的 AI 助手按技能处理用户提供的字幕；尚未包含自动下载、字幕解析 CLI、模型 API 服务或 BiliNote 部署。下载成功、生成文件、格式检查通过均不等于视频分析验收通过。

## 目录
```text
R/
├── README.md
├── AGENTS.md
├── .gitignore
├── docs/
│   ├── implementation-spec.md
│   └── acceptance.md
├── examples/
│   ├── request.json
│   └── transcript.txt
└── skills/bilibili-study/
    ├── SKILL.md
    ├── agents/openai.yaml
    ├── references/contract.md
    └── assets/report-template.md
```
以上所有文件均为完整首版内容；输出目录 reports/ 在调用时创建，不提交真实字幕、笔记或凭据。

## 安装位置
克隆本仓库到 Windows，例如 C:\Projects\R（命令在 PowerShell 执行）：
```powershell
git clone https://github.com/rulf822/R.git
Set-Location R
```
项目技能源文件位于 skills/bilibili-study/SKILL.md。最小用法不依赖客户端自动发现：在打开此仓库的 AI 编程助手中，明确要求读取该文件，并提供素材路径。
若客户端支持个人技能目录，可将整个 bilibili-study 文件夹复制到其配置的 skills 目录；默认 Codex 目录通常为 $HOME/.codex/skills，设置了 CODEX_HOME 时使用其下的 skills。是否已加载以客户端技能列表为准。不要仅复制 SKILL.md，不要覆盖同名已有技能。仓库文件不会自动安装到网页版 ChatGPT 项目，也不会启动 BiliNote 服务。

## 立即调用
打开仓库后发送：
> 请读取 skills/bilibili-study/SKILL.md，使用 examples/request.json 作为参数，分析 examples/transcript.txt，把报告写入 reports/demo/report.md。这是合成验收材料，不是真实视频。

安装并确认技能已加载后，可说：
> 使用 $bilibili-study，分析我上传的字幕，按标准模板输出总结、精华、结构和复现清单。

真实视频调用：
> 请按 skills/bilibili-study/SKILL.md 分析 https://www.bilibili.com/video/BV1Nd596vEyU ，只分析第 1 P。若无法获取字幕，请准确报告阻塞原因，不要凭标题总结。

request.json 是助手输入契约，**不是已实现的命令行程序**。只给链接时，需要宿主已有、获授权的素材获取工具；若无工具，报告 E_BACKEND_UNAVAILABLE，并请求提供 SRT/VTT/TXT/JSON 字幕。已有 bilinote-video-study 可作为可选适配器，需现场检查，不能假定已配置。

## 依赖
- 基础路径：能够读取 UTF-8 文件并理解字幕的 AI 助手；无必需 pip 包、Node 包、API key。
- Git 仅用于克隆与版本管理。
- 后续确定性导入器：拟用 Python 3.11+ 标准库；尚未实现，接口见 docs/implementation-spec.md。
- 在线获取与转写是可选外部能力，需单独验证网络、权限和依赖；不预装、不暗示可用。

## 验收标准
详见 docs/acceptance.md。必须分别检查：
1. 材料来源、范围、分 P 和完整性有记录，正文事实可追溯到片段。
2. 总结、精华、结构、复现清单、分类和验收表均有实质内容。
3. 缺参数、无画面、只有片段时明确标注；不编造时间戳或操作结果。
4. 无字幕、登录限制、错误链接和不支持输入均给出明确错误与下一步。
5. “复现步骤已整理”和“实际执行已成功”独立判定。

当前没有 BV1Nd596vEyU 的获取或端到端验收记录。离线合成样例通过也不能替代真实视频验收。
