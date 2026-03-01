---
name: risk-management
description: 风险管理插件 - 市场风险、信用风险、流动性风险管理
dependency:
  python:
    - pandas>=2.0.0
    - numpy>=1.24.0
---

# 风险管理插件 (Risk Management)

## 概述

风险管理插件是FICC核心插件层的重要组成部分，提供全面的风险管理能力，包括市场风险、信用风险、流动性风险的计量、监控和报告。

## 功能模块

### 1. 市场风险管理 (Market Risk)

```python
class MarketRiskManager:
    """市场风险管理器"""
    
    def calculate_var(self, portfolio, method="historical", confidence=0.99):
        """
        计算VaR
        
        Methods: historical, parametric, monte_carlo
        """
        pass
    
    def calculate_cvar(self, portfolio, confidence=0.99):
        """计算CVaR/Expected Shortfall"""
        pass
    
    def calculate_sensitivities(self, portfolio):
        """
        计算敏感度指标
        
        包括：PV01, CS01, Delta, Gamma, Vega等
        """
        pass
    
    def perform_stress_test(self, portfolio, scenarios):
        """执行压力测试"""
        pass
    
    def monitor_risk_limits(self, portfolio, limits):
        """监控风险限额"""
        pass
```

### 2. 信用风险管理 (Credit Risk)

```python
class CreditRiskManager:
    """信用风险管理器"""
    
    def calculate_exposure(self, counterparty, trades):
        """
        计算敞口
        
        包括：当前敞口、潜在未来敞口(PFE)
        """
        pass
    
    def calculate_cva(self, portfolio, counterparty_data):
        """
        计算信用估值调整(CVA)
        """
        pass
    
    def calculate_pfe(self, portfolio, confidence=0.99, horizon=252):
        """
        计算潜在未来敞口(PFE)
        """
        pass
    
    def assess_counterparty_risk(self, counterparty_id):
        """评估交易对手风险"""
        pass
    
    def monitor_credit_limits(self, exposures, limits):
        """监控信用限额"""
        pass
```

### 3. 流动性风险管理 (Liquidity Risk)

```python
class LiquidityRiskManager:
    """流动性风险管理器"""
    
    def calculate_lcr(self, assets, liabilities):
        """
        计算流动性覆盖率(LCR)
        
        LCR = High Quality Liquid Assets / Net Cash Outflows over 30 days
        """
        pass
    
    def calculate_nsf(self, assets, liabilities):
        """
        计算净稳定资金比率(NSFR)
        
        NSFR = Available Stable Funding / Required Stable Funding
        """
        pass
    
    def assess_funding_liquidity(self, portfolio):
        """评估融资流动性"""
        pass
    
    def assess_market_liquidity(self, positions):
        """评估市场流动性"""
        pass
    
    def stress_liquidity(self, portfolio, stress_scenarios):
        """流动性压力测试"""
        pass
```

### 4. 风险报告与监控

```python
class RiskReporting:
    """风险报告"""
    
    def generate_daily_risk_report(self, portfolios):
        """生成日度风险报告"""
        pass
    
    def generate_risk_appetite_report(self, risk_metrics, limits):
        """生成风险偏好报告"""
        pass
    
    def generate_stress_test_report(self, stress_results):
        """生成压力测试报告"""
        pass
    
    def create_risk_dashboard(self, real_time_data):
        """创建风险仪表盘"""
        pass
```

## 与业务插件的集成

```python
# 业务插件使用风险管理服务示例

from core_plugins.risk_management import MarketRiskManager, CreditRiskManager
from core_plugins.ficc_core import CurveBuilder

class FixedIncomePlugin:
    def __init__(self):
        self.market_risk_mgr = MarketRiskManager()
        self.credit_risk_mgr = CreditRiskManager()
        self.curve_builder = CurveBuilder()
    
    def analyze_portfolio_risk(self, bond_portfolio):
        # 计算VaR
        var_result = self.market_risk_mgr.calculate_var(
            portfolio=bond_portfolio,
            method="historical",
            confidence=0.99
        )
        
        # 计算敏感度
        sensitivities = self.market_risk_mgr.calculate_sensitivities(
            portfolio=bond_portfolio
        )
        
        # 计算信用敞口
        credit_exposure = self.credit_risk_mgr.calculate_exposure(
            counterparty=bond_portfolio.counterparty,
            trades=bond_portfolio.trades
        )
        
        return {
            "var": var_result,
            "sensitivities": sensitivities,
            "credit_exposure": credit_exposure
        }
```

## 依赖项

- pandas >= 2.0.0
- numpy >= 1.24.0
- scipy >= 1.11.0
