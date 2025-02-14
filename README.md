# TangleCon Workshop 2023
## Teaching material for my TangleCon workshop on how to build a simple NFT collection on IOTA's ShimmerEVM testnet 

### Prerequisites

1. Download the Brave browser from: https://brave.com/
2. Download and set up MetaMask from: https://metamask.io/. If you need guidance during the setup, check out: https://mirror.xyz/marcobesier.eth/Ry7z4cug6EBI1ptJxZqQEmCNR_zD-IVXxRmPFyG2cGY
3. Register the ShimmerEVM testnet inside your MetaMask and fund your wallet using the public faucet. You can find the connection details and the link to the faucet in the following blog post: https://blog.shimmer.network/shimmerevm-testnet-launch/


### Artwork Creation

1. Go to https://labs.openai.com to open DALL-E 2 and write a prompt or click "Surprise me" to generate 4 images.
2. Download the images, store them in a folder on your Desktop called "MyNFTAssets", and name them: `1.png`, `2.png`, `3.png`, `4.png`
3. Sign up for Pinata: https://www.pinata.cloud/
4. After logging in to Pinata, create a new public folder by selecting Files/Public (should be the case by default) and then clicking the "+ Add Files" button and choosing the "Folder" option. **IMPORTANT:** If you’re on a Mac, your folder might contain a hidden `.DS_STORE` file. You may want to remove that before uploading the folder. Give your Pinata folder the name "MyNFTAssets" and click "Upload".
5. Next, let’s create the metadata for our NFTs, by following the standard that’s described here: https://docs.opensea.io/docs/metadata-standards#metadata-structure For the sake of simplicity, we’ll only include the "name", "description", and "image" keys in our metadata. Create a json file for each of your images using the text editor of your choice (or something like https://jsoneditoronline.org/), and store them in a folder on your desktop called "MyNFTMetadata", naming them `1.json`, `2.json`, `3.json`, and `4.json`, respectively. Here’s an example of what the first metadata file could look like:
```
{
  "name": "Aja",
  "description": "Aja Trading Card", 
  "image": "ipfs://QmXpNUXACZeZ7cdJPCmXhmsRPXc8Uc3nPzbM7RA3svzVYc/1.png"
}
```
6. Next, rename all the metadata files from `1.json`, `2.json`, etc. to `1`, `2`, etc. (i.e., remove the file extension). (We can’t get into the reason why you need to do that just yet, but we actually need this renaming for our smart contract code to work properly.)
7. Finally, upload this folder to Pinata as well like you did for the "MyNFTAssets" folder.


### Contract Creation

1. Head over to https://remix.ethereum.org.
2. Create a new file in the "contracts" directory called "MyNFT.sol".
3. Add an SPDX License Identifier to the file. (If you don't know what that is, check out: https://docs.soliditylang.org/en/latest/layout-of-source-files.html#spdx-license-identifier)
```
// SPDX-License-Identifier: UNLICENSE
```
4. Add the version pragma. (If you don't know what that is, check out: https://docs.soliditylang.org/en/latest/layout-of-source-files.html#pragmas)
```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;
```
5. Import OpenZeppelin's ERC-721 implementation:
```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
```
6. In case you've never heard of ERC-721 tokens before, here's the original proposal: https://eips.ethereum.org/EIPS/eip-721
7. In case you've never seen OpenZeppelin's ERC-721 implementation, you can check it out on their GitHub repo: https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC721/ERC721.sol
8. Declare the main contract and inherit functionality from OpenZeppelin's implementation:
```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    
}
```
9. When trying to compile this contract, we get an error, because OpenZeppelin’s implementation requires us to override the constructor to be able to specify the name and symbol of our NFT collection during deployment.
```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    
    constructor() ERC721("MyNFT", "MN"){
        
    }
}
```
10. Now the compilation should work. Next, let’s add the mint functionality by using the `_mint` function from OpenZeppelin's contract. Looking at OpenZeppelin’s implementation, we need to specify the `to` address and the `tokenId` that our function should mint.
```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {
    
    constructor() ERC721("MyNFT", "MN"){

    }

    function mint() public {
        _mint(msg.sender, tokenId)
    } 
}
```
11. But if we try to compile, it won’t work because we haven’t specified `tokenId`. Now, the classic way of handling this is to specify the token supply manually.
```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {

    uint256 public tokenSupply = 0;
    
    constructor() ERC721("MyNFT", "MN"){

    }

    function mint() public {
        _mint(msg.sender, tokenId);
    } 
}
```
12. Now we can use the `tokenSupply + 1` as the `tokenId` during the mint, and increment the `tokenSupply` variable by 1 after each mint:
```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {

    uint256 public tokenSupply = 0;
    
    constructor() ERC721("MyNFT", "MN"){

    }

    function mint() public {
        _mint(msg.sender, tokenSupply + 1);
        tokenSupply++;
    } 
}
```
13. Okay, that’s pretty cool already, but let’s make this a little more sophisticated. Because one thing we typically wanna do is still missing here: Generally we want to limit the supply of NFTs! Fortunately, limiting the supply is pretty easy:
 ```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {

    uint256 public tokenSupply = 0;
    uint256 public constant MAX_SUPPLY = 4;
    
    constructor() ERC721("MyNFT", "MN"){

    }

    function mint() public {
        require(tokenSupply < MAX_SUPPLY, "supply used up");
        _mint(msg.sender, tokenSupply + 1);
        tokenSupply++;
    } 
}
```
14. Now, the final thing we have to do is to override OpenZeppelin’s `_baseURI` function so that `tokenURI` actually returns the correct URI for each token. **IMPORTANT:** Don’t forget to include the trailing slash (`/`) at the end of the base URI. Also, make sure to use the CID of the _metadata_, not the one of the assets.
 ```
// SPDX-License-Identifier: UNLICENSE
pragma solidity 0.8.19;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";

contract MyNFT is ERC721 {

    uint256 public tokenSupply = 0;
    uint256 public constant MAX_SUPPLY = 4;
    
    constructor() ERC721("MyNFT", "MN"){

    }

    function mint() public {
        require(tokenSupply < MAX_SUPPLY, "supply used up");
        _mint(msg.sender, tokenSupply + 1);
        tokenSupply++;
    }

    function _baseURI() internal pure override returns (string memory) {
        return "ipfs://QmQBuk8qe2DKYtVteuqYWFy6TcbtT6fMYrQbyNVvNrx69h/";
    }
}
```


### Deployment and Testing

1. Open your MetaMask and ensure you selected the ShimmerEVM testnet as well as the account holding your SMR test funds.
2. In Remix, compile your contract by going to the "Solidity compiler" tab in the sidebar.
3. Then, deploy your contract by going to the "Deploy & run transactions" tab. Confirm the MetaMask popup. After a few seconds, your contract should be successfully deployed.
4. Save your contract’s address (you can find it right below the "Deployed contracts" section) in a text file for later reference.
5. Next, execute the mint function by expanding your deployed contract’s dropdown in Remix and clicking the "mint" button.
6. Finally, you can now import that NFT into MetaMask using MetaMask's import feature and specifying your deployed contract's address and the `tokenId` 0.
