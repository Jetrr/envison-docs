# PubSub Implementation

## Publishing a message

Publishing data to a topic in Google Cloud Pub/Sub using TypeScript involves using the `@google-cloud/pubsub` library. Here's a simple example:

First, install the PubSub library:

```bash
npm install --save @google-cloud/pubsub
```

Then, you can use the following code to publish a message:

```typescript
import {PubSub} from '@google-cloud/pubsub';

async function publishMessage() {
  // Creates a client; cache this for further use
  const pubSubClient = new PubSub();

  const dataBuffer = Buffer.from('Your message here');

  try {
    const messageId = await pubSubClient.topic('your-topic-name').publish(dataBuffer);
    console.log(`Message ${messageId} published.`);
  } catch (error) {
    console.error(`Received error while publishing: ${error.message}`);
    process.exitCode = 1;
  }
}

publishMessage();
```

In this example, replace `'your-topic-name'` with the name of your topic, and `'Your message here'` with the message you want to publish. 

This code creates a new PubSub client, then uses it to publish a message to a topic. The message is sent as a Buffer, which is a binary representation of your data. 

The `publish` method returns a Promise that resolves with the ID of the message that was published. If there was an error publishing the message, the Promise is rejected and the error is logged to the console.

Remember to authenticate your application with Google Cloud, you can do this by setting the `GOOGLE_APPLICATION_CREDENTIALS` environment variable to the path of your service account key file.

## HTTP Triggers

Yes, you can set up an HTTP trigger to publish messages to a Pub/Sub topic. Here's an example of how you can do it using Express.js:

First, install the necessary libraries:

```bash
npm install --save express @google-cloud/pubsub body-parser
```

Then, you can use the following code to create an HTTP server that listens for POST requests and publishes the request body to a Pub/Sub topic:

```typescript
import express from 'express';
import bodyParser from 'body-parser';
import {PubSub} from '@google-cloud/pubsub';

const app = express();
const pubSubClient = new PubSub();

app.use(bodyParser.json());

app.post('/publish', async (req, res) => {
  const data = JSON.stringify(req.body);
  const dataBuffer = Buffer.from(data);

  try {
    const messageId = await pubSubClient.topic('your-topic-name').publish(dataBuffer);
    console.log(`Message ${messageId} published.`);
    res.status(200).send(`Message ${messageId} published.`);
  } catch (error) {
    console.error(`Received error while publishing: ${error.message}`);
    res.status(500).send(`Error publishing message: ${error.message}`);
  }
});

const port = process.env.PORT || 8080;
app.listen(port, () => {
  console.log(`Server started on port ${port}`);
});
```

In this example, replace `'your-topic-name'` with the name of your topic. This code creates an Express.js server that listens for POST requests on the `/publish` endpoint. When a request is received, it publishes the request body to a Pub/Sub topic and sends a response back to the client with the message ID.

The request body is converted to a string using `JSON.stringify` and then to a Buffer using `Buffer.from`. This is because the `publish` method expects the message to be a Buffer.

If there was an error publishing the message, a 500 response is sent back to the client with the error message. If the message was published successfully, a 200 response is sent back with the message ID.

## Subscription methods (Push and Pull)

Google Cloud Pub/Sub doesn't directly make HTTP API calls when a message is published to a topic. Instead, it uses a push or pull mechanism to deliver messages to subscribers.

1. **Push Subscriptions**: In a push subscription, Pub/Sub sends each message as an HTTP request to a pre-configured endpoint. This can be any publicly accessible HTTPS endpoint, such as a Google Cloud Function, Google App Engine, or your own service. The endpoint must have a valid SSL certificate.

Here is an example of how you can handle a Pub/Sub message in an Express.js application:

```typescript
app.post('/pubsub/push', (req, res) => {
  const message = Buffer.from(req.body.message.data, 'base64').toString();
  console.log(`Received message: ${message}`);
  res.status(200).send();
});
```

2. **Pull Subscriptions**: In a pull subscription, the subscriber application needs to call the Pub/Sub service to retrieve messages. This is more suitable for batch-oriented processing. Here's how you can do it:

```typescript
async function pullMessages() {
  const subscriptionName = 'your-subscription-name';
  const timeout = 60;

  const subscription = pubSubClient.subscription(subscriptionName);
  let messageCount = 0;

  const messageHandler = (message: any) => {
    console.log(`Received message ${message.id}:`);
    console.log(`\tData: ${message.data}`);
    console.log(`\tAttributes: ${message.attributes}`);
    messageCount += 1;

    message.ack();
  };

  subscription.on('message', messageHandler);

  setTimeout(() => {
    subscription.removeListener('message', messageHandler);
    console.log(`${messageCount} message(s) received.`);
  }, timeout * 1000);
}

pullMessages();
```

In this code, replace `'your-subscription-name'` with the name of your subscription. This code creates a message handler that logs the message data and attributes, and then acknowledges the message. The message handler is added as a listener to the subscription's 'message' event. After a timeout, the message handler is removed.

Remember to create a subscription for your topic in the Google Cloud Console or using the Pub/Sub client library. For push subscriptions, you'll need to provide the URL of your HTTP endpoint.