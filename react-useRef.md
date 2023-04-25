# useRef

`useRef`是一个可以让你指定一个不需要渲染的值

```react
const ref = useRef(initialValue)
```



**参考**

* `useRef(initialValue)`

**使用**

* 用ref来指定一个值
* 用ref操作DOM
* 避免重复创建一个ref内容

**排错**

* 我无法获取到一个自定义组件通过ref





## 参考

`useRef(initialValue)`

调用`useRef`在组件的顶端来声明一个ref

```react
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

**参数**

* `initialValue`:  传入你想让ref对象中`current`属性的初始值, 这个初始值可以是任何类型,  在初次渲染之后这个参数将被忽略

**返回值**

`useRef`通过一个单独的属性来返回一个对象

* `current`:  初次使用时,它被设定为`initialValue`, 在之后你可以重新设定.  如果你将 ref 对象作为 ref 属性传递给 React 到 JSX 节点,那么react会将它设置为`current`属性

在下一次渲染时,`useRef`将会返回相同的属性

**警告**

* 你可以让`ref.current`属性发生突变,  不像state, `ref.current`是可以突变的.  然鹅, 如果属性的值是一个用于渲染的对象(例如, 一块state), 那么你不能让这个对象突变

* 当你改变`ref.current`属性时,  React不会重绘你的组件. 

  React不会意识到你修改了它,  因为ref被解释为一个JavaScript对象.

* 不要再渲染时读取`ref.current`,  除了在初始化时.   这会让你的组件行为变得无法预测

* 在严格模式下,react会调用你的组件两次以帮助你寻找无意间的错误.  这是开发阶段才有的,不会影响到产品.

  每个ref对象也会被创建两次,但是其中之一会被忽略.如果你的组件是一个纯函数, 那么这么做不会影响到行为的改变

## **使用**

### 用ref指定一个值

在组件顶部调用`useRef`来声明一个ref

```react
import { useRef } from 'react';

function Stopwatch() {
  const intervalRef = useRef(0);
  // ...
```

`useRef`会返回一个对象,一个带有current属性的对象,current的值是你传入的初始值

在下一次渲染时,`useRef`将返回同一个对象.  你可以修改current属性来存储信息并可以读取.  这个可能会想到state,  但是他们之间很不同

修改ref不会触发重绘,  这意味着,ref擅长存储那些不会影响到组件的视觉输出的信息.  例如,  如果你需要存储一个interval ID, 并且待会儿检索它.  你可以将其放入到ref.  你需要手动修改current属性的值来更新ref内的值.

```react
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

之后,你可以通过ref的current属性来获取intervalId,这样就可以调用clearInterval来清除ID了

```react
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```



通过使用ref,你需要确定:

* 你可以在重新渲染期间存储信息(不像普通变量, 在每次渲染之后重置)
* 修改ref不会触发重绘(不像state,修改state会触发重绘)
* 信息会在你的组件本地中复制一份,(不像外部变量,是分享的)

修改ref不会触发重绘,所以ref不适合存储你要渲染到屏幕上的信息.  用state来代替它.  

---



**陷阱**
**不要在渲染期间读或者写ref.current**

react期待你的组件体的行为像一个纯函数

* 如果输入相同(props,state,context),那么应该返回相同的JSX
* 调用不同的命令并携带不同的参数不应该影响到其他调用函数的结果

而如果在渲染期间读取或者写入ref就会破坏这种期许

```react
function MyComponent() {
  // ...
  // 🚩 Don't write a ref during rendering
  myRef.current = 123;
  // ...
  // 🚩 Don't read a ref during rendering
  return <h1>{myOtherRef.current}</h1>;
}
```

你可以在事件处理函数中读取或者写入ref

```react
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ You can read or write refs in effects
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ You can read or write refs in event handlers
    doSomething(myOtherRef.current);
  }
  // ...
}
```

如果你必须要在渲染期间读取或者写入,那么用state来代替

如果你不按照规则做,那么你的组件也许依旧可以工作,但是大多数新特性都会依赖这种期许

---



### **使用ref操作dom**

使用ref的主要原因是使用它操作DOM,   react内部支持这个特性.  首先,声明一个ref.object,并让这个obj是一个null

```react
import { useRef } from 'react';

function MyComponent() {
  const inputRef = useRef(null);
  // ...

```

然后将ref object作为一个ref属性值设置给你想要操作的JSX

```react
return <input ref={inputRef} />;
```

在react将DOM节点创建好并且在屏幕上渲染之后,react将会把ref的current属性的值设置为那个DOM节点.  现在你可以获取那个`input`的权限了,并且可以通过fouce方法来使用它

```react
 function handleClick() {
    inputRef.current.focus();
  }
```

当你的DOM节点从屏幕上移除时,react将会将current从新设置为null

### **避免重复创建ref context**

react会保存初始的ref值并且在后续渲染时忽略这个初始值

```react
function Video() {
  const playerRef = useRef(new VideoPlayer());
  // ...
```

尽管`new VideoPLayer`只是在初次渲染时会被使用, 但你仍然在每次渲染时调用这个函数, 这会导致重复创建对象而浪费

为了解决这个问题,你可以像这样创建ref

```react
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```



通常,在渲染时读取或者设置ref是不被允许的,  然鹅 在这种情况下是可以的因为结果总是相同的. 并且条件语句只会在初始化时执行,所以它完全可以预测



---

**如何避免在初始化useRef后做null值检查**

如果不想让你的类型检查器总是检查null,你可以尝试下面的模板

```react
function Video() {
  const playerRef = useRef(null);

  function getPlayer() {
    if (playerRef.current !== null) {
      return playerRef.current;
    }
    const player = new VideoPlayer();
    playerRef.current = player;
    return player;
  }

  // ...
```



在这里,`playerRef`自己是一个null.  然鹅,你应该能够设置你的类型检查器以至于不会出现`getPlayer`返回null的情况

然后将`getPlayer`放入事件处理函数中

---

## **排错**

### **我用ref无法得到一个自定义组件**

当你尝试获取自定义组件时可能会出现错误

```react
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

```jsx
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

默认情况下，你的自定义组件不会暴露ref给他们内部的DOM 节点

为了解决这个问题，你可以像下面这样做

```react
export default function MyInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}
```

然后用`forwardRef`捕获它

```react
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```











