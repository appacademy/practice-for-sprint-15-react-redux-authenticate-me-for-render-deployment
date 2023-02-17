# Authenticate Me - Deploying your Express + React Project to Render

Before you begin deploying, **make sure to remove any `console.log`s or
`debugger`s in any production code**. You can search your entire project folder
to see if you are using them anywhere by clicking on the magnifying glass icon
on the top left sidebar of VSCode.

You will set up Render to run on a production, not development, version of your
application. When a Node.js application like yours is deployed through Render,
it is identified as a Node.js application because of the `package.json` file.

In the following phases, you will configure your application to work in
production, not just in development, and configure the `package.json` scripts
to install and build your React application, and start the Express production server.

## Phase 1: Setting up your Express + React application

Right now, your React application is on a different localhost port than your
Express application. However, since your React application only consists of
static files that don't need to bundled continuously with changes in production,
your Express application can serve the React assets in production too. These
static files live in the `frontend/build` folder after running `npm run build`
in the `frontend` folder.

Add the following changes into your `backend/routes.index.js` file.

At the root route, serve the React application's static `index.html` file along
with `XSRF-TOKEN` cookie. Then serve up all the React application's static
files using the `express.static` middleware. Serve the `index.html` and set the
`XSRF-TOKEN` cookie again on all routes that don't start in `/api`. You should
already have this set up in `backend/routes/index.js` which should now look
like this:

```js
// backend/routes/index.js
const express = require('express');
const router = express.Router();
const apiRouter = require('./api');

router.use('/api', apiRouter);

// Static routes
// Serve React build files in production
if (process.env.NODE_ENV === 'production') {
  const path = require('path');
  // Serve the frontend's index.html file at the root route
  router.get('/', (req, res) => {
    res.cookie('XSRF-TOKEN', req.csrfToken());
    res.sendFile(
      path.resolve(__dirname, '../../frontend', 'build', 'index.html')
    );
  });

  // Serve the static assets in the frontend's build folder
  router.use(express.static(path.resolve("../frontend/build")));

  // Serve the frontend's index.html file at all other routes NOT starting with /api
  router.get(/^(?!\/?api).*/, (req, res) => {
    res.cookie('XSRF-TOKEN', req.csrfToken());
    res.sendFile(
      path.resolve(__dirname, '../../frontend', 'build', 'index.html')
    );
  });
}

// Add a XSRF-TOKEN cookie in development
if (process.env.NODE_ENV !== 'production') {
  router.get('/api/csrf/restore', (req, res) => {
    res.cookie('XSRF-TOKEN', req.csrfToken());
    res.status(201).json({});
  });
}

module.exports = router;
```

To deploy your Express site with a React frontend, you will need to modify some
scripts in the root `package.json` file. Currently, the `install` script should
be set to install the packages only from the `backend` folder. Overwrite the
`install` script in the root `package.json` to also install the packages in the
`frontend` folder:

```bash
npm --prefix backend install backend && npm --prefix frontend install frontend
```

This will run `npm install` in the `backend` folder then run `npm install` in
the `frontend` folder.

Next, define a `render-postbuild` script that will run the `npm run build`
command in the `frontend` folder. This will build the React static files in
the `frontend` folder. The `render-postbuild` script should run the `build`
script in the `frontend` folder:

```bash
npm run build --prefix frontend
```

The root `package.json`'s scripts should look like this:

```json
  "scripts": {
    "render-postbuild": "npm run build --prefix frontend",
    "install": "npm --prefix backend install backend && npm --prefix frontend install frontend",
    "dev:backend": "npm install --prefix backend start",
    "dev:frontend": "npm install --prefix frontend start",
    "sequelize": "npm run --prefix backend sequelize",
    "sequelize-cli": "npm run --prefix backend sequelize-cli",
    "start": "npm start --prefix backend",
    "build": "npm run build --prefix backend"
  },
```

The `dev:backend` and `dev:frontend` scripts are optional and will not be used
for Render.

<!-- Not using CSP in helmet anymore -->
<!-- There's just one more thing to edit. For the `build` script in the
`frontend/package.json` file, add an `INLINE_RUNTIME_CHUNK=false` environment
variable before `react-scripts build`. This is necessary because the `helmet`
backend package is a middleware you added as an extra layer of security to the
Express application in production. The `helmet` middleware adds a [Content
Security Policy] which doesn't allow unsafe-inline JavaScript scripts. React,
by default, adds their JavaScript scripts as unsafe-inline. To remove this,
you need to have an environment variable of `INLINE_RUNTIME_CHUNK` set to
`false` before running `react-scripts build`.

`frontend/package.json`'s scripts should now look like this:

```json
  "scripts": {
    "start": "react-scripts start",
    "build": "INLINE_RUNTIME_CHUNK=false react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject"
  },
``` -->

Finally, commit your changes.

## Phase 2: Deploy to Render

Once you're finished setting this up, make sure the changes are pushed on the `main` branch.

Now, it's time to make a small change to the Web Service on Render.

Navigate to your Render [Dashboard] and click on the name of the Web Service
that already includes your backend deployment.

Adjust the Build command for the project through the Render form.

Previously, you had used this build command:

```shell
# OLD Build Script
npm install &&
npm run build &&
npm run sequelize --prefix backend db:migrate &&
npm run sequelize --prefix backend db:seed:all
```

Click "Edit", and replace this with the following build command, which now includes the `render-postbuild` script to build the frontend:

```shell
# NEW Build Script
npm install &&
npm run render-postbuild &&
npm run build &&
npm run sequelize --prefix backend db:migrate &&
npm run sequelize --prefix backend db:seed:all
```

Enter `npm start` in the Start command input. Save the changes. 

Click on the blue "Manual Deploy" button, and choose
"Clear Build Cache & Deploy". You will be able to see the logs and confirm that
your re-deployment is successful.

Open your deployed site and check to see if you successfully deployed your
Express + React application to Render!

If you see an `Application Error` or are experiencing different behavior than
what you see in your local environment, check the logs.

The logs may clue you into why you are experiencing errors or different
behavior.

### Wrapping up

Congratulations, you've created a production-ready, dynamic, full-stack website
that can be securely accessed anywhere in the world! Give yourself a pat on the
back. You're a web developer!


[Dashboard]: https://dashboard.render.com/