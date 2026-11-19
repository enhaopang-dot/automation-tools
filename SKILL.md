---
name: automation-tools
version: 1.0.0
description: |
  自动化工具参考：SSH 远程操作（先写脚本再 scp 上传执行）、飞书发送、
  PDF 文本提取降级方案（read_pdf.py）、后台任务输出可见性、免费行情源抓取防风控、
  PC 定时提醒（弹窗+飞书）。
  当需要 SSH 远程执行多行脚本、发送飞书消息/文件、提取 PDF 文本、排查后台任务输出、
  或设置定时提醒时，使用此 skill。
triggers:
  - SSH
  - 远程服务器
  - 飞书发送
  - 飞书
  - read_pdf
  - PDF 提取
  - 后台任务
  - 定时提醒
  - 批量抓取
  - scp 上传
allowed-tools:
  - Bash
---

# 自动化工具参考

> 迁移记录：原 `D:/Claude/配置/.claude/rules/automation-tools.md`（常驻自动加载），2026-09-24 改为 skill 按需加载。命中触发词时必须调用本 skill。

> **Owner**：自动化与远程操作 ｜ **触发**：SSH 远程 / 飞书 / PDF 提取 / 后台任务 / 定时提醒 / 批量抓取 ｜ **产出**：可复跑脚本 + 可见输出 ｜ **禁止**：复杂多行命令 inline、后台输出串管道 ｜ **验收**：脚本入库、输出可追踪
> 分级：[P1]

## SSH / 远程服务器操作（2026-07-30 固化）

SSH 远程执行多行、含 `async with` 或多层引号的 Python/SQL 命令时，一律先写本地脚本再 `scp` 上传执行，禁止 inline。

- 流程：本地写入 `_构建脚本/` → `scp` 到服务器 → `.venv/bin/python _scripts/xxx.py` → 视情况清理。
- 例外：单行简单命令（`ls`、`systemctl status`、`grep`）可直接 inline。

## 飞书发送

```bash
python D:/Claude/配置/.claude/tools/feishu_send.py <文件路径> [附言]
python D:/Claude/配置/.claude/tools/feishu_text.py "消息内容"
```

## PDF 文本提取（降级方案）

```bash
python D:/Claude/配置/.claude/tools/read_pdf.py <文件路径> [-o 输出文件]
```

默认直接 Read；>100 页或 >30MB、纯文字想省 token、Read 失败时回退此脚本。

## 后台 Python 任务输出可见性

后台任务（`run_in_background`）进度 print 会"消失"：
- `python -u` 或 `print(..., flush=True)`
- 不串任何管道，过滤在 Python 内完成
- 长循环周期性打印（如每 500 次迭代带计数和耗时）

## 免费行情源抓取防风控（2026-07-19 选股流水线实战）

单一免费源扛不住全市场 5000+ 连续抓取：
- 批量接口粗筛，只对幸存者逐只拉 K 线
- K 线准备 ≥3 源，按连续失败数动态调序
- 每 500 只休息 25 秒；并发 ≤16 线程
- 本地缓存，断点续传

## PC 定时提醒（2026-07-21 固化）

```bash
python D:/Claude/配置/.claude/tools/pc_reminder.py --time 21:00 --msg "该做XX了"
python D:/Claude/配置/.claude/tools/pc_reminder.py --time 09:30 --date 2026-08-14 --title "晨会" --msg "9点半晨会"
python D:/Claude/配置/.claude/tools/pc_reminder.py --time 21:00 --msg "..." --no-feishu
```

- 默认电脑弹窗 + 飞书同步；`--no-feishu` 只弹窗
- `--date` 省略为今天，时间已过顺延明天
- Windows 计划任务 + PowerShell 弹窗 + 铃声；飞书通道先发送再弹窗
