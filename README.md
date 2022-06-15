# 打造一个属于你的NFT Mint 网页!

> 本教程在@Buildspace 的基础上做了对应的优化工作，方便中级班的学员进行上手编程。

## 准备环境

1. Metamask 钱包（需要有一些 Rinkeby 测试币）

教程：https://chainide.gitbook.io/chainide-chinese/huan-jing-pei-zhi

1. ChainIDE.com

## 目标成果

一个网页，用户可以使用它来“铸造”一个 NFT。用户连接 Metamask 钱包 -- 点击 “Mint NFT” -- 确认成功后即可在 Opensea 上转售该 NFT

![https://i.imgur.com/n2gtgFC.png](https://i.imgur.com/n2gtgFC.png)

![https://i.imgur.com/2nQ6Csp.png](https://i.imgur.com/2nQ6Csp.png)



中间的三个单词是随机生成的，而且还是完全存储在链上，就像 Loot 一样，这很酷，是不是？

让我们一起来 Buidl 吧

## 智能合约部分

* 打开 www.chainide.com
* 点击 ETH 图标进入编程页面（可选择登录 github）
* 点击 Terminal - + - npm-hardhat
* 复制下该 github 文档，方便查看

```
git clone 
```



* 输入以下指令，创建一个 epic-nfts 工作目录

 ```
 mkdir epic-nfts
 cd epic-nfts
 ```

* 安装 hardhat 的一个示例项目

```
npx hardhat
```

遇到选项全部回车即可

* 安装 hardhat-waffle , hardhat-ether 和 openzepplin 库 

```
npm install --save-dev @nomiclabs/hardhat-waffle ethereum-waffle chai @nomiclabs/hardhat-ethers ethers
npm install @openzeppelin/contracts
```

* 运行脚本

```
npx hardhat run scripts/sample-script.js
```

* 当出现以下输出时，说明你的 hardhat 环境已配置好，并且还部署了合约到本地区块链上

![](https://i.imgur.com/LIYT9tf.png)

* 删除 test 文件夹下的 sample-test.js，scripts 下的 sample-script.js 和 contracts 下的 Greeter.sol
* 在 contracts 文件夹下创建一个名为 MyEpicNFT.sol 的智能合约

，内容如下

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.1;

// We first import some OpenZeppelin Contracts.
import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
import "@openzeppelin/contracts/utils/Counters.sol";
import "hardhat/console.sol";

// We inherit the contract we imported. This means we'll have access
// to the inherited contract's methods.
contract MyEpicNFT is ERC721URIStorage {
  // Magic given to us by OpenZeppelin to help us keep track of tokenIds.
  using Counters for Counters.Counter;
  Counters.Counter private _tokenIds;

  // We need to pass the name of our NFTs token and its symbol.
  constructor() ERC721 ("SquareNFT", "SQUARE") {
    console.log("This is my NFT contract. Woah!");
  }

  // A function our user will hit to get their NFT.
  function makeAnEpicNFT() public {
     // Get the current tokenId, this starts at 0.
    uint256 newItemId = _tokenIds.current();

     // Actually mint the NFT to the sender using msg.sender.
    _safeMint(msg.sender, newItemId);

    // Set the NFTs data.
    _setTokenURI(newItemId, "blah");

    // Increment the counter for when the next NFT is minted.
    _tokenIds.increment();
  }
}
```

* 安装 openzepplin contracts 包

```
npm install @openzeppelin/contracts
```

> 备注： Token URL (如：https://ipfs.io/ipfs/QmTv6vgUrdYnVDnZQQ8BXrY2pCNyYWk1X1yZxM9PxnDbbi)会链接到 JSON 文件，称为 Metadata，例如
>
> ```
> {
> "description":"Doge meme",
> "name":"Doge",
> "image":"https://ipfs.io/ipfs/QmaszG3dd5XR2swraqTP1gyyw5azkjVnGKtpqttiuZeiDX"
> }
> ```
>
> Metadata 的内容是可以自定义的，但是一般得符合 [Opensea 标准](https://docs.opensea.io/docs/metadata-standards?utm_source=buildspace.so&utm_medium=buildspace_project)，不然在opensea上是看不到你的图片的。

* 在 chainide 的 ERC721模板里的 index.html 将你的图片上传到 ipfs，并复制你的链接
* 替换掉智能合约这一行的 blah

```
_setTokenURI(newItemId, "blah");
```

将其改为我们的 json 文件链接

```
_setTokenURI(newItemId, "https://ipfs.io/ipfs/QmTv6vgUrdYnVDnZQQ8BXrY2pCNyYWk1X1yZxM9PxnDbbi");
```

在该行下，我们添加一个`console.log`来帮助我们查看 NFT 的铸造时间和铸造者！

```
console.log("An NFT w/ ID %s has been minted to %s", newItemId, msg.sender);
```

* 在 scripts 文件夹下新建 run.js 文件，并填入

```js
const main = async () => {
  const nftContractFactory = await hre.ethers.getContractFactory('MyEpicNFT');
  const nftContract = await nftContractFactory.deploy();
  await nftContract.deployed();
  console.log("Contract deployed to:", nftContract.address);

  // Call the function.
  let txn = await nftContract.makeAnEpicNFT()
  // Wait for it to be mined.
  await txn.wait()

  // Mint another NFT for fun.
  txn = await nftContract.makeAnEpicNFT()
  // Wait for it to be mined.
  await txn.wait()

};

const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};

runMain();
```

运行

```
npx hardhat run scripts/run.js
```

可以看到

![](https://i.imgur.com/EfsOs5O.png)

我们已经在本地区块链铸造了一个NFT！

* 去 alchemy 或者 infura  获取一个 rinkeby  rpc
* 搜索 rinkeby faucet 获取一定量的测试币
* 在 scripts 文件夹下生成 deploy.js 文件（run.js 文件主要用来测试）

```js
const main = async () => {
  const nftContractFactory = await hre.ethers.getContractFactory('MyEpicNFT');
  const nftContract = await nftContractFactory.deploy();
  await nftContract.deployed();
  console.log("Contract deployed to:", nftContract.address);

  // Call the function.
  let txn = await nftContract.makeAnEpicNFT()
  // Wait for it to be mined.
  await txn.wait()
  console.log("Minted NFT #1")

  txn = await nftContract.makeAnEpicNFT()
  // Wait for it to be mined.
  await txn.wait()
  console.log("Minted NFT #2")
};

const runMain = async () => {
  try {
    await main();
    process.exit(0);
  } catch (error) {
    console.log(error);
    process.exit(1);
  }
};

runMain();
```

* 部署到 Rinkeby 测试网，修改我们的 hardhat.config.js 文件

```js
require('@nomiclabs/hardhat-waffle');
require("dotenv").config({ path: ".env" });

module.exports = {
  solidity: '0.8.1',
  networks: {
    rinkeby: {
      url: process.env.ALCHEMY_API_KEY_URL,
      accounts: [process.env.RINKEBY_PRIVATE_KEY],
    },
  },
};
```

* 在 terminal 输入，安装 dotenv 环境变量包

```
npm install dotenv
```

* 在 hardhat.config.js  同文件夹下创建一个 .env 文件，并输入以下内容

```
ALCHEMY_API_KEY_URL=<YOUR API URL>
RINKEBY_PRIVATE_KEY=<YOUR PRIVATE KEY>
```

将 YOUR API URL 替换为你的rpc url，YOUR PRIVATE KEY 替换为你的私钥

* 在 .gitignore 的文件里添加 .env,它应该看起来像这样, 防止将 .env 上传到github

```
node_modules
.env
coverage
coverage.json
typechain

#Hardhat files
cache
artifacts
```

* 运行

```
npx hardhat run scripts/deploy.js --network rinkeby
```

使用该行命令将合约部署到 rinkeby 上，成功后你将看到

![](https://i.imgur.com/nLSX6PM.png)

* 进入 testnets.opensea.io ，输入你的合约地址，就可以看到对应的 NFT了

* 如果你的 NFT 没有出现，点击右边的 refresh，等待几分钟后，你就可以在 opensea 上看到你们的metadata了

![](https://i.imgur.com/dVACrDl.png)

![image-20220609180141830](https://chainide-doc.s3.amazonaws.com/ERC721+deployment+on+Rinkeby+Etherum/ERC721+deployment+on+Rinkeby+Etherum/7.png)



## 将 NFT 存储在链上

Loot 将图片数据存储在了链上，这是怎么完成的呢，我们也可以做个类似的 NFT 吗？

* 什么是 SVG 

SVG 是一个用代码构建的图片。

例如，这是一个非常简单的 SVG，它呈现了一个有一些白色文本的黑框。

```svg
<svg xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMinYMin meet" viewBox="0 0 350 350">
    <style>.base { fill: white; font-family: serif; font-size: 14px; }</style>
    <rect width="100%" height="100%" fill="black" />
    <text x="50%" y="50%" class="base" dominant-baseline="middle" text-anchor="middle">EpicLordHamburger</text>
</svg>
```

去该[网站](https://www.svgviewer.dev/?utm_source=buildspace.so&utm_medium=buildspace_project)粘贴它并试试。

前往该[网站](https://www.utilities-online.info/base64?utm_source=buildspace.so&utm_medium=buildspace_project) 并粘贴上面完整的SVG代码，点击 Encode 获得其 base64编码，打开浏览器一个新标签，在 URL栏中粘贴：

```
data:image/svg+xml;base64,INSERT_YOUR_BASE64_ENCODED_SVG_HERE
```

将 INSERT_YOUR_BASE64_ENCODED_SVG_HERE 替换为上面的 base64 编码，如：

```
data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHByZXNlcnZlQXNwZWN0UmF0aW89InhNaW5ZTWluIG1lZXQiIHZpZXdCb3g9IjAgMCAzNTAgMzUwIj4NCiAgICA8c3R5bGU+LmJhc2UgeyBmaWxsOiB3aGl0ZTsgZm9udC1mYW1pbHk6IHNlcmlmOyBmb250LXNpemU6IDE0cHg7IH08L3N0eWxlPg0KICAgIDxyZWN0IHdpZHRoPSIxMDAlIiBoZWlnaHQ9IjEwMCUiIGZpbGw9ImJsYWNrIiAvPg0KICAgIDx0ZXh0IHg9IjUwJSIgeT0iNTAlIiBjbGFzcz0iYmFzZSIgZG9taW5hbnQtYmFzZWxpbmU9Im1pZGRsZSIgdGV4dC1hbmNob3I9Im1pZGRsZSI+RXBpY0xvcmRIYW1idXJnZXI8L3RleHQ+DQo8L3N2Zz4=
```

`data:image/svg+xml;base64`基本上是在说，“嘿，我要给你base64编码的数据，请将它作为SVG处理，谢谢！",我们如果将这段字符存储在区块链上，那么就会实现永久可用的效果了（前提是该区块链不会凉，哈哈）

![](https://i.imgur.com/f9mXVSb.png)

* 生成随机单词 NFT

  在 contracts 文件夹下创建名为 libraries 文件夹，并在里面创建一个名为 Base64.sol 文件，代码内容为

  ```solidity
  /**
   *Submitted for verification at Etherscan.io on 2021-09-05
   */
  
  // SPDX-License-Identifier: MIT
  
  pragma solidity ^0.8.0;
  
  /// [MIT License]
  /// @title Base64
  /// @notice Provides a function for encoding some bytes in base64
  /// @author Brecht Devos <brecht@loopring.org>
  library Base64 {
      bytes internal constant TABLE =
          "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
  
      /// @notice Encodes some bytes to the base64 representation
      function encode(bytes memory data) internal pure returns (string memory) {
          uint256 len = data.length;
          if (len == 0) return "";
  
          // multiply by 4/3 rounded up
          uint256 encodedLen = 4 * ((len + 2) / 3);
  
          // Add some extra buffer at the end
          bytes memory result = new bytes(encodedLen + 32);
  
          bytes memory table = TABLE;
  
          assembly {
              let tablePtr := add(table, 1)
              let resultPtr := add(result, 32)
  
              for {
                  let i := 0
              } lt(i, len) {
  
              } {
                  i := add(i, 3)
                  let input := and(mload(add(data, i)), 0xffffff)
  
                  let out := mload(add(tablePtr, and(shr(18, input), 0x3F)))
                  out := shl(8, out)
                  out := add(
                      out,
                      and(mload(add(tablePtr, and(shr(12, input), 0x3F))), 0xFF)
                  )
                  out := shl(8, out)
                  out := add(
                      out,
                      and(mload(add(tablePtr, and(shr(6, input), 0x3F))), 0xFF)
                  )
                  out := shl(8, out)
                  out := add(
                      out,
                      and(mload(add(tablePtr, and(input, 0x3F))), 0xFF)
                  )
                  out := shl(224, out)
  
                  mstore(resultPtr, out)
  
                  resultPtr := add(resultPtr, 4)
              }
  
              switch mod(len, 3)
              case 1 {
                  mstore(sub(resultPtr, 2), shl(240, 0x3d3d))
              }
              case 2 {
                  mstore(sub(resultPtr, 1), shl(248, 0x3d))
              }
  
              mstore(result, encodedLen)
          }
  
          return string(result);
      }
  }
  ```

  

  将下面的代码替换掉 contracts 里 MyEpicNFT.sol。

  ```solidity
  pragma solidity ^0.8.1;
  
  
  import "@openzeppelin/contracts/utils/Strings.sol";
  import "@openzeppelin/contracts/token/ERC721/extensions/ERC721URIStorage.sol";
  import "@openzeppelin/contracts/utils/Counters.sol";
  import "hardhat/console.sol";
  
  // We need to import the helper functions from the contract that we copy/pasted.
  import { Base64 } from "./libraries/Base64.sol";
  
  contract MyEpicNFT is ERC721URIStorage {
    using Counters for Counters.Counter;
    Counters.Counter private _tokenIds;
  
    string baseSvg = "<svg xmlns='http://www.w3.org/2000/svg' preserveAspectRatio='xMinYMin meet' viewBox='0 0 350 350'><style>.base { fill: white; font-family: serif; font-size: 24px; }</style><rect width='100%' height='100%' fill='black' /><text x='50%' y='50%' class='base' dominant-baseline='middle' text-anchor='middle'>";
  
    string[] firstWords = ["YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD"];
    string[] secondWords = ["YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD"];
    string[] thirdWords = ["YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD"];
    
    event NewEpicNFTMinted(address sender, uint256 tokenId);
  
    constructor() ERC721 ("SquareNFT", "SQUARE") {
      console.log("This is my NFT contract. Woah!");
    }
  
    function pickRandomFirstWord(uint256 tokenId) public view returns (string memory) {
      uint256 rand = random(string(abi.encodePacked("FIRST_WORD", Strings.toString(tokenId))));
      rand = rand % firstWords.length;
      return firstWords[rand];
    }
  
    function pickRandomSecondWord(uint256 tokenId) public view returns (string memory) {
      uint256 rand = random(string(abi.encodePacked("SECOND_WORD", Strings.toString(tokenId))));
      rand = rand % secondWords.length;
      return secondWords[rand];
    }
  
    function pickRandomThirdWord(uint256 tokenId) public view returns (string memory) {
      uint256 rand = random(string(abi.encodePacked("THIRD_WORD", Strings.toString(tokenId))));
      rand = rand % thirdWords.length;
      return thirdWords[rand];
    }
  
    function random(string memory input) internal pure returns (uint256) {
        return uint256(keccak256(abi.encodePacked(input)));
    }
  
    function makeAnEpicNFT() public {
      uint256 newItemId = _tokenIds.current();
  
      string memory first = pickRandomFirstWord(newItemId);
      string memory second = pickRandomSecondWord(newItemId);
      string memory third = pickRandomThirdWord(newItemId);
      string memory combinedWord = string(abi.encodePacked(first, second, third));
  
      string memory finalSvg = string(abi.encodePacked(baseSvg, combinedWord, "</text></svg>"));
  
      // Get all the JSON metadata in place and base64 encode it.
      string memory json = Base64.encode(
          bytes(
              string(
                  abi.encodePacked(
                      '{"name": "',
                      // We set the title of our NFT as the generated word.
                      combinedWord,
                      '", "description": "A highly acclaimed collection of squares.", "image": "data:image/svg+xml;base64,',
                      // We add data:image/svg+xml;base64 and then append our base64 encode our svg.
                      Base64.encode(bytes(finalSvg)),
                      '"}'
                  )
              )
          )
      );
  
      // Just like before, we prepend data:application/json;base64, to our data.
      string memory finalTokenUri = string(
          abi.encodePacked("data:application/json;base64,", json)
      );
  
      console.log("\n--------------------");
      console.log(finalTokenUri);
      console.log("--------------------\n");
  
      _safeMint(msg.sender, newItemId);
      
      // Update your URI!!!
      _setTokenURI(newItemId, finalTokenUri);
    
      _tokenIds.increment();
      console.log("An NFT w/ ID %s has been minted to %s", newItemId, msg.sender);
      emit NewEpicNFTMinted(msg.sender, newItemId);
    }
  }
  ```

  

  将这里的 YOUR_WORD 替换为不同的，你喜欢的单词

  ```solidity
   string[] firstWords = ["YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD"];
    string[] secondWords = ["YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD"];
    string[] thirdWords = ["YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD", "YOUR_WORD"];
  ```

* 运行一下脚本

  ```
  npx hardhat run scripts/sample-script.js
  ```

* 部署到 Rinkeby 上

```
npx hardhat run scripts/deploy.js --network rinkeby
```

这样我们就可以去 [testnet opensea](testnets.opensea.io) 上看到自己发行的随机 NFT 了。

## 生成前端铸造网页

* 将事先准备好的前端框架安装好

```
git clone https://github.com/wufengtao1/buildspace-nft-course-starter.git
```

* 配置好 metamask，切换到rinkeby，并获取一定的测试币
* 使用 window.ethereum()

让钱包可以连接到我们网站, 进入 buildspace-nft-course-starter > src > App.js, 改为以下内容

```js
import React, { useEffect, useState } from "react";
import './styles/App.css';
import twitterLogo from './assets/twitter-logo.svg';

const TWITTER_HANDLE = '_buildspace';
const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;
const OPENSEA_LINK = '';
const TOTAL_MINT_COUNT = 50;

const App = () => {
  const [currentAccount, setCurrentAccount] = useState("");
  
  const checkIfWalletIsConnected = async () => {
    const { ethereum } = window;

    if (!ethereum) {
      console.log("Make sure you have metamask!");
      return;
    } else {
      console.log("We have the ethereum object", ethereum);
    }

    const accounts = await ethereum.request({ method: 'eth_accounts' });

    if (accounts.length !== 0) {
      const account = accounts[0];
      console.log("Found an authorized account:", account);
      setCurrentAccount(account);
    } else {
      console.log("No authorized account found");
    }
  }

  /*
  * Implement your connectWallet method here
  */
  const connectWallet = async () => {
    try {
      const { ethereum } = window;

      if (!ethereum) {
        alert("Get MetaMask!");
        return;
      }

      /*
      * Fancy method to request access to account.
      */
      const accounts = await ethereum.request({ method: "eth_requestAccounts" });

      /*
      * Boom! This should print out public address once we authorize Metamask.
      */
      console.log("Connected", accounts[0]);
      setCurrentAccount(accounts[0]); 
    } catch (error) {
      console.log(error);
    }
  }

  // Render Methods
  const renderNotConnectedContainer = () => (
    <button onClick={connectWallet} className="cta-button connect-wallet-button">
      Connect to Wallet
    </button>
  );

  useEffect(() => {
    checkIfWalletIsConnected();
  }, [])

  /*
  * Added a conditional render! We don't want to show Connect to Wallet if we're already connected :).
  */
  return (
    <div className="App">
      <div className="container">
        <div className="header-container">
          <p className="header gradient-text">My NFT Collection</p>
          <p className="sub-text">
            Each unique. Each beautiful. Discover your NFT today.
          </p>
          {currentAccount === "" ? (
            renderNotConnectedContainer()
          ) : (
            <button onClick={null} className="cta-button connect-wallet-button">
              Mint NFT
            </button>
          )}
        </div>
        <div className="footer-container">
          <img alt="Twitter Logo" className="twitter-logo" src={twitterLogo} />
          <a
            className="footer-text"
            href={TWITTER_LINK}
            target="_blank"
            rel="noreferrer"
          >{`built on @${TWITTER_HANDLE}`}</a>
        </div>
      </div>
    </div>
  );
};

export default App;
```

* 在前端上添加铸造 NFT 功能，将 makeEpicNFT 合约函数添加到 connectWallet 中。在上面的函数下继续添加以下内容

```js
const askContractToMintNft = async () => {
  // INSERT_YOUR_DEPLOYED_RINKEBY_CONTRACT_ADDRESS 为之前部署的合约地址
  const CONTRACT_ADDRESS = "INSERT_YOUR_DEPLOYED_RINKEBY_CONTRACT_ADDRESS";

  try {
    const { ethereum } = window;

    if (ethereum) {
      const provider = new ethers.providers.Web3Provider(ethereum);
      const signer = provider.getSigner();
      const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, myEpicNft.abi, signer);

      console.log("Going to pop wallet now to pay gas...")
      let nftTxn = await connectedContract.makeAnEpicNFT();

      console.log("Mining...please wait.")
      await nftTxn.wait();
      
      console.log(`Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`);

    } else {
      console.log("Ethereum object doesn't exist!");
    }
  } catch (error) {
    console.log(error)
  }
}
```

在顶部添加 ether.js 内容

```js
import { ethers } from "ethers";
```

当有人点击 “Mint NFT” 时，我们需要调用 makeEpicNFT 函数，所以修改之前的代码

```js
return (
  {currentAccount === "" 
    ? renderNotConnectedContainer()
    : (
      /** Add askContractToMintNft Action for the onClick event **/
      <button onClick={askContractToMintNft} className="cta-button connect-wallet-button">
        Mint NFT
      </button>
    )
  }
);
```

* 复制 artifacts/contracts/MyEpicNFT.sol/MyEpicNFT.json 内容，然后在 buildspace-nft-course-starter/src 文件夹下创建一个 utils 文件夹，并在 utils 文件夹下创建 MyepicNFT.json 文件，内容为复制的内容，路径看起来像 src/utils/MyEpicNFT.json

* 在 App.js 顶部加入

```js
import myEpicNft from './utils/MyEpicNFT.json';
```

* 为了添加铸造完成时 opensea 链接，并提醒用户切换到rinkeby，将 App.js 替换为

```js
import './styles/App.css';
import twitterLogo from './assets/twitter-logo.svg';
import { ethers } from "ethers";
import React, { useEffect, useState } from "react";
import myEpicNft from './utils/MyEpicNFT.json';

const TWITTER_HANDLE = '_buildspace';
const TWITTER_LINK = `https://twitter.com/${TWITTER_HANDLE}`;
const OPENSEA_LINK = '';
const TOTAL_MINT_COUNT = 50;

// 替换为你的合约地址
const CONTRACT_ADDRESS = "0xF1aD06077E05ebD0e0c0e8eBC104fE436c560D6F";

const App = () => {

    const [currentAccount, setCurrentAccount] = useState("");
    
    const checkIfWalletIsConnected = async () => {
      const { ethereum } = window;

      if (!ethereum) {
          console.log("Make sure you have metamask!");
          return;
      } else {
          console.log("We have the ethereum object", ethereum);
      }
        
    let chainId = await ethereum.request({ method: 'eth_chainId' });
    console.log("Connected to chain " + chainId);

    // String, hex code of the chainId of the Rinkebey test network
    const rinkebyChainId = "0x4"; 
    if (chainId !== rinkebyChainId) {
        alert("You are not connected to the Rinkeby Test Network!");
    }

      const accounts = await ethereum.request({ method: 'eth_accounts' });

      if (accounts.length !== 0) {
          const account = accounts[0];
          console.log("Found an authorized account:", account);
					setCurrentAccount(account)
          
          // Setup listener! This is for the case where a user comes to our site
          // and ALREADY had their wallet connected + authorized.
          setupEventListener()
      } else {
          console.log("No authorized account found")
      }
  }

  const connectWallet = async () => {
    try {
      const { ethereum } = window;

      if (!ethereum) {
        alert("Get MetaMask!");
        return;
      }

      const accounts = await ethereum.request({ method: "eth_requestAccounts" });

      console.log("Connected", accounts[0]);
      setCurrentAccount(accounts[0]);

      // Setup listener! This is for the case where a user comes to our site
      // and connected their wallet for the first time.
      setupEventListener() 
    } catch (error) {
      console.log(error)
    }
  }

  // Setup our listener.
  const setupEventListener = async () => {
    // Most of this looks the same as our function askContractToMintNft
    try {
      const { ethereum } = window;

      if (ethereum) {
        // Same stuff again
        const provider = new ethers.providers.Web3Provider(ethereum);
        const signer = provider.getSigner();
        const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, myEpicNft.abi, signer);

        // THIS IS THE MAGIC SAUCE.
        // This will essentially "capture" our event when our contract throws it.
        // If you're familiar with webhooks, it's very similar to that!
        connectedContract.on("NewEpicNFTMinted", (from, tokenId) => {
          console.log(from, tokenId.toNumber())
          alert(`Hey there! We've minted your NFT and sent it to your wallet. It may be blank right now. It can take a max of 10 min to show up on OpenSea. Here's the link: https://testnets.opensea.io/assets/${CONTRACT_ADDRESS}/${tokenId.toNumber()}`)
        });

        console.log("Setup event listener!")

      } else {
        console.log("Ethereum object doesn't exist!");
      }
    } catch (error) {
      console.log(error)
    }
  }

  const askContractToMintNft = async () => {
    try {
      const { ethereum } = window;

      if (ethereum) {
        const provider = new ethers.providers.Web3Provider(ethereum);
        const signer = provider.getSigner();
        const connectedContract = new ethers.Contract(CONTRACT_ADDRESS, myEpicNft.abi, signer);

        console.log("Going to pop wallet now to pay gas...")
        let nftTxn = await connectedContract.makeAnEpicNFT();

        console.log("Mining...please wait.")
        await nftTxn.wait();
        console.log(nftTxn);
        console.log(`Mined, see transaction: https://rinkeby.etherscan.io/tx/${nftTxn.hash}`);

      } else {
        console.log("Ethereum object doesn't exist!");
      }
    } catch (error) {
      console.log(error)
    }
  }


  useEffect(() => {
    checkIfWalletIsConnected();
  }, [])

  const renderNotConnectedContainer = () => (
    <button onClick={connectWallet} className="cta-button connect-wallet-button">
      Connect to Wallet
    </button>
  );

  const renderMintUI = () => (
    <button onClick={askContractToMintNft} className="cta-button connect-wallet-button">
      Mint NFT
    </button>
  )

  return (
    <div className="App">
      <div className="container">
        <div className="header-container">
          <p className="header gradient-text">My NFT Collection</p>
          <p className="sub-text">
            Each unique. Each beautiful. Discover your NFT today.
          </p>
          {currentAccount === "" ? renderNotConnectedContainer() : renderMintUI()}
        </div>
        <div className="footer-container">
          <img alt="Twitter Logo" className="twitter-logo" src={twitterLogo} />
          <a
            className="footer-text"
            href={TWITTER_LINK}
            target="_blank"
            rel="noreferrer"
          >{`built on @${TWITTER_HANDLE}`}</a>
        </div>
      </div>
    </div>
  );
};

export default App;
```

## Luanch

```
cd buildspace-nft-course-starter
npm install
npm run start
```

打开左边的端口转发，就可以看到你的 mint 页面了!

如果想保存到本地，点击 EXPLORER 下的下载键就可以了, 上传到自己的 github 也是不错的选择。

Be happy!

