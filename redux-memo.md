#Redux私人备忘录

- action
  - [普通的actin](#普通的actin)
  - [使用了redux-thunk以后的action](#使用了redux-thunk以后的action)
    - [可以加入异步请求](#可以加入异步请求)
    - [使用方法](#使用方法)
- [搭配React](#搭配react)
  - [React和Redux的连接react-redux](#React和Redux的连接react-redux)



##Action
####普通的actin
```javascript
export function Foo(value) {
  return {
    type: SOME_NAME,
    payload: value
  }
}
```
####使用了redux-thunk以后的action
```javascript
export function Foo() {
  return (dispatch, getState) => {
    const { counter } = getState()

    if (counter % 2 === 0) {
      return
    }

    dispatch(increment())
  }
}
```
####可以加入异步请求
```javascript
export function fetchPosts(subreddit) {

  // Thunk middleware 知道如何处理函数。
  // 这里把 dispatch 方法通过参数的形式传给函数，
  // 以此来让它自己也能 dispatch action。

  return function (dispatch) {

    // 首次 dispatch：更新应用的 state 来通知
    // API 请求发起了。

    dispatch(requestPosts(subreddit))

    // thunk middleware 调用的函数可以有返回值，
    // 它会被当作 dispatch 方法的返回值传递。

    // 这个案例中，我们返回一个等待处理的 promise。
    // 这并不是 redux middleware 所必须的，但这对于我们而言很方便。

    return fetch(`http://www.subreddit.com/r/${subreddit}.json`)
      .then(response => response.json())
      .then(json =>

        // 可以多次 dispatch！
        // 这里，使用 API 请求结果来更新应用的 state。

        dispatch(receivePosts(subreddit, json))
      )

      // 在实际应用中，还需要
      // 捕获网络请求的异常。
  }
}
```
####使用方法：
redux-thunk是redux的一个middleware
```bash
npm install --save redux-thunk
```
```javascript
import { createStore, applyMiddleware } from 'redux';
import thunk from 'redux-thunk';
import rootReducer from './reducers/index';

// Note: this API requires redux@>=3.1.0
const store = createStore(
  rootReducer,
  applyMiddleware(thunk)
);
```
更多可以参考<http://cn.redux.js.org/docs/advanced/AsyncActions.html>

##搭配React
```javascript
import { connect } from 'react-redux'
import { toggleTodo } from '../actions'
import TodoList from '../components/TodoList'

const getVisibleTodos = (todos, filter) => {
  switch (filter) {
    case 'SHOW_ALL':
      return todos
    case 'SHOW_COMPLETED':
      return todos.filter(t => t.completed)
    case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
  }
}

const mapStateToProps = (state) => {
  return {
    todos: getVisibleTodos(state.todos, state.visibilityFilter)
  }
}

const mapDispatchToProps = (dispatch) => {
  return {
    onTodoClick: (id) => {
      dispatch(toggleTodo(id))
    }
  }
}

const VisibleTodoList = connect(
  mapStateToProps,
  mapDispatchToProps
)(TodoList)

export default VisibleTodoList
```
####[React和Redux的连接react-redux](https://leozdgao.me/reacthe-reduxde-qiao-jie-react-redux/)
