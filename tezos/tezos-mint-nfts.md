# Introduction

Hi, in this tutorial, we will learn how to mint NFTs on Tezos. We will write a smart contract in SmartPy and deploy it, upload the NFT metadata on IPFS and mint the token using better-call.dev. We will also cover how to mint it using React and taquito.

# Prerequisites

- Basic Knowledge of SmartPy
- React

# Requirements

- Temple Wallet Installed in your browser. Get it from https://templewallet.com/
- Few XTZs on Granada Testnet is also needed in the wallet account. To get it, you can refer to Faucet and Temple Wallet section in this [tutorial](https://learn.figment.io/tutorials/using-the-smartpy-ide-to-deploy-tezos-smart-contracts#faucet-and-temple-wallet).
- NodeJS, npm and git needs to be installed for the frontend.

# SmartPy

SmartPy is an intuitive and powerful smart contract development platform for Tezos.  
The SmartPy language is available through a Python library for building and analyzing Tezos smart contracts.  
And with its IDE, we can quickly write our code in the browser itself. We can also deploy our code from the browser. No local setup is required.  
So, Let's start. Open https://smartpy.io/ide

![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_1.png)

You can explore and look at the template codes given.  
To move to the editor, click CLOSE.

# Token Contract and FA2 Standard

The left panel is where we are going to code.  
We are going to follow the FA2 token standard([TZIP-12](https://tzip.tezosagora.org/proposal/tzip-12/)) for our NFTs.  
Let’s start coding our contract.

First, we will import the smartpy library

```python
import smartpy as sp
```

Now we will import the FA2 template. Most of the features(mint) we need have already been implemented in this template.

```python
FA2 = sp.io.import_template("FA2.py")
```

Let’s use this template and inherit it in our NFT class, and the entry points we need for this tutorial are already in the template, so we don’t need to change our class and simply pass. And this will be our contract.

```python
class NFT(FA2.FA2):
   pass
```

Let’s write some tests for our contract.

We need some test accounts for tests; let's add two test accounts, Tom and Jerry. And as we also need to set admin for our contract, we require that also. For admin also, you can use a test account and set your address as admin later from storage JSON while deploying, but I prefer putting it in tests itself.

```python
@sp.add_test(name="tests")
def test():
  jerry = sp.test_account("Jerry")
  tom = sp.test_account("Tom")
  admin = sp.address("tz1aV2DuPYXEK2mEVc4VBhP9o4gguFGxBky4")
```

Change the address inside the sp.address to your address(your wallet account public key).

For tests, we need to create test_scenario and assign a heading to it using h1.

```python
  scenario = sp.test_scenario()
  scenario.h1("tutorial tests")
```

Now we will originate our NFT contract. As we inherit the FA2 template, we need to understand what parameters we need to send and can change that according to our needs.
We set non_fungible to true in the FA2_config, pass the admin address and set the metadata of the contract. For metadata, we will use big_map, in which first we set where to look for the metadata. Here from tezos-storage:content we can understand we need to look for content in the storage. And in storage, we will define the metadata. We will define the name and description.  
You can refer to [TZIP-16](https://tzip.tezosagora.org/proposal/tzip-16/) for more details on contract metadata.

```python
  nft = NFT(FA2.FA2_config(non_fungible=True), admin=admin, metadata= sp.big_map({"": sp.utils.bytes_of_string("tezos-storage:content"),"content": sp.utils.bytes_of_string("""{"name": "Tutorial Contract", "description": "NFT contract for the tutorial"}""")}))
```

Add the nft to scenario

```python
  scenario += nft
```

Test the mint function.  
We will send three parameters to the mint entry point.

- token_id  
  the token id of the NFT usually starts with 0 and then needs to be increment by 1 for each subsequent NFT.

- address  
  the address on which we want to mint the token.

- metadata  
  for token metadata, we are using IPFS (will come to it later in this tutorial).  
  The metadata is stored in a map(string, bytes) so, we need to convert the strings to IPFS for which sp.utils.bytes_of_string is used.  
  For now, use the same IPFS URL (ipfs://bafkreih36m3d4yfbpyteluvntuph5xybwtgxdvyksbgyg66es44drk4hqy)

And remember only the admin can run the mint query.

```python
  nft.mint(token_id=0, address=jerry.address, amount=1, metadata = sp.map({"": sp.utils.bytes_of_string("ipfs://bafkreih36m3d4yfbpyteluvntuph5xybwtgxdvyksbgyg66es44drk4hqy")})).run(sender=admin)
```

This is our contract.  
Here's the whole code you can refer to:

```python
import smartpy as sp
FA2 = sp.io.import_template("FA2.py")

class NFT(FA2.FA2):
   pass

@sp.add_test(name="tests")
def test():
  jerry = sp.test_account("Jerry")
  tom = sp.test_account("Tom")
  admin = sp.address("tz1aV2DuPYXEK2mEVc4VBhP9o4gguFGxBky4")
  scenario = sp.test_scenario()
  scenario.h1("tutorial tests")
  nft = NFT(FA2.FA2_config(non_fungible=True), admin=admin, metadata= sp.big_map({"": sp.utils.bytes_of_string("tezos-storage:content"),"content": sp.utils.bytes_of_string("""{"name": "Tutorial Contract", "description": "NFT contract for the tutorial"}""")}))
  scenario += nft
  nft.mint(token_id=0, address=jerry.address, amount=1, metadata = sp.map({"": sp.utils.bytes_of_string("ipfs://bafkreih36m3d4yfbpyteluvntuph5xybwtgxdvyksbgyg66es44drk4hqy")})).run(sender=admin)

```

Run the code using the RUN button above the left(editor) panel or can use ctrl+enter also, cmd+enter for mac.

![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_run.png)

If there are no errors, we will get the output in the right terminal.  
We can check it according to the tests defined. We defined the heading in h1 in tests as “tutorial tests”, so we can see that on the top. Other than that, we can look at the NFT contract and on scrolling, we can see the mint transaction as defined in the tests. So this looks perfect, and our contract is working correctly.

![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_output.png)

# Deploy Contract

It’s time to deploy this contract. For this tutorial, we are going to deploy the contract on Granada testnet.  
We can deploy the contract using smartpy IDE itself.
After running, we get the option to Deploy Michelson Contract in the output panel. Move to it.

Click on the Deploy Michelson Contract button.

![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_deploy_1.png)

A new tab will open up.

![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_deploy_2.png)

Choose Granadanet in the Node. Now Click on Temple Wallet. Choose one account (if you have multiple). And then press connect.

Account loaded with success will come.
![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_deploy_3.png)

Scroll down below and click on Estimate Cost From RPC. It will refresh the cost. And after that, click on the DEPLOY CONTRACT button.
![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_deploy_4.png)

Accept the pre-signature information.

![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_deploy_5.png)

Then your wallet popup will come. Press sign.

Contract Originated Successfully will be shown. Copy the contract address(KT1Rv5Nvc4A2e4hTHi34QhChYFdJ2aBjAp7P) and store it.

![SmartPy](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_smartpy_deploy_6.png)

Hurray, we deployed our contract successfully.

# IPFS

IPFS is a distributed system for storing and accessing files, websites, applications, and data.  
And, we will use IPFS to store the metadata of our tokens. And to upload our files to IPFS, we will use nft.storage. It provides us with a user interface and also a js library to upload files.  
In this tutorial, we will create a NFT of an image. Firstly, we will upload the image and then use the image hash in the token metadata.
So to upload files, visit [nft.storage](https://nft.storage/).

![IPFS](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_ipfs_1.png)

Log in and click on upload and choose the image file you want to upload and upload.

![IPFS](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_ipfs_2.png)

Upload it now.  
The file will get uploaded, and this is your hash.

![IPFS](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_ipfs_3.png)

You can view your files visiting ipfs://{hash} (If your browser doesn’t support IPFS, you can use https://ipfs.io/ipfs/{hash} )

For token metadata, decimals is the required field according to the FA2 standard.  
name and symbol fields are highly recommended.
And according to [TZIP-21](https://tzip.tezosagora.org/proposal/tzip-21/) for NFTs, these are some other recommended fields.

- artifactUri
- displayUri
- thumbnailUri
- description
- minter
- creators
- isBooleanAmount

We will be storing the metadata in JSON file. Create `metadata.json` and open it in any editor of your choice.

```json
{
  "name": "First NFT",
  "symbol": "TUT",
  "description": "First NFT",
  "decimals": 0,
  "isTransferable": true,
  "shouldPreferSymbol": false,
  "isBooleanAmount": true,
  "artifactUri": "ipfs://bafkreiezptgh36vfxpsux4ebj3oyvpephpmhabey2v2p4yuwzkzjyzynqq",
  "displayUri": "ipfs://bafkreiezptgh36vfxpsux4ebj3oyvpephpmhabey2v2p4yuwzkzjyzynqq",
  "thumbnailUri": "ipfs://bafkreiezptgh36vfxpsux4ebj3oyvpephpmhabey2v2p4yuwzkzjyzynqq",
  "creators": ["priyanshu"]
}
```

Here, we will be using the same IPFS URL for artifactUri, displayUri and thumbnailUri.  
For NFTs, decimals must be 0, creators is an array of string and isBooleanAmount is boolean, and it describes whether an account can have an amount of exactly 0 or 1.  
There are other fields also which can be used. You can read the details of all this in the [TZIP-21](https://tzip.tezosagora.org/proposal/tzip-21/).

Now we will upload this JSON file to IPFS using nft.storage.

**Small Exercise:-**  
Repeat the same procedure we did to upload the image and upload this JSON file and store the hash that will be needed later. Also, view the file in browser.

# Mint using better-call.dev

It’s time to mint our first NFT. Open up [better-call.dev](https://better-call.dev/).

Move to interact tab, open mint from the right sidebar.  
Put the address you want to mint the NFT to. The amount should be 1 and token_id to 0. And for metadata, we will take the hash of the `metadata.json` file we uploaded above and use ipfs://{hash} to set the token metadata. But for the token metadata, the ipfs URL needs to be in bytes. You can convert the strings to bytes from [here](https://tutorial-mint-nfts.vercel.app/char2bytes).

![better-call.dev](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_better_1.png)

Press Execute and choose the wallet option. From the list of wallets, choose the temple wallet. And then use the account which was set to admin for the contract. This is important; we won’t be able to mint using any other address. Only admin can mint according to our smart contract.

![better-call.dev](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_better_2.png)

You can check the fees and also explore the operations which wallet is going to send. Then click confirm.

![better-call.dev](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_better_3.png)

The message will come if it’s successfully sent.

![better-call.dev](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_better_4.png)

After few seconds, refresh and move to the operations tab, you can see the mint operation.

![better-call.dev](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_better_5.png)

You can look at the NFT token metadata in tokens section.

![better-call.dev](https://github.com/figment-networks/learn-tutorials/raw/master/assets/mint_better_6.png)

Hurray, we minted our token.

# Frontend

Import this boilerplate code using the following command.

```text
git clone -b start --single-branch https://github.com/PriyanshuDangi/Tutorial_Mint_NFTs.git
```

cd into the folder

```text
cd Tutorial_Mint_NFTs
```

Install all the dependencies

```text
npm i
```

Try running the app using

```text
npm start
```

And visit localhost:3000 to view the app. There won't be much as of now. So let's build it.

The boilerplate has the folder structure and pre-code for redux storage, where we will store the NFTs to show.  
And there are three pages.

- Char2bytes (we used before, code is already in boilerplate)
- Home (All the NFTs will be visible here)
- Mint (Form to mint the NFTs)

Open `src/config/config.js`. This is where we store our configurations.

```js
export const NAME = 'Tutorial'; // the name of the app
export const CONTRACT_ADDRESS = 'KT1J4Qk8pfTeMEmN9WW9x12smt3ce5yXE4yZ';
export const RPC_URL = 'https://granadanet.smartpy.io'; // network rpc url
export const NETWORK = 'granadanet';
```

Change the name and contract address accordingly, and as we deployed to Granada Testnet, we are using that network. If deployed on any other network, it can be changed accordingly.

Now let's code some of the utilities we will be using. Open `src/utils/wallet.js`.
In this file, there will be all our wallets and interacting with smart contract code.

Let’s import the TezosToolkit and beacon SDK, which we will use to connect to the wallet.
We also need to import our config object, axios and bytes2Char and char2Bytes from the Taquito utils.

```js
import {TezosToolkit, MichelsonMap} from '@taquito/taquito';
import {BeaconWallet} from '@taquito/beacon-wallet';
import * as config from '../config/config';
import {bytes2Char, char2Bytes} from '@taquito/utils';
import axios from 'axios';
```

Next, we will be configuring the beacon wallet options and setting the wallet provider.

```js
const Tezos = new TezosToolkit(config.RPC_URL);

const options = {
  name: config.NAME,
  iconUrl: 'https://tezostaquito.io/img/favicon.png',
  preferredNetwork: config.NETWORK,
};

const wallet = new BeaconWallet(options);

Tezos.setWalletProvider(wallet);
```

Next, we will write functions to connect the wallet (requesting the user permission) and disconnect the wallet.

```js
const connectWallet = async () => {
  await wallet.requestPermissions({
    network: {
      type: config.NETWORK,
    },
  });
  return wallet;
};

const disconnectWallet = async () => {
  await wallet.clearActiveAccount();
};
```

To get the Public Key Hash (pkh) of the user and get the contract will use the following functions.

```js
const getPKH = async () => {
  const pkh = await wallet.getPKH();
  return pkh;
};

const getContract = async () => {
  const contract = await Tezos.wallet.at(config.CONTRACT_ADDRESS);
  return contract;
};
```

Now to mint NFTs we need the user to connect his wallet and then sign the mint method.

```js
const mintNFT = async (address, url, token_id) => {
  await disconnectWallet();
  await connectWallet();
  const amount = 1;
  const contract = await getContract();
  url = char2Bytes(url);
  const op = await contract.methods.mint(address, amount, MichelsonMap.fromLiteral({'': url}), token_id).send();
  return await op.confirmation(3);
};
```

We will be using the tzkt api to get all the token_metadata.

```js
const getNFTs = async () => {
  const response = await axios.get(
    `https://api.granadanet.tzkt.io/v1/contracts/${config.CONTRACT_ADDRESS}/bigmaps/token_metadata/keys`
  );
  const data = response.data;
  let tokens = [];
  for (let i = 0; i < data.length; i++) {
    let url = data[i].value.token_info[''];
    if (url) {
      url = bytes2Char(url);
    }
    const token = {
      token_id: data[i].value.token_id,
      url,
    };
    tokens.push(token);
  }
  return tokens;
};
```

And exporting all these functions from the file at the end.

```js
export {connectWallet, disconnectWallet, getPKH, getContract, mintNFT, getNFTs};
```

This was our `src/utils/wallet.js`.

We will also be needing one more util, uploadFileToIPFS.  
For that, you need to get the API Keys from nft.storage. Login and get one.
Open `src/utils/upload.js`.

```js
import {NFTStorage, File} from 'nft.storage';

const symbol = 'TUT';

const apiKey = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJkaWQ6ZXRocjoweDY4QTMwQzA1ZjY3RTc3NTc3MjI2RjBlOEFmNjQzODA4ZDc2MzA1ZTQiLCJpc3MiOiJuZnQtc3RvcmFnZSIsImlhdCI6MTYzNDEzMDUwNTkzMCwibmFtZSI6Im1pbnRfdHV0b3JpYWwifQ.jCpzhkEJdFPuM0NtKtJoywX__m6xrJuPRmbagccwarU';
const client = new NFTStorage({token: apiKey});

const uploadToIpfs = async (name, description, imgFile) => {
  const metadata = await client.store({
    name: name,
    description: description,
    image: new File([imgFile], imgFile.name, {type: imgFile.type}),
    symbol: symbol,
    decimals: 0,
    shouldPreferSymbol: false,
    isBooleanAmount: true,
    artifactUri: new File([imgFile], imgFile.name, {type: imgFile.type}),
    displayUri: new File([imgFile], imgFile.name, {type: imgFile.type}),
    thumbnailUri: new File([imgFile], imgFile.name, {type: imgFile.type}),
    creators: ['priyanshu'],
  });
  return metadata.url;
};

export {uploadToIpfs};
```

Change the API Key, and to the function, we will be sending the name, description and our imgFile, and it will return us the IPFS URL after uploading it to IPFS.

Now in `src/App.js`, add:

```jsx
import {getNFTs} from './utils/wallet';
```

And uncomment these lines(line 22, 23)

```jsx
const nfts = await getNFTs();
dispatch(setStorage(nfts));
```

This will get all the NFTs metadata URLs and store them in the redux storage.

Now lets code our `src/components/home/Home.jsx` (remove the already present code).  
Here we will show all our NFTS.  
We have a card for each NFT that will show the image, name and description.

```jsx
import axios from 'axios';
import React, {useEffect, useState} from 'react';
import {useSelector} from 'react-redux';
import {selectLoaded, selectStorage} from '../../store/reducers/storage';

const Card = (props) => {
  return (
    <div className="col">
      <div className="card h-100">
        <img src={props.img} className="card-img-top" alt="..." />
        <div className="card-body">
          <h5 className="card-title">{props.name}</h5>
          <p className="card-text">{props.description}</p>
        </div>
      </div>
    </div>
  );
};
```

Next, we have our Home component. In this, we are using useEffect, and as the storage gets loaded, we are taking the image, name and description from it and storing it in our nfts state.  
We need to change the URL format as we can't fetch ipfs:// using axios as it’s not supported for now. So we are using the HTTPS URL for it.

```jsx
const Home = () => {
  const loaded = useSelector(selectLoaded);
  const storage = useSelector(selectStorage);
  const [nfts, setNfts] = useState([]);

  useEffect(() => {
    const func = async () => {
      let tokens = [];
      for (let i = 0; i < storage.length; i++) {
        if (storage[i].url) {
          let token = {};
          token.token_id = storage[i].token_id;
          const response = await axios.get('https://ipfs.io/ipfs/' + storage[i].url.slice(7));
          let data = response.data;
          token.name = data.name;
          token.description = data.description;
          if (data.artifactUri) token.img = 'https://ipfs.io/ipfs/' + data.artifactUri.slice(7);
          tokens.push(token);
        }
      }
      setNfts(tokens);
      console.log('storage set!');
    };
    if (loaded) {
      func();
    }
  }, [loaded]);

  return (
    <div>
      <div className="row row-cols-1 row-cols-md-3 g-4">
        {nfts.map((nft, index) => {
          return <Card {...nft} key={index} />;
        })}
      </div>
    </div>
  );
};

export default Home;
```

Time for `src/components/mint/Mint.jsx` (remove the already present code).
Let’s import everything first.

```jsx
import React, {useState} from 'react';
import {uploadToIpfs} from '../../utils/upload';

import {mintNFT} from '../../utils/wallet';
import {useSelector} from 'react-redux';
import {selectLoaded, selectStorage} from '../../store/reducers/storage';
```

Now we will set the state and selectors.

```jsx
const Mint = () => {
    const loaded = useSelector(selectLoaded);
    const storage = useSelector(selectStorage);
    const [loading, setLoading] = useState(false);
    const [message, setMessage] = useState('');
```

In our submit function, we take all the details and upload to IPFS using the uploadToIPFS util we coded above. And then use this URL to mint using the mintNFT util from `src/utils/wallet.js`.
And once minted, we add the NFT to redux storage.

```jsx
  const submit = async (event) => {
    try {
      event.preventDefault();
      setLoading(true);
      const address = event.target.address.value;
      const name = event.target.name.value;
      const description = event.target.description.value;
      const file = event.target.image.files[0];

      const ipfsUrl = await uploadToIpfs(name, description, file);
      console.log('Uploaded To IPFS!');
      const token_id = storage.length;
      const op = await mintNFT(address, ipfsUrl, token_id);

      console.log(op);
      setMessage('Minted Successfully!');

      const stor = [...storage];
      stor.push({
        token_id: token_id,
        url: ipfsUrl,
      });

      setLoading(false);
    } catch (err) {
      console.log(err);
      setLoading(false);
      setMessage('Error: Not Able to Mint');
    }
  };
```

To close the toast, we set the message to empty.

```jsx
  const closeMessage = () => {
    setMessage('');
  };
```

This is our page code

```jsx
  return (
    <div className="container">
      {message && (
        <div className="position-fixed top-0 end-0 p-3" style={{zIndex: 11}}>
          <div id="liveToast" className="toast fade show d-flex" role="alert" aria-live="assertive" aria-atomic="true">
            <div className="toast-body">{message}</div>
            <button type="button" className="btn-close me-2 m-auto" onClick={closeMessage}></button>
          </div>
        </div>
      )}
      <form onSubmit={submit}>
        <div className="mb-3">
          <label htmlFor="tokenID" className="form-label">
            Token ID
          </label>
          <input
            type="number"
            className="form-control"
            id="tokenID"
            aria-describedby="tokenID"
            required
            value={storage.length}
            disabled
          />
        </div>
        <div className="mb-3">
          <label htmlFor="address" className="form-label">
            Address
          </label>
          <input type="text" className="form-control" id="address" aria-describedby="address" required />
        </div>
        <div className="mb-3">
          <label htmlFor="name" className="form-label">
            Name
          </label>
          <input type="text" className="form-control" id="name" aria-describedby="name" required />
        </div>
        <div className="mb-3">
          <label htmlFor="description" className="form-label">
            Description
          </label>
          <input type="text" className="form-control" id="description" aria-describedby="description" required />
        </div>
        <div className="mb-3">
          <label htmlFor="image" className="form-label">
            Image
          </label>
          <input className="form-control" type="file" id="image" accept="image/*" required />
        </div>
        <button type="submit" className="btn btn-primary" disabled={!loaded || loading}>
          {loading && <span className="spinner-border spinner-border-sm" role="status" aria-hidden="true"></span>}
          Mint
        </button>
      </form>
    </div>
  );
```

And in the end, we close the Mint function and default export it.

```jsx
};

export default Mint;
```

Now run in terminal

```text
npm start
```

Open locahost:3000 in browser and mint a token using `/mint` and view them on `/`.

We completed the frontend. You can checkout the complete code on [Github](https://github.com/PriyanshuDangi/Tutorial_Mint_NFTs) and view the demo [here](tutorial-mint-nfts.vercel.app)

# Conclusion

In this tutorial, we learned about coding in SmartPy, deploying a contract on the blockchain, and interacting with the contract using better-call.dev. We also built the frontend using React and learned how to interact using taquito.

And most importantly, we learned how to mint NFTs on Tezos.

# Next Steps

Try out all the other entry points of FA2. You can modify the smart code and using transfer entrypoint can make a marketplace for NFTs. Can refer to this [tutorial](https://learn.figment.io/tutorials/create-an-nft-marketplace-on-tezos).

# About The Author

This tutorial was written by Priyanshu Dangi, who is a Tezos India 2.0 Fellow. You can reach out to him on [Github](https://github.com/PriyanshuDangi).
