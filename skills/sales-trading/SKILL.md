---
name: sales-trading
description: 销售交易插件 - 做市分析、执行分析、报价引擎
dependency:
  python:
    - pandas>=2.0.0
    - numpy>=1.24.0
---

# 销售交易插件 (Sales & Trading)

## 概述

销售交易插件是FICC对客插件层的重要组成部分，专注于面向机构客户的销售交易服务，包括做市报价、交易执行、客户订单管理等功能。

## 功能模块

### 1. 做市分析

```python
class MarketMakerAnalytics:
    """做市分析器"""
    
    def calculate_spread_metrics(self, quotes, trades):
        """
        计算价差指标
        
        - 报价价差 (Quoted Spread)
        - 有效价差 (Effective Spread)
        - 实现价差 (Realized Spread)
        - 买卖反弹 (Bid-Ask Bounce)
        """
        pass
    
    def analyze_inventory_risk(self, positions, risk_limits):
        """
        分析库存风险
        
        监控：
        - 库存偏离目标水平
        - 单向持仓集中度
        - 资金占用情况
        """
        pass
    
    def calculate_adverse_selection(self, trades, mid_prices):
        """
        计算逆向选择成本
        
        衡量信息不对称导致的做市损失
        """
        pass
    
    def optimize_quoting_strategy(self, market_conditions, constraints):
        """
        优化报价策略
        
        动态调整：
        - 报价价差
        - 报价深度
        - 更新频率
        """
        pass
    
    def generate_market_making_report(self, analytics_results):
        """
        生成做市分析报告
        
        包括：
        - 收入分解（价差收入、库存损益）
        - 成本分析（逆向选择、操作成本）
        - 风险指标
        """
        pass
```

### 2. 执行分析

```python
class ExecutionAnalyzer:
    """执行分析器"""
    
    def calculate_implementation_shortfall(self, order, execution_data, market_data):
        """
        计算实施缺口 (Implementation Shortfall)
        
        衡量实际执行与决策价格之间的差异
        """
        pass
    
    def calculate_vwap_slippage(self, execution_data, vwap):
        """
        计算VWAP滑点
        
        衡量执行价格相对于市场VWAP的偏离
        """
        pass
    
    def analyze_market_impact(self, execution_data, market_data):
        """
        分析市场冲击
        
        量化大额订单对市场价格的影响
        """
        pass
    
    def calculate_timing_cost(self, order, execution_data):
        """
        计算时机成本
        
        衡量执行时机选择的影响
        """
        pass
    
    def generate_execution_report(self, order, analytics_results):
        """
        生成执行分析报告
        
        包括：
        - 执行摘要
        - 成本归因
        - 执行质量评估
        - 改进建议
        """
        pass
```

### 3. 报价引擎

```python
class QuotingEngine:
    """报价引擎"""
    
    def generate_two_way_price(self, instrument, market_data, risk_params):
        """
        生成双边报价
        
        考虑因素：
        - 中价计算
        - 价差设置
        - 库存调整
        - 风险调整
        """
        pass
    
    def calculate_spread(self, instrument, market_conditions, risk_metrics):
        """
        计算报价价差
        
        考虑：
        - 基础价差（市场流动性）
        - 调整项（波动率、库存、信用）
        """
        pass
    
    def apply_skew(self, base_price, inventory_position, risk_limits):
        """
        应用库存偏置
        
        根据库存方向调整买卖报价倾向
        """
        pass
    
    def validate_quote(self, quote, market_data, risk_limits):
        """
        验证报价合理性
        
        检查：
        - 报价是否在市场合理范围
        - 是否超出风险限额
        - 是否符合合规要求
        """
        pass
    
    def update_quotes(self, market_data_update, active_quotes):
        """
        更新活跃报价
        
        响应市场变化实时更新报价
        """
        pass
```

## 与业务插件的集成

```python
# 销售交易插件使用业务插件示例

from business_plugins.fx_desk import FXDeskPlugin
from business_plugins.fixed_income import FixedIncomePlugin
from core_plugins.risk_management import MarketRiskManager

class SalesTradingPlugin:
    def __init__(self):
        self.fx_plugin = FXDeskPlugin()
        self.fixed_income_plugin = FixedIncomePlugin()
        self.risk_manager = MarketRiskManager()
        self.quoting_engine = QuotingEngine()
        self.market_maker = MarketMakerAnalytics()
        self.execution_analyzer = ExecutionAnalyzer()
    
    def provide_two_way_price(self, instrument, client_profile, market_data):
        """向客户提供双边报价"""
        
        # 1. 计算中价
        if instrument.instrument_type == "FX_SPOT":
            mid_price = self.fx_plugin.get_spot_rate(
                currency_pair=instrument.currency_pair,
                market_data=market_data
            )
        elif instrument.instrument_type == "FX_FORWARD":
            mid_price = self.fx_plugin.price_forward(
                spot_rate=market_data.spot,
                forward_points=market_data.forward_points,
                tenor=instrument.tenor
            )
        elif instrument.instrument_type == "BOND":
            mid_price = self.fixed_income_plugin.price_bond(
                bond=instrument,
                yield_curve=market_data.yield_curve
            )
        
        # 2. 评估客户风险
        client_risk = self.risk_manager.assess_counterparty_risk(
            counterparty=client_profile
        )
        
        # 3. 计算价差
        spread = self.quoting_engine.calculate_spread(
            instrument=instrument,
            market_conditions=market_data,
            client_risk=client_risk,
            trading_volume=client_profile.avg_monthly_volume
        )
        
        # 4. 生成报价
        two_way_price = self.quoting_engine.generate_two_way_price(
            mid_price=mid_price,
            spread=spread,
            direction="TWO_WAY"
        )
        
        # 5. 风险限额检查
        is_within_limits = self.risk_manager.validate_quote_risk(
            quote=two_way_price,
            instrument=instrument,
            client=client_profile
        )
        
        if not is_within_limits:
            two_way_price = self._adjust_quote_for_risk_limits(two_way_price)
        
        return {
            "instrument": instrument.instrument_id,
            "bid": two_way_price.bid,
            "ask": two_way_price.ask,
            "mid": mid_price,
            "spread": spread,
            "valid_until": two_way_price.valid_until,
            "risk_approved": is_within_limits
        }
```

## 依赖项

- pandas >= 2.0.0
- numpy >= 1.24.0
