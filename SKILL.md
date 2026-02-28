# Financial Services Plugins - FICC Edition

## 项目概述

基于 Anthropic financial-services-plugins 架构的商业银行 FICC（固定收益、外汇、大宗商品）业务完整技能体系。

## 架构设计

```
├── Core Plugins（核心插件）
│   ├── ficc-core/          # FICC 核心 - 数据连接、基础建模、统一接口
│   └── risk-management/    # 风险管理 - 市场风险、信用风险、流动性风险
│
├── Business Plugins（业务插件）
│   ├── fixed-income/       # 固收业务 - 利率债、信用债、衍生品
│   ├── fx-desk/            # 外汇业务 - 汇率分析、外汇交易、衍生品
│   └── commodities/        # 大宗商品 - 贵金属、能源、基本金属
│
└── Client Plugins（对客插件）
    ├── client-solutions/   # 对客服务 - 套保方案、结构性产品、交易执行
    └── sales-trading/      # 销售交易 - 报价、做市、撮合
```

## 核心能力

### 1. 固收业务 (Fixed Income)

**利率债分析**:
- 国债、政金债、地方债分析
- 收益率曲线构建与解读
- 骑乘策略、久期管理
- 利率敏感度分析

**信用债分析**:
- 金融债、企业债、中票短融分析
- 信用评级与利差分析
- 信用风险预警
- 城投债、地产债专项分析

**利率衍生品**:
- IRS（利率互换）定价与交易
- 国债期货策略
- 利率期权应用
- 套期保值方案设计

### 2. 外汇业务 (FX Desk)

**汇率分析**:
- 美元指数、人民币汇率走势分析
- 中美利差、经常账户、资本流动分析
- 央行干预预期
- 技术分析与量化模型

**外汇交易**:
- 即期、远期、掉期交易
- 结售汇、外汇买卖
- 套息交易分析
- 流动性管理

**外汇衍生品**:
- 外汇期权策略
- 货币掉期应用
- 奇异期权介绍
- 套保方案设计

### 3. 大宗商品 (Commodities)

**贵金属**:
- 黄金、白银分析框架
- 实际利率、美元指数、避险需求
- 央行购金、ETF持仓
- 投资策略与风险管理

**能源**:
- 原油市场分析（WTI、布伦特）
- OPEC+政策、地缘政治
- 库存数据、炼厂开工率
- 油气产业链投资

**基本金属**:
- 铜、铝、锌等分析
- 供需平衡、库存周期
- 中国需求、全球宏观
- 产业链投资机会

### 4. 风险管理

**市场风险**:
- VaR、CVaR计算
- 久期、凸性、基点价值
- Greeks 风险指标
- 压力测试与情景分析

**信用风险**:
- 信用评级迁移
- 违约概率、违约损失率
- 信用敞口计量
- 信用风险缓释

**流动性风险**:
- 流动性覆盖率、净稳定资金比率
- 现金流缺口分析
- 应急融资计划
- 流动性压力测试

**操作风险**:
- 操作风险识别
- 关键风险指标
- 损失数据收集
- 业务连续性计划

### 5. 对客服务

**套期保值方案**:
- 利率风险对冲（IRS、国债期货）
- 汇率风险对冲（远期、期权、货币掉期）
- 商品价格对冲（期货、期权、互换）
- 套保会计处理

**结构性产品**:
- 结构性存款设计
- 收益凭证（固收+期权）
- 雪球、安全气囊等结构
- 产品定价与风险管理

**交易执行**:
- 代客交易（代理买卖）
- 做市服务（双边报价）
- 撮合服务（大额交易）
- 算法交易（TWAP、VWAP）

## 🛠️ 使用指南

### 安装

```bash
# 添加到 Claude 插件市场
claude plugin marketplace add your-org/financial-services-plugins-ficc

# 安装核心插件
claude plugin install ficc-core@financial-services-plugins-ficc
claude plugin install risk-management@financial-services-plugins-ficc

# 安装业务插件
claude plugin install fixed-income@financial-services-plugins-ficc
claude plugin install fx-desk@financial-services-plugins-ficc
claude plugin install commodities@financial-services-plugins-ficc

# 安装对客插件
claude plugin install client-solutions@financial-services-plugins-ficc
claude plugin install sales-trading@financial-services-plugins-ficc
```

### 常用命令

```bash
# 利率债分析
/fixed-income government-bond --type国债 --maturity 10y --yield 2.65%


# 信用债分析
/fixed-income credit-bond --issuer城投 --rating AA+ --spread 150bp

# 收益率曲线分析
/fixed-income yield-curve --build --tenor 1y-30y

# IRS 定价
/fixed-income irs-pricing --notional 1亿 --tenor 5y --freq季度

# 汇率分析
/fx-desk rate-analysis --pair USD/CNY --spot 7.20 --forward 3m

# 外汇交易
/fx-desk trade --type远期 --pair USD/CNY --amount 1000万 --tenor 3m

# 黄金分析
/commodities gold --type现货 --price 480 --currency CNY

# 原油分析
/commodities crude-oil --type布伦特 --price 85 --contract主力

# 风险管理
/risk var-calculation --portfolio组合A --confidence 99% --horizon 1d

# 套保方案
/client-solutions hedge --risk汇率风险 --notional 5000万 --strategy远期

# 结构性产品
/client-solutions structured-product --type雪球 --tenor 2y --strike 80%
```

## 🏛️ 架构对齐 Anthropic

本项目严格对齐 Anthropic financial-services-plugins 的架构设计：

| Anthropic 插件 | 本项目对应模块 | 适配说明 |
|----------------|----------------|----------|
| `financial-analysis` | `ficc-core/` + `fixed-income/` + `fx-desk/` + `commodities/` | 从股票/债券分析扩展到 FICC 全品类 |
| `equity-research` | `fixed-income/credit-research/`, `fx-desk/currency-research/` | 从个股研究转为信用债、汇率研究 |
| `wealth-management` | `client-solutions/` + `risk-management/` | 从零售财富转为机构对客服务 |
| `investment-banking` | `sales-trading/` + `client-solutions/structured-products/` | 从投行转为销售交易和结构性产品 |

## 📊 与投顾项目对比

| 维度 | 投顾项目 (To C) | FICC 项目 (To B) |
|------|-----------------|------------------|
| **目标用户** | 个人投资者 | 商业银行、金融机构 |
| **核心产品** | 公募基金 | 利率债、信用债、外汇、衍生品 |
| **服务模式** | 资产配置、投资建议 | 做市、交易执行、风险管理 |
| **风险等级** | R1-R5 | 专业投资者级别 |
| **合规要求** | 适当性管理 | 更严格的交易合规、风控 |
| **分析维度** | 基金筛选、资产配置 | 利率分析、信用研究、衍生品定价 |

## 🚀 未来规划

### 短期（1-3个月）
- [ ] 接入 Bloomberg、Reuters 等专业数据源
- [ ] 对接银行交易系统（Summit、Murex 等）
- [ ] 完善定价模型（IRS、Swaption、FX Option 等）
- [ ] 开发实时风险监控系统

### 中期（3-6个月）
- [ ] 开发资产负债管理（ALM）模块
- [ ] 建立客户信用评级系统
- [ ] 开发监管报送自动化（1104、LE 等）
- [ ] 接入更多市场数据源（Tradeweb、MarketAxess 等）

### 长期（6-12个月）
- [ ] AI 增强定价和风险管理
- [ ] 自动化交易执行和做市
- [ ] 跨境业务能力扩展
- [ ] 开放 API 生态建设

## 📞 联系我们

- **GitHub**: https://github.com/cyhzzz/financial-services-plugins-ficc
- **Issues**: https://github.com/cyhzzz/financial-services-plugins-ficc/issues
- **Email**: your-email@example.com

## 📄 许可证

Apache License 2.0（与 Anthropic financial-services-plugins 保持一致）

---

**Made with ❤️ for FICC Professionals**
