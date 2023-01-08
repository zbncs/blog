# 背景
相信大家在写react时都有这样的经历：在项目中使用了大量的useEffect，以至于让我们的代码变得混乱和难以维护。<br />难道说useEffect这个hook不好吗？并不是这样的，只是我们一直在滥用而已。<br />在这篇文章中，我将展示怎样使用其他方法来代替useEffect。
# 什么是useEffect
useEffect允许我们在函数组件中执行副作用。它可以模拟 componentDidMount、componentDidUpdate 和componentWillUnmount。我们可以用它来做很多事情。但是它也是一个非常危险的钩子，可能会导致很多bug。
# 为什么useEffect是容易出现bug的
来看一个定时器的例子：
```javascript
import React, { useEffect } from 'react'

const Counter = () => {
  const [count, setCount] = useState(0)

  useEffect(() => {
    const timer = setInterval(() => {
      setCount((c) => c + 1)
    }, 1000)
  })

  return <div>{count}</div>
}
```
这是一个非常常见的例子，但是它是非常不好。因为如果组件由于某种原因重新渲染，就会重新设置定时器。该定时器将每秒调用两次，很容易导致内存泄漏。
## 怎样修复它？
### useRef
```javascript
import React, { useEffect, useRef } from 'react'

const Counter = () => {
  const [count, setCount] = useState(0)
  const timerRef = useRef()

  useEffect(() => {
    timerRef.current = setInterval(() => {
      setCount((c) => c + 1)
    }, 1000)
    return () => clearInterval(timerRef.current)
  }, [])

  return <div>{count}</div>
}
```
它不会在每次组件重新渲染时设置定时器。但是我们在项目中并不是这么简单的代码。而是各种状态，做各种事情。
# 你以为你写的useEffect
```javascript
useEffect(() => {
  doSomething()

  return () => cleanup()
}, [whenThisChanges])
```
# 实际上是这样的
```javascript
useEffect(() => {
  if (foo && bar && (baz || quo)) {
    doSomething()
  } else {
    doSomethingElse()
  }
  // 遗忘清理函数。
}, [foo, bar, baz, quo, ...])
```
写了一堆的逻辑，这种代码非常混乱难以维护。
# useEffect 到底是用来干啥的
> useEffect是一种将React与一些外部系统(网络、订阅、DOM)同步的方法。如果你没有任何外部系统，只是试图用useEffect管理数据流，你就会遇到问题。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2180858/1672209506465-c58af1f3-b68a-439d-b8db-1435387e3971.png#averageHue=%23fdfdfd&clientId=u7abd42e9-ed60-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=230&id=uabe43ee8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=460&originWidth=980&originalType=binary&ratio=1&rotation=0&showTitle=false&size=100936&status=done&style=stroke&taskId=ue52a85c1-a36a-4d85-8cac-88ceab26841&title=&width=490)
# 有时我们并不需要useEffect
### 1.我们不需要useEffect转化数据
```javascript
const Cart = () => {
  const [items, setItems] = useState([])
  const [total, setTotal] = useState(0)

  useEffect(() => {
    setTotal(items.reduce((total, item) => total + item.price, 0))
  }, [items])

  // ...
}
```
上面代码使用useEffect来进行数据的转化，效率很低。其实并不需要使用useEffect。当某些值可以从现有的props或state中计算出来时，不要把它放在状态中，在渲染期间计算它。
```javascript
const Cart = () => {
  const [items, setItems] = useState([])
  const [total, setTotal] = useState(0)

  const totalNum = items.reduce((total, item) => total + item.price, 0)

  // ...
}
```
如果计算逻辑比较复杂，可以使用useMemo：
```javascript
const Cart = () => {
  const [items, setItems] = useState([])
  const total = useMemo(() => {
    return items.reduce((total, item) => total + item.price, 0)
  }, [items])

  // ...
}
```
### 2.使用useSyncExternalStore代替useEffect
[useSyncExternalStore](https://beta.reactjs.org/reference/react/useSyncExternalStore#usage)<br />常见方式：
```javascript
const Store = () => {
  const [isConnected, setIsConnected] = useState(true)

  useEffect(() => {
    const sub = storeApi.subscribe(({ status }) => {
      setIsConnected(status === 'connected')
    })

    return () => {
      sub.unsubscribe()
    }
  }, [])

  // ...
}
```
更好的方式：
```javascript
const Store = () => {
  const isConnected = useSyncExternalStore(
    storeApi.subscribe,
    () => storeApi.getStatus() === 'connected',
    true
  )

  // ...
}
```
### 3.没必要使用useEffect与父组件通信
```javascript
const ChildProduct = ({ onOpen, onClose }) => {
  const [isOpen, setIsOpen] = useState(false)

  useEffect(() => {
    if (isOpen) {
      onOpen()
    } else {
      onClose()
    }
  }, [isOpen])

  return (
    <div>
      <button
        onClick={() => {
          setIsOpen(!isOpen)
        }}
      >
        Toggle quick view
      </button>
    </div>
  )
}
```
更好的方式，可以使用事件处理函数代替：
```javascript
const ChildProduct = ({ onOpen, onClose }) => {
  const [isOpen, setIsOpen] = useState(false)

const handleToggle = () => {
  const nextIsOpen = !isOpen;
  setIsOpen(nextIsOpen)

  if (nextIsOpen) {
    onOpen()
  } else {
    onClose()
  }
}

  return (
    <div>
      <button
        onClick={}
      >
        Toggle quick view
      </button>
    </div>
  )
}
```
### 4.没必要使用useEffect初始化应用程序
```javascript
const Store = () => {
  useEffect(() => {
    storeApi.authenticate()
  }, [])

  // ...
}
```
更好的方式：<br />方式一：
```javascript
const Store = () => {
  const didAuthenticateRef = useRef()

  useEffect(() => {
    if (didAuthenticateRef.current) return

    storeApi.authenticate()

    didAuthenticateRef.current = true
  }, [])

  // ...
}
```
方式二：
```javascript
let didAuthenticate = false

const Store = () => {
  useEffect(() => {
    if (didAuthenticate) return

    storeApi.authenticate()

    didAuthenticate = true
  }, [])

  // ...
}
```
方式三：
```javascript
if (typeof window !== 'undefined') {
  storeApi.authenticate()
}

const Store = () => {
  // ...
}
```
### 5.没必要在useEffect请求数据
常见写法
```javascript
const Store = () => {
  const [items, setItems] = useState([])

  useEffect(() => {
    let isCanceled = false

    getItems().then((data) => {
      if (isCanceled) return

      setItems(data)
    })

    return () => {
      isCanceled = true
    }
  })

  // ...
}
```
更好的方式：<br />没有必要使用useEffect，可以使用swr:
```javascript
import useSWR from 'swr'

export default function Page() {
  const { data, error } = useSWR('/api/data', fetcher)

  if (error) return <div>failed to load</div>
  if (!data) return <div>loading...</div>

  return <div>hello {data}!</div>
}
```
使用react-query:
```javascript
import { getItems } from './storeApi'
import { useQuery, useQueryClient } from 'react-query'

const Store = () => {
  const queryClient = useQueryClient()

  return (
    <button
      onClick={() => {
        queryClient.prefetchQuery('items', getItems)
      }}
    >
      See items
    </button>
  )
}

const Items = () => {
  const { data, isLoading, isError } = useQuery('items', getItems)

  // ...
}
```
没有正式发布的react的 [use函数](https://github.com/acdlite/rfcs/blob/first-class-promises/text/0000-first-class-support-for-promises.md#usepromise)：
```javascript
function Note({ id }) {
  const note = use(fetchNote(id))
  
  return (
    <div>
      <h1>{note.title}</h1>
      <section>{note.body}</section>
    </div>
  )
}
```
# reference
[https://www.youtube.com/watch?v=bGzanfKVFeU](https://www.youtube.com/watch?v=bGzanfKVFeU)
