# ethereum-sample-app

## Install Geth

Follow instruction on https://geth.ethereum.org/docs/install-and-build/installing-geth

On Ubuntu:

```
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

## Setup default account

**Notes:** you can copy folder `node-data-backup` to `node-data` and skip the step to create default account

```
cp -r node-data-backup node-data
```

Or if you want to create new root account key, you can create your own

### Create default account (skip this step if you re-use folder node-data-backup)

```
mkdir ./node-data/root-node
geth --datadir ./node-data/root-node account new
```

I am using password: `abc@123` and my Public address of the key: `0x132278bFB3eC1b066DE3fBe595Fd805AfAb55e27`

Update address into alloc `./root-node-config/genesis.json`

### Create enode for bootnode

Create node key (you can skip this step if reuse folder `node-data-backup`)

```
vi ./node-data/root-node/geth/nodekey
e94610acbc6831950198f401ba445c7f4f194ac6810b61ae9fed3faeefd3d270
cat ./node-data/root-node/geth/nodekey
```

```
bootnode -nodekey ./node-data/root-node/geth/nodekey -writeaddress
60163d47ccfeb9f32eaf7c5f554e2352e6dc225d7f40e4556a0237f6d520fb4d4fc2c903fa3eacaae41d170de2dda2f2f33946dad2b795e411a99bada2dfe506
```

## Init node data

Init root data folder:

```
mkdir ./node-data/root-node
geth init --datadir ./node-data/root-node ./root-node-config/genesis.json
```

### Init node 1 data folder:

```
mkdir ./node-data/node1
geth init --datadir ./node-data/node1 ./root-node-config/genesis.json
```

## Start nodes

```
docker-compose -f docker-compose.nodes.yml up -d
docker ps
```

## Check data on first node

Access `root_ethdb`

```
docker exec -it root_ethdb sh
geth attach /home/data/geth.ipc
```

Check first block:

```
eth.getBlock(0)
```

Get balance

```
eth.getBalance('0x132278bFB3eC1b066DE3fBe595Fd805AfAb55e27')
```

Output

```
1e+27
```

## Add new node to network

On `root_ethdb`, get `enode`

```
admin.nodeInfo.enode
enode://60163d47ccfeb9f32eaf7c5f554e2352e6dc225d7f40e4556a0237f6d520fb4d4fc2c903fa3eacaae41d170de2dda2f2f33946dad2b795e411a99bada2dfe506@127.0.0.1:30303?discport=0
```

Open new terminal, run `docker inspect myeth-network` to get network IP of `root_ethdb`. It should be `172.28.0.10`

Open new terminal to access `node1_ethdb`

```
docker exec -it node1_ethdb sh
geth attach /home/data/geth.ipc
```

Replace `127.0.0.1` by network IP, then run `admin.addPeer` to connect `root_ethdb`

```
admin.addPeer("enode://60163d47ccfeb9f32eaf7c5f554e2352e6dc225d7f40e4556a0237f6d520fb4d4fc2c903fa3eacaae41d170de2dda2f2f33946dad2b795e411a99bada2dfe506@172.28.0.10:30303?discport=0")
admin.peers
```

Output

```
[{
    caps: ["eth/65", "eth/66", "snap/1"],
    enode: "enode://60163d47ccfeb9f32eaf7c5f554e2352e6dc225d7f40e4556a0237f6d520fb4d4fc2c903fa3eacaae41d170de2dda2f2f33946dad2b795e411a99bada2dfe506@172.28.0.10:30303?discport=0",
    id: "69b18d3527e3313be30d3afc76cf735284b8f95f5c214574c3e204bf04c01f21",
    name: "Geth/v1.10.7-unstable/linux-amd64/go1.16.7",
    network: {
      inbound: false,
      localAddress: "172.28.0.11:34460",
      remoteAddress: "172.28.0.10:30303",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 1,
        head: "0x5565de2b177067051b60f5a3e5ca205533ca7dbcf0cc705b879a80a0c282e16e",
        version: 66
      },
      snap: {
        version: 1
      }
    }
}]
```

## Wait for network syncing all data

It will take time that all nodes are syncing data

```
eth.syncing
false (or it will show block number if true)
```

## Create new account on `node1_ethdb`

Open new terminal to access `node1_ethdb`

```
docker exec -it node1_ethdb sh
geth attach /home/data/geth.ipc
```

Create new account (I am using password `abc@123`)

```
personal.newAccount()
```

Output

```
0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2 (this is new account)
```

Check balance of new account

```
eth.getBalance('0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2')
```

Output

```
0
```

# Start miner

Show all accounts

```
eth.accounts
```

Output

```
["0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2"]
```

Set account

```
miner.setEtherbase('0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2')
eth.getBalance('0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2')
```

Start miner

```
miner.start()
```

It will take long time to have a new block. Run `eth.blockNumber` to check new block. It depends on CPU configuration in docker-compose

Stop miner

```
miner.stop()
```

## Test to send token to new account

On `root_ethdb`, check balance of 2 accounts were create on `root_ethdb`

```
eth.getBalance('0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2')
eth.getBalance('0x132278bFB3eC1b066DE3fBe595Fd805AfAb55e27')
```

Unlock default account

```
personal.unlockAccount("0x132278bFB3eC1b066DE3fBe595Fd805AfAb55e27", "abc@123", 1000);
```

Send 1 token to `0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2`

```
eth.sendTransaction({
from: '0x132278bFB3eC1b066DE3fBe595Fd805AfAb55e27',
to: '0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2',
value: '1'
})
```

Output

```
0x8bd668add121145c7295f7f5e54e206125a988d3412bbe4411566a4accabdb0d
```

Check transaction

```
eth.getTransaction('0x8bd668add121145c7295f7f5e54e206125a988d3412bbe4411566a4accabdb0d')
```

Check balance again

```
eth.getBalance('0x5b8b5e6c4ba00c3779001aba1790ee977f8059b2')
eth.getBalance('0x132278bFB3eC1b066DE3fBe595Fd805AfAb55e27')
```
