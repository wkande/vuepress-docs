---
title: Airnode Starter
---

::: tip
 Check out the project source code here [https://github.com/api3dao/airnode-starter](https://github.com/api3dao/airnode-starter)
:::

This project is composed of two steps:

1. Deploy an Airnode on Ropsten
2. Make a request to the deployed Airnode in a contract

You can skip the first step and use the Airnode that we have deployed as well. You are recommended to read the contents of the scripts as you run them, and read the entire readme before starting.

## Setup

First, you need to create a Ropsten wallet with some ETH in it.

1. Clone this repo
2. Run the following to install the dependencies

   ```bash
   npm install
   ```

3. Run the following to build the contracts

   ```bash
   npm run build
   ```

4. Run the following to generate a wallet, whose mnemonic phrase will be displayed on the terminal and recorded in a `.env` file at the project root.

   ```bash
   npm run generate-wallet
   ```

5. Install [Metamask](https://metamask.io/) to your web browser
6. Import the mnemonic phrase to Metamask
7. Use the [faucet](https://faucet.metamask.io/) to get some Ropsten ETH

Then, you need to get a Ropsten provider URL. This will be used both by the deployed Airnode and by you while interacting with contracts. 1. Go to [Infura](https://infura.io/), create an account and get a Ropsten provider URL 2. Replace `https://ropsten.infura.io/v3/{YOUR_KEY}` in your `.env` file with the URL you got from Infura

Note that you can use any other provider or your own Ropsten node. However, if you will be deploying your own Airnode, the provider endpoint must be publicly accessible \(i.e., `127.0.0.1:8545` will not work\).

_\(You only need cloud credentials if you will not be skipping Step 1.\)_

Follow the [docs](https://github.com/api3dao/api3-docs/blob/master/provider-guides/deploying-airnode.md#creating-cloud-credentials) to create your cloud credentials. Place them at `/config/.env`, similar to [`/config/example.env`](/config/example.env). Do not confuse this `.env` file with the one in the project root that keeps your mnemonic phrase and provider URL.

## Step 1: Deploy an Airnode on Ropsten

Normally, you would need to do two things before you deploy an Airnode: 1. [Specify the API integration](https://github.com/api3dao/api3-docs/blob/master/provider-guides/api-integration.md) 1. [Configure your Airnode](https://github.com/api3dao/api3-docs/blob/master/provider-guides/configuring-airnode.md)

For this project, we specified a minimal integration to the popular and free [CoinGecko API](https://www.coingecko.com/en/api), and prepared the configuration files to serve it over the Ropsten testnet. We only integrated a single API operation, `GET` for `/coins/{id}`, which you can see below. The `localization`, `tickers`, `community_data`, `developer_data` and `sparkline` parameters are [fixed](https://github.com/api3dao/api3-docs/blob/master/provider-guides/api-integration.md#fixedoperationparameters) as `"false"`, while `market_data` is fixed as `"true"`. The `id` parameter will be provided by the requester \(e.g., `"ethereum"`\) under the name `coinId`. You can make test calls over the [CoinGecko API docs](https://www.coingecko.com/en/api) to see the response format.

![](https://user-images.githubusercontent.com/19530665/103151070-be14ea00-478b-11eb-9608-a967c4282d9f.png)

See [`config.example.json`](/config/config.example.json) for how this integration is achieved. We fixed the [reserved parameters](https://github.com/api3dao/api3-docs/blob/master/provider-guides/api-integration.md#reservedparameters) to read the value from `market_data.current_price.usd`, cast it as an `int256` and multiply it by `1,000,000` before returning. No security scheme \(i.e., API key\) is defined in `config.json` or [`security.json`](/config/security.json) because the CoinGecko API is publicly accessible.

### Customize your `config.json`

Run the following to insert the contents of `.env` to `config/config.example.json` and save it as `config/config.json`

```bash
npm run customize-config
```

### Deploy

Now your `/config` directory should have the required [`config.json`](https://github.com/api3dao/api3-docs/blob/master/airnode/config-json.md), [`security.json`](https://github.com/api3dao/api3-docs/blob/master/airnode/security-json.md) and [`.env`](https://github.com/api3dao/api3-docs/blob/master/provider-guides/deploying-airnode.md#creating-cloud-credentials) files. Run the following to deploy your node:

```bash
cd config
# The deployer has to be run in the directory where the configuration files are
docker run -it --rm \
  --env-file .env \
  --env COMMAND=deploy-first-time \
  -v $(pwd):/airnode/out \
  api3/airnode:latest
```

This will output a receipt file with the extension `.receipt.json`.

### Fund your master wallet

Run the following to send your master wallet 0.1 ETH for it to [create a provider record](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/provider.md#creating-a-provider-record) for you on-chain.

```bash
npm run fund-master-wallet
```

Your deployed Airnode will use these funds to make the transaction that will create the provider record on Ropsten, and send the leftover ETH back to your address automatically.

### Make your endpoint publicly accessible

`config.json` defines an [endpoint](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/endpoint.md) named `coinMarketData`, whose [`endpointId`](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/endpoint.md#endpointid) is `0xf466b8feec41e9e50815e0c9dca4db1ff959637e564bb13fefa99e9f9f90453c`. Endpoints are not publicly accessible by default, so you will have to make a transaction for this. Run the following to set your endpoint's [authorizers](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/authorizer.md) to `[0x0000000000000000000000000000000000000000]`, which makes it [publicly accessible](https://github.com/api3dao/api3-docs/blob/master/provider-guides/setting-authorizers.md#allow-all):

```bash
npm run update-authorizers
```

## Step 2: Make a request

The scripts in this step will use the Airnode you have deployed if you have completed Step 1. Otherwise, it will use the `providerId` of the Airnode that we have deployed given in [`parameters.js`](/src/parameters.js). Note that the `endpointId` will be the same either way because it is [derived from the OIS and endpoint name](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/endpoint.md#endpointid).

### Create a requester

Run the following to create an on-chain [requester](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/requester.md) record:

```bash
npm run create-requester
```

You can use this requester denoted with an index in other projects as well. Note that `requesterIndex` is chain-specific, so you will have to create another requester record on other chains.

### Deploy the client contract

Run the following to deploy `ExampleClient.sol`:

```bash
npm run deploy-client
```

### Endorse the client

Run the following to [endorse](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/endorsement.md) your deployed [client](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/client.md) contract using the requester you have created:

```bash
npm run endorse-client
```

### Derive and fund the designated wallet

First run the following to derive the [designated wallet](https://github.com/api3dao/api3-docs/blob/master/request-response-protocol/designated-wallet.md) for the provider–requester pair:

```bash
npm run derive-designated-wallet-address
```

and then fund this designated wallet with 0.1 ETH:

```bash
npm run fund-designated-wallet
```

The requests that the client contract will make will be funded by this 0.1 ETH. Note that you may have to run `fund-designated-wallet` again if you make too many requests and use up this 0.1 ETH \(unlikely on Ropsten because the gas price is low\).

### Make a request

Run the following to make a request:

```text
npm run make-request
```

which should be fulfilled by the Airnode and printed out on the terminal. Note that now that the price is on-chain, you can use it in your contract to implement any arbitrary logic.

Try replacing the `coinId` value in [`make-request.js`](/scripts/make-request.js) from `"ethereum"` to `"bitcoin"` and make another request. You can see the API docs to see which coin IDs are supported.

## Conclusion

You deployed an Airnode, made a request to it and received the response at the contract. If you want to learn more, see the following resources:

* [API3 whitepaper](https://github.com/api3dao/api3-whitepaper) will give you a broad overview of the project
* [Medium posts](https://github.com/api3dao/api3-docs/blob/master/medium.md) are a more digestible version of the whitepaper
* [API3 docs](https://github.com/api3dao/api3-docs) will provide you with the theory of how Airnode and its protocol works
* [`airnode-admin`](https://github.com/api3dao/airnode-admin) lets you interact with the Airnode contract \(to create a request, endorse a client, etc.\) using a CLI tool
* [Airnode client examples](https://github.com/api3dao/airnode-client-examples) demonstrate different request patterns that the Airnode protocol supports \(for example, we used a full request in this starter project\)
