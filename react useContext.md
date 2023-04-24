# useContext

useContext是一个可以从你的组件中读取和申请内容的react hook函数

```react
const value = useContext(SomeContext)
```

* 参考
  * useContext（someContext）
* 用法
  * 在树内深度传送数据
  * 通过传递内容（context）更新数据
  * 设定回调函数默认值
  * 在树的一部分中最主要的内容
  * 当传递对象和函数时优化重绘
* 排错
  * 我的组件从provider那里看不到值了
  * 尽管值不同了，但是我一直在我的内容中获取undefined



## 参考

通过调用useContext在组件的顶端来读取和获得context

```react
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
```

**参数**

`SomeContext`: 参数是之前通过createContext创建的。context本身不包含任何信息，它仅代表一种你能从哪个组件中提供和获取信息

**返回值**

useContext会从对应的组件中获得value。他的值由组件树中最近的由 `SomeContext.Provider`包裹的组件传递。如果没有相应的provider，那么返回值就会是 `defaultValue` 。它的返回值总是实时更新。react会自动重绘如果组件的内容发生修改

**警告**

* `useContext()`函数的呼叫不会被来自相同组件的providers的返回值影响。相应的`<Context.Provider>`需要在调用`useContext()`函数的组件之上(也就是包围组件)
* react会自动重绘所有接收一个不同值的子元素. 之前的值和之后的值都会用`Object.is()`进行比较.  使用memo进行重绘不会阻止子元素接收新的context值
* 如果你在output中建立一些系统副本模块(那些能导致符号链接的).,这会导致破坏context.     通过context传来的一些东西必须在用于传值的`someContext`与用于读取值的`someContext`是同一个对象时才能使用.. 



## 使用

#### 在组件树中深度传值

同样的在组件顶端获取到`useContext`

```react
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

`useContext`将会返回一个context,这个context待会儿会传给组件.  传给哪个组件就会获取哪个组件的值. react会搜索组件树并寻找最近的对应的context provider 

下面这个例子就是想给一个`button`传递信息,将想传递信息的组件或者其父组件用对应的provider包裹

```react
function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... renders buttons inside ...
}
```

在provider和`button`之间存在有多少层组件并不重要. 当表单中任意位置的按钮调用useContext（ThemeContext）时，它将接收“dark”作为值。

---

**陷阱**

`useContext`总是寻找在组件上层最近的provider并且调用它.它向上寻找但是不会考虑自己设置的provider

```react
import React, { useState, createContext, useContext } from "react";
const themeContext = createContext();

export default function App() {
  const [theme, setTheme] = useState("black");
  const { newTheme, setNewTheme } = useContext(themeContext);
  return (
    <div>
      <themeContext.Provider value={{ theme, setTheme }}>
        <button
          onClick={() =>
            theme === "black" ? setNewTheme("white") : setNewTheme("black")
          }
        >
          theme is: {newTheme}
        </button>
      </themeContext.Provider>
    </div>
  );
}
```

useState会找调用它的组价之上的provider,不会"自导自演"

---



#### 通过context更新数据

通常,你会希望context值改变.  通过结合state来更新context可以刷新页面. 

```react
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
        Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

如今，所有provider内的button都会接收当前的theme值。  如果你调用setTheme来更新theme值，那么这个值会传给provider，因此所有的button组件将会重新渲染新的值.



#### 具体化一个fallback默认值

如果react在上层组件树中并没有找到任何相关context的provider. 那么调用`useContext`将等于默认值,这个默认值是你创建那个context时设置的

```react
const ThemeContext = createContext(null);
```

默认值永不改变,这里有一些有意义的默认值,例如

```react
const ThemeContext = createContext('light');
```

用这种方法,  如果在渲染某个组件时无意间没有设置provider,那么也不会报错.  这样也会让你的组件在测试环境下更好的工作且可用在测试时不需要设置大量的provider

在下面的例子中, "toggle theme"这个按钮将总是light 因为

这个按钮在theme context provider的外部并且默认context值是'light'.  试着编辑默认主题为 `'dark'`。

```react
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <>
      <ThemeContext.Provider value={theme}>
        <Form />
      </ThemeContext.Provider>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        Toggle theme
      </Button>
    </>
  )
}

function Form({ children }) {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}

```

#### 覆盖组件树一部分的 context 

通过在 provider 中使用不同的值包装树的某个部分，可以覆盖该部分的 context。

```react
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

你可以根据需要多次嵌套和覆盖 provider。

#### 在传递对象和函数时优化重新渲染 

你可以通过 context 传递任何值，包括对象和函数。

```react
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

此处，context value 是一个具有两个属性的 JavaScript 对象，其中一个是函数。每当 `MyApp` 出现重新渲染（例如，路由更新）时，这里将会是一个 **不同的** 对象指向 **不同的** 函数，因此 React 还必须重新渲染树中调用 `useContext(AuthContext)` 的所有组件。

在较小的应用程序中，这不是问题。但是，如果基础数据如 `currentUser` 没有更改，则不需要重新渲染它们。为了帮助 React 利用这一点，你可以使用 [`useCallback`](https://react.docschina.org/reference/react/useCallback) 包装 `login` 函数，并将对象创建包装到 [`useMemo`](https://react.docschina.org/reference/react/useMemo) 中。这是一个性能优化的例子：

```react
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

根据以上改变，即使 `MyApp` 需要重新渲染，调用 `useContext(AuthContext)` 的组件也不需要重新渲染，除非 `currentUser` 发生了变化。

阅读更多关于 [`useMemo`](https://react.docschina.org/reference/react/useMemo#skipping-re-rendering-of-components) 和 [`useCallback`](https://react.docschina.org/reference/react/useCallback#skipping-re-rendering-of-components) 的内容。

## 疑难解答 

### 我的组件获取不到 provider 传递的值

这里有几种常见的情况会引起这个问题：

1. 你在调用 `useContext()` 的同一组件（或下层）渲染 `<SomeContext.Provider>`。把 `<SomeContext.Provider>` 向调用 `useContext()` 组件 **之上和之外** 移动。
2. 你可能忘记了使用 `<SomeContext.Provider>` 包装组件，或者你可能将组件放在树的不同部分。使用 [React DevTools](https://react.docschina.org/learn/react-developer-tools) 检查组件树的层级是否正确。
3. 你的工具可能会遇到一些构建问题，导致你在传值组件中的所看到的 `SomeContext` 和读值组件中所看到的 `SomeContext` 是两个不同的对象。例如，如果使用符号链接，就会发生这种情况。你可以通过将它们赋值给全局对象如 `window.SomeContext1` 和 `window.SomeContext2` 来验证这种情况。然后在控制台检查 `window.SomeContext1 === window.SomeContext2` 是否相等。如果它们是不相等的，就在构建工具层面修复这个问题。

### 尽管设置了不一样的默认值，但是我总是从 context 中得到 `undefined` 

你可能在组件树中有一个没有设置 `value` 的 provider：

```react
// 🚩 不起作用：没有 value 作为 prop
<ThemeContext.Provider>
   <Button />
</ThemeContext.Provider>
```

如果你忘记了指定 `value`，它会像这样传值 `value={undefined}`。

你可能还错误地使用了一个不同的 prop 名：

```react
// 🚩 不起作用：prop 应该是“value”
<ThemeContext.Provider theme={theme}>
   <Button />
</ThemeContext.Provider>
```

在这两种情况下，你都应该在控制台中看到 React 发出的警告。要解决这些问题，使用 `value` 作为 prop：

```react
// ✅ 传递 value 作为 prop
<ThemeContext.Provider value={theme}>
   <Button />
</ThemeContext.Provider>
```

注意，只有在 **上层根本没有匹配的 provider** 时才使用 [`createContext(defaultValue)`调用的默认值](https://react.docschina.org/reference/react/useContext#specifying-a-fallback-default-value)。如果存在 `<SomeContext.Provider value={undefined}>` 组件在父树的某个位置，调用 `useContext(SomeContext)` 的组件 **将会** 接收到 `undefined` 作为 context 的值。













