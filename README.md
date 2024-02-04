# unstorage-driver-aws-dynamodb

AWS DynamoDB driver for [unstorage](https://unstorage.unjs.io).

This driver uses the DynamoDB table as a key value store. By default it uses the `key` as the table partition key and the `value` as content. This options can be changed using `attributes`.

## Installation

```bash
# Using pnpm
pnpm add unstorage-driver-aws-dynamodb

# Using yarn
yarn add unstorage-driver-aws-dynamodb

# Using npm
npm install unstorage-driver-aws-dynamodb
```

## Usage

Dependend on the target enviroment it will need `@aws-sdk/client-dynamodb` and `@aws-sdk/lib-dynamodb` in your project

```bash
npm i -D @aws-sdk/client-dynamodb @aws-sdk/lib-dynamodb
```

```js
import { createStorage } from "unstorage";
import dynamoDBDriver from "unstorage/drivers/aws-dynamodb";

const storage = createStorage({
  driver: dynamoDBDriver({
    table: "my-persistent-storage", // required
    region: "us-east-1", // optional, retrieved via environment variables
    credentials: {
      // optional, retrieved by AWS SDK via environment variables
      accessKeyId: "xxxxxxxxxx", // DO NOT HARD-CODE SECRETS
      secretAccessKey: "xxxxxxxxxxxxxxxxxxxx", // DO NOT HARD-CODE SECRETS
    },
  }),
});
```

Persistent configuration usage:

```js
import { createStorage } from "unstorage";
import dynamoDBDriver from "unstorage/drivers/aws-dynamodb";

const storage = createStorage({
  driver: dynamoDBDriver({
    table: "my-table-name", // required
    attributes: {
      key: "key", // optional, configure attributes name
      value: "value", // optional, configure attributes name
    },
  }),
});
```

Temporary configurations:

```js
import { createStorage } from "unstorage";
import dynamoDbCacheDriver from "unstorage/drivers/aws-dynamodb";

const storage = createStorage({
  driver: dynamoDbCacheDriver({
    table: "my-table-name", // required
    attributes: {
      key: "key", // optional but recommended
      value: "value", // optional but recommended
      ttl: "ttl", // optional, configure attributes name
    },
    ttl: 300, // optional, values in seconds or 0 to disable
  }),
});
```

When `ttl` is set to a number greater than 0 the driver will add seconds to the current timestamp and set the TTL attribute.
Otherwise removing the `ttl` option or setting it to 0 will disable this functionality.

The `setItem` method support an additional options which allows you to override the general `ttl` option:

```js
await storage.setItem("key", "value", { ttl: 900 });
```

Since the DynamoDB items deletion is asynchronous the driver will check the validity of the TTL attribute before returning them from `getItem` and `getKeys` operations. This in order to ensure that no expired items will be returned.

**Authentication:**

The driver supports the default [AWS SDK credentials](https://docs.aws.amazon.com/sdk-for-javascript/v3/developer-guide/setting-credentials-node.html).

The IAM role or IAM user that use the driver need the following permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:Scan",
        "dynamodb:DeleteItem"
      ],
      "Resource": "arn:aws:dynamodb:*:*:table/my-table-name"
    }
  ]
}
```

**Options:**

- `table`: The name of the DynamoDB table.
- `region`: The AWS region to use.
- `credentials`: The AWS SDK credentials object.
- `attributes`: The key, value and TTL attributes mapping to table item attributes.
- `ttl`: The number of seconds to add to the current timestamp to set the TTL attribute. Set to 0 to disable it.

### Contribution

Contributions are what make the open source community such an amazing place to be learn, inspire, and create. Any contributions you make are greatly appreciated.

1. Fork the Project
1. Create your Feature Branch (git checkout -b feature/AmazingFeature)
1. Commit your Changes (git commit -m 'Add some AmazingFeature')
1. Push to the Branch (git push origin feature/AmazingFeature)
1. Open a Pull Request

### Original Source

The original source code can be found at this [GitHub Pull Request](https://github.com/unjs/unstorage/pull/234) from [Fabio Gollinucci](https://github.com/daaru00).

### License

Distributed under the "bsd-2-clause" License. See [LICENSE.txt](LICENSE.txt) for more information.
