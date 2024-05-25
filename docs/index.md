# Camouflage Service Virtualization

**Camouflage is a backend mocking tool for HTTP, gRPC, Websockets and Thrift protocols**, which helps you carry out your front end prototyping, unit testing, functional/performance testing in silos, in absence of one or more Microservices/APIs.

# What happened so far?

Camouflage was born as a small tool inspired by **namshi/mockserver**. Mockserver felt like a breath of fresh air, compared to the other tools at the moment, which required you to remember tool specific JSON schema to be able to create/manage your mocks. If not, it came with complex GUIs and some tool specific terminologies. And let's face it, no one wants to "learn" how to create mocks. It just something we have to do so that we can focus on what we actually want to do, which is building frontend prototypes and independent microservices without waiting for everything to be ready.

Camouflage took that idea from mockserver, which allows you to create mocks in seconds, no learning curve, no JSON schema, no specific terminologies. Just copy and paste your expected response in a mock file, and you're good to go. And of course, if you want to enhance the mocks, Camouflage provides you intuitive ways to do that. Technically, you could build a fully functional backend connected to a database using Camouflage.

_We wouldn't recommend doing so! Just saying you could...if you don't want to live by the rules and enjoy chaos._

# What went wrong?

The first version of [Camouflage](https://github.com/testinggospels/camouflage) was not something that was built to scale. It was something we needed quick, so we put no consideration into long term maintainability and just crammed it with everything we could think of. It was a hobby project, built to do the job in line with requirements of one specific organization. It quickly became apparent that the code was clunky, extending it was a nightmare and maintaining it...well, there is a reason why it collects dust at v0.15.0, staring longingly at v1.0.0, a destination it knows it'll never arrive at.

So here we are, making a second attempt at simplifying mocking/service virtualization.

# What has changed

### Extensible by design

Camouflage is no longer a rigid tool. It's now a flexible library you use to build your own tool. You write code to configure the mock server the way you want. You use your middlewares of choice, write your own reusable custom helpers and all of that just plays along with core Camouflage functionalities.

### Lightweight and modular

Instead of downloading one large tool which contains code for things you are never going to use, you only download things you need. Camouflage will now come with 6 modules:

1. @camouflage/helpers
2. @camouflage/http
3. @camouflage/grpc
4. @camouflage/websockets - WIP
5. @camouflage/thrift - WIP
6. @camouflage/soap - WIP

### Improved security

Reduced usage of `eval()`. Since you use it as a library now, if your routes can not be built using Camouflage, you can just add functionalities to the Camouflage app like you would if you were creating a route in a normal express app.

There are a few features that have been dropped to keep things simple. These features were not extensively used, based on the usage we noticed in our previous attempt, or were security risks.

- We have dropped helpers such as `code`, `pg` and `proxy`. You can always add them back in. Creating custom handlebar helpers is easier than ever.
- No backup and restore features. This allows you the freedom to write your own cron jobs or store the project on S3 like cloud services.
- No distributed mode. Single instance of Camouflage should be fairly scalable and support large loads. You can use tools like pm2 to add more instances if needed.

# Why is it different this time?

The primary reason this project went cold was the difficulties maintaining it. It was very hard to extend or modify certain functionalities without hell breaking loose. So why should you trust that this revival attempt will not meet the same fate?

- With the recent announcement of [jsr.io](https://jsr.io/), we have removed the build step completely. JSR is still new, so we'll see how it progresses. While it works as expected for node projects, we are yet to see if we can support bun and deno as well.
- The significant change, that makes us hopeful that this time it will work, is the way you can extend Camouflage. The previous version of Camouflage had very limited ways to extend it, which could be done via handlebars and code injection, which wasn't intuitive at all. You can still add new handlebar helpers, however you can now use something called hooks, which allow you to intercept the mock response process and make necessary customizations.

Let's get started
