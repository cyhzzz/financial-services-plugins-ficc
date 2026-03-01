---
name: fixed-income
description: 固收业务插件 - 利率债、信用债、衍生品分析与估值
dependency:
  python:
    - pandas>=2.0.0
    - numpy>=1.24.0
---

# 固收业务插件 (Fixed Income)

## 概述

固收业务插件是FICC业务插件层的重要组成部分，专注于固定收益类产品的分析、估值和风险管理，包括利率债、信用债、利率衍生品等。

## 功能模块

### 1. 债券估值与分析

```python
class BondAnalyzer:
    """债券分析器"""
    
    def calculate_ytm(self, bond, price):
        """计算到期收益率(YTM)"""
        pass
    
    def calculate_duration(self, bond, yield_curve):
        """
        计算久期
        
        - Modified Duration: 修正久期
        - Macaulay Duration: 麦考利久期
        - Effective Duration: 有效久期（含权债）
        """
        pass
    
    def calculate_convexity(self, bond, yield_curve):
        """计算凸性"""
        pass
    
    def calculate_carry(self, bond, funding_rate):
        """
        计算Carry
        
        Carry = Coupon Income - Funding Cost
        """
        pass
    
    def calculate_roll_down(self, bond, yield_curve, horizon=1):
        """
        计算Roll-down收益
        
        假设收益率曲线不变，债券沿曲线向短期移动带来的价格变化
        """
        pass
```

### 2. 债券组合分析

```python
class BondPortfolioAnalyzer:
    """债券组合分析器"""
    
    def calculate_portfolio_yield(self, portfolio):
        """计算组合收益率"""
        pass
    
    def calculate_portfolio_duration(self, portfolio):
        """计算组合久期"""
        pass
    
    def calculate_key_rate_duration(self, portfolio, key_tenors):
        """
        计算关键利率久期(KRD)
        
        分析组合对不同期限利率变动的敏感度
        """
        pass
    
    def calculate_yield_curve_exposure(self, portfolio):
        """
        计算收益率曲线敞口
        
        分析组合对曲线平行移动、陡峭化、平坦化的敏感度
        """
        pass
    
    def attribution_analysis(self, portfolio, start_date, end_date):
        """
        归因分析
        
        分解组合收益来源：
        1. Carry (票息收入)
        2. Roll-down (骑乘效应)
        3. Rate Change (利率变动)
        4. Spread Change (利差变动)
        5. Selection (个券选择)
        """
        pass
```

### 3. 信用分析

```python
class CreditAnalyzer:
    """信用分析器"""
    
    def calculate_spread_metrics(self, bond, benchmark_curve):
        """
        计算利差指标
        
        - G-Spread: 相对于政府债的利差
        - I-Spread: 相对于互换曲线的利差
        - Z-Spread: 零波动率利差
        - ASW: 资产互换利差
        """
        pass
    
    def calculate_credit_metrics(self, bond):
        """
        计算信用指标
        
        - 违约概率(PD)
        - 违约损失率(LGD)
        - 预期损失(EL)
        """
        pass
    
    def analyze_credit_curve(self, issuer_curve):
        """
        分析信用曲线
        
        分析同一发行人在不同期限的信用利差结构
        """
        pass
```

### 4. 利率衍生品分析

```python
class RatesDerivativeAnalyzer:
    """利率衍生品分析器"""
    
    def price_ir_swap(self, swap_params, discount_curve, forward_curve):
        """
        定价利率互换
        
        计算：
        - 固定端现值
        - 浮动端现值
        - 互换价值（对固定端支付方）
        """
        pass
    
    def price_swaption(self, swaption_params, discount_curve, vol_surface):
        """
        定价互换期权
        
        Models: Black, Normal, SABR
        """
        pass
    
    def price_cap_floor(self, cap_floor_params, discount_curve, vol_surface):
        """定价利率上限/下限"""
        pass
    
    def calculate_swap_risk_metrics(self, swap, curves):
        """
        计算互换风险指标
        
        - PV01: 利率变动1bp对价值的影响
        - Bucketed PV01: 各期限的PV01
        - Convexity: 凸性调整
        """
        pass
```

## 与核心插件的集成

```python
# 固收业务插件使用核心服务示例

from core_plugins.ficc_core import CurveBuilder, PricingEngine
from core_plugins.risk_management import MarketRiskManager

class FixedIncomePlugin:
    def __init__(self):
        self.curve_builder = CurveBuilder()
        self.pricing_engine = PricingEngine()
        self.risk_manager = MarketRiskManager()
        self.bond_analyzer = BondAnalyzer()
        self.portfolio_analyzer = BondPortfolioAnalyzer()
    
    def analyze_bond_portfolio(self, portfolio, market_data):
        # 构建收益率曲线
        yield_curve = self.curve_builder.build_yield_curve(
            instruments=market_data.get_instruments(),
            method="cubic_spline"
        )
        
        # 组合分析
        portfolio_metrics = {
            "yield": self.portfolio_analyzer.calculate_portfolio_yield(portfolio),
            "duration": self.portfolio_analyzer.calculate_portfolio_duration(portfolio),
            "krd": self.portfolio_analyzer.calculate_key_rate_duration(
                portfolio, 
                key_tenors=[0.5, 1, 2, 5, 10]
            ),
            "attribution": self.portfolio_analyzer.attribution_analysis(
                portfolio, 
                start_date="2024-01-01", 
                end_date="2024-12-31"
            )
        }
        
        # 风险计算
        var_result = self.risk_manager.calculate_var(
            portfolio=portfolio,
            method="historical",
            confidence=0.99
        )
        
        return {
            "portfolio_metrics": portfolio_metrics,
            "risk_metrics": var_result,
            "yield_curve": yield_curve
        }
```

## 依赖项

- pandas >= 2.0.0
- numpy >= 1.24.0
- scipy >= 1.11.0
