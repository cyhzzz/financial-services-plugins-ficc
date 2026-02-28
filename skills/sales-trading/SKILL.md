# Sales Trading

## Description
销售交易部核心技能体系，覆盖做市报价、代理交易、撮合服务、同业销售等全链条业务能力。

## Capabilities

### 做市与报价 (Market Making)
- **双边报价**：国债、政策性金融债、地方债做市
- **报价策略**：基于库存、风险限额、市场条件的动态报价
- **价差管理**：买卖价差优化、市场深度维护
- **库存管理**：债券持仓管理、久期对冲、曲线头寸
- **自动做市**：算法报价、高频调整、市场信号响应

### 代理交易 (Agency Trading)
- **客户订单执行**：代理买卖债券、执行客户指令
- **最佳执行义务**：价格改进、市场时机、交易成本分析
- **大额交易执行**：冰山订单、分批执行、市场影响最小化
- **跨市场交易**：交易所、银行间、柜台市场比较执行
- **交易报告**：成交确认、交易细节、监管报送

### 撮合服务 (Intermediation)
- **大宗交易撮合**：大额债券买卖方匹配
- **协议交易安排**：一对一大宗交易、协商定价
- **流动性提供**：为稀缺债券寻找对手方
- **跨机构撮合**：银行、券商、基金、保险间撮合
- **匿名撮合服务**：保护客户身份的大额交易执行

### 同业销售 (Interbank Sales)
- **金融机构覆盖**：银行、券商、基金、保险、信托客户关系
- **产品推介**：债券、衍生品、结构性产品推介
- **市场信息传递**：利率走势、信用事件、交易机会
- **定制化服务**：根据客户需求设计交易策略
- **客户维护**：定期拜访、服务跟进、满意度管理

### 交易支持与运营 (Trading Support)
- **交易确认**：交易要素核对、确认单发送
- **清算结算**：DVP结算、资金划拨、托管对接
- **风险监控**：敞口监控、限额检查、异常预警
- **损益分析**：交易P&L归因、策略绩效评估
- **监管报送**：交易报告、持仓报告、大额报告

## Commands

```bash
# 做市报价
/sales-trading quote --instrument 国债230015 --side two-way --size 1000万 --spread 0.5bp

# 代理交易执行
/sales-trading agency --client基金A --order买入国开债 --amount 5000万 --strategy TWAP --duration 2h

# 大宗撮合
/sales-trading block-match --security农发债220408 --amount 2亿 --price 2.85 --buyer机构X --seller机构Y

# 同业销售服务
/sales-trading client-service --institution券商B --service市场信息 --content今日利率债走势分析

# 交易确认
/sales-trading confirm --trade-id 202406280001 --confirm-details交易要素核对
```

## Implementation

### 做市定价模型

```python
class MarketMakerPricing:
    """做市商定价模型"""
    
    def __init__(self, inventory, risk_limits, market_conditions):
        self.inventory = inventory  # 当前持仓
        self.risk_limits = risk_limits  # 风险限额
        self.market_conditions = market_conditions  # 市场条件
        
    def calculate_two_way_price(self, bond, base_price):
        """计算双边报价"""
        
        # 1. 基础价差
        base_spread = self._get_base_spread(bond.liquidity)
        
        # 2. 库存调整
        inventory_adjustment = self._calculate_inventory_adjustment(bond)
        
        # 3. 风险调整
        risk_adjustment = self._calculate_risk_adjustment(bond)
        
        # 4. 市场条件调整
        market_adjustment = self._calculate_market_adjustment()
        
        # 综合调整
        total_adjustment = (inventory_adjustment + risk_adjustment + market_adjustment)
        
        bid = base_price - base_spread/2 - total_adjustment
        ask = base_price + base_spread/2 + total_adjustment
        
        return {
            'bid': bid,
            'ask': ask,
            'mid': (bid + ask) / 2,
            'spread': ask - bid,
            'adjustments': {
                'inventory': inventory_adjustment,
                'risk': risk_adjustment,
                'market': market_adjustment
            }
        }
```

## Dependencies

- numpy / pandas (numerical computation)
- scipy (optimization)
- redis (market data caching)
- kafka (market data streaming)

## License

Apache-2.0 (aligns with Anthropic financial-services-plugins)
