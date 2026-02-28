# Risk Management

## Description

商业银行 FICC 业务全面风险管理体系，严格遵循 Anthropic financial-services-plugins 架构标准，覆盖市场风险、信用风险、流动性风险、操作风险四大类别，提供风险识别、计量、监控、报告完整能力。深度融合 ficc-analysis-skill 的风险指标与归因分析能力。

## Capabilities

### 1. 市场风险 (Market Risk)

#### 1.1 风险价值模型 (VaR/CVaR)

**参数法 (Parametric/Variance-Covariance)**:

```python
class ParametricVaR:
    """参数法VaR计算"""
    
    def __init__(self, confidence_level=0.99, holding_period=1):
        self.confidence_level = confidence_level
        self.holding_period = holding_period
        self.z_score = self._get_z_score(confidence_level)
        
    def calculate(self, portfolio_value, returns_std, correlation_matrix=None, weights=None):
        """
        计算参数法VaR
        
        公式: VaR = Z_α × σ_p × √t × PortfolioValue
        其中 σ_p = √(w^T × Σ × w) 为组合标准差
        """
        if correlation_matrix is not None and weights is not None:
            # 多资产组合
            portfolio_std = np.sqrt(
                weights.T @ correlation_matrix @ weights
            ) * returns_std
        else:
            # 单一资产
            portfolio_std = returns_std
            
        var = (self.z_score * 
               portfolio_std * 
               np.sqrt(self.holding_period) * 
               portfolio_value)
        
        return {
            'var': var,
            'var_pct': var / portfolio_value * 100,
            'confidence_level': self.confidence_level,
            'holding_period': self.holding_period,
            'z_score': self.z_score,
            'portfolio_std': portfolio_std,
            'method': 'parametric'
        }
```

**历史模拟法 (Historical Simulation)**:

```python
class HistoricalSimulationVaR:
    """历史模拟法VaR计算"""
    
    def __init__(self, confidence_level=0.99, holding_period=1, lookback_days=252):
        self.confidence_level = confidence_level
        self.holding_period = holding_period
        self.lookback_days = lookback_days
        
    def calculate(self, historical_returns, current_portfolio_value, weights=None):
        """
        计算历史模拟法VaR
        
        方法：使用历史收益率数据，按持有期滚动计算组合损益，取分位数
        """
        # 处理多资产情况
        if weights is not None:
            # 加权计算组合收益率
            portfolio_returns = (historical_returns * weights).sum(axis=1)
        else:
            portfolio_returns = historical_returns
            
        # 考虑持有期的复利效应
        if self.holding_period > 1:
            # 滚动窗口计算持有期收益率
            rolling_returns = portfolio_returns.rolling(window=self.holding_period).apply(
                lambda x: (1 + x).prod() - 1
            ).dropna()
        else:
            rolling_returns = portfolio_returns
            
        # 计算损益分布
        pnl_distribution = rolling_returns * current_portfolio_value
        
        # 计算VaR（分位数）
        var_percentile = (1 - self.confidence_level) * 100
        var = np.percentile(pnl_distribution, var_percentile)
        
        # 计算CVaR（Expected Shortfall）
        cvar = pnl_distribution[pnl_distribution <= var].mean()
        
        return {
            'var': abs(var),
            'var_pct': abs(var) / current_portfolio_value * 100,
            'cvar': abs(cvar),
            'cvar_pct': abs(cvar) / current_portfolio_value * 100,
            'confidence_level': self.confidence_level,
            'holding_period': self.holding_period,
            'lookback_days': self.lookback_days,
            'method': 'historical_simulation',
            'pnl_distribution': pnl_distribution.tolist()
        }
```

**蒙特卡洛模拟法 (Monte Carlo Simulation)**:

```python
class MonteCarloVaR:
    """蒙特卡洛模拟法VaR计算"""
    
    def __init__(self, confidence_level=0.99, holding_period=1, n_simulations=10000, n_days=252):
        self.confidence_level = confidence_level
        self.holding_period = holding_period
        self.n_simulations = n_simulations
        self.n_days = n_days
        
    def calculate(self, current_portfolio_value, mean_return, std_return, 
                  correlation_matrix=None, factor_loadings=None):
        """
        计算蒙特卡洛VaR
        
        方法：基于统计分布假设生成随机路径，模拟组合价值变化
        """
        np.random.seed(42)  # 保证可重复性
        
        if factor_loadings is not None and correlation_matrix is not None:
            # 多因子蒙特卡洛模拟
            n_factors = correlation_matrix.shape[0]
            
            # 生成相关的随机因子
            L = np.linalg.cholesky(correlation_matrix)
            random_factors = np.random.standard_normal((self.n_simulations, n_factors))
            correlated_factors = random_factors @ L.T
            
            # 通过因子载荷计算资产收益
            simulated_returns = (
                mean_return * self.holding_period / self.n_days +
                (correlated_factors @ factor_loadings.T) * 
                np.sqrt(self.holding_period / self.n_days)
            )
        else:
            # 单资产蒙特卡洛模拟
            simulated_returns = (
                mean_return * self.holding_period / self.n_days +
                std_return * np.sqrt(self.holding_period / self.n_days) * 
                np.random.standard_normal(self.n_simulations)
            )
            
        # 计算组合价值变化
        portfolio_changes = current_portfolio_value * simulated_returns
        
        # 计算VaR
        var_percentile = (1 - self.confidence_level) * 100
        var = np.percentile(portfolio_changes, var_percentile)
        
        # 计算CVaR
        cvar = portfolio_changes[portfolio_changes <= var].mean()
        
        # 压力测试结果
        stress_99 = np.percentile(portfolio_changes, 1)
        stress_95 = np.percentile(portfolio_changes, 5)
        
        return {
            'var': abs(var),
            'var_pct': abs(var) / current_portfolio_value * 100,
            'cvar': abs(cvar),
            'cvar_pct': abs(cvar) / current_portfolio_value * 100,
            'confidence_level': self.confidence_level,
            'holding_period': self.holding_period,
            'n_simulations': self.n_simulations,
            'method': 'monte_carlo',
            'stress_tests': {
                '99_percentile': abs(stress_99),
                '95_percentile': abs(stress_95)
            },
            'distribution_stats': {
                'mean': np.mean(portfolio_changes),
                'std': np.std(portfolio_changes),
                'skewness': self._calculate_skewness(portfolio_changes),
                'kurtosis': self._calculate_kurtosis(portfolio_changes)
            }
        }
```

#### 1.2 风险归因分析 (Risk Attribution)

```python
class RiskAttribution:
    """风险归因分析"""
    
    def __init__(self, portfolio_positions, factor_exposures, factor_covariance):
        self.positions = portfolio_positions
        self.factor_exposures = factor_exposures
        self.factor_covariance = factor_covariance
        
    def calculate_factor_contribution(self):
        """计算因子风险贡献"""
        # 计算组合对因子的暴露
        portfolio_factor_exposure = (
            self.positions.T @ self.factor_exposures
        )
        
        # 计算因子对总风险的贡献
        factor_contribution = (
            portfolio_factor_exposure * 
            (self.factor_covariance @ portfolio_factor_exposure)
        )
        
        # 计算百分比贡献
        total_risk = np.sum(factor_contribution)
        factor_contribution_pct = factor_contribution / total_risk * 100
        
        return {
            'factor_exposures': portfolio_factor_exposure,
            'factor_contribution': factor_contribution,
            'factor_contribution_pct': factor_contribution_pct,
            'total_risk': total_risk
        }
        
    def calculate_position_contribution(self):
        """计算持仓风险贡献"""
        # 计算每个持仓的风险贡献（边际风险）
        portfolio_variance = (
            self.positions.T @ 
            self.factor_exposures @ 
            self.factor_covariance @ 
            self.factor_exposures.T @ 
            self.positions
        )
        
        # 计算每个持仓的边际风险贡献
        marginal_contributions = []
        for i, position in enumerate(self.positions):
            # 边际风险 = 组合对该持仓的敏感度 × 持仓大小
            marginal_risk = (
                self.factor_exposures[i] @ 
                self.factor_covariance @ 
                self.factor_exposures.T @ 
                self.positions
            )
            marginal_contributions.append({
                'position_id': i,
                'position_size': position,
                'marginal_risk': marginal_risk,
                'risk_contribution': position * marginal_risk
            })
            
        # 计算风险贡献百分比
        total_risk_contribution = sum(m['risk_contribution'] for m in marginal_contributions)
        for m in marginal_contributions:
            m['risk_contribution_pct'] = (
                m['risk_contribution'] / total_risk_contribution * 100
            )
            
        return {
            'marginal_contributions': marginal_contributions,
            'total_risk_contribution': total_risk_contribution,
            'portfolio_variance': portfolio_variance
        }
```

### 2. 信用风险 (Credit Risk)

#### 2.1 信用风险计量模型

```python
class CreditRiskModel:
    """信用风险计量模型"""
    
    def __init__(self, model_type='internal_rating'):
        self.model_type = model_type
        
    def calculate_pd(self, entity_data):
        """计算违约概率 (Probability of Default)"""
        if self.model_type == 'internal_rating':
            return self._internal_rating_pd(entity_data)
        elif self.model_type == 'merton':
            return self._merton_pd(entity_data)
        elif self.model_type == 'logistic':
            return self._logistic_pd(entity_data)
            
    def _internal_rating_pd(self, entity_data):
        """内部评级法计算PD"""
        # 基于内部评级映射
        rating = entity_data['internal_rating']
        # 评级到PD的映射表
        pd_mapping = {
            'AAA': 0.0001, 'AA+': 0.0002, 'AA': 0.0003,
            'AA-': 0.0005, 'A+': 0.0007, 'A': 0.0010,
            'A-': 0.0014, 'BBB+': 0.0020, 'BBB': 0.0028,
            'BBB-': 0.0040, 'BB+': 0.0060, 'BB': 0.0090,
            'BB-': 0.0135, 'B+': 0.0200, 'B': 0.0300,
            'B-': 0.0450, 'CCC+': 0.0700, 'CCC': 0.1100,
            'CCC-': 0.1700, 'CC': 0.2600, 'C': 0.4000,
            'D': 1.0000
        }
        return pd_mapping.get(rating, 0.05)
        
    def calculate_lgd(self, facility_data, downturn=True):
        """计算违约损失率 (Loss Given Default)"""
        # 基础LGD
        base_lgd = facility_data.get('base_lgd', 0.45)
        
        if downturn:
            # 经济下行期的LGD（ downturn LGD）
            downturn_adjustment = facility_data.get('downturn_adjustment', 0.10)
            return min(base_lgd + downturn_adjustment, 0.99)
        else:
            return base_lgd
            
    def calculate_ead(self, exposure_data, method='current_exposure'):
        """计算违约风险敞口 (Exposure at Default)"""
        if method == 'current_exposure':
            # 当前敞口法
            return exposure_data['current_exposure']
        elif method == 'original_exposure':
            # 原始敞口法
            return exposure_data['original_exposure']
        elif method == 'expected_positive_exposure':
            # 预期正敞口法（用于衍生品）
            return self._calculate_epe(exposure_data)
            
    def calculate_expected_loss(self, pd, lgd, ead):
        """计算预期损失 (Expected Loss)"""
        return pd * lgd * ead
        
    def calculate_unexpected_loss(self, pd, lgd, ead, correlation):
        """计算非预期损失 (Unexpected Loss / Economic Capital)"""
        # 使用Asymptotic Single Risk Factor (ASRF) 模型
        # UL = EAD × LGD × Φ(Φ^-1(PD) × (1-R)^-0.5 + Φ^-1(0.999) × (R/(1-R))^0.5) - PD × LGD
        
        r = correlation  # 资产相关性
        
        numerator = sp.stats.norm.ppf(pd) + sp.stats.norm.ppf(0.999) * np.sqrt(r)
        denominator = np.sqrt(1 - r)
        
        capital_requirement = ead * lgd * sp.stats.norm.cdf(numerator / denominator) - pd * lgd * ead
        
        return capital_requirement
```

### 3. 流动性风险 (Liquidity Risk)

#### 3.1 流动性覆盖率 (LCR) 与净稳定资金比率 (NSFR)

```python
class LiquidityRiskMetrics:
    """流动性风险指标计算"""
    
    def calculate_lcr(self, hqla, net_cash_outflows):
        """
        计算流动性覆盖率 (Liquidity Coverage Ratio)
        
        公式: LCR = 高质量流动性资产 / 未来30天净现金流出 ≥ 100%
        """
        lcr = hqla / net_cash_outflows if net_cash_outflows > 0 else float('inf')
        
        return {
            'lcr': lcr,
            'lcr_pct': lcr * 100,
            'hqla': hqla,
            'net_cash_outflows': net_cash_outflows,
            'compliance': lcr >= 1.0,
            'regulatory_requirement': '≥ 100%'
        }
        
    def calculate_nsfr(self, available_stable_funding, required_stable_funding):
        """
        计算净稳定资金比率 (Net Stable Funding Ratio)
        
        公式: NSFR = 可用稳定资金 / 所需稳定资金 ≥ 100%
        """
        nsfr = (available_stable_funding / required_stable_funding 
                if required_stable_funding > 0 else float('inf'))
        
        return {
            'nsfr': nsfr,
            'nsfr_pct': nsfr * 100,
            'available_stable_funding': available_stable_funding,
            'required_stable_funding': required_stable_funding,
            'compliance': nsfr >= 1.0,
            'regulatory_requirement': '≥ 100%'
        }
        
    def calculate_liquidity_gap(self, cash_inflows, cash_outflows, time_buckets):
        """
        计算流动性缺口分析
        
        分析不同时间段的现金流入、流出和缺口
        """
        gap_analysis = []
        cumulative_gap = 0
        
        for bucket in time_buckets:
            # 计算该时间段的流入和流出
            inflow = sum(cf['amount'] for cf in cash_inflows 
                        if bucket['start'] <= cf['date'] <= bucket['end'])
            outflow = sum(cf['amount'] for cf in cash_outflows 
                         if bucket['start'] <= cf['date'] <= bucket['end'])
            
            # 计算缺口
            gap = inflow - outflow
            cumulative_gap += gap
            
            gap_analysis.append({
                'time_bucket': bucket['label'],
                'start_date': bucket['start'],
                'end_date': bucket['end'],
                'cash_inflow': inflow,
                'cash_outflow': outflow,
                'liquidity_gap': gap,
                'cumulative_gap': cumulative_gap,
                'gap_ratio': gap / outflow if outflow > 0 else 0
            })
            
        return gap_analysis
```

### 4. 操作风险 (Operational Risk)

#### 4.1 损失分布法 (LDA - Loss Distribution Approach)

```python
class OperationalRiskLDA:
    """操作风险损失分布法"""
    
    def __init__(self, business_line, event_type):
        self.business_line = business_line
        self.event_type = event_type
        self.frequency_model = None
        self.severity_model = None
        
    def fit_frequency_model(self, historical_loss_counts, model='poisson'):
        """
        拟合损失频率模型
        
        常用模型：
        - Poisson：适用于损失事件发生相对独立且平均发生率稳定
        - Negative Binomial：适用于损失事件存在过度离散（方差>均值）
        """
        if model == 'poisson':
            # Poisson分布: P(X=k) = (λ^k × e^-λ) / k!
            lambda_param = np.mean(historical_loss_counts)
            self.frequency_model = {
                'type': 'poisson',
                'lambda': lambda_param
            }
        elif model == 'negative_binomial':
            # Negative Binomial分布
            mean = np.mean(historical_loss_counts)
            variance = np.var(historical_loss_counts)
            p = mean / variance
            r = mean * p / (1 - p)
            self.frequency_model = {
                'type': 'negative_binomial',
                'r': r,
                'p': p
            }
            
    def fit_severity_model(self, historical_loss_amounts, model='lognormal'):
        """
        拟合损失严重度模型
        
        常用模型：
        - Lognormal：适用于损失金额右偏分布
        - Pareto：适用于极端损失（厚尾）
        - Weibull：适用于复杂形状的分布
        """
        # 剔除极值（如内部欺诈的极端值可能单独处理）
        threshold = np.percentile(historical_loss_amounts, 99)
        body_losses = historical_loss_amounts[historical_loss_amounts <= threshold]
        tail_losses = historical_loss_amounts[historical_loss_amounts > threshold]
        
        if model == 'lognormal':
            # Lognormal分布拟合
            log_losses = np.log(body_losses)
            mu = np.mean(log_losses)
            sigma = np.std(log_losses)
            self.severity_model = {
                'type': 'lognormal',
                'mu': mu,
                'sigma': sigma,
                'tail_threshold': threshold,
                'tail_losses': tail_losses
            }
        elif model == 'pareto':
            # Pareto分布拟合（用于尾部）
            xmin = np.min(body_losses)
            alpha = len(body_losses) / np.sum(np.log(body_losses / xmin))
            self.severity_model = {
                'type': 'pareto',
                'xmin': xmin,
                'alpha': alpha
            }
            
    def aggregate_loss_distribution(self, n_simulations=100000):
        """
        聚合损失分布
        
        方法：蒙特卡洛模拟
        1. 从频率模型抽样损失次数
        2. 从严重度模型抽样每次损失金额
        3. 聚合得到年度总损失分布
        """
        annual_losses = []
        
        for _ in range(n_simulations):
            # 抽样损失次数
            if self.frequency_model['type'] == 'poisson':
                n_losses = np.random.poisson(self.frequency_model['lambda'])
            elif self.frequency_model['type'] == 'negative_binomial':
                n_losses = np.random.negative_binomial(
                    self.frequency_model['r'],
                    self.frequency_model['p']
                )
                
            # 抽样损失金额
            total_loss = 0
            for _ in range(n_losses):
                if self.severity_model['type'] == 'lognormal':
                    loss = np.random.lognormal(
                        self.severity_model['mu'],
                        self.severity_model['sigma']
                    )
                elif self.severity_model['type'] == 'pareto':
                    loss = (np.random.pareto(self.severity_model['alpha']) + 1) * \
                           self.severity_model['xmin']
                total_loss += loss
                
            annual_losses.append(total_loss)
            
        annual_losses = np.array(annual_losses)
        
        # 计算风险指标
        var_99 = np.percentile(annual_losses, 99)
        var_95 = np.percentile(annual_losses, 95)
        cvar_99 = annual_losses[annual_losses >= var_99].mean()
        
        return {
            'annual_loss_distribution': annual_losses,
            'mean_loss': np.mean(annual_losses),
            'std_loss': np.std(annual_losses),
            'var_99': var_99,
            'var_95': var_95,
            'cvar_99': cvar_99,
            'maximum_loss': np.max(annual_losses)
        }
```

## Commands

```bash
# VaR计算
/risk var-calculation --portfolio组合A --confidence 99 --horizon 1d --method historical

# 风险归因
/risk risk-attribution --portfolio组合A --factors "利率,信用,汇率"

# 信用风险计量
/risk credit-exposure --counterparty对手A --netting-set主协议 --collateral CSA

# RAROC/EVA计算
/risk performance-metrics --business-line FICC --period 2024Q1

# 流动性风险
/risk liquidity-coverage --hqla 1000 --net-outflows 800

# 操作风险
/risk operational-lda --business-line trading --event-type execution

# 压力测试
/risk stress-test --scenarios "利率上行200bp,信用利差扩大100bp"
```

## Integration with FICC Skills

### 与 ficc-analysis-skill 的深度融合

| ficc-analysis 能力 | risk-management 融合 | 输出成果 |
|-------------------|---------------------|----------|
| VaR/ES 计算 | 实时风险计量 | 动态风险限额监控 |
| 损益归因 | 风险归因分解 | 风险调整后绩效 |
| RAROC/EVA | 经济资本计算 | 风险调整后回报 |
| 压力测试 | 情景分析 | 极端损失预估 |
| 对冲有效性 | 基差风险计量 | 套保效果评估 |

## License

Apache-2.0 (aligns with Anthropic financial-services-plugins)
