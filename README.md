# mx-sdk-js-wallet-connect-provider

Signing provider for dApps: Wallet Connect.

Documentation is available on [docs.multiversx.com](https://docs.multiversx.com/sdk-and-tools/erdjs/erdjs-signing-providers/), while an integration example can be found [here](https://github.com/multiversx/mx-sdk-js-examples/tree/main/signing-providers).

Note that **we recommend using [dapp-core](https://github.com/multiversx/mx-sdk-dapp)** instead of integrating the signing provider on your own.

## Distribution

[npm](https://www.npmjs.com/package/@multiversx/sdk-wallet-connect-provider)

## Installation

`sdk-wallet-connect-provider` is delivered via [npm](https://www.npmjs.com/package/@multiversx/sdk-wallet-connect-provider), therefore it can be installed as follows:

```bash
npm install @multiversx/sdk-wallet-connect-provider
```

### Building the library

In order to compile the library, run the following:

```bash
npm install
npm run compile
```

## Project ID

The WalletConnect 2.0 Signing Provider can use the [WalletConnect Cloud Relay](https://docs.walletconnect.com/2.0/cloud/relay) default address: `wss://relay.walletconnect.com`, in order to be able to access the Cloud Relay you will need to generate a Project ID

The Project ID can be generated for free here: https://cloud.walletconnect.com/sign-in

The WalletConnect Project ID grants you access to the WalletConnect Cloud Relay that securely manages communication between the device and the dApp.

## Usage Examples

For this example we will use the WalletConnect 2.0 provider since 1.0 is no longer mantained and it will be [deprecated soon](https://medium.com/walletconnect/weve-reset-the-clock-on-the-walletconnect-v1-0-shutdown-now-scheduled-for-june-28-2023-ead2d953b595)

First, let's see a (simple) way to build a QR dialog using [`qrcode`](https://www.npmjs.com/package/qrcode) (and bootstrap):

```js
import QRCode from "qrcode";

async function openModal(connectorUri) {
    const svg = await QRCode.toString(connectorUri, { type: "svg" });

    // The referenced elements must be added to your page, in advance
    $("#MyWalletConnectQRContainer").html(svg);
    $("#MyWalletConnectModal").modal("show");
}

function closeModal() {
    $("#MyWalletConnectModal").modal("hide");
}
```

In order to create an instance of the provider, do as follows:

```js
import { WalletConnectV2Provider } from "@multiversx/sdk-wallet-connect-provider";

// Generate your own WalletConnect 2 ProjectId here: 
// https://cloud.walletconnect.com/app
const projectId = "9b1a9564f91cb6...";
// The default WalletConnect V2 Cloud Relay
const relayUrl = "wss://relay.walletconnect.com";
// T for Testnet, D for Devnet and 1 for Mainnet
const chainId = "T" 

const callbacks = {
    onClientLogin: async function () {
        // closeModal() is defined above
        closeModal();
        const address = await provider.getAddress();
        console.log("Address:", address);
    },
    onClientLogout: async function () {
        console.log("onClientLogout()");
    },
    onClientEvent: async function (event) {
        console.log("onClientEvent()", event);
    }
};

const provider = new WalletConnectProvider(callbacks, chainId, relayUrl, projectId);
```

> You can customize the Core WalletConnect functionality by passing `WalletConnectProvider` an optional 5th parameter: `options`
For example `metadata` and `storage` for [React Native](https://docs.walletconnect.com/2.0/javascript/guides/react-native) or `{ logger: 'debug' }` for a detailed under the hood logging

Before performing any operation, make sure to initialize the provider:

```js
await provider.init();
```

### Login and logout

Then, ask the user to login using xPortal on her phone:

```js
const { uri, approval } = await provider.connect();
// connect will provide the uri required for the qr code display 
// and an approval Promise that will return the connection session 
// once the user confirms the login

// openModal() is defined above
openModal(uri);        

// pass the approval Promise
await provider.login({ approval });
```

The `login()` method supports the `token` parameter (similar to other providers):

```js
// A custom identity token (opaque to the signing provider)
const authToken = "aaaabbbbaaaabbbb";

await provider.login({ approval, token: authToken });

console.log("Address:", provider.address);
console.log("Token signature:", provider.signature);
```

> The pairing proposal between a wallet and a dapp is made using an [URI](https://docs.walletconnect.com/2.0/specs/clients/core/pairing/pairing-uri). In WalletConnect v2.0 the session and pairing are decoupled from each other. This means that a URI is shared to construct a pairing proposal, and only after settling the pairing the dapp can propose a session using that pairing. In simpler words, the dapp generates an URI that can be used by the wallet for pairing.

Once the user confirms the login, the `onClientLogin()` callback (declared above) is executed.

In order to log out, do as follows:

```js
await provider.logout();
```

### Signing transactions

Transactions can be signed as follows:

```js
import { Transaction } from "@multiversx/sdk-core";

const firstTransaction = new Transaction({ ... });
const secondTransaction = new Transaction({ ... });

await provider.signTransactions([firstTransaction, secondTransaction]);

// "firstTransaction" and "secondTransaction" can now be broadcasted.
```

Alternatively, one can sign a single transaction using the method `signTransaction()`.

### Signing messages

Arbitrary messages can be signed as follows:

```js
import { SignableMessage } from "@multiversx/sdk-core";

const message = new SignableMessage({
    message: Buffer.from("hello")
});

await provider.signMessage(message);

console.log(message.toJSON());
```