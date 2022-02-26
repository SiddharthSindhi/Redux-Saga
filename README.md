# Redux-Saga

# Dispatching an Action

Suppose we have a UI to fetch some user data from a remote server when a button is clicked. (For brevity, we'll just show the action triggering code.)

````
class UserComponent extends React.Component {
  ...
  onSomeButtonClicked() {
    const { userId, dispatch } = this.props
    dispatch({type: 'USER_FETCH_REQUESTED', payload: {userId}})
  }
  ...
}
````

# Initiate a side effect

The component dispatches a plain object action to the store. We'll create a Saga that watches for all USER_FETCH_REQUESTED actions and triggers an API call to fetch the user data.

````
import { call, put, takeEvery, takeLatest } from 'redux-saga/effects'
import Api from '...'

// Worker saga will be fired on USER_FETCH_REQUESTED actions
function* fetchUser(action) {
   try {
      const user = yield call(Api.fetchUser, action.payload.userId);
      yield put({type: "USER_FETCH_SUCCEEDED", user: user});
   } catch (e) {
      yield put({type: "USER_FETCH_FAILED", message: e.message});
   }
}

// Starts fetchUser on each dispatched USER_FETCH_REQUESTED action
// Allows concurrent fetches of user
function* mySaga() {
  yield takeEvery("USER_FETCH_REQUESTED", fetchUser);
}
````

# Connect to store

To run our Saga, we have to connect it to the Redux store using the redux-saga middleware.

````
import { createStore, applyMiddleware } from 'redux'
import createSagaMiddleware from 'redux-saga'

import reducer from './reducers'
import mySaga from './sagas'

// Create the saga middleware
const sagaMiddleware = createSagaMiddleware()
// Mount it on the Store
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)

// Then run the saga
sagaMiddleware.run(mySaga)

// Render the application
````

