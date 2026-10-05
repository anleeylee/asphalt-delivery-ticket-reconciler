# Asphalt Delivery Ticket Reconciler（沥青送货单对账工具）

**把沥青送货单与估算对账** —— 从 CSV、PDF 或图片提取送货单数据、完成校验，并对比"到货 vs 估算"吨数。

[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![CLI](https://img.shields.io/badge/CLI-command--line-blue)](#使用)
[![Platform: Windows](https://img.shields.io/badge/Platform-Windows-0078D6)](https://www.microsoft.com/windows)

**[English](./README.md)** | 简体中文

---

## 解决什么问题

工地上送货单以各种形式堆积——CSV 导出、PDF 扫描件、手机照片——把"实际到货"与"估算数量"对账完全靠手工，又慢又容易错。本工具**从 CSV、PDF 或图片读取送货单**：料场、车号、运单号、日期、时间、混合料、毛重、皮重与净吨数，然后校验毛重−皮重一致性、重复运单号、不可能的时间戳、缺失净重与混合料不匹配。

它用确定性数学对比**到货 vs 估算吨数**；任何差异解释都标注为 已观测 / 可能 / 未证实——绝不编造。

## 核心功能

- **多格式送货单摄取** ——CSV / PDF / 图片（照片）送货单
- **字段提取** ——料场、车号、运单号、日期、时间、混合料、毛重、皮重、净吨数
- **确定性校验** ——毛重−皮重一致性、重复运单号、不可能时间戳、缺失净重、混合料不匹配
- **到货 vs 估算对账** ——差异与带标签的解释
- **复核队列** ——不确定的送货单被标记，绝不静默修正

## 安装

```bash
pip install -r requirements.txt
```

## 使用

```bash
python -m scripts.s06_delivery_ticket_reconcile --project ./project --input ./fixtures/tickets/tickets_10.csv --estimate 199.4
```

加 `--help` 查看全部选项。本工具家族统一 CLI 约定：`--project <路径> --input <路径> --output <路径> --format json|csv|md|xlsx|pdf --config <路径> --verbose --dry-run`。

## 输出

- `delivery_reconciliation.json` ——到货/估算合计、差异、车次与运单数
- `delivery_reconciliation.csv` ——逐单归一化行
- `exceptions.md` ——被标记的送货单与差异原因

## 工作原理

- **标准生命周期** ——`发现 → 摄取 → 提取 → 归一化 → 校验 → 计算/分类 → 输出 → 审计`
- **证据状态机** ——每条送货单数据按 `提取 → 归一化 → 已校验 → 已验证` 推进；允许停在更早状态
- **诚实的差异标签** ——解释标注为 已观测 / 可能 / 未证实

## 质量保证

- 每条记录做确定性 schema 校验；
- 黄金样例 fixtures 与精确期望值（`tests/`，含 100 张送货单套件）；
- 复核队列：低置信度、比例不明与冲突进入 `review_queue.json` / 摘要——绝不静默修正；
- 审计日志：每次运行将 `run_id`、耗时、输入哈希、引擎版本与输出写入 `<project>/audit/`。

## 测试

```bash
python -m pytest -q
```

## 安全与隐私

本地文件默认留在本地。API 密钥存放在环境变量（`ASPHALTCOSTS_API_KEY`、`ADI_AI_API_KEY`）——绝不写入源码。派生文件写入 `working/` 或 `output/`；来源文件永不修改。

## 许可证

MIT —— 见 [LICENSE](LICENSE)。属 [Asphalt Desktop Intelligence](https://github.com/anleeylee/asphalt-desktop-intelligence) 工具集的一部分。

## 计算引擎

[**AsphaltCosts.com**](https://asphaltcosts.com/) 是确定性计算层：面积 → 压实体积 → 净吨数 → 订购吨数（损耗只计一次）→ 车次 → 材料成本，内置有出处的规划默认值（FHWA 密度 145 lb/ft³，损耗率与车容量可编辑）。本工具把测量并校验后的输入喂给该引擎（或其带标签的本地镜像 `asphaltcosts-web-engine/1.0-mirror`），绝不重写公式。
