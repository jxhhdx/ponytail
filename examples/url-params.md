# URL 参数

**任务：** “解析和构建 URL 查询字符串。”

## 未启用 Ponytail

```bash
npm install query-string
# gzip 后 4.5 kB，每周 350 万下载
```

```js
import qs from "query-string";

// 解析
const params = qs.parse(location.search);
// → { page: "2", sort: "name", tags: ["js", "css"] }

// 构建
const url = qs.stringify({ page: 2, sort: "name", tags: ["js", "css"] });
// → "page=2&sort=name&tags=js&tags=css"
```

## 启用 Ponytail

```js
// ponytail: URLSearchParams 就是做这个的
const params = new URLSearchParams(location.search);

// 读取
params.get("page");         // "2"
params.getAll("tags");      // ["js", "css"]

// 构建
const out = new URLSearchParams({ page: 2, sort: "name" });
out.append("tags", "js");
out.append("tags", "css");
out.toString(); // "page=2&sort=name&tags=js&tags=css"
```

**1 个依赖 → 0 个依赖。** `URLSearchParams` 存在于所有浏览器中，Node.js 从 v10 起也支持。它会处理编码、重复键和迭代。这个包曾经是在给一个早已全平台发布的 API 做 polyfill。
