---
layout: ../../../layouts/ArticleLayout.astro
title: React useMemo 详解：原理、误区与性能优化实战
description: 全面理解 useMemo 的缓存机制、依赖管理、典型场景与滥用风险。
category: frontend
pubDate: 2026-04-10
---

# React useMemo 详解：原理、误区与性能优化实战

很多人把 useMemo 当成“性能万能药”，结果代码更复杂、收益却不明显。正确理解 useMemo，关键在于先搞清楚它解决的到底是什么问题。

## 1. useMemo 是做什么的

useMemo 用来缓存“计算结果”，在依赖不变时复用上一次结果，避免重复计算。

基础写法：

```tsx
const memoizedValue = useMemo(() => compute(a, b), [a, b]);
```

可以把它理解为：

- 首次渲染：执行 compute，保存结果。
- 后续渲染：若依赖数组不变，直接返回缓存结果。
- 依赖变化：重新计算并更新缓存。

## 2. useMemo 不能做什么

常见误解：

- 它不会阻止组件重新渲染。
- 它不会自动让页面变快。
- 它不是副作用工具，不能替代 useEffect。

如果组件频繁重渲染是因为父组件状态变化，useMemo 只缓存值，不会拦截渲染流程。

## 3. 什么时候该用 useMemo

适合使用的场景：

- 计算成本高：如大数组排序、复杂过滤、聚合统计。
- 需要稳定引用：传给 React.memo 子组件的对象或数组 props。
- 第三方库依赖对象引用稳定：避免无意义重算或重复订阅。

不建议使用的场景：

- 计算非常轻量（字符串拼接、简单判断）。
- 仅仅为了“看起来更专业”。
- 依赖项经常变化，缓存命中率很低。

## 4. 场景一：缓存高开销计算

```tsx
const filtered = useMemo(() => {
  return products
    .filter((item) => item.price >= minPrice)
    .filter((item) => item.name.includes(keyword))
    .sort((a, b) => b.score - a.score);
}, [products, minPrice, keyword]);
```

这类场景里，useMemo 的价值在于减少重复的 filter 和 sort。

## 5. 场景二：给子组件稳定 props

假设子组件用了 React.memo：

```tsx
const Toolbar = React.memo(function Toolbar({ options }) {
  return <div>{options.theme}</div>;
});
```

父组件如果每次都创建新对象，会导致子组件仍然重渲染：

```tsx
const options = { theme, dense: true };
```

改为：

```tsx
const options = useMemo(() => ({ theme, dense: true }), [theme]);
```

这样在 theme 不变时，options 引用也不变，React.memo 才能发挥作用。

## 6. 依赖数组的常见坑

### 6.1 少写依赖导致脏数据

```tsx
const value = useMemo(() => expensiveCompute(a, b), [a]);
```

如果遗漏了 b，就可能得到过期结果。

### 6.2 依赖本身每次都新建

```tsx
const query = { keyword, page };
const result = useMemo(() => search(data, query), [data, query]);
```

query 每次渲染都是新对象，缓存会失效。正确做法：

```tsx
const query = useMemo(() => ({ keyword, page }), [keyword, page]);
const result = useMemo(() => search(data, query), [data, query]);
```

## 7. useMemo、useCallback、React.memo 的关系

三者经常配合使用：

- useMemo：缓存值。
- useCallback：缓存函数。
- React.memo：缓存组件渲染结果（基于 props 浅比较）。

一个常见组合：

1. 父组件用 useMemo 准备稳定对象 props。
2. 父组件用 useCallback 准备稳定回调 props。
3. 子组件用 React.memo 避免无意义重渲染。

## 8. 用数据证明优化有效

不要靠“感觉快了”，建议用 React DevTools Profiler 验证：

- 优化前记录一次交互的渲染耗时。
- 加入 useMemo 后再记录一次。
- 对比 commit 时间和重渲染次数。

如果收益几乎没有，优先保持代码简单。

## 9. 实战建议清单

你可以按下面的顺序判断：

1. 这段计算是否真的重。
2. 依赖是否稳定可控。
3. 缓存命中率是否足够高。
4. 是否会增加理解成本。
5. 是否通过 Profiler 证明确有收益。

满足 1 到 3，再考虑 4 和 5，通常是比较稳妥的策略。

## 总结

useMemo 的本质是“用内存换计算”，不是“自动性能优化开关”。

正确姿势是：

- 在高成本计算和引用稳定需求下使用。
- 严格维护依赖数组。
- 用 Profiler 做证据驱动优化。

把 useMemo 用在真正有价值的地方，React 应用会更快，也更可维护。