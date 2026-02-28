# FICC BI & Decision Platform Integration

## Description
FICC 智能 BI 与决策平台集成技能，将 ficc-analysis-skill 经营分析报告生成能力与 FICC 业务系统深度融合，构建实时数据驱动的智能决策支持平台。

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    FICC 智能 BI 与决策平台                        │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  数据接入层   │  │  分析计算层   │  │    决策支持层       │  │
│  ├──────────────┤  ├──────────────┤  ├──────────────────────┤  │
│  │ • 交易数据    │  │ • 实时损益   │  │ • 预警提醒         │  │
│  │ • 市场数据    │  │ • 风险指标   │  │ • 策略建议         │  │
│  │ • 风险数据    │  │ • RAROC/EVA │  │ • 情景分析         │  │
│  │ • 参考数据    │  │ • 敞口归因   │  │ • 决策模拟         │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐  │
│  │  报告生成层   │  │  可视化层    │  │    平台集成层       │  │
│  ├──────────────┤  ├──────────────┤  ├──────────────────────┤  │
│  │ • 日报/周报  │  │ • 仪表盘    │  │ • 交易系统API      │  │
│  │ • 月报/季报  │  │ • 移动H5   │  │ • 风险系统API      │  │
│  │ • 年度/专项  │  │ • PC大屏   │  │ • 数据仓库API      │  │
│  │ • 临时/定制  │  │ • 报告导出  │  │ • 消息推送API      │  │
│  └──────────────┘  └──────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Integration Points

### 1. 数据源集成 (Data Source Integration)

```yaml
data_sources:
  trading_systems:
    - name:  Summit
      type:  treasury_system
      data:  [trades, positions, pnl]
    - name:  Murex
      type:  derivatives_system
      data:  [swaps, options, structured]
    - name:  恒生
      type:  bond_trading
      data:  [bonds, repos, lending]
      
  market_data:
    - name:  Bloomberg
      type:  market_data_terminal
      data:  [prices, curves, volatilities]
    - name:  Wind
      type:  domestic_market_data
      data:  [china_bonds, rates, fx]
    - name:  Reuters
      type:  global_market_data
      data:  [fx_rates, commodities]
      
  risk_systems:
    - name:  Algorithmics
      type:  risk_platform
      data:  [var, stress_test, exposure]
    - name:  Murex Risk
      type:  derivatives_risk
      data:  [greeks, scenario_analysis]
    - name:  自研系统
      type:  domestic_risk
      data:  [limits, breaches, concentrations]
      
  data_warehouse:
    - name:  Teradata
      type:  enterprise_dw
      data:  [historical_trades, positions, pnl]
    - name:  Hadoop
      type:  big_data_platform
      data:  [logs, events, analytics]
    - name:  数据湖
      type:  data_lake
      data:  [raw_data, unstructured, documents]
```

### 2. 实时计算引擎 (Real-time Calculation Engine)

```python
class RealTimeCalculationEngine:
    """FICC 实时计算引擎"""
    
    def __init__(self):
        self.cache = RedisCache()
        self.stream = KafkaStream()
        self.db = TimeSeriesDB()
        
    async def calculate_realtime_pnl(self, portfolio_id):
        """实时损益计算"""
        # 获取实时市场价格
        market_data = await self.stream.get_latest(
            topics=['bond_prices', 'fx_rates', 'swap_rates']
        )
        
        # 获取组合持仓
        positions = await self.db.query(
            f"SELECT * FROM positions WHERE portfolio_id = {portfolio_id}"
        )
        
        # 计算盯市损益
        mtm_pnl = 0
        for pos in positions:
            current_price = market_data.get(pos.instrument_id)
            if current_price:
                unrealized = (current_price - pos.avg_cost) * pos.quantity
                mtm_pnl += unrealized
        
        # 计算已实现损益
        realized_pnl = await self.calculate_realized_pnl(portfolio_id)
        
        # 计算资金成本
        funding_cost = await self.calculate_funding_cost(portfolio_id)
        
        total_pnl = mtm_pnl + realized_pnl - funding_cost
        
        return {
            'portfolio_id': portfolio_id,
            'timestamp': datetime.now(),
            'mtm_pnl': mtm_pnl,
            'realized_pnl': realized_pnl,
            'funding_cost': funding_cost,
            'total_pnl': total_pnl,
            'ytd_pnl': await self.get_ytd_pnl(portfolio_id),
            'mtd_pnl': await self.get_mtd_pnl(portfolio_id)
        }
```

## Capabilities

### BI 分析能力 (BI Analytics)
- **多维度损益归因**：按产品、币种、期限、交易员等多维度分析
- **实时风险监控**：VaR、敞口、敏感度实时计算与预警
- **RAROC/EVA 绩效评估**：风险调整后的绩效评估体系
- **情景分析与压力测试**：多情景下的损益预测
- **同业比较分析**：与同业机构的业绩比较

### 决策支持能力 (Decision Support)
- **交易策略建议**：基于市场数据和历史回测的策略推荐
- **风险限额管理**：实时监控与限额预警
- **资本配置优化**：基于RAROC的资本优化配置建议
- **对冲策略设计**：自动化的对冲方案生成
- **审批流程支持**：交易审批的风险提示与决策支持

### 报告与可视化 (Reporting & Visualization)
- **实时仪表盘**：PC大屏、移动端H5、交易员桌面
- **自动化报告**：日报、周报、月报、季报自动生成
- **临时/定制报告**：灵活的自定义报告生成
- **交互式分析**：下钻、联动、过滤的交互式分析
- **多格式导出**：PDF、Excel、PPT、HTML导出

## Commands

```bash
# 实时损益查询
/ficc-bi realtime-pnl --portfolio 自营组合A --breakdown 品种+期限

# 风险监控仪表盘
/ficc-bi risk-dashboard --view 交易员 --limits true --alerts true

# RAROC绩效评估
/ficc-bi raroc-analysis --team 贵金属 --period 2024Q1 --benchmark 同业

# 情景分析
/ficc-bi scenario-analysis --shock 利率上行100bp --portfolio 全行FICC

# 自动化报告生成
/ficc-bi generate-report --type 日报 --date 2024-06-28 --distribution 邮件+钉钉

# 交易决策支持
/ficc-bi trade-decision --trade 买入10Y国债5亿 --show-risk-impact true
```

## Implementation

### 数据流架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        数据源层                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ 交易系统  │ │ 行情数据  │ │ 风险系统  │ │ 数据仓库  │           │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     实时计算引擎 (Flink/Spark)                    │
│  ┌────────────────┐ ┌────────────────┐ ┌────────────────┐        │
│  │   实时损益计算  │ │   风险指标计算  │ │   归因分析计算  │        │
│  └────────────────┘ └────────────────┘ └────────────────┘        │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│    实时缓存层    │ │    时序数据库    │ │    关系数据库    │
│    (Redis)      │ │  (InfluxDB)     │ │  (PostgreSQL)   │
└─────────────────┘ └─────────────────┘ └─────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    应用服务层 (API Gateway)                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │ BI分析服务 │ │ 报告服务  │ │ 决策服务  │ │ 推送服务  │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│    PC Web端      │ │     移动端H5     │ │     大屏展示     │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

## Dependencies
- apache-flink / apache-spark (stream processing)
- redis (real-time caching)
- influxdb (time-series database)
- postgresql (relational database)
- kafka (message streaming)
- grafana (visualization)
- python-fastapi (api gateway)

## License
Apache-2.0 (aligns with Anthropic financial-services-plugins)
