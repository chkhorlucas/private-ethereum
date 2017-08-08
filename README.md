# Setting up a private Ethereum network
## Prerequisites
1. Register an account with [AWS](https://aws.amazon.com/)

## Quick setup of AWS Instance
1. Click **Launch Instance**.
2. Select **Ubuntu Server 16.04 LTS**.
3. Make sure *free tier* is selected, click **Review and Launch**.
4. Click **Launch**.
5. Select **Create a new key pair**, give a name to your key pair, and **Download key pair**.
6. Click **Launch Instance**.
7. Click **View Instances**.

## Setup second instance
1. Make sure the existing instance is selected.
2. Click **Actions → Launch** more like this from the dashboard.
3. Click **Launch**.
4. Select **Choose an existing key pair and Launch Instances**.

## Configure security group
1. In the AWS dashboard, click **Security Groups** under **Network & Security**.
2. Select the group that both instances under.
3. Click **Action → Edit inbound rules**.
4. Set Type: All Traffic; Protocol: All; Port Range: 0 - 65535; Source: Anywhere.
5. **Save** settings.

## (Optional) Full Detail
[Getting Started with Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html?icmpid=docs_ec2_console)

## Connect to an Instance
1. Launch terminal (command prompt).
2. Make sure the downloaded key pair file is in the root directory, else move it here.
3. In the AWS dashboard, select an Instance and click **Connect**.
4. Key in below command into the terminal, hit enter.
```
$ chmod 400 <your key pair file name>
```

5. Copy the ssh command in Example above and paste into terminal, hit enter.
```
$ ssh -i "eth_test.pem" ubuntu@ec2-13-229-61-159.ap-southeast-1.compute.amazonaws.com
```

6. Type ‘yes’ to continue.
7. We have successfully connect to an instance hosted by AWS.

## Setting up a private testnet
1. Key in below commands in the session with the instance:
```
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository -y ppa:ethereum/ethereum
$ sudo apt-get update
$ sudo apt-get install ethereum
```

## Pre-allocate balance to account before initializing genesis block
1. Generate a new account:
```
$ geth --datadir="eth_testdata/" account new
```

2. Refer to *Setup the genesis block* for configuring the genesis block, replace the newly created address under “alloc”.
3. Reason we don’t use mining to generate ether, is because we are setting up our nodes in VM with only 1GB memory, which is a bit insufficient to generate DAG files. DAG is part of Dagger Hashimoto Algorithm, consensus algorithm of Ethereum blockchain.
4. [For more details about DAG.](https://github.com/ethereum/wiki/blob/master/Dagger-Hashimoto.md)

## Setup the genesis block
1. We need a CLI text editor. Download nano.
```
$ sudo apt-get install nano
```

2. Create *genesis.json* by below command:
```
$ touch genesis.json
```

3. Open *genesis.json* with nano.
```
$ nano genesis.json
```

4. Copy below and paste it into the text editor. Press CTRL-O, hit enter to save file.

*genesis.json*
```
{
  "config": {
        "chainId": 0,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
  "alloc"      : {
    "0x0000000000000000000000000000000000000001": {"balance": "100000000"},
    "0x0000000000000000000000000000000000000002": {"balance": "200000000"}
  },
  "coinbase"   : "0x0000000000000000000000000000000000000000",
  "difficulty" : "0x20000",
  "extraData"  : "",
  "gasLimit"   : "0x2fefd8",
  "nonce"      : "0x0000000000009942",
  "mixhash"    : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "parentHash" : "0x0000000000000000000000000000000000000000000000000000000000000000",
  "timestamp"  : "0x00"
}
```

5. Press CTRL-X to exit.
6. Make new folder
```
$ mkdir eth_testdata
```

7. Run geth init with below command:
```
$ geth --datadir="eth_testdata" init genesis.json
```

8. Launch geth console.
```
$ geth --datadir="eth_testdata" --networkid 12020 --nodiscover console
```

9. Open a new terminal session.
10. Go to AWS dashboard and connect to second instance. (Refer to *Connect to an Instance* above, skip the chmod command)
11. Repeat steps in **Setting up a private testnet** and **Setup the genesis block** for the second instance.

## Connecting the nodes (Method 1 - manual connect)
1. We now have 2 nodes running on geth (node A & node B).
2. Key in below command to check enode for both:
```
> admin.nodeInfo.enode
```

3. Copy the whole return string from node A.
```
"enode://a2d377166f5bb3ab25f588ff384ab8509ba848b22b29fa357af8f776a8c7c9589dc3f63c323539a24ace20161a87cf97dd5974867e031521cbd06256213ecee9@[::]:30303?discport=0"
```

4. In node B, key in below command and paste the copied string into the parentheses. Replace the **[ : : ]** with node A’s IP address, hit enter.
```
> admin.addPeer(<enode url>)
```

5. Do vice versa for node A.

## Connecting the nodes (Method 2 - auto connect)
1. Create a *static-nodes.json* in *<datadir>*.
```
$ touch eth_testdata/static-nodes.json
```

2. Setup peer’s enode into the json file.
```
[
"enode://7df329ff5de04d0071fade02eaeb33e24df91cb8bacc553dc67c46fedbc6bd3932d37f1f60064ebcc85d0aa3401644d96b9b1041191e844104b143b9938cd11c@[::]:30303"
]
```

3. Replace **[ : : ]** with IP address of node that wish to connect upon launching geth.

### More info about connected peers
```
> admin.peers
```
### Checking for connectivity
```
> net.listening
true
> net.peerCount
1
```
### Create new account
```
> personal.newAccount
```
### List all accounts
```
> personal.listAccounts
```
### Check account by index
```
> eth.accounts[0]
```
### Unlock account
```
> personal.unlockAccount(address, "password")
```
### Check account balance
```
> web3.eth.getBalance("0x24b1404030807903beea6a3d75bd4abd31847032")

> web3.fromWei(eth.getBalance(eth.coinbase), "ether")
```
### Set Ether base
```
> miner.setEtherbase(eth.coinbase)
```
### Start mining
```
> miner.start(1)
```
### Stop mining
```
> miner.stop
```
### Send transaction
```
> var sender = eth.accounts[0];
> var receiver = eth.accounts[1];
> var amount = web3.toWei(0.01, "ether")
> eth.sendTransaction({from:sender, to:receiver, value: amount})
```
### Check pending transactions
```
> eth.pendingTransactions
```

## Conclusion
1. From the above example, after sending a transaction, it is reside in the memory pool until a new block being mined.
2. One of mining function is to append transactions into the blockchain, by accumulating them into a block.
3. For private blockchain, *Proof of Work* (mining) might not be the best solution to achieve consensus within the network. (PoW is unnecessary and pointless to implement in a private/consortium kind of network, where by the nodes are recognized and permissioned.)
4. We can either explore other protocol, eg: Hyperledger Fabric, with PBFT as the consensus algorithm, or replace the Ethereum's POW with other consensus algorithm.

## Reference
[More details on web3.js](https://github.com/ethereum/wiki/wiki/JavaScript-API)
