# Command: var-calculation

## Description
计算投资组合的风险价值 (Value at Risk)，支持参数法、历史模拟法和蒙特卡洛模拟法。

## Usage

```bash
/risk var-calculation [OPTIONS]
```

## Options

| Option | Description | Default |
|--------|-------------|---------|
| `--portfolio` | 投资组合标识 | - |
| `--confidence` | 置信水平 (95%, 99%) | 99% |
| `--horizon` | 持有期 (1d, 10d) | 1d |
| `--method` | 计算方法 (parametric, historical, monte-carlo) | historical |
| `--data-window` | 历史数据窗口 (1Y, 3Y) | 2Y |

## Examples

```bash
# 历史模拟法计算 VaR
/risk var-calculation --portfolio 组合A --confidence 99 --horizon 1d --method historical

# 参数法计算 VaR
/risk var-calculation --portfolio 组合B --confidence 95 --horizon 10d --method parametric
```

## Output

```json
{
  "var": {
    "value": -1250000,
    "confidence": 0.99,
    "horizon": "1d",
    "method": "historical"
  },
  "cvar": -1850000,
  "worst_case": -5200000,
  "breaches_1y": 2,
  "backtest_result": "GREEN"
}
```
