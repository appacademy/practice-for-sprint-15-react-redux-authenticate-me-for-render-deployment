# Phase 1: Login form page

The Login Form Page is the first page that you will add to your frontend
application.

## Session actions and reducer

First, you will add the Redux store actions and reducers that you need for this
feature. You will use the `POST /api/session` backend route to login in a user
as well as add the session user's information to the frontend Redux store.

Make a file called `session.js` in the `frontend/src/store` folder. This file
will contain all the actions specific to the session user's information and the
session user's Redux reducer.

In this file, add a `session` reducer that will hold the current session user's
information. The `session` slice of state should look like this if there is a
current session user:

```js
{
  user: {
    id,
    email,
    username,
    firstName,
    lastName,
    createdAt,
    updatedAt
  }
}
```

If there is no session user, then the `session` slice of state should look like
this:

```js
{
  user: null
}
```

By default, there should be no session user in the `session` slice of state.

Create two POJO action creators. One that will set the session user in the
`session` slice of state to the action creator's input parameter, and another
that will remove the session user. Their types should be extracted as a
constant and used by the action creator and the `session` reducer.

You need to call the API to login then set the session user from the response,
so add a thunk action for the `POST /api/session`. Make sure to use the custom
`csrfFetch` function from `frontend/src/store/csrf.js`. The `POST /api/session`
route expects the request body to have a key of `credential` with an existing
username or email and a key of `password`. After the response from the AJAX call
comes back, parse the JSON body of the response, and dispatch the action for
setting the session user to the user in the response's body.

Export the login thunk action, and export the reducer as the default export.

Import the reducer in `session.js` into the file with the root reducer,
`frontend/src/store/index.js`.

Set a key of `session` in the `rootReducer`'s `combineReducer` object argument
to the session reducer.

### Test the session actions and reducer

Login should be working so give it a try! Test the login thunk action and the
`session` reducer.

Import all the actions from the `session.js` file into the frontend application
entry file, `frontend/src/index.js`. Then attach the actions to the `window`
at the key of `sessionActions`:

```js
// frontend/src/index.js
// ... other imports
import * as sessionActions from './store/session';

const store = configureStore();

if (process.env.NODE_ENV !== 'production') {
  restoreCSRF();

  window.csrfFetch = csrfFetch;
  window.store = store;
  window.sessionActions = sessionActions;
}
// ...
```

Navigate to [http://localhost:3000] and in the browser's dev tools console, try
dispatching the login thunk action with the demo user login credentials.

The `previous state` in the console should look like this:

```js
{
  session: {
    user: null
  }
}
```

The `next state` in the console should look something like this:

```js
{
  session: {
    user: {
      createdAt: "<Some date time format>",
      email: "demo@user.io",
      id: 1,
      updatedAt: "<Some date time format>",
      username: "Demo-lition",
      firstName: "Demo",
      lastName: "Lition",
    }
  }
}
```

If there is an error or if the previous or next state does not look like this,
then check your logic in your session reducer and your actions. If you are
still having issues, scroll to check the example code below. 

After you finished testing, **commit your code**.

### Example session actions and reducer

There is no absolute "right" way of doing this. As long as your `session`
actions and reducers are displaying the expected initial state and states after
each dispatched action, then your setup is fine.

Here's an example for the `session` actions and reducer:

```js
// frontend/src/store/session.js
import { csrfFetch } from './csrf';

const SET_USER = 'session/setUser';
const REMOVE_USER = 'session/removeUser';

const setUser = (user) => {
  return {
    type: SET_USER,
    payload: user,
  };
};

const removeUser = () => {
  return {
    type: REMOVE_USER,
  };
};

export const login = (user) => async (dispatch) => {
  const { credential, password } = user;
  const response = await csrfFetch('/api/session', {
    method: 'POST',
    body: JSON.stringify({
      credential,
      password,
    }),
  });
  const data = await response.json();
  dispatch(setUser(data.user));
  return response;
};

const initialState = { user: null };

const sessionReducer = (state = initialState, action) => {
  let newState;
  switch (action.type) {
    case SET_USER:
      newState = Object.assign({}, state);
      newState.user = action.payload;
      return newState;
    case REMOVE_USER:
      newState = Object.assign({}, state);
      newState.user = null;
      return newState;
    default:
      return state;
  }
};

export default sessionReducer;
```

Here's an example for the `rootReducer` setup:

```js
// frontend/src/store/index.js
// ...
import sessionReducer from './session';

const rootReducer = combineReducers({
  session: sessionReducer,
});
// ...
```

Here's an example for the login thunk action test in the browser's dev tools
console:

```js
window.store.dispatch(window.sessionActions.login({
  credential: 'Demo-lition',
  password: 'password'
}));
```

## `LoginFormPage` component

After finishing the Redux actions and the reducer for the login feature, the
React components are next.

Create a `components` folder in the `frontend/src` folder. This is where all
your components besides `App` will live.

Make a folder called `LoginFormPage` nested in the new `components` folder which
will hold all the files for the login form. Add an `index.js` file in the
`LoginFormPage`. Inside of this file, add a React functional component named
`LoginFormPage`.

Render a form with a controlled input for the user login credential (username or
email) and a controlled input for the user password.

On submit of the form, dispatch the login thunk action with the form input
values. Make sure to handle and display errors from the login thunk action
if there are any.

Export the `LoginFormPage` component at the bottom of the file, then render it
in `App.js` at the `"/login"` route.

If there is a current session user in the Redux store, then redirect the user
to the `"/"` path if trying to access the `LoginFormPage`.

Test your component by navigating to the `"/login"` page. Try logging into the
form there with the demo user's credentials. Once you login, you should be
redirected to the `"/"` route. Check your code for the `LoginFormPage` and the
`App` component if this is not the flow that you are experiencing.

Also try logging in with invalid fields to test your handling and displaying of
error messages.

After testing, **commit your `LoginFormPage` code**!

### Example `LoginFormPage` component

Again, there is no absolute "right" way of doing this. As long as your React
application is behaving as expected, then you don't need to make your code look
exactly like the example code.

Here's an example for `LoginFormPage` component:

```js
// frontend/src/components/LoginFormPage/index.js
import React, { useState } from 'react';
import * as sessionActions from '../../store/session';
import { useDispatch, useSelector } from 'react-redux';
import { Redirect } from 'react-router-dom';

function LoginFormPage() {
  const dispatch = useDispatch();
  const sessionUser = useSelector(state => state.session.user);
  const [credential, setCredential] = useState('');
  const [password, setPassword] = useState('');
  const [errors, setErrors] = useState([]);

  if (sessionUser) return (
    <Redirect to="/" />
  );

  const handleSubmit = (e) => {
    e.preventDefault();
    setErrors([]);
    return dispatch(sessionActions.login({ credential, password }))
      .catch(async (res) => {
        const data = await res.json();
        if (data && data.errors) setErrors(data.errors);
      });
  }

  return (
    <form onSubmit={handleSubmit}>
      <ul>
        {errors.map((error, idx) => <li key={idx}>{error}</li>)}
      </ul>
      <label>
        Username or Email
        <input
          type="text"
          value={credential}
          onChange={(e) => setCredential(e.target.value)}
          required
        />
      </label>
      <label>
        Password
        <input
          type="password"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
      </label>
      <button type="submit">Log In</button>
    </form>
  );
}

export default LoginFormPage;
```

Here's an example for how `App.js` should look like now:

```js
// frontend/src/App.js
import React from 'react';
import { Route, Switch } from 'react-router-dom';
import LoginFormPage from './components/LoginFormPage';

function App() {
  return (
    <Switch>
      <Route path="/login">
        <LoginFormPage />
      </Route>
    </Switch>
  );
}

export default App;
```

### `LoginForm` CSS

Add a `LoginForm.css` file in your `LoginFormPage` folder. Import this CSS
file into the `frontend/src/components/LoginFormPage/index.js` file.

```js
// frontend/src/components/LoginFormPage/index.js
// ...
import './LoginForm.css';
// ...
```

Define all your CSS styling rules for the `LoginFormPage` component in the
`LoginForm.css` file. Practice doing some CSS now to make your login page
look better. Make sure to **commit your code afterwards**!

# Restore the session user

Right now, if you login successfully, you get redirected to the `"/"` route. If
you refresh at that `"/"` page and navigate to the `"/login"` page, then you
will not be redirected because the store does not retain the session user
information on a refresh. How do you retain the session user information
across a refresh? By loading the application after accessing the route to
get the current session user `GET /api/session` and adding the user info to the
Redux store again.

Add a thunk action in `frontend/src/store/session.js` that will call the
`GET /api/session`, parse the JSON body of the response, and dispatch the action
for setting the session user to the user in the response's body.

Test your thunk action by logging in then refreshing at the
[http://localhost:3000] route. Make sure you have a `token` in your cookies. In
the browser's dev tools console, try dispatching the restore session user
thunk action.

The `previous state` in the console should look like this:

```js
{
  session: {
    user: null
  }
}
```

The `next state` in the console should look something like this:

```js
{
  session: {
    user: {
      createdAt: "<Some date time format>",
      email: "demo@user.io",
      id: 1,
      updatedAt: "<Some date time format>",
      username: "Demo-lition",
      firstName: "Demo",
      lastName: "Lition",
    }
  }
}
```

If you don't see this behavior, then check your syntax for the restore user
thunk action.

After you test it to see if it works, then use this thunk action inside of
`App.js` after the `App` component's first render.

**Commit after testing!**

## Example restore session user thunk action

Again, there is no absolute "right" way of doing this. As long as your React
application is behaving as expected, then you don't need to make your code look
exactly like the example code.

Here's an example of the restore session user thunk action:

```js
// frontend/src/store/session.js
// ...
export const restoreUser = () => async dispatch => {
  const response = await csrfFetch('/api/session');
  const data = await response.json();
  dispatch(setUser(data.user));
  return response;
};
// ...
```

Here's an example of how to test the `restoreUser` thunk action:

```js
window.store.dispatch(window.sessionActions.restoreUser());
```

Here's an example for how `App.js` could look like now:

```js
// frontend/src/App.js
import React, { useState, useEffect } from "react";
import { useDispatch } from "react-redux";
import { Route, Switch } from "react-router-dom";
import LoginFormPage from "./components/LoginFormPage";
import * as sessionActions from "./store/session";

function App() {
  const dispatch = useDispatch();
  const [isLoaded, setIsLoaded] = useState(false);
  useEffect(() => {
    dispatch(sessionActions.restoreUser()).then(() => setIsLoaded(true));
  }, [dispatch]);

  return isLoaded && (
    <Switch>
      <Route path="/login">
        <LoginFormPage />
      </Route>
    </Switch>
  );
}

export default App;
```

[test-redux-store-image]: https://appacademy-open-assets.s3-us-west-1.amazonaws.com/Modular-Curriculum/content/react-redux/topics/react-redux-auth/authenticate-me/assets/test-redux-store-setup.png
[Font Awesome]: https://fontawesome.com/start
[Choose a Font Awesome Icon]: https://fontawesome.com/icons?d=gallery&m=free
[carrot icon]: https://fontawesome.com/icons/carrot?style=solid
[Portals in React]: https://reactjs.org/docs/portals.html
[http://localhost:3000]: http://localhost:3000