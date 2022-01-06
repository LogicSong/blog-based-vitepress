---
date: 2022-01-06 19:25
title: redux系列二：redux中间件实现原理
tags:
  - react
  - redux
describe: 中间件是redux的精华部分,让我们看看它的精妙原理
---

## redux中间件原理

### redux中间件中使用的聚合函数(Compose)

先看几个函数

```js
function f1(args) {
    console.log('fn1:',args)
}
function f2(args) {
    console.log('fn2:',args)
}
function f3(args) {
    console.log('fn3:',args)
}
```

如何执行这三个函数?

```js
fn1('args')
fn2('args')
fn3('args')
```

这样很啰嗦,优化一下

```js
// 先将函数改写一下,使其返回改写后的参数
function f1(args) {
    args = args + 1;
    console.log('fn1:',args);
    return args;
}
function f2(args) {
    args = args * 2;
    console.log('fn2:',args);
    return args;
}
function f3(args) {
    args = args / 3;
    console.log('fn3:',args);
    return args;
}
```

现在可以这样调用:

```js
f1(f2(f3(1)));
```

还是不够友好,能否将三个函数聚合成一个函数,然后直接调用聚合后的函数即可?

可以!下面就是聚合函数的实现,redux中的中间件就是用compose函数聚合的.

```js
function compose(funcArray: Function[]): Function {
    if (funcArray.length === 0) {
        return args => args
    }
    if (funcArray.length === 1) {
        return funcArray[0]
    }
    return funcArray.reduce((res, cur) => (...args) => res(cur(...args)))
}
```

### redux中间件实现原理

#### 先看applyMiddleWare

使用

```js
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import reducers from './reducers';

let store = createStore(reducers, applyMiddleware(thunk));
```

首先明确一点:applyMiddleware在创建store时就调用了,看看源码它做了什么,返回了什么.

```js
// 核心源码
// 接受多个中间件(函数),返回一个函数
applyMiddleware(
  ...middlewares: Middleware[]
): StoreEnhancer<any> {
    // 返回一个函数,该函数的参数是createStore,返回值是改写原store
  return (createStore: StoreEnhancerStoreCreator) =>
    <S, A extends AnyAction>(
      reducer: Reducer<S, A>,
      preloadedState?: PreloadedState<S>
    ) => {
      const store = createStore(reducer, preloadedState)
      let dispatch: Dispatch = () => {
        throw new Error(
          'Dispatching while constructing your middleware is not allowed. ' +
            'Other middleware would not be applied to this dispatch.'
        )
      }

      const middlewareAPI: MiddlewareAPI = {
        getState: store.getState,
        // dispatch
        dispatch: (action, ...args) => dispatch(action, ...args)
      }
      // 中间件的第一层函数是在这里调用的
      const chain = middlewares.map(middleware => middleware(middlewareAPI))
      // 使用compose进行聚合
      dispatch = compose<typeof dispatch>(...chain)(store.dispatch)
    // 可以看到,返回值与原store唯一的区别就是改写(增强)了dispatch,这便是applyMiddleWare的作用
    // 也是中间件最终的目的
      return {
        ...store,
        dispatch
      }
    }
}
```

##### 总结一下applyMiddle

- 首先,它是一个三箭头函数,接受多个中间件(函数),返回一个双箭头函数
- 最下层的返回值函数,它的参数是createStore,它的返回值是改写原store
- 最下层返回值函数的返回值,实际上就是用createStore创建原store,然后通过中间件将dispatch增强,然后再将新的store作为返回值
- 中间件的第一层函数是在这里调用的,并将调用返回的函数用compose函数聚合

#### createStore做了什么

除了上文说的,源码中的createStore实际上首先看入参里有没有enhancer(applyMiddleware的执行结果),如果有则执行enhancer:
```js
if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error(
        `Expected the enhancer to be a function. Instead, received: '${kindOf(
          enhancer
        )}'`
      )
    }

    return enhancer(createStore)(
      reducer,
      preloadedState as PreloadedState<S>
    ) as Store<ExtendState<S, StateExt>, A, StateExt, Ext> & Ext
  }
```

### MiddleWare

#### MiddleWare长什么样?

```js
// Middleware written as ES5 functions
// Outer function:
function exampleMiddleware(storeAPI) {
  return function wrapDispatch(next) {
    return function handleAction(action) {
      // Do anything here: pass the action onwards with next(action),
      // or restart the pipeline with storeAPI.dispatch(action)
      // Can also use storeAPI.getState() here

      return next(action)
    }
  }
}
```

- 最外层就是middleware本身,它在applyMiddleware中被调用`const chain = middlewares.map(middleware => middleware(middlewareAPI))`,当我们在业务代码中调用dispatch时就会将action派发到第一个middleware.只执行一遍
- wrapDispatch:(官方文档中写This function is actually the next middleware in the pipeline.)该函数也是在applyMiddleware中调用`dispatch = compose<typeof dispatch>(...chain)(store.dispatch)`,最后一个middleware中的next就是strore.dispatch.只执行一遍.
- handleAction: 每次dispatch都会触发,接受action参数.

#### 动手写一个logger中间件

```js
const loggerMiddleware = storeAPI => next => action => {
  console.log('dispatching', action)
  let result = next(action)
  console.log('next state', storeAPI.getState())
  return result
}
```