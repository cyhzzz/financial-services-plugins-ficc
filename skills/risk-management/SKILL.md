# Risk Management

## Description
FICC 业务全面风险管理体系，覆盖市场风险、信用风险、流动性风险、操作风险四大类别，提供风险识别、计量、监控、报告完整能力。

## Capabilities

### 市场风险 (Market Risk)
- **风险价值 (VaR/CVaR)**：参数法、历史模拟法、蒙特卡洛模拟
- **敏感性分析 (Greeks)**：Delta、Gamma、Vega、Theta、Rho
- **固定收益久期分析**：DV01、修正久期、有效久期、关键利率久期
- **情景分析与压力测试**：历史情景、假设情景、反向压力测试
- **风险归因与分解**：风险贡献度、边际风险、成分风险
- **P&L 归因分析**：市场变动贡献、新品种贡献、交易活动贡献

### 信用风险 (Credit Risk)
- **违约概率 (PD)**：内部评级法、外部评级映射、Merton模型、机器学习模型
- **违约损失率 (LGD)**：历史数据、回收率分析、 downturn LGD
- **违约风险敞口 (EAD)**：当前敞口、潜在未来敞口、净额结算、 collateral
- **信用风险缓释**：担保、信用衍生品、净额结算、 collateral
- **信用组合模型**：CreditMetrics、CreditRisk+、CreditPortfolioView
- **交易对手信用风险 (CVA)**：CVA/DVA/FVA、Wrong-way risk
- **信用评级迁徙**：迁移矩阵、信用评级变化预测
- **集中度风险**：单一对手、行业、地区、产品集中度
### 流动性风险 (Liquidity Risk)
- **融资流动性风险**：现金流缺口分析、期限错配、融资集中度
- **市场流动性风险**：买卖价差、市场深度、市场弹性、冲击成本
- **压力情景分析**：机构特定压力、市场系统性压力、组合压力
- **流动性覆盖率 (LCR)**：高质量流动性资产、净现金流出
- **净稳定资金比率 (NSFR)**：稳定资金来源、稳定资金需求
- **应急融资计划 (CFP)**：融资应急方案、资产出售、央行工具
- **日内流动性监控**：日间支付流量、实时头寸、透支额度
- **货币错配分析**：跨币种流动性、外汇敞口、央行互换额度
### 操作风险 (Operational Risk)
- **损失数据收集 (LDC)**：内部损失数据、外部损失数据、情景分析
- **关键风险指标 (KRI)**：预警指标、阈值设定、监控频率
- **风险评估与控制自我评估 (RCSA)**：流程识别、风险评估、控制设计
- **业务连续性计划 (BCP)**：关键业务识别、恢复时间目标、演练测试
- **第三方风险管理**：供应商风险评估、合同管理、持续监控
- **网络与信息安全**：网络安全评估、数据保护、事件响应
- **模型风险管理**：模型验证、独立审查、持续监控
- **欺诈风险**：欺诈识别、调查程序、恢复措施
- **法律与合规风险**：监管变化、合规监控、报告义务
## Commands

```bash
# 市场风险计算
/risk market-var --portfolio组合A --method historical --confidence 99 --horizon 1d
/risk market-cvar --portfolio组合A --confidence 99
/risk sensitivities --instrument债券 --shock +1bp
/risk key-rate-durations --portfolio组合A --tenors 6M-30Y

# 信用风险计算
/risk credit-exposure --counterparty对手A --netting-set主协议 --collateral无
/risk cva-calculation --portfolio衍生品组合 --wrong-way-risk false
/risk pd-estimation --entity实体A --method internal-rating
/risk lgd-estimation --collateral-type房产 --region一线城市

# 流动性风险计算
/risk lcr-calculation --hqla高质量流动性资产 --net-outflows净现金流出
/risk nsfr-calculation --asf稳定资金来源 --rsf稳定资金需求
/risk cashflow-gap --tenors隔夜-5Y --currency CNY
/risk liquidity-stress --scenario系统性压力 --horizon 30d
# 操作风险计算
/risk operational-lda --frequency频率 --severity严重度 --threshold阈值
/risk kri-monitor --indicator系统故障次数 --threshold 5 --breach-action escalate
/risk scenario-analysis --scenario网络攻击 --expert-input专家判断
# 风险报告
/risk report-market --date每日 --content VaR-attribution-stress-test
/risk report-credit --date每周 --content exposure-concentration-migration
/risk report-liquidity --date每月 --content LCR-NSFR-gap-analysis
/risk report-operational --date每季 --content losses-KRIs-scenarios
/risk report-integrated --date每半年 --content all-risk-types-correlation
```
## Implementation
### 风险模型实现

```python
# 市场风险 - VaR 计算
class VaRModel:
    def __init__(self, method="historical", confidence=0.99, horizon=1):
        self.method = method  # historical, parametric, monte_carlo
        self.confidence = confidence
        self.horizon = horizon  # in days
        
    def calculate(self, portfolio_returns, weights=None):
        if self.method == "historical":
            return self._historical_var(portfolio_returns)
        elif self.method == "parametric":
            return self._parametric_var(portfolio_returns, weights)
        elif self.method == "monte_carlo":
            return self._monte_carlo_var(portfolio_returns, weights)
            
    def _historical_var(self, returns):
        """历史模拟法"""
        # 根据持有期调整收益率
        if self.horizon > 1:
            returns = self._aggregate_returns(returns, self.horizon)
        
        # 计算分位数
        var_percentile = 1 - self.confidence
        var = np.percentile(returns, var_percentile * 100)
        return abs(var)  # 通常报告为正数
    
    def _parametric_var(self, returns, weights):
        """方差-协方差法（参数法）"""
        # 计算组合收益的均值和标准差
        portfolio_mean = np.mean(returns)
        portfolio_std = np.std(returns)
        
        # 根据持有期调整
        if self.horizon > 1:
            portfolio_mean *= self.horizon
            portfolio_std *= np.sqrt(self.horizon)
        
        # 计算 VaR（假设正态分布）
        z_score = stats.norm.ppf(1 - self.confidence)
        var = -(portfolio_mean + z_score * portfolio_std)
        return var
    
    def _monte_carlo_var(self, returns, weights, n_simulations=10000):
        """蒙特卡洛模拟法"""
        # 估计收益率分布参数
        mean = np.mean(returns)
        std = np.std(returns)
        
        # 生成随机收益率路径
        np.random.seed(42)
        simulated_returns = np.random.normal(mean, std, n_simulations)
        
        # 计算 VaR
        var = np.percentile(simulated_returns, (1 - self.confidence) * 100)
        return abs(var)


# 信用风险 - CVA 计算
class CVAModel:
    def __init__(self, discount_curve, recovery_rate=0.4, wrong_way_risk=False):
        self.discount_curve = discount_curve
        self.recovery_rate = recovery_rate
        self.wrong_way_risk = wrong_way_risk
        
    def calculate(self, derivatives_portfolio, counterparty_pd, correlation=None):
        """计算 CVA（信用估值调整）"""
        cva = 0
        
        for derivative in derivatives_portfolio:
            # 计算预期敞口（EE）
            expected_exposure = self._calculate_expected_exposure(
                derivative, time_grid
            )
            
            # 计算违约概率（PD）
            default_probability = self._calculate_default_probability(
                counterparty_pd, time_grid
            )
            
            # 计算 LGD
            loss_given_default = 1 - self.recovery_rate
            
            # 考虑 Wrong Way Risk
            if self.wrong_way_risk and correlation is not None:
                adjustment = self._apply_wrong_way_risk(
                    expected_exposure, default_probability, correlation
                )
                expected_exposure *= adjustment
            
            # 计算 CVA 贡献
            for t in time_grid:
                cva += (expected_exposure[t] * 
                       default_probability[t] * 
                       loss_given_default *
                       self.discount_curve.discount_factor(t))
        
        return cva


# 流动性风险 - LCR/NSFR 计算
class LiquidityMetrics:
    def __init__(self, reporting_date):
        self.reporting_date = reporting_date
        
    def calculate_lcr(self, hqla, net_cash_outflows):
        """计算流动性覆盖率 (Liquidity Coverage Ratio)"""
        """
        LCR = High Quality Liquid Assets / Net Cash Outflows over 30 days
        Requirement: >= 100%
        """
        lcr = hqla / net_cash_outflows if net_cash_outflows > 0 else float('inf')
        return {
            'ratio': lcr,
            'hqla': hqla,
            'net_cash_outflows': net_cash_outflows,
            'compliance': lcr >= 1.0
        }
    
    def calculate_nsfr(self, available_stable_funding, required_stable_funding):
        """计算净稳定资金比率 (Net Stable Funding Ratio)"""
        """
        NSFR = Available Stable Funding / Required Stable Funding
        Requirement: >= 100%
        """
        nsfr = (available_stable_funding / required_stable_funding 
                if required_stable_funding > 0 else float('inf'))
        return {
            'ratio': nsfr,
            'available_stable_funding': available_stable_funding,
            'required_stable_funding': required_stable_funding,
            'compliance': nsfr >= 1.0
        }
    
    def calculate_liquidity_gap(self, cash_flows, time_buckets):
        """计算流动性缺口分析"""
        gaps = {}
        cumulative_gap = 0
        
        for bucket in time_buckets:
            inflows = sum(cf['amount'] for cf in cash_flows 
                         if cf['type'] == 'inflow' and cf['date'] <= bucket['end'])
            outflows = sum(cf['amount'] for cf in cash_flows 
                          if cf['type'] == 'outflow' and cf['date'] <= bucket['end'])
            
            gap = inflows - outflows
            cumulative_gap += gap
            
            gaps[bucket['name']] = {
                'inflows': inflows,
                'outflows': outflows,
                'gap': gap,
                'cumulative_gap': cumulative_gap
            }
        
        return gaps

```
## Dependencies
- quantlib-python (QuantLib Python bindings)
- numpy / pandas / scipy (numerical computation)
- statsmodels (statistical models)
- cvxpy (convex optimization for portfolio/hedging)
## License
Apache-2.0 (aligns with Anthropic financial-services-plugins)