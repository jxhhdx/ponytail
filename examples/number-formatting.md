# 数字格式化

**任务：** “把数字格式化为货币，并加上千位分隔符。”

## 未启用 Ponytail

```bash
npm install numeral
# 或：npm install accounting
```

```js
import numeral from "numeral";

numeral(1234567.89).format("$1,234.00"); // "$1,234,567.89"
numeral(0.745).format("0.0%");           // "74.5%"
numeral(1500).format("0.0a");            // "1.5k"
```

## 启用 Ponytail

```js
// ponytail: Intl.NumberFormat 就是做这个的，并且支持 locale
new Intl.NumberFormat("en-US", { style: "currency", currency: "USD" })
  .format(1234567.89);
// → "$1,234,567.89"

new Intl.NumberFormat("en-US", { style: "percent" })
  .format(0.745);
// → "74.5%"

new Intl.NumberFormat("en-US", { notation: "compact" })
  .format(1500);
// → "1.5K"
```

**1 个依赖 → 0 个依赖。** `Intl.NumberFormat` 内建于所有 JS 运行时，能正确处理每种 locale，并为任何市场正确处理货币符号、小数分隔符和分组。硬编码格式的库总会对某些人出错。
