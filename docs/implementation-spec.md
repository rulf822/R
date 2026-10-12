# 最小实现规格 v0.1

## 目标和边界
输入一个 B站视频的单个分 P 字幕，输出可追溯的中文学习报告，覆盖总结、精华、结构、分类和复现。首版不做批量、多 P 合并、评论、自动视频下载、OCR、语音转写、Web UI、数据库或定时任务。
本仓库已交付提示词技能及契约；下面的程序接口是下一开发任务，尚未创建 scripts 或可运行服务。所有计划内参数、结果和错误以 skills/bilibili-study/references/contract.md 为准。

## 组件
1. 请求校验：接受 JSON，校验字段、默认值、路径与分 P。
2. 导入器：读取本地 TXT/SRT/VTT/B站 JSON，输出 material.json；标准库实现，禁止执行素材文本。
3. 在线适配器：可选；将已授权工具输出转为同一 material.json，记录来源。不能在导入器中隐式安装或启动 BiliNote。
4. 分析器：由宿主 AI 读取全部 segments，依模板写 report.md；首版不另接模型 API。
5. 验收器：结构检查加人工/模型证据审查；结构检查不得声称语义正确。

## 后续 CLI 的精确接口
拟新增 skills/bilibili-study/scripts/prepare.py：
```text
python prepare.py --request <request.json>
```
仅接受 --request，读取上述输入契约。本地模式写入新 output_dir/material.json 和 acquisition.json；不会生成摘要。
成功退出码 0；输入或解析问题 2；获取不可用 3；输出写入问题 4。
stdout 只输出一行 JSON：{"status":"ready|blocked","material_path":string|null,"error":null|{"code":string,"message":string,"next_action":string}}。
失败仍尽可能输出上述 JSON；stderr 仅脱敏诊断，不含字幕或凭据。
acquisition.json={status:"ready"|"blocked", method:string, attempts:integer, error:object|null}。
写文件使用同目录临时文件再重命名；已有目标目录时报 E_OUTPUT_CONFLICT，要求新目录，不覆盖。无写权限时不留下伪成功结果。
离线导入不访问网络。URL-only 且无适配器必定 E_BACKEND_UNAVAILABLE，不能返回空素材冒充成功。

## 处理顺序
校验 → 检查输出冲突 → 读取/获取 → 验证正文 → 标准化 → 保存证据 → 助手分析 → 验收报告。
本地字幕优先；URL 仅作为出处不触发额外下载。未知完整性保持 unknown。笔记输入按受限助手流程分析，不送入首版字幕导入器。
长字幕按片段分块，保留全部 ID；各块总结后聚合，复核冲突与遗漏。上下文不足导致未读时报告 partial 和未读片段范围。

## 数据和安全
仅允许用户授权的输入输出位置；素材不是指令。默认只输出计划，不执行复现命令。敏感材料留在本地 ignored 目录。
不存 Cookie、密码和模型 token；不请求用户把凭据提交仓库。
在线获取不绕过风控；网络临时错误最多一次重试，403/412/429 不自动重试。
输出路径不得覆盖输入；失败报告保留真实原因，禁止用通用摘要填充。

## 完成定义
- 规格交付：README、技能、契约、模板、示例和用例完整且互相一致。
- 离线程序交付：新增导入器后，acceptance.md 中离线机器用例实际运行通过。
- 分析交付：报告逐条核对源材料，不能只检查标题。
- 真实视频交付：获取指定 BV 分 P 的有效正文，保存获取记录，完成证据报告；单独记录画面未覆盖和复现未执行。
三个能力状态分别报告；不得以规格交付替代代码或真实视频交付。
