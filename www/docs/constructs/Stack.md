---
description: "Docs for the sst.Stack construct in the @serverless-stack/resources package"
---

The `Stack` construct extends [`cdk.Stack`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html). It automatically prefixes the stack names with the stage and app name to ensure that they can be deployed to multiple regions in the same AWS account. It also ensure that the stack uses the same AWS profile and region as the app.

## Initializer

```ts
new Stack(scope: Construct, id: string, props: StackProps)
```

_Parameters_

- scope [`Construct`](https://docs.aws.amazon.com/cdk/api/latest/docs/constructs.Construct.html)
- id `string`
- props [`StackProps`](#stackprops)

## Examples

### Creating a new stack

Create a new stack by adding this in `lib/MyStack.js`.

```js
import { Stack } from "@serverless-stack/resources";

export default class MyStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    // Define your stack
  }
}
```

### Adding to an app

Add it to your app in `lib/index.js`.

```js
import MyStack from "./MyStack";

export default function main(app) {
  new MyStack(app, "my-stack");

  // Add more stacks
}
```

Here `app` is an instance of [`App`](constructs/App.md).

Note that, setting the env for an individual stack is not allowed.

```js
new MyStack(app, "my-stack", { env: { account: "1234", region: "us-east-1" } });
```

It will throw this error.

```
Error: Do not directly set the environment for a stack
```

This is by design. The stacks in SST are meant to be re-deployed for multiple stages (like Serverless Framework). And so they depend on the region and AWS profile that's passed in through the CLI. If a stack is hardcoded to be deployed to a specific account or region, it can break your deployment pipeline.

### Accessing app properties

The stage, region, and app name can be accessed through the app object. In your stacks (for example, `lib/MyStack.js`) you can use.

```js
class MyStack extends sst.Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    scope.stage;
    scope.region;
    scope.name;
  }
}
```

And in TypeScript.

```ts
class MyStack extends sst.Stack {
  constructor(scope: sst.App, id: string, props?: sst.StackProps) {
    super(scope, id, props);

    scope.stage;
    scope.region;
    scope.name;
  }
}
```

You can use this to conditionally add stacks or resources to your app.

### Prefixing resource names

You can optionally prefix resource names to make sure they don't thrash when deployed to different stages in the same AWS account.

You can do so in your stacks.

```js
scope.logicalPrefixedName("MyResource"); // Returns "dev-my-sst-app-MyResource"
```

This invokes the `logicalPrefixedName` method in [`App`](constructs/App.md) that your stack is added to. This'll return `dev-my-sst-app-MyResource`, where `dev` is the current stage and `my-sst-app` is the name of the app.

### Adding stack outputs

```js {8-11}
export default class MyStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const topic = new Topic(this, "Topic");
    const queue = new Queue(this, "Queue");

    this.addOutputs({
      TopicArn: topic.snsTopic.topicArn,
      QueueArn: topic.sqsQueue.queueArn,
    });
  }
}
```

### Adding stack exports

```js {7-9}
export default class MyStack extends Stack {
  constructor(scope, id, props) {
    super(scope, id, props);

    const topic = new Topic(this, "Topic");

    this.addOutputs({
      TopicArn: { value: topic.snsTopic.topicArn, exportName: "MyTopicArn" },
    });
  }
}
```

### Accessing AWS account info

To access the AWS account and region your app is being deployed to, use the following in your `Stack` instances.

```js
this.region;
this.account;
```

The region here is the same as the one you can find in the `scope` instance in the constructor.

## Methods

An instance of `Stack` contains the following methods.


### setDefaultFunctionProps

```ts
setDefaultFunctionProps(props: FunctionProps | ((stack: cdk.Stack) => FunctionProps))
```

_Parameters_

- **props** `FunctionProps | ((stack: cdk.Stack) => FunctionProps)`

The default function props to be applied to all the Lambda functions in the stack. These default values will be overridden if a [`Function`](Function.md) sets its own props. This cannot be called after any resources have been added to the stack.

Takes a [`FunctionProps`](Function.md#functionprops). Or a callback function takes [`cdk.Stack`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.Stack.html) that returns a [`FunctionProps`](Function.md#functionprops).

### addDefaultFunctionEnv

```ts
addDefaultFunctionEnv(env: Record<string, string>)
```

_Parameters_

- **env** `Record<string,string>`

Adds additional default environment variables to be applied to all Lambda functions in the stack. Any Lambda functions created before this call will not include the variables


### addDefaultFunctionPermissions

```ts
addDefaultFunctionPermissions(permissions: Permissions)
```

_Parameters_

- **permissions** `Permissions`

Adds additional default [`Permissions`](../util/Permissions.md) to be applied to all Lambda functions in the stack. Any Lambda functions created before this call will not include the permissions. 


### addOutputs

```ts
addOutputs(outputs: { [key: string]: string | cdk.CfnOutputProps })
```

_Parameters_

- **outputs** `{ [key: string]: string | cdk.CfnOutputProps }`

An associative array with the key being the output name as a string and the value is either a `string` as the output value or the [`cdk.CfnOutputProps`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.CfnOutputProps.html).

## StackProps

Extends [`cdk.StackProps`](https://docs.aws.amazon.com/cdk/api/latest/docs/@aws-cdk_core.StackProps.html).
