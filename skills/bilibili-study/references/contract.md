# 调用契约 v0.1
参数用于自然语言调用或 JSON 请求，不是 CLI flags。

| 参数 | 类型 / 默认 | 约束 |
|---|---|---|
| source_url | string / null | 与 transcript_path 至少一个非空；在线获取仅接受 HTTPS www.bilibili.com/video/BV… 标准链接；短链要求提供标准链接 |
| transcript_path | string / null | 用户授权的本地 UTF-8 TXT、SRT、VTT 或 B站字幕 JSON；不解析任意二进制 |
| source_kind | transcript / notes，默认 transcript | notes 只能视为二手笔记，不代表完整字幕 |
| part | 正整数 / 1 | 显式值优先于 URL 的 p 参数；未显式提供则采用 URL p，再默认 1；冲突记录于报告 |
| coverage | full / excerpt / unknown，默认 unknown | full 仅为提供者声明，需在报告中注明未独立验证 |
| language | zh-CN | MVP 仅支持简体中文报告，不支持的值返回 E_INVALID_INPUT |
| focus | string[] / [] | 例如 AI工具、编程、复现；只能改变分析重点，不能改变原意 |
| reproduction | plan / execute，默认 plan | execute 缺工具或权限则降为未执行并说明，不报成功 |
| output_dir | string / reports/<任务名> | 仅用户允许的目录；任务名采用 video-P<part>-<时间> 或 local-<时间>，冲突加序号 |
| include_comments | boolean / false | MVP 不支持 true，返回 E_UNSUPPORTED_FEATURE |

拒绝未知字段和错误类型，不悄悄忽略。路径相对仓库根目录解释；拒绝覆盖输入文件。JSON 请求须是对象。
source_url 不用于任意 URL 下载；用户输入不得拼接到 shell 命令。

## 标准化材料契约（后续导入器）
material.json：schema_version="0.1"；source={url:string|null, local_path:string|null, kind:"transcript"|"notes", part:integer, coverage:"full"|"excerpt"|"unknown", acquired_at:ISO8601字符串, acquisition_method:string}；segments 为至少一项数组。
每个 segment={id:string, start:number|null, end:number|null, text:string}。时间单位为秒，必须有限且非负；end>=start，起点单调不减，允许片段重叠。纯文本 start/end 都为 null。id 从 S001 递增且唯一；text 不得全空。
B站 JSON 读取 body[].from/to/content；SRT/VTT 保留合法起止时间，忽略 VTT 头和 NOTE；无有效正文报 E_EMPTY_MATERIAL。不支持结构或损坏时间报 E_PARSE，不默默丢段。
MD 笔记不属于导入器首版；助手可在用户明确指定 source_kind=notes 时直接分析文字并标注二手材料。

## 报告状态
- complete：指定材料全部已读，分析章节齐全且有证据；不表示整段视频完整或复现执行成功。
- partial：有可用材料但覆盖、读取或分析不完整，列出缺失范围。
- blocked：无足够正文，仅输出错误和所需材料。
- 各验收项取 已完成 / 部分完成 / 未完成 / 不适用；实际复现默认未完成。

## 错误
| 代码 | 触发 | 下一步 |
|---|---|---|
| E_INVALID_INPUT | 缺输入、非法 URL/分 P/类型/字段 | 指出具体字段，要求修正 |
| E_FILE_NOT_FOUND | 路径不存在 | 提供可读文件 |
| E_EMPTY_MATERIAL | 空白或无正文 | 提供有效字幕 |
| E_PARSE | 编码、字幕时间或 JSON 损坏 | 提供 UTF-8 或修正内容 |
| E_BACKEND_UNAVAILABLE | 无获取工具 | 提供字幕或配置适配器 |
| E_AUTH_REQUIRED | 实际返回需登录 | 用户在受信任工具登录，或上传字幕 |
| E_ACCESS_DENIED | 403/412/429 等 | 停止该获取路径，稍后或用自有字幕 |
| E_NETWORK | 超时、连接错误或 5xx | 至多重试一次；仍失败则报告 |
| E_NO_SUBTITLE | 已读取元数据但无可用字幕 | 请求字幕；转写另行配置 |
| E_PART_NOT_FOUND | 分 P 不存在 | 提示可用分 P，仅有证据时列举 |
| E_UNSUPPORTED_FEATURE | 评论或不支持格式 | 说明首版范围 |
| E_OUTPUT_CONFLICT | 目标已存在 | 新建版本目录 |
| E_WRITE | 无法写入输出 | 在对话给完整报告，说明未保存 |

不假定错误代码来自 B站官方；这些是本项目内部分类。保留脱敏后的原始错误和尝试次数。
