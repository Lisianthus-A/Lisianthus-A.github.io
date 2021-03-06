---
sidebarDepth: 3
---
# React
## 使用 Hooks API
### useState
`useState` 用于在组件中保存状态。

用法：
``` js
const [state, setState] = useState(0);
```
返回值是一个数组，分别是当前 `state` 和 改变 `state` 的方法，调用 `setState` 后，组件会重新渲染。

与 `class` 组件的 `this.setState` 不同，`useState` 返回的 `setState` 方法不会合并对象：
``` js
const [state, setState] = useState({ a: 1, b: 2 });
setState({ a: 2 });  //state 会变成 { a: 2 } 而不是 { a: 2, b: 2 }
```

### useEffect
`useEffect` 会在浏览器渲染结束后运行，通常用于执行副作用函数，例如异步获取数据、改变 DOM 等。

在依赖项发生改变时，会先执行消除副作用的函数，再执行副作用函数。

用法：
``` js
const [value, setValue] = useState(0);

useEffect(() => {
    const id = setInterval(() => console.log(value), 1000);
    return () => clearInterval(id);
}, [value]);
```

第一个参数是副作用函数，函数的返回值是用于消除副作用的函数；

第二个参数是数组，里面是 `useEffect` 的依赖项，当依赖项发生改变时，`useEffect` 中的副作用函数会再次运行。

> 可以在一个组件中使用多个 `useEffect` 方法。

### useLayoutEffect
`useLayoutEffect` 在浏览器渲染结束前执行，其余与 `useEffect` 相同。

### useContext
用来保存应用的全局数据，或者避免在嵌套组件中多次传递 `props`。

用法：
``` js
export const CountContext = React.createContext();  //写在外部，可以被外界组件引入

const Parent = () => {
    const [count, setCount] = useState(0);  //使用 state 控制 Context 的值

    return (
        <CountContext.Provider value={count}>
            <div>
                <Child />
                <button onClick={() => setState(count  + 1)}>click</button>
            </div>
        </CountContext.Provider>
    );
}

const Child = () => {
    const count = useContext(CountContext);
    
    return (
        <div>count = {count}</div>
    );
}
```
> 使用了 `useContext` 的组件在 `Context` 更新时会强制重新渲染，无视 `React.memo`。

> 要避免这个问题，可以在使用 `useContext` 的组件上面包一层父组件，由父组件使用 `props` 传递 `Context` 的值。

### useCallback
用于保存已定义函数的引用，而不是重新定义一个函数。

用法：
``` js
const handleClick = useCallback(() => {
    //do something
}, []);
```
第一个参数用于执行的函数；

第二个参数是数组，里面是 `useCallback` 的依赖项，依赖项没有发生改变时，`handleClick` 总是指向同一个函数。

### useMemo
用于保存变量的值，避免重复进行繁重的计算。

用法：
``` js
const id = useMemo(() => {
    let result = null;
    //繁重的计算
    return result;
}, []);
```
第一个参数用于执行的函数；

第二个参数是数组，里面是 `useMemo` 的依赖项，依赖项没有发生改变时，直接返回之前的计算结果，不会再次执行函数。

### useRef
保存对象的引用，可以用于操作真实 DOM 等。

用法：
``` js
const Demo = () => {
    const divRef = useRef(null);

    return (
        <div ref={divRef} />
    );
}
```

要获取函数组件的 `ref` 时，需要结合 `React.forwardRef` 使用：
``` js
const Parent = () => {
    const childRef = useRef(null);

    useEffect(() => {
        const div = childRef.current;
        console.log(div.classList[0]);  //child
    }, []);

    return (
        <div>
            <Child ref={childRef} />
        </div>
    );
}

const Child = React.forwardRef((props, ref) => {
    return (
        <div className="child" ref={ref} />
    );
});
```

### useReducer
比较少用，以后再补充。

### useImperativeHandle
没用过，以后再补充。

### useDebugValue
没用过，以后再补充。

## 自定义 Hook
### useSetState
由于官方 API `useState` 不会合并对象，可以写一个支持合并对象的 `useSetState` 方法：
``` js
//定义
const useSetState = (initialState = {}) => {
    const [state, saveState] = useState(initialState);
    const setState = useCallback((newState) => {
        saveState(prevState => ({ ...prevState, ...newState }));
    }, []);
    return [state, setState];
}

const [state, setState] = useSetState({ a: 1, b: 2 });
setState({ a: 2 });  //state 变为 { a: 2, b: 2}
```

### useInterval
将 `setInterval` 进行包装，自动清除旧的定时器，并且可以通过 `state` 来开始、暂停定时器。
``` js
//定义
const useInterval = (callback, delay) => {
    const savedCallback = useRef();

    // 保存新回调
    useEffect(() => {
        savedCallback.current = callback;
    });

    // 建立 interval
    useEffect(() => {
        function tick() {
            savedCallback.current();
        }
        if (delay !== null) {
            let id = setInterval(tick, delay);
            return () => clearInterval(id);
        }
    }, [delay]);
}

const [isRunning, setIsRunning] = useState(true);

//定时器启动
useInterval(() => {}, isRunning ? 1000 : null);
//暂停定时器
setIsRunning(false);
```

## React-Redux
> 好久没用了，把常用的 API 抄下来，方便以后要用的时候快速掌握。

### 基础
- store: 保存数据的容器，有以下 API：
    - getState(): 获取当前 `state`
    - dispatch(action): 发出 `action`
    - subscribe(listener): 订阅 `store` 的变化，返回取消该订阅的函数。每次 `dispatch(action)` 将运行一次 `listener` 函数
    - replaceReducer(nextReducer): 用 `nextReducer` 替换当前的 `reducer`
- state: `store` 中保存的数据
- action: 通知 `state` 改变，带有 `type` 字段的普通 JS 对象
- reducer `(state: S, action: A) => S`: 接收当前 `state` 和 `action`，根据 `action` 来返回下一个 `state`
- dispatch: 发出 `action`，调用方式：
    - `dispatch({ type: 'ACTION_NAME' })`
    - `dispatch(fn())`  这里 `fn` 为生成 `action` 的函数

### createStore (redux)
创建 `store` 的函数。

`createStore(reducer, preloadedState?, enhancer?)`
- reducer
- preloadedState: 初始状态
- enhancer: 中间件 `applyMiddleware(...middleware)`

### combineReducers (redux)
合并多个 `reducer`，返回合并后的 `reducer`。

`combineReducers(reducers)`
- reducers: 对象，各属性的值为 `reducer`

例子：
``` js
const a = (state = 0, action) => {
    switch(action.type) {
        case 'A':
            return state + 1;
        default: 
            return state;
    }
}
const b = (state = 0, action) => {
    switch(action.type) {
        case 'B':
            return state + 1;
        default: 
            return state;
    }
}

const reducer = combineReducers({ a, b });  //合并 reducer
const store = createStore(reducer);  //创建 store
store.getState();  //{ a: 0, b: 0 }

store.dispatch({ type: 'A' });  //发送 action
store.getState();  //{ a: 1, b: 0 }
```

### Provider (react-redux)
传递 `store`。

``` jsx
<Provider store={store}>
    <App />
</Provider>
```

### connect (react-redux)
将组件与 `store` 关联起来，返回关联后的组件。

`connect(mapStateToProps?, mapDispatchToProps?, mergeProps?, options?)`
- mapStateToProps `(state, ownProps?) => Object`: 将 `state` 中的某些值映射到组件的 `props`
- mapDispatchToProps `Object | (dispatch, ownProps?) => Object`: 包装 `dispatch`，映射到组件的 `props`
- mergeProps `(stateProps, dispatchProps, ownProps) => props`: 将 `mapStateToProps` 和 `mapDispatchToProps` 提供的 `props` 进行包装，返回组件最终会收到的 `props`
- options `Object`: 好像比较少用

### 例子
``` jsx
//  index.js
import React from 'react';
import { createStore } from 'redux';
import { Provider } from 'react-redux';
import Counter from './Counter';

const reducer = (state = 0, action) => {
    switch (action.type) {
        case 'ADD':
            return state + 1;
        default:
            return state;
    }
}
const store = createStore(reducer);

const App = () => {
    <Provider store={store}>
        <Counter />
    </Provider>
}

//  Counter/index.js
import React from 'react';
import { connect } from 'react-redux';

const Counter = ({ count, onClick: handleClick }) => (
    <button onClick={handleClick}>count: {count}</button>
);

const mapStateToProps = (state, ownProps) => ({
    count: state 
});
const mapDispatchToProps = (dispatch, ownProps) => ({
    onClick: () => dispatch({ type: 'ADD' })
});

export default connect(mapStateToProps, mapDispatchToProps)(Counter);
```

## DvaJS

### 用法
创建一个 Dva 应用：
``` js
import UIComponent from './UIComponent';  //使用过 connect 的组件
import dva, { connect } from 'dva';

const app = dva();

const modelObject = {
    namespace: 'counter',
    state: 0,
    reducers: {
        add(state, action) {
            return state + action.payload;
        }
    },
    effects: {
        *asyncAdd5(action, { call }) {
            const five = yield call(() => new Promise(resolve => {
                setTimeout(resolve, 1000, 5);
            }));
            yield put({ type: 'add', payload: five });
        }
    }
};

app.model(modelObject);  //注册 model

app.router((history, app) => <UIComponent />);  //路由

app.start('#root');  //挂载到 id = root 的 DOM 元素中
```

### model 对象
Dva 是通过 `model` 来管理状态的，`model` 的一些配置： 
- namespace: `model` 的命名空间，同时也是全局 `state` 上的属性
- state: 初始值，优先级低于传给` dva(options)` 的 `options.initialState`
- reducers `(state, action) => newState`:  根据 `action` 来返回新的 `state`
- effects `*(action, effects) => void`: generator 函数，处理异步操作和业务逻辑，不直接修改 `state`
    - `effects.put(action)`: 发出 `action`
    - `effects.call(fn, params)`: 调用异步函数 `fn(params)`，取得函数返回值
    - `effects.select(fn = (state) => any)`: 调用函数，取得函数返回值。`fn` 接收当前全局 `state`
- subscriptions: 用于订阅一个数据源，然后根据需要 `dispatch` 相应的 `action`，暂时没用过