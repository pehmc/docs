## CTA策略

当价格触及布林带下轨时做多，触及上轨时平多并反向做空，预期价格回归中轨。

### 策略逻辑

- **理论依据**：价格在无强趋势时会围绕移动平均线波动，布林带将波动范围量化。
- **适用市场**：震荡市、无明确趋势的品种。
- **失效场景**：单边强趋势市场（价格会不断沿上下轨运行）。

### 指标计算

- 中轨 $MA = \text{SMA}(Close, N)$
- 上轨 $Upper = MA + K \times \sigma$
- 下轨 $Lower = MA - K \times \sigma$  
  ($\sigma$ 为过去 $N$ 期收盘价标准差，$K$ 通常为 2)

### 交易信号

| 条件 | 动作 |
|------|------|
| $Close \le Lower$ 且无持仓 | 开多仓 |
| $Close \ge Upper$ 且持多仓 | 平多仓 |
| $Close \ge Upper$ 且无持仓 | 开空仓 |
| $Close \le Lower$ 且持空仓 | 平空仓 |

### 风险控制

- 单笔止损：ATR(14) 的 2 倍
- 持仓超过 10 根 K 线未回归则强制平仓

### 实现代码（关键部分）

```python
def calculate_signals(df, period=20, std_dev=2):
    df['ma'] = df['close'].rolling(period).mean()
    df['std'] = df['close'].rolling(period).std()
    df['upper'] = df['ma'] + std_dev * df['std']
    df['lower'] = df['ma'] - std_dev * df['std']

    df['signal'] = 0
    df.loc[df['close'] <= df['lower'], 'signal'] = 1   # 买入
    df.loc[df['close'] >= df['upper'], 'signal'] = -1  # 卖出
    return df
```

### 回测结果记录

- **测试品种**：沪深300 指数，2023-01-01 ~ 2025-12-31
- **绩效指标**：
  - 年化收益率：8.2%
  - 最大回撤：12.3%
  - 夏普比率：0.92
  - 胜率：48%
- **观察与思考**：
  - 2024 年趋势行情中出现了连续 5 次止损，说明需要加入趋势过滤器（如 ADX）。
  - 手续费敏感，需使用限价单并设置滑点。

### 参考链接

- 原书章节摘要
- 相关论文：Bollinger, J. (2002). *Bollinger on Bollinger Bands*
