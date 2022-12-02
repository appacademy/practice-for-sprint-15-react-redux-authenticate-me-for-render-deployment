# Phase 3: Logout, Navigation, and Profile Dropdown

The last part of the authentication flow is logging out. The logout button will
be placed in a dropdown menu in a navigation bar only when a session user
exists.

## Logout action

You will use the `DELETE /api/session` backend route to logout a user.

In the session store file, add a logout thunk action that will hit the logout
backend route. After the response from the AJAX call comes back, dispatch the
action for removing the session user.

Export the logout thunk action.

### Test the logout action

Test the logout thunk action.

Navigate to [http://localhost:3000]. If there is no `token` cookie, add one by
logging in or signing up. In the browser's dev tools console, try dispatching
the logout thunk action.

The `previous state` in the console should look like this:

```js
{
  session: {
    user: {
      createdAt: "<Some date time format>",
      email: "<new email>",
      id: "<new id>",
      updatedAt: "<Some date time format>",
      username: "<new password>",
      firstName: "<new first name>",
      lastName: "<new last name>",
    }
  }
}
```

The `next state` in the console should look something like this:

```js
{
  session: {
    user: null
  }
}
```

If there is an error or if the previous or next state does not look like this,
then check your logic in your logout action and in your session reducer.

**Commit your code for the logout action.**

### Example logout action

Again, there is no absolute "right" way of doing this. As long as your logout
action is displaying the expected initial state and states after each dispatched
action, then your setup is fine.

Here's an example for the logout thunk action:

```js
// frontend/src/store/session.js
// ...
export const logout = () => async (dispatch) => {
  const response = await csrfFetch('/api/session', {
    method: 'DELETE',
  });
  dispatch(removeUser());
  return response;
};
// ...
```

Here's an example for the logout thunk action test in the browser's dev tools
console:

```js
window.store.dispatch(window.sessionActions.logout());
```

## `Navigation` component

After finishing the Redux action for the logout feature, the React components
are next. The `Navigation` component will render navigation links and a logout
button.

Make a folder called `Navigation` nested in the `frontend/src/components`
folder which will hold all the files for the signup form. Add an `index.js` file
in the `Navigation` folder. Inside of this file, add a React functional
component named `Navigation`.

Your navigation should render an unordered list with a navigation link to the
home page. It should only contain navigation links to the login and signup
routes when there is no session user and a logout button when there is.

Make a `ProfileButton.js` file in the `Navigation` folder. Create a React
functional component called `ProfileButton` that will render a generic user
profile icon of your choice from [Font Awesome].

Follow the [instructions here for setting up Font Awesome][Font Awesome]. The
easiest way to connect Font Awesome to your React application is by sharing your
email and creating a new kit. The kit should let you copy an HTML `<script>`.
Add this script to the `<head>` of your `frontend/public/index.html` file.

**If you don't want to signup for Font Awesome** and are okay with using Font
Awesome icons that may not be up to date, you can just add the following `link`
to the `<head>` of your `frontend/public/index.html` file:

```html
<link
  rel="stylesheet"
  href="https://use.fontawesome.com/releases/v5.5.0/css/all.css"
  integrity="sha384-B4dIYHKNBt8Bc12p+WXckhzcICo0wtJAoU8YZTY5qE0Id1GSseTk6S+L3BlXeVIU"
  crossorigin="anonymous" />
```

Now you can use any of the [free icons available in Font Awesome][Choose a Font Awesome Icon]
by adding the `<i>` element with the desired `className` to be
rendered in a React component. To change the size or color of the icon, wrap
the `<i>` element in a parent element like a `div`. Manipulating the `font-size`
of the parent element changes the size of the icon. The color of the parent
element will be the color of the icon. For example, to render a big
orange [carrot icon]:

```js
const Carrot = () => {
  return (
    <div style={{ color: "orange", fontSize: "100px" }}>
      <i className="fas fa-carrot"></i>
    </div>
  );
};
```

[Choose an icon][Choose a Font Awesome Icon] that will represent the user
profile button and render it in the `ProfileButton` component.

Export the `ProfileButton` component at the bottom of the file, and import it
into the `Navigation` component. Render the `ProfileButton` component only when
there is a session user.

Export the `Navigation` component and import it into the `App` component. Render
the `Navigation` component so that it shows up at the top of each page.

Navigate to the [http://localhost:3000] and remove the `token` cookie if there
is one. Refresh and see if there is a navigation bar with links to the login
and signup pages. After logging in, the navigation bar should have the links
to login and signup replaced with the Font Awesome user icon.

Here's an example for how `Navigation/index.js` should look like:

```js
// frontend/src/components/Navigation/index.js
import React from 'react';
import { NavLink } from 'react-router-dom';
import { useSelector, useDispatch } from 'react-redux';
import ProfileButton from './ProfileButton';
import * as sessionActions from '../../store/session';
import './Navigation.css';

function Navigation({ isLoaded }){
  const sessionUser = useSelector(state => state.session.user);
  const dispatch = useDispatch();

  const logout = (e) => {
    e.preventDefault();
    dispatch(sessionActions.logout());
  };

  let sessionLinks;
  if (sessionUser) {
    sessionLinks = (
      <li>
        <ProfileButton user={sessionUser} />
        <button onClick={logout}>Log Out</button>
      </li>
    );
  } else {
    sessionLinks = (
      <li>
        <NavLink to="/login">Log In</NavLink>
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

Here's an example for how `App.js` should look like now:

```js
// frontend/src/App.js
import React, { useState, useEffect } from "react";
import { useDispatch } from "react-redux";
import { Route, Switch } from "react-router-dom";
import LoginFormPage from "./components/LoginFormPage";
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
          <Route path="/login">
            <LoginFormPage />
          </Route>
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

**Now is a good time to commit your working code.**

## `ProfileButton` component

Change the `ProfileButton` component to show a user profile icon as a button and
a list containing the session user's username, full name (by putting the user's
firstName and lastName next to each other), and email and logout button. Move
the logout button logic from the `Navigation` component to the `ProfileButton`
component. Remember, the logout button should dispatch the logout action when
clicked. The `ProfileButton` component should look something like this:

```js
// frontend/src/components/Navigation/ProfileButton.js
import React, { useState } from "react";
import { useDispatch } from 'react-redux';
import * as sessionActions from '../../store/session';

function ProfileButton({ user }) {
  const dispatch = useDispatch();

  const logout = (e) => {
    e.preventDefault();
    dispatch(sessionActions.logout());
  };

  const ulClassName = "profile-dropdown";

  return (
    <>
      <button>
        <i className="fas fa-user-circle" />
      </button>
      <ul className="profile-dropdown">
        <li>{user.username}</li>
        <li>{user.firstName} {user.lastName}</li>
        <li>{user.email}</li>
        <li>
          <button onClick={logout}>Log Out</button>
        </li>
      </ul>
    </>
  );
}

export default ProfileButton;
```

Here's an example for how `Navigation/index.js` should look like without the
logout logic:

```js
// frontend/src/components/Navigation/index.js
import React from 'react';
import { NavLink } from 'react-router-dom';
import { useSelector } from 'react-redux';
import ProfileButton from './ProfileButton';
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
        <NavLink to="/login">Log In</NavLink>
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

Here's an example for how `frontend/public/index.html` should look like now with
the recommended [Font Awesome] setup. Replace `{kit_id}` in the `script`'s `src`
with the value of your Font Awesome starter kit's id.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Simple React App</title>
    <script src="https://kit.fontawesome.com/{kit_id}.js" crossorigin="anonymous"></script>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

As an alternative, you can also use the somewhat outdated Font Awesome CSS
stylesheet if you don't want to register for Font Awesome:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Simple React App</title>
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.5.0/css/all.css" integrity="sha384-B4dIYHKNBt8Bc12p+WXckhzcICo0wtJAoU8YZTY5qE0Id1GSseTk6S+L3BlXeVIU" crossorigin="anonymous" />
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

### `Navigation` CSS

Add a `Navigation.css` file in your `Navigation` folder. Import this CSS
file into the `frontend/src/components/Navigation/index.js` file.

```js
// frontend/src/components/Navigation/index.js
// ...
import './Navigation.css';
// ...
```

Define all your CSS styling rules for the `Navigation` component in the
`Navigation.css` file. Make your navigation bar look good and your dropdown menu
flow well with the rest of the elements. **Afterwards, commit!**

### Dropdown menu

When clicked, the user profile button should trigger a component state change
and cause the session user information and logout button to be rendered in a
dropdown menu. When there is a click outside of the dropdown menu list or on the
profile button again, then the dropdown menu should disappear.

Dropdown menus in React is a little challenging. You will need to use your
knowledge of vanilla JavaScript DOM manipulation for this feature.

First, create a state variable called `showMenu` to control displaying the
dropdown. `showMenu` defaults to `false` indicating that the menu is hidden.
When the `ProfileButton` is clicked, toggle `showMenu` to `true` indicating that
the menu should now be shown. Modify the return value of your functional
component conditionally to either show or hide the menu based on the `showMenu`
_state variable_ using CSS. For example, if the `showMenu` state variable is
`false`, then apply a `className` of "hidden" to the dropdown menu element.
Otherwise, don't apply that `className`. Add a `.hidden` CSS rule that will
add a CSS style of `display: none` to the dropdown menu element.

```js
// frontend/src/components/Navigation/ProfileButton.js
import React, { useState } from "react";
import { useDispatch } from 'react-redux';
import * as sessionActions from '../../store/session';

function ProfileButton({ user }) {
  const dispatch = useDispatch();
  const [showMenu, setShowMenu] = useState(false);

  const logout = (e) => {
    e.preventDefault();
    dispatch(sessionActions.logout());
  };

  const ulClassName = "profile-dropdown" + (showMenu ? "" : " hidden");

  return (
    <>
      <button onClick={() => setShowMenu(!showMenu)}>
        <i className="fas fa-user-circle" />
      </button>
      <ul className={ulClassName}>
        <li>{user.username}</li>
        <li>{user.firstName} {user.lastName}</li>
        <li>{user.email}</li>
        <li>
          <button onClick={logout}>Log Out</button>
        </li>
      </ul>
    </>
  );
}

export default ProfileButton;
```

Test this out by navigating to [http://localhost:3000]. If you click the profile
button, the menu list with the logout button should appear with the session
user's username, full name, and email. When you click the logout button, the profile button
and menu list should disappear. If you try logging in again and clicking the
profile button, there is currently no way to close the menu list once it's open
unless you logout. Let's work on this next, but first, make sure that you have
the above behavior in your navigation bar.

### Closing the dropdown menu on any click

Let's now make the dropdown menu close when anywhere outside the dropdown menu
is clicked. To do this, you need to add an event listener to the entire document
to listen to any click changes and set the `showMenu` state variable to `false`
for any clicks outside of the dropdown menu.

Create a function called `openMenu` in the `ProfileButton` component. If
`showMenu` is `false`, nothing should happen. If `showMenu` is `true`, then
set the `showMenu` to `true`. When the profile button is clicked, it should call
`openMenu`.

```js
import React, { useState } from "react";
import { useDispatch } from 'react-redux';
import * as sessionActions from '../../store/session';

function ProfileButton({ user }) {
  const dispatch = useDispatch();
  const [showMenu, setShowMenu] = useState(false);

  const openMenu = () => {
    if (showMenu) return;
    setShowMenu(true);
  };

  const logout = (e) => {
    e.preventDefault();
    dispatch(sessionActions.logout());
  };

  const ulClassName = "profile-dropdown" + (showMenu ? "" : " hidden");

  return (
    <>
      <button onClick={openMenu}>
        <i className="fas fa-user-circle" />
      </button>
      <ul className={ulClassName}>
        <li>{user.username}</li>
        <li>{user.firstName} {user.lastName}</li>
        <li>{user.email}</li>
        <li>
          <button onClick={logout}>Log Out</button>
        </li>
      </ul>
    </>
  );
}

export default ProfileButton;
```

The profile button click should no longer cause the menu to both
open and close, only open.

When the dropdown menu is open, you need to register an event listener for
`click` events on the entire page (the `document`), in order to know when to
close the menu. Use an `useEffect` hook to create, register, and remove this
listener.

Inside the `useEffect`, create a function called `closeMenu`. When this function
is called set the `showMenu` state variable to `false` to trigger the dropdown
menu to close. Register the `closeMenu` function as an event listener for
`click` events on the entire page or `document`. The cleanup function for the
`useEffect` should remove this event listener.

```js
  useEffect(() => {
    const closeMenu = (e) => {
      setShowMenu(false);
    };

    document.addEventListener('click', closeMenu);

    return () => document.removeEventListener("click", closeMenu);
  }, []);
```

If you try to test this on [http://localhost:3000], you'll notice that the
dropdown menu just doesn't open at all. Why do you think that is? Add a
`debugger` in the `openMenu` and the `closeMenu` functions. When you click on
the profile button, both `debugger`'s in the `openMenu` and `closeMenu`
functions will be triggered. To prevent this behavior, the listener should only
be added when `showMenu` changes to `true`. Make sure to only add the event
listener and return the cleanup function if `showMenu` is `true`. Add `showMenu`
to the dependencies array for `useEffect`.

```js
  useEffect(() => {
    if (!showMenu) return;

    const closeMenu = (e) => {
      setShowMenu(false);
    };

    document.addEventListener('click', closeMenu);

    return () => document.removeEventListener("click", closeMenu);
  }, [showMenu]);
```

Now, navigate to the home page and try opening and closing the dropdown menu.
You should see the dropdown menu open and close!

### Closing the dropdown menu an outside click only

Now, what if you try opening the menu and then clicking inside the dropdown
menu? Your dropdown menu should close. But why? Since you added the `closeMenu`
event listener to the entire HTML `document` and the dropdown menu is part of
the `document`, clicking the dropdown menu will close it! But actually, you want
the dropdown menu to close **only if the click happened OUTSIDE the dropdown**.

You will solve this problem by examining where the click happened. If the click
was inside of the dropdown menu, **do NOT** close the dropdown menu. Otherwise,
if the click happened any other element on the page, close the dropdown menu.

To apply this logic, you can use the [`.contains`] method on an HTML element to
check whether the `target` of a click `event` is in the boundaries of the
HTML element.

```js
// EXAMPLE OF AN EVENT LISTENER
onClick((event) => {
  element.contains(event.target); // true/false
  // evaluates to true if click happened inside of the element
  // evaluates to false if click happened outside of the element
});
```

To get the reference to the HTML element of the dropdown menu, you can use the
[`useRef`] React hook. One of its uses include capturing the real DOM element
that a virtual DOM element maps to. Call it in the component and attach the
reference to the dropdown menu JSX element.

```js
import React, { useState, useEffect, useRef } from "react";
import { useDispatch } from 'react-redux';
import * as sessionActions from '../../store/session';

function ProfileButton({ user }) {
  const dispatch = useDispatch();
  const [showMenu, setShowMenu] = useState(false);
  const ulRef = useRef();

  const openMenu = () => {
    if (showMenu) return;
    setShowMenu(true);
  };

  useEffect(() => {
    if (!showMenu) return;

    const closeMenu = (e) => {
      setShowMenu(false);
    };

    document.addEventListener('click', closeMenu);

    return () => document.removeEventListener("click", closeMenu);
  }, [showMenu]);

  const logout = (e) => {
    e.preventDefault();
    dispatch(sessionActions.logout());
  };

  const ulClassName = "profile-dropdown" + (showMenu ? "" : " hidden");

  return (
    <>
      <button onClick={openMenu}>
        <i className="fas fa-user-circle" />
      </button>
      <ul className={ulClassName} ref={ulRef}>
        <li>{user.username}</li>
        <li>{user.firstName} {user.lastName}</li>
        <li>{user.email}</li>
        <li>
          <button onClick={logout}>Log Out</button>
        </li>
      </ul>
    </>
  );
}

export default ProfileButton;
```

In the `closeMenu` function change `showMenu` to `false` only if the `target`
of the click `event` does **NOT** contain the HTML element of the dropdown menu.

```js
    const closeMenu = (e) => {
      if (!ulRef.current.contains(e.target)) {
        setShowMenu(false);
      }
    };
```

Now your dropdown menu should be fully functional and should look something
like this!

```js
// frontend/src/components/Navigation/ProfileButton.js
import React, { useState, useEffect, useRef } from "react";
import { useDispatch } from 'react-redux';
import * as sessionActions from '../../store/session';

function ProfileButton({ user }) {
  const dispatch = useDispatch();
  const [showMenu, setShowMenu] = useState(false);
  const ulRef = useRef();

  const openMenu = () => {
    if (showMenu) return;
    setShowMenu(true);
  };

  useEffect(() => {
    if (!showMenu) return;

    const closeMenu = (e) => {
      if (!ulRef.current.contains(e.target)) {
        setShowMenu(false);
      }
    };

    document.addEventListener('click', closeMenu);

    return () => document.removeEventListener("click", closeMenu);
  }, [showMenu]);

  const logout = (e) => {
    e.preventDefault();
    dispatch(sessionActions.logout());
  };

  const ulClassName = "profile-dropdown" + (showMenu ? "" : " hidden");

  return (
    <>
      <button onClick={openMenu}>
        <i className="fas fa-user-circle" />
      </button>
      <ul className={ulClassName} ref={ulRef}>
        <li>{user.username}</li>
        <li>{user.firstName} {user.lastName}</li>
        <li>{user.email}</li>
        <li>
          <button onClick={logout}>Log Out</button>
        </li>
      </ul>
    </>
  );
}

export default ProfileButton;
```

Congratulations on implementing an awesome dropdown menu all in React! **Make
sure to commit your code!**

[test-redux-store-image]: https://appacademy-open-assets.s3-us-west-1.amazonaws.com/Modular-Curriculum/content/react-redux/topics/react-redux-auth/authenticate-me/assets/test-redux-store-setup.png
[Font Awesome]: https://fontawesome.com/start
[Choose a Font Awesome Icon]: https://fontawesome.com/icons?d=gallery&m=free
[carrot icon]: https://fontawesome.com/icons/carrot?style=solid
[`.contains`]: https://developer.mozilla.org/en-US/docs/Web/API/Node/contains
[`useRef`]: https://reactjs.org/docs/hooks-reference.html#useref
[Portals in React]: https://reactjs.org/docs/portals.html
[http://localhost:3000]: http://localhost:3000