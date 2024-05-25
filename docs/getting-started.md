# Getting Started

Create a new project, and install the Camouflage modules you need. Available modules are:

1. @camouflage/helpers
2. @camouflage/http
3. @camouflage/grpc
4. @camouflage/websockets - WIP
5. @camouflage/thrift - WIP
6. @camouflage/soap - WIP

## Quick Start

1. Create a new project: `npm init --y`
2. Set `"type": "module"` in your `package.json`
3. Install the module relevant to your API protocol. For http/https/http2 protocols: `npx jsr add @camouflage/http`. For grpc protocol: `npx jsr add @camouflage/grpc`
4. You are now ready to build your mocks. Learn more about each module below.

### @camouflage/helpers

Camouflage helpers form the backbone of each protocol. You can build your mocks without them, but helpers add advanced features to your mocks.

Install helpers

```bash
npx jsr add @camouflage/helpers
```

When you create a helper object, it loads some inbuilt helpers that are listed [here](helpers.md). Once you have the object, you only need to parse your template string.

```javascript
import Helpers from "@camouflage/helpers";

const helpers = new Helpers();
const todaysDate = helpers.parse("{{now format='yyyy-MM-dd'}}");
console.log(todaysDate); // 2023-12-21
```

Helpers class takes in two arguments i.e. `injectionAllowed` and `loglevel`.

!!! note

    Note that `@camouflage/helpers` is not intended for standalone usage. It is bundled with other camouflage modules such as http and grpc. You could use it in non-camouflage projects, however such use cases will not be supported.

### @camouflage/http

Camouflage HTTP Module helps you create mocks for your http/https/http2 endpoints. To start install `@camouflage/http` in your project.

```bash
npx jsr add @camouflage/http
```

Once you have the required packages installed, you can start your http server as shown below.

```javascript
import CamouflageHttp from "@camouflage/http";

const camouflageHttp = new CamouflageHttp();
camouflageHttp.loadConfigFromJson("./config.json");
/**
 * You can follow the instructions in the link below
 * to generate self signed certificates if you don't already have them.
 *      https://www.akadia.com/services/ssh_test_certificate.html
 * FOR HTTPS/HTTP2 servers, you would need to setup credentials
 * const httpsServerOptions = {
 *    key: fs.readFileSync("location/to/server.key"),
 *    cert: fs.readFileSync("location/to/server.crt"),
 *    // more options
 * };
 * camouflageHttp.setServerOptionsHttps(httpsServerOptions);
 * */

camouflageHttp.start();
```

Read more about Camouflage http module [here](http.md)

### @camouflage/gprc

Camouflage GRPC Module helps you create mocks for your grpc endpoints. To start, install `@camouflage/grpc` in your project.

```bash
npx jsr add @camouflage/grpc
```

Once you have the required packages installed, you can start your grpc server as shown below.

```javascript
import CamouflageGrpc from "@camouflage/grpc";
import * as protoloader from "@grpc/proto-loader";
import * as grpc from "@grpc/grpc-js";

// Create CamouflageGrpc object and load config.
const camouflageGrpc = new CamouflageGrpc();
camouflageGrpc.loadConfigFromJson("./config_grpc.json");

// Get an instance of available camouflage grpc handlers
const handlers = camouflageGrpc.getHandlers();

// Load your proto file
const blogPackageDef = protoloader.loadSync("./blog.proto", {});
const blogGrpcObject = grpc.loadPackageDefinition(blogPackageDef);
const blogPackage = blogGrpcObject.blogPackage;

if (handlers) {
  /**
   * Depending on the type of your method, use one of available handlers
   * i.e.
   * - unaryHandler
   * - serverSideStreamingHandler
   * - clientSideStreamingHandler
   * - bidiStreamingHandler
   *
   * Add your service to Camouflage server.
   * */
  camouflageGrpc.addService(blogPackage.BlogService.service, {
    createBlog: handlers.unaryHandler,
    listBlogs: handlers.unaryHandler,
  });
}

camouflageGrpc.start();
```

Read more about Camouflage grpc module [here](grpc.md)

### @camouflage/websockets

WIP

### @camouflage/thrift

WIP

### @camouflage/soap

WIP
