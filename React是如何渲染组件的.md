# React 是如何渲染组件的

 `React 18` 废弃了之前的 `ReactDOM.render()` 函数，改为 `createRoot` 函数。在本文中，我们将:

1. 展示一个 demo

2. 跟踪`createRoot`的执行过程，看看它具体做了些什么工作

## Demo 展示

```jsx
import { createRoot } from 'react-dom/client'
const Element = () => {
  return (
    <h1>
      Hello, World!
    </h1>
  );
}

const root = createRoot(document.getElementById('root'));
root.render(<Element />);
```

因为`jsx` 文件需要编译，我们使用`babel`来看下编译后的样子

很容易操作，只需要将上面的代码复制粘贴到 [Babel 的 playground](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.21&spec=false&loose=false&code_lz=JYWwDg9gTgLgBAbzgYygUwIYzQJQheAXzgDMoIQ4BydDZGAWgBMKB6ZAG2DQDsYqAUMgg8AzvACiHNCF7wAvHAAUASjjyAfIgFw46GAFcoPZTt1wAPAAsAjBrPm4ACTQcOEADRwA6tA5MAQgdLVlt7XRUAbgFCASERcT18BRRabDwCJRZkA1k-ADoAczQYKRk5ACEATwBJJiUaZKoVKIFyAnz0HiY0KCULMrz4Vg1WoA&debug=false&forceAllTransforms=false&modules=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.21.9&externalPlugins=&assumptions=%7B%7D) 就可以在右侧看到变以后的代码了

```js
import { createRoot } from 'react-dom/client';
import { jsx as _jsx } from "react/jsx-runtime";
const Element = () => {
  return _jsx("h1", {
    children: "Hello, World!"
  });
};
const root = createRoot(document.getElementById('root'));
root.render(_jsx(Element, {}));
```

OK, 现在是熟悉的原生`js` 代码，我们可以开始看看 ``createRoot`到底做了些什么了

> 可以看到 jsx 被编译为递归调用的 _jsx 函数。在 React 17 以前，jsx 代码会被编译为 React.createElement()，v17 可以通过设置babel 配置决定是否使用旧的 createElement。[查看更多](https://legacy.reactjs.org/blog/2020/09/22/introducing-the-new-jsx-transform.html)

## 跟踪 createRoot

> 这个过程里，我会关注核心逻辑，忽略某些边界判断和内部标记
> 
> 如果遇到没有核心逻辑的包装函数，我将使用 fn->fn 

在调用真正的`createRoot` 函数之前，会经过两个包装函数:

1. `packages/react-dom/client.js` 

这个`createRoot` 在开发模式下(`__DEV__`)会设置一个标志来标记入口点

2. `packages/react-dom/src/client/ReactDOM.js`

这里的`createRoot` 会检查模块是否是从客户端入口点导入的，如果不是，则打印一个警告

OK，现在我们来到了真正实现函数`packages/react-dom/src/client/ReactDOMRoot.js`

```js
export function createRoot(
  container: Element | Document | DocumentFragment,
  options?: CreateRootOptions,
): RootType {
  // 校验container是否是合法的 DOM 元素
  if (!isValidContainer(container)) {...}

  // 检查容器是否为 BODY 元素或已被标记为 React 容器，如果是，函数会给出相应的警告，如果是则打印警告
  warnIfReactDOMContainerInDEV(container);

  // 给一些标记设置默认值
  let isStrictMode = false;
  let concurrentUpdatesByDefaultOverride = false;
  let identifierPrefix = '';
  let onRecoverableError = defaultOnRecoverableError;
  let transitionCallbacks = null;

  // 如果传入了 options 参数，则使用 options 参数中的值覆盖默认值
  if (options !== null && options !== undefined) {
    // 删去判断逻辑
  }

  // 使用传入的参数创建一个新的容器 FiberRootNode ，这个容器将用于渲染 React 组件树
  const root = createContainer(
    container,
    ConcurrentRoot,
    null,
    isStrictMode,
    concurrentUpdatesByDefaultOverride,
    identifierPrefix,
    onRecoverableError,
    transitionCallbacks,
  );

  // 将创建的容器标记为根容器
  markContainerAsRoot(root.current, container);

  // 获取根容器的 DOM 元素
  const rootContainerElement: Document | Element | DocumentFragment =
    container.nodeType === COMMENT_NODE
      ? (container.parentNode: any)
      : container;
  
  // 绑定所有支持的事件
  listenToAllSupportedEvents(rootContainerElement);

  return new ReactDOMRoot(root);
}
```

其中 `createContainer` 创建了一个容器对象 `FiberRootNode`
