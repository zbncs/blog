<a name="NAWYF"></a>
# 为什么需要错误处理
我们都不希望自己的代码出问题，但是现实总是事与愿违。如果不进行错误处理，页面上就会出现error的场景，非常显眼。这是测试会说：小伙子，你的代码报错了。作为一个有追求的程序员，我们怎么可能承认自己的代码报错。<br />![f5dba0789297b53efded9f150e545cb4.gif](https://cdn.nlark.com/yuque/0/2023/gif/2180858/1677317077089-65cb7eb5-66a0-4262-a3f9-ddb64e2b50e5.gif#averageHue=%23241e1d&clientId=u54903199-4479-4&from=ui&height=185&id=u93ead69e&name=f5dba0789297b53efded9f150e545cb4.gif&originHeight=232&originWidth=416&originalType=binary&ratio=2&rotation=0&showTitle=false&size=1224628&status=done&style=none&taskId=u98394d26-befd-4dda-b6db-4b4ee94939e&title=&width=331)<br />为了提升用户体验（为了避免尴尬），我们必须优雅的处理它，让用户看到一个优雅的错误页面（让用户看不出来是代码错误），然后（偷偷地）修复它。<br />所以我们必须要进行错误处理。
<a name="cHDZX"></a>
# 错误处理方式
<a name="tSL3B"></a>
## try catch
try catch 捕获错误是我们最常用的方式。所以我们经常这样用：<br />方式一：
```javascript
const component = () => {
  const [error, setError] = useState(false)
  
  useEffect(() => {
    try {
      // 请求数据, async await
    } catch(e) {
      console.log(e)
      setError(e)
    }
  }, [])

  if (error) return <div>页面异常了...</div>

  return <div>page</div>
}
```
方式二：
```javascript
const component = () => {
  try {
    // 在组件渲染期间，状态计算等等。。。
  } catch(e) {
    console.log(e)
    return <ErrorPage>
  }

  return <div>page</div>
}
```
<a name="JLzxs"></a>
### try catch 存在一些局限性: 

1. 无法捕获后代组件错误
2. 不能捕获异步任务错误
<a name="nimjv"></a>
## ErrorBoundary
另一种在React中常见的错误处理方式就是：错误边界处理。错误边界可以处理React生命周期内发生的错误。所以他可以处理子组件错误和useEffect内部错误。
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      error: false
    }
  }

  static getDerivedStateFromError(error) {
    return { error: true };
  }

  componentDidCatch(error, errorInfo) {
    console.log(error, errorInfo)
  }

  render() {
    if (this.state.error) {
      return <>error!!!</>
    }

    return this.props.children;
  }
}
```
使用方式：
```javascript
const App = () => {
  return (
    <ErrorBoundary>
      <Component />
    </ErrorBoundary>
  )
}
```
<a name="ey3bh"></a>
### 错误边界也有自己的限制
不能捕获的错误：

1. promise异常
2. setTimeout, 各种回调函数的错误
3. 事件处理过程中错误
```javascript
const Component = () => {
  useEffect(() => {
    // 可以捕获
    throw new Error('哈哈哈!');
  }, [])

  const onClick = () => {
    // 不可以捕获
    throw new Error('哈哈哈!');
  }

  useEffect(() => {
    // promise 请求出错，不能捕获
    fetch('/bla')
  }, [])

  return <button onClick={onClick}>click me</button>
}

const App = () => {
  return (
    <ErrorBoundary>
      <Component />
    </ErrorBoundary>
  )
}
```
<a name="VdDVo"></a>
# 怎样在ErrorBoundary中处理异步等其他错误
在**React GitHub的issue种有一个hack方法**：使用try catch，然后再catch中触发React的重新渲染，从而进入React生命周期中，这样ErrorBoundary就可以捕获到错误了。
<a name="Yc8Kh"></a>
## 异步错误
```javascript
const Component = () => {
  const [state, setState] = useState();

  const onClick = () => {
    try {
      // 发生错误
    } catch (e) {
      // 触发state更新
      setState(() => {
        throw e;
      })
    }
  }
}
```
封装成一个自定义hook，方便使用：
```javascript
const useCatchAsyncError = () => {
  const [error, setError] = useState()

  return error => {
    setError(() => throw error)
  }
}


// 使用
const Component = () => {
  const setError = useCatchAsyncError();

  useEffect(() => {
    fetch('/abcd').then().catch((e) => {
      setError(e)
    })
  })
}
```
<a name="uoOlY"></a>
## 回调函数等错误
```javascript
const useCatchCallbackError = (callback) => {
  const [state, setState] = useState();

  return (...args) => {
    try {
      callback(...args);
    } catch(e) {
      setState(() => throw e);
    }
  }
}

// 使用
const Component = () => {
  const onClick = () => {
    throw new Error('哈哈哈!');
  }

  const onClickWithCatchError = useCatchCallbackError(onClick);

  return <button onClick={onClickWithCatchError}>点我一下试试!</button>
}
```
<a name="q2kmr"></a>
# 总结

- try/catch 可以捕获useEffect内部的错误，但不会捕获异步代码和事件处理程序中的错误
- ErrorBoundary可以捕获React是生命周期中发生的错误，不能捕获promise等回调函数中的错误
- 捕获promise等回调函数中的错误，只需要用 ErrorBoundary ，然后让它们重新进入React 生命周期 ，使用try/catch捕获
<a name="wFdoB"></a>
# reference
hack方法：[https://github.com/facebook/react/issues/14981#issuecomment-468460187](https://github.com/facebook/react/issues/14981#issuecomment-468460187)

