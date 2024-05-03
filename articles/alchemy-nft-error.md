---
title: "Alchemy NFT API で Undefined が出た時の対処法"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nft", "error", "alchemy"]
published: true
---

## はじめに

[Alchemy NFT API の QuickStart](https://docs.alchemy.com/reference/nft-api-quickstart) を試していたら、エラーが出たのでその対処法をメモします。

## 環境

```
PC : M1 MacBook Pro (Apple M1)
yarn: 1.22.19
```

## ソースコード

```javascript
// This script demonstrates access to the NFT API via the Alchemy SDK.
import { Network, Alchemy } from "alchemy-sdk";

// Optional Config object, but defaults to demo api-key and eth-mainnet.
const settings = {
    apiKey: "Rf822Ol9MVHIp8W2S4SOb-iIJYWayVKW", // Replace with your Alchemy API Key.
    network: Network.ETH_MAINNET, // Replace with your network.
};

const alchemy = new Alchemy(settings);

// vitalikのNFTを取得する工程がありますが、今回関係ないので省略します

// Fetch metadata for a particular NFT:
console.log("fetching metadata for a Crypto Coven NFT...");
const response = await alchemy.nft.getNftMetadata(
  "0x5180db8F5c931aaE63c74266b211F580155ecac8",
  "1590"
);

// Uncomment this line to see the full api response:
// console.log(response);

// Print some commonly used fields:
console.log("NFT name: ", response.title);
console.log("token type: ", response.tokenType);
console.log("tokenUri: ", response.tokenUri.gateway);
console.log("image url: ", response.rawMetadata.image);
console.log("time last updated: ", response.timeLastUpdated);
console.log("===");

```


## エラー内容

```
$ node alchemy-sdk-script.js

fetching metadata for a Crypto Coven NFT...
NFT name:  undefined
token type:  ERC721
tokenUri:  undefined
undefined
file:///Users/mio256/alchemy-nft-api/alchemy-sdk-script.js:45
console.log("image url: ", response.rawMetadata.image);
                                                ^

TypeError: Cannot read properties of undefined (reading 'image')
    at file:///Users/mio256/alchemy-nft-api/alchemy-sdk-script.js:45:49
    at process.processTicksAndRejections (node:internal/process/task_queues:95:5)

Node.js v18.18.2
FAIL
```

## 解決法

エラー内容を見ると、`response.rawMetadata.image` が `undefined` となっていることが原因であることがわかります。

コメントを外してResponseを確認すると、Jsonが破壊的変更されていることがわかります。

```json
{
  contract: {
    address: '0x5180db8F5c931aaE63c74266b211F580155ecac8',
    name: 'Crypto Coven',
    symbol: 'WITCH',
    totalSupply: undefined,
    tokenType: 'ERC721',
    contractDeployer: '0xac9d54ca08740A608B6C474e5CA07d51cA8117Fa',
    deployedBlockNumber: 13547115,
    openSeaMetadata: {
      floorPrice: 0.079498,
      collectionName: 'Crypto Coven',
      collectionSlug: 'cryptocoven',
      safelistRequestStatus: 'verified',
      imageUrl: 'https://i.seadn.io/gae/E8MVasG7noxC0Fa_duhnexc2xze1PzT1jzyeaHsytOC4722C2Zeo7EhUR8-T6mSem9-4XE5ylrCtoAsceZ_lXez_kTaMufV5pfLc3Fk?w=500&auto=format',
      description: "it's the season of the witch. ????",
      externalUrl: undefined,
      twitterUsername: 'crypto_coven',
      discordUrl: 'https://discord.gg/cryptocoven',
      bannerImageUrl: 'https://i.seadn.io/gae/M42Xf9Vbu_yodzKVFA1I6TYXIx5Hz699gEtp2lDg9vGT7g-S4z_5cx2iYPub1kytnOlexV5WDdGOmpGeuH4-N0CYXi7FaC_iqEm4gQ?w=500&auto=format',
      lastIngestedAt: '2024-05-02T06:09:34.000Z'
    },
    isSpam: undefined,
    spamClassifications: []
  },
  tokenId: '1590',
  tokenType: 'ERC721',
  name: 'balsa vault',
  description: 'You are a WITCH with eyes that hold eons. You write poems filled with charms. Your magic spawns from a few hours of sleep. You arch your back into a bridge between the living and the dead. SHINE!',
  tokenUri: 'https://alchemy.mypinata.cloud/ipfs/QmaXzZhcYnsisuue5WRdQDH6FDvqkLQX1NckLqBYeYYEfm/1590.json',
  image: {
    cachedUrl: 'https://nft-cdn.alchemy.com/eth-mainnet/d19db83206d00d210696cba9be86f754',
    thumbnailUrl: 'https://res.cloudinary.com/alchemyapi/image/upload/thumbnailv2/eth-mainnet/d19db83206d00d210696cba9be86f754',
    pngUrl: 'https://res.cloudinary.com/alchemyapi/image/upload/convert-png/eth-mainnet/d19db83206d00d210696cba9be86f754',
    contentType: 'image/png',
    size: 2724759,
    originalUrl: 'https://cryptocoven.s3.amazonaws.com/a7875f5758f85544dcaab79a8a1ca406.png'
  },
  raw: {
    tokenUri: 'ipfs://QmaXzZhcYnsisuue5WRdQDH6FDvqkLQX1NckLqBYeYYEfm/1590.json',
    metadata: {
      image: 'https://cryptocoven.s3.amazonaws.com/a7875f5758f85544dcaab79a8a1ca406.png',
      external_url: 'https://www.cryptocoven.xyz/witches/1590',
      background_color: '',
      coven: [Object],
      name: 'balsa vault',
      description: 'You are a WITCH with eyes that hold eons. You write poems filled with charms. Your magic spawns from a few hours of sleep. You arch your back into a bridge between the living and the dead. SHINE!',
      attributes: [Array]
    },
    error: undefined
  },
  collection: {
    name: 'Crypto Coven',
    slug: 'cryptocoven',
    externalUrl: undefined,
    bannerImageUrl: 'https://i.seadn.io/gae/M42Xf9Vbu_yodzKVFA1I6TYXIx5Hz699gEtp2lDg9vGT7g-S4z_5cx2iYPub1kytnOlexV5WDdGOmpGeuH4-N0CYXi7FaC_iqEm4gQ?w=500&auto=format'
  },
  mint: {
    mintAddress: undefined,
    blockNumber: undefined,
    timestamp: undefined,
    transactionHash: undefined
  },
  owners: undefined,
  timeLastUpdated: '2024-03-05T16:01:31.108Z',
  acquiredAt: undefined
}
```

以下のように変更します。

```javascript
// Print some commonly used fields:
console.log("NFT name: ", response.name);
console.log("token type: ", response.tokenType);
console.log("tokenUri: ", response.raw.tokenUri);
console.log("image url: ", response.raw.metadata.image);
console.log("time last updated: ", response.timeLastUpdated);
```
