# Command: fx-forward

## Description
外汇远期合约定价与交易执行工具，覆盖即期、远期、掉期全产品线。

## Usage

```bash
/fx-desk fx-forward [OPTIONS]
```

## Options

| Option | Description | Required |
|--------|-------------|----------|
| `--pair` | 货币对 (USD/CNY, EUR/USD) | Yes |
| `--direction` | 方向 (buy/sell) | Yes |
| `--notional` | 名义本金 | Yes |
| `--tenor` | 期限 (SPOT, 1W, 1M, 3M, 1Y) | Yes |
| `--settlement` | 交割方式 (DELIVERABLE/NON-DELIVERABLE) | No |

## Examples

```bash
# 3个月美元/人民币远期购汇
/fx-desk fx-forward --pair USD/CNY --direction buy --notional 10000000 --tenor 3M --settlement DELIVERABLE

# 1年 EUR/USD 远期
/fx-desk fx-forward --pair EUR/USD --direction sell --notional 5000000 --tenor 1Y
```

## Output

```json
{
  "trade": {
    "pair": "USD/CNY",
    "direction": "BUY",
    "notional": 10000000,
    "tenor": "3M",
    "value_date": "2024-09-15"
  },
  "pricing": {
    "spot": 7.2450,
    "forward_points": -450,
    "outright_forward": 7.2000,
    "swap_rate": -1.80
  },
  "market_data": {
    "usd_cny_carry": -1.80,
    "implied_yield_diff": -2.40
  }
}
```
