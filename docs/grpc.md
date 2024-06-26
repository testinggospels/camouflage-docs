# GRPC

GRPC module of Camouflage lets you mock your backends based on grpc protocols. You can create a Camouflage grpc object from CamouflageGrpc class and configure it to serve mocks for your incoming requests.

Start by installing required dependencies

```bash
npx jsr add @camouflage/grpc
```

- You can create the Camouflage object without any parameters, and load the required options as needed

```javascript
import CamouflageGrpc from "@camouflage/grpc";
import * as protoloader from "@grpc/proto-loader";
import * as grpc from "@grpc/grpc-js";

const camouflageGrpc = new CamouflageGrpc();
camouflageGrpc.loadConfigFromJson("./config_grpc.json");

//...add your services

camouflageGrpc.start();
```

- Or you can create the Camouflage object with the options

```javascript
import CamouflageGrpc, { CamouflageGrpcConfig } from "@camouflage/grpc";

const config = {};

const camouflageGrpc = new CamouflageGrpc(config);
camouflageGrpc.start();
```

## Available methods

###### _loadConfigFromJson = (configFilePath: string): void_

While you can include your config as part of your code and ensure the types yourself, you may at times want to maintain the configuration for your Camouflage server separate from the application code. This is usually a good practice from maintainability point of view, or even practical if you want to maintain multiple config files for different usecases.

loadConfigFromJson lets you load a config via a .json file. You don't need to worry about validating your config file, Camouflage takes care of validating your config and prints relevant errors which help you fix your config files, if you miss something.

###### _getHandlers = (): CamouflageGrpcHandler | undefined_

Camouflage provides some ready to use handlers which you can use to load your services/methods into the Camouflage grpc servers. Depending on the type of method your mocks require, you can use one of the following:

- unaryHandler
- serverSideStreamingHandler
- clientSideStreamingHandler
- bidiStreamingHandler

###### _getHelpers = (): Helpers_

When you create a CamouflageGrpc object, it automatically creates an instance of helpers. You can use getHelpers() to get a reference to this helpers object. This is useful when you want add custom helpers that are specific to your requirements.

```javascript
import Helpers from "@camouflage/helpers";

const helpers = camouflageGrpc.getHelpers();

helpers.addHelper("ping", (context) => {
  return "pong";
});

camouflageGrpc.start();
```

You can take a look at how inbuilt helpers have been created, in case you want to understand how custom helpers can be created. Refer to the [helper source code](https://github.com/testinggospels/camouflage-helpers)

###### _addService = (service: grpc.ServiceDefinition<grpc.UntypedServiceImplementation>, implementation: grpc.UntypedServiceImplementation): void_

`camouflageGrpc.addService` is wrapper on `@grpc/grpc-js` `addService` method. It allows you to load your proto package definitions/services/methods into Camouflage's grpc server. In the following example we load the `blog.proto` definition into a grpcObject and use it to provide implementation of the `createBlog` and `listBlogs` methods required by the proto definition

```javascript
import CamouflageGrpc, { CamouflageGrpcHandler } from "@camouflage/grpc";
import * as protoloader from "@grpc/proto-loader";
import * as grpc from "@grpc/grpc-js";
const camouflageGrpc = new CamouflageGrpc();
camouflageGrpc.loadConfigFromJson("./config_grpc.json");
const handlers = camouflageGrpc.getHandlers();

const blogPackageDef = protoloader.loadSync("./blog.proto", {});
const blogGrpcObject = grpc.loadPackageDefinition(blogPackageDef);
const blogPackage = blogGrpcObject.blogPackage;

if (handlers) {
  camouflageGrpc.addService(blogPackage.BlogService.service, {
    createBlog: handlers.unaryHandler,
    listBlogs: handlers.unaryHandler,
  });
}

camouflageGrpc.start();
```

Here we are using Camouflage's `unaryHandler` as the implementation of the required methods, but you might as well write your own implementation, making it easier to mock only the required methods instead of everything.

###### _start = async (): Promise<void>_

Self explanatory. Starts the Camouflage grpc server.

###### _stop = async (): Promise<void>_

Self explanatory. Stops the Camouflage grpc server.

## Hooks

!!! note

    COMING SOON

## Camouflage GRPC Configuration

You can provide following configuration options in your `config.json` file and load it to Camouflage before you start the server

```json
{
  "log": {
    "enable": true, // enables or disables the logs
    "level": "trace" // // if enable=true, sets the log level. Available options are "fatal", "error", "warn", "info", "debug", "trace"
  },
  "host": "0.0.0.0", // host part of the address on which you'd want grpc server to listen on
  "port": 8082, // port part of the address on which you'd want grpc server to listen on
  "ssl": {
    "enable": false, // enables or disabled ssl, if disabled credentials will be created using grpc.ServerCredentials.createInsecure()
    "cert": "location/to/server.cert", // if enable=true, required config for .cert file
    "key": "location/to/server/key", // if enable=true, required config for .key file
    "rootCert": "location/to/rootCert" // Optionally, provide location to root cert
  },
  "mocksDir": "./grpcMocks", // location of the mocks folder
  "monitoring": {
    "enable": true, // enables or disables monitoring
    "port": 40000 // required port for monitoring server
  }
}
```

## Folder Structure

The way you organize your directories inside the mocksDir, determine how your endpoints will be available. Assuming that you have configured your mocksDir as ./grpcMocks, the folder structure would follow the pattern as shown below.

Let's take a look at this proto file

```protobuf
syntax = "proto3";

package foo.todoPackage;

import "./todoEnum.proto";

service TodoService{
    rpc readTodo(Empty) returns (Todos);
    rpc readTodoStream(Empty) returns (stream Todo);
    rpc createTodoStream(stream Todo) returns (Todos);
    rpc createTodoBidiStream(stream Todo) returns (stream Todo);
}
```

The expected mock file path for each required methods would be:

- ./grpcMocks/foo/todoPackage/readTodo.mock
- ./grpcMocks/foo/todoPackage/readTodoStream.mock
- ./grpcMocks/foo/todoPackage/createTodoStream.mock
- ./grpcMocks/foo/todoPackage/createTodoBidiStream.mock

## Camouflage GRPC Helpers

### `capture` Helper

Usage:

- **{{capture from='metadata' key='firstName'}}** - Pretty self-explanatory, but if you want to capture some data from the request metadata, you can do so by providing the required key.
- **{{capture from='request' using="jsonpath" selector='$.title'}}** - To capture values from the actual request, your options are either `using='regex'` or `using='jsonpath'`. Selector will change accordingly.

## What data to put in .mock files

#### Unary Response

A typical unary response mock file would look like following snippet

```json
{
  "id": {{num_between lower=100 upper=500}},
  "title": "something",
  "metadata": {
    "headers": {
      "random": "{{random}}"
    },
    "trailers": {
      "now": "{{now format='epoch'}}",
      "random": "{{random}}"
    }
  },
  "delay": 2000
}
```

You only need to provide the a json object matching your expected response. In our example, let's say we are creating a mock for `createBlog` unary method which return a `Blog` as shown below

```protobuf
syntax = "proto3";

package blogPackage;

message Blog {
    int32 id = 1;
    string title = 2;
}

service BlogService{
    rpc createBlog(Blog) returns (Blog);
}
```

Corresponding to our `Blog` schema as per the proto file, we are responding with an random integer between 100-500 using `num_between` helper. And a random string as title using `random` helper. Optionally, you can also send metadata, i.e. headers or trailers or both, as shown in the example above. And finally, and optionally, you can simulate a delay of 2 seconds, by including `delay: 2000` field. And you might already know by now, you can make the delay random by using the `num_between` helper.

#### Server Side Streaming Response

Mock file for server side streaming response would be mostly similar to unary, except for one distinction. Since we are streaming a response back to client, you'll be providing multiple responses. You can do so by separating each response by a delimiter, i.e. "====" (four equals). Other than that, as you would in unary response, metadata.headers, metadata.trailers and delay are optional properties

```json
{
    "id": "{{random type='UUID'}}",
    "text": "{{random type='ALPHABETIC' length='100'}}",
    "delay": {{num_between lower=500 upper=600}}
}
====
{
    "id": "{{random type='UUID'}}",
    "text": "{{random type='ALPHABETIC' length='100'}}",
    "delay": {{num_between lower=500 upper=600}}
}
====
{
    "id": "{{random type='UUID'}}",
    "text": "{{random type='ALPHABETIC' length='100'}}",
    "delay": {{num_between lower=500 upper=600}}
}
====
{
    "id": "{{random type='UUID'}}",
    "text": "{{random type='ALPHABETIC' length='100'}}",
    "delay": {{num_between lower=500 upper=600}}
}
```

#### Client Side Streaming Response

Client streaming and unary responses are identical.

```json
{
    "todos": [
        {
            "id": "{{random type='UUID'}}",
            "text": "{{random type='ALPHABETIC' length='100'}}"
        },
        {
            "id": "{{random type='UUID'}}",
            "text": "{{random type='ALPHABETIC' length='100'}}"
        },
        {
            "id": "{{randomValue type='UUID'}}",
            "text": "{{random type='ALPHABETIC' length='100'}}"
        }
    ],
    "metadata": {
        "trailers":{
            "key": "value"
        }
    },
    "delay": {{num_between lower=500 upper=600}}
}
```

#### Bidi Streaming Response

Bidi streaming supported currently by Camouflage is like ping-pong in nature. For each request you stream to the server, you get one response back.

Bidi streaming responses differ from the other responses. Your mockfile would include a required object `data`. This is what Camouflage will respond back for each of your requests. You can include an optional `end` object, which would be sent when you end the client side stream. This is an optional object, in absence of which, Camouflage will simply end the server side stream without any response.

```json
{
  "data": {
    "id": "{{random type='UUID'}}",
    "text": "{{random type='ALPHABETIC' length='100'}}"
  },
  "end": {
    "id": "{{random type='UUID'}}",
    "text": "{{random type='ALPHABETIC' length='100'}}"
  }
}
```

## Request matching

Request matching in Camouflage grpc module can be done with a combination of helpers like `if`, `unless` and `is` along with `capture` helper.

GRPC `capture` helper has access to `request` and `metadata` objects from your requests. Usage can be found in the helper section [above](#camouflage-grpc-helpers)

!!! note

    Structure of `request` object might vary depending on the calls you are making. For example:

    - **Unary**: In this case, request object is what you would expect it to be. The object you send from the client, as is.
    - **Server Side Streaming**: Since you make one call from client, and recieve n streams in response as defined in your mock file, same request object is available for each of your responses.
    - **Client Side Streaming**: All of your request objects are stored in an array, and this array of objects (instead of an object), is made available to your response
    - **Bidi Streaming**: In this case, request object that you send from the client, is available for each of streams. However the `end` object that you create, would have access to the array of all request objects sent.
