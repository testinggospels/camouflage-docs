# Getting Started

Create a new project, and install the camouflage modules you need. Available modules are:

1. @camouflage/helpers
2. @camouflage/http
3. @camouflage/grpc
4. @camouflage/websockets - WIP
5. @camouflage/thrift - WIP
6. @camouflage/soap - WIP

## Quick Start

### @camouflage/helpers

Camouflage helpers form the backbone of each protocol. You can build your mocks without them, but helpers add advanced features to your mocks. You can chose the protocol specific package that you need to download from the following list, however helpers are required for every camouflage project.

Install helpers

```bash
npm i @camouflage/helpers
```

When you create a helper object, it loads some inbuilt helpers that are listed [here](helpers.md). Once you have the object, you only need to parse your template string.

```javascript
import Helpers from "@camouflage/helpers";

const helpers: Helpers = new Helpers();
const todaysDate: string = helpers.parse("{{now format='yyyy-MM-dd'}}");
console.log(todaysDate); // 2023-12-21
```

Helpers class takes in two arguments i.e. `injectionAllowed` and `loglevel`.

### @camouflage/http

Camouflage HTTP Module helps you create mocks for your http/https/http2 endpoints. To start install `@camouflage/helpers` and `@camouflage/http` in your project.

```bash
npm i @camouflage/helpers  @camouflage/http
```

Once you have the required packages installed, you can start your http server as shown below.

```javascript
import CamouflageHttp from "@camouflage/http";

const camouflageHttp: CamouflageHttp = new CamouflageHttp();
camouflageHttp.loadConfigFromJson("./config.json");
/**
 * You can follow the instructions in the link below
 * to generate self signed certificates if you don't already have them.
 *      https://www.akadia.com/services/ssh_test_certificate.html
 * FOR HTTPS/HTTP2 servers, you would need to setup credentials
 * const httpsServerOptions: https.ServerOptions = {
 *    key: fs.readFileSync("location/to/server.key"),
 *    cert: fs.readFileSync("location/to/server.crt"),
 *    // more options
 * };
 * camouflageHttp.setServerOptionsHttps(httpsServerOptions);
 * */

camouflageHttp.start();
```

Read more about camouflage http module [here](http.md)

### @camouflage/gprc

Camouflage GRPC Module helps you create mocks for your grpc endpoints. To start, install `@camouflage/helpers` and `@camouflage/grpc` in your project.

```bash
npm i @camouflage/helpers @camouflage/grpc
```

Once you have the required packages installed, you can start your grpc server as shown below.

```javascript
import CamouflageGrpc, { CamouflageGrpcHandler } from "@camouflage/grpc";
import * as protoloader from "@grpc/proto-loader";
import * as grpc from "@grpc/grpc-js";

// Create camouflageGrpc object and load config.
const camouflageGrpc: CamouflageGrpc = new CamouflageGrpc();
camouflageGrpc.loadConfigFromJson("./config_grpc.json");

// Get an instance of available camouflage grpc handlers
const handlers: CamouflageGrpcHandler | undefined = camouflageGrpc.getHandlers();

// Load your proto file
const blogPackageDef: protoloader.PackageDefinition = protoloader.loadSync("./blog.proto", {});
const blogGrpcObject: grpc.GrpcObject = grpc.loadPackageDefinition(blogPackageDef);
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
   * Add your service to camouflage server.
   * */
  camouflageGrpc.addService(blogPackage.BlogService.service, {
    createBlog: handlers.unaryHandler,
    listBlogs: handlers.unaryHandler,
  });
}

camouflageGrpc.start();
```

Read more about camouflage grpc module [here](grpc.md)

### @camouflage/websockets

WIP

### @camouflage/thrift

WIP

### @camouflage/soap

WIP
