## Running the APP

The example assumes that there is a running Whisper v5 node exposing an RPC interface at URL `http://localhost:8545`. For this, you can use `geth` with the folloing parameters:

    $ geth <usual p2p flags> --shh --rpc --rpccorsdomain '*'


Then, clone this repository and download the dependencies:

    $ npm install

Finally, start the example with:

    $ npm run dev

The example should be started and the application will be available at `http://localhost:8080`.
