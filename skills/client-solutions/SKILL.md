# Client Solutions

## Description
商业银行 FICC 对客服务完整解决方案体系，为企业客户、金融机构客户提供利率风险、汇率风险、商品价格风险的全方位套期保值服务，以及结构性产品设计与交易执行服务。

## Capabilities

### 利率风险管理 (Interest Rate Risk Management)
- **风险识别与计量**：
  - 利率敏感性缺口分析（RSA/RSL）
  - 久期缺口分析（Duration Gap）
  - 基点价值（DV01）计算
  - 收益风险（Earnings at Risk）
  - 经济价值风险（Economic Value of Equity）
  
- **套期保值策略**：
  - 利率互换（IRS）：收固定/付浮动，锁定融资成本
  - 利率上限/下限（Cap/Floor）：限制利率波动区间
  - 利率区间（Collar）：零成本套保结构
  - 利率互换期权（Swaption）：保留利率下行收益
  - 国债期货：低成本、高流动性套保
  
- **企业客户场景**：
  - 浮动利率贷款客户：转为固定成本
  - 重资产企业：长期负债利率锁定
  - 房地产企业：项目融资利率管理
  - 城投平台：存量债务成本优化


### 汇率风险管理 (FX Risk Management)
- **风险识别与计量**：
  - 交易敞口（Transaction Exposure）
  - 折算敞口（Translation Exposure）
  - 经济敞口（Economic Exposure）
  - 风险价值（VaR）与压力测试
  - 现金流缺口分析
  
- **套期保值工具**：
  - 远期外汇（FX Forward）：锁定未来汇率
  - 外汇期权（FX Option）：保留有利波动
  - 区间远期（Forward Extra）：零成本增强型
  - 参与远期（Participating Forward）：部分参与
  - 货币掉期（Cross Currency Swap）：长期负债币种转换
  - 结构性存款：存款+期权的组合
  
- **企业客户场景**：
  - 进出口企业：收付汇汇率锁定
  - 跨境投融资：外债汇率风险管理
  - 境外上市：募集资金汇率保值
  - 跨国企业：全球资金池汇率管理
  - 跨境电商：多币种收款管理


### 商品价格风险管理 (Commodity Risk Management)
- **贵金属套保**：
  - 黄金生产企业：卖出远期、领式期权
  - 黄金珠宝商：买入远期、库存管理
  - 黄金精炼商：加工价差锁定
  - 黄金投资者：组合保险策略
  
- **基本金属套保**：
  - 铜冶炼企业：TC/RC 谈判、价格分享
  - 电线电缆厂：铜价采购锁定
  - 铜加工企业：销售定价保值
  - 废铜回收商：库存价差管理
  
- **能源套保**：
  - 航空公司：航煤价格对冲
  - 航运企业：燃油成本控制
  - 炼油企业：裂解利润锁定
  - 化工企业：原料成本管理


### 结构性产品设计 (Structured Products)
- **结构性存款**：
  - 收益增强型：区间累积、目标收益
  - 保本浮动型：本金保障+挂钩收益
  - 杠杆参与型：放大挂钩标的表现
  - 多资产型：股票+商品+汇率组合
  
- **结构化票据**：
  - 雪球结构 (Snowball)：区间累积收益，敲出/敲入机制
  - 安全气囊 (Airbag)：下跌保护+收益增强
  - 鲨鱼鳍 (Shark Fin)：单边看涨/看跌增强
  - 彩虹期权 (Rainbow)：多资产最优表现
  - 凤凰结构 (Phoenix)：自动重启，月度派息
  
- **收益凭证**：
  - 固定收益+期权组合
  - 保本型、非保本型
  - 场内交易、场外定制
  
- **产品设计要素**：
  - 挂钩标的：股票指数、商品、汇率、利率
  - 期限结构：短期（3-6月）、中期（1-2年）、长期（3-5年）
  - 收益结构：固定收益、浮动收益、触发式
  - 风险等级：R1-R5，对应不同客户群体
  - 流动性安排：赎回条款、转让机制、做市安排


### 交易执行与运营 (Trade Execution & Operations)
- **交易执行服务**：
  - 代客交易：代理买卖、最优执行、交易报告
  - 做市服务：双边报价、流动性提供、点差管理
  - 撮合服务：大额交易、协议交易、匿名撮合
  - 算法交易：TWAP、VWAP、Implementation Shortfall、Dark Pool
  
- **交易后处理**：
  - 交易确认：确认单匹配、异议处理
  - 清算结算：中央对手方、双边清算、DVP/PVP
  - 对账管理：双边对账、异常处理、余额调节
  - 报表生成：交易报告、持仓报告、风险报告
  
- **文档与合规**：
  - ISDA 主协议：NAFMII、SAC 协议
  - CSA/VMIA：保证金协议、担保品管理
  - 交易授权：内部审批、限额管理、事前/事后监控
  - 监管报送：央行、外管局、银保监、交易所


## Commands

```bash
# 利率风险管理
/client-solutions irs-hedge --exposure 5亿浮息贷款 --risk利率上行 --tool IRS-pay-fixed --tenor 5Y --target锁定融资成本
/client-solutions cap-floor --exposure 3亿 --structure零成本领式 --cap-rate 4% --floor-rate 2.5%
/client-solutions swaption --notional 2亿 --option-type收固定 --expiry 1Y --underlying-tenor 5Y

# 汇率风险管理
/client-solutions fx-forward --exposure收美元1000万 --settlement 3M --hedge-ratio 100%
/client-solutions fx-option --type USD-call-CNY-put --notional 500万 --strike 7.20 --expiry 6M --strategy零成本区间远期
/client-solutions ccs --debt 1亿美元贷款 --currency USD --swap-to CNY --tenor 3Y --amortizing true

# 商品风险管理
/client-solutions gold-hedge --producer年产金1吨 --strategy卖出远期+买入看跌 --coverage 80%
/client-solutions copper-hedge --exposure年用铜5000吨 --tool买入看涨+卖出看跌 --structure领式期权
/client-solutions oil-hedge --airline航煤年需求100万桶 --strategy裂解利润互换+价格上限

# 结构性产品设计
/client-solutions structured-deposit --principal保本 --tenor 1Y --return-type区间累积 --underlying黄金 --range 450-500 --coupon年化3.5%
/client-solutions snowball --nominal 1000万 --underlying中证500 --strike 100% --ko 103% --ki 75% --coupon年化15% --tenor 24M
/client-solutions phoenix --nominal 500万 --underlying恒生指数 --ko 102% --monthly-coupon 1.2% --autocall-monthly --tenor 36M

# 交易执行
/client-solutions execution --trade买入国债10亿 --method TWAP --duration 2h --urgency medium
/client-solutions rfq --product USD/CNY远期 --amount 5000万 --tenor 1Y --counterparties 5
/client-solutions algo-vwap --trade卖出股票组合 --participation 15% --market-cap-weighted true

# 文档与合规
/client-solutions isda-checklist --counterparty央企财务公司 --threshold A- --collateral CSA
/client-solutions margin-calculation --exposure 2亿 --threshold 500万 --mta 25万 --collateral-cash
/client-solutions regulatory-reporting --type外管局-对外资产负债 --frequency季度 --currency-breakdown true
```

## Implementation

### 套期保值效果评估模型

```python
class HedgeEffectiveness:
    """套期保值效果评估"""
    
    def __init__(self, hedged_portfolio, unhedged_portfolio, hedge_instruments):
        self.hedged = hedged_portfolio
        self.unhedged = unhedged_portfolio
        self.hedge_instruments = hedge_instruments
        
    def calculate_effectiveness(self, method='dollar-offset'):
        """计算套期保值有效性"""
        
        if method == 'dollar-offset':
            # 美元抵消法
            hedged_pnl = self.hedged.pnl()
            unhedged_pnl = self.unhedged.pnl()
            
            effectiveness = abs(hedged_pnl / unhedged_pnl) if unhedged_pnl != 0 else 0
            
        elif method == 'variance-reduction':
            # 方差减少法
            unhedged_variance = np.var(self.unhedged.returns())
            hedged_variance = np.var(self.hedged.returns())
            
            effectiveness = 1 - (hedged_variance / unhedged_variance)
            
        elif method == 'regression':
            # 回归法（80%-125% 有效性标准）
            from sklearn.linear_model import LinearRegression
            
            X = self.hedge_instruments.returns().reshape(-1, 1)
            y = self.unhedged.returns()
            
            model = LinearRegression().fit(X, y)
            effectiveness = model.coef_[0]  # 斜率应在 0.8 到 1.25 之间
        
        return {
            'method': method,
            'effectiveness': effectiveness,
            'is_highly_effective': 0.8 <= effectiveness <= 1.25 if method == 'regression' else effectiveness >= 0.8,
            'hedge_ratio': self._optimal_hedge_ratio(),
            'documentation': self._generate_documentation()
        }
```

## Dependencies

- quantlib-python (QuantLib Python bindings)
- numpy / pandas / scipy (numerical computation)
- cvxpy (convex optimization for hedging)
- sklearn (machine learning for hedge ratio optimization)

## License

Apache-2.0 (aligns with Anthropic financial-services-plugins)
