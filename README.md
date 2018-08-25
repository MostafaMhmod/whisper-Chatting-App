## Running the APP

The App assumes that there is a running geth client node with an RPC at URL `http://localhost:8545`. 
#### To install geth refer to the go-ethereum documentation 
[Ethereum Geth installation guide.](https://github.com/ethereum/go-ethereum/wiki/Installing-Geth)

First, Start geth on the testnet with enabling the Whisper protocol and the RPC connection by using this command.

(This should take around an hour to Synchronize the whole rinkeby testnet).

    geth --syncmode "light" --shh --rpc --rinkeby --rpccorsdomain "http://localhost:8080"

Then make sure to create an account an to send some ether to it by using any supported online wallet like metamask,MEW, etc. 
you can get some Ether using the Rinkeby testnet [From here](https://faucet.rinkeby.io/)

Then, clone this repository and download the dependencies:

    git clone https://github.com/MostafaMhmod/whisper-Chatting-App


    npm install

Finally, start the example with:

    npm start

The example should be started and the application will be available at `http://localhost:8080`.
