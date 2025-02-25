# Ethereum Batch Deposit

This repository contains the smart contract code for batch deposits into native staking on Ethereum.
The contracts are organized in the [`contracts`](/contracts) directory. [`interfaces`](/contracts/interfaces) implement external interfaces we depend on, [`lib`](/contracts/lib) contains our actual contracts and [`test`](/contracts/test) the mocked contracts for testing.

The following table lists the networks and respective addresses that the BatchDeposit contract has been deployed to:

| Network | Chain Id | Address                                                                                                                       |
| ------- | -------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Mainnet | 1        | [0x8fC32441C13706bB981A506FdE65DeDD5dEF3981](https://etherscan.io/address/0x8fc32441c13706bb981a506fde65dedd5def3981)         |
| Holesky | 17000    | [0xf079e540F598a85C22dA6eb570236B2169bE4962](https://holesky.etherscan.io/address/0xf079e540f598a85c22da6eb570236b2169be4962) |

## Contracts

### BatchDeposit

The BatchDeposit contract enables the deployment of multiple Ethereum validators at once. The `BatchDeposit.sol` contract interacts with the native Ethereum staking deposit contract. For more information, see `contracts/lib/BatchDeposit.sol`.

Credits also go to [Justfarming](https://www.justfarming.xyz), [stakefish](https://www.stake.fish), and [abyss](https://www.abyss.finance) who have built their batch depositors in the open:

- [Justfarming BatchDeposit & StakingRewards](https://github.com/justfarming/contracts)
- [stakefish BatchDeposits contract (GitHub)](https://github.com/stakefish/eth2-batch-deposit/blob/main/contracts/BatchDeposits.sol)
- [Abyss Eth2 Depositor contract (GitHub)](https://github.com/abyssfinance/abyss-eth2depositor/blob/main/contracts/AbyssEth2Depositor.sol)

## Design Decisions

### The Ethereum Deposit Process

Ethereum staking deposits, i.e., `depositdata`, generated with the official [`Ethereum staking-deposit-cli`](https://github.com/ethereum/staking-deposit-cli) are flexible regarding the per-validator withdrawal credential. This implementation of batch deposits assume, and require, all validators of a batch to deposit for the same withdrawal address. This design-decision is in line with two observation-based assumptions:

1. Batches of validators are usually deployed on behalf of a single entity that usually wants all validators of one batch to be associated to the same withdrawal address.
2. People want confidence when depositing to validators: Verifying Ethereum staking deposits on hardware wallets is difficult. The stakes are high but confidence when reviewing a transaction of 100 Validator deposits with 100 different withdrawal credentials hidden in _a lot_ of hex data on a small screen.

To the latter point: This implementation helps by presenting the withdrawal address (not credential) as the first parameter of the contract, re-enabling a human to conduct the verification of the batch deposit transaction.

## Setup

The Justfaring contracts project uses the [Truffle Suite](https://trufflesuite.com/) and [Node.js with `npm`](https://nodejs.org/en) to manage its dependencies.

With `node`, you can proceed with installing the project-specific dependencies:

```shell
npm install
```

For static analysis, [Slither](https://github.com/crytic/slither) is used to perform automated security analysis on the smart contracts.

```shell
pip3 install slither-analyzer
```

We use `solc-select` for managing the active solidity version.

```shell
pip3 install solc-select
solc-select install 0.8.24
solc-select use 0.8.24
```

## Developing

### Compiling

To compile the contracts for use in web applications using [`wagmi`](https://wagmi.sh), you can run:

```shell
npm run compile
npx hardhat generate-typescript-abi
```

### Linting

You can lint `.js` and `.ts` files with

```shell
npm run lint:ts
```

as well as the `.sol` files with

```shell
npm run lint:sol
```

Most issues can be fixed with

```shell
npm run lint:ts -- --fix
npm run fmt
```

### Analysis

To run the analyzer:

```shell
npm run analyze
```

### Testing

To run the tests, you can run:

```shell
npm run test
```

#### Generate test-coverage

To generate test coverage information, you can run:

```shell
npm run test:coverage
```

#### Manual testing

Manual tests can be conducted using [`ethdo`](https://github.com/wealdtech/ethdo), which can be installed using `go`:

```shell
go install github.com/wealdtech/ethdo@latest
```

```shell
export ETHDO_WALLET_NAME="Development"
export ETHDO_PASSPHRASE="Development"
export ETHDO_MNEMONIC="..."
export ETHDO_ACCOUNT_INDEX=1
export ETHDO_CONFIG_WITHDRAWAL_ADDRESS=0x...

# Create a wallet
ethdo wallet create --wallet="${ETHDO_WALLET_NAME}" --type="hd" --wallet-passphrase="${ETHDO_PASSPHRASE}" --mnemonic="${ETHDO_MNEMONIC}" --allow-weak-passphrases

# Delete a wallet
ethdo wallet delete --wallet="${ETHDO_WALLET_NAME}"

# Create an account
ethdo account create --account="${ETHDO_WALLET_NAME}/Validators/${ETHDO_ACCOUNT_INDEX}" --wallet-passphrase="${ETHDO_PASSPHRASE}" --passphrase="${ETHDO_PASSPHRASE}" --allow-weak-passphrases --path="m/12381/3600/${ETHDO_ACCOUNT_INDEX}/0/0"

# Create deposit data for a new validator
ethdo validator depositdata --validatoraccount="${ETHDO_WALLET_NAME}/Validators/${ETHDO_ACCOUNT_INDEX}" --depositvalue="32Ether" --withdrawaladdress="${ETHDO_CONFIG_WITHDRAWAL_ADDRESS}" --passphrase="${ETHDO_PASSPHRASE}" --forkversion="0x40000038"
```

#### Integration testing

##### Local Testnet

In addition to the previously mentioned requirements, you will need the following in order to run the local testnet:

- [Setup docker](https://docs.kurtosis.com/next/install#i-install--start-docker)
- [Install kurtosis](https://docs.kurtosis.com/install/#ii-install-the-cli)

###### Launch a local ethereum network

```shell
kurtosis run --enclave batch-deposit-contract config/localnet/main.star "$(cat ./config/localnet/params.json)"
```

Please note that you will need to wait 20 seconds until genesis.

With default settings being used, the network will run at `http://127.0.0.1:64248`.
Prefunded accounts use the following private keys ([source](https://github.com/kurtosis-tech/ethereum-package/blob/main/src/prelaunch_data_generator/genesis_constants/genesis_constants.star)):

```md
bcdf20249abf0ed6d944c0288fad489e33f66b3960d9e6229c1cd214ed3bbe31 -> 0x8943545177806ED17B9F23F0a21ee5948eCaa776
39725efee3fb28614de3bacaffe4cc4bd8c436257e2c8bb887c4b5c4be45e76d -> 0xE25583099BA105D9ec0A67f5Ae86D90e50036425
53321db7c1e331d93a11a41d16f004d7ff63972ec8ec7c25db329728ceeb1710 -> 0x614561D2d143621E126e87831AEF287678B442b8
ab63b23eb7941c1251757e24b3d2350d2bc05c3c388d06f8fe6feafefb1e8c70 -> 0xf93Ee4Cf8c6c40b329b0c0626F28333c132CF241
5d2344259f42259f82d2c140aa66102ba89b57b4883ee441a8b312622bd42491 -> 0x802dCbE1B1A97554B4F50DB5119E37E8e7336417
27515f805127bebad2fb9b183508bdacb8c763da16f54e0678b16e8f28ef3fff -> 0xAe95d8DA9244C37CaC0a3e16BA966a8e852Bb6D6
7ff1a4c1d57e5e784d327c4c7651e952350bc271f156afb3d00d20f5ef924856 -> 0x2c57d1CFC6d5f8E4182a56b4cf75421472eBAEa4
3a91003acaf4c21b3953d94fa4a6db694fa69e5242b2e37be05dd82761058899 -> 0x741bFE4802cE1C4b5b00F9Df2F5f179A1C89171A
bb1d0f125b4fb2bb173c318cdead45468474ca71474e2247776b2b4c0fa2d3f5 -> 0xc3913d4D8bAb4914328651C2EAE817C8b78E1f4c
850643a0224065ecce3882673c21f56bcf6eef86274cc21cadff15930b59fc8c -> 0x65D08a056c17Ae13370565B04cF77D2AfA1cB9FA
94eb3102993b41ec55c241060f47daa0f6372e2e3ad7e91612ae36c364042e44 -> 0x3e95dFbBaF6B348396E6674C7871546dCC568e56
```

###### Deploying

Find the rpc port of your locally running execution client by running

```shell
npx hardhat update-rpc-port
# Remember to source the env
source .env
```

This command updates the local environment (`.env`) used for the `networks.localnet` configuration in [./hardhat.config.ts](/hardhat.config.ts) to use this port.

You can now deploy and interact with the contract.

###### Interacting

32 validators are pre-configured and await activation. Their keys are derived from the following mnemonic:

```text
flee title shaft evoke stable vote injury ten strong farm obtain pause record rural device cotton hollow echo good acquire scrub buzz vacant liar
```

You'll need create deposit data first to deposit to one or multiple of these validators. The following examples use `ethdo` as mentioned in the testing section.

```shell
export WITHDRAWAL_ADDRESS=0x4E9A3d9D1cd2A2b2371b8b3F489aE72259886f1A
export ETHDO_CONFIG_WALLET=Development
export ETHDO_CONFIG_PASSPHRASE=test
export ETHDO_CONFIG_MNEMONIC="flee title shaft evoke stable vote injury ten strong farm obtain pause record rural device cotton hollow echo good acquire scrub buzz vacant liar"
export ETHDO_CONFIG_WITHDRAWAL_ADDRESS=$WITHDRAWAL_ADDRESS

ethdo wallet create --wallet="${ETHDO_CONFIG_WALLET}" --type="hd" --wallet-passphrase="${ETHDO_CONFIG_PASSPHRASE}" --mnemonic="${ETHDO_CONFIG_MNEMONIC}" --allow-weak-passphrases

for ETHDO_VALIDATOR_INDEX in {0..31}
do
    ethdo account create --account="${ETHDO_CONFIG_WALLET}/Validators/${ETHDO_VALIDATOR_INDEX}" --wallet-passphrase="${ETHDO_CONFIG_PASSPHRASE}" --passphrase="${ETHDO_CONFIG_PASSPHRASE}" --allow-weak-passphrases --path="m/12381/3600/${ETHDO_VALIDATOR_INDEX}/0/0"
    ethdo validator depositdata --validatoraccount="${ETHDO_CONFIG_WALLET}/Validators/${ETHDO_VALIDATOR_INDEX}" --depositvalue="32Ether" --withdrawaladdress="${ETHDO_CONFIG_WITHDRAWAL_ADDRESS}" --passphrase="${ETHDO_CONFIG_PASSPHRASE}" --forkversion="0x40000038" > /tmp/local-validator-depositdata-${ETHDO_VALIDATOR_INDEX}.json
done

# Get public keys
ls /tmp/local-validator-*.json | xargs -I {} jq -r '.[0].pubkey' {} | awk 'BEGIN{ORS=","} {print}' | sed 's/,$/\n/' > /tmp/local-validator-pubkeys.txt

# Get signatures
ls /tmp/local-validator-*.json | xargs -I {} jq -r '.[0].signature' {} | awk 'BEGIN{ORS=","} {print}' | sed 's/,$/\n/' > /tmp/local-validator-signatures.txt

# Get deposit data roots
ls /tmp/local-validator-*.json | xargs -I {} jq -r '.[0].deposit_data_root' {} | awk 'BEGIN{ORS=","} {print}' | sed 's/,$/\n/' > /tmp/local-validator-deposit-data-roots.txt

# Concatenate depositdata
echo '[' > /tmp/local-validator-depositdata.json
cat /tmp/local-validator-depositdata-*.json | jq -c '.[]' | sed '$!s/$/,/' >> /tmp/local-validator-depositdata.json
echo ']' >> /tmp/local-validator-depositdata.json

# Remove the development wallet
ethdo wallet delete --wallet="${ETHDO_CONFIG_WALLET}"
```

You can now extract the respective depositdata from the created depositdata files in `/tmp`. For example, get the pubkeys for registering them with the BatchDeposit contract:

```shell
# deploy the BatchDeposit contract
npx hardhat batch-deposit:deploy --network localnet --ethereum-deposit-contract-address 0x4242424242424242424242424242424242424242
# export the address afterwards: export BATCH_DEPOSIT_CONTRACT_ADDRESS=0x...

# deposit to multiple validators
npx hardhat batch-deposit:batch-deposit --network localnet --batch-deposit-contract-address $BATCH_DEPOSIT_CONTRACT_ADDRESS --withdrawal-address $WITHDRAWAL_ADDRESS --validator-public-keys "0x8e1b5d5d2938c6ae35445875f5a6410d8a8f6b93b486ee795632ef1cc9329849e91098a4d86108199ea9f017a4f57ce3,0x8c35be170b4741be1314e22d46e0a8ddca9d08c182bcd9f37e85a1fd1ea0d37dbcf972e13a86f2ba369066d098140694,0xb8c4b28d46a73aa82c400b7f159645b097953d37e2ca98908bc236b5b6292a6ba3a0612e8454867a3f9f38a1c8184d0f" --validator-signatures "..." --validator-deposit-data-roots "..."

# execute a native ethereum staking deposit
npx hardhat native-staking:deposit --network localnet --ethereum-deposit-contract-address 0x4242424242424242424242424242424242424242 --deposit-data-path /tmp/local-validator-depositdata-16.json

# Listing all deposits
npx hardhat --network localnet debug:list-deposits --ethereum-deposit-contract-address 0x4242424242424242424242424242424242424242

# debugging a transaction
npx hardhat --network localnet debug:transaction --tx-hash 0xbc0ce66317705141485622a5c30e91f6a54fbae7a601056563887688c72e6949
```

You can use the following command to get the public keys of the validators:

```bash
ls /tmp/local-validator-*.json | xargs -I {} jq -r '.[0].pubkey' {}
```

###### Observing

You can stream the logs of these validator nodes with:

```shell
docker logs -f $(docker ps | grep development-lighthouse-validator | awk '{print $1}' | tr -d '\n')
```

This is especially helpful for observing validators become active upon using `BatchDepoit.sol`.

You can use `ethdo` to inspect validators:

```shell
# Get general chain info
ethdo --connection=http://localhost:$CL_RPC_PORT chain info

# Get info about validator #1
ethdo --connection=http://localhost:$CL_RPC_PORT validator info --validator 0x8e1b5d5d2938c6ae35445875f5a6410d8a8f6b93b486ee795632ef1cc9329849e91098a4d86108199ea9f017a4f57ce3
ethdo --connection=http://localhost:$CL_RPC_PORT validator info --validator 0x8c35be170b4741be1314e22d46e0a8ddca9d08c182bcd9f37e85a1fd1ea0d37dbcf972e13a86f2ba369066d098140694
ethdo --connection=http://localhost:$CL_RPC_PORT validator info --validator 0xb8c4b28d46a73aa82c400b7f159645b097953d37e2ca98908bc236b5b6292a6ba3a0612e8454867a3f9f38a1c8184d0f
```

###### Cleanup

Remember to clean up when you are done.

```shell
# Clean up kurtosis
kurtosis clean -a

# Remove the development wallet
ethdo wallet delete --wallet="${ETHDO_CONFIG_WALLET}"
```

## Deployment

Compile the contracts with:

```shell
npm run compile
```

And continue by following the network-specific instructions below to deploy the contracts to a network.

### Localnet

Deploying to the locally running network is as simple as running:

```shell
npx hardhat batch-deposit:deploy --network localnet --ethereum-deposit-contract-address 0x4242424242424242424242424242424242424242
```

### Holesky

Deploying to Holesky requires [`Frame`](https://frame.sh) as the external signer. This setup enables the use of hardware wallets during the deployment.

```shell
npx hardhat batch-deposit:deploy-via-frame --network holesky --ethereum-deposit-contract-address 0x4242424242424242424242424242424242424242

# export the address afterwards: export BATCH_DEPOSIT_CONTRACT_ADDRESS=0x...

# Verify the contract
npm run verify -- --network holesky $BATCH_DEPOSIT_CONTRACT_ADDRESS "0x4242424242424242424242424242424242424242"
```

### Mainnet

Deploying to the Ethereum mainnet requires [`Frame`](https://frame.sh) as the external signer. This setup enables the use of hardware wallets during the deployment.

The Beacon Chain Deposit Contract address is `0x00000000219ab540356cBB839Cbe05303d7705Fa` and can be verified in different sources:

- <https://ethereum.org/staking/deposit-contract>
- <https://etherscan.io/address/0x00000000219ab540356cbb839cbe05303d7705fa>
- <https://github.com/ethereum/consensus-specs/blob/dev/configs/mainnet.yaml#L110>

```shell
npx hardhat batch-deposit:deploy-via-frame --network mainnet --ethereum-deposit-contract-address 0x00000000219ab540356cBB839Cbe05303d7705Fa

# export the address afterwards: export BATCH_DEPOSIT_CONTRACT_ADDRESS=0x...

# Verify the contract
npm run verify -- --network mainnet $BATCH_DEPOSIT_CONTRACT_ADDRESS "0x00000000219ab540356cBB839Cbe05303d7705Fa"
```
