# 速率限制

**任务：** “给我的 FastAPI endpoint 添加速率限制，防止用户刷接口。”

以下内容源自一次 benchmark 运行的模型输出，Claude Haiku 4.5，未启用 skill 组 vs ponytail 组，temperature 1，来源 `benchmarks/output.json`。复现：`npx promptfoo@latest eval -c benchmarks/promptfooconfig.yaml`。

## 未启用 Ponytail，128 行代码

# FastAPI 中的速率限制

下面是几种方案，从最简单到最健壮：

## 1. **使用 `slowapi`（推荐，最简单）**

```bash
pip install slowapi
```

```python
from fastapi import FastAPI, Request
from slowapi import Limiter
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from fastapi.responses import JSONResponse

app = FastAPI()

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, lambda request, exc: JSONResponse(
    status_code=429,
    content={"detail": "Rate limit exceeded"}
))

@app.get("/api/data")
@limiter.limit("5/minute")
async def get_data(request: Request):
    return {"message": "Success"}
```

**速率限制格式：**
- `"5/minute"` - 每分钟 5 次请求
- `"100/hour"` - 每小时 100 次请求
- `"1000/day"` - 每天 1000 次请求

---

## 2. **使用 `limits` 库（更多控制）**

```bash
pip install limits
```

```python
from fastapi import FastAPI, HTTPException, Request
from limits import parse
from limits.storage import MemoryStorage
from limits.strategies import MovingWindowRateLimiter

app = FastAPI()

storage = MemoryStorage()
limiter = MovingWindowRateLimiter(storage)
rate_limit = parse("5/minute")

@app.get("/api/data")
async def get_data(request: Request):
    client_id = request.client.host
    
    if not limiter.hit(rate_limit, client_id):
        raise HTTPException(status_code=429, detail="Rate limit exceeded")
    
    return {"message": "Success"}
```

---

## 3. **自定义中间件（完全控制）**

```python
from fastapi import FastAPI, HTTPException, Request
from datetime import datetime, timedelta
from collections import defaultdict
import asyncio

app = FastAPI()

# 按 IP 存储请求时间戳
request_history = defaultdict(list)
RATE_LIMIT = 5  # 请求数
TIME_WINDOW = 60  # 秒

@app.middleware("http")
async def rate_limit_middleware(request: Request, call_next):
    client_ip = request.client.host
    now = datetime.now()
    
    # 清理时间窗口之外的旧请求
    request_history[client_ip] = [
        req_time for req_time in request_history[client_ip]
        if now - req_time < timedelta(seconds=TIME_WINDOW)
    ]
    
    # 检查是否超过限制
    if len(request_history[client_ip]) >= RATE_LIMIT:
        return JSONResponse(
            status_code=429,
            content={"detail": "Rate limit exceeded"}
        )
    
    # 记录本次请求
    request_history[client_ip].append(now)
    
    return await call_next(request)

@app.get("/api/data")
async def get_data():
    return {"message": "Success"}
```

---

## 4. **按用户限流（带认证）**

```python
from fastapi import FastAPI, Depends, HTTPException
from slowapi import Limiter
from slowapi.util import get_remote_address

app = FastAPI()
limiter = Limiter(key_func=get_remote_address)

def get_user_id(token: str = Header(None)) -> str:
    # 你的认证逻辑
    return token or "anonymous"

@app.get("/api/data")
@limiter.limit("10/minute")
async def get_data(request: Request, user_id: str = Depends(get_user_id)):
    return {"message": f"Success for {user_id}"}
```

---

## 5. **基于 Redis 的速率限制（生产环境）**

```bash
pip install slowapi redis
```

```python
from fastapi import FastAPI, Request
from slowapi import Limiter
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from slowapi.storage import RedisStorage
from redis import Redis
from fastapi.responses import JSONResponse

redis_client = Redis.from_url("redis://localhost:6379")
storage = RedisStorage(redis_client)
limiter = Limiter(key_func=get_remote_address, storage=storage)

app = FastAPI()
app.state.limiter = limiter

app.add_exception_handler(
    RateLimitExceeded,
    lambda request, exc: JSONResponse(
        status_code=429,
        content={"detail": "Rate limit exceeded"}
    )
)

@app.get("/api/data")
@limiter.limit("5/minute")
async def get_data(request: Request):
    return {"message": "Success"}
```

---

## 6. **带多个 endpoint 的完整示例**

```python
from fastapi import FastAPI, Request
from slowapi import Limiter
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded
from fastapi.responses import JSONResponse

app = FastAPI()
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.exception_handler(RateLimitExceeded)
async def rate_limit_handler(request: Request, exc: RateLimitExceeded):
    return JSONResponse(
        status_code=429,
        content={
            "detail": "Too many requests",
            "retry_after": exc.detail
        }
    )

# 严格限制
@app.post("/api/login")
@limiter.limit("3/minute")
async def login(request: Request):
    return {"token": "xxx"}

# 中等限制
@app.get("/api/data")
@limiter.limit("30/minute")
async def get_data(request: Request):
    return {"data": []}

# 宽松限制
@app.get("/api/public")
@limiter.limit("100/minute")
async def get_public(request: Request):
    return {"public": "data"}
```

---

## 对比表

| 方法 | 优点 | 缺点 | 最适合 |
|--------|------|------|----------|
| **slowapi** | 简单，基于装饰器 | 仅内存 | 开发、小应用 |
| **limits** | 灵活，多个后端 | 配置更多 | 自定义逻辑 |
| **中间件** | 完全控制，无依赖 | 手动清理 | 学习、简单场景 |
| **Redis** | 分布式，可扩展 | 需要 Redis | 生产、多服务器 |

---

## 测试速率限制

```python
import httpx

async def test_rate_limit():
    async with httpx.AsyncClient(app=app, base_url="http://test") as client:
        for i in range(6):
            response = await client.get("/api/data")
            print(f"Request {i+1}: {response.status_code}")
            # 前 5 次应为 200，第 6 次应为 429
```

**建议：** 大多数场景使用 `slowapi`，它最流行，也最容易实现。

## 启用 Ponytail，10 行代码

```python
from fastapi import FastAPI, HTTPException
from slowapi import Limiter
from slowapi.util import get_remote_address

app = FastAPI()
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

@app.get("/api/endpoint")
@limiter.limit("10/minute")
async def my_endpoint(request):
    return {"status": "ok"}
```

省掉了：自定义限流逻辑、Redis、滑动窗口，`slowapi` 会处理这些。什么时候再加：你需要跨多台服务器的分布式限流（把 `Limiter` 换成 Redis 后端），或者需要按用户限流（添加 `key_func=lambda r: r.headers.get("authorization")`）。

**128 → 10 行代码**，同一个模型，同一个提示。
