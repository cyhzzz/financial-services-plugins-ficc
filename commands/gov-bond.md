# Command: gov-bond

## Description
国债及利率债分析工具，提供收益率曲线、估值、策略分析全链条能力。

## Usage

```bash
/fixed-income gov-bond [OPTIONS]
```

## Options

| Option | Description | Required |
|--------|-------------|----------|
| `--country` | 国家/地区 (CN/US/DE/JP) | Yes |
| `--maturity` | 期限 (2Y/5Y/10Y/30Y) | Yes |
| `--yield` | 当前收益率 | No |
| `--price` | 当前价格 | No |
| `--analysis` | 分析类型 (basic/technical/fundamental/full) | No |

## Examples

```bash
# 中国10年期国债全面分析
/fixed-income gov-bond --country CN --maturity 10Y --yield 2.65 --analysis full

# 美国国债技术分析
/fixed-income gov-bond --country US --maturity 10Y --yield 4.25 --analysis technical
```

## Output

```json
{
  "bond": {
    "country": "CN",
    "maturity": "10Y",
    "yield": 2.65,
    "duration": 8.45,
    "dv01": 0.0845
  },
  "analysis": {
    "fair_value": 2.58,
    "yield_spread": 65,
    "curve_position": "steepener",
    "recommendation": "BUY"
  }
}
```
