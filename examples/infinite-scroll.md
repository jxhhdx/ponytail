# 无限滚动

**任务：** “当用户滚动到底部时加载更多项目。”

## 未启用 Ponytail

```bash
npm install react-infinite-scroll-component
```

```jsx
import InfiniteScroll from "react-infinite-scroll-component";

export function Feed({ items, fetchMore, hasMore }) {
  return (
    <InfiniteScroll
      dataLength={items.length}
      next={fetchMore}
      hasMore={hasMore}
      loader={<Spinner />}
      endMessage={<p>No more items</p>}
      scrollThreshold={0.9}
    >
      {items.map(item => <Card key={item.id} item={item} />)}
    </InfiniteScroll>
  );
}
```

为了监听滚动位置并触发回调，引入了一个依赖。

## 启用 Ponytail

```jsx
// ponytail: IntersectionObserver 就是做这个的，不需要 scroll listener
import { useEffect, useRef } from "react";

export function Feed({ items, fetchMore, hasMore }) {
  const sentinel = useRef(null);

  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting && hasMore) fetchMore();
    });
    if (sentinel.current) observer.observe(sentinel.current);
    return () => observer.disconnect();
  }, [hasMore, fetchMore]);

  return (
    <>
      {items.map(item => <Card key={item.id} item={item} />)}
      <div ref={sentinel} />
    </>
  );
}
```

**1 个依赖 → 0 个依赖。** `IntersectionObserver` 只在哨兵元素进入视口时触发，没有 scroll 事件，没有节流，也没有卡顿。所有浏览器都支持。那个库包装的正是这个 API。
