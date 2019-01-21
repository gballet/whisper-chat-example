# whisper-chat-example

This demonstrates how to use the Ethereum Whisper v6 API. It will also work with v5, but not between versions.

## Running the example

The example assumes that there is a running Whisper v6 node exposing an RPC interface at URL `http://localhost:8545`. For this, you can use `geth` with the folloing parameters:

    $ geth <usual p2p flags> --shh --rpc --rpccorsdomain '*'

`--shh` is the option that enables Whisper for the node.

`--rpc` enables the HTTP RPC interface and `--rplccorsdomain '*'` will disable this annoying CORS verification in the browser. Needless to say, this is only acceptable because this is an example.

Then, clone this repository and download the dependencies:

    $ npm install

Finally, start the example with:

    $ npm run dev

The example should be started and the application will be available at `http://localhost:8080`.
