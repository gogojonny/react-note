# react state

```react
  // 细谈state
  /*
  state
      -state实际上是react的一个变量，通过调用setState可以让组件重新渲染
      -setState在传入对象时需要注意，由于setState会比较原始state的值和传入的值是否相等来决定是否重绘，所以在传入对象时若只有属性 值发生了修改而对象地址没有修改，那么不会发生重绘，因此必须传入一个新的对象，若是想修改某一对象，建议使用解构操作符进行操作
      
      -setState的执行是异步的，这会导致一些问题
            * 这可能导致重绘的值出现错误，建议的做法是在setState内部使用回调函数，接收上一次重绘的值并执行这一次和返回这一次的值
   
  
  */
```



```react
const [num, setNum] = useState(0);
  let [user, setUser] = useState({ name: "matt", age: 18 });
  let add = () => {
    setNum(num + 1);
  };

  function change() {
    // 直接改为新的对象，会重绘
    // let newUser = { name: "foo" };

    // 只修改源对象的属性，没有修改地址，不会进行渲染
    // user.name = "foo";
    // console.log(user); //{name: 'foo', age: 18}

    // 比较优雅的方式修改对象
    setUser({ ...user, name: "foo" });
  }
  function showNum() {
    console.log(num);
  }

  return (
    <div>
      <div className="add">
        <h1>{num}</h1>
        <h1>
          name:{user.name},age:{user.age}
        </h1>
        <button
          onClick={() => {
            add();
          }}
        >
          add
        </button>
        <button
          onClick={() => {
            change();
          }}
        >
          change user
        </button>
        <button
          onClick={() => {
            showNum();
          }}
        >
          show num
        </button>
      </div>
    </div>
  );
}
```



## state 异步渲染

![image-20230413144931533](C:\Users\35392\AppData\Roaming\Typora\typora-user-images\image-20230413144931533.png)

![image-20230413144956909](C:\Users\35392\AppData\Roaming\Typora\typora-user-images\image-20230413144956909.png)

**解决方案**

```react
console.log("进行了一次渲染");
  const [num, setNum] = useState(0);
  let add = () => {
    // 点击两下会让两个settimeout进入异步队列等待
    // 1s 后执行时两个队列中的任务都会拿到num=0
    setTimeout(() => {
      setNum((num) => num + 1);
      // setNum里的函数中，num代表将上次返回的结果作为这次传入的参数，
      console.log(num);
    }, 1000);
  };
```

![image-20230413145422466](C:\Users\35392\AppData\Roaming\Typora\typora-user-images\image-20230413145422466.png)





## useState

useState是一个react hook 用于添加一个``state``到你的组件中

```react
const [state, setState] = useState(initialState);
```



**参考**

* ``[`useState(initialState)`]``
* ``set``函数,类似于`[`setSomething(nextState)`]`

**使用**

* 给一个组件添加state
* 基于之前的state更新state
* 在state中更新数组与对象
* 避免重复创建初始state
* 用key来重设state值
* 从上一次渲染结果中存储信息

**排错**

* 我要更新数据,但是打印的结果是旧的state
* 我要更新数据,但是屏幕没有发生变化
* 我获得一个错误提示: [“Too many re-renders”]
* 我的initializer或者updater函数执行了两次
* 我尝试给一个函数设置state,但是替代为执行这个函数



### 参考

#### ```useState(initialState)```

```react
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
```



**参数**

* initialState: 用于初始化你的state值,理论上可以是任意类型,但是对于函数类型有不同的行为.
  * 如果你传入一个函数,  那么这个函数会被当做 ``initializer function``.  这个函数应该是一个纯函数,并且没有传入参数,返回一个任意类型的值.  react将会调用你设置的函数当初始化组件时, 并且将会把返回值作为初始值传入state中

**返回值**

``useState``将会返回两个值

* 当前的state. 在第一次渲染时,它将会与你传入 的initialState值进行匹配
* 一个set函数,可以让你更新state的值并触发重绘

**警告**

* `useState`是一个hook,所以你只能在你的组件顶部调用或者在你的自定义hook中调用.  你不能在loop或者条件语句中调用.  如果你需要这样做,那就提取并放入一个新的组件中
* 在严格模式下, react将会调用两次你的initializer function ,这样做的目的是帮助你找到无意识间的杂质. 这个行为是在开发时期才会有的,并不会影响到产品 .如果你的initializer function是一个纯函数,  那将不会影响到原本的行为. 从其中一次调用的结果将会被忽略



____



#### `set` functions, like `setSomething(nextState)`

set函数源于`useState`并且可以帮助你更细state的值并且会触发重绘.你可以直接传入下一次的值或者传入一个函数并从上一次的state值中计算出这次的state值

```react
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
```



**参数**

* `nextState`: 下一次你想传入的值,它可以是任意类型的值,但是如果是一个函数,就会有不同的行为
  *  如果你传入一个函数.  那么这个函数就会被当做一个更新函数.这个函数必须是一个纯函数,  这个函数的参数是一个期待的state值,并且应该返回下一个state的值.  react将会将这个函数丢到队列中并且重新渲染组件.  在下一次渲染时,react将会计算新的state值通过调用所有的队列中的更新函数并更新之前的state.

**返回值**

set函数没有返回值

**警告**

* set函数只会在下一次渲染时更新state.  如果你在set函数之后读取state的值,那么state的值就还是会是旧值.  
* 如果你提供的新值与旧值相同(通过Object.is来比较),  react将会跳过渲染该组件以及其子元素.   这就是优化方案 .   尽管在一些情况下react也会仍会在跳过子元素之前渲染组件, 也不会影响你的代码

* react批次状态更新:  在所有事件处理函数已经执行并且调用set函数后便会更新屏幕.  这会阻止在一个事件中反复渲染多次.  在某些情况下 ,你需要强迫react提早更新屏幕,例如在获取DOM时,这时你可以使用flushSync
* 在渲染时调用set函数的情况仅仅发生在当前正在渲染的组件中.  react将会丢弃输出并且立即尝试用新的state再次渲染.  这种模式很少被需要,但是你可以使用这个模式去储存一些来自上一次渲染的信息.

---



例如:

**从上一次渲染结果中存储信息**

通常,你会通过事件处理函数来更新state.  然而在某些情况下,你只是想重新渲染一下页面.  例如: 你也许只是想修改一下state值当props发生修改时.   在大多数情况下,你不需要这样做:

* 当你从props或者其他的state中就已经能得到想要的值时,就没必要在创建一个state来获得这个值.如果你担心重算次数太多,那么可以使用`useMemo`hook来帮助你

  ---

  

  * **移除多余的state**

    如果你能从组件的props中或者已存的state中获得一些信息.  那么你不应该将那份信息放入到组件的state中.  

    **在state中不要映射props:**

    下面是一个常见的多余state代码的案例

    ```react
    function Message({ messageColor }) {
      const [color, setColor] = useState(messageColor);
    ```

    在这里,color这个state被初始化为`messageColor`.  问题是如果父组件等一会儿会传递一个不同的`messageColor`值,那么color这个state不会更新.  state的初始化只会发生在第一次渲染时

    这也是为什么"mirroring"一些props时会导致困惑.  这就像在代码中直接使用props的`messageColor`,如果你想给这个`messageColor`一个暂时的名字,并且使用一个常量来表示

    ```react
    function Message({ messageColor }) {
      const color = messageColor;
    ```

    这样就不会从父组件中传来的props中脱离同步.

    在state中使用"mirroring"props只会在当你想忽略所有的props更新时才会有意义, 这时你可以使用initialColor或者defaultColor来表示初始color值,这样也更易于阅读

    ```react
    function Message({ initialColor }) {
      // The `color` state variable holds the *first* value of `initialColor`.
      // Further changes to the `initialColor` prop are ignored.
      const [color, setColor] = useState(initialColor);
    ```

    

* 如果你想完全重新设置组件树的state,那么就传入一个新的key过来

  * **传入一个新的key**来重置state:

    当你渲染一个列表时总是会遇到key.  然鹅, key也提供了另一个目的.

    你可以向一个组件中传入一个不同的key用于重置组件的状态. 在这个案例中, reset按钮会改变`version`这个state的值,通过向`Form`中传递一个key. 当key发生改变时, react将会重新渲染`From`组件(以及所有子元素) ,因此state就会重置.

    其基本的原理就是diff算法会更改所有发生改变的组件,key发生了改变的话,那么组件就会重绘,同时组件内部的state也会重新设置,这样就达到重置组件的state的目的了

    ```react
    import { useState } from 'react';
    
    export default function App() {
      const [version, setVersion] = useState(0);
    
      function handleReset() {
        setVersion(version + 1);
      }
    
      return (
        <>
          <button onClick={handleReset}>Reset</button>
          <Form key={version} />
        </>
      );
    }
    
    function Form() {
      const [name, setName] = useState('Taylor');
    
      return (
        <>
          <input
            value={name}
            onChange={e => setName(e.target.value)}
          />
          <p>Hello, {name}.</p>
        </>
      );
    }
    ```

* 只要你想,你可以在一个事件处理函数中更新所有的state

---



react没有提供有关变化率的函数,下面是一个用于更新state基于最近渲染的值的案例,通过调用set函数.

下面是案例,这个`CountLabel`组件展示`count`

```react
export default function CountLabel({ count }) {
  return <h1>{count}</h1>
}
```

假如你想展示count的数量是增多了还是减少了,count本身是不会告诉你这些的--你需要追踪其前一个值.  在组件中添加一个`prevCount`的state来追踪它.在加一个`trend`的state来确定count的值是增加了还是减少了. 通过比较`prevCount`与`count`来确定趋势是增加还是减少

```react
import { useState } from 'react';

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? 'increasing' : 'decreasing');
  }
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}

```



**注意当你在渲染时调用set函数,必须添加一个内置条件`prevCount !== count`,并且必须在条件中调用`setPrevCount(count)`,如果不这样做,那么你的组件将会一直重绘直到崩溃. 此外, 像这样同步更新state必须是在当前组件中发生的.在渲染期间调用其他组件中的set函数将导致出现err.   最后,在你调用set函数时应该注意不要修改, 而是使用代替---但这也不意味着破坏其他规则**

`update state without mutation`

* 你可以将数组与对象放入state中,在react中state是只读的,所以你应该代替而不是修改已存在的对象. 例如你有一个from对象在state中,不要尝试修改它

  ```react
  // 🚩 Don't mutate an object in state like this:
  form.firstName = 'Taylor';
  ```

  相反,你应该直接拿一个新的对象去代替

  ```react
  // ✅ Replace state with a new object
  setForm({
    ...form,
    firstName: 'Taylor'
  });
  ```



这个案例也许很难理解并且通常最好避免的.  然鹅, 这个比在effect中更新state要好.  当你在渲染时调用set函数, react将会立刻渲染那个组件 在你的组件存在且声明了return之后,并且在渲染子元素之前. 用这种方法,子元素就不必渲染两次.  其余的组件函数将会继续执行(但是会丢掉结果值), 如果您的条件低于所有钩子调用，您可以添加提前返回以提前重新开始渲染

```react
functionsomeFunction(someCondition) {
    if (!someCondition) {
        return;
    }   // Do something
}
```

---





* 在严格模式下, react将会调用更新函数两次,目的是帮助你找到暇点. 这个行为只是在开发阶段出现,不会影响到产品. 如果你的更新函数时纯函数,这应该不会影响到行为.  其中之一的结果会被忽略





### 使用

**给组件添加state**

在组件的顶部调用`useState`来声明一个或多个state变量

```react
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
  // ...
```

`useState`返回一个包含两个值的数组

* 一个是state变量,用initialValue赋初值,注意,这个赋初值只会在第一次渲染时生效,后面想修改这个state值只能使用setState函数来修改,即使整个组件重绘也不会修改这个state值

* 一个set函数,专门用于修改state值的

react将会存储next state,渲染你的组件用new value,并且更新UI

---



**陷阱**

在set函数后面读取state返回的是旧值,因为set函数是异步执行的

**排错**

* 我更新了state,但是log打印的是旧值

  调用set函数不会在正在执行的code中改变state

  ```react
  function handleClick() {
    console.log(count);  // 0
  
    setCount(count + 1); // Request a re-render with 1
    console.log(count);  // Still 0!
  
    setTimeout(() => {
      console.log(count); // Also 0!
    }, 5000);
  }
  ```

  这是因为state的行为像一个快照. 更新state会用new value请求重新渲染,而不是影响`count`这个跑在执行中的代码的JavaScript变量.  

  如果你需要使用更新后的`count`,那么就先把更新的值保存在一个变量中

  ```react
  const nextCount = count + 1;
  setCount(nextCount);
  
  console.log(count);     // 0
  console.log(nextCount); // 1
  ```

  

---



**基于前面的state来更新state**

`age`是42,这个函数会调用`setAge(age+1)`三次

```react
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

然鹅,在点击触发这个函数之后,结果只会是43,而不是45. 这是因为在调用set函数时并没有改变在正在执行的代码中的`age`的值.  所以每个set函数都会只返回43.

为了解决这个问题,你可以传入一个update函数来代替传入的state

```react
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

在这里,`a => a + 1`就是update函数,它会取走pending状态的state并计算下一次state值.

react将update函数放入到队列中,然后,在下一次渲染时,它会用同样的命令调用他们.

习惯性的,你可以用prev来称呼在pending状态的state.

---



**深入探讨**

**是否使用update函数总是更好的**

你也许听从建议总是将code写成`setAge(a => a + 1)`,当你写state时.  这并没有坏处,但并不总是必要的

在大多数情况,两种调用set函数的方法是无差的,react总是会确认用户的有意行为, 例如点击,`age`state将会在下一次点之前更新. 这意味着事件函数一开始就寻找一个过期的`age`作为点击事件是没有风险的.

然鹅,如果你用相同的事件做反复更新. 那么update函数就会有用.   如果变量自己是inconvenient的状态(你也许在最优重绘时使用它）,那么这个update函数也是有用的

如果你喜欢一致性以至于能忍受稍微复杂的语法,那么总是这样写是完全可以的.你也可以使用reducer来简化这一过程

---



**避免重复创造初始state**

react只会保存一次初始的state，并且在下一次渲染时忽略uesState这段代码

```react
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```

在这里,虽然`createInitialTodos()`这个初始化函数的值只会被用于初次渲染.  但上面的代码表示,你仍然在每次渲染时调用这个函数. 这样做如果它创造一个大的数组或者执行复杂的运算,那么就会导致浪费.

为了解决这个问题, 你可以传递一个initializer函数来代替

```react
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
```

注意,你传入的是一个函数,而不是函数的返回值.  如果你传入的是一个函数,那么这个函数就只会在这个state的初次渲染时起作用

另外,react会调用你设定的initializers两次用于确定这个函数是一个纯函数



**传入一个新的key来重置state:**

当你渲染一个列表时总是会遇到key.  然鹅, key也提供了另一个目的.

你可以向一个组件中传入一个不同的key用于重置组件的状态. 在这个案例中, reset按钮会改变`version`这个state的值,通过向`Form`中传递一个key. 当key发生改变时, react将会重新渲染`From`组件(以及所有子元素) ,因此state就会重置.

其基本的原理就是diff算法会更改所有发生改变的组件,key发生了改变的话,那么组件就会重绘,同时组件内部的state也会重新设置,这样就达到重置组件的state的目的了

以前通过props给state设置值的时候,并不能通过修改props来改变state的初始值,这是因为state的初始值只会执行一次.  而将组件reset后, 就相当于整个组件被删除然后重新设置,这样就可以通过props来设置初始值,并且props改变时,也会让初始值改变了.  但是reset一个组件,往往会导致性能有所降低, 其实通过props来修改state的方式并不推荐,也没必要.

```react
import { useState } from 'react';

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState('Taylor');

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <p>Hello, {name}.</p>
    </>
  );
}
```



---



### 排错

**我更新了state,但是屏幕上没有更新**

如果上一次的state值与这一次新设置的state值一样,那么react就会忽略这次的更新.  这种比较是通过`Object.is`进行的,这种情况通常出现在对象或者数组中

```react
obj.x = 10;  // 🚩 Wrong: mutating existing object
setObj(obj); // 🚩 Doesn't do anything
```

这种情况应该这样使用

```react
// ✅ Correct: creating a new object
setObj({
  ...obj,
  x: 10
});
```



**我获取到一个错误提示:"too many re-renders''**

有时你也许会获得这样一个错误提示:``Too many re-renders. React limits the number of renders to prevent an infinite loop.``   这意味着在你进行渲染时没有给state设置条件 ,  所以你的组件将进入循环:  渲染---遇到setState----在渲染---在遇到setState......

这种错误在具体的事件处理函数中也经常遇到

```react
// 🚩 Wrong: calls the handler during render
return <button onClick={handleClick()}>Click me</button>

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>

// ✅ Correct: passes down an inline function
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

如果你找不到具体的错误原因, 那么点击紧挨着err报告的箭头并且监视JavaScript栈调用去寻找具体的set函数



**我的initializer或者update函数执行了两次**

在严格模式下,react会调用你的函数两次

```react
function TodoList() {
  // This component function will run twice for every render.

  const [todos, setTodos] = useState(() => {
    // This initializer function will run twice during initialization.
    return createTodos();
  });

  function handleClick() {
    setTodos(prevTodos => {
      // This updater function will run twice for every click.
      return [...prevTodos, createTodo()];
    });
  }
  // ...
```



这个仅在开发阶段执行的行为会保证你的组件够纯.  react使用其中一个调用的结果并忽略另一次调用的结果.  只要你的组件组件,  initializer, updater函数是纯函数,这就不会影响到逻辑, 然鹅, 如果他们其中有哪些不纯, 这将会帮助你察觉到问题

例如,下面这个不纯的update修改了一个数组

```react
setTodos(prevTodos => {
  // 🚩 Mistake: mutating state
  prevTodos.push(createTodo());
});
```

react会两次调用你的update函数,来检查两次相同的输入是否得到相同的输出,上面的代码明显两次输入后得到的值不一样,因此就会提示你这不是一个纯函数

解决上述问题的方案是,用代替而不是修改

```react
setTodos(prevTodos => {
  // ✅ Correct: replacing with new state
  return [...prevTodos, createTodo()];
});
```

注意: 只有组件,initializer,updater函数需要是纯函数,而事件处理函数可以不必是纯函数,因此react从来不会调用事件处理函数两次



**我尝试将一个函数作为初始值,但这个函数居然执行并将结果作为初始值了**

你不可以像这样将函数作为初始值

```react
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

因为你传入的函数被react认为是一个inititlizar函数, 并且

`someOtherFunction`被当做一个updater函数,  所以react将尝试调用他们并储存返回值.  如果你真想让一个函数作为初始值,应该这样写

```react
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```























