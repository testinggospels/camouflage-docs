# HTTP

HTTP module of camouflage lets you mock your backends based on **http, https and http2** protocols. You can create a camouflage http object from `CamouflageHttp` class and configure it to serve mocks for your incoming requests.

Start by installing required dependencies

```bash
npm i @camouflage/helpers @camouflage/http
```

- You can create the camouflage object without any parameters, and load the required options as needed

```javascript
import CamouflageHttp from "@camouflage/http";

const camouflageHttp: CamouflageHttp = new CamouflageHttp();

camouflageHttp.loadConfigFromJson("./config.json");
camouflageHttp.setServerOptions(options);
camouflageHttp.setSecureServerOptions(options);
camouflageHttp.setupCacheWithOptions(options);
camouflageHttp.setupCorsWithOptions(options);

camouflageHttp.start();
```

- Or you can create the camouflage object with the options

```javascript
import CamouflageHttp, { CamouflageHttpConfig } from "@camouflage/http";
import type http from "http";
import type https from "https";
import type apicache from "apicache";
import type cors from "cors";

const config: CamouflageHttpConfig = {};
const httpOptions: http.ServerOptions = {};
const httpsOptions: https.ServerOptions = {};
const cacheOptions: apicache.Options = {};
const corsOptions: cors.Options = {};

const camouflageHttp: CamouflageHttp = new CamouflageHttp(config, httpOptions, httpsOptions, cacheOptions, corsOptions);
camouflageHttp.start();
```

## Available methods

###### _getHelpers = (): Helpers_

When you create a CamouflageHttp object, it automatically creates an instance of helpers. You can use `getHelpers()` to get a reference to this helpers object. This is useful when you want add custom helpers that are specific to your requirements.

```javascript
import Helpers from "@camouflage/helpers";

const helpers: Helpers = camouflageHttp.getHelpers();

helpers.addHelper("ping", (context: any) => {
  return "pong";
});

camouflageHttp.start();
```

You can take a look at how inbuilt helpers have been created, in case you want to understand how custom helpers can be created. Refer to the [helper source code](UPDATE THIS).

###### _loadConfigFromJson(configFilePath: string): void_

While you can include your config as part of your code and ensure the types yourself, you may at times want to maintain the configuration for your camouflage server separate from the application code. This is usually a good practice from maintainability point of view, or even practical if you want to maintain multiple config files for different usecases.

`loadConfigFromJson` lets you load a config via a .json file. You don't need to worry about validating your config file, Camouflage takes care of validating your config and prints relevant errors which help you fix your config files, if you miss something.

###### _setServerOptionsHttp(options: http.ServerOptions): void_

Depending on your use case, you might want to set additional options. Use `setServerOptionsHttp2` to pass those options to camouflage server. Read more about the available options in the official [documentation](https://nodejs.org/api/http.html#httpcreateserveroptions-requestlistener)

###### _setServerOptionsHttps(options: https.ServerOptions): void_

In case you are creating an https server, you would need to use `setServerOptionsHttps` to provide the necessary credentials. You can use it to add other [available options](https://nodejs.org/api/https.html#httpscreateserveroptions-requestlistener) as well to your https servers.

###### _setServerOptionsHttp2(options: spdy.server.ServerOptions): void_

In case you are creating an http2 server, you would need to use `setServerOptionsHttps` to provide the necessary credentials. You can use it to add other [available options](https://www.npmjs.com/package/spdy#options) as well to your http2 servers.

###### _setupCacheWithOptions(options: apicache.Options): void_

Camouflage HTTP uses, `apicache` to configure a cache middleware for your mocks. By default, the cache is saved in memory and you can provide a ttl in seconds via config. However in case you want more control over the options, you can fine tune the settings using `setupCacheWithOptions`.

Following example shows how you can cache with redis instead of in memory.

```bash
npm i redis
```

```javascript
import redis from "redis";
import type apicache from "apicache";

let cacheOptions: apicache.Options = {
  redisClient: redis.createClient(),
  respectCacheControl: true,
  statusCodes: {
    exclude: [401, 403, 404],
    include: [200],
  },
  // more options
};

camouflageHttp.setupCacheWithOptions(cacheOptions);
camouflageHttp.start();
```

You can refer to [apicache documentation](https://www.npmjs.com/package/apicache#available-options-first-value-is-default) for more details on the available options and how to configure them.

###### _setupCorsWithOptions(corsOptions: cors.CorsOptions): void_

Camouflage uses `cors` middleware to configure cors for your mocks. By default, cors is enabled for all origins and methods, however you can control this by providing camouflage a corsOptions before you start the server.

```javascript
import type cors from "cors";

const corsOptions: cors.Options = {
  origin: ["http://localhost:3000", "http://instrukti.com/"],
  methods: "GET,POST",
};
camouflageHttp.setupCorsWithOptions(corsOptions);
camouflageHttp.start();
```

Read more about available options on [cors documentation](https://www.npmjs.com/package/cors#configuration-options).

###### _setupValidationWithOptions = (validationOpts: OpenApiValidatorOpts): void_

Allows you to setup validations using an Open API 3 specification. Read more on the usage in the [OpenAPI Validation](#openapi-validation) section.

###### _setupCompressionWithOptions = (compressionOpts: CompressionOptions): void_

Allows you to setup compression options provided by [compression middleware](https://www.npmjs.com/package/compression#options). Read more on usage in the [compression](#compression) section.

###### _addHook = (route: string, event: "onRequest" | "beforeResponse" | "afterResponse", fn: CamouflageHook): void_

Apart from the camouflage helpers, ability to create custom helpers, camouflage allows you to add hooks to specific routes. You can add hooks to listen to certain events and manipulate the request/response as you wish.

Available hooks are:

1. **onRequest**: `onRequest` hooks are executed as soon as camouflage recieves the request. You can use `onRequest` hooks to intercept the incoming request. You can either **make some changes to your incoming request object** and then let camouflage run the response builder on the modified request, or **bypass the camouflage response builder entirely** and send the response from within the hook itself without refering to any mock files.
2. **beforeResponse**: `beforeResponse` hooks are executed right before camouflage is about to send the response. `beforeResponse` hooks are useful when you want camouflage response builder to use the provided mock file to build a response, however you want to modify the response before it's sent.
3. **afterResponse**: `afterResponse` hooks are executed once camouflage has sent the response. `afterResponse` hooks is useful for logging or other such activities.

In the following section, you'll see how we can configure and use these hooks.

###### _start(): void_

Self explanatory. Starts the camouflage http server.

###### _stop(): void_

Self explanatory. Stops the camouflage http server.

###### _restart(): void_

Self explanatory. Restarts the camouflage http server.

## Hooks

#### `onRequest`

In the following example, we'll see how can we use `onRequest` hook to intercept the incoming request:

```javascript
camouflageHttp.addHook("/user/:userId/wallet/:walletId", "onRequest", (req, res) => {
  console.log("Hello from hook", req.route.path); // You can do some logging
  res.set("Sent-From", "onRequestHook"); // You can set some headers
  // You can check some conditions
  if (req.param.userId === 1) {
    // If the condition passes, you can choose to bypass camouflage entirely by sending the response from within the hook
    res.set("Content-Type", "application/json");
    const body = {
      message: "Sent from onRequestHook",
    };
    res.status(200).send(JSON.stringify(body));
  }
  // Or you can do nothing and let camouflage take over after you are done modifying the request/response objects
});
```

#### `beforeResponse`

Similarily you can use `beforeResponse`, to intercept and manipulate the responses generated by camouflage response builder

```javascript
camouflage.addHook("/user/:userId/wallet/:walletId", "beforeResponse", (req, res, camouflageResponse: CamouflageResponse | undefined) => {
  if (camouflageResponse) console.log(camouflageResponse);
  res.set("Added-In-Hook", "SomeHeader");
});
```

#### `afterResponse`

Finally, you can use `afterResponse` hooks to measure time or log some messages as shown below

```javascript
let time = 0;
let startTime = 0;
camouflageHttp.addHook("/user/:userId/wallet/:walletId", "onRequest", (req, res) => {
  startTime = Date.now();
  console.log("Hello from hook", req.route.path);
});
camouflageHttp.addHook("/user/:userId/wallet/:walletId", "afterResponse", (req, res) => {
  time = Date.now() - startTime;
  console.log(time);
  time = 0;
  startTime = 0;
});
```

## Camouflage Http Configuration

You can provide following configuration options in your `config.json` file and load it to camouflage before you start the server

```json
{
  "log": {
    "enable": true, // enables or disables the logs entirely
    "level": "trace", // if enable=true, sets the log level. Available options are "fatal", "error", "warn", "info", "debug", "trace"
    "disableRequestLogs": true // if not disabled, each incoming request will be logged.
  },
  "http": {
    "enable": true, // enables or disables http server
    "port": 8080 // port on which http server would be available
  },
  "https": {
    "enable": false, // enables or disables https server
    "port": 8443 // port on which https server would be available
  },
  "http2": {
    "enable": false, // enables or disables http2 server
    "port": 9443 // port on which http2 server would be available
  },
  "monitoring": true, // if enabled, provides a /monitoring endpoint with some dashboards for monitoring
  "cache": {
    "enable": true, // enables or disables cache
    "timeInSeconds": 5 // if enabled, sets cache ttl to 5 seconds
  },
  "enableCors": true, // enables or disables cors
  "mocksDir": "./mocks" // location of the directory where your mocks live.
}
```

## Folder structure

The way you organize your directories inside the `mocksDir`, determine how your endpoints will be available. Following examples will help you understand the folder structure you need to maintain. We assume that you have configured your `mocksDir` as `./mocks` in following examples

### GET Request to /hello/world

- Create a directory `./mocks/hello/world`.
- Create a `GET.mock` file inside it with your required response.

```javascript
HTTP/1.1 200 OK
X-Provided-By: CamouflageHttp
Content-Type: application/json

{
    "hello": "world"
}
```

### GET Request to /user/:userId/wallet/:walletId

- Create a directory `/user/:userId/wallet/:walletId`
- Create a `GET.mock` file inside it with your required response

```javascript
HTTP/1.1 200 OK
Content-Type: application/json

{
  "userId": {{capture from='path' key='userId'}},
  "walletId": {{capture from='path' key='walletId'}},
  "createdAt": "{{now format='epoch'}}",
}
```

!!! note

    Notice the use of `helpers` in the above mock. Alongwith the inbuilt `now` helper, camouflage http module provides some additional helpers, `capture` being one of them. In the following section, you can read more about additional helpers that specific to http module.

Similarly you can create PUT.mock, DELETE.mock etc in your intended path as required by your mocked endpoint.

!!! warning

    If you are coming from the previous version of Camouflage, note that we have dropped support for wildcards i.e. __ / double underscores. They were primarily intended to be used for dynamic path params which are now handled the way express handles them, which makes it more robust and easy to understand.

## Camouflage HTTP Helpers

In addition to the available helpers that come with `@camouflage/helpers` module, camouflage http module provides some protocol specific helpers.

### `capture` Helper

Usage:

- **{{capture from='query' key='firstName'}}** - Pretty self-explanatory, but if your endpoint looks like `/hello/world?firstName=John&lastName=Wick`. And your response is `{"message": "Hello Wick, John"}`, you can make the response dynamic by formatting your response as

```json
{
  "message": "Hello {{capture from='query' key='lastName'}}, {{capture from='query' key='firstName'}}"
}
```

- **{{capture from='cookies' key='mycookie'}}** - For cookies, you'd need to specify a key to capture a value.
- **{{capture from='path' key='walletId'}}** - For path, you'd need to specify a key to capture a value.
- **{{capture from='headers' key='Authorization'}}** - For headers, you'd need to specify a key to capture a value.
- **{{capture from='body' using='jsonpath' selector='$.lastName'}}** - To capture values from the request body, your options are either `using='regex'` or `using='jsonpath'`. Selector will change accordingly.

### `file` Helper

Usage:

**{{file path='/location/of/the/image/or/text/or/any/file'}}**: If you want to serve a file as a response, maybe an image, or text file, a pdf document, or any type of supported files, use file helper to do so. Content-Type header is decided automatically based on the file type. An example is shown below:

```javascript
HTTP/1.1 200 OK

{{file path="./docs/camouflage.png"}}
```

### `state` Helper

Usage: State helper gets the mocked state value using a key send within the cookie header. If no value is found it will use the default context within the block.

For example:

```javascript
{
    "has_pending_order": {{#state key='has-pending-order'}}false{{/state}},
    "cart": {{#state key='cart'}}[
        {"id": 999, "name": "default prod"}
    ]{{/state}}
}
```

To set a value just send cookie with a specific prefix.

```javascript
const prefix = "mocked-state";
const key = "has-pending-order";
setCookie(`${prefix}-has-pending-order`, "true");
setCookie(`${prefix}-cart`, '[{id: 1, name: "prod1"}, {id: 2, name: "prod2"}]');
```

#### Usage in Cypress

If you use Camouglage with [Cypress](https://www.cypress.io/) you could add the following custom command to make life easier.

```javascript
/**
 * Custom cypress command to set a mocked state
 */
Cypress.Commands.add("setState", (key: string, value: unknown) => {
  cy.setCookie(`mocked-state-${key}`, typeof value === "string" ? value : JSON.stringify(value));
});
```

Then in your tests

```javascript
cy.setState("has_pending_order", true);
cy.setState("cart", [
  { id: 1, name: "prod1" },
  { id: 2, name: "prod2" },
]);
```

## What data to put in .mock files

Camouflage expects a raw HTTP Response to be placed in the .mock files. Please refer to this [Wikipedia](https://en.wikipedia.org/wiki/HTTP_message_body) page, if you are not sure what the response looks like.

Each mock file can have the HTTP Responses in following manner:

- One response per .mock file.
- Multiple responses in one .mock file with conditions defined to help Camouflage decide which response should be sent under what conditions. (Read Handlebars section for more)
- Multiple responses separated by Camouflage's delimiter i.e. "====" (four equals). Camouflage will pick one response at random and send it to the client. An example of this can be found here

The data you want in your mock file can be easily fetched using a curl command with -i -X flags as shown in the example below.

```bash
curl -i -X GET https://jsonplaceholder.typicode.com/users/1 > GET.mock
```

Running this command, gives you a `GET.mock` file with following content. Modify it according to your requirement and place it in the location `./mocks/users/:userId`, and you have successfully mocked jsonplaceholder API.

```javascript
HTTP/1.1 200 OK
date: Sat, 17 Apr 2021 05:21:51 GMT
content-type: application/json; charset=utf-8
content-length: 509
set-cookie: __cfduid=ddf6b687a745fea6ab343400b5dfe9f141618636911; expires=Mon, 17-May-21 05:21:51 GMT; path=/; domain=.typicode.com; HttpOnly; SameSite=Lax
x-powered-by: Express
x-ratelimit-limit: 1000
x-ratelimit-remaining: 998
x-ratelimit-reset: 1612952731
vary: Origin, Accept-Encoding
access-control-allow-credentials: true
cache-control: max-age=43200
pragma: no-cache
expires: -1
x-content-type-options: nosniff
etag: W/"1fd-+2Y3G3w049iSZtw5t1mzSnunngE"
via: 1.1 vegur
cf-cache-status: HIT
age: 14578
accept-ranges: bytes
cf-request-id: 097fe04d2c000019d97db7d000000001
expect-ct: max-age=604800, report-uri="https://report-uri.cloudflare.com/cdn-cgi/beacon/expect-ct"
report-to: {"endpoints":[{"url":"https:\/\/a.nel.cloudflare.com\/report?s=%2FkpNonG0wnuykR5xxlGXKBUxm5DN%2BI1PpQ0ytmiw931XaIVBNqZMJLEr0%2F3kDTrOhbX%2FCCPZtI4iuU3V%2F07wO5uwqov0d4c12%2Fcdpiz7TIFqzGkr7DwUrzt40CLH"}],"max_age":604800,"group":"cf-nel"}
nel: {"max_age":604800,"report_to":"cf-nel"}
server: cloudflare
cf-ray: 6413365b7e9919d9-SIN
alt-svc: h3-27=":443"; ma=86400, h3-28=":443"; ma=86400, h3-29=":443"; ma=86400

{
  "id": 1,
  "name": "Leanne Graham",
  "username": "Bret",
  "email": "Sincere@april.biz",
  "address": {
    "street": "Kulas Light",
    "suite": "Apt. 556",
    "city": "Gwenborough",
    "zipcode": "92998-3874",
    "geo": {
      "lat": "-37.3159",
      "lng": "81.1496"
    }
  },
  "phone": "1-770-736-8031 x56442",
  "website": "hildegard.org",
  "company": {
    "name": "Romaguera-Crona",
    "catchPhrase": "Multi-layered client-server neural-net",
    "bs": "harness real-time e-markets"
  }
}
```

Another, easier, approach to create mocks is by installing the [REST Client VS Code Extension](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) and using it to fetch the required data for mocks.

- Launch VS Code and install "REST Client" Extension by Huachao Mao or simply open the link above.
- Create a .http file in your project to document your actual http endpoints and make the requests.
- Visit [REST Client github repository](https://github.com/Huachao/vscode-restclient) for more details on usage

### Line breaks

!!! note

    Camouflage by default looks for the OS specific line breaks. For example, if you are on MacOS or Unix based systems, the default line break/new line is `\n`, whereas on windows it's `\r\n`. This is known to cause issues if your development environment and testing environment are different for Camouflage. For example, if you have created your mock file on a windows machine and uploaded it to a Camouflage server running on linux, your mocks might not work as expected. Or in case your text editor's line break settings do not match your OS default line break, you might not get an expected response.

    Though Camouflage detects new lines used in the file irrespective of the OS default, you should not face any issues. However, if you face any issues where you are getting a blank response or any unexpected response, please create an issue attaching your log files. REMEMBER TO REMOVE SENSITIVE DATA, IF ANY, FROM YOUR LOGS.

## Request Matching

There are scenarios when you would need to change your response based on some conditions met by fields on request objects. For example, if the end user passes an Authorization header, you'd want to send a 200 OK response if not you'd want to send a 401 Unauthorized response.

To do so you can utilize the beauty of handlebars. Simply provide an if else condition and you are good to go. Let's understand how to do this in the following example:

You expect the user to call the endpoint `/hello` in two ways.

1. By simple making a GET request to `/hello`; Or
2. By adding a query parameter name in the GET request to `/hello`. i.e. `/hello?name=John`

Based on how the user calls the API, you'd want to send different responses. Let's see how we can achieve the desired result.

Start by creating a `GET.mock` file at the location `./mocks/hello`. Paste the following content in the mock file.

```javascript
{{#if request.query.name}}
HTTP/1.1 200 OK
X-Provided-By: CamouflageHttp
Content-Type: application/json

{
    "greeting": "Hello {{capture from='query' key='name'}}",
    "phone": {{randomValue length=10 type='NUMERIC'}},
    "dateOfBirth": "{{now format='MM/DD/YYYY'}}",
    "test": "{{randomValue}}"
}
{{else}}
HTTP/1.1 200 OK
X-Provided-By: CamouflageHttp
Content-Type: application/json

{
    "greeting": "Hello World",
    "phone": {{randomValue length=10 type='NUMERIC'}},
    "dateOfBirth": "{{now format='MM/DD/YYYY'}}",
    "test": "{{randomValue}}"
}
{{/if}}
```

In the example above, we have provided two responses that camouflage can pick from, one with `greeting: "Hello World"`, and another with `greeting: "Hello John"`. In the first line of the mock `{{#if request.query.name}}`, we are checking for the condition, if there exists a query parameter called `name`. And that's it. If your request is made with the query param `name`, camouflage will respond with first response, if not then the 2nd second response is what you get. And the value of `name` can be anything. We are using `capture` helper to help us make our response dynamic.

!!! note

    `if` and `unless` helpers are provided by handlebarjs, which don't have comparison capabilities. These helpers only check if the provided value is truthy or falsy. i.e. you can not do something like this: `{{#if something = something}}`. For comparisons, you'd need to use `is` helper. See Helpers page for example.

### Request Matching using headers

You can use the approach shown above to perform request matching with query, path params and cookies. Use `request.query.name` or `request.params.name` or `request.cookies.something` as required.

However for headers and body, we need to follow a slightly different approach. In the following example, we are using `capture` helper to capture a specific header value which then can be passed to other helpers like `is` or `if`.

```javascript
{{#if (capture from='headers' key='Authorization') }}
HTTP/1.1 200 OK
Content-Type: application/json

{
    "response": "response if auth header is present."
}
{{else}}
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
    "response": "response if no auth header present."
}
{{/if}}
```

If you want to validate a given header against a specific value, the mock file would be as shown below:

```javascript
{{#is (capture from='headers' key='Authorization') 'Basic c2h1YmhlbmR1Om1hZGh1a2Fy' }}
HTTP/1.1 200 OK
Content-Type: application/json

{
    "response": "response if auth header is present."
}
{{else}}
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
    "response": "response if no auth header present."
}
{{/is}}
```

### Request model

Request object made available by Camouflage is simply an instance of express request object for a given incoming request. Following are the properties/objects available on the request object which can be used in request matching or to extract information out of the request.

- request.baseUrl
- request.body
- request.cookies
- request.method
- request.originalUrl
- request.path
- request.protocol
- request.query
- request.body

Refer to [Express Documentation](http://expressjs.com/en/4x/api.html#req) for more information on each of these properties.

## Response Delays

You can add a Response-Delay header in raw response placed in your .mock file.

For example, if you'd like to simulate a delay of 2 seconds for `GET /hello` endpoint, contents of your `./mocks/hello/GET.mock` file would be as follows:

```javascript
HTTP/1.1 200 OK
X-Requested-By: Shubhendu Madhukar
Content-Type: application/json
Response-Delay: 2000

{
    "greeting": "Hello World",
    "phone": {{randomValue length=10 type='NUMERIC'}},
    "dateOfBirth": "{{now format='MM/DD/YYYY'}}",
    "test": "{{randomValue}}"
}
```

Additionally you can also simulate a dynamic delay using the {{num_between}} handlebar as follows

```javascript
Response-Delay: {{num_between lower=500 upper=600}}
```

## OpenAPI Conversion

If you have access to the OpenAPI specification for the APIs/Endpoints you want to mock, Camouflage supports the conversion via the `camoswag` utility.

- To use `camoswag`, you would need your OpenAPI specification file in either .json or .yaml format.
- You don't need to install `camoswag` locally on your machine, you can simply run the script using npx.
- Run the command: `npx camoswag --spec ./swagger.yaml` or `npx camoswag --spec ./swagger.json`. (Replace file location with your spec file location)
- If you would like to install camoswag locally, you can do so by running the command: `npm i -g camoswag`. For conversion use, `camoswag --spec ./swagger.yaml`
- This would create a new folder with the name `camouflage-${current_timestamp}` containing the required folder structure and mock files corresponding to each endpoint defined in your spec file.
- You can either delete or modify the dummy responses placed in the mockfiles as per your expectations. Once you are satisfied with the modifications, you can move the contents of the folder to your original `mocksDir` of your running Camouflage server.
- Note that if your spec file doesn't contain a response defined for a given endpoint, `camoswag` would put the following default response in the mock file.

```javascript
{
  "message": "More Configuration Needed"
}
```

!!! caution

    `camoswag` currently supports JSON responses only.

## OpenAPI Validation

Camouflage uses `express-openapi-validator` to enable you to validate your requests/responses against a provided OpenAPI 3 schema.

You can configure validation in two ways.

- **Basic Usage:** You can enable it via config.json. Add following options to you config:

```json
{
  // Other camouflage options
  "validation": {
    "enable": true,
    "apiSpec": "./apiSpec.yaml",
    "validateRequests": true,
    "validateResponses": true
  }
}
```

Modify the above config as per your requirements, and you are good to go.

- **Advanced Usage:** If you want more control over how to configure validation, you can set the [supported validation options](https://github.com/cdimascio/express-openapi-validator/wiki/Documentation#advanced-usage) via the camouflage method `setupValidationWithOptions`

Enable it via config

```json
{
  // Other camouflage options
  "validation": {
    "enable": true
  }
}
```

And then configure rest of the options as you wish

```javascript
camouflageHttp.setupValidationWithOptions({
  apiSpec: "./openapi.yaml",
  validateRequests: true,
  validateResponses: true,
  // Other options
});
```

## Compression

You can optionally enable the compression options via config.json. Once enabled, compression is applied on all the routes, however you can restrict this behaviour using the `setupCompressionWithOptions` method and [available options](https://www.npmjs.com/package/compression#options)

Enable compression in config.json

```json
{
  // Other camouflage options
  "compression": true
}
```

Then provide required options

```javascript
function shouldCompress(req, res) {
  if (req.headers["x-no-compression"]) {
    // don't compress responses with this request header
    return false;
  }

  // fallback to standard filter function
  return compression.filter(req, res);
}
camouflageHttp.setupCompressionWithOptions({ filter: shouldCompress });
```
