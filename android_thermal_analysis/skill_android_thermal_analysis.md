---
name: android-thermal-power-rca
version: v0.0.0
description: 自动分析 Android 系统的发热、功耗异常及高温关机问题，并生成结构化分析报告。
---

# Android 系统性能与功耗分析 (android-thermal-power-rca)

## 简介与适用场景

本 Skill 是一个用于 Android 系统性能与功耗分析的自动化工具，专注于排查设备**异常发热、耗电过快、高温关机**等问题。它通过深度解析 logcat 日志，从温度、环境、电池、功耗、前台应用、底层负载等多个维度提取关键指标，构建事件时间线，并最终生成一份包含根本原因和优化建议的结构化中文报告。

### 何时使用

当您遇到以下情况时，应使用此 Skill：

- **现象**: 设备在日常使用或待机时感觉明显发热。
- **现象**: 电池电量下降速度远超预期，续航表现差。
- **问题**: 设备因温度过高而自动关机（Thermal Shutdown），需要进行事后分析。
- **任务**: 需要对应用的功耗和发热表现进行评估和优化。

## 输入要求

- **输入类型**: 一个包含 Android 日志文件的目录路径。
- **文件格式**:
    - Skill 会自动扫描并处理输入目录下的所有文本文件。
    - 目录中应至少包含 `logcat_main.txt` 或 `logcat_system.txt` 之一，也支持 `logcat.txt`, `dmesg.txt` 等近似命名。
    - 日志文件必须为 **UTF-8** 编码。

## 执行步骤

通过调用封装好的脚本 `run_analysis.sh` 来启动整个分析流程。

```bash
# 示例：分析位于 /home/user/logs/case-123/ 目录下的日志
sh /path/to/android-thermal-power-rca/scripts/run_analysis.sh /home/user/logs/case-123/
```

脚本会依次执行以下步骤：

1.  **提取指标**: 调用 `scripts/extract_metrics.py`，从所有日志文件中提取与功耗、发热相关的原始数据，并保存为 `outputs/metrics.json`。
2.  **构建时间线**: 调用 `scripts/build_timeline.py`，根据提取的指标构建关键事件时间线，并保存为 `outputs/timeline.json` (中间文件)。
3.  **生成报告**: 调用 `scripts/render_report.py`，结合 `metrics.json` 和 `timeline.json`，使用 `assets/report_template.md` 模板，最终渲染出详细的分析报告 `outputs/report.md`。

## 输出规范

分析完成后，将在执行目录的 `outputs/` 子目录中生成以下文件：

### 1. 分析报告 (`outputs/report.md`)

一份详细的中文 Markdown 报告，包含以下核心部分：

- **关键指标时间线 (Timeline)**: 清晰展示各维度关键事件（如温度攀升、应用切换、充电状态改变等）随时间变化的“故事线”。
- **各维度异常单项分析**:
    - **温度演进**: 峰值温度、关键升温节点，单位为摄氏度 (°C)。
    - **环境光**: 判断是否暴晒，单位为勒克斯 (lux)。
    - **充放电与电池状态**: 充电类型、电量变化趋势。
    - **整机功耗**: 区间平均电流 (mA)、累计耗电 (mAh)。
    - **前台应用与屏幕状态**: 确认高耗电场景下的前台应用和屏幕状态。
    - **底层负载与高耗电进程**: 找出 CPU 占用高的“元凶”进程。
    - **其他发热强相关信息**: 通信、硬件、内核等异常。
- **结构化的根本原因结论 (Root Cause)**: 精准概括问题根因。
- **软件优化策略 (Key Suggestion)**: 针对性地提出优化建议。

### 2. 结构化数据 (`outputs/metrics.json`)

一个 JSON 文件，包含了从日志中提取的所有原始数据点。每个数据点都是一个包含时间戳、键、值、单位和来源等信息的对象。可用于进一步的数据分析或可视化。

**字段说明**: 请参考 `references/output_spec.md` 获取详细的 JSON 字段定义。

## 注意事项

- **数据驱动**: 所有分析和结论都应基于从日志中提取的**数值证据**，避免主观臆断。
- **时间与时区**: 日志中的时间戳通常是设备本地时间。在分析跨时区问题时需特别注意。
- **多文件聚合**: Skill 会将输入目录下的所有日志文件视为一个整体，按时间戳进行排序和聚合分析，以构建完整的事件序列。
- **单位换算**: Skill 会自动进行单位换算，例如将日志中常见的毫度 (millidegree Celsius) 转换为摄氏度 (°C)。详细规则请参考 `references/patterns.md`。
