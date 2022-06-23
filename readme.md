# Svelte Router

Simple [Svelte](#https://svelte.dev) Router for Single Page Applications (SPA), inspired by [Vue Router](https://router.vuejs.org/). Features include:

- Nested route/view mapping.
- Modular, component-based router configuration.
- Route params, query, wildcards _(powered by [path-to-regexp](https://github.com/pillarjs/path-to-regexp))_.
- Navigation guards for navigation control.
- Links with automatic active CSS classes.
- HTML5 history mode or hash mode _(powered by [history](https://github.com/ReactTraining/history))_.

To see the details code documentation, please read the [Code Documentation](https://spaceavocado.github.io/svelte-router/).

## Notice

> This project is not being actively maintained by the author of package. Any community contribution would be more than welcomed keep this project alive.

**Quick Links**

- [Webpack Setup](#webpack-setup)
- [Webpack Boilerplate Project Template](#webpack-boilerplate-project-template)
- [Rollup Setup](#rollup-setup)

**Table of Content**

- [Svelte Router](#svelte-router)
  - [Installation via NPM or Yarn](#installation-via-npm-or-yarn)
  - [Webpack Setup](#webpack-setup)
  - [Webpack Boilerplate Project Template](#webpack-boilerplate-project-template)
  - [Rollup Setup](#rollup-setup)
  - [Essentials](#essentials)
    - [Setup the Router](#setup-the-router)
    - [Route Configuration](#route-configuration)
    - [Dynamic Route Configuration](#dynamic-route-configuration)
    - [Route Redirection](#route-redirection)
    - [Passing Props to Route Components](#passing-props-to-route-components)
      - [Automatically Pass Route Params as Component Props](#automatically-pass-route-params-as-component-props)
      - [Pass Custom Object as Component Props](#pass-custom-object-as-component-props)
      - [Use a Function to Resolve the Component Props](#use-a-function-to-resolve-the-component-props)
      - [Auto Passed Route Prop](#auto-passed-route-prop)
    - [Nested Routes](#nested-routes)
    - [Router Link Component](#router-link-component)
      - [Router Link Events](#router-link-events)
    - [Router View Component](#router-view-component)
  - [Advanced](#advanced)
    - [Programmatic Navigation](#programmatic-navigation)
    - [Navigation Guard](#navigation-guard)
      - [Create a Navigation Guard](#create-a-navigation-guard)
      - [Navigation Guard Next Action](#navigation-guard-next-action)
        - [Continue](#continue)
        - [Abort](#abort)
        - [Redirect](#redirect)
        - [Error](#error)
    - [Lazy Loaded Component](#lazy-loaded-component)
  - [API](#api)
    - [Create Router](#create-router)
      - [Router Options](#router-options)
      - [Access Router Instance](#access-router-instance)
    - [Router Methods](#router-methods)
      - [start](#start)
      - [push](#push)
      - [replace](#replace)
      - [back](#back)
      - [forward](#forward)
      - [go](#go)
      - [routeURL](#routeurl)
      - [navigationGuard](#navigationguard)
      - [onBeforeNavigation](#onbeforenavigation)
      - [onNavigationChanged](#onnavigationchanged)
      - [onError](#onerror)
    - [Location Object](#location-object)
    - [Route Object](#route-object)
    - [Route Record Object](#route-record-object)
  - [Changes](#changes)
  - [About](#about)
  - [Contributing](#contributing)
    - [Pull Request Process](#pull-request-process)
  - [License](#license)

## Installation via NPM or Yarn

```sh
npm install -D @etip/svelte-router-from-crutch-builder
```

```sh
yarn add @etip/svelte-router-from-crutch-builder -D
```

## Webpack Setup

Please see the [Svelte Webpack Template](https://github.com/sveltejs/template-webpack).
Important setup in the **webpack.config.js**:

```javascript
resolve: {
  // This alias is important to prevent svelte mismatch
  // between your code, and the 3rd party components.
  alias: {
    svelte: path.resolve('node_modules', 'svelte')
  },
  extensions: ['.mjs', '.js', '.svelte'],
  mainFields: ['svelte', 'browser', 'module', 'main']
},

module: {
  rules: [
    {
      test: /\.svelte$/,
      // Do not exclude: /(node_modules)/ since the router
      // components are located in the node_modules
      use: {
        loader: 'svelte-loader',
        options: {
          emitCss: true,
          hotReload: true
        }
      }
    }
  ]
}
```

## Webpack Boilerplate Project Template

For a quick start, you can use [svelte-router-template](https://github.com/spaceavocado/svelte-router-template) - Webpack boilerplate template project for spaceavocado/svelte-router.

[Live Preview](https://spaceavocado.github.io/svelte-router-template/).

## Rollup Setup

rollup.config.js:

```javascript
import babel from "rollup-plugin-babel";
import resolve from "rollup-plugin-node-resolve";
import commonjs from "rollup-plugin-commonjs";
import svelte from "rollup-plugin-svelte";

export default {
  input: "./src/index.js",
  output: {
    file: "./dist/bundle.js",
    format: "iife",
  },
  plugins: [
    resolve(),
    commonjs({
      include: "node_modules/**",
    }),
    svelte(),
    babel({
      exclude: "node_modules/**",
    }),
  ],
};
```

## Essentials

Note: All code below uses ES2015+.

### Setup the Router

All we need to do is map our components to the routes and add root RouterView component, here's a basic example of app component uses as the main [Svelte](#https://svelte.dev) component.

index.js:

```javascript
// The base component
import App from "./app.svelte";

// Turn on the engine
new App({
  target: document.body,
});
```

app.svelve:

```html
<script>
  import createRouter from "@etip/svelte-router-from-crutch-builder";
  import RouterView from "@etip/svelte-router-from-crutch-builder/component/view";

  // View components
  import ViewHome from "./views/home.svelte";
  import View404 from "./views/404.svelte";

  createRouter({
    routes: [
      {
        path: "/",
        name: "HOME",
        component: ViewHome,
      },
      {
        path: "*",
        component: View404,
      },
    ],
  });
</script>

<RouterView />
```

This is an example of the minimalist plug & play setup of the Svelte Router. For more details:

- [Create Router](#create-router)
- [Route Configuration](#route-configuration)
- [Route View Component](#router-view-component)

### Route Configuration

A route can be configure with these properties:

| Property  | Description                                                                                                                                                                                          | Type                      |
| :-------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------ |
| path      | A string that equals the path of the current route, always resolved as an absolute path. e.g. "/foo/bar". Please see [Dynamic Route Configuration](#dynamic-route-configuration) for advanced usage. | string                    |
| redirect  | Redirection to different route, or to external site. Please see [Route Redirection](#route-redirection) for advanced usage.                                                                          | boolean, object, function |
| name      | The name of the current route, optional.                                                                                                                                                             | string                    |
| component | [Svelte](#https://svelte.dev) component. It could be be omitted if the route has nested routes.                                                                                                      | function                  |
| meta      | Route meta object, meta is used a bucket for your custom data on route object.                                                                                                                       | object                    |
| props     | Declaration of component properties passed to the component by the route. Please see [Passing Props to Route Components](#passing-props-to-route-components) for more details.                       | boolean, object, function |
| children  | Collection of children/nested routes. Please see [Nested Routes](#nested-routes) for more details.                                                                                                   | object[]                  |

```javascript
createRouter({
  routes: [
    {
      path: "/",
      name: "HOME",
      component: ViewHome,
      meta: {
        static: 5,
        dynamic: () => {
          return 5;
        },
      },
    },
    {
      path: "/services",
      name: "SERVICES",
      children: [
        {
          path: "/",
          name: "SERVICES_HOME",
          component: ViewServices,
        },
        {
          path: "/design",
          name: "SERVICES_DESIGN",
          component: ViewServicesDesign,
          props: {
            title: "Services / Design",
          },
        },
      ],
    },
    {
      path: "/users/:id(\\d+)",
      name: "USER_PROFILE",
      component: ViewUserProfile,
      props: (route) => {
        return {
          userId: route.params.id,
        };
      },
    },
  ],
});
```

### Dynamic Route Configuration

- The route **path** could contain dynamic parameters, e.g. `path: '/users/:id'`. Please see [path-to-regexp](https://github.com/pillarjs/path-to-regexp) for more information how to configure the name, optional, etc,. parameters.
- Special case is `path: '*'` which means any URL, this route should be the last route in your routes definition.
- All resolved dynamic parameters are accessible on the resolved route object.

### Route Redirection

To redirect a route, you can configure it like so:

```javascript
// Redirect from /a to /b
{
  path: '/a',
  redirect: '/b',
}

// External site redirect
// Must start with 'http' or 'https'
{
  path: '/a',
  redirect: 'https://github.com/spaceavocado/svelte-router',
}

// Redirect to named route
{
  path: '/a',
  redirect: {
    name: 'ROUTE_NAME',
  },
}

// Dynamic redirecting
// "to" is Route Object
// Return value must be URL or a Route.
{
  path: '/a',
  redirect: (to) => {
    return {
      name: 'ROUTE_NAME',
    }
  },
}
```

> Please see [Route Object](#route-object) for more details.

### Passing Props to Route Components

By default, props are not automatically passed to the route component, this could be change to:

#### Automatically Pass Route Params as Component Props

All route dynamic props are auto-passed to the component.

```javascript
// app.svelte
{
  path: '/users/:id(\\d+)',
  name: 'USER_PROFILE',
  component: ViewUserProfile,
  props: true,
}

// ViewUserProfile.svelte
export let id;
```

#### Pass Custom Object as Component Props

```javascript
// app.svelte
{
  path: '/',
  name: 'HOME',
  component: ViewHome,
  props: {
    title: 'Hello World',
  },
}

// ViewHome.svelte
export let title;
```

#### Use a Function to Resolve the Component Props

Please see [Route Object](#route-object) for more details.

```javascript
// app.svelte
{
  path: '/users/:id(\\d+)',
  name: 'USER_PROFILE',
  component: ViewUserProfile,
  props: (route) => {
    return {
      title: 'User Profile',
      id: route.params.id,
    }
  },
}

// ViewUserProfile.svelte
export let title;
export let id;
```

#### Auto Passed Route Prop

Route property of type [Route Object](#route-object) is auto-passed to the route component.

component.svelte:

```html
<script>
  export let route;
</script>
```

### Nested Routes

- Please see [Route Configuration](#route-configuration) for the base information about the routes configuration.
- Nested route has access to its own parameters and all parent's route parameters:

  ```javascript
  {
    path: '/users/:id(\\d+)',
    children: [
      {
        path: '/',
        name: 'USER_DETAIL',
        component: ViewUserDetail,
      },
      {
        path: '/message/:msgId(\\d+)',
        name: 'USER_MESSAGE',
        component: ViewUserMessageDetail,
      }
    ],
  },
  {
    path: '/services',
    component: ViewServices,
    children: [
      path: '/design',
      component: ViewServicesDesign,
    ]
  }

  // Resolved USER_MESSAGE route will have access to
  // id, and msgId params.
  ```

- The **component** could be omitted if the route has nested routes (please see the example above), it this case ViewRouter component will be used to pass through the components, please see [Router View](#router-view-component) for more details.
- If the route has nested routes, and it has defined component, than the component must internally use [Router View](#router-view-component) component to pass though the routed view components on it's own nested routes.
- All routes should start with trailing slash.

### Router Link Component

```html
<script>
import RouterLink from '@etip/svelte-router-from-crutch-builder/component/link';

<!-- URL -->
<RouterLink to='/services/design'>Navigate by URL</RouterLink>

<!-- Location Object -->
<RouterLink to={{name: 'USER_DETAIL', params:{id: 5}}>Navigate by Location</RouterLink>
```

The route link is the base component for routing action. The route link renders a **< a >** DOM element with click action triggering route change. The active class is auto-resolved. Component parameters:

| Property    | Description                                                                                                                                                                                                               | Type             |
| :---------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------- |
| to          | navigation URL or navigation Location, please see [Location Object](#location-object) for more details.                                                                                                                   | string, Location |
| replace     | Replace rather the than push into the history, defaults to false.                                                                                                                                                         | boolean          |
| exact       | The default active class matching behavior is inclusive match, i.e. it matches if the URL starts with link url. In the exact mode, the url must be be the same (excluding query param and hash param). Defaults to false. | boolean          |
| cls         | Link base class name, defaults to ''.                                                                                                                                                                                     | string           |
| activeClass | Link active class name, if not defined, it defaults to the active class defined on the router.                                                                                                                            | string           |
| disabled    | Disable navigation action, and set "disabled" css class. Defaults to false.                                                                                                                                               | boolean          |

#### Router Link Events

```html
<script>
  import RouterLink from "@etip/svelte-router-from-crutch-builder/component/link";

  // Navigation has been completed
  function handleOnCompleted() {}

  // Navigation has been aborted
  function handleOnAborted() {}
</script>

<RouterLink
  to="/services/design"
  on:completed="{handleOnCompleted}"
  on:aborted="{handleOnAborted}"
  >Navigate by URL</RouterLink
>
```

### Router View Component

```html
<script>
  import RouterView from "@etip/svelte-router-from-crutch-builder/component/view";
</script>

<RouterView />
```

- The route view acts as **SLOT** for resolved route component, i.e. it will be replaced run-time with the component defined on the given route.
- For nested routes, if the parent route (the one having nested routes) has defined it's own component, the component must use internally ViewRouter component to pass through the nested route components. If the component is not defined on the parent route, it auto pass through.

## Advanced

### Programmatic Navigation

Besides the [Router Link Component](#router-link-component), the route could be changed like so:

```javascript
import { router } from "@etip/svelte-router-from-crutch-builder";

// URL
$router.push("/services/design");

// Location Object
$router.push({
  name: "USER_DETAIL",
  params: {
    id: 5,
  },
  query: {
    code: "system_code",
  },
  hash: "sidebar",
});
```

> Note: the **router** can be accessed after the **createRouter** function is executed, and it must be accessed as **$router** since it is Svelte read-able store object.

More information:

- [Location Object](#location-object)
- [Router Methods](#router-methods)

### Navigation Guard

The navigation guards are primarily used to guard navigations either by redirecting it or canceling it.

#### Create a Navigation Guard

```javascript
import { router } from "@etip/svelte-router-from-crutch-builder";

// The guard callback is called when a navigation occurs,
// the next function must be called to continue the navigation.
$router.navigationGuard((from, to, next) => {
  next();
});
```

> Note: the **router** can be accessed after the **createRouter** function is executed, and it must be accessed as **$router** since it is Svelte read-able store object.

"from, to" are [Route Object](#route-object).

#### Navigation Guard Next Action

##### Continue

```javascript
next();
```

Move on to the next hook in the pipeline. If no hooks are left, the navigation is confirmed.

##### Abort

```javascript
next(false);
```

Abort the current navigation.

##### Redirect

```javascript
// URL
next("/services/design");

// Location Object
next({
  name: "USER_DETAIL",
  params: {
    id: 5,
  },
  query: {
    code: "system_code",
  },
  hash: "sidebar",
});
```

Abort the current navigation and redirect to new navigation.

More information:

- [Location Object](#location-object)

##### Error

```javascript
next(new Error("internal error"));
```

Abort the current navigation, and trigger onError event on the router.

More information:

- [onError](#onerror)

### Lazy Loaded Component

View components could be loaded asynchronously, i.e. lazy loading, e.g:

app.svelve

```html
<script>
  import createRouter from '@etip/svelte-router-from-crutch-builder';
  import RouterView from '@etip/svelte-router-from-crutch-builder/component/view';

  // View components
  import ViewHome from './views/home.svelte';
  import View404 from './views/404.svelte';

  // Webpack dynamic import
  const asyncView = (view) => {
    return new Promise((resolve) => {
      const component = import(/* webpackChunkName: "view-[request]" */ `./view/${view}.svelte`);
      resolve(component);
    });
  };

  createRouter({
    routes: [
      {
        path: '/',
        name: 'HOME',
        component: ViewHome,
      },
      {
        path: 'some-page',
        component: asyncView('some-page'),
      }
      {
        path: '*',
        component: View404,
      },
    ],
  });
</script>

<RouterView />
```

**Note**: To load async component, the load function must return a Promise, and resolve must return a module object:

```javascript
{
  // The default (default module export function) must the Svelte Component function
  // Please see the webpack example above.
  default: function(){}
}.
```

## API

To see the details code documentation, please read the [Code Documentation](https://spaceavocado.github.io/svelte-router/)

### Create Router

```javascript
import createRouter from "@etip/svelte-router-from-crutch-builder";

// Please see the opts below.
const routerOpts = {};

// Router is Svelte read-able store, therefore it must be
// accessed with $ prefix, e.g. $router.
// To access the router instance in other files, please see "Access Router Instance".
const router = createRouter(routerOpts);
```

> Throws when the route options are invalid.

#### Router Options

| Property    | Description                                                                          | Type     |
| :---------- | :----------------------------------------------------------------------------------- | :------- |
| mode        | History mode. Supported values: 'HISTORY', 'HASH'.                                   | string   |
| basename    | The base URL of the app, defaults to ''.                                             | string   |
| hashType    | Hash type. Relevant only for HISTORY_MODE.HASH. Supported values: 'PUSH', 'REPLACE'. | string   |
| routes      | router routes.                                                                       | object[] |
| activeClass | CSS class applied on the active route link. Defaults to "active".                    | string   |

> Note: History modes could be accessed like so:

```javascript
import { ROUTER_MODE } from "@etip/svelte-router-from-crutch-builder";
```

> Note: Hash types could be accessed like so:

```javascript
import { HASH_TYPE } from "@etip/svelte-router-from-crutch-builder";
```

More information:

- [Route Configuration](#route-configuration)

#### Access Router Instance

```javascript
import { router } from "@etip/svelte-router-from-crutch-builder";
```

> Note: It must be accessed as **$router** since it is the Svelte read-able store object, to resolved auto subscribe/unsubscribe.

### Router Methods

All route methods are accessible on the router, please see [Access Router Instance](#access-router-instance).

#### start

```javascript
$router.start();
```

This method is auto-called by the root [Router View Component](#router-view-component); it handles the on page load route resolution.

#### push

Push to navigation.

```javascript
$router.push(rawLocation, onComplete, onAbort);
```

> Throws when the rawLocation is invalid or when the path is invalid.

Parameters:

| Name        | Description                              | Type                                 |
| :---------- | :--------------------------------------- | :----------------------------------- |
| rawLocation | raw path or location object.             | string, [Location](#location-object) |
| onComplete  | On complete callback function. Optional. | function                             |
| onAbort     | On abort callback function. Optional.    | function                             |

#### replace

Replace in navigation.

```javascript
$router.replace(rawLocation, onComplete, onAbort);
```

> Throws when the rawLocation is invalid or when the path is invalid.

Parameters:

| Name        | Description                              | Type                                 |
| :---------- | :--------------------------------------- | :----------------------------------- |
| rawLocation | raw path or location object.             | string, [Location](#location-object) |
| onComplete  | On complete callback function. Optional. | function                             |
| onAbort     | On abort callback function. Optional.    | function                             |

#### back

Go one step back in the navigation history.

```javascript
$router.back();
```

#### forward

Go one step forward in the navigation history.

```javascript
$router.forward();
```

#### go

Go to a specific history position in the navigation history.

```javascript
$router.go(n);
```

Parameters:

| Name | Description                                                | Type   |
| :--- | :--------------------------------------------------------- | :----- |
| n    | number of steps to forward or backwards (negative number). | number |

#### routeURL

Generate route URL from the the raw location.

```javascript
const url = $router.routeURL(rawLocation);
```

Parameters:

| Name        | Description          | Type                         |
| :---------- | :------------------- | :--------------------------- |
| rawLocation | raw location object. | [Location](#location-object) |

#### navigationGuard

Register a navigation guard which will be called whenever a navigation is triggered. All registered navigation guards are resolved in sequence. Navigation guard must call the next() function to continue the execution of navigation change. Please see [Navigation Guard](#navigation-guard)

```javascript
const unregister = $router.navigationGuard(guard);

// To unregister the navigation guard:
unregister();
unregister = null;
```

Parameters:

| Name  | Description                                                                                                                       | Type     |
| :---- | :-------------------------------------------------------------------------------------------------------------------------------- | :------- |
| guard | guard callback function with (from, to, next) signature. Please see [Navigation Guard Next Action](#navigation-guard-next-action) | function |

"from, to" are [Route Object](#route-object).

#### onBeforeNavigation

Register a callback which will be called before execution of navigation guards.

```javascript
const unregister = $router.onBeforeNavigation(callback);

// To unregister the event:
unregister();
unregister = null;
```

Parameters:

| Name     | Description                                    | Type     |
| :------- | :--------------------------------------------- | :------- |
| callback | callback function with fn(from, to) signature. | function |

"from, to" are [Route Object](#route-object).

#### onNavigationChanged

Register a callback which will be called when all navigation guards are resolved, and the final navigation change is resolved.

```javascript
const unregister = $router.onNavigationChanged(callback);

// To unregister the event:
unregister();
unregister = null;
```

Parameters:

| Name     | Description                                    | Type     |
| :------- | :--------------------------------------------- | :------- |
| callback | callback function with fn(from, to) signature. | function |

"from, to" are [Route Object](#route-object).

#### onError

Register a callback which will be called when an error is caught during a route navigation.

```javascript
const unregister = $router.onError(callback);

// To unregister the event:
unregister();
unregister = null;
```

Parameters:

| Name     | Description                                 | Type     |
| :------- | :------------------------------------------ | :------- |
| callback | callback function with fn(error) signature. | function |

### Location Object

| Property | Description                                                                                                                                             | Type   |
| :------- | :------------------------------------------------------------------------------------------------------------------------------------------------------ | :----- |
| name     | Name of the route.                                                                                                                                      | string |
| params   | Route params dictionary object, if the route has defined dynamic route parameters, this object is required, with valid params to resolve the route URL. | object |
| query    | Route query dictionary object. Optional.                                                                                                                | object |
| hash     | Route hash parameter. Optional.                                                                                                                         | string |

```javascript
// Location object example
{
  name: 'USER_DETAIL',
  params: {
    id: 5,
  },
  query: {
    code: 'system_code',
  },
  hash: 'sidebar',
}

// Considering that USER_DETAIL is /user/:id(\\d+)
// the resolved URL will be:
// /user/5?code=system_code#sidebar
```

### Route Object

| Property | Description                                                                                                                                                                               | Type     |
| :------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------- |
| name     | name of the route.                                                                                                                                                                        | string   |
| path     | location path use to resolve the route.                                                                                                                                                   | string   |
| hash     | url hash.                                                                                                                                                                                 | string   |
| fullPath | the full resolved URL including query and hash.                                                                                                                                           | string   |
| params   | route resolved params.                                                                                                                                                                    | object   |
| query    | query parameters.                                                                                                                                                                         | object   |
| meta     | route meta.                                                                                                                                                                               | object   |
| action   | route action.                                                                                                                                                                             | string   |
| matched  | resolved route records, please see [Route Record Object](#route-record-object) In the case of nested route, it contains all matched routes, starting from root to the deepest route node. | object[] |

### Route Record Object

| Property  | Description                                                                                                    | Type                      |
| :-------- | :------------------------------------------------------------------------------------------------------------- | :------------------------ |
| path      | location path use to resolve the route.                                                                        | string                    |
| name      | name of the route.                                                                                             | string                    |
| component | svelte component.                                                                                              | string                    |
| meta      | route meta.                                                                                                    | string                    |
| params    | route resolved params.                                                                                         | string                    |
| props     | props passed to component, please see [Passing Props to Route Components](#passing-props-to-route-components). | boolean, object, function |

## Changes

To see the changes that were made in a given release, please lookup the tag on the releases page. The full changelog could be seen here [changelog.md](https://github.com/spaceavocado/svelte-router/blob/master/changelog.md)

## About

This project was inspired by [Vue Router](https://router.vuejs.org/), designed mainly to explore and test [Svelte](#https://svelte.dev) in SPA realm. Any feedback, contribution to this project is welcomed.

The project is in a beta phase, therefore there might be major changes in near future, the annotation should stay the same, though.

## Contributing

When contributing to this repository, please first discuss the change you wish to make via issue, email, or any other method with the owners of this repository before making a change.

### Pull Request Process

1. Fork it
2. Create your feature branch (git checkout -b ft/new-feature-name)
3. Commit your changes (git commit -am 'Add some feature')
4. Push to the branch (git push origin ft/new-feature-name)
5. Create new Pull Request
   > Please make an issue first if the change is likely to increase.

## License

Svelte Router is released under the MIT license. See [LICENSE.txt](https://github.com/spaceavocado/svelte-router/blob/master/LICENSE.txt).
