# Bonus Phase: Make the login form page into a modal

Modals are everywhere in modern applications. Here's one way of implementing a
modal in React without any external libraries/packages.

You will create a modal with using `ReactDOM`'s `createPortal` method.
[Portals in React] provide a way to render React elements into an entirely
separate HTML DOM element from where the React component is rendered.

Let's get started!

## Modal context

First, make a folder in `frontend/src` called `context`. This folder will hold
all the different context and context providers for your application. Add a file
in the `context` folder called `Modal.js`. Create a React context called a
`ModalContext`.

Create a functional component called `ModalProvider` that renders the
`ModalContext.Provider` component with all the `children` from the props as a
child. Render a `div` element as a sibling and right after the
`ModalContext.Provider`.

Create a React ref called `modalRef`. Set the `ref` prop on the rendered `div`
element to this `modalRef`. `modalRef.current` will be set to the actual HTML
DOM element that gets rendered from the `div`. Create a component state variable
called `value` that will be set to `modalRef.current` after the initial render
(hint: use the `useEffect` hook). Pass this `value` as the `value` prop to the
`ModalContext.Provider` component. Export the `ModalProvider` component. Import 
the `ModalProvider` component in `frontend/src/index.js` and wrap all the
contents of the Root component with it.

Create a functional component called `Modal` that expects an `onClose` function
and `children` as props. Get the value of the `ModalContext` into the `Modal`
component by using the `useContext` hook and setting the value equal to a
variable called `modalNode`. Render a `div` with an id of `modal` and nest a
`div` with an id of `modal-background` and another `div` with an id of
`modal-content`. In the `modal-content` div, render the `children` props. When
the `modal-background` is clicked, the `onClose` prop should be invoked. Return
`null` if `modalNode` is falsey.

The `modal-background` div needs to be rendered **before** the `modal-content`
because it will naturally be placed "behind" the depth of the `modal-content`
if it comes before the `modal-content` in the DOM tree.

To get these elements to show up in the `div` in the `ModalProvider` component,
pass the rendered elements in the `Modal` component as the first argument of
`ReactDOM.createPortal` and pass in the `modalNode` as the second argument,
which is the reference to the actual HTML DOM element of the `ModalProvider`'s
`div`. Return the invocation of `ReactDOM.createPortal`. Make sure to import
`ReactDOM` from the `react-dom` package.

Add a CSS file in the `context` folder called `Modal.css`. The `modal` div
should have a `position` `fixed` and take up the entire width and height of the
window. The `modal-background` should also take up the entire width and height
of the window and have a `position` `absolute`. The `modal-content` div should
be centered inside of the `modal` div by flexing the `modal` div and have a
`position` of `absolute`. You may want to give the `modal-background` a
`background-color` of `rgba(0, 0, 0, 0.7)` and the `modal-content` a
`background-color` of `white` just to see them better.

Import the `Modal.css` file into the `Modal.js` context file.

## Login form modal

Now it's time to refactor the `LoginFormPage` component to be a modal instead
of a page.

Rename the `LoginFormPage` folder to `LoginFormModal`. Create a file called
`LoginForm.js` in this folder and move all the code from the `index.js` file in
the `LoginFormModal` file over to the `LoginForm.js` file. Rename the component
from `LoginFormPage` to just `LoginForm`. The code for redirecting the user
if there is no session user in the Redux store can be removed.

In the `index.js` file, import the `LoginForm` component. Create a functional
component called `LoginFormModal`. Add a component state variable called
`showModal` and default it to `false`. Render a button with the text `Log In`
that, when clicked, will set the `showModal` state variable to `true`.

Import the `Modal` component into this file. Render the `Modal` component with
the `LoginForm` component as its child **only when** the `showModal` state
variable is `true`. Add an `onClose` prop to the `Modal` component set to a
function that will change the `showModal` state variable to `false` when
invoked. Export the `LoginFormModal` component as default from this file.

Import the new `LoginFormModal` component into the `Navigation` component.
Replace the link to the login page with this `LoginFormModal` component.

Remove the `LoginFormPage` component from the `App` component.

It's finally time to test out your login form modal! Head to the home page,
[http://localhost:3000], and make sure you are logged out. Click the `Log In`
button. The login form modal should pop up. It should close when you click
anywhere outside of the form. Make sure the login functionality still works!

**Commit, commit, commit!**

## Example modal and login form modal

Here's an example for how `Modal.js` should look like:

```js
// frontend/src/context/Modal.js
import React, { useContext, useRef, useState, useEffect } from 'react';
import ReactDOM from 'react-dom';
import './Modal.css';

const ModalContext = React.createContext();

export function ModalProvider({ children }) {
  const modalRef = useRef();
  const [value, setValue] = useState();

  useEffect(() => {
    setValue(modalRef.current);
  }, [])

  return (
    <>
      <ModalContext.Provider value={value}>
        {children}
      </ModalContext.Provider>
      <div ref={modalRef} />
    </>
  );
}

export function Modal({ onClose, children }) {
  const modalNode = useContext(ModalContext);
  if (!modalNode) return null;

  return ReactDOM.createPortal(
    <div id="modal">
      <div id="modal-background" onClick={onClose} />
      <div id="modal-content">
        {children}
      </div>
    </div>,
    modalNode
  );
}
```

Here's an example for how `Modal.css` should look like:

```css
/* frontend/src/context/Modal.css */
#modal {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  display: flex;
  justify-content: center;
  align-items: center;
}

#modal-background {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.7);
}

#modal-content {
  position: absolute;
  background-color:white;
}
```

Here's an example for how `LoginFormModal/index.js` should look like:

```js
// frontend/src/components/LoginFormModal/index.js
import React, { useState } from 'react';
import { Modal } from '../../context/Modal';
import LoginForm from './LoginForm';

function LoginFormModal() {
  const [showModal, setShowModal] = useState(false);

  return (
    <>
      <button onClick={() => setShowModal(true)}>Log In</button>
      {showModal && (
        <Modal onClose={() => setShowModal(false)}>
          <LoginForm />
        </Modal>
      )}
    </>
  );
}

export default LoginFormModal;
```

Here's an example for how `LoginForm.js` should look like:

```js
// frontend/src/components/LoginFormModal/LoginForm.js
import React, { useState } from "react";
import * as sessionActions from "../../store/session";
import { useDispatch } from "react-redux";

function LoginForm() {
  const dispatch = useDispatch();
  const [credential, setCredential] = useState("");
  const [password, setPassword] = useState("");
  const [errors, setErrors] = useState([]);

  const handleSubmit = (e) => {
    e.preventDefault();
    setErrors([]);
    return dispatch(sessionActions.login({ credential, password })).catch(
      async (res) => {
        const data = await res.json();
        if (data && data.errors) setErrors(data.errors);
      }
    );
  };

  return (
    <form onSubmit={handleSubmit}>
      <ul>
        {errors.map((error, idx) => (
          <li key={idx}>{error}</li>
        ))}
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

export default LoginForm;
```

Here's an example for how `Navigation.js` should look like now:

```js
// frontend/src/components/Navigation/index.js
import React from 'react';
import { NavLink } from 'react-router-dom';
import { useSelector } from 'react-redux';
import ProfileButton from './ProfileButton';
import LoginFormModal from '../LoginFormModal';
import './Navigation.css';

function Navigation({ isLoaded }){
  const sessionUser = useSelector(state => state.session.user);

  let sessionLinks;
  if (sessionUser) {
    sessionLinks = (
      <ProfileButton user={sessionUser} />
    );
  } else {
    sessionLinks = (
      <>
        <LoginFormModal />
        <NavLink to="/signup">Sign Up</NavLink>
      </>
    );
  }

  return (
    <ul>
      <li>
        <NavLink exact to="/">Home</NavLink>
        {isLoaded && sessionLinks}
      </li>
    </ul>
  );
}

export default Navigation;
```

Here's an example of what `App.js` should look like now:

```js
// frontend/src/App.js
import React, { useState, useEffect } from "react";
import { useDispatch } from "react-redux";
import { Route, Switch } from "react-router-dom";
import SignupFormPage from "./components/SignupFormPage";
import * as sessionActions from "./store/session";
import Navigation from "./components/Navigation";

function App() {
  const dispatch = useDispatch();
  const [isLoaded, setIsLoaded] = useState(false);
  useEffect(() => {
    dispatch(sessionActions.restoreUser()).then(() => setIsLoaded(true));
  }, [dispatch]);

  return (
    <>
      <Navigation isLoaded={isLoaded} />
      {isLoaded && (
        <Switch>
          <Route path="/signup">
            <SignupFormPage />
          </Route>
        </Switch>
      )}
    </>
  );
}

export default App;
```

Here's an example of how `frontend/src/index.js` should look:

```js
// frontend/src/index.js
import React from "react";
import ReactDOM from "react-dom";
import "./index.css";
import { Provider } from "react-redux";
import { BrowserRouter } from "react-router-dom";
import App from "./App";
import { ModalProvider } from "./context/Modal";

import configureStore from "./store";
import { restoreCSRF, csrfFetch } from "./store/csrf";
import * as sessionActions from "./store/session";

const store = configureStore();

if (process.env.NODE_ENV !== "production") {
  restoreCSRF();

  window.csrfFetch = csrfFetch;
  window.store = store;
  window.sessionActions = sessionActions;
}

function Root() {
  return (
    <Provider store={store}>
      <ModalProvider>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </ModalProvider>
    </Provider>
  );
}

ReactDOM.render(
  <React.StrictMode>
    <Root />
  </React.StrictMode>,
  document.getElementById('root')
);
```

[test-redux-store-image]: https://appacademy-open-assets.s3-us-west-1.amazonaws.com/Modular-Curriculum/content/react-redux/topics/react-redux-auth/authenticate-me/assets/test-redux-store-setup.png
[Font Awesome]: https://fontawesome.com/start
[Choose a Font Awesome Icon]: https://fontawesome.com/icons?d=gallery&m=free
[carrot icon]: https://fontawesome.com/icons/carrot?style=solid
[Portals in React]: https://reactjs.org/docs/portals.html
[http://localhost:3000]: http://localhost:3000