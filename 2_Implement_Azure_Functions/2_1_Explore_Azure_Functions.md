# Discover Azure Functions

Azure Functions is a serverless solution that allows you to write less code, maintain less infrastructure, and save on costs. Instead of worrying about deploying and maintaining servers, the cloud infrastructure provides all the up-to-date resources needed to keep your applications running.

We often build systems to react to a series of critical events. Whether you're building a web API, responding to database changes, processing IoT data streams, or even managing message queues - every application needs a way to run some code as these events occur.

Azure Functions supports triggers, which are ways to start execution of your code, and bindings, which are ways to simplify coding for input and output data. There are other integration and automation services in Azure and they all can solve integration problems and automate business processes. They can all define input, actions, conditions, and output.

## Compare Azure Functions and Azure Logic Apps

`Azure Functions` vs `Azure Logic Apps` vs `WebJobs`

Both Functions and Logic Apps are Azure Services that enable serverless workloads.
Azure Functions is a serverless compute service, whereas Azure Logic Apps is a serverless workflow integration platform. Both can create complex orchestrations. An orchestration is a collection of functions or steps, called actions in Logic Apps, that are executed to accomplish a complex task.

For Azure Functions, you develop orchestrations by writing code and using the Durable Functions extension. For Logic Apps, you create orchestrations by using a GUI or editing configuration files.

Azure Functions is built on the WebJobs SDK, so it shares many of the same event triggers and connections to other Azure services

Azure Functions offers more developer productivity than Azure App Service WebJobs does. It also offers more options for programming languages, development environments, Azure service integration, and pricing. For most scenarios, it's the best choice.

## Compare Azure Functions hosting options

- Consumption plan
- Flex Consumption plan
- Premium plan
- Dedicated plan
- Container Apps

Azure App Service infrastructure facilitates Azure Functions hosting on both Linux and Windows virtual machines.

### Consumption plan

The Consumption plan is the default hosting plan. Pay for compute resources only when your functions are running (pay-as-you-go) with automatic scale. On the Consumption plan, instances of the Functions host are dynamically added and removed based on the number of incoming events.

### Flex Consumption plan

Get high scalability with compute choices, virtual networking, and pay-as-you-go billing. On the Flex Consumption plan, instances of the Functions host are dynamically added and removed based on the configured per instance concurrency and the number of incoming events.

You can reduce cold starts by specifying the number of pre-provisioned (always ready) instances. Scales automatically based on demand.

### Premium plan

Automatically scales based on demand using prewarmed workers, which run applications with no delay after being idle, runs on more powerful instances, and connects to virtual networks.

Consider the Azure Functions Premium plan in the following situations:

- Your function apps run continuously, or nearly continuously.
- You want more control of your instances and want to deploy multiple function apps on the same plan with event-driven scaling.
- You have a high number of small executions and a high execution bill, but low GB seconds in the Consumption plan.
- You need more CPU or memory options than are provided by consumption plans.
- Your code needs to run longer than the maximum execution time allowed on the Consumption plan.
- You require virtual network connectivity.
- You want to provide a custom Linux image in which to run your functions.

### Dedicated plan

Run your functions within an App Service plan at regular App Service plan rates. Best for long-running scenarios where Durable Functions can't be used.

Consider an App Service plan in the following situations:

- You must have fully predictable billing, or you need to manually scale instances.
- You want to run multiple web apps and function apps on the same plan.
- You need access to larger compute size choices.
- Full compute isolation and secure network access provided by an App Service Environment (ASE).
- High memory usage and high scale (ASE).

### Container Apps

Create and deploy containerized function apps in a fully managed environment hosted by Azure Container Apps.

Use the Azure Functions programming model to build event-driven, serverless, cloud native function apps. Run your functions alongside other microservices, APIs, websites, and workflows as container-hosted programs.

## Function app time-out duration

The `functionTimeout` property in the `host.json` project file specifies the time-out duration for functions in a function app. This property applies specifically to function executions. After the trigger starts function execution, the function needs to return/respond within the time-out duration.

## Scale Azure Functions

1. Consumption plan: Event driven. Scales out automatically, even during periods of high load. Functions infrastructure scales CPU and memory resources by adding more instances based on the number of incoming trigger events.
2. Flex Consumption plan: Per-function scaling. Event-driven scaling decisions are calculated on a per-function basis, which provides a more deterministic way of scaling the functions in your app.
3. Premium plan: Event driven. Scale out automatically based on the number of events that its functions are triggered on.
4. Dedicated plan: Manual/autoscale
5. Container Apps: Event driven. Scale out automatically by adding more instances of the Functions host, based on the number of events that its functions are triggered on.
