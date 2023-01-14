现实很多项目存在大量的useMemo和useCallback，大多数的使用并没有起到实际作用，使得项目的渲染效率更低了。<br />我们在应用程序中使用useMemo和useCallback这两个Hook，主要是因为想要缓存结果：

1. 缓存props值，防止重复渲染。
2. 缓存复杂的计算，避免重复计算。

**所以我们经常这样使用这两个Hook：**
## 缓存值
useMemo 缓存value，防止重新渲染。以下是伪代码：
```javascript
const AppItem = ({item, data}) => <button>{item.val + newDate}</button>

const App = ({data}) => {
  const newDate = useMemo(() => data.value, [data.value])
  
  return (
    <>
      {
        list.map(item => {
          return <AppItem key={item.id} item={item} data={newDate} />
        })
      }
    </>
  )
}
```
## 缓存函数
useCallback缓存click 事件，防止重复渲染。以下是伪代码：
```javascript
const AppItem = ({item, data, onClick}) => <button onClick={onClick}>{item.val + newDate}</button>

const App = ({data}) => {
  const newDate = useMemo(() => data.value, [data.value])

  const onClick = useCallback(() => {
    // ...
  }, [])
  
  return (
    <>
      {
        list.map(item => {
          return <AppItem key={item.id} item={item} data={newDate} onClick={onClick} />
        })
      }
    </>
  )
}
```
**上述是我们经常见到的场景。但是这样使用就一定是正确的吗？答案是否定的**。之前写过一篇React为什么会重新渲染的文章。这里在解释下。
## 组件为什么会重新渲染
重新渲染其中一个原因就是：state 或者 props 发生变化时。所以我们很天真的认为只要state或者props不变，组件就不会重新渲染了。<br />组件的重新渲染还有一个原因，我们知道但是经常在我们写代码的时候会忽略的原因：就是他的父组件重新渲染了。父组件渲染了，我们只在子组件内部缓存值或者函数是没有作用的。<br />看一个例子：
```javascript
const App = () => {
	const [count, setCount] = useState(0)
  
  return (
    <>
      <button onClick={() => setCount(count+1)}>点我</button>
      <OtherComp />
    </>
  )
}
```
我们可以看到：点击按钮，App组件重新渲染，他的子组件OtherComp虽然没有任何的state或者props变化，但是他也重新渲染了。如果这个子组件也有子组件，以此类推，就会形成一条渲染链。<br />但是我们不是经常这样写吗：在这个组件中使用缓存手段。
```javascript
const App = ({val}) => {
	const [count, setCount] = useState(0)

  const onClick = useCallback(() => {
    // ...
  }, [])

  const data = useMemo(() => val, [val])
  
  return (
    <>
      <button onClick={() => setCount(count+1)}>点我</button>
      <OtherComp onClick={onClick} data={data} />
    </>
  )
}
```
这样并不会阻止子组件的重新渲染。怎样解决呢？当然是还要缓存子组件了。
```javascript
const OtherCompMemo = React.memo(OtherComp)

const App = ({val}) => {
	const [count, setCount] = useState(0)

  const onClick = useCallback(() => {
    // ...
  }, [])

  const data = useMemo(() => val, [val])
  
  return (
    <>
      <button onClick={() => setCount(count+1)}>点我</button>
      <OtherCompMemo onClick={onClick} data={data} />
    </>
  )
}
```
现在React会识别props没有变化，onClick 和 data也被缓存了，所以需要配合使用。
## 缓存复杂的计算
根据React文档：useMemo是用来缓存复杂计算的。假设有一个100000项的数组需要排序操作，此时我们应该缓存复杂的计算：
```javascript
const dataList = ({data}) => {

  const contentNode = useMemo(() => {
    return data.map(item => {
        return <div key={item}>{item}</div>
      })
  }, [data, sort]);

  return contentNode;
}
```
## 什么是复杂的计算

1. 你可以通过 preformance.now() 进行计算消耗的时间。
2. React官方的开发工具中的profiler 记录查看，记录里会显示复杂的计算。

![image.png](https://cdn.nlark.com/yuque/0/2023/png/2180858/1673622218453-eb84f5fb-4fce-4ab5-a189-4a8fee0784b4.png#averageHue=%23fcfcfc&clientId=u889b6d1b-3573-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=147&id=u9eb61df3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=294&originWidth=1552&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28096&status=done&style=stroke&taskId=u7922d556-ff5b-409c-897c-5083b06ab81&title=&width=776)
## 何时进行优化呢

1. 如果你可以通过组织代码结构来提高性能，那就没有必要使用useCallback和useMemo。
2. 如果你不知道使用useCallback和useMemo能否带来更大的好处，就不需要使用它们，因为使用它们也需要消耗性能。
## 总结

- useMemo和useCallback只是针对重新渲染才是有帮助的，对第一次的渲染是有害的，消耗性能的。
- 大多数情况下，单独使用useMemo和useCallback或memo是没有帮助的，需要结合父组件具体情况来看。
- 其实大多数情况下，我们并不需要这两个hook，使用它们只会影响初始化的渲染。

