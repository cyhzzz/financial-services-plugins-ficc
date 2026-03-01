---
name: ficc-core
description: FICC 核心插件 - 数据连接、基础建模、统一接口
dependency:
  python:
    - pandas>=2.0.0
    - numpy>=1.24.0
---

# FICC 核心插件 (FICC Core)

## 概述

FICC核心插件是整个FICC技能集合的基础层，提供统一的数据连接、基础建模和标准化接口，为上层的业务插件和对客插件提供服务。

## 功能模块

### 1. 数据连接层 (Data Connection)

```python
class DataConnectionManager:
    """数据连接管理器"""
    
    def connect_database(self, db_type, connection_params):
        """
        连接数据库
        
        支持类型：Oracle, MySQL, PostgreSQL, MongoDB
        """
        pass
    
    def connect_market_data(self, provider):
        """
        连接行情数据
        
        支持：Bloomberg, Reuters, Wind, 内部系统
        """
        pass
    
    def connect_trading_system(self, system_name):
        """
        连接交易系统
        
        支持：Summit, Calypso, Murex, 自研系统
        """
        pass
```

### 2. 基础建模层 (Base Modeling)

```python
class CurveBuilder:
    """曲线构建器"""
    
    def build_yield_curve(self, instruments, method="cubic_spline"):
        """
        构建收益率曲线
        
        Methods: linear, cubic_spline, nelson_siegel
        """
        pass
    
    def build_forward_curve(self, spot_curve, tenors):
        """构建远期曲线"""
        pass
    
    def build_volatility_surface(self, options_data):
        """构建波动率曲面"""
        pass

class PricingEngine:
    """定价引擎"""
    
    def price_bond(self, bond_params, market_data):
        """债券定价"""
        pass
    
    def price_swap(self, swap_params, curve):
        """利率互换定价"""
        pass
    
    def price_option(self, option_params, market_data, model="black_scholes"):
        """期权定价"""
        pass
```

### 3. 统一接口层 (Unified Interface)

```python
class FICCServiceBus:
    """FICC服务总线"""
    
    def register_service(self, service_name, service_instance):
        """注册服务"""
        pass
    
    def get_service(self, service_name):
        """获取服务"""
        pass
    
    def invoke_service(self, service_name, method, params):
        """调用服务"""
        pass

class DataAdapter:
    """数据适配器"""
    
    def adapt_market_data(self, raw_data, target_format):
        """适配市场数据"""
        pass
    
    def adapt_trade_data(self, raw_data, target_format):
        """适配交易数据"""
        pass
    
    def adapt_risk_data(self, raw_data, target_format):
        """适配风险数据"""
        pass
```

## 与上层插件的集成

```python
# 业务插件使用核心服务示例

from ficc_core import CurveBuilder, PricingEngine, DataConnectionManager

class FixedIncomePlugin:
    def __init__(self):
        self.curve_builder = CurveBuilder()
        self.pricing_engine = PricingEngine()
        self.data_conn = DataConnectionManager()
    
    def analyze_bond_portfolio(self, portfolio):
        # 使用核心服务获取市场数据
        market_data = self.data_conn.connect_market_data("bloomberg")
        
        # 使用核心服务构建收益率曲线
        yield_curve = self.curve_builder.build_yield_curve(
            instruments=market_data.get_instruments(),
            method="cubic_spline"
        )
        
        # 使用核心服务定价
        for bond in portfolio.bonds:
            price = self.pricing_engine.price_bond(
                bond_params=bond.params,
                market_data=market_data
            )
```

## 依赖项

- pandas >= 2.0.0
- numpy >= 1.24.0
- scipy >= 1.11.0
