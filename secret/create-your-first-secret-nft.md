# Introduction

[Non-Fungible Tokens](https://en.wikipedia.org/wiki/Non-fungible_token) implement the idea of uniqueness in the blockchain world. With classical, fungible tokens the only important characteristic is how _many_ of them you own. You can think of those as currency, or to make a more exotic example, carrots. It is interesting to know how many kilograms of carrots you have, but nobody is likely to be really into you telling them which carrots you own precisely. For non-fungible tokens on the other hand, the number you are holding is not as important as _which_ token you hold. Given an example: If I tell you that I own three paintings, it's not that impressive. If I tell you on the other hand that I own the Mona Lisa, Guernica and The Persistence of Time you might be impressed and still want me to prove it.

At a high level, an NFT has a number of important properties. First we are interested in which account holds a token, as this can decide who is allowed to interact with it. Secondly we need a way to store the metadata, which is the information that users will see and interact with that makes this NFT more than just a tokenID!

{% embed url="https://youtu.be/jRuSOos9ig4" %}

# Prerequisites

This tutorial assumes that you have completed the [Secret Pathway](https://learn.figment.io/protocols/secret) already, as we will be building upon that foundation of knowledge and skill. If you have not already done so, you would be wise to take the time to complete the Pathway. We will start with the same project folder as in section 5 of the Pathway.
# Requirements

* The latest version of NodeJS installed \(use of nvm, the node version manager, is _encouraged_ for Web 3 developers\)
* A code editor like VSCode, Theia, Atom, _etc_.
* Required JavaScript packages –
  * secretjs - for the Secret Network JavaScript API
  * dotenv - for working with environment variables

It is not required to install docker and the Rust toolchain _yet_, as we are using a pre-deployed contract in this case. A future installment in this series will guide you through writing and compiling your own variant of a snip721 token.

# What is different between ERC721 and snip721

As mentioned above each NFT contains unique data or at least a way to identify the unique data. Due to the distributed public ledger property of most blockchains, this means that anyone can read this data. So even if you bought an expensive painting as an NFT no one is stopping me from finding the NFT on a blockchain explorer, having a look at the data and downloading that image for myself.

Secret Network is focused on privacy by default and this paradigm extends to its implementation of NFT standards. The snip721 standard mimics the functionality of the corresponding ERC721 standard on the Ethereum blockchain but provides more granular control over what information is private, rather than everything being publicly available. When a new snip721 token contract is deployed to the secret network you can define if the total supply and the owners of tokens should be publicly available. The deployer of the contract decides which data is public and which is only available to the current owner of the NFT.

# Getting started with secretNFTs

We will connect to an already deployed instance of the [snip721 reference implementation](https://github.com/baedrik/snip721-reference-impl): The _SecretFigment token_. We will build the functionality needed to interact with an NFT on secret. Namely the ability to mint new tokens, to set the Viewing key on tokens and to query the token private metadata.
# Minting

As we will be frequently communicating with the contract, the first thing we will do is add the contract address to our `.env` file:

```text
SECRET_NFT_CONTRACT='secret166tjlgmahhjrl8ndegq8xmzjxfe6p6h4hdvx6a'
```

Next, we create a new file in the project folder called `mint.js` and add the following code into it:

```javascript
const {
  EnigmaUtils,
  Secp256k1Pen,
  SigningCosmWasmClient,
  pubkeyToAddress,
  encodeSecp256k1Pubkey,
} = require("secretjs");

// Requiring the dotenv package in this way
// lets us use environment variables defined in .env
require("dotenv").config();

const customFees = {
  upload: {
    amount: [{ amount: "2000000", denom: "uscrt" }],
    gas: "2000000",
  },
  init: {
    amount: [{ amount: "500000", denom: "uscrt" }],
    gas: "500000",
  },
  exec: {
    amount: [{ amount: "500000", denom: "uscrt" }],
    gas: "500000",
  },
  send: {
    amount: [{ amount: "80000", denom: "uscrt" }],
    gas: "80000",
  },
};

const main = async () => {
  const httpUrl = process.env.SECRET_REST_URL;

  // Use the mnemonic created in step #2 of the Secret Pathway
  const mnemonic = process.env.MNEMONIC;

  // A pen is the most basic tool you can think of for signing.
  // This wraps a single keypair and allows for signing.
  const signingPen = await Secp256k1Pen.fromMnemonic(mnemonic).catch((err) => {
    throw new Error(`Could not get signing pen: ${err}`);
  });

  // Get the public key
  const pubkey = encodeSecp256k1Pubkey(signingPen.pubkey);

  // get the wallet address
  const accAddress = pubkeyToAddress(pubkey, "secret");

  // initialize client
  const txEncryptionSeed = EnigmaUtils.GenerateNewSeed();

  const client = new SigningCosmWasmClient(
    httpUrl,
    accAddress,
    (signBytes) => signingPen.sign(signBytes),
    txEncryptionSeed,
    customFees
  );
  console.log(`Wallet address=${accAddress}`);

  // 1. Define your metadata

  // 2. Mint a new token to yourself

};

main().catch((err) => {
  console.error(err);
});
```

Here we are getting our account data from the `.env` file and generating a secretjs client from it, which we can use to sign our transactions. The next step is to define the metadata that we want to attach to our Secret NFT. In this example, we use strings: One public, one private and only accessible to the holder of the NFT. Under `// 1. Define your metadata` add both constant declarations. Feel free to change the string to whatever you like. If we wanted to use string interpolation to import data programmatically, remember to swap the double quotes with backticks!

```javascript
  // String interpolation example
  // const derivedMetadata = "<additional payload>";
  // const publicMetadata = `<public metadata> ${derivedMetadata}`;


  const publicMetadata = "<public metadata>";
  const privateMetadata = "<private metadata>";
```

The next step is for us to construct a message object to pass into our client, populate it with data, then send it to our pre-defined contract. Paste the snippet below under `// 2. Mint a new token to yourself`.

```javascript
  const handleMsg = {
    mint_nft: {
      owner: accAddress,
      public_metadata: {
        name: publicMetadata,
      },
      private_metadata: {
        name: privateMetadata,
      },
    },
  };

  console.log("Minting yourself a nft");
  const response = await client
    .execute(process.env.SECRET_NFT_CONTRACT, handleMsg)
    .catch((err) => {
      throw new Error(`Could not execute contract: ${err}`);
    });
  console.log("response: ", response);
```

`handleMsg` is the data we pass into our Secret client, which mints a new token for our own address then assigns the values we created above as public and private metadata. We pass and execute the object to the contract and read the response. If there were no errors, the output will be in the form of the reponse object:

```javascript
  response:  {
  logs: [ { msg_index: 0, log: '', events: [Array] } ],
  transactionHash: '644332C51EEB404E68B0B73BDEFAD36209A2C78C532DDFF03F83AD3AD68EE5F13',
  data: Uint8Array(256) [
    123, 34, 109, 105, 110, 116,  95, 110, 102, 116, 34, 58,
    123, 34, 116, 111, 107, 101, 110,  95, 105, 100, 34, 58,
     34, 48,  34, 125, 125,  32,  32,  32,  32,  32, 32, 32,
     32, 32,  32,  32,  32,  32,  32,  32,  32,  32, 32, 32,
     32, 32,  32,  32,  32,  32,  32,  32,  32,  32, 32, 32,
     32, 32,  32,  32,  32,  32,  32,  32,  32,  32, 32, 32,
     32, 32,  32,  32,  32,  32,  32,  32,  32,  32, 32, 32,
     32, 32,  32,  32,  32,  32,  32,  32,  32,  32, 32, 32,
     32, 32,  32,  32,
    ... 156 more items
  ]
}
```

Congratulations! We have just minted a Non-Fungible Token on the Secret network using an already deployed contract!

### Querying the contract

In this section, we want to query information about our token from the contract. For this create a new file called `query_token.js` and paste the following code into it:

```javascript
const {
  EnigmaUtils,
  Secp256k1Pen,
  SigningCosmWasmClient,
  pubkeyToAddress,
  encodeSecp256k1Pubkey,
} = require('secretjs');

// Load environment variables
require('dotenv').config();

const customFees = {
  upload: {
    amount: [{ amount: '2000000', denom: 'uscrt' }],
    gas: '2000000',
  },
  init: {
    amount: [{ amount: '500000', denom: 'uscrt' }],
    gas: '500000',
  },
  exec: {
    amount: [{ amount: '500000', denom: 'uscrt' }],
    gas: '500000',
  },
  send: {
    amount: [{ amount: '80000', denom: 'uscrt' }],
    gas: '80000',
  },
};

const main = async () => {
  const httpUrl = process.env.SECRET_REST_URL;

  // Use key created in tutorial #2
  const mnemonic = process.env.MNEMONIC;

  // A pen is the most basic tool you can think of for signing.
  // This wraps a single keypair and allows for signing.
  const signingPen = await Secp256k1Pen.fromMnemonic(mnemonic).catch((err) => {
    throw new Error(`Could not get signing pen: ${err}`);
  });

  // Get the public key
  const pubkey = encodeSecp256k1Pubkey(signingPen.pubkey);

  // get the wallet address
  const accAddress = pubkeyToAddress(pubkey, 'secret');

  // initialize client
  const txEncryptionSeed = EnigmaUtils.GenerateNewSeed();

  const client = new SigningCosmWasmClient(
    httpUrl,
    accAddress,
    (signBytes) => signingPen.sign(signBytes),
    txEncryptionSeed,
    customFees
  );
  console.log(`Wallet address=${accAddress}`);

  // 1. Get a list of all tokens

  if (response.token_list.tokens.length == 0)
    console.log(
      'No token was found for you account, make sure that the minting step completed successfully'
    );
  const token_id = response.token_list.tokens[0];

  // 2. Query the public metadata

  // 3. Query the token dossier

  // 4. Set our viewing key

  // 5. Query the dossier again

};

main().catch((err) => {
  console.error(err);
});
```

The overall structure of this file should look familiar by now. It contains the setup for the client and signing functionality that we will use to interact with the contract. The first step is to get a list of all our tokens. For this, we will add the following code under `// 1. Get a list of all tokens`:

```javascript
 let queryMsg = {
    tokens: {
      owner: accAddress,
    },
  };

  console.log("Reading all tokens");
  let response = await client
    .queryContractSmart(process.env.SECRET_NFT_CONTRACT, queryMsg)
    .catch((err) => {
      throw new Error(`Could not execute contract: ${err}`);
    });
  console.log("response: ", response);
```

We will ask the contract for a list of all token ids, which are currently assigned to our address. _**Note -**_ The deployed contract is configured to have public ownership for the sake of this tutorial. If you would deploy the contract yourself you can also decide to make it private, so that nobody but you will be able to see which tokens you own. This will be covered in a later installment of this series.

Next, we will take the first token id and ask for the details of this NFT. There are a few different queries supported by the reference implementation. After `// 2. Query the public metadata` insert this code block:

```javascript
  queryMsg = {
    nft_info: {
      token_id: token_id,
    },
  };

  console.log(`Query public data of token #${token_id}`);
  response = await client
    .queryContractSmart(process.env.SECRET_NFT_CONTRACT, queryMsg)
    .catch((err) => {
      throw new Error(`Could not execute contract: ${err}`);
    });
  console.log("response: ", response);
```

This will retrieve the message you added as `publicMetadata` during the mint process. Next, add this code block after `// 3. Query the token dossier`:

```javascript
 queryMsg = {
    nft_dossier: {
      token_id: token_id,
    },
  };

  console.log(`Query dossier of token #${token_id}`);
  response = await client
    .queryContractSmart(process.env.SECRET_NFT_CONTRACT, queryMsg)
    .catch((err) => {
      throw new Error(`Could not execute contract: ${err}`);
    });
  console.log("response: ", response);
```

The token dossier returns all of the data that we have access to, including public and private metadata, as well as some overall configuration of this token. If you run this file using `node query-token.js` you should receive similar output to this:

```text
Reading all tokens
response:  { token_list: { tokens: [ '0' ] } }
Query public data of token #0
response:  {
  nft_info: {
    name: '<your public meta data>',
    description: null,
    image: null
  }
}
Query dossier of token #0
response:  {
  nft_dossier: {
    owner: '<your address>',
    public_metadata: {
      name: '<your public meta data>',
      description: null,
      image: null
    },
    private_metadata: null,
    display_private_metadata_error: 'You are not authorized to perform this action on token 0',
    owner_is_public: true,
    public_ownership_expiration: 'never',
    private_metadata_is_public: false,
    private_metadata_is_public_expiration: null,
    token_approvals: null,
    inventory_approvals: null
  }
}
```

Of interest is this line: `display_private_metadata_error: 'You are not authorized to perform this action on token 0',`. Why are we not able to see our private metadata even if we _are_ owning the NFT?

To answer this we need to take a small detour through the inner workings of secret contract interactions.

Interaction with a contract falls into one of two distinct categories, you either _execute_ a contract or you _query_ a contract. The main difference here is that execution modifies the internal state of the contract and incurs a gas fee for doing so, while a query only reads from the internal state and it usually free.

_**Note -**_ Queries also have a calculated gas cost but are executed for free by the secret nodes. Node runners can define how expensive a query might be until they refuse to execute it. Also if you run a query during an execution call, its gas cost will be added to the total cost of the query.

In regards to the [privacy model](https://build.scrt.network/dev/privacy-model-of-secret-contracts.html#outputs) of Secret contracts - whenever we send a query, the sending address is not verified, as a query does not happen on-chain but rather just between the client and the contract. This means ultimately that we need to execute on a contract if we want to make sure who the contract is talking to. This is not optimal because of gas costs.

This leads to the problem above: We do not want to execute on the contract, because we are not changing its inner state. As a query is not recorded on-chain we have no way of making sure _who_ is sending that query. So even if we tell the contract that the query was sent by us, it rejects our request for the private data, as there is no tamperproof way to make _sure_ it is us.

The concept of Viewing keys is a clever solution for this problem. Instead of executing every time you will execute a single call and set a secret phrase, the so called key, for the contract. Whenever you send a query in the future you include this key with your query and the contract can look it up and validate it is actually you it is talking to. As secret network transmits fully encrypted messages no one will ever know your viewing keys.

To be able to use this mechanic to get access to our private metadata, we must add a string for the viewing key to our `.env` file, so that we can reuse it in our code.

```
SECRET_VIEWING_KEY = "<random phrase>"
```

We will use this phrase in the following code, which should go add after `// 4. Set our viewing key`, as a kind of passphrase, uniquely identifying our address.

```javascript
 const handleMsg = {
    set_viewing_key: {
      key: process.env.SECRET_VIEWING_KEY,
    },
  };

  console.log('Set viewing key');
  response = await client
    .execute(process.env.SECRET_NFT_CONTRACT, handleMsg)
    .catch((err) => {
      throw new Error(`Could not execute contract: ${err}`);
    });
  console.log('response: ', response);
```

After we set our viewing key, we also need to pass it along with the query for the token dossier. Add the following code after `// 5. Query the dossier again`.

```javascript
queryMsg = {
    nft_dossier: {
      token_id: token_id,
      viewer: {
        address: accAddress,
        viewing_key: process.env.SECRET_VIEWING_KEY,
      },
    },
  };

  console.log(`Query dossier of token #${token_id} with viewing key`);
  response = await client
    .queryContractSmart(process.env.SECRET_NFT_CONTRACT, queryMsg)
    .catch((err) => {
      throw new Error(`Could not execute contract: ${err}`);
    });
  console.log('response: ', response);
```

Take note of how we changed `queryMsg` to specify who is viewing the contract information, passing our own address and the Viewing key. When we run the script at this point using `node query_token.js` there will be two results of querying the contract in your output. The first one, that you already saw above has no access to the private metadata. The second entry where we supplied the Viewing key, will return the private metadata along with all the other info and should look like this:

```
response:  {
  nft_dossier: {
    owner: '<your address>',
    public_metadata: {
      name: '<your public meta data>',
      description: null,
      image: null
    },
    private_metadata: {
      name: '<your private meta data>',
      description: null,
      image: null
    },
    display_private_metadata_error: null,
    owner_is_public: true,
    public_ownership_expiration: 'never',
    private_metadata_is_public: false,
    private_metadata_is_public_expiration: null,
    token_approvals: [],
    inventory_approvals: []
  }
}
```

# Conclusion

Congratulations! We have made it to the end of the first installment of this Secret NFT series. We have covered a lot of information, and I feel you can really be proud of what you have achieved. Just to recap:

* You have minted your own, personalized NFT on the Secret network
* You learned about the privacy model and how Viewing keys help us to save gas, while making sure we can verify ownership of an account
* You had hands-on experience interacting with the Secret network, learning about public, private and extended metadata.

This is a solid foundation to play with and build upon!

# Next Steps

Of course, we are not yet at the end of our journey. In the coming tutorial, we will have a look together into interesting and more complex examples of NFT properties. You can look forward to:

* Customizing your NFT by deploying your own contract
* Sending NFTs between different accounts
* Building a simple NFT viewer
* Adding complex logic to your NFTs, making them much more than _just_ a store of secret data

Let's keep on building!

# About the author

This tutorial was created by [Florian Uhde](https://twitter.com/florianuhde), a software engineer and game developer with a passion for blockchain, creativity and systemic design. You can get in touch with the author on [Figment Forum](https://community.figment.io/u/floar) if you have any queries pertaining to the tutorial, secretNFTs, etc.hello
