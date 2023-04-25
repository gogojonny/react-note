# memo

`memo` 允许您在组件的prop不变时跳过重新渲染组件。

```react
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```

**参数**

* `memo(SomeComponent, arePropsEqual?)`

**使用**

* 当prop未发生变化时跳过重绘
* 使用state更新缓存组件
* 使用context更新缓存组件
* 最小化props的改变
* 指定自定义比较功能

疑难解答

* 当props是对象，函数或者数组时，我的组件总会重绘

## 参考

### `memo(Component, arePropsEqual?)`

将组件包装在`memo`中以获取该组件的记忆版本。只要它的 props 没有更改，当它的父组件被重新渲染时，这个缓存的组件通常不会重新渲染。但 React 可能仍然会重新渲染它：缓存是一种性能优化，而不是保证。

```react
import { memo } from 'react';

const SomeComponent = memo(function SomeComponent(props) {
  // ...
});
```

**参数**

* `component`： 要缓存的组件，memo不会修改此组件，而是返回一个新的缓存组件。任何有效的 React 组件，包括函数和` forwardRef `组件，都被接受。
* （可选）`arePropsEqual`：一个接受两个参数的函数：组件以前的 props 和它的新 props。如果新旧 props 相等，它应该返回 true：也就是说，如果组件将呈现相同的输出并且新 props 的行为方式与旧 props 相同。否则它应该返回 false。通常，您不会指定此功能。默认情况下，React 会将每个 prop 与 Object.is 进行比较。

**返回值**
memo 返回一个新的 React 组件。它的行为与提供给 memo 的组件相同， React 不会总是在其父级被重新渲染时重新渲染它，除非它的props发生了变化。



## 使用

### 当props未发生改变时跳过重绘

React 通常会在组件的父组件重新渲染时重新渲染组件。使用 memo，您可以创建一个组件，只要其新props与旧props相同，当其父级重新渲染时，React 就不会重新渲染该组件。这样的组件是被缓存的。

```react
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```

React 组件应该始终具有纯渲染逻辑。这意味着如果它的 props、state和context没有改变，它必须返回相同的输出。通过使用 memo，你告诉 React 你的组件符合这个要求，所以只要它的props没有改变，React 就不需要重新渲染。**即使使用 memo，如果组件自己的状态发生更改或它正在使用的context发生更改，组件也会重绘。**

在此示例中，请注意，每当name更改时，Greeting 组件都会重新呈现（因为这是它的 props 之一），但在address更改时不会（因为它不会作为 proping 传递给 Greeting）：

```react
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        Name{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        Address{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  return <h3>Hello{name && ', '}{name}!</h3>;
});

```

---

**注意：**

您应该只依靠memo作为性能优化。如果您的代码在没有它的情况下无法工作，请找到根本问题并首先修复它。然后，您可以添加memo以提高性能。

---

**深度探讨**
**你应该在任何地方加memo吗？**

如果您的应用程序与此网站类似，并且大多数交互都很粗糙（例如替换页面或整个部分），则通常不需要记忆。另一方面，如果你的应用更像是一个绘图编辑器，并且大多数交互都是精细的（如移动形状），那么你可能会发现记忆非常有用。

仅当您的组件经常使用完全相同的props重新渲染时，使用 memo 进行优化才有价值，并且其重新渲染逻辑非常昂贵。如果在组件重新渲染时没有明显的延迟，则不需要 memo。请记住，如果传递给组件的 props 总是不同的，例如，如果您传递渲染期间定义的对象或普通函数，则 memo 是完全无用的。这就是为什么你经常需要useMemo和useCallback以及memo。

在其他情况下，将组件包装在memo中没有任何好处。这样做也没有什么大不了的，所以一些团队选择不考虑个别情况，并尽可能多地缓存。这种方法的缺点是代码变得不那么可读了。此外，并不是所有的缓存都是有效的：一个“永远是新的”的单一值就足以破坏整个组件的缓存。

**在实践中，您可以通过遵循以下几个原则来使大量不必要的缓存**：

* 当一个组件在视觉上包装其他组件时，让它接受 JSX 作为子组件。这样，当包装组件更新自己的状态时，React 知道它的子组件不需要重新渲染。
* 更加偏好本地state并且尽量不要提升状态。例如，不要使用短暂的state例如form，无论是在组件树顶还是在本地state库中
* 保持渲染逻辑纯净。如果重新渲染组件会导致问题或产生一些明显的视觉伪影，则这是组件中的错误！修复错误，而不是添加缓存。
* 避免更新状态的不必要的效果。React 应用程序中的大多数性能问题都是由源自 Effects 的更新链引起的，这些更新链导致组件一遍又一遍地渲染。
* 尝试从Effect中删除不必要的依赖项。例如，将某些对象或函数移动到Effect内部或组件外部通常更简单，而不是缓存。

如果特定的交互仍然感觉滞后，请使用[React Developer Tools profiler](https://legacy.reactjs.org/blog/2018/09/10/introducing-the-react-profiler.html) 查看哪些组件将从缓存中受益最大，并在需要时添加缓存。这些原则使您的组件更易于调试和理解，因此在任何情况下都最好遵循它们。从长远来看，我们正在研究自动进行精细缓存，以一劳永逸地解决这个问题。

---

### 使用state更新缓存组件

即使一个组件被缓存，当它自己的state发生变化时，它仍然会重新渲染。缓存只与从父组件传递给组件的道具有关

```react
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        Name{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        Address{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log('Greeting was rendered at', new Date().toLocaleTimeString());
  const [greeting, setGreeting] = useState('Hello');
  return (
    <>
      <h3>{greeting}{name && ', '}{name}!</h3>
      <GreetingSelector value={greeting} onChange={setGreeting} />
    </>
  );
});

function GreetingSelector({ value, onChange }) {
  return (
    <>
      <label>
        <input
          type="radio"
          checked={value === 'Hello'}
          onChange={e => onChange('Hello')}
        />
        Regular greeting
      </label>
      <label>
        <input
          type="radio"
          checked={value === 'Hello and welcome'}
          onChange={e => onChange('Hello and welcome')}
        />
        Enthusiastic greeting
      </label>
    </>
  );
}

```

如果您将状态变量设置为当前值,即使没有memo,React▁也会跳过重新渲染您的组件。您可能仍然会看到您的组件函数被额外调用一次,但结果将被丢弃。

### 使用context更新缓存组件

即使缓存组件，当它使用的context发生变化时，它仍然会重绘。缓存只与从其父级传递给组件的props有关。

使用context也会改变组件的主题，因此，组件重绘了

```react
import { createContext, memo, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('dark');

  function handleClick() {
    setTheme(theme === 'dark' ? 'light' : 'dark'); 
  }

  return (
    <ThemeContext.Provider value={theme}>
      <button onClick={handleClick}>
        Switch theme
      </button>
      <Greeting name="Taylor" />
    </ThemeContext.Provider>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  const theme = useContext(ThemeContext);
  return (
    <h3 className={theme}>Hello, {name}!</h3>
  );
});

```

为了让你的组件只会在context改变时重绘，你可以将组件一分为二。获取你想要在上层组件中获取的内容，然后将其用context传入并作为组件的prop。

### 最小化props改变

当你使用 memo 时，只要任何 prop 与之前的 prop 不相等，你的组件就会重新渲染。这意味着 React 使用 Object.is 比较将组件中的每个 prop 与其之前的值进行比较。请注意，Object.is(3, 3) 为真，但 Object.is({}, {}) 为假。

为了充分利用memo，请尽量减少prop更改的次数。例如，如果 prop 是一个对象，则使用 useMemo 防止父组件每次重新创建该对象：

```react
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  const person = useMemo(
    () => ({ name, age }),
    [name, age]
  );

  return <Profile person={person} />;
}

const Profile = memo(function Profile({ person }) {
  // ...
});
```

最小化道具更改的更好方法是确保组件在其prop中接受最少的必要信息。例如，它可以接受单个值而不是整个对象：

```react
function Page() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);
  return <Profile name={name} age={age} />;
}

const Profile = memo(function Profile({ name, age }) {
  // ...
});
```

甚至单个值有时也可以被预测为变化频率较低的值。例如,在这里,一个组件接受一个布尔值,表示值的存在,而不是值本身:

```react
function GroupsLanding({ person }) {
  const hasGroups = person.groups !== null;
  return <CallToAction hasGroups={hasGroups} />;
}

const CallToAction = memo(function CallToAction({ hasGroups }) {
  // ...
});
```

当您需要将函数传递给缓存组件时，请在组件外部声明它，以便它永远不会更改，或者使用 Callback 在重新渲染之间缓存其定义。

### 指定自定义比较功能

在极少数情况下，最小化记忆组件的prop更改可能是不可行的。在这种情况下，你可以提供一个自定义的比较函数，React 将使用它来比较新旧prop，而不是使用浅相等。此函数作为第二个参数传递给memo。仅当新prop产生与旧prop相同的输出时，它才应返回 true;否则它应该返回 false。

```react
const Chart = memo(function Chart({ dataPoints }) {
  // ...
}, arePropsEqual);

function arePropsEqual(oldProps, newProps) {
  return (
    oldProps.dataPoints.length === newProps.dataPoints.length &&
    oldProps.dataPoints.every((oldPoint, index) => {
      const newPoint = newProps.dataPoints[index];
      return oldPoint.x === newPoint.x && oldPoint.y === newPoint.y;
    })
  );
}
```

如果执行此操作，请使用浏览器开发人员工具中的“性能”面板，以确保比较函数实际上比重新呈现组件更快。你可能会感到惊讶。

当你进行性能测量时，请确保 React 在生产模式下运行。

---

**陷阱**
如果您提供自定义的 arePropsEqual 实现，则必须比较每个 prop，包括函数。函数通常会让父组件的 props 和state成为闭包。如果你在oldProps.onClick ！== newProps.onClick时返回true，你的组件将在其onClick处理程序中不断“看”来自先前渲染的props和state，从而导致非常混乱的错误。

避免在 arePropsEqual 中进行深度相等检查，除非您 100% 确定您正在使用的数据结构具有已知的有限深度。深度相等性检查可能会变得非常慢，如果以后有人更改数据结构，可能会冻结您的应用几秒钟。

---

## 疑难解答

### 当我的prop是对象，函数或者数组时，我的组件总是重绘

React 通过浅相等来比较新旧prop：也就是说，它考虑每个新prop是否与旧prop的引用相等。如果你每次重新渲染父元素时都会创建一个新对象或数组，即使各个元素都相同，React 仍然会认为它被更改了。同样，如果您在渲染父组件时创建了一个新函数，即使该函数具有相同的定义，React 也会认为它已更改。为避免这种情况，请简化 props 或记住父组件中的 props。

























