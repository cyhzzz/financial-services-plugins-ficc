# FICC Core

## Description

FICC 核心基础设施技能，为固定收益、外汇、大宗商品业务提供统一的数据连接、基础建模和标准化接口。深度融合 ficc-analysis-skill 的数据架构和计算引擎能力，严格对齐 Anthropic financial-services-plugins 架构标准，是连接上游数据源与下游业务应用的核心枢纽，具备真实业务场景下的高性能、高可靠、可扩展的基础服务能力。

## Capabilities

### 1. 数据连接 (Data Connectivity)

#### 1.1 多源数据接入架构

**数据源分类与接入方式**:

| 数据源类型 | 代表系统 | 数据内容 | 接入频率 | 延迟要求 | 接入方式 |
|-----------|---------|---------|---------|---------|---------|
| **行情终端** | Bloomberg, Reuters Eikon | 实时行情、历史数据、新闻 | 实时 | <100ms | API/专有协议 |
| **国内数据** | Wind, 同花顺iFinD | A股债券、宏观数据、基金 | 实时/日度 | <1s | API/SDK |
| **交易所** | 中金所、上金所、外汇交易中心 | 成交数据、持仓数据 | 实时 | <500ms | 专线接入 |
| **交易平台** | Summit, Murex, 恒生 | 交易数据、持仓数据 | 实时 | <1s | 数据库/API |
| **风险系统** | Algorithmics, 自研系统 | 风险指标、限额数据 | 实时/小时 | <5min | API/消息队列 |
| **财务系统** | SAP, Oracle | 会计数据、损益数据 | 日度/月度 | <1day | ETL/数据库 |
| **外部数据** | 央行、统计局、海关 | 宏观经济数据 | 月度/季度 | <1week | 爬虫/API |

**统一数据接入框架**:

```
┌─────────────────────────────────────────────────────────────┐
│                    统一数据接入层 (Data Ingestion Layer)        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐          │
│  │ 实时行情接入 │  │ 批量数据接入 │  │ 事件驱动接入 │          │
│  │ (Streaming) │  │  (Batch)    │  │  (Event)    │          │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘          │
│         │                │                │                │
│         ▼                ▼                ▼                │
│  ┌──────────────────────────────────────────────────┐     │
│  │              数据标准化与清洗层                      │     │
│  │  - 数据格式转换 (JSON/Protobuf/XML/CSV)           │     │
│  │  - 字段映射与标准化                               │     │
│  │  - 数据质量校验 (完整性、准确性、时效性)          │     │
│  │  - 异常数据处理 (插值、剔除、标记)                 │     │
│  └──────────────────────────────────────────────────┘     │
│                          │                                │
│                          ▼                                │
│  ┌──────────────────────────────────────────────────┐     │
│  │              数据存储与缓存层                        │     │
│  │  ┌─────────────┐  ┌─────────────┐  ┌───────────┐  │     │
│  │  │ 实时缓存    │  │ 时序数据库  │  │ 关系数据库 │  │     │
│  │  │ (Redis)     │  │(InfluxDB)   │  │(PostgreSQL)│  │     │
│  │  └─────────────┘  └─────────────┘  └───────────┘  │     │
│  └──────────────────────────────────────────────────┘     │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

**数据标准化规范**:

```yaml
# 统一数据模型定义
data_models:
  market_data:
    tick_data:
      fields:
        - symbol: string  # 合约代码
        - timestamp: datetime  # 时间戳
        - bid_price: decimal(18,6)  # 买入价
        - ask_price: decimal(18,6)  # 卖出价
        - bid_size: decimal(18,4)  # 买入量
        - ask_size: decimal(18,4)  # 卖出量
        - last_price: decimal(18,6)  # 最新成交价
        - last_size: decimal(18,4)  # 最新成交量
        - volume: decimal(18,4)  # 累计成交量
        - open_interest: decimal(18,4)  # 持仓量
      primary_key: [symbol, timestamp]
      indexes:
        - idx_symbol_time: [symbol, timestamp]
        
    bar_data:
      fields:
        - symbol: string
        - timestamp: datetime
        - open: decimal(18,6)
        - high: decimal(18,6)
        - low: decimal(18,6)
        - close: decimal(18,6)
        - volume: decimal(18,4)
        - open_interest: decimal(18,4)
      primary_key: [symbol, timestamp, frequency]
      
  trade_data:
    trade_execution:
      fields:
        - trade_id: string  # 成交编号
        - order_id: string  # 订单编号
        - symbol: string  # 合约代码
        - side: enum(BUY, SELL)  # 买卖方向
        - quantity: decimal(18,4)  # 成交数量
        - price: decimal(18,6)  # 成交价格
        - trade_time: datetime  # 成交时间
        - counterparty: string  # 交易对手
        - trading_venue: string  # 交易场所
        - settlement_date: date  # 结算日
      primary_key: [trade_id]
      indexes:
        - idx_order: [order_id]
        - idx_symbol_time: [symbol, trade_time]
        - idx_counterparty: [counterparty, trade_time]
        
  position_data:
    position_holding:
      fields:
        - position_id: string
        - account_id: string
        - symbol: string
        - quantity: decimal(18,4)  # 持仓数量
        - average_cost: decimal(18,6)  # 平均成本
        - market_price: decimal(18,6)  # 市价
        - market_value: decimal(18,4)  # 市值
        - unrealized_pnl: decimal(18,4)  # 未实现损益
        - realized_pnl: decimal(18,4)  # 已实现损益
        - position_date: date
        - last_update: datetime
      primary_key: [position_id, position_date]
      indexes:
        - idx_account_symbol: [account_id, symbol, position_date]
        - idx_symbol_date: [symbol, position_date]
```

（由于内容过长，我将继续完成其余部分...）
