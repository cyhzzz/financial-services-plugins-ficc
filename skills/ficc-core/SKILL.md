# FICC Core

## Description
FICC 核心插件，为固定收益、外汇、大宗商品业务提供统一的数据连接、基础建模和标准化接口。这是所有 FICC 业务插件的基础设施。

## Capabilities

### 数据连接 (Data Connectivity)
- 统一接入多个金融市场数据终端（Bloomberg、Reuters、Wind、同花顺等）
- 标准化行情数据接口（实时行情、历史数据）
- 参考数据管理（日历、 holiday、到期日等）
- 数据质量监控和异常处理

### 基础建模 (Base Modeling)
- 日历和日期运算（工作日计算、到期日推算）
- 利率曲线构建（ bootstrapping、插值方法）
- 贴现因子计算和现值计算
- 日期计数约定（ACT/360、30/360、ACT/ACT 等）

### 标准化接口 (Standardized API)
- 统一的数据请求格式
- 标准化的返回数据结构
- 错误处理和异常机制
- 缓存策略和性能优化

### 市场监控 (Market Monitoring)
- 实时行情监控面板
- 关键指标预警（价格波动、利差变化等）
- 市场异常检测
- 数据断线提醒

## Commands

```bash
# 测试数据连接
/ficc-core test-connection --source Bloomberg

# 获取行情数据
/ficc-core fetch-market-data --type bond --ticker CNY10Y Govt

# 构建收益率曲线
/ficc-core build-curve --type swap --tenor 1M-30Y --currency USD

# 计算现值
/ficc-core calculate-pv --cashflows [100,100,1100] --dates [2024-06-15,2024-12-15,2025-06-15] --discount-curve USD-SOFR

# 查看市场监控面板
/ficc-core market-monitor --view dashboard

# 设置预警
/ficc-core set-alert --condition "US10Y > 5.0%" --action notify
```

## Implementation

### Data Source Adapters

| 数据源 | 适配器 | 支持功能 |
|--------|--------|----------|
| Bloomberg | BBAdapter | 实时行情、历史数据、参考数据 |
| Refinitiv (Reuters) | RefinitivAdapter | 实时行情、历史数据、新闻 |
| Wind | WindAdapter | A股债券、宏观数据 |
| 同花顺 iFinD | iFinDAdapter | A股债券、基金数据 |
| 内部系统 | InternalAdapter | 交易数据、持仓数据 |

### Core Models

```python
# 日历和日期
class Calendar:
    - holidays: List[Date]
    - weekend_days: List[int]  # 0=Monday, 6=Sunday
    - is_business_day(date): bool
    - add_business_days(date, n): Date
    - days_between(start, end, convention): int

# 利率曲线
class YieldCurve:
    - pillars: List[Tuple[Date, float]]  # (maturity, rate)
    - interpolation_method: str  # linear, cubic, etc.
    - discount_factor(date): float
    - zero_rate(date): float
    - forward_rate(start, end): float

# 现金流
class CashFlow:
    - amount: float
    - currency: str
    - date: Date
    - cash_flow_type: str  # principal, interest, fee, etc.

# 金融工具基类
class FinancialInstrument:
    - instrument_id: str
    - instrument_type: str
    - currency: str
    - valuation_date: Date
    - pv(discount_curve): float
    - cash_flows(): List[CashFlow]
    - risk_metrics(): RiskMetrics
```

### API Design

```python
# 统一数据接口
class MarketDataService:
    def get_realtime_price(self, instrument_id: str, source: str = None) -> Price:
        pass
    
    def get_historical_prices(self, instrument_id: str, start: Date, end: Date, 
                              freq: str = "daily") -> List[Price]:
        pass
    
    def get_reference_data(self, instrument_id: str, field: str) -> Any:
        pass

# 曲线构建服务
class CurveBuildingService:
    def build_swap_curve(self, market_data: SwapMarketData, 
                         curve_definition: CurveDefinition) -> YieldCurve:
        pass
    
    def build_bond_curve(self, bond_prices: List[BondPrice], 
                         calendar: Calendar) -> YieldCurve:
        pass

# 估值服务
class ValuationService:
    def calculate_pv(self, instrument: FinancialInstrument, 
                     market_data: MarketDataEnvironment) -> ValuationResult:
        pass
    
    def calculate_risk_metrics(self, instrument: FinancialInstrument,
                              market_data: MarketDataEnvironment,
                              shock_scenarios: List[Scenario]) -> RiskReport:
        pass
```

## Dependencies

```json
{
  "python": "3.9+",
  "dependencies": [
    "numpy>=1.20.0",
    "pandas>=1.3.0",
    "scipy>=1.7.0",
    "quantlib-python>=1.25",
    "blpapi>=3.19.0",
    "requests>=2.25.0",
    "pydantic>=1.8.0",
    "fastapi>=0.70.0",
    "redis>=4.0.0"
  ]
}
```

## Usage Example

```python
from ficc_core import MarketDataService, CurveBuildingService, ValuationService
from ficc_core.models import Calendar, YieldCurve, SwapMarketData

# 初始化服务
market_data_service = MarketDataService(sources=["Bloomberg", "Internal"])
curve_service = CurveBuildingService()
valuation_service = ValuationService()

# 获取市场数据
swap_rates = market_data_service.get_swap_rates(currency="USD", tenor="3M")

# 构建曲线
market_data = SwapMarketData(
    currency="USD",
    spot_date=date(2024, 6, 14),
    swap_rates=swap_rates
)
curve = curve_service.build_swap_curve(market_data, curve_definition)

# 估值
irs = InterestRateSwap(
    notional=100000000,
    fixed_rate=0.05,
    tenor="5Y",
    currency="USD"
)
result = valuation_service.calculate_pv(irs, market_data_environment)
print(f"PV: {result.pv:,.2f}")
```

## License

Apache License 2.0（与 Anthropic financial-services-plugins 保持一致）
