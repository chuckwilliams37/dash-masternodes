# Setting up a Dash Masternode against your SALT Collateral Wallet

In order to run a masternode using your collateral, DO NOT send it onto the platform in a standard transaction.
If you have already sent your collateral to your wallet, withdraw it and follow the below steps.
If you are unable to withdraw because you have an active loan, contact support with the information listed in section 3
and we will issue the ProRegTx for you.

## 1. Setup the Masternode Software
Follow the instructions at https://docs.dash.org/en/stable/masternodes/setup.html
SKIP the sections `Send the collateral` and `Register your masternode`.

## 2. Set up a Dash Core Wallet and send 1000 Dash to it
This can be your masternode itself, or Dash-QT.
1. Go to *Tools > Debug console*
2. Run `getnewaddress`
3. *Backup your wallet* in case anything goes wrong in later steps.
3. Send at least 1000 Dash to this address.
4. Save this address for later. It is your `fundAddress`.

## 3. Collect ProRegTx Parameters
Find or generate, and write down the following parameters:
1. Your Dash collateral wallet address from the SALT platform (`collateralAddress`)
  - You can find this by logging into the portal, going to your Dash wallet, and clicking "Deposit"
2. Your Masternode IP address and port (`ipAndPort`)
  - The IP is the public IP of the host running your masternode
  - The port is the number found after the `:` of the `bind` field of `~/.dashcore/dash.conf` or `9999` if not specified.
  - The combination of these values should be separated by `:`. For example, `172.0.1.134:9999`.
3. Your Owner Key Address (`ownerKeyAddr`)
  - This must be an address generated from a *private key you know* and must be generated by, or imported into your Dash Core wallet.
4. Operator PubKey (`operatorPubKey`)
  - You can find this by running `bls generate` in your Dash Core wallet (*Tools > Debug console*). Make sure to also save the corresponding `secret` for later.
5. Voting Key Address (`votingKeyAddr`)
  - This must be an address generated from a *private key you know*. It does not necessarily need to be in your Dash Core wallet, but you will need it to vote on proposals later.
6. Operator Reward (`operatorReward`)
  - The percentage of the reward you want to go to the node operator.
  - Note: under this guide you are both the owner and the operator, so the recommendation is to set this value to `0.00`.
7. Payout Address (`payoutAddress`)
  - This is the address you want your masternode payouts to go to. This can be the same as your `collateralAddress` if you want to help shore up your LTV.
8. Fund Address (`fundAddress`)
  - This should have been generated in section 2.
9. Legacy Masternode Private Key (`masternodePrivKey`)
  - From *Tools > Debug console*, run `masternode genkey` to generate this.

## 4. Register your Masternode
1. From *Tools > Debug console* run the following command with the parameters collected in section 3:
  - `protx register_fund <collateralAddress> <ipAndPort> <ownerKeyAddr> <operatorPubKey> <votingKeyAddr> <operatorReward> <payoutAddress> <fundAddress>`
    - Note: if your wallet has a passphrase, it must be unlocked during this step.
  - This will return a `txid`, copy this for later.
  - This will also send your collateral to your Dash Collateral Wallet on the SALT platform
2. With the txid from the previous step, run `getrawtransaction <txid> true`.
  - This will return a large JSON blob. Look through it for the key `proRegTx`. Within that section look for `collateralIndex` and copy the value after it.

## 5. Configure your Masternode
1. Using the `secret` generated in *3.4*, and the `masternodePrivKey` generated in *3.9*, add the following lines to `~/.dashcore/dash.conf`:
```
masternode=1
masternodeprivkey=<masternodePrivKey>
masternodeblsprivkey=<secret>
```
2. Using the `ipAndPort` from *3.2*, the `masternodePrivKey` from *3.9*, the `txid` from *4.1*, and the `collateralIndex` from *4.2* add the following line to `~/.dashcore/masternode.conf`:
```
mn1 <ipAndPort> <masternodePrivKey> <txid> <collateralIndex>
```
3. Restart your masternode:
```
~/.dashcore/dash-cli stop
~/.dashcore/dashd
```

## 6. Check your masternode status
Run `~/.dashcore/dash-cli masternode status`. Once your ProTx is confirmed on chain, the `state` should be `READY`.
