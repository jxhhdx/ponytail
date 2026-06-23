# 邮箱校验

**任务：** “给我写一个校验邮箱地址的 Python 函数。”

以下内容源自一次 benchmark 运行的模型输出，Claude Haiku 4.5，未启用 skill 组 vs ponytail 组，temperature 1，来源 `benchmarks/output.json`。复现：`npx promptfoo@latest eval -c benchmarks/promptfooconfig.yaml`。

## 未启用 Ponytail，75 行代码

# 邮箱校验函数

下面是一个实用的 Python 邮箱地址校验函数：

```python
import re

def validate_email(email: str) -> bool:
    """
    使用正则表达式校验邮箱地址。
    
    Args:
        email (str): 要校验的邮箱地址
        
    Returns:
        bool: 有效返回 True，否则返回 False
    """
    # 邮箱校验的正则表达式
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    
    return re.match(pattern, email) is not None


# 测试用例
if __name__ == "__main__":
    test_emails = [
        "user@example.com",           # 有效
        "john.doe@company.co.uk",     # 有效
        "test+tag@domain.org",        # 有效
        "invalid.email@",             # 无效
        "missing@domain",             # 无效
        "@nodomain.com",              # 无效
        "spaces in@email.com",        # 无效
        "user@domain..com",           # 无效（连续点）
    ]
    
    for email in test_emails:
        result = validate_email(email)
        print(f"{email:<30} -> {result}")
```

## 更健壮的版本（带额外检查）

```python
import re

def validate_email_advanced(email: str) -> tuple[bool, str]:
    """
    带详细反馈的高级邮箱校验。
    
    Args:
        email (str): 要校验的邮箱地址
        
    Returns:
        tuple: (是否有效, 消息)
    """
    # 基础检查
    if not email or not isinstance(email, str):
        return False, "Email must be a non-empty string"
    
    email = email.strip()
    
    if len(email) > 254:
        return False, "Email is too long (max 254 characters)"
    
    # 正则表达式
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    
    if not re.match(pattern, email):
        return False, "Email format is invalid"
    
    # 额外检查
    local_part, domain = email.rsplit('@', 1)
    
    if len(local_part) > 64:
        return False, "Local part is too long (max 64 characters)"
    
    if local_part.startswith('.') or local_part.endswith('.'):
        return False, "Local part cannot start or end with a dot"
    
    if '..' in email:
        return False, "Email cannot contain consecutive dots"
    
    return True, "Email is valid"


# 测试高级版本
if __name__ == "__main__":
    test_emails = [
        "user@example.com",
        "invalid..email@domain.com",
        ".invalid@domain.com",
        "user@domain",
    ]
    
    for email in test_emails:
        is_valid, message = validate_email_advanced(email)
        print(f"{email:<35} -> {is_valid:5} ({message})")
```

## 使用第三方库（生产环境推荐）

```python
from email_validator import validate_email, EmailNotValidError

def validate_email_production(email: str) -> bool:
    """
    使用 email_validator 库校验邮箱。
    对生产环境更可靠。
    
    安装命令：pip install email-validator
    """
    try:
        # 校验并规范化邮箱
        valid = validate_email(email)
        return True
    except EmailNotValidError:
        return False


# 使用方式
if __name__ == "__main__":
    print(validate_email_production("user@example.com"))  # True
    print(validate_email_production("invalid@"))          # False
```

## 对比

| 方法 | 优点 | 缺点 |
|--------|------|------|
| **正则（简单）** | 快，无依赖 | 校验有限 |
| **正则（高级）** | 更全面 | 仍不符合 RFC |
| **email-validator** | 符合 RFC，可靠 | 外部依赖 |

**建议**：生产应用使用 `email-validator` 库，因为它能正确处理 RFC 5321/5322 标准。快速脚本或想避免依赖时，使用简单正则版本。

## 启用 Ponytail，3 行代码

```python
import re

def is_valid_email(email: str) -> bool:
    return bool(re.match(r'^[^@]+@[^@]+\.[^@]+$', email))
```

省掉了：RFC 5322 解析器、DNS MX 查询、确认邮件。等你真的需要拒绝 `user+tag@sub.domain.co.uk` 或捕捉拼写错误时再加；在那之前，这能抓住 99% 的“手滑输错”。

**75 → 3 行代码**，同一个模型，同一个提示。
