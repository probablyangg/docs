# Key Management Strategies

The following article aims to be a guide through integration of a key management strategy on client side of your Decentralised Application on Matic Network.

The following strategies will be discussed:

- Metamask
- Wallet Connect
- Portis

The overall steps would essentially remain the same for any client side application to talk to the blockchain:
 
1. **Set up Web3**: [web3.js](https://web3js.readthedocs.io/) is a javascript library that allows our client-side application to talk to the blockchain. We configure web3 to communicate via Metamask/Wallet Connect/Portis. 
> Note: Refer [Web3.js](https://web3js.readthedocs.io/en/v1.2.2/getting-started.html#adding-web3-js) docs to 
add web3 to your project 
2. **Set up Account**: To send transactions from (specifically for transactions that alter the state of the blockchain) 
3. **Instantiate contracts**: Once we have our web3 object in place, we next instantiate our deployed contract, with which we interact. 
4. **Call functions**: we fetch data via functions in the contract - through our contract object.

> The article is made with Vue.js client side code in mind - but the flow and functions would essentially remain the same for any other framework.

## Metamask

Metamask is a browser add-on that manages a user’s Ethereum wallet by storing their private key on their browser’s data store and the seed phrase encrypted with their password. It is a non-custodial wallet, meaning, the user has full access and responsibility their private key. Once lost, the user can no longer control the savings or restore access to the wallet.

**Type**: Non-custodial/HD 
**Private Key Storage**: User’s local browser storage 
**Communication to Ethereum Ledger**: Infura 
**Private key encoding**: Mnemonic

### 1. Set up Web3

**Step 1**

Install the following in your DApp:

    npm install --save web3

Create a new file, name it `web3.js` and insert the following code in it:

    import Web3 from 'web3';
    
    const getWeb3 = () => new Promise((resolve) => {
      window.addEventListener('load', () => {
        let currentWeb3;
    
        if (window.ethereum) {
          currentWeb3 = new Web3(window.ethereum);
          try {
            // Request account access if needed
            window.ethereum.enable();
            // Acccounts now exposed
            resolve(currentWeb3);
          } catch (error) {
            // User denied account access...
            alert('Please allow access for the app to work');
          }
        } else if (window.web3) {
          window.web3 = new Web3(web3.currentProvider);
          // Acccounts always exposed
          resolve(currentWeb3);
        } else {
          console.log('Non-Ethereum browser detected. You should consider trying MetaMask!');
        }
      });
    });
    
    export default getWeb3;

The above file exports a function called `getWeb3()` - the purpose of which is to request metamask account’s access via detecting a global object (`ethereum` or `web3`) injected by Metamask. According to [Metamask’s API documentation](https://metamask.github.io/metamask-docs/API_Reference/Ethereum_Provider#api-reference): > MetaMask injects a global API into websites visited by its users at window.ethereum (Also available at window.web3.currentProvider for legacy reasons). This API allows websites to request user login, load data from blockchains the user has a connection to, and suggest the user sign messages and transactions. You can use this API to detect the user of a web3 browser.

In simpler terms, it basically means, having Metamask’s extension/add-on installed in your browser, you’d have a global variable defined, called `ethereum` (`web3` for older versions) - using this variable we instantiate our web3 object.

**Step 2**

Now, in your client code, import the above file,

    import getWeb3 from '/path/to/web3';

and call the function:

    getWeb3().then((result) => {this.web3 = result;// we instantiate our contract next});

### 2. Set up account

Now to send transactions (specifically those that alter the state of the blockchain) we’ll need an account to sign those transactions from We instantiate our contract instance from the web3 object we created above:

    this.web3.eth.getAccounts().then((accounts) => {this.account = accounts[0];})

The `getAccounts()` function returns an array of all the accounts on user’s metamask, and `accounts[0]` is the one currently selected by the user.

### 3. Instantiate your contracts

Once we have our `web3` object in place, we’ll next instantiate our contracts > Assuming you have your contract ABI and address already in place :)

    const myContractInstance = new this.web3.eth.Contract(myContractAbi, myContractAddress)

### 4. Call functions

Now for any function you’d want to call from your contract, we directly interact with our instantiated contract object (which is `myContractInstance` declared in Step 2)

A quick review: - Functions that alter the state of the contract are called `send()` functions - Functions that do not alter the state of the contract are called `call()` functions

**Calling `call()` Functions**

    this.myContractInstance.methods.myMethod(myParams).call().then (// do stuff with returned values)

**Calling `send()` Functions**

    this.myContractInstance.methods.myMethod(myParams).send({from: this.account,gasPrice: 0}).then ((receipt) => {// returns a transaction receipt})

> For any transactions you make on Matic’s testnet, the gas price can be safely set to 0. :)

## Wallet Connect

Wallet Connect is an open protocol - not a wallet - built to create a communication link between DApps and Wallets. A wallet and an application supporting this protocol will enable a secure link through a shared key between the two peers. A connection is initiated by the DApp displaying a QR code with a standard WalletConnect URI and the connection is established when the wallet application approves the connection request. Further requests regarding funds transfer are confirmed on the wallet application itself.

### 1. Set up Web3

To set up your DApp to connect to user’s Matic Wallet we can use Wallet Connect’s provider to directly connect to Matic Network. Install the following in your DApp:

    npm install --save @maticnetwork/walletconnect-providernpm install --save maticjs

And add the following code in your App,

    import WalletConnectProvider from "@maticnetwork/walletconnect-provider"import Web3 from "web3"import Matic from "maticjs"

Next, we set up Matic and Ropsten provider via Wallet Connect’s object:

    const maticProvider = new WalletConnectProvider({host: `https://testnet2.matic.network`,callbacks: {onConnect: console.log('connected'),onDisconnect: console.log('disconnected!')}})const ropstenProvider = new WalletConnectProvider({host: `https://ropsten.infura.io/v3/70645f042c3a409599c60f96f6dd9fbc`,callbacks: {onConnect: console.log('connected'),onDisconnect: console.log('disconnected')}})

We created the above two provider objects to instantiate our Web3 object with:

    const maticWeb3 = new Web3(maticProvider)const ropstenWeb3 = new Web3(ropstenProvider)

### 2. Instantiating contracts

Once we have our web3 object, the instantiating of contracts involves the same steps we followed for metamask.

> Again, assuming you have your contract ABI and address already in place :)

    const myContractInstance = new this.maticWeb3.eth.Contract(myContractAbi, myContractAddress)

### 3. Calling functions

Like discussed above, we have two types of functions in Ethereum, depending upon the interaction with the blockchain. We `call()` when we read data and `send()` when we write data.

### Calling `call()` Functions

Now reading data doesn’t require a signature, therefore the process is the same as discussed above:

    this.myContractInstance.methods.myMethod(myParams).call().then (// do stuff with returned values)

### Calling `send()` Functions

Since writing to the blockchain requires a signature, we prompt the user on their wallet (that supports wallet connect) to sign the transaction. This involves two steps: 1. Constructing a transaction 2. Getting a signature on the transaction 3. Sending signed transaction

    const tx = {from: this.account,to: myContractAddress,gas: 800000,gasPrice: 0, // for matic testnetdata: this.myContractInstance.methods.myMethod(myParams).encodeABI(),gasPrice: "0x0",}

The above code creates a transaction object which is then sent to user’s wallet for signature:

    maticWeb3.eth.signTransaction(tx).then((result) =>{maticWeb3.eth.sendSignedTransaction(result).then((receipt) => console.log (receipt))})

`signTransaction()` function prompts the user for their signature and `sendSignedTransaction()` sends the signed transaction over (returns a transaction receipt on success).

> NOTE: all this while, the private key is in user’s wallet and the app does not access it any way. :)

## Portis

Portis is a web-based wallet built keeping easy user-onboarding in mind. It comes with a javascript SDK that integrates into the DApp and creates a local wallet-less experience for the user. Further, it handles setting up the wallet, transactions and gas fees. Like Metamask, it is non-custodial - users control their keys, Portis just stores them securely. But, unlike Metamask, it is integrated into the application and not the browser. Users have their keys associated with their login id and passwords.

**Type**: Non-custodial/HD **Private Key Storage**: Encrypted and stored on portis’ servers **Communication to Ethereum Ledger**: Developer defined **Private key encoding**: Mnemonic

### 1. Setup Web3

Install the following in your DApp:

    npm install --save @portis/web3

And register your DApp with Portis to obtain a Dapp ID: > [Portis Dashboard](https://dashboard.portis.io/)

Import `portis` and `web3` object:

    import Portis from '@portis/web3';import Web3 from 'web3';

Portis constructor takes first argument as the DApp ID (we got from the previous step) and second argument as the network you’d like to connect to. This can either be a string or an object.

    const portis = new Portis('YOUR_DAPP_ID', 'maticTestnet');const web3 = new Web3(portis.provider);

### 2. Set up account

If the installation and instantiation of web3 was successful, the following should successfully return the connected account:

    this.web3.eth.getAccounts().then((accounts) => {this.account = accounts[0];})

### 3. Instantiating Contracts

Instantiation of contracts would remain the same, as discussed above:

    const myContractInstance = new this.web3.eth.Contract(myContractAbi, myContractAddress)

### 4. Calling functions

Calling functions would remain the same as discussed above: #### Calling `call()` Functions

    this.myContractInstance.methods.myMethod(myParams).call().then (// do stuff with returned values)

### Calling `send()` Functions

    this.myContractInstance.methods.myMethod(myParams).send({from: this.account,gasPrice: 0}).then ((receipt) => {// returns a transaction receipt})

## Working Demo

For demonstration purposes the following example DApp was created to give a better understanding of the user’s perspective for the three approaches. Please refer this [repository](https://github.com/nglglhtr/key-management).

An example [ERC20 and ERC721 tokens](https://gist.github.com/nglglhtr/cf1686322449365e21eb9b32d0754939) were deployed on Matic chain, the DApp supports minting, transfer and checking balance/owner of tokens.

### Usage

1. Clone the repository
2. Install dependencies `bash npm install`
3. Checkout on respective branches for the separate approaches:
    - Metamask `bash git checkout master`
    - Wallet Connect `bash git checkout walletconnect`
    - Portis `bash git checkout portis`
4. `cd client && npm run serve`