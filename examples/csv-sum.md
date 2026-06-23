# CSV 求和

**任务：** “写一段 Python 代码，读取 sales.csv，并求 'amount' 列的总和。”

以下内容源自一次 benchmark 运行的模型输出，Claude Haiku 4.5，未启用 skill 组 vs ponytail 组，temperature 1，来源 `benchmarks/output.json`。复现：`npx promptfoo@latest eval -c benchmarks/promptfooconfig.yaml`。

## 未启用 Ponytail，20 行代码

# 读取 sales.csv 并求 'amount' 列总和的 Python 代码

```python
import pandas as pd

# 读取 CSV 文件
df = pd.read_csv('sales.csv')

# 求 'amount' 列总和
total_amount = df['amount'].sum()

# 打印结果
print(f"Total amount: ${total_amount:,.2f}")
```

## 替代方法：

**使用 CSV 模块（不使用 pandas）：**
```python
import csv

total_amount = 0

with open('sales.csv', 'r') as file:
    reader = csv.DictReader(file)
    for row in reader:
        total_amount += float(row['amount'])

print(f"Total amount: ${total_amount:,.2f}")
```

**使用 pandas 并带错误处理：**
```python
import pandas as pd

try:
    df = pd.read_csv('sales.csv')
    total_amount = df['amount'].sum()
    print(f"Total amount: ${total_amount:,.2f}")
except FileNotFoundError:
    print("Error: sales.csv not found")
except KeyError:
    print("Error: 'amount' column not found")
```

**推荐 pandas 方法**，因为它：
- 更简洁、可读
- 自动处理数据类型
- 提供内建错误处理
- 对大文件更高效

## 启用 Ponytail，3 行代码

```python
import csv

total = sum(float(row['amount']) for row in csv.DictReader(open('sales.csv')))
print(total)
```

省掉了：pandas、错误处理、关闭文件。等 CSV 很大、格式不可靠，或你需要更多分析时再加。

**20 → 3 行代码**，同一个模型，同一个提示。
