# react-immutable-example

This project demonstrates how to convert to an [Immutable Web App](https://immutablwebapps.org) from a React app that was generated with [Create React App](https://github.com/facebook/create-react-app) version .

## Getting Started

This project was created by running:

```bash
> npx create-react-app react-immutable-example

> cd react-immutable-example
```

## Immutable Conversion

Converting this this new application to build an Immutable Web App requires the following steps:

1. Referencing environment variables defined on `window`
2. Rendering an `index.html` template for deployments
3. Running locally

## Referencing environment variables defined on `window`

Create React App has documentation for [adding custome environment variables](https://facebook.github.io/create-react-app/docs/adding-custom-environment-variables). As they mention in the documentation, the environment variables are embedded during the build-time. To make the assets immutable, the setting of values must be shifted from build-time to run-time. Create React App has documentation for [injecting data from the server into the page](https://facebook.github.io/create-react-app/docs/title-and-meta-tags#injecting-data-from-the-server-into-the-page) which is similar to how this Immutable Web App example will handle environment variables.

Change all references to point to the run-time browser `window` object instead of the build-time node `process` object:

```diff
- process.env.REACT_APP_VARIABLE
+ window.env.REACT_APP_VARIABLE
```

## Rendering an `index.html` template for deployments

In theory, the role of `index.html` in an Immutable Web App is to be a deployment manifest that only contains configuration that is unique to the environment where it is deployed.

In practice, there is often a need to include markup or scripts in the `index.html` that are more than just configuration, or that doesn't vary by environment, or that does change between versions of the app. This issue can be resolved by creating an _immutable_ `index.html` template, that is published with the other immutable assets.

### Converting index.html to a template

This example uses [EJS](https://ejs.co/) as the templating language to render `index.html`, but any templating language can be used.

1) Identify the parts of the `index.html` that vary by environment:

    - [`PUBLIC_URL`](https://facebook.github.io/create-react-app/docs/using-the-public-folder#adding-assets-outside-of-the-module-system): URL where the versioned immutable assets will be deployed.
    - `window.env`: Environment-specific JavaScript configuration.

2) Set `PUBLIC_URL` to an EJS template string in the `.env`:

```diff
+ PUBLIC_URL=<%=PUBLIC_URL%>
```

4) Set `REACT_APP_ENV` to an EJS template string in the `.env`. This will contain the JavaScript environment-specific variables.:

```diff
  PUBLIC_URL=<%=PUBLIC_URL%>
+ REACT_APP_ENV=<%-JSON.stringify(env)%>
```

3) Add a new script tag to render the JavaScript environment-specific variables in `public/index.html`:

```diff
<head>
    ...
+   <script>
+       env = %REACT_APP_ENV%;
+   </script>
</head>
```

### Example

#### `public/index.html`:

__The source template that is converted into an immutable template during the build.__

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
    <meta name="theme-color" content="#000000">
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json">
    <title>React App</title>
    <script>
      env = %REACT_APP_ENV%;
    </script>
  </head>
  <body>
    <noscript>
      You need to enable JavaScript to run this app.
    </noscript>
    <div id="root"></div>
  </body>
</html>
```

#### `.env`

__The build-time environment variables used to generate the immutable template.__

```sh
PUBLIC_URL=<%=PUBLIC_URL%>
REACT_APP_ENV=<%-JSON.stringify(env)%>
```

#### `build/index.html`

__The immutable template that is published along with the other versioned, immutable assets.__ It is combined with `config.json` to render the `index.html` that is a deployment manifest.

```html
<!doctype html>
<html lang="en">
  <head>
      <meta charset="utf-8">
      <link rel="shortcut icon" href="<%=PUBLIC_URL%>/favicon.ico">
      <meta name="viewport" content="width=device-width,initial-scale=1,shrink-to-fit=no">
      <meta name="theme-color" content="#000000">
      <link rel="manifest" href="<%=PUBLIC_URL%>/manifest.json">
      <title>React App</title>
      <script>env = <%-JSON.stringify(env)%>;</script>
      <link href="<%=PUBLIC_URL%>/static/css/main.6bd13355.chunk.css" rel="stylesheet">
  </head>
  <body>
      <noscript>
        You need to enable JavaScript to run this app.
      </noscript>
      <div id="root"></div>
      <script>...</script>
      <script src="<%=PUBLIC_URL%>/static/js/1.a700ff87.chunk.js"></script>
      <script src="<%=PUBLIC_URL%>/static/js/main.b75225a8.chunk.js"></script>
  </body>
</html>
```

#### `config.json`:

__The environments-specific values that vary.__ This is combined with the published immutable template to be rendered into `index.html`. It is not an immutable asset and should be managed as deployment-specific configuration.

```json
{
    "PUBLIC_URL": "https://assets.react-immutable-example.com/1.0.0",
    "env": {
        "api": "https://api.react-immutable-example.com"
    }
}
```

#### Rendered `index.html`

__The environment-specific deployment manifest.__ It is the product of rendering an immutable template against the `config.json`. Publishing this file to the web application environment is an atomic deployment.

```html
<!doctype html>
<html lang="en">
  <head>
      <meta charset="utf-8">
      <link rel="shortcut icon" href="https://assets.react-immutable-example.com/1.0.0/favicon.ico">
      <meta name="viewport" content="width=device-width,initial-scale=1,shrink-to-fit=no">
      <meta name="theme-color" content="#000000">
      <link rel="manifest" href="https://assets.react-immutable-example.com/1.0.0/manifest.json">
      <title>React App</title>
      <script>env = { api: "https://api.react-immutable-example.com" };</script>
      <link href="https://assets.react-immutable-example.com/1.0.0/static/css/main.6bd13355.chunk.css" rel="stylesheet">
  </head>
  <body>
      <noscript>
        You need to enable JavaScript to run this app.
      </noscript>
      <div id="root"></div>
      <script>...</script>
      <script src="https://assets.react-immutable-example.com/1.0.0/static/js/1.a700ff87.chunk.js"></script>
      <script src="https://assets.react-immutable-example.com/1.0.0/static/js/main.b75225a8.chunk.js"></script>
  </body>
</html>
```

## Running Locally

Without making any additional changes, `npm start` will run and serve the static assets as well as attempt to serve the immutable template `index.html`. A proper `index.html` can be rendered by [overriding the `.env`](https://facebook.github.io/create-react-app/docs/adding-custom-environment-variables#what-other-env-files-can-be-used) file with values instead of template strings in an `.env.local` file.

`.env`:

```sh
PUBLIC_URL=<%=PUBLIC_URL%>
REACT_APP_ENV=<%-JSON.stringify(env)%>
```

`.env.local`:

```sh
PUBLIC_URL=/
REACT_APP_ENV={ api: "https://localhost:3001/api" }
```

## Future Work

This project will be updated as better patterns for building Immutable Web Apps are established and as [Create React App](https://github.com/facebook/create-react-app) changes.
