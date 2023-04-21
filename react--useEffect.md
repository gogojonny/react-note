# useEffect

`useEffect`是一个react hook 帮助你用外部系统来同步组件

```react
useEffect(setup, dependencies?)
```

## 索引

**参考**

* `useEffect(setup, dependencies?)`

**使用**

* 连接外部系统
* 在自定义hook中包装Effects
* 控制一个非react工具
* 用Effects捕获数据
* 指定react依赖
* 从Effect中基于上一次state的值更新state
* 去除不必要的对象依赖
* 去除不必要的函数依赖
* 从Effect中读取最新的props和state
* 在服务端与客户端展示不同的内容

**排错**

* 当组件挂载时我的Effect执行了两次
* 我的Effect每次渲染完都会执行
* 我的Effect反复运行进入死循环了
* 即使我的组件并没有卸载我的清除逻辑还是执行了
* 我的Effect表面上执行了某些东西,并且我看到在它执行前闪了一下



## 参考

``useEffect(setup, dependencies?)``

在组件顶部执行`useEffect`来声明一个Effect

```react
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```



**参数**

* `setup`: 一个Effect的逻辑函数, 你设置的函数也许会返回一个cleanup函数.  当你的组件第一次加到DOM中时,  react将会执行你设置的setup函数. 在之后每一次用changed 依赖重绘 ,react将会首先使用旧值返回cleanup函数. 然后用新值执行setup函数.  当你的组件从DOM中移除时,react将最后一次执行你提供的setup函数.

* 可选的`dependencies`:  数组中所有的reactive值引用着内部的setup code.    reactive 值是指props,state,以及所有直接定义在组件函数内部的变量和函数.  如果你的linter(代码检查器)是react配置的,  那linter将会验证所有的作为依赖的reactive值是否指明.    所有的数组依赖值必须有恒定的依赖数量并且被写成`[dep1, dep2, dep3]` .
  react会将每一个依赖于上一次的值通过`Object.is`进行比较.   如果你省去这个参数,  那么你的Effect将会在每一次重绘之后都执行一次. 
  **下面是比较传入一个数组时,传入空数组时以及忽略这个参数时Effect行为的不同**

  * **传入一个"依赖"数组**
    如果你指明了依赖值,那么你的Effect将会在初始化时执行,也会在依赖值变化时执行

    ```react
    useEffect(() => {
      // ...
    }, [a, b]); // Runs again if a or b are different
    ```

    

  * **传入一个空的依赖数组**

    ```react
    useEffect(() => {
      // ...
    }, []); // Does not run again (except once in development)
    ```

    即使传入一个空的依赖数组,setup和clearup函数也会额外执行一次(仅在开发阶段) 来帮助你找bug

  * **忽略第二个参数**
    如果你`useEffect`第二个参数什么都不传,那么你的Effect将在每一次渲染后都会执行

    ```react
    useEffect(() => {
      // ...
    }); // Always runs again
    ```

    需要注意的是,由于每一次重绘都会执行Effect,所以这个Effect是很敏感的,当有与setup函数中不相干的reactive value触发重绘时,也会导致Effect执行. 这是不可取的.  这也是为什么通常你应该传入依赖数组



**返回值**

`useEffect`没有返回值

**警告**

* `useEffect`是一个hook ,所以你只能在组件的上一级呼叫.或者在你的自定义hook中呼叫.  你不能在条件语句或者循环中呼叫. 如果你需要这样做, 提取一个新的组件并且将state移入依赖中

* 如果你不需要尝试去同步外部系统,你可能不需要Effect

* 在严格模式下, react将会在第一次真正设置之前额外执行setup和cleanup函数.  这是一个压力测试,目的是确保你的cleanup函数对照着你的setup函数,并且能够停止或撤销setup函数正在做的事情. 如果这导致了一些问题, 青实现一个cleanup函数

  * **如何处理在开发模式下Effect执行两次的问题**

    react故意重新挂载你的组件在开发阶段,为的是寻找bug.  真正的问题并不是"如何让Effect只执行一次"而是"如何修改的的Effect使其在重复挂载后依然能正常工作"

    通常, 处理办法就是设置一个cleanup函数, cleanup函数应该停止或者撤销setup函数做的事情, 首要规则是用户应该没法查觉Effect只执行一次和Effect经理setup->cleanup->setup的流程

* 如果你其中某个依赖是一个对象或者函数. 那么可能会导致一个风险, 即这些对象或者函数将导致Effect多执行了好几次,  可以通过移除不必要的对象或者函数依赖来解决这个问题.  你也可以在Effect外部将函数提取为state或者non-reactive逻辑来解决这个问题.

  * **基于从Effect中拿到的上一个state来更新state**
    当你想基于上一次state来更新这次的state,你可能会遇到这样的问题

    ```react
    function Counter() {
      const [count, setCount] = useState(0);
    
      useEffect(() => {
        const intervalId = setInterval(() => {
          setCount(count + 1); // You want to increment the counter every second...
        }, 1000)
        return () => clearInterval(intervalId);
      }, [count]); // 🚩 ... but specifying `count` as a dependency always resets the interval.
      // ...
    }
    ```

    自`count`是一个reactive value,那必须指定为依赖数组中的一员.  燃鹅,这将导致每一次`count`发生变化时, Effect将执行setup和cleanup, 这并不是我们想要的结果

    可以用下面这样的方式进行修改,传入`c=>c+1`这个updater,并让依赖指定为空数组

    ```react
      const [count, setCount] = useState(0);
    
      useEffect(() => {
        const intervalId = setInterval(() => {
          setCount(c => c + 1); // ✅ Pass a state updater
        }, 1000);
        return () => clearInterval(intervalId);
      }, []); // ✅ Now count is not a dependency
    
      return <h1>{count}</h1>;
    }
    ```

    

    现在,你传入`c=>c+1`来代替`count+1`,你的Effect将不再依赖`count`,这时Effect将不再需要每次`count`发生变化时cleanup和setup了

    **移除不必要的对象依赖**

    如果你的Effect在执行时需要以一个对象或者函数作为依赖.   那么这可能导致Effect执行的过于频繁.  例如, 在下面这个案例中,Effect在每一次因`options`对象渲染之后会进行重连, 

    ```react
    const serverUrl = 'https://localhost:1234';
    
    function ChatRoom({ roomId }) {
      const [message, setMessage] = useState('');
    
      const options = { // 🚩 This object is created from scratch on every re-render
        serverUrl: serverUrl,
        roomId: roomId
      };
    
      useEffect(() => {
        const connection = createConnection(options); // It's used inside the Effect
        connection.connect();
        return () => connection.disconnect();
      }, [options]); // 🚩 As a result, these dependencies are always different on a re-render
      // ...
    ```

    这样做有一个隐藏的问题,那就是options这个对象在组件中定义着,  每当组件更新时,相应的options的地址也会更新,这个是独属于对象和函数的问题.  每当组件更新时,对象地址也会更新,从而影响Effect,  让其莫名其妙的运行.  所以应该避免在Effect的依赖中传入对象或者函数,相应的, 可以将对象或者函数定义在组件之外或者定义在Effect之内.  

  * 从Effect中获取最新的props和state
    由于该API仍处于实验阶段，所以暂时不用考虑

* 如果你的Effect不是**由交互事件触发的**（例如点击），React会让浏览器先去渲染屏幕，在去执行Effect。如果你的Effect处理一些视觉上的事情（例如展示工具栏），并且延迟很明显，用`useLayoutEffect`来代替`useEffect`

* 即使你的Effect是被交互行为触发的（例如点击），浏览器也会在Effect内的state更新之前重绘屏幕。通常，那才是你需要的。然鹅，如果你必须阻止浏览器重绘屏幕，那么可以使用`useLayoutEffect`。

* Effect只会在客户端执行，而不会在服务端渲染时执行。

  * 什么是浏览器端渲染？

    CSR是Client Server Render的简称。页面上的内容使我们通过加载js文件渲染出来的，js文件运行在浏览器上，服务端只返回一个html模板

    ![img](https://pic3.zhimg.com/v2-38d189b470279c5ac795b259930e3762_r.jpg)

    

  * 什么是服务端渲染？
    SSR是Server Side Render简称，页面上的内容是通过服务端渲染生成的，浏览器直接显示服务端返回的html就可以了

    ![img](https://pic1.zhimg.com/v2-3ee48624b904cbb1bf0ff34ed2c766b8_r.jpg)




## 使用

### 连接外部系统

一些组件需要保持与网络连接，或者保持连接浏览器API，第三方库，当他们展示到页面上时，这些外来系统不受react控制，所以被称为外部系统

为了将你的组件与外部系统连接，在组件的顶部call一个`useEffect`。

你需要给`useEffect`传入两个参数：

* 一个setup函数用于连接外部系统
  它返回一个cleanup函数来断开与外部系统的连接

* 一个数组依赖
  包含要在setup与cleanup使用的参数，数组依赖的每个元素都要求是Reactive value

在某些情况下，React将会调用setup和cleanup函数，这可能涉及以下方面：

* 当你的组件加入到页面时（mount）会调用setup
* 在你的组件每次重绘后且依赖发生改变时：
  * 首先，你的cleanup code会用旧的props与state运行
  * 然后，你的setup code会用新的props与state运行
* 当你的组件移出页面时，你的cleanup code会运行最后一次

让我们用例子来阐述这些因素

当`chatRoom`组件添加到页面时，它会用初始的`serverUrl`与`roomId`来连接chat room。 如果`serverUrl`与`roomId`之一发生改变，并由此因此重新渲染（或者说用户在下拉单中选中了其他room），你的Effect将会断开与上一个房间的连接转而连接下一个room。当你的`chatRoom`组件从页面中删除时，你的Effect将最后一次执行disconnect方法



为了帮助你寻找bug，在开发阶段React将会在setup函数执行之前执行一次setup与cleanup函数。这是一种压力测试，测试你的Effect逻辑是可以正确的实施。cleanup函数应该停止或者撤销setup函数做的事情。这份规则的重点是用户应该无法区别setup被调用一次还是执行了setup-》cleanup-》setup的过程。

尝试**将Effect写成独立程序**并且**想清楚setup与cleanup一次循环的执行流程**。你不必考虑组件是处于挂载，更新还是卸载状态。当你的cleanup逻辑正确的映射着setup逻辑时，你的Effect将根据需要随时运行setup与cleanup。

* **将Effect写成独立程序**

  将不同的功能分别写到独立的Effect而不相互影响

* **想清楚setup与cleanup一次循环的执行流程**。

  再一次观察中，总是专注于一个单独的start/stop循环，而不是在乎组件处于什么状态。你所需要做的是描述如何开始同步以及如何停止它。如果能做好这件事，那么你的Effect将根据需要随时运行setup与cleanup

---



**Note**

* Effect能使你的组件与外部系统同步。在这里，外部系统意思是任何不由react进行控制的代码，例如
  * `setInterval`与`clearInterval`
  * 事件监听器`addEventListener`与`removeEventListener`
  * 第三方动效库，例如`animation.start()`与`animation.reset()`
* 需要注意的是，如果你不需要连接任何外部系统，那么你也许不需要Effect

---









































