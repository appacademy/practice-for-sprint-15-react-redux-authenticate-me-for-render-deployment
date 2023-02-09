# Phase 4: Make the login and signup form pages into modals

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

Create and export a functional component called `ModalProvider` that renders the
`ModalContext.Provider` component with all the `children` from the props as a
child. Make sure to export it as a named export, not a default export. Render a
`div` element as a sibling and right after the `ModalContext.Provider`.

Create a React ref called `modalRef` using the [`useRef`] React hook. Set the
`ref` prop on the rendered `div` element to this `modalRef`. `modalRef.current`
will be set to the actual HTML DOM element that gets rendered from the `div`.

```js
// frontend/src/context/Modal.js
import React, { useRef } from 'react';

const ModalContext = React.createContext();

export function ModalProvider({ children }) {
  const modalRef = useRef();

  return (
    <>
      <ModalContext.Provider>
        {children}
      </ModalContext.Provider>
      <div ref={modalRef} />
    </>
  );
}
```

Create an object literal as a dynamic context value and add the `modalRef` as a
key-value pair. Pass the dynamic context value into the `ModalContext.Provider`
as the `value` prop.

```js
// frontend/src/context/Modal.js
import React, { useRef } from 'react';

const ModalContext = React.createContext();

export function ModalProvider({ children }) {
  const modalRef = useRef();

  const contextValue = {
    modalRef, // reference to modal div
  };

  return (
    <>
      <ModalContext.Provider value={contextValue}>
        {children}
      </ModalContext.Provider>
      <div ref={modalRef} />
    </>
  );
}
```

Create `2` component state variables:

1. `modalContent` - The React component to render inside modal
   * default: `null`
   * data type: a React component
   * When this is set to `null` or any falsey value, the modal is closed and
     should not be rendered.
2. `onModalClose` - A callback function to be called when the modal is closing
   * default: `null`
   * data type: `function`

When the `modalContent` state variable is `null` or any falsey value,
**that means the modal is closed**. When it is set to a React component or any
truthy value, **that means that the modal is open.**

Add the state variables and the functions that update it as key-value pairs in
the dynamic context value.

```js
export function ModalProvider({ children }) {
  const modalRef = useRef();
  const [modalContent, setModalContent] = useState(null);
  const [onModalClose, setOnModalClose] = useState(null);

  const contextValue = {
    modalRef, // reference to modal div
    modalContent, // React component to render inside modal
    setModalContent, // function to set the React component to render inside modal
    setOnModalClose, // function to set the callback function to be called when modal is closing
  };

  return (
    <>
      <ModalContext.Provider value={contextValue}>
        {children}
      </ModalContext.Provider>
      <div ref={modalRef} />
    </>
  );
}
```

Create a function called `closeModal` that should be called to trigger the modal
to be closed. This function should be called in any component when you wish to
trigger the modal to close. The `closeModal` function should trigger the closing
of the modal by setting the `modalContent` to `null`. It should also call
`onModalClose` if `onModalClose` is a function.

```js
export function ModalProvider({ children }) {
  const modalRef = useRef();
  const [modalContent, setModalContent] = useState(null);
  const [onModalClose, setOnModalClose] = useState(null);

  const closeModal = () => {
    setModalContent(null); // clear the modal contents
    // If callback function is truthy, call the callback function and reset it
    // to null:
    if (typeof onModalClose === 'function') {
      setOnModalClose(null);
      onModalClose();
    }
  };

  const contextValue = {
    modalRef, // reference to modal div
    modalContent, // React component to render inside modal
    setModalContent, // function to set the React component to render inside modal
    setOnModalClose, // function to set the callback function to be called when modal is closing
    closeModal // function to close the modal
  };

  return (
    <>
      <ModalContext.Provider value={contextValue}>
        {children}
      </ModalContext.Provider>
      <div ref={modalRef} />
    </>
  );
}
```

Import the `ModalProvider` component in `frontend/src/index.js` and wrap all the
contents of the `Root` component with it.

The `Root` component should look something like this:

```js
// frontend/src/index.js
// ...
function Root() {
  return (
    <ModalProvider>
      <Provider store={store}>
        <BrowserRouter>
          <App />
        </BrowserRouter>
      </Provider>
    </ModalProvider>
  );
}
```

Back in the `frontend/src/context/Modal.js` file, create and export a functional
component called `Modal`. Export it as a named export.

The `Modal` component should consume the value of the `ModalContext` by using
the `useContext` React hook. Extract the `modalRef`, `modalContent`, and the
`closeModal` keys from the context value.

Return `null` if `modalRef`, `modalRef.current`, or `modalContent` is falsey.

Otherwise, render a `div` with an `id` of `modal`. Inside, nest two `div`s:

1. a `div` with an `id` of `modal-background`
2. another `div` with an id of `modal-content`

In the `modal-content` div, render the value of `modalContent`.

Add an `onClick` listener to the `modal-background` so that when it is clicked,
the `closeModal` function should be invoked.

The `modal-background` div needs to be rendered **before** the `modal-content`
because it will naturally be placed "behind" the depth of the `modal-content`
if it comes before the `modal-content` in the DOM tree.

To get these elements to show up in the `div` referenced by the `modalRef` in
the `ModalProvider` component, pass the `div` with the `id` of `modal` and all
its nested elements as the first argument of `ReactDOM.createPortal`. Pass
value of `modalRef.current` as the second argument of `ReactDOM.createPortal`.
This will transfer all those elements as the children of the `div` referenced by
the `modalRef` in the `ModalProvider` component. Remember, the value of
`modalRef.current` is the reference to the actual HTML DOM element of the
`ModalProvider`'s `div`. Return the invocation of `ReactDOM.createPortal` from
the `Modal` component.

Make sure to import `ReactDOM` from the `react-dom` package to be able to use
the `ReactDOM.createPortal` function in this file.

```js
import React, { useRef, useState, useContext } from 'react';
import ReactDOM from 'react-dom';

const ModalContext = React.createContext();

export function ModalProvider({ children }) {
  const modalRef = useRef();
  const [modalContent, setModalContent] = useState(null);
  // callback function that will be called when modal is closing
  const [onModalClose, setOnModalClose] = useState(null);

  const closeModal = () => {
    setModalContent(null); // clear the modal contents
    // If callback function is truthy, call the callback function and reset it
    // to null:
    if (typeof onModalClose === 'function') {
      setOnModalClose(null);
      onModalClose();
    }
  };

  const contextValue = {
    modalRef, // reference to modal div
    modalContent, // React component to render inside modal
    setModalContent, // function to set the React component to render inside modal
    setOnModalClose, // function to set the callback function called when modal is closing
    closeModal // function to close the modal
  };

  return (
    <>
      <ModalContext.Provider value={contextValue}>
        {children}
      </ModalContext.Provider>
      <div ref={modalRef} />
    </>
  );
}

export function Modal() {
  const { modalRef, modalContent, closeModal } = useContext(ModalContext);
  // If there is no div referenced by the modalRef or modalContent is not a
  // truthy value, render nothing:
  if (!modalRef || !modalRef.current || !modalContent) return null;

  // Render the following component to the div referenced by the modalRef
  return ReactDOM.createPortal(
    <div id="modal">
      <div id="modal-background" onClick={closeModal} />
      <div id="modal-content">
        {modalContent}
      </div>
    </div>,
    modalRef.current
  );
}
```

Add a CSS file in the `context` folder called `Modal.css`. The `modal` div
should have a `position` `fixed` and take up the entire width and height of the
window. The `modal-background` should also take up the entire width and height
of the window and have a `position` `absolute`. The `modal-content` div should
be centered inside of the `modal` div by flexing the `modal` div and have a
`position` of `absolute`. You may want to give the `modal-background` a
`background-color` of `rgba(0, 0, 0, 0.7)` and the `modal-content` a
`background-color` of `white` just to see them better.

```css
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

Import the `Modal.css` file into the `Modal.js` context file.

Create and export as a named export a custom React hook called `useModal` that
can be used by React components to easily consume the `ModalContext`.

Your `Modal.js` file should now look like this:

```js
import React, { useRef, useState, useContext } from 'react';
import ReactDOM from 'react-dom';
import './Modal.css';

const ModalContext = React.createContext();

export function ModalProvider({ children }) {
  const modalRef = useRef();
  const [modalContent, setModalContent] = useState(null);
  // callback function that will be called when modal is closing
  const [onModalClose, setOnModalClose] = useState(null);

  const closeModal = () => {
    setModalContent(null); // clear the modal contents
    // If callback function is truthy, call the callback function and reset it
    // to null:
    if (typeof onModalClose === 'function') {
      setOnModalClose(null);
      onModalClose();
    }
  };

  const contextValue = {
    modalRef, // reference to modal div
    modalContent, // React component to render inside modal
    setModalContent, // function to set the React component to render inside modal
    setOnModalClose, // function to set the callback function called when modal is closing
    closeModal // function to close the modal
  };

  return (
    <>
      <ModalContext.Provider value={contextValue}>
        {children}
      </ModalContext.Provider>
      <div ref={modalRef} />
    </>
  );
}

export function Modal() {
  const { modalRef, modalContent, closeModal } = useContext(ModalContext);
  // If there is no div referenced by the modalRef or modalContent is not a
  // truthy value, render nothing:
  if (!modalRef || !modalRef.current || !modalContent) return null;

  // Render the following component to the div referenced by the modalRef
  return ReactDOM.createPortal(
    <div id="modal">
      <div id="modal-background" onClick={closeModal} />
      <div id="modal-content">
        {modalContent}
      </div>
    </div>,
    modalRef.current
  );
}

export const useModal = () => useContext(ModalContext);
```

You need to make one last adjustment to `frontend/src/index.js` to be able to
use the modal. Import the `Modal` component from the
`frontend/src/context/Modal.js` file and render it in the `Root` component as a
sibling right under the `App` component.

Your `frontend/src/index.js` should look something like this:

```js
// frontend/src/index.js
import React from 'react';

import './index.css';

import ReactDOM from 'react-dom';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import { ModalProvider, Modal } from './context/Modal';
import App from './App';

import configureStore from './store';
import { restoreCSRF, csrfFetch } from "./store/csrf";
import * as sessionActions from './store/session';

const store = configureStore();

if (process.env.NODE_ENV !== "production") {
  restoreCSRF();

  window.csrfFetch = csrfFetch;
  window.store = store;
  window.sessionActions = sessionActions;
}

// Wrap the application with the Modal provider and render the Modal component
// after the App component so that all the Modal content will be layered as
// HTML elements on top of the all the other HTML elements:
function Root() {
  return (
    <ModalProvider>
      <Provider store={store}>
        <BrowserRouter>
          <App />
          <Modal />
        </BrowserRouter>
      </Provider>
    </ModalProvider>
  );
}

ReactDOM.render(
  <React.StrictMode>
    <Root />
  </React.StrictMode>,
  document.getElementById('root')
);
```

## `OpenModalButton`

The `OpenModalButton` component should be a generic button that can be used to
trigger a modal with any content to open.

Make a folder in `frontend/src/components` called `OpenModalButton`. Create an
`index.js` file in this folder that will hold the code for the `OpenModalButton`
component.

Create and export a functional component called `OpenModalButton` that takes in
`4` props:

1. `modalComponent` - the component to render inside the modal once the button
   is clicked
2. `buttonText` - the text of the button that triggers the modal to open
3. `onButtonClick` - (optional) a callback function that will be called when the
   the button is clicked
4. `onModalClose` - (optional) a callback function that will be called when the
   modal is closing

It should consume the `setModalContent` and `setOnModalClose` from the
`ModalContext`.

It should render a button that displays the `buttonText` as the text inside the
button.

When the button is clicked, it should:

* Invoke `onButtonClick` if `onButtonClick` is a function
* Invoke `setOnModalClose` with `onModalClose` only if `onModalClose` is a
  function
* Open the modal with the `modalComponent` as the content of the modal by
  invoking `setModalContent` with `modalComponent`

The `OpenModalButton` component should now look something like this:

```js
// frontend/src/components/OpenModalButton/index.js
import React from 'react';
import { useModal } from '../../context/Modal';

function OpenModalButton({
  modalComponent, // component to render inside the modal
  buttonText, // text of the button that opens the modal
  onButtonClick, // optional: callback function that will be called once the button that opens the modal is clicked
  onModalClose // optional: callback function that will be called once the modal is closed
}) {
  const { setModalContent, setOnModalClose } = useModal();

  const onClick = () => {
    if (typeof onButtonClick === 'function') onButtonClick();
    if (typeof onModalClose === 'function') setOnModalClose(onModalClose);
    setModalContent(modalComponent);
  };

  return (
    <button onClick={onClick}>{buttonText}</button>
  );
}

export default OpenModalButton;
```

### Examples of how to use the `OpenModalButton`

Let's say you want to render a popup modal text of "Hello World!" when a button
with the text of `"Greeting"` is clicked. You can use the `OpenModalButton` to
create a component which renders this button that triggers this modal.

Here's an example of one such component:

```js
const Greeting = () => {
  return (
    <OpenModalButton
      buttonText="Greeting"
      modalComponent={<h2>Hello World!</h2>}
    />
  );
};
```

Let's say you want to print "Greeting initiated" in the console logs whenever
the `"Greeting"` button is clicked. You could add the following callback
function as the `onButtonClick` prop to the `OpenModalButton` component:

```js
const Greeting = () => {
  return (
    <OpenModalButton
      buttonText="Greeting"
      modalComponent={<h2>Hello World!</h2>}
      onButtonClick={() => console.log("Greeting initiated")}
    />
  );
};
```

Now, let's say you wanted to print "Greeting completed" in the console logs
whenever the user closes the `"Hello World!"` modal. You could add the following
callback function as the `onModalClose` prop to the `OpenModalButton`
component:

```js
const Greeting = () => {
  return (
    <OpenModalButton
      buttonText="Greeting"
      modalComponent={<h2>Hello World!</h2>}
      onButtonClick={() => console.log("Greeting initiated")}
      onModalClose={() => console.log("Greeting completed")}
    />
  );
};
```

The `Greeting` component will render a `button` element that, when clicked, will
trigger a modal with an `h2` element of `"Hello World!"` and will print
"Greeting initiated" to the console logs. When the modal is closed, it will
print "Greeting completed" to the console logs.

If you have already observed, you can use the `OpenModalButton` component for
many different use cases! You can use it to render buttons trigger the login
and sign up forms as modals, but you could use it anywhere in your application
where you want to trigger a modal to open by the click of a button!

## Login form modal

Now it's time to refactor the `LoginFormPage` component to be a modal instead
of a page.

Remove the `LoginFormPage` component from the `App` component.

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

Rename the `LoginFormPage` folder to `LoginFormModal`. Rename the
`LoginFormPage` component in the `index.js` file to `LoginFormModal`. Remove the
code for redirecting the user if there is no session user in the Redux store.
Export the `LoginFormModal` component as default from this file.

When the `LoginFormModal` performs a successful form login submission, the
login modal should close. To do this, the `LoginFormModal` should consume the
`ModalContext`'s `closeModal` value and invoke the `closeModal` function when
the `login` action is successful.

Here's an example for how `LoginFormModal/index.js` should look like:

```js
// frontend/src/components/LoginFormModal/index.js
import React, { useState } from "react";
import * as sessionActions from "../../store/session";
import { useDispatch } from "react-redux";
import { useModal } from "../../context/Modal";
import "./LoginForm.css";

function LoginFormModal() {
  const dispatch = useDispatch();
  const [credential, setCredential] = useState("");
  const [password, setPassword] = useState("");
  const [errors, setErrors] = useState([]);
  const { closeModal } = useModal();

  const handleSubmit = (e) => {
    e.preventDefault();
    setErrors([]);
    return dispatch(sessionActions.login({ credential, password }))
      .then(closeModal)
      .catch(
        async (res) => {
          const data = await res.json();
          if (data && data.errors) setErrors(data.errors);
        }
      );
  };

  return (
    <>
      <h1>Log In</h1>
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
    </>
  );
}

export default LoginFormModal;
```

Import both the `OpenModalButton` and `LoginFormModal` component into the
`Navigation` component.

In the rendering of the `Navigation` component, replace the `NavLink` to the
login page with the `OpenModalButton` component. Pass in the `buttonText` prop
of `"Log In"` and the `modalComponent` prop of the `<LoginFormModal />` to the
`OpenModalButton`.

Here's an example for how `Navigation` component should look like now:

```js
// frontend/src/components/Navigation/index.js
import React from 'react';
import { NavLink } from 'react-router-dom';
import { useSelector } from 'react-redux';
import ProfileButton from './ProfileButton';
import OpenModalButton from '../OpenModalButton';
import LoginFormModal from '../LoginFormModal';
import './Navigation.css';

function Navigation({ isLoaded }){
  const sessionUser = useSelector(state => state.session.user);

  let sessionLinks;
  if (sessionUser) {
    sessionLinks = (
      <li>
        <ProfileButton user={sessionUser} />
      </li>
    );
  } else {
    sessionLinks = (
      <li>
        <OpenModalButton
          buttonText="Log In"
          modalComponent={<LoginFormModal />}
        />
        <NavLink to="/signup">Sign Up</NavLink>
      </li>
    );
  }

  return (
    <ul>
      <li>
        <NavLink exact to="/">Home</NavLink>
      </li>
      {isLoaded && sessionLinks}
    </ul>
  );
}

export default Navigation;
```

It's finally time to test out your login form modal! Head to the home page,
[http://localhost:3000], and make sure you are logged out. Click the `Log In`
button. The login form modal should pop up. It should close when you click
anywhere outside of the form. Make sure the login functionality still works and
that the modal closes only when there is a successful login attempt!

**Commit, commit, commit!**

## Signup form modal

Now, turn the `SignupFormPage` into a `SignupFormModal` the same way that you
turned the `LoginFormPage` into a `LoginFormModal` just now!

Here's an example for how the `App` component should look like now:

```js
import React, { useState, useEffect } from "react";
import { useDispatch } from "react-redux";
import { Switch } from "react-router-dom";
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
        </Switch>
      )}
    </>
  );
}

export default App;
```

Here's an example for how `frontend/src/components/SignupFormModal/index.js`
should look like:

```js
import React, { useState } from "react";
import { useDispatch } from "react-redux";
import { useModal } from "../../context/Modal";
import * as sessionActions from "../../store/session";
import './SignupForm.css';

function SignupFormModal() {
  const dispatch = useDispatch();
  const [email, setEmail] = useState("");
  const [username, setUsername] = useState("");
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");
  const [password, setPassword] = useState("");
  const [confirmPassword, setConfirmPassword] = useState("");
  const [errors, setErrors] = useState([]);
  const { closeModal } = useModal();

  const handleSubmit = (e) => {
    e.preventDefault();
    if (password === confirmPassword) {
      setErrors([]);
      return dispatch(sessionActions.signup({ email, username, firstName, lastName, password }))
        .then(closeModal)
        .catch(async (res) => {
          const data = await res.json();
          if (data && data.errors) setErrors(data.errors);
        });
    }
    return setErrors(['Confirm Password field must be the same as the Password field']);
  };

  return (
    <>
      <h1>Sign Up</h1>
      <form onSubmit={handleSubmit}>
        <ul>
          {errors.map((error, idx) => <li key={idx}>{error}</li>)}
        </ul>
        <label>
          Email
          <input
            type="text"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            required
          />
        </label>
        <label>
          Username
          <input
            type="text"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
            required
          />
        </label>
        <label>
          First Name
          <input
            type="text"
            value={firstName}
            onChange={(e) => setFirstName(e.target.value)}
            required
          />
        </label>
        <label>
          Last Name
          <input
            type="text"
            value={lastName}
            onChange={(e) => setLastName(e.target.value)}
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
        <label>
          Confirm Password
          <input
            type="password"
            value={confirmPassword}
            onChange={(e) => setConfirmPassword(e.target.value)}
            required
          />
        </label>
        <button type="submit">Sign Up</button>
      </form>
    </>
  );
}

export default SignupFormModal;
```

Here's an example for how the `Navigation` component should look like now:

```js
import React from 'react';
import { NavLink } from 'react-router-dom';
import { useSelector } from 'react-redux';
import ProfileButton from './ProfileButton';
import OpenModalButton from '../OpenModalButton';
import LoginFormModal from '../LoginFormModal';
import SignupFormModal from '../SignupFormModal';
import './Navigation.css';

function Navigation({ isLoaded }){
  const sessionUser = useSelector(state => state.session.user);

  let sessionLinks;
  if (sessionUser) {
    sessionLinks = (
      <li>
        <ProfileButton user={sessionUser} />
      </li>
    );
  } else {
    sessionLinks = (
      <li>
        <OpenModalButton
          buttonText="Log In"
          modalComponent={<LoginFormModal />}
        />
        <OpenModalButton
          buttonText="Sign Up"
          modalComponent={<SignupFormModal />}
        />
      </li>
    );
  }

  return (
    <ul>
      <li>
        <NavLink exact to="/">Home</NavLink>
      </li>
      {isLoaded && sessionLinks}
    </ul>
  );
}

export default Navigation;
```

**Commit, commit, commit!**

[test-redux-store-image]: https://appacademy-open-assets.s3-us-west-1.amazonaws.com/Modular-Curriculum/content/react-redux/topics/react-redux-auth/authenticate-me/assets/test-redux-store-setup.png
[Font Awesome]: https://fontawesome.com/start
[Choose a Font Awesome Icon]: https://fontawesome.com/icons?d=gallery&m=free
[carrot icon]: https://fontawesome.com/icons/carrot?style=solid
[Portals in React]: https://reactjs.org/docs/portals.html
[http://localhost:3000]: http://localhost:3000
[`useRef`]: https://reactjs.org/docs/hooks-reference.html#useref