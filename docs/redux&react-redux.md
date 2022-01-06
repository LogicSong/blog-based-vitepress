---
date: 2022-01-05 20:05
title: redux系列一：redux与react-redux主要API
tags:
  - react
  - redux
describe: redux与react-redux核心API的实现原理
---

## redux的实现

### 为什么需要redux，它解决了什么问题

React作为一个组件化开发框架，组件之间存在大量通信，有时这些通信跨越多个组件，或者多个组件之间共享一套数据，简单的父子组件间传值不能满足我们的需求，自然而然地，我们需要有一个地方存取和操作这些公共状态。而redux就为我们提供了一种管理公共状态的方案。

### redux的简单实现

```js
//store.js
import { reducer } from './reducer'
export const createStore = (reducer) => {        
    let currentState = {}        
    let observers = []             //观察者队列        
    function getState() {                
        return currentState        
    }        
    function dispatch(action) {                
        currentState = reducer(currentState, action)                
        observers.forEach(fn => fn())        
    }        
    function subscribe(fn) {                
        observers.push(fn)        
    }        
    dispatch({ type: '@@REDUX_INIT' })  //初始化store数据        
    return { getState, subscribe, dispatch }
}
```

## react-redux的实现

一个组件如果想从store存取公用状态，需要进行四步操作：import引入store、getState获取状态、dispatch修改状态、subscribe订阅更新，代码相对冗余，我们想要合并一些重复的操作，而react-redux就提供了一种合并操作的方案：react-redux提供Provider和connect两个API

### Provider的源码实现

Provider的核心是使用React Context API，将state注入到父组件，达到跨层级传递数据的效果。

```ts
function Provider<A extends Action = AnyAction>({
  store,
  context,
  children,
  serverState,
}: ProviderProps<A>) {
  const contextValue = useMemo(() => {
    const subscription = createSubscription(store)
    return {
      store,
      subscription,
      getServerState: serverState ? () => serverState : undefined,
    }
  }, [store, serverState])

  const previousState = useMemo(() => store.getState(), [store])

  useIsomorphicLayoutEffect(() => {
    const { subscription } = contextValue
    subscription.onStateChange = subscription.notifyNestedSubs
    subscription.trySubscribe()

    if (previousState !== store.getState()) {
      subscription.notifyNestedSubs()
    }
    return () => {
      subscription.tryUnsubscribe()
      subscription.onStateChange = undefined
    }
  }, [contextValue, previousState])

  const Context = context || ReactReduxContext

  // @ts-ignore 'AnyAction' is assignable to the constraint of type 'A', but 'A' could be instantiated with a different subtype
  return <Context.Provider value={contextValue}>{children}</Context.Provider>
}
```

### connect的简单实现

我们已经知道，connect接收mapStateToProps、mapDispatchToProps两个方法，然后返回一个高阶函数，这个高阶函数接收一个组件，返回一个高阶组件（其实就是给传入的组件增加一些属性和功能）connect根据传入的map，将state和dispatch(action)挂载子组件的props上：
```ts
export function connect(mapStateToProps, mapDispatchToProps) {    
    return function(Component) {      
        class Connect extends React.Component {        
            componentDidMount() {          
                //从context获取store并订阅更新          
                this.context.store.subscribe(this.handleStoreChange.bind(this));        
            }       
            handleStoreChange() {          
                // 触发更新          
                // 触发的方法有多种,这里为了简洁起见,直接forceUpdate强制更新,读者也可以通过setState来触发子组件更新          
                this.forceUpdate()        
            }        
            render() {          
                return (            
                    <Component              
                        // 传入该组件的props,需要由connect这个高阶组件原样传回原组件              
                        { ...this.props }              
                        // 根据mapStateToProps把state挂到this.props上              
                        { ...mapStateToProps(this.context.store.getState()) }               
                        // 根据mapDispatchToProps把dispatch(action)挂到this.props上              
                        { ...mapDispatchToProps(this.context.store.dispatch) }                 
                    />              
                )        
            }      
        }      
        //接收context的固定写法      
        Connect.contextTypes = {        
            store: PropTypes.object      
        }      
        return Connect    
    }
}
```