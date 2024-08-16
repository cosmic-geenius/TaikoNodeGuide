## Clone simple-taiko-node

```bash
git clone https://github.com/taikoxyz/simple-taiko-node.git

cd simple-taiko-node
```

## Copy the sample `.env` files

> [!NOTE]
> To run a `hekla` Taiko node please copy `.env.sample.hekla` instead of `.env.sample`.

```bash
cp .env.sample .env
```

## Set the L1 node endpoint

First, open the `.env` in your preferred text editor:

```bash
nano .env
```

> [!NOTE]
> You can use any L1 endpoint to run a Taiko L1 node unless you want to prove blocks past the last 128 blocks, in which case you will need an archive node.
>
> It’s recommended to [run a local L1 node](https://docs.taiko.xyz/guides/node-operators/run-an-ethereum-testnet-node) but you browse around for other RPC Providers. Keep in mind they will **eventually rate limit your node** and it will stop syncing, so a local L1 node is recommended for a proper setup.
>
> For Mainnet, the L1 Node will need to be an Ethereum node; for Testnet, the L1 Node will need to be a Etherem testnet (Holesky) node.

Next, you will set the L1 node endpoints. If you are running a local L1 node, you cannot reference the L1 endpoints  as `http://127.0.0.1:8545`,  `ws://127.0.0.1:8546`  and `http://127.0.0.1:5052`  because that is local to inside the simple-taiko-node Docker networking. Instead you can try:

- Using `host.docker.internal` (see: [stack overflow](https://stackoverflow.com/questions/24319662)).
- Using the private ip address of your machine (use something like `ip addr show` to get this ip address).

After getting the address of the L1 node, set the following L1 node endpoints in your `.env` file. Here is an example:

- `L1_ENDPOINT_HTTP=http://192.168.1.15:8545`
- `L1_ENDPOINT_WS=ws://192.168.1.15:8546`
- `L1_BEACON_HTTP=http://192.168.1.15:5052`

## Remove old testnet volumes

If you ran a testnet node previously, make sure to first remove the old volumes:

```bash
docker compose down -v
```

## Set the profiles you want to run

In your `.env`, please set `COMPOSE_PROFILES` to whichever set of services you’d like to run in a comma separated list (i.e., `l2_execution_engine,proposer,prover` or any combination of the three).

Note that if you include `proposer` or `prover`, the services will still be disabled if you do not set `ENABLE_PROPOSER=true` or `ENABLE_PROVER=true` respectively.

## Start the node

> [!NOTE]
> Make sure Docker is running and then run the following command to start the node (you may need to use `sudo docker compose up -d` if your user is not in the `docker` group):
>
> To run a `hekla` Taiko node please use the following command instead: `docker compose -f docker-compose-hekla.yml up -d`.


```bash
docker compose up -d
```

## Verify node is running

### **Option 1: Check with the node dashboard**

A node dashboard will be running on `localhost` on the `GRAFANA_PORT` you set in your `.env` file, which defaults to `3001`:  http://localhost:3001/d/L2ExecutionEngine/l2-execution-engine-overview.

You can verify that your node is syncing by checking that the **chain head** on the dashboard is increasing. Once the chain head matches what’s on the block explorer, you are fully synced.

### **Option 2: Check with curl commands**

1. Check if the Execution Layer client is connected to Taiko L2:
    
    ```bash
    curl http://localhost:8547 \
      -X POST \
      -H "Content-Type: application/json" \
      --data '{"method":"eth_chainId","params":[],"id":1,"jsonrpc":"2.0"}'
    ```
    
    which should return the chainId as `0x28c61` (167009):
    
    ```json
    { "jsonrpc": "2.0", "id": 1, "result": "0x28c61" }
    ```
    
2. Check if the Execution Layer client is synced by requesting the latest Taiko L2 / L3 block from the Execution Layer client:
    
    ```bash
    curl http://localhost:8547 \
      -X POST \
      -H "Content-Type: application/json" \
      --data '{"method":"eth_chainId","params":[],"id":1,"jsonrpc":"2.0"}'
    ```
    
3. If the blockNumber response value is `0` or not growing, check the Taiko L2 logs here:
    
    ```bash
    docker compose logs -f
    ```
    
    > Note: You may need to use sudo docker compose logs -f if you are not in the docker group.
