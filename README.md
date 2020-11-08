This is an example [Next.js](https://nextjs.org/) project consuming Shopify App Bridge Session Tokens for authentication.

## Getting Started

First, set your Shopify App's public & private keys in `.local.env`:

```
SHOPIFY_API_PUBLIC_KEY='your public api key from the Shopify app dashboard here'
SHOPIFY_API_PRIVATE_KEY='your private api key from the Shopify app dashboard here'
NEXT_PUBLIC_SHOPIFY_API_PUBLIC_KEY='same value as SHOPIFY_API_PUBLIC_KEY, this will expose your public key to the frontend'
SHOPIFY_AUTH_CALLBACK_URL='<your-sub-domain>.ngrok.io/api/auth/callback'
HOME_PATH = '/dashboard' # or wherever you'd like the user to be sent to after successfully authenticating
```

Then make sure your app is configured to use `<your-sub-domain>.ngrok.io` as the entry point.

Second, start up [ngrok](https://ngrok.io) and configure it to use `localhost:3000`.

Then, run the development server:

```bash
npm run dev
# or
yarn dev
```

## How it works

### OAuth Handshake

First, the OAuth flow begins at `/api/auth.js`. It will redirect to Shopify's authorization page. No need to do anything else besides export the build in middleware:

```javascript
// pages/api/auth.js

import handleAuthStart from '../../middleware/handleAuthStart';

export default handleAuthStart;
```

Once the user accepts your app's scopes and terms, they will be redirected to `/api/auth/callback.js`.

That route will then verify the signature of the request and retrieve the merchant's Shopify access token.

The `afterAuth` function is called after the access token is successfully retrieved. Use this function to store the shop's access token:

```javascript
// pages/api/auth/callback.js
import handleAuthCallback from '../../../middleware/handleAuthCallback';

const afterAuth = async(req, res, accessToken) => {
  // save accessToken with the shop
  db.collection('shop').insertOne({name: req.query.shop, accessToken});

  // redirect is handled by handleAuthCallback, no need to res.send() or res.redirect() here.
};

export default handleAuthCallback(afterAuth);
```

Now that the merchant's OAuth handshake is complete, the customer is finally redirected to `/pages/dashboard.js`, or whichever path you provide in `process.env.HOME_PATH`.

### App Bridge Session Token Retrieval

After the handshake is complete, in the `_app.js` the App Bridge is instantiated and the session token is retrieved. The shop name (aka `shopOrigin`) is transfered via a query parameter (`?shop={shop})`.

The `pages/dashboard.js` is not rendered until the session token is available for consumption.

Once the page loads, then an HTTP request with the session token is sent to `/api/verify-token.js` where it's decoded and validated with your app's private key.

## TODO

* Track the `nonce` in session and verify it in the `/api/auth/callback.js` route.
* Implement a client side redirecting scheme to detect unauthorized use and return the user back to the oauth flow.
  * For example: https://github.com/Shopify/koa-shopify-auth/blob/master/src/auth/redirection-page.ts

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js/) - your feedback and contributions are welcome!

