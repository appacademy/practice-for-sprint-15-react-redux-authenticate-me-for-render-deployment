# Phase 0: Choose your path

If you want to set up Redux from scratch, follow Method 1. Otherwise, you can
follow Method 2, which will allow you to set up Redux easily. (To go to Method
2, just search for it within this page.)

## Method 1: Set up Redux from scratch

Use the `create-react-app` command from inside your `frontend` folder to
initialize React inside of the `frontend` folder:

```bash
npx create-react-app . --template @appacademy/react-v17 --use-npm
```

## Dependencies

In the `frontend` folder, `npm install` the following packages as dependencies:

- `js-cookie` - extracts cookies
- `react-redux` - React components and hooks for Redux
- `react-router-dom@^5` - routing for React
- `redux` - Redux
- `redux-thunk` - add Redux thunk

`npm install -D` the following packages as dev-dependencies:

- `redux-logger` - log Redux actions in the browser's dev tools console

## Setting up the Redux store

First, setup your Redux store. Make a folder in `frontend/src` called `store`
and add an `index.js` file. In this file, import `createStore`,
`combineReducers`, `applyMiddleware`, and `compose` from the `redux` package.
Import `thunk` from `redux-thunk`.

```js
// frontend/src/store/index.js
import { createStore, combineReducers, applyMiddleware, compose } from 'redux';
import thunk from 'redux-thunk';
```

Create a `rootReducer` that calls `combineReducers` and pass in an empty object
for now.

```js
// frontend/src/store/index.js
// ...
const rootReducer = combineReducers({
});
```

Initialize an `enhancer` variable that will be set to different store enhancers
depending on if the Node environment is in development or production.

In production, the `enhancer` should only apply the `thunk` middleware.

In development, the `logger` middleware and Redux dev tools compose enhancer as
well. To use these tools, create a `logger` variable that uses the default
export of `redux-logger`.  Then, grab the Redux dev tools compose enhancer with
`window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__` and store it in a variable called
`composeEnhancers`. You can use an __or__ `||` to keep the Redux's original
`compose` as a fallback. Then set the `enhancer` variable to the return of the
`composeEnhancers` function passing in `applyMiddleware` invoked with `thunk`
then `logger`.

```js
// frontend/src/store/index.js
// ...

let enhancer;

if (process.env.NODE_ENV === 'production') {
  enhancer = applyMiddleware(thunk);
} else {
  const logger = require('redux-logger').default;
  const composeEnhancers =
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
  enhancer = composeEnhancers(applyMiddleware(thunk, logger));
}
```

Next, create a `configureStore` function that takes in an optional
`preloadedState`. Return `createStore` invoked with the `rootReducer`, the
`preloadedState`, and the `enhancer`.

```js
// frontend/src/store/index.js
// ...

const configureStore = (preloadedState) => {
  return createStore(rootReducer, preloadedState, enhancer);
};

export default configureStore;
```

Finally, export the `configureStore` function at the bottom of the file as the
default export. This function will be used by `index.js` to attach the Redux
store to the React application.

## Redux `Provider` and `BrowserRouter`

In your React application, you'll be using `BrowserRouter` from React Router for
routing and `Provider` from Redux to provide the Redux store. Import those
components as well as the `configureStore` function that you just wrote in
`frontend/src/store/index.js`.

Your imports should now look something like this:

```js
// frontend/src/index.js
import React from 'react';

import './index.css';

import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import App from './App';

import configureStore from './store';
```

Create a variable to access your store and expose it to the `window`. It should
not be exposed in production, be sure this is only set in development.

```js
// frontend/src/index.js
// ...
const store = configureStore();

if (process.env.NODE_ENV !== 'production') {
  window.store = store;
}
```

Next, define a `Root` React functional component that returns the `App`
component wrapped in Redux's `Provider` and React Router DOM's `BrowserRouter`
provider components.

```js
// frontend/src/index.js
// ...
function Root() {
  return (
    <Provider store={store}>
      <BrowserRouter>
        <App />
      </BrowserRouter>
    </Provider>
  );
}
```

Make sure to pass in the key of `store` with the value of `store` to the
`Provider`.

After defining the `Root` functional component, call `ReactDOM.render` function
passing in the `Root` component and the HTML element with the id of `"root"`.

```js
// frontend/src/index.js
// ...
ReactDOM.render(
  <React.StrictMode>
    <Root />
  </React.StrictMode>,
  document.getElementById('root'),
);
```

Congratulations, you have finished Method 1! You may now skip to the "Test the
Redux store setup" section below. (Just scroll through Method 2; it's not long.)

## Method 2: Use Redux template

Use the `create-react-app` command from inside your `frontend` folder to
initialize React inside of the `frontend` folder:

```bash
npx create-react-app . --template @appacademy/react-redux-v17 --use-npm
```

You will also need to install `js-cookie` as a dependency to continue. This
dependency will allow your frontend to extract cookies from the browser.

```sh
npm install js-cookie
```

## Test the Redux store setup

**From this point, Method 1 and Method 2 have the same instructions.**

Test your Redux store setup by starting your React frontend server (run
`npm start` in your `frontend` folder) and navigate to [http://localhost:3000].

Check to see if your Redux dev tools was successfully connected and if there is
a `store` on the `window` in your browser's dev tools console.

You can ignore the "Store does not have a valid reducer" error. This error is a
result of not passing in anything into the `rootReducer`'s `combineReducer`.

Try to dispatch an action from your browser's dev tools console. Make sure to
include a `type` key in the action that you dispatch.

```js
window.store.dispatch({ type: 'hello' });
```

![test-redux-store-image]

If you cannot dispatch an action or if you cannot see the action in the Redux
dev tools, check the syntax in your `frontend/src/store/index.js` and in your
`frontend/src/index.js`.

**Now is a good time to commit your initial set up!**

## Wrapping `fetch` requests with CSRF

Your Express backend server is configured to be CSRF protected and will only
accept requests that have the right CSRF secret token in a header and the right
CSRF token value in a cookie.

First, you need to add a `"proxy"` in your `frontend/package.json`. Add a
`"proxy"` key with the value of `http://localhost:8000` or wherever you are
serving your backend Express application. This proxy will force the frontend
server to act like it's being served from the backend server. So if you do a
`fetch` request in the React frontend like `fetch('/api/csrf/restore)`, then the
`GET /api/csrf/restore` request will be made to the backend server instead of
the frontend server.

Your `frontend/package.json`'s `"proxy"` key should like this:

```json
  "proxy": "http://localhost:8000"
```

**Remember to restart the frontend server after you make any edits to the
`package.json` file.**

Next, to make `fetch` requests with any HTTP verb other than `GET`, you need to
set a `XSRF-TOKEN` header on the request and the value of the header should be
set to the value of the `XSRF-TOKEN` cookie. To do this, you are going to wrap
the `fetch` function on the `window` that will be used in place of the default
`fetch` function.

Add a `csrf.js` file in the `frontend/src/store` folder. Import `Cookies` from
`js-cookie` that will be used to extract the `XSRF-TOKEN` cookie value. Define
an `async` function called `csrfFetch` that will take in a `url` parameter and
an `options` parameter that defaults to an empty object. If `options.headers` is
not set, default it to an empty object. If `options.method` is not set, set it
to the `GET` method. If it is any method other than a `GET` method, set the
`XSRF-TOKEN` header on the `options` object to the extracted value of the
`XSRF-TOKEN` cookie. Call and `await` the `window.fetch` with the `url` and the
`options` object to get the response.

If the response status code is 400 or above, `throw` the response as the error.
Otherwise, return the response.

```js
// frontend/src/store/csrf.js
import Cookies from 'js-cookie';

export async function csrfFetch(url, options = {}) {
  // set options.method to 'GET' if there is no method
  options.method = options.method || 'GET';
  // set options.headers to an empty object if there is no headers
  options.headers = options.headers || {};

  // if the options.method is not 'GET', then set the "Content-Type" header to
    // "application/json", and set the "XSRF-TOKEN" header to the value of the 
    // "XSRF-TOKEN" cookie
  if (options.method.toUpperCase() !== 'GET') {
    options.headers['Content-Type'] =
      options.headers['Content-Type'] || 'application/json';
    options.headers['XSRF-Token'] = Cookies.get('XSRF-TOKEN');
  }
  // call the default window's fetch with the url and the options passed in
  const res = await window.fetch(url, options);

  // if the response status code is 400 or above, then throw an error with the
    // error being the response
  if (res.status >= 400) throw res;

  // if the response status code is under 400, then return the response to the
    // next promise chain
  return res;
}
```

Export the custom `csrfFetch` function from this file.

## Restore the XSRF-TOKEN cookie

In development, the backend and frontend servers are separate. In production
though, the backend also serves up all the frontend assets, including the
`index.html` and any JavaScript files in the `frontend/build` folder after
running `npm start` in the `frontend` folder.

In production, the `XSRF-TOKEN` will be attached to the `index.html` file in the `frontend/build` folder. In the `backend/routes/index.js` file, serve the
`index.html` file at the `/` route and any routes that don't start with `/api`.
Along with it, attach the `XSRF-TOKEN` cookie to the response. Serve the static
files in the `frontend/build` folder using the `express.static` middleware.

```js
// backend/routes/index.js
// ... after `router.use('/api', apiRouter);`

// Static routes
// Serve React build files in production
if (process.env.NODE_ENV === 'production') {
  const path = require('path');
  // Serve the frontend's index.html file at the root route
  router.get('/', (req, res) => {
    res.cookie('XSRF-TOKEN', req.csrfToken());
    return res.sendFile(
      path.resolve(__dirname, '../../frontend', 'build', 'index.html')
    );
  });

  // Serve the static assets in the frontend's build folder
  router.use(express.static(path.resolve("../frontend/build")));

  // Serve the frontend's index.html file at all other routes NOT starting with /api
  router.get(/^(?!\/?api).*/, (req, res) => {
    res.cookie('XSRF-TOKEN', req.csrfToken());
    return res.sendFile(
      path.resolve(__dirname, '../../frontend', 'build', 'index.html')
    );
  });
}

// ...
```

In development, you need another way to get the `XSRF-TOKEN` cookie on your
frontend application because the React frontend is on a different server than
the Express backend. To solve this, add a backend route, `GET /api/csrf/restore`
in the same file that can be accessed only in development and will restore the
`XSRF-TOKEN` cookie.

```js
// backend/routes/index.js
// ...

// Add a XSRF-TOKEN cookie in development
if (process.env.NODE_ENV !== 'production') {
  router.get('/api/csrf/restore', (req, res) => {
    res.cookie('XSRF-TOKEN', req.csrfToken());
    return res.json({});
  });
}

// ...
```

Back in the React frontend, this `GET /api/csrf/restore` route needs to be
called when the application is loaded.

Define and export a function called `restoreCSRF` in the
`frontend/src/store/csrf.js` that will call the custom `csrfFetch` function with
`/api/csrf/restore` as the `url` parameter.

```js
// frontend/src/store/csrf.js
// ...

// call this to get the "XSRF-TOKEN" cookie, should only be used in development
export function restoreCSRF() {
  return csrfFetch('/api/csrf/restore');
}
```

In the frontend entry file (`frontend/src/index.js`), call the `restoreCSRF`
function when in development before defining the `Root` functional component.
Also, attach the custom `csrfFetch` function onto the `window` when in development
as `window.csrfFetch`.

```js
// frontend/src/index.js
// ... other imports
import { restoreCSRF, csrfFetch } from './store/csrf';

// ... const store = configureStore();

if (process.env.NODE_ENV !== 'production') {
  restoreCSRF();

  window.csrfFetch = csrfFetch;
  window.store = store;
}
```

### Test custom `csrfFetch` with CSRF

To test the custom `csrfFetch` function that attaches the CSRF token to the header,
navigate to root route of the React application, [http://localhost:3000]. In the
browser's dev tools console, make a request to `POST /api/test` with the demo
user credentials using the `window.csrfFetch` function. There is no need to
specify the headers because the default header for `"Content-Type"`, set to
`"application/json"`, and the `"XSRF-TOKEN"` header are added by the custom
`csrfFetch`.

```js
window.csrfFetch('/api/test', {
  method: 'POST',
  body: JSON.stringify({ credential: 'Demo-lition', password: 'password' })
}).then(res => res.json()).then(data => console.log(data));
```

If you see an object with a key of `requestBody` logged in the terminal with the
value as the object that you passed into the body of the request, then you
successfully set up CSRF protection on the frontend. If you don't then check
your syntax in the `frontend/src/store/csrf.js` and the `frontend/src/index.js`.

You can now remove the `POST /api/test` test route in your backend code, as you
won't be needing it anymore.

At this point, all the frontend setup is been complete. **Commit your code!**

Now it's time to render some React components!

[test-redux-store-image]: https://appacademy-open-assets.s3-us-west-1.amazonaws.com/Modular-Curriculum/content/react-redux/topics/react-redux-auth/authenticate-me/assets/test-redux-store-setup.png
[Font Awesome]: https://fontawesome.com/start
[Choose a Font Awesome Icon]: https://fontawesome.com/icons?d=gallery&m=free
[carrot icon]: https://fontawesome.com/icons/carrot?style=solid
[Portals in React]: https://reactjs.org/docs/portals.html
[http://localhost:3000]: http://localhost:3000