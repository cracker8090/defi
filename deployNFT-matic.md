仪表板：https://nightly.thirdweb.com/dashboard

SDK:npm install @thirdweb-dev/sdk@nightly

https://blog.thirdweb.com/thirdweb-v2 

Javascript初级教程

https://www.w3school.com.cn/web/web_javascript.asp

https://www.php.cn/course/18.html

https://www.runoob.com/js/js-tutorial.html 



https://www.pointer.gg/tutorials

https://thirdweb.com/ 

合约地址：0x4Aa9843a493aA709d29d2cCc73e0fb8F35284E1D

- Smart contracts that they deploy for us: https://github.com/nftlabs/nftlabs-protocols
- Typescript SDK: https://github.com/nftlabs/nftlabs-sdk-ts

https://discord.gg/thirdweb

Polygon Mumbai network 获取测试币matic  https://faucet.polygon.technology

# 操作

 git clone https://github.com/pointer-gg/thirdweb-lootbox-tutorial

内部安装依赖：npm install

npm run dev后localhost:3000



 [the thirdweb dashboard](https://thirdweb.com/dashboard?utm_source=pointergg&utm_medium=article&utm_campaign=lootbox_tutorial) 创建项目，Polygon Mumbai，会看到一个合约地址， [mumbai.polygonscan.com](http://mumbai.polygonscan.com) 

创建 .env  ,添加WALLET_PRIVATE_KEY=...。

安装依赖：npm install @3rdweb/sdk ethers dotenv

## 创建新脚本scripts/helpers.js

```
import { ThirdwebSDK } from "@3rdweb/sdk"; // thirdweb SDK
import ethers from "ethers"; // wallet instance

// Read environment variables from .env 跨平台
import dotenv from "dotenv";
dotenv.config();

const walletPrivateKey = process.env.WALLET_PRIVATE_KEY;

if (!walletPrivateKey) {
  console.error("Wallet private key missing")
  process.exit(1)
}

export const sdk = new ThirdwebSDK(
  new ethers.Wallet(
    // Wallet private key. NEVER CHECK THE KEY IN. ALWAYS USE ENVIRONMENT VARIABLES.
    process.env.WALLET_PRIVATE_KEY,
    // We use Polygon Mumbai network
    ethers.getDefaultProvider("https://winter-icy-sun.matic-testnet.quiknode.pro/f36aa318f8f806e4e15a58ab4a1b6cb9f9e9d9b9/")
  ),
);

const appAddress = '0x...'; // your project address from thirdweb

export async function getApp() {
  const app = await sdk.getAppModule(appAddress);
  return app;
}

```



## 创建scripts/1-create-bundle-module.js

```
import { getApp } from './helpers.js';

async function main() {
  const app = await getApp();

  console.log('Deploying bundle collection module...');

  const bundleModule = await app.deployBundleModule({
    name: 'Lootbox Bundle',
    sellerFeeBasisPoints: 0, // 100意思就是1%费用
  });

  console.log(`Deployed bundle collection module with address ${bundleModule.address}`);
}
// 每个脚本都重复的东西
try {
	await main();
} catch (error) {
  console.error("Error creating the bundle collection module", error);
  process.exit(1);
}
```

Deployed bundle collection module with address 0xB3789Cdd6BD8e12fA5995aD04392222e91e0757B

# mint NFT

将调用我们刚创建的那个bundle集合契约，并要求它为我们创建一些nft。

图片之类，至少两个，这里选择汽车图片，三个。 [unsplash.com](http://unsplash.com) 。

## 创建scripts/2-mint-bundle-nfts.js

```
import { readFileSync } from 'fs';
import { sdk } from './helpers.js';

async function main() {
  // Paste in the address from when you created the bundle collection module
  const bundleModuleAddress = '0x...'; // 这里就是上面bundle的地址
  const bundleModule = sdk.getBundleModule(bundleModuleAddress); // 用sdk.getXXXModule一个存在的合约（类型）

  console.log('Creating NFT batch...');

  // 数据去创建NFT metadata数据，supply数量
  const created = await bundleModule.createAndMintBatch([
    {
      metadata: {
        name: 'Tesla Model 3',
        description: 'A pretty fancy car!',
        image: readFileSync('./assets/tesla-model3.jpg'),
        properties: {
          rarity: 'a bit rare',
          fanciness: 7,
        }
      },
      supply: 50,
    },
    {
      metadata: {
        name: 'Porsche 911',
        description: 'A pretty fancy car!',
        image: readFileSync('./assets/porsche-911.jpg'),
        properties: {
          rarity: 'a bit rare',
          fanciness: 7,
        }
      },
      supply: 50,
    },
    {
      metadata: {
        name: 'Mclaren P1',
        description: 'A super fancy car!',
        image: readFileSync('./assets/mclaren-p1.jpg'),
        properties: {
          rarity: 'super rare!',
          fanciness: 10,
        }
      },
      supply: 10,
    }
  ]);

  console.log('NFTs created!')
  console.log(JSON.stringify(created, null, 2));
}

try {
  await main();
} catch (error) {
  console.error("Error minting the NFTs", error);
  process.exit(1);
}cd
```

node scripts/2-mint-bundle-nfts.js



# Packing NFTs

## scripts/3-create-pack-module.js

```
import { getApp } from './helpers.js';

async function main() {
  const app = await getApp();

  console.log('Deploying pack module...');

  const packModule = await app.deployPackModule({
    name: 'Lootbox Pack',
    sellerFeeBasisPoints: 0,
  });

  console.log(`Deployed pack module with address ${packModule.address}`);
}

try {
  await main();
} catch (error) {
  console.error("Error creating the pack module", error);
  process.exit(1);
}
```

0x3a1bdF1142A74b7a1CAd9eaa522e4827Ca4a2dd0

## From bundle to pack

scripts/4-create-pack-from-bundle.js

nft集合变成一个华丽的战利品盒。当我们在thirdweb中创建一个包时，我们必须告诉它从哪个包获取nft(也就是我们的bundle模块地址)，以及使用哪个nft。所以我们可以选择使用其中的一个子集或者通过改变供应来改变稀缺性

```
import { readFileSync } from 'fs';
import { sdk } from './helpers.js';

async function main() {
  const bundleModuleAddress = '0xB3789Cdd6BD8e12fA5995aD04392222e91e0757B'; // your bundle module address
  const bundleModule = sdk.getBundleModule(bundleModuleAddress);

  const packModuleAddress = '0x3a1bdF1142A74b7a1CAd9eaa522e4827Ca4a2dd0'; // your pack module address
  const packModule = sdk.getPackModule(packModuleAddress);

  console.log('Getting all NFTs from bundle...');
  const nftsInBundle = await bundleModule.getAll();

  console.log('NFTs in bundle:');
  console.log(nftsInBundle);

  console.log('Creating a pack containing the NFTs from bundle...');
  const created = await packModule.create({
    assetContract: bundleModuleAddress,
    metadata: {
      name: 'Fancy Cars Pack!',
      image: readFileSync('./assets/fancy-cars.jpeg'),
    },
    assets: nftsInBundle.map(nft => ({
      tokenId: nft.metadata.id,
      amount: nft.supply,
    })),
  });

  console.log('Pack created!')
  console.log(created);
}

try {
  await main();
} catch (error) {
  console.error("Error minting the NFTs", error);
  process.exit(1);
}
```



# Opening a pack

如果您已经玩过区块链开发，那么您可能会遇到生成随机数的困难。智能合约是确定性的，不能生成适当的随机数。我们需要使用一个助手为我们提供随机性，这样我们的包打开是不可预测的，不能被操纵的。

Thirdweb使用Chainlink来实现这一点，这是一个非常流行的解决方案。

## scripts/5-deposit-link.js

```
import { ethers } from "ethers";
import { sdk } from "./helpers.js";

async function main() {
  const packModuleAddress = '0x3a1bdF1142A74b7a1CAd9eaa522e4827Ca4a2dd0'; // your pack module address
  const packModule = sdk.getPackModule(packModuleAddress);

  console.log('Depositing link...')

  // LINK uses 18 decimals, same as Eth. So this gives us the amount to use for 2 LINK
  const amount = ethers.utils.parseEther('2');

  await packModule.depositLink(amount);
  console.log('Deposited!')

  const balance = await packModule.getLinkBalance();
  console.log(balance);
}

try {
  await main();
} catch (error) {
  console.error("Error depositing the LINK", error);
  process.exit(1);
}
```



## scripts/6-open-pack.js

这里的0是我们要打开的包的ID。我们在模块中只创建了一个包，但是可以有多个——就像我们的NFT bundle有许多NFT一样。当我们创建包时，ID就打印出来了

```
import { sdk } from "./helpers.js";

async function main() {
  const packModuleAddress = '0x3a1bdF1142A74b7a1CAd9eaa522e4827Ca4a2dd0';
  const packModule = sdk.getPackModule(packModuleAddress);

  console.log('Opening the pack...');
  const opened = await packModule.open('0');
  console.log('Opened the pack!');
  console.log(opened);
}

try {
  await main();
} catch (error) {
  console.error("Error opening the pack", error);
  process.exit(1);
}
```

如果愿意，可以多次调用脚本来收集不同的nft







npm install @3rdweb/react

npm install @3rdweb/hooks

export const bundleAddress = '0xB3789Cdd6BD8e12fA5995aD04392222e91e0757B';
export const packAddress = '0x3a1bdF1142A74b7a1CAd9eaa522e4827Ca4a2dd0';



# 安装Fly CLI

curl -L https://fly.io/install.sh | sh

export FLYCTL_INSTALL="/root/.fly"
export PATH="$FLYCTL_INSTALL/bin:$PATH"

```
flyctl auth signup
```













