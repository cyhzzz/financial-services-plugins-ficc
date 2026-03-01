---
name: commodities
description: 大宗商品业务插件 - 贵金属、能源、基本金属分析与估值
dependency:
  python:
    - pandas>=2.0.0
    - numpy>=1.24.0
---

# 大宗商品业务插件 (Commodities)

## 概述

大宗商品业务插件是FICC业务插件层的重要组成部分，专注于大宗商品市场的分析、交易和风险管理，包括贵金属（黄金、白银、铂金）、能源（原油、天然气）、基本金属（铜、铝、锌）等。

## 功能模块

### 1. 贵金属分析

```python
class PreciousMetalAnalyzer:
    """贵金属分析器"""
    
    def analyze_gold_market(self, market_data):
        """
        分析黄金市场
        
        包括：价格走势、波动率分析、与美元指数相关性、实际利率影响
        """
        pass
    
    def calculate_lease_rate(self, spot_price, forward_price, tenor):
        """
        计算黄金租赁利率
        
        Lease Rate = (Forward / Spot - 1) / Tenor
        """
        pass
    
    def analyze_gold_silver_ratio(self, gold_price, silver_price):
        """
        分析金银比
        
        识别金银比交易机会
        """
        pass
    
    def calculate_storage_cost(self, metal_type, quantity, duration):
        """
        计算仓储成本
        
        包括：保险、安保、运输、融资成本
        """
        pass
```

### 2. 能源分析

```python
class EnergyAnalyzer:
    """能源分析器"""
    
    def analyze_crude_oil_curve(self, futures_data):
        """
        分析原油期货曲线
        
        包括：期限结构（Contango/Backwardation）、裂解价差
        """
        pass
    
    def calculate_crack_spread(self, crude_price, product_prices):
        """
        计算裂解价差
        
        3:2:1 Crack Spread = (2 * Gasoline + 1 * Diesel - 3 * Crude) / 3
        """
        pass
    
    def analyze_natural_gas_storage(self, storage_data, injection_withdrawal):
        """
        分析天然气库存
        
        季节性分析、供需平衡
        """
        pass
    
    def calculate_convenience_yield(self, spot_price, futures_price, rate, storage_cost, tenor):
        """
        计算便利收益
        
        反映持有实物商品的非货币收益
        """
        pass
```

### 3. 基本金属分析

```python
class BaseMetalAnalyzer:
    """基本金属分析器"""
    
    def analyze_copper_market(self, market_data, inventory_data, china_demand):
        """
        分析铜市场
        
        关注：全球库存、中国需求、矿山供应、废铜供应
        """
        pass
    
    def calculate_lme_basis(self, lme_price, domestic_price, fx_rate, duties, freight):
        """
        计算LME基差
        
        识别内外盘套利机会
        """
        pass
    
    def analyze_inventory_cycle(self, inventory_data, consumption_data, production_data):
        """
        分析库存周期
        
        识别主动补库、被动补库、主动去库、被动去库阶段
        """
        pass
    
    def calculate_mining_cost_curve(self, mine_production_data):
        """
        计算矿山成本曲线
        
        分析价格支撑位、90分位成本线
        """
        pass
```

### 4. 商品衍生品定价

```python
class CommodityDerivativePricer:
    """商品衍生品定价器"""
    
    def price_commodity_future(self, spot_price, cost_of_carry, convenience_yield, rate, tenor):
        """
        定价商品期货
        
        F = S × exp((r + u - y) × T)
        
        u: storage cost rate
        y: convenience yield
        """
        pass
    
    def price_commodity_option(self, option_params, market_data, model="black_76"):
        """
        定价商品期权
        
        Models: Black-76, Black-Scholes, American Binomial
        """
        pass
    
    def price_commodity_swap(self, swap_params, forward_curve, discount_curve):
        """
        定价商品互换
        
        包括：浮动对固定、浮动对浮动、基差互换
        """
        pass
    
    def price_gold_lease(self, lease_params, gold_rate_curve):
        """
        定价黄金租赁
        """
        pass
```

## 与核心插件的集成

```python
# 商品业务插件使用核心服务示例

from core_plugins.ficc_core import CurveBuilder, PricingEngine
from core_plugins.risk_management import MarketRiskManager

class CommoditiesPlugin:
    def __init__(self):
        self.curve_builder = CurveBuilder()
        self.risk_manager = MarketRiskManager()
        self.gold_analyzer = PreciousMetalAnalyzer()
        self.energy_analyzer = EnergyAnalyzer()
        self.pricer = CommodityDerivativePricer()
    
    def analyze_gold_portfolio(self, gold_positions, market_data):
        # 分析黄金市场
        market_analysis = self.gold_analyzer.analyze_gold_market(
            market_data=market_data
        )
        
        # 计算租赁利率
        for position in gold_positions:
            if position.is_lease:
                lease_rate = self.gold_analyzer.calculate_lease_rate(
                    spot_price=market_data.spot_price,
                    forward_price=market_data.forward_price,
                    tenor=position.tenor
                )
        
        # 衍生品定价
        for option in gold_positions.options:
            option_price = self.pricer.price_commodity_option(
                option_params=option.params,
                market_data=market_data,
                model="black_76"
            )
        
        # 风险计算
        var_result = self.risk_manager.calculate_var(
            portfolio=gold_positions,
            method="historical",
            confidence=0.99
        )
        
        return {
            "market_analysis": market_analysis,
            "var_result": var_result
        }
```

## 依赖项

- pandas >= 2.0.0
- numpy >= 1.24.0
