# Debounce

**任务：** “给一个原生 JavaScript 搜索输入添加 debounce。它现在每次按键都会触发一次 API 调用。”

以下内容源自一次 benchmark 运行的模型输出，Claude Haiku 4.5，未启用 skill 组 vs ponytail 组，temperature 1，来源 `benchmarks/output.json`。复现：`npx promptfoo@latest eval -c benchmarks/promptfooconfig.yaml`。

## 未启用 Ponytail，116 行代码

# Debounce 搜索输入

下面是一个防止过量 API 调用的实用实现：

## 基础 Debounce 函数

```javascript
function debounce(func, delay) {
  let timeoutId;
  return function (...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func(...args), delay);
  };
}

// 使用方式
const searchInput = document.getElementById('search');
const resultsContainer = document.getElementById('results');

const handleSearch = async (query) => {
  if (!query.trim()) {
    resultsContainer.innerHTML = '';
    return;
  }
  
  try {
    const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
    const data = await response.json();
    displayResults(data);
  } catch (error) {
    console.error('Search failed:', error);
  }
};

// 对搜索函数做 debounce（300ms 延迟）
const debouncedSearch = debounce(handleSearch, 300);

searchInput.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});
```

## 带加载状态的增强版本

```javascript
const searchInput = document.getElementById('search');
const resultsContainer = document.getElementById('results');
const loadingIndicator = document.getElementById('loading');

const handleSearch = async (query) => {
  if (!query.trim()) {
    resultsContainer.innerHTML = '';
    loadingIndicator.style.display = 'none';
    return;
  }
  
  loadingIndicator.style.display = 'block';
  
  try {
    const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
    if (!response.ok) throw new Error('Search failed');
    
    const data = await response.json();
    displayResults(data);
  } catch (error) {
    resultsContainer.innerHTML = '<p class="error">Search failed. Try again.</p>';
    console.error('Search error:', error);
  } finally {
    loadingIndicator.style.display = 'none';
  }
};

const debouncedSearch = debounce(handleSearch, 300);

searchInput.addEventListener('input', (e) => {
  debouncedSearch(e.target.value);
});

function displayResults(data) {
  resultsContainer.innerHTML = data
    .map(item => `<div class="result">${item.title}</div>`)
    .join('');
}
```

## 高级版：带取消和立即执行选项的 Debounce

```javascript
function debounce(func, delay, options = {}) {
  let timeoutId;
  let lastArgs;
  
  const debounced = function (...args) {
    lastArgs = args;
    clearTimeout(timeoutId);
    
    if (options.immediate && !timeoutId) {
      func(...args);
    }
    
    timeoutId = setTimeout(() => {
      if (!options.immediate) {
        func(...lastArgs);
      }
      timeoutId = null;
    }, delay);
  };
  
  // 允许手动取消
  debounced.cancel = () => clearTimeout(timeoutId);
  
  return debounced;
}

// 使用选项
const debouncedSearch = debounce(handleSearch, 300, { immediate: false });

// 需要时取消待执行搜索
searchInput.addEventListener('blur', () => {
  debouncedSearch.cancel();
});
```

## HTML 示例

```html
<div class="search-container">
  <input 
    type="text" 
    id="search" 
    placeholder="Search..."
    autocomplete="off"
  >
  <div id="loading" style="display: none;">Loading...</div>
  <div id="results"></div>
</div>

<style>
  .search-container {
    max-width: 500px;
    margin: 20px auto;
  }
  
  #search {
    width: 100%;
    padding: 10px;
    font-size: 16px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }
  
  #results {
    margin-top: 10px;
  }
  
  .result {
    padding: 10px;
    border-bottom: 1px solid #eee;
    cursor: pointer;
  }
  
  .result:hover {
    background-color: #f5f5f5;
  }
  
  .error {
    color: red;
    padding: 10px;
  }
</style>
```

## 主要收益

| 功能 | 收益 |
|---------|---------|
| **Debounce** | 把每次搜索 100+ 次 API 调用降到 1-2 次 |
| **300ms 延迟** | 在响应速度和性能之间取得平衡 |
| **加载状态** | 给用户反馈 |
| **错误处理** | 优雅处理失败 |
| **取消方法** | 需要时清理 |

**典型按键减少量：** 10 个字符 = 10 次 API 调用 → debounce 后 1-2 次 API 调用

## 启用 Ponytail，10 行代码

```javascript
const searchInput = document.querySelector('input[type="search"]');
let debounceTimer;

searchInput.addEventListener('input', (e) => {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(() => {
    fetch(`/api/search?q=${encodeURIComponent(e.target.value)}`)
      .then(r => r.json())
      .then(data => console.log(data)); // 替换成你的渲染逻辑
  }, 300);
});
```

省掉了：debounce 工具函数、class wrapper、配置对象。`setTimeout` + `clearTimeout` 就是 debounce。等你需要在 3 个以上输入框复用时，再加工具函数。

**116 → 10 行代码**，同一个模型，同一个提示。
