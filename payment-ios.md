# Payment Flow

To verify the receipt from the Apple Store, your server needs to send a POST request to Apple's `/verifyReceipt` endpoint with the receipt data. Here is a simple example of how you can do this in Node.js using the `request` library:

```javascript
const request = require('request');

function verifyReceipt(receipt) {
  const requestData = {
    'receipt-data': receipt,
    'password': 'your-shared-secret' // replace with your app's shared secret
  };

  request.post({
    url: 'https://buy.itunes.apple.com/verifyReceipt',
    body: requestData,
    json: true
  }, function (error, response, body) {
    if (!error && response.statusCode == 200) {
      if (body.status == 0) {
        // receipt is valid
        // update the user to paid user in your database
      } else {
        // receipt is not valid
        // handle the error
      }
    } else {
      // handle the error
    }
  });
}
```

In this code, we are sending a POST request to Apple's `/verifyReceipt` endpoint with the receipt data and the app's shared secret. If the receipt is valid, the status in the response body will be 0.

Please replace `'your-shared-secret'` with your actual shared secret.

This is a basic example and does not handle all possible edge cases. You might need to handle more cases based on your app requirements.

Also, note that for testing, you should use the sandbox URL `https://sandbox.itunes.apple.com/verifyReceipt` instead of the production URL.

## Apple Store Docs

[link](https://developer.apple.com/documentation/appstoreserverapi/get_transaction_history)