# Computer Networking（Stanford CS144）

Stanford CS144 *Introduction to Computer Networking* 课程资料整理。

## 目录结构

```
01-课堂讲义/      课堂（Live Lecture）讲义
02-慕课/          MOOC 单元讲义
    Unit1-Internet-and-IP
    Unit2-Transport
    Unit3-Packet-Switching
    Unit4-Congestion-Control
    Unit5-Applications-and-NATs
    Unit6-Routing
    Unit7-Lower-Layers
    Unit8-Security
03-实验Lab/       Lab 讲义与说明
04-考试/          模拟考试与解答
05-教材/          Peterson & Davie《Computer Networking: A Systems Approach》v6.1
06-视频字幕/       课程视频中文字幕（01-主课 143 个 / 02-嘉宾讲座 8 个）
00-课程计划.pdf
```

## 说明

- `02-慕课/` 按 **Unit（单元）** 为粒度组织，与课程进度对应。
- `06-视频字幕/` 为视频配套的 `.srt` 中文字幕（编号与视频集数对应）。
- 其余材料按性质在**顶层一层**归类，不再向下细分。
- 课程材料版权归 Stanford 与原作者所有；教材版权归出版方所有。

## 目录结构

> 2026-11 目录归一化后的实际结构（上方旧的「目录结构」一节保留为历史记录，已不再描述现行布局）。

```
01-教材/                教材原书 PDF（Peterson & Davie v6.1）
02-讲义/                讲义 md + 课程计划 PDF（平铺，不设子目录）
视频/                   视频字幕 .srt
    01-主课/            主课字幕 143 个
    02-嘉宾讲座/        嘉宾讲座与补充字幕 8 个
04-题目/                官方题目原文 + 译本
    作业/               Lab 0–6 实验指导、抓包练习题目
    考试/               期中/期末练习题目
    题解/               抓包练习与期中/期末练习答案
06-工具/                llms.txt（给模型读的仓库索引）
README.md / LICENSE / NOTICE / .gitignore    仓库元文件，原地不动
```

- 目录以 `0N-` 前缀统一为 7 类：教材 / 讲义 / 已处理文件 / 题目 / 代码 / 工具 / 待定；本仓只用到其中的 01、02、04、06，另有顶层 `视频/` 收全部字幕。
- `视频/` 下按「主课 / 嘉宾讲座」两源保留一层分组，避免 143+8 份字幕平铺。
- 全部搬移均用 `git mv`，文件内容零改动；唯一的内容变更是本节追加（字幕随后由 `02-讲义/字幕/` 迁至 `视频/`）。
