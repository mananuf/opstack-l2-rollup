# 🚀 OP Stack Rollup Tutorial for Beginners

[![OP Stack Docs](https://img.shields.io/badge/Docs-OP%20Stack-blue?logo=ethereum&style=flat-square)](https://docs.optimism.io/operators/chain-operators/tutorials/create-l2-rollup)  
[![GitHub Repository](https://img.shields.io/badge/GitHub-OP%20Stack-171515?logo=github&style=flat-square)](https://github.com/ethereum-optimism/optimism.git)  
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)

## 📖 Overview

This **hands-on tutorial** is designed for developers who want to:  
✅ **Learn** how the OP Stack works by deploying a testnet chain.  
✅ **Understand** core components like the rollup node, batcher, and proposer.  
✅ **Build** a fully functional testnet Optimistic Rollup from scratch.

By the end, you'll have:  
🔹 **Your own OP Stack testnet**  
🔹 **Knowledge of L2 architecture**  
🔹 **A foundation for production deployments**

---

## 📗 Table of Contents

- [📖 Overview](#-overview)
- [📂 Setting Up Project Folder](#-setting-up-project-folder)
- [⚙️ Configure your network](#️-configure-your-network)
- [🚀 Op-deployer](#-op-deployer)
- [💻 Op-geth](#-op-geth)
- [📄 License](#-license)

## 📂 Setting Up Project Folder

### Clone Optimism Repo

```
cd ~/Documents
git clone https://github.com/ethereum-optimism/optimism.git
```

### Change Directory to optimism/

```
cd ~/optimism
```

### Clear/remove existing git settings

```
rm -rf .git
```

### Check Required Dependencies

```
./packages/contracts-bedrock/scripts/getting-started/versions.sh
```

```
Dependency                               | Minimum         | Actual
git (https://git-scm.com)                  2                2.45.2
go (https://go.dev)                        1.21             1.22.7
node (https://nodejs.org)                  20               23.5.0
foundry (https://getfoundry.sh)            0.2.0 (a5efe4f)  No version, commit hash, and timestamp found.
make (https://www.gnu.org/software/make)   3                3.81
jq (https://jqlang.github.io/jq)           1.6              1.6
direnv (https://direnv.net/)               2                2.35.0
just (https://github.com/casey/just)       1.34.0           1.40.0
```

### Install pnpm

```
pnpm install
```

### Duplicate the sample environment variable file

```
cp .envrc.example .envrc
```

Open up the environment variable file and fill out the following variables:

- L1_RPC_URL URL for your L1 node (a Sepolia node in this case).
- L1_RPC_KIND Kind of L1 RPC you're connecting to, used to inform optimal transactions receipts fetching. Valid options: alchemy, quicknode, infura, parity, nethermind, debug_geth, erigon, basic, any.

### Generate addresses

You'll need four addresses and their private keys when setting up the chain:

- The Admin address has the ability to upgrade contracts.
- The Batcher address publishes Sequencer transaction data to L1.
- The Proposer address publishes L2 transaction results (state roots) to L1.
- The Sequencer address signs blocks on the p2p network.

run

```
./packages/contracts-bedrock/scripts/getting-started/wallets.sh
```

```
Copy the following into your .envrc file:

# Admin address
export GS_ADMIN_ADDRESS=0x9625B9aF7C42b4Ab7f2C437dbc4ee749d52E19FC
export GS_ADMIN_PRIVATE_KEY=0xbb93a75f64c57c6f464fd259ea37c2d4694110df57b2e293db8226a502b30a34

# Batcher address
export GS_BATCHER_ADDRESS=0xa1AEF4C07AB21E39c37F05466b872094edcf9cB1
export GS_BATCHER_PRIVATE_KEY=0xe4d9cd91a3e53853b7ea0dad275efdb5173666720b1100866fb2d89757ca9c5a

# Proposer address
export GS_PROPOSER_ADDRESS=0x40E805e252D0Ee3D587b68736544dEfB419F351b
export GS_PROPOSER_PRIVATE_KEY=0x2d1f265683ebe37d960c67df03a378f79a7859038c6d634a61e40776d561f8a2

# Sequencer address
export GS_SEQUENCER_ADDRESS=0xC06566E8Ec6cF81B4B26376880dB620d83d50Dfb
export GS_SEQUENCER_PRIVATE_KEY=0x2a0290473f3838dbd083a5e17783e3cc33c905539c0121f9c76614dda8a38dca
```

Copy the output from the previous step and paste it into your .envrc file as directed.

### Fund Addressess

You will need to send ETH to the Admin, Proposer, and Batcher addresses. The exact amount of ETH required depends on the L1 network being used. You do not need to send any ETH to the Sequencer address as it does not send transactions.

It's recommended to fund the addresses with the following amounts when using Sepolia:

- Admin — 0.5 Sepolia ETH
- Batcher — 0.1 Sepolia ETH
- Proposer — 0.2 Sepolia ETH

### Load the variables with direnv

Next you'll need to allow direnv to read this file and load the variables into your terminal using the following command.

```
direnv allow
```

After running direnv allow you should see output that looks something like the following (the exact output will vary depending on the variables you've set, don't worry if it doesn't look exactly like this):

```
bankat@Bankats-MacBook-Pro-2 optimism % direnv allow
direnv: loading ~/Documents/rollups/optimism/.envrc
direnv: export +BEACON_URL +DEPLOYMENT_CONTEXT +ETHERSCAN_API_KEY +GS_ADMIN_ADDRESS +GS_ADMIN_PRIVATE_KEY +GS_BATCHER_ADDRESS +GS_BATCHER_PRIVATE_KEY +GS_PROPOSER_ADDRESS +GS_PROPOSER_PRIVATE_KEY +GS_SEQUENCER_ADDRESS +GS_SEQUENCER_PRIVATE_KEY +IMPL_SALT +L1_RPC_KIND +L1_RPC_URL +PRIVATE_KEY +TENDERLY_PROJECT +TENDERLY_USERNAME

```

if you dont see any output, try setting up direnv following the docs: https://direnv.net/docs/hook.html

## ⚙️ Configure your network

Once you've built your repository, you'll need to set up the configuration file for your chain. Currently, chain configuration lives inside of the contracts-bedrock package in the form of a JSON file.

### Move into the contracts-bedrock package

```
cd optimism/packages/contracts-bedrock
```

### Install Foundry dependencies

```
forge install
```

### Generate the configuration file

Run the following script to generate the getting-started.json configuration file inside of the deploy-config directory

```
./scripts/getting-started/config.sh
```

### Check if the factory exists

If you're deploying an OP Stack chain to a network other than Sepolia, you may need to deploy a Create2 factory contract to the L1 chain. This factory contract is used to deploy OP Stack smart contracts in a deterministic fashion.

The Create2 factory contract will be deployed at the address 0x4e59b44847b379578588920cA78FbF26c0B4956C. You can check if this contract has already been deployed to your L1 network with a block explorer or by running the following command:

```
cast codesize 0x4e59b44847b379578588920cA78FbF26c0B4956C --rpc-url $L1_RPC_URL
```

If the command returns `0` then the contract has not been deployed yet. If the command returns `69` then the contract has been deployed and you can safely skip this section.

**⚠️warning**
I ran into failed builds for `op-deployer`, `op-node`, `op-batcher`, `op-proposer` in the first optimism repo cloned (i'm guessing WIP, cos there are still ongoing merges and prs, mostlikey cause od the break). But i was able to fix by using a different optimisim git repo. we are about to clone a new optimism folder and we will refer to it as
`optimism2`

Dependency needed: intsall `just` a command runner
linux: https://snapcraft.io/just
mac: `brew install just`


## Cloning Optimism2 for stable build of `op-deployer`, `op-node`, `op-batcher`, `op-proposer`

```
git clone https://github.com/ethereum-optimism/optimism.git

// rename to optimism2
```

## 🚀 Op-Deployer

The op-deployer tool simplifies the creation of genesis and rollup configuration files (genesis.json and rollup.json). These files are crucial for initializing the execution client (op-geth) and consensus client (op-node) for your network.

**ℹ️ INFO**  
 you'll need to have sufficient gas to do this. most preferrable when there is low network congestion.

```
forge script scripts/Deploy.s.sol:Deploy --private-key $GS_ADMIN_PRIVATE_KEY --broadcast --rpc-url $L1_RPC_URL --slow
```

### move into op-deployer dir

```
cd ~/Documents/optimism2/op-deployer // or wherever your optimism folder is located
```

### build op-deployer

```
go build -o ~/op-deployer/bin/op-deployer ./cmd/op-deployer
```

### grant required permissions

```
chmod +x ~/op-deployer/bin/op-deployer
```

**ℹ️ INFO**  
the op-deployer gets built to tour home `~/` directory.


```
cd ~/op-deployer
./bin/op-deployer init --l1-chain-id 11155111 --l2-chain-ids 42069 --workdir .deployer
```

this generates a .deployer/ that has a config file named intent.toml
open the file in your code editor

```
code ~/op-deployer/.deployer/intent.toml  // for those using vs-code
```

modify the config to look like this.
_(replace all the env variables with the actual values from your .envrc)_

```
configType = "standard-overrides"
l1ChainID = 11155111
fundDevAccounts = true // make this true
useInterop = false
l1ContractsLocator = "tag://op-contracts/v3.0.0-rc.2"
l2ContractsLocator = "tag://op-contracts/v3.0.0-rc.2"

// do not alter the superchain roles, they are system specific
[superchainRoles]
  proxyAdminOwner = "0x1eb2ffc903729a0f03966b917003800b145f56e2"
  protocolVersionsOwner = "0xfd1d2e729ae8eee2e146c033bf4400fe75284301"
  guardian = "0x7a50f00e8d05b95f98fe38d8bee366a7324dcf7e"

[[chains]]
  id = "0x000000000000000000000000000000000000000000000000000000000000a455"
  baseFeeVaultRecipient = "<your wallet address>"
  l1FeeVaultRecipient = "<your wallet address>"
  sequencerFeeVaultRecipient = "<your wallet address>"
  eip1559DenominatorCanyon = 250
  eip1559Denominator = 50
  eip1559Elasticity = 6
  operatorFeeScalar = 0
  operatorFeeConstant = 0
  [chains.roles]
    l1ProxyAdminOwner = "$GS_ADMIN_ADDRESS"
    l2ProxyAdminOwner = "$GS_ADMIN_ADDRESS"
    systemConfigOwner = "$GS_ADMIN_ADDRESS"
    unsafeBlockSigner = "$GS_SEQUENCER_ADDRESS"
    batcher = "$GS_BATCHER_ADDRESS"
    proposer = "$GS_PROPOSER_ADDRESS"
    challenger = "$GS_ADMIN_ADDRESS"
```

### Generate genesis.json and rollup.json

```
./bin/op-deployer inspect genesis --workdir .deployer 42069 > .deployer/genesis.json
./bin/op-deployer inspect rollup --workdir .deployer 42069 > .deployer/rollup.json
```

## 💻 Op-geth

change dir to ~/Documents or any dir of your choice and clone op-geth

```
git clone https://github.com/ethereum-optimism/op-geth.git

cd op-geth/

make geth

build/bin/geth init --state.scheme=hash --datadir=datadir ~/op-deployer/.deployer/genesis.json
```

### start the op-geth

```
./build/bin/geth \
  --datadir ./datadir \
  --http \
  --http.corsdomain="*" \
  --http.vhosts="*" \
  --http.addr=0.0.0.0 \
  --http.api=web3,debug,eth,txpool,net,engine \
  --ws \
  --ws.addr=0.0.0.0 \
  --ws.port=8546 \
  --ws.origins="*" \
  --ws.api=debug,eth,txpool,net,engine \
  --syncmode=full \
  --gcmode=archive \
  --nodiscover \
  --maxpeers=0 \
  --networkid=42069 \
  --authrpc.vhosts="*" \
  --authrpc.addr=0.0.0.0 \
  --authrpc.port=8551 \
  --authrpc.jwtsecret=./jwt.txt \
  --rollup.disabletxpoolgossip=true
```

your op-geth is now up and running. Hurray!🎉

## Op-node

follow the commands one by one to run a successful build for op-node.

```
cd ~/Documents/optimism2/op-node

just op-node ⁠// this command builds the go program ⁠

// to start op-node run:
----------------------------------

./bin/op-node \
  --rollup.config=./rollup.json \
  --l1=<your $L1_RPC_URL> \
  --l1.beacon=https://ethereum-sepolia-beacon-api.publicnode.com \
  --l2=ws://localhost:8546 \
  --l2.jwt-secret=./jwt-secret.txt \
  --syncmode=execution-layer \
  --p2p.priv.path=opnode_p2p_priv.txt \
  --p2p.peerstore.path=opnode_peerstore_db
```

## Op-batcher

follow the commands one by one to run a successful build for op-batcher.

```
cd optimism2/op-batcher/

//⁠ this command builds the go program ⁠
just op-batcher ⁠

//then start op-batcher using:
./bin/op-batcher \
  --l2-eth-rpc=http://localhost:8545 \
  --rollup-rpc=http://localhost:9545 \
  --poll-interval=1s \
  --sub-safety-margin=6 \
  --num-confirmations=1 \
  --safe-abort-nonce-too-low-count=3 \
  --resubmission-timeout=30s \
  --rpc.addr=0.0.0.0 \
  --rpc.port=8548 \
  --rpc.enable-admin \
  --max-channel-duration=25 \
  --l1-eth-rpc=<your $l1_RPC_URL> \
  --private-key=<your $BATCHER_PRIVATE_KEY>
```

## Op-proposer

follow the commands one by one to run a successful build for op-proposer.

```
cd optimism2/op-proposer/

//⁠ this command builds the go program ⁠

just op-proposer ⁠

//then start op-proposer using:

./bin/op-proposer \
  --poll-interval=12s \
  --rpc.port=8560 \
  --rollup-rpc=http://localhost:9545 \
  --l2oo-address=$(cat ~/Documents/optimism/packages/contracts-bedrock/deployments/getting-started/.deploy | jq -r .L2OutputOracleProxy) \
  --private-key=<your $GS_PROPOSER_PRIVATE_KEY> \
  --l1-eth-rpc=<your $L1_RPC_UR>L

```

Note:

```
~/Documents/optimism/packages/contracts-bedrock/deployments/getting-started/.deploy

// _this should be path to your .deploy file you created in the first optimism/ folder and replace above_
```

## Connect Your wallet to your chain

To connect your wallet to your chain, Open a browser where you have a wallet and open this url to configure
[wallect connection link](https://chainid.link/?network=opstack-getting-started)

## Get ETH on your chain

Once you've connected your wallet, you'll probably notice that you don't have any ETH to pay for gas on your chain. The easiest way to deposit Sepolia ETH into your chain is to send ETH directly to the L1StandardBridge contract.

NB: navigate to your first optimism folder we cloned.

```
cd ~/optimism/packages/contracts-bedrock
```

to get contract address:

```
cat deployments/getting-started/.deploy | jq -r .L1StandardBridgeProxy
```

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
