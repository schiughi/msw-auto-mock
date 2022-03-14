# openapi-to-msw

![GitHub Workflow Status](https://github.com/schiughi/openapi-to-msw/workflows/Test/badge.svg)
[![npm version](https://badge.fury.io/js/openapi-to-msw.svg)](https://badge.fury.io/js/openapi-to-msw)

A cli tool to generate random mock data from OpenAPI descriptions for [msw](https://github.com/mswjs/msw) and Storybook.

## Usage

Install:

```sh
yarn add openapi-to-msw faker-js/faker -D
```

Read from your OpenAPI descriptions and output generated code:

```sh
# can be http url or a file path on your machine, support both yaml and json.
npx openapi-to-msw http://your_openapi.json -o ./mock.js
```

See [here for Github API example](https://raw.githubusercontent.com/schiughi/openapi-to-msw/main/example/src/mock.js). The msw mocking handlers was generated by following command:

```sh
npx openapi-to-msw https://raw.githubusercontent.com/github/rest-api-description/main/descriptions/ghes-3.3/ghes-3.3.json --output ./example/src/mock.js
```

Integrate with msw, see [Mock Service Worker's doc](https://mswjs.io/docs/getting-started/integrate/browser) for more detail:

```sh
# Install msw
yarn add msw --dev

# Init service worker
npx msw init public/ --save
```

Then import those mock definitions in you app entry:

```js
import { getHandlers } from "openapi-to-msw"
import { factories } from './mock';

if (process.env.NODE_ENV === 'development') {
  startWorker(getHandlers(factories));
}
```

### Storybook Integration
```js
// preview.js
import {
  initialize,
  mswDecorator,
} from "msw-storybook-addon";
import { getHandlersWithKey } from "openapi-to-msw"
import { factories } from './mock';

initialize();

export const parameters = {
  msw: {
    handlers: getHandlersWithKey(factories),
  },
};
```

```js
// SomeComponent.stories.tsx
export default {
  component: SomeComponent,
}

export const Default: Story = {}

// for Error case
const customMockData = {
  ...
}

const customHandlers = getHandlersWithKey(customMockData, { statusCode: "error" })

export const Error: Story = {
  parameters: {
    msw: {
      handlers: {
        ...customHandlers
      }
    },
  },
}
```

Run you app then you'll see a successful activation message from Mock Service Worker in your browser's console.


## Options

 - `-o, --output`: specify output file path or output to stdout.
 - `-m, --max-array-length <number>`: specify max array length in response, it'll cost some time if you want to generate a huge chunk of random data.
 - `-t, --match <keywords>`: specify keywords to match if you want to generate mock data only for certain requests, multiple keywords can be seperated with comma.
 - `-h, --help`: show help info.

### Response Generation
- openapi-to-msw generates random value according to `type`, `format`, and `x-faker` property like Stoplight/Prism

```yml
Pet:
  type: object
  properties:
    id:
      type: integer
      format: int64
    name:
      type: string
      x-faker: name.firstName
      example: doggie
    photoUrls:
      type: array
      items:
        type: string
        x-faker: image.imageUrl
```

```json
{
  "id": 12608726,
  "name": "Addison",
  "photoUrls": [
    "http://lorempixel.com/640/480",
    "http://lorempixel.com/640/480",
    "http://lorempixel.com/640/480",
    "http://lorempixel.com/640/480"
  ]
}
```
