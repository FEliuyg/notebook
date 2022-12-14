# React 18 更新

## 1. 自动批处理 （Automatic Batching)

在没有批处理之前，react 只能对 react 事件的更新做批处理，像 promise 、setTimout、原生事件等非 react 事件就不会做批处理，就会导致执行多少次更新，就有多少次的渲染。

官方示例：

```jsx
// Before: only React events were batched.
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React will render twice, once for each state update (no batching)
}, 1000);

// After: updates inside of timeouts, promises,
// native event handlers or any other event are batched.
setTimeout(() => {
  setCount((c) => c + 1);
  setFlag((f) => !f);
  // React will only re-render once at the end (that's batching!)
}, 1000);
```

## 2.过渡 （Transitions)

主要用于区别紧急和非紧急的任务更新。
一般输入、点击事件是需要立即响应的，属于紧急更新。

官方示例：

```jsx
import { startTransition } from 'react';

// Urgent: Show what was typed
setInputValue(input);

// Mark any state updates inside as transitions
startTransition(() => {
  // Transition: Show the results
  setSearchQuery(input);
});
```

在 `startTransition` 中的任务为不紧急的更新

对应有个 hook , `useTransition`

官方示例：

```jsx
function App() {
  const [isPending, startTransition] = useTransition();
  const [count, setCount] = useState(0);

  function handleClick() {
    startTransition(() => {
      setCount((c) => c + 1);
    });
  }

  return (
    <div>
      {isPending && <Spinner />}
      <button onClick={handleClick}>{count}</button>
    </div>
  );
}
```

## 3.新的 Suspense 特性

Suspense 用于在组件树还没有渲染完成时，显示一个 loading 的状态。

官方示例:

```jsx
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

在 react18 之前也有 Suspense，不过算是一个功能不完整的版本，不支持服务端渲染，只能用于配合 React.lazy 做代码分割的场景。

## 4.新的客户端和服务端渲染 API

客户端：
**createRoot**: 用原来的 `ReactDOM.render` 将不支持 `React18的新特性`。包的导出由 `react-dom` 变为 `react-dom/client`

服务端：
**hydrateRoot**: 用于服务端渲染的，替换原来的 `ReactDOM.hydrate`。也是一样，没有使用就不能使用 react18 的新特性。

还有两个 api，
**renderToPipeableStream**: 以流的形式用于 node 环境
**renderToReadableStream**: 边缘运行环境，例如 Deno

替换原来的 `renderToString`，虽然还能用，但已不建议用了。

## 5.严格模式

用于检验一些副作用，以及未来的一些新特性。

例如：开发模式下，组件会渲染两次

## 6.新的 hook

- useId: 用于在服务端和客户端生成一个唯一 ID，避免服务端渲染注水不匹配。
- useTransition: 用于非紧急任务的更新
- useDeferredValue: 延迟计算出新值，在紧急任务之后，类似于防抖
- useSyncExternalStore: 外部状态，目标是用于库开发，不是应用
- useInsertionEffect: 用于 css-in-js 库渲染插入样式，目标也是用于库开发，不是应用
