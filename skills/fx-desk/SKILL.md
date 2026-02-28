# FX Desk

## Description
外汇业务完整技能体系，覆盖汇率分析、外汇交易、外汇衍生品、跨境资金管理等全链条业务能力。

## Capabilities

### 汇率分析 (FX Rate Analysis)
- **基本面分析**：购买力平价、利率平价、国际收支、经济增长差异
- **技术面分析**：趋势、支撑阻力、形态、技术指标
- **宏观驱动**：央行政策、通胀差异、贸易平衡、资本流动
- **市场情绪**：持仓数据、风险情绪、事件驱动
- **量化模型**：统计套利、机器学习预测、高频信号

### 外汇交易 (FX Trading)
- **即期交易**： spot 汇率报价、买卖价差、点值计算
- **远期交易**： outright forward、远期点、掉期点
- **掉期交易**： FX swap、期限结构、滚动操作
- **期权交易**： vanilla options、exotic options、波动率交易
- **交叉汇率**： cross rates、三角套利、流动性管理
- **算法交易**： TWAP、VWAP、市场冲击、智能路由
### 外汇衍生品 (FX Derivatives)
- **远期外汇合约**：定价、对冲、展期、提前交割
- **外汇期权**： Black-Scholes、Garman-Kohlhagen、波动率曲面
- **货币互换**： cross-currency swap、基差、利差
- **结构性产品**： range accrual、目标赎回、可敲出/敲入
- **奇异期权**： barrier、binary、asian、lookback、compound
- **对冲策略**： delta hedge、gamma scalping、vega exposure
### 跨境资金管理 (Cross-border Treasury)
- **多币种现金池**： physical cash pool、notional cash pool、target balancing
- **内部银行**： in-house bank、资金集中、内部定价
- **净额结算**： payment netting、receipt netting、bilateral/multilateral
- **跨境流动性**： trapped cash、regulatory restrictions、tax efficient
- **汇率风险管理**： natural hedge、financial hedge、accounting hedge
- **资金预测**： cash forecasting、working capital management、seasonality
### 监管与合规 (Regulatory & Compliance)
- **外管局政策**： SAFE regulations、RMB convertibility、cross-border RMB
- **资本管制**： FD/ODI restrictions、SAFE filing、banking documentation
- **报告义务**： regulatory reporting、statistical reporting、large transaction
- **制裁合规**： OFAC、EU sanctions、screening、blocked accounts
- **反洗钱**： AML/KYC、suspicious activity monitoring、transaction surveillance
- **税务筹划**： withholding tax、permanent establishment、transfer pricing、tax treaties
## Commands

```bash
# 汇率分析
/fx-desk rate-analysis --pair USD/CNY --method technical+fundamental --forecast 3M
/fx-desk sentiment-analysis --currency CNH --sources positioning+news+orderflow
/fx-desk carry-trade --base AUD --funding JPY --notional 10M
# 外汇交易
/fx-desk spot-trade --pair EUR/USD --side buy --amount 5M --settlement T+2
/fx-desk forward-trade --pair USD/CNY --side sell --amount 10M --tenor 3M
/fx-desk swap-trade --near-leg sell-USD-buy-CNY --far-leg buy-USD-sell-CNY --notional 10M --tenor-1 1W --tenor-2 1M
# 外汇衍生品
/fx-desk option-price --type EUR-call-USD-put --strike 1.1000 --expiry 3M --vol-atm 7.5 --valuation Vanna-Volga
/fx-desk structured-product --type accumulator --underlying USD/CNH --strike 7.2000 --knock-out 7.0000 --notional-per-fixing 1M
# 跨境资金管理
/fx-desk cash-pooling --structure notional --currencies USD+EUR+CNH --master USD --sweep-frequency daily
/fx-desk cross-border-loan --borrower 境内子公司 --lender 境外总部 --amount 50M --currency USD --tenor 1Y
/fx-desk hedge-accounting --exposure net-investment-in-foreign-operation --instrument FX-forward --effectiveness 80-125%
# 监管与合规
/fx-desk SAFE-filing --transaction-type FD --amount 10M --currency USD --counterparty 境外投资者
/fx-desk sanctions-screening --counterparty Bank-of-Iran --lists OFAC-EU-UN --result match-found
```
## Implementation
### 汇率预测模型
```python
class ExchangeRateModel:
    """汇率预测综合模型"""
    
    def __init__(self, pair, methods=['ppppp', 'irp', 'microstructure', 'ml']):
        self.pair = pair
        self.methods = methods
        self.models = {}
        
    def fit(self, historical_data, macro_data, order_flow_data):
        """训练各个子模型"""
        
        # 1. 购买力平价 (PPP)
        if 'pppp' in self.methods:
            self.models['pppp'] = self._fit_ppp_model(
                historical_data['spot'],
                macro_data['cpi_differential'],
                macro_data['real_exchange_rate']
            )
        
        # 2. 利率平价 (IRP)
        if 'irp' in self.methods:
            self.models['irp'] = self._fit_irp_model(
                historical_data['forward'],
                historical_data['spot'],
                macro_data['interest_rate_differential']
            )
        
        # 3. 微观结构模型 (Order Flow)
        if 'microstructure' in self.methods:
            self.models['microstructure'] = self._fit_microstructure_model(
                order_flow_data['signed_volume'],
                order_flow_data['trade_intensity'],
                historical_data['price_change']
            )
        
        # 4. 机器学习模型
        if 'ml' in self.methods:
            features = self._engineer_features(
                historical_data, macro_data, order_flow_data
            )
            self.models['ml'] = self._fit_ml_model(
                features, historical_data['future_return']
            )
    
    def predict(self, horizon, scenario=None):
        """综合预测"""
        predictions = {}
        
        for method, model in self.models.items():
            pred = self._predict_single(model, method, horizon, scenario)
            predictions[method] = pred
        
        # 加权组合（可基于历史表现动态调整权重）
        ensemble = self._ensemble_predictions(predictions)
        
        return {
            'individual_predictions': predictions,
            'ensemble_prediction': ensemble,
            'prediction_interval': self._calculate_interval(ensemble),
            'confidence': self._assess_confidence(predictions)
        }
```
### 外汇期权定价
```python
class FXOptionPricer:
    """外汇期权定价器，支持多种模型"""
    
    def __init__(self, model='garman-kohlhagen'):
        self.model = model
        
    def price(self, option_type, S, K, T, r_d, r_f, sigma, 
              method='closed-form'):
        """
        定价外汇期权
        
        Parameters:
        - option_type: 'call' or 'put'
        - S: spot FX rate (domestic/foreign)
        - K: strike
        - T: time to maturity in years
        - r_d: domestic risk-free rate
        - r_f: foreign risk-free rate
        - sigma: volatility
        - method: 'closed-form', 'monte-carlo', 'pde'
        """
        
        if self.model == 'garman-kohlhagen':
            if method == 'closed-form':
                return self._garman_kohlhagen_closed_form(
                    option_type, S, K, T, r_d, r_f, sigma
                )
            elif method == 'monte-carlo':
                return self._garman_kohlhagen_monte_carlo(
                    option_type, S, K, T, r_d, r_f, sigma
                )
            elif method == 'pde':
                return self._garman_kohlhagen_pde(
                    option_type, S, K, T, r_d, r_f, sigma
                )
        
        elif self.model == 'vanna-volga':
            # Vanna-Volga 模型用于考虑 smile 的定价
            return self._vanna_volga_pricing(
                option_type, S, K, T, r_d, r_f, 
                atm_vol, rr, bf  # risk reversal, butterfly
            )
        
        elif self.model == 'local-vol':
            # 局部波动率模型
            return self._local_vol_pricing(
                option_type, S, K, T, r_d, r_f, local_vol_surface
            )
        
        elif self.model == 'heston':
            # Heston 随机波动率模型
            return self._heston_pricing(
                option_type, S, K, T, r_d, r_f,
                v0, kappa, theta, sigma, rho
            )
    
    def _garman_kohlhagen_closed_form(self, option_type, S, K, T, 
                                       r_d, r_f, sigma):
        """
        Garman-Kohlhagen 封闭解
        适用于欧式外汇期权
        """
        from scipy.stats import norm
        
        d1 = (np.log(S/K) + (r_d - r_f + 0.5*sigma**2)*T) / (sigma*np.sqrt(T))
        d2 = d1 - sigma*np.sqrt(T)
        
        if option_type == 'call':
            price = (S * np.exp(-r_f*T) * norm.cdf(d1) - 
                    K * np.exp(-r_d*T) * norm.cdf(d2))
        else:  # put
            price = (K * np.exp(-r_d*T) * norm.cdf(-d2) - 
                    S * np.exp(-r_f*T) * norm.cdf(-d1))
        
        return price
```
## Dependencies
- quantlib-python (QuantLib Python bindings)
- numpy / pandas / scipy (numerical computation)
- statsmodels (statistical models)
- cvxpy (convex optimization for portfolio/hedging)
## License
Apache-2.0 (aligns with Anthropic financial-services-plugins)