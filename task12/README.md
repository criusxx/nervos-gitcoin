
## Document Porting An Existing Ethereum DApp To Polyjuice
### **Step 1 —  Setup the Godwoken Testnet Network in MetaMask**

First of all, you have to have a metamask extension wallet in your browser.
It is generally recommended that Google Chrome can directly support it.
Create a wallet or import an existing eth wallet. I believe that this metamask plugin wallet has been used in the previous tasks.
This step is mainly to Add New RPC Network to MetaMask：


    Network Name: Godwoken Test Network
    New RPC URL: https://godwoken-testnet-web3-rpc.ckbapp.dev/
    Chain ID: 71393
    Currency Symbol (optional): N/A
    Block Explorer URL (optional): N/A

You can now interact with a Polyjuice-powered DApp, but first you have to build one.

### **Step 2 — Clone Your Existing DApp**

Open your local server side and enter the command：


    cd ~/projects
    git@github.com:criusxx/blockchain-upload-image.git
    cd blockchain-upload-image
    yarn
    yarn build
    yarn start:ganache
    


Open another command output window under the current path,enter the command:


    yarn ui
  

now open a browser tab to http://localhost:3000 to view the dApp UI!


### **Step 3 — Install  Dependencies**

There are two dependencies you need to add to your existing web3 Dapp to get it working with the Layer 2 solution Polyjuice.

The first, @polyjuice-provider/web3, is a custom Polyjuice web3 provider to use instead of the web3 library. It is required for interaction with Nervos Layer 2 smart contracts.

The second, nervos-godwoken-integration, is a tool that can generate a Polyjuice address based on your Ethereum address. You might be required to use it if you store values mapped to addresses in any of your smart contracts.

Install both using the follow commands:

    cd ~/projects/blockchain-upload-image
    yarn add @polyjuice-provider/web3@0.0.1-rc7 nervos-godwoken-integration@0.0.6

 Now, you can properly configure your existing DApp for Polyjuice.
 
### Step 4 — Edit **Configuration**

Next, create a new configuration file and input the following values for it:


    export const CONFIG = {
    WEB3_PROVIDER_URL: 'https://godwoken-testnet-web3-rpc.ckbapp.dev'
    ROLLUP_TYPE_HASH: '0x4cc2e6526204ae6a2e8fcf12f7ad472f41a1606d5b9624beebd215d780809f6a'
    ETH_ACCOUNT_LOCK_CODE_HASH: '0xdeec13a7b8e100579541384ccaf4b5223733e4a5483c3aec95ddc4c1d5ea5b22'
}


The web3 provider URL should be configured to the same URL you used for the RPC network added to MetaMask.

The rollup_type_hash and eth_account_lock are values that are also required for the configuration file.

### **Step 5 — Import and Replace our dapp Dependencies**

 We will update the main UI in the file,

     cd ~/projects/blockchain-upload-image/src/ui
    vi app.tsx
 
 and input the following values for it:


    import { PolyjuiceHttpProvider } from '@polyjuice-provider/web3';

    import { AddressTranslator } from 'nervos-godwoken-integration';

    import { CONFIG } from '../config';
    
  now we need to add PolyjuiceHttpProvider to our DApp to utilize it for our web3 instance instead of web3.js.

    const godwokenRpcUrl = CONFIG.WEB3_PROVIDER_URL;
    const providerConfig = {
    rollupTypeHash: CONFIG.ROLLUP_TYPE_HASH,
    ethAccountLockCodeHash: CONFIG.ETH_ACCOUNT_LOCK_CODE_HASH,
    web3Url: godwokenRpcUrl
    };
    const provider = new PolyjuiceHttpProvider(godwokenRpcUrl, providerConfig);
    const web3 = new Web3(provider);
    

With the above code, we have replaced our old Ethereum web3 instance with our new, shiny Polyjuice web3 instance.

At the completion of this step, your DApp is now capable of communicating with Polyjuice!


### **Step 6 — Set Higher Gas Limit**

 Godwoken requires a higher-than-usual gas limit to be set for transactions on the test network (testnet). This may be subject to change in the future.
 
 Open the file and update this file,

     cd ~/projects/blockchain-upload-image/src/lib/contracts
    vi SimpleStorageWrapper.ts  
    const DEFAULT_SEND_OPTIONS = {
    gas: 6000000
    };
 
You create a simple object that changes the gas property used by the MetaMask extension to modify objects passed through the send() function.

The modified version of this file in my dapp is:

     this.contract.methods.awardItem(fromAddress, imgUrl).send({
            ...DEFAULT_SEND_OPTIONS,
            from: fromAddress
        });


### **Step 7 — Change Polyjuice Address Display in DApp's UI**

Each Ethernet address can be translated into a Polyjuice address at layer 2 of Nervos We will use the nervos-godwoken-integration  AddressTranslator class to derive a Polyjuice address from an Ethernet address:

     const addressTranslator = new AddressTranslator();

    const polyjuiceAddress = addressTranslator.ethAddressToGodwokenShortAddress(ethereumAddress);

now, you can implement this Polyjuice Address in the user interface (UI) of your DApp.


### **Step 8 — View the Completed Your Updated DApp**

now we have a working Polyjuice DApp, that can run in a browser and is interactable with the MetaMask Wallet extension as soon as you switch its' network to the Godwoken Test network ,Refer to our second step to start the dapp ui

finally enter the URL in your browser to view http://localhost:3000/
