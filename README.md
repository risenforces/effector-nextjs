# Effector Next

A HOCs that brings Effector and Next.js together

## Installation

```bash
npm install effector-next
```

or yarn

```bash
yarn add effector-next
```

**effector-next** requires `effector`, `effector-react` to be installed

**effector/babel-plugin** is necessary if you do not want to manually name the units

## Usage

1. To load the initial state on the server, you must attach `withFork` wrapper to your `_document` component

   <details>
   <summary>pages/_document.jsx</summary>

   ```jsx
   import Document from "next/document";
   import { withFork } from "effector-next";

   const enhance = withFork({ debug: false });

   export default enhance(Document);
   ```

   </details>

2. To get the initial state on the client and drop it into the application, you must attach `withHydrate` wrapper to your `_app` component

   <details>
   <summary>pages/_app.jsx</summary>

   ```jsx
   import { withHydrate } from "effector-next";
   import App from "next/app";

   const enhance = withHydrate();

   export default enhance(App);
   ```

   </details>

3. To bind events/stores on the server to the scope, add aliases from `effector-react` to`effector-react/ssr` in `next.config.js`

   <details>
   <summary>next.config.js</summary>

   ```js
   const { withEffectoReactAliases } = require("effector-next/tools");

   const enhance = withEffectoReactAliases();

   module.exports = enhance({});
   ```

   </details>

4. Replace import from `"effector"` to `"effector-next"`

   ```diff
   - import { createEvent, forward } from "effector"
   + import { createEvent, forward } from "effector-next"
   ```

5. Configure what event will be triggered when the page is requested from the server using `withStart`

   <details>
   <summary>pages/index.js</summary>

   ```jsx
   import React from "react";
   import { withStart } from "effector-next";
   import { useStore } from "effector-react";

   import { pageLoaded } from "../model";

   const enhance = withStart(pageLoaded);

   function HomePage() {
     return (
       <div>
         <h1>Hello World</h1>
       </div>
     );
   }
   
   export default enhance(HomePage);
   ```

   </details>

### Example

```jsx
// models/index.js
import { forward, createEvent, createStore, createEffect } from "effector-next";

export const pageLoaded = createEvent();

const effect = createEffect({
  handler() {
    return Promise.resolve({ name: "someName" });
  },
});

export const $data = createStore(null);

$data.on(effect.done, (_, { result }) => result);

forward({
  from: pageLoaded,
  to: effect,
});
```

```jsx
// pages/index.jsx
import React from "react";
import { withStart } from "effector-next";
import { useStore } from "effector-react";

import { $data, pageLoaded } from "../models";

const enhance = withStart(pageLoaded);

function HomePage() {
  const data = useStore($data);

  return (
    <div>
      <h1>Data preloaded on the server:</h1>
      {JSON.stringify(data)}
    </div>
  );
}

export default enhance(HomePage);
```

## Configuration

The `withFork` accepts a config object as a parameter:

- `debug` (optional, boolean) : enable debug logging

## Server payload

When the unit passed to `assignStart` is called, the object will be passed as a payload:

- `req` : incoming request
- `res` : serever response
- `cookies` : parsed cookies
- `pathname` : path section of `URL`
- `query` : query string section of `URL` parsed as an object.
