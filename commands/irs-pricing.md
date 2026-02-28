# Command: irs-pricing

## Description
利率互换（IRS）定价与估值工具，支持多曲线贴现和 Greeks 计算。

## Usage

```bash
/fixed-income irs-pricing [OPTIONS]
```

## Options

| Option | Description | Required |
|--------|-------------|----------|
| `--notional` | 名义本金 | Yes |
| `--maturity` | 期限 (1Y/3Y/5Y/10Y) | Yes |
| `--fixed-rate` | 固定端利率 | No |
| `--float-leg` | 浮动端基准 (3M-Shibor/1Y-LPR/SOFR) | Yes |
| `--valuation` | 估值方法 (single-curve/multi-curve/OIS) | No |
| `--calculate-greeks` | 是否计算 Greeks | No |

## Examples

```bash
# 5年期 IRS 定价，多曲线框架
/fixed-income irs-pricing --notional 100000000 --maturity 5Y --fixed-rate 2.85 --float-leg 3M-Shibor --valuation multi-curve --calculate-greeks
```

## Output

```json
{
  "irs": {
    "notional": 100000000,
    "fixed_rate": 2.85,
    "maturity": "5Y",
    "npv": -125000,
    "par_rate": 2.82
  },
  "greeks": {
    "dv01": -45000,
    "duration": 4.5,
    "convexity": 0.23
  },
  "risk_report": {
    "scenario_+100bp": -4500000,
    "scenario_+200bp": -8900000
  }
}
```
