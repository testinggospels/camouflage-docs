# Getting Started

Create a new project, and install the Camoflage modules you need. Available modules are:

1. @camoflage/helpers
2. @camoflage/http
3. @camoflage/grpc
4. @camoflage/websockets - WIP
5. @camoflage/thrift - WIP
6. @camoflage/soap - WIP

## Quick Start

### @camoflage/helpers

Camoflage helpers form the backbone of each protocol. You can build your mocks without them, but helpers add advanced features to your mocks. You can chose the protocol specific package that you need to download from the following list, however helpers are required for every Camoflage project.

Install helpers

```bash
npm i @camoflage/helpers
```

When you create a helper object, it loads some inbuilt helpers that are listed [here](helpers.md). Once you have the object, you only need to parse your template string.

```javascript
import Helpers from "@camoflage/helpers";

const helpers: Helpers = new Helpers();
const todaysDate: string = helpers.parse("{{now format='yyyy-MM-dd'}}");
console.log(todaysDate); // 2023-12-21
```

Helpers class takes in two arguments i.e. `injectionAllowed` and `loglevel`.

### @camoflage/http

Camoflage HTTP Module helps you create mocks for your http/https/http2 endpoints. To start install `@camoflage/helpers` and `@camoflage/http` in your project.

```bash
npm i @camoflage/helpers  @camoflage/http
```

Once you have the required packages installed, you can start your http server as shown below.

```javascript
import CamoflageHttp from "@camoflage/http";

const CamoflageHttp: CamoflageHttp = new CamoflageHttp();
CamoflageHttp.loadConfigFromJson("./config.json");
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
 * CamoflageHttp.setServerOptionsHttps(httpsServerOptions);
 * */

CamoflageHttp.start();
```

Read more about Camoflage http module [here](http.md)

### @camoflage/gprc

Camoflage GRPC Module helps you create mocks for your grpc endpoints. To start, install `@camoflage/helpers` and `@camoflage/grpc` in your project.

```bash
npm i @camoflage/helpers @camoflage/grpc
```

Once you have the required packages installed, you can start your grpc server as shown below.

```javascript
import CamoflageGrpc, { CamoflageGrpcHandler } from "@camoflage/grpc";
import * as protoloader from "@grpc/proto-loader";
import * as grpc from "@grpc/grpc-js";

// Create CamoflageGrpc object and load config.
const camoflageGrpc: CamoflageGrpc = new CamoflageGrpc();
camoflageGrpc.loadConfigFromJson("./config_grpc.json");

// Get an instance of available camoflage grpc handlers
const handlers: CamoflageGrpcHandler | undefined = camoflageGrpc.getHandlers();

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
   * Add your service to Camoflage server.
   * */
  camoflageGrpc.addService(blogPackage.BlogService.service, {
    createBlog: handlers.unaryHandler,
    listBlogs: handlers.unaryHandler,
  });
}

camoflageGrpc.start();
```

Read more about Camoflage grpc module [here](grpc.md)

### @camoflage/websockets

WIP

### @camoflage/thrift

WIP

### @camoflage/soap

WIP
