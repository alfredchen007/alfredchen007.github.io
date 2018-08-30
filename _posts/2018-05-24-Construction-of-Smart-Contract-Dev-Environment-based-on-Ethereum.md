---

layout: post

title: "Construction of Smart Contract Dev-Environment based on Ethereum"

date: 2018-05-24 20:40:20 +0300

description:  

img: ethereum.jpg # Add image post (optional)

tags: [blockchain]

---

We will introduce how to build development environment of smart contract in this blog and you will get the method how to kick off with a simple "Hello World" demo.<!-- more -->

### Solidity

We will use Remix-Solidity IDE and this is the easier way to start due to you do not need install Solidity, you  can still install [Solidity](https://solidity.readthedocs.io/en/develop/installing-solidity.html) as you will. 

### Install geth in Mac

```shell
AlfreddeMacBook-Pro:~ alfred$ brew tap ethereum/ethereum
Updating Homebrew...
==> Downloading https://homebrew.bintray.com/bottles-portable-ruby/portable-ruby-2.3.7.leopard_64.bottle.tar.gz
######################################################################## 100.0%
==> Pouring portable-ruby-2.3.7.leopard_64.bottle.tar.gz
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> New Formulae

...(省略部分列表）

==> Renamed Formulae
cdiff -> ydiff                                       latexila -> gnome-latex                              php71 -> php@7.1
crystal-lang -> crystal                              php56 -> php@5.6                                     saltstack -> salt
geth -> ethereum                                     php70 -> php@7.0                                     wpcli-completion -> wp-cli-completion
==> Deleted Formulae
arm                       bokken                    i3status                  mal4s                     nazghul                   voltdb
artifactory-cli-go        ecj                       llvm@3.8                  mimetic                   picolisp                  wry
aws-cloudsearch           i3                        luciddb                   monotone                  ufoai

==> Tapping ethereum/ethereum
Cloning into '/usr/local/Homebrew/Library/Taps/ethereum/homebrew-ethereum'...
remote: Counting objects: 6, done.
remote: Compressing objects: 100% (6/6), done.
remote: Total 6 (delta 1), reused 0 (delta 0), pack-reused 0
Unpacking objects: 100% (6/6), done.
Tapped 3 formulae (33 files, 33.6KB).
```



```shell
AlfreddeMacBook-Pro:~ alfred$ brew install ethereum
Updating Homebrew...
==> Downloading https://homebrew.bintray.com/bottles/ethereum-1.8.12.high_sierra.bottle.tar.gz
######################################################################## 100.0%
==> Pouring ethereum-1.8.12.high_sierra.bottle.tar.gz
🍺  /usr/local/Cellar/ethereum/1.8.12: 20 files, 228MB
```



### Start Geth 

Geth is a ethereum client, now we start an ethereum network node with geth

```shell
AlfreddeMacBook-Pro:chenjingchao alfred$ mkdir blockchain
AlfreddeMacBook-Pro:chenjingchao alfred$ cd blockchain/
AlfreddeMacBook-Pro:blockchain alfred$ ls
AlfreddeMacBook-Pro:blockchain alfred$ geth --datadir testNet --dev console 2>> test.log
Welcome to the Geth JavaScript console!

instance: Geth/v1.8.12-stable/darwin-amd64/go1.10.3
coinbase: 0xf70888b4f9f3f10f13ac7939765636dbef62b77c
at block: 0 (Thu, 01 Jan 1970 08:00:00 CST)
 datadir: /Users/alfred/chenjingchao/blockchain/testNet
 modules: admin:1.0 clique:1.0 debug:1.0 eth:1.0 miner:1.0 net:1.0 personal:1.0 rpc:1.0 shh:1.0 txpool:1.0 web3:1.0

> 

```

**--dev** means develope network mode, POA will be used by this mode and a developer account will preallocated which surpport  automatic mining. 

**–datadir** meas the directory where the block data and key stored, a testNet directory will created when you input the command first time.

**console** just the console.

**2>> test.log** console log output to test.log

### prepare account

```shell
> eth.accounts
["0xf70888b4f9f3f10f13ac7939765636dbef62b77c"]
> 

```

or

```shell
> personal.listAccounts
["0xf70888b4f9f3f10f13ac7939765636dbef62b77c"]
> 

```

### Check balance

```shell
> eth.getBalance(eth.accounts[0])
1.15792089237316195423570985008687907853269984665640564039457584007913129639927e+77
> 

```

**eth.accounts[0]** means the first one of account list, it shows pretty huge balance when you click the enter. 

Beacause of much balance in developer account, nothing changed shall we see when deploying contract. We choose create a new account in terms of better show.

### Create account

```shell
> personal.newAccount("jingchao")
"0xe78bd82783486461e47028a216f441064adb6db7"
> 

```

"jingchao" is password of the new account, it returns a new account as clicking the Enter.

check the account list

```shell
> eth.accounts
["0xf70888b4f9f3f10f13ac7939765636dbef62b77c", "0xe78bd82783486461e47028a216f441064adb6db7"]
> 
```

We will see there have 2 accounts including the one we just created which index number is "1",  and also you can check it in the way below as you will get the same result.

```shell
> personal.listAccounts
["0xf70888b4f9f3f10f13ac7939765636dbef62b77c", "0xe78bd82783486461e47028a216f441064adb6db7"]
> 
```

check the balance of new account

```shell
> eth.getBalance(eth.accounts[1])
0
> 
```

### Transfer accounts

```shell
> eth.sendTransaction({from: '0xf70888b4f9f3f10f13ac7939765636dbef62b77c', to: '0xe78bd82783486461e47028a216f441064adb6db7', value: web3.toWei(1, "ether")})
"0xc29da3bf5a9af4eb6279c0938196690a306a5ed132e08b17d1367647da93d16d"
> eth.getBalance(eth.accounts[1])
1000000000000000000
>
```

### Unlock account

```shell
> personal.unlockAccount(eth.accounts[1], "jingchao");
true
> 
```

### Write contract code

Open [Browser-Solidity](https://ethereum.github.io/browser-solidity/) and write code, click 'details' to deploy.

![]({{site.baseurl}}/assets/img/dev-env-ethereum/01.jpg)

Find the part tagged with "WEB3DEPLOY" and click copy button, modify initial string to "Hello World" after pasting the part into editor.

![]({{site.baseurl}}/assets/img/dev-env-ethereum/02.jpg)

```shell
> var _greeting = "Hello World" ;
undefined
> var helloContract = web3.eth.contract([{"constant":true,"inputs":[],"name":"say","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"}]);
undefined
> var hello = helloContract.new(
...    _greeting,
...    {
......      from: web3.eth.accounts[0], 
......      data: '0x608060405234801561001057600080fd5b506040516102a83803806102a8833981018060405281019080805182019291905050508060009080519060200190610049929190610050565b50506100f5565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061009157805160ff19168380011785556100bf565b828001600101855582156100bf579182015b828111156100be5782518255916020019190600101906100a3565b5b5090506100cc91906100d0565b5090565b6100f291905b808211156100ee5760008160009055506001016100d6565b5090565b90565b6101a4806101046000396000f300608060405260043610610041576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063954ab4b214610046575b600080fd5b34801561005257600080fd5b5061005b6100d6565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009b578082015181840152602081019050610080565b50505050905090810190601f1680156100c85780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b606060008054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561016e5780601f106101435761010080835404028352916020019161016e565b820191906000526020600020905b81548152906001019060200180831161015157829003601f168201915b50505050509050905600a165627a7a723058203cc10a2d759eeaf0cba7a6420a865bcf258ae66fee2577bde1db0b31f0fc0cf10029', 
......      gas: '4700000'
......    }, function (e, contract){
......     console.log(e, contract);
......     if (typeof contract.address !== 'undefined') {
.........          console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
.........     }
......  })
null [object Object]
undefined
> null [object Object]
Contract mined! address: 0x528754900317f565cd18328c430ed71cdb0db239 transactionHash: 0xa75702df9f7729a8c93a6b2616fa4c6a8faea5c3193b103c58d22cb7b105756d
null [object Object]
Contract mined! address: 0xf4a443f2febe597c7ba665cd45378b7f962d32fd transactionHash: 0x1bb023ea80bc3a07e8078d0ccde6ffa3081d67f5a1a4a7460de305a7e2b2f832


```

Contact has been deployed successfully.

### Run contract

check hello

```shell
> hello
{
  abi: [{
      constant: true,
      inputs: [],
      name: "say",
      outputs: [{...}],
      payable: false,
      stateMutability: "view",
      type: "function"
  }, {
      inputs: [{...}],
      payable: false,
      stateMutability: "nonpayable",
      type: "constructor"
  }],
  address: "0xf4a443f2febe597c7ba665cd45378b7f962d32fd",
  transactionHash: "0x1bb023ea80bc3a07e8078d0ccde6ffa3081d67f5a1a4a7460de305a7e2b2f832",
  allEvents: function(),
  say: function()
}

```

run

```shell
> hello.say()
"Hello World"
> 
```



## Remix IDE

### nvm install

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
```

```shell
AlfreddeMacBook-Pro:blockchain alfred$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 12819  100 12819    0     0  11138      0  0:00:01  0:00:01 --:--:-- 11146
=> Downloading nvm from git to '/Users/alfred/.nvm'
=> Cloning into '/Users/alfred/.nvm'...
remote: Counting objects: 267, done.
remote: Compressing objects: 100% (242/242), done.
remote: Total 267 (delta 31), reused 86 (delta 15), pack-reused 0
Receiving objects: 100% (267/267), 119.47 KiB | 249.00 KiB/s, done.
Resolving deltas: 100% (31/31), done.
=> Compressing and cleaning up git repository

=> Appending nvm source string to /Users/alfred/.bash_profile
=> Appending bash_completion source string to /Users/alfred/.bash_profile
=> Close and reopen your terminal to start using nvm or run the following to use it now:

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
AlfreddeMacBook-Pro:blockchain alfred$ 
```

### install node with nvm

```shell
AlfreddeMacBook-Pro:blockchain alfred$ nvm install 7
Downloading and installing node v7.10.1...
Downloading https://nodejs.org/dist/v7.10.1/node-v7.10.1-darwin-x64.tar.xz...
######################################################################## 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v7.10.1 (npm v4.2.0)
Creating default alias: default -> 7 (-> v7.10.1)
AlfreddeMacBook-Pro:blockchain alfred$ 
```

### check version

```shell
AlfreddeMacBook-Pro:blockchain alfred$ node --version
v7.10.1
AlfreddeMacBook-Pro:blockchain alfred$ npm --version
4.2.0
AlfreddeMacBook-Pro:blockchain alfred$ nvm --version
0.33.11

```

### install Remix ide with command line

```shell
npm install remix-ide -g
```

Start Remix IDE

```shell
AlfreddeMacBook-Pro:blockchain alfred$ remix-ide
setup notifications for /Users/alfred/chenjingchao/blockchain
Shared folder : /Users/alfred/chenjingchao/blockchain
Starting Remix IDE at http://localhost:8080 and sharing /Users/alfred/chenjingchao/blockchain
Sat Jul 21 2018 01:20:20 GMT+0800 (CST) Remixd is listening on 127.0.0.1:65520
```

![]({{site.baseurl}}/assets/img/dev-env-ethereum/03.jpg)
