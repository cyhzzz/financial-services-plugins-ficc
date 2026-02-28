# Fixed Income

## Description
商业银行固定收益业务完整技能体系，覆盖利率债、信用债、利率衍生品等全品类分析与交易。

## Capabilities

### 利率债分析 (Government Bonds)
- 国债、政金债、地方债收益率分析
- 收益率曲线构建与解读（bootstrapping、Nelson-Siegel、Svensson）
- 骑乘策略与久期管理
- 利率敏感度分析（DV01、久期、凸性）
- 国债期货策略（基差交易、跨期套利）

### 信用债分析 (Credit Bonds)
- 金融债、企业债、中票短融分析
- 信用评级与利差分析（G-spread、I-spread、Z-spread、ASW）
- 信用风险预警模型（PD、LGD、EAD）
- 城投债专项分析（区域财政、债务率、隐性债务）
- 地产债专项分析（销售回款、土储质量、三道红线）
- 可转债、可交债分析

### 利率衍生品 (Rates Derivatives)
- IRS（利率互换）定价与估值（OIS discounting、多曲线框架）
- IRS 策略（曲线利差、基差交易、Carry & Roll）
- 国债期货定价与套期保值（CTD、基差、期权性）
- 利率期权（Cap/Floor、Swaption、结构化产品）
-  cross-currency basis 分析
- 套期保值方案设计与会计处理

## Commands

```bash
# 收益率曲线构建
/fixed-income yield-curve --type swap --currency USD --tenors 3M-30Y --method bootstrapping

# 国债分析
/fixed-income gov-bond --country CN --maturity 10Y --yield 2.65 --analysis full

# 信用债分析
/fixed-income credit-bond --issuer 城投A --rating AA+ --province 江苏 --debt-ratio 180% --spread 85bp

# IRS 定价
/fixed-income irs-pricing --notional 1亿 --maturity 5Y --fixed-rate 2.85% --float-leg 3M-Shibor --valuation OIS-discounting

# 国债期货分析
/fixed-income bond-futures --contract T2406 --ctd CTD-bond --basis 0.45 --carry-positive true

# 套期保值方案
/fixed-income hedge-design --exposure 5亿浮息贷款 --risk利率上行 --tool IRS-pay-fixed --target降低融资成本波动
```

## Implementation

### 收益率曲线模型

```python
class YieldCurve:
    def __init__(self, currency, curve_date, instruments):
        self.currency = currency
        self.curve_date = curve_date
        self.instruments = instruments  # Deposit, Futures, Swap, etc.
        self.pillars = []
        self.discount_factors = []
        
    def build(self, method="bootstrapping"):
        """构建曲线方法: bootstrapping, nelson_siegel, svensson"""
        if method == "bootstrapping":
            self._bootstrapping()
        elif method == "nelson_siegel":
            self._nelson_siegel()
        elif method == "svensson":
            self._svensson()
            
    def discount_factor(self, date):
        """获取某日期的贴现因子"""
        return np.interp(date, self.pillars, self.discount_factors)
    
    def zero_rate(self, date, compounding="continuous"):
        """获取某期限的零息利率"""
        df = self.discount_factor(date)
        t = (date - self.curve_date).days / 365.0
        if compounding == "continuous":
            return -np.log(df) / t
        elif compounding == "annual":
            return (1/df)**(1/t) - 1
    
    def forward_rate(self, start_date, end_date):
        """获取远期利率"""
        df1 = self.discount_factor(start_date)
        df2 = self.discount_factor(end_date)
        t = (end_date - start_date).days / 365.0
        return (df1/df2 - 1) / t
```

### 债券定价模型

```python
class Bond:
    def __init__(self, face_value, coupon_rate, maturity_date, 
                 frequency=2, day_count="ACT/ACT", currency="CNY"):
        self.face_value = face_value
        self.coupon_rate = coupon_rate
        self.maturity_date = maturity_date
        self.frequency = frequency  # 每年付息次数
        self.day_count = day_count
        self.currency = currency
        self.cash_flows = self._generate_cash_flows()
        
    def _generate_cash_flows(self):
        """生成债券现金流"""
        cash_flows = []
        # 逻辑：从发行日/估值日到到期日生成每期现金流
        return cash_flows
    
    def price(self, yield_to_maturity, settlement_date=None):
        """用到期收益率计算债券全价"""
        if settlement_date is None:
            settlement_date = date.today()
            
        pv = 0
        for cf in self.cash_flows:
            if cf.date > settlement_date:
                t = (cf.date - settlement_date).days / 365.0
                pv += cf.amount / ((1 + yield_to_maturity/self.frequency)**(t*self.frequency))
        
        # 加上应计利息得到全价
        accrued_interest = self._calculate_accrued_interest(settlement_date)
        return pv + accrued_interest
    
    def yield_to_maturity(self, market_price, settlement_date=None):
        """从市场价格反算到期收益率（用数值方法迭代）"""
        # 使用牛顿法或二分法求解
        pass
    
    def dv01(self, yield_level, bump=0.0001):
        """计算 DV01（收益率变动1bp时的价格变化）"""
        price_up = self.price(yield_level + bump)
        price_down = self.price(yield_level - bump)
        return (price_down - price_up) / 2
    
    def duration(self, yield_level):
        """计算修正久期"""
        return self.dv01(yield_level) / self.price(yield_level) * 10000
    
    def convexity(self, yield_level, bump=0.0001):
        """计算凸性"""
        price_base = self.price(yield_level)
        price_up = self.price(yield_level + bump)
        price_down = self.price(yield_level - bump)
        return (price_up + price_down - 2*price_base) / (bump**2 * price_base)
```

### IRS 定价模型

```python
class InterestRateSwap:
    def __init__(self, notional, fixed_rate, maturity, 
                 fixed_leg_freq=2, float_leg_freq=4,
                 float_index="LIBOR", discount_curve=None):
        self.notional = notional
        self.fixed_rate = fixed_rate
        self.maturity = maturity
        self.fixed_leg_freq = fixed_leg_freq
        self.float_leg_freq = float_leg_freq
        self.float_index = float_index
        self.discount_curve = discount_curve
        
    def present_value(self, valuation_date=None):
        """计算 IRS 现值（固定端 - 浮动端）"""
        if valuation_date is None:
            valuation_date = date.today()
            
        pv_fixed = self._pv_fixed_leg(valuation_date)
        pv_float = self._pv_float_leg(valuation_date)
        
        return pv_fixed - pv_float
    
    def _pv_fixed_leg(self, valuation_date):
        """固定端现值"""
        pv = 0
        for coupon_date in self._generate_fixed_coupon_dates():
            if coupon_date > valuation_date:
                t = (coupon_date - valuation_date).days / 365.0
                df = self.discount_curve.discount_factor(coupon_date)
                coupon = self.notional * self.fixed_rate / self.fixed_leg_freq
                pv += coupon * df
        return pv
    
    def _pv_float_leg(self, valuation_date):
        """浮动端现值（使用远期利率估计）"""
        pv = 0
        for period_start, period_end in self._generate_float_periods():
            if period_start >= valuation_date:
                # 使用远期利率估计浮动利率
                forward_rate = self._estimate_forward_rate(period_start, period_end)
                t = (period_end - period_start).days / 365.0
                df = self.discount_curve.discount_factor(period_end)
                coupon = self.notional * forward_rate * t
                pv += coupon * df
        # 加上本金贴现（如果是到期结算）
        # pv += self.notional * self.discount_curve.discount_factor(self.maturity)
        return pv
    
    def par_rate(self, valuation_date=None):
        """计算使 PV=0 的平价利率"""
        # 使用数值方法求解
        from scipy.optimize import brentq
        
        def pv_as_function_of_rate(rate):
            swap = InterestRateSwap(
                self.notional, rate, self.maturity,
                self.fixed_leg_freq, self.float_leg_freq,
                self.float_index, self.discount_curve
            )
            return swap.present_value(valuation_date)
        
        # 在合理范围内寻找根
        return brentq(pv_as_function_of_rate, 0.0, 0.5)
```

## Workflows

### 典型工作流程

```
1. 数据获取与验证
   - 从多个数据源获取市场数据
   - 验证数据质量和完整性
   - 处理异常值和缺失值

2. 曲线构建
   - 选择适当的构建方法
   - 输入市场数据
   - 验证曲线质量（无套利、平滑性）

3. 产品定价
   - 确定适当的定价模型
   - 输入市场数据和曲线
   - 计算价格和风险指标

4. 风险管理
   - 计算 VaR、CVaR 等风险指标
   - 进行压力测试和情景分析
   - 生成风险报告

5. 对客服务
   - 理解客户需求
   - 设计合适的解决方案
   - 执行交易和后续管理
```

## Examples

### 示例 1：收益率曲线构建与债券定价

```python
from ficc_core import CurveBuildingService, MarketDataService
from ficc_core.models import Calendar, YieldCurve

# 1. 获取市场数据
market_data_service = MarketDataService()
depo_rates = market_data_service.get_deposit_rates(currency="USD", tenors=["1M", "3M", "6M"])
futures_prices = market_data_service.get_futures_prices(contract="Eurodollar", expiries=["H24", "M24", "U24"])
swap_rates = market_data_service.get_swap_rates(currency="USD", tenors=["2Y", "5Y", "10Y", "30Y"])

# 2. 构建曲线
curve_builder = CurveBuildingService()
curve = curve_builder.build_swap_curve(
    currency="USD",
    deposit_rates=deposit_rates,
    futures_prices=futures_prices,
    swap_rates=swap_rates,
    method="bootstrapping",
    interpolation="cubic"
)

# 3. 使用曲线定价债券
from fixed_income import Bond

bond = Bond(
    face_value=1000000,
    coupon_rate=0.045,
    maturity=date(2034, 6, 15),
    frequency=2,
    currency="USD"
)

# 计算理论价格
price = bond.price(yield_to_maturity=0.0485, settlement_date=date(2024, 6, 14))
print(f"Bond theoretical price: {price:,.2f}")

# 计算 DV01 和久期
dv01 = bond.dv01(yield_level=0.0485)
duration = bond.duration(yield_level=0.0485)
print(f"DV01: {dv01:,.2f}, Duration: {duration:.2f}")
```

### 示例 2：IRS 定价与风险分析

```python
from ficc_core import CurveBuildingService
from fixed_income import InterestRateSwap


# 1. 构建贴现曲线和远期曲线
curve_builder = CurveBuildingService()

# OIS 贴现曲线（用于贴现）
ois_curve = curve_builder.build_ois_curve(
    currency="USD",
    ois_rates=ois_market_data,
    collateral_currency="USD"
)

# 远期曲线（用于估计浮动利率）
forward_curve = curve_builder.build_forward_curve(
    currency="USD",
    tenor="3M",
    reference_index="LIBOR",
    futures_data=eurodollar_futures,
    swap_rates=swap_rates
)

# 2. 创建 IRS
irs = InterestRateSwap(
    notional=100000000,  # 1亿
    fixed_rate=0.0285,   # 2.85%
    maturity=date(2029, 6, 14),
    fixed_leg_freq=2,    # 半年付息
    float_leg_freq=4,    # 季度付息
    float_index="USD-3M-LIBOR",
    discount_curve=ois_curve,
    forward_curve=forward_curve
)

# 3. 估值
valuation_result = irs.present_value(valuation_date=date(2024, 6, 14))
print(f"IRS PV: {valuation_result.pv:,.2f}")
print(f"Fixed Leg PV: {valuation_result.fixed_leg_pv:,.2f}")
print(f"Float Leg PV: {valuation_result.float_leg_pv:,.2f}")

# 4. 风险分析
# DV01 分析
dv01_result = irs.dv01_analysis(shock_bp=1)
print(f"\nDV01 Analysis:")
print(f"Parallel Shift DV01: {dv01_result.parallel:,.2f}")
print(f"2Y Key Rate DV01: {dv01_result.key_rates['2Y']:,.2f}")
print(f"5Y Key Rate DV01: {dv01_result.key_rates['5Y']:,.2f}")
print(f"10Y Key Rate DV01: {dv01_result.key_rates['10Y']:,.2f}")

# 曲线非平行移动风险
steepener_risk = irs.curve_risk_scenario(steepener=True, flattening=False)
print(f"\nCurve Risk (Steepener): {steepener_risk:,.2f}")

# 历史情景分析
historical_scenarios = irs.historical_scenario_analysis(
    scenarios=["2008_crisis", "2013_taper_tantrum", "2020_covid"]
)
for scenario, pnl in historical_scenarios.items():
    print(f"{scenario}: {pnl:,.2f}")
```

## Development

### 扩展新技能

```
skills/
└── your-skill/
    ├── .claude-plugin/
    │   └── plugin.json
    ├── commands/
    │   └── your-command.md
    └── skills/
        └── your-skill.md
```

### 插件清单示例

```json
{
  "id": "fixed-income",
  "name": "Fixed Income",
  "version": "1.0.0",
  "description": "固收业务完整技能，包括利率债、信用债、利率衍生品分析与交易",
  "skills": [
    "yield-curve-analysis",
    "bond-pricing",
    "credit-analysis",
    "irs-pricing"
  ],
  "commands": [
    "yield-curve",
    "gov-bond",
    "credit-bond",
    "irs-pricing"
  ],
  "dependencies": [
    "ficc-core"
  ]
}
```

## License

Apache License 2.0（与 Anthropic financial-services-plugins 保持一致）
