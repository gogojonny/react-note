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

**连接外部系统的例子**

* **连接到chat服务器**
  在这个例子中，`chatRoom`组件使用Effect保持与定义在`chat.js`中定义的外部系统连接.  点击"open chat" 使你的`chatRoom`组件出现.  

* **连接全局对象事件**

  在这个例子中,外部系统是浏览器DOM自己,  通常,你可以使用JSX连接具体的事件.  但是你不能连接到window对象上.  Effect可以帮助你连接到window对象并监听事件.  监听`pointermove`事件让你追踪到鼠标位置并且更新红色的圆点来更新它

  ```react
  export default function App() {
    const [position, setPosition] = useState({ x: 0, y: 0 });
  
    useEffect(() => {
      function handleMove(e) {
        setPosition({ x: e.clientX, y: e.clientY });
      }
      window.addEventListener('pointermove', handleMove);
      return () => {
        window.removeEventListener('pointermove', handleMove);
      };
    }, []);
      //.......
  ```

* **触发动画**
  在这个例子或者,外部系统是动画库. 它提供一个叫做`FadeInAnimation`的JavaScript class.   这个class接收一个DOM node作为参数.并且暴露出`start()`和`stop()`两个方法来控制动画. 这个组件使用ref获取DOM node根节点.  Effect从ref中读取DOM 节点并且当组件出现时会自动在那个节点上播放动画

  ```react
    const ref = useRef(null);
  
    useEffect(() => {
      const animation = new FadeInAnimation(ref.current);
      animation.start(1000);
      return () => {
        animation.stop();
      };
    }, []);
  ```

* **控制模态弹框(modal dialog)**

  在这个案例中,外部系统是浏览器DOM.  `ModalDialog`组件渲染一个`<dialog>`元素,  这个组件使用Effect来同步`isopen`props去调用`showModal`和`close`方法

* **追踪元素可见性**

  再这个案例中，外部系统又是浏览器DOM，APP组件展示了一个长列表，然后是一个`box`组件,  再然后是另一个长列表.  向下滚动列表,  注意到当`box`组件出现在屏幕上时,背景颜色变为黑色.  为了实现这个,  `box`组件使用Effect来管理`IntersectionObserver`.  这个浏览器API

  会通知你当你的DOM元素在视口中是可见的时候



---





### **用自定义hook包装Effect**

Effect是一个紧急出口：当你需要“脱离React”并且对于你的使用场景没有更好的解决方案时使用他们。当你发现你自己经常需要手动写Effect时,  那通常意味着你需要在你的组件中引入一些自定义hook做公共行为.

例如,  `useChatRoom`自定义组件"隐藏"了Effect在背后声明了的的逻辑

自定义hook,封装了Effect的逻辑代码

```react
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

然后你可以像这样在组件中使用

```react
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });
  // ...
```

在react生态里有更多优秀的为了各种目的而设置的自定义hook 

**实例**

* 自定义`useChatRoom`hook
* 自定义`useWindowLIstener`hook
* 自定义``useIntersectionObserver``hook

---

### 控制non-react工具



有时候,你想让外部系统与你组件内的props与state同步

例如，如果你有一个视频播放器组件且这个组件不是React写的， 那么你可以使用Effect调用他们并且这样会让他们的state与你的组件的state匹配。这个Effect创造力一个叫`MapWidget`的class并定义在`map-widget.js`。当你改变`map`组件中的`zoomLevel`prop时，Effect会在class实例上调用`setZoom`来维持同步

在这个案例中，cleanup函数是不需要的，因为`MapWidget`这个class只管理着被传入的DOM节点，在`Map`组件从组件树中移除时，DOM节点与`MapWidget`类的实例将会自动的被JavaScript引擎打包收走



### 用Effect捕获数据

你可以使用Effect从你的组件中捕获数据。 注意，如果你是用框架，使用你框架的数据获取机制要比手动写Effect要高效的多

如果你想手动写Effect来获取数据，那么你的code可能长这样

```react

import { useState, useEffect } from 'react';
import { fetchBio } from './api.js';

export default function Page() {
  const [person, setPerson] = useState('Alice');
  const [bio, setBio] = useState(null);

  useEffect(() => {
    let ignore = false;
    setBio(null);
    fetchBio(person).then(result => {
      if (!ignore) {
        setBio(result);
      }
    });
    return () => {
      ignore = true;
    };
  }, [person]);

  // ...
```

现在`ignore`变量被初始化为`false`,  并且在执行cleanup时会被设置为`true`. 这样会确保你的code不会遭受"race conditions": 网络响应的到达顺序可能与您发送的顺序不同

你也可以使用await/async语法，但是你仍需要提供一个cleanup函数

直接用Effect来获取数据会变得重复并且变得难以优化和渲染。 更好的方式是写一个自定义hook，无论是你自己写的还是来自社区的。

---

**深度探讨**

**对于用Effect获取数据，什么是更好的替代品**

在Effect内部写一个`fetch`是一个获取数据的流行方法。除了完全是客户端的app。然而这个方法是一个非常需要手动并且它有很强的缺点：

* **Effect并不跑在服务端**。这意味着初始时在服务端渲染的HTML将只包含一个loading的状态并且没有数据。客户端电脑上必须下载JavaScript并且渲染你的app时才发现需要加载数据。这明显很低效
* **直接从Effect获取容易造成“网络瀑布”**：当你渲染你的父组件，并且获取到一些data，并渲染你的子组件，然后再获取子组件的data。如果你的网络不够快，这会很明显比平行获取所有数据慢。
* **直接用Effect获取数据通常意味着你不能预加载或者高速缓存数据：**例如，如果你的组件被卸载然后又很快挂载，那么它将不得不再次重新获取数据
* **很不符合人体工程学：**这相当于引入一种未定案的code
  。就像写一段没有遭受过bug的`fetch`调用一样.

这一系列的缺点并不是针对react. 它针对任何当挂载时才获取数据的库.  所以我们推荐下面的方法

* 如果你使用了框架,那么使用它内部搭载好的获取数据的机制.  现代react框架拥有完整的数据获取机制, 他们是高效的并且不会出上面的问题
* 此外, 考虑建立一个用户端缓冲.  大多数开源解决方法例如 React Query, useSWR,和React Router 6.4+.  你也可以建立自己的解决方案.  在这种情况下你会使用Effect,但也要添加用于删除重复数据请求、缓存响应和避免网络瀑布的逻辑(（通过预加载数据或将数据要求提升到路由）。)

如果这些方法适合你,那么你可以继续使用Effect来获取数据

---

### **具体化reactive依赖**

注意到你不能"选择"Effect的依赖.  每一个被用于Effect的reactive value必须被声明为一个依赖.  你Effect的依赖列表由包围的代码决定

```react
function ChatRoom({ roomId }) { // This is a reactive value
  const [serverUrl, setServerUrl] = useState('https://localhost:1234'); // This is a reactive value too

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // This Effect reads these reactive values
    connection.connect();
    return () => connection.disconnect();
  }, [serverUrl, roomId]); // ✅ So you must specify them as dependencies of your Effect
  // ...
}
```

如果`serverUrl`和`roomID`其中之一发生了改变, 那么你的Effect将会用新的值来重新链接chat room

reactive values 包含props以及所有变量及函数被声明在组件中.  自从`serveUrl`和`roomID`变为reactive value,你就不能将他们从依赖中移除. 如果你尝试去省去他们并且linter正确地被react配置, 那么linter将会将其标记为一个错误并且需要你去修补它

```react
function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');
  
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect has missing dependencies: 'roomId' and 'serverUrl'
  // ...
}
```

如果要移除依赖,那就必须向linter"证明"不需要这个依赖.

例如, 你可以将`serverUrl`从组件中移除来证明这个值在重新渲染时是不变的

```react
const serverUrl = 'https://localhost:1234'; // Not a reactive value anymore

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
}
```

现在`serverUrl`就不会是一个reactive value(在重新渲染时不会改变). 它并不需要被依赖. 如果你的Effect不需要任何reactive value ,那么他的依赖将会是一个[]

```react
const serverUrl = 'https://localhost:1234'; // Not a reactive value anymore
const roomId = 'music'; // Not a reactive value anymore

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ All dependencies declared
  // ...
}
```



依赖是[]的Effect只会在初始化时执行一次.

---

**陷阱**
如果你有一个正在存在的代码库, 你可能会像这样抑制linter

```react
useEffect(() => {
  // ...
  // 🔴 Avoid suppressing the linter like this:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

当依赖与code不匹配时,会有很高的风险引发bug.  通过抑制linter,你欺骗了react关于你Effect所需的依赖值, 反之,证明他们不必要

* **移除不必要的依赖**
  每次当你改变依赖值来重启code,  看一看依赖列表, 他们真的对Effect的重新执行有意义吗?有时候,答案是否定的
  * 你可能只是想在不同的条件下执行不同的行为,但是他们都封装到一个Effect上了
  * 你也许只是想获取最新的值
  * 也许依赖是一个对象,并且因此他们的更新太频繁了

为了寻找正确的解决方案,  你只需要回答以下几个简短的问题

* **这段代码应该移到事件处理函数中吗?**
  第一件事你需要想的就是是否这段code应该成为Effect.

  想象一个form, 当触发submit时,你将setState设置为true,你需要发送一个POST请求并且显示通知.  你将这段逻辑写到Effect中

  ```react
  function Form() {
    const [submitted, setSubmitted] = useState(false);
  
    useEffect(() => {
      if (submitted) {
        // 🔴 Avoid: Event-specific logic inside an Effect
        post('/api/register');
        showNotification('Successfully registered!');
      }
    }, [submitted]);
  
    function handleSubmit() {
      setSubmitted(true);
    }
  
    // ...
  }
  ```

  之后你想根据当前主题来显示通知信息,所以你获取当前的主题. 由于theme是定义在组件中的,所以是一个reactive value. 所以你将其加入为一个依赖:

  ```react
  function Form() {
    const [submitted, setSubmitted] = useState(false);
    const theme = useContext(ThemeContext);
  
    useEffect(() => {
      if (submitted) {
        // 🔴 Avoid: Event-specific logic inside an Effect
        post('/api/register');
        showNotification('Successfully registered!', theme);
      }
    }, [submitted, theme]); // ✅ All dependencies declared
  
    function handleSubmit() {
      setSubmitted(true);
    }  
  
    // ...
  }
  ```

  如果你这样做,那么你就会引入一个bug. 想象如果首先你将form submit出去,然后切换主题. 那么theme将会改变,Effect也会重新run一次,因此他将会再一次展示相同的主题. 

  这里的问题是，这首先不应该是一个Effect。您希望发送此 POST 请求并显示通知以响应提交表单，这是一个特定的交互。若要运行一些代码来响应特定的交互，请将该逻辑直接放入相应的事件处理程序中：

  ```react
  function Form() {
    const theme = useContext(ThemeContext);
  
    function handleSubmit() {
      // ✅ Good: Event-specific logic is called from event handlers
      post('/api/register');
      showNotification('Successfully registered!', theme);
    }  
  
    // ...
  }
  ```

  现在code就在一个事件处理函数中了,就不在是一个reactive value. 所以它只会在form submit时执行. 

* **你的Effect是否做了些不相关的事情**
  下一个你需要问的问题是是否你的Effect做了些不相关的事情。

  想象你正在创建一个运输表单，运输地点是用户选择的城市与区域。你从服务器捕获到了`cities`列表,根据被选中的`country`来展示他们

  ```react
  function ShippingForm({ country }) {
    const [cities, setCities] = useState(null);
    const [city, setCity] = useState(null);
  
    useEffect(() => {
      let ignore = false;
      fetch(`/api/cities?country=${country}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setCities(json);
          }
        });
      return () => {
        ignore = true;
      };
    }, [country]); // ✅ All dependencies declared
  
    // ...
  ```

  这是一个好的案例.  你正在通过`country`prop来同步`cities`state. 你没法在事件处理函数中这样做,你需要获取被展示出的`shippingFrom`数据并且在`country`改变后立即提取(无论是什么原因导致的改变)

  现在让我们假设为city area添加了第二个多选框,  应该获取哪个`city`的`area`数据? 你也许会添加第二个`fetch`在相同的Effect中来获取areas数据

  ```react
  function ShippingForm({ country }) {
    const [cities, setCities] = useState(null);
    const [city, setCity] = useState(null);
    const [areas, setAreas] = useState(null);
  
    useEffect(() => {
      let ignore = false;
      fetch(`/api/cities?country=${country}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setCities(json);
          }
        });
      // 🔴 Avoid: A single Effect synchronizes two independent processes
      if (city) {
        fetch(`/api/areas?city=${city}`)
          .then(response => response.json())
          .then(json => {
            if (!ignore) {
              setAreas(json);
            }
          });
      }
      return () => {
        ignore = true;
      };
    }, [country, city]); // ✅ All dependencies declared
  
    // ...
  ```

  然鹅,Effect现在使用`city`这个state,你还需要添加一个`city`作为依赖. 那将会引发一个问题: 当用户选择不同的城市,那么Effect将会在run一次并且调用`fetchCities(country)`.  作为结果,你也许将会非必要的重复获取cities多次.

  **问题的关键在于你你将两个不相关的事情进行了同步**

  1  你想同步`cities`state基于`country`props

  2  你想同步`areas`state通过`city`props

  将两段逻辑分离每个都关联着需要关联的props

  ```react
  function ShippingForm({ country }) {
    const [cities, setCities] = useState(null);
    useEffect(() => {
      let ignore = false;
      fetch(`/api/cities?country=${country}`)
        .then(response => response.json())
        .then(json => {
          if (!ignore) {
            setCities(json);
          }
        });
      return () => {
        ignore = true;
      };
    }, [country]); // ✅ All dependencies declared
  
    const [city, setCity] = useState(null);
    const [areas, setAreas] = useState(null);
    useEffect(() => {
      if (city) {
        let ignore = false;
        fetch(`/api/areas?city=${city}`)
          .then(response => response.json())
          .then(json => {
            if (!ignore) {
              setAreas(json);
            }
          });
        return () => {
          ignore = true;
        };
      }
    }, [city]); // ✅ All dependencies declared
  
    // ...
  
  ```

  现在第一个Effect只会在`country`改变时重新运行,同样的第二个Effect将会在`city`发生改变时重新运行. 你将他们分离: 两个不同的Effect同步不同的事情.  Effect有两个分开的依赖, 所以他们不会彼此触发。

  最终的code要比原始的长,但是分离这些Effect是正确的.  每个Effect应该代表一个独立的同步程序.  在这个例子中,  删除其中一个Effect不会破坏另一个Effect的逻辑.   这意味着他们同步不同的事情, 并且最好分离他们.  如果你担心Effect重复的问题. 你可以将这段代码抽象为一个自定义hook并引用

  ---

  

### 用Effect基于之前的State来更新State

在上面

### 移除不必要的对象依赖

在上面

### 移除不必要的函数依赖

与移除不必要的对象依赖类似



### 用Effect读取最新的props和state

暂时不稳定

### 在服务端和客户端展示不同的内容

如果你的app使用的是服务端渲染（无论是直接还是通过框架），你的组件将在不同的环境下渲染。在服务端，组件会渲染来产生初始HTML页面，在客户端，React将会在执行一遍code来增加事件处理函数。这也是为什么合成事件对于你的初次渲染必须在客户端与服务端是可识别的。

在很罕见的情况下，你可能需要在客户端展示不同的内容，例如，如果你的app从`localStorage`读取到一些数据，那么他就不可能在服务端读取，下面展示如何才能让其生效

```react  

function MyComponent() {
  const [didMount, setDidMount] = useState(false);

  useEffect(() => {
    setDidMount(true);
  }, []);

  if (didMount) {
    // ... return client-only JSX ...
  }  else {
    // ... return initial JSX ...
  }
}
```

当app正在加载时，用户将会看到初始渲染。然后，当组件加载并合并好后，你的Effect将会执行并将`didMount`设置为true，从而触发组件重绘。



## 排错

### 我的Effect在组件挂载时执行了两次

严格模式下会进行压力测试来证实你的Effect是完全正确的。

### 我的Effect每一次渲染都会执行

首先检查你的Effect是否遗忘了【】

如果你加上了【】但是仍旧每一次都会渲染，那么说明你传入的依赖每一次重绘都会发生改变

你可以通过手动打印这个依赖来解决bug

```react

  useEffect(() => {
    // ..
  }, [serverUrl, roomId]);

  console.log([serverUrl, roomId]);
```

然后，您可以在控制台中右键单击来自不同重新渲染的数组，并为它们选择“存储为全局变量”。假设第一个被保存为 temp1，第二个被保存为 temp2，然后您可以使用浏览器控制台检查两个数组中的每个依赖项是否相同：

```react
Object.is(temp1[0], temp2[0]); // Is the first dependency the same between the arrays?
Object.is(temp1[1], temp2[1]); // Is the second dependency the same between the arrays?
Object.is(temp1[2], temp2[2]); // ... and so on for every dependency ...
```

如果你发现每一次重绘时依赖值都不同，那么用下列方法修改他们

* 基于上一次state来更新state
* 移除不必要的对象
* 移除不必要的函数
* 用Effect读取罪行的props和state

如果上面的方法都不管用，尝试用useMemo或者useCallBack方法



### 我的Effect陷入无限循环

如果你的Effect陷入无限循环，那么两件事情必须确认

* 你的Effect正在更新State
* 那个state导致更新重绘，这导致Effect的依赖发生改变

在你解决这个问题之前，询问一下自己是否链接外部系统（例如DOM，netWork，第三方工具包，等等）为什么你的Effect内需要设置state？这样会同步外部系统吗？或者你正在尝试用它来管理应用的数据流？

如果没有外部系统，考虑是否完全移除Effect会简化逻辑？

如果你真的在同步某个外部系统，思考为什么以及在什么条件下需要更新Effect内的State。是当有什么东西变化时会影响到你的屏幕输出吗？如果你要追踪某些数据，并且这些数据不需要渲染到屏幕，那么ref也许更加合适。你需要证实是否真的需要用Effect来更新state从而触发渲染

最后，如果你的Effect在正确的时机更新State，但是仍旧导致有循环出现，那是因为更新State会触发依赖的改变。



### 我的Effect做了一些和视觉相关的内容，并且我看见在执行之前会闪一下。

如果你必须使用Effect让浏览器更新屏幕，用useLayoutEffect来代替。 注意对于大多数Effect这不是必要的。你只需要注意在浏览器进入paint状态之前执行Effect是否非常重要就可以。例如，在用户看见工具栏之前测量和定位工具栏。

























































