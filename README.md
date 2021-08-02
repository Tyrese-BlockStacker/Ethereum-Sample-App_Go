# ethereum-sample-app
## Run nodes
```
docker-compose -f docker-compose.nodes.yml up -d
```

Test
```
docker exec -it root_ethdb sh
geth attach /home/data/geth.ipc
eth.getBalance('0x0000000000000000000000000000000000000001')
eth.getBalance('0x0000000000000000000000000000000000000002')
```

Create new account on root
```
personal.newAccount()
```
Output
```
abc@123
0xf08f722155afd6c1a69cb4b831d4db8c6208b35a
```

```
eth.getBalance('0xf08f722155afd6c1a69cb4b831d4db8c6208b35a')
```

Start miner
```
miner.start() 
```

Start miner
```
miner.stop() 
```

## Start peer
Open terminal to connect node1
```
docker exec -it node1_ethdb sh
geth attach /home/data/geth.ipc
admin.nodeInfo.enode
```

Output is
```
"enode://edd8dcf563ad1513777c64b92f7afda6b565de35e46c9a2a2cc55b212b1fdb532c447009f481c4184a831608572c71a0e2f3e68cffceaeac293c81866fc4d1df@127.0.0.1:30303?discport=0"
```

Open terminal to connect root node, then run
```
admin.addPeer("enode://edd8dcf563ad1513777c64b92f7afda6b565de35e46c9a2a2cc55b212b1fdb532c447009f481c4184a831608572c71a0e2f3e68cffceaeac293c81866fc4d1df@node1_ethdb:30303?discport=0")

```