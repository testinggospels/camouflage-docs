# Camoflage Service Virtualization

**Camoflage is a backend mocking tool for HTTP, gRPC, Websockets and Thrift protocols**, which helps you carry out your front end prototyping, unit testing, functional/performance testing in silos, in absence of one or more Microservices/APIs.

## History

Camoflage was born as a small tool inspired by **namshi/mockserver**. Mockserver felt like a breath of fresh air, compared to the other tools at the moment which required you to remember tool specific JSON schema to be able to create/manage your mocks. If not, it came with complex GUIs and some tool specific terminologies. And let's face it, no one wants to "learn" how to create mocks. It just something we have to do so that we can focus on what we actually want to do, which is building frontend prototypes and independent microservices without waiting for everything to be ready.

Camoflage took that idea from mockserver, which allows you to create mocks in seconds, no learning curve, no JSON schema, no specific terminologies. Just copy and paste your expected response in a mock file and you're good to go. And of course, if you want to enhance the mocks, Camoflage provides you intuitive ways to do that. Technically you could build a fully functional backend connected to a database using Camoflage.

_We wouldn't recommend doing so! Just saying you could...if you don't want to live by the rules and enjoy chaos._

First version of [Camoflage](https://github.com/testinggospels/camouflage) was something that wasn't built to scale. It was a hobby project, built to do the job in line with requirements of one specific organization. It quickly became apparant that the code was clunky, extending it was a nightmare and maintaining it...well, there is a reason why it collects dust at v0.15.0, staring longingly at v1.0.0, a desitination it knows it'll never arrive at.

So here we are, making a second attempt at simplifying mocking/service virtualization. Let's dive in.

## What has changed

### Extensible by design

Camoflage is no longer a rigid tool. It's now a flexible library you use to build your own tool. You write code to configure the mock server the way you want. You use your middlewares of choice, write your own reusable custom helpers and all of that just plays along with core Camoflage functionalities.

### Lightweight and modular

Instead of downloading one large tool which contains code for things you are never going to use, you only download things you need. Camoflage now comes with 6 modules:

1. @camoflage/helpers
2. @camoflage/http
3. @camoflage/grpc
4. @camoflage/websockets - WIP
5. @camoflage/thrift - WIP
6. @camoflage/soap - WIP

### Improved security

Reduced usage of `eval()`. Since you use it as a library now, if your routes can not be built using Camoflage, you can just add functionalities to the Camoflage app like you would if you were creating a route in a normal express app.

!!! caution

    There are a few features that have been dropped to keep things simple. These features were not extensively used or were security risks based on the feedback we got in our previous attempt.

    - We have dropped helpers such as `code`, `pg` and `proxy`. You can always add them back in. Creating custom handlebar helpers is easier than ever.
    - No backup and restore features. This allows you the freedom to write your own cron jobs or store the project on S3 like cloud services.
    - No distributed mode. Single instance of Camoflage should be fairly scalable and support large loads.

Let's get started.
