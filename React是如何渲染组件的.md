# React 是如何渲染组件的

 `React 18` 废弃了之前的 `ReactDOM.render()` 函数。，改为 `createRoot` 函数。在本文中，我们将:

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

首先找到它的目录，它从 `react-dom/client` 引入，直接打开 `client.js`

```js
import {
  createRoot as createRootImpl,
  hydrateRoot as hydrateRootImpl,
  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED as Internals,
} from './';

export function createRoot(
  container: Element | Document | DocumentFragment,
  options?: CreateRootOptions,
): RootType {
  if (__DEV__) {
    Internals.usingClientEntryPoint = true;
  }
  try {
    return createRootImpl(container, options);
  } finally {
    if (__DEV__) {
      Internals.usingClientEntryPoint = false;
    }
  }
}
```
